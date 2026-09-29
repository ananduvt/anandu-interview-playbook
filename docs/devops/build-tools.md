# Build Tools

## Gradle vs Maven
| Basis | Gradle | Maven |
|-------|--------|-------|
| Config | Groovy/Kotlin **DSL** | **XML** (`pom.xml`) |
| Model | task dependency graph | fixed linear phases (lifecycle) |
| Performance | faster (incremental builds, build cache, daemon) | slower (no incremental by default) |
| Flexibility | highly customizable | convention-heavy |
| Learning curve | steeper | familiar/mature |
| Languages | Java, Groovy, Kotlin, C/C++ | Java (others via plugins) |

## Maven basics
- **Lifecycle phases**: validate → compile → test → package → verify → install → deploy.
- `pom.xml` declares dependencies, plugins, build config. Local repo `~/.m2`.

## Gradle basics
- `build.gradle` with tasks; dependency configurations (`implementation`, `api`, `testImplementation`, `runtimeOnly`).
- Incremental builds + build cache + daemon for speed.
- Wrapper (`gradlew`) pins the Gradle version per project.

## Dependency scopes (Gradle)
- `implementation` — compile + runtime, not exposed to consumers.
- `api` — exposed transitively.
- `compileOnly` / `runtimeOnly` / `testImplementation`.

## Interview points
- Transitive dependencies + conflict resolution; version pinning for CVE remediation.
- Reproducible builds; separate compile vs runtime classpaths.
