# Designs Finished Views Architecture Review

Date: 2026-07-17  
Committed baseline: `c46fb40 Record machine identity on test grids`  
Scope: read-only architecture review and implementation planning  
Status: **READY FOR IMPLEMENTATION**

No application files were changed. Existing modified and untracked worktree items were preserved.

## 1. Executive recommendation

Use a small hybrid architecture:

- Keep `Cut Layout` as the authoritative exported preview for every template.
- Add a per-template preview selector only where a trustworthy assembled view exists.
- Reuse a few safe screen-only primitives for labels, dimensions, panel outlines, walls, lids, and simple depth cues.
- Keep the actual finished-view composition in a small renderer owned by each assembled template.
- Do not build a global 3D scene graph, generic CAD assembly layer, or shared fake-perspective engine in this phase.

The first implementation batch should be:

1. Finger-jointed box — top-down or restrained pseudo-isometric assembled view.
2. Sliding-lid finger box — closed/open lid states using the existing authoritative panel and dimension model.

These are the best next targets because both already have structured panel models, named components, dimensions, and assembly guidance. They offer high user value for boxes and storage while avoiding the larger risk of deriving a view from legacy flattened SVG strings.

Dice Tray and Divider Tray should follow only after their legacy generator is split into an authoritative tray model. A finished view built by guessing wall positions from the current SVG would create false confidence about orientation and assembly.

## 2. Current template inventory

The current Designs selector contains eight exact templates:

| Exact template | Generator / model / result functions | Current preview / assembly status | Finished View readiness | Proposed view | Difficulty | User value | Priority |
|---|---|---|---|---|---|---|---|
| Freestanding QR / sign stand (`qr-stand`) | `designStandSvg()` → `legacyDesignSvg()` → `buildLegacyDesignResult()` | Two flat parts: base and upright, intended to slot together; no structured assembly model | Insufficient for a trustworthy view; current output is only SVG strings | No Finished View; Cut Layout is sufficient | N/A | Low for an assembled preview; the slot relationship is simple but not modeled | No Finished View |
| Hanging sign (`hanging-sign`) | `designHangingSignSvg()` → `legacyDesignSvg()` → `buildLegacyDesignResult()` | One flat rounded panel with two holes; not an assembled object | No assembly view is needed | No Finished View; Cut Layout is sufficient | N/A | Low | No Finished View |
| Dice tray (`dice-tray`) | `designTraySvg()` → `legacyDesignSvg()` → `buildLegacyDesignResult()` | Assembled tray concept: slotted base plus four walls; emitted as flat SVG geometry | Not sufficient yet; no panel/component model or stable wall orientation object | Top-down assembled tray with optional shallow perspective | Medium after model extraction | High for practical storage and tabletop use | P2 |
| Divider Tray (`divider-tray`) | `designTraySvg()` → `legacyDesignSvg()` → `buildLegacyDesignResult()` | Tray plus removable divider-related geometry; current output is flattened SVG shapes | Not sufficient yet; divider count is known but divider placement/components are not represented as an assembly model | Top-down assembled organizer with divider slots/dividers | Large after model extraction | High for desk and drawer organization | P2 |
| Finger-jointed box (`finger-box`) | `buildBoxModel()` → `buildBoxDesignResult()` → `buildDesignResult()`; `buildFingerPanel()` and `layoutDesignPanelRows()` support the cut result | Five-panel open-top box, optionally with a separate loose flat lid; named panels and dimensions are structured | Sufficient for a restrained view; authoritative model already contains panel IDs, dimensions, points, edges, lid state, and warnings | Top-down assembled view for open top; pseudo-isometric for loose-lid state | Medium | High; directly answers what the box looks like and where the loose lid sits | P1 |
| Sliding-lid finger box (`sliding-lid-box`) | `buildSlidingLidBoxModel()` → `buildSlidingLidDesignResult()` → `buildDesignResult()`; optional `buildSlidingCouponModel()` | Five body panels, shouldered sliding lid, two inner channel rails, optional guide marks, labels, and coupon | Sufficient for a focused view; model exposes dimensions, Front/Back direction, rail data, lid engagement, and optional components | Isometric or pseudo-3D with Finished Closed / Finished Open | Medium-Large | Very high; orientation and lid direction are the main assembly questions | P1 |
| Drawer Cabinet (`drawer-cabinet`) | `buildDrawerCabinetModel()` → `buildDrawerCabinetDesignResult()`; `buildDrawerCabinetFrontElevationSvg()` | Multi-panel cabinet with one to three drawers, shelves, running clearances, and named rows | Already sufficient and implemented | Front elevation plus existing Cut Layout | Existing Finished View | Very high for storage and drawer fit orientation | Existing |
| Joint Fit Coupon (`joint-fit-coupon`) | `buildJointCouponModel()` → `buildJointCouponDesignResult()`; `buildFingerPanel()` and `layoutDesignPanelRows()` | Multiple flat A/B test coupons assembled at 90 degrees for fit testing, not a finished product | Deliberately not a Finished View target; its value is explicit flat test layout and pair identity | No Finished View; Cut Layout and assembly instructions are sufficient | N/A | High as a test artifact, low as a finished-object preview | No Finished View |

