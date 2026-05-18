---
package:    core
type:       concept
name:       authorization_role
version:    1.0.0
status:     approved
tags:       [authorization, access-control, core]
---

# Authorization Role

## Summary

The authorization role system defines what actions a user may perform within a tenant. It uses a role-based access control (RBAC) model with two distinct object types: an immutable **role definition** (the named permission) and a runtime **role assignment** (linking a role to a user in a tenant). The concept defines five foundational patterns.

---

## Pattern: Role-Based Access Control

Access to data operations (create, read, update, delete) is governed by **named roles**. A user may perform an action only if they hold a role that grants that action within the relevant tenant context. There is no implicit access — every permitted action requires an explicit role assignment.

Roles are grouped by domain and action using the naming pattern `{package}_{domain}_{action}` (e.g. `core_user_reader`, `core_tenant_updater`).

**Why:** Named roles make permissions explicit, auditable, and uniform. Adding a new capability means defining a new role and assigning it — there are no hidden grants or ambient authority.

---

## Pattern: Tenant-Scoped Permissions

Role assignments are **scoped to a tenant**. The same user may hold different roles in different tenants — for example, a reader in one tenant and an editor in another. There is no workspace-level or global role assignment; every grant is always within a specific tenant context.

**Why:** Tenants are the data ownership boundary. Scoping permissions to the tenant ensures that access rights follow the same isolation boundary as the data they protect, making it impossible to grant a role that "leaks" across tenants.

---

## Pattern: System-Defined Role Catalog

Role **definitions** are created only at **application startup** and cannot be created, modified, or deleted via the API. The set of available roles is determined by the application code, not by end users or operators. The `is_assignable` flag marks whether a role may be assigned via the API; non-assignable roles are reserved for internal system use.

**Built-in roles:**

| Role | Description |
|---|---|
| `core_user_reader` | Read user information |
| `core_user_updater` | Update user information |
| `core_tenant_reader` | Read tenant information |
| `core_tenant_updater` | Update tenant information |
| `core_tenant_deleter` | Delete a tenant |

**Why:** Keeping role definitions system-controlled prevents privilege escalation through API calls and guarantees that the set of available permissions is always consistent with the application's actual capabilities.

---

## Pattern: Automatic Role Provisioning

Roles are assigned automatically without any explicit request from the user:

**On user registration (default tenant):** All built-in roles are assigned **except** `core_tenant_deleter`. The default tenant is tied to the user's lifecycle and must not be explicitly deletable.

**Why:** Automatic provisioning ensures that a newly created user can immediately use the system without a separate administration step. The deliberate exclusion of `core_tenant_deleter` from the default set protects the default tenant from accidental deletion.

---

## Pattern: Assignment Immutability

Role assignments **cannot be updated in place**. To change a user's roles, the existing assignment must be deleted and a new one created. This means every assignment record is a complete, stable statement of a grant — there is no mutable state to track.

**Why:** Immutable assignments simplify audit trails. Every change to a user's permissions produces a new record with its own `created_at` timestamp, making the history of grants and revocations fully traceable without a separate audit table.

---

## References
- [core/concept-workspace](core/concept-workspace.md)
- [core/concept-tenant](core/concept-tenant.md)
- [core/concept-user](core/concept-user.md)