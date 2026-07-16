# Drawer Cabinet Finished Front View — Architecture Review

Date: 2026-07-16  
Baseline: `9e8377a Apply proven settings in Designs`  
Scope: read-only architecture and implementation planning; no application implementation performed.

## 1. Executive recommendation

Implement the Finished Front View as a session-only, screen-only renderer layered beside the existing exact exported SVG preview. The existing Drawer Cabinet model is the correct source of truth and already exposes nearly all required geometry data. Add only additive, non-serialized preview metadata to the model/result boundary: explicit row descriptors, drawer-front dimensions/positions, opening bounds, and shelf descriptors.

Keep `result.svg` exactly as it is. The new view must consume the same validated model but must never pass its SVG to `downloadCurrentDesignSvg()` or `serializeDesignSvg()`.

Recommendation: **READY FOR IMPLEMENTATION**, with the row-label convention below treated as an explicit implementation decision.

## 2. Current Drawer Cabinet model and result architecture

The current flow is:

1. `designDefaults()` supplies a session-only draft (`index.html:405-406`).
2. `normalizeDesignDraft()` validates Drawer Cabinet inputs, including one to three rows and non-negative running clearances (`index.html:2002-2029`).
3. `buildDesignResult()` dispatches Drawer Cabinet drafts to `buildDrawerCabinetDesignResult()` (`index.html:2972-2977`).
4. `buildDrawerCabinetDesignResult()` builds one authoritative model, arranges production panels, validates translated cut segments, optionally emits assembly-label paths, and serializes the production SVG (`index.html:2950-2970`).
5. `designResultsHtml()` currently renders metrics plus an `<object>` containing the exact production SVG (`index.html:2981-3058`).
6. `refreshDesignPreview()` rebuilds that result and replaces the result area (`index.html:3060-3064`).
7. `downloadCurrentDesignSvg()` calls the same production result path and downloads only `result.svg` (`index.html:3066-3071`).

`buildDrawerCabinetModel()` already returns `dimensions`, production `panels`, `metrics`, and deterministic panel IDs (`index.html:2261-2301`). It is sufficient as the single geometry authority. It does not currently expose a structured installed-front view model, so the smallest safe additive change is to derive and attach preview metadata after the existing dimensions and shelf calculations. Do not alter panel points, serialized SVG, panel IDs, or production layout ordering.

The existing result object is transient and is not persisted. That makes it safe to retain a `drawerFrontView`/`frontElevation` metadata object on the result for rendering, provided it contains no replacement production SVG and no draft/state references.

## 3. Authoritative dimensions and row-order rules

`drawerCabinetDimensions()` is authoritative (`index.html:2245-2256`):

```text
drawerOutsideWidth  = drawerInsideWidth  + 2t
drawerOutsideDepth  = drawerInsideDepth + 2t
drawerOutsideHeight = drawerInsideHeight + t
cellWidth           = drawerOutsideWidth + totalLateralClearance
cellHeight          = drawerOutsideHeight + verticalClearance
cabinetInsideWidth  = cellWidth
cabinetInsideDepth  = drawerOutsideDepth + rearClearance
cabinetInsideHeight = rows * cellHeight + (rows - 1) * t
cabinetOutsideWidth = cabinetInsideWidth + 2t
cabinetOutsideHeight= cabinetInsideHeight + 2t
```

The front view must use `cabinetOutsideWidth`, `cabinetOutsideHeight`, `cabinetInsideWidth`, `cabinetInsideHeight`, `drawerOutsideWidth`, `drawerOutsideHeight`, `t`, and the three running clearances from this model. Do not recalculate them from raw draft strings in a preview-only calculator.

Current physical row semantics are bottom-up even though the visible IDs are only `Drawer 1`, `Drawer 2`, and `Drawer 3`:

- Drawer row 1 is the bottom drawer. The assembly copy explicitly says the bottom drawer rests on the cabinet bottom (`index.html:1880`, `3054`).
- `cabinet-shelf-r01` is the first shelf above row 1; subsequent shelf IDs are generated in ascending order.
- Shelf bottom elevations are `(index + 1) * cellHeight + index * t` (`index.html:2299`), measured above the inside floor.
- Production layout order is bottom/top shell, side/back shell, shelves, then drawer rows in ascending `r01`, `r02`, `r03` order (`index.html:2954-2959`).

