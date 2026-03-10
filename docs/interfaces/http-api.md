# HTTP API Contract

Scope: Network interface between `llm-benchmark-client` and `llm-scheduler-server`, plus observability export surfaces.

## Primary Endpoints

- `GET /health`: Liveness/health status.
- `GET /v1/models`: Available model metadata.
- `POST /v1/completions`: OpenAI-compatible completion generation.
- `GET /metrics`: Prometheus text export.
- `GET /metrics/json`: Structured JSON metric export.
- `GET /metrics/csv`: CSV metric export.
- `GET /metrics/summary`: Aggregated metric summary.
- `GET /metrics/throughput-latency`: Throughput/latency correlation export.
- `GET /metrics/history`: Time-windowed history export (when enabled).

## `/v1/completions` Semantics

- `stream=false`: server returns standard JSON completion object.
- `stream=true` in non-vLLM custom scheduler modes:
  - response is `text/event-stream`,
  - token deltas are emitted incrementally,
  - terminal event includes usage data,
  - stream ends with `[DONE]`.

vLLM-backed modes may produce mode-specific streaming details, but must remain compatible with the client's OpenAI-style streaming expectations.

## Contract Boundaries

- Request/response transport and payload contract are owned by this boundary.
- Internal queueing, batching, and model execution details are not part of the HTTP contract unless explicitly exported.
- Client-side derived metrics must not be interpreted as direct substitutes for server-exported queue/service telemetry.

## Compatibility Rules

- Endpoint path changes are breaking and require versioning or explicit migration handling.
- Response schema changes affecting client parsing are breaking unless additive and backward-compatible.
- Metrics endpoint changes require synchronized updates to:
  - `docs/data-models/telemetry-schema.md`
  - `docs/principles/metrics-semantics.md`
  - client telemetry parsing implementation.

## Related Docs

- `docs/interfaces/client-cli.md`
- `docs/data-models/telemetry-schema.md`
- `docs/principles/metrics-semantics.md`
