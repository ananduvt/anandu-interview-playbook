# Anandu's Interview Playbook

A living, shareable interview-prep playbook. Notes live as Markdown and are published as a static site
(Material for MkDocs).

## Published site

**Live**: https://ananduvt.github.io/anandu-interview-playbook/

Hosted on GitHub Pages. Auto-rebuilds on merge to `main`.


## Structure
```
.
├── docs/              # site content (Markdown), published to GitHub Pages
│   ├── index.md       # home page
│   ├── java-core/     # Java fundamentals
│   ├── spring-web/    # Spring Framework
│   └── ...            # other topic sections
├── todo/              # content intake pipeline and internal task list (not published)
├── mkdocs.yml         # site config (theme, navigation)
├── CONTRIBUTING.md    # contributor guide
└── README.md          # this file
```

## Contributing

All content under `docs/` is **public**. Before contributing, read **[CONTRIBUTING.md](CONTRIBUTING.md)**. It covers the git workflow, where new content goes, size limits, formatting, question templates, and the checklist that keeps the site working.

### Quick Guidelines
- **No client or internal identifiers**: no company, system, or service names, endpoints, ticket IDs, hostnames, or internal URLs
- **Describe generically**: e.g., "an aggregation microservice", "a downstream service"
- **Interview-focused**: use real-world examples and general technical knowledge
- **Keep the site working**: every change also updates `mkdocs.yml` nav, section index pages, and inbound links
- **Review before merge**: the owner commits to `main`; other contributors open a PR for approval

## Run locally

```sh
python3 -m venv .venv && source .venv/bin/activate   # macOS/Linux
# OR
python3 -m venv .venv && .venv\Scripts\Activate.ps1  # Windows PowerShell

pip install mkdocs-material
mkdocs serve                                           # http://127.0.0.1:8000
```

## Maintaining

Contributors: clone → branch → open a PR → review → merge. The site rebuilds automatically on merge to `main`.

See [CONTRIBUTING.md](CONTRIBUTING.md) for content guidelines.
