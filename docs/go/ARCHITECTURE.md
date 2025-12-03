# Go Architecture Playbook

This file distills Clean Architecture principles for Go services so that agents keep codebase structure stable, testable, and framework-agnostic. All guidance is derived from Robert C. Martin's "The Clean Architecture".

## Core Idea
- Keep the business model at the center; drive dependencies from outer details toward inner policy.
- Make frameworks, databases, UIs, and external services replaceable plugins.
- Shape packages so they "scream" the domain rather than the technical stacks.

## Layer Responsibilities
- **Entities (Enterprise/business rules):** Core types and invariants that survive technology shifts. Avoid knowing about transport, storage, or UI.
- **Use Cases (Application rules):** Orchestrate interactions between entities to fulfill specific scenarios. Enforce application-specific policies and validation. Coordinate but do not implement UI/DB details.
- **Interface Adapters:** Translate between external representations and the shapes the use cases/entities expect (e.g., HTTP handlers, gRPC servers, CLI commands, DB repositories). Keep mapping logic and DTOs here.
- **Frameworks & Drivers:** Delivery and infrastructure details (HTTP server, gRPC runtime, database clients, queues, logging). These should depend on the adapters, not the other way around.

## Dependency Rule
- Source code dependencies point inward. Outer layers may depend on inner layers; inner layers never import outer packages.
- Data crosses boundaries as simple structures (DTOs) without behavior; avoid leaking framework types inward.
- Use interfaces defined in the inner layers to invert dependencies; outer layers provide implementations.

## Data Flow
1) Request enters via a driver (HTTP/gRPC/CLI).  
2) Adapter validates/parses input and calls a use case with boundary DTOs and interfaces.  
3) Use case invokes entities and calls out through interfaces for I/O.  
4) Adapter maps use case responses to transport-specific responses.  
5) Driver writes the response.

## Package Layout Heuristics (Go-Friendly)
- Keep `cmd/<app>` for wiring; it imports adapters and implementations.
- Put domain and use cases in `internal/<domain>/...` or similar to discourage external coupling.
- Keep adapter code (handlers, mappers, repository implementations) separate from pure domain/use case code.
- Isolate framework details (HTTP server setup, gRPC registration, ORMs, message broker setup) so they are easy to swap.

## Test Strategy
- Entities: pure unit tests with no fakes required.
- Use cases: test with in-memory doubles for gateways/ports; verify business rules and flows.
- Adapters: contract/translation tests; ensure correct mapping between transports and use cases.
- Drivers: minimal integration tests to prove wiring.

## Additional Principles
- Independent of UI: swap HTTP for CLI without touching business logic.
- Independent of DB: repository interfaces live inside the use case layer; specific DB code lives in outer layers.
- Independent of frameworks: frameworks are tools, not architecture; keep their types from polluting core layers.
- Plug-in ability: add or remove external systems by adding adapters that satisfy inner interfaces.

## References
- Paraphrased from: Robert C. Martin, "The Clean Architecture" (blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html).
