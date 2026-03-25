# Local Development Runbook

Scope: Local setup and execution for client and server repositories.

## Environment Setup

- Maintain separate virtual environments per repo (`.venv` under each repo).
- Activate the matching environment before install/run.
- Install dependencies from each repo's packaging/dependency files.

### Activate Repo-Specific `.venv`

Use the environment for the repo you are currently running from.

Server repo:
```bash
source llm-scheduler-server/.venv/bin/activate
which python
```

Client repo:
```bash
source llm-benchmark-client/.venv/bin/activate
which python
```

Notes:
- Only one virtual environment can be active per shell session.
- Re-run `source <repo>/.venv/bin/activate` when switching repos.
- `which python` should point to the selected repo's `.venv/bin/python`.

## Server Startup Paths

- Custom scheduler mode (default benchmarking path).
- vLLM runtime (managed subprocess + forwarding).

### Run Each Server Option

From workspace root:
```bash
source llm-scheduler-server/.venv/bin/activate
cd llm-scheduler-server
```

1) Custom runtime + naive scheduler:
```bash
python -m batching_scheduler \
  --runtime custom \
  --scheduler naive \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --max-batch-size 32 \
  --batch-timeout 0.1 \
  --host 0.0.0.0 \
  --port 8000
```

2) Custom runtime + dynamic scheduler:
```bash
python -m batching_scheduler \
  --runtime custom \
  --scheduler dynamic \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --max-batch-size 32 \
  --dynamic-bs-min 2 \
  --dynamic-bs-mid 16 \
  --dynamic-bs-max 32 \
  --dynamic-q1 2 \
  --dynamic-q2 16 \
  --dynamic-timeout-min-ms 2 \
  --dynamic-timeout-mid-ms 12 \
  --dynamic-timeout-max-ms 30 \
  --dynamic-max-wait-ms 150 \
  --host 0.0.0.0 \
  --port 8000
```

3) vLLM runtime (spawn-only):
```bash
python -m batching_scheduler \
  --runtime vllm \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --vllm-host 127.0.0.1 \
  --vllm-port 8001 \
  --vllm-launch-arg=--dtype \
  --vllm-launch-arg=half \
  --host 0.0.0.0 \
  --port 8000
```

Quick verification (all options):
```bash
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/v1/models
```

Legacy compatibility note:
- `--scheduler vllm` and `--vllm-mode ...` are accepted during deprecation window and normalized to `--runtime vllm`.

Use `docs/interfaces/server-cli-env.md` for exact flag semantics.
Use `docs/operations/vllm-gpu-settings.md` for GPU-specific launch presets and feature requirements.

## Client Startup Paths

- Installed command path (`llm-bench`) after editable install.
- Module invocation path (`python -m llm_bench`) without install.

Use `docs/interfaces/client-cli.md` for exact flag semantics.

## Recruiter/Hiring Manager Quick Path

Use this path when you want a simple "does it run" benchmark sweep without tuning every flag.

### 1) Start the server (Terminal A)

From workspace root:
```bash
source llm-scheduler-server/.venv/bin/activate
cd llm-scheduler-server
python -m batching_scheduler \
  --runtime custom \
  --scheduler naive \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --host 0.0.0.0 \
  --port 8000
```

Sanity check (new shell):
```bash
curl http://127.0.0.1:8000/health
```

### 2) Run a small sweep (Terminal B)

From workspace root:
```bash
source llm-benchmark-client/.venv/bin/activate
cd llm-benchmark-client
python -m llm_bench \
  --api-url http://127.0.0.1:8000/v1 \
  --metrics-url http://127.0.0.1:8000/metrics \
  --model TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --workload demo \
  --arrival poisson \
  --horizon-s 30 \
  --max-concurrent 16 \
  --sweep \
  --sweep-lambdas 1 2 4 \
  --sweep-repeats 2 \
  --outdir ../sweep_runs_recruiter_quickstart
```

### 3) Confirm outputs

You should see:
- `sweep_runs_recruiter_quickstart/sweep_raw.csv`
- `sweep_runs_recruiter_quickstart/sweep_summary.csv`
- `sweep_runs_recruiter_quickstart/capacity_curve.png`
- `sweep_runs_recruiter_quickstart/lambda*_rep*/` per-run artifact folders

Notes:
- This uses small defaults to complete quickly; it is for demonstration, not exhaustive capacity analysis.
- For detailed tuning and canonical patterns, continue with `docs/operations/benchmark-recipes.md`.

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
