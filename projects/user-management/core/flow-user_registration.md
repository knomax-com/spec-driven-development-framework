---
package:    core
type:       flow
name:       user_registration
version:    1.1.0
status:     draft
tags:       [lifecycle, integration]
---

# User Registration (Flow)

## Summary

Registers a person into a workspace on their first successful login. The flow spans the `User`, `Tenant`, `user_tenant` membership, and `Authorization Role Assignment` entities. It is transparent to the end user — it runs automatically during authentication resolution and produces a fully provisioned user context. Subsequent logins bypass the creation steps and resolve directly to the existing user.

---

## Triggers

First successful authentication to a workspace by a person who does not yet have a user record in that workspace. Initiated by the **Authentication Provider** (external system) issuing a signed JWT containing `external_id` (e.g. Auth0 `sub`) and `workspace_name`; executed automatically by the **System** during JWT resolution at the API entry point.

This flow is **idempotent at the workspace level**: if a user already exists for the given `(workspace_id, external_id)` pair, the creation steps are skipped entirely and the existing user is returned.

---

## Preconditions

- A valid, properly signed JWT has been presented to the system.
- The JWT contains an `external_id` field (e.g. Auth0 `sub`).
- The JWT contains a `workspace_name` field.

---

## Steps

```
1. Auth provider issues JWT
        │
        ▼
2. System receives request with JWT
        │
        ▼
3. System resolves workspace_id from workspace_name
        │ ── workspace not found → STOP (error: workspace not found)
        │ ── workspace inactive  → STOP (error: workspace inactive)
        ▼
4. System looks up user by (workspace_id, external_id)
        │
        ├── user found ──────────────────────────────────────┐
        │                                                     │
        ▼                                                     │
5. Create User record                                         │
   · id = generated UUID                                      │
   · workspace_id = resolved workspace                        │
   · external_id = from JWT                                   │
   · is_active = true                                         │
   · created_at = updated_at = now()                          │
   · updated_by = newly created user.id (self-reference)      │
        │                                                     │
        ▼                                                     │
6. Create default Tenant                                      │
   · name = user.id                                           │
   · workspace_id = resolved workspace                        │
   · default_for_user = user.id                               │
   · is_active = true                                         │
        │                                                     │
        ▼                                                     │
7. Create user_tenant membership                              │
   · user_id = user.id                                        │
   · tenant_id = default tenant.id                            │
        │                                                     │
        ▼                                                     │
8. Create Authorization Role assignments                      │
   · For every built-in role EXCEPT core_tenant_deleter:      │
     create assignment (workspace_id, tenant_id, user_id,     │
     authorization_role_id)                                   │
        │                                                     │
        ▼                                                     │
9. Return resolved user context ◄────────────────────────────┘
   · user_id, workspace_id, default tenant_id
```

### Step notes

- Steps 5–8 execute atomically. If any step fails, the entire registration is rolled back — no partial state is persisted.
- `core_tenant_deleter` is excluded from the default role set because the default tenant is tied to the user's lifecycle and must not be explicitly deletable; it is removed only as part of user deletion.

---

## Acceptance Criteria

```
AC-1: New user is registered on first login
  Given a valid JWT referencing a workspace where no user exists for that external_id
  When  the system processes the request
  Then  a User record exists with the correct workspace_id, external_id, and is_active = true

AC-2: Default tenant is provisioned
  Given a newly created User (AC-1)
  When  provisioning completes
  Then  a Tenant record exists with name = user.id and default_for_user = user.id

AC-3: User is a member of their default tenant
  Given a newly created User and default Tenant (AC-1, AC-2)
  When  provisioning completes
  Then  a user_tenant membership record exists linking the user to the default tenant

AC-4: Default role assignments are created
  Given the User and default Tenant from AC-1 and AC-2
  When  provisioning completes
  Then  one Authorization Role Assignment exists for every built-in role except core_tenant_deleter
  And   no assignment for core_tenant_deleter exists for this user in this tenant

AC-5: Subsequent login does not re-create the user
  Given a User already registered in a workspace
  When  a new JWT for the same (workspace_id, external_id) is presented
  Then  no new User, Tenant, membership, or assignment records are created
  And   the existing user context is returned

AC-6: Registration is atomic
  Given the system encounters a failure during any of steps 5–8
  When  the failure occurs
  Then  no partial records persist in the database
  And   the system state is as if the flow had never started
```

---

## Error Cases

| Condition | Outcome |
|---|---|
| JWT signature invalid | Request rejected before this flow begins (auth middleware) |
| `workspace_name` not found in system | Flow stops at step 3; returns error: workspace not found |
| Workspace exists but `is_active = false` | Flow stops at step 3; returns error: workspace inactive |
| Partial provisioning failure (steps 5–8) | Full rollback; no records created |

---

## References
- [core/entity-user](core/entity-user.md)
- [core/concept-tenant](core/concept-tenant.md)
- [core/concept-authorization_role](core/concept-authorization_role.md)
