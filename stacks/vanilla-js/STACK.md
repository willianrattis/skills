# Stack profile: Vanilla JS + Firebase

Browser-native ES modules, no bundler, no transpiler. Firebase Auth and
Firestore as the backend. Read this whole file before applying a stack-coupled
skill (`tdd`, `codebase-design`, `code-review`, `implement`, `prototype`).

Do not assume this is the TypeScript profile with types removed. There is no
`package.json` build step, no `tsc`, no import map resolution beyond what the
browser does natively. Bare specifiers (`import x from "chart.js"`) do not
work; imports are relative paths or full URLs.

## The honest state of the feedback loop

**There is no test runner in this project.** That is a fact about the repo, not
an oversight to route around.

The consequence for `/tdd`: do not write test files. There is nothing to run
them with, and a suite that never executes is worse than no suite — it rots
silently and lies about coverage.

If a task genuinely calls for TDD, the first slice is *adding the runner*, not
skipping the discipline. Say so and ask. Vitest runs against browser-native ES
modules with almost no configuration and does not require adopting a bundler.
Until that decision is made, work through the loop below instead.

## The loop that does exist

| Purpose            | Command                                       |
| ------------------ | --------------------------------------------- |
| Serve the app      | `npx serve .`                                   |
| Firebase emulators | `firebase emulators:start`                      |
| Deploy rules only  | `firebase deploy --only firestore:rules`        |

The red → green cycle runs through the emulator and the browser: reproduce the
behaviour in the UI, change one thing, reload, observe. It is slower and less
durable than a runner, but it is a real loop and it observes real behaviour.

**Never point the emulator-less app at production Firestore to try something
out.** The emulator exists precisely so experiments cannot corrupt real
training history.

## Seams

- **Exported function of an ES module** — the public entry of a unit of
  behaviour. This is the primary seam.
- **The Firestore boundary** — the functions that read and write documents.
  Everything above them should be reachable without touching Firebase at all.
- **Security rules** — a seam in their own right, tested against the emulator
  with `@firebase/rules-unit-testing` if a runner is ever added. Rules are the
  only enforcement of the coach-sharing access model; UI checks are not.

Not seams: DOM structure, CSS classes, element IDs, anything reached with
`document.querySelector` from outside the module that owns it.

## The rule that keeps this codebase testable

Firebase calls belong in a thin adapter layer. Domain logic — computing
progression, grouping sets into supersets, deriving chart series — takes plain
objects in and returns plain objects out, with no `db`, no `doc()`, no
`onSnapshot` anywhere in it.

This is the single highest-value structural constraint here. It is what makes a
runner cheap to add later, and it is what lets you reason about a bug without
booting the emulator.

## Vertical slice

One slice is: behaviour visible in the UI → the handler that drives it → the
domain function → the adapter call → green in the browser.

Not: all the Firestore functions first, then the UI.

## Naming

Function and variable names come from `CONTEXT.md`. If the glossary says
"série efetiva", the code does not say `validSet`.

## Anti-patterns specific to this stack

- **Firestore documents shaped for the UI.** The array-to-object change for
  superset persistence exists because arrays lost identity across writes.
  Model for the data's own constraints, not for what renders conveniently.
- **`onSnapshot` listeners without teardown.** Every listener registered on a
  view must be unsubscribed when the view goes away, or the app leaks and
  double-renders.
- **Security enforced in the UI.** Hiding a button is not access control. Any
  change to who can see whose data is a change to the rules file first.
- **Logic in event handlers.** A click handler that computes anything is a
  domain function that has not been extracted yet.
- **Global mutable state on `window`.** Module scope is already private; use it.
- **Chart.js instances recreated without `.destroy()`.** Old canvases keep
  their listeners and the page slows down over a session.
- **`async` without error handling on Firebase calls.** Offline is the normal
  case for a PWA in a gym basement, not an edge case.

## Deep modules here

A deep module is an ES module with one or two exports and real behaviour
inside. Signals of shallow: a module that exports a function per Firestore
collection and does nothing else; a `utils.js` that has become a bag; a
"service" that only forwards arguments to another module.

## Prototypes

A single throwaway `.html` file with an inline `<script type="module">`, served
by `npx serve`. Delete it afterwards. Do not add a prototype route to the real
app and leave it there.
