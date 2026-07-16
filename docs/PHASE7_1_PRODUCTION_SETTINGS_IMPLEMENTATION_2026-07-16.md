# Phase 7.1 Proven Production Settings Implementation

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `c93ebd1` — `Add production joint-fit tuning`

## Result

Phase 7.1 is implemented as an additive, backward-compatible Library feature. Library profiles can now store typed `productionSettings` records with stable nested evidence IDs, manual lifecycle controls, signed kerf and fit values, machine/material context, and backup/import support.

Phase 7.2 was not implemented. Designs does not select, apply, or write production settings.

Final readiness: **READY FOR INDEPENDENT AUDIT**

## Repository state before editing

The tracked working tree was clean at `c93ebd1`.

Pre-existing unrelated untracked content included `.claude/`, `LightBurn Projects/`, `parametric_qr_stand_generator.py`, the Phase 7 architecture review, and multiple older reports under `docs/`. Those files were preserved and were not staged, moved, deleted, or modified.

Initial checks:

- `git status -sb`: `main...origin/main`, tracked files clean.
- `git diff --check`: passed.
- `git diff --stat`: empty for tracked files.
- `git diff --cached --name-only`: empty.

## Files changed

- `index.html`
- `README.md`
- `docs/PHASE7_1_PRODUCTION_SETTINGS_IMPLEMENTATION_2026-07-16.md`

No unrelated tracked file changed.

## Persisted model

Every normalized Library profile now has:

```javascript
profile.productionSettings = []
```

Production records contain:

- stable record ID;
- label and cut operation;
- machine key, readable label, lens, and focus method;
- nominal/measured thickness in canonical millimeters;
- measurement scope/date and optional Inventory/supplier/product/batch snapshots;
- cut speed, min/max power, passes, air assist/pressure, focus, and software;
- signed LightBurn kerf offset;
- independent finger-joint and Drawer Cabinet lateral clearances;
- verification status, preferred flag, confidence, verified date, and notes;
- stable-ID evidence entries;
- superseding link and created/updated dates.

Known enums are offered by the UI. Unknown future string values are preserved and shown as unrecognized values rather than silently discarded.

## Normalization and numeric safety

Added:

- `strictOptionalNumber()`
- `productionKnownValue()`
- `productionMachineDefaultLabel()`
- `normalizeProductionEvidence()`
- `normalizeProductionSetting()`
- `normalizeProductionSettings()`

`strictOptionalNumber()` keeps blank values blank, preserves numeric zero and negative values, and rejects malformed, `NaN`, and infinite values as blank rather than converting them to zero.

The normalizers are pure with respect to source objects and arrays. They preserve unknown top-level and nested fields, retain valid IDs, create missing IDs once in the normalized in-memory record, and remain stable when normalizing an already normalized record.

`normalizeLibraryProfiles()` now normalizes both:

```javascript
tests
productionSettings
```

Old profiles without the new field receive an empty array.

## Library editor preservation

Added:

- `buildLibraryProfileRecord()`
- `saveProfileRecord()`
- `profileDraftProductionSettings`
- `productionEvidenceDraft`

The existing profile form now overlays recognized form values onto the previous profile object and then attaches normalized Material Test and production-setting drafts. This prevents ordinary edits from erasing:

- Material Tests;
- production settings;
- evidence;
- unknown future profile fields;
- profile or nested IDs;
- notes and tags.

`saveProfileRecord()` retains the existing storage-failure rollback behavior. A failed persist restores the previous profile or removes the unsaved new record.

Canceling the profile or production-setting modal discards only the session draft and does not write persisted data.

## Library UI

The existing material editor now contains a collapsed section after Material Tests:

```text
Proven Production Settings
```

It shows:

- total saved count;
- preferred count;
- record label;
- machine and operation;
- measured thickness and scope;
- verification badge/date;
- speed, power, passes, and air assist;
- signed LightBurn kerf offset;
- finger-joint clearance;
- Drawer Cabinet total lateral clearance;
- expandable evidence summary;
- Add, Edit, Supersede, and Delete actions.

