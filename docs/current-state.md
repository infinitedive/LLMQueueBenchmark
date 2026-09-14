# Current Investigation

Updated: 2026-09-14. This is a resumable research snapshot, not a claim that the
listed branches are deployed or that historical experiments used these commits.

## Objective

Restore the educational experiment's ability to measure scheduling policy over
real batched inference. Then determine which historical comparisons remain
interpretable. A faster system alone does not establish a valid policy experiment.

## Evidence and Status

| Finding | Status | Evidence |
|---|---|---|
| The inspected client unconditionally requests streaming. | Verified by source inspection | [Client at 43103e2](https://github.com/Mentiforce-013/llm-benchmark-client/blob/43103e2dba316102f247c374de7158c436b369f5/src/llm_bench/loadgen/client.py) |
| The inspected custom streaming executor processes requests sequentially within a released group. | Verified by source inspection | [Executor at 5cc5597](https://github.com/Mentiforce-013/llm-scheduler-server/blob/5cc5597a87c4f06ab3cdef26e5eaf29700528c37/batching_scheduler/engine/batch_processor.py) |
| Model and inputs are placed on the configured device; default configuration uses CUDA. | Verified configuration/code path, not a runtime profile | [Model loading](https://github.com/Mentiforce-013/llm-scheduler-server/blob/5cc5597a87c4f06ab3cdef26e5eaf29700528c37/batching_scheduler/engine/model_manager.py) |
| A candidate patch sends compatible streaming requests through one batched generate call. | Committed; runtime correctness unverified | [Patch 485dc2e](https://github.com/Mentiforce-013/llm-scheduler-server/commit/485dc2ea4501eb322577c2d15cb6dcd73d7ee3b5) |
| Historical sweeps used the inspected client/server revisions. | Unresolved | Recover run manifests, local commits, or execution records. |
| Serialization explains the exact reported GPU-utilization gap. | Hypothesis, not established | Requires matched execution evidence and profiling. |

The patch branch is `fix/streaming-tensor-batching` in the server repository.
It groups consecutive equal-output-limit requests, routes text by row, and adds
`tests/test_batch_streaming.py`. Tests were not run during patch authoring because
that session had GitHub API access but no Python/GPU execution environment.
Existing non-streaming parameter and metric limitations remain outside that patch.

## Next Discriminating Action

In the intended component environment, run the patch's focused CPU regression
tests, resolve failures, and verify streaming behavior on CUDA with multiple
compatible requests. Capture actual tensor batch dimensions. Do not assume a
particular GPU-utilization percentage; identify the metric and its sampling
semantics before interpreting it.

Before attributing old results, recover the historical code/configuration.
Some KB command references are absent from the inspected client checkout; verify
which local or remote revision contains those tools rather than assuming either
the KB or the inspected checkout is current.

## Interpretation Boundaries

Existing tables and the [claim ledger](operations/ml-sys-paper-claim-ledger.md)
are retained as historical records. They do not yet establish policy-only effects.
Source inspection confirms an execution mismatch in specific revisions; it does
not identify the entire historical bottleneck or show that the candidate fix works.

The [methodology](operations/ml-sys-report-methodology.md) describes the research
contract and expected outputs. The [validity checklist](operations/ml-sys-comparison-validity-checklist.md)
identifies evidence needed for claim acceptance.

## Maintaining This Snapshot

When the investigation advances, replace stale status with dated evidence,
including commit/run identifiers. Keep supported findings distinct from
hypotheses and planned work. Archive lengthy history in plans or research records;
leave the next actionable question here.
