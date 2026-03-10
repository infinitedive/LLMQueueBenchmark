# Glossary

Canonical terminology for the LLMQueueBenchmark knowledge base.

## Core System Terms

- **Workspace**: The combined two-repo system rooted at `LLMQueueBenchmark`.
- **Client**: `llm-benchmark-client` (`llm_bench`), responsible for load generation and analysis.
- **Server**: `llm-scheduler-server` (`batching_scheduler`), responsible for request serving and scheduling.
- **Boundary**: An external contract surface (CLI/env, HTTP API, artifact schema).
- **Layering**: Intended dependency direction among packages within one repo.

## Load Generation Terms

- **Workload**: Prompt source and prompt selection mode (`demo`, `short`, `long`, `mixed`, `file`, dataset-backed modes).
- **Arrival Pattern**: Request timing policy (`closed` deterministic loop or `poisson` open-loop process).
- **Lambda (lambda-rps)**: Target request arrival rate for Poisson mode.
- **Horizon**: Duration window for Poisson run scheduling.
- **Request Budget**: Optional cap on request count in open-loop runs.
- **Concurrency Limit**: Maximum in-flight client requests, controlled by semaphore.

## Runtime and Response Terms

- **Streaming Request**: `/v1/completions` call with `stream=true` returning SSE chunks.
- **Non-Streaming Request**: `/v1/completions` call with `stream=false` returning one JSON response.
- **TTFT**: Time to first token from client send timestamp to first token timestamp.
- **TPOT**: Time per output token for generated tokens.
- **Total Latency**: End-to-end client-observed latency from send to completion.
- **Queue Time / Service Time**: Server-side timing dimensions obtained from server metrics, not derived from client timestamps alone.

## Data and Artifact Terms

- **RequestRecord**: Canonical per-request client record used for persistence and analysis.
- **Run Artifacts**: Files produced for one run, typically under `runs/<run_id>`.
- **Sweep Artifacts**: Aggregated multi-lambda outputs under `sweep_runs/`.
- **Per-Bin Summary**: Aggregates grouped by prompt length bins.
- **Throughput-Latency Time Series**: Windowed performance series over run time.

## Telemetry Terms

- **Telemetry Poller**: Client component scraping server `/metrics*` endpoints.
- **Metrics Endpoint Family**: `/metrics`, `/metrics/json`, `/metrics/csv`, `/metrics/summary`, `/metrics/throughput-latency`, `/metrics/history`.
- **Metrics History**: Server-side buffered time series, available when enabled.
- **GPU Monitor**: Server-side component publishing GPU utilization and memory gauges when available.

## Documentation Terms

- **Canonical Doc**: Single source of truth for a concept.
- **Routing Doc**: Navigation-focused doc (`AGENTS.md`, `docs/index.md`).
- **Architecture Contract**: Top-level constraints and boundaries (`ARCHITECTURE.md`).
- **Principles Doc**: Rule-level semantics without low-level schema duplication.
- **Reference Doc**: Code map and module responsibility index.
