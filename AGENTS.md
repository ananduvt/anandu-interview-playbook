# AGENTS.md — Instructions for repository editors

Read these instructions before adding or changing repository content. This repository maintains a structured interview preparation knowledge base in Markdown and publishes it with MkDocs.

## Publishing model
- All content under `docs/` is **published to a public site** (GitHub Pages).
- **Never put employer/client-specific info in `docs/`**: no internal system/service names, endpoints, ticket IDs, hostnames, cluster names, account/client IDs, or internal URLs. Describe things generically (e.g. "an aggregation microservice", "a downstream transaction service").
- Use real-world examples and general technical knowledge; prioritize interview-useful content.

## Content style
- One topic per Markdown file. Keep pages focused and scannable.
- Use headings, bullet points, tables, and admonitions (`!!! note`) over long prose.
- Mermaid diagrams are supported via fenced ```mermaid blocks.
- Prefer concrete, interview-useful explanations with likely follow-up questions.
- Prioritize interview questions grounded in user-provided or documented project experience. Do not invent details about the user's work or projects.
- Distinguish verified project facts from general technical knowledge. Include concise explanations, technical detail, trade-offs, production considerations, and practical Java or Spring Boot examples where relevant.
- Prefer updating existing topic files over creating duplicates, and follow the existing directory structure and writing style.
- Preserve existing content unless the requested change requires editing it.

## Content patterns for complex topics

### Annotations & API References
When documenting APIs, annotations, or large feature sets:
- **Summary table**: List annotation/API with purpose (2–3 columns max)
- **Code examples**: Show constructor, setter, and field patterns; highlight best practices (constructor first)
- **Configuration YAML**: Provide realistic config with comments explaining each field
- **Profile-specific examples**: Show dev vs. prod configurations when relevant
- **Example**: See `docs/spring-web/spring-boot.md` (common annotations section)

### Design Patterns & Architecture
When documenting patterns (microservices, distributed systems, deployment strategies):
- **Organize by category**: Group similar patterns (e.g., resilience patterns, communication patterns, data patterns)
- **One pattern = motivation + phases + example + trade-offs**
- **Code example**: Show real Spring/Java code when feasible; include both framework-native and custom implementations
- **Comparison table**: When multiple similar patterns exist, contrast by throughput, latency, cost, rollback speed, etc.
- **Diagrams**: Use Mermaid for flow (event choreography, state transitions, deployment cutover)
- **Example**: See `docs/microservices-distributed/principles.md` (design patterns, deployment patterns sections)

### Performance & Tuning
When documenting performance, GC, caching, or profiling:
- **Issue → Root Cause → Fix** table format for quick lookup
- **Real config examples**: Show JVM flags, YAML tuning parameters, with concrete values
- **Tools section**: List tools with their specific use case and one-liner command
- **Before/After**: Show poor pattern vs. optimized pattern (e.g., N+1 query problem)
- **Example**: See `docs/java-patterns-advanced/advanced.md` (GC section), `docs/java-patterns-advanced/logging-performance.md` (performance tuning)

## File size targets & splitting strategy
- **Target range**: 300–600 lines per file (optimal for readability and maintenance)
- **When to split**: If a file grows beyond 600 lines, break into subtopics with `-advanced`, `-deep-dive`, or topic-specific suffixes
  - Example: `spring-boot.md` (fundamentals) + `spring-boot-advanced.md` (DI, lifecycle, profiles)
  - Add cross-references between files to guide readers
- **When to keep together**: Related concepts that readers need simultaneously (e.g., all Spring Boot annotations in one file with cross-references)

## Updating related files after content changes
When adding or significantly modifying content:
1. **Update section index pages** (e.g., `spring-web/index.md`) with new one-liners and keywords for new topics
2. **Register new files in `mkdocs.yml`** navigation if creating separate files
3. **Update parent/overview pages** with links and descriptions
4. **Cross-reference** between related topics (e.g., "See [Spring Boot Advanced](spring-boot-advanced.md) for...")
5. **Check file sizes** and split if exceeding 600 lines before committing

## Structure
- Add new pages under `docs/` and register them in `mkdocs.yml` `nav`.
- Keep `docs/index.md` as the landing/welcome page.

## Workflow
- Work on a branch, open a PR, get a review, and merge to `main`. The public site rebuilds on merge.
- Check Markdown formatting and internal links, validate MkDocs configuration and navigation, and run a documentation build after meaningful documentation changes.
- Summarize changed files and any outstanding issues when reporting completed work.

## Safety
- Do not commit or push changes without explicit approval.
- Do not delete existing notes without approval.
