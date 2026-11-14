# Spec-Driven Development Framework

Spec-Driven Development Framework is a software development approach where the specification is the **primary artifact** and implementation is derived from it.

Instead of starting with code, teams start by clearly defining:
- the problem to solve - concept
- entities (optional) - logical objects and their behavior
- expected behavior between entities (optional)
- specific desired implementation (optional)
- testing scenarios (optional)

Then humans or AI tools use that specification to generate, implement, test, and evolve the system.

> **The spec is the single source of truth. Code is an implementation of the spec.**

The specifications are defined as a set of **Markdown files with YAML front matter**, organized into folders based on their domain packages.

The file structure give users strong control over the implementation process.
They can choose how much detail to include, and this can be done separately for each package.

---

## File Naming Convention

```
{type}-{name}
{type}-{layer}-{name}     ← for component and test types
```

**Separator rules:**
- `-` (hyphen) separates structural parts: `type`, `layer`, and `name`
- `_` (underscore) joins words within a single part
- `name` must never contain a hyphen — use underscores for multi-word names

---

## File Organization

Files are organized into folders by **package** (bounded context). A **package** is a direct child folder of `specs/`. Folders *inside* a package are **groups** — sorting only; they do not get their own package identity or `concept-*` entry point. When a spec file is not clearly owned by any single package, it lives directly in the root `specs/` folder.

**Cross-package placement rule:** If a spec applies to more than one package and has no clear primary owner, place it at the root `specs/` level, regardless of type. A named folder directly under `specs/` is a package, not a catch-all for cross-cutting files.

**Package entry point:** In every package, start by reading `concept-{package}` at the **package root** (the concept named after that package). It is the overview of the package’s purpose and approach; other specs in the package — including those in groups — should be read after it.

**Groups:** Optional nested folders under a package organize related specs (e.g. by capability area). Groups may nest. They are not packages: the owning package is always the first path segment under `specs/`.

**Promotion:** When a group becomes its own capability, move it to `specs/{new-package}/` with its own `concept-{new-package}.md`.

```
specs/
  glossary-{term}.md                     ← cross-package: no single owner
  {package}/
    concept-{package}.md                 ← package entry point: read first
    concept-{name}.md
    entity-{name}.md
    flow-{name}.md
    component-{layer}-{name}.md
    test-{layer}-{name}.md
    {group}/                             ← sorting only; nestable
      entity-{name}.md
      flow-{name}.md
      component-{layer}-{name}.md
      test-{layer}-{name}.md
```

`specs/` holds definition Markdown only (`*.md`). Never put source, DDL, scripts, tests, or other code under `specs/`. Implementation lives under `app/…` and, when virtualization is used, `infra/<service>/<stack>/`.

---

## YAML Front Matter

Every file contains the following YAML parameters:

```yaml
---
package:    string        # package root name (first path segment under specs/) or empty; never a group path
type:       string        # see File Types chapter
layer:      string        # only for component and test — see layers in each type
name:       string        # semantic name only, words joined with underscores
version:    string        # semver e.g. 1.0.0
status:     string        # draft | review | approved | deprecated
tags:       []string      # free-form tags for categorization
---
```

---

---

### Version

The `version` field tracks the evolution of a spec file using **semantic versioning** (`MAJOR.MINOR.PATCH`). It reflects changes to the specification itself, not to the implementation.

New files start at `1.0.0`.

| Segment | Bump when |
|---|---|
| `PATCH` | Typos, wording clarifications, reformatting — no semantic change |
| `MINOR` | New content added: new attributes, new acceptance criteria, expanded behavior — backward-compatible |
| `MAJOR` | Breaking change: renamed or removed fields, changed invariants, altered contracts that invalidate existing implementations |

Spec version is independent of any implementation version. A component built against `entity-{name}` at a specific version should reference that version explicitly in its own documentation if version pinning matters to the team.

---

### Status

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

## File Types

Each file belongs to a specific type that determines the information it contains and its relation to other files.
See the list of all file types and their layering:

| Type | Question answered | Technology-agnostic? | Cross-entity? |
|---|---|---|---|
| `glossary` | What does this specific term mean? | ✅ | ✅ |
| `concept` | How does this pattern or approach work, and why? | ✅ | ✅ |
| `entity` | What is this object and how does it behave alone? | ✅ | ❌ |
| `flow` | What happens across entity boundaries? | ✅ | ✅ |
| `component` | How is this implemented in a specific layer? | ❌ | — |
| `test` | How do we verify the spec is correctly implemented? | ❌ | — |

**File Types Layering**
```
glossary         ← term definitions, domain vocabulary
concept          ← patterns, architectural decisions (may reference glossary)
  └── entity     ← domain model, implementation-agnostic
       └── flow  ← cross-entity logic, policies, usecases
            └── component-{layer}  ← concrete implementation per layer
            └── test-{layer}       ← verification of spec per layer
```

Each layer depends on the ones above it. A `component` implements `entity` and `flow` definitions. A `flow` references `entity` definitions. An `entity` may reference `concept` definitions for shared terminology. A `concept` or `entity` may reference `glossary` entries for precise term definitions.

All the file types are defined in the following subchapters.

