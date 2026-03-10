# Release Checklist

Scope: Documentation and contract checks for behavior-affecting changes.

## 1) Boundary and Contract Review

- Confirm whether change affects any interface boundary:
  - client CLI,
  - server CLI/env,
  - HTTP API,
  - artifact schemas,
  - telemetry schemas.
- If yes, update corresponding docs in `docs/interfaces/*` and/or `docs/data-models/*`.

## 2) Architecture and Principle Review

- If layering/domain ownership changed, update `ARCHITECTURE.md`.
- If ownership/routing rules changed, update `AGENTS.md` and `docs/index.md`.
- If semantic rules changed, update relevant `docs/principles/*`.

## 3) Artifact and Metrics Integrity

- Verify benchmark artifacts still conform to documented structure.
- Verify metric naming/units/ownership remain consistent with docs.
- Confirm telemetry optionality and mode-specific behaviors are still documented.

## 4) Validation Pass

- Run representative local benchmark(s) and verify expected outputs.
- Check for obvious regressions in summary and time-series artifacts.
- Validate endpoint and startup command examples remain usable.

## 5) De-duplication Check

- Ensure no duplicated contract definitions across:
  - top-level docs,
  - interfaces,
  - data models,
  - principles.
- Keep canonical definition in one location and convert others to links/references.

## 6) Legacy and Migration Hygiene

- If old docs are superseded, ensure they clearly indicate archival/redirect status.
- Keep `docs/index.md` routes complete and current.

## Sign-off

- [ ] Contract docs updated
- [ ] Architecture/principles updated where required
- [ ] Artifacts and telemetry verified
- [ ] De-duplication pass complete
- [ ] Index/routes current
