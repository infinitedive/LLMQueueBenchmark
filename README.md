# LLMQueueBenchmark

This workspace hosts a two-repository benchmark system for LLM serving:

- `llm-scheduler-server`: inference server with batching/scheduling runtimes.
- `llm-benchmark-client`: load generator, telemetry correlation, and artifact pipeline.

The canonical docs live under `docs/`:

- Start here: `docs/index.md`
- Local runbook: `docs/operations/local-dev.md`
- Benchmark recipes: `docs/operations/benchmark-recipes.md`

## Quick Demo Path

1. Start the server from `llm-scheduler-server`.
2. Run a short sweep from `llm-benchmark-client`.
3. Confirm aggregate outputs:
   - `sweep_raw.csv`
   - `sweep_summary.csv`
   - `capacity_curve.png`

Use the exact copy/paste commands in `docs/operations/local-dev.md` under the recruiter/hiring manager quick path section.

## Workspace Note

Each component repo has its own Git history and packaging metadata.