### Inventory conclusions

The templates divide cleanly into three groups:

- Structured assembled products ready for a view: Finger Box, Sliding Lid Box, and Drawer Cabinet.
- Assembled concepts whose generator is currently only a flattened SVG: Dice Tray and Divider Tray.
- Flat artifacts or simple flat-part products where Cut Layout is the correct primary representation: QR stand, Hanging sign, and Joint Fit Coupon.

All eight templates currently use the Designs result area’s exact exported SVG preview (`designResultsHtml()` with an SVG object/data URL) rather than a separate assembled renderer. Only Drawer Cabinet adds a second screen-only renderer today; its `Finished Front View` is not serialized or downloaded.

The QR stand is technically assembled, but a view would add little value and would require deciding how to depict the slot without an authoritative part model. It should not drive a general Finished View abstraction.

## 3. Current Drawer Cabinet implementation review

The Drawer Cabinet is the strongest existing pattern and should be treated as a narrow precedent, not copied as a universal renderer.

### Authoritative source

`buildDrawerCabinetModel()` owns the geometry and returns structured cabinet and drawer panels, dimensions, patterns, shelf data, row information, and `frontElevation` data. `buildDrawerCabinetDesignResult()` uses that model to produce the ordinary cut layout and carries `frontElevation` into the result. This is the correct dependency direction: the screen-only view consumes the model result; it does not parse the exported SVG or recalculate cabinet dimensions.

### UI mode handling

`designPreviewMode` is session-only. `designPreviewModeForTemplate()` limits `finished-front` to the Drawer Cabinet, and `designPreviewSelectorHtml()` renders the compact `Cut Layout` / `Finished Front View` selector only for that template. `designResultsHtml()` chooses between the exact exported SVG preview and `buildDrawerCabinetFrontElevationSvg()`.

The current selector is safe because:

- changing mode only refreshes the result area;
- the result is rebuilt from the same `designDraft`;
- download still calls `downloadCurrentDesignSvg()` and uses `result.svg`;
- the front renderer is explicitly labeled screen-only and not included in the downloaded cut file;
- no mode is persisted or included in backup data.

### Renderer strengths

`buildDrawerCabinetFrontElevationSvg()` uses front-elevation primitives rather than pretending to be a full 3D model. It shows cabinet boundaries, drawer openings, drawer fronts, running-gap marks, row labels, support shelves, outside dimensions, and compact labels for short drawer fronts. These are exactly the questions a front view should answer.

### Generalizable parts

Safe patterns to reuse:

