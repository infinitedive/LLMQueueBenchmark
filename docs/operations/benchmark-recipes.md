# Benchmark Recipes

Scope: Canonical usage patterns for benchmark execution and artifact generation.

## Recipe 1: Basic Single Run

Goal: produce one run with request-level and summary artifacts.

Pattern:
- Start server.
- Run client with explicit API URL, model, workload, and output directory.
- Validate `requests.*`, `summary.*`, and plot outputs.

## Recipe 2: Poisson Capacity Probe

Goal: observe latency/throughput behavior under open-loop pressure.

Pattern:
- Use `--arrival poisson` with explicit `--lambda-rps` and `--horizon-s`.
- Configure `--max-concurrent` high enough for target pressure profile.
- Inspect throughput-latency time-series and percentiles.

## Recipe 3: Closed-Loop Deterministic Pass

Goal: collect deterministic iteration-based measurements.

Pattern:
- Use `--arrival closed` and `--iterations`.
- Keep prompt set fixed for repeatability.
- Compare runs for regression checks.

## Recipe 4: Multi-Lambda Sweep

Goal: generate aggregate performance envelope.

Pattern:
- Enable `--sweep` and configure lambda range/set and repetitions.
- Validate `sweep_raw.csv`, `sweep_summary.csv`, and `capacity_curve.png`.
- Drill into per-lambda subdirectories for run-level diagnostics.

## Recipe 5: Telemetry-Correlated Run

Goal: combine client and server observability outputs.

Pattern:
- Provide metrics endpoint URL and ensure `/metrics*` endpoints are available.
- Run benchmark with telemetry polling enabled.
- Inspect `server_metrics_timeseries.csv` and summary metadata.

## Output Validation Checklist

- Artifact directory structure matches `docs/data-models/run-artifacts.md`.
- Request-level records have expected canonical fields.
- Summary files include latency and throughput dimensions.
- Telemetry outputs are present when metrics scraping is enabled.

## Related Docs

- `docs/interfaces/client-cli.md`
- `docs/interfaces/http-api.md`
- `docs/principles/artifact-contract.md`