Recommended visible convention: `Drawer 1 — Bottom`, `Drawer 2 — Middle`, `Drawer 3 — Top`. This is consistent with the current physical assembly guidance and avoids changing production IDs. The report should call out that these labels are a view annotation; existing part IDs remain `drawer-r01-*`, etc.

## 4. Recommended screen-preview architecture

Add a compact result-area selector only when the template is `drawer-cabinet`, for example a fieldset labelled `Preview` with `Cut Layout` and `Finished Front View` buttons or radios. Buttons with `aria-pressed` are a good fit for the existing compact UI; radios are equally valid if the surrounding form semantics are simpler.

Use a generic session variable such as `designPreviewMode`, normalized to `cut-layout` unless the current template is Drawer Cabinet and the value is `finished-front`. It must:

- live outside `designDraft` and `state`;
- never be included in `backupObject()` or `persist()`;
- never be written to `localStorage`;
- reset to `cut-layout` in `resetDesignWorkspaceSession()`;
- leave non-Drawer templates on the existing production preview;
- default to `cut-layout` after reload.

The render path should be:

```text
buildDesignResult(draft)
  -> normalized Drawer Cabinet model
  -> production result.svg (unchanged)
  -> additive frontElevation metadata

designResultsHtml(result, previewMode)
  -> Cut Layout: existing production SVG object
  -> Finished Front View: inline screen-only SVG from frontElevation

downloadCurrentDesignSvg()
  -> existing result.svg only
```

Do not build the front view by parsing production SVG or by invoking a second dimension calculator.

## 5. Front-view drawing specification

Use an inline SVG with a viewBox in millimetres and a screen-only `preserveAspectRatio="xMidYMid meet"`. The drawing coordinate system should be cabinet-front coordinates: `(0, 0)` at the outside top-left, x increasing right, y increasing down. The outer rectangle is exactly `cabinetOutsideWidth × cabinetOutsideHeight`.

Recommended visual layers, each distinguishable without color alone:

1. Exterior outline: dark solid stroke, labelled in the legend as `Cabinet exterior`.
2. Side walls/top/bottom: filled or hatched bands representing actual material thickness.
3. Shelf bands: horizontal solid bands at the actual shelf elevation and actual thickness; label as `Support shelf`.
4. Openings: light, dashed or lightly filled rectangles behind drawer fronts.
5. Drawer fronts: heavier solid outlined rectangles, no invented pull, scoop, overlay, or knob.
6. Clearance annotations: thin dashed dimension lines with text, not production-style cut paths.
7. Labels: centered text or deterministic screen text on the drawer front, with a nearby legend/table if the front becomes too short.

The front view should show the outer width and height, drawer count, drawer order, shelf locations, per-side lateral gap, vertical gaps, and a note: `Screen-only assembly preview — not included in downloaded cut file.` Use actual `t` for top/bottom/side/shelf bands when the scale permits.

The cabinet is open at the front and closed at the back, but rear depth should not be drawn as a perspective feature. The front elevation may simplify finger joints and rear depth. It must not imply a front overlay or a pull that the generated geometry does not contain.

## 6. Drawer and shelf placement formulas

For each physical row `r` from 1 through `rows`, where row 1 is bottom:

```text
outerX       = t + lateralClearance / 2
outerWidth   = drawerOutsideWidth
openingX     = t
openingWidth = cabinetInsideWidth

bottomInteriorY = t + (r - 1) * (cellHeight + t)
drawerFrontY    = bottomInteriorY + verticalClearance / 2
drawerFrontH    = drawerOutsideHeight
openingY        = bottomInteriorY
openingH       = cellHeight
```

The remaining vertical free space is intentionally represented as the row's clearance above the drawer: `cellHeight - drawerOutsideHeight = verticalClearance`. The total lateral free space is `lateralClearance`, represented as `lateralClearance / 2` on each side. The front-view model should expose both the total and per-side values so annotations cannot accidentally treat the total as one-side clearance.

For each support shelf `s` from 1 through `rows - 1`:

```text
shelfBottomY = t + s * cellHeight + (s - 1) * t
shelfTopY    = shelfBottomY + t
shelfWidth   = cabinetInsideWidth
```

This is the same elevation already emitted as `shelfBottomElevations` and must not produce a shelf for a one-row cabinet. The cabinet bottom is the first support surface; the top panel is not an additional drawer shelf. The top row's opening terminates at the cabinet top band.

The model should include a row descriptor with `row`, `position: bottom|middle|top`, `drawerPrefix`, `drawerPanelIds`, `label`, `front`, `opening`, and `clearance` fields. This is additive metadata only; production IDs and panel arrays remain unchanged.

