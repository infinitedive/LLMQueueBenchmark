# Symphony Orchestration Runbook

This workspace uses OpenAI Symphony as an opt-in dispatcher over GitHub Issues.
Only open issues carrying the `symphony` label are eligible for autonomous work.

## Architecture

- Tracker/control plane: GitHub Issues in `infinitedive/LLMQueueBenchmark`.
- Worker runtime: Codex App Server launched by Symphony.
- Workspace isolation: one Symphony workspace per issue under `~/symphony-workspaces/LLMQueueBenchmark`.
- Parent workspace repo: `infinitedive/LLMQueueBenchmark`.
- Component repos cloned inside each issue workspace:
  - `Mentiforce-013/llm-benchmark-client`
  - `Mentiforce-013/llm-scheduler-server`
- Repository guidance: `AGENTS.md` plus task-specific docs.
- Current research status: `docs/current-state.md`.

`WORKFLOW.md` is the executable orchestration contract.

## Windows host

The current Symphony reference binaries target macOS and Linux. On Windows, run Symphony inside WSL2.
The repository workspaces can also live inside the WSL filesystem to avoid cross-filesystem Git and file-watcher overhead.

## Prerequisites inside WSL

Required commands:

```bash
git --version
codex --version
```

Install the Symphony reference implementation using the upstream instructions. The reference implementation recommends `mise` for Erlang/Elixir version management:

```bash
git clone https://github.com/openai/symphony.git ~/src/symphony
cd ~/src/symphony/elixir
mise trust
mise install
mise exec -- mix setup
mise exec -- mix build
```

Codex must already be authenticated on the WSL host before Symphony launches workers.

## GitHub tracker credential

Create a GitHub token with the minimum repository access needed for Symphony to read and update issues in `infinitedive/LLMQueueBenchmark`, then expose it only to the Symphony host process:

```bash
export GITHUB_TOKEN='...'
```

Do not commit the token or place a literal token in `WORKFLOW.md`.
Symphony executes GitHub tracker operations host-side and does not pass the configured tracker token into the Codex child process.

Code pushes are therefore a separate concern. Configure normal Git/SSH or GitHub CLI authentication inside WSL for the repositories that workers may push to:

```bash
gh auth status
# or verify SSH authentication
ssh -T git@github.com
```

The authenticated account must have push access to any component repository a worker is expected to modify.

## First run

From a checkout of this repository on the `setup/symphony` branch (or after this setup is merged):

```bash
cd ~/src/symphony/elixir
mise exec -- ./bin/symphony /path/to/LLMQueueBenchmark/WORKFLOW.md --port 4000
```

Open the dashboard at:

```text
http://localhost:4000/
```

The JSON state endpoint is available at `/api/v1/state`.

## Opt-in dispatch

Create the repository label `symphony` once in GitHub.

An issue is dispatchable only when all of the following are true:

1. It is an issue in `infinitedive/LLMQueueBenchmark` (not a pull request).
2. It is open.
3. It carries the `symphony` label.
4. Symphony has not already claimed it beyond the configured concurrency limit.

Removing the `symphony` label prevents new dispatch and is the normal handoff mechanism when a worker needs human review.
Closing the issue is terminal and causes Symphony to stop/clean up work for that issue.

## Recommended issue shape

Use issues as bounded work packets rather than broad project themes. Include:

- objective or question to resolve;
- evidence/context pointers;
- owned repository if known;
- constraints/invariants;
- acceptance criteria;
- tests or experiment expected;
- explicit hardware requirement if a GPU run is needed.

Example:

```md
## Objective
Determine whether the custom streaming executor still serializes requests after batch release.

## Evidence
See `docs/current-state.md` and the current server `batch_processor.py`.

## Acceptance criteria
- Trace the execution path from scheduler release to model generation.
- Add or update a focused regression test if the defect is reproducible.
- Do not alter scheduling policy while testing execution semantics.
- Report what was actually run; GPU behavior requires observed CUDA evidence.
```

Add the `symphony` label only when the work packet is ready to execute autonomously.

## Concurrency

The initial configuration permits 3 concurrent agents. Keep this deliberately small while validating:

- repository bootstrap cost;
- Codex authentication;
- branch/push behavior across all three repositories;
- issue comments and label handoff;
- resource contention for CPU/GPU experiments.

Increase concurrency only after isolated jobs complete cleanly.

## GPU work

Do not treat the local WSL Symphony host as automatically GPU-capable. Issues requiring CUDA should state where execution is expected to occur. A worker that cannot access the required GPU must record the blocker rather than substituting an unobserved claim.

Long benchmark sweeps should remain explicit work items with artifact locations and validity criteria rather than implicit steps in unrelated coding issues.

## Branch convention

Workers use `symphony/GH-<issue-number>` in every repository they modify. Because the parent and component repos have independent Git histories, a cross-repository issue may produce multiple branches and PRs. The issue is the coordination record tying them together.

## Safe rollout

1. Start Symphony with no `symphony`-labelled issues and confirm the dashboard is healthy.
2. Create a small documentation-only smoke-test issue and add `symphony`.
3. Confirm workspace creation, issue comment, branch handling, and human-review handoff.
4. Run one focused component-code issue that has CPU-only tests.
5. Only then dispatch GPU/benchmark work or raise concurrency.
