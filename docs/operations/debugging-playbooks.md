# Debugging Playbooks

Scope: Common failure patterns and investigation paths for benchmark and serving workflows.

## Playbook: Missing or Broken Streaming Output

Symptoms:
- client waits indefinitely or receives incomplete stream,
- missing usage terminal event in custom scheduler mode.

Checks:
1. Verify request uses `stream=true`.
2. Verify server mode and streaming semantics for that mode.
3. Confirm endpoint response content type and stream termination behavior.
4. Inspect server request lifecycle and stream event queue handling.

## Playbook: Telemetry Not Collected

Symptoms:
- empty or missing `server_metrics_timeseries.csv`,
- summary indicates no telemetry.

Checks:
1. Validate metrics endpoint URL and accessibility.
2. Confirm server exposes `/metrics` and required related endpoints.
3. Confirm telemetry poller started and polling interval is reasonable.
4. Verify mode-specific metric availability and history enablement.

## Playbook: Inconsistent Latency Metrics

Symptoms:
- unexpected TTFT/TPOT shifts,
- disagreement between client and server timing aggregates.

Checks:
1. Confirm source ownership (client vs server metric domain).
2. Verify no mixing of independent clocks without alignment.
3. Confirm warmup/cooldown and window settings used for aggregation.
4. Inspect request records for missing timestamp/token fields.

## Playbook: Sweep Outputs Incomplete

Symptoms:
- missing sweep summary files or capacity curve.

Checks:
1. Validate sweep argument configuration and output paths.
2. Confirm each per-lambda run produced expected run artifacts.
3. Check aggregate writer step and schema expectations.
4. Confirm no early run termination due to request or server failures.

## Playbook: Scheduler Behavior Unexpected

Symptoms:
- unexpected queue growth, batch sizes, or latency profile.

Checks:
1. Confirm selected scheduler and relevant batch parameters.
2. Verify runtime path (custom scheduler vs spawned vLLM runtime).
3. Inspect queue/batch metric series and request outcomes together.
4. Validate extension changes did not bypass intended scheduler contracts.

## Related Docs

- `docs/interfaces/http-api.md`
- `docs/principles/metrics-semantics.md`
- `docs/architecture/server.md`
