# Phase 7.1 Production Settings Cleanup

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `c93ebd1` — Add production joint-fit tuning  
Scope: narrowly bounded cleanup of the existing uncommitted Phase 7.1 implementation

## Repository state before editing

The requested inspection was completed before changes:

- `git status -sb`: branch `main` tracking `origin/main`; `index.html` and `README.md` were already modified; the Phase 7 reports and many unrelated files were untracked.
- `git diff --check`: passed, with only the repository's existing LF-to-CRLF warnings.
- `git diff --stat`: `README.md` had 14 changed lines and `index.html` had 530 changed lines relative to `c93ebd1`.
- `git diff --cached --name-only`: empty.
- The complete current diff from `c93ebd1` and every helper named in the cleanup brief were inspected.

No existing changes or unrelated untracked files were reset, cleaned, stashed, deleted, moved, staged, committed, or pushed.

## Cleanup changes

### Persisted numeric bounds

Added a production-setting-specific bounded parser layered on top of the existing `strictOptionalNumber()` behavior.

The following fields now normalize as required:

- `powerMinPercent`: blank or 0–100;
- `powerMaxPercent`: blank or 0–100;
- `nominalThicknessMm`: blank or at least 0;
- `measuredThicknessMm`: blank or at least 0.

Blank values remain blank, zero remains numeric zero, malformed/non-finite values become blank, and signed kerf and fit values remain unchanged. Exponent notation remains accepted by `strictOptionalNumber()` as explicitly deferred.

If both persisted power values exist and minimum exceeds maximum, both invalid range values normalize to blank rather than being silently swapped. The production editor separately displays a blocking validation message before form normalization, so a user-entered inverted range is not silently discarded or rearranged.

### Supersede compatibility

Added one shared compatibility predicate used by both:

- `openSupersedeProductionSetting()`;
- `supersedeProductionSettingDraft()`.

A replacement must:

- exist;
- differ from the current record;
- not already be superseded;
- have the same operation;
- have the same `machine.key`.

Invalid and stale actions return `null` without mutating either source record.

### Dangling Inventory links

The Inventory selector now retains a missing stored ID as an explicit selected option:

```text
Unavailable: <snapshot name>
```

The editor also shows:

```text
Linked Inventory item is no longer available - retained reference: <snapshot name>.
```

Saving an unrelated edit preserves:

- `inventoryItemId`;
- `inventoryItemName`;
- supplier snapshot;
- product snapshot;
- other material-condition snapshots.

Choosing `No Inventory link` explicitly clears only the ID. Selecting a valid Inventory record replaces the ID and refreshes the intended name, supplier, and available product snapshots. No Inventory record is recreated or modified.

### Preferred-conflict prompt

The preferred-record applicability key remains unchanged: same parent profile, operation, and machine key.

The confirmation message now identifies every conflicting preferred record using available:

- label;
- measurement scope;
- batch label;
- measured thickness;
- machine.

Declining still returns without mutation. Accepting demotes only the identified matching conflict records; records on other machines remain untouched.

### Nested merge correction and fixture

The requested adversarial merge fixture exposed a real edge case: both sides were normalized before merging, so defaults inserted for absent incoming nested fields could overwrite valid local values.

The Library merge preparation now:

1. retains raw valid production-setting subobjects through ID matching;
2. merges matching nested subobjects and evidence by ID;
3. applies incoming-wins only to direct conflicts;
4. normalizes the completed merged profile once afterward.

The regression fixture exercises `mergeLibraryProfiles()` and confirms:

- local `materialCondition.supplier` survives;
- incoming `materialCondition.product` survives;
- local and incoming unknown machine fields survive;
- local and incoming unknown cut-setting fields survive;
- direct conflicts use incoming-wins;
- local and incoming evidence IDs survive;
- matching evidence conflicts use incoming-wins;
- source objects remain unmutated.

## Fixture coverage

The production-settings fixture group increased from 47 to 66 assertions.

New coverage includes:

- 0 and 100 power acceptance;
- out-of-range power rejection;
- negative thickness rejection;
- zero thickness preservation;
- inverted power-range normalization and editor validation;
- signed kerf/fit preservation;
- preferred-conflict prompt context;
- valid supersede compatibility;
- missing, self, cross-machine, cross-operation, and already-superseded rejection;
- rejected supersede non-mutation;
- dangling Inventory ID and snapshot preservation;
- explicit link clearing;
- valid Inventory replacement;
- visible unavailable Inventory option;
- adversarial nested merge preservation and non-mutation.

## Validation results

### Static validation

- `git diff --check`: passed; only LF-to-CRLF warnings were emitted.
- Python `html.parser`: passed.
- JavaScript runtime/startup: passed through Microsoft Edge with no page errors.
- `STORAGE_KEY`: unchanged at `genmitsu-l8-tracker-v1`.
- `SCHEMA_VERSION`: unchanged at `2`.
- No external runtime dependency was added.
- No staged files were present.

### Direct `file://` browser validation

Microsoft Edge opened:

```text
file:///C:/Genmitsu%20L8%20Tracker/index.html
```

Results:

- page title: `Genmitsu L8 Tracker`;
- application content visible;
- no page errors;
- normal startup did not require a server;
- `?selftest=all` also rendered normally with no page errors.

### Fixture totals

| Group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material Test normalization | 12 | 0 |
| Production settings | 66 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs geometry | 559 | 0 |
| **Complete suite** | **1157** | **0** |

### Focused browser checks

A direct-file browser scenario loaded a Library profile whose production setting referenced a deleted Inventory item. After opening the editor, changing only the setting label, saving the setting, and saving the profile:

- the unavailable-link notice was visible;
- the selector retained `deleted-inventory`;
- the saved `inventoryItemId` remained `deleted-inventory`;
- the snapshot name remained `Deleted basswood sheet`;
- supplier remained `Old supplier`;
- product remained `Old product`;
- no page errors occurred.

### Regression boundaries

- Profile edit-preservation fixtures remain green.
- Nested merge-import and replace-import fixtures remain green.
- Backup round-trip fixtures remain green.
- Storage recovery remains 8/0.
- Supersede compatibility fixtures are green.
- Designs geometry remains 559/0.
- The cleanup did not alter Designs, Material Tests, Project Wizard, SVG generation, storage keys, schema, backup structure, or localStorage behavior outside the bounded production-setting paths.

## Intentionally unchanged

Per the cleanup brief, this pass did not change:

- exponent notation accepted by `strictOptionalNumber()`;
- evidence `sourceProfileId` behavior during profile duplication;
- preferred-record applicability beyond operation plus machine;
- Designs integration;
- Material Test promotion;
- Joint Fit Coupon promotion.

## Files changed

Tracked:

- `index.html`
- `README.md` — fixture totals only, because the previous counts became inaccurate

New requested report:

- `docs/PHASE7_1_PRODUCTION_SETTINGS_CLEANUP_2026-07-16.md`

No files were staged, committed, or pushed.

## Unverified

- No human-operated visible-browser walkthrough was performed; browser interaction used local headless Microsoft Edge.
- No operating-system file-picker import/export walkthrough was performed.
- No physical laser/material test was applicable to this data-management cleanup.

READY TO COMMIT
