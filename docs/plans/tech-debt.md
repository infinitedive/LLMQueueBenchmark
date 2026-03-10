# Technical Debt Tracker

This file tracks architecture-aligned improvement opportunities and longer-horizon follow-up work.

## Status Legend

- `proposed`: identified but not scheduled.
- `candidate`: scoped enough for prioritization.
- `active`: currently being executed in `docs/plans/active/`.
- `deferred`: intentionally postponed.

## Debt Items

### TD-001: Token-Accurate Client Counting

- Status: `proposed`
- Origin: legacy section 9 prediction
- Problem: token estimation can be approximate without usage-aligned tokenization in all paths.
- Impact: potential drift in TPOT/token-throughput interpretation.
- Next step: define backend-aware tokenizer strategy and validation dataset.

### TD-002: Arrival Trace Replay Mode

- Status: `proposed`
- Origin: legacy section 9 prediction
- Problem: no first-class trace replay arrival mode for reproducing real traffic traces.
- Impact: limited replay fidelity for production-like arrival shapes.
- Next step: define trace file format and drift tolerance policy.

### TD-003: vLLM Proxy Request Identity and Timing Propagation

- Status: `candidate`
- Origin: legacy section 9 prediction
- Problem: proxy paths may limit request-level timing correlation richness.
- Impact: reduced observability parity between custom and proxied modes.
- Next step: specify backward-compatible enrichment strategy for proxy metadata.

### TD-004: Adaptive Sweep Controller

- Status: `proposed`
- Origin: legacy section 9 prediction
- Problem: sweep orchestration is static and may over/under-sample useful capacity regions.
- Impact: slower iteration on SLA-oriented capacity exploration.
- Next step: define stopping criteria and convergence policy.

### TD-005: Metrics History Query Extensions

- Status: `proposed`
- Origin: legacy section 9 prediction
- Problem: history querying/filtering capabilities may be limited for larger investigations.
- Impact: heavier downstream post-processing load.
- Next step: define supported label/window filters and performance constraints.

### TD-006: Scheduler Strategy Expansion

- Status: `candidate`
- Origin: legacy section 9 prediction
- Problem: strategy surface is currently limited to existing scheduler implementations.
- Impact: constrained experimentation space for fairness/latency tradeoffs.
- Next step: define extension API expectations and fairness evaluation criteria.

## Promotion Rules

- Promote item to `active` only when:
  - success criteria are explicit,
  - interface/schema impact is understood,
  - owner and scope are identified.

When promoted, create a dedicated execution plan in `docs/plans/active/` and link it from this file.
