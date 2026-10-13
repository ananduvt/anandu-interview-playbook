# AGENTS.md — Core Guidelines for Repository Editors

Read these instructions before adding or changing repository content. This repository maintains a structured interview preparation knowledge base in Markdown and publishes it with MkDocs.

## ⚠️ Publishing Model & Safety

- All content under `docs/` is **published to a public site** (GitHub Pages).
- **Never put employer/client-specific info in `docs/`**: no internal system/service names, endpoints, ticket IDs, hostnames, cluster names, account/client IDs, or internal URLs. Describe things generically (e.g., "an aggregation microservice", "a downstream transaction service").
- **Do not commit or push changes without explicit approval.**
- **Do not delete existing notes without approval.**

## Core File Management Rules

### File Size Targets & Splitting
- **Target range**: 300–600 lines per file (optimal for readability and maintenance)
- **When to split**: If a file grows beyond 600 lines, break into subtopics with `-advanced`, `-deep-dive`, or topic-specific suffixes
  - Example: `spring-boot.md` (fundamentals) + `spring-boot-advanced.md` (DI, lifecycle, profiles)
  - Add cross-references between files to guide readers

### Subtopic Limits per Section
- **Maximum subtopics per index page**: 6 subtopics
- **When to reorganize**: If a section exceeds 6 subtopics, create a new parent category
  - Example: If `java-core/` grows beyond 6 subtopics, split into `java-core/` (basics, OOP, exceptions) and `java-advanced/` (advanced topics)
  - Link both sections from the home page

### ✅ Mandatory Checklist: Updating Related Files After Any Changes

**Every content change must update related files.** Before committing:

1. **Section index pages** (e.g., `docs/spring-web/index.md`)
   - Add new topic with keywords and one-liner description
   - Update existing topic descriptions if content changed
   - Verify count does not exceed 6 subtopics

2. **Navigation in `mkdocs.yml`**
   - Register new files in the `nav:` section
   - Update section labels if topics were renamed
   - Maintain hierarchy (parent → children)

3. **Parent/overview pages** (e.g., `docs/index.md`)
   - Add links to new sections
   - Update section descriptions
   - Fix broken cross-references

4. **Internal cross-references**
   - Add "See also" links to related topics
   - Link to new/updated files from related files
   - Example: "See [Spring Boot Advanced](spring-boot-advanced.md) for..."

5. **File size check**
   - If file exceeds 600 lines, split into subtopics (`-advanced`, `-deep-dive`, etc.)
   - Add cross-references between split files

6. **Consistency**
   - Match existing formatting, style, and heading structure
   - Keep keyword-dash format in index pages (e.g., "DI - Lifecycle - Profiles")
   - Preserve one-liner descriptions for quick scanning

### Directory Structure
- Add new pages under `docs/` and register them in `mkdocs.yml` `nav`.
- Keep `docs/index.md` as the landing/welcome page.

## Workflow
- Work on a branch, open a PR, get a review, and merge to `main`. The public site rebuilds on merge.
- Check Markdown formatting and internal links, validate MkDocs configuration and navigation, and run a documentation build after meaningful documentation changes.
- Summarize changed files and any outstanding issues when reporting completed work.

---

**For content style, patterns, and writing guidance**, see [CONTRIBUTING.md](CONTRIBUTING.md).
