---
name: rs-deep
description: Run a multi-pass deep research sweep before planning or coding — official docs, GitHub issues, community reports, prior art, CVEs, plus an adversarial creative pass. Use when starting a non-trivial feature, library swap, migration, or bug investigation, or whenever the user asks for "deep research" before implementation.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write, WebSearch, WebFetch"
---

# RS-Deep — Pre-Plan Deep Research

Sits **above** `pl-grill` in the Enpitech pipeline:

```
rs-deep  →  pl-grill  →  implementation  →  cr-*
```

Goal: ground every plan in real-world evidence before a line of code is written.

Follow the research criteria defined in `rules/research.md` (8 passes, hard guardrails, output template).

## Step 1 — Confirm the topic with the user

Ask one focused question if the topic is ambiguous: *"What problem are we researching? Feature, library choice, bug, or migration?"*

Do not start fetching until the topic is clear.

## Step 2 — Detect project context

Read every manifest in the repo root: `package.json`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, `composer.json`, `build.gradle`, `pom.xml`.

Record installed versions for every library named in the topic. These become the anchor for Pass 2 (docs) and Pass 6 (CVEs).

## Step 3 — Run all 8 passes from `rules/research.md`

Apply each pass in order. Honour the budget (5 fetches/pass, 35 total). Honour the stop condition (3 consecutive empty searches before synthesis).

The **creative pass** (Pass 7) is mandatory and not optional — if you skip it the brief is incomplete.

## Step 4 — Write `research-brief.md`

Write the synthesis to `research-brief.md` in the project root, following the template in `rules/research.md`.

The **Open questions for grill-me** section is the hand-off contract — `pl-grill` will consume it as its starting question bank.

## Step 5 — Surface the brief

Show the user:
1. The TL;DR (3 bullets max).
2. The top 3 risks.
3. The list of open questions for grill-me.
4. Path to the full brief.

End with the suggestion: *"Run `/enpitech:pl-grill` next to lock down the plan against these open questions."*

## Stop conditions

- **No invention.** Hallucinated URLs are a critical failure — every link comes from a real search result.
- **Cite or strike.** Uncited claims go in the `[INFERENCE]` block or get removed.
- **Budget cap.** 35 fetches total; flag any pass that hit the cap.
- **No planning.** This skill researches only. Do not produce a plan or write code — that is `pl-grill`'s job.
