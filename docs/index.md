# LLMQueueBenchmark Docs Index

This is the canonical navigation hub for repository knowledge.

Use this file to find the single source of truth for architecture, interfaces, metrics semantics, data models, operations, and planning.

## Start Here

1. Read `../AGENTS.md` for high-level routing rules.
2. Read `../ARCHITECTURE.md` for system context and package layering.
3. Use the task-based routes below to find the exact authoritative document.

## Task-Based Routes

### I am changing metrics or timing semantics

Read in this order:
1. `principles/metrics-semantics.md`
2. `data-models/requestrecord.md`
3. `data-models/telemetry-schema.md`
4. `architecture/end-to-end-flow.md`

### I am changing HTTP/API behavior

Read in this order:
1. `interfaces/http-api.md`
2. `architecture/server.md`
3. `architecture/dependency-boundaries.md`

### I am changing CLI arguments or environment variables

Read in this order:
1. `interfaces/client-cli.md` or `interfaces/server-cli-env.md`
2. `architecture/client.md` or `architecture/server.md`
3. `principles/client-server-separation.md`

### I am changing artifact outputs, reports, or schemas

Read in this order:
1. `principles/artifact-contract.md`
2. `data-models/run-artifacts.md`
3. `architecture/end-to-end-flow.md`

### I am changing scheduling, batching, or inference behavior

Read in this order:
1. `architecture/server.md`
2. `architecture/dependency-boundaries.md`
3. `principles/client-server-separation.md`

### I am changing load generation, arrival, or workloads

Read in this order:
1. `architecture/client.md`
2. `architecture/end-to-end-flow.md`
3. `principles/client-server-separation.md`

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
- `data-models/requestrecord.md`
- `data-models/run-artifacts.md`
- `data-models/telemetry-schema.md`

### Architecture
- `architecture/client.md`
- `architecture/server.md`
- `architecture/end-to-end-flow.md`
- `architecture/dependency-boundaries.md`

### Operations
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
- `plans/active/`
- `plans/completed/`
- `plans/tech-debt.md`

### References
- `references/llm-benchmark-client-map.md`
- `references/llm-scheduler-server-map.md`

## Update Rules

When behavior changes:
1. Update the relevant detailed doc first (principle/interface/data model/architecture).
2. Update `../AGENTS.md` only if routing or global rules changed.
3. Update `../ARCHITECTURE.md` only if top-level domain boundaries or package layering changed.

If code and docs conflict, code is temporary source of truth until docs are corrected in the same change stream.

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
