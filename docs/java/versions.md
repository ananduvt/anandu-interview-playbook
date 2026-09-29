# Java Versions & Features

Since Java 9, releases ship every 6 months; **LTS** releases (long-term support) land every ~2–3 years
(8, 11, 17, 21). Know the LTS versions and the headline features per release.

| Version | Year | Headline features |
|---------|------|-------------------|
| 1.0 | 1996 | Initial release: classes, objects, inheritance, interfaces, JVM |
| 1.1 | 1997 | Inner classes, JDBC, improved AWT/event handling |
| 1.2 (J2SE) | 1998 | **Collections Framework**, Swing, JNDI |
| 1.3 | 2000 | HotSpot JVM, Java Sound API |
| 1.4 | 2002 | `assert`, regular expressions, NIO |
| 5 | 2004 | **Generics**, enums, autoboxing, enhanced for-loop, **annotations** |
| 6 | 2006 | Performance, scripting (Compiler API), JAXB/web services |
| 7 | 2011 | try-with-resources, diamond `<>`, multi-catch |
| 8 | 2014 | **Lambdas, Stream API, `java.time`**, default methods (LTS) |
| 9 | 2017 | Module system (Jigsaw), HTTP/2 client (incubator), JShell |
| 10 | 2018 | `var` (local-variable type inference) |
| 11 | 2018 | Standard **HTTP client**, run single-file source, removed Java EE (LTS) |
| 12–13 | 2019 | switch expressions (preview), text blocks (preview) |
| 14 | 2020 | records (preview), pattern matching for `instanceof` (preview), helpful NPEs |
| 15 | 2020 | text blocks (standard), sealed classes (preview) |
| 16 | 2021 | records (standard), pattern matching for `instanceof` (standard) |
| 17 | 2021 | **sealed classes (standard)** (LTS) |
| 21 | 2023 | **Virtual threads (Loom)**, record patterns, pattern matching for `switch`, sequenced collections (LTS) |

## Interview tips
- Know the **LTS** line: **8, 11, 17, 21**.
- Be ready to explain the marquee features: **8** (lambdas/streams), **11** (HTTP client, `var` from 10),
  **17** (sealed classes), **21** (**virtual threads**).
- Since Java 9, Oracle uses a **6-month cadence** with LTS every few years.
