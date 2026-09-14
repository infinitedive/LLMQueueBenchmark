# Experiment State Record

The [JSON template](experiment-state.template.json) records the repository and
execution state associated with one experiment. Store a populated copy as
`experiment-state.json` alongside that experiment's outputs. This is supplemental
provenance; it does not change the existing client-generated metric schemas.
Automatic capture is not implemented by this KB change.

## Capture semantics

Capture from the actual execution checkouts before the run. The parent KB,
client, and server each have independent commits and worktrees. For remote
execution, capture the server's deployed checkout and environment on that host,
not a local checkout assumed to match it.

- `experiment_id`: unique run/experiment identifier matching the output location.
- `captured_at_utc`: ISO 8601 UTC timestamp of capture.
- `record_status`: `template`, `captured`, or `reconstructed`. Reconstruction
  describes historical evidence recovered later, not an observation at run time.
- `repositories.*.commit`: full Git commit SHA, not a mutable branch name.
- `repositories.*.dirty`: `true` for tracked changes or non-ignored untracked
  files, `false` only after inspection, `null` when unknown.
- `changes_ref`: preserved patch plus relevant untracked-file snapshot when dirty.
  A dirty flag alone cannot reconstruct executed code. Capture ignored files
  that affect execution explicitly in configuration or environment evidence.
- `configuration.source`: repository key, repo-relative config path, and its
  commit. This identifies the original specification.
- `executed_snapshot_ref` and `executed_snapshot_sha256`: saved effective config
  including overrides and its SHA-256 digest. A source spec alone may omit
  effective CLI/env/default settings.
- `launch_commands_ref`: exact client/server command record, with effective
  configuration overrides. Omit credentials; identify any consequential redactions.
- `environment`: saved dependency/version records from both environments and
  hardware/runtime details, including model/tokenizer revisions and GPU/driver.
- `artifacts_ref`: outputs to which this record applies.
- `missing_evidence`: explicit unresolved fields or unavailable source records.

Except for `configuration.source.path` and workspace paths, local references are
relative to this record's directory. Durable URLs are also permitted. Preserve
the referenced files with the record. Unknown values remain null and are listed
in missing_evidence; do not substitute current HEAD for an unknown historical SHA.

Useful read-only Git queries, run separately in each actual repository:

```bash
git rev-parse HEAD
git status --porcelain=v1 --untracked-files=all
git diff --binary HEAD
```

The diff covers tracked working-tree changes relative to HEAD, including staged
changes; it does not include untracked-file contents. Capture these separately
when they affect execution. Inspect nested repositories independently even if
the parent ignores their directories.

The KB SHA identifies the guidance used at execution time, before saving this
new record. Do not amend it to the later commit that stores the record: that
would create a circular reference. Prefer storing outputs outside source
checkouts or in established ignored output directories.

Use a separate record when code, runtime configuration, or environment changes.
A sweep can share one record only while that state remains fixed; its effective
configuration must identify the full grid and per-run overrides. Freeze captured
records with their outputs. Record subsequent corrections separately with their
evidence; current-state.md links to these records instead of duplicating them.
