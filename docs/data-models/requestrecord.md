# RequestRecord Data Model

Scope: Canonical per-request client record for persistence and analysis.

## Purpose

`RequestRecord` is the authoritative row-level model for client-observed request outcomes. It is used by:
- JSONL/CSV run outputs,
- aggregate metric computation,
- report and plot generation.

## Canonical Field Groups

- **Identity and prompt context**
  - request index/id fields
  - prompt text or references
  - prompt length/token count metadata
- **Timing fields**
  - `sent_ts`
  - `first_token_ts`
  - `completion_ts`
- **Derived latency fields**
  - `ttft_s`
  - `total_latency_s`
  - `tpot_s`
- **Token usage fields**
  - prompt token count
  - completion token count
  - total token count (when available)
- **Output and status fields**
  - output text (or stream-assembled output)
  - error metadata/status
  - finish reason and related completion metadata

## Semantics

- Timing fields represent client-observed timestamps.
- Derived fields must be computed consistently with `docs/principles/metrics-semantics.md`.
- Queue/service timing is not directly encoded in `RequestRecord` unless explicitly sourced from server telemetry and modeled separately.

## Serialization Contract

- CSV output column ordering must match canonical schema/column definitions in implementation.
- JSONL and CSV representations must encode equivalent per-request information.
- Downstream analysis assumes required columns are present and correctly typed.

## Change Rules

Any field addition/removal/semantic change requires synchronized updates to:
- this document,
- `docs/data-models/run-artifacts.md`,
- `docs/principles/metrics-semantics.md` (if metrics-related),
- client parsing/analysis code and tests.

## Related Docs

- `docs/principles/metrics-semantics.md`
- `docs/data-models/run-artifacts.md`
- `docs/architecture/end-to-end-flow.md`
