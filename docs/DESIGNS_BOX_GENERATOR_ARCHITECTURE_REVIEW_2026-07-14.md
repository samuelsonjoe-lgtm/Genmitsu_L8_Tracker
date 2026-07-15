# Genmitsu L8 Tracker — Designs Tab Architecture and Geometry Review

**Review type:** Read-only static architecture and geometry review  
**Repository:** `C:\Genmitsu L8 Tracker` / `samuelsonjoe-lgtm/Genmitsu_L8_Tracker`  
**Reviewed baseline:** `6a2757972713a8ff495b6a6cbd965388be180c73 — Add inventory-guided project wizard`  
**Reported fixture baseline:** `532 passed / 0 failed`  
**Date:** 2026-07-14  

## Review constraints and verification status

- No repository files were edited, staged, committed, or pushed.
- The synchronized GitHub copy at commit `6a27579` was reviewed because this environment cannot directly open Joe's local `C:\Genmitsu L8 Tracker` working tree.
- This was a static review of `index.html`, `README.md`, the retained `Genmitsu L8 Tracker.dc.html` reference artifact, relevant commit history, and the built-in fixture dispatch.
- The existing `532 passed / 0 failed` result is documented in the reviewed baseline, but the fixtures were not re-run in this environment.
- The retained `.dc.html` artifact contains no earlier box generator. The existing tray generators appear to be the partially completed “box-like” design work remembered from the earlier phase.

---

## Executive conclusion

The safest bounded Phase 2 is **not** a storage-backed CAD subsystem. It should remain a session-only, offline SVG generator and add:

1. A shared, pure geometry-result pipeline for every Designs template.
2. Exact-SVG inline preview, calculated dimensions, warnings, and blocking validation.
3. A new rectangular finger-jointed box template with internal/external dimension modes.
4. Open-top and simple loose flat-lid options only.
5. Deterministic panel layout and design-specific fixture coverage.

Phase 2 should **defer** saved presets, automatic Inventory/Library material selection, Project creation, sliding lids, hinges, captive lids, dividers, nesting, and baked-in kerf offsets. Those additions would expand the storage schema, import/export contract, cross-tab coupling, and geometry risk well beyond a bounded first box implementation.

The new box generator should use a new generic orthogonal finger-joint edge engine. It should **not** reuse `designWallPath()` as the primary box-joint primitive because that helper creates only bottom tabs and cannot generate complementary corner fingers.

---

## 1. Current Designs architecture

### 1.1 UI and state

The Designs tab is rendered by `renderDesigns()` and currently exposes four templates:

- `qr-stand`
- `hanging-sign`
- `dice-tray`
- `divider-tray`

The editable values live in the module-level `designDraft`, initialized by `designDefaults()`. The form updates that draft on input/change, and a template change triggers a full render.

The current defaults include:

- Material thickness: `3 mm`
- Fit clearance: `0.15 mm`
- Sign dimensions
- Tray inside width/depth and wall height
- Divider count
- Hanging-hole values

The design draft is intentionally outside the persisted `state` object. It is not written by `persist()` or returned by `backupObject()`. The README explicitly states that the Designs form is session-only and is not included in JSON backups.

### 1.2 Export flow

The current flow is:

1. Read form values into `designDraft`.
2. Dispatch through `designSvg()`.
3. Generate an SVG string.
4. Download it through `downloadTextFile()` as `image/svg+xml`.

The filename pattern is:

`l8-<template>-<date>.svg`

There is no intermediate design result object, no error collection, no warning collection, no geometry metadata, and no preview.

### 1.3 SVG document helper

`designSvgDocument(width, height, shapes)` creates a valid-looking standalone SVG with:

- millimeter document width and height;
- a matching numeric `viewBox`;
- `fill="none"`;
- red stroke;
- `stroke-width="0.1"`;
- all shapes in one group.

This is appropriately simple for LightBurn, but it lacks:

- panel grouping or IDs;
- design metadata;
- calculated bounds checking;
- independent cut/guide layers;
- evidence that every shape remains inside the viewBox;
- a parse check before download.

### 1.4 Validation behavior

`designNumber(value, fallback, minimum)` silently replaces invalid or undersized values with a fallback. HTML `min` attributes provide browser hints, but download does not call `checkValidity()` and geometry generation has no blocking error path.

Consequences:

