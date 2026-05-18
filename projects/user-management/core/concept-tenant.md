---
package:    core
type:       concept
name:       tenant
version:    1.0.0
status:     approved
tags:       [identity, core, multi-tenancy, data-isolation]
---

# Tenant

## Summary

A `Tenant` is the soft data separation group within a `Workspace`. It groups one or more users and may own data objects. Every user always has at least one tenant — their **default tenant** — created automatically when they first register. The tenant concept defines below foundational patterns applied throughout the system.

---

## Pattern: Soft Data Separation

A tenant provides **logical isolation** within a workspace. Data objects are always scoped to a tenant via a `tenant_id` column — queries never return data across tenant boundaries unless the caller explicitly requests multiple tenants. This is in contrast to the hard (schema-level) isolation that workspaces provide.

**Why:** Soft separation via `tenant_id` allows flexible grouping without the operational cost of a separate database schema per group.

---

## Pattern: Multi-User Grouping

Multiple users can be members of the same tenant simultaneously. A user can also belong to more than one tenant at the same time. Membership is tracked through an explicit user - tenant connection which links a user to a tenant within a workspace.

**Why:** Collaboration requires shared data spaces. Multi-user membership lets teams share data without duplicating it or creating separate accounts.

---

## Pattern: Default Tenant Provisioning

Every user always has at least one tenant: their **default tenant**. It is created automatically at the moment the user is first registered in a workspace. Its name equals the user's internal `id`, and it is marked as the default for that user.

The default tenant cannot be explicitly deleted while the user exists — it is removed only as part of user removal. This guarantees the invariant that every active user always has a tenant context available.

**Why:** The tenant is the primary unit of data ownership. Without a guaranteed default, every data operation would require the caller to specify a tenant, creating unnecessary friction and fragile states where a user could exist without any data context.

---

## Pattern: Tenant as Data Ownership Boundary

Every data object in the system belongs to exactly one tenant. A tenant cannot be deleted until all of its data objects are removed first. This makes the tenant the terminal ownership boundary for all user content.

Role assignments are also scoped to a tenant — a user's permissions differ per tenant. The same user may be a reader in one tenant and an editor in another.

**Why:** Attaching ownership to the tenant (rather than the user or workspace) keeps access control and data lifecycle cleanly aligned. It also enables delegation — multiple users can co-own and operate on the same set of data objects through shared tenant membership.

---

## References
- [core/concept-authorization_role](core/concept-authorization_role.md)
- [core/concept-user](core/concept-user.md)
- [core/concept-workspace](core/concept-workspace.md)