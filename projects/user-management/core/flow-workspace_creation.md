---
package:    core
type:       flow
name:       workspace_creation
version:    1.0.0
status:     draft
tags:       [lifecycle, interaction]
---

# Workspace Creation (Flow)

## Summary

Creates a new workspace and provisions its dedicated database schema. The flow spans the `Workspace` entity and the database infrastructure layer. It is an operator-initiated action — no user exists within the workspace before it is created. Workspace creation is the prerequisite for all other domain activity: users, tenants, and data objects cannot exist until a workspace is provisioned.

---

## Triggers

An operator explicitly requests the creation of a named workspace via the API. Initiated by a **System Operator** providing a workspace name; executed by the **System** after validating input and resolving global uniqueness.

---

## Preconditions

- The operator presents valid credentials authorizing workspace creation.
- The provided workspace `name` is not already in use across the entire system.

---

## Steps

```
1. Operator provides workspace name via API
        │
        ▼
2. System validates workspace name uniqueness globally
        │ ── name already in use  → STOP (warning: workspace already exists)
        ▼
3. Create Workspace record
   · id = generated UUID
   · name = provided name
   · is_active = true
   · created_at = now()
        │
        ▼
4. Provision dedicated database schema for the workspace
   · Schema name derived from workspace name
   · Schema scoped to workspace data objects
        │
        ▼
5. Return created workspace context
   · workspace_id, name, is_active, created_at
```

### Step notes

- Steps 3–4 execute atomically. If schema provisioning fails after the workspace record is created, the entire flow rolls back — no partial state is persisted.
- The schema provisioned in step 4 is empty at creation; tables and objects within it are populated by subsequent domain operations.
- No users, tenants, or role assignments are created as part of this flow. User provisioning occurs separately via the user registration flow, after the workspace exists.

---

## Acceptance Criteria

```
AC-1: New workspace is created successfully
  Given a workspace name that is not already in use in the system
  When  the operator requests workspace creation with that name
  Then  a Workspace record exists with the provided name and is_active = true
  And   created_at is set to the current timestamp

AC-2: Dedicated database schema is provisioned
  Given the workspace from AC-1
  When  creation completes
  Then  a dedicated database schema exists for the workspace
  And   the schema is ready to hold data objects

AC-3: Duplicate workspace name is rejected
  Given a workspace with name "acme" already exists
  When  an operator requests creation of a workspace named "acme"
  Then  no new Workspace record is created
  And   no new schema is provisioned
  And   the flow terminates with a "workspace already exists" warning

AC-4: Creation is atomic
  Given the system encounters a failure during schema provisioning (step 4)
  When  the failure occurs
  Then  no Workspace record persists in the database
  And   no partial schema is left in the database
  And   the system state is as if the flow had never started
```

---

## Error Cases

| Condition | Outcome |
|---|---|
| Workspace name already in use | Flow stops at step 2; returns warning: workspace already exists |
| Schema provisioning fails | Full rollback; no workspace record or schema created |

---

## References
- [core/entity-workspace](core/entity-workspace.md)
- [core/concept-workspace](core/concept-workspace.md)