- a template-scoped mode gate;
- a result field carrying structured view metadata;
- a screen-only renderer consuming that metadata;
- explicit `Screen-only assembly preview` text;
- separate preview and download paths;
- fixtures comparing Cut Layout and Finished View against the same authoritative result.

Do not generalize the cabinet's row and shelf calculations into a universal assembly engine. Those calculations are cabinet-specific.

## 4. Architecture options

### Option A — Per-template renderers

Each template owns its complete finished renderer.

Advantages:

- Simple control flow.
- Template-specific orientation and styling remain local.
- Lower risk of a generic renderer misrepresenting a lid, tray divider, or joint.
- Easy to keep legacy templates Cut Layout only.

Costs:

- Repeated SVG styling and label code.
- Repeated dimension and panel-outline logic.
- More visual drift between views over time.

### Option B — Shared finished-view primitives

All templates use a shared scene or renderer abstraction for panels, edges, lids, and perspective.

Advantages:

- Consistent visual language.
- Reusable labels, dimensions, and simple depth cues.
- Potential support for future visible joint-style overlays.

Costs and risks:

- High risk of designing a pseudo-3D system before the product needs one.
- Forces unlike products into one coordinate and orientation model.
- Encourages views derived from incomplete legacy data.
- Makes hidden, rebated, and decorative joint semantics harder to keep honest.

This is too broad for the next batch.

### Recommended Option C — Hybrid

Use shared, deliberately small primitives plus template-owned renderers:

- `finishedPanelRect()` / `finishedPanelPath()` for screen-only panel silhouettes;
- `finishedEdgeBand()` for visible wall thickness or a restrained side edge;
- `finishedDimensionLabel()` and `finishedOrientationLabel()`;
- `finishedLidOverlay()` for a lid shown separately or in a closed position;
- `finishedTrayWall()` for future tray renderers only after a tray model exists.

Each template renderer should receive a structured model/result and return screen-only markup. It should not accept the raw draft as its source of truth and should not call geometry builders with altered values. Shared primitives should render, not calculate, product geometry.

This hybrid keeps the current Drawer Cabinet structure, supports the P1 box renderers, and leaves room for future hidden/faux-dovetail overlays without committing to a generic 3D engine.

## 5. P1 implementation batch

### P1-A — Finger-jointed box

- View type: top-down assembled view for open top; optional restrained pseudo-isometric arrangement for the loose lid.
- Authoritative inputs: `buildBoxModel()` result, especially `dimensions`, `panels`, `patterns`, `lid`, `materialThickness`, and panel IDs.
- Visible components: bottom, front, back, left, right, and loose lid when `boxLid === 'loose'`.
- Orientation: front panel at the lower/front edge; back opposite; left/right retained from panel IDs; open cavity visible from above.
- Lid state: `boxLid === 'open'` shows no lid; `boxLid === 'loose'` supports two deterministic states only: beside the box or above it as a clearly separate loose panel. Do not imply hinges, retention, or a fitted lid.
- Labels/dimensions: outside width/depth/height, `Front`, `Back`, `Left`, `Right`, `Bottom`, and `Loose lid` where space allows. Keep joint and fit values as text warnings, not visually fabricated tolerances.
- Multiple states: open top and loose-lid; no closed retained state because the model has no retention feature.
- Fixture strategy: pure model panel-count/dimension checks; renderer checks for named panels and deterministic positions; browser selector checks that mode switching preserves draft and SVG.
- Limitations: no full 3D depth proof, no fit guarantee, and no inference of which face should be decorative beyond the named model panels.
- Estimated size: **Medium**.

### P1-B — Sliding-lid finger box