## 7. Label and part-identity strategy

Current drawer production IDs are generated with the `drawer-rNN` prefix and panel suffixes such as `bottom`, `front`, `back`, `left`, and `right` (`index.html:2294`, `2956-2959`). Cabinet shelves use `cabinet-shelf-rNN`; the shell uses `cabinet-bottom`, `cabinet-top`, `cabinet-left`, `cabinet-right`, and `cabinet-back`.

The front view should display `Drawer 1 — Bottom`, etc., and retain a machine-readable `data-drawer-row="1"` and `data-panel-prefix="drawer-r01"` on the SVG group. A small text legend can map each row to `drawer-r01-*`. Do not change current names or add front-view labels to the production blue score layer.

Future hover/click highlighting can use those prefixes to highlight matching flattened panels. It is not required for the first implementation.

## 8. UI and session-state proposal

Put the selector immediately above the result area, not in the production form. It should be rendered only for a valid or invalid Drawer Cabinet result, with the mode disabled or reset when another template is selected. Switching modes should call only the result-area refresh path; it should not call `updateDesignDraft()` or any persistence function.

Reset must set the mode to `cut-layout` before `render()`. Switching templates should follow the predictable policy of resetting to Cut Layout. Returning to Drawer Cabinet should therefore show the production preview by default. This is safest and avoids carrying an assembly view into a newly changed draft.

## 9. Production-SVG isolation

The production SVG is currently constructed by `serializeDesignSvg(partial)` in `buildDrawerCabinetDesignResult()` and downloaded directly from `result.svg`. Preserve that path unchanged. The front view must be generated by a separate helper such as `buildDrawerCabinetFrontElevationSvg(frontElevation)` that returns a screen-only SVG string or DOM-safe markup.

The front-view string must never be assigned to `result.svg`, `labelPaths`, `panels`, or the download source. `downloadCurrentDesignSvg()` should remain mode-agnostic and continue to call `buildDesignResult()` and download its production `svg`.

The existing `backupObject()` contains application data and preferences but no Designs draft (`index.html:506-520`); the new mode must likewise be absent. Existing fixture evidence already checks that Designs production sessions are absent from backup, so add a specific preview-mode absence assertion rather than broadening the backup schema.

## 10. Responsive and accessibility plan

Inline SVG is the best implementation for this standalone offline app: it scales cleanly, requires no dependency, and is clearly separable from the production SVG object. Give it a bounded wrapper with `width:100%`, a sensible `max-height`, and `overflow:hidden` or a controlled viewBox rather than horizontal scrolling.

Use `role="img"`, an SVG `<title>`, and `<desc>` containing cabinet dimensions, row count, and the screen-only warning. Keep labels at a minimum readable screen size; when a drawer is too short, place its label in an adjacent keyed legend rather than shrinking text until it becomes meaningless. Dimension text should remain in mm, matching Designs.

Use stroke patterns, hatching, line weight, and labels in addition to color. Selector controls need visible focus, an accessible group label, and an explicit active state.

## 11. Invalid and edge-case behavior

Never render a Finished Front View from partial invalid data. If normalization or model generation fails, show the existing validation errors and the existing `Preview unavailable` state; hide or disable the front-view option until a valid model exists.

Required handling:

- one row: one drawer, no intermediate shelf, bottom drawer on cabinet bottom;
- two or three rows: every drawer once, deterministic bottom-to-top order, exactly `rows - 1` shelves;
- zero clearance: show a coincident/very small gap using a minimum screen stroke and an annotation of `0.00 mm`, without changing geometry;
- extreme valid clearances: preserve numeric values and use bounded annotation placement;
- very short/tall drawers: keep aspect ratio, clamp label size, and use an external legend when needed;
- invalid dimensions, rows, or clearances: no front view and no misleading fallback model;
- all generated coordinates: finite, bounded, and validated before rendering;
- no perspective or depth cue that could misrepresent rear clearance or the closed back.

## 12. Fixture plan

Add focused front-view fixtures to the existing Designs fixture group exposed by `runDesignGeometryFixtures()` and retain the complete baseline suites documented in `README.md`.

Model agreement fixtures should assert:

