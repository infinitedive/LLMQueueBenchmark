# Client-Server Separation Principle

This principle defines responsibility boundaries between `llm-benchmark-client` and `llm-scheduler-server`.

## Ownership Split

- **Client owns**
  - workload sourcing and prompt selection,
  - arrival generation and concurrency gating,
  - request submission and client timestamp capture,
  - telemetry scraping and offline correlation,
  - artifact/report generation.
- **Server owns**
  - request acceptance and lifecycle state,
  - scheduling and batching policy,
  - model inference execution,
  - server-side telemetry production and export.

## Allowed Coupling

- HTTP API contracts.
- CLI/env contracts per repo.
- Documented artifact and metric semantics.

## Disallowed Coupling

- Runtime cross-imports between repos.
- Client-side reimplementation of server queue/service internals as source of truth.
- Server-side generation of benchmark analysis artifacts intended to be client outputs.

## Boundary Examples

- Good: client scrapes `/metrics` and includes those values in analysis outputs.
- Good: server exposes richer telemetry endpoints while preserving API compatibility.
- Bad: client infers server queue time from client timestamps when server telemetry exists.
- Bad: server assumes specific client arrival implementation details beyond contract.

## Change Discipline

If a change affects boundary behavior, update:
- `docs/interfaces/http-api.md` and/or CLI docs,
- related data model docs,
- this principle if ownership lines changed.

## Related Docs

- `docs/interfaces/http-api.md`
- `docs/interfaces/client-cli.md`
- `docs/interfaces/server-cli-env.md`
- `ARCHITECTURE.md`