- Invalid values can export a different design than the user entered without explaining the substitution.
- Extremely large values are generally unbounded.
- Cross-field impossibilities are not detected.
- `NaN`, overlapping features, off-panel holes, or self-intersecting paths are possible even when each individual field passes its HTML minimum.

For Phase 2, fallback coercion is acceptable only for initial defaults. User-entered geometry should be normalized into either a valid result or explicit blocking errors.

---

## 2. Existing generator geometry findings

## 2.1 Freestanding QR/sign stand

`designStandSvg()` exports two rectangular pieces with centered open slots.

### Positive aspects

- Uses measured material thickness plus fit clearance for slot width.
- Caps slot depth against base and upright dimensions.
- Produces a compact two-piece layout.

### Risks and likely defect

Both the base and upright paths create their slot from the same corresponding edge. Two identical edge-open slots do not form the usual cross-lap arrangement unless one part is intentionally rotated and the slot orientations still complement one another. The current geometry should be manually assembled from scrap before treating it as proven.

Additional issues:

- Slot width can exceed panel width, producing a negative `slotLeft`.
- The geometry silently falls back rather than reporting invalid input.
- There is no preview to reveal the orientation or fit.

This should not block the box generator, but it should receive a targeted fixture and physical test before future documentation calls it a proven stand.

## 2.2 Hanging sign

`designHangingSignSvg()` exports one rounded rectangle and two circular holes.

### Positive aspects

- Simple and appropriate LightBurn geometry.
- Attempts to clamp the hole inset.

### Risks

- Corner radius is not capped to half the smaller panel dimension.
- Hole diameter is not capped relative to the panel.
- The calculated `safeInset` can become negative when the hole or panel dimensions are impossible.
- Hole circles can therefore leave the panel or overlap its edge.
- No minimum web thickness is enforced between hole and panel edge.

Phase 2 validation infrastructure should cover this template without otherwise redesigning it.

## 2.3 Dice tray and divider tray

`designTraySvg()` creates:

- one slotted rectangular base;
- two front/back wall panels;
- two side wall panels;
- optional full-width divider slots and divider rectangles.

`designWallPath()` creates a rectangular wall with tabs only along its bottom edge.

### What the tray actually implements

The current `jointStyle` control changes the number of bottom tabs:

- `finger` -> four bottom tabs;
- `tab-slot` -> two bottom tabs.

It does **not** generate alternating finger joints at the four vertical tray corners. The walls remain plain vertical rectangles and rely on glue at the corners. The interface text partly acknowledges this by recommending corner glue, but “finger” can still be misread as a fully finger-jointed tray.

### Geometry concerns

1. **Overlapping tabs at small dimensions**  
   Tab width is clamped to at least 8 mm even when the edge is too short. For a short edge with four tabs, tab positions can be closer together than the tab width, causing overlap or a self-intersecting wall path.

2. **No impossible-dimension rejection**  
   The current minimum tray width/depth of 20 mm is not enough to guarantee four non-overlapping tabs, usable wall joints, or a sensible interior after material thickness is considered.

3. **Clearance is slot-only**  
   `fitClearance` is added directly to slot width. Tabs retain nominal material thickness. That is understandable, but it is not documented as a distinct joint-fit model and it is not symmetric around the nominal slot centerline.

4. **Slot placement is not consistently centered**  
   Clearance is added to slot width after offsets are calculated from material thickness. This shifts some slot geometry inward rather than widening it evenly around a shared nominal joint line.

5. **Not reusable for box corners**  
   `designWallPath()` knows only one tabbed edge and cannot express complementary alternating fingers on top, bottom, left, and right edges.

6. **Divider semantics are simple but incomplete**  
   Divider panels are plain rectangles inserted into full-width base slots. There are no wall notches, removable-divider retention details, or clearance on divider panel length. This is acceptable for an experimental tray but should not be generalized into a box-divider system yet.

### Architectural conclusion from tray review

The tray generator is useful as an export prototype, but its helper should not be promoted into the general box geometry foundation. The box phase needs a new shared edge-pattern representation and should leave existing tray output unchanged unless a separate, specifically tested correction is approved.

---

## 3. Material thickness, fit clearance, and kerf

## 3.1 Material thickness in Designs

Designs uses its own millimeter-only `materialThickness` value. It does not use the app's stored unit preference, Inventory parsing, Library profiles, or Project material fields.

