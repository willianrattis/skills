# Stack profile: Vanilla JS + Vite + Firebase

Plain JavaScript ES modules — no framework, no TypeScript. Vite for dev and
build, Vitest for tests, ESLint 9 flat config. Firebase Auth and Firestore as
the backend, Chart.js for graphs, jsPDF for exports.

Read this whole file before applying a stack-coupled skill (`tdd`,
`codebase-design`, `code-review`, `implement`, `prototype`).

There is no type checker. Do not propose `tsc`, do not add type annotations,
do not suggest converting files to `.ts` as part of an unrelated task.

## Commands

| Purpose       | Command                                       |
| ------------- | --------------------------------------------- |
| Dev server    | `npm run dev`                                   |
| Run all tests | `npm test`                                      |
| Watch tests   | `npm run test:watch`                            |
| Run one file  | `npx vitest run tests/<name>.test.js`           |
| Run one test  | `npx vitest run -t "<test name>"`               |
| Lint          | `npm run lint`                                  |
| Autofix       | `npm run lint:fix`                              |
| Build         | `npm run build`                                 |
| Full verify   | `npm run lint && npm test && npm run build`      |

Run the full verify before declaring a slice green. The build is part of it:
Vite catches broken imports that Vitest does not, because the test environment
resolves differently from the bundler.

## Test layout

Tests live in `tests/` at the repo root, **not** colocated with source. One
file per domain concern, named after the concern rather than the source file:
`session.test.js`, `deload.test.js`, `suggestion.test.js`.

`tests/fixtures.js` holds shared builders. Reach for it before inventing new
sample data — divergent fixtures are how two tests come to disagree about what
a valid session looks like.

Follow the existing naming when adding a file. If a new test does not fit any
existing concern, that is a signal worth raising: either the concern is new, or
it belongs in a file that already exists.

## Seams

- **Exported function of a module under `src/domain/`** — the primary seam.
  Pure logic, plain objects in and out. Almost every test should live here.
- **`src/core/` and `src/features/` entry points** — orchestration. Testable
  with fakes passed in, never with `vi.mock` on a relative path.
- **The Firestore adapter boundary** — the small set of functions that read and
  write documents. Everything above them must be reachable without Firebase.
- **`firestore.rules`** — a seam in its own right. Rules are the only real
  enforcement of who can read whose training data; a UI check is not access
  control. Test them against the emulator with `@firebase/rules-unit-testing`.

Not seams: DOM structure, CSS classes, element IDs, Chart.js internals.

## The rule that keeps this codebase testable

Firebase calls belong in the adapter layer. Domain logic — progression,
superset grouping, deload decisions, chart series — takes plain objects and
returns plain objects, with no `db`, no `doc()`, no `onSnapshot` in sight.

The existing `tests/` suite works because that separation already largely
holds. Every new feature either preserves it or erodes it.

Anything time-dependent takes `now` as a parameter. Never call `Date.now()`
inside a domain function — it makes the behaviour untestable and, for anything
involving elapsed time, wrong the moment the app is backgrounded.

## Vertical slice

One slice is: failing test in `tests/` → domain function → orchestration →
adapter → UI → green.

Not: all the domain functions first, then all the wiring.

## Naming

Names come from the project's glossary. If the domain says "série efetiva",
the code does not say `validSet`. Test names read as specifications:

```js
it("suggests a deload after three stalled sessions", ...)
```

Not `it("works")`, not `it("test deload 2")`.

## Anti-patterns specific to this stack

- **`vi.mock` on a relative import.** Couples the test to file layout. Pass the
  dependency in instead. If that is awkward, the seam is in the wrong place.
- **Mocking the Firebase SDK.** Use the emulator, or test the domain function
  that does not touch Firebase at all.
- **`Date.now()` inside domain logic.** See above.
- **Firestore documents shaped for the UI.** The array-to-object change for
  superset persistence exists because arrays lost identity across writes. Model
  for the data's constraints, not for what renders conveniently.
- **`onSnapshot` listeners without teardown.** Every listener a view registers
  must be unsubscribed when the view goes away, or the app leaks and
  double-renders.
- **Chart.js instances recreated without `.destroy()`.** Old canvases keep their
  listeners; the app degrades over a long session.
- **Logic in event handlers.** A click handler that computes anything is a
  domain function that has not been extracted yet.
- **`async` without error handling on Firebase calls.** Offline is the normal
  case for a PWA used in a gym, not an edge case.
- **Snapshot tests as the primary assertion.** They record whatever the code
  does, bug included.

## Deep modules here

A deep module is an ES module with one or two exports and real behaviour
inside. Signals of shallow: a module exporting one function per Firestore
collection and nothing else; a `utils.js` that has become a bag; a module that
forwards arguments to another module and adds nothing.

## Prototypes

A throwaway route or a scratch file under the Vite dev server, deleted
afterwards. Do not leave a prototype route in the shipped app, and do not add
a dependency to answer a design question.
