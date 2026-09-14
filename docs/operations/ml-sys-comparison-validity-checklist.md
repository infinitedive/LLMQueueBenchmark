# ML Systems Comparison Validity Checklist

Use this checklist before accepting any cross-mode result (tables, plots, or claims) for:
- RQ1 (batching strategy effects),
- RQ2 (saturation),
- RQ3 (custom scheduler vs vLLM parity).

If any critical check fails, mark the comparison as invalid or conditional.

## Execution Evidence (Critical for the Claim Being Made)

- [ ] Record exact client/server commits, local changes, dependency versions, and executed configuration.
- [ ] Verify request flags reach the intended runtime path (including streaming).
- [ ] For batching claims, observe actual model tensor batch size and compare it with scheduler release size.
- [ ] Verify effective prompt truncation, token limits, decoding settings, and token-count semantics across conditions.
- [ ] Isolate scheduler policy using a common executor, or label the comparison as a whole-system comparison.
- [ ] Link these checks to tests, traces, or run records; a checklist declaration alone is insufficient.

## A. Must-Be-Fixed Controls (Critical)

- [ ] **Model parity:** identical model identifier and revision across compared runs.
- [ ] **Tokenizer parity:** identical tokenizer identifier and revision.
- [ ] **Workload parity:** same workload mode and prompt-source semantics.
- [ ] **Prompt population parity:** same seed, `num_prompts`, and token-window constraints.
- [ ] **Request shape parity:** same `max_tokens`.
- [ ] **Arrival protocol parity:** same arrival mode and matching run control (`horizon_s` or `iterations`).
- [ ] **Load envelope parity:** same `max_concurrent`.
- [ ] **Sweep grid parity:** same lambda set and repeat count for compared modes.
- [ ] **Windowing parity:** same `warmup_s`, `cooldown_s`, and `window_s`.
- [ ] **Telemetry cadence parity:** same metrics endpoint and scrape interval.
- [ ] **Hardware parity:** same GPU/node class and no mixed machine class in one comparison set.
- [ ] **vLLM profile disclosure:** dtype, memory utilization, and launch args are recorded.

## B. Artifact Integrity Gates (Critical)

- [ ] Sweep-level required files exist:
  - `sweep_raw.csv`
  - `sweep_summary.csv`
  - `capacity_curve.png`
- [ ] Per-lambda/per-repeat run directories exist for all expected combinations.
- [ ] Run-level required files exist in each run directory:
  - `requests.jsonl`, `requests.csv`,
  - `summary.json`, `summary.md`,
  - `throughput_latency_timeseries.csv`, `per_bin_summary.csv`.
- [ ] No missing artifacts in `artifact_validation.csv` for runs used in final claims.

## C. Metric Semantics and Attribution

- [ ] Client-derived metrics (TTFT, TPOT, total latency, req/s, tok/s) are sourced only from client artifacts.
- [ ] Server-internal metrics (queue/batch/GPU) are sourced only from `/metrics*` telemetry artifacts.
- [ ] No claim interprets client and server timestamps as a single synchronized clock.
- [ ] Percentile labels clearly indicate source domain (client vs server).

## D. Silent Invalidators (Blockers unless addressed)

- [ ] Same model name but different underlying revision/weights.
- [ ] Seed drift or prompt-file ordering drift between modes.
- [ ] Different vLLM launch args between compared runs without explicit conditioning.
- [ ] Warmup/cooldown/window mismatch across modes.
- [ ] Client-side bottleneck (not server saturation) due to low concurrency cap.
- [ ] Inconsistent retry/error handling affecting completion counts.
- [ ] Missing or partially available telemetry in one mode treated as equivalent to present telemetry.
- [ ] Significant background load or thermal throttling during only one mode's runs.

## E. RQ-Specific Acceptance Checks

### RQ1 (Batching Strategy)
- [ ] Compared modes share identical lambda points.
- [ ] Throughput and latency comparisons are reported with repeat dispersion (std/IQR).
- [ ] Claims distinguish central tendency from tail behavior.

### RQ2 (Saturation)
- [ ] Saturation rule parameters are explicitly stated (throughput-ratio and latency-growth thresholds).
- [ ] Lambda grid is dense enough around the knee, or limitations are declared.
- [ ] “Not detected in grid” is treated as inconclusive, not “no saturation.”

### RQ3 (Custom vs vLLM)
- [ ] Deltas are computed on matched lambda points only.
- [ ] Configuration parity and vLLM profile are documented with each comparison.
- [ ] Operational constraints (startup/profile limits) are separated from pure performance claims.

## F. Decision Outcome

- **Accept comparison:** all critical checks pass, no unaddressed blockers.
- **Accept with caveats:** minor non-critical issues, explicitly documented in report.
- **Reject comparison:** any critical gate fails or blockers remain unresolved.

## Canonical References

- `docs/operations/ml-sys-report-methodology.md`
- `docs/operations/ml-sys-report-spec.json`
- `docs/principles/metrics-semantics.md`
- `docs/principles/artifact-contract.md`
- `docs/data-models/run-artifacts.md`
- `docs/interfaces/http-api.md`