Elsewhere, the tracker supports `mm`/`in` conversion and stores thickness with an explicit unit. Designs deliberately presents all dimensions in millimeters, which is appropriate for SVG and LightBurn geometry.

Recommendation: keep the box generator millimeter-native in Phase 2. Do not add unit conversion to Designs during the same change.

## 3.2 Fit clearance

The current positive clearance means “make the receiving slot wider.” That convention is understandable and should remain directionally consistent:

- `0.00 mm` = nominal material-thickness fit;
- positive value = looser fit;
- negative values should remain disallowed in the UI for the first implementation.

For true finger joints, clearance should be applied at **internal finger transitions**, not by moving the outer panel envelope. A safe rule is:

- widen each receiving recess by `clearance / 2` at each side;
- narrow the corresponding finger by the same total amount;
- keep exterior panel boundaries fixed;
- reject clearance values that collapse a finger or recess.

The preview should display the actual generated finger width after adjustment.

## 3.3 Kerf

The app stores kerf offset in Log, Library, Test Grids, and Project setting snapshots, but Designs does not use it. That separation is currently safer than automatically borrowing a saved kerf value.

### Phase 2 recommendation

Do **not** bake general kerf compensation into the box SVG.

Reasons:

- Correct kerf compensation is a polygon-offset problem, not the same operation as widening a slot.
- Exterior and interior contours require opposite offset directions.
- LightBurn already supports kerf offset at the cut layer.
- Saved Library kerf values may be material-, thickness-, focus-, power-, speed-, and pass-specific.
- Automatic Library coupling could silently produce a design for the wrong material or cut process.

The Designs UI should clearly state:

> SVG geometry is nominal centerline geometry. Joint clearance changes the finger fit. Apply and verify kerf compensation separately in LightBurn using a material-specific test.

A future phase can add optional baked-in kerf only after a tested polygon-offset implementation exists.

---

## 4. Fixture and validation coverage

The built-in fixture dispatcher currently includes:

- baseline resolution;
- material-test normalization;
- Test Grid promotion;
- Grid Browser;
- Material Browser;
- Library Browser;
- Project Browser;
- Project Wizard;
- wizard metadata;
- storage recovery.

There is no Designs or SVG geometry fixture group. The documented `532 passed / 0 failed` baseline therefore does not verify the current stand, hanging sign, tray, divider, or SVG export geometry.

Phase 2 should add a separate `design` fixture group and include it in `selftest=all`.

---

## 5. Recommended bounded Phase 2 box design

## 5.1 Scope statement

Add one new template, tentatively `finger-box`, to the existing session-only Designs tab. It generates a rectangular box body using six possible panels:

- bottom;
- front;
- back;
- left;
- right;
- optional loose lid.

The box body uses true complementary rectangular finger joints at:

- all four vertical corners;
- all four bottom-to-wall edges.

The loose lid is a separate plain panel and is not hinged, sliding, captured, or finger-jointed.

## 5.2 Dimension basis

Provide a required selector:

- `Inside dimensions`
- `Outside body dimensions`

Recommended default: **Inside dimensions**, because storage boxes are usually designed around the object that must fit.

Inputs:

- Width
- Depth
- Height
- Material thickness
- Joint clearance
- Preferred finger width
- Lid option

### Dimension definitions

Let:

- `t` = measured material thickness;
- `Wi`, `Di`, `Hi` = internal width, depth, height;
- `Wo`, `Do`, `Ho` = external body width, depth, height.

For this body construction:

- `Wo = Wi + 2t`
- `Do = Di + 2t`
- `Ho = Hi + t`

And inversely:

- `Wi = Wo - 2t`
- `Di = Do - 2t`
- `Hi = Ho - t`

Definitions must be shown next to the selector:

- External body height is measured from the underside of the bottom panel to the top of the walls.
- Internal height is measured from the inside face of the bottom panel to the top of the walls.
- A loose lid sits on top and adds one material thickness to the overall closed height.

This avoids ambiguous “overall height including lid” behavior in the first phase.

## 5.3 Lid options

Include only:

1. `Open top`
2. `Loose flat lid`

The loose lid should default to the external body footprint (`Wo × Do`) and may offer a small nonnegative overhang input later, but overhang is not necessary for the first implementation.

