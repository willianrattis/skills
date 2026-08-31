---
name: use-stack
description: Set or change the active stack profile for this repo, so the stack-coupled skills know which toolchain to use. Use when starting work in a repo whose stack is unset, when the user says "use .NET", "switch to Python", "this part is TypeScript", or when a stack-coupled skill reports no active profile.
disable-model-invocation: true
---

# Use Stack

The engineering skills carry discipline that holds in any language. The
concrete toolchain — test runner, assertion library, what counts as a seam,
which anti-patterns bite — lives in a **stack profile** under `stacks/`.

This skill picks the active profile and records it.

## Available profiles

Read the directory names under `stacks/`. Each contains a `STACK.md`. Do not
assume the list; the repo may have profiles beyond the ones shipped here.

## Resolving the active stack

In order:

1. If the user named a stack in this invocation, use it.
2. Otherwise read `.agents/stack`. If it holds a valid profile name, report it
   and ask whether to keep it or switch.
3. Otherwise, look at the repo for evidence — `*.csproj` / `*.sln`,
   `pyproject.toml` / `requirements.txt`, `package.json` — and propose the one
   the evidence supports.
4. Ask the user to confirm. Never guess silently.

If the evidence points at more than one stack, say so and name what you found,
then ask which one this session's work targets. Do not pick the one with the
most files.

## Recording it

Write the chosen profile name, and nothing else, to `.agents/stack`:

```
dotnet
```

`.agents/stack` is per-repo state, not per-user preference. Commit it if the
repo has one stack. Gitignore it if contributors work in different parts of a
mixed repo.

## After setting it

Confirm in one line which profile is active and what its full verify command
is. Then continue with whatever the user was actually doing.

Do not read the profile into context here. The stack-coupled skills load it
themselves when they run; loading it now just spends tokens on a session that
may never reach the TDD loop.
