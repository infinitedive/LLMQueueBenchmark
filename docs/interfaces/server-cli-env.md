# Server CLI and Environment Contract

Scope: `llm-scheduler-server` process configuration via CLI arguments and environment-backed settings.

## Entry Point

- Module command: `python -m batching_scheduler`

## Core Server Arguments

- `--runtime`: Runtime selection (`custom`, `vllm`).
- `--scheduler`: Scheduler selection for custom runtime (`naive`, `dynamic`).
- `--max-batch-size`: Upper bound for batch size in custom scheduling path.
- `--batch-timeout`: Wait threshold for batch formation.
- `--model-name`: Inference model identifier.
- `--tokenizer-name`: Optional tokenizer override.
- `--device`: Target device configuration for model execution.
- `--host`, `--port`: Server bind settings.
- Logging-related options for level and formatting.

## vLLM Integration Arguments

- vLLM runtime is spawn-only.
- `--vllm-host`, `--vllm-port`: Spawned vLLM host/port controls.
- `--vllm-base-url`: Target URL used by server-side proxy forwarding (defaults from host/port).
- `--vllm-launch-arg`: Extra launch argument passed to spawned vLLM subprocess (repeatable).
- `--vllm-mode`: deprecated compatibility flag; accepted for one release and normalized to spawn-only runtime behavior.

## Environment Contract

- Server configuration may be populated from `.env` using pydantic settings.
- GPU monitoring and metrics history behavior are controlled by settings fields.
- Environment values are configuration inputs; they do not redefine API payload semantics.

## Behavioral Contract

- CLI/env controls startup wiring and runtime behavior selection.
- Server process owns scheduling and inference behavior.
- Changing CLI/env must not silently break published HTTP contracts.
- Legacy startup flags are normalized with deprecation warnings during the compatibility window.

## Compatibility Rules

- Preserve backwards compatibility for common startup flags where feasible.
- Breaking changes to startup configuration require updates to:
  - this file,
  - `docs/operations/local-dev.md`,
  - `docs/operations/release-checklist.md` if rollout-sensitive.

## Related Docs

- `docs/interfaces/http-api.md`
- `docs/architecture/server.md`
- `docs/principles/client-server-separation.md`
