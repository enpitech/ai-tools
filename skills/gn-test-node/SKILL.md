---
name: gn-test-node
description: Generate Node.js tests for a module, route handler, or service using the project's existing test runner (Vitest/Jest/Mocha/node:test). Covers async paths, error handling, event-loop concerns, and route-level integration. Use when adding or shoring up tests for Node code.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Edit, Write"
---

# GN-Test-Node — Node.js Test Generation

Follow the generation criteria defined in `rules/test-gen.md` and the Node review rules in `rules/node.md`.

## Step 1 — Detect test stack

Read `package.json` and detect:
- **Runner**: `vitest`, `jest`, `mocha`, `tap`, `node:test` (priority by what's installed).
- **HTTP test client**: `supertest`, `light-my-request` (Fastify), Hono's built-in test client.
- **Database fakes**: `pg-mem`, `mongodb-memory-server`, `prisma-client-js` test DB; or real test DB if integration.
- **Mocking**: `vi.mock`, `jest.mock`, `proxyquire`, `mock-require`.

## Step 2 — Detect framework

- `express`, `fastify`, `koa`, `hono`, `hapi`, `nestjs` → route-level test patterns.
- Bare HTTP / RPC → handler-level tests.
- ORM: `prisma`, `drizzle`, `typeorm`, `sequelize`, `mongoose`.

## Step 3 — Read the unit under test

Read target plus one level of consumers and any middleware that sits in front of the route (for route tests).

## Step 4 — Run the 5-pass model from `rules/test-gen.md`

Node-specific overlays:

### Case enumeration overlay (Pass 2)
For every async function, include:
- Happy path.
- Each `throw` site in the body, exercised with the input that triggers it.
- Promise rejection from an awaited dependency.
- Cancellation (if an `AbortSignal` is accepted).
- Concurrency — two calls in flight at once (especially for caches and connection pools).

For every route handler, include:
- 2xx happy path.
- 4xx for each validation branch.
- 401/403 for auth/authz failures (if the handler is protected).
- 5xx for downstream errors (mock the downstream).
- Request body too large (if size limits are configured).
- Headers: CORS, content-type, rate limit.

For every stream / event emitter, include:
- Normal end.
- Early consumer disconnect (`'close'` mid-stream).
- Error event handling.
- Backpressure (if writes are large).

### Test double overlay (Pass 3)
- HTTP downstream: prefer **`nock`** or **MSW for node** if installed.
- Database: prefer **in-memory fake** (`pg-mem`, `mongodb-memory-server`) over mocks; otherwise a transactional real test DB.
- Filesystem: use `memfs` or write to `os.tmpdir()` with cleanup.
- Time: fake timers; for `Date.now()` use the runner's clock control.
- Random: seed deterministically.

### Async overlay (Pass 5)
- Every `it`/`test` returns a promise or uses `async`.
- No `.then(done)` patterns — use `async/await`.
- `Promise.all` rather than serial awaits in test setup unless ordering matters.
- Verify no dangling timers / open handles (Jest will report; `node:test` requires `--test-detect-handles`).

## Step 5 — Write tests

Match repo convention: `*.test.ts` vs `*.spec.ts` vs `__tests__/`. Default to co-located when the repo is mixed.

## Step 6 — Surface

Show the user:
1. The case table.
2. The generated file paths.
3. The command to run them.
4. Any cases skipped and why (e.g., "skipped concurrency test — handler has no shared state").

## Stop conditions

- Never introduce a new test runner.
- Never hit real network / real production DB in unit or integration tests — flag if no fake is available.
- Never use `done()` callback style — async/await only.
- Never silence unhandled rejections to make tests pass.
