# CONTRIBUTING.md — Guidelines for All Contributors

**Read this before editing, adding content, or contributing to this repository.** This single file contains all standards, patterns, rules, and workflow needed to work on the interview playbook.

---

## 🎯 Repository Overview

This is a structured interview preparation knowledge base published via MkDocs to GitHub Pages. All content under `docs/` is **public**. The repository follows strict standards to ensure consistency, quality, and safety across all contributions.

---

## ⚠️ Publishing Model & Safety

### Content Visibility
- All content under `docs/` is **published to a public site** (GitHub Pages).
- Only content intended for public consumption belongs in `docs/`.
- Internal guidelines, format instructions, and meta-content belong in this file or in root-level documentation files.

### What NOT to Include in Published Content
- **No employer/client-specific information**: no internal system/service names, endpoints, ticket IDs, hostnames, cluster names, account/client IDs, or internal URLs
- Use generic descriptions instead (e.g., "an aggregation microservice", "a downstream transaction service", "our payment provider")
- **No internal editing notes or format instructions** (those go in CONTRIBUTING.md, not published pages)
- **No meta-guidance** like "how to write questions" on published pages

### Workflow Constraints
- **Do not commit or push changes without explicit approval** from the repository owner
- **Do not delete existing content without approval**
- All changes must follow the mandatory checklist before committing

---

## 🏗️ File Management Rules

### File Size Targets
- **Target range**: 300–600 lines per file (optimal for readability and maintenance)
- Files over 600 lines must be split into subtopics

### Splitting Large Files
- Use suffixes: `-advanced`, `-deep-dive`, `-fundamentals`, or topic-specific names
- **Example**: `spring-boot.md` (fundamentals) + `spring-boot-advanced.md` (DI, lifecycle, profiles)
- Add cross-references between split files (use "See also:" sections)

### Subtopic Limits per Section
- **Maximum 6 subtopics per index page** (e.g., `docs/spring-web/index.md`)
- If a section exceeds 6 subtopics, create a new parent category
- **Example**: If `java-core/` grows beyond 6 topics, split into `java-core/` (basics, OOP, exceptions) and `java-advanced/` (advanced topics), then link both from home page

### Directory Structure
- Add new pages under `docs/` with descriptive names
- Register all new files in `mkdocs.yml` under the `nav:` section
- Keep `docs/index.md` as the landing/welcome page
- Organize by topic area (e.g., `docs/spring-web/`, `docs/java-core/`, `docs/microservices-distributed/`)

---

## 📝 Content Style & Writing Standards

### General Principles
- **One topic per Markdown file**: Keep pages focused and scannable
- **Use structured formatting**: headings, bullet points, tables, admonitions over long prose
- **Prefer concrete, interview-useful explanations**: Include likely follow-up questions and edge cases
- **Ground in reality**: Use project experience and verified facts; do not invent technical details
- **Distinguish verified facts from general knowledge**: Label user-specific experience separately
- **Prioritize examples**: Show real Java, Spring, or deployment code when feasible
- **Preserve existing content**: Update instead of creating duplicates; keep existing structure and style

### Markdown Formatting
- Use standard Markdown headings (`#`, `##`, `###`, etc.)
- Use code blocks with language tags (` ```java `, ` ```yaml `, etc.)
- Use admonitions for notes: `!!! note "Title"`
- Use Mermaid diagrams for flows: ` ```mermaid ... ``` `
- Use tables for comparisons (2–3 columns max for readability)

### Cross-References & Internal Links
- Link to related topics using "See also:" sections at the end of files
- Use relative paths: `[Link text](../other-section/file.md)`
- Verify all links point to existing files (broken links break navigation)

---

## 🎓 Content Patterns by Topic Type

### Annotations & API References
When documenting APIs, annotations, or large feature sets:

- **Summary table**: List annotation/API with purpose (2–3 columns)
- **Code examples**: Show constructor, setter, and field patterns; highlight best practices (constructor first)
- **Configuration YAML**: Provide realistic config with comments explaining each field
- **Profile-specific examples**: Show dev vs. prod configurations when relevant

**Example**: See `docs/spring-web/spring-boot.md` (common annotations section)

### Design Patterns & Architecture
When documenting patterns (microservices, distributed systems, deployment strategies):

- **Organize by category**: Group similar patterns (resilience patterns, communication patterns, data patterns)
- **Pattern structure**: motivation + phases + example + trade-offs
- **Code examples**: Show real Spring/Java code; include both framework-native and custom implementations
- **Comparison tables**: When multiple similar patterns exist, contrast by throughput, latency, cost, rollback speed
- **Diagrams**: Use Mermaid for flows (event choreography, state transitions, deployment cutover)

**Example**: See `docs/microservices-distributed/principles.md` (design patterns, deployment patterns sections)

### Performance & Tuning
When documenting performance, GC, caching, or profiling:

- **Issue → Root Cause → Fix table**: Format for quick lookup
- **Real config examples**: Show JVM flags, YAML tuning parameters with concrete values
- **Tools section**: List tools with specific use cases and one-liner commands
- **Before/After**: Show poor pattern vs. optimized pattern (e.g., N+1 query problem)

**Example**: See `docs/java-patterns-advanced/advanced.md` (GC section) and `docs/java-patterns-advanced/logging-performance.md` (performance tuning)

