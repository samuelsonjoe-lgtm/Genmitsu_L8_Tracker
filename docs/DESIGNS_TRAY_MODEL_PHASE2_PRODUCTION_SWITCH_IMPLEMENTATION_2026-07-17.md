# Designs Tray Model Phase 2 Production Switch Implementation

Date: 2026-07-17
Baseline: `6ed566b Add structured tray model verification`

## Outcome

Dice Tray and Divider Tray now use the structured Tray model for live production SVG. Their preview and download output remain byte-identical to the Phase 1 legacy contract: one anonymous red cut group, unchanged dimensions/viewBox, shape ordering, attributes, filenames, MIME type, and anonymous `Shape N` result panels.

**Final status: READY FOR PRODUCTION-SWITCH AUDIT**

## Files changed

- `index.html`
  - Removed the legacy executable tray generator.
  - Routed live Dice/Divider result generation through the Tray model and compatibility serializer.
  - Replaced shadow old-vs-new tray fixtures with pinned live-production signatures, structural checks, and preview/download checks.
- `README.md`
  - States that the structured Tray model now drives live production SVG, with no Tray Finished View or storage/schema migration.
- `docs/DESIGNS_TRAY_MODEL_PHASE2_PRODUCTION_SWITCH_IMPLEMENTATION_2026-07-17.md`
  - This implementation record.

No unrelated modified or untracked files were changed, staged, moved, deleted, committed, or pushed.

## Routing before and after

Before Phase 2:

```text
buildDesignResult(draft)
  -> buildLegacyDesignResult(normalized)
    -> legacyDesignSvg(normalized.draft)
      -> designTraySvg(draft)
```

After Phase 2:

```text
buildDesignResult(draft)
  -> buildTrayDesignResult(normalized)
    -> existing validateLegacyDesign(normalized)
    -> buildTrayModel(normalized.values)
    -> serializeTrayCompatibilitySvg(model)
    -> existing SVG validation and anonymous Shape N panel projection
```

`buildLegacyDesignResult()` remains unchanged for the other legacy templates. `legacyDesignSvg()` no longer has a Dice/Divider branch. `designTraySvg()` was removed entirely; there is no executable legacy tray fallback or competing tray geometry implementation.

## Functions added, removed, and modified

- Added `buildTrayDesignResult(normalized)`.
  - Preserves existing validation ordering, error/warning result shape, SVG validation, dimensions, `metrics.template`, and anonymous `Shape N` panel projection.
- Removed `designTraySvg(draft)`.
- Modified `legacyDesignSvg(draft)` to remove tray dispatch.
- Modified `buildDesignResult(rawDraft)` to route only `dice-tray` and `divider-tray` through `buildTrayDesignResult()`.
- Added `trayLivePreviewDownloadFixture(draft)` for fixture-only browser-DOM preview/download interception.
- Retained `buildTrayModel()` and `serializeTrayCompatibilitySvg()` as the sole tray geometry and SVG projection path.

`serializeDesignSvg()`, download filename generation, storage/import helpers, and other Designs templates were not changed.

## Compatibility serializer and result behavior

`serializeTrayCompatibilitySvg()` is now the live Dice/Divider serializer. It still uses `designSvgDocument()` rather than `serializeDesignSvg()`, preserving:

- document width/height and viewBox;
- a single anonymous group with `fill="none"`, `stroke="#ff0000"`, and `stroke-width="0.1"`;
- identical rect/path order, attributes, numeric formatting, margins, gaps, and layout dimensions;
- no IDs, titles, descriptions, labels, score group, metadata, or named LightBurn layers.

The live result continues to expose anonymous `Shape N` panels parsed from the output SVG. Semantic components remain internal to the model; no UI component names, Finished View, saved-Design format, backup/export format, or record/schema change was added.

## Phase 1 fixture transition

The focused Tray suite remains **222 assertions** and is still included exactly once in Designs geometry.

Retained:

- semantic model, orientation, Divider linkage and equal-spacing assertions;
- deterministic-model and source-input nonmutation checks;
- current validation/model invalid rejection checks and existing UI wording check;
- no-storage-leakage check;
- default Dice/Divider length/hash goldens;
- layout bounds and material-panel non-overlap checks.