Optional safe convenience: a centered semicircular finger notch in one lid edge. This can be deferred if it complicates the initial geometry.

Explicitly defer:

- sliding lids and grooves;
- hinged lids;
- living hinges;
- magnetic or captured lids;
- inset plug lids and glued locating rails;
- snap fits;
- finger-jointed removable lid frames.

Those require additional fit models, assembly assumptions, hardware, grain/orientation decisions, or fragile geometry.

## 5.4 Finger-joint model

Use one joint style only: alternating rectangular fingers.

Do not expose the current tray `jointStyle` choices for the box template. “Tab-slot” and “finger” should not share one ambiguous implementation.

### Shared edge pattern

For each mating edge length:

1. Determine a valid odd segment count of at least three.
2. Calculate the exact segment width as `edgeLength / segmentCount`.
3. Alternate full-edge and recessed segments.
4. Generate the mating edge with the opposite phase.
5. Apply clearance only at internal transitions.

An odd segment count makes the edge pattern deterministic and allows both ends to share the intended phase. The implementation must define and fixture the corner parity for every panel.

### Preferred finger width

Treat the entered finger width as a target, not an exact promise. The generator should display:

- requested finger width;
- actual generated width;
- segment count for width, depth, and height edges.

Suggested initial default: `12 mm` for 3 mm plywood.

Suggested minimum actual segment width:

`max(2 × materialThickness, 4 mm)`

A design is impossible when an edge cannot support at least three valid segments at the minimum width.

## 5.5 Panel geometry

Use a new orthogonal panel-outline helper rather than modifying `designWallPath()` into a many-purpose function.

Suggested architecture:

```text
normalizeDesignDraft(rawDraft)
  -> validated numeric box parameters

buildBoxModel(params)
  -> internal/external dimensions
  -> mating-edge patterns
  -> panel definitions
  -> errors/warnings/metrics

buildFingerPanel(panelDefinition)
  -> closed orthogonal path
  -> local bounds

layoutDesignPanels(panels)
  -> translated panel paths/groups
  -> document bounds

serializeDesignSvg(result)
  -> standalone SVG string
```

Each panel definition should name its four edges and their pattern phase. The geometry engine should produce only horizontal and vertical path commands.

The outer extents of the assembled body must remain equal to `Wo × Do × Ho`; clearance must not expand those external dimensions.

## 5.6 Panel layout

Use a deterministic non-nesting layout. Do not attempt automatic sheet optimization in Phase 2.

Recommended layout:

- Row 1: bottom and optional lid.
- Row 2: front and back.
- Row 3: left and right.

Use a fixed margin and panel gap, then calculate the document width/height from actual panel bounds.

Requirements:

- no panel bounding-box overlaps;
- no negative coordinates after translation;
- every path remains inside the viewBox;
- panel orientation is stable between exports;
- the preview uses the exact exported layout.

Do not export visible text labels on the cut layer. Panel names may be stored as group IDs or SVG `<title>` metadata. The web preview can overlay HTML labels without adding cut geometry.

## 5.7 Impossible-dimension handling

Generation must return structured errors instead of silently applying fallback values.

Block download when any of these occurs:

- a required number is blank, non-finite, zero, or negative;
- material thickness is invalid;
- external width or depth is `<= 2t`;
- external height is `<= t`;
- calculated internal dimensions are nonpositive;
- an edge cannot support at least three valid finger segments;
- clearance collapses a finger or recess;
- any generated point is non-finite;
- any panel bounds exceed the document bounds;
- panel layout boxes overlap;
- SVG parsing reports an error.

Warnings, rather than blocking errors, can cover:

- very loose clearance;
- finger width unusually small relative to thickness;
- very large SVG layout relative to the L8 work area;
- a loose lid with no retention;
- user dimensions that create a box larger than common sheet blanks.

The app should never silently replace impossible user dimensions with default dimensions at export time.

## 5.8 SVG preview

Add an inline preview panel that renders the **same serialized SVG** used for download.

The preview should include:

- exact cut layout;
- calculated internal and external body dimensions;
- closed height when a lid is selected;
- panel count;
- layout width and height;
- actual finger counts and widths;
- errors and warnings;
- a disabled Download button when invalid.

Avoid a second preview-only geometry implementation. One result object must feed both preview and download.

