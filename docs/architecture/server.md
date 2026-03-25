# Server Architecture

Scope: Internal architecture of `llm-scheduler-server` (`batching_scheduler`).

## Domain Responsibilities

- `batching_scheduler.server`: FastAPI app, endpoint wiring, request lifecycle objects, mode routing.
- `batching_scheduler.schedulers`: scheduler interface and implementations.
- `batching_scheduler.engine`: model manager and batch processing execution.
- `batching_scheduler.utils`: config, logging, metrics, exporters, history, GPU monitor.
- `batching_scheduler.server.vllm_*`: spawned vLLM subprocess and forwarding integration.

## Layering Intent

1. Foundation layer: `utils`
2. Core execution layer: `engine`, `schedulers`
3. Boundary and composition layer: `server`

`server` owns HTTP behavior and composes lower layers; lower layers should not own HTTP contract details.

## Runtime Modes

- **Custom scheduler mode**: request queue + scheduler + batch processor.
- **vLLM runtime**: managed vLLM subprocess plus forwarding.

All modes share API boundary obligations and observability expectations.

## Key Runtime Components

- Request lifecycle model for queueing and streaming event coordination.
- Scheduler implementations with batch size/timeout logic.
- Batch processor for model/tokenization execution.
- Metrics instrumentation, exporters, and optional history buffering.
- GPU monitoring facilities where hardware/runtime support exists.

## Extension Points

- Add scheduler strategy via scheduler interface and registration.
- Extend batch/model processing in engine modules.
- Extend spawned vLLM launch/forwarding controls within vLLM integration modules.
- Extend metrics/export/history surfaces with compatibility discipline.

## Architectural Constraints

- Server should not own benchmark report generation.
- Internal scheduler/engine details should not leak as undocumented API assumptions.
- Metrics and streaming behavior changes must preserve published contracts or ship with coordinated migration updates.

## Scheduler Invariants

- Batch-release timing authority belongs to scheduler policy, not to a fixed server polling interval.
- The server may orchestrate wakeups and execution, but it must honor scheduler-provided release deadlines.
- Scheduler timeout/deadline comparisons use a monotonic clock basis to avoid wall-clock drift effects.
- Dynamic low-load policy must preserve a nonzero accumulation opportunity before sparse arrivals are released.
- Dynamic scheduler policy is three-tier (low/mid/high) and must preserve monotone threshold ordering (`q1 < q2`).
- Dynamic tier targets must remain monotone across tiers (`bs_min <= bs_mid <= bs_max`, `timeout_min_ms <= timeout_mid_ms <= timeout_max_ms`).
- Bounded completion still applies: timeout and max-wait caps can release underfilled batches when deadlines are reached.

## Related Docs

- `docs/interfaces/server-cli-env.md`
- `docs/interfaces/http-api.md`
- `docs/data-models/telemetry-schema.md`
- `docs/architecture/end-to-end-flow.md`
