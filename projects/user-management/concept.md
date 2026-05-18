---
package:    
type:       concept
name:       user_management
version:    1.0.0
status:     approved
tags:       [identity, authorization, multi-tenancy, data-isolation]
---

# User Management

## Summary

User Management is a solution that provides the foundational layer for managing users, their organizational grouping, and their authorization relevant to the solution. It is designed as the backend for any application that needs multi-tenant user identity and authorization — but deliberately excludes authentication, which is delegated to an external identity provider.

The solution manages five core domain objects organized in a strict containment hierarchy:

- **Workspace** — the top-level container providing hard data isolation between separate organizations or environments.
- **User** — a person registered within a workspace, identified by both an external (provider-issued) and an internal (system-generated) identity.
- **Tenant** — a logical group within a workspace that owns data and scopes permissions. Every user receives a personal default tenant on registration and may create or join additional tenants for collaboration.

The solution covers three primary lifecycle flows: workspace creation (operator-initiated), user registration (automatic on first login), and tenant creation (user-initiated). 

---

## Pattern: Identity Without Authentication

User management defines who users are and what they are allowed to do, but never verifies that a caller is who they claim to be. Authentication is handled entirely by an external identity provider. The system receives a verified external identifier at the API boundary and resolves it to an internal identity — from that point onward, only the internal identity is used.

**Why:** Separating identity management from authentication allows the authentication provider to change independently. It also keeps this domain focused on structure and access rather than cryptographic verification, reducing its surface area and complexity.

---

## Pattern: Containment Hierarchy

Every object in the system belongs to a strict containment chain: **Workspace → Tenant → Data**. Users exist within a workspace and participate in tenants through explicit memberships. Data objects are always owned by exactly one tenant within exactly one workspace. This hierarchy is enforced at every level — there are no global or unscoped objects.

**Why:** A fixed containment hierarchy makes data ownership, access control, and lifecycle management predictable. Deletion, isolation, and permission checks all follow the same structural path, eliminating ambiguity about which boundary governs a given object.

---

## Pattern: Layered Data Isolation

Data separation is achieved through two complementary strategies operating at different levels of the hierarchy:

- **Hard isolation** at the workspace level — each workspace maps to a physically separated storage partition, making cross-workspace access structurally impossible.
- **Soft isolation** at the tenant level — within a workspace, tenants provide logical grouping enforced at the application level through a tenant identifier.

**Why:** Two isolation layers address different trust boundaries. Workspaces protect between distinct organizations or environments where data must never mix. Tenants enable flexible, fine-grained grouping within a workspace without the operational cost of physical separation.

---

## Pattern: Automatic Provisioning on First Contact

Users are not pre-created by administrators. When a person authenticates to a workspace for the first time, the system automatically creates their user record, provisions a personal default tenant, establishes the membership link, and assigns the standard set of roles — all within a single atomic operation. Subsequent logins resolve to the existing user without repeating any creation steps.

**Why:** Automatic provisioning eliminates manual onboarding steps and ensures that every authenticated person has a fully operational context from their first interaction. The atomic nature prevents partially provisioned users from ever existing in the system.

---

## Pattern: Role-Based Authorization Scoped to Tenants

Access to operations is governed by named roles assigned per user per tenant. A user's permissions in one tenant are independent of their permissions in another, even within the same workspace. Roles are defined in a system-controlled catalog at application startup and cannot be created or modified through the API. There is no workspace-level or global role grant — every permission is always within a specific tenant context.

**Why:** Scoping authorization to the tenant aligns the permission boundary with the data ownership boundary. This prevents accidental cross-tenant privilege leakage and enables flexible collaboration models where the same user can hold different levels of access in different groups. A system-controlled role catalog prevents privilege escalation through API manipulation.

---

## Scope

**In scope:**
- Workspace lifecycle (creation, activation, deactivation)
- User lifecycle (registration, activation, deactivation)
- Tenant lifecycle (creation, activation, deactivation, renaming)
- User-tenant membership management
- Role-based authorization (role definitions and assignments)

**Out of scope:**
- Authentication — proof of identity is delegated to an external provider (e.g. Auth0)

---

## References
- core/concept-user
- core/concept-workspace
- core/concept-tenant
- core/concept-authorization_role