### Interview Questions (Verbal Q&A)
Interview questions use a specific collapsible format for interview prep readability:

```html
<details>
<summary>What is a design pattern?</summary>

**Answer:** A reusable solution to a common design problem...

📚 [Learn more: Design Patterns](../../path/to/topic.md)
</details>
```

**Structure requirements:**
1. **Summary tag**: Concise question (no colon, no "Answer:" prefix)
2. **Answer block**: 3–4 lines maximum for the minimal answer
   - Bold the opening statement
   - Use bullet points for key points if needed
3. **Topic link**: Emoji + link to detailed topic (e.g., `📚 [Learn more: Design Patterns](../../java-core/design-patterns.md)`)

**Why this format:**
- Minimal, scannable view (collapsed by default)
- Readers expand only questions they need
- Clean structure for interview prep
- Keeps page readable without overwhelming content

**When writing Q&A pages:**
- Organize questions by category with brief headers
- Include 3–5 collapsible Q&A per category (or more if grouped well)
- Use "See also:" at the end to link to detailed topic pages
- Do NOT include format instructions on published pages (this belongs in CONTRIBUTING.md)

**File naming:** `{topic}-questions.md` (e.g., `verbal-java-core-questions.md`, `verbal-spring-boot-questions.md`)

### Section Index Pages
When creating or updating a section index (e.g., `docs/spring-web/index.md`):

- **Brief intro** (1–2 sentences) on the topic area
- **Topic list** with keywords and one-liner descriptions for quick scanning
- **Optional**: Quick Interview Prep section (bullet points of essentials to memorize)

**Example**: See `docs/spring-web/index.md`

---

## ✅ Mandatory Checklist Before Committing

**Every content change requires updating related files.** Complete this checklist before committing:

### 1. Section Index Pages
- [ ] Add new topics with keywords and one-liner descriptions
- [ ] Update existing topic descriptions if content changed
- [ ] Verify count does not exceed 6 subtopics
- [ ] Use keyword-dash format (e.g., "DI - Lifecycle - Profiles")

### 2. Navigation Configuration
- [ ] Register new files in `mkdocs.yml` under the `nav:` section
- [ ] Update section labels if topics were renamed
- [ ] Maintain hierarchy (parent → children)
- [ ] Test navigation locally (no broken internal links)

### 3. Parent/Overview Pages
- [ ] Update `docs/index.md` with links to new sections
- [ ] Update section descriptions if content changed significantly
- [ ] Fix any broken cross-references
- [ ] Verify relative paths are correct

### 4. Internal Cross-References
- [ ] Add "See also" links to related topics
- [ ] Update related files that should link to your new content
- [ ] Example: "See [Spring Boot Advanced](spring-boot-advanced.md) for..."
- [ ] Use relative paths accounting for file depth

### 5. File Size & Structure
- [ ] If file exceeds 600 lines, split into subtopics (`-advanced`, `-deep-dive`, etc.)
- [ ] Add cross-references between split files
- [ ] Verify consistent formatting with existing files in the section

### 6. Consistency Check
- [ ] Match existing formatting, style, and heading structure
- [ ] Follow the content patterns specified above
- [ ] Preserve one-liner descriptions for quick scanning
- [ ] Verify code examples follow project conventions

### 7. Markdown & Link Validation
- [ ] Check all Markdown formatting (no syntax errors)
- [ ] Verify all internal links exist and use correct relative paths
- [ ] Run local MkDocs build: `mkdocs serve` and test navigation in browser
- [ ] Verify no 404 errors when clicking links

---

## 🔄 Workflow

### Local Development
1. **Clone the repository** and create a feature branch
2. **Edit files** in `docs/` following the patterns above
3. **Run locally**: `mkdocs serve` (builds site locally at `http://localhost:8000`)
4. **Test navigation**: Click all links and verify they work
5. **Complete the mandatory checklist** before committing

### Before Committing
- Verify all changes follow this guide
- Ensure the mandatory checklist is complete
- Summarize which files changed and why

### Committing & Pushing
- Create clear commit messages (e.g., "Add Spring Boot dependency injection Q&A")
- **Do not commit or push without explicit approval from the repository owner**
- Link to issues or PRs if applicable
- Include co-authorship if working with others

### After Merging
- The public site rebuilds automatically on merge to `main`
- Changes go live within minutes
- Monitor for any broken links or formatting issues

---

## 💡 Quick Reference for AI Contributors

**If you're an AI agent adding content to this repository:**

1. **Read this entire file first** — all standards and patterns are here
2. **Before editing any file**:
   - Check existing files in the same section for style consistency
   - Verify file size (if adding to a file over 500 lines, consider splitting)
   - Identify related files that might need cross-references

3. **When adding new content**:
   - Follow the content patterns matching your topic type
   - Update section index pages (docs/*/index.md)
   - Update mkdocs.yml navigation
   - Add cross-references to related topics
   - Run through the mandatory checklist

4. **When editing existing content**:
   - Preserve existing structure unless explicitly asked to change it
   - Update related files if description or structure changed
   - Verify all links still work
   - Don't delete content without explicit approval

5. **Before the final commit**:
   - Summarize which files changed and why
   - Report any outstanding issues or decisions
   - Wait for explicit approval before committing or pushing

---

**Questions?** Refer back to the relevant section above. This file contains everything needed to contribute confidently.
