## 1) ONE-SENTENCE IDENTITY
Bench harness + configurable vLLM-style inference server for comparing scheduling/batching strategies; not a generic ML training stack or model zoo.

**EVIDENCE:** `llm-benchmark-client/readme.md`; `llm-scheduler-server/README.md`.

## 1.1) MAINTENANCE REQUIREMENT (AGENTS)
- If you change code or behavior, update `readme.agents.md` to keep this doc accurate.

## 2) CORE MENTAL MODEL (PIPELINE)
- Load prompts from demo/short/long/mixed/file/HF datasets via `load_prompts`.
- Choose arrival: closed-loop deterministic or Poisson open-loop; asyncio tasks gated by semaphore.
- Async client streams `/v1/completions`, capturing `sent_ts`, `first_token_ts`, `completion_ts`, tokens into `RequestRecord`.
- Telemetry poller scrapes server `/metrics` (KV cache, GPU) on interval.
- Aggregate results to DataFrames; split failures; compute throughput/latency windows with warmup/cooldown.
- Derive per-bin (prompt length) summaries + percentiles; aggregate server metrics mean/max when present.
- Persist artifacts (JSONL/CSV/plots/Markdown/summary JSON); optional sweep orchestrates multiple λ runs.
- Custom server: enqueue → scheduler batch (FCFS size/timeout) → batch processor tokenizes, runs HF model, updates Prometheus metrics/history.
- vLLM modes: proxy/embed/spawn to AsyncLLMEngine or external vLLM; streaming handler records TTFT/TPOT; GPU monitor thread updates gauges.
- Metrics endpoints export Prometheus text/JSON/CSV/summary/history; client correlates scraped telemetry offline.

**EVIDENCE:** `llm_bench/cli/runner.py`; `llm_bench/loadgen/arrival_patterns.py`; `llm_bench/loadgen/client.py`; `llm_bench/telemetry/poller.py`; `llm_bench/analysis/plots.py`; `batching_scheduler/server/server.py`; `batching_scheduler/server/vllm_server.py`; `batching_scheduler/utils/metrics_exporter.py`.

## 3) TOPOLOGY (MODULE MAP)
path | responsibility | key identifiers | main callers
- `llm_bench/cli/parser.py` | CLI args + run context | `build_parser`, `prepare_run` | `cli/runner.main`
- `llm_bench/cli/runner.py` | Single-run orchestration, persistence, plots | `_run`, `_execute_load_test`, `_compute_summary_aggregates` | `python -m llm_bench`
- `llm_bench/cli/sweep.py` | Multi-λ sweep orchestration | `run_sweep` | `cli/runner.main` when `--sweep`
- `llm_bench/loadgen/workload.py` | Prompt generation/loading | `load_prompts` | `_setup_run_components`
- `llm_bench/loadgen/arrival_patterns.py` | Arrival generators | `closed_loop_tasks`, `poisson_item_stream` | `_execute_load_test`
- `llm_bench/loadgen/client.py` | OpenAI-compatible async sender | `VLLMClient.send` | arrival generators
- `llm_bench/loadgen/records.py` | Canonical request record | `RequestRecord` | client, persistence, analysis
- `llm_bench/telemetry/poller.py` | `/metrics` scraper thread | `MetricsPoller` | `_start_telemetry_watchers`
- `llm_bench/analysis/{metrics,plots,reports,schema}.py` | Aggregations, plotting, artifact naming | `percentiles`, `build_throughput_latency_windows`, `write_markdown_summary`, `artifact_paths` | runner, sweep
- `batching_scheduler/server/main.py` | FastAPI app, wiring modes, endpoints | `app`, `create_completion`, metrics routes | uvicorn entry
- `batching_scheduler/server/server.py` | Custom batching server loop | `BatchingServer._processing_loop` | `main`, HTTP handler
- `batching_scheduler/server/vllm_server.py` | Embedded vLLM engine + streaming metrics | `VLLMServer.add_request` | `main` when `scheduler=vllm` embed
- `batching_scheduler/server/vllm_proxy.py` | Proxy router to external vLLM | `create_vllm_proxy_router` | `main` vllm proxy/spawn
- `batching_scheduler/server/vllm_spawn.py` | Manage vLLM subprocess | `VLLMSubprocessManager` | `main` spawn mode
- `batching_scheduler/schedulers/{base,naive,dynamic}.py` | Scheduler interface & FCFS implementations | `BaseScheduler`, `NaiveScheduler.get_batch`, `DynamicScheduler.get_batch` | `BatchingServer`
- `batching_scheduler/engine/{model_manager,batch_processor}.py` | HF model load + batch inference | `ModelManager.load_model`, `BatchProcessor.process_batch` | `BatchingServer`
- `batching_scheduler/utils/{config,metrics,metrics_exporter,metrics_history,gpu_monitor_async,logging}.py` | Config, Prometheus metrics, exporters, history, GPU monitors, logging | various | server/main, BatchingServer, VLLMServer

