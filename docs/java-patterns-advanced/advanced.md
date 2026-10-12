# Advanced Java

## Modern language features
- **Records** (16) — immutable data carriers: `record Point(int x, int y) {}` (auto ctor/getters/equals/hashCode/toString).
- **Sealed classes** (17) — restrict which types can extend/implement (`sealed ... permits ...`).
- **Pattern matching** — `instanceof` patterns, `switch` patterns, **record patterns** (destructuring).
- **Text blocks** (15) — multi-line string literals `"""..."""`.
- **`var`** (10) — local type inference (still statically typed).

## Generics
- Type safety at compile time; **type erasure** at runtime (no reified generics).
- Bounded wildcards — **PECS**: `<? extends T>` (producer/read), `<? super T>` (consumer/write).

## Annotations
- Metadata; **marker** (no members, e.g. `@Override`), single-value, full.
- Retention: SOURCE / CLASS / **RUNTIME** (readable via reflection).
- Custom: `@Retention @Target @interface Audit {}`; processed by frameworks (Spring) or annotation processors (Lombok).

## Reflection
Inspect/invoke types at runtime (`Class`, `Method`, `Field`). Powerful but slow + breaks encapsulation — used by frameworks.

## Other
- **`Optional`** — null-safety (don't overuse).
- **equals/hashCode** contract; **immutability** as a design + concurrency tool.
- **Serialization** — `Serializable`, `serialVersionUID`; prefer JSON for interop.
- **Atomic reference**, `ThreadLocal`, **Cleaner** (replaces finalize).
