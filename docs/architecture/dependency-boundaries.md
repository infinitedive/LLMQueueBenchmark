# Dependency Boundaries

This document defines allowed dependency directions and boundary constraints across repos and within each repo.

## Cross-Repo Boundary

### Allowed

- HTTP API communication.
- Documented contracts for CLI/env, metrics semantics, and artifact schemas.
- Shared documentation and planning artifacts.

### Disallowed

- Runtime Python imports between `llm-benchmark-client` and `llm-scheduler-server`.
- Hidden coupling through undocumented assumptions.

## Client Dependency Boundaries

Intended direction:
- `loadgen`, `telemetry` -> `analysis` -> `cli`

Rules:
- `analysis` consumes collection outputs; collection modules should not depend on analysis modules.
- `cli` assembles the pipeline and may depend on lower layers.
- Interface parsing belongs at boundary/orchestration layers, not deeply in analysis internals.

## Server Dependency Boundaries

Intended direction:
- `utils` -> (`engine`, `schedulers`) -> `server`

Rules:
- `server` owns HTTP boundary and runtime wiring.
- `schedulers` and `engine` are internal execution policy/mechanism layers.
- `utils` provides foundational capabilities without owning endpoint semantics.

## API Boundary Ownership

- CLI/env boundaries are owned by entrypoint and config layers.
- HTTP boundary is owned by server API layer.
- Artifact schema boundary is owned by client analysis/schema pipeline.

Internal modules may support boundary behavior but should not become de facto contract authorities.

## Change Checklist for Boundary-Sensitive Work

When modifying dependency edges or boundary ownership:
1. Confirm layering direction is preserved.
2. Update architecture docs if top-level contracts changed.
3. Update interface/data-model docs if boundary contracts changed.
4. Validate no new cross-repo runtime coupling was introduced.

## Related Docs

- `ARCHITECTURE.md`
- `docs/architecture/client.md`
- `docs/architecture/server.md`
- `docs/principles/client-server-separation.md`
