# Designs Tray Model Phase 1 Implementation

Date: 2026-07-17
Baseline: `16128e0 Add sliding lid box finished views`

## Outcome

Phase 0–1 is complete. Dice Tray and Divider Tray now have a pure internal structured model and a fixture-only legacy compatibility serializer. Production preview and download still use the untouched `designTraySvg()` legacy path; no Finished View, production routing switch, storage change, form change, or SVG contract migration occurred.

**Final status: READY FOR FOCUSED MODEL AUDIT**

## Files changed

- `index.html`
  - Added the internal Tray model, compatibility serializer, and isolated tray-model fixture group.
  - Added normalized `jointStyle` to tray values so the model receives validated normalized values rather than a raw draft.
  - Added the isolated fixture group to existing Designs geometry fixtures without adding a new self-test query value.
- `README.md`
  - Documents the internal-only model/legacy verification status and observed fixture totals.
- `docs/DESIGNS_TRAY_MODEL_PHASE1_IMPLEMENTATION_2026-07-17.md`
  - This implementation record.

No unrelated untracked files were changed, staged, moved, deleted, committed, or pushed.

## Helpers and functions

Added pure helpers:

- `trayTabProfile(lengthMm, jointStyle)`
- `trayModelValidationErrors(values)`
- `trayRectComponent(...)`
- `trayWallComponent(...)`
- `trayLayoutItem(...)`
- `traySvgComparison(svg)` (fixture-only DOM comparison)
- `trayModelMaterialBoxes(model)` (fixture-only layout check)

Added functions:

- `buildTrayModel(values)`
- `serializeTrayCompatibilitySvg(model)`
- `runTrayModelFixtures()`

Modified only integration points needed for Phase 1:

- `normalizeDesignDraft()` now carries the already-validated tray `jointStyle` in `values`.
- `runDesignGeometryFixtures()` includes the isolated Tray model results exactly once.

The following protected production functions remain behaviorally unchanged: `designTraySvg()`, `legacyDesignSvg()`, `buildLegacyDesignResult()`, `buildDesignResult()`, `serializeDesignSvg()`, and `downloadCurrentDesignSvg()`.

## Approved orientation contract

The model records, without deriving from sheet-layout coordinates:

- Interior face: `top`.
- Front: first full-width wall / near width edge.
- Back: opposing second full-width wall.
- Left and Right: side walls while facing Front.
- Divider Tray dividers span `left-to-right`.
- Divider Tray dividers order `front-to-back`; `divider-01` is nearest Front.

This semantic contract does not alter cut geometry, layout order, filenames, exported SVG, or current UI wording.

## Model structure

`buildTrayModel(values)` accepts only normalized numeric tray values and returns a deterministic valid or invalid model. It does not read form elements or `designDraft`, parse or call legacy SVG generation, mutate inputs, write storage, or create records.

Valid models contain:

- identity: template, validity, errors/warnings, `tray-model-phase1` generator version;
- inputs: material thickness, fit clearance, current two/four-tab joint-style meaning, inside dimensions, wall height, and Divider-only count;
- dimensions: base outside dimensions, slot width, and width/depth tab profiles;
- orientation: the approved stable wall IDs and Divider-only direction/order;
- components, cut-only operations, legacy-compatible layout, and piece/wall/divider/compartment metrics.

Invalid or incomplete normalized values return a deterministic invalid model with no semantic components, layout items, or SVG.

## Semantic components and Divider linkage

Stable component IDs are emitted in current legacy order:

- `tray-base`
- `base-slot-front-01` …, `base-slot-back-01` …, `base-slot-left-01` …, `base-slot-right-01` …
- `wall-front`, `wall-back`, `wall-left`, `wall-right`
- Divider-only `base-divider-slot-01` … and `divider-01` …

Each Divider panel references exactly one base-divider slot through `baseSlotId`. The model records its equal depth-axis position, Left-to-Right span, Front-to-Back order, and the current UI-supported removable intent. Its retention is explicitly `unspecified`; no fixed-fit, cross-divider, liner, lid, score, label, or production-safe claim was introduced.

