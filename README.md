<div align="center">

<!-- Replace with your actual logo: add .github/assets/logo.png to the repo -->
<img src=".github/assets/banner.png" alt="Enpitech" width="" />

<br />
<br />

# AI Tools

**We've distilled 10+ years of frontend and fullstack engineering into a structured AI toolkit —<br />a skills repository purpose-built for AI agents and LLMs.**

[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-7C3AED)](https://code.claude.com/docs/en/plugins)
[![GitHub Actions](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF)](https://github.com/enpitech/ai-tools/actions)

[What's Inside](#whats-inside) · [Code Review](#code-review) · [Implementation Skills](#implementation-skills) · [All Skills](#available-skills) · [Installation](#installation) · [CI Setup](#ci-setup)

</div>

<br />

> **Built by [Enpitech](https://enpitech.com)** | a comprehensive AI engineering toolkit for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), VS Code Copilot, Cursor, and more. Multi-pass code review, Figma-to-code implementation, and growing. Works in CI and locally. Supports React, Node.js, Python, and any language.

<br />

## What's Inside

This toolkit packages Enpitech's 10+ years of frontend and fullstack expertise into structured AI skills. Each skill gives your agent a **deterministic, repeatable process** — not a vague prompt, but a step-by-step system.

<table>
<tr>
<td width="50%">

### 🔬 Code Review
Multi-pass review system: 5–7 sequential passes per language. Bugs → Security → Architecture → Performance → Quality. CI-ready.

### 🎨 Figma → Code
Pixel-perfect implementation from Figma designs via MCP. Auto breakpoints, design tokens, asset export, visual verification.

### 🔄 Two review scopes
`cr-*` reviews PR diffs (CI + local).<br />`cra-*` audits the full codebase (local, includes deps).

</td>
<td width="50%">

### 🧠 Auto-detection
Fullstack and general skills detect your language and framework automatically.

### ⚡ CI-ready
Ships with a GitHub Actions workflow. Comment `/cr-react` on a PR → threaded review comments with `/fix` autofix support.

### 🛠 Customizable
All criteria live in plain markdown files. Add passes, change thresholds, adapt to your stack. More skills coming.

</td>
</tr>
</table>

<br />

## Example Use Cases

<table>
<tr>
<td>

#### 💬 PR review in CI
A developer opens a PR with React changes. A teammate comments `/cr-react`. Claude runs 7 review passes on the diff and posts a threaded comment with findings. Each finding ends with "reply `/fix`" and the summary ends with "reply `/fix-all`".

</td>
<td>

#### 🔍 Pre-merge local check
Before pushing a Node.js branch, run `/enpitech:cr-node` locally. Get a findings report, then choose: save to file, fix step by step (approve each change), or fix all at once.

</td>
</tr>
<tr>
<td>

#### 🏗 Full codebase audit
A tech lead assesses a Python project before a refactor. `/enpitech:cra-python` scans every file + system-level checks (dead code, circular imports, config drift) + full dependency audit.

</td>
<td>

#### 🎨 Figma to production code
A designer hands off a Figma section. `/enpitech:figma-to-code` pulls the design via MCP, classifies nodes as assets or UI elements, uses your design tokens, and screenshots every breakpoint until it matches.

</td>
</tr>
<tr>
<td>

#### 🌐 Fullstack cross-layer review
A PR touches React frontend and Express backend. `/enpitech:cr-fullstack` auto-detects the stack, applies the right passes per layer, and checks cross-layer issues like API mismatches and auth flow gaps.

</td>
<td>

#### 📦 Dependency health check
Before a release, `/enpitech:cr-deps` checks for CVEs, outdated packages, deprecated deps, license issues, unused packages, and lockfile integrity.

</td>
</tr>
</table>

<br />

---

## Available Skills

### Code Review

Two scope prefixes, applied uniformly across all languages:

| Prefix | Scope | Where | Includes deps? |
|--------|-------|-------|:---:|
| `cr-` | PR diff + affected files | CI + Local | ✗ |
| `cra-` | Full codebase audit | Local only | ✓ |

<br />

| Skill | Scope | What it does | CI Trigger |
|:------|:-----:|:-------------|:----------:|
| `cr-react` | Diff | 7-pass React/Next.js review | `/cr-react` |
| `cra-react` | Full | React audit + system checks + dep audit | — |
| `cr-node` | Diff | 7-pass Node.js review | `/cr-node` |
| `cra-node` | Full | Node.js audit + system checks + dep audit | — |
| `cr-python` | Diff | 7-pass Python review | `/cr-python` |
| `cra-python` | Full | Python audit + system checks + dep audit | — |
| `cr-general` | Diff | 5-pass language-agnostic review | `/cr-general` |
| `cra-general` | Full | General audit + system checks + dep audit | — |
| `cr-deps` | Deps | 6-pass dependency health audit | `/cr-deps` |
| `cr-fullstack` | Diff | Auto-detect stack + cross-layer checks | `/cr-fullstack` |
| `cra-fullstack` | Full | Full audit per layer + cross-layer + dep audit | — |

> All review skills report only **CRITICAL** and **WARNING** findings at **8/10+ confidence**.

<br />

### Implementation Skills

| Skill | What it does | Requirements |
|:------|:-------------|:-------------|
| `figma-to-code` | Pixel-perfect Figma → responsive production code. Auto breakpoints, DS tokens, asset export, visual verification loop. | Figma MCP + Playwright MCP |

<br />

---

## Detailed Review Passes

<details>
<summary><strong>React/Next.js</strong> — 7 passes</summary>

<br />

| Pass | Focus | Examples |
|:-----|:------|:--------|
| 1. BUGS | Logic errors | null access, race conditions, wrong conditionals |
| 2. SECURITY | Vulnerabilities | XSS, exposed secrets, unauthenticated Server Actions |
| 3. COMPONENT ARCHITECTURE | Structure | God components, prop drilling, cross-feature imports |
| 4. HOOKS & STATE | React patterns | Derived state in useEffect, stale closures, joinable hooks |
| 5. PERFORMANCE | Speed | Sequential awaits, missing dynamic imports, barrel file imports |
| 6. CODE QUALITY | React-specific | Array mutation, missing error boundaries, duplicated logic |
| 7. INTENT CHECK | PR scope | Unrelated changes that snuck into the diff |

**Context-aware**: React Compiler, Next.js SSR/RSC, design systems (`@radix-ui`, `shadcn`).

```
/enpitech:cr-react          # PR diff review
/enpitech:cra-react         # Full codebase audit + system checks + dep audit
```

</details>

<details>
<summary><strong>Node.js</strong> — 7 passes (OWASP + eslint-plugin-security)</summary>

<br />

| Pass | Focus | Examples |
|:-----|:------|:--------|
| 1. BUGS | Logic errors | Unhandled promise rejections, race conditions, event loop blocking |
| 2. SECURITY | Vulnerabilities | eval/exec injection, prototype pollution, ReDoS, SSRF, missing helmet/CSRF |
| 3. ASYNC PATTERNS | Event loop | Blocking calls in async, missing Promise.all, stream backpressure |
| 4. ERROR HANDLING | Resilience | Bare catch, missing error events, uncaughtException without exit |
| 5. API DESIGN | Express/Fastify | Missing request size limits, rate limiting, input validation, permissive CORS |
| 6. PERFORMANCE | Runtime | Sync fs/crypto ops, missing connection pooling, N+1 queries, memory leaks |
| 7. CODE QUALITY | Node-specific | `require(variable)`, `new Buffer()`, deprecated APIs, missing graceful shutdown |

**Context-aware**: Express, Fastify, Koa; TypeScript; Prisma, Sequelize, TypeORM, Mongoose.

```
/enpitech:cr-node            # PR diff review
/enpitech:cra-node           # Full codebase audit + system checks + dep audit
```

</details>

<details>
<summary><strong>Python</strong> — 7 passes (Bandit + Ruff + Pylint)</summary>

<br />

| Pass | Focus | Examples |
|:-----|:------|:--------|
| 1. BUGS | Logic errors | Mutable default arguments, loop variable closures, unreachable code |
| 2. SECURITY | Vulnerabilities | eval/exec, pickle, subprocess shell=True, SQL injection, unsafe YAML, XML attacks |
| 3. TYPE SAFETY | Type correctness | Missing annotations, inconsistent returns, overly broad `Any` |
| 4. ASYNC PATTERNS | asyncio/threading | Blocking in async, missing await, sync sleep, thread safety |
| 5. API DESIGN | Django/Flask/FastAPI | Missing auth decorators, debug mode, insecure uploads, rate limiting |
| 6. PERFORMANCE | Efficiency | Generator vs list comprehension, quadratic string concat, inefficient loops |
| 7. CODE QUALITY | Pythonic | Bare except, mutable defaults, unused imports, missing context managers |

**Context-aware**: Django, Flask, FastAPI, SQLAlchemy, Pydantic, pytest, mypy/pyright.

```
/enpitech:cr-python          # PR diff review
/enpitech:cra-python         # Full codebase audit + system checks + dep audit
```

</details>

<details>
<summary><strong>General — Any Language</strong> — 5 passes</summary>

<br />

Works with Vue.js, Angular, Svelte, Go, Rust, Ruby, PHP, Java, Kotlin, Swift, C#, and more.

| Pass | Focus | Examples |
|:-----|:------|:--------|
| 1. BUGS | Logic errors | Null access, resource leaks, concurrency issues, off-by-one |
| 2. SECURITY | Vulnerabilities | Injection, hardcoded secrets, insecure crypto, SSRF, XSS, CSRF |
| 3. ERROR HANDLING | Resilience | Silent failures, bare exception catching, missing cleanup |
| 4. PERFORMANCE | Efficiency | Blocking I/O, N+1 queries, quadratic algorithms, memory issues |
| 5. CODE QUALITY | Maintainability | Dead code, duplication, overly complex functions, deprecated APIs |

**Auto-detects** language and framework, then applies framework-specific checks (Vue `v-html` XSS, Go unchecked errors, Rails mass assignment, Laravel raw queries, Spring injection, etc.).

```
/enpitech:cr-general         # PR diff review
/enpitech:cra-general        # Full codebase audit + system checks + dep audit
```

</details>

<details>
<summary><strong>Dependencies</strong> — 6 audit passes</summary>

<br />

| Pass | Focus | What It Checks |
|:-----|:------|:---------------|
| 1. VULNERABILITIES | Security | `npm audit` / `pip-audit` — CVEs by severity |
| 2. OUTDATED | Freshness | Major version lag, security-related updates |
| 3. DEPRECATIONS | Lifecycle | Deprecated packages, suggested replacements |
| 4. LICENSE | Compliance | Copyleft in permissive projects, missing licenses |
| 5. UNUSED | Bloat | Declared but never imported dependencies |
| 6. LOCKFILE | Integrity | Missing lockfile, unpinned versions, sync issues |

Auto-detects npm, yarn, pnpm, pip, poetry, uv, pipenv.

```
/enpitech:cr-deps
```

> Runs standalone, or automatically as part of any `cra-*` full audit. Not included in `cr-*` diff reviews.

</details>

<details>
<summary><strong>Fullstack — Cross-Layer</strong> (Auto-Detect)</summary>

<br />

**Dynamically detects** which languages are in the project and applies the right criteria per layer:

| Detected | Criteria applied |
|:---------|:-----------------|
| React/Next.js | `rules/react.md` |
| Express/Fastify/Koa | `rules/node.md` |
| Django/Flask/FastAPI | `rules/python.md` |
| Vue, Angular, Svelte, Go, Rust, Ruby, PHP, Java, etc. | `rules/general.md` |
| Monorepo | Reads each workspace's config to classify |

**Stack examples:**
- React + Express → React passes on frontend, Node passes on backend
- Vue + Go → General passes on both (auto-adapted)
- React + FastAPI → React passes on frontend, Python passes on backend

**Cross-layer checks** (always applied):
1. API contract validation
2. Shared type drift
3. Environment variable hygiene
4. Authentication flow consistency
5. Error contract matching
6. Data flow security
7. API versioning & deprecation

```
/enpitech:cr-fullstack       # PR diff + cross-layer checks
/enpitech:cra-fullstack      # Full audit per layer + cross-layer + system checks + dep audit
```

</details>

<details>
<summary><strong>Figma → Code</strong> — Implementation Skill</summary>

<br />

Converts Figma designs into pixel-perfect, responsive, production-ready code using MCP tools.

| Step | What it does |
|:-----|:-------------|
| 1. Variables | Collects section name, Figma URL, node IDs, route, selector from user |
| 2. Baselines | Pulls mobile/tablet/desktop images via Figma MCP |
| 3. Token mapping | Auto-discovers breakpoints, maps design values to existing DS tokens |
| 4. Asset classification | Classifies each node as ASSET (export as-is) or UI ELEMENT (build with code) |
| 5. Scaffold | Generates responsive code using DS primitives and tokens |
| 6. Visual verification | Screenshots via Playwright MCP, compares to Figma baseline, iterates |

**Requires**: Figma MCP server + Playwright MCP server running.

**Policies**: No invented content/styles. No new breakpoints. No custom sizes outside token scale. Assets used as-is (never recreated with CSS).

```
/enpitech:figma-to-code      # Interactive — asks for Figma URL and section details
```

</details>

<br />

---

## Installation

The skills and rules are structured markdown files. They work natively as a **Claude Code plugin**, but can also be used with **any AI coding assistant** that reads instructions from your repo — including VS Code with GitHub Copilot, Cursor, Windsurf, and others.

### Option A: Claude Code Plugin

**From a marketplace** search for "Enpitech" and install the "AI Tools" plugin. or in claude code, run:

```
/plugins install enpitech
```

**From a local directory:**

```bash
git clone https://github.com/enpitech/ai-tools.git
claude --plugin-dir ./ai-tools
```

Or add it permanently to your project's `.claude/plugins.json`.

Skills become `/enpitech:cr-react`, `/enpitech:cra-react`, `/enpitech:cr-node`, etc.

> **Note:** Plugins don't install workflow files. Copy the CI workflow manually:
> ```bash
> cp -r ai-tools/.github your-project/
> ```

### Option B: Copy into your project (works with any AI tool)

```bash
# Clone this repo
git clone https://github.com/enpitech/ai-tools.git

# Copy into your project
cp -r ai-tools/rules your-project/rules
cp -r ai-tools/skills your-project/skills
cp -r ai-tools/.github your-project/        # CI workflow (optional)
```

Once the files are in your repo, any AI coding assistant can use them:

| Tool | How it picks up the skills |
|:-----|:---------------------------|
| **Claude Code** | Reads `skills/` and `rules/` automatically. Invoke with `/cr-react`, `/cra-node`, etc. |
| **GitHub Copilot (VS Code)** | Reference the rules files as context in chat, or add them to `.github/copilot-instructions.md` |
| **Cursor** | Add rules files to `.cursor/rules/` or reference them in chat context |
| **Windsurf** | Reference the criteria markdown files as project context |
| **Other AI assistants** | Point the agent to the relevant `rules/*.md` file — they're self-contained review criteria |

> The `rules/*.md` files are the core value — they contain all the review criteria and work with any LLM. The `skills/*/SKILL.md` files add agent-specific automation (diff collection, file scanning, output formatting, MCP orchestration).

<details>
<summary><strong>Cherry-pick what you need</strong></summary>

<br />

**Rules** (review criteria — pick by language):

| File | Purpose |
|:-----|:--------|
| `rules/react.md` | React/Next.js 7-pass review criteria |
| `rules/node.md` | Node.js 7-pass review criteria |
| `rules/python.md` | Python 7-pass review criteria |
| `rules/general.md` | Language-agnostic 5-pass review criteria |
| `rules/deps.md` | Dependency audit criteria |
| `rules/fullstack.md` | Cross-layer check criteria |
| `rules/autofix.md` | Autofix workflow (local options + CI comment format) |

**Skills** (pick by language + scope):

| File | Skill |
|:-----|:------|
| `skills/cr-react/SKILL.md` | PR diff React review |
| `skills/cra-react/SKILL.md` | Full codebase React audit |
| `skills/cr-node/SKILL.md` | PR diff Node.js review |
| `skills/cra-node/SKILL.md` | Full codebase Node.js audit |
| `skills/cr-python/SKILL.md` | PR diff Python review |
| `skills/cra-python/SKILL.md` | Full codebase Python audit |
| `skills/cr-general/SKILL.md` | PR diff general review |
| `skills/cra-general/SKILL.md` | Full codebase general audit |
| `skills/cr-deps/SKILL.md` | Dependency health audit |
| `skills/cr-fullstack/SKILL.md` | PR diff fullstack review |
| `skills/cra-fullstack/SKILL.md` | Full codebase fullstack audit |
| `skills/figma-to-code/SKILL.md` | Figma → pixel-perfect production code |

</details>

<br />

---

## CI Setup

1. Copy `.github/workflows/claude-code-review.yml` into your repo
2. Add `ANTHROPIC_API_KEY` to your repo secrets (Settings → Secrets → Actions)
3. Comment one of these on any PR:

### Review Commands

| Trigger | Review Type |
|:--------|:------------|
| `/cr-react` | React/Next.js code review |
| `/cr-node` | Node.js code review |
| `/cr-python` | Python code review |
| `/cr-general` | Language-agnostic code review |
| `/cr-deps` | Dependency health audit |
| `/cr-fullstack` | Fullstack auto-detect + cross-layer |

> `cra-*` skills (full audits) are local-only and not triggered in CI.

### Autofix Commands

After a review posts findings, reply to apply fixes:

| Command | What it does | Reply to |
|:--------|:-------------|:---------|
| `/fix` | Apply the fix for a single finding | An individual finding comment |
| `/fix-all` | Apply all suggested fixes at once | The main review summary comment |

The autofix job checks out the PR branch, applies the fix(es), commits, and pushes automatically.

### How it works

The workflow:
- Detects the trigger keyword and selects the appropriate review criteria
- Only runs for repo collaborators (OWNER/MEMBER/COLLABORATOR)
- Posts a summary comment with all findings, each linking to `/fix`
- Posts individual finding replies with full details and suggested code changes
- `/fix` and `/fix-all` replies trigger a separate job that applies fixes and pushes

<details>
<summary><strong>Optional: Auto-trigger on PR</strong></summary>

<br />

Uncomment the `pull_request` trigger in the workflow:

```yaml
on:
  pull_request:
    types: [opened, synchronize]
  issue_comment:
    types: [created]
```

</details>

<br />

---

## Security

- Trigger restricted to repo OWNER/MEMBER/COLLABORATOR only
- Autofix only applies changes suggested in review findings — no arbitrary modifications
- Fix job requires explicit `/fix` or `/fix-all` reply from an authorized collaborator
- All commits are attributed to `github-actions[bot]`

## Customization

Edit the criteria files in `rules/` to add/remove review passes, adjust confidence thresholds, change severity levels, or add framework-specific checks.

| File | Controls |
|:-----|:---------|
| `rules/react.md` | React/Next.js review rules |
| `rules/node.md` | Node.js review rules |
| `rules/python.md` | Python review rules |
| `rules/general.md` | Language-agnostic review rules |
| `rules/deps.md` | Dependency audit rules |
| `rules/fullstack.md` | Cross-layer check rules |
| `rules/autofix.md` | Autofix workflow (local + CI) |

All review skills reference these files — single source of truth per concern.

Implementation skills like `figma-to-code` are self-contained in their `SKILL.md` — no separate rules file needed.

<br />

---

<div align="center">

**Built with ❤️ by [Enpitech](https://enpitech.com)**

MIT License

</div>

## Contributors

<a href="https://github.com/lirankor">
  <img src="https://github.com/lirankor.png" width="60" height="60" style="border-radius:50%" alt="lirankor" />
</a>
&nbsp;&nbsp;
<a href="https://github.com/nir2002">
  <img src="https://github.com/nir2002.png" width="60" height="60" style="border-radius:50%" alt="nir2002" />
</a>

## License

MIT
