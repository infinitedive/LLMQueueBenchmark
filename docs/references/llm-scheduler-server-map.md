# Server Module Map (`llm-scheduler-server`)

Reference map for `batching_scheduler` modules, responsibilities, and common runtime wiring.

## Entrypoints

- `batching_scheduler.__main__`: module entrypoint.
- `batching_scheduler.server.main`: FastAPI app startup and endpoint wiring.

## Module Responsibilities

- `batching_scheduler.server.main`
  - app creation, mode wiring, endpoint registration, metrics routes.
- `batching_scheduler.server.server`
  - custom batching server queue/process loop orchestration.
- `batching_scheduler.server.request`
  - request lifecycle model and stream event handling primitives.
- `batching_scheduler.server.vllm_proxy`
  - forwarding layer to spawned vLLM server.
- `batching_scheduler.server.vllm_spawn`
  - managed vLLM subprocess lifecycle.
- `batching_scheduler.schedulers.base`
  - scheduler interface contract.
- `batching_scheduler.schedulers.naive`
  - FCFS/timeout scheduling implementation.
- `batching_scheduler.schedulers.dynamic`
  - adaptive queue/age-based scheduling implementation.
- `batching_scheduler.engine.model_manager`
  - model loading and setup.
- `batching_scheduler.engine.batch_processor`
  - tokenization and batch inference execution.
- `batching_scheduler.utils.config`
  - settings and configuration plumbing.
- `batching_scheduler.utils.metrics`
  - metric definitions and update helpers.
- `batching_scheduler.utils.metrics_exporter`
  - JSON/CSV/summary/throughput-latency exporters.
- `batching_scheduler.utils.metrics_history`
  - in-memory history buffer/query logic.
- `batching_scheduler.utils.gpu_monitor_async`
  - GPU utilization/memory monitoring support.
- `batching_scheduler.utils.logging`
  - logging setup helpers.

## Typical Call Graph (High-Level)

- entrypoint -> server.main -> mode selection
- custom mode -> server.server + schedulers + engine
- vllm runtime -> vllm_spawn + vllm_proxy
- all modes -> metrics instrumentation/export utilities

## Notes

- This file is a reference map, not a contract definition.
- API and data contract definitions live in `docs/interfaces/*` and `docs/data-models/*`.
