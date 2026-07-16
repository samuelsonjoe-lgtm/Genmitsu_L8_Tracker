# Drawer Cabinet Finished Front View — Independent Audit

Date: 2026-07-16  
Audit type: read-only independent audit of the current uncommitted working tree  
Committed baseline: `9e8377a Apply proven settings in Designs`

## Required conclusion

**NOT SAFE TO COMMIT**

The static implementation review finds no production-SVG or persistence leak and the coordinate formulas are geometrically consistent. However, the required isolated Edge runtime totals and manual visual/download checks could not be obtained, and one minor selector-state defect remains: an active Finished Front View can become visually Cut Layout after the draft becomes invalid while the disabled Finished Front button remains `aria-pressed="true"`. This audit does not modify that behavior.

## 1. Repository and scope

Observed status:

- `git status -sb`: branch `main...origin/main`; `README.md` and `index.html` modified; the implementation and prior reports are untracked, along with many pre-existing unrelated untracked files.
- `git log -1 --oneline`: `9e8377a Apply proven settings in Designs`.
- No staged paths: `git diff --cached --name-only` was empty.
- `git diff --check`: passed, with normal line-ending warnings.
- `git diff --stat`: `README.md` 2 inserted lines; `index.html` 110 changed lines, 107 insertions and 5 deletions. The new audit report is untracked and therefore absent from `git diff --stat`.

No unexpected tracked application files were found. The unrelated untracked files listed by status were not edited, staged, or removed.

## 2. Runtime validation

Attempted fresh isolated Microsoft Edge headless startup with a unique `C:\tmp` user-data directory for direct `file://` startup and self-test URLs. The available Edge invocation exited without producing DOM or screenshot output; the earlier browser attempt also reported an existing Edge profile/process lock. Process inspection was restricted by local access permissions. Therefore:

- Designs geometry: actual total **not obtained**;
- Designs production application: actual total **not obtained**;
- Production settings: actual total **not obtained**;
- Storage recovery: actual total **not obtained**;
- complete suite: actual total **not obtained**.

The implementation report's expected `577`, `118`, `66`, `8`, and `1293` totals are not accepted as runtime evidence. No browser pass count is claimed by this audit.

Static validations that did run:

- Python `html.parser` accepted `index.html`.
- `git diff --check` passed.
- The current diff was inspected against the baseline.

## 3. Authoritative-model agreement

Verified statically:

- `drawerCabinetDimensions()` remains unchanged and continues to derive drawer outside dimensions, cell dimensions, cabinet inside dimensions, and cabinet outside dimensions (`index.html:2252-2266` in the current working tree).
- `buildDrawerCabinetFrontElevation(model)` receives the completed valid Drawer Cabinet model result, not raw `designDraft` strings (`index.html:2312-2349`).
- It reads outside width/height, inside bounds, drawer outside width/height, cell width/height, row count, clearances, and shelf elevations from `model.dimensions`/`model.metrics`.
- Material thickness is recovered from the model's outside-minus-inside width difference and divided by two. This is derived from the authoritative normalized result rather than a second raw-input calculator; adding explicit thickness to the model would be clearer but is not required for correctness here.
- Wall, shelf, and drawer rectangles use that derived thickness; drawer width/height and clearance values come from the model.
- Prefixes and the five drawer panel IDs are generated as `drawer-rNN-{bottom,front,back,left,right}`. Shelf IDs are `cabinet-shelf-rNN`.

The model agreement is structurally sound, but runtime fixture execution was not available to independently prove all three configurations.

## 4. Row ordering and coordinate conversion

The observed formulas are:

```text
perSide = metrics.lateralClearance / 2
bottomInteriorY = t + (row - 1) * (cellHeight + t)
openingPhysicalTop = bottomInteriorY + cellHeight
opening.y = cabinetOutsideHeight - openingPhysicalTop
front.y = cabinetOutsideHeight - bottomInteriorY - drawerOutsideHeight
shelf.y = cabinetOutsideHeight - (t + shelfBottomElevation + t)
```

