# ML Systems Portfolio Report Template

Use this template after completing the experiment matrix and generating RQ outputs.

Expected analysis outputs (from `llm_bench.analysis.mlsys_report`):
- `artifact_validation.csv`
- `rq1_mode_comparison.csv`
- `rq2_saturation_points.csv`
- `rq3_custom_vs_vllm_deltas.csv`
- `mlsys_report_summary.md`

## 1. Methodology

### 1.1 System Under Test

Describe:
- server runtime and scheduler modes evaluated (`custom+naive`, `custom+dynamic`, `vllm`);
- model/tokenizer identifiers;
- hardware profile and GPU capability details.

Use:
- `docs/operations/ml-sys-report-spec.json`
- `docs/interfaces/server-cli-env.md`
- `docs/operations/vllm-gpu-settings.md`

### 1.2 Workload and Traffic Model

Describe:
- workload mode and prompt token envelope;
- arrival mode(s): Poisson sweep and closed-loop controls;
- lambda grid, horizon/iterations, repeats, and concurrency cap.

Use:
- `docs/operations/ml-sys-report-spec.json`
- `docs/interfaces/client-cli.md`

### 1.3 Metrics and Attribution

Describe:
- client-owned metrics: TTFT, TPOT, total latency, req/s, tok/s;
- server-owned metrics: queue/batch/resource telemetry from `/metrics*`;
- aggregation approach (mean/std + p50/p95/p99 where applicable).

Use:
- `docs/principles/metrics-semantics.md`
- `docs/data-models/requestrecord.md`
- `docs/data-models/telemetry-schema.md`

### 1.4 Execution Protocol

Describe:
- server startup verification (`/health`, `/v1/models`);
- benchmark run orchestration and output locations;
- artifact completeness checks before interpretation.

Use:
- `docs/operations/local-dev.md`
- `docs/operations/benchmark-recipes.md`
- `artifact_validation.csv`

## 2. Results by Research Question

### 2.1 RQ1: Effect of batching strategy on latency and throughput

Primary evidence:
- `rq1_mode_comparison.csv`

Report:
- throughput trends across lambda;
- latency p95 behavior by mode;
- TTFT/TPOT differences under matched load.

Suggested claim guardrail:
- keep client-derived metrics separate from server-derived telemetry.

### 2.2 RQ2: Saturation load levels

Primary evidence:
- `rq2_saturation_points.csv`
- per-mode sweep summaries (`sweep_summary.csv`)

Report:
- detected saturation point per mode;
- definition used (throughput ratio + latency growth rule);
- whether saturation was not observed in grid for any mode.

### 2.3 RQ3: Custom scheduler vs vLLM under identical workloads

Primary evidence:
- `rq3_custom_vs_vllm_deltas.csv`

Report:
- per-lambda deltas for throughput and latency;
- crossover points where advantage flips;
- variance notes from repeated runs.

## 3. Threats to Validity

Include at minimum:
- **Clock domains:** client and server clocks are independent timelines.
- **Stochastic arrivals:** Poisson variability requires repeated runs and spread reporting.
- **Telemetry availability:** mode/config differences can change visible server metrics.
- **Hardware constraints:** vLLM dtype and GPU-memory settings may affect comparability.
- **External validity:** conclusions are bounded by model, workload, and hardware profile.

## 4. Reproducibility Appendix

Attach:
- exact spec file used (`ml-sys-report-spec.json`);
- runtime launch commands per mode;
- client commands (or `report_matrix` invocations);
- artifact directory tree snapshot;
- schema/version notes where relevant.
