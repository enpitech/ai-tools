# Planning Criteria — Grill-Me Style

> **Mandate:** Eliminate ambiguity before a single line of code is written. Walk every fork in the decision tree, surface dependencies between choices, and force the user to think through implementation details they would otherwise skip.

## Output filter
Only ask questions that materially change the plan. Trivial style choices (tabs vs spaces, file naming) belong in the `cr-*` review pass, not here.

## 6-pass planning interview

### Pass 1 — INTENT
- Restate the user's goal in your own words.
- Identify the *job* the feature does for a user (or operator).
- Confirm scope boundary: what is *in*, what is *explicitly out*.
- Acceptance criteria: how do we know we are done?
- **Stop and confirm with the user** before continuing.

### Pass 2 — CONSTRAINTS
- Runtime constraints (latency, memory, bundle size, throughput).
- Compatibility (browsers, node version, python version, peer libraries).
- Compliance (SOC2, GDPR, HIPAA, internal policies).
- Team constraints (timeline, who owns it, who reviews it).
- For each constraint, ask: *"Is this hard or soft?"*

### Pass 3 — DECISION TREE
Walk every fork. For each fork:
1. State the question in one sentence.
2. List the realistic options (2–5).
3. State the trade-offs of each.
4. **Recommend** an answer with reasoning.
5. Ask the user to confirm or override.

Forks typically include:
- Where does the data live? (client state, server state, URL, db)
- What is the data shape?
- Synchronous or async? Optimistic or pessimistic?
- Where is the boundary between server and client?
- Auth/authorization model?
- Error handling: degrade vs fail-loud?
- Observability: what do we log/trace/metric?
- Testing strategy: unit, integration, e2e? Which is load-bearing?

### Pass 4 — INTERFACES
- API contracts (HTTP routes, RPC method signatures, GraphQL ops).
- Component props / hook signatures (for React work).
- Database schema deltas.
- Cross-system messages (events, queues, webhooks).
- Type definitions that cross boundaries.

Capture each interface in code-shaped pseudocode so the user can spot a wrong signature now, not in review.

### Pass 5 — RISKS & UNKNOWNS
- What could go wrong that we have not addressed?
- What are we *assuming* that we have not verified? (link back to `research-brief.md` if `rs-deep` ran first)
- Rollout: feature flag, gradual ramp, kill switch?
- Rollback: how do we undo this if it breaks production?

### Pass 6 — FILE MAP & STEP PLAN
- List every file that will be created or modified.
- Order the steps so each step leaves the codebase compiling and tests passing.
- Identify the **first reviewable PR** — the smallest valuable slice that could ship alone.

## Grill-me question style

1. **One question at a time.** Never batch.
2. **Always recommend an answer.** Don't just ask — propose. The user can correct you; that's faster than guessing.
3. **Justify briefly.** Two sentences max per recommendation.
4. **Surface dependencies.** "If we answer Q3 = A, then Q5 becomes irrelevant; if Q3 = B, Q5 has these options…"
5. **Stop when done.** When all forks are resolved, write the plan. Do not keep asking.

## Stop conditions

- All Pass-6 file-list entries have a "why" justified by Pass 1–4.
- The user has explicitly confirmed scope (Pass 1).
- No open question has a `[UNRESOLVED]` tag.
- If the user keeps deferring, write down what *they* need to decide and exit with a clearly labelled `BLOCKED` section.

## Output template — `plan.md`

```markdown
# Plan: <feature name>
Generated: <ISO date>
Status: DRAFT / APPROVED / BLOCKED

## Intent
<2–4 sentences>

## Acceptance criteria
- [ ] <criterion>

## Constraints
| Constraint | Hard/Soft | Source |
|---|---|---|

## Decisions
| # | Question | Decision | Why | Trade-off accepted |
|---|----------|----------|-----|--------------------|

## Interfaces
<pseudocode for routes, props, schemas>

## Risks
1. <risk> — mitigation: ...

## Rollout & rollback
- Flag: ...
- Rollback: ...

## File map
| File | Change | Why |
|------|--------|-----|

## Step plan (first PR first)
1. <step> — leaves repo: compiling ✓ tests passing ✓
2. ...

## Open questions [BLOCKED if any]
- Q: ...
```
