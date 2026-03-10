# Client Architecture

Scope: Internal architecture of `llm-benchmark-client` (`llm_bench`).

## Domain Responsibilities

- `llm_bench.loadgen`: workloads, arrivals, HTTP request execution, request records.
- `llm_bench.telemetry`: metrics scraping and telemetry collection.
- `llm_bench.analysis`: aggregations, schema naming, reports, plots.
- `llm_bench.cli`: top-level orchestration for run and sweep execution.

## Layering Intent

1. Collection layer: `loadgen`, `telemetry`
2. Analysis layer: `analysis`
3. Orchestration layer: `cli`

Expected direction is upward (collection -> analysis -> orchestration), avoiding reverse dependency from collection modules into analysis/report modules.

## Key Runtime Components

- Workload loader determines prompt sets and selection behavior.
- Arrival generators schedule requests under deterministic or stochastic patterns.
- Async client execution handles JSON and streaming completion paths.
- Request record persistence anchors downstream analysis.
- Telemetry poller collects server metrics in parallel with request load.
- Analysis pipeline writes tabular, summary, and plot artifacts.

## Extension Points

- Add or modify workload modes in workload loaders and CLI selection.
- Add arrival policies in arrival module and orchestration selection.
- Add computed metrics/plots via analysis modules and reporting wiring.
- Add artifact files by extending schema definitions and writer orchestration.

## Architectural Constraints

- Client does not implement server batching/scheduler logic.
- Client metrics remain client-observed; server internals are consumed via telemetry interfaces.
- Artifact generation remains client-owned.

## Related Docs

- `docs/interfaces/client-cli.md`
- `docs/data-models/requestrecord.md`
- `docs/principles/client-server-separation.md`
- `docs/architecture/end-to-end-flow.md`
