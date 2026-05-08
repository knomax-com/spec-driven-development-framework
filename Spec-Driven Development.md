# Spec-Driven Development

Spec-Driven Development is a software development approach where the specification is the **primary artifact** and implementation is derived from it.

Instead of starting with code, teams start by clearly defining:
- the problem to solve
- expected behavior
- constraints
- interfaces
- acceptance criteria
- environment and architecture boundaries
- quality requirements

Then humans or AI tools use that specification to generate, implement, test, and evolve the system.

> **The spec is the source of truth. Code is an implementation of the spec.**

---

## Specification Structure

Specifications are defined as a set of **Markdown files with YAML front matter**, organized into folders based on their domain packages.

### File Naming Convention

```
{type}-{name}
{type}-{layer}-{name}     ← for component and test types
```

**Separator rules:**
- `-` (hyphen) separates structural parts: `type`, `layer`, and `name`
- `_` (underscore) joins words within a single part
- `name` must never contain a hyphen — use underscores for multi-word names

**Examples:**
```
concept-soft_deletion
entity-user
flow-user_registration
component-db-user
component-app-knowledge_base
component-api-user
```

### YAML Front Matter

Every file contains the following YAML parameters:

```yaml
---
package:    string        # domain package / bounded context
type:       string        # see Types below
layer:      string        # only for component and test — see layers in each type
name:       string        # semantic name only, words joined with underscores, e.g. "user", "user_registration"
version:    string        # semver e.g. 1.0.0
status:     string        # draft | review | approved | deprecated
tags:       []string      # free-form tags for categorization
references: []string      # paths to related spec files: {package}/{type}-{name} (see References below)
---
```

---


## Types


### `glossary`

**Question it answers:** *What does this specific term mean in this domain?*

Provides precise, unambiguous definitions for individual terms, abbreviations, or business vocabulary used across one or more packages. A glossary entry contains only meaning — no behavior, no pattern, no architectural rationale, no technology references. It is the most foundational spec type: everything else may reference it, but it references nothing.

**Distinction from `concept`:** A `concept` describes a pattern or approach that has rules, rationale, and behavioral implications (e.g. soft deletion as a pattern). A `glossary` entry defines what a word or phrase means in this domain — nothing more.

**Use for:**
- Business jargon or abbreviations that need a canonical definition
- Terms that appear in multiple packages with the same precise meaning
- Domain vocabulary that cannot be inferred from common usage

**Example files:**
```
glossary-active_status
glossary-mau
glossary-tenant
```


### `concept`

**Question it answers:** *How does this pattern or approach work in our domain, and why?*

Describes a behavioral pattern, architectural approach, or system-wide policy applied consistently across entities. A concept carries rules, rationale, and implications — it explains not just what something is called, but how it works and why it was chosen.

A concept is completely technology-agnostic — it must never reference specific databases, storage engines, programming languages, frameworks, or any other implementation technology. How a pattern is realized in a specific stack belongs in a `component` spec.

**Use for:**
- Patterns applied consistently across entities (e.g. soft deletion, ownership, tenancy)
- Architectural approaches and their rationale
- System-wide policies with behavioral implications

**Chapters:**

| Chapter | Purpose | Required? |
|---|---|---|
| `Summary` | What this concept is, what problem it addresses, and why it was chosen. One to three sentences grounding the reader before any pattern detail. | Always |
| `Pattern: {name}` | One section per behavioral rule. Describes the rule and includes an inline **Why:** explaining the rationale. Repeat for each distinct pattern. | At least one |
| `Implications` | Consequences for anything that applies this concept — behavioral obligations, constraints, or side-effects that are not obvious from the patterns alone. | When non-obvious |
| `Related Patterns` | Other concepts this one builds on, composes with, or intentionally diverges from. | When they exist |

#### Writing Patterns

Each pattern section follows this structure:

