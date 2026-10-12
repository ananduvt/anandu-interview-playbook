# Anandu's Interview Playbook

A living, shareable interview-prep playbook. Notes live as Markdown and are published as a static site
(Material for MkDocs). The **repo is private**; the **published site is public** (scrubbed content only).

## Structure
```
.
├── docs/            # published to the site (Markdown)
│   └── index.md     # welcome page
├── internal/        # private notes kept in the repo, NOT published
├── mkdocs.yml       # site config (theme, nav)
├── AGENTS.md        # conventions for humans + AI editors
└── README.md
```

## Golden rule
Anything under `docs/` becomes **public**. Keep it free of employer/client identifiers
(internal system names, endpoints, ticket IDs, hostnames, account/client IDs). Private context goes in
`internal/`, which is never built into the site.

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
