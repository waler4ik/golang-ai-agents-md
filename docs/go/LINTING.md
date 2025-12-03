# Go Linting & Quality Checklist

Use this checklist to keep Go codebases healthy. Content is drawn from Effective Go, Go Code Review Comments, and common Go tooling docs.

## Baseline Tools
- `gofmt`: Mandatory formatting; run before committing.
- `goimports`: Formats and prunes/organizes imports.
- `go vet`: Static analysis for suspicious constructs (format string mistakes, unreachable code, copylocks, etc.).
- `golangci-lint`: Aggregates linters (staticcheck, ineffassign, errcheck, gosimple, revive, govet). Configure per repo to balance signal vs noise.
- `go test -race`: Detects data races; run on packages with concurrency.

## Common Lint Rules
- Unused: remove unused variables/imports; do not keep dead code.
- Errors: check returned errors; wrap with `%w`; avoid swallowing errors. Do not use panics for normal errors.
- Naming: avoid stutter; keep exported names meaningful; follow standard initialisms (`ID`, `HTTP`).
- Imports: avoid blank imports unless for side effects (document why); avoid dot-imports in production code.
- Concurrency: guard shared maps; close channels only from senders; avoid goroutine leaks (tie to context).
- Defer: close resources immediately after open (`defer f.Close()`); beware of defers in loops for large iterations.
- Range pitfalls: avoid taking the address of loop variables; use index-based capture.
- Struct tags: keep JSON/DB tags consistent; avoid unused tags; ensure backticks are balanced.
- Composite literals: use field names for structs when many fields or exported fields are involved.
- Context: place `context.Context` as the first parameter when meaningful; do not store contexts inside structs.

## Suggested CI Gates
- `go fmt ./...`
- `go vet ./...`
- `golangci-lint run ./...` (or tailored config)
- `go test ./...` (add `-race` for critical paths)

## References
- Paraphrased from: Effective Go (go.dev/doc/effective_go), Go Code Review Comments (go.dev/wiki/CodeReviewComments), and common Go tooling practices.
