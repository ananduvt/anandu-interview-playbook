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

## Run the site locally
```sh
python3 -m venv .venv && source .venv/bin/activate
pip install mkdocs-material
mkdocs serve            # preview at http://127.0.0.1:8000
```

## Publish (to be set up)
Site hosting (e.g. Cloudflare Pages / Netlify) will build from this private repo and serve a public site,
auto-rebuilding on every push to `main`. Configuration will be added later.

## Maintaining
Clone → branch → open a PR → review → merge. The site rebuilds automatically on merge to `main`.
