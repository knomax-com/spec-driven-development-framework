---
package:    core
type:       concept
name:       user
version:    1.0.0
status:     approved
tags:       [identity, core, multi-tenancy]
---

# User

## Summary

A `User` represents a person connecting to the application. The user concept defines below foundational patterns that apply throughout the system.

---

## Pattern: Workspace-Scoped Identity

A user does not exist globally — they exist within a `Workspace`. The same person must be registered separately in every workspace they need access to. This means identity is always a combination of the person and the workspace: two records with the same real-world person but different workspaces are treated as fully distinct users.

**Why:** Workspaces provide hard data isolation. Each workspace (typically created based on a request form a specific company) gets a fully separated data space. Scoping identity to the workspace enforces this boundary at the user level and prevents cross-tenant data leakage.

---

## Pattern: Dual-Identity Model

Every user has two identifiers:

- `external_id` — provided by the external authentication provider (e.g. Auth0 `sub`). This is what arrives at the API boundary with every request.
- `id` — an internal, system-generated UUID. This is what all internal references use.

The system resolves `external_id` → `id` once at the entry point. All downstream logic operates exclusively on the internal `id`. The `external_id` is never used as a foreign key inside the system.

**Why:** Decoupling internal identity from provider-issued identity means the authentication provider can change or issue new identifiers without requiring migrations of internal data. It also keeps internal relationships stable and provider-agnostic.

---

## Pattern: Multi-Tenant Membership

A user can belong to multiple `Tenant`s simultaneously within the same workspace. Tenants are the unit of soft data separation; a user's permissions and data visibility depend on which tenants they are a member of and which roles they hold in each.

**Why:** Real-world teams share data across groups. Multi-tenant membership lets a single user participate in different projects or organizational units without needing separate accounts.

---

## Pattern: Default Tenant Provisioning

Every user always has at least one tenant: their **default tenant**. This tenant is created automatically at the moment the user is first registered in a workspace. Its name equals the user's internal `id`, and it is marked as the default for that user.

The default tenant cannot be explicitly deleted while the user exists — it is removed only as part of user removal. This guarantees the invariant that every active user always has a tenant context available.

**Why:** Default tenant provides user hwy how to share the data. Tenant works as the main identifier to separate data.

---

## References
- [core/concept-workspace](core/concept-workspace.md)
- [core/concept-tenant](core/concept-tenant.md)
