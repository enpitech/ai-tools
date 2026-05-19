---
name: rs-migrate
description: Research-only mode for version migrations (React 18→19, Node 20→22, Python 3.11→3.12, framework majors). Surfaces breaking changes, codemods, real-world migration write-ups, and gotchas before you plan the upgrade. Use when planning any framework or runtime version bump.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write, WebSearch, WebFetch"
---

# RS-Migrate — Migration Research

Specialized variant of `rs-deep` focused on **version migrations**.

Follow the research criteria from `rules/research.md`, with the migration-specific overlays below.

## Step 1 — Confirm migration scope

Ask the user (one prompt only):
- From version: ?
- To version: ?
- Constrained by: (any pinned peer deps, CI, infra)

## Step 2 — Detect current versions

Read manifests and lockfiles. Record:
- The actual installed version (from the lockfile, not the range in `package.json`).
- All peer dependencies that constrain or are constrained by the migrating library.
- Direct dependents in the workspace (other packages that consume this).

## Step 3 — Migration-overlay passes

Run the 8 passes from `rules/research.md` with these focus shifts:

| Pass | Migration overlay |
|---|---|
| 2. OFFICIAL DOCS | Read the upgrade/migration guide for the target version; capture every "breaking change" entry verbatim |
| 3. ISSUE TRACKER | Filter issues labelled `breaking-change`, `migration`, `regression`, or referencing the target version |
| 4. COMMUNITY SIGNAL | Search for "upgrade to <X>", "migrating from <prev> to <X>", "<X> broke our app" |
| 5. PRIOR ART | Find real-world migration write-ups from large OSS projects; capture their PR diffs |
| 6. SECURITY | Specifically check whether the migration is forced by a CVE in the current version |
| 7. CREATIVE PASS | "Should we skip this version?" "What broke for people who already upgraded?" "Are there forks or LTS branches we should stay on?" |

## Step 4 — Codemod & tooling discovery

In addition to the standard passes, search for:
- Official codemods (e.g. `npx @next/codemod`, `2to3`, `pyupgrade`, `eslint-plugin-react/migrating`).
- Third-party migration tools.
- ESLint/Pyright/Mypy plugins that flag deprecated APIs.

List each tool with: install command, what it covers, what it does *not* cover.

## Step 5 — Write `migration-brief.md`

Use the template from `rules/research.md` with these added sections:

```markdown
## Breaking changes (verbatim, with citation)
| # | Change | Affects this repo? | Source |
|---|--------|--------------------|--------|

## Codemods & tooling
| Tool | Covers | Doesn't cover | Source |
|---|---|---|---|

## Real-world migration reports
| Project | From → To | Time taken | Pain points | Source |
|---|---|---|---|---|

## Recommended upgrade order
1. ...
```

## Step 6 — Surface the brief

Show the user:
1. Count of breaking changes that *actually affect this repo*.
2. Available codemods.
3. The recommended upgrade order.
4. Open questions for `pl-grill`.

Stop conditions: same as `rs-deep` (no invention, cite or strike, budget cap, no planning).
