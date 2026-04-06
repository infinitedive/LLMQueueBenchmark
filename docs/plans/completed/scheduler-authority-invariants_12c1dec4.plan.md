---
name: scheduler-authority-invariants
overview: Restore scheduler-controlled batch-release timing and prevent low-load singleton dynamic dispatch, then document these as canonical scheduler invariants in the project knowledge base.
todos:
  - id: scheduler-authority-api
    content: Add scheduler timing/deadline API in base scheduler and implement in naive/dynamic schedulers
    status: pending
  - id: server-loop-authority-shift
    content: Refactor BatchingServer processing loop to wait on scheduler-derived timing and new-request wake events
    status: pending
  - id: dynamic-lowload-guard
    content: Enforce non-singleton low-tier dynamic behavior and update config defaults/validation
    status: pending
  - id: tests-refresh
    content: Update/add scheduler-focused tests for timing authority and low-load accumulation invariants
    status: pending
  - id: kb-invariants-update
    content: Document invariants in server/dependency architecture docs and add AGENTS.md pointer
    status: pending
isProject: false
---

# Restore Scheduler Authority And KB Invariants

## Scope
Implement the two fixes in dependency order inside `llm-scheduler-server`, then update canonical docs so the invariants are explicit and enforceable.

## Changes
- **1) Restore scheduler timing authority (first)**
  - Update scheduler interface in [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/base.py`] to expose scheduler-owned wake/deadline information (non-blocking).
  - Implement wake/deadline logic in [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/naive.py`] and [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/dynamic.py`].
  - Replace fixed `await asyncio.sleep(0.01)` poll cadence in [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/server/server.py`] with scheduler-derived wait plus immediate wake on new request.
  - Preserve existing processing semantics and metrics emission (`BATCH_SIZE`, `DYNAMIC_RELEASE_REASON`, `batch_completed`) unless explicitly required by tests.

- **2) Prevent low-load singleton dynamic dispatch (second)**
  - Enforce effective low-tier dynamic batch target `>= 2` in [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/schedulers/dynamic.py`].
  - Update defaults/validation in [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/batching_scheduler/utils/config.py`] so `dynamic_bs_min` cannot silently degrade to singleton behavior for low tier.
  - Keep timeout/max-wait escape hatches so sparse arrivals still complete boundedly.

- **3) Tests and verification**
  - Update unit coverage in [`/home/garuc/projects/LLMQueueBenchmark/llm-scheduler-server/tests/test_scheduler.py`] for:
    - scheduler-timed release checks (not fixed server poll cadence),
    - dynamic low-tier policy mapping and non-singleton accumulation behavior,
    - timeout/max-wait bounded completion behavior.
  - Adjust related tests only if behavior contracts changed (`test_server.py`, `test_async_monitoring.py`, `test_metrics.py`).
  - Run scheduler/server test subset and lint diagnostics on touched files.

- **4) Add invariants back into KB**
  - Add canonical scheduler invariants to [`/home/garuc/projects/LLMQueueBenchmark/docs/architecture/server.md`], e.g.:
    - scheduler owns batch-release timing authority,
    - dynamic low-load policy must preserve nonzero accumulation opportunity before sparse release.
  - Add/align boundary wording in [`/home/garuc/projects/LLMQueueBenchmark/docs/architecture/dependency-boundaries.md`] to prevent timing authority from drifting into server poll mechanics.
  - Update entry-point discoverability in [`/home/garuc/projects/LLMQueueBenchmark/AGENTS.md`] (brief pointer only) so the invariant source of truth is easy to find.

## Acceptance Criteria
- No fixed server poll interval acts as release-time authority; scheduler deadlines/events drive release checks.
- Dynamic low-tier policy cannot immediately self-release singletons due to `bs_min=1` default/config.
- Existing API contracts remain unchanged (`/v1/completions`, metrics endpoints).
- Tests cover both invariants and pass for touched areas.
- KB docs explicitly encode these invariants in canonical architecture docs.