**EVIDENCE:** files above.

## 4) CONTROL SURFACES
- CLI (client): Run via `llm-bench` (if installed) or `python -m llm_bench` (without installation). Args: `--api-url`, `--metrics-url`, `--model`, `--arrival`, `--lambda-rps`, `--horizon-s`, `--request-budget`, `--iterations`, `--max-concurrent`, `--max-tokens`, `--workload*`, `--warmup-s`, `--cooldown-s`, `--window-s`, `--sweep*`.
- CLI (server): `python -m batching_scheduler` with args: `--scheduler`, `--max-batch-size`, `--batch-timeout`, `--model-name`/`tokenizer-name`, `--device`, `--vllm-mode`/host/port/base-url, logging, host/port.
- Installation: Client can be installed via `pip install -e .` in `llm-benchmark-client/` to enable `llm-bench` command; or run as module without install.
- Env: server uses `.env` via pydantic settings; GPU monitoring interval/history toggles in config fields.
- HTTP: `/health`, `/v1/models`, `/v1/completions`, `/metrics`, `/metrics/json`, `/metrics/csv`, `/metrics/summary`, `/metrics/throughput-latency`, `/metrics/history`.
- Telemetry inputs: Prometheus scrape by client `MetricsPoller`; optional NVML; Prometheus registry inside server.
- Virtualenvs: per-repo `.venv` (client and server); activate matching env before installing/running.

**EVIDENCE:** `llm_bench/cli/parser.py`; `llm_bench/__main__.py`; `batching_scheduler/utils/config.py`; `batching_scheduler/server/main.py`; `llm_bench/telemetry/poller.py`.

## 5) DATA MODEL & METRICS SEMANTICS
- `RequestRecord`: client-side canonical row: prompt/context, token counts, timestamps (`sent_ts`, `first_token_ts`, `completion_ts`), derived `ttft_s`, `total_latency_s`, `tpot_s`, output/error metadata.
- TTFT = `first_token_ts - sent_ts` (client); server histograms also record arrival→first token. TPOT = generation span / (`completion_tokens`-1). Total latency = `completion_ts - sent_ts`. Queue/service time only from server metrics.
- Client metrics: latency percentiles, throughput req/s & tok/s, per-bin summaries.
- Server metrics: Prometheus gauges/histograms for queue length, batch size/latency, request latency, TTFT, TPOT, token throughput, GPU memory/utilization.
- Client percentiles via pandas quantiles; server percentiles interpolated from histograms in exporters/history.
- Distinction: client logs contain no queue/service times; `/metrics` is authoritative for server timings + GPU.

**EVIDENCE:** `llm_bench/loadgen/records.py`; `llm_bench/analysis/metrics.py`; `llm_bench/cli/runner.py`; `batching_scheduler/utils/metrics.py`; `metrics_exporter.py`.

## 6) ARTIFACT MODEL
- Per-run outputs (default `runs/<run_id>`): `requests.jsonl`, `requests.csv`, `per_bin_summary.csv`, `throughput_latency_timeseries.csv`, `server_metrics_timeseries.csv` (if scraped), `summary.json`, `summary.md`, plots (`ttft_vs_prompt_len.png`, `cdf_*`, throughput/latency, KV usage).
- Naming centralized via `artifact_paths` / `ARTIFACTS`.
- Sweep outputs under `sweep_runs/` with aggregated CSV/plots (capacity curve) via sweep orchestration.
- Reports include per-bin tables and plot references; summary JSON carries throughput, percentiles, KV metrics presence.
- Server telemetry available as Prometheus text/JSON/CSV; history endpoint returns JSON arrays with throughput/latency/percentiles metadata.

**EVIDENCE:** `llm_bench/analysis/schema.py`; `llm_bench/analysis/reports.py`; `llm_bench/analysis/plots.py`; `batching_scheduler/server/main.py`.

### 6.1) ARTIFACT STRUCTURE (FILES + CONTENTS)
- Per-run directory layout:
  - `requests.jsonl`: per-request records (schema from `RequestRecord`).
  - `requests.csv`: same as JSONL, columns aligned to `RequestRecord.csv_columns`.
  - `per_bin_summary.csv`: aggregates by prompt length bins.
  - `throughput_latency_timeseries.csv`: windowed throughput/latency pairs.
  - `server_metrics_timeseries.csv`: scraped `/metrics` time-series (if enabled).
  - `summary.json`: aggregate metrics (throughput, percentiles, request counts, KV/cache stats).
  - `summary.md`: human-readable report referencing plots and tables.
  - `plots/`: latency-vs-prompt, CDFs, throughput/latency, KV cache plots.
