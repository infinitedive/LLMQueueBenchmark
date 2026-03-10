# Artifact Contract

Scope: Rules for benchmark output production, structure, and compatibility.

## Producer of Record

- Benchmark artifacts must be produced by the canonical `llm-benchmark-client` pipeline.
- Ad-hoc scripts may be used for temporary local debugging but are not authoritative artifact producers.

## Artifact Classes

- **Run artifacts**: one benchmark execution and associated tables/plots.
- **Sweep artifacts**: aggregated multi-rate runs and capacity outputs.
- **Telemetry artifacts**: server metric series captured during runs.

Canonical file-level schemas are defined in `docs/data-models/run-artifacts.md`.

## Contract Rules

- Artifact naming and placement should remain centralized through analysis schema utilities.
- Machine-consumable outputs (`jsonl`, `csv`, `json`) must remain stable unless intentionally versioned/migrated.
- Human-readable summaries and plots should be derived from canonical data artifacts rather than hand-maintained narratives.

## Compatibility Expectations

- Additive artifact evolution is preferred.
- Breaking schema changes require:
  - explicit migration note,
  - synchronized docs and parser updates,
  - recipe updates where user workflow changes.

## Quality and Integrity

- Outputs should distinguish request-level failures from successful completions.
- Aggregates should clearly indicate whether server telemetry was present.
- Sweep summaries should maintain clear per-lambda aggregation semantics.

## Change Discipline

When artifact behavior changes, update:
- this file,
- `docs/data-models/run-artifacts.md`,
- `docs/operations/benchmark-recipes.md`,
- related reporting/analysis docs.

## Related Docs

- `docs/data-models/run-artifacts.md`
- `docs/interfaces/client-cli.md`
- `docs/principles/metrics-semantics.md`
