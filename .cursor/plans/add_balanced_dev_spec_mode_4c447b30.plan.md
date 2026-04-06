---
name: Add Balanced Dev Spec Mode
overview: Introduce a dedicated balanced dev benchmark profile via a separate spec file and docs updates, without changing report_matrix CLI behavior.
todos:
  - id: add-dev-spec
    content: Create `docs/operations/ml-sys-report-spec.dev.json` with balanced dev cadence values and schema-parity with prod spec.
    status: completed
  - id: document-dev-mode
    content: Update methodology and recipe docs with explicit dev spec usage commands and guidance.
    status: completed
  - id: index-dev-spec
    content: Add dev spec reference in docs index operations section.
    status: completed
  - id: validate-dev-flow
    content: Dry-run report_matrix with dev spec and verify generated command profile + downstream analysis compatibility.
    status: completed
isProject: false
---

# Add Balanced Dev Spec Mode

## Goal

Create a faster iteration path for local development by adding a separate spec config that reduces sweep breadth and runtime while preserving methodology compatibility.

## Implementation

- Add a new dev spec file at [docs/operations/ml-sys-report-spec.dev.json](/home/garuc/projects/LLMQueueBenchmark/docs/operations/ml-sys-report-spec.dev.json) derived from [docs/operations/ml-sys-report-spec.json](/home/garuc/projects/LLMQueueBenchmark/docs/operations/ml-sys-report-spec.json).
- Keep production spec unchanged; dev mode is selected by passing `--spec` explicitly to [llm-benchmark-client/src/llm_bench/cli/report_matrix.py](/home/garuc/projects/LLMQueueBenchmark/llm-benchmark-client/src/llm_bench/cli/report_matrix.py).
- Apply balanced dev profile values in dev spec:
  - Poisson lambdas: `[1, 2, 4, 8]`
  - Poisson repeats: `2`
  - Poisson horizon: `30`
  - Closed-loop repeats: `1`
  - Keep fairness control structure and key lock fields aligned with prod spec for comparability.

## Docs and Operator Flow

- Update [docs/operations/ml-sys-report-methodology.md](/home/garuc/projects/LLMQueueBenchmark/docs/operations/ml-sys-report-methodology.md) with a “Dev Mode” section showing:
  - when to use dev spec vs prod spec,
  - exact `report_matrix` commands for dry-run and execution.
- Update [docs/operations/benchmark-recipes.md](/home/garuc/projects/LLMQueueBenchmark/docs/operations/benchmark-recipes.md) to add a short “dev cadence” recipe that points to the new dev spec.
- Update [docs/index.md](/home/garuc/projects/LLMQueueBenchmark/docs/index.md) to include the dev spec path in operations references.

## Validation

- Run `report_matrix --dry-run` against the new dev spec and confirm emitted commands reflect reduced lambdas/repeats/horizon.
- Run `mlsys_report` on dev outputs (same command pattern) to confirm artifact validation + RQ CSV generation still works with smaller grids.

## Notes

- No changes to CLI interface or mode-selection logic are required.
- This keeps developer iteration fast while preserving the production experiment contract in the original spec.

