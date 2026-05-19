# Test Generation Criteria

> **Mandate:** Generate tests that catch real bugs, not coverage filler. Tests must mirror the rules used by `cr-*` reviewers — what you generate, the reviewer accepts.

## Hard rules

1. **Test behaviour, not implementation.** Never assert on private state, internal call order, or render counts (unless the test is *about* render counts).
2. **One reason to fail per test.** If a test could fail for two unrelated reasons, split it.
3. **Arrange–Act–Assert** layout. Use blank lines to separate.
4. **No magic literals in assertions** — name the expected value with a variable describing intent.
5. **No flakiness** — never use `setTimeout` waits or arbitrary `sleep`. Use the framework's wait primitives.
6. **No conditional assertions** — `if … expect … else expect …` masks bugs. Use a parameterized test or two tests.
7. **Test names are sentences** — "returns 404 when user is not found", not "test_get_user_2".

## 5-pass generation model

### Pass 1 — UNDERSTAND THE UNIT
- Read the file under test plus its direct consumers.
- Identify public API: exported functions, components, classes, hooks, endpoints.
- Identify side effects: network, db, fs, time, randomness.
- Identify dependencies that should be faked vs real.

### Pass 2 — ENUMERATE CASES
Produce a case table:

| # | Input / state | Expected outcome | Why this case matters |
|---|---|---|---|

Must include:
- Happy path — the obvious successful case.
- Empty input — `null`, `[]`, `''`, `undefined`, missing keys.
- Boundary values — 0, -1, max, off-by-one.
- Failure modes — network down, db error, invalid auth, permission denied.
- Race conditions — concurrent calls, stale closures, cancelled requests.
- Accessibility states (UI tests) — disabled, loading, error, empty.

### Pass 3 — CHOOSE TEST DOUBLES
For each dependency from Pass 1:
- **Real** — pure functions, in-memory data structures, lightweight modules.
- **Stub** — return a fixed value, no behaviour verification.
- **Mock** — verify the call was made with specific args.
- **Fake** — in-memory implementation (in-memory db, fake clock).

Default to **real or fake** wherever possible. Mocks are the most brittle option.

### Pass 4 — WRITE THE TESTS
- Co-locate next to the file under test unless the repo convention is a `__tests__` folder.
- Use the framework already in `package.json` / `pyproject.toml`. Never introduce a new test runner.
- Follow existing file naming: `foo.test.ts` vs `foo.spec.ts` vs `test_foo.py`.

### Pass 5 — REVIEW AGAINST THE LANGUAGE RULES
Run the corresponding `rules/<stack>.md` checklist mentally against the generated tests:
- React: tests use `@testing-library/react` accessible queries (`getByRole`, `getByLabelText`), not `getByTestId` unless no other option.
- Node: async tests `await` correctly, no hanging promises, no real network unless integration.
- Python: pytest fixtures used over `setUp`, no mutable defaults in helpers, asyncio tests properly marked.

If a generated test would fail a `cr-*` pass, **rewrite it** before showing the user.

## Output format

Show the user:
1. The case table from Pass 2 (so they can spot missing cases).
2. The generated test file(s).
3. The command to run them.
4. Any cases you *intentionally skipped* with a one-line reason.

## What test-gen does NOT do

- Mutate production code (even to make it more testable — flag instead).
- Lower thresholds in existing coverage configs.
- Introduce new test runners or frameworks.
- Generate tests against the implementation (e.g., asserting an internal helper was called). Tests should target the public API.