- outer front-view width/height equal `dimensions.cabinetOutsideWidth/Height`;
- row and drawer counts equal `metrics.rows/drawerCount`;
- shelf count equals `metrics.shelfCount`;
- shelf bottom elevations equal `metrics.shelfBottomElevations`;
- every front size equals drawer outside width/height;
- total lateral clearance equals the model and each side equals half;
- vertical clearance equals the model;
- row prefixes, drawer panel IDs, and labels match production IDs.

One-row fixtures should assert one front, zero shelves, bottom placement, and a valid SVG. Multi-row fixtures should assert each front and shelf exactly once, bottom-to-top order, no overlap, and deterministic repeated output.

Invalid/edge fixtures should assert no front SVG for invalid models, finite coordinates, valid SVG markup, safe handling of zero/small clearances, and bounded extreme aspect ratios.

Isolation fixtures should assert:

- `result.svg` is byte-identical before and after switching modes;
- front-view markup is not equal to or embedded in production `result.svg`;
- download receives exact production SVG while Finished Front View is selected;
- draft and `localStorage` remain unchanged;
- `backupObject()` contains no preview mode;
- existing Drawer Cabinet signatures and all existing fixture groups remain unchanged.

Session fixtures should cover default mode, switching, reset, template changes, reload/default behavior, and non-Drawer templates. The documented regression targets remain 1275/0 complete baseline, 559/0 Designs geometry, 118/0 Designs production application, 66/0 production settings, 8/0 storage recovery, plus existing Drawer Cabinet signatures.

## 13. Manual browser plan

Use direct `file://` validation with one, two, and three rows. Generate Cut Layout, switch to Finished Front View, verify dimensions/order/shelves/labels/gaps, change lateral and vertical clearances, and switch repeatedly. Apply a saved production setting and confirm ordinary values are reflected. Download while the front view is visible and verify that the downloaded file is still the flattened production SVG with no front-view paths. Open it in LightBurn, reset, reload, and run the dedicated and complete fixture suites.

## 14. Expected files and functions affected

Expected implementation scope:

- `index.html`: additive session-only mode, front-elevation metadata/helper, result-area selector, screen-only SVG renderer, and fixtures;
- `README.md`: only if the preview behavior or fixture command list needs documentation;
- this review/report document and, after implementation, an implementation report.

Expected function touchpoints are `buildDrawerCabinetModel()`, `buildDrawerCabinetDesignResult()`, `renderDesigns()`, `designResultsHtml()`, `refreshDesignPreview()`, reset/template event binding, and `runDesignGeometryFixtures()`.

No change should be required to shared geometry algorithms, panel serialization, storage schema, import/export, Library records, production-setting application, filenames, or `Genmitsu L8 Tracker.dc.html`.

## 15. Risks and deferred behavior

Primary risks are confusing bottom-up physical order with visual top-down order, treating total lateral clearance as one-side clearance, drawing a shelf at the wrong y coordinate, and accidentally routing screen SVG into the download path. Explicit row metadata and isolation fixtures address these risks.

Deferred safe extensions include drawer pulls/knobs, unequal drawer heights, overlay fronts, open-drawer state, a side elevation, related-piece highlighting, a printable assembly diagram, and an exploded view. None should be included in the first implementation.

## 16. Independent-audit recommendation

After implementation, perform an independent audit against this report. The auditor should inspect the diff, compare production SVG bytes with the pre-feature baseline, verify that mode changes do not touch draft/state/storage, inspect one- and three-row rendered views, and run the complete fixture suite. A second reviewer should specifically trace the download path and row/shelf coordinate formulas.

## 17. Explicit conclusion

**READY FOR IMPLEMENTATION.** The existing Drawer Cabinet model is authoritative and sufficiently complete for a safe front elevation. The implementation can remain within screen rendering, transient session state, additive model metadata, result binding, fixtures, and documentation. The production cut SVG, panel layout, IDs, storage, Library, Designs production settings, and export behavior should remain unchanged.

## Review evidence

- `README.md`: standalone offline architecture and current fixture baseline.
- `index.html:405-406`: session-only Design defaults.
- `index.html:2002-2029`: Drawer Cabinet normalization and validation inputs.
- `index.html:2245-2256`: authoritative dimension formulas.
- `index.html:2261-2301`: authoritative model, shelf elevations, IDs, and metrics.
- `index.html:2950-2970`: production result and SVG serialization boundary.
- `index.html:2981-3064`: current result-area renderer and refresh path.
- `index.html:3066-3071`: production download path.
- `index.html:506-520`: backup-object boundary.
- `index.html:1794-1885`: current Designs form and Drawer Cabinet assembly guidance.

