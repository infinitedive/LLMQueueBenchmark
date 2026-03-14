# vLLM GPU Settings and Feature Requirements

Scope: practical vLLM spawn-mode settings for different GPU capabilities when running `llm-scheduler-server`.

## When To Use This

Use this page when `--runtime vllm` starts but:
- fails during dtype selection (`bfloat16` unsupported),
- fails with GPU memory utilization checks,
- or needs tuning to fit VRAM on local/dev hardware.

This page complements:
- `docs/operations/local-dev.md` (startup flow),
- `docs/interfaces/server-cli-env.md` (flag contract).

## Launch Arg Syntax (Important)

For vLLM flags that begin with `--`, pass each value with `=`:

```bash
--vllm-launch-arg=--dtype --vllm-launch-arg=half
```

Avoid:

```bash
--vllm-launch-arg --dtype
```

Without `=`, argparse may treat `--dtype` as a new top-level flag instead of a value.

## GPU Feature Requirements (Quick Reference)

- `dtype=bfloat16` requires NVIDIA compute capability `>= 8.0`.
- `dtype=half` (float16) is the safe default for older GPUs (`< 8.0`).
- FlashAttention v2 may be unavailable on older GPUs; vLLM will log fallback behavior.
- Default `gpu_memory_utilization=0.9` can fail on constrained or busy GPUs.

## Preset Profiles

All presets are spawn-mode via the scheduler entrypoint:

```bash
python -m batching_scheduler \
  --runtime vllm \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --vllm-host 127.0.0.1 \
  --vllm-port 8001 \
  ... \
  --host 0.0.0.0 \
  --port 8000
```

### Profile A: Older GPU / 6-8 GB VRAM (safe default)

Use this first on consumer GPUs where `bfloat16` is unsupported or memory is tight.

```bash
python -m batching_scheduler \
  --runtime vllm \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --vllm-host 127.0.0.1 \
  --vllm-port 8001 \
  --vllm-launch-arg=--dtype \
  --vllm-launch-arg=half \
  --vllm-launch-arg=--gpu-memory-utilization \
  --vllm-launch-arg=0.7 \
  --host 0.0.0.0 \
  --port 8000
```

### Profile B: Newer GPU with bfloat16 support

Use this when compute capability is `>= 8.0` and memory headroom is sufficient.

```bash
python -m batching_scheduler \
  --runtime vllm \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --vllm-host 127.0.0.1 \
  --vllm-port 8001 \
  --vllm-launch-arg=--dtype \
  --vllm-launch-arg=bfloat16 \
  --vllm-launch-arg=--gpu-memory-utilization \
  --vllm-launch-arg=0.85 \
  --host 0.0.0.0 \
  --port 8000
```

### Profile C: Memory-constrained fallback

If startup reports low free memory, lower utilization further.

```bash
python -m batching_scheduler \
  --runtime vllm \
  --model-name TinyLlama/TinyLlama-1.1B-Chat-v1.0 \
  --vllm-host 127.0.0.1 \
  --vllm-port 8001 \
  --vllm-launch-arg=--dtype \
  --vllm-launch-arg=half \
  --vllm-launch-arg=--gpu-memory-utilization \
  --vllm-launch-arg=0.6 \
  --host 0.0.0.0 \
  --port 8000
```

## Verification Steps

After startup:

```bash
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/v1/models
```

Expected:
- `/health` reports `{"status":"ok", ...}` once the spawned vLLM server is fully initialized.
- `/v1/models` returns the configured model in a list payload.

## Troubleshooting Signals

- `Bfloat16 is only supported ... compute capability ...`: switch to `dtype=half`.
- `Free memory on device ... less than desired GPU memory utilization`: lower `--gpu-memory-utilization` (for example `0.7` then `0.6`).
- `status: unavailable` during startup can be transient while model loads and warms up; re-check after logs show vLLM routes started.
