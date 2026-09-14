---
name: mlsys-paper-draft
overview: Draft a conference-style ML systems manuscript from the completed Poisson matrix run, using only approved structured evidence and explicit setup disclosures while separating results from interpretation.
todos:
  - id: lock-evidence-contract
    content: Create a claim ledger that maps each planned quantitative statement to one of the five approved report_outputs artifacts.
    status: completed
  - id: draft-manuscript-shell
    content: Create the conference-style manuscript file with the exact required section order and explicit Poisson-only scope.
    status: completed
  - id: populate-methodology-setup
    content: Write Methodology and Experimental Setup from the SSH-tunnel spec and provided hardware/runtime disclosures with endpoint redaction.
    status: completed
  - id: populate-rq-results
    content: Write RQ1-RQ3 results using only structured outputs at 3-decimal precision, adding clearly labeled variability notes only when artifact-sourced.
    status: completed
  - id: write-interpretation-validity-conclusion
    content: Draft Interpretation, Threats to Validity, and Conclusion with evidence-backed implications only and no external citations.
    status: completed
  - id: run-claim-audit
    content: Perform a final consistency pass for claim provenance, precision policy, phrasing policy for RQ2, and redaction compliance.
    status: completed
isProject: false
---

# ML Systems Paper Draft Plan

## Scope Lock

- Use only these authoritative claim sources: `[llm-benchmark-client/report_runs/report_outputs/rq1_mode_comparison.csv](llm-benchmark-client/report_runs/report_outputs/rq1_mode_comparison.csv)`, `[llm-benchmark-client/report_runs/report_outputs/rq2_saturation_points.csv](llm-benchmark-client/report_runs/report_outputs/rq2_saturation_points.csv)`, `[llm-benchmark-client/report_runs/report_outputs/rq3_custom_vs_vllm_deltas.csv](llm-benchmark-client/report_runs/report_outputs/rq3_custom_vs_vllm_deltas.csv)`, `[llm-benchmark-client/report_runs/report_outputs/artifact_validation.csv](llm-benchmark-client/report_runs/report_outputs/artifact_validation.csv)`, `[llm-benchmark-client/report_runs/report_outputs/mlsys_report_summary.md](llm-benchmark-client/report_runs/report_outputs/mlsys_report_summary.md)`.
- Treat `[docs/operations/ml-sys-report-spec.ssh-tunnel.json](docs/operations/ml-sys-report-spec.ssh-tunnel.json)` as the executed protocol reference, with endpoint/IP details redacted in prose.
- Limit reported experiment scope to Poisson conditions C1-C3; state that closed-loop controls were planned but out of scope for present results.

## Manuscript Authoring Approach

- Create a new manuscript draft at `docs/operations/ml-sys-paper-draft.md`, seeded from structure and guardrails in `[docs/operations/ml-sys-report-template.md](docs/operations/ml-sys-report-template.md)`.
- Fill exactly the requested section order: Abstract, Introduction, Methodology, Experimental Setup, Results (RQ1-RQ3), Interpretation, Threats to Validity, Conclusion.
- Enforce a strict boundary:
  - `Results` = factual, artifact-backed observations only.
  - `Interpretation` = evidence-backed implications only (no unmeasured mechanisms, no speculation).

## Section-by-Section Data Mapping

- **Methodology + Setup**: derive protocol/fairness settings from `[docs/operations/ml-sys-report-methodology.md](docs/operations/ml-sys-report-methodology.md)` and `[docs/operations/ml-sys-report-spec.ssh-tunnel.json](docs/operations/ml-sys-report-spec.ssh-tunnel.json)`, plus provided hardware/runtime disclosures (RunPod, RTX PRO 6000 Blackwell 96GB, Xeon 6952P 16 vCPU, RAM range, Ubuntu, driver 580.126.20, CUDA 13.0, vLLM bf16 + memory util 0.75, SSH-tunnel/public-endpoint topology).
- **Results (RQ1-RQ3)**: anchor all main claims to the three RQ CSVs; render numbers at fixed 3-decimal precision.
- **Variability notes**: include only if directly sourced from canonical artifacts and clearly labeled as spread/dispersion; keep primary claim statements anchored to structured outputs.
- **RQ2 phrasing**: use "No saturation was observed within the tested load range" for vLLM.

## Validity and Compliance Pass

- Reflect user-confirmed assumption that core validity checks pass, while still writing explicit threats and bounded generalization statements.
- Align threats language with checklist categories in `[docs/operations/ml-sys-comparison-validity-checklist.md](docs/operations/ml-sys-comparison-validity-checklist.md)` without introducing unsupported claims.
- Remove external related-work citations and keep framing self-contained.

## Final Quality Gate

- Run a claim-audit pass over the manuscript: each quantitative statement must map to one of the five approved outputs.
- Verify redaction requirements (no raw endpoint/IP leakage), precision policy (3 decimals), and section boundary discipline (Results vs Interpretation).
- Ensure terminology is precise and consistent with the benchmark docs and artifact semantics.

