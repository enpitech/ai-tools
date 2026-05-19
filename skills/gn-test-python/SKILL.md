---
name: gn-test-python
description: Generate Python tests using pytest (or the project's existing runner) for a module, function, class, or API route. Covers happy path, edge cases, async paths, and framework-level integration (Django/Flask/FastAPI). Use when adding or shoring up tests for Python code.
disable-model-invocation: true
allowed-tools: "Read, Grep, Glob, Bash, Edit, Write"
---

# GN-Test-Python — Python Test Generation

Follow the generation criteria defined in `rules/test-gen.md` and the Python review rules in `rules/python.md`.

## Step 1 — Detect test stack

Read `pyproject.toml` / `setup.cfg` / `requirements*.txt` and detect:
- **Runner**: `pytest` (default), `unittest`, `nose2`.
- **Fixtures**: existing `conftest.py` files — read the closest one.
- **HTTP test client**: `httpx.AsyncClient`, `fastapi.TestClient`, `django.test.Client`, Flask `app.test_client()`.
- **DB fakes / fixtures**: `pytest-django`, `pytest-asyncio`, `pytest-postgresql`, `factory_boy`, `model_bakery`.
- **Mocking**: `unittest.mock`, `pytest-mock`, `responses` (for `requests`), `respx` (for `httpx`).

## Step 2 — Detect framework

- `django` → ORM fixtures via `pytest-django`, `TestCase` vs function-style.
- `flask` → app factory pattern usage.
- `fastapi` → `TestClient` and dependency overrides.
- `sqlalchemy` → session fixtures, transaction rollback per test.
- `pydantic` → use models in test data, not raw dicts.

## Step 3 — Read the unit under test

Read target plus the closest `conftest.py` and one existing test as a style guide.

## Step 4 — Run the 5-pass model from `rules/test-gen.md`

Python-specific overlays:

### Case enumeration overlay (Pass 2)
For every function, include:
- Happy path.
- Each `raise` site in the body.
- `None` / empty input where the type allows.
- Boundary values for any numeric or sliceable input.
- Type-coerced inputs — what does it do with a `str` when expecting `int`? (Often a real bug.)
- Mutable default argument trap — if the signature has one, write a test that catches it.

For every async coroutine, include:
- All of the above + cancellation (`asyncio.CancelledError`).
- Concurrent `gather()` — does shared state corrupt?

For every API route, include:
- 2xx happy.
- 4xx validation per field (pydantic errors).
- 401/403 for protected routes.
- 5xx when a downstream raises.
- Pagination, filtering, sorting if the route supports them.

For every Django model / SQLAlchemy model, include:
- `__str__` / `__repr__` produces expected string.
- Required fields raise on missing.
- Unique constraints raise on duplicate.
- Custom `clean()` / validators trigger.

### Fixture overlay (Pass 3)
- Prefer **pytest fixtures** over `setUp`/`tearDown`.
- Use `tmp_path` for filesystem, not `/tmp` directly.
- Use `monkeypatch` for env vars / `os.environ`.
- Time: `freezegun` if installed, otherwise patch `datetime.now()` with a fixture.
- Random: seed via fixture; never rely on default seeding.

### Async overlay (Pass 5)
- Mark with `@pytest.mark.asyncio` (or use `anyio` if the project does).
- Never call `asyncio.run()` inside a test — let the plugin handle the loop.
- Use `respx` for `httpx` and `aioresponses` for `aiohttp`.

## Step 5 — Write tests

Match repo convention: `test_*.py` in a `tests/` tree, or co-located with `_test.py` suffix. Pick whichever the existing tests use.

## Step 6 — Surface

Show the user:
1. The case table.
2. The generated file paths.
3. The command to run them (e.g., `pytest tests/test_foo.py -v`).
4. Any cases skipped and why.

## Stop conditions

- Never introduce a new test runner.
- Never hit real network in unit tests — use `responses` / `respx` / fixture overrides.
- Never use `time.sleep` to coordinate; use the runner's primitives.
- Never lower coverage thresholds to make tests pass.
