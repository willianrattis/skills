# Stack profile: TypeScript / JavaScript

The skills supply the discipline. This file supplies the concrete tooling for
that discipline in a TS/JS codebase. Read it whole before applying a
stack-coupled skill (`tdd`, `codebase-design`, `code-review`, `implement`,
`prototype`).

This is the stack the upstream skills were written against, so the existing
`tests.md` and `mocking.md` examples already match. Keep this profile thin and
let those files carry the detail.

## Commands

| Purpose       | Command                                    |
| ------------- | ------------------------------------------ |
| Run all tests | `pnpm test`                                  |
| Run one test  | `pnpm vitest run -t "<name>"`                |
| Types         | `pnpm tsc --noEmit`                          |
| Lint          | `pnpm lint`                                  |
| Full verify   | `pnpm tsc --noEmit && pnpm lint && pnpm test` |

Swap `pnpm` for whatever the repo's lockfile says.

## Test stack

- Runner: Vitest
- HTTP doubles: MSW at the network boundary
- Browser: Playwright, only for flows that genuinely need a browser
- Real dependencies: Testcontainers where a fake would lie

## Seams

- **Exported function or route handler** — the module's public entry.
- **Network boundary** — intercepted with MSW, not by mocking the fetch wrapper.
- **Component contract** — props in, rendered output and callbacks out. Never
  internal state.

Not seams: anything reached via `vi.mock` on a relative import path.

## Anti-patterns specific to this stack

- **`vi.mock` on internal modules.** Couples the test to file layout.
- **Snapshot tests as the primary assertion.** They record whatever the code
  does, including the bug.
- **`await waitFor` wrapping the whole test body** to paper over a race.
- **Testing through `data-testid`** when a role or label would do.

## Prototypes

A single self-contained HTML file for state and logic questions. For UI
questions, several radically different variations behind one route, toggleable.
