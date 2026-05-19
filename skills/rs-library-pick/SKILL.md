---
name: rs-library-pick
description: Research-only mode for choosing between libraries ("should we use X or Y?"). Builds an evidence-backed comparison matrix from real-world reports — not vendor marketing. Use before adding a new dependency or replacing an existing one.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write, WebSearch, WebFetch"
---

# RS-Library-Pick — Library Comparison Research

Specialized variant of `rs-deep` focused on **choosing between libraries**.

Follow the research criteria from `rules/research.md`, with the comparison-specific overlays below.

## Step 1 — Confirm candidates

Ask the user (one prompt only):
- Capability needed: ?
- Candidates: ? (minimum 2, maximum 5)
- Constraints: (license, framework, runtime, bundle-size budget, etc.)

If the user gives only the capability, *you* propose 3–5 candidates from current OSS landscape (Pass 5) and confirm before continuing.

## Step 2 — Detect project context

Read manifests. Capture: current stack, existing similar libraries already used (avoid double-adding), TypeScript / Python typing strictness, runtime targets (node version, browser support).

## Step 3 — Comparison-overlay passes

Run the 8 passes from `rules/research.md`, but produce findings **per candidate** in a single shared matrix.

| Pass | Comparison overlay |
|---|---|
| 2. OFFICIAL DOCS | Pull capabilities, install size, peer deps, TypeScript story, runtime support |
| 3. ISSUE TRACKER | Open issue count, % closed in last 90 days, time-to-first-response, maintainer activity |
| 4. COMMUNITY SIGNAL | Stack Overflow tag activity (last year), Reddit mentions, "X vs Y" blog posts |
| 5. PRIOR ART | Which large OSS projects use it? Find one famous user per candidate |
| 6. SECURITY | CVE history, frequency, time-to-patch |
| 7. CREATIVE PASS | "Why people left X" / "X considered harmful" / archived alternatives that suggest a category-wide problem |

## Step 4 — Bundle / install impact

For each candidate:
- npm: query bundlephobia for min+gzip and tree-shakeability.
- PyPI: install size + transitive deps count.
- Other: package registry size.

## Step 5 — Build the decision matrix

Score each candidate on these axes (1–5, with one-line justification + source per cell):

| Axis | Why it matters |
|---|---|
| Capability fit | Does it actually solve the problem |
| Maintenance health | Commits last 90 days, issue close rate |
| Community size | StackOverflow tag count, GitHub stars + trend |
| Bundle / install cost | min+gzip or install size |
| Type safety | TS types quality / Python type stubs |
| Security history | CVE count, time-to-patch |
| Lock-in risk | Easy to replace? Standard interface? |
| Documentation | Completeness, examples, recipes |

## Step 6 — Write `library-pick-brief.md`

Use the template from `rules/research.md` with these added sections:

```markdown
## Candidates
- A: <name> @ <version>
- B: <name> @ <version>
- C: ...

## Decision matrix
| Axis | A | B | C | Notes |
|------|---|---|---|-------|

## Recommendation
**<Winner>** — because <2 sentences, citing the strongest evidence>.

## When you would pick a different one
- Pick B if: <condition>
- Pick C if: <condition>

## Migration cost from current solution
<if replacing an existing library>
```

## Step 7 — Surface the brief

Show the user:
1. The recommendation in one sentence.
2. The full matrix.
3. Conditions under which you would pick a different one.
4. Open questions for `pl-grill`.

Stop conditions: same as `rs-deep`. The recommendation must cite at least one source per scored cell — no scoring on vibes.
