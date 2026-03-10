---
name: Migrate Agent Knowledge Base
overview: Migrate all durable content from readme.agents.md into the new docs-based knowledge base with single-source ownership and minimal top-level duplication.
todos:
  - id: map-sections
    content: Create and validate one-to-one mapping from readme.agents.md sections to canonical destination files.
    status: completed
  - id: author-contract-docs
    content: Populate interfaces and data-model docs first, then principles and architecture deep dives.
    status: completed
  - id: populate-reference-ops
    content: Fill references and operations docs, then migrate predicted features to plans/tech-debt.
    status: completed
  - id: dedupe-and-retire-legacy
    content: Run de-duplication audit and convert readme.agents.md to archive or redirect stub.
    status: completed
isProject: false
---

# Comprehensive Knowledge Base Migration Plan

## Objective

Move the authoritative content from [readme.agents.md](/home/garuc/projects/LLMQueueBenchmark/readme.agents.md) into the new docs structure, while preserving architecture boundaries and preventing redundancy across [AGENTS.md](/home/garuc/projects/LLMQueueBenchmark/AGENTS.md), [ARCHITECTURE.md](/home/garuc/projects/LLMQueueBenchmark/ARCHITECTURE.md), and topic docs.

## Migration Principles

- Single-source ownership: each concept has one canonical file.
- Keep top-level docs thin: [AGENTS.md](/home/garuc/projects/LLMQueueBenchmark/AGENTS.md) routes, [ARCHITECTURE.md](/home/garuc/projects/LLMQueueBenchmark/ARCHITECTURE.md) governs high-level contracts only.
- No repeated formulas/contracts across files; deep details belong in topic docs.
- Preserve client/server separation and interface-first coupling.

## Section-to-Destination Mapping

- Source section 1 + 1.1 -> [AGENTS.md](/home/garuc/projects/LLMQueueBenchmark/AGENTS.md), [docs/glossary.md](/home/garuc/projects/LLMQueueBenchmark/docs/glossary.md)
- Source section 2 -> [docs/architecture/end-to-end-flow.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/end-to-end-flow.md)
- Source section 3 -> [docs/references/llm-benchmark-client-map.md](/home/garuc/projects/LLMQueueBenchmark/docs/references/llm-benchmark-client-map.md), [docs/references/llm-scheduler-server-map.md](/home/garuc/projects/LLMQueueBenchmark/docs/references/llm-scheduler-server-map.md)
- Source section 4 -> [docs/interfaces/client-cli.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/client-cli.md), [docs/interfaces/server-cli-env.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/server-cli-env.md), [docs/interfaces/http-api.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/http-api.md)
- Source section 5 -> [docs/principles/metrics-semantics.md](/home/garuc/projects/LLMQueueBenchmark/docs/principles/metrics-semantics.md), [docs/data-models/requestrecord.md](/home/garuc/projects/LLMQueueBenchmark/docs/data-models/requestrecord.md), [docs/data-models/telemetry-schema.md](/home/garuc/projects/LLMQueueBenchmark/docs/data-models/telemetry-schema.md)
- Source section 6 + 6.1 -> [docs/principles/artifact-contract.md](/home/garuc/projects/LLMQueueBenchmark/docs/principles/artifact-contract.md), [docs/data-models/run-artifacts.md](/home/garuc/projects/LLMQueueBenchmark/docs/data-models/run-artifacts.md)
- Source section 7 -> architecture-level subset in [ARCHITECTURE.md](/home/garuc/projects/LLMQueueBenchmark/ARCHITECTURE.md); detailed assumptions in topic docs above
- Source section 8 -> [docs/architecture/client.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/client.md), [docs/architecture/server.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md), [docs/architecture/dependency-boundaries.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/dependency-boundaries.md)
- Source section 9 -> [docs/plans/tech-debt.md](/home/garuc/projects/LLMQueueBenchmark/docs/plans/tech-debt.md) (or promote selected items into [docs/plans/active/](/home/garuc/projects/LLMQueueBenchmark/docs/plans/active/))

## Execution Phases

### Phase 1: Foundation and Vocabulary

- Populate [docs/glossary.md](/home/garuc/projects/LLMQueueBenchmark/docs/glossary.md) with canonical terms used across client/server/metrics/artifacts.
- Normalize wording in [AGENTS.md](/home/garuc/projects/LLMQueueBenchmark/AGENTS.md), [ARCHITECTURE.md](/home/garuc/projects/LLMQueueBenchmark/ARCHITECTURE.md), and [docs/index.md](/home/garuc/projects/LLMQueueBenchmark/docs/index.md) to use glossary terms.