A lightweight `refreshDesignPreview()` can update only the preview/results region during input. This avoids full-tab rebuilds and reduces focus loss, following the pattern already used by other tracker tabs.

---

## 6. Recommended Phase 2 architecture changes

## 6.1 Introduce a result object

Instead of every generator returning only an SVG string, return a structure such as:

```js
{
  valid: true,
  errors: [],
  warnings: [],
  svg: "...",
  widthMm: 0,
  heightMm: 0,
  panels: [],
  metrics: {}
}
```

`designSvg()` can remain as a compatibility wrapper if useful, but the UI should work from the structured result.

## 6.2 Separate normalization, geometry, layout, and serialization

Do not mix form fallback handling, physical dimension formulas, path generation, layout, and XML assembly in one function. Pure functions make the geometry fixtureable without a DOM.

## 6.3 Preserve existing storage behavior

Keep `designDraft` session-only in Phase 2. Do not add it to `state`, `persist()`, backup/export, or import normalization.

This avoids:

- changing `SCHEMA_VERSION`;
- backup compatibility work;
- merge/replace import semantics;
- storage-recovery fixture changes;
- permanent storage of experimental geometry.

## 6.4 Preserve existing template output unless explicitly fixing it

The new preview/validation pipeline may wrap existing generators, but do not casually rewrite the QR stand, hanging sign, or tray geometry in the same implementation.

Recommended implementation sequence:

1. Add result/preview/validation infrastructure while snapshotting current valid legacy SVG output.
2. Add the new box edge engine and box template.
3. Add targeted fixes to legacy templates only when backed by explicit fixtures and a documented reason.

This keeps diffs reviewable and makes regressions easier to isolate.

---

## 7. Saved presets and cross-tab integration decision

## 7.1 Saved design presets

**Defer.**

Saved presets would require a new persisted collection, normalization, import/export support, merge behavior, delete/undo behavior, and storage fixtures. The geometry should first prove stable through actual box cuts.

A later phase can add a dedicated `designPresets` collection rather than overloading Projects or Library.

## 7.2 Library integration

**Defer automatic integration.**

Library profiles represent tested laser settings for a material/thickness/operation. A design preset represents geometry. They are related but not the same record type.

Potential later integration:

- user explicitly chooses a Library cut profile;
- Designs copies measured thickness and displays the saved kerf value as a suggestion;
- no automatic geometry change occurs without confirmation.

Do not silently inherit the closest Library profile in Phase 2.

## 7.3 Inventory integration

**Defer.**

Inventory can eventually suggest measured thickness, sheet dimensions, and stock availability. That would require material selection, confidence handling, and sheet-layout constraints. It is not needed to prove the box geometry.

## 7.4 Project integration

**Defer.**

A generated SVG is not yet a completed Project, and Projects currently track production settings, accounting, photos, and results rather than editable design geometry.

A later explicit action could create a Project draft containing:

- design template name;
- dimensions;
- exported filename;
- selected material/profile references;
- notes.

That should happen only after a stable design preset format exists.

---

## 8. Required fixture plan

Add `runDesignGeometryFixtures()` and expose it as:

- console function `runDesignGeometryFixtures()`;
- query value `?selftest=design`;
- part of `?selftest=all`.

Minimum fixture categories:

### 8.1 Legacy generators

- Defaults for all existing templates generate parseable SVG.
- No output contains `NaN`, `Infinity`, or `undefined`.
- Document dimensions are positive.
- Every default shape remains within the viewBox.
- Valid legacy default output remains stable if geometry was not intentionally changed.

### 8.2 Dimension conversion

- Internal -> external formulas.
- External -> internal formulas.
- Round-trip conversion within numeric tolerance.
- Lid closed height calculation.

### 8.3 Finger patterns

- Segment count is odd and at least three.
- Actual segment widths are positive.
- Mating edges have equal segment boundaries and opposite phases.
- Clearance changes fit transitions monotonically without changing outer panel bounds.
- Corner parity is deterministic.

### 8.4 Panel generation

- Open-top box yields five named panels.
- Loose-lid box yields six named panels.
- Each path is closed and orthogonal.
- All coordinates are finite.
- Calculated panel bounds match expected nominal panel dimensions.

### 8.5 Layout

- No panel bounding boxes overlap.
- Margin and gap are respected.
- All translated paths are nonnegative.
- Document bounds contain every panel.
- Repeated input generates byte-stable output.

