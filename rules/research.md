# Research Criteria — Pre-Plan Deep Research

> **Mandate:** Do not plan, design, or code until the agent has surveyed real-world evidence. Most plans fail because the agent works from training-data assumptions instead of the current state of the libraries, ecosystem, and user reports.

## Output filter
Report claims at **8/10+ confidence**, every claim cited by at least one URL with a date. Speculation must be explicitly labelled `[INFERENCE]`.

## 8-pass research model

### Pass 1 — SCOPE
- Restate the user's intent in your own words.
- Extract entities: feature names, libraries, APIs, file paths, frameworks, versions.
- List the explicit and implicit stack involved (read `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` / etc.).
- Output: a 5–10 line scope block at the top of the brief.

### Pass 2 — OFFICIAL DOCS
- Read the canonical documentation for every involved library/API.
- Use `context7` MCP if available (`resolve-library-id` → `query-docs`); otherwise WebFetch the docs site.
- Capture: current stable version, deprecations, migration notes, RFCs, breaking-change pages.
- Flag any divergence between the installed version and the docs you are reading.

### Pass 3 — ISSUE TRACKER
- Search the relevant GitHub repos' Issues and Discussions for the exact symptom or capability.
- Use `gh search issues "<keyword>" --repo <repo>` and `gh api` for Discussions.
- Capture: open bugs (with thumbs-up count), maintainer responses, "known limitations", closed-as-wontfix.
- Mine for: "doesn't work with X", "regression in v…", "memory leak", "race condition".

### Pass 4 — COMMUNITY SIGNAL
- Search Stack Overflow, Reddit (`/r/reactjs`, `/r/node`, `/r/python`, framework subs), Hacker News, dev.to, Medium.
- Use WebSearch with `site:` filters (e.g., `site:stackoverflow.com`, `site:news.ycombinator.com`).
- Capture: what real users complain about, recommended workarounds, sentiment shift over time.

### Pass 5 — PRIOR ART
- Search for existing OSS implementations, awesome-lists, and reference projects.
- `gh search repos`, `gh search code`, GitHub Topics.
- Capture: 2–5 representative projects that solved the same problem, link to the relevant file/commit.

### Pass 6 — SECURITY & CVES
- Search GHSA, NVD, Snyk, npm audit advisories, PyPI advisories for every library named in Pass 1.
- Capture: CVE IDs, affected version ranges, patched versions, exploitability notes.
- Cross-check the installed version against advisories.

### Pass 7 — CREATIVE PASS (mandatory)
- Re-read Passes 1–6. Ask: **"What would make my plan wrong?"**
- Run adversarial searches: "<library> considered harmful", "why we moved off <X>", "<X> vs <Y> benchmarks", "<X> post-mortem", "<X> 2026 alternative".
- Surface dissenting blog posts, conference talks, archived/maintainer-burnout signals.
- If Pass 7 contradicts Pass 2–6, **re-run** the contradicted pass with the new angle. Do not average — investigate.

### Pass 8 — SYNTHESIS
- Produce the structured `research-brief.md` (see template below).
- Every claim must cite. Every section must end with "Open questions for grill-me".

## Source-priority hints by stack

| Stack | High-trust sources |
|---|---|
| React | react.dev, Next.js docs/blog, RFCs, Dan Abramov / Sebastian Markbåge posts, GitHub `facebook/react` issues |
| Node.js | nodejs.org, V8 blog, OpenJS Foundation, `nodejs/node` GitHub, node-tap |
| Python | python.org, PEPs, python-dev mailing list, real-python.com, PyPA |
| Next.js | nextjs.org, Vercel blog, `vercel/next.js` issues |
| Django | docs.djangoproject.com, djangoproject.com/weblog, `django/django` issues |
| FastAPI | fastapi.tiangolo.com, `tiangolo/fastapi` issues |

## Hard guardrails

1. **Budget cap** — 5 fetches per pass, 35 total. If you hit the cap mid-pass, synthesize from what you have and flag gaps explicitly.
2. **Recency bias** — prefer sources from the last 18 months unless the topic is mature/stable. Print the date next to every link.
3. **No invented URLs** — every link must come from a real search result. Hallucinated links are a critical failure.
4. **Distrust the first answer** — if a search returns a confident single result, run two more queries with different phrasings before accepting it.
5. **Self-declared stop** — synthesize only after 3 consecutive searches yield no new claims. Otherwise keep digging.
6. **Cite or strike** — uncited claims must be removed or moved to a labelled `[INFERENCE]` block at the bottom.

## Output template — `research-brief.md`

```markdown
# Research Brief: <topic>
Generated: <ISO date>
Stack detected: <list>
Total sources consulted: <count>

## TL;DR
- <bullet 1, ≤ 20 words>
- <bullet 2>
- <bullet 3>

## Scope
<restated intent + entities + stack>

## Stack involved
| Library | Installed | Current stable | Deprecated? | Notes |
|---|---|---|---|---|

## Known issues & gotchas
- [HIGH] <issue> — source: <url> (<date>, <signal: 47👍 / wontfix / etc.>)
- [MED]  <issue> — source: <url>

## Community sentiment
- <pattern observed> — <evidence>

## Prior art
- <project> — <approach> — <link>

## Security
- <CVE / advisory> — <affected versions> — <patch> — <source>

## Risks for this work
1. <risk> — <why> — <evidence>
2. ...

## Open questions for grill-me
- Q1: <ambiguity that must be resolved before planning>
- Q2: ...

## [INFERENCE] section (optional)
<claims without direct citation, clearly labelled>

## Sources
1. <url> — <title> — <date>
2. ...
```