---

### `glossary`

**Question it answers:** *What does this specific term mean in this domain?*

Provides precise, unambiguous definitions for individual terms, abbreviations, or business vocabulary used across one or more packages. A glossary entry contains only meaning — no behavior, no pattern, no architectural rationale, no technology references. It is the most foundational spec type: everything else may reference it, but it references nothing.

**Distinction from `concept`:** A `concept` describes a pattern or approach that has rules, rationale, and behavioral implications. A `glossary` entry defines what a word or phrase means in this domain — nothing more.

**Use for:**
- Business jargon or abbreviations that need a canonical definition
- Terms that appear in multiple packages with the same precise meaning
- Domain vocabulary that cannot be inferred from common usage

---

### `concept`

**Question it answers:** *How does this pattern or approach work in our domain, and why?*

Describes a behavioral pattern, architectural approach, or system-wide policy applied consistently across entities. A concept carries rules, rationale, and implications — it explains not just what something is called, but how it works and why it was chosen.

A concept is completely technology-agnostic — it must never reference specific databases, storage engines, programming languages, frameworks, or any other implementation technology. How a pattern is realized in a specific stack belongs in a `component` spec.

**Use for:**
- Package overviews — each package has a `concept-{package}` file (named after the package) that is the first file to read in that package
- Patterns applied consistently across entities
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
- **Define before use.** Never use a domain term until it is defined earlier in the same document (or a linked glossary). Patterns may build on earlier patterns in the same file; they must not introduce undefined jargon.
- **Explain simply.** Write so a new reader can follow without prior jargon — short sentences, one idea at a time. Prefer defining a term in one sentence, then using it.
- **Process in order.** When a concept describes a procedure, introduce vocabulary first, then numbered steps (Step 1, Step 2, …). Do not jump into mid-process language before the terms exist.
- Keep precision. Simple wording must still state the real rules, formulas, and edge cases.

---

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
{attribute_name}:
  type:        {domain_type}
  source:      {source_value}
  constraints: [{constraint}, ...]
  description: {what this attribute represents}
```

**`source` values:**

| Value | Meaning |
|---|---|
| `user-provided` | Supplied directly by the actor initiating the action |
| `system-generated` | Created automatically by the system (e.g. UUID, timestamp) |
| `system-resolved` | Derived by the system from context (e.g. session or parent) |
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

---

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

#### Cross-Package Flows

Flows frequently span more than one package. The `package` field always contains a single string — the **owning package** (package root name), even when the file sits in a group subfolder. Cross-package dependencies are expressed through `references`, not through a multi-valued `package` field.

**Primary-package rule:** Place the flow under the package of the entity whose state is most fundamentally changed by the flow.

When the primary entity is ambiguous, apply these tie-breakers in order:

1. The entity that is *written to* — whose attribute or lifecycle state is mutated
2. The entity that is the *target* of the action, not the initiator
3. The more "downstream" or "owned" entity in the domain model

---

### `component`

**Question it answers:** *How is this implemented in a specific layer?*

A technology-specific **definition** of how entities and flows are realized in a given technical layer. It describes schemas, interfaces, signatures, and behavior in enough detail to generate or implement code elsewhere — it is not a place to store code. Tables, functions, endpoints, and services are specified here as definitions only; actual implementation files live under `app/…` and, when virtualization is used, `infra/<service>/<stack>/`.

**Naming convention:**
```
component-{layer}-{name}
```

**Layers:**

The following are common layers for typical software projects. Each project defines the layers relevant to its domain — this list is not exhaustive.

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
- Optional links to implementation paths under `app/…` or `infra/<service>/<stack>/` (never under `specs/`)

---

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

**Contains:**
- Reference to the flow or entity being tested
- Test cases with input, expected output, and expected errors
- Preconditions and setup requirements

---

## File References

Every file can include a `references` chapter — with list of paths to other spec files that are directly relevant to understanding or implementing the current spec.
Or reference another files directly in the text.

References make the dependency graph explicit and machine-readable, allowing both humans and AI tools to navigate from a definition to its implementation and from an implementation back to its specification.

---

### What to reference

| Type | Should reference |
|---|---|
| `glossary` | nothing (foundational layer; no upward references) |
| `concept` | `glossary` entries it uses; other `concept` files it builds upon (optional) |
| `entity` | `concept` files whose patterns or terminology it applies; `glossary` entries for key terms (optional) |
| `flow` | `entity` files involved in the flow |
| `component` | `entity` and `flow` files it implements; other `component` files it depends on |
| `test` | `flow`, `entity`, or `component` files it verifies |

---

### References Rules

- References are **unidirectional** — a higher-layer spec references lower-layer specs, not the reverse. A `component` references the `flow` it implements; the `flow` does not reference back to the `component`.
- References use the standard markdown links format and the **full path under `specs/`**, including groups when present. Package-root files: `[{package}/{type}-{name}]({package}/{type}-{name}.md)`. Files in a group: `[{package}/{group}/{type}-{name}]({package}/{group}/{type}-{name}.md)`. This maps directly to a file on disk.
- A spec with no meaningful references may omit the chapter or leave it as an empty list.