### 8.6 Invalid input

- Blank and nonnumeric dimensions block export.
- External dimensions too small for material thickness block export.
- Too-short edges for minimum finger count block export.
- Excessive clearance blocks export.
- Invalid lid option or dimension mode normalizes safely or reports an error.

### 8.7 SVG serialization

- `DOMParser` reports no parser error.
- SVG width/height use millimeters.
- `viewBox` matches calculated dimensions.
- Cut geometry remains in a single predictable cut group.
- No preview-only labels leak into the exported cut geometry.

---

## 9. Manual verification plan

After implementation, verify from a clean direct-file session:

1. Open `index.html` directly without a server.
2. Confirm all eight tabs still function.
3. Confirm Designs remains usable with storage disabled or empty.
4. Enter valid inside dimensions and verify calculated outside dimensions.
5. Switch to outside dimensions and verify the inverse values.
6. Try impossible values and confirm Download is disabled with a specific error.
7. Confirm preview updates without losing focus.
8. Export open-top and loose-lid SVGs.
9. Import both into LightBurn.
10. Confirm document dimensions, panel count, closed paths, and red cut lines.
11. Confirm no duplicate or overlapping cut segments.
12. Cut a small scrap prototype using measured plywood thickness.
13. Evaluate joint fit at `0.00`, `0.10`, and `0.15 mm` clearance before trusting production dimensions.
14. Confirm existing QR stand, hanging sign, dice tray, and divider tray exports still open in LightBurn.
15. Run `index.html?selftest=all` and report the new exact pass/fail count.

Never treat a mathematically valid SVG as a proven physical fit without a scrap test; plywood thickness, glue layers, focus, kerf, moisture, and flatness can alter assembly.

---

## 10. Suggested implementation boundaries

### In scope

- Shared design result model.
- Inline exact-SVG preview.
- Blocking validation and warnings.
- Parametric rectangular box.
- Inside/outside body dimensions.
- Open top and loose flat lid.
- True rectangular finger joints on body corners and bottom.
- Joint clearance separate from kerf.
- Deterministic panel layout.
- Design geometry fixtures.
- README update describing the new template and limitations.

### Out of scope

- Persistence or saved presets.
- Storage schema changes.
- Project, Library, Inventory, or Pricing writes.
- Automatic material/profile selection.
- Baked-in kerf compensation.
- Sheet nesting or optimization.
- Sliding, hinged, living-hinge, magnetic, captive, or snap-fit lids.
- Internal dividers.
- DXF, LightBurn native, or G-code export.
- Changes to unrelated tracker tabs.
- Large rewrite of existing generators.

---

## 11. Risk assessment

### Highest-risk areas

1. Complementary edge parity at all twelve box-body mating edges.
2. Maintaining exact intended internal/external dimensions after joint construction.
3. Clearance adjustment without changing panel envelopes or collapsing segments.
4. Corner transitions where two patterned edges meet.
5. SVG bounds and duplicate cut-line prevention.
6. Refactoring existing export code without changing legacy output.

### Moderate-risk areas

- Live preview focus behavior.
- LightBurn interpretation of grouped paths.
- Large layouts exceeding the usable bed or material sheet.

### Low-risk areas

- Adding the box option to the template selector.
- Session-only draft fields.
- Loose lid rectangle.
- Filename generation.

---

## 12. Audit recommendation

An independent Grok Code/Build or Claude read-only audit **is worthwhile after the implementation and full fixture run, before committing**. This phase introduces shared SVG helpers and nontrivial complementary geometry; a second model should specifically inspect edge parity, physical dimension formulas, clearance semantics, path self-intersections, duplicate segments, and viewBox bounds.

An extra audit is unnecessary before implementation because this review already defines a bounded design. It becomes valuable once there is concrete geometry code and fixture evidence to challenge.

---

## Final recommendation

Proceed with a session-only Phase 2 box generator built on a new pure finger-edge geometry engine and a shared preview/validation result pipeline. Default to internal dimensions, support conversion to external body dimensions, generate a five-panel open-top body plus an optional loose flat lid, keep kerf in LightBurn, and reject impossible geometry rather than silently falling back.

Do not add saved presets or cross-tab integration until physical test cuts confirm the geometry and the design record format is stable.
