---
package:    core
type:       entity
name:       user
version:    1.1.0
status:     draft
tags:       [identity, core]
---

# User (Entity)

## Summary

A `User` is the domain object representing a person registered in a workspace. It holds the identity record that all other objects reference when tracking authorship or scoping access.

---

## Identity

A user is uniquely identified within a workspace by the combination of `workspace_id` + `external_id`. Internally, all references use the system-generated `id`.

| Key | Type | Role |
|---|---|---|
| `id` | UUID | Internal primary key — used by all internal references |
| `workspace_id` + `external_id` | UUID + string | Natural key at the API boundary — resolves to `id` on entry |

---

## Attributes

```yaml
workspace_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references workspace.id]
  description: The workspace this user belongs to

id:
  type:        UUID
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Internal primary identifier for the user

external_id:
  type:        string
  source:      system-resolved
  constraints: [required, immutable, unique within workspace (workspace_id + external_id)]
  description: External identifier from the authentication provider, resolved from the JWT on entry

is_active:
  type:        boolean
  source:      system-generated
  constraints: [required, default: true]
  description: Whether the user account is active

created_at:
  type:        timestamp
  source:      system-generated
  constraints: [required, immutable]
  description: Timestamp when the user was created

updated_at:
  type:        timestamp
  source:      system-generated
  constraints: [required]
  description: Timestamp of the last update; set to creation time initially and updated on every change

updated_by:
  type:        UUID
  source:      user-provided
  constraints: [required, references user.id]
  description: ID of the user who last modified this record
```

---

## Lifecycle

```
Active ──── deactivate ────► Inactive
  ▲                              │
  └──────── reactivate ──────────┘
```

| State | `is_active` | Meaning |
|---|---|---|
| Active | `true` | User can authenticate and interact with the system |
| Inactive | `false` | User record exists but cannot be used for authentication or system actions |

Valid transitions:
- `Active → Inactive`: explicit deactivation
- `Inactive → Active`: explicit reactivation

No other states exist for a user.

---

## Invariants

These rules must always hold true for a user entity, regardless of who or what triggers a change:

1. A user always belongs to exactly one workspace. The `workspace_id` is set on creation and never changes.
2. `external_id` is immutable after creation. The combination `(workspace_id, external_id)` must be unique across all users in the system.
3. `id` is system-generated and immutable. It is never reused after deletion.
4. `updated_by` must reference an existing, valid user `id` at the time of the write. Self-reference is permitted (a user may be recorded as the updater of their own record).
5. `created_at` is set once at creation and never changes.
6. Deletion of a user is blocked if any of the following conditions are true:
   - The user has any `user_tenant` membership records.
   - The user has any `Authorization Role` assignment records.
   - The user's `id` appears in any `updated_by` reference in the system (tenants, memberships, role assignments, ...).
7. The `default_for_user` reference on a `Tenant` pointing to this user does not block deletion — it is set to `null` automatically when the user is removed.

---

## Behavior

**Activate**
Transitions `is_active` from `false` to `true`. No effect if the user is already active.

**Deactivate**
Transitions `is_active` from `true` to `false`. No effect if the user is already inactive.

---

## References
- [core/concept-user](core/concept-user.md)