These formulas correctly convert bottom-up physical positions into top-down SVG coordinates:

- row 1 has the greatest front/opening y and is lowest on screen;
- the final row has the smallest y and is highest;
- `shelfBottomElevations` are used rather than recomputed from raw draft values;
- the front bottom aligns with the opening bottom, leaving the vertical running clearance above the drawer;
- shelves are placed between cells with material-thickness bands.

Static reasoning confirms no reversal, upside-down transform, or extra shelf. The existing fixture additions also compare row y coordinates and check front separation, but browser rendering was not independently observed.

## 5. One-row cabinet

Static path review confirms one row produces one front, zero shelves, row label `Drawer 1 - Bottom`, and the cabinet bottom/top/side wall bands. The bottom drawer's front bottom reaches the inside opening bottom while the outside bottom band remains visible. The production result path is unchanged.

Unverified: actual browser visual appearance, clipping, readable dimensions, and rendered SVG validity in Edge.

## 6. Two-row cabinet

Static path review confirms two rows produce `Drawer 1 - Bottom`, `Drawer 2 - Top`, and exactly one `cabinet-shelf-r01`. The shelf uses the model's first elevation and its width is the model's cabinet inside width. The lateral gap is split symmetrically by the front x coordinate.

Unverified: actual browser appearance and visual readability.

## 7. Three-row cabinet

Static path review confirms three rows produce `Drawer 1 - Bottom`, `Drawer 2 - Middle`, `Drawer 3 - Top`, exactly two shelves, and prefixes `drawer-r01` through `drawer-r03`. The renderer sorts visual row groups by descending row number, while their y coordinates perform the physical top-down placement.

Unverified: actual browser appearance, proportional scaling, and LightBurn inspection.

## 8. Clearance behavior

The implementation uses total lateral clearance only through `perSide = total / 2`, and each row exposes total, per-side, and vertical clearance metadata. Zero values remain zero in the geometry; linework has a minimum screen stroke so the diagram remains visible without changing the physical rectangles. The model's existing validation rejects negative running clearances and impossible clearances before front metadata is created.

Static review finds no NaN/Infinity source in valid front metadata. The new fixture block exercises zero, extreme-valid, and invalid paths. Runtime observation of larger clearances and visual boundedness remains unverified.

## 9. SVG validity and accessibility

Verified in source:

- inline SVG uses a finite numeric viewBox from the model width and height;
- `role="img"`, `title`, and `desc` are emitted;
- visible warning text states the view is screen-only and not included in the downloaded cut file;
- dimensions and drawer count are included in the SVG;
- row groups carry `data-drawer-row` and `data-panel-prefix`;
- hatching, stroke styles, labels, and line patterns provide non-color cues;
- short fronts use compact `D1`-style markers and a legend rather than shrinking all text below the configured minimum.

Finding: the short-front legend is incomplete for some cases. It always describes only `D1 Bottom / Dn Top`; for a three-row short-front cabinet it does not explicitly map `D2` to Middle, and for a one-row short-front cabinet it says both `D1 Bottom` and `D1 Top`. This is a Minor clarity issue.

## 10. Preview selector behavior

Verified statically:

- selector markup is emitted only when `designDraft.template === 'drawer-cabinet'`;
- buttons use `type="button"` and `aria-pressed`;
- Finished Front is disabled when the result is invalid or has no front metadata;
- mode changes assign only the module-local `designPreviewMode` and call `refreshDesignPreview()`;
- no `updateDesignDraft()` or `persist()` call is made by the selector handler;
- rerendering replaces the selector and `bindDesignPreviewActions()` assigns one handler to each current button.

Finding (Minor): if Finished Front is selected and a later input makes the Drawer Cabinet invalid, `designPreviewMode` remains `finished-front`. `frontVisible` becomes false, so the displayed content falls back to Cut Layout, but `designPreviewSelectorHtml()` still marks Finished Front `aria-pressed="true"` while disabling it. The display and control state disagree until reset/template change/mode change.

