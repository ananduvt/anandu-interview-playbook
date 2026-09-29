# OOP Concepts

## Four pillars
- **Encapsulation** — hide internal state; expose via methods; protects invariants.
- **Abstraction** — expose *what*, hide *how* (abstract classes / interfaces).
- **Inheritance** — reuse + specialize via "is-a" (`extends`).
- **Polymorphism** — one interface, many forms: **overriding** (runtime) vs **overloading** (compile-time).

## Abstraction vs inheritance
- **Abstraction** reduces complexity by hiding detail (design-time).
- **Inheritance** reuses code via "is-a". Prefer **composition over inheritance** for flexibility.

## Class relationships
- **Association** — general "uses-a".
- **Aggregation** — "has-a", independent lifecycles (Department has Professors).
- **Composition** — "has-a", dependent lifecycle (House has Rooms; rooms die with the house).
- **Inheritance** — "is-a".

## Interface vs abstract class
| | Interface | Abstract class |
|--|-----------|----------------|
| Multiple inheritance | yes | no (single) |
| State | constants + (default/static/private) methods | fields + constructors |
| Use | capability/contract | shared base with partial impl |

## Diamond problem
Multiple inheritance ambiguity. Java avoids it for classes (single inheritance). With interface **default
methods**, if two interfaces provide the same default, the class **must override** and can call
`InterfaceName.super.method()`.

## Related
- `final` (no override/extend/reassign), `static` (class-level), `this`/`super`.
- SOLID → [Design Principles](../foundations/design-principles.md).
