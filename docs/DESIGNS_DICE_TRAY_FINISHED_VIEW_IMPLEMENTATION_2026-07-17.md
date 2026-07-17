# Dice Tray Finished View Implementation

**Date:** 2026-07-17  
**Baseline:** `edd07d5` — Switch trays to structured production model  
**Scope:** Screen-only Dice Tray Finished View; no commit or push

## Delivered

- Added session-only `finished-view` support for `dice-tray`; Cut Layout remains the default.
- Added `trayFinishedViewProjection(model)`, a Dice-only, transient projection created from the live `buildTrayModel(normalized.values)` result inside `buildTrayDesignResult()`.
- Added `buildDiceTrayFinishedViewSvg(result)`, a template-owned top-down view with a shallow base-depth cue. It renders the open interior, base, four wall positions, Front/Back/Left/Right labels, inside usable dimensions, wall height, outside footprint, accurate two- or four-tab wall-profile wording, and the note “Walls align with base slots.”
- Added focused model, renderer, mode, form/download, invalid-state, and production-isolation checks to `runTrayModelFixtures()`.

## Semantic source and orientation

The projection contains only screen-view semantics: inside dimensions, wall height, material thickness, fit-clearance value, outside base dimensions, joint style, tab-profile metadata, canonical wall IDs, and each wall’s recorded base-slot IDs. It contains no production SVG markup and is omitted for invalid results and Divider Tray results.

The renderer uses the model’s explicit orientation, not Cut Layout positions: interior from above; Front is `wall-front` on the near full-width edge; Back is `wall-back`; Left and Right are `wall-left` and `wall-right` while facing Front. It draws simple assembled wall bands and references their recorded slot relationships without recreating slots, tab cuts, clearance gaps, glue, or corner joinery.

## Isolation and behavior

Finished View markup never enters `result.svg`, `serializeTrayCompatibilitySvg()`, the anonymous red cut group, downloaded bytes, filename, MIME type, storage, backups, records, or schemas. Downloading while Finished View is active still writes the unchanged Cut Layout SVG.

Divider Tray remains Cut Layout only. Finger Box, Sliding Lid Box, and Drawer Cabinet retain their existing preview modes. Invalid Dice Tray dimensions disable/reset Finished View safely, and mode changes do not mutate the draft.

## Validation

- `git diff --check` — clean
- `python -m html.parser index.html` — passed
- Fresh isolated Edge `file://` startup — `document.readyState: complete`
- Dice Tray model/live production fixtures — **233 passed / 0 failed**
- Designs geometry fixtures, including Finger Box Finished View, Sliding Lid Finished View, and Drawer Cabinet Finished Front checks — **861 passed / 0 failed**
- Designs production-application fixtures — **118 passed / 0 failed**
- Test Grid machine identity — **18 passed / 0 failed**
- Evidence Promotion — **58 passed / 0 failed**
- Production Settings — **66 passed / 0 failed**
- Storage recovery — **8 passed / 0 failed**
- Complete suite — **1,653 passed / 0 failed**

The retained 23-case tray production matrix remains green, including the pinned Dice `1726 / 51a55721` and Divider `1965 / a55dda6e` byte contracts, dimensions, element order/attributes, anonymous red group, preview/download identity, filename, MIME type, validation parity, and source-value isolation.

## Protected boundaries

Unchanged: storage/schema functions, import/backup behavior, evidence/production normalization and application, record schemas, tray production geometry, `serializeTrayCompatibilitySvg()`, `downloadCurrentDesignSvg()`, SVG filename/MIME behavior, LightBurn group contract, Divider Tray production behavior, and all non-tray generators.

## Unverified areas

This is not a physical fit, strength, glue, kerf, material-behavior, laser-safety, or production-safety verification. The Finished View intentionally does not depict lids, liners, dice, hardware, dividers, decorative joinery, or a tested wall fit.

**Final status: READY FOR FOCUSED AUDIT**
