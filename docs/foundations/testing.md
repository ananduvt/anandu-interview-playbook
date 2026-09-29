# Testing / QA

## Test pyramid
- **Unit** (most) — fast, isolated, mock dependencies.
- **Integration** — components together (DB, external services / virtualization).
- **End-to-end** (fewest) — full system through the UI/API; slow, brittle.

## Test types
- Functional: unit, integration, system, acceptance/UAT, smoke, regression.
- Non-functional: performance/load, stress, security, usability.

## Principles
- **AAA** — Arrange, Act, Assert.
- Test **behavior**, not implementation. One logical assertion per test.
- Deterministic, independent, fast. Name tests by intent.

## TDD vs BDD
- **TDD**: red → green → refactor. Test drives design.
- **BDD**: Given-When-Then behavior specs in business language (Cucumber/Gherkin).

## Java tooling
- **JUnit 5** (Jupiter), **Mockito** (mocks/stubs/spies), **AssertJ/Hamcrest** (assertions).
- `@SpringBootTest` (integration), **MockMvc** (controllers), Testcontainers (real deps in Docker).
- Coverage tools (JaCoCo) — coverage is a guide, not a goal.

## Test doubles
Dummy · stub (canned answers) · spy (records calls) · mock (verifies interactions) · fake (working lightweight impl).
