# Metrics Semantics

Scope: Canonical meaning and ownership model for performance metrics across client and server.

## Ownership Model

- Client is authoritative for client-observed request timings and derived client metrics.
- Server is authoritative for server-internal timing/resource metrics (queueing, batching internals, GPU telemetry).
- Aggregated reports may combine both sources, but source ownership must remain explicit.

## Primary Metric Definitions

- **TTFT (client)**: first token timestamp minus send timestamp.
- **Total latency (client)**: completion timestamp minus send timestamp.
- **TPOT (client)**: generation span normalized by generated token count.
- **Throughput (client-derived)**: requests/sec and tokens/sec over configured windows.
- **Queue/service dimensions (server-derived)**: from server metrics exports, not inferred from client-only events.

Detailed field-level formulas and schema live in:
- `docs/data-models/requestrecord.md`
- `docs/data-models/telemetry-schema.md`

## Streaming Semantics Impact

- Streaming responses are first-class for TTFT/TPOT capture.
- In custom non-vLLM streaming mode, usage-bearing terminal events enable robust client output and token accounting.
- Non-streaming mode remains valid for total latency and completion outcomes, with reduced token-arrival granularity.

## Percentiles and Aggregation

- Client percentile and distribution outputs are computed from client records.
- Server percentile outputs exposed by exporters/history are server-side derived and should be labeled accordingly.
- Reports must not blend client and server percentile sources without explicit attribution.

## Timing and Alignment Principles

- Client and server clocks are independent timelines unless explicitly aligned.
- Correlation across sources should use scrape/request context windows, not implicit timestamp equivalence.
- Warmup/cooldown exclusion windows apply to steady-state throughput/latency interpretations.

## Change Discipline

Changes to metric meaning, units, or attribution require synchronized updates to:
- this file,
- `docs/data-models/requestrecord.md`,
- `docs/data-models/telemetry-schema.md`,
- `docs/interfaces/http-api.md` when endpoint payload semantics change.

## Related Docs

- `docs/data-models/requestrecord.md`
- `docs/data-models/telemetry-schema.md`
- `docs/architecture/end-to-end-flow.md`
