---
package:    core
type:       concept
name:       workspace
version:    1.0.0
status:     approved
tags:       [identity, core, data-isolation]
---

# Workspace

## Summary

A `Workspace` is the top-level container in the system. It provides **hard data isolation** — data in one workspace is completely inaccessible from another, enforced at the storage layer level via dedicated, isolated partitions. Any other objects from other packages exist within a workspace boundary. The workspace concept defines following foundational patterns.

---

## Pattern: Storage-Level Hard Isolation

Each workspace maps to a **dedicated, isolated storage partition** — for example, a separate database schema, a distinct folder hierarchy, or a separate storage bucket. Data objects are created inside that partition, making cross-workspace access physically impossible without an explicit, intentional traversal of workspace boundaries. This is hard separation — it cannot be bypassed by a query or a filter; it is enforced by the storage layer itself.

This contrasts with the soft (logical) isolation provided by tenants, where isolation is enforced through a tenant identifier at the application level.

**Why:** Hard storage-level isolation eliminates entire categories of data leakage bugs. Even if application-level tenant filtering were bypassed, a workspace's data remains unreachable from another workspace. This is the appropriate guarantee for customers treating their workspace as a private data environment.

---

## Pattern: Global Workspace Namespace

A workspace's `name` is **globally unique** across the entire system.

**Why:** The workspace name must serve as a stable, human-readable routing key.

---

## Pattern: Scope Container

Every object in the system — user, tenant, data object — belongs to exactly one workspace.

A workspace cannot be deleted until it is empty. This enforces that the workspace boundary is the outermost lifecycle unit — nothing inside it outlives it.

**Why:** Making the workspace the outermost container with hard lifecycle constraints prevents orphaned data. It also makes the cost of workspace deletion explicit and operator-visible — you cannot accidentally delete a workspace that still contains any data.

---

## References
- [core/concept-tenant](core/concept-tenant.md)