---
name: commit-write
description: Read staged changes and write a Conventional Commits message. Detects type (feat/fix/refactor/docs/chore/test/perf), infers scope from changed paths, flags breaking changes, and suggests splitting when staged changes mix concerns. Use whenever the user is about to commit.
disable-model-invocation: true
allowed-tools: "Bash, Read, Grep"
---

# Commit-Write — Conventional Commit Messages

## Step 1 — Read staged context

Run in parallel (independent):
1. `git status --short` — see what's staged.
2. `git diff --cached` — full staged diff.
3. `git diff --cached --name-only` — file list.
4. `git log -n 5 --pretty=format:'%s'` — recent commit style in this repo.

If nothing is staged, say so and stop.

## Step 2 — Detect type and scope

**Type** (pick exactly one, in this priority):
- `fix` — repo had a bug, this corrects it (look for: error handling added, null check, off-by-one corrected, test that was failing now passes).
- `feat` — new user-visible capability.
- `perf` — measurable performance change with no behaviour change.
- `refactor` — internal restructuring, no behaviour change.
- `test` — only test files changed.
- `docs` — only docs (`.md`, `docs/`) changed.
- `build` / `ci` — build config, CI workflow.
- `chore` — version bumps, lockfile-only changes, formatting.

**Scope** — infer from the most-changed top-level directory or package:
- `src/components/foo/*` → `foo`
- `packages/api/*` → `api`
- Multiple unrelated scopes → consider splitting (Step 4).

## Step 3 — Detect breaking changes

Flag as breaking (`feat!:` or `BREAKING CHANGE:` footer) if any of:
- A public function signature removed or arg removed/reordered.
- An exported type's required field removed or made stricter.
- A CLI flag removed/renamed.
- An HTTP route removed or response shape changed.
- A migration that is not backward compatible.

When you find one, name it explicitly in the body.

## Step 4 — Detect mixed-concern stages

If the staged diff covers 2+ distinct concerns (e.g., a feature AND an unrelated refactor), do **not** write one combined message. Instead:

1. Tell the user the staged changes look like N separate commits.
2. List the suggested split with `git reset HEAD <file>` commands.
3. Stop and let the user re-stage.

## Step 5 — Compose the message

Format:

```
<type>(<scope>): <subject in imperative, ≤ 72 chars>

<body — what and why, not how. Wrap at 80. Optional.>

<footer — BREAKING CHANGE: …, Refs: #123. Optional.>
```

Rules:
- Subject in imperative mood ("add", not "added").
- No period at end of subject.
- Body explains *why* the change exists, not the line-by-line diff.
- If the repo's recent commits use a different style (from Step 1 log), **match the existing style** instead of forcing Conventional Commits — note this in your output.

## Step 6 — Surface and commit

Show the user the proposed message in a code block. Then ask: *"Commit with this message? (y / edit / split)"*.

- `y` → run `git commit -m "<msg>"` using a HEREDOC.
- `edit` → let the user revise.
- `split` → re-run Step 4 logic.

## Stop conditions

- Never commit without explicit user confirmation.
- Never include `Co-Authored-By` lines unless the user asks for them.
- Never use `--no-verify` unless the user asks.
- Never amend a previous commit unless the user explicitly asks.
