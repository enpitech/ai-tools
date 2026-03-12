# React Review

7-pass React/Next.js code review system for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — CI + local modes.

## What It Does

Runs 7 sequential review passes on your React/Next.js code, catching real bugs instead of style nits:

| Pass | Focus | Examples |
|------|-------|---------|
| 1. BUGS | Logic errors | null access, race conditions, wrong conditionals |
| 2. SECURITY | Vulnerabilities | XSS, exposed secrets, unauthenticated Server Actions |
| 3. COMPONENT ARCHITECTURE | Structure | God components, prop drilling, cross-feature imports |
| 4. HOOKS & STATE | React patterns | Derived state in useEffect, stale closures, joinable hooks |
| 5. PERFORMANCE | Speed | Sequential awaits, missing dynamic imports, barrel file imports |
| 6. CODE QUALITY | React-specific | Array mutation, missing error boundaries, duplicated logic |
| 7. INTENT CHECK | PR scope | Unrelated changes that snuck into the diff |

Only reports **CRITICAL** (will break in prod) and **WARNING** (likely bug/security risk) at **8/10+ confidence**. No noise.

### Context-Aware

Automatically adjusts based on your `package.json`:
- **React Compiler** detected → skips unnecessary memoization findings
- **Next.js** detected → enables SSR/RSC rules (Server Action auth, serialization, Suspense)
- **Design system** detected (`@radix-ui`, `shadcn`) → flags excessive inline styling

## Two Modes

### Mode 1: PR Review (CI + Local)

Scoped to the PR diff + directly affected files. Fast and focused.

**CI — triggered by `/claude-review` comment on any PR:**

Posts inline comments directly on the changed lines.

**Local — via Claude Code CLI:**

```
/pr-review
```

Outputs findings to `pr-review-findings.md` and offers to apply fixes.

### Mode 2: System Review (Local Only)

Full codebase audit. Runs all 7 passes across every source file, plus system-level checks:

```
/system-review
```

Catches things a PR review never could:
- Dead exports, circular dependencies
- Duplicated patterns across distant files
- Architecture drift, inconsistent state patterns
- Bundle size hotspots

Outputs organized report to `system-review-findings.md`.

## Installation

### Option A: Copy files into your project

```bash
# Clone this repo
git clone https://github.com/enpitech/react-review.git

# Copy into your project
cp -r react-review/.claude your-project/
cp -r react-review/.github your-project/
```

### Option B: Cherry-pick what you need

| File | Purpose | Required? |
|------|---------|-----------|
| `.claude/rules/pr-review-criteria.md` | Shared 7-pass review criteria | Yes |
| `.claude/commands/pr-review.md` | Local `/pr-review` command | For local use |
| `.claude/commands/system-review.md` | Local `/system-review` command | For local use |
| `.github/workflows/claude-code-review.yml` | CI workflow | For GitHub Actions |

## CI Setup

1. Copy `.github/workflows/claude-code-review.yml` into your repo
2. Add `ANTHROPIC_API_KEY` to your repo secrets (Settings → Secrets → Actions)
3. Comment `/claude-review` on any PR

The workflow:
- Only runs for repo collaborators (OWNER/MEMBER/COLLABORATOR)
- Posts an acknowledgment comment, then inline review comments
- Claude cannot post arbitrary PR comments (hardened against prompt injection)

### Optional: Auto-trigger on PR

Uncomment the `pull_request` trigger in the workflow to run automatically on every PR:

```yaml
on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]
```

## Security

- `gh pr comment` removed from Claude's allowed tools — prevents prompt injection via malicious PR descriptions
- Claude can only post structured inline comments via MCP tool
- Completion message is posted by the workflow itself, outside Claude's control
- Trigger restricted to repo OWNER/MEMBER/COLLABORATOR only

## Customization

Edit `.claude/rules/pr-review-criteria.md` to:
- Add/remove review passes
- Adjust confidence threshold (default: 8/10)
- Change severity levels
- Add framework-specific checks
- Modify context-aware detection rules

Both modes and the CI workflow reference this single file — one source of truth.

## License

MIT
