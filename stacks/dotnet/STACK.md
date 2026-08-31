# Stack profile: .NET / C#

The skills supply the discipline. This file supplies the concrete tooling for
that discipline in a .NET codebase. Read it whole before applying a
stack-coupled skill (`tdd`, `codebase-design`, `code-review`, `implement`,
`prototype`).

## Commands

| Purpose        | Command                                                        |
| -------------- | -------------------------------------------------------------- |
| Run all tests  | `dotnet test`                                                    |
| Run one test   | `dotnet test --filter "FullyQualifiedName~<Name>"`                |
| Build (strict) | `dotnet build -warnaserror`                                      |
| Format check   | `dotnet format --verify-no-changes`                              |
| Full verify    | `dotnet format --verify-no-changes && dotnet build -warnaserror && dotnet test` |

Run the full verify before declaring a slice green.

## Test stack

- Framework: xUnit
- Assertions: FluentAssertions
- Test doubles: NSubstitute
- Integration host: `WebApplicationFactory<Program>`
- Real dependencies: Testcontainers (SQL Server, Redis, Kafka) over in-memory fakes

## Seams

A seam in this stack is normally one of:

- **HTTP endpoint** — driven through `WebApplicationFactory`, asserted on status
  code and response body. This is the default seam for a feature slice.
- **Application service / handler** — the public entry point of a use case,
  resolved from the DI container rather than constructed by hand.
- **Published message** — an event landing on the broker, asserted by consuming
  it, never by inspecting the producer's internals.

Not seams: controllers instantiated directly, private methods, repository
internals, `DbContext`.

## Vertical slice

One slice is: failing endpoint test → route → handler → repository → green.
Not: all handlers, then all repositories.

## Naming

Test method names read as specifications, in the domain language of
`CONTEXT.md`:

```
public async Task Customer_with_unverified_email_cannot_complete_checkout()
```

Not `TestCheckout1`, not `Should_Return_400`.

## Anti-patterns specific to this stack

- **Mocking `DbContext` or `IQueryable`.** Use a Testcontainer. A mocked
  `DbContext` tests LINQ-to-Objects, which is not what production runs.
- **Instantiating a controller with mocked services.** Skips model binding,
  filters, auth and serialisation — the parts most likely to break.
- **Asserting on `IActionResult` type** instead of the HTTP response.
- **`Thread.Sleep` in async tests.** Use polling with a timeout, or the
  broker's own wait primitive.
- **One test project per assembly by reflex.** Group by seam, not by DLL.

## Deep modules in C#

A deep module here is an interface with few members and a lot of behaviour
behind it, placed at an assembly or namespace boundary. Signals of a shallow
module: an interface that mirrors its single implementation method for method,
a service class that only forwards to a repository, a DTO layer that exists
only to be mapped one-to-one.

## Prototypes

Throwaway prototypes go to a minimal API in a single `Program.cs`, or a LINQPad
script. Do not scaffold a full solution to answer a design question.
