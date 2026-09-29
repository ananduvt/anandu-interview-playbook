# SDLC & Methodologies

## SOLID principles
See [Design Principles](design-principles.md) for SOLID, DRY, KISS, YAGNI, Law of Demeter.

## Project management methodologies
### Waterfall
- Sequential phases: requirements → design → implementation → testing → deployment → maintenance.
- Each phase completes before the next starts. Predictable, heavy up-front planning.
- **Weakness**: inflexible to change; issues surface late.

### Agile
- Iterative, incremental delivery in short sprints; embraces changing requirements.
- Frameworks: **Scrum** (sprints, standups, backlog, retro), **Kanban** (continuous flow, WIP limits).
- Values working software, collaboration, and responding to change over rigid plans.

## BDD & TDD
- **TDD (Test-Driven Development)**: write a failing test → make it pass → refactor (red-green-refactor).
  Drives design, gives a safety net.
- **BDD (Behavior-Driven Development)**: describe behavior in business language (Given-When-Then,
  Gherkin/Cucumber). Bridges devs, QA, and business.

## Semantic Versioning (SemVer)
`MAJOR.MINOR.PATCH`:
- **MAJOR** — incompatible/breaking API changes.
- **MINOR** — backward-compatible new features.
- **PATCH** — backward-compatible bug fixes.
- Pre-release/build metadata: `1.4.0-beta.1+build.5`.

## Programming paradigms
- **Imperative / Procedural** — step-by-step statements (C).
- **Object-Oriented** — objects encapsulating state + behavior (Java).
- **Functional** — pure functions, immutability, no side effects (streams, lambdas).
- **Declarative** — describe *what*, not *how* (SQL, HTML).

## Environments
`dev → test/QA → UAT → staging → prod` — progressively production-like; config externalized per env.
