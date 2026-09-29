# Software Design Principles

## SOLID (object-oriented design)
- **S — Single Responsibility**: a class should have one reason to change.
- **O — Open/Closed**: open for extension, closed for modification.
- **L — Liskov Substitution**: subtypes must be substitutable for their base types without breaking correctness.
- **I — Interface Segregation**: clients shouldn't depend on interfaces they don't use — prefer small, focused interfaces.
- **D — Dependency Inversion**: high-level modules shouldn't depend on low-level modules; both depend on abstractions.

## Other core principles
- **DRY (Don't Repeat Yourself)** — avoid duplication via reusable abstractions.
- **KISS (Keep It Simple)** — simplest design that works; favor readability.
- **YAGNI (You Aren't Gonna Need It)** — don't build features until they're actually needed.
- **Law of Demeter** — talk only to your immediate dependencies; minimize coupling ("don't talk to strangers").

## Why they matter (interview framing)
These principles reduce coupling and increase cohesion, which makes code easier to change, test, and extend.
Be ready to give a one-line example of each — especially SRP, OCP, and DIP, which come up most.