Manage cards and Library Browse detail show a concise preferred-setting summary. Existing `Best Known` Material Test status behavior was not changed.

## Production-setting editor

The bounded editor includes:

- Identity;
- Material condition;
- Cut settings;
- LightBurn kerf reference;
- independent fit settings;
- Verification;
- manual evidence.

The operation is limited to Cut in Phase 7.1.

Known machines are:

- Genmitsu L8 20W;
- Genmitsu L8 40W;
- Custom.

A custom machine requires a readable label. Selecting a known machine fills its standard label. Selecting an Inventory item can copy a name and supplier snapshot without creating a persistent foreign-key dependency.

The LightBurn help explicitly states that signed kerf is for the red through-cut layer, preserves the entered sign, requires LightBurn Preview verification, is not applied to blue score/engrave geometry, and is not baked into generated SVG dimensions.

## Evidence editing

Manual evidence entries support:

- stable ID;
- known or future evidence kind;
- source ID;
- source profile ID;
- source snapshot label;
- date;
- result;
- notes.

Adding, editing, or deleting evidence changes only the production-setting draft. It does not mutate a Material Test, Test Grid, Project, coupon, or other source.

Readable source labels remain in the production record if the source object is later deleted.

## Preferred and superseded behavior

`upsertProductionSettingDraft()` detects active preferred records with the same profile, operation, and machine key.

When a conflicting preferred record exists, the UI explicitly asks whether to demote the old preferred record. Declining cancels the update. Accepting changes only the conflicting preferred flag; the old record and evidence remain stored.

`supersedeProductionSettingDraft()`:

- keeps the old record;
- changes its status to `superseded`;
- clears its preferred flag;
- records `supersededById`;
- retains all evidence;
- leaves the replacement active.

Delete removes only the selected nested record after confirmation.

## Duplicate-profile behavior

Added `duplicateProductionSettings()`.

Duplicating a Library material:

- creates a new parent profile ID when saved;
- creates new production-setting IDs;
- creates new evidence IDs;
- preserves setting values and source snapshots;
- clears preferred flags;
- changes copied verification status to `estimated`;
- clears superseding links;
- adds a re-verification note;
- leaves the original profile unchanged.

Existing Material Test duplication behavior remains unchanged.

## Merge and replace import

Added:

- `mergeNestedRecordsById()`
- `mergeProductionSettingRecord()`
- `mergeLibraryProfileRecord()`
- `mergeLibraryProfiles()`

Library merge import now:

- merges profiles by profile ID;
- merges Material Tests by test ID;
- merges production settings by production-setting ID;
- merges evidence by evidence ID;
- keeps every distinct local or incoming nested ID;
- uses incoming-wins behavior for direct same-field conflicts;
- preserves nonconflicting unknown fields from both sides;
- produces deterministic ID ordering independent of incoming profile order.

The parent profile still counts as updated in existing merge statistics.

Replace import retains its original whole-collection meaning and normalizes nested production records. Malformed nested entries are omitted without discarding their valid parent profile.

## Backup and old-backup compatibility

No new top-level backup collection was added. Production settings are nested under profiles and therefore flow through the existing `backupObject()`.

Verified behavior:

- old profiles/backups without `productionSettings` load with `[]`;
- nested records and evidence survive JSON serialization and normalization;
- valid IDs survive round-trip;
- zero, positive, and negative signed values survive;
- unknown future fields survive;
- malformed nested records do not invalidate the parent profile;
- replace and merge paths normalize the new data.

## Unit behavior

Production material dimensions, kerf, and fit values are stored canonically in millimeters.

`convertStoredUnits()` was not extended to production settings. Repeated mm/in display-unit switches leave historical production evidence byte-for-byte unchanged.

