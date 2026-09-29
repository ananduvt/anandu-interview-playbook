# Strings & Primitives

## Primitive types
`byte`(8) `short`(16) `int`(32) `long`(64) `float`(32) `double`(64) `char`(16) `boolean`. Have wrapper
classes (`Integer`, `Double`…) for use in collections/generics.

## Integer cache
Autoboxing caches `Integer` values **-128..127** → `==` is true in that range, false outside. Always compare
wrappers with `.equals()`.
```java
Integer a = 127, b = 127;   // a == b  -> true (cached)
Integer c = 128, d = 128;   // c == d  -> false (new objects)
```

## String pool & immutability
- **String is immutable** — thread-safe, cacheable, safe as map keys. Any "modification" creates a new String.
- **String pool** (in heap): string *literals* are interned and reused; `new String("x")` creates a new object.
  `"a" == "a"` (pooled) but `new String("a") != "a"`. Use `.intern()` to pool explicitly.

## String vs StringBuilder vs StringBuffer
| | String | StringBuilder | StringBuffer |
|--|--------|---------------|--------------|
| Mutable | no | yes | yes |
| Thread-safe | (immutable) | **no** | **yes** (synchronized) |
| Speed | — | fast | slower |
| Since | 1.0 | 1.5 | 1.0 |
Use **StringBuilder** for single-threaded concatenation in loops; StringBuffer only when shared across threads.

## Overflow
`int` wraps around silently: `Integer.MAX_VALUE + 1 == Integer.MIN_VALUE`. Use `long`/`Math.addExact` to guard.
`0.0/0.0` → `NaN`; `1/0` (int) → `ArithmeticException`.
