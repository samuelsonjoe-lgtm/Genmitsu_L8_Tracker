# Release Identity and Import Safety — Implementation Report

## Baseline and initial state

- Actual HEAD: `56bff51 — Add Designs to Projects handoff`.
- Branch: `main`, synchronized with `origin/main` (`0` ahead / `0` behind).
- Initial tracked diff and staging area were clean.
- Existing untracked `.claude/`, `LightBurn Projects/`, `debug.log`, utilities, prior reports, and the release-readiness roadmap report were preserved exactly as found.

## Files changed

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/RELEASE_IDENTITY_IMPORT_SAFETY_IMPLEMENTATION_2026-07-19.md`

## Release identity

Added code-level constants without changing the stored-state schema:

- `APP_ID = 'genmitsu-l8-tracker'`
- `APP_NAME = 'Genmitsu L8 Tracker'`
- `APP_VERSION = '0.9.0'`
- `BUILD_DATE = '2026-07-19'`
- `BACKUP_FORMAT = 'genmitsu-l8-tracker-backup-v1'`

The shell footer renders `Genmitsu L8 Tracker v0.9.0` from those constants and provides a compact native About disclosure with build date, storage schema, backup format, and the offline/local-browser statement.

## Backups and imports

New exports add the top-level `app`, `appVersion`, and `backupFormat` metadata while retaining the existing `schemaVersion` and all prior backup fields. `STORAGE_KEY` remains `genmitsu-l8-tracker-v1`; `SCHEMA_VERSION` remains `2`; no migration or new localStorage key was added.

`validateBackupImport()` is a side-effect-free pre-decision guard. Before the merge/replace prompt, it refuses a numeric future schema version, a present mismatched `app`, or a present unsupported `backupFormat`. Refusals alert the user and return before merge, replace, normalization, persistence, or state mutation. Missing metadata and missing schema version retain legacy behavior. Accepted imports retain the existing merge/replace functions and persistence path.

## Verification

- Storage fixtures now cover release metadata, legacy acceptance, each refusal rule, refusal isolation, and current merge/replace behavior with additive record fields.
- Storage recovery fixtures: `15 / 0`.
- Tray-model fixtures: `264 / 0`.
- Designs production fixtures: `1093 / 0`; production geometry, serializers, downloads, Finished Views, and output bytes were not changed.
- Complete browser suite: `1909 / 0`.
- `git status -sb`, `git log -1 --oneline`, `git diff --check`, `git diff --stat`, `git diff`, `git diff --cached --stat`, branch/ahead-behind checks, and protected-boundary diffs were inspected.
- `git diff --check` and `python -m html.parser index.html` pass.
- The established headless Edge harness ran the focused storage group, direct `file://` startup, release footer/About/duplicate-ID check, and `?selftest=all` complete suite. No dependency or network requirement was introduced.

Remaining unverified areas: no physical cutting is needed because production output is unchanged. Import refusals were fixture-tested at the validation/application seam; no operating-system file-picker accessibility work was added in this bounded phase.

Nothing was staged, committed, or pushed.
