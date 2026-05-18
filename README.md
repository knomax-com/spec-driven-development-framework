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


