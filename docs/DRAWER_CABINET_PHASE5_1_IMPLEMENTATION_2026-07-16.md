# Drawer Cabinet Phase 5.1 Implementation Report

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `67860a6` — `Refine sliding-lid guides and add assembly labels`

## Outcome

Implemented a new session-only Designs template named `Drawer Cabinet` without refactoring the existing Finger Box or Sliding Lid Box generators.

The template produces:

- one five-panel open-front, closed-back cabinet shell;
- one to three equal removable open-top finger-jointed drawers;
- five individually emitted panels per drawer;
- one individually emitted full-width support shelf for each upper drawer row;
- optional deterministic vector assembly labels;
- blue score geometry before red cut geometry;
- one deterministic, validated SVG shared by preview and download.

The bottom drawer rests on the cabinet bottom. Rows two and three use plain full-width edge-glued support shelves. These shelves keep the outer cabinet shell at five panels while making the stacked outputs physically supportable. The app warns that shelf alignment and glue strength require a scrap prototype.

## Initial repository state

- Branch: `main`, synchronized with `origin/main`
- HEAD: `67860a6`
- Tracked working tree: clean
- Staging area: empty
- Existing unrelated untracked files and reports: preserved
- Initial `git diff --check`: passed, with only the existing inaccessible user-level Git ignore warning

No reset, clean, stash, move, delete, stage, commit, or push operation was performed.

## Files changed

- `index.html`
- `README.md`
- `docs/DRAWER_CABINET_PHASE5_1_IMPLEMENTATION_2026-07-16.md`

No unrelated application file was modified.

## UI and session state

Added `Drawer Cabinet` to the Designs template selector and added session-only draft defaults for:

- measured material thickness;
- drawer inside width;
- drawer inside depth;
- drawer inside height;
- rows: 1, 2, or 3;
- cabinet joint clearance;
- drawer joint clearance;
- preferred finger width;
- total drawer lateral clearance;
- drawer vertical clearance;
- drawer rear clearance;
- optional assembly labels.

The new values remain in `designDraft`. They were not added to `state`, `freshState()`, `loadState()`, `persist()`, backup objects, import handling, `STORAGE_KEY`, or `SCHEMA_VERSION`.

## Geometry implementation

### New independent cabinet path

Added dedicated helpers:

- `drawerCabinetDimensions()`
- `namespaceDesignPanel()`
- `buildDrawerCabinetModel()`
- `buildDrawerCabinetDesignResult()`

`normalizeDesignDraft()` and `buildDesignResult()` gained one bounded `drawer-cabinet` branch. Existing Finger Box and Sliding Lid Box model builders were not generalized or rewritten.

### Reused low-level helpers

The cabinet reuses:

- `buildFingerPattern()`
- `designPatternEdge()`
- `buildSlidingLidBodyPanel()` as the existing corner-safe orthogonal finger-panel composer
- `buildBoxModel()` as the authoritative five-panel open-top drawer model
- `designPanelFromPoints()` for plain support shelves
- `designPanelGeometryErrors()` for newly introduced cabinet/shelf outlines
- `layoutDesignPanelRows()`
- `serializeDesignSvg()`
- `designSvgValidation()`
- assembly-label containment and serialized validation helpers.

The stricter sliding-panel geometry checker is applied only to the new shell and shelf geometry. The cloned drawer panels remain governed by the existing proven `buildBoxModel()` validity rules; reapplying the stricter checker incorrectly classified the legacy Finger Box finger transitions as self-intersections during the first runtime pass.

### Dimension formulas

For material thickness `t` and requested drawer inside dimensions:

```text
drawer outside width  = drawer inside width  + 2t
drawer outside depth  = drawer inside depth  + 2t
drawer outside height = drawer inside height + t

cell width  = drawer outside width  + total lateral clearance
cell height = drawer outside height + vertical clearance

cabinet inside width  = cell width
cabinet inside depth  = drawer outside depth + rear clearance
cabinet inside height = rows × cell height + (rows - 1)t

cabinet outside width  = cabinet inside width  + 2t
cabinet outside depth  = cabinet inside depth  + t
cabinet outside height = cabinet inside height + 2t
```

Cabinet outside depth adds one thickness because the front is open and only the closed back occupies depth.

### Physical pieces and IDs

The five shell IDs are:

```text
cabinet-bottom
cabinet-top
cabinet-left
cabinet-right
cabinet-back
```

Upper support shelves use:

```text
cabinet-shelf-r01
cabinet-shelf-r02
```

