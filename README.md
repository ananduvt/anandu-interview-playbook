# Anandu's Interview Playbook

A living, shareable interview-prep playbook. Notes live as Markdown and are published as a static site
(Material for MkDocs).

## Published site

**Live**: https://ananduvt.github.io/anandu-interview-playbook/

Hosted on GitHub Pages. Auto-rebuilds on merge to `main`.


## Structure
```
.
├── docs/            # site content (Markdown) — published to GitHub Pages
│   ├── index.md     # home page
│   ├── java-core/   # Java fundamentals
│   ├── spring-web/  # Spring Framework
│   ├── ...          # other topic sections
├── mkdocs.yml       # site config (theme, navigation)
├── AGENTS.md        # conventions for content contributors
└── README.md        # this file
```

## Contributing

All content under `docs/` is **public**. Before contributing, read:

1. **[AGENTS.md](AGENTS.md)** — Core rules for editors (file size targets, splitting strategy, no commit without review, safety guidelines)
2. **[CONTRIBUTING.md](CONTRIBUTING.md)** — Content style and patterns (how to write annotations, design patterns, performance sections; section index page templates)

### Quick Guidelines
- **No internal identifiers**: No system/service names, endpoints, ticket IDs, hostnames, cluster names, account/client IDs, or internal URLs
- **Describe generically**: e.g., "an aggregation microservice", "a downstream service"
- **Interview-focused**: Use real-world examples and general technical knowledge
- **Stay within limits**: Aim for 300–600 lines per file; split if it grows beyond 600 lines
- **Max 6 subtopics per section**: If more, create a new parent section
- **🔄 Update related files**: When adding/changing content, ALWAYS update:
  - Section index pages (`docs/*/index.md`)
  - `mkdocs.yml` navigation
  - `docs/index.md` home page
  - Cross-references between topics
- **Review before merge**: Never commit without approval

## Run locally

```sh
python3 -m venv .venv && source .venv/bin/activate   # macOS/Linux
# OR
python3 -m venv .venv && .venv\Scripts\Activate.ps1  # Windows PowerShell

pip install mkdocs-material
mkdocs serve                                           # http://127.0.0.1:8000
```

## Maintaining

Clone → branch → open a PR → review → merge. The site rebuilds automatically on merge to `main`.

See [AGENTS.md](AGENTS.md) for content guidelines.
