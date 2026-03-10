# Client CLI Contract

Scope: `llm-benchmark-client` command-line interface for run configuration and artifact production.

## Entry Points

- Installed command: `llm-bench`
- Module command: `python -m llm_bench`

Both entry points are expected to provide equivalent behavior.

## Core Arguments

- `--api-url`: Base URL for OpenAI-compatible server API.
- `--metrics-url`: URL for server metrics scraping (optional but recommended for telemetry).
- `--model`: Model identifier used in request payloads and reporting labels.
- `--arrival`: Arrival mode (`closed` or `poisson`).
- `--lambda-rps`: Poisson arrival rate (requests per second).
- `--horizon-s`: Poisson run duration.
- `--request-budget`: Optional hard cap on number of requests in open-loop mode.
- `--iterations`: Closed-loop iterations per prompt.
- `--max-concurrent`: Max in-flight requests in client load generator.
- `--max-tokens`: Target max generation length per request.
- `--warmup-s`, `--cooldown-s`: Timing windows excluded from steady-state aggregates.
- `--window-s`: Window size for throughput-latency time series.
- `--outdir`: Output directory for run artifacts.

## Workload Arguments

- `--workload` selects workload mode.
- Additional workload-specific flags configure dataset/file inputs and sampling behavior.

Workload argument semantics are owned by workload loading and orchestration code; this document captures only the interface surface.

## Sweep Arguments

- `--sweep` enables multi-rate orchestration.
- Sweep arguments control lambda ranges/sets, repetitions, and output location for aggregate artifacts.

## Behavioral Contract

- CLI configuration must deterministically map to run configuration inputs.
- Client remains responsible for:
  - request generation and dispatch,
  - client-side timestamp capture,
  - telemetry scraping (if enabled),
  - artifact generation.
- Client CLI must not introduce server-side scheduling behavior simulation.

## Compatibility Rules

- Additive arguments are preferred over breaking renames/removals.
- Breaking changes to argument names or defaults require:
  - update to this file,
  - update to `docs/operations/benchmark-recipes.md`,
  - update to any relevant report/schema docs.

## Related Docs

- `docs/interfaces/http-api.md`
- `docs/data-models/run-artifacts.md`
- `docs/principles/client-server-separation.md`
