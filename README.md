# Venture Forge Toolbox

> **15 repo-agnostic DevOps tools that pull all config from `.env`.**
> You never edit the scripts. You edit `.env` once.

## Quick Start

### New project (GitHub Template)
1. Click **"Use this template"** on GitHub
2. `cp .env.template .env.local` → fill in your values
3. Done. All tools work.

### Existing project (Bootstrap)
```bash
git clone https://github.com/YOU/venture-forge-toolbox.git /tmp/toolbox
/tmp/toolbox/bootstrap.sh /path/to/your-repo
```

Or one-liner:
```bash
curl -fsSL https://raw.githubusercontent.com/YOU/venture-forge-toolbox/main/bootstrap.sh | bash -s /path/to/your-repo
```

## What's Included

### Shell Tools (`tools/`)

| Tool | What it does | Key .env vars |
|------|-------------|---------------|
| `infra/check_env.sh` | Validates .env (missing vars, localhost-in-Docker, placeholders) | reads `.env.required` |
| `infra/docker_validate.sh` | Validates Dockerfiles + compose syntax | auto-discovers |
| `infra/test_everything.sh` | 11-section test suite (Docker, API, auth, DB, WebSocket) | `APP_API_KEY`, `APP_PORT` |
| `infra/deep_mri.sh` | 10-section production VPS diagnostics | `POSTGRES_*`, all ports |
| `infra/precommit_smoke.sh` | Builds stack in isolation, smoke tests, tears down | `COMPOSE_*`, `API_CONTAINER` |
| `dev/dev_up.sh` | Local dev server launcher (venv + .env + uvicorn) | `APP_MODULE` |
| `hooks/install_hooks.sh` | Auto-discovers and installs git hooks | none |
| `deploy/deploy.sh` | Full VPS deploy (git SSOT + env sync + rebuild + health) | `VPS_HOST`, `VPS_REPO` |

### CI/CD (`.github/workflows/`)

| File | What it does |
|------|-------------|
| `ci-quality.yml` | Ruff, MyPy, SonarCloud, GitGuardian, Codecov, Semgrep, YAML lint |

### Lint Rules (`.semgrep/`)

| File | What it does |
|------|-------------|
| `semgrep-rules.yaml` | 8 Python policy rules (no eval, no pickle, parameterized SQL, etc.) |

### Templates (`templates/`)

| File | Purpose |
|------|---------|
| `.env.required.template` | Required env var names (one per line) |
| `.env.recommended.template` | Recommended env var names |
| `.env.vps.template` | All VPS deployment variables |
| `dev.conf.template` | Local dev server config |
| `styleguide.template.md` | Fill-in-the-blanks code style guide |

## Architecture

```
Your Repo
├── .env.local              ← YOU edit this (gitignored)
├── .env.template           ← checked in, shows what vars exist
├── .github/workflows/
│   └── ci-quality.yml      ← from toolbox (never edit)
├── .semgrep/
│   └── semgrep-rules.yaml  ← from toolbox (never edit)
├── tools/
│   ├── ci/
│   ├── deploy/
│   ├── dev/
│   ├── hooks/
│   ├── infra/
│   └── lint/
├── templates/
│   └── *.template
└── your actual code...
```

## Updating Tools

### If you used the template:
```bash
# Pull latest toolbox into a temp dir, re-run bootstrap with -f (force)
git clone https://github.com/YOU/venture-forge-toolbox.git /tmp/toolbox
/tmp/toolbox/bootstrap.sh . -f
```

### Future: Copier support
For automatic upstream updates across all repos, we plan to add Copier support.
Run `copier update` and all tools get the latest version while preserving your .env.

## Design Principles

1. **Config in `.env`, logic in `tools/`** — You never edit scripts
2. **Fail fast on missing config** — Scripts use `${VAR:?message}` for required vars
3. **Skip gracefully on missing optional** — Set a port to `0` and that section is skipped
4. **Auto-discover when possible** — Docker services, git hooks, Dockerfiles
5. **Safe to re-run** — Bootstrap never overwrites existing files (unless `-f`)
