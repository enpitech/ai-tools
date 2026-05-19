---
name: gn-test-react
description: Generate React/Next.js tests for a component, hook, or route using the project's existing test runner (Jest/Vitest + React Testing Library / Playwright). Produces accessible-query-based tests covering happy path, loading/error/empty states, and accessibility. Use when adding or shoring up tests for React code.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Edit, Write"
---

# GN-Test-React — React Test Generation

Follow the generation criteria defined in `rules/test-gen.md` and the React review rules in `rules/react.md`.

## Step 1 — Detect test stack

Read `package.json` and detect:
- **Runner**: `vitest`, `jest`, `bun:test`, `node:test` (in priority order of what's installed).
- **DOM library**: `@testing-library/react`, `enzyme` (legacy — warn the user), `@testing-library/jest-dom`.
- **E2E**: `@playwright/test`, `cypress`.
- **Snapshot policy**: check `package.json` and existing `__snapshots__/` — match the repo's existing approach.

If multiple are present, prefer the one used by *most* existing tests (`git ls-files '*.test.*' | head -20`).

## Step 2 — Detect Next.js / RSC / Compiler context

From `package.json`:
- `next` → flag Server Components vs Client Components; use `next/test` helpers if available.
- `babel-plugin-react-compiler` or `react-compiler` → skip tests that assert on memoization behaviour.
- `@radix-ui`, `shadcn` — use accessible roles matching those primitives.

## Step 3 — Read the unit under test

Read the target file(s) plus:
- One level of direct consumers.
- The closest existing test file as a *style guide* (file naming, helper imports, common setup).

## Step 4 — Run the 5-pass model from `rules/test-gen.md`

Apply Passes 1–5. React-specific overlays:

### Case enumeration overlay (Pass 2)
For every component, include:
- Renders happy path with required props.
- Renders with optional props omitted.
- Loading state (if component fetches or accepts a `loading` prop).
- Error state (if component shows errors).
- Empty state (if component shows lists).
- User interaction — click, type, submit (using `userEvent`, not `fireEvent`).
- Accessibility — element has correct role, label, and is reachable by keyboard.
- Cleanup — for components that subscribe / use `useEffect`, verify unmount doesn't leak.

For every custom hook, include:
- Initial state.
- Each state transition.
- Cleanup on unmount.
- Behaviour when arguments change (re-render with new deps).

### Test double overlay (Pass 3)
- Network: prefer **MSW** if installed; otherwise fake `fetch` / mock module.
- Time: `vi.useFakeTimers()` / `jest.useFakeTimers()`.
- Routing: `next/navigation` mocks for App Router; `next/router` mocks for Pages Router.
- Context: use the **real** provider with a test-only initial value, not a mocked Context.

### Query overlay (Pass 5)
- **Required**: `getByRole`, `getByLabelText`, `getByText`.
- **Last resort**: `getByTestId` — only if no semantic alternative exists; add a comment explaining why.
- **Never**: querying by class name, by CSS selector, or by `container.querySelector` unless the test is *about* the DOM structure.

## Step 5 — Write tests

Co-locate next to the file under test unless the repo uses `__tests__/`. Match the existing test-file naming exactly.

## Step 6 — Surface

Show the user:
1. The case table.
2. The generated file paths.
3. The command to run them (e.g., `npx vitest run path/to/file.test.tsx`).
4. Any cases intentionally skipped with one-line reasons.

## Stop conditions

- Never introduce a new test runner.
- Never mutate the component under test to "make it testable" — flag the issue instead and let `pl-grill` decide whether to refactor.
- Never assert on render count unless the test is explicitly about performance / memoization.
- Never use `act()` wrappers unless RTL warns — `userEvent` handles act automatically.
