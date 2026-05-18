---
package:    core
type:       entity
name:       authorization_role
version:    1.0.0
status:     draft
tags:       [authorization, access-control, core]
---

# Authorization Role (Entity)

## Summary

An `Authorization Role` is a named permission entry in the system's role catalog. It defines what action a user may perform within a tenant context. Role definitions are created exclusively at application startup and are immutable for the lifetime of the system. They are global — not scoped to any workspace or tenant.

---

## Identity

A role is uniquely identified by its `id`. Its `name` is also globally unique and serves as the human-readable identifier used by both operators and the application logic when seeding and resolving roles.

| Key | Type | Role |
|---|---|---|
| `id` | UUID | Internal primary key — used by all assignment references |
| `name` | string | Global natural key — used for role resolution and seeding |

---

## Attributes

```yaml
id:
  type:        UUID
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Internal primary identifier for the role

name:
  type:        string
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Unique role identifier following the pattern {package}_{domain}_{action}, e.g. core_user_reader

is_assignable:
  type:        boolean
  source:      system-generated
  constraints: [required, immutable, default: true]
  description: Whether this role can be assigned to users via the assignment API; false marks roles reserved for internal use

created_at:
  type:        timestamp
  source:      system-generated
  constraints: [required, immutable]
  description: Timestamp when the role was created at application startup
```

---

## Lifecycle

Authorization role definitions have no state transitions. They are created at application startup and remain unchanged for the lifetime of the system. There is no API to create, update, or delete role definitions.

| State | Meaning |
|---|---|
| Active (only state) | Role exists in the catalog and may be assigned (if `is_assignable = true`) |

---

## Invariants

These rules must always hold true for an authorization role entity:

1. `id` is system-generated and immutable. It is set at application startup and never changes.
2. `name` is globally unique across all roles and immutable after creation.
3. `is_assignable` is immutable after creation. It cannot be changed via the API.
4. `created_at` is set once at startup and never changes.
5. No API endpoint exists to create, update, or delete a role definition. Management is exclusively through application startup code.
6. Deletion of a role definition is blocked if any `authorization_role_assignment` record references it.

---

## Behavior

Authorization role definitions have no self-contained behavior. They are read-only catalog entries consumed by the assignment system. All mutating operations (creation, deletion) are performed exclusively at the infrastructure level during application startup.

---

## References
- [core/concept-authorization_role](core/concept-authorization_role.md)