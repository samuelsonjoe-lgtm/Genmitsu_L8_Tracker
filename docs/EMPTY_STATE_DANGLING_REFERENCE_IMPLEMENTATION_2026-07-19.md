# Empty-state and dangling-reference polish — implementation report

Date: 2026-07-19  
Baseline: `50861bbe48c02919fb30a6c26231aedccba238ce` (`Improve responsive workshop layouts`)

## Scope

This bounded release-readiness pass improves empty result states and the presentation of unavailable related records. It does not repair, recreate, reassign, migrate, or delete stored data.

## Implementation

- Added a shared neutral empty-state renderer with optional existing-workflow actions.
- Added context-appropriate create actions for empty Log, Library, Test Grid, Project, raw Inventory, and finished-batch collections.
- Added clear-filter actions for the corresponding non-empty/filter-miss result lists. These reset only the current in-memory filter values and do not persist a new preference.
- Preserved the existing first-run Log onboarding alongside the ordinary Log empty state.
- Added a defensive ID lookup helper for optional related records.
- Project details now call missing source records **unavailable** and explain that the Project retains its own saved fields and settings snapshot.
- Finished batches with a missing Project now say **Linked Project unavailable**. Their editor keeps the selected missing Project value unless the user explicitly changes it.
- Guarded stale Project-related action targets and stale Test Grid cell selection so they return safely instead of opening an unintended new record or throwing.
- Added a short Help-and-Safety explanation: an unavailable original relation remains historical, may be restored by importing an appropriate backup, and is never recreated or reassigned automatically.

## Protected boundaries

No storage key, schema version, backup format, import/export behavior, merge/replace semantics, automatic cleanup, delete cascade, production SVG, Downloads, Designs geometry, or material/test/promotion evidence semantics changed. Missing relationships remain missing; saved snapshots remain historical context, not restored verification or production-safety proof.

## Fixtures

`runEmptyStateDanglingReferenceFixtures()` adds 60 focused assertions covering empty first-use and filter-miss rendering, primary actions, clear filters, first-run coexistence, safe lookup behavior, missing Project source presentation, missing finished-batch Project options, stale Grid-cell recovery, and storage/backup identity boundaries.

Observed complete-suite arithmetic is `1089 non-Design assertions + 1093 Designs assertions = 2182`, all passing.

## Validation

- `git diff --check`
- `python -m html.parser index.html`
- Direct `file://` Edge startup with `?selftest=all`
- Focused empty-state and dangling-reference fixtures: `60 / 0`
- Existing full suite: `2182 / 0`
- The separate 320px Edge measurement helper timed out before returning viewport values in this environment; the responsive fixture group still passed in the complete file:// suite (`45 / 0`).

## Remaining validation

This is presentation and defensive-rendering polish only. Continue normal real-data backup/restore checks before relying on a browser profile, and continue physical material, fit, and laser-safety validation independently of Tracker record history.