Recommended correction: normalize the mode to `cut-layout` whenever `result.valid` or `result.frontElevation` is false before producing selector markup, or render the selector's active state from the effective `frontVisible` mode. No fixture currently detects this invalid-after-selection transition.

## 11. Reset and template behavior

Verified statically:

- module-local mode starts as `cut-layout`;
- `resetDesignWorkspaceSession()` resets it;
- the template change handler resets it before rerendering;
- no mode field appears in `freshState()`, `persist()`, or `backupObject()`;
- the mode is not written to `localStorage`.

Fresh reload and repeated browser switching were not runtime-observed.

## 12. Production-SVG isolation

This boundary is verified statically and by the existing fixture design:

- `buildDrawerCabinetDesignResult()` still serializes only the production `partial` containing production panels and label paths;
- `frontElevation` is an additive result property and is not passed to `serializeDesignSvg()`;
- front markup is inserted only by `designResultsHtml()` in the screen result area;
- `downloadCurrentDesignSvg()` still downloads `result.svg` and does not branch on preview mode;
- `designSvg()` still returns `buildDesignResult(d).svg`;
- no front SVG class, screen warning, dimension markup, or front data attributes are routed to production serialization;
- `STORAGE_KEY` remains `genmitsu-l8-tracker-v1` and `SCHEMA_VERSION` remains `2`.

Existing production signature assertions remain in the working source, including the known one-row and three-row default signatures (`4456/a6dd23dc` and `9153/8c286797`) and the explicit `0.40` compatibility signatures. Runtime re-execution of those assertions was not available, so this is static confirmation only.

## 13. Download isolation

The new fixture creates a temporary form/result/download DOM seam, invokes `downloadCurrentDesignSvg()` while `designPreviewMode` is `finished-front`, intercepts the text callback, and compares it to the production result. That is a real function-path test rather than a string self-comparison.

The fixture does not independently verify filename and MIME type, although the unchanged download implementation still uses `l8-${fileSlug(designDraft.template)}-${today()}.svg` and `image/svg+xml;charset=utf-8`. LightBurn opening was not performed.

No static production-source path can route the inline front SVG into the download callback.

## 14. Data isolation

Verified statically and in fixture code:

- mode is module-local;
- selector code does not touch `designDraft`, `state`, profiles, or storage;
- `backupObject()` has no preview field;
- download uses the ordinary design form/result path;
- fixture snapshots `localStorage` and `state.profiles` around preview rendering and snapshots the backup object.

The fixture checks actual `localStorage.getItem(STORAGE_KEY)` equality and profile JSON equality. It does not snapshot the full `designDraft` around every mode switch, and it does not cover repeated reset/template/download combinations in one test. Those are fixture gaps, not observed leaks.

## 15. Invalid drafts

The model returns no front metadata when invalid, and `designResultsHtml()` displays the existing `Preview unavailable` block for invalid results. The Finished Front button is disabled for invalid results. The invalid fixture invokes an invalid row-count path and checks that no front SVG is produced.

Finding: the invalid-after-active-mode selector state described in section 10 is a Minor presentation defect. It does not render a misleading front geometry and does not bypass validation.

## 16. Extreme proportions

The SVG wrapper is width-responsive with `max-height:620px` and a smaller media-query maximum. The viewBox preserves aspect ratio. The renderer uses model coordinates without scale distortion and switches to compact labels for short fronts.

Unverified: actual narrow/tall and wide/short browser screenshots, horizontal overflow behavior, and readability at the extremes.

## 17. Production-setting interaction

Static review finds no direct read of Library production records in the front renderer. Production-setting application still updates ordinary `designDraft` values, then the next `buildDesignResult()` derives front metadata from the resulting normalized model. The preview mode itself does not alter production-setting session state.

Runtime application of thickness, cabinet-joint, drawer-joint, and lateral-clearance settings was not performed in this audit.

## 18. Fixture quality

