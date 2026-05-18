---
package:    core
type:       entity
name:       workspace
version:    1.0.0
status:     draft
tags:       [identity, core]
---

# Workspace (Entity)

## Summary

A `Workspace` is the root domain object that provides hard data isolation for all objects it contains. It is the outermost lifecycle boundary — nothing inside a workspace outlives it, and nothing inside it is accessible from another workspace.

---

## Identity

A workspace is uniquely identified by its `id`. Its `name` also uniquely identifies it across the entire system.

| Key | Type | Role |
|---|---|---|
| `id` | UUID | Internal primary key — used by all internal references |
| `name` | string | Global natural key — used for API routing and JWT resolution |

---

## Attributes

```yaml
id:
  type:        UUID
  source:      system-generated
  constraints: [required, unique, immutable]
  description: Internal primary identifier for the workspace

name:
  type:        string
  source:      user-provided
  constraints: [required, unique]
  description: Human-readable identifier; globally unique across all workspaces; used in JWT routing

is_active:
  type:        boolean
  source:      system-generated
  constraints: [required, default: true]
  description: Whether the workspace is accepting new requests

created_at:
  type:        timestamp
  source:      system-generated
  constraints: [required, immutable]
  description: Timestamp when the workspace was created
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
| Active | `true` | Workspace accepts authentication and API requests |
| Inactive | `false` | Workspace exists but blocks new authentication (existing records unaffected), it cannot be used |

Valid transitions:
- `Active → Inactive`: explicit deactivation
- `Inactive → Active`: explicit reactivation

Deletion is a separate terminal operation — it requires the workspace to be empty first (see Invariants).

---

## Invariants

These rules must always hold true for a workspace entity:

1. `id` is system-generated and immutable. It is never reused after deletion.
2. `name` is globally unique across all workspaces and cannot be changed to a value already in use.
3. `created_at` is set once at creation and never changes.
4. Deletion is blocked if any of the following conditions are true:
   - Any `User` record references this workspace via `workspace_id`.
   - Any `Tenant` record references this workspace via `workspace_id`.
   - Any `user_tenant` membership record references this workspace via `workspace_id`.
5. A dedicated database schema exists for each workspace to enforce hard data isolation. This schema is provisioned at creation and must be deprovisioned at deletion.

---

## Behavior

**Activate**
Transitions `is_active` from `false` to `true`. No effect if the workspace is already active.

**Deactivate**
Transitions `is_active` from `true` to `false`. No effect if the workspace is already inactive. Does not cascade to users, tenants, or data objects — their `is_active` states are independent.

---

## References
- [core/concept-workspace](core/concept-workspace.md)