Replaced:

- runtime `legacyDesignSvg()` versus compatibility-serializer comparison for 23 cases;
- direct legacy default SVG generation inside the Tray fixture.

The production matrix now pins Phase 1 lengths, FNV-style fixture hashes, dimensions/viewBoxes, element counts, and element order/attribute signature hashes. Each live result is also required to equal the compatibility serializer output and preserve anonymous `Shape N` panels. This keeps a fixed production oracle after removal of the executable legacy generator rather than comparing two geometry engines indefinitely.

Added live workflow coverage:

- every matrix case renders the exact exported SVG in the preview object;
- every matrix case downloads identical bytes through `downloadCurrentDesignSvg()`;
- every matrix case preserves `l8-dice-tray-YYYY-MM-DD.svg` or `l8-divider-tray-YYYY-MM-DD.svg` plus `image/svg+xml;charset=utf-8`.

## Production matrix

All 23 live cases passed their pinned production checks:

| Template | Cases |
| --- | --- |
| Dice Tray | Default Finger; default Tab-and-slot; zero clearance; thicker material; minimum valid Finger plan; large plan; shallow wall; tall wall; square plan; long narrow plan. |
| Divider Tray | Default Finger count 2; default Tab-and-slot count 2; zero clearance count 2; thicker material count 2; minimum valid Finger plan count 1; large plan count 6; shallow wall count 1; tall wall count 6; square plan count 2; long narrow plan count 6; shallow depth count 1; deep narrow spacing count 6; wide spacing count 1. |

For every case, the live result is valid, uses the model serializer, matches the pinned byte signature, preserves dimensions/viewBox, preserves element count/order/attributes and anonymous red group, contains no NaN/Infinity/undefined, and has preview/download/filename identity.

Pinned default goldens remain:

- Dice Tray: 1,726 characters / `51a55721`
- Divider Tray: 1,965 characters / `a55dda6e`

## Validation parity, download, and workflow

Existing validation remains first in the live route through `validateLegacyDesign()`. The model is not a separate user-facing validator: invalid material thickness, clearance, width, depth, height, too-short dimensions, invalid joint style, and invalid Divider counts remain rejected before model serialization, with existing UI wording and non-downloadable results.

The isolated direct-file Edge workflow exercised Dice and Divider template selection, Finger and Tab-and-slot variants, changed dimensions/clearance, Divider counts 1/2/6, preview extraction, intercepted download, filename/MIME verification, and invalid-result blocking. Switching to the non-tray templates remains covered by existing Designs geometry/Finished View fixtures.

## Production isolation and protected boundaries

The model is derived from existing normalized draft values on each refresh. It is not stored in localStorage, backups, imports/exports, saved Designs, Projects, Production Settings, Material Tests, or Test Grids. `STORAGE_KEY` and `SCHEMA_VERSION` are unchanged.

Protected storage, import/merge, evidence, production-setting, Material Test, Test Grid, Project, form/default, filename, LightBurn grouping, Finger Box, Sliding Lid Box, Drawer Cabinet, Joint Fit Coupon, QR stand, and Hanging sign boundaries are unchanged. The only intentional protected-function changes are removal of the old tray dispatch and the new tray branch in `buildDesignResult()`.

## Validation results

Completed successfully:

- `git diff --check`
- `python -m html.parser index.html`
- isolated direct `file://` Edge startup (`document.readyState: complete`)
- Tray model/live production fixtures: **222 passed / 0 failed**
- Designs geometry fixtures: **850 passed / 0 failed**
- Designs production-application fixtures: **118 passed / 0 failed**
- Test Grid machine-identity fixtures: **18 passed / 0 failed**
- Evidence Promotion fixtures: **58 passed / 0 failed**
- Production Settings fixtures: **66 passed / 0 failed**
- Storage recovery fixtures: **8 passed / 0 failed**
- Complete suite: **1,642 passed / 0 failed**
- protected-function comparison against `6ed566b`

## Unverified areas

This pass does not add or validate a Tray Finished View, physical assembly, material fit, laser safety, or an actual LightBurn import. The single anonymous red group and byte-identical SVG contract are verified in browser-side production output, but real material testing remains required before production use.