The 18 new assertions classify as:

- **Independent:** 10. These check y-coordinate ordering, SVG metadata, one-row shelf absence, overlap calculations, invalid-path absence, and actual temporary download interception.
- **Partially circular:** 8. These compare additive front metadata to the same model/result values that supplied it, or inspect renderer output generated from the metadata. They are useful consistency checks but do not independently recompute all authoritative values.
- **Circular:** 0 identified. The download assertion exercises `downloadCurrentDesignSvg()` with a temporary DOM seam; it is not merely `frontMarkup === expectedMarkup`.

Missing high-value cases:

- invalid-after-active-mode should assert the selector returns to Cut Layout or removes the active Finished state;
- filename and MIME should be captured by the download fixture;
- full `designDraft`, full backup JSON, and storage snapshots should be checked around repeated mode/reset/template sequences;
- front SVG viewBox numeric validity and parser error detection should be explicit rather than only checking `nodeName`, title, description, and no invalid numeric token;
- production setting application and fresh reload behavior need runtime tests.

## 19. Protected boundaries

Comparison of the current diff against baseline confirms:

- `drawerCabinetDimensions()` has no changed lines;
- existing production panel points, IDs, layout row arrays, and serialization calls are unchanged;
- `serializeDesignSvg()`, `designSvg()`, and `buildDesignResult()` dispatch semantics are unchanged;
- `normalizeDesignDraft()` is unchanged;
- download filename generation is unchanged;
- `STORAGE_KEY`, `SCHEMA_VERSION`, `persist()`, `backupObject()`, import/replace/merge paths, Library save paths, and production-setting application have no feature changes;
- only the Drawer Cabinet model's returned object gains additive transient metadata, and only the Drawer Cabinet result gains that metadata;
- no 3D, pulls, knobs, overlay fronts, unequal drawers, printable sheets, exploded views, or exported front SVG path was introduced.

## 20. Findings summary

### Minor — stale selector state after invalidation

- Area: `designPreviewSelectorHtml()`, `designResultsHtml()`.
- Scenario: select Finished Front, then change a Drawer Cabinet input to an invalid value.
- Expected: Cut Layout is visibly active, or the Finished Front option is omitted/clearly inactive.
- Observed: content falls back to Cut Layout, while Finished Front may remain `aria-pressed="true"` and disabled.
- Consequence: accessible state and visible content disagree; no production or geometry leak.
- Recommended correction: normalize the mode when the result becomes invalid and add a fixture.
- Detected by current fixtures: **No**.

### Minor — incomplete compact-label legend

- Area: `buildDrawerCabinetFrontElevationSvg()`.
- Scenario: very short three-row fronts, or very short one-row front.
- Expected: every compact marker has an unambiguous row/position mapping.
- Observed: the legend only names bottom and top, and the one-row wording names the same drawer as both bottom and top.
- Consequence: reduced clarity at an edge case; geometry remains truthful.
- Recommended correction: generate a complete row legend from `frontElevation.rows`.
- Detected by current fixtures: **Partially**; compact output is exercised, but legend semantics are not asserted.

### Verified — production and persistence boundaries

- Area: model/result/render/download/storage paths.
- Scenario: normal Cut Layout and Finished Front selection.
- Observed: front metadata and inline SVG remain outside production serialization; mode is module-local and absent from state/backup/persistence.
- Consequence: no static Blocker or Major boundary violation found.
- Runtime confirmation: **not obtained** because browser execution was unavailable.

## Future boundary

The implementation remains a 2D screen-only front elevation. No 3D rendering, pulls, knobs, scoops, overlay fronts, unequal drawer heights, printable assembly sheets, exploded views, or exported front-view SVGs were introduced.

## Final handoff

The implementation is structurally close to safe: authoritative-model reuse, bottom-up coordinate conversion, production SVG isolation, and session-only state are all sound in the source. Before commit, resolve the stale selector state and compact legend wording, then run the isolated Edge fixture/manual validation and record actual totals rather than expected totals.

