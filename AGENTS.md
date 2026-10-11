# AGENTS.md — Instructions for repository editors

Read these instructions before adding or changing repository content. This repository maintains a structured interview preparation knowledge base in Markdown and publishes it with MkDocs.

## Publishing model
- Files under `docs/` are **published to a public site**. Files under `internal/` are **private** (never built).
- **Never put employer/client-specific info in `docs/`**: no internal system/service names, endpoints,
  ticket IDs, hostnames, cluster names, account/client IDs, or internal URLs. Describe things generically
  (e.g. "an aggregation microservice", "a downstream transaction service").
- Internal/context-only material (with real names, tickets, etc.) may live in `internal/` for reference.

## Content style
- One topic per Markdown file. Keep pages focused and scannable.
- Use headings, bullet points, tables, and admonitions (`!!! note`) over long prose.
- Mermaid diagrams are supported via fenced ```mermaid blocks.
- Prefer concrete, interview-useful explanations with likely follow-up questions.
- Prioritize interview questions grounded in user-provided or documented project experience. Do not invent details about the user's work or projects.
- Distinguish verified project facts from general technical knowledge. Include concise explanations, technical detail, trade-offs, production considerations, and practical Java or Spring Boot examples where relevant.
- Prefer updating existing topic files over creating duplicates, and follow the existing directory structure and writing style.
- Preserve existing content unless the requested change requires editing it.

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
