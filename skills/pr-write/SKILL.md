---
name: pr-write
description: Generate a pull request description from the branch diff. Pulls context from plan.md and any cr-*-findings.md if present. Covers what changed, why, how it was implemented, test plan, and reviewer notes. Use before opening or updating a PR.
disable-model-invocation: true
allowed-tools: "Bash, Read, Grep, Glob, Write"
---

# PR-Write — Pull Request Description Generator

## Step 1 — Detect inputs in parallel

Run independently:
1. `git rev-parse --abbrev-ref HEAD` — current branch.
2. `git symbolic-ref refs/remotes/origin/HEAD` (fallback: `main`) — base branch.
3. `git log <base>..HEAD --pretty=format:'%h %s'` — commits on the branch.
4. `git diff <base>...HEAD --stat` — high-level diff.
5. `git diff <base>...HEAD --name-only` — changed files.
6. Read `plan.md` if present — supplies Intent, Acceptance Criteria, Decisions.
7. Read `cr-*-findings.md` (any matching) if present — supplies remaining/known limitations.
8. `gh pr view --json title,body,url` if a PR already exists — supplies an existing description to update rather than overwrite.

## Step 2 — Classify the change

Determine the dominant change type by looking at the commit subjects and file paths:
- New feature (mostly `feat:` commits, new files in `src/`).
- Bug fix (`fix:` commits, small diff, often tests added).
- Refactor (`refactor:` commits, no behaviour change in tests).
- Migration / dep bump (lockfile / config / version changes).
- Mixed → call this out at the top of the PR description.

## Step 3 — Compose the description

Use this template (adapt sections to remote — GitHub vs GitLab vs Bitbucket; default GitHub):

```markdown
## Summary
<2–4 bullets, user-facing first if it's a feature; root-cause-first if it's a fix>

## Why
<the motivation — pull from plan.md Intent if available, otherwise infer from commits>

## What changed
- <bullet per significant change, grouped by file or area>

## How
<implementation notes that aren't obvious from the diff — non-trivial choices, trade-offs accepted>

## Test plan
- [ ] <how the reviewer should verify>
- [ ] <edge case>
- [ ] <existing tests still pass>

## Risks / known limitations
<from cr-*-findings.md or plan.md Risks; "none" if genuinely none>

## Screenshots
<placeholder if frontend change, else omit>

## Linked issues
<closes #N if discoverable from branch name or commit footers, else omit>
```

## Step 4 — Quality checks before showing

- **Length**: under ~400 words. If longer, the section is probably padded — tighten.
- **No re-stating the diff**: the reviewer can read the diff; the PR explains *why*, not *what line did what*.
- **No invented context**: every fact must come from commits, plan.md, findings, or the diff itself.
- **First-person plural** ("we") or imperative — match the repo's existing PRs (sample from `gh pr list --state merged --limit 5`).

## Step 5 — Surface and apply

Show the user the proposed title and body. Then ask: *"Update PR? (push / edit / skip)"*.

- `push` → `gh pr edit --body "$(cat <<'EOF' … EOF)"`, or `gh pr create` if no PR exists yet.
- `edit` → write to `pr-description.md` for manual editing.
- `skip` → output to stdout only.

## Stop conditions

- Never push to remote without explicit confirmation.
- Never invent linked issues — only include if discoverable.
- If `plan.md` and the diff materially disagree, flag the divergence in the description rather than papering over it.
