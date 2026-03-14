---
name: spawn-only-runtime-refactor
overview: Refactor server startup into two explicit runtimes (`custom` and `vllm`) and remove embedded/proxy vLLM modes, while keeping one-release CLI compatibility via deprecation mapping.
todos:
  - id: config-runtime-contract
    content: Add runtime-based config contract and legacy-flag normalization/deprecation warnings in config parsing.
    status: completed
  - id: main-spawn-only-wiring
    content: Refactor main startup/lifespan/endpoints to two-path runtime branching and remove embed/proxy branches.
    status: completed
  - id: scheduler-config-injection
    content: Remove scheduler dependence on global config by passing parsed config through factory/server.
    status: completed
  - id: tests-and-docs-alignment
    content: Update impacted tests and documentation to reflect spawn-only vLLM and new runtime CLI.
    status: completed
isProject: false
---

# Spawn-Only vLLM Runtime Refactor Plan

## Goal

Converge startup wiring into two clean paths:

- `custom` runtime for `naive|dynamic` batching schedulers
- `vllm` runtime for spawned vLLM subprocess only

Also keep one-release compatibility for legacy flags and mode values with clear deprecation warnings.

## Canonical docs consulted

- [/home/garuc/projects/LLMQueueBenchmark/docs/index.md](/home/garuc/projects/LLMQueueBenchmark/docs/index.md)
- [/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/server-cli-env.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/server-cli-env.md)
- [/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md)
- [/home/garuc/projects/LLMQueueBenchmark/docs/architecture/dependency-boundaries.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/dependency-boundaries.md)

## Implementation steps

- Update configuration contract in [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/utils/config.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/utils/config.py):
  - Add `runtime` with values `custom|vllm` (default `custom`).
  - Restrict `scheduler` to `naive|dynamic` for new path.
  - Keep vLLM settings required for spawn path (`host/port/base_url`, launch args).
  - Add compatibility normalization: map `--scheduler vllm` and legacy `--vllm-mode {embed,proxy,spawn}` to new runtime semantics.
  - Emit explicit deprecation warnings with migration guidance.
- Simplify startup and request flow in [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/main.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/main.py):
  - Replace `scheduler == "vllm"`/`vllm_mode` branches with `runtime == "vllm"` check.
  - Remove embedded/proxy-specific startup and shutdown branches.
  - Keep a single vLLM startup path: spawn child then proxy all `/v1/*` traffic.
  - Keep custom server path unchanged for scheduler behavior.
  - Log deprecation warnings after logging setup so they are visible.
- Remove dead mode implementations and references:
  - Retire embedded-only surface in [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/vllm_server.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/vllm_server.py) (delete file or keep as deprecated shim if still referenced in tests during transition).
  - Keep [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/vllm_proxy.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/vllm_proxy.py) only as internal forwarding utility for spawn runtime.
- Fix scheduler configuration consistency (same refactor stream, low risk/high value):
  - Update [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/factory.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/factory.py), [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/naive.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/naive.py), and [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/dynamic.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/dynamic.py) to take config explicitly instead of importing module-global `config`.
  - Ensure [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/server.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/server.py) passes parsed config into scheduler construction.
- Update tests and docs for new contract:
  - Adjust/remove embedded-mode tests in [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/tests/test_vllm_server_telemetry.py](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/tests/test_vllm_server_telemetry.py).
  - Update CLI docs and examples in [/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/server-cli-env.md](/home/garuc/projects/LLMQueueBenchmark/docs/interfaces/server-cli-env.md), [/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md](/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md), and [/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/README.md](/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/README.md).
  - Remove references to embed/proxy from top-level architecture docs that describe runtime modes.

## Compatibility behavior (deprecation window)

- Accept legacy flags for one release.
- Normalize internally to:
  - `--runtime vllm` for any legacy `scheduler=vllm`/`vllm-mode=*` usage.
  - `spawn` as the only effective vLLM launch behavior.
- Emit one warning block at startup with exact replacement command examples.

## Validation

- Run server startup matrix:
  - `--runtime custom --scheduler naive`
  - `--runtime custom --scheduler dynamic`
  - `--runtime vllm` (spawn path)
  - legacy combinations to verify warning + correct normalization
- Run existing scheduler/server tests and targeted spawn path tests.
- Confirm `/health`, `/v1/models`, and `/v1/completions` behavior is preserved in both runtimes.

