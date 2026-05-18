---
package:    core
type:       flow
name:       tenant_creation
version:    1.0.0
status:     draft
tags:       [lifecycle, interaction]
---

# Tenant Creation (Flow)

## Summary

Creates a new non-default tenant within a workspace and fully provisions the creating user as its owner. The flow spans the `Tenant`, `user_tenant` membership, and `authorization_role_assignment` entities. It differs from the default tenant created during user registration in one critical way: the creating user receives **all** built-in roles including `core_tenant_deleter`, making them the explicit owner who may delete the tenant.

---

## Triggers

A user explicitly requests the creation of a named tenant via the API. Initiated by an **authenticated User** providing a tenant name; executed by the **System** after validating inputs and resolving context.

---

## Preconditions

- The requestor presents a valid, properly signed JWT.
- The workspace referenced by the JWT exists and `is_active = true`.
- The requestor exists as a user in the workspace and `is_active = true`.
- The provided tenant `name` is not already in use within the workspace.

---

## Steps

```
1. Caller provides tenant name via API
        │
        ▼
2. System resolves workspace_id from JWT
        │ ── workspace not found  → STOP (error: workspace not found)
        │ ── workspace inactive   → STOP (error: workspace inactive)
        ▼
3. System resolves requestor user_id from JWT external_id
        │ ── user not found       → STOP (error: user not found)
        │ ── user inactive        → STOP (error: user inactive)
        ▼
4. System checks tenant name uniqueness within workspace
        │ ── name already in use  → STOP (error: tenant name already exists)
        ▼
5. Create Tenant record
   · id = generated UUID
   · workspace_id = resolved workspace
   · name = provided name
   · default_for_user = null  (not a default tenant)
   · is_active = true
   · created_at = updated_at = now()
   · updated_by = requestor user.id
        │
        ▼
6. Create user_tenant membership
   · workspace_id = resolved workspace
   · user_id = requestor user.id
   · tenant_id = new tenant.id
   · updated_by = requestor user.id
        │
        ▼
7. Create Authorization Role assignments
   · For every built-in role INCLUDING core_tenant_deleter:
     create assignment (tenant_id, user_id, authorization_role_id)
   · updated_by = requestor user.id
        │
        ▼
8. Return created tenant context
   · tenant_id, name, workspace_id
```

### Step notes

- Steps 5–7 execute atomically. If any step fails, the entire flow is rolled back — no partial state is persisted.
- `core_tenant_deleter` **is included** in the role set (contrast with user registration, where it is excluded for the default tenant). The creating user becomes the owner of this tenant and may delete it.

---

## Acceptance Criteria

```
AC-1: New tenant is created successfully
  Given an active user in an active workspace
  And   a tenant name that is not already in use in that workspace
  When  the user requests tenant creation with that name
  Then  a Tenant record exists with the provided name, the resolved workspace_id, and is_active = true

AC-2: Creator is a member of the new tenant
  Given the tenant from AC-1
  When  creation completes
  Then  a user_tenant membership record exists linking the requestor to the new tenant

AC-3: All built-in roles including core_tenant_deleter are assigned
  Given the tenant and membership from AC-1 and AC-2
  When  creation completes
  Then  one authorization_role_assignment exists for every built-in role, including core_tenant_deleter
  And   no built-in role is missing from the assignment set

AC-4: Duplicate tenant name is rejected
  Given a tenant with name "alpha" already exists in the workspace
  When  a user requests creation of a tenant named "alpha" in the same workspace
  Then  no new Tenant, membership, or assignment records are created
  And   the flow terminates with a "tenant already exists" error

AC-5: Creation is atomic
  Given the system encounters a failure during any of steps 5–7
  When  the failure occurs
  Then  no partial records persist in the database
  And   the system state is as if the flow had never started
```

---

## Error Cases

| Condition | Outcome |
|---|---|
| JWT signature invalid | Request rejected before this flow begins (auth middleware) |
| `workspace_name` not found in system | Flow stops at step 2; returns error: workspace not found |
| Workspace exists but `is_active = false` | Flow stops at step 2; returns error: workspace inactive |
| Requestor user not found in workspace | Flow stops at step 3; returns error: user not found |
| Requestor user `is_active = false` | Flow stops at step 3; returns error: user inactive |
| Tenant name already in use in workspace | Flow stops at step 4; returns warning: tenant already exists |
| Partial provisioning failure (steps 5–7) | Full rollback; no records created |

---

## References
- [core/entity-tenant](core/entity-tenant.md)
- [core/entity-user_tenant](core/entity-user_tenant.md)
- [core/entity-authorization_role_assignment](core/entity-authorization_role_assignment.md)
- [core/concept-tenant](core/concept-tenant.md)
- [core/concept-authorization_role](core/concept-authorization_role.md)