### Phase 2: Contracts First (Interfaces + Data Models)

- Write interface contracts in:
  - [docs/interfaces/client-cli.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/client-cli.md)
  - [docs/interfaces/server-cli-env.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/server-cli-env.md)
  - [docs/interfaces/http-api.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/http-api.md)
- Write canonical schemas/fields in:
  - [docs/data-models/requestrecord.md](/home/garuc/projects/LLMQueueBenchmark/docs/data-models/requestrecord.md)
  - [docs/data-models/telemetry-schema.md](/home/garuc/projects/LLMQueueBenchmark/docs/data-models/telemetry-schema.md)
  - [docs/data-models/run-artifacts.md](/home/garuc/projects/LLMQueueBenchmark/docs/data-models/run-artifacts.md)

### Phase 3: Principles and Flows

- Write rule-level docs:
  - [docs/principles/metrics-semantics.md](/home/garuc/projects/LLMQueueBenchmark/docs/principles/metrics-semantics.md)
  - [docs/principles/artifact-contract.md](/home/garuc/projects/LLMQueueBenchmark/docs/principles/artifact-contract.md)
  - [docs/principles/client-server-separation.md](/home/garuc/projects/LLMQueueBenchmark/docs/principles/client-server-separation.md)
- Capture runtime narrative in [docs/architecture/end-to-end-flow.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/end-to-end-flow.md).

### Phase 4: Architecture Deep Dives and Extension Points

- Fill module architecture docs:
  - [docs/architecture/client.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/client.md)
  - [docs/architecture/server.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md)
  - [docs/architecture/dependency-boundaries.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/dependency-boundaries.md)
- Encode extension-point guidance from source section 8 without duplicating interface/schema details.

### Phase 5: References, Operations, and Planning

- Populate code map references:
  - [docs/references/llm-benchmark-client-map.md](/home/garuc/projects/LLMQueueBenchmark/docs/references/llm-benchmark-client-map.md)
  - [docs/references/llm-scheduler-server-map.md](/home/garuc/projects/LLMQueueBenchmark/docs/references/llm-scheduler-server-map.md)
- Fill operational docs:
  - [docs/operations/local-dev.md](/home/garuc/projects/LLMQueueBenchmark/docs/operations/local-dev.md)
  - [docs/operations/benchmark-recipes.md](/home/garuc/projects/LLMQueueBenchmark/docs/operations/benchmark-recipes.md)
  - [docs/operations/debugging-playbooks.md](/home/garuc/projects/LLMQueueBenchmark/docs/operations/debugging-playbooks.md)
  - [docs/operations/release-checklist.md](/home/garuc/projects/LLMQueueBenchmark/docs/operations/release-checklist.md)
- Move predicted features and debt to [docs/plans/tech-debt.md](/home/garuc/projects/LLMQueueBenchmark/docs/plans/tech-debt.md), with selected execution-ready items in [docs/plans/active/](/home/garuc/projects/LLMQueueBenchmark/docs/plans/active/).

### Phase 6: De-duplication and Legacy Handling

- Perform a redundancy pass:
  - ensure formulas and endpoint semantics appear in one place only,
  - remove duplicated statements from top-level docs,
  - verify every concept is reachable via [docs/index.md](/home/garuc/projects/LLMQueueBenchmark/docs/index.md).
- Convert [readme.agents.md](/home/garuc/projects/LLMQueueBenchmark/readme.agents.md) into either:
  - an archival snapshot with clear status, or
  - a redirect stub to [AGENTS.md](/home/garuc/projects/LLMQueueBenchmark/AGENTS.md) and [docs/index.md](/home/garuc/projects/LLMQueueBenchmark/docs/index.md).

## Acceptance Criteria

- 100% of sections from [readme.agents.md](/home/garuc/projects/LLMQueueBenchmark/readme.agents.md) mapped and migrated to new canonical docs.
- [AGENTS.md](/home/garuc/projects/LLMQueueBenchmark/AGENTS.md) remains routing-focused; [ARCHITECTURE.md](/home/garuc/projects/LLMQueueBenchmark/ARCHITECTURE.md) remains top-level-contract-focused.
- No duplicate metric formulas or endpoint semantics across top-level and topic docs.
- `docs/index.md` provides a complete route to every canonical topic.
- Legacy handling decision for [readme.agents.md](/home/garuc/projects/LLMQueueBenchmark/readme.agents.md) implemented and documented.

## Suggested Work Cadence

- Implement phases in small PR-sized batches (contracts first, then architecture, then references/operations, then cleanup).
- End each batch with a focused redundancy audit before moving to the next phase.

