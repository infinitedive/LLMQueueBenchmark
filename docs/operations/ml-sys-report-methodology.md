# ML Systems Report Methodology (Portfolio Track)

Scope: conference-style experiment protocol for the three report research questions.

This playbook is designed for agent execution and reproducible analysis using the canonical
`llm-benchmark-client` artifact pipeline.

## Research Questions

1. How do different batching strategies affect latency and throughput under controlled workloads?
2. At what load levels do LLM inference systems reach saturation?
3. How does a custom scheduler compare to vLLM under identical workload conditions?

## Condition Matrix (Finalized)

All conditions below are run with the same model/hardware and fixed fairness controls.

| Condition ID | Runtime Mode | Scheduler | Arrival | Lambda Grid (rps) | Repeats | Horizon (s) | Max Concurrent | Purpose |
|---|---|---|---|---|---:|---:|---:|---|
| C1 | `custom` | `naive` | `poisson` | `1 2 4 8 12 16 24` | 5 | 90 | 64 | RQ1, RQ2, RQ3 baseline |
| C2 | `custom` | `dynamic` | `poisson` | `1 2 4 8 12 16 24` | 5 | 90 | 64 | RQ1, RQ2, RQ3 custom adaptive |
| C3 | `vllm` | n/a | `poisson` | `1 2 4 8 12 16 24` | 5 | 90 | 64 | RQ1, RQ2, RQ3 parity target |
| C4 | `custom` | `dynamic` | `closed` | n/a | 3 | n/a | 64 | Deterministic variance check |
| C5 | `vllm` | n/a | `closed` | n/a | 3 | n/a | 64 | Deterministic parity check |

Notes:
- Poisson conditions are the primary source for capacity/saturation analysis.
- Closed-loop conditions are control runs used to check stability and non-arrival variance.

## Fairness Controls (Locked)

- Model and tokenizer: identical IDs across all conditions.
- Prompt set: same workload mode, seed, prompt count, prompt token window.
- Request shape: same `--max-tokens`.
- Run windows: same `--warmup-s`, `--cooldown-s`, and `--window-s`.
- Load envelope: same `--max-concurrent`.
- Telemetry cadence: same `--metrics-interval`.
- Runtime host/port topology: fixed API and metrics URLs.
- vLLM profile disclosure: exact dtype + memory utilization flags are recorded and reported.

The machine-readable fairness lock is stored in:
- `docs/operations/ml-sys-report-spec.json`

## Agent Execution Workflow

### Agent A: Experiment Runner

Tasks:
- Execute all matrix conditions in `docs/operations/ml-sys-report-spec.json`.
- Write sweep artifacts into a mode-specific directory:
  - `report_runs/custom_naive/`
  - `report_runs/custom_dynamic/`
  - `report_runs/vllm/`
- For closed-loop controls, write runs under:
  - `report_runs/controls/custom_dynamic_closed/`
  - `report_runs/controls/vllm_closed/`

### Agent B: Artifact Validator

Tasks:
- Validate required sweep outputs:
  - `sweep_raw.csv`
  - `sweep_summary.csv`
  - `capacity_curve.png`
- Validate per-lambda subdirectories exist for all repeats.
- Validate run-level required files in each subdirectory:
  - `requests.jsonl`, `requests.csv`, `summary.json`, `summary.md`,
  - `throughput_latency_timeseries.csv`, `per_bin_summary.csv`.

### Agent C: RQ Analytics Generator

Tasks:
- Build RQ-specific tables and derived conclusions from canonical sweep artifacts.
- Keep metric ownership explicit:
  - client-owned: TTFT, TPOT, total latency, req/s, tok/s.
  - server-owned: queue/batch/GPU telemetry.

### Agent D: Interpretation Writer

Tasks:
- Generate report-ready interpretation text and threats-to-validity notes.
- Include config-profile disclosures (runtime, scheduler, vLLM launch args, hardware constraints).

## Canonical Commands

### Server startup examples

- `custom + naive`: see `docs/operations/local-dev.md`
- `custom + dynamic`: see `docs/operations/local-dev.md`
- `vllm`: see `docs/operations/vllm-gpu-settings.md`

For report reproducibility, use an explicit 3-tier dynamic profile for `custom_dynamic`:
- `--dynamic-bs-min 2 --dynamic-bs-mid 16 --dynamic-bs-max 32`
- `--dynamic-q1 2 --dynamic-q2 16`
- `--dynamic-timeout-min-ms 2 --dynamic-timeout-mid-ms 12 --dynamic-timeout-max-ms 30`
- `--dynamic-max-wait-ms 150`

### Matrix command planner/runner

From `llm-benchmark-client`:

```bash
python -m llm_bench.cli.report_matrix \
  --spec ../docs/operations/ml-sys-report-spec.json \
  --phase all \
  --dry-run
```

Remove `--dry-run` to execute client benchmark commands for the selected phase/modes.

### Client sweep template (Poisson)

From `llm-benchmark-client`:

```bash
python -m llm_bench \
  --api-url http://127.0.0.1:8000/v1 \
  --metrics-url http://127.0.0.1:8000/metrics \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --workload demo \
  --arrival poisson \
  --horizon-s 90 \
  --max-concurrent 64 \
  --max-tokens 128 \
  --seed 42 \
  --warmup-s 10 \
  --cooldown-s 10 \
  --window-s 1.0 \
  --sweep \
  --sweep-lambdas 1 2 4 8 12 16 24 \
  --sweep-repeats 5 \
  --outdir ../report_runs/custom_dynamic
```

### RQ analysis generator

From `llm-benchmark-client`:

```bash
python -m llm_bench.analysis.mlsys_report \
  --mode custom_naive=../report_runs/custom_naive \
  --mode custom_dynamic=../report_runs/custom_dynamic \
  --mode vllm=../report_runs/vllm \
  --outdir ../report_runs/report_outputs
```

## Threats to Validity Checklist

- Clock-domain mismatch between client and server timelines.
- Stochastic variance for Poisson arrival process (handled by repeats + spread reporting).
- Differences in telemetry availability by runtime mode.
- GPU capability constraints affecting vLLM dtype/memory settings.
- External validity limited to selected model/workload/hardware profile.

## Pre-Publication Validity Gate

Before accepting any RQ result as portfolio evidence, run:
- `docs/operations/ml-sys-comparison-validity-checklist.md`

Treat the checklist outcome as a hard gate for figures, tables, and comparative claims.
