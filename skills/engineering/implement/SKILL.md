---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Before applying anything below, load the active stack profile at
`stacks/<stack>/STACK.md`, where `<stack>` is the value in `.agents/stack`. If
that file is absent or empty, ask the user which stack this work targets and
write the answer there before continuing. This skill supplies the discipline;
the profile supplies the toolchain.

Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the current branch.
