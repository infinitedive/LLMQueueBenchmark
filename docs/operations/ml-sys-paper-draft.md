# ML Systems Benchmarking Under Controlled Poisson Load

> Status (2026-09-14): historical material under execution-validity review.
> Numeric content is preserved. Artifact provenance does not establish isolated
> scheduling effects. The inspected streaming path executes requests serially;
> its correspondence to these historical runs is unresolved. See
> [current state](../current-state.md) before reusing comparative claims.

## Abstract

This paper reports a controlled ML systems comparison across three inference modes:
`custom_naive`, `custom_dynamic`, and `vllm`. The experiment uses a fixed Poisson
arrival matrix (`lambda = 1, 2, 4, 8, 12, 16, 24` req/s), 5 repeats per load point, and
strict fairness controls. Under this setup, `custom_naive` and `custom_dynamic` reach
their highest mean throughputs at lambda 2 (2.111 and 2.055 req/s, respectively), while
`vllm` reaches 20.779 req/s at lambda 24. Tail latency diverges sharply across modes:
at lambda 24, p95 total latency is 1215.306 s (`custom_naive`), 1134.444 s
(`custom_dynamic`), and 0.606 s (`vllm`). Saturation detection marks both custom modes
at lambda 4, while `vllm` is `not_detected_in_grid`; therefore, no saturation was
observed within the tested load range for `vllm`. Matched-load deltas
(`custom_dynamic - vllm`) are throughput-positive only at lambda 1-2 (+0.064, +0.023
req/s) and become increasingly throughput-negative from lambda 4 to 24.

## Introduction

Serving-side scheduling and batching policies determine the latency-throughput operating
region of LLM inference systems. This study evaluates three research questions under one
fixed experimental contract:

1. **RQ1:** How do batching strategies affect latency and throughput under controlled
   workload conditions?
2. **RQ2:** At what offered load does each mode reach saturation?
3. **RQ3:** How does a custom scheduler compare with `vllm` under matched conditions?

The analysis follows a pipeline constraint: claims are derived only from five structured
report outputs produced by the canonical benchmark client artifact flow.

## Methodology

### Experiment Contract

- **Executed protocol source:** `docs/operations/ml-sys-report-spec.ssh-tunnel.json`
  (endpoint/IP details redacted in this document).
- **Active scope:** Poisson conditions C1-C3 (`custom_naive`, `custom_dynamic`, `vllm`).
- **Out-of-scope in this report:** closed-loop controls (planned but not included in
  present results).
- **Poisson matrix:** lambdas `1, 2, 4, 8, 12, 16, 24`; horizon `90 s`; repeats `5`.

### Fairness Controls

All compared conditions share:

- model/tokenizer: `TinyLlama/TinyLlama-1.1B-Chat-v1.0`
- workload: `demo`
- prompt controls: `num_prompts = 20`, prompt-token window `[128, 2048]`, `seed = 42`
- request/output bounds: `max_tokens = 128`
- concurrency cap: `max_concurrent = 64`
- windowing: `warmup_s = 10`, `cooldown_s = 10`, `window_s = 1.0`
- telemetry interval: `metrics_interval_s = 0.5`

### Metrics and Attribution

The report follows explicit ownership boundaries:

- **client-derived:** TTFT, TPOT, total latency, requests/s, tokens/s
- **server-derived:** queue/batch/resource telemetry from `/metrics*`

All comparative quantitative claims in this document are grounded in:

- `llm-benchmark-client/report_runs/report_outputs/rq1_mode_comparison.csv`
- `llm-benchmark-client/report_runs/report_outputs/rq2_saturation_points.csv`
- `llm-benchmark-client/report_runs/report_outputs/rq3_custom_vs_vllm_deltas.csv`
- `llm-benchmark-client/report_runs/report_outputs/artifact_validation.csv`
- `llm-benchmark-client/report_runs/report_outputs/mlsys_report_summary.md`

Artifact integrity is complete for the analyzed scope: 639 checks, 0 missing required
artifacts.

### Saturation Rule

Saturation classification uses protocol thresholds:

- minimum throughput ratio: `0.800`
- minimum latency growth ratio: `1.200`

## Experimental Setup

### Infrastructure and Topology

- provider/node type: RunPod single-node GPU instance
- GPU: RTX PRO 6000 Blackwell (96 GB VRAM)
- CPU: Intel Xeon 6952P (16 vCPU)
- RAM: approximately 157-175 GB
- OS: Ubuntu
- driver: 580.126.20
- CUDA: 13.0
- execution topology: benchmark client executed remotely from the server host; server
  exposed via redacted public endpoint/SSH tunnel path

### Runtime Modes

- `custom_naive`: custom runtime with naive scheduler
- `custom_dynamic`: custom runtime with dynamic scheduler profile
  - `dynamic_bs_min = 2`, `dynamic_bs_mid = 16`, `dynamic_bs_max = 32`
  - `dynamic_q1 = 2`, `dynamic_q2 = 16`
  - `dynamic_timeout_min_ms = 2`, `dynamic_timeout_mid_ms = 12`,
    `dynamic_timeout_max_ms = 30`
  - `dynamic_max_wait_ms = 150`
- `vllm`:
  - dtype: `bfloat16`
  - `gpu_memory_utilization = 0.75`
  - launch form: `vllm serve TinyLlama/TinyLlama-1.1B-Chat-v1.0 --gpu-memory-utilization 0.75` (host/port redacted)

Server and client command families follow canonical project recipes; this document
reports only the resulting structured artifacts.

## Results (RQ1-RQ3)

### RQ1: Batching Strategy Effects on Throughput and Latency

#### Throughput by Offered Load (req/s)

