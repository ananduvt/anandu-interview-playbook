# AGENTS.md — Conventions for editors (humans + AI)

Read this before adding or changing content.

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

## Structure
- Add new pages under `docs/` and register them in `mkdocs.yml` `nav`.
- Keep `index.md` as the landing/welcome page.

## Workflow
- Work on a branch, open a PR, get a review, merge to `main`. The public site rebuilds on merge.
