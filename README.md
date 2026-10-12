# Anandu's Interview Playbook

A living, shareable interview-prep playbook. Notes live as Markdown and are published as a static site
(Material for MkDocs). The **repo is private**; the **published site is public** (scrubbed content only).

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

## Guidelines for Contributors
All content under `docs/` is **public**. Keep it free of employer/client identifiers:
- No internal system/service names, endpoints, ticket IDs, hostnames
- No cluster names, account/client IDs, or internal URLs
- Describe things generically (e.g., "an aggregation microservice", "a downstream service")
- Use real-world examples and general technical knowledge

## Run locally

```sh
python3 -m venv .venv && source .venv/bin/activate   # macOS/Linux
# OR
python3 -m venv .venv && .venv\Scripts\Activate.ps1  # Windows PowerShell

pip install mkdocs-material
mkdocs serve                                           # http://127.0.0.1:8000
```

## Published site

**Live**: https://ananduvt.github.io/anandu-interview-playbook/

Hosted on GitHub Pages. Auto-rebuilds on merge to `main`.

## Maintaining

Clone → branch → open a PR → review → merge. The site rebuilds automatically on merge to `main`.

See [AGENTS.md](AGENTS.md) for content guidelines.
