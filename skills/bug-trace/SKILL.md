---
name: bug-trace
description: Trace a bug from symptom to root cause without writing any fix. Uses repo history, logs, stack traces, and the web (via rs-deep if needed) to find the original trigger. Use when a bug is reported and the cause is non-obvious — runs before pl-grill plans the fix.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write, WebSearch"
---

# Bug-Trace — Root Cause Investigation

Sits between bug report and `pl-grill`:

```
bug-trace  →  pl-grill (plan the fix)  →  implementation  →  cr-*
```

**This skill does not write fixes.** Its only output is a root-cause report.

## Step 1 — Capture the symptom

Ask the user (one prompt, batched):
- What was observed? (error message / behaviour / stack trace).
- What was expected?
- Last known good state? (commit SHA, deploy, date).
- Reproduction steps if available.

If a stack trace is provided, paste it verbatim into your scratch — it is the most valuable input.

## Step 2 — Anchor in the code

If a stack trace exists:
- Resolve every frame to a file and line.
- Read each frame's surrounding context (10 lines above/below).
- Identify the *first* frame that's in this repo (not `node_modules` / `site-packages`).

If no stack trace but an error message:
- `git grep` the error message string; messages are usually unique.
- Find every site that could have produced it.

If neither:
- Identify the feature area from the user's description.
- Map to files via convention or `git log -- <path>`.

## Step 3 — Bisect history

Use `git log -p` and `git blame`, not `git bisect` (too slow without a reproducer):

- `git log --since='<last good>' --oneline -- <suspect files>` — candidate commits.
- `git blame <file>` on the exact lines from Step 2 — when was this line last touched, and by whom and why?
- Read the PR/commit message of each candidate change.

If a real `git bisect` is feasible (the user has a reliable reproducer and the bug bisects cleanly), suggest it explicitly with the commands.

## Step 4 — Cross-reference with the web

If the symptom looks like it could be a library bug:
- Search the relevant library's GitHub Issues for the error message.
- Check the changelog for the installed version vs latest.
- Search Stack Overflow / Reddit for the exact symptom.

If `rs-deep` would help (e.g., subtle library interaction), invoke its passes ad hoc — but keep the budget tight (≤ 10 fetches).

## Step 5 — Form a hypothesis chain

Write the chain as numbered claims, each with evidence:

```
1. The symptom is X. Evidence: stack trace line N, error message "…".
2. The error originates in file:line. Evidence: trace frame, code reading.
3. The bug was introduced in commit ABC. Evidence: `git blame`, PR description.
4. The trigger condition is: <condition>. Evidence: reading the diff of ABC.
5. The root cause is: <one sentence>.
```

Every step must cite. If a step is uncertain, label it `[INFERENCE]` and surface a verification step the user can run.

## Step 6 — Verify the hypothesis

Suggest the *minimum* experiment to confirm:
- A focused test that should fail today and pass after the fix.
- A specific log line to add temporarily and re-run.
- A git revert dry-run (`git revert -n ABC`) to confirm the chain.

Do not perform destructive verification automatically — propose, let the user choose.

## Step 7 — Write `bug-trace.md`

```markdown
# Bug Trace: <one-line symptom>
Generated: <ISO date>

## Symptom
<verbatim from user, plus stack trace>

## Anchor
- First in-repo frame: file:line
- Surrounding context: …

## Hypothesis chain
1. … — evidence: …
2. … — evidence: …
...
N. Root cause: <one sentence>

## Verification suggested
- <experiment>

## Related sources (if any)
- <library issue / SO post / similar bug>

## Open questions for pl-grill
- Q: <e.g., "Fix at the call site, the helper, or change the API contract?">
```

## Step 8 — Hand off

Show the user:
1. The root-cause sentence.
2. The verification step.
3. End with: *"Verify, then run `/enpitech:pl-grill` to plan the fix."*

## Stop conditions

- **No fixes written.** This skill investigates only.
- **No commits, no edits.** All proposed verification is suggested, not executed.
- **No invented commits or PRs** — every cited commit must exist in `git log`.
- **Cite every claim** — root-cause assertions without evidence are not allowed in the chain.
