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

## Local Development

### Setup
```sh
# Clone and activate environment
git clone https://github.com/ananduvt/anandu-interview-playbook.git
cd anandu-interview-playbook

# Create virtual environment
python3 -m venv .venv

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows PowerShell)
.venv\Scripts\Activate.ps1

# Install dependencies
pip install mkdocs-material
```

### Run locally
```sh
mkdocs serve
```

**Access points:**
- **Main site**: http://127.0.0.1:8000
- **Live reload**: Enabled (changes visible immediately on save)
- **Stop server**: `Ctrl+C`

### Navigation (local)
- **Home**: http://127.0.0.1:8000/
- **Java Core**: http://127.0.0.1:8000/java-core/basics/
- **Collections & Concurrency**: http://127.0.0.1:8000/java-collections-concurrency/functional-programming/
- **Streams**: http://127.0.0.1:8000/java-collections-concurrency/streams/
- **Microservices**: http://127.0.0.1:8000/microservices-distributed/principles/
- **System Design**: http://127.0.0.1:8000/system-design/fundamentals/
- **DevOps**: http://127.0.0.1:8000/devops/git/
- **All sections**: Sidebar navigation (left side)

## Remote / Published Site

**Live public site**: https://ananduvt.github.io/anandu-interview-playbook/

### Remote Access
- Hosted on **GitHub Pages**
- Auto-rebuilds on every merge to `main` branch
- Publicly accessible (no authentication required)
- Scrubbed content only (no internal identifiers, client names, endpoints)

### Remote Page URLs
All pages follow the pattern: `https://ananduvt.github.io/anandu-interview-playbook/{section}/{page}/`

| Section | URL |
|---------|-----|
| **Home** | https://ananduvt.github.io/anandu-interview-playbook/ |
| **Java Core - Basics** | https://ananduvt.github.io/anandu-interview-playbook/java-core/basics/ |
| **Java Core - OOP** | https://ananduvt.github.io/anandu-interview-playbook/java-core/oop/ |
| **Java Core - Exceptions** | https://ananduvt.github.io/anandu-interview-playbook/java-core/exceptions/ |
| **Java Core - Versions & Features** | https://ananduvt.github.io/anandu-interview-playbook/java-core/versions-features/ |
| **Collections & Concurrency - Collections** | https://ananduvt.github.io/anandu-interview-playbook/java-collections-concurrency/collections/ |
| **Collections & Concurrency - Functional Programming** | https://ananduvt.github.io/anandu-interview-playbook/java-collections-concurrency/functional-programming/ |
| **Collections & Concurrency - Streams** | https://ananduvt.github.io/anandu-interview-playbook/java-collections-concurrency/streams/ |
| **Collections & Concurrency - Concurrency & Threads** | https://ananduvt.github.io/anandu-interview-playbook/java-collections-concurrency/concurrency/ |
| **Patterns & Advanced - Design Patterns** | https://ananduvt.github.io/anandu-interview-playbook/java-patterns-advanced/design-patterns/ |
| **Patterns & Advanced - Logging & Performance** | https://ananduvt.github.io/anandu-interview-playbook/java-patterns-advanced/logging-performance/ |
| **Patterns & Advanced - Advanced Topics** | https://ananduvt.github.io/anandu-interview-playbook/java-patterns-advanced/advanced/ |
| **Spring & Web - Spring Boot** | https://ananduvt.github.io/anandu-interview-playbook/spring-web/spring-boot/ |
| **Spring & Web - Web Services** | https://ananduvt.github.io/anandu-interview-playbook/spring-web/web-services/ |
| **Spring & Web - API Styles & Protocols** | https://ananduvt.github.io/anandu-interview-playbook/spring-web/api-styles/ |
| **Spring & Web - Security** | https://ananduvt.github.io/anandu-interview-playbook/spring-web/security/ |
| **Spring & Web - JPA & ORM** | https://ananduvt.github.io/anandu-interview-playbook/spring-web/jpa-orm/ |
| **Microservices & Distributed - Principles** | https://ananduvt.github.io/anandu-interview-playbook/microservices-distributed/principles/ |
| **Microservices & Distributed - Spring Cloud** | https://ananduvt.github.io/anandu-interview-playbook/microservices-distributed/spring-cloud/ |
| **Microservices & Distributed - Messaging (Kafka)** | https://ananduvt.github.io/anandu-interview-playbook/microservices-distributed/messaging/ |
| **Databases - SQL & Relational** | https://ananduvt.github.io/anandu-interview-playbook/databases/sql/ |
| **Databases - Scaling & NoSQL** | https://ananduvt.github.io/anandu-interview-playbook/databases/scaling/ |
| **System Design - Fundamentals** | https://ananduvt.github.io/anandu-interview-playbook/system-design/fundamentals/ |
| **System Design - Building Blocks** | https://ananduvt.github.io/anandu-interview-playbook/system-design/building-blocks/ |
| **DevOps - Git** | https://ananduvt.github.io/anandu-interview-playbook/devops/git/ |
| **DevOps - Docker & Containers** | https://ananduvt.github.io/anandu-interview-playbook/devops/docker/ |
| **DevOps - Kubernetes** | https://ananduvt.github.io/anandu-interview-playbook/devops/kubernetes/ |
| **DevOps - CI/CD** | https://ananduvt.github.io/anandu-interview-playbook/devops/ci-cd/ |
| **Data Structures & Algorithms** | https://ananduvt.github.io/anandu-interview-playbook/dsa/data-structures/ |
| **Interview Prep - Question Bank** | https://ananduvt.github.io/anandu-interview-playbook/interview-prep/question-bank/ |
| **Resources** | https://ananduvt.github.io/anandu-interview-playbook/resources/ |
| **Backlog** | https://ananduvt.github.io/anandu-interview-playbook/backlog/ |

## Workflow

### Development
1. **Clone** the repository
2. **Create a branch** for your changes
3. **Run locally** (`mkdocs serve`) to preview changes
4. **Commit** changes with descriptive messages
5. **Push** to remote and **open a PR**
6. **Review** → merge to `main`

### Publishing
- Merge to `main` automatically triggers a GitHub Actions workflow
- Site rebuilds and deploys to GitHub Pages in ~1-2 minutes
- Changes visible at https://ananduvt.github.io/anandu-interview-playbook/ immediately

## Maintaining

**Guidelines**: See [AGENTS.md](AGENTS.md) for content patterns and best practices.

**Branch strategy**:
- `main` → production (public site)
- Feature branches → PR for review
- No direct commits to `main`

**Before committing**:
- Validate Markdown syntax
- Check internal links are correct (relative paths)
- Ensure no internal/private identifiers in `docs/`
- Verify navigation in `mkdocs.yml` includes new pages
