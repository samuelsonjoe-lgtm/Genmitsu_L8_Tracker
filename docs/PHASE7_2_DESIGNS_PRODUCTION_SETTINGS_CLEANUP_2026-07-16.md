# Phase 7.2 Designs Production Settings Cleanup

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Committed baseline: `d54b05d` (`Add proven production settings`)

## Repository state before cleanup

- Branch: `main`, tracking `origin/main`.
- Committed HEAD: `d54b05d`.
- Existing tracked Phase 7.2 changes: `index.html`, `README.md`.
- Existing Phase 7.2 architecture, implementation, and independent-audit reports were untracked.
- Numerous unrelated untracked reports, `LightBurn Projects/`, and `parametric_qr_stand_generator.py` were present and were left untouched.
- Staging area was empty.
- Initial `git diff --check` passed.
- Initial verified browser totals were 93/0 for Designs production application, 559/0 for Designs geometry, 66/0 for production settings, and 1250/0 overall.

## Cleanup implemented

### 1. Order-stable evidence fingerprints

Added `designProductionEvidenceSnapshots()` and changed only fingerprint construction in `designProductionFingerprint()`.

The helper creates new fingerprint-only snapshots containing every previously fingerprinted evidence field and sorts them by:

1. evidence ID;
2. kind;
3. source ID;
4. source profile ID;
5. source label;
6. date;
7. result;
8. notes.

Stored evidence arrays and source records are not reordered or mutated. Reversed and shuffled evidence now produce the same fingerprint, while edits, additions, and deletions still change it. Missing and duplicate evidence IDs remain deterministic through the secondary keys.

### 2. Explicit grouped chooser contract

Added these pure helpers:

- `compareDesignProductionRanks()`
- `designProductionSettingGroups()`
- `designProductionNominalThicknessMm()`

`designProductionSettingChoices()` now calculates a child rank containing:

1. exact current machine;
2. active before superseded;
3. preferred before non-preferred;
4. verification rank;
5. label;
6. setting ID.

`designProductionSettingGroups()` keeps each Library profile's children together. It sorts children by the complete child rank and ranks each parent group by its best child's quality rank, followed by parent material, canonical nominal thickness in millimeters, profile ID, and best-child setting ID. `designProductionPanelHtml()` consumes this same grouping helper instead of rebuilding groups independently.

The resulting contract is deterministic under reversed profile arrays and shuffled production-setting arrays. Duplicate setting IDs under different parents remain separate and still require both profile ID and setting ID for resolution. Existing superseded visibility behavior is unchanged.

### 3. Canonical nominal-thickness sorting

`designProductionNominalThicknessMm()` is read-only and does not consult `state.unit`:

- millimeter values retain their numeric value;
- inch values are multiplied by 25.4;
- blank, malformed, missing, negative, and unsupported-unit values receive the deterministic last-place rank `Infinity`;
- display values and stored profile fields remain unchanged.

The mixed-unit fixture order is:

`3 mm`, `0.124 in`, `0.125 in`, `3.175 mm`, malformed, missing.

The physically equivalent `0.125 in` and `3.175 mm` entries use stable profile IDs as the tie-breaker. The same ordering was verified with reversed input, shuffled input, `state.unit = 'mm'`, and `state.unit = 'in'`.

### 4. Fixture corrections

All 11 circular or overclaimed assertions were removed.

Replaced with exercised behavior:

- selection uses `designProductionSessionForSelection()` and verifies unchanged draft and generated SVG bytes;
- checked-but-unapplied state uses a real checked session target and verifies unchanged generated SVG bytes;
- Reset uses the same `resetDesignWorkspaceSession()` helper as the Reset button handler and verifies draft, identity, checks, Show superseded, fingerprint, stale reason, and notice are cleared;
- template transition uses the same `designProductionSessionForTemplateChange()` helper as the form handler and verifies identity, Show superseded, and fingerprint preservation with checks/stale state cleared;
- kerf-only and cut-reference-only source variants verify identical ordinary mappings, unchanged draft/SVG bytes, and source nonmutation;
- preview/download identity renders the actual preview data URL, invokes `downloadCurrentDesignSvg()` with an intercepted download callback, and compares exact SVG bytes after mapped values are applied;
- global millimeter/inch equivalence actually changes `state.unit`, applies measured thickness and signed joint clearance, builds both SVGs, compares draft/SVG bytes, restores the unit, and verifies no storage or evidence change;
- a crafted checked fit target under a known machine mismatch proves the blocked fit value cannot apply while checked measured thickness can;
- changed-record synchronization modifies application-relevant data without changing `updatedDate`, proves the fingerprint changes, clears checks, renders Apply disabled pending review, and preserves already-applied Design values.

