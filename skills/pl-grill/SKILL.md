---
name: pl-grill
description: Interview the user with structured grill-me-style questions until every fork in the design is resolved, then write a plan.md. Use when starting any non-trivial feature, bugfix, or refactor. Do not write code until pl-grill has produced an approved plan.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Write"
---

# PL-Grill — Grill-Me Planner

Sits between `rs-deep` and implementation in the Enpitech pipeline:

```
rs-deep  →  pl-grill  →  implementation  →  cr-*
```

Follow the planning criteria defined in `rules/plan.md` (6 passes, grill-me question style, output template).

## Step 1 — Detect prior research

Check whether `research-brief.md` exists in the repo root. If it does:
- Read it.
- Seed Pass 5 (Risks & Unknowns) with the brief's risk section.
- Seed the first round of grill-me questions from the brief's "Open questions for grill-me" section.

If no brief exists and the topic is non-trivial, *suggest* the user run `/enpitech:rs-deep` first — but proceed if they decline.

## Step 2 — Detect project context

Read `package.json`, `pyproject.toml`, `requirements.txt`, etc. Pre-load:
- Detected stack(s) → adjust question framing (React → ask about RSC/client boundary; Node → ask about event-loop blocking; Python → ask about sync/async).
- Existing conventions visible in the repo (folder structure, naming, state library).

Do not lecture the user about these — use them silently to ask better questions.

## Step 3 — Run the 6-pass interview

Apply Passes 1–6 from `rules/plan.md`.

**Critical rules:**
- Ask **one question at a time**. Never batch.
- **Always include a recommended answer** with brief justification. The user can correct you — that is faster than guessing.
- Surface dependencies between questions ("if Q3 = A, then Q5 becomes irrelevant").
- Stop asking when the decision tree is resolved. Do not pad with trivial questions.

## Step 4 — Write `plan.md`

Use the template from `rules/plan.md`. Write to the project root (or `plans/` directory if it exists).

## Step 5 — Surface the plan

Show the user:
1. The Intent section.
2. The Decisions table.
3. The Step plan (numbered).
4. Any `BLOCKED` items.

End with: *"Plan written to `plan.md`. Approve to start implementation, or push back on any decision."*

## Stop conditions

- All file-map entries have a "why" justified by Passes 1–4.
- No `[UNRESOLVED]` tags remain in the plan.
- The user has explicitly approved the scope (Pass 1).
- If blocked on a decision the user keeps deferring, write the `BLOCKED` section and exit — do not invent an answer.

## What this skill does NOT do

- Write code.
- Make assumptions about which library to add (that is `rs-library-pick`).
- Skip questions to "save time" — the whole point is to surface ambiguity now, not in review.
