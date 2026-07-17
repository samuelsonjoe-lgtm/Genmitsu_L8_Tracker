# Test Grid Machine-Identity Enhancement

Date: 2026-07-17  
Status: **READY FOR FOCUSED AUDIT**

## Files changed

- `index.html`
- `README.md`
- `docs/TEST_GRID_MACHINE_IDENTITY_IMPLEMENTATION_2026-07-17.md`

No unrelated files were changed. No commit or push was performed.

## Implementation

### Grid machine data shape

New and explicitly assigned Grids store the additive optional snapshot:

```js
machine: {
  key: 'genmitsu-l8-20w',
  label: 'Genmitsu L8 20W'
}
```

`gridMachineSnapshot()` uses the existing canonical production machine keys. L8 20W and L8 40W use their canonical labels; custom uses the explicit `custom` key and supplied label. Older records without `machine` remain machine-less.

### Automatic and override behavior

`openGridForm()` now snapshots the active machine for a new Grid and shows a compact read-only summary. Ordinary creation defaults to the active machine without an extra selection step. The Advanced machine control supports explicit L8 20W, L8 40W, or custom selection.

Existing Grids show their recorded machine and keep it during unrelated edits. A machine-less Grid shows `Machine: Not recorded`; `Assign active machine` is an explicit reviewed action and affects only that Grid. Machine assignment preserves cell values, ratings, notes, dates, IDs, and unknown fields. The global machine preference never rewrites stored Grid snapshots.

Machine identity is displayed in Grid cards, Grid detail, Browse detail, and the selected-cell result modal. Internal keys are not shown in ordinary UI text.

### Promotion and Material Test bridge

`promoteSelectedGridCell()` and the existing Grid promotion candidate path use only `grid.machine`. Stored machine identity therefore reaches the Phase 7.3A review and enables machine-dependent values for newly identified Grids. Machine-less Grids retain the existing missing-machine warning and do not silently use the active preference.

The Grid-to-Material-Test bridge now receives the stored machine snapshot and records the existing machine label plus additive `machineKey` context. A machine-less Grid is stopped with a clear assignment message before the bridge saves a cell or substitutes a machine.

### Backward compatibility

The machine field is optional. `STORAGE_KEY`, `SCHEMA_VERSION`, storage, backup, replace, and merge functions were not changed. Existing raw Grid records and unknown fields continue through the existing additive import paths; no migration or bulk backfill was added.

## Functions

Added:

- `gridMachineSnapshot()`
- `activeMachineSnapshot()`
- `gridMachineLabel()`
- `normalizeTestGrid()`
- `gridMachineFromForm()`
- `gridMachineControls()`
- `runTestGridMachineIdentityFixtures()`

Modified:

- `openGridForm()`
- `gridCard()`
- `gridBrowserDetailHtml()`
- `bindGridResultActions()`
- `renderGridDetail()`
- `openCell()`
- `buildGridCellMaterialTestRecord()`
- `promoteCell()`
- self-test routing and README fixture documentation

## Fixtures and observed totals

The new `runTestGridMachineIdentityFixtures()` contains **18 passed / 0 failed** checks covering:

- active L8 20W and L8 40W creation;
- explicit custom identity;
- global preference changes;
- unrelated edit preservation;
- selected-only machine changes;
- old machine-less display and explicit assignment;
- preservation of cell and unknown data;
- stored-machine selected-cell promotion;
- promotion review source immutability;
- stored-machine Grid-to-Material-Test context;
- machine-less bridge blocking;
- Replace and Merge retention;
- unknown-field retention.

The final isolated Chromium `?selftest=all` run reported **1379 passed / 0 failed** across the complete deduplicated suite. The individual groups included 58 Evidence Promotion, 66 Production Settings, 18 Test Grid machine identity, 23 Grid Promotion, 67 Grid Browser, 12 Material Test, 57 Material Browser, 56 Library Browser, 61 Project Browser, 216 Project Wizard, 118 Designs production application, 587 Designs geometry, 20 baseline, 12 normalization, 12 metadata, and 8 storage checks. Chromium still emits the pre-existing Finished Front View `height="auto"` SVG warning; no fixture failed.

## Browser and validation work

Executed from `C:\Genmitsu L8 Tracker`:

- `git status -sb`, `git log -1 --oneline`, `git diff --check`, and `git diff --stat` before editing.
- `python -m html.parser index.html` — passed.
- `git diff --check` after implementation — passed.
- Direct `file://` startup in a fresh isolated Chromium context — loaded.
- Fresh Chromium form paths for new 20W, new 40W, custom override, existing edit, old-Grid assignment, selected-cell promotion review, and machine-less bridge blocking — passed through the focused fixture.
- `?selftest=grid-machine` — 18 passed / 0 failed.
- `?selftest=all` — 1379 passed / 0 failed.

## Protected boundaries

Compared against baseline commit `4dd7a6b`, these function bodies are unchanged:

`persist`, `backupObject`, `replaceData`, `mergeData`, `normalizeProductionEvidence`, `normalizeProductionSetting`, `normalizeProductionSettings`, `buildDesignResult`, `buildJointCouponModel`, `buildDrawerCabinetModel`, `buildDrawerCabinetDesignResult`, `serializeDesignSvg`, and `downloadTextFile`.

No schema/storage key change, geometry/SVG change, migration, bulk backfill, or machine inference from Grid names, notes, material, power, or preference was introduced.

## Unverified areas

- Physical machine operation and production safety were not tested; the snapshot records provenance only.
- A global custom active-machine profile is not part of the existing active-machine selector, so custom identity was verified through the explicit Grid override supported by this implementation.
- Native browser file-picker interaction for manual backup import was not automated; Replace and Merge retention were exercised through their existing functions and fixture data.