- Sweep directory layout:
  - `sweep_raw.csv`: per-(λ, repeat) summary rows.
  - `sweep_summary.csv`: mean/std per λ across repeats.
  - `capacity_curve.png`: throughput vs latency curve built from sweep_summary.
  - `lambda{λ}_rep{r}/`: per-run artifacts as above.
  - Sweep raw/summary columns include throughput, total-latency p50/p95/p99, and TTFT/TPOT mean/p50/p95/p99.

**EVIDENCE:** `llm_bench/analysis/schema.py`; `llm_bench/analysis/reports.py`; `llm_bench/analysis/plots.py`; `llm_bench/cli/sweep.py`; `llm_bench/loadgen/records.py`.

## 7) INVARIANTS & ASSUMPTIONS
- Client timestamps are wall-clock/perf_counter; server times independent—do not mix without alignment.
- Concurrency limited only by client semaphore; batching exclusively server-side.
- Workload token counts approximate via whitespace unless usage provided; tokenizer TODO noted.
- Poisson arrival uses exponential inter-arrival times; when `--seed` is set, NumPy RNG is seeded for reproducible timing and random prompt selection per arrival.
- Closed-loop iterates deterministically.
- CSV schema must match `RequestRecord.csv_columns`; analysis assumes required columns exist.
- Custom server assumes HF model fits device; tokenizer pad_token ensured (fallback to eos).
- Naive scheduler batches when size ≥ max_batch_size or timeout since first queued; queue length metric updated each loop.
- Dynamic scheduler adapts batch size/timeout by queue length and oldest request age (monotonic), with a max-wait cap for starvation.
- Metrics history requires `metrics_history_enabled`; history endpoint errors otherwise.
- GPU monitoring is no-op if CUDA unavailable; utilization only when NVML present.
- vLLM proxy/spawn bypasses local scheduler/model; depends on external `/v1/models` health.

**EVIDENCE:** `llm_bench/loadgen/client.py`; `llm_bench/loadgen/arrival_patterns.py`; `llm_bench/loadgen/records.py`; `llm_bench/analysis/schema.py`; `batching_scheduler/server/server.py`; `schedulers/naive.py`; `engine/batch_processor.py`; `utils/gpu_monitor_async.py`; `server/main.py`.

## 8) EXTENSION POINTS
- Workloads: add modes in `loadgen/workload.py` + CLI choices.
- Arrival: new generator in `arrival_patterns.py` and selection in `_execute_load_test`.
- Metrics/plots: extend `analysis/metrics.py`/`plots.py` and wire into runner/report.
- Artifacts: add names in `analysis/schema.py`, persist in `cli/runner.py`.
- Sweeps/orchestration: extend `cli/sweep.py` for new grid/search parameters.
- Server schedulers: implement + register in `schedulers/factory.py`.
- Batch/model handling: evolve `engine/batch_processor.py` / `model_manager.py`.
- Telemetry scraping: broaden `KV_KEYS_CANDIDATES` or server metrics in `utils/metrics.py`; adjust poller parsing.
- vLLM integration: streaming enhancements or spawn args in `vllm_server.py` / `vllm_spawn.py`.

**EVIDENCE:** `loadgen/workload.py`; `arrival_patterns.py`; `analysis/schema.py`; `analysis/plots.py`; `cli/runner.py`; `cli/sweep.py`; `schedulers/factory.py`; `engine/batch_processor.py`; `telemetry/poller.py`; `server/vllm_*`.

## 9) NEXT NATURAL FEATURES (PREDICTED)
- Adaptive scheduler (priority/dynamic) → new scheduler class + factory registration; possible preemption fields on `Request`; tests on batch size/latency under synthetic load; risks: fairness/starvation. (UNCERTAIN priority rule)
- Token-accurate client counting → integrate tokenizer selection (`utils.tokens`), pass model/backend hints; adjust `RequestRecord` usage; validate vs server counts; risks: perf overhead, tokenizer mismatch.
- Arrival trace replay → arrival pattern reading timestamped traces; CLI `--arrival trace --trace-file`; validate timing drift; risks: async drift.
- Server-side per-request IDs through proxy → propagate IDs/latencies from vLLM responses; modify proxy to stream timings; validate against client TTFT; risks: vLLM API drift.
- Autosweep controller → adaptive λ until latency SLA; orchestrate in `cli/sweep.py`; validate via simulations; risks: runtime, convergence stability.
- Expanded metrics history filters → label/window filters in `/metrics/history`; adjust `metrics_history.py`; validate buffer queries; risks: performance.
- Model-specific generation params per workload → prompt metadata controls `max_tokens`/temperature; pass through client/server; validate with goldens; risks: schema bloat.

**EVIDENCE:** `schedulers/factory.py`; `loadgen/client.py`; `loadgen/arrival_patterns.py`; `cli/sweep.py`; `server/vllm_proxy.py`; `metrics_history.py`; `engine/batch_processor.py`.