- View type: restrained isometric or pseudo-3D view with `Finished Closed` and `Finished Open` states.
- Authoritative inputs: `buildSlidingLidBoxModel()` and `buildSlidingLidDesignResult()` fields for `dimensions`, `panels`, `patterns`, rail lengths/heights, `frontHeight`, `stopLength`, `engagement`, `open end`, and optional guide/label flags.
- Visible components: five body panels, two inner rails, lid, Front opening, Back stop, and optionally the six-piece fit coupon as a separate test inset only if enabled.
- Orientation: Front on the open end; lid slides Front toward Back; the shortened Front and stop Back must be visually labeled.
- Lid state: closed shows the lid engaged in the rails; open shows the lid translated toward Front with the channel and Back stop visible. Both states must use fixed deterministic offsets derived from model dimensions, not arbitrary percentage scaling.
- Labels/dimensions: outside body dimensions, usable cavity, Front/Back, rail direction, lid engagement, and a concise “screen-only assembly preview” notice. Guide marks and score labels remain secondary annotations.
- Multiple states: required; closed and open answer different user questions.
- Fixture strategy: pure model checks for unchanged panel count and dimensions; renderer checks for stable lid translation and Front/Back labels; browser checks for state/mode switching and export identity.
- Limitations: the view must not imply a friction result, glue strength, kerf correction, or exact rail fit. It should not show the optional coupon as part of the box.
- Estimated size: **Medium-Large**.

### Why not make the tray the first batch?

Dice Tray and Divider Tray are valuable, but `designTraySvg()` currently emits a flattened collection of SVG shapes. It does not return named base/wall/divider components, assembled coordinates, or stable orientation metadata. A first tray Finished View would therefore either duplicate dimensions or parse SVG, both of which violate the authoritative-model boundary. Extracting a `buildTrayModel()` first is the correct P2 prerequisite.

## 6. P2 and P3 roadmap

### P2 — Extract and render tray models

1. Introduce a pure `buildTrayModel()` for Dice Tray and Divider Tray.
2. Keep `designTraySvg()` as a serializer of that model or preserve its output byte-for-byte where practical.
3. Add named base, front/back/left/right wall, slot, and divider components.
4. Add a top-down assembled view with an optional shallow depth cue.
5. Add divider state/placement metadata only after the model defines it.

Estimated size: **Large** for the model extraction plus both views; **Medium** for each renderer after extraction.

### P3 — Selective views and no-view templates

- QR / sign stand: consider a very small slot-assembly thumbnail only if a structured stand model is later introduced. Otherwise keep Cut Layout only.
- Hanging sign: keep Cut Layout only. A flat sign with holes does not need a Finished View.
- Joint Fit Coupon: keep Cut Layout only, with clear A/B 90-degree assembly instructions. It is a test artifact, not a storage object.

## 7. Future joint-style compatibility

Finished Views should represent the actual supported structural assembly at the level needed to explain orientation, while keeping decorative appearance separate.

Recommended separation:

1. **Structural joint geometry** remains in the authoritative model and controls cut paths, panel edges, tabs, slots, and fit calculations.
2. **Visible finished appearance** is a screen-only rendering layer derived from named panels, faces, and supported assembly metadata.
3. **Decorative overlays** are optional screen annotations or previews only when the corresponding decorative geometry is formally supported. They must never be mistaken for structural joints.

For future styles:

- Visible finger joints: show the external finger rhythm only if the model exposes it; do not redraw a second joint pattern in the Finished View.
- Hidden or rebated-style joints: show the finished external face and a restrained seam/edge indication; do not reveal hidden construction unless an explicit inspection mode is later designed.
- Tab-and-slot joints: show the assembled outer wall and only expose slot/tab direction when it answers an assembly question.
- Decorative/faux dovetail faces: use an optional decorative overlay clearly labeled as decorative; it must not imply actual dovetail strength.
- Structural dovetails: once formally supported, derive the visible joint from the structural model; do not approximate dovetails from generic finger-joint primitives.

Future joint styles should not require Finished View code to know how geometry is cut. The renderer should consume semantic panel/face/joint metadata. If a style lacks sufficient semantic metadata, retain Cut Layout only until that metadata exists.

## 8. Protected boundaries

Every Finished View implementation must remain:

