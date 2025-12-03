# Go Style Guide for Agents

This guide merges the core practices from Effective Go, Uber's Go Style Guide, and the Go project's CodeReviewComments so agents keep output idiomatic, consistent, and review-friendly. All items are drawn from the referenced sources.

## Core Principles
- Prefer clarity and simplicity over cleverness; make the code obvious to future readers.
- Let the compiler and tools help: run `gofmt`, `goimports`, `go vet`, and linters.
- Lean on zero values, short declarations, and the standard library instead of reinventing utilities.
- Avoid premature abstractions; add interfaces or patterns only when they clarify behavior.

## Formatting & Imports
- Always format with `gofmt`; never hand-tune alignment.
- Use `goimports` to organize imports into standard library, third-party, and local groups.
- Keep files organized: package comment at top, imports next, then declarations.
- Avoid long lines when it hurts readability, but don't obsess over hard limits.
- Do not leave commented-out code or unused imports.

## Naming
- Package names: short, lower-case, no underscores; name after what the package provides. Avoid stutter (`bytes.Buffer` not `bytes.ByteBuffer`).
- Exported identifiers: use CamelCase and meaningful names; doc comments start with the identifier name.
- Local variables: short names for short scopes (`i`, `err`); longer names for clarity when needed.
- Constants: prefer well-scoped `const` blocks; use `iota` for enumerations.
- Avoid abbreviations unless common (`HTTP`, `ID`); keep consistent capitalization (`userID`).

## Comments & Documentation
- Write package, type, function, and method comments as full sentences; start with the name being described.
- Favor explaining why over what; keep comments in sync with code.
- For Godoc examples, include small compilable snippets showing typical use.

## Declarations & Initialization
- Use `:=` for short-lived locals; use `var` when zero value matters or when declaring at package scope.
- Prefer composite literals when initializing structs, slices, and maps; include field names for readability when many fields exist.
- Let zero values work: avoid unnecessary `make`/`new` unless you need capacity or pointer semantics.
- Use `const` for values that never change; avoid `#define` style macros (not needed in Go).

## Control Flow
- Return early on errors; reduce nesting. Example:
  ```go
  if err := do(); err != nil {
      return err
  }
  // happy path continues
  ```
- Use `switch` over chains of `if/else` when comparing the same value or handling type assertions.
- Prefer `for` with range for collections; remember that range copies the iteration value—take the address carefully (use index to take addresses).
- Avoid `goto` except in rare, simple cases (e.g., breaking out of nested loops).

## Functions & Methods
- Keep functions small and focused; pass only what is needed.
- When options grow, prefer functional options or small config structs with meaningful defaults.
- Avoid out parameters; return multiple values instead.
- Use pointer receivers when methods need to mutate state or avoid copying large structs; otherwise prefer value receivers.
- Do not declare methods on interfaces; declare them on concrete types and satisfy interfaces implicitly.

## Interfaces
- Define interfaces where they are consumed, not at the implementation site.
- Keep interfaces small and behavior-focused (often one to three methods).
- Accept interfaces, return concrete types; expose concrete types unless multiple implementations are expected.
- Check interface satisfaction at compile time via var assignments when helpful:
  ```go
  var _ io.Reader = (*MyType)(nil)
  ```

## Errors
- Return errors as the last return value; prefer the pattern `if err != nil { return ... }`.
- Wrap context with `fmt.Errorf("doing X: %w", err)` instead of string concatenation; expose sentinel errors via `errors.New` or typed errors when callers need to branch.
- Avoid panics for routine errors; reserve panics for programmer bugs or truly unrecoverable states.
- Do not log and return the same error at the same layer; choose one place to report.
- Expose retryable or classification signals via error wrapping or helpers (e.g., `errors.Is`, `errors.As`).

## Concurrency
- Share memory by communicating: use channels when ownership transfer fits; otherwise prefer mutexes for shared state.
- Always pair goroutines with a clear lifecycle; avoid leaks by canceling via `context.Context` or done channels.
- Use buffered channels only when it clarifies backpressure; avoid unbounded queues.
- Protect shared maps with mutexes or use `sync.Map` only for special cases (highly concurrent read-mostly).
- Prefer worker pools or bounded concurrency for bulk tasks.
- Close channels only on the sending side when signaling completion; receivers should range until closed.
- Be careful with loop variables in goroutines; capture the variable inside the loop.

## Collections & Data Structures
- Slices: be mindful of sharing underlying arrays; copy when isolating data. Use `make` with capacity when growth is known.
- Maps: check for presence with the two-value assignment; delete with `delete(m, key)`; avoid nil map writes (initialize before use).
- Structs: prefer field names over positional literals when many fields; use zero values and omit unnecessary constructors.
- Avoid embedding types that might unexpectedly expose methods or fields to external packages unless intentional.

## Package Design
- Keep packages cohesive around one idea; avoid giant utility packages.
- Avoid import cycles; extract interfaces or small packages when cycles appear.
- Minimize exported surface area; export only what callers need.
- Keep files per package organized by topic (e.g., handlers.go, repo.go) when it aids navigation.

## Logging & Observability
- Use structured logging where possible; avoid overly verbose logs in hot paths.
- Prefer leveled logging consistent across services; log with context (request IDs, user IDs) near boundaries.
- Avoid logging secrets; scrub sensitive data.
- Use metrics and tracing hooks in adapters/drivers, not deep in core business logic.

## Testing
- Write table-driven tests; use subtests for grouping.
- Keep tests deterministic; avoid sleep-based timing—use fakes or channels for synchronization.
- Use test helper functions and `t.Helper()` for shared setup.
- Name tests after the behavior under test, e.g., `TestParser_ValidInput`.
- Prefer real types over interfaces in tests; use fakes/stubs to isolate I/O.
- For concurrency, use contexts or channels to avoid flakiness; consider race detector (`go test -race`).
- Add benchmarks only when measuring hot paths; keep benchmarks stable.

## Code Review Expectations
- Ensure readability first, performance second unless in critical paths.
- Follow idioms: range loops, short variable names in short scopes, early returns, minimal stutter, minimal getters/setters.
- Avoid magic numbers and unclear defaults; document decisions that may surprise readers.
- Keep APIs unsurprising: constructors named `NewType`, close methods named `Close`, contexts as first parameter when relevant.

## Safety & Correctness Patterns
- Defer resource cleanup immediately after acquisition (e.g., `defer file.Close()`).
- Use contexts for cancelation/timeouts; honor them in loops and I/O calls.
- Guard against nil pointer dereferences by checking interface values that wrap nil concrete pointers (`var r io.Reader = (*bytes.Buffer)(nil)` is non-nil).
- Prefer immutable data where possible; copy inputs that must not be mutated by callers.

## References
- Paraphrased from: Effective Go (go.dev/doc/effective_go), Uber Go Style Guide (github.com/uber-go/guide), and Go Code Review Comments (go.dev/wiki/CodeReviewComments).
