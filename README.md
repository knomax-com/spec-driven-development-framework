# Spec-Driven Development Framework

Spec-Driven Development Framework is a software development approach where the specification is the **primary artifact** and implementation is derived from it.

Instead of starting with code, teams start by clearly defining:

- the problem to solve — concept
- entities (optional) — logical objects and their behavior
- expected behavior between entities (optional)
- specific desired implementation (optional)
- testing scenarios (optional)

Then humans or AI tools use that specification to generate, implement, test, and evolve the system.

**Advantage:** you describe the project in natural language (ideas, goals, constraints). The framework turns that into **technology-agnostic** specifications that are readable by **humans and AI**. Those specs can be reviewed, then implemented independently of any particular stack or tooling. Spec files stay as the **structured app definition** — the single source of truth.

> **The spec is the single source of truth. Code is an implementation of the spec.**

Specifications are **Markdown files with YAML front matter**, organized into folders by domain package. You control how much detail to include, separately for each package.

Full reference (naming, front matter, file types): [`template/specs/Spec-Driven Development Framework.md`](template/specs/Spec-Driven%20Development%20Framework.md)

---

## What you need

| Item | Required? | Notes |
|---|---|---|
| `specs/` folder | **Yes** | The only mandatory folder. Holds SDD definition Markdown (`*.md`) only. |
| `spec-driven-development.mdc` | **Yes** (for AI-assisted use) | Place in your AI tool’s **rules** directory. Exact path depends on the tool. |
| `app/`, `docs/`, `infra/`, etc. | No | Optional layout from [`template/`](template/). Useful as you grow; not required to start. |

---

## How to adopt this framework

1. **Create `specs/`** at the root of your application or repository. This is the only mandatory folder.
2. **Copy the framework reference** into that folder:
   - From: `template/specs/Spec-Driven Development Framework.md`
   - To: `specs/Spec-Driven Development Framework.md`
3. **Install the AI rules file** so your assistant follows SDD:
   - From: [`template/rules/spec-driven-development.mdc`](template/rules/spec-driven-development.mdc)
   - To: your AI tool’s **rules** location (tool-specific). Examples:
     - Cursor: `.cursor/rules/spec-driven-development.mdc`
     - Other tools: often `rules/`, `.github/`, or an equivalent project rules folder — use whatever that tool documents.
4. **Add an entry-point concept** for the application/repository, e.g. `specs/concept-{application-or-repo-name}.md`. You can ask the AI to create it from your rough ideas — you do not need to write it by hand. Start reading from there.
5. **Grow by package** under `specs/{package}/` as needed. Add only the file types you need (`entity`, `flow`, `component`, `test`, …). Cross-package specs with no clear owner stay at the `specs/` root.
6. **Implement against approved specs.** When behavior, schema, or interfaces change, update the relevant spec first (or in the same change), then update the code.

Optional: copy other folders from [`template/`](template/) (`app/`, `docs/`, `infra/`, …) if you want the recommended repository layout described in the rules file and framework doc.

---

## How to use the framework

You do **not** need to create every specs file manually.

1. Complete the adoption steps above (especially `specs/` and the rules file).
2. Tell the AI your rough ideas: the problem, domain, goals, and desired behavior — in natural language.
3. Ask it to create or update specs under `specs/` using this framework (naming, front matter, file types).
4. Review the generated specs. Adjust wording or status until they match your intent (`approved` when ready to build against).
5. Implement — yourself or via AI — against those specs. When intent changes, update the specs first, then the code.

The specs remain the structured app definition and single source of truth, independent of technology.

---

## License

This software is available under the [PolyForm Noncommercial License 1.0.0](LICENSE).

Non-commercial use is permitted under those terms.

Commercial use requires a separate license. For commercial licensing inquiries, contact: paulservice79@gmail.com
