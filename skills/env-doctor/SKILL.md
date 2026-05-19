---
name: env-doctor
description: Diagnose why a project won't start, build, or test locally. Systematically checks runtime versions, dependency install state, environment variables, ports, and external service reachability. Use when a fresh clone, post-pull setup, or "works on my machine" issue blocks progress.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write"
---

# Env-Doctor — Local Environment Diagnosis

Pure read-only diagnosis. Writes nothing to the repo — outputs a report and remediation steps.

## Step 1 — Capture the failure

Ask the user (one prompt):
- What command failed? (`npm run dev`, `pytest`, `make`, etc.)
- What was the error? (paste verbatim, or path to a log).
- Fresh clone, post-pull, or post-merge?

## Step 2 — Detect project requirements

Read in parallel:
- `package.json` → `engines.node`, `engines.npm`, `packageManager`.
- `.nvmrc`, `.node-version` — required Node version.
- `pyproject.toml` / `setup.py` — `python_requires`.
- `.python-version`, `runtime.txt`.
- `go.mod` — Go version.
- `Cargo.toml` — `rust-version`.
- `.tool-versions` (asdf / mise).
- `Dockerfile` — base image version.
- `.env.example` — required env vars.
- `docker-compose.yml` — required services & ports.

Build an expected-state table.

## Step 3 — Probe actual state (read-only commands only)

Run each independently:
- `node --version`, `npm --version`, `pnpm --version`, `yarn --version`, `bun --version`.
- `python --version`, `python3 --version`, `pip --version`, `uv --version`.
- `go version`, `cargo --version`, `rustc --version`.
- `git --version`, `gh --version`.
- `docker --version`, `docker compose version`.
- `which <tool>` for anything in the project's package scripts.

For each, compare to the expected version. Mismatch → flag.

## Step 4 — Probe dependency install state

- **Node**: does `node_modules` exist? Is the lockfile in sync? (`npm ci --dry-run`, `pnpm install --frozen-lockfile --dry-run`, `yarn install --check-files --immutable`).
- **Python**: is the venv active? (`echo $VIRTUAL_ENV`). Are installed packages in sync with the lockfile? (`pip check`, `uv pip check`, `poetry check --lock`).
- **Go**: `go mod verify`.
- **Rust**: `cargo check --offline` (only if a target dir exists).

Do not run `install` — that's mutating. Only check.

## Step 5 — Probe environment variables

Compare `.env.example` keys against `.env` keys:
- Missing keys → flag.
- Keys with placeholder values (`changeme`, `xxx`, empty) → flag.

Do not print values — only key names and presence.

## Step 6 — Probe ports & services

For each port mentioned in `docker-compose.yml`, `package.json` scripts, or `.env`:
- `lsof -i :<port>` (macOS/Linux) — is something already listening?
- For services declared in compose: are they `Up`? (`docker compose ps`).

## Step 7 — Cross-check the original error

Re-read the user's failure message. Match it against the most common known patterns:

| Error contains | Likely cause | Quick check |
|---|---|---|
| `ENOENT` for a binary | tool not installed or PATH issue | Step 3 |
| `Unsupported engine` | Node version mismatch | Step 3 |
| `Cannot find module` | install state stale or lockfile out of sync | Step 4 |
| `EADDRINUSE` | port already taken | Step 6 |
| `EACCES` | permissions; or trying to install globally | Re-check command |
| `Could not connect to <host>` | service down, wrong env var, network | Steps 5+6 |
| `ImportError` / `ModuleNotFoundError` | venv inactive or dep missing | Step 4 |
| `migration not applied` | DB not up or migrations not run | Step 6 |
| `SSL certificate` | proxy / corp network / wrong CA | Step 3 |

## Step 8 — Surface remediation

Show the user a prioritized list:

```
## Likely cause (ranked)
1. <hypothesis> — evidence: <step output>
   Fix: <single concrete command, never destructive>

2. <hypothesis> — evidence: <step output>
   Fix: <command>

## Diagnostic summary
| Component | Expected | Actual | Status |
|---|---|---|---|
| Node | >= 20.0.0 | 18.17.0 | ❌ mismatch |
| node_modules | in sync | stale (lockfile newer) | ❌ |
| DB port 5432 | listening | not listening | ❌ |
| API_KEY env var | set | missing | ❌ |
```

End with: *"Apply the top fix and re-run the failing command. If still broken, paste the new error and re-run env-doctor."*

## Stop conditions

- **No mutations.** Never run `install`, `rm`, `kill`, `migrate`, or anything that changes local state. Suggest only.
- **No secrets printed.** Only report key names and presence/absence, never values.
- **No remote network calls** beyond what tool-version checks naturally do.
- **No assumptions about the user's shell** — `which` and `command -v` both work.