```markdown
## Pattern: {descriptive name}

{One or more paragraphs describing the rule — what it enforces and how it works.}

**Why:** {The rationale — why this approach was chosen, what problem it solves, what trade-offs were accepted.}
```

**Rules:**
- The pattern name must be descriptive and self-explanatory in isolation.
- The body states what the rule enforces — no implementation details, no technology references.
- `**Why:**` is required in every pattern section. A rule without a rationale is incomplete.
- Each pattern must be independently understandable; avoid cross-references between patterns.

**Example files:**
```
concept-user
concept-tenant
concept-soft_deletion
concept-eventual_consistency
```


### `entity`

**Question it answers:** *What is this domain object, what does it contain, and how does it behave on its own?*

Describes a logical domain object — its identity, attributes, invariants, and self-contained behavior. Completely independent of any specific implementation technology.

An entity must never define tables, functions, frameworks, or any implementation detail.

**Chapters:**

| Chapter | Purpose |
|---|---|
| `Summary` | Why this entity exists — its role and purpose in the domain. One or two sentences grounding the reader before any detail. |
| `Identity` | What uniquely identifies an instance — a surrogate key, natural key, or composite, described in domain terms only. |
| `Attributes` | All logical fields defined using the standard attribute format (see [Writing Attributes](#writing-attributes)). Domain types only — no implementation-specific types. |
| `Lifecycle` | Explicit list of states and valid transitions, expressed as a state machine. Omit for stateless entities. |
| `Invariants` | Rules that must always be true for this entity in isolation — constraints any implementation must enforce, regardless of caller or context. |
| `Behavior` | Actions this entity performs on itself — the operations it owns. For code these become methods; for business logic these are the responsibilities of whoever owns this entity. |


#### Writing Attributes

Attributes are defined using a YAML map. Each key is the attribute name; nested properties describe it.

```yaml
workspace_id:
  type:        UUID
  source:      system-resolved
  constraints: [required, immutable, references workspace.id]
  description: The workspace this user belongs to

display_name:
  type:        string
  source:      user-provided
  constraints: [required, max: 100]
  description: The name shown in the UI

status:
  type:        enum(active, suspended, deleted)
  source:      system-generated
  constraints: [required, default: active]
  description: Current lifecycle state of the user

deleted_at:
  type:        timestamp
  source:      system-generated
  constraints: [optional, immutable-once-set]
  description: Set when the user is soft-deleted; null while active
```

**`source` values:**

| Value | Meaning |
|---|---|
| `user-provided` | Supplied directly by the actor initiating the action |
| `system-generated` | Created automatically by the system (e.g. UUID, timestamp) |
| `system-resolved` | Derived by the system from context (e.g. session, tenant, or parent) |
| `derived` | Computed from other attributes of the same entity |

**Standard constraint vocabulary:**

| Constraint | Meaning |
|---|---|
| `required` | Must always have a value |
| `optional` | May be null or absent |
| `immutable` | Cannot be changed after creation |
| `immutable-once-set` | Cannot be changed after a non-null value is assigned |
| `unique` | Must be unique across all instances |
| `default: X` | Assigned this value if not provided |
| `min: N` / `max: N` | Numeric or length bounds |
| `references X.id` | Logical foreign reference to another entity |

**Domain type vocabulary** (entity layer only — no implementation types):

| Type | Meaning |
|---|---|
| `string` | Text value |
| `boolean` | True or false |
| `integer` | Whole number |
| `decimal` | Fractional number |
| `timestamp` | Point in time (date and time) |
| `date` | Calendar date without time |
| `UUID` | Universally unique identifier |
| `enum(a, b, …)` | Closed set of named values |
| `[]Type` | List of the specified type |
| `map[K]V` | Key-value map |

> The same attribute format is used in `component` specs for schema definitions. At the component layer, `type` uses implementation-specific types (e.g. `varchar(100)`, `timestamptz`, `jsonb`) instead of domain types.


### `flow`

**Question it answers:** *What happens across entity boundaries?*

Describes logic that cannot be owned by a single entity — cross-entity rules, lifecycle triggers, policies, and integrations with external systems. Flows are the home of acceptance criteria and multi-entity use-cases.

A flow is completely technology-agnostic — it describes what happens and why, not how it is implemented. No databases, frameworks, programming languages, or infrastructure details belong here; those go in a `component` spec.

**Use for:**
- Lifecycle events that affect multiple entities
- Business policies spanning more than one entity
- Multi-entity interactions and use-cases
- Integration with external systems

**Flow is named by its function, not by entity ownership.** When a flow clearly involves one primary entity, it can be grouped under that entity's name. When ownership is ambiguous, it gets an explicit descriptive name.

**Flow categories (expressed as tags):**

| Tag | Purpose |
|---|---|
| `lifecycle` | Creation, state change, or destruction triggering cross-entity effects |
| `policy` | Business constraint with no single entity owner |
| `interaction` | Multi-entity usecase driven by an actor |
| `integration` | Boundary with an external system |

**Chapters:**

| Chapter | Purpose |
|---|---|
| `Summary` | What this flow does, why it exists, and which entities it spans. |
| `Triggers` | What causes the flow to start — a user action, an external event, a state change, a schedule, or an internal condition. Includes the actor when one is present. |
| `Preconditions` | What must be true before the flow can begin — required entity states, permissions, or data that must exist. |
| `Steps` | The ordered sequence of actions and decisions that make up the flow, naming which entity is affected at each step. |
| `Acceptance Criteria` | Formal, testable conditions that define successful completion. The direct source of truth for test specs. |
| `Error Cases` | What can go wrong, what triggers each failure, and what the expected outcome is. |

#### Writing Acceptance Criteria

Each criterion is written using the **Given / When / Then** format — one criterion per scenario, numbered sequentially.

```
AC-1: <short title>
  Given <the state of the system or context before the action>
  When  <the action or event that occurs>
  Then  <the expected observable outcome>
  And   <additional outcome, if needed>
```

**Rules:**
- Each criterion must be independently testable and unambiguous.
- Write in domain language — no implementation details, no technology references.
- Cover the primary success path first, then edge cases within the happy path.
- Failure paths belong in `Error Cases`, not here.

**Example:**

```
AC-1: Successful registration creates an active user
  Given no account exists for the provided email
  When  a user submits a valid registration form
  Then  a new user record is created with status "active"
  And   a verification email is sent to the provided address

AC-2: Duplicate email is rejected
  Given an account already exists for the provided email
  When  a user submits a registration form with that email
  Then  no new user record is created
  And   the flow terminates with an "email already in use" error
```

**Example files:**
```
flow-user_registration
flow-user_suspension
flow-tenant_onboarding
flow-attribute_assignment
flow-knowledge_import
```


#### Cross-Package Flows

Flows frequently span more than one package. The `package` field always contains a single string — the folder the file physically lives in. Cross-package dependencies are expressed through `references`, not through a multi-valued `package` field.

**Primary-package rule:** Place the flow under the package of the entity whose state is most fundamentally changed by the flow.

When the primary entity is ambiguous, apply these tie-breakers in order:

1. The entity that is *written to* — whose attribute or lifecycle state is mutated
2. The entity that is the *target* of the action (e.g. recipient of a transfer, not the initiator)
3. The more "downstream" or "owned" entity in the domain model

**Examples:**

| Flow | Package | Reasoning |
|---|---|---|
| `flow-attribute_assignment` | `knowledge` | The attribute entity is what is being mutated |
| `flow-user_registration` | `core` | A user is being created; any side-effects on other packages are secondary |
| `flow-knowledge_import` | `knowledge` | The knowledge base drives the flow; user/tenant effects are downstream |


### `component`

**Question it answers:** *How is this implemented in a specific layer?*

Describes the concrete implementation of entities and flows within a specific technical layer. This is where tables, functions, endpoints, and services are defined with enough detail to generate or implement actual code.

**Naming convention:**
```
component-{layer}-{name}
```

**Layers:**

| Layer | Purpose |
|---|---|
| `db` | Tables, indexes, constraints, database functions, triggers |
| `app` | Application-level services and business logic functions |
| `api` | REST or GraphQL endpoints, request/response shapes |
| `worker` | Background jobs, scheduled tasks |
| `event` | Event definitions, publishers, consumers |

**Contains:**
- Overview of responsibility
- Schema or interface definition
- Functions with signature, logic, and error cases
- Dependencies on other components
- References to entities and flows being implemented

**Example files:**
```
component-db-user
component-db-tenant
component-app-user
component-app-knowledge_base
component-api-user
component-worker-knowledge_import
```


### `test`

**Question it answers:** *How do we verify that a flow or entity behaves as specified?*

Defines formal test cases derived directly from acceptance criteria in `flow` and invariants in `entity` files. Test files are the bridge between specification and verification — they make acceptance criteria executable.

**Naming convention follows the same pattern as `component`:**
```
test-{layer}-{name}
```

**Layers:**

| Layer | Purpose |
|---|---|
| `unit` | Isolated tests for a single entity invariant or component function |
| `integration` | Tests spanning multiple components within a layer (e.g. DB functions together) |
| `e2e` | End-to-end tests covering a full flow across all layers |

**Example files:**
```
test-unit-user
test-integration-user_registration
test-e2e-knowledge_import
```

**Contains:**
- Reference to the flow or entity being tested
- Test cases with input, expected output, and expected errors
- Preconditions and setup requirements

---

## Type Summary

| Type | Question answered | Technology-agnostic? | Cross-entity? |
|---|---|---|---|
| `glossary` | What does this specific term mean? | ✅ | ✅ |
| `concept` | How does this pattern or approach work, and why? | ✅ | ✅ |
| `entity` | What is this object and how does it behave alone? | ✅ | ❌ |
| `flow` | What happens across entity boundaries? | ✅ | ✅ |
| `component` | How is this implemented in a specific layer? | ❌ | — |
| `test` | How do we verify the spec is correctly implemented? | ❌ | — |

---

## Layering

```
glossary         ← term definitions, domain vocabulary
concept          ← patterns, architectural decisions (may reference glossary)
  └── entity     ← domain model, implementation-agnostic
       └── flow  ← cross-entity logic, policies, usecases
            └── component-{layer}  ← concrete implementation per layer
            └── test-{layer}       ← verification of spec per layer
```

Each layer depends on the ones above it. A `component` implements `entity` and `flow` definitions. A `flow` references `entity` definitions. An `entity` may reference `concept` definitions for shared terminology. A `concept` or `entity` may reference `glossary` entries for precise term definitions.

---

## References

Every spec file must include a `references` field in its YAML front matter — a list of paths to other spec files that are directly relevant to understanding or implementing the current spec.

References make the dependency graph explicit and machine-readable, allowing both humans and AI tools to navigate from a definition to its implementation and from an implementation back to its specification.

### What to reference

| Type | Should reference |
|---|---|
| `glossary` | nothing (foundational layer; no upward references) |
| `concept` | `glossary` entries it uses; other `concept` files it builds upon (optional) |
| `entity` | `concept` files whose patterns or terminology it applies; `glossary` entries for key terms (optional) |
| `flow` | `entity` files involved in the flow |
| `component` | `entity` and `flow` files it implements; other `component` files it depends on |
| `test` | `flow`, `entity`, or `component` files it verifies |

### Example

```yaml
---
package:    core
type:       component
layer:      db
name:       user
version:    1.0.0
status:     approved
tags:       [db]
references:
  - core/entity-user
  - core/flow-user_registration
  - core/flow-user_suspension
  - knowledge/component-db-attribute
---
```

```yaml
---
package:    knowledge
type:       test
layer:      e2e
name:       knowledge_import
version:    1.0.0
status:     approved
tags:       [e2e]
references:
  - knowledge/flow-knowledge_import
  - knowledge/component-api-knowledge
  - knowledge/component-app-knowledge_base
  - knowledge/component-db-attribute
---
```

### Rules

- References are **unidirectional** — a higher-layer spec references lower-layer specs, not the reverse. A `component` references the `flow` it implements; the `flow` does not reference back to the `component`.
- References use the path format **`{package}/{filename-without-extension}`**, e.g. `core/entity-user`, `core/flow-user_registration`, or `knowledge/component-db-attribute`. This maps directly to a file on disk (append `.md`).
- Every `component` and `test` file must reference at least one `entity` or `flow`.
- A spec with no meaningful references may omit the field or leave it as an empty list.

---

## Version

The `version` field tracks the evolution of a spec file using **semantic versioning** (`MAJOR.MINOR.PATCH`). It reflects changes to the specification itself, not to the implementation.

New files start at `1.0.0`.

| Segment | Bump when |
|---|---|
| `PATCH` | Typos, wording clarifications, reformatting — no semantic change |
| `MINOR` | New content added: new attributes, new acceptance criteria, expanded behavior — backward-compatible |
| `MAJOR` | Breaking change: renamed or removed fields, changed invariants, altered contracts that invalidate existing implementations |

Spec version is independent of any implementation version. A component built against `entity-user 2.1.0` should reference that version explicitly in its own documentation if version pinning matters to the team.

---

## Status

The `status` field describes the **review and approval state** of a spec file. It governs whether a spec is safe to implement against.

| Status | Meaning |
|---|---|
| `draft` | Work in progress. Content is incomplete or actively changing. Do not implement. |
| `review` | Ready for team review. Content is complete but not yet approved. |
| `approved` | Reviewed and agreed upon. Safe to implement. |
| `deprecated` | No longer active. Superseded by another spec or removed from scope. |

**Valid transitions:**

```
draft → review → approved → deprecated
draft → deprecated   (abandoned before approval)
approved → draft     (major rework required)
```

A spec should only be implemented against when its status is `approved`. Referencing a `draft` or `review` spec in a `component` is permitted during active development but must be resolved before that component reaches `approved` status itself.

---

## File Organization

Files are organized into folders by **package** (bounded context). Every folder is a package — there is no special or reserved package for cross-package content. When a spec file is not clearly owned by any single package, it lives directly in the root `specs/` folder.

**Cross-package placement rule:** If a spec applies to more than one package and has no clear primary owner, place it at the root `specs/` level, regardless of type. A named folder (e.g. `core/`, `knowledge/`) is just another package, not a catch-all for cross-cutting files.

```
specs/
  glossary-active_status.md              ← cross-package: no single owner
  glossary-mau.md                        ← cross-package: no single owner
  core/
    concept-soft_deletion.md             ← owned by the core package
    concept-user.md
    concept-tenant.md
    entity-user.md
    entity-tenant.md
    flow-user_registration.md
    flow-user_suspension.md
    flow-tenant_onboarding.md
    component-db-user.md
    component-db-tenant.md
    component-app-user.md
    component-app-tenant.md
    component-api-user.md
    test-unit-user.md
    test-integration-user_registration.md
    test-e2e-user_registration.md
  knowledge/
    concept-attribute.md
    concept-knowledge.md
    entity-attribute.md
    entity-knowledge_base.md
    flow-attribute_assignment.md
    flow-knowledge_import.md
    component-db-attribute.md
    component-app-knowledge_base.md
    component-api-knowledge.md
    component-worker-knowledge_import.md
    test-unit-attribute.md
    test-e2e-knowledge_import.md
```






