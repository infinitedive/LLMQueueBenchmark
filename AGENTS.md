# LLMQueueBenchmark

## Purpose and workspace

This parent repository holds the project's working knowledge and research context.
The client and server are cloned inside it as independent Git repositories:
- `llm-benchmark-client/`: load generation, client measurements, reports.
- `llm-scheduler-server/`: scheduling, inference, server telemetry.

The project began as an educational investigation of scheduling and batching.
Preserve an inspectable experimental system; maximizing throughput alone does
not satisfy that purpose. Client/server integration stays through documented
interfaces, without runtime cross-imports.

## Context when needed

- Resuming research or interpreting results: `docs/current-state.md`.
- Architecture decisions and boundaries: `ARCHITECTURE.md`.
- Measurement definitions: `docs/principles/metrics-semantics.md`.
- Benchmark execution and claim acceptance: `docs/operations/ml-sys-report-methodology.md`.
- Setup and repo-specific environments: `docs/operations/local-dev.md`.
- Other topics: `docs/index.md`.

These are reference routes, not a mandatory reading sequence. Inspect the code
and documents relevant to the task. Completed plans describe historical work.

## Working contract

Use the affected component's virtualenv interpreter (or activate that virtualenv).
Keep changes in the appropriate Git repository. This KB is maintained separately;
do not copy private working notes into public component repositories by default.

Choose the implementation and verification procedure appropriate to the task.
For implementation work, continue through relevant available checks and fixes
within the authorized scope. Report unavailable checks explicitly; a committed
patch is not a verified result. User instructions determine action permissions.

Official benchmark artifacts come from the canonical client pipeline.
Temporary diagnostic probes are appropriate when clearly distinguished from
report evidence. Preserve existing run outputs.

Code establishes implemented behavior; research contracts establish intended
behavior. Resolve discrepancies explicitly rather than redefining intent to
match a bug. Claims about scheduling require evidence of actual inference
execution, not only queue configuration or artifact presence.

Update affected contracts when behavior changes. For research work, leave a
concise current state: supported findings, unresolved questions, evidence
references, and the next discriminating action.