Existing Library thickness/focus/Z-offset conversion behavior remains unchanged.

## Fixture coverage

Added `runProductionSettingsFixtures()` and:

```text
index.html?selftest=production
```

The production-settings group contains 47 assertions covering:

- old/malformed records;
- strict optional numerics;
- ID stability;
- source non-mutation;
- unknown fields and enum values;
- profile edit preservation;
- persist rollback;
- preferred demotion;
- superseding;
- deletion isolation;
- duplicate downgrade and new IDs;
- multiple machines and batches;
- deterministic ordering;
- nested merge semantics;
- replace and merge import paths;
- backup round-trip;
- unit stability;
- unchanged storage key/schema;
- unchanged Library Best Known semantics.

## Complete fixture totals

| Group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material Test normalization | 12 | 0 |
| Production settings | 47 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs geometry | 559 | 0 |
| **Total** | **1138** | **0** |

The prior 1091 assertions remain green.

## Browser validation

Microsoft Edge headless validation used direct `file://` operation.

Verified through the actual UI:

- normal direct-file startup;
- tabs and Library visible;
- production summary visible on Library cards;
- production editor visible and reopenable;
- two production settings saved on one profile;
- one 20W sheet-scoped verified/preferred record;
- one 40W batch-scoped estimated record;
- one manual Joint Coupon evidence entry;
- signed `+0.095`, `-0.02`, and numeric zero values persisted;
- Material Test, notes, and tags survived a production-setting edit;
- one setting was explicitly superseded by the other;
- duplicated records received new IDs and were downgraded to nonpreferred estimated records;
- deleting a draft setting and canceling the profile modal left persisted data unchanged;
- repeated unit switching did not alter nested evidence.

Manual-verification record counts:

- original material: 2 production settings;
- duplicated material: 2 copied/downgraded production settings;
- total created in the isolated browser harness: 4 records.

The `file://...index.html?selftest=all` page also completed startup.

## Other validation

- `git diff --check`: passed.
- Python HTML parser: passed.
- JavaScript runtime: passed through Edge startup and fixture execution.
- Static external-dependency scan: no external runtime script, stylesheet, `fetch`, or XHR dependency found; only SVG namespace strings matched `https`.
- `STORAGE_KEY`: unchanged at `genmitsu-l8-tracker-v1`.
- `SCHEMA_VERSION`: unchanged at `2`.
- No migration file or top-level state collection added.
- No localStorage key added.
- No Project Wizard classification helper changed.
- No Designs geometry/default/application behavior changed.
- No Material Test field meaning changed.

## README changes

README now documents:

- optional proven production settings on Library materials;
- material/machine-specific evidence rather than universal defaults;
- manual verification/status and preferred/superseded history;
- signed LightBurn kerf as red through-cut-layer reference data;
- separate joint and Drawer Cabinet fit fields;
- JSON backup inclusion and old-backup compatibility;
- Designs does not consume the records yet;
- the `production` self-test group and 1138/0 total.

## Unverified areas

- No physical laser or plywood test was performed.
- The native visible Edge UI was not operated by a person; browser interaction used the local headless Edge harness.
- The operating-system file picker was not manually used for JSON import/export. Replace, merge, backup round-trip, and malformed-record behavior were exercised through the same implementation functions and browser fixtures.
- No older application binary/version was used to open a newer backup. As documented in the architecture review, older versions may preserve unknown fields during passive round-trip but can lose them if an old profile editor reconstructs the record.

## Scope confirmation

Not implemented:

- Designs profile selection or application;
- SVG changes based on profiles;
- Joint Fit Coupon promotion;
- Material Test promotion;
- Project evidence promotion;
- automatic physical winner detection;
- machine management;
- raw-sheet/batch entities;
- sheet-aware layout;
- schema/storage-key changes;
- application/repository renaming.

No file was staged, committed, or pushed.

**READY FOR INDEPENDENT AUDIT**
