# Telemetry Schema

Scope: Server-exported metrics data consumed by the client telemetry pipeline.

## Sources

- `GET /metrics` (Prometheus text)
- `GET /metrics/json`
- `GET /metrics/csv`
- `GET /metrics/summary`
- `GET /metrics/throughput-latency`
- `GET /metrics/history` (when enabled)

## Metric Families

- **Request-level**
  - in-flight requests
  - total request counters by status
  - request latency distributions
- **Batch/scheduler-level**
  - queue length
  - batch size distributions
  - batch processing latency
- **Token-level**
  - TTFT and TPOT distributions
  - input/output token counters
  - token throughput gauges
- **GPU-level**
  - memory allocated/reserved/peak
  - utilization metrics when available

## Time-Series and History Semantics

- Telemetry samples are server-origin measurements scraped at poll intervals.
- `/metrics/history` exposes buffered history and may support windowing/query parameters.
- History endpoint availability depends on server-side history enablement settings.

## Client Ingestion Rules

- Client poller treats server metrics as authoritative for server timing/resource dimensions.
- Missing metrics should be handled gracefully; ingestion must tolerate partial availability.
- Metric names and labels are contract surface; parsing logic must be updated for renames.

## Change Rules

Any telemetry schema change requires synchronized updates to:
- this document,
- `docs/interfaces/http-api.md`,
- `docs/principles/metrics-semantics.md`,
- telemetry parser implementation.

## Related Docs

- `docs/interfaces/http-api.md`
- `docs/principles/metrics-semantics.md`
- `docs/data-models/run-artifacts.md`
