# LLMQueueBenchmark Docs Index

This is the canonical navigation hub for repository knowledge.

Use this file to find the single source of truth for architecture, interfaces, metrics semantics, data models, operations, and planning.

## Orientation

- [Agent entry point](../AGENTS.md): purpose and working contract.
- [Current state](current-state.md): investigation status and evidence.
- [Architecture](../ARCHITECTURE.md): intended system boundaries.

Use the references relevant to the task; the lists below are not reading orders.

## Task-Based Routes

### I am changing metrics or timing semantics

Relevant references:
- `principles/metrics-semantics.md`
- `data-models/requestrecord.md`
- `data-models/telemetry-schema.md`
- `architecture/end-to-end-flow.md`

### I am changing HTTP/API behavior

Relevant references:
- `interfaces/http-api.md`
- `architecture/server.md`
- `architecture/dependency-boundaries.md`

### I am changing CLI arguments or environment variables

Relevant references:
- `interfaces/client-cli.md` or `interfaces/server-cli-env.md`
- `architecture/client.md` or `architecture/server.md`
- `principles/client-server-separation.md`

### I am changing artifact outputs, reports, or schemas

Relevant references:
- `principles/artifact-contract.md`
- `data-models/run-artifacts.md`
- `architecture/end-to-end-flow.md`

### I am changing scheduling, batching, or inference behavior

Relevant references:
- `architecture/server.md`
- `architecture/dependency-boundaries.md`
- `principles/client-server-separation.md`

### I am changing load generation, arrival, or workloads

Relevant references:
- `architecture/client.md`
- `architecture/end-to-end-flow.md`
- `principles/client-server-separation.md`

## Documentation Map

### Principles
- `principles/client-server-separation.md`
- `principles/metrics-semantics.md`
- `principles/artifact-contract.md`

### Interfaces
- `interfaces/client-cli.md`
- `interfaces/server-cli-env.md`
- `interfaces/http-api.md`

### Data Models
- [Experiment state record](data-models/experiment-state.md)
- `data-models/requestrecord.md`
- `data-models/run-artifacts.md`
- `data-models/telemetry-schema.md`

### Architecture
- `architecture/client.md`
- `architecture/server.md`
- `architecture/end-to-end-flow.md`
- `architecture/dependency-boundaries.md`

### Operations
- `operations/ml-sys-paper-claim-ledger.md` (historical numeric provenance; see current status)
- `operations/ml-sys-paper-draft.md` (draft; see current status)
- `operations/local-dev.md`
- `operations/vllm-gpu-settings.md`
- `operations/benchmark-recipes.md`
- `operations/ml-sys-comparison-validity-checklist.md`
- `operations/ml-sys-report-methodology.md`
- `operations/ml-sys-report-spec.json` (production matrix config)
- `operations/ml-sys-report-spec.dev.json` (dev-mode config)
- `operations/ml-sys-report-spec.remote.json` (remote endpoint matrix config)
- `operations/ml-sys-report-spec.ssh-tunnel.json` (SSH tunnel matrix config)
- `operations/ml-sys-report-template.md`
- `operations/debugging-playbooks.md`
- `operations/release-checklist.md`

### Planning
- [Current state](current-state.md)
- `plans/active/` (create when a task needs a persistent execution plan)
- `plans/completed/`
- `plans/tech-debt.md`

### References
- `references/llm-benchmark-client-map.md`
- `references/llm-scheduler-server-map.md`

## Maintenance

Update the specific document whose contract changed. Update the agent entry
point only for shared guidance or routing changes. Implementation facts and
intended contracts can disagree; record and resolve the discrepancy.
Completed plans and the legacy mapping below provide historical context.

## Legacy Mapping (from `readme.agents.md`)

- Section 1 + 1.1 -> `../AGENTS.md`, `glossary.md`
- Section 2 -> `architecture/end-to-end-flow.md`
- Section 3 -> `references/llm-benchmark-client-map.md`, `references/llm-scheduler-server-map.md`
- Section 4 -> `interfaces/client-cli.md`, `interfaces/server-cli-env.md`, `interfaces/http-api.md`
- Section 5 -> `principles/metrics-semantics.md`, `data-models/requestrecord.md`, `data-models/telemetry-schema.md`
- Section 6 + 6.1 -> `principles/artifact-contract.md`, `data-models/run-artifacts.md`
- Section 7 -> `../ARCHITECTURE.md` (top-level only) + corresponding detailed topic docs
- Section 8 -> `architecture/client.md`, `architecture/server.md`, `architecture/dependency-boundaries.md`
- Section 9 -> `plans/tech-debt.md` and selective promotion into `plans/active/`
