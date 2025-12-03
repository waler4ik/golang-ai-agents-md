# Go Agent Guide

This guide instructs agents how to write Go code that mirrors community best practices. Content is drawn from Effective Go, Uber's Go Style Guide, Go Code Review Comments, the Clean Architecture essay, and the Awesome Go catalog. Supporting detail lives in:
- `docs/go/STYLE_GUIDE.md` (idioms and formatting)
- `docs/go/ARCHITECTURE.md` (layering and dependency rules)
- `docs/go/PATTERNS.md` (common solutions and resource selection)
- `docs/go/LINTING.md` (tooling and checks)

## Operating Rules
- Default to standard library and idiomatic patterns before adding dependencies; vet any new library (maintenance, license, API stability) via Awesome Go or upstream docs.
- Always format (`gofmt`/`goimports`) and lint (`go vet`, `golangci-lint`) before shipping. Run tests with `go test` and `-race` when concurrency is involved.
- Keep architecture clean: business logic does not depend on frameworks, databases, or transport types. Wire dependencies at the edges.
- Prefer clarity over cleverness; small, focused functions; early returns on errors.

## Effective Go Essentials
- **Formatting:** `gofmt` is canonical; keep imports grouped stdlib/third-party/local.
- **Comments:** Full sentences starting with the name being described; explain why, not just what.
- **Names:** Package names are short and lower case. Avoid stutter (`bytes.Buffer`). Use common initialisms (`ID`, `HTTP`).
- **Declarations:** Use `:=` for locals; let zero values work; composite literals with field names when many fields.
- **Control:** Prefer `switch` over long `if/else`; range over slices/maps. Watch range variable capture when spawning goroutines.
- **Functions:** Return multiple values instead of out parameters. Keep receiver choice intentional (pointer to mutate or avoid copies; value otherwise).
- **Interfaces:** Define small, behavior-first interfaces at the consumption site. Accept interfaces, return concrete types. Interface satisfaction is implicit.
- **Errors:** Return errors as last value; wrap with `%w` for context; avoid panics for expected problems.
- **Concurrency:** Share memory by communicating. Use channels for ownership transfer or signaling; use mutexes for shared state. Close channels from senders only. Tie goroutines to a lifecycle (context or done channel).
- **Packages:** Export only what callers need; organize code so package names express the domain; avoid cyclic imports.
- **Examples:** Prefer table-driven tests; show simple usage in Godoc examples.

## Uber Go Style Highlights
- Keep APIs minimal and stable; document breaking changes.
- Avoid globals; inject dependencies explicitly.
- Prefer explicit nil checks and short-lived variables; use `var buf bytes.Buffer` instead of pointers when zero value suffices.
- Use functional options or config structs when constructors need flexibility.
- Avoid shadowing important variables (especially `err` across scopes that matter).
- Use typed errors or sentinel errors only when callers must branch; otherwise wrap and return.
- For concurrency, manage lifecycles carefully; avoid unbounded goroutine creation; prefer worker pools or bounded semaphores for bulk work.
- Keep logging structured; avoid logging and returning the same error at multiple layers.

## Go Code Review Comments
- No dot-imports in production; avoid package stutter and needless getters/setters.
- Use `defer` immediately for resource cleanup; beware of defers in hot loops.
- Do not copy loop variables when taking addresses; use index capture.
- Keep JSON/struct tags consistent; check map presence with the two-value assignment.
- Do not store `context.Context` in structs; pass it as the first parameter where relevant.
- Tests should be deterministic, table-driven, and concise.

## Clean Architecture Checklist
- Dependency rule: outer layers depend on inner layers; never the reverse.
- Entities and use cases contain business rules; they do not import transport or DB packages.
- Interface adapters map between outer representations (HTTP/gRPC/DB) and inner DTOs/domain models.
- Frameworks and drivers (HTTP server, gRPC runtime, DB client) are plugins; replaceable without rewriting business logic.
- Package layout heuristic: `cmd/<app>` wires; `internal/<domain>` holds entities/use cases; adapters live alongside drivers; keep DTOs at boundaries. See `docs/go/ARCHITECTURE.md`.

## Common Patterns & Snippets
- **Error wrapping:**  
  ```go
  if err != nil {
      return fmt.Errorf("fetch profile %s: %w", userID, err)
  }
  ```
- **Context-aware handler skeleton:**  
  ```go
  func (s *Server) handle(w http.ResponseWriter, r *http.Request) {
      ctx := r.Context()
      req, err := decode(r)
      if err != nil {
          http.Error(w, err.Error(), http.StatusBadRequest)
          return
      }
      resp, err := s.usecase.Run(ctx, req)
      if err != nil {
          renderError(w, err)
          return
      }
      encode(w, resp)
  }
  ```
- **Worker pool:**  
  ```go
  jobs := make(chan Job)
  var wg sync.WaitGroup
  for i := 0; i < n; i++ {
      wg.Add(1)
      go func() {
          defer wg.Done()
          for j := range jobs {
              _ = process(j)
          }
      }()
  }
  // feed jobs then close(jobs); wg.Wait()
  ```
- **Table-driven test:**  
  ```go
  func TestParse(t *testing.T) {
      cases := []struct {
          name string
          in   string
          want int
          err  bool
      }{
          {"ok", "42", 42, false},
          {"bad", "x", 0, true},
      }
      for _, tc := range cases {
          t.Run(tc.name, func(t *testing.T) {
              got, err := Parse(tc.in)
              if tc.err && err == nil {
                  t.Fatalf("expected error")
              }
              if !tc.err && got != tc.want {
                  t.Fatalf("got %d want %d", got, tc.want)
              }
          })
      }
  }
  ```

## Tooling & Quality
- Always run `gofmt`/`goimports` before committing.
- Run `go vet` for static checks; add `golangci-lint run` to CI with linters tuned per repo.
- Use `go test` with `-race` for concurrent code paths.
- Keep CI fast; cache module downloads; avoid flaky tests.

## Library Discovery
- Start with the standard library. If a dependency is needed, search Awesome Go by category, then evaluate: maintenance cadence, documentation, test coverage, API clarity, license. Prefer stable, well-maintained options.

## References
- Effective Go: go.dev/doc/effective_go  
- Uber Go Style Guide: github.com/uber-go/guide  
- Go Code Review Comments: go.dev/wiki/CodeReviewComments  
- Clean Architecture: blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html  
- Awesome Go: awesome-go.com
