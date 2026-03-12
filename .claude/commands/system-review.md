# System Review — Full Codebase React Audit (Mode 2)

Review scope: **Entire codebase.** Local-only — not available in CI.

Follow the review criteria defined in `.claude/rules/pr-review-criteria.md`.

## Step 1 — Detect project context

Read `package.json` and check for:
- `next` dependency → enable SSR/RSC rules (Pass 2 Server Actions, Pass 5 Suspense/serialization)
- `babel-plugin-react-compiler` or `react-compiler` → skip unnecessary memoization findings
- `@radix-ui`, `shadcn`, or similar → flag excessive inline styling more aggressively

## Step 2 — Map the codebase

Build a mental model of the project:
- List all source files: `find src -name '*.ts' -o -name '*.tsx' -o -name '*.js' -o -name '*.jsx' | head -200`
- Identify feature boundaries, shared folders, and entry points
- Note the component tree structure and routing layout

## Step 3 — Run 7 review passes across all source files

Apply all 7 review passes and filtering rules from the criteria file across every source file.

Additionally, since full codebase is visible, check for these **system-level issues** that a diff review cannot catch:

### System-Level Checks
- **Dead Exports**: exported functions/components/types with zero consumers across the codebase
- **Circular Dependencies**: import cycles between modules or feature folders
- **Duplicated Patterns**: same fetch/transform/render logic repeated in 2+ distant files — should be a shared hook or utility
- **Architecture Drift**: feature folders violating the intended structure (e.g. shared components importing from features)
- **Inconsistent State Patterns**: conflicting approaches (some features use Context, others use Zustand, others prop drill) without clear reason
- **Missing Error Boundaries**: route-level component trees without error boundaries
- **Bundle Size Hotspots**: statically imported heavy libraries that should use dynamic imports

## Step 4 — Output findings

Write all findings to `system-review-findings.md` in the project root, organized by pass:

```
# System Review Findings

Generated: [date]
Files scanned: [count]

## Pass 1 — BUGS
(findings or "✅ Clean")

## Pass 2 — SECURITY
(findings or "✅ Clean")

## Pass 3 — COMPONENT ARCHITECTURE
(findings or "✅ Clean")

## Pass 4 — HOOKS & STATE MANAGEMENT
(findings or "✅ Clean")

## Pass 5 — PERFORMANCE
(findings or "✅ Clean")

## Pass 6 — CODE QUALITY (React)
(findings or "✅ Clean")

## Pass 7 — INTENT CHECK
(skip — not applicable to full system review)

## System-Level Issues
(findings or "✅ Clean")

## Summary
- Total findings: [count]
- CRITICAL: [count]
- WARNING: [count]
- Top 3 files needing attention: [list]
```

Each finding should follow this format:
```
### [SEVERITY] file:line

**Issue:** one-line description
**Why:** reason you're confident this is a real issue
**Fix:** suggested code change
```

Show the user the contents of the findings file.

## Step 5 — Offer to fix

Ask the user: "Would you like me to apply fixes? I can do all CRITICAL first, then WARNING."

If yes, apply CRITICAL fixes first (one at a time with user confirmation), then WARNING fixes.
- Do not flag issues that are clearly intentional based on git blame context.
- Run everything autonomously without asking to confirm each step (except the final fix offer).
