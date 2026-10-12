---
name: cr-brief
description: Write a decision-level PR brief — what was decided, what behaves differently before vs after, the few files that matter, and the real risks — in plain language a non-developer can follow, plus a short technical section for the reviewer. Use when the user says "pr brief", "brief this PR", "summarize the PR", "what's the meat of this PR", or wants a summary before reviewing. Args: optional branch or PR number, optional base branch.
disable-model-invocation: true
---

# CR-Brief — Decision-Level PR Brief

Scope: **PR diff + PR metadata.** This skill summarizes; it does not review. Run `cr-<language>` for findings.

Follow the brief criteria and output template defined in `rules/brief.md`.

## Step 1 — Resolve what to brief

Arguments are optional. Interpret them in this order:
- No argument → the PR for the current branch (`gh pr view`); if there is no PR, the current branch against its default base (`main` or `master`, whichever exists)
- A number → that PR (`gh pr view <number>`)
- A branch name → that branch against the default base
- A second argument → use it as the base branch instead of the default

State what you resolved in one line before continuing: `Briefing PR #123 (feature/export → main)`.

## Step 2 — Get PR context

Run these commands sequentially (no nested substitution):
1. `gh pr view <ref> --json number,title,body,baseRefName,headRefName,url,additions,deletions,changedFiles` — store `baseRefName`. If there is no PR, skip to step 2 with the base you resolved in Step 1.
2. `git log --oneline <baseRefName>...HEAD` — commit messages are a primary source for the decision
3. `git diff --stat <baseRefName>...HEAD` — size and shape of the change
4. `git diff <baseRefName>...HEAD` — the full diff

If the diff exceeds ~2000 changed lines, read the non-test, non-generated files fully and sample the rest. Say so in the brief.

## Step 3 — Run the brief passes

Apply the 5 passes from `rules/brief.md` in order: decision, before/after, files that matter, real risks, open questions. Read surrounding code (not just the diff) wherever you need it to state before-behavior accurately.

## Step 4 — Write the brief

Produce the brief using the output template in `rules/brief.md`. Enforce the two hard rules before showing it:
- **Plain-language summary** contains no file names, function names, or jargon.
- **Total length** is under 250 words excluding the files table.

Show the brief to the user as text.

## Step 5 — Deliver

Detect the environment the same way `rules/autofix.md` does: `CI=true` means CI mode.

### Local mode

After showing the brief, present these options:

> **What would you like to do next?**
> 1. **Save as a file** — write the brief to `cr-brief.md` in the project root
> 2. **Post to the PR** — add the brief as a PR comment with `gh pr comment`
> 3. **Nothing** — the text above is enough

For option 1, write the file and show its path. For option 2, post it and show the comment URL. If there is no PR, offer only options 1 and 3.

### CI mode

Do not prompt. Post the brief as a single PR comment with `gh pr comment <number> --body "<brief>"`. Do not post individual replies and do not add `/fix` instructions — a brief has nothing to fix.

If a previous `## PR Brief` comment from this workflow already exists on the PR, post the new one anyway and begin it with `_Updated brief — supersedes the one above._`

- Run everything autonomously without asking to confirm each step (except the delivery choice in local mode).
- Do not include review findings, severity labels, or suggested code changes. That is the job of the `cr-*` review skills.
