# Rails 8 Devcontainer

A zero-setup, AI-powered development environment for Rails 8 — runs identically on GitHub Codespaces and any machine with Docker or Podman.

Open it, run one command, and start shipping.

---

## What's inside

| Tool | Version | Purpose |
| --- | --- | --- |
| Ruby | 3.3 | Runtime — zero compile, transplanted from official image |
| Node | 22 | Asset pipeline and JS toolchain |
| PostgreSQL | 16 | Primary database with persistent volume |
| Redis | 7 | Cache and background jobs (Sidekiq) |
| Mailcatcher | latest | Catches outgoing emails in development |
| Adminer | latest | Lightweight web UI for the database |
| Claude Code | latest | AI pair programmer in your terminal |
| OpenCode | latest | Agentic software development (sst/opencode) |
| Bitwarden CLI | latest | Secrets management — stays in your shell, never in containers |
| Mise | latest | Per-project runtime version management |

---

## Quickstart

### 1. Open in Codespace

Click **Code → Codespaces → Create codespace on main** on GitHub.
The devcontainer builds automatically. All tools are ready when the terminal opens.

### 2. Start infrastructure

```bash
bin/setup_dev
```

Checks prerequisites, starts PostgreSQL and Redis via Docker Compose, and waits until both are healthy.

### 3. Set up the database

```bash
rails db:create db:migrate
```

### 4. Start the server

```bash
rails server
```

Port 3000 is forwarded automatically by Codespaces. Open it from the **Ports** tab in VS Code or JetBrains Gateway.

---

## Services and ports

| Service | Address |
| --- | --- |
| Rails app | http://localhost:3000 |
| PostgreSQL | localhost:5432 |
| Redis | localhost:6379 |
| Mailcatcher UI | http://localhost:1080 |
| Adminer (DB client) | http://localhost:8080 |

> Inside Adminer, connect to PostgreSQL using **Server: `db`**, not `localhost`. Containers reach each other by service name.

---

## AI tools

### Claude Code

```bash
export ANTHROPIC_API_KEY=$(bw get password "anthropic-api-key")
claude
```

Full AI pair programming in the terminal. Reads your codebase, writes and edits files, runs tests, and explains anything on demand.

### OpenCode

```bash
opencode
```

Agentic coding assistant. Give it a goal and let it plan and execute multi-step development tasks.

---

## Secrets with Bitwarden

The Bitwarden CLI runs **isolated in your shell session**. No key ever touches a container, a file, or an environment variable that outlives your terminal.

**First login**

```bash
bw login
```

Interactive flow — email, password, and 2FA. Happens once.

**Unlock on subsequent sessions**

```bash
export BW_SESSION=$(bw unlock --raw)
```

**Inject a secret into your shell**

```bash
# Fetch by item name
export STRIPE_SECRET_KEY=$(bw get password "stripe-prod")
export ANTHROPIC_API_KEY=$(bw get password "anthropic-api-key")
```

`bin/setup_dev` checks whether you have an active session and tells you what to do if not — but it never blocks the container startup.

---

## Environment variables

Set automatically by `devcontainer.json`:

| Variable | Default |
| --- | --- |
| `DATABASE_URL` | `postgres://postgres:password@localhost:5432/postgres` |
| `REDIS_URL` | `redis://localhost:6379/1` |

Override locally with a `.env` file at the project root (already in `.gitignore` by default in Rails apps).

---

## Useful commands

```bash
# Follow logs for a service
docker compose logs -f db
docker compose logs -f redis

# Stop all containers
docker compose down

# Full reset — destroys database volumes
docker compose down -v

# Open a psql session
docker compose exec db psql -U postgres

# Open a Redis session
docker compose exec redis redis-cli
```

---

## CI — Container image

The workflow at [`.github/workflows/devcontainer.yml`](.github/workflows/devcontainer.yml) triggers automatically when the `Dockerfile`, `devcontainer.json`, or `docker-compose.yml` change.

| Event | Behavior |
| --- | --- |
| Pull request | Builds the image — validates the Dockerfile without publishing |
| Push to `main` | Builds and pushes to `ghcr.io` tagged `latest` and `sha-<commit>` |

Authentication uses the auto-generated `GITHUB_TOKEN` — no extra secrets to configure.

### Use the pre-built image

After the first push to `main`, swap the build step in `devcontainer.json` for the published image. Codespaces will open in seconds instead of minutes:

```json
"image": "ghcr.io/carloskvasir/simple_devcontainer/devcontainer:latest"
```

### Delete the image

Go to **github.com → your profile → Packages → devcontainer → Delete this package**.
The image is gone completely — no public trace remains.

---

## Local usage (outside Codespaces)

Requires Docker Engine or Podman. The setup script detects either automatically.

```bash
git clone https://github.com/carloskvasir/simple_devcontainer
cd simple_devcontainer
bin/setup_dev
```

To open the full devcontainer locally, install the [VS Code Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) and run **Dev Containers: Reopen in Container**.

---

## Repository layout

```
.
├── .devcontainer/
│   ├── Dockerfile          # Multi-stage build — Ruby and Node transplanted, zero compile
│   └── devcontainer.json   # Codespace config — ports, extensions, VS Code and RubyMine
├── .github/
│   └── workflows/
│       └── devcontainer.yml  # CI: validate on PR, publish image on push to main
├── bin/
│   └── setup_dev           # Bootstrap script — starts infra, checks Bitwarden session
└── docker-compose.yml      # PostgreSQL, Redis, Mailcatcher, Adminer
```