## Compatibility serializer contract

`serializeTrayCompatibilitySvg(model)` is used only by fixtures. It projects the model’s ordered layout items through `designSvgDocument()` and preserves the legacy contract:

- same document width/height and viewBox;
- one anonymous red cut group;
- same red stroke attributes;
- same rect/path sequence, element order, numeric formatting, margin/gap, and Divider interleaving;
- no SVG IDs, titles, labels, metadata, score group, named layers, or LightBurn grouping change.

It is not used by the live preview or download path, and it does not call `designTraySvg()`.

## Legacy-to-model comparison matrix

All 23 valid matrix cases are byte-identical between `legacyDesignSvg(normalized.draft)` and the compatibility serializer:

| Template | Cases |
| --- | --- |
| Dice Tray | Default Finger; default Tab-and-slot; zero clearance; thicker material; minimum valid Finger plan (42 × 48); large plan (500 × 420); shallow wall; tall wall; square plan; long narrow plan. |
| Divider Tray | Default Finger count 2; default Tab-and-slot count 2; zero clearance count 2; thicker material count 2; minimum valid Finger plan count 1; large plan count 6; shallow wall count 1; tall wall count 6; square plan count 2; long narrow plan count 6; shallow depth count 1; deep narrow spacing count 6; wide spacing count 1. |

For every case, fixtures assert valid model/SVG, byte equality, document dimensions and viewBox, element count, element type/attributes/order, anonymous red group/stroke structure, absence of NaN/Infinity/undefined, and unmutated normalized source values.

The existing default golden fixtures remain unchanged:

- Dice Tray: 1,726 characters / `51a55721`
- Divider Tray: 1,965 characters / `a55dda6e`

## Semantic and invalid-input coverage

The 222 focused Tray-model assertions include:

- Dice base, four stable walls, no Divider components, dimensions, cut-only operations, deterministic output, orientation, unique IDs, bounds, and non-overlapping material-panel layout.
- Divider slot/panel count, `dividerCount + 1` compartments, one-to-one slot linkage, exact equal-spacing formula, approved orientation/numbering, no cross/fixed-retention semantics, deterministic output, unique IDs, bounds, and non-overlapping material-panel layout.
- Legacy/model rejection parity for invalid material thickness, clearance, zero/negative width/depth/height, too-short width/depth, invalid style, and Divider counts below 1, above 6, and non-integer.
- No semantic components on invalid models; unchanged legacy UI error wording; no model local-storage writes; and no SVG for invalid models.

## Production isolation and protected boundaries

The production tray generator and routing remain legacy-only. The model is neither persisted nor backed up, and no preview selector or Tray Finished View is exposed. Existing form fields/defaults, validation wording, download filenames, and legacy one-group red SVG behavior remain intact.

Comparison against `16128e0` found no changes to protected storage, backup/import, evidence, production-setting, Material Test, Test Grid, Project, download, serializer, or existing Finished View boundaries. `STORAGE_KEY` and `SCHEMA_VERSION` remain unchanged. The only protected-name diff reference is a fixture assertion that verifies the model does not write `localStorage`.

## Validation

Completed successfully:

- `git diff --check`
- `python -m html.parser index.html`
- isolated headless Edge direct `file://` startup (`document.readyState: complete`)
- Tray model fixtures: **222 passed / 0 failed**
- Designs geometry fixtures: **850 passed / 0 failed**
- Designs production-application fixtures: **118 passed / 0 failed**
- Test Grid machine-identity fixtures: **18 passed / 0 failed**
- Evidence Promotion fixtures: **58 passed / 0 failed**
- Production Settings fixtures: **66 passed / 0 failed**
- Storage recovery fixtures: **8 passed / 0 failed**
- Complete suite: **1,642 passed / 0 failed**
- protected-function diff comparison against `16128e0`

The browser validation used an isolated temporary Edge profile and direct `file://` loading. It validates application startup and browser-side fixture execution; it does not prove physical material fit, laser-cut safety, or an actual LightBurn import. Tray Finished Views and the eventual production routing switch are intentionally unverified because they were not implemented in this phase.
