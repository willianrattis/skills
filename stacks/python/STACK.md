# Stack profile: Python

The skills supply the discipline. This file supplies the concrete tooling for
that discipline in a Python codebase. Read it whole before applying a
stack-coupled skill (`tdd`, `codebase-design`, `code-review`, `implement`,
`prototype`).

## Commands

| Purpose        | Command                                          |
| -------------- | ------------------------------------------------ |
| Run all tests  | `uv run pytest`                                    |
| Run one test   | `uv run pytest -k "<expr>"`                        |
| Lint           | `uv run ruff check .`                              |
| Format check   | `uv run ruff format --check .`                     |
| Types          | `uv run mypy .`                                    |
| Full verify    | `uv run ruff check . && uv run mypy . && uv run pytest` |

If the repo uses Poetry or a bare venv instead of uv, drop the `uv run` prefix
and keep the rest.

## Test stack

- Framework: pytest
- Assertions: plain `assert`, with `pytest.approx` for floats
- HTTP doubles: respx (httpx) or responses (requests) — never monkeypatch the
  client object itself
- Real dependencies: Testcontainers, or the service's own docker-compose
- Async: `pytest-asyncio` in strict mode

## Seams

- **HTTP endpoint** — through `TestClient` / `httpx.AsyncClient` against the app
  object. Default seam for a feature slice.
- **Use case function or class** — the public callable, with its dependencies
  injected as arguments, not patched.
- **Consumed or published message** — asserted at the broker.

Not seams: anything reached through `unittest.mock.patch` on a module path.
Patching by string path couples the test to import layout; if you need it, the
seam is in the wrong place.

## Vertical slice

One slice is: failing endpoint test → route → use case → adapter → green.

## Naming

```python
def test_customer_with_unverified_email_cannot_complete_checkout(): ...
```

Names come from `CONTEXT.md`, not from the function under test.

## Anti-patterns specific to this stack

- **`mock.patch` as the default tool.** It is the last resort, not the first.
  Prefer passing a fake in.
- **Fixtures that build the whole world.** A fixture that every test uses and
  no test reads is a hidden global.
- **Asserting on `caplog` for behaviour.** Logs are a side channel; assert on
  the interface.
- **`# type: ignore` added to make mypy pass.** Fix the type or widen it
  deliberately, with a comment saying why.
- **Tautological parametrize.** `@pytest.mark.parametrize` that recomputes the
  expected value with the same expression as the code passes by construction.

## Deep modules in Python

A deep module is a package with a small `__init__` surface and real behaviour
inside. Signals of shallow: a `services.py` whose functions each call one
`repository.py` function; a module that re-exports another module's names and
adds nothing.

## Prototypes

A single `.py` file run with `uv run --with <deps> script.py`, or a notebook
that is deleted afterwards. No package scaffolding to answer a design question.
