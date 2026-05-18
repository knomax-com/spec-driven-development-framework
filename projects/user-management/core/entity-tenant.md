---
package:    core
type:       entity
name:       tenant
version:    1.0.0
status:     draft
tags:       [identity, core, multi-tenancy]
---

# Tenant (Entity)

## Summary

A `Tenant` is the domain object representing a soft data separation group within a workspace. It groups one or more users and owns all data objects associated with it. Every user always has at least one tenant — their default tenant.

---

## Identity

A tenant is uniquely identified within the system by its `id`. Within a workspace, the combination of `workspace_id` + `name` is a natural key used for human-readable lookups.

| Key | Type | Role |
|---|---|---|
| `id` | UUID | Internal primary key — used by all internal references |
| `workspace_id` + `name` | UUID + string | Natural key within the workspace for human-readable resolution |

---

## Attributes

```yaml
workspace_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references workspace.id]
  description: The workspace this tenant belongs to

id:
  type:        UUID
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Internal primary identifier for the tenant

name:
  type:        string
  source:      user-provided
  constraints: [required, unique within workspace (workspace_id + name)]
  description: Human-readable tenant identifier; for default tenants equals the owner user's id

is_active:
  type:        boolean
  source:      system-generated
  constraints: [required, default: true]
  description: Whether the tenant is active

default_for_user:
  type:        UUID
  source:      system-generated
  constraints: [optional, references user.id, set to null on user deletion]
  description: If set, marks this as the default tenant for the referenced user; set only at creation of the default tenant

created_at:
  type:        timestamp
  source:      system-generated
  constraints: [required, immutable]
  description: Timestamp when the tenant was created

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
| Active | `true` | Tenant is available for data operations and role assignments |
| Inactive | `false` | Tenant record exists but is not available for new operations |

Valid transitions:
- `Active → Inactive`: explicit deactivation
- `Inactive → Active`: explicit reactivation

Deletion is a separate terminal operation — it requires the tenant to be empty first (see Invariants).

---

## Invariants

These rules must always hold true for a tenant entity:

1. `workspace_id` is set at creation and never changes.
2. `id` is system-generated and immutable. It is never reused after deletion.
3. `name` must be unique within the workspace at all times. Two tenants in the same workspace cannot share a name.
4. `created_at` is set once at creation and never changes.
5. `updated_by` must reference an existing, valid user `id` at the time of the write.
6. `default_for_user` is set only when the tenant is created as a user's default tenant. It is null for all non-default tenants. When the referenced user is deleted, `default_for_user` is automatically set to null.
7. A tenant must always have at least one `user_tenant` membership record. A tenant with zero members is an invalid state.
8. Deletion is blocked if any of the following conditions are true:
   - Any `user_tenant` membership record references this tenant.
   - Any `authorization_role_assignment` record references this tenant.
   - Any data object is scoped to this tenant via `tenant_id`.
9. The default tenant (where `default_for_user` is set) cannot be explicitly deleted while the owner user still exists. It is removed only as a consequence of user deletion.

---

## Behavior

**Activate**
Transitions `is_active` from `false` to `true`. No effect if the tenant is already active.

**Deactivate**
Transitions `is_active` from `true` to `false`. No effect if the tenant is already inactive.

**Rename**
Changes `name` to a new value. The new value must not already be in use within the same workspace. Updates `updated_at` and `updated_by`.

---

## References
- [core/concept-tenant](core/concept-tenant.md)
- [core/concept-workspace](core/concept-workspace.md)
- [core/concept-user](core/concept-user.md)