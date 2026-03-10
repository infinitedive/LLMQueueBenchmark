# Local Development Runbook

Scope: Local setup and execution for client and server repositories.

## Environment Setup

- Maintain separate virtual environments per repo (`.venv` under each repo).
- Activate the matching environment before install/run.
- Install dependencies from each repo's packaging/dependency files.

## Server Startup Paths

- Custom scheduler mode (default benchmarking path).
- vLLM runtime (managed subprocess + forwarding).

Use `docs/interfaces/server-cli-env.md` for exact flag semantics.

## Client Startup Paths

- Installed command path (`llm-bench`) after editable install.
- Module invocation path (`python -m llm_bench`) without install.

Use `docs/interfaces/client-cli.md` for exact flag semantics.

## Recommended Local Workflow

1. Start server and verify `GET /health`.
2. Verify metrics endpoint availability (`/metrics` at minimum).
3. Run a short benchmark from client with explicit `--outdir`.
4. Confirm expected run artifacts are generated.
5. Optionally run a small sweep to verify aggregation pipeline.

## Troubleshooting Checklist

- If no telemetry appears, check metrics endpoint URL and server mode support.
- If GPU metrics are absent, verify CUDA/NVML availability and config.
- If streaming behavior differs, verify scheduler mode and `stream` usage.
- If artifacts are missing columns/files, check schema and writer alignment.

## Related Docs

- `docs/interfaces/client-cli.md`
- `docs/interfaces/server-cli-env.md`
- `docs/interfaces/http-api.md`
- `docs/data-models/run-artifacts.md`
