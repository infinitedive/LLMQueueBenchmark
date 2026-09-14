# Benchmark Recipes

Scope: Canonical usage patterns for benchmark execution and artifact generation.

## Recipe 0: Recruiter-Friendly Quick Sweep

Goal: provide a low-friction, copy/paste sweep that validates end-to-end benchmark execution.

Pattern:
- Start server with a minimal custom runtime configuration.
- Run client with `--sweep`, a short horizon, and a small lambda set.
- Validate `sweep_raw.csv`, `sweep_summary.csv`, and `capacity_curve.png`.
- Use this as a demonstration path before deeper tuning.

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

## Recipe 6: ML Systems Report Matrix + RQ Tables

Goal: execute the report condition matrix and produce RQ1/RQ2/RQ3 analysis tables.

Pattern:
- Follow condition matrix and fairness controls from `docs/operations/ml-sys-report-methodology.md`.
- Run one sweep per mode (`custom_naive`, `custom_dynamic`, `vllm`) using matched client settings.
- Validate expected sweep and per-run artifacts before interpretation.
- Generate report tables/summary with:
  - `python -m llm_bench.analysis.mlsys_report --mode ... --outdir ...`
- Keep metric attribution explicit (client-derived vs server-derived) in all report claims.

## Recipe 7: Dev Cadence Matrix (Balanced)

Goal: run a faster matrix loop for development and debugging before full production sweeps.

Pattern:
- Use dev spec: `docs/operations/ml-sys-report-spec.dev.json`.
- Run `report_matrix` with the dev spec to reduce horizon/lambda/repeats.
- Write outputs to `report_runs_dev/*` to avoid mixing with production artifacts.
- Validate and analyze with the same `mlsys_report` command pattern, pointing to dev output paths.

## Recipe 8: One-Command Portfolio Repro (Naive + Dynamic)

Goal: run a fully reproducible portfolio workflow in one command without vLLM.

Pattern:
- Start custom+naive server, run client sweep (`lambda = [1, 2, 4]`, repeats `2`), stop server.
- Start custom+dynamic server with fixed dynamic profile, run same sweep, stop server.
- Run `mlsys_report` in no-vLLM mode (`--allow-missing-vllm`).
- Print naive/dynamic/report output directories at the end.

Command:

```bash
python -m llm_bench.cli.portfolio_run
```

## Recipe 8: Remote Server Matrix Execution

Goal: run the report matrix from this machine against a remote inference server.

Pattern:
- Use remote spec: `docs/operations/ml-sys-report-spec.remote.json`.
- Keep fairness controls identical to production spec; only remote endpoint URLs differ.
- Plan with `--dry-run`, then execute `--phase all` for the full matrix.
- If remote metrics/API are not directly reachable, establish SSH port forwarding before client runs.

## Recipe 9: SSH Tunnel Matrix Execution

Goal: run matrix benchmarks through an SSH tunnel when only SSH is exposed.

Pattern:
- Start tunnel from WSL using Windows-hosted key path (example):
  - `ssh -i /mnt/c/Users/garuc/.ssh/id_rsa -o IdentitiesOnly=yes -p 11296 -L 8000:127.0.0.1:8000 root@205.196.144.74`
- Use tunnel spec: `docs/operations/ml-sys-report-spec.ssh-tunnel.json`.
- Run mode-specific matrix with `report_matrix --mode-id ...` or full run with `--phase all`.

## Output Validation Checklist

- Artifact directory structure matches `docs/data-models/run-artifacts.md`.
- Request-level records have expected canonical fields.
- Summary files include latency and throughput dimensions.
- Telemetry outputs are present when metrics scraping is enabled.

## Related Docs

- `docs/interfaces/client-cli.md`
- `docs/interfaces/http-api.md`
- `docs/principles/artifact-contract.md`
- `docs/operations/ml-sys-report-methodology.md`