Claims narrowed rather than overstated:

- the reload assertion is now named `Fresh-session helper recreates no selection or checked state`; it does not claim a browser reload;
- the import assertion is now named `Imported application-state shapes do not represent Design production session`; it does not claim execution of the real Replace or Merge handlers.

Additional focused fixtures cover evidence order/content behavior, grouped ranking, child ordering, mixed units, duplicate parent/setting identity, deterministic malformed-thickness handling, and source-array nonmutation.

## Functions added or changed

Added:

- `resetDesignWorkspaceSession()`
- `designProductionSessionForTemplateChange()`
- `designProductionEvidenceSnapshots()`
- `designProductionNominalThicknessMm()`
- `compareDesignProductionRanks()`
- `designProductionSettingGroups()`
- `designProductionSessionForSelection()`
- `downloadCurrentDesignSvg()`

Changed within cleanup scope:

- `designProductionFingerprint()`
- `designProductionSettingChoices()`
- `designProductionPanelHtml()`
- `bindDesignProductionActions()`
- Designs template-change, download, and Reset bindings
- `runDesignProductionSettingsFixtures()`
- self-test routing, adding `designs` as an alias for `design`
- README fixture totals and the Designs query alias note

`downloadCurrentDesignSvg()` is a bounded interception seam around the existing download operations. Production still updates the session draft from the form, calls `refreshDesignPreview()`, validates that exact result, and passes `result.svg` to `downloadTextFile()` with the same filename and MIME type.

## Validation

### Static validation

- `git diff --check`: passed; only existing LF-to-CRLF warnings were emitted.
- `python -m html.parser index.html`: passed.
- JavaScript syntax/runtime: passed through Microsoft Edge loading and executing the page and all fixtures; Node.js is not installed.
- Direct `file://` startup without a query: passed; header, tabs, and main content rendered.
- `?selftest=design-production`: loaded and passed.
- `?selftest=designs`: alias loaded and Designs geometry passed.
- `?selftest=production`: loaded and passed.
- `?selftest=all`: loaded and passed.

### Exact fixture totals

| Fixture group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Production settings | 66 | 0 |
| Designs production application | 118 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs geometry | 559 | 0 |
| **Complete suite** | **1275** | **0** |

### Focused results

- Evidence reorder fingerprint: identical for reversed and shuffled arrays.
- Evidence semantic changes: field edit, addition, and deletion each changed the fingerprint.
- Grouped chooser: best-child group rank and within-group child rank passed under reversed and shuffled arrays.
- Mixed-unit ordering: canonical millimeter order passed and did not vary with global display unit.
- SVG isolation: selection, checked-but-unapplied state, kerf-only changes, and cut-reference-only changes preserved exact bytes.
- Preview/download identity: rendered preview bytes exactly matched intercepted download text after application.
- Unit equivalence: millimeter and inch global preferences produced byte-identical drafts and SVGs.
- Crafted blocked target: mismatched fit remained unchanged while measured thickness applied.
- Library nonmutation: profile, production-setting, evidence, mixed-unit profile, and grouped chooser sources remained byte-identical.
- localStorage nonmutation: helper/application, preview/download, and unit-equivalence probes produced no storage write.

## Protected boundaries

Normalized function-level byte comparison against `d54b05d:index.html` confirmed these functions remain identical:

- `backupObject()`
- `persist()`
- `replaceData()`
- `mergeData()`
- `normalizeProductionSetting()`
- `normalizeProductionSettings()`
- `normalizeDesignDraft()`
- `buildDesignResult()`
- `buildFingerPattern()`
- `serializeDesignSvg()`

`STORAGE_KEY` and `SCHEMA_VERSION` are also byte-identical. No state persistence shape, schema, Library save path, Material Test behavior, Project Wizard behavior, shared finger geometry, Drawer Cabinet geometry, SVG serialization, or LightBurn layer behavior was changed. No Finished Front View work was added.

## Files changed

- `index.html`
- `README.md`
- `docs/PHASE7_2_DESIGNS_PRODUCTION_SETTINGS_CLEANUP_2026-07-16.md`

All pre-existing unrelated untracked files and reports remain untouched. Nothing was staged, committed, or pushed.

## Unverified areas

- No human-operated visible-browser session was performed; browser validation used fresh headless Microsoft Edge direct-file sessions.
- The renamed import-shape fixture does not execute real Replace/Merge handlers, and the fresh-session fixture does not perform a full browser reload. Their names now state those limits accurately.
- No physical laser or LightBurn workflow was needed because this cleanup did not change geometry or SVG serialization.

READY TO COMMIT
