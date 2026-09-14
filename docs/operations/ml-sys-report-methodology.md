# ML Systems Report Methodology (Portfolio Track)

Scope: conference-style experiment protocol for the three report research questions.

This playbook is designed for agent execution and reproducible analysis using the canonical
`llm-benchmark-client` artifact pipeline.

## Research Questions

1. How do different batching strategies affect latency and throughput under controlled workloads?
2. At what load levels do LLM inference systems reach saturation?
3. How does a custom scheduler compare to vLLM under identical workload conditions?

## Recorded Condition Matrix

The protocol intends the controls below to be held fixed. Their presence in a
specification does not establish that each runtime implemented them identically.
See [current state](../current-state.md) for the open execution-validity review.
The existing matrix and numerical settings are retained for reproducibility.

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
- `docs/operations/ml-sys-report-spec.dev.json` (balanced dev cadence)
- `docs/operations/ml-sys-report-spec.remote.json` (remote-host matrix target)
- `docs/operations/ml-sys-report-spec.ssh-tunnel.json` (SSH tunnel target via localhost)

## Research Outcome and Completion

Determine which claims the comparison supports. Establish what each condition
actually executes, distinguish measured outcomes from causal explanations, and
identify confounds relevant to the research question.

Choose the work decomposition and order based on the uncertainty. The previous
runner/validator/analytics/writer roles are not required. A full matrix is
appropriate only when it can answer the current question.

Capture repository and execution provenance using the
[experiment state record](../data-models/experiment-state.md). The template is
supplemental metadata to populate at execution time, not an automatic capture tool.

A completed comparison includes:
- Code commits for both components, any local changes, dependency versions,
  model/tokenizer revisions, hardware, runtime flags, and the executed protocol.
- Evidence that the intended variable reaches execution. For batching, distinguish
  scheduler release size from the tensor batch dimension at model invocation.
- Canonical run artifacts and RQ summaries, with client/server metric ownership
  explicit. File presence establishes completeness, not experimental validity.
- A bounded conclusion, relevant limitations, and unresolved questions. If the
  available evidence cannot settle the question, specify the smallest useful
  follow-up experiment and record what prevented its execution.

Preserve established output contracts: mode-specific runs, including controls,
feed the canonical analysis pipeline described below. Keep historical outputs
separate from new experiments.

For policy-only claims, hold the executor fixed. A custom-runtime versus vLLM
comparison measures whole serving systems unless execution differences are
controlled or explicitly included in the claim.

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

### Dev Mode (Balanced Iteration)

Use dev mode when you need faster iteration on orchestration and analysis wiring.

Dev profile differences vs production:
- Poisson horizon: `30` seconds
- Poisson lambdas: `1 2 4 8`
- Poisson repeats: `2`
- Closed-loop repeats: `1`
- Outputs are written under `report_runs_dev/*`

Dry-run dev commands:

```bash
python -m llm_bench.cli.report_matrix \
  --spec ../docs/operations/ml-sys-report-spec.dev.json \
  --phase all \
  --dry-run
```

Execute dev Poisson matrix:

```bash
python -m llm_bench.cli.report_matrix \
  --spec ../docs/operations/ml-sys-report-spec.dev.json \
  --phase poisson
```

### Remote Server Mode

Use remote mode when the benchmark client runs on one host and the inference server is exposed on
another host.

For the current remote endpoint:
- API URL: `http://205.196.144.74:8000/v1`
- Metrics URL: `http://205.196.144.74:8000/metrics`
- Spec file: `docs/operations/ml-sys-report-spec.remote.json`

Dry-run remote matrix:

```bash
python -m llm_bench.cli.report_matrix \
  --spec ../docs/operations/ml-sys-report-spec.remote.json \
  --phase all \
  --dry-run
```

Execute remote matrix:

```bash
python -m llm_bench.cli.report_matrix \
  --spec ../docs/operations/ml-sys-report-spec.remote.json \
  --phase all
```

### SSH Tunnel Mode (WSL + Windows key path)

Use tunnel mode when SSH is reachable but API/metrics ports are not directly reachable.

Start tunnel (keep this terminal open):

```bash
ssh -i /mnt/c/Users/garuc/.ssh/id_rsa \
  -o IdentitiesOnly=yes \
  -p 11296 \
  -L 8000:127.0.0.1:8000 \
  root@205.196.144.74
```

Run matrix from `llm-benchmark-client` using tunnel spec:

```bash
python -m llm_bench.cli.report_matrix \
  --spec ../docs/operations/ml-sys-report-spec.ssh-tunnel.json \
  --phase all
```

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
