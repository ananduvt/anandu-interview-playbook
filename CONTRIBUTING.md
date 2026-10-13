# CONTRIBUTING.md — Content Style & Patterns

This guide shows best practices and patterns for writing interview playbook content. Use these as references, not strict rules.

## Content Style

- One topic per Markdown file. Keep pages focused and scannable.
- Use headings, bullet points, tables, and admonitions (`!!! note`) over long prose.
- Mermaid diagrams are supported via fenced ```mermaid blocks.
- Prefer concrete, interview-useful explanations with likely follow-up questions.
- Prioritize interview questions grounded in user-provided or documented project experience. Do not invent details about the user's work or projects.
- Distinguish verified project facts from general technical knowledge. Include concise explanations, technical detail, trade-offs, production considerations, and practical Java or Spring Boot examples where relevant.
- Prefer updating existing topic files over creating duplicates, and follow the existing directory structure and writing style.
- Preserve existing content unless the requested change requires editing it.

## Content Patterns by Topic Type

### Annotations & API References
When documenting APIs, annotations, or large feature sets:
- **Summary table**: List annotation/API with purpose (2–3 columns max)
- **Code examples**: Show constructor, setter, and field patterns; highlight best practices (constructor first)
- **Configuration YAML**: Provide realistic config with comments explaining each field
- **Profile-specific examples**: Show dev vs. prod configurations when relevant
- **Example**: See [`docs/spring-web/spring-boot.md`](docs/spring-web/spring-boot.md) (common annotations section)

### Design Patterns & Architecture
When documenting patterns (microservices, distributed systems, deployment strategies):
- **Organize by category**: Group similar patterns (e.g., resilience patterns, communication patterns, data patterns)
- **One pattern = motivation + phases + example + trade-offs**
- **Code example**: Show real Spring/Java code when feasible; include both framework-native and custom implementations
- **Comparison table**: When multiple similar patterns exist, contrast by throughput, latency, cost, rollback speed, etc.
- **Diagrams**: Use Mermaid for flow (event choreography, state transitions, deployment cutover)
- **Example**: See [`docs/microservices-distributed/principles.md`](docs/microservices-distributed/principles.md) (design patterns, deployment patterns sections)

### Performance & Tuning
When documenting performance, GC, caching, or profiling:
- **Issue → Root Cause → Fix** table format for quick lookup
- **Real config examples**: Show JVM flags, YAML tuning parameters, with concrete values
- **Tools section**: List tools with their specific use case and one-liner command
- **Before/After**: Show poor pattern vs. optimized pattern (e.g., N+1 query problem)
- **Example**: See [`docs/java-patterns-advanced/advanced.md`](docs/java-patterns-advanced/advanced.md) (GC section), [`docs/java-patterns-advanced/logging-performance.md`](docs/java-patterns-advanced/logging-performance.md) (performance tuning)

## Section Index Pages

When creating or updating a section, include an index page (e.g., `spring-web/index.md`) with:
- **Brief intro** (1–2 sentences) on the topic area
- **Topic list** with keywords and one-liner descriptions for quick scanning
- **Quick Interview Prep** section (bullet points of essentials to memorize)

**Example**: See [`docs/spring-web/index.md`](docs/spring-web/index.md)

## Before Committing: Always Check Related Files

After making content changes, verify you've updated:

- ✅ Section index pages (`docs/*/index.md`) with new/updated topic descriptions
- ✅ `mkdocs.yml` navigation if adding/removing files
- ✅ `docs/index.md` home page with links to new sections
- ✅ Cross-references between related topics
- ✅ File sizes (no file should exceed 600 lines)

**See [AGENTS.md](AGENTS.md) for the full mandatory checklist.**

## Questions?

Refer to [AGENTS.md](AGENTS.md) for core file management rules, safety guidelines, and workflow.
