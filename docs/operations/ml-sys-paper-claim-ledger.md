# ML Systems Paper Claim Ledger (Poisson C1-C3)

This ledger maps quantitative performance claims in the manuscript draft to the approved
evidence set under `llm-benchmark-client/report_runs/report_outputs`.

Scope note:
- This ledger covers quantitative result claims in Abstract/Results/Interpretation.
- Experimental setup disclosures (hardware profile, runtime flags, and topology) are
  configuration context, not inferred performance claims.

## Approved Evidence Sources

- `llm-benchmark-client/report_runs/report_outputs/rq1_mode_comparison.csv`
- `llm-benchmark-client/report_runs/report_outputs/rq2_saturation_points.csv`
- `llm-benchmark-client/report_runs/report_outputs/rq3_custom_vs_vllm_deltas.csv`
- `llm-benchmark-client/report_runs/report_outputs/artifact_validation.csv`
- `llm-benchmark-client/report_runs/report_outputs/mlsys_report_summary.md`

## Quantitative Claim Mapping

| claim_id | manuscript_section | quantitative_claim (3 decimals) | source_artifact | source_locator |
|---|---|---|---|---|
| AV-001 | Methodology | Artifact validation covers 639 checks with 0 missing required artifacts. | `artifact_validation.csv` | Count all rows; count rows where `exists != True` |
| RQ1-TBL-THR | Results (RQ1) | The throughput table reproduces all 21 `throughput_req_per_s_mean` values for every (`mode_id`, `lambda_rps`) pair in scope. | `rq1_mode_comparison.csv` | Entire table, columns `mode_id`, `lambda_rps`, `throughput_req_per_s_mean` |
| RQ1-TBL-LAT | Results (RQ1) | The p95 latency table reproduces all 21 `lat_p95_mean` values for every (`mode_id`, `lambda_rps`) pair in scope. | `rq1_mode_comparison.csv` | Entire table, columns `mode_id`, `lambda_rps`, `lat_p95_mean` |
| RQ1-TAIL-001 | Results (RQ1) | Selected TTFT/TPOT values (lambda 2 and 24) are directly copied from `ttft_p95_mean` and `tpot_p95_mean`. | `rq1_mode_comparison.csv` | Rows `lambda_rps = 2.0, 24.0`; columns `ttft_p95_mean`, `tpot_p95_mean` |
| RQ1-001 | Results (RQ1) | At lambda 24 rps, throughput is 1.642 (`custom_dynamic`), 1.569 (`custom_naive`), and 20.779 (`vllm`) req/s. | `rq1_mode_comparison.csv` | Rows where `lambda_rps = 24.0` |
| RQ1-002 | Results (RQ1) | `custom_dynamic` reaches its highest mean throughput at lambda 2 rps: 2.055 req/s. | `rq1_mode_comparison.csv` | `mode_id = custom_dynamic`, max `throughput_req_per_s_mean` |
| RQ1-003 | Results (RQ1) | `custom_naive` reaches its highest mean throughput at lambda 2 rps: 2.111 req/s. | `rq1_mode_comparison.csv` | `mode_id = custom_naive`, max `throughput_req_per_s_mean` |
| RQ1-004 | Results (RQ1) | `vllm` throughput rises from 1.153 req/s (lambda 1) to 20.779 req/s (lambda 24). | `rq1_mode_comparison.csv` | `mode_id = vllm`, rows `lambda_rps = 1.0, 24.0` |
| RQ1-005 | Results (RQ1) | At lambda 24 rps, p95 latency is 1134.444 (`custom_dynamic`), 1215.306 (`custom_naive`), and 0.606 (`vllm`) s. | `rq1_mode_comparison.csv` | Rows where `lambda_rps = 24.0`, field `lat_p95_mean` |
| RQ1-006 | Results (RQ1) | For lambdas 2-24, `vllm` p95 latency stays within 0.445-0.707 s. | `rq1_mode_comparison.csv` | `mode_id = vllm`, rows `lambda_rps = 2.0..24.0`, field `lat_p95_mean` |
| RQ1-007 | Results (RQ1) | At lambda 24 rps, `custom_dynamic` exceeds `custom_naive` by 0.073 req/s throughput and lowers p95 latency by 80.862 s. | `rq1_mode_comparison.csv` | Rows `mode_id = custom_dynamic/custom_naive`, `lambda_rps = 24.0`; arithmetic differences |
| RQ1-008 | Results (RQ1) | At lambda 2 rps, p95 latency is 0.445 (`vllm`), 19.813 (`custom_dynamic`), and 22.002 (`custom_naive`) s. | `rq1_mode_comparison.csv` | Rows where `lambda_rps = 2.0`, field `lat_p95_mean` |
| RQ2-001 | Results (RQ2) | `custom_dynamic` saturation is detected at lambda 4 rps with throughput ratio 0.406 and latency-growth ratio 6.301. | `rq2_saturation_points.csv` | Row `mode_id = custom_dynamic` |
| RQ2-002 | Results (RQ2) | `custom_naive` saturation is detected at lambda 4 rps with throughput ratio 0.427 and latency-growth ratio 5.545. | `rq2_saturation_points.csv` | Row `mode_id = custom_naive` |
| RQ2-003 | Results (RQ2) | `vllm` reports `not_detected_in_grid` for saturation over the tested lambda range. | `rq2_saturation_points.csv` | Row `mode_id = vllm`, field `status` |
| RQ2-TBL-ALL | Results (RQ2) | The RQ2 table reproduces all rows and numeric fields in the saturation output. | `rq2_saturation_points.csv` | Entire table |
| RQ3-TBL-ALL | Results (RQ3) | The RQ3 delta table reproduces all rows and all four delta fields in the output file. | `rq3_custom_vs_vllm_deltas.csv` | Entire table, columns `delta_*` |
| RQ3-001 | Results (RQ3) | Throughput delta (`custom_dynamic - vllm`) is positive at lambdas 1 and 2 (+0.064, +0.023) and negative from lambda 4 onward. | `rq3_custom_vs_vllm_deltas.csv` | Field `delta_throughput_req_per_s` across all rows |
| RQ3-002 | Results (RQ3) | At lambda 24 rps, throughput delta is -19.136 req/s. | `rq3_custom_vs_vllm_deltas.csv` | Row `lambda_rps = 24.0`, field `delta_throughput_req_per_s` |
| RQ3-003 | Results (RQ3) | p95 latency delta is positive for all lambdas, from +5.205 s (lambda 1) to +1133.837 s (lambda 24). | `rq3_custom_vs_vllm_deltas.csv` | Field `delta_lat_p95_s`, rows `lambda_rps = 1.0, 24.0` |
| RQ3-004 | Results (RQ3) | p95 TTFT delta is positive for all lambdas, from +4.052 s (lambda 1) to +1129.176 s (lambda 24). | `rq3_custom_vs_vllm_deltas.csv` | Field `delta_ttft_p95_s`, rows `lambda_rps = 1.0, 24.0` |
| RQ3-005 | Results (RQ3) | p95 TPOT delta remains positive, +0.006 s (lambda 1) and approximately +0.077 s at higher lambdas. | `rq3_custom_vs_vllm_deltas.csv` | Field `delta_tpot_p95_s`, rows `lambda_rps = 1.0..24.0` |