- screen-only and session/UI state only;
- derived from the authoritative model/result;
- excluded from `serializeDesignSvg()` and downloaded LightBurn SVG;
- excluded from LightBurn layers, material/cut calculations, and geometry validation;
- unable to mutate `designDraft` dimensions, kerf, clearance, or joint values;
- unable to write Designs, Production Settings, Material Tests, or Projects;
- deterministic for the same validated model and selected mode.

Mode changes should only change the result-area renderer. They must not change generated panel count, panel paths, dimensions, SVG bytes, filenames, saved records, or backup content. The Drawer Cabinet pattern of a session-only selector and a separate screen renderer is the correct boundary.

## 9. Fixture strategy

### Pure model fixtures

For each new template, verify:

- the authoritative model remains valid for normal and boundary dimensions;
- panel/component count is unchanged across Finished View modes;
- dimensions and named orientation metadata are stable;
- lid/drawer/open-state metadata is deterministic;
- invalid and extreme dimensions return errors without throwing;
- the source draft is not mutated.

### Renderer fixtures

For each renderer, verify:

- repeated rendering is byte-stable for the same model and state;
- expected labels and component IDs are present;
- displayed dimensions come from model dimensions;
- open/closed or open-top/loose-lid positions are deterministic;
- no renderer markup enters `result.svg` or download serialization;
- no internal geometry is altered by rendering.

### Browser/UI fixtures

Verify through the actual Designs form and selector:

- switching Cut Layout / Finished View preserves `designDraft` JSON;
- switching modes preserves `buildDesignResult(draft).svg` byte-for-byte;
- download still emits the Cut Layout SVG and original filename pattern;
- mode selection resets safely when switching templates or invalidating dimensions;
- ordinary UI remains compact;
- existing Drawer Cabinet selectors and all existing Design fixture groups remain unchanged.

The first P1 batch should add a small, explicit mode-state fixture per template rather than a global registry fixture. A shared helper test can prove screen-only markup is excluded from serialization, but each renderer still needs its own orientation and component assertions.

## 10. UI recommendation

Keep the selector per template. The compact baseline is:

`Cut Layout | Finished View`

Use additional states only where the product has a real assembly distinction:

`Cut Layout | Finished Closed | Finished Open`

for Sliding Lid Box, and:

`Cut Layout | Finished Front View`

for Drawer Cabinet.

Do not add a global Finished View toolbar, persisted global mode, or template-independent state machine. Per-template selectors make the valid states explicit and prevent a hanging sign or coupon from appearing to support an assembled mode it cannot represent.

## 11. Exact recommended next Codex implementation task

Implement **P1-A Finger-jointed box Finished View** only:

1. Add a session-only selector for `finger-box` with `Cut Layout` and `Finished View`.
2. Derive the view from `buildBoxModel()` / `buildBoxDesignResult()` panel and dimension data.
3. Render a restrained top-down assembled box and a separate loose lid only when `boxLid === 'loose'`.
4. Keep `serializeDesignSvg()`, `downloadCurrentDesignSvg()`, and all geometry/model functions unchanged except for the minimum result metadata needed to consume existing model data.
5. Add pure, renderer, and browser fixtures for mode isolation, dimensions, panel count, deterministic orientation, invalid dimensions, and export exclusion.
6. Update only the focused Designs documentation after validation.

Then perform an independent read-only audit of that implementation before starting Sliding Lid Box.

## 12. Independent audit requirement

Yes. An independent read-only audit is recommended after each P1 template or tightly coupled P1 batch. The audit should verify that the new renderer is model-derived, mode changes do not affect SVG or draft state, labels/orientation are honest, and no screen-only markup entered the production download path.

## Final readiness

**READY FOR IMPLEMENTATION**

The current Drawer Cabinet implementation provides a safe narrow pattern, Finger Box and Sliding Lid Box have sufficient authoritative model data for the next batch, and the remaining templates can be staged behind explicit model-extraction work. No application change is proposed by this review.
