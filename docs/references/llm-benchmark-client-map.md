# Client Module Map (`llm-benchmark-client`)

Reference map for `llm_bench` modules, responsibilities, and common call paths.

## Entrypoints

- `llm_bench.__main__`: module entrypoint.
- `llm_bench.cli.runner`: single-run orchestration.
- `llm_bench.cli.sweep`: multi-rate sweep orchestration.

## Module Responsibilities

- `llm_bench.cli.parser`
  - CLI argument parsing and run context preparation.
- `llm_bench.cli.runner`
  - one-run execution lifecycle, artifact writing, summary/plot orchestration.
- `llm_bench.cli.sweep`
  - repeated run orchestration and aggregate sweep outputs.
- `llm_bench.loadgen.workload`
  - prompt loading/generation from synthetic/file/dataset sources.
- `llm_bench.loadgen.arrival_patterns`
  - closed-loop and Poisson arrival generation.
- `llm_bench.loadgen.client`
  - OpenAI-compatible async request sending and stream handling.
- `llm_bench.loadgen.records`
  - `RequestRecord` canonical schema model.
- `llm_bench.telemetry.poller`
  - metrics scraping and telemetry sample collection.
- `llm_bench.analysis.metrics`
  - aggregates and percentile/statistical helpers.
- `llm_bench.analysis.reports`
  - summary/report writers.
- `llm_bench.analysis.plots`
  - chart generation.
- `llm_bench.analysis.schema`
  - artifact naming/paths and output schema conventions.

## Typical Call Graph (High-Level)

- CLI -> parser -> runner/sweep
- runner -> workload + arrival + client + poller
- runner -> analysis (metrics/reports/plots/schema)
- sweep -> repeated runner invocations -> sweep summary outputs

## Notes

- This file is a reference map, not a contract definition.
- Contract semantics live in `docs/interfaces/*`, `docs/data-models/*`, and `docs/principles/*`.