Each drawer uses padded row IDs such as:

```text
drawer-r01-bottom
drawer-r01-front
drawer-r01-back
drawer-r01-left
drawer-r01-right
```

Every physical copy is serialized separately. Exact piece counts are:

- one row: 10 pieces;
- two rows: 16 pieces;
- three rows: 22 pieces.

## Assembly labels

The existing embedded vector label system is reused. Added a deterministic `P` glyph so the cabinet top can use `TOP` without SVG text or a font dependency.

Optional labels cover safe cabinet shell and drawer faces. Support shelves are left unlabeled. Labels that cannot satisfy polygon and cut-clearance checks are omitted with warnings while valid cut output remains available.

## Validation and warnings

The template blocks:

- rows outside integer values 1–3;
- blank, zero, negative, or non-finite dimensions;
- negative clearances;
- lateral clearance at least as large as drawer outside width;
- vertical clearance at least as large as drawer outside height;
- rear clearance at least as large as drawer outside depth;
- cabinet or drawer edges too small for valid finger patterns;
- joint clearances that collapse required finger web;
- malformed cabinet/shelf panel geometry;
- duplicate translated cuts;
- overlapping layout geometry;
- invalid or out-of-bounds SVG.

Warnings cover zero running clearances, unusually large clearances, large cabinet/layout size, loose joints, experimental drawer fit, measured-sheet testing, and edge-glued support-shelf prototyping.

Kerf remains separate LightBurn/material-test work.

## Fixtures added

Added 48 Designs assertions covering:

- valid one-, two-, and three-row cabinets;
- exact drawer inside/outside formulas;
- exact cabinet inside/outside formulas;
- open-front depth semantics;
- five-panel shell count;
- support shelf counts;
- exact support-shelf elevations above the cabinet inside floor;
- five panels per drawer;
- exact 10/16/22 piece counts;
- stable IDs for all three row counts;
- unique IDs and individually emitted repeated pieces;
- deterministic SVG and layout positions;
- no overlap;
- final SVG validity and finite numeric output;
- preview/download identity;
- score/cut ordering;
- optional vector labels without SVG text;
- final label containment;
- all eight shell mating boundaries and opposite phases;
- independent cabinet and drawer joint clearances;
- lateral, vertical, and rear clearance formulas;
- draft nonmutation;
- localStorage nonmutation;
- invalid rows, dimensions, clearances, and undersized panels;
- legacy Finger Box and Sliding Lid signatures.

## Validation results

### Static

- HTML parsing: passed
- `git diff --check`: passed
- No external dependency was added

### Microsoft Edge runtime

Direct local `file://` startup and public fixture functions were exercised in isolated Edge profiles.

| Fixture group | Passed | Failed |
| --- | ---: | ---: |
| Baseline resolution | 20 | 0 |
| Material-test normalization | 12 | 0 |
| Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs | 424 | 0 |
| **Complete suite** | **956** | **0** |

### Live form check

The isolated Edge runtime navigated to Designs, selected `Drawer Cabinet`, and confirmed:

- all requested controls rendered;
- default row count was 1;
- no validation error appeared;
- exact SVG preview rendered;
- download was enabled.

## Manual testing not performed

Not performed:

- physical plywood cutting;
- LightBurn import and layer-operation inspection;
- cabinet or drawer assembly;
- shelf glue-strength/alignment testing;
- measured running-fit testing after glue cure;
- visible human-operated browser testing outside the automated Edge session.

The generated shelf system and clearances remain experimental until physically prototyped.

## Documentation

README now documents:

- the one-column Drawer Cabinet template;
- one to three equal drawers;
- independently emitted drawer pieces;
- edge-glued support shelves for upper rows;
- independent cabinet, drawer, lateral, vertical, and rear clearances;
- glue and physical-prototype requirements;
- verified complete fixture total of 956 passed / 0 failed.

## Recommended next gate

Before spending plywood and laser time, perform a focused independent audit of only the drawer-cabinet implementation. Claude or Grok should inspect:

- cabinet mating-edge phase assignments;
- cabinet outside-depth formula for the open front;
- row-height and shelf-thickness accumulation;
- namespaced IDs and piece counts;
- support-shelf dimensions and physical placement instructions;
- clearance validation;
- SVG score/cut order and label containment;
- fixture strength versus actual production geometry.

After that audit, cut joint coupons and a one-row cabinet before attempting two or three rows.

Nothing was staged, committed, or pushed.