| lambda_rps | custom_dynamic | custom_naive | vllm |
|---:|---:|---:|---:|
| 1 | 1.217 | 1.277 | 1.153 |
| 2 | 2.055 | 2.111 | 2.032 |
| 4 | 1.622 | 1.708 | 3.720 |
| 8 | 1.567 | 1.478 | 7.468 |
| 12 | 1.622 | 1.518 | 10.978 |
| 16 | 1.609 | 1.554 | 14.726 |
| 24 | 1.642 | 1.569 | 20.779 |

Observed from structured outputs:

- `custom_dynamic` throughput peaks at 2.055 req/s (lambda 2).
- `custom_naive` throughput peaks at 2.111 req/s (lambda 2).
- `vllm` throughput rises from 1.153 req/s (lambda 1) to 20.779 req/s (lambda 24).

#### p95 Total Latency by Offered Load (s)

| lambda_rps | custom_dynamic | custom_naive | vllm |
|---:|---:|---:|---:|
| 1 | 10.291 | 3.741 | 5.086 |
| 2 | 19.813 | 22.002 | 0.445 |
| 4 | 124.847 | 122.001 | 0.446 |
| 8 | 353.655 | 402.743 | 0.584 |
| 12 | 534.700 | 600.323 | 0.549 |
| 16 | 760.643 | 821.272 | 0.707 |
| 24 | 1134.444 | 1215.306 | 0.606 |

Additional tail metrics at selected load points:

- At lambda 2, TTFT p95 is 18.137 (`custom_dynamic`), 19.446 (`custom_naive`), 0.163
  (`vllm`) s.
- At lambda 24, TTFT p95 is 1129.490 (`custom_dynamic`), 1210.552 (`custom_naive`),
  0.314 (`vllm`) s.
- At lambda 24, TPOT p95 is 0.083 (`custom_dynamic`), 0.088 (`custom_naive`), 0.006
  (`vllm`) s.

### RQ2: Saturation Points

| mode_id | saturation_lambda_rps | throughput_ratio_at_saturation | latency_growth_ratio_at_saturation | status |
|---|---:|---:|---:|---|
| custom_dynamic | 4.000 | 0.406 | 6.301 | detected |
| custom_naive | 4.000 | 0.427 | 5.545 | detected |
| vllm | - | - | - | not_detected_in_grid |

Under the configured saturation rule (throughput ratio <= 0.800 and latency growth
>= 1.200), both custom modes are detected at lambda 4. No saturation was observed
within the tested load range for `vllm`.

### RQ3: Custom Dynamic vs vLLM Deltas (custom_dynamic - vllm)

| lambda_rps | delta_throughput_req_per_s | delta_lat_p95_s | delta_ttft_p95_s | delta_tpot_p95_s |
|---:|---:|---:|---:|---:|
| 1 | 0.064 | 5.205 | 4.052 | 0.006 |
| 2 | 0.023 | 19.368 | 17.974 | 0.027 |
| 4 | -2.097 | 124.401 | 122.982 | 0.076 |
| 8 | -5.901 | 353.071 | 351.723 | 0.076 |
| 12 | -9.357 | 534.151 | 530.189 | 0.077 |
| 16 | -13.117 | 759.936 | 754.294 | 0.077 |
| 24 | -19.136 | 1133.837 | 1129.176 | 0.077 |

Observed from the structured delta table:

- Throughput delta is positive at lambdas 1 and 2, and negative from lambda 4 onward.
- Latency, TTFT, and TPOT deltas are positive at all tested lambdas.

## Interpretation

Within the tested workload/hardware envelope, the data implies three bounded outcomes.

First, both custom scheduling modes enter the saturation regime by lambda 4 under the
configured rule, and subsequent load increases primarily increase latency tails rather
than delivered throughput.

Second, `custom_dynamic` and `custom_naive` behave similarly on saturation onset
(both at lambda 4), but `custom_dynamic` maintains modestly better high-load behavior
than `custom_naive` at lambda 24 (higher throughput and lower p95 latency).

Third, matched-load deltas indicate that `custom_dynamic` only has a small throughput
advantage at very low load (lambda 1-2), while `vllm` dominates throughput from lambda 4
through 24 and retains much lower client-observed tail latency throughout that range.

These interpretations are evidence-backed summaries of measured outputs only; they do
not assert unmeasured internal mechanisms.

## Threats to Validity

1. **Clock-domain separation:** client and server clocks are independent; claims are
   restricted to explicitly attributed metrics.
2. **Poisson stochasticity:** arrival randomness can affect tail behavior; the matrix
   uses 5 repeats per lambda, but stochastic variance remains a factor.
3. **Telemetry asymmetry:** server telemetry availability can vary by runtime mode; this
   report anchors comparative claims to client-derived structured outputs.
4. **Scope restriction:** closed-loop controls were planned but are out of scope here,
   so this manuscript reports only Poisson C1-C3 outcomes.
5. **External validity bounds:** conclusions are limited to this model, workload,
   hardware class, and selected configuration profile.
6. **Validity-gate assumption:** this draft assumes core validity checks pass, per the
   stated reporting assumption.

## Conclusion

Under matched Poisson loads on a single-node RunPod RTX PRO 6000 Blackwell setup,
`custom_naive` and `custom_dynamic` both saturate at lambda 4, while `vllm` shows no
saturation within the tested lambda grid. `custom_dynamic` slightly improves over
`custom_naive` at high load, but `vllm` provides substantially higher throughput and
lower tail latency from lambda 4 onward in this experiment. These findings are bounded
to the tested configuration and should be interpreted as workload- and platform-specific
evidence rather than universal scheduler behavior.

