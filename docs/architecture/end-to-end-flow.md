# End-to-End Flow

This document describes the runtime flow across client and server, including request handling and telemetry correlation.

## Benchmark Execution Flow

1. Client CLI resolves run configuration and workload inputs.
2. Workload loader prepares prompts and related metadata.
3. Arrival generator emits request schedule:
   - closed-loop deterministic sequencing, or
   - Poisson inter-arrival process.
4. Client request executor sends `/v1/completions` requests, optionally streaming.
5. Client captures request timestamps and output/token metadata into request records.
6. Server accepts requests and routes by configured mode:
   - custom batching path (`BatchingServer` + scheduler + batch processor), or
   - spawned vLLM runtime path (managed subprocess + forwarding).
7. Server returns completion responses (JSON or SSE stream).
8. Client persists request-level artifacts and computes aggregate reports.

## Server Processing Flow (Custom Scheduler Mode)

1. HTTP handler creates request lifecycle state.
2. Request enters scheduler queue.
3. Scheduler forms batch based on size/timeout policy.
4. Batch processor performs tokenization and model inference.
5. Stream events or final completion are emitted to requester.
6. Metrics are updated for queue, batch, token, latency, and request status.

## vLLM Mode Flow

- Server starts a vLLM subprocess via launch arguments and host/port settings.
- Server forwards `/v1/*` traffic to that spawned vLLM endpoint.
- Health checks validate upstream vLLM reachability via `/v1/models`.

Across runtimes, API compatibility and telemetry integration should remain consistent with published interfaces.

## Telemetry Correlation Flow

1. Server continuously updates Prometheus-backed metrics.
2. Client telemetry poller scrapes metrics endpoints at configured intervals.
3. Client aligns telemetry samples with run windows and request aggregates.
4. Output includes time series and summary data where telemetry is available.

## Ground vs Derived in Flow Context

- **Ground events**: request sends/receives, stream chunks, server metric snapshots.
- **Derived outputs**: percentiles, per-bin summaries, throughput-latency windows, sweep curves.

## Extension Hooks in Flow

- Client extension points:
  - workload sources,
  - arrival policies,
  - analysis/report outputs.
- Server extension points:
  - scheduler strategies,
  - model/batch handling,
  - vLLM spawn launch/forwarding controls,
  - metrics exporter/history capabilities.

## Related Docs

- `docs/architecture/client.md`
- `docs/architecture/server.md`
- `docs/principles/metrics-semantics.md`
- `docs/data-models/run-artifacts.md`
