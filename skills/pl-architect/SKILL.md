---
name: pl-architect
description: Produce a multi-file architectural plan for large features that span more than one layer, package, or subsystem. Use after pl-grill for features too big to fit in a single plan.md — splits the work into a sequence of reviewable PRs, each leaving the repo in a shippable state.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write"
---

# PL-Architect — Multi-PR Architecture Plan

Use **after** `pl-grill` for features that are too large to fit in a single `plan.md`.

Follow the planning criteria from `rules/plan.md` for the per-PR plans. This skill orchestrates a *sequence* of plans.

## When to use

- Feature touches 3+ packages or layers (frontend + API + db + worker).
- File map from `pl-grill` exceeds ~20 files.
- Migration that cannot land in one PR without breaking production.
- Refactor that requires phased rollout (introduce new path → migrate callers → remove old).

If the feature is smaller than this, **use `pl-grill` directly**.

## Step 1 — Read inputs

- `plan.md` from `pl-grill` (required).
- `research-brief.md` from `rs-deep` (optional but preferred).
- `migration-brief.md` from `rs-migrate` (if a migration).

If `plan.md` is missing, suggest running `/enpitech:pl-grill` first.

## Step 2 — Detect project context

Identify packages/workspaces, deploy units, and ownership boundaries. Read:
- Monorepo config (`workspaces`, `lerna.json`, `nx.json`, `turbo.json`, `pnpm-workspace.yaml`).
- Service boundaries (separate `Dockerfile`s, separate deploy configs).
- Test boundaries (which tests live where).

## Step 3 — Slice into PRs

Apply these slicing rules in order:

1. **Each PR must leave the repo shippable** — compiling, tests passing, no broken contracts.
2. **Bottom-up where possible** — schemas before APIs, APIs before UI consumers.
3. **Introduce-then-migrate** — new code path lands first behind a flag or feature toggle, callers migrate next, old path is removed last.
4. **Maximum ~400 LOC per PR** (excluding generated code, lockfiles). Larger PRs only with explicit justification.
5. **No PR depends on an un-merged sibling** unless that dependency is stated explicitly in the plan.

## Step 4 — Cross-PR concerns

For each of these, decide once and document in `architecture-plan.md`:

- **Feature flag strategy** — single flag for the whole feature, or per-PR flags?
- **Data migration** — single migration, or sequence? Online or offline?
- **Telemetry rollout** — when do metrics/logs get added so we can detect regressions during the rollout?
- **Rollback story** — what is the rollback for each PR? Some PRs are forward-only (DB migrations); flag those.
- **Owners and reviewers** — per PR, suggest the right reviewer based on touched code.

## Step 5 — Write `architecture-plan.md`

```markdown
# Architecture Plan: <feature>
Generated: <ISO date>
Status: DRAFT / APPROVED

## TL;DR
<3 bullets>

## Cross-PR decisions
| Decision | Choice | Why |
|---|---|---|

## PR sequence
### PR 1 — <title>
- Scope: ...
- Files: ~N
- Leaves repo: compiling ✓ tests ✓ feature on/off via flag ✓
- Depends on: —
- Rollback: ...
- Reviewer hint: ...

### PR 2 — <title>
- Depends on: PR 1 merged
...

## Risk timeline
| Phase | Risk window | Mitigation |
|---|---|---|

## Decommission plan (if applicable)
- Old path stays until: <condition>
- Removed in: PR N
```

For each PR in the sequence, also write a stub `plans/pr-N-<slug>.md` using the `rules/plan.md` template — `pl-grill` can be re-invoked per PR to fill them in.

## Step 6 — Surface the plan

Show the user:
1. The PR sequence (titles only).
2. The cross-PR decisions table.
3. The risk timeline.
4. End with: *"Approve the sequence, then run `/enpitech:pl-grill` per PR to detail each one."*

## Stop conditions

- Every PR has a rollback story (or is explicitly flagged forward-only).
- No PR exceeds ~400 LOC without justification.
- Feature-flag strategy is decided and documented.
- The user has approved the sequence before implementation begins.
