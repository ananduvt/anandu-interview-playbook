# RegEx Cheatsheet

## Character classes
| Pattern | Matches |
|---------|---------|
| `.` | any character except newline |
| `\w` `\d` `\s` | word, digit, whitespace |
| `\W` `\D` `\S` | not word, digit, whitespace |
| `[abc]` | any of a, b, or c |
| `[^abc]` | not a, b, or c |
| `[a-g]` | character between a & g |

## Anchors
| Pattern | Matches |
|---------|---------|
| `^abc$` | start / end of string |
| `\b` `\B` | word / not-word boundary |

## Escaped characters
| Pattern | Matches |
|---------|---------|
| `\.` `\*` `\\` | escaped special characters |
| `\t` `\n` `\r` | tab, linefeed, carriage return |

## Groups & lookaround
| Pattern | Meaning |
|---------|---------|
| `(abc)` | capture group |
| `\1` | backreference to group #1 |
| `(?:abc)` | non-capturing group |
| `(?=abc)` | positive lookahead |
| `(?!abc)` | negative lookahead |

## Quantifiers & alternation
| Pattern | Meaning |
|---------|---------|
| `a*` `a+` `a?` | 0+, 1+, 0 or 1 |
| `a{5}` `a{2,}` | exactly 5, two or more |
| `a{1,3}` | between one & three |
| `a+?` `a{2,}?` | lazy — match as few as possible |
| `ab|cd` | match ab or cd |

## Java notes
- Escape backslashes in string literals: `"\\d+"`. Use `Pattern`/`Matcher`, or `String.matches/replaceAll/split`.
- Prefer pre-compiled `Pattern` for reuse; `matches()` anchors the whole input implicitly.
