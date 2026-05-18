---
package:    core
type:       entity
name:       user_tenant
version:    1.0.0
status:     draft
tags:       [identity, core, multi-tenancy]
---

# User Tenant (Entity)

## Summary

A `User Tenant` is the membership record that links a user to a tenant within a workspace. It is the explicit join between the `User` and `Tenant` entities. Without a `user_tenant` record, a user has no access to a tenant's data or operations. Every user always has at least one membership record — the one linking them to their default tenant.

---

## Identity

A membership is uniquely identified by its `id`. The natural key is `(user_id, tenant_id)` — a user can only have one membership per tenant.

| Key | Type | Role |
|---|---|---|
| `id` | UUID | Internal primary key |
| `user_id` + `tenant_id` | UUID + UUID | Natural key — enforces that a user is a member of each tenant at most once |

---

## Attributes

```yaml
workspace_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references workspace.id]
  description: The workspace this membership belongs to; derived from the user's workspace context

id:
  type:        UUID
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Internal primary identifier for the membership record

user_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references user.id]
  description: The user who is a member of the tenant

tenant_id:
  type:        UUID
  source:      caller-provided
  constraints: [required, immutable, references tenant.id]
  description: The tenant the user is joining

created_at:
  type:        timestamp
  source:      system-generated
  constraints: [required, immutable]
  description: Timestamp when the membership was created

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
(none) ──── create ────► Member ──── delete ────► (none)
```

Memberships have no intermediate states. A user either is a member of a tenant or is not. There is no update operation — membership attributes are immutable after creation.

---

## Invariants

These rules must always hold true for a user_tenant entity:

1. `id` is system-generated and immutable. It is never reused after deletion.
2. The combination `(user_id, tenant_id)` must be unique — a user cannot hold more than one membership record for the same tenant.
3. `workspace_id`, `user_id`, and `tenant_id` are set at creation and never change.
4. `created_at` is set once at creation and never changes.
5. `updated_by` must reference an existing, valid user `id` at the time of the write.
6. Deletion of the referenced `user`, `tenant`, or `workspace` is blocked while any membership record references them.
7. Removing the last `user_tenant` record from a tenant is blocked — a tenant must always retain at least one member.
8. The default tenant membership (created automatically during user registration) is removed only as part of user deletion; it cannot be explicitly deleted while the user exists.

---

## References
- [core/concept-tenant](core/concept-tenant.md)
- [core/entity-user](core/entity-user.md)
- [core/entity-tenant](core/entity-tenant.md)