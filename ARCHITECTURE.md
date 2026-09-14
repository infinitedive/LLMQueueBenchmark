# LLMQueueBenchmark Architecture

This document records intended domains and package layering across the two component repositories. Consult it for boundary decisions; ordinary edits do not require a full architecture review.

The component repositories are cloned inside this parent KB workspace and retain independent Git histories. For the current investigation and implementation discrepancies, see [current state](docs/current-state.md).

## System Context

The system is split into two repos with strict responsibility boundaries:

- `llm-benchmark-client` (`llm_bench`): generates benchmark traffic, records client-side timings, scrapes server telemetry, and produces analysis artifacts.
- `llm-scheduler-server` (`batching_scheduler`): serves OpenAI-compatible inference endpoints, performs batching/scheduling/inference, and exports server-side metrics.

Primary integration points:
- HTTP API (`/v1/completions`, `/v1/models`, `/health`, `/metrics*`)
- CLI/env configuration
- Artifact contracts produced by the client

## Scope and Non-Goals

This document captures only architecture-level contracts:
- domain ownership,
- package layering,
- boundary contracts,
- high-level runtime flows.

Detailed metric formulas, endpoint payload details, and artifact field schemas intentionally live in topic docs under `docs/` to avoid duplicated sources of truth.

## Domain Map

### Client Domains (`llm-benchmark-client`)

- **Workload Domain** (API Boundary: no)
  - Prompt sourcing and prompt construction.
  - Representative package: `llm_bench.loadgen.workload`.
- **Arrival Domain** (API Boundary: no)
  - Closed-loop and Poisson request timing.
  - Representative package: `llm_bench.loadgen.arrival_patterns`.
- **Request Execution Domain** (API Boundary: yes, outbound HTTP)
  - Async HTTP and streaming response handling.
  - Representative package: `llm_bench.loadgen.client`.
- **Telemetry Domain** (API Boundary: yes, outbound metrics scraping)
  - Polling and parsing server metrics endpoints.
  - Representative package: `llm_bench.telemetry.poller`.
- **Analysis and Artifact Domain** (API Boundary: yes, artifact files)
  - Aggregations, plots, reports, and schema-constrained outputs.
  - Representative packages: `llm_bench.analysis.*`.
- **Orchestration Domain** (API Boundary: yes, CLI)
  - CLI parsing, run setup, single-run/sweep execution.
  - Representative packages: `llm_bench.cli.*`.

### Server Domains (`llm-scheduler-server`)

- **API Domain** (API Boundary: yes, inbound HTTP)
  - FastAPI routes, request/response contract, endpoint wiring.
  - Representative package: `batching_scheduler.server.main`.
- **Request Lifecycle Domain** (API Boundary: no)
  - Request queueing state and stream event lifecycle.
  - Representative package: `batching_scheduler.server.request`.
- **Scheduling Domain** (API Boundary: no)
  - Batching policy implementations and selection.
  - Representative package: `batching_scheduler.schedulers.*`.
- **Inference Engine Domain** (API Boundary: no)
  - Model loading and batch inference execution.
  - Representative package: `batching_scheduler.engine.*`.
- **vLLM Integration Domain** (API Boundary: yes, external engine protocol)
  - Spawn management and request forwarding.
  - Representative modules: `batching_scheduler.server.vllm_*`.
- **Observability Domain** (API Boundary: yes, metrics exports)
  - Metrics definitions, exporters, history buffers, GPU monitors.
  - Representative package: `batching_scheduler.utils.*`.

## Package Layering

### Client Layering (`llm_bench`)

Intended dependency direction:

1. `loadgen` and `telemetry` (data collection)
2. `analysis` (post-processing and summarization)
3. `cli` (top-level orchestration)

Guidance:
- `analysis` consumes request/telemetry outputs but should not drive request execution.
- `loadgen` should not depend on `analysis`.
- `cli` is the assembly layer and can compose lower layers.

### Server Layering (`batching_scheduler`)

Intended dependency direction:

1. `utils` (shared low-level config, logging, metrics primitives)
2. `engine` and `schedulers` (core processing policies and inference)
3. `server` (HTTP/API boundary, mode selection, runtime wiring)

Guidance:
- `server` composes and orchestrates `engine`, `schedulers`, and `utils`.
- `schedulers` define policy; they do not own HTTP behavior.
- `engine` owns model execution details; API contracts stay in `server`.

## Architecture Invariants (Non-Redundant, Top-Level)

These invariants are intentionally minimal and should remain stable even as detailed docs expand:

1. **Separation invariant**: Client and server remain independent runtime systems; integration happens only through documented interfaces.
2. **Boundary ownership invariant**: External contracts are owned at boundaries (CLI/env, HTTP, artifact schema) and not in internal modules.
3. **Layering invariant**: Dependency direction must follow declared layer order within each repo.
4. **Derivation invariant**: Derived analytics/reports are computed from recorded inputs and documented transforms, not undocumented side channels.
5. **Single-source invariant**: Detailed semantics live in specialized docs under `docs/`; this file does not duplicate low-level definitions.

## Cross-Repo Boundary Contract

Allowed cross-repo coupling:
- Network protocol (OpenAI-compatible HTTP endpoints).
- Shared documented semantics (metrics names/units, artifact fields).
- Operational conventions captured in docs.

Disallowed coupling:
- Importing runtime Python modules across repos.
- Hidden assumptions not captured in interface or data-model docs.

## Cross-Cutting Concerns

- **Observability**: Metrics are first-class interfaces and must remain scrapeable and versioned by contract.
- **Error handling**: Boundary failures should surface as explicit API or artifact-level signals, not silent drops.
- **Reproducibility**: Benchmark runs must remain reproducible from declared config and recorded outputs.
- **Performance discipline**: Telemetry and instrumentation should preserve benchmark validity (measure without materially distorting load behavior).

## Top-Level Runtime Flows

### Benchmark Execution Flow

1. Client CLI resolves run config and workload.
2. Arrival pattern emits request schedule.
3. Client sends HTTP requests and records client-side timestamps.
4. Server handles request via scheduler/inference path and returns stream or final response.
5. Client aggregates request records into reports and artifacts.

### Telemetry Flow

1. Server updates in-process metrics during request handling.
2. Server exposes metrics via `/metrics` and related export endpoints.
3. Client poller scrapes telemetry and aligns it with benchmark windows.
4. Client writes telemetry time series and summary outputs.

## Architecture-Level Testing Boundaries

- **Unit boundary tests**: enforce intra-repo layering and module responsibilities.
- **Contract tests**: validate HTTP, CLI/env, and artifact schema compatibility at boundaries.
- **Integration tests**: validate end-to-end run and telemetry flow across both repos.

Guideline: prefer testing at architectural boundaries over testing internal wiring details that are likely to change.

## Evolution

Update affected topic contracts with behavioral changes. Update this map when
domain boundaries or layering change. A documented invariant describes intent;
verify the relevant implementation before treating it as an observed fact.

## Source of Truth Index

- Navigation and change-routing: `AGENTS.md`
- Detailed index: `docs/index.md`
- Interfaces: `docs/interfaces/*`
- Data models and schemas: `docs/data-models/*`
- Boundary and metric rules: `docs/principles/*`
- Deeper architecture docs: `docs/architecture/*`
