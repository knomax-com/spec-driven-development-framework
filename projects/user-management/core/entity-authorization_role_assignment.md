---
package:    core
type:       entity
name:       authorization_role_assignment
version:    1.0.0
status:     draft
tags:       [authorization, access-control, core]
---

# Authorization Role Assignment (Entity)

## Summary

An `Authorization Role Assignment` is the runtime record that grants a specific role to a specific user within a specific tenant. It is the concrete link between the static role catalog and the users who hold those roles. Assignments are immutable once created — to change a user's roles, the existing assignment must be deleted and a new one created.

---

## Identity

An assignment is uniquely identified by its `id`. The natural key is the combination of `(tenant_id, user_id, authorization_role_id)` — the same role cannot be assigned twice to the same user in the same tenant.

| Key | Type | Role |
|---|---|---|
| `id` | UUID | Internal primary key |
| `tenant_id` + `user_id` + `authorization_role_id` | UUID + UUID + UUID | Natural key — enforces uniqueness of the grant |

---

## Attributes

```yaml
id:
  type:        UUID
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Internal primary identifier for the assignment

tenant_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references tenant.id]
  description: The tenant context in which the role applies

user_id:
  type:        UUID
  source:      caller-provided
  constraints: [required, immutable, references user.id]
  description: The user receiving the role grant

authorization_role_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references authorization_role.id]
  description: The role being granted; resolved from role name at assignment time

created_at:
  type:        timestamp
  source:      system-generated
  constraints: [required, immutable]
  description: Timestamp when the assignment was created

updated_at:
  type:        timestamp
  source:      system-generated
  constraints: [required]
  description: Timestamp of the last update; set to creation time initially

updated_by:
  type:        UUID
  source:      user-provided
  constraints: [required, references user.id]
  description: ID of the user who last modified this record
```

---

## Lifecycle

```
(none) ──── create ────► Assigned ──── delete ────► (none)
```

Assignments have no intermediate states. An assignment either exists (granted) or does not (revoked). There is no update operation — updating is modeled as a delete followed by a create.

---

## Invariants

These rules must always hold true for an authorization role assignment entity:

1. `id` is system-generated and immutable. It is never reused after deletion.
2. The combination `(tenant_id, user_id, authorization_role_id)` must be unique — the same role cannot be granted twice to the same user in the same tenant.
3. `authorization_role_id` must reference a role where `is_assignable = true`. Roles with `is_assignable = false` cannot be assigned via the API.
4. The referenced `user_id` must be an active user at the time of assignment creation.
5. `created_at` is set once at creation and never changes.
6. `updated_by` must reference an existing, valid user `id` at the time of the write.
7. Deletion of the referenced `tenant`, `user`, or `authorization_role` is blocked while any assignment record references them.
8. Assignments cannot be updated in place. Any change to the set of roles a user holds in a tenant requires deleting the relevant assignment and creating a new one.
9. The workspace for an assignment is always derived through the tenant: `tenant.workspace_id`. There is no direct `workspace_id` column on this entity.

---

## References
- [core/concept-authorization_role](core/concept-authorization_role.md)
- [core/entity-authorization_role](core/entity-authorization_role.md)
- [core/entity-tenant](core/entity-tenant.md)
- [core/entity-user](core/entity-user.md)