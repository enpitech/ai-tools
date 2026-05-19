---
name: a11y-audit
description: Run a focused accessibility audit on the current PR diff or a target directory. Checks against WCAG 2.2 AA, ARIA practices, and keyboard reachability — separate from the main cr-react review so it can run standalone in CI or locally. Use before shipping any user-facing UI change.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write"
---

# A11Y-Audit — Accessibility Audit (Standalone)

Splits accessibility findings out of `cr-react` so they can run independently in CI or locally.

## Step 1 — Determine scope

Two modes:

**Diff mode (default in CI):**
1. `gh pr view --json baseRefName` — store base.
2. `git diff <base>...HEAD --name-only` — limit checks to changed files + direct consumers.

**Full mode (`/enpitech:a11y-audit --full`):**
1. Glob all UI files (`*.tsx`, `*.jsx`, `*.vue`, `*.svelte`, `*.html`).

## Step 2 — Detect UI library

From `package.json`:
- `@radix-ui/*`, `@headlessui/*`, `react-aria` → high-trust primitives; flag overrides that strip a11y props.
- `shadcn-ui` (no package, uses Radix under the hood) → same.
- `material-ui` / `@mui/*`, `@chakra-ui/*`, `mantine` → reasonable defaults; check overrides.
- Custom / unknown → audit every interactive element from scratch.

## Step 3 — Run the audit checklist

Apply each check to the in-scope files. Report only **CRITICAL** and **WARNING** at 8/10+ confidence.

### Semantics & structure
- Heading hierarchy: no skipped levels (h1 → h3 without h2). One h1 per page.
- Landmarks: `header`, `nav`, `main`, `footer` present on top-level pages.
- Lists use `<ul>` / `<ol>` / `<dl>`, not stacked `<div>`s.
- Buttons vs links: `<button>` for actions, `<a>` for navigation. No `<div onClick>`.

### Forms
- Every input has a programmatic label (`<label htmlFor>` or `aria-label` or `aria-labelledby`).
- Error messages associated via `aria-describedby` / `aria-errormessage`.
- Required fields marked with `aria-required` and visually.
- Fieldsets group related radios/checkboxes.
- Autocomplete attributes where applicable.

### Images & media
- `<img>` has meaningful `alt`, or `alt=""` for decorative.
- SVG icons have `aria-label` or `aria-hidden="true"` paired with adjacent text.
- Video/audio has captions or transcripts.

### Color & contrast
- Text contrast ≥ 4.5:1 (3:1 for large text).
- UI components and graphical objects ≥ 3:1.
- Information is not conveyed by color alone (icon + text, not red text only).

### Keyboard
- Every interactive element reachable by Tab.
- Focus visible (no `outline: none` without replacement).
- No keyboard traps (modal must support Esc and Tab cycling).
- Custom widgets follow WAI-ARIA Authoring Practices for keyboard interaction.

### ARIA hygiene
- No `role` that contradicts the element (`<button role="link">` etc.).
- `aria-hidden="true"` not on a focusable element.
- Live regions (`aria-live`) used for dynamic updates — but not for everything.
- No `tabindex` greater than 0.

### Dynamic UI
- Modals: focus moves in, returns on close, background inert.
- Toasts: announced via `aria-live="polite"` or status role.
- Disclosure widgets (`aria-expanded`), tabs (`aria-selected`), menus (`role="menu"` + `aria-haspopup`).

## Step 4 — Output findings

Same format as `cr-react`:

```
### [SEVERITY] file:line

**Issue:** one-line description
**WCAG:** 2.2 SC X.Y.Z (Level A/AA)
**Why:** reason you're confident this is a real issue
**Fix:** suggested code change
```

Write to `a11y-findings.md` in local mode; output to PR comments in CI (same posting mechanism as `cr-react`).

## Step 5 — Autofix

Follow `rules/autofix.md`. Local mode offers three options; CI mode supports `/fix` and `/fix-all`.

## Stop conditions

- Do not flag visual styling that has no a11y impact — that's `cr-react` Pass 6.
- Do not require ARIA where native semantics already cover it ("the first rule of ARIA is don't use ARIA").
- Do not down-score known-good primitives (Radix/Headless UI defaults) unless the consumer broke them.
