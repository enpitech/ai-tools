---
name: perf-audit
description: Run a focused performance audit on the current PR diff or a target directory. Prioritizes high-impact issues — request waterfalls, blocking work, bundle bloat, N+1 queries — over micro-optimizations. Use before shipping any change touching hot paths.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write"
---

# Perf-Audit — Performance Audit (Standalone)

Splits performance findings out of `cr-react` / `cr-node` so they can run independently. Inspired by Vercel's "high-impact-first" prioritization.

## Step 1 — Determine scope

**Diff mode (default in CI):**
1. `gh pr view --json baseRefName` — store base.
2. `git diff <base>...HEAD --name-only` — changed files + direct consumers.

**Full mode (`/enpitech:perf-audit --full`):**
1. Glob all source files; sample top 50 hot-looking files (entry points, route handlers, render-heavy components).

## Step 2 — Detect stack

- React / Next.js → bundle analysis, RSC boundary, Suspense, dynamic imports.
- Node.js → event-loop blocking, connection pooling, N+1, payload size.
- Python → sync/async mixing, ORM N+1, generator vs list comprehension.

Run the relevant sections only; skip the others.

## Step 3 — Audit by priority

Findings are bucketed by **impact tier**. Tier 1 is mandatory to flag; Tier 3 only if the file already passes Tier 1 and 2.

### Tier 1 — Architecture-scale (mandatory)
- **Request waterfalls** — sequential awaits where `Promise.all` would parallelize; client-side fetch chains.
- **Blocking work on the event loop / render thread** — sync `fs`, sync `crypto`, large JSON parses, regex catastrophic backtracking, heavy work in render.
- **N+1 database queries** — loop containing a query without a batch loader or `IN`.
- **Missing pagination / unbounded queries** — `find()` without `limit`.
- **Statically imported heavy libraries** — moment.js, lodash (full), three.js — should be dynamic / tree-shaken.
- **Missing cache headers / CDN bypass** on static assets.
- **Memory leaks** — long-lived listeners not removed, unbounded in-memory caches.

### Tier 2 — Resource hygiene
- **Bundle bloat** — barrel files re-exporting whole modules; named imports from non-tree-shakeable packages.
- **Image policy** — raw `<img>` without `loading="lazy"`, no `width`/`height`, no responsive `srcset`; not using the framework's image component (`next/image`).
- **Font policy** — missing `font-display: swap`; loading entire variable-font axis range when only one is used.
- **Stream backpressure ignored** — `pipe()` without error handling, manual loops that don't await drain.

### Tier 3 — Micro (only if Tier 1+2 clean)
- React: `useMemo` / `useCallback` placement — only flag if data structure is identifiably hot and React Compiler is *not* enabled.
- Map vs object lookup, `Set` vs array `includes` in hot loops.
- String concat in tight loops (Python especially).

## Step 4 — Output findings

```
### [SEVERITY] Tier N — file:line

**Issue:** one-line description
**Why:** impact estimate ("this fires on every keystroke", "this runs on every request")
**Fix:** suggested code change
**Measurement:** how to verify the fix (e.g., "check `next build` output", "run with `--prof`")
```

Write to `perf-findings.md` locally; post as PR comments in CI.

## Step 5 — Autofix

Follow `rules/autofix.md`. Tier 1 fixes first, then Tier 2, then Tier 3.

## Stop conditions

- Never recommend `useMemo` / `useCallback` blanket-application — only when there's evidence of hot path.
- Never recommend micro-optimizations on cold paths.
- Never flag perf issues the framework or compiler handles (React Compiler, Vite tree-shaking, esbuild minification).
- If the recommended fix would require architectural change (e.g., switching ORMs), surface it as a question for `pl-grill`, not a direct fix.
