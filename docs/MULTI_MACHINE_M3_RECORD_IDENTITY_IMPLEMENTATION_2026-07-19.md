# M3 — Owned-machine identity for new production-critical records

Date: 2026-07-19  
Baseline: `7a9ec6d Add multi-machine manager and switching`  
Scope: bounded record identity propagation on the existing M1/M2 model.

## Outcome

M3 adds an optional stable `machineId` to new production-critical record snapshots without changing the storage key, schema version, backup format, or historical records. The active owned machine is captured as `{ machineId, key: 'custom', label }`; when no valid owned machine is active, existing legacy behavior remains in place.

Implemented in `index.html`:

- New Grids snapshot the active owned machine; edits retain the stored snapshot.
- Grid-derived Material Tests inherit the Grid snapshot.
- Standalone Material Tests snapshot the active owned machine.
- Production settings created manually use the active owned machine.
- Production settings promoted from a source retain source identity.
- Promoted evidence receives the source `machineId` and does not resnapshot the active machine.
- Project settings preserve the source setting identity through hidden snapshot fields.
- Reference Copy to Library remains unowned and keeps its existing attribution.
- Production promotion matching requires equal IDs when either side has an ID. One-sided IDs do not exact-match; records without IDs retain legacy key matching.
- Existing M2 recursive reference scanning automatically finds record-level IDs and protects referenced machines from deletion.
- Normalization, Merge, Replace, and backup paths retain unknown IDs without fabricating owned machines or backfilling historical records.

## Focused fixture

The new `runMachineRecordIdentityFixtures()` group is registered exactly once for `?selftest=machines-m3` and `?selftest=all`. It covers active snapshots, unnamed fallback, legacy fallback, Grid and Material Test inheritance, production settings, evidence promotion, ID matching, Project settings, all five M2 reference roots, unknown-ID retention, and no historical backfill.

Focused result after the bounded correction pass: **27 passed / 0 failed**.

The correction pass also makes the Test Grid choices explicit: new grids say **Use active workshop machine** and existing grids say **Assign active workshop machine**. The focused fixture now proves isolated one-sided-ID rejection, same-name/different-ID rejection, same-ID/renamed-label acceptance, source identity frozen across an active-machine switch during promotion, Project hidden-field edit/save round-tripping, unowned Reference Copy behavior, and per-record-family deletion protection. No M3 identity semantics were changed.

## Validation

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Direct `file://` Edge/Playwright run of `?selftest=machines-m3`: **27 / 0**, no page errors.
- Direct `file://` Edge/Playwright runs of `?selftest=machines-m1` and `?selftest=machines-m2`: no page errors; M1 remains **29 / 0** and M2 remains **31 / 0**.
- Direct `file://` Edge/Playwright run of `?selftest=all`: **2293 / 0** by the established direct-suite arithmetic.
- Complete-suite arithmetic: `1200 non-Design assertions + 1093 Designs assertions = 2293`.
- Machine compatibility remains **50 / 0**; modal accessibility remains **28 / 0**; the standalone structured Tray-model group remains **264 / 0**; the established promotion-switch standalone result remains **16 / 0**.
- Constants remain unchanged: `APP_ID='genmitsu-l8-tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, and `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.

The browser emitted pre-existing SVG attribute warnings (`height="auto"`) during the broad route run; they did not produce page exceptions or fixture failures and are outside M3 scope.

## Files changed

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_IMPLEMENTATION_2026-07-19.md`

No unrelated files were staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.
