# Sliding-Lid Box Architecture and Geometry Review

Date: 2026-07-15

Repository: `C:\Genmitsu L8 Tracker`

Reviewed baseline: `0a304d8` — `Add parametric finger-box generator`

Scope: read-only source/geometry review; this report is the only file added

## 1. Executive recommendation

Implement the sliding-lid box as a new `sliding-lid-box` Designs template. Do not add a lid mode to the existing `finger-box` template, and do not rewrite the physically proven five-panel body generator into a general joinery framework.

The recommended construction is an eight-piece, all-through-cut design:

1. Bottom.
2. Shortened front wall at the lid insertion end.
3. Full back wall.
4. Left wall.
5. Right wall.
6. Sliding lid.
7. Left laminated inner channel rail.
8. Right laminated inner channel rail.

Each side receives one identical, glue-on inner rail. A through-cut channel in each rail captures the lid below the wall tops. The rail channel remains open at the front and has an integral stop web at the back. The lid has side shoulders that stop against those webs and a narrower center tongue that continues to the back wall, closing the otherwise unavoidable stop gap.

This is the safest bounded approach because it:

- uses only reliable through cuts in sheet material;
- avoids controlled-depth engraving in variable plywood glue layers;
- preserves the tested standard finger-box geometry and output;
- keeps finger-joint fit separate from sliding fit;
- is expressible with the existing orthogonal SVG path engine;
- needs no hardware, persistence, schema change, or dependency;
- confines new complexity to a dedicated model and a few small pure helpers.

The existing standard box's successful LightBurn import, cut, orientation, and assembly are meaningful evidence that its mating-edge phases are correct. The slightly loose physical joints are a material-fit calibration issue already addressed by the independent joint-clearance control, not a reason to redesign the body.

## 2. Baseline and repository verification

The review began with the requested repository checks.

- `HEAD`: `0a304d8`
- `main`: `0a304d8`
- `origin/main`: `0a304d8`
- `origin/HEAD`: synchronized with `origin/main`
- Latest commit: `0a304d8 Add parametric finger-box generator`
- Tracked working tree before this report: clean
- Staging area: empty
- `git diff --check`: passed with no tracked diff
- `git diff --stat`: empty before this report
- Existing unrelated untracked files: observed and left untouched

The following reports were reviewed together with the live code:

- `DESIGNS_BOX_GENERATOR_ARCHITECTURE_REVIEW_2026-07-14.md`
- `BOX_GENERATOR_PHASE2_IMPLEMENTATION_2026-07-14.md`
- `DESIGNS_BOX_GENERATOR_INDEPENDENT_AUDIT_2026-07-15.md`
- `DESIGNS_BOX_GENERATOR_SECOND_PASS_REVIEW_2026-07-15.md`

Live direct-file browser verification produced:

| Check | Result |
|---|---:|
| `index.html?selftest=design` | 82 passed / 0 failed |
| `index.html?selftest=all` | 614 passed / 0 failed |
| HTML parsing (`python -m html.parser index.html`) | passed |
| Normal `file://` startup | rendered eight tabs; no self-tests ran |

The full-suite groups were: baseline 20, normalization 12, grid 23, grid browser 67, Material Browser 57, Library Browser 56, Project Browser 61, Project Wizard 216, metadata 12, storage 8, and Designs 82. Their sum is 614.

Representative open-top and loose-lid finger boxes were generated in the browser's memory and inspected through the existing result and SVG-validation paths. The default model retains the expected five cut groups for open top and six for loose lid, one red `cut` group, millimeter document dimensions, deterministic panel IDs, and no `<text>` in cut geometry. Repeated generation is byte-stable under the current fixtures. No temporary artifact was added to the repository.

This review does not claim a LightBurn import or physical fit test for the proposed sliding-lid geometry. Only the existing standard finger box has that evidence.

## 3. Current Designs architecture assessment

### Existing flow

`normalizeDesignDraft()` validates template-specific form values and returns normalized values plus errors and warnings. `buildDesignResult()` dispatches to a model builder, lays out valid pieces, serializes the SVG, validates the serialized result, and supplies the preview/results model. `refreshDesignPreview()` rebuilds from the module-level `designDraft`; preview and download use the same generated SVG.

The existing box path is well divided for this phase:

- `designBoxDimensions()` owns the proven standard inside/outside body conversion.
- `buildFingerPattern()` chooses an odd segment count and exact segment width.
- `designEdgePoints()` applies phase, thickness, and half-clearance boundary offsets.
- `buildFingerPanel()` composes four orthogonal edges into one closed panel path.
- `buildBoxModel()` owns the standard box's mating-edge definitions and optional loose lid.
- `layoutDesignPanels()` places known panel IDs in fixed rows.
- `serializeDesignSvg()` emits one predictable red cut group with stable IDs and titles.
- `designSvgValidation()` parses the actual SVG and validates document bounds.
- `designResultsHtml()` presents dimensions, pieces, warnings, and export readiness.
- `runDesignGeometryFixtures()` currently supplies 82 passing assertions.

### Reuse and boundaries

Reuse unchanged:

- `designRound()` and `designNumberText()`.
- `buildFingerPattern()` for full-length and shortened vertical patterns.
- `designPatternEdge()` and `designEdgePoints()`.
- `designCompactPoints()` and `designPointsPath()`.
- `buildFingerPanel()` for orthogonal body panels.
- `serializeDesignSvg()` if every new piece remains one closed path.
- the current SVG preview/download identity flow.

Keep standard-box-specific:

- `designBoxDimensions()` semantics as used by `finger-box`.
- `buildBoxModel()` and its current five/six definitions.
- current `finger-box` defaults, UI labels, warnings, panel order, layout, and serialized bytes.

Generalize narrowly:

- Add a row-driven layout helper, for example `layoutDesignPanelRows(panels, rows, margin, gap)`, while retaining `layoutDesignPanels()` as a legacy wrapper with its current row list. This preserves legacy order and bytes.
- Add a pure orthogonal-outline helper for the rail and shouldered lid if `buildFingerPanel()` cannot represent their internal concavities clearly.
- Add actual path-AABB separation checks instead of relying only on nominal width/height envelopes for the new layout fixtures.

Add separately:

- `slidingLidBoxDimensions()` for the new usable-cavity semantics.
- `buildSlidingLidRail()` for the integral-stop U-channel strip.
- `buildSlidingLidPanel()` for the shoulder/tongue lid outline and optional rectangular pull.
- `buildSlidingLidBoxModel()` as the sole owner of this construction.

Do not expand `buildBoxModel()` with repeated `if (lidMode === ...)` branches. The sliding design changes front height, front-to-side joint span, interior-volume meaning, piece count, layout, validation, and results. A separate model is easier to review and much less likely to disturb the physically validated path.

## 4. Recommended sliding-lid construction method

### Compared methods

**Partial-depth engraved grooves:** exclude from Phase 3. A diode laser can engrave a recess, but repeatable groove depth in plywood is not a dependable structural dimension. Veneer density, glue layers, moisture, focus, flatness, and batch variation can change depth. Multiple passes also char the bearing surface. LightBurn can command the operation, but it cannot make material depth uniform.

**Slots through the side walls plus inserted rail tabs:** potentially self-registering, but not recommended for this first version. Long registration slots would be close to the bottom and corner finger fields, weaken the wall, add exposed marks, and create collision rules substantially more complex than the rail itself. Small isolated slots would still require alignment and introduce loose parts or protrusions.

**Wall tabs forming a channel:** possible but forces new interrupted side-wall outlines and extra mating geometry. It couples lid fit to corner construction and makes the standard body less reusable.

**Separate laminated rails:** recommended. Each rail is a through-cut U-channel strip glued to the inner side wall, top-flush and back-flush. The wall supplies the outside channel face; the rail supplies the lower web, inner face, and closed-end stop. Identical pieces are easy to cut, inspect, and replace.

### Registration decision

Use top-flush and back-flush physical edges as the Phase 3 registration references. Do not cut wall registration slots yet. A simple assembly spacer equal to the intended channel height can be used during glue-up, but it need not be exported as a production part in this phase.

This is less mechanically indexed than tabs, but it is safer for a first sliding prototype because it preserves wall strength and eliminates registration-slot/finger collisions. If physical tests show alignment is difficult, small isolated indexing features can be reviewed in a later bounded phase.

### Lid travel and orientation

- The **Front** is the open/removable end.
- The lid is inserted from **Front toward Back**.
- The **Back** remains a full-height, finger-jointed closed end.
- The rails are open at Front and stopped at Back.
- The front wall is shortened below the channel so it does not block insertion.

The form should state: `Open end: Front — lid slides toward Back`. Exported piece titles should use `Front (open end)`, `Back (closed stop end)`, `Left rail`, and `Right rail`. That is enough orientation information without putting non-cut text into the SVG.

## 5. Exact dimensional model

All dimensions below are finished geometry dimensions before any machine kerf compensation.

### Symbols

| Symbol | Meaning |
|---|---|
| `t` | measured material thickness |
| `Cj` | finger-joint fit adjustment used by the existing edge engine |
| `Cs` | total lid side-clearance reduction across both sides |
| `Cv` | extra vertical clearance above a thickness-`t` lid in its channel |
| `Ci` | open-end clearance between the front-wall top and lid underside |
| `Uw`, `Ud`, `Uh` | requested usable cavity width, depth, and height |
| `Wi`, `Di`, `Hi` | full wall-to-wall inside body width, depth, and height |
| `Wo`, `Do`, `Ho` | outside body width, depth, and height |
| `U` | upper rail web |
| `L` | lower rail web |
| `S` | integral closed-end stop length |
| `Ch` | clear channel height |
| `Rh`, `Rl`, `Rc` | rail height, rail length, and open channel length |
| `G` | gap between inner rail faces |
| `Lw` | lid maximum width, including side shoulders |
| `Tw` | lid back-tongue width |
| `E` | lid engagement into each rail channel |
| `Hf` | shortened front-wall outside height |

Use conservative construction constants:

```text
U = max(t, 4 mm)
L = max(t, 4 mm)
S = max(t, 4 mm)
```

These are geometry constants, not hidden kerf adjustments. They keep the rail webs and stop inspectable in common 3 mm stock.

### Usable-cavity mode

In this template, “usable cavity” means unobstructed space below the lid and between the two inner rail faces. The rails consume `t` on each side; the lid/rail stack consumes vertical height above the requested cavity.

```text
Wi = Uw + 2t
Di = Ud
Hi = Uh + t + Cv + U

Wo = Wi + 2t = Uw + 4t
Do = Di + 2t = Ud + 2t
Ho = Hi + t = Uh + U + 2t + Cv
```

Depth remains the usable distance from the inside face of the shortened front wall to the inside face of the back wall. The rails extend along that same inside depth.

### Outside-body mode

```text
Wi = Wo - 2t
Di = Do - 2t
Hi = Ho - t

Uw = Wi - 2t = Wo - 4t
Ud = Di = Do - 2t
Uh = Hi - U - t - Cv = Ho - U - 2t - Cv
```

The UI and results must show all three concepts rather than calling each one “inside”:

- requested usable cavity below the lid;
- full wall-to-wall body interior before rail intrusion;
- overall exterior body size.

### Rail and lid formulas

```text
Ch = t + Cv
Rh = U + Ch + L
Rl = Di
Rc = Di - S

G = Wi - 2t
Lw = Wi - Cs
E = (Lw - G) / 2 = t - Cs/2
Tw = G - Cs
```

Each rail is `Rl` long and `Rh` high. Its channel begins at the Front, has clear height `Ch`, and remains open for `Rc`; the final `S` at the Back is solid stop material.

Place each rail with its bottom at `Uh - L` above the inside floor. Its lower web therefore ends at the lid underside plane `Uh`; its channel spans `Uh` through `Uh + t + Cv`; and its upper web ends at `Hi`, flush with the side-wall top.

The lid outline is a shouldered orthogonal part:

- maximum shoulder width: `Lw`;
- center tongue width: `Tw`;
- material thickness: `t` physically, not encoded as a 2D value;
- overall front-to-back length: `Di + t`;
- front edge starts one wall thickness beyond the inside front plane, so it reaches the outside front face when closed;
- side shoulders end at `y = Rc` from the inside front plane;
- the center tongue continues to `y = Di` and reaches the back wall.

At closure, both shoulders contact the integral rail stops. The centered tongue passes between the rail stop webs and fills the last `S` to the back wall. This explicitly avoids a permanent stop-length opening at the rear of the lid.

### Front wall

The lower surface of the lid is at usable cavity height `Uh` above the inside floor. The front wall stops `Ci` below that surface:

```text
Hf = t + Uh - Ci
```

`Hf` is an outside panel height including the bottom-joint thickness. The front wall keeps its full bottom finger relationship. Its two vertical corner joints use a new pattern derived from `Hf`, so the side panels have matching lower finger fields and plain upper-front edges.

This asymmetric side-front edge is the principal new body geometry. It must be built explicitly in `buildSlidingLidBoxModel()`; the current full-height front/back definition must not be silently reused.

## 6. Exact clearance semantics

The three fit concepts must remain independent.

### Finger-joint clearance (`Cj`)

This is the existing finger-pattern boundary adjustment. It changes male/female mating widths by half the entered value at each internal boundary. It affects body corner and bottom joints only. It does not change the lid or rail channel.

The prior physical test found the chosen setting slightly loose. That confirms the control works but does not establish a production default for other plywood batches.

### Lid side clearance (`Cs`)

This is the **total** width removed from the lid across both sides. Each side receives `Cs / 2` free lateral clearance against the side-wall reference. It is entered once and is neither doubled nor halved elsewhere.

Because the lid shoulders extend into the rail channels, engagement per side is `E = t - Cs/2`. Excessive `Cs` can make the lid escape vertically and must be blocked.

### Lid vertical clearance (`Cv`)

This is added once to the material thickness to form channel height: `Ch = t + Cv`. It is the total free vertical space above a lid resting on the lower rail web. It is not applied above and below separately.

### Open-end sill clearance (`Ci`)

This is the one-sided vertical gap between the shortened front-wall top and the lid underside. It allows insertion without scraping the sill. It does not affect the channel or lid width.

### Kerf

Kerf remains absent from generated geometry. It should be handled in LightBurn or by later, separately reviewed machine compensation. No Phase 3 formula should quietly combine kerf with `Cj`, `Cs`, `Cv`, or `Ci`.

Initial UI defaults may be modest experimental values such as `0.20 mm` for each lid clearance, but they must be labeled starting points and validated on scrap. No production fit should be claimed.

## 7. Rail, stop, open-end, and finger-pull geometry

### Rail path

Each rail is one closed orthogonal U-channel outline:

- an uninterrupted lower web of height `L`;
- an inner upper web of height `U`;
- an open channel of height `Ch` from Front through `Rc`;
- a solid stop web of length `S` at Back joining upper and lower portions.

The rail's outer long edge is glued flush to the side wall. Both rail pieces use identical cut geometry; “Left” and “Right” differ only by assembly placement and metadata.

Rails do not need finger joints. They are glue-only laminations. Their top and back edges are registration surfaces. They have no side-wall slots in Phase 3.

### Captured lid behavior

The lid rides below the wall tops, inside the laminated rail channels. The lower rail webs support it; upper rail webs prevent upward escape while the shoulders are engaged. The rear stop webs block the shoulders at the closed position. The tongue fits between the rail inner faces and closes the center to the back wall.

At the Front, the rail channels are unobstructed and the shortened wall remains below the lid underside. The lid can therefore be removed by sliding it toward Front.

### Pull

Support two Phase 3 choices:

- `None` (default).
- `Centered rectangular edge notch`.

Use a notch entering from the lid's front edge, not an internal hole. Make width user-selectable and derive a conservative depth such as `max(t, 6 mm)`. The notch remains orthogonal, works with the current `M/H/V/Z` parser, and does not require introducing arc serialization or validation.

Defer a semicircular thumb notch. Although visually preferable in some products, it would require extending the path parser, actual-bounds logic, and self-intersection fixtures to arcs. An engraved mark is not a functional pull.

## 8. Proposed panel list and deterministic layout

Use these stable IDs and names in this order:

| Order | ID | Name |
|---:|---|---|
| 1 | `bottom` | Bottom |
| 2 | `sliding-lid` | Sliding lid |
| 3 | `front-open` | Front (open end) |
| 4 | `back` | Back (closed stop end) |
| 5 | `left` | Left |
| 6 | `right` | Right |
| 7 | `rail-left` | Left rail |
| 8 | `rail-right` | Right rail |

Use the existing `10 mm` margin and `15 mm` gap with four fixed rows:

```text
Row 1: bottom, sliding-lid
Row 2: front-open, back
Row 3: left, right
Row 4: rail-left, rail-right
```

Both physical rails must appear in the SVG. A quantity note with one rail outline is not fully cut-ready and invites an incomplete job.

No separate stop piece is needed; the stops are integral to the rails. The optional pull changes only the lid outline. A future coupon must be placed in a fifth row so the first eight pieces retain stable order and positions when the coupon is absent.

Continue to emit one red `g#cut`, one stable group per piece, and a `<title>` for metadata. Textual results and assembly guidance are sufficient; do not add visible orientation labels to exported cut geometry in Phase 3.

## 9. Blocking errors and nonblocking warnings

### Blocking errors

Reject generation when any of the following is true:

- any required value is blank, nonnumeric, non-finite, or outside its allowed sign/range;
- template, dimension mode, pull mode, open end, or lid mode is unknown;
- `t <= 0`, `Cj < 0`, `Cs < 0`, `Cv < 0`, or `Ci < 0`;
- outside mode does not satisfy `Wo > 4t`, `Do > 2t`, and `Ho > U + 2t + Cv`;
- any of `Uw`, `Ud`, `Uh`, `Wi`, `Di`, or `Hi` is nonpositive;
- `G`, `Lw`, or `Tw` is nonpositive;
- lid engagement is below `Omin = max(1 mm, t/3)`;
- equivalently, `Cs > 2(t - Omin)`;
- `Rc = Di - S` is not greater than `max(3t, 12 mm)`;
- `Rh > Hi` or the rail cannot fit between floor and wall top;
- `Uh <= Ci`, making the front insertion gap collapse;
- `Hf` is nonpositive or cannot support a valid odd pattern of at least three vertical segments at the current minimum segment width;
- the lower side-front joint span leaves less than `max(t, 4 mm)` of structural floor-side web;
- the rail stop or lid tongue leaves its panel bounds;
- the pull notch width/depth is nonpositive, exceeds the front edge, intersects the shoulder transition, or leaves less than `max(2t, 6 mm)` material on either side;
- any cut path is open, non-orthogonal, self-intersecting, zero-length, duplicated, non-finite, or outside the viewBox;
- actual translated path AABBs overlap or are separated by less than the requested layout gap.

Registration-slot collision is structurally impossible in the recommended Phase 3 model because there are no registration slots. Add a fixture explicitly confirming zero registration features so a later refactor cannot introduce them accidentally.

### Nonblocking warnings

Warn, but allow export, when:

- a lid clearance is zero, because a nominal zero-fit channel is unlikely to slide after cutting;
- `Cj > t/4`, matching the existing loose-joint warning;
- `Cs`, `Cv`, or `Ci` is unusually large relative to `t`;
- engagement is close to `Omin`;
- the pull is enabled near its minimum remaining web;
- preferred finger width is below `2t`;
- the box or full four-row layout is large relative to likely L8 work area;
- usable cavity differs materially from full wall-to-wall dimensions due to rails;
- plywood fit remains unverified and a scrap sliding-fit test is recommended;
- glue is required for rails and likely for the current material's body joints.

Practical work-area excess should remain a warning unless the application gains an authoritative configured machine envelope. The current code has only a general large-layout warning, not a guaranteed bed profile.

## 10. Proposed UI fields and result metrics

### Form fields

Present only fields that change Phase 3 output:

1. **Template** — add `Sliding-lid finger box`.
2. **Dimensions mean** — `Usable cavity below lid` or `Outside body`.
3. **Usable/Outside width**.
4. **Usable/Outside depth**.
5. **Usable/Outside height**.
6. **Measured material thickness**.
7. **Finger-joint clearance** — “Body joints only.”
8. **Preferred finger width**.
9. **Lid side clearance (total)**.
10. **Lid vertical clearance**.
11. **Front insertion clearance**.
12. **Finger pull** — `None` or `Centered rectangular notch`.
13. **Pull width** — shown only when enabled.

Show, but do not make configurable:

- `Open end: Front (slides toward Back)`.
- `Rail construction: laminated through-cut inner channels`.

Do not add a direction selector or rail-style selector until another orientation or construction has been designed and tested. Disabled choices imply capabilities that do not exist.

### Results and assembly summary

Show:

- usable cavity `Uw × Ud × Uh`;
- wall-to-wall inside `Wi × Di × Hi`;
- outside body `Wo × Do × Ho`;
- lid shoulder width, tongue width, and overall length;
- side clearance per side and total;
- channel height and vertical clearance;
- engagement per side;
- rail length, height, open-channel length, and stop length;
- Front insertion direction and Back stop;
- eight required pieces;
- SVG layout width and height;
- body joint clearance separately from all lid clearances;
- explicit “kerf not included” text;
- blocking errors and experimental-fit warnings.

The assembly summary should say to build the body, glue each identical rail top-flush/back-flush inside its side, verify channels face inward, allow glue to cure with a thickness/clearance spacer, and insert the lid from Front toward Back. This is clearer to a non-engineer than extra marks mixed into cut geometry.

## 11. Helper/function change plan

The bounded implementation should follow this function ownership:

### `normalizeDesignDraft()`

Add `sliding-lid-box` to the recognized templates and parse only its fields. Keep `finger-box` parsing unchanged. Normalize all numeric values once and reject unknown fixed modes defensively even if the UI does not expose them.

### `buildDesignResult()`

Add one dispatch branch to `buildSlidingLidBoxModel()`. Retain existing branches and serializer order unchanged.

### `designBoxDimensions()`

Leave unchanged for the existing template. Do not overload “inside” to mean usable cavity. Add `slidingLidBoxDimensions()` with the formulas in section 5.

### Finger helpers

Reuse `buildFingerPattern()`, `designEdgePoints()`, and `buildFingerPanel()` unchanged. Build a second vertical pattern for `Hf`; do not crop points from the full-height side pattern.

For each side panel, compose a front edge with a lower complementary finger field and a plain upper run. A small `designCompositeEdgePoints()` helper is preferable to special cases inside `designEdgePoints()`.

### New model helpers

Recommended pure helpers:

```javascript
slidingLidBoxDimensions(parameters)
buildSlidingLidRail(dimensions, parameters)
buildSlidingLidPanel(dimensions, parameters)
buildSlidingLidSidePanel(definition)
buildSlidingLidBoxModel(parameters)
```

Each helper should return data and errors without reading DOM or global state. Inputs must not be mutated.

### Layout

Introduce:

```javascript
layoutDesignPanelRows(panels, rows, margin = 10, gap = 15)
```

Keep `layoutDesignPanels()` as a wrapper passing the exact current rows. The new model passes its four rows. This is the only generalization needed; do not create a configuration-driven nesting framework.

### Serialization and validation

Keep `serializeDesignSvg()` unchanged if rail and lid remain one path each. Enhance validation or fixture-only inspection to calculate actual translated path AABBs and test path segment intersections. The prior audit correctly noted that nominal panel envelopes alone do not prove actual-path separation.

The optional rectangular pull keeps the existing orthogonal parser sufficient. Do not add arcs as incidental scope.

### UI and refresh

Extend the current template-conditional form and results text. Preserve the module-level `designDraft`, partial numeric-entry behavior, active-field/caret behavior, exact preview/download SVG identity, and reset semantics. Do not call `persist()` for any Designs field.

## 12. Fixture plan with golden cases

Keep all 82 current Designs assertions unchanged. Add a clearly named sliding-lid subsection to `runDesignGeometryFixtures()`; do not calculate golden expectations by calling the same dimension helper under test.

### Golden A — usable-cavity mode

Inputs:

```text
mode = usable cavity
Uw = 100 mm
Ud = 80 mm
Uh = 40 mm
t = 3 mm
Cj = 0.10 mm
Cs = 0.20 mm total
Cv = 0.20 mm
Ci = 0.20 mm
U = L = S = 4 mm
```

Hand-derived expected values:

```text
Wi = 106 mm
Di = 80 mm
Hi = 47.2 mm
Wo = 112 mm
Do = 86 mm
Ho = 50.2 mm

G = 100 mm
Lw = 105.8 mm
E = 2.9 mm per side
Tw = 99.8 mm

Ch = 3.2 mm
Rh = 11.2 mm
Rl = 80 mm
Rc = 76 mm

lid overall length = 83 mm
shoulder stop plane = y 76 mm from inside Front
tongue end plane = y 80 mm at inside Back
Hf = 42.8 mm
piece count = 8
```

Arithmetic check: `Wi = 100 + 6`; `Hi = 40 + 4 + 3 + .2`; `Lw = 106 - .2`; `E = (105.8 - 100)/2`; and `Hf = 3 + 40 - .2`.

### Golden B — outside-body mode

Inputs:

```text
mode = outside body
Wo = 132 mm
Do = 102 mm
Ho = 63 mm
t = 3 mm
Cj = 0.10 mm
Cs = 0.20 mm total
Cv = 0.20 mm
Ci = 0.20 mm
U = L = S = 4 mm
```

Hand-derived expected values:

```text
Wi = 126 mm
Di = 96 mm
Hi = 60 mm
Uw = 120 mm
Ud = 96 mm
Uh = 52.8 mm

G = 120 mm
Lw = 125.8 mm
E = 2.9 mm per side
Tw = 119.8 mm

Ch = 3.2 mm
Rh = 11.2 mm
Rl = 96 mm
Rc = 92 mm

lid overall length = 99 mm
shoulder stop plane = y 92 mm from inside Front
tongue end plane = y 96 mm at inside Back
Hf = 55.6 mm
piece count = 8
```

Arithmetic check: `Wi = 132 - 6`; `Uh = 60 - 4 - 3 - .2`; `G = 126 - 6`; and `Hf = 3 + 52.8 - .2`.

### Required fixture categories

**Dimensions and semantics**

- Golden A exact values.
- Golden B exact values.
- usable cavity excludes both rail thicknesses;
- full interior and usable interior are reported separately;
- outside conversion round-trips within `1e-6`;
- lid length includes exactly one front-wall thickness;
- shoulder stop and tongue-end planes match the formulas.

**Rail geometry**

- exactly two rail pieces;
- rails have distinct IDs but identical local paths and dimensions;
- each rail has one open Front channel and integral Back stop;
- rail stop length is exactly `S`, not `2S`;
- no registration slots are generated;
- rail, stop, and webs remain in bounds;
- rail open length remains above minimum.

**Clearance**

- `Cs = 0` gives `Lw = Wi` and `E = t`, with a warning rather than a formula error;
- `Cv = 0` gives `Ch = t`, with a warning;
- a typical `0.20 mm` set yields the goldens above;
- changing `Cj` changes body joint coordinates but not lid/rail dimensions;
- changing `Cs` changes width by exactly the entered total and engagement by half;
- changing `Cv` changes channel and body stack once;
- excessive `Cs` below minimum engagement blocks export;
- generated geometry contains no kerf field or kerf adjustment.

**Assembly relationships**

- shoulders overlap both channel capture regions in width;
- lid tongue fits between rail inner faces with `Cs/2` clearance each side;
- upper webs overlap shoulders enough to prevent upward escape when closed;
- shoulder end equals rail stop start;
- tongue reaches the inside Back plane;
- the front-wall inside top is exactly `Ci` below the lid underside, leaving the intended insertion gap;
- insertion ray toward Front is unobstructed;
- existing bottom/back/side mating phases remain exactly those of the proven body where applicable;
- new shortened-front vertical pairs have matching boundaries and opposite phases.

**Paths and layout**

- all eight piece paths close with `Z`;
- paths remain orthogonal and finite;
- no self-intersections or zero-length segments;
- no translated segment is duplicated;
- actual path AABBs are separated by at least `15 mm`;
- every actual path is inside the viewBox;
- panel order and four rows are exact;
- repeated generation is byte-identical;
- preview SVG equals download SVG;
- SVG has one cut group and eight stable panel groups;
- no `<text>` enters exported cut geometry.

**Invalid input**

- blank, nonnumeric, `NaN`, and infinite values block;
- rails larger than the available interior block;
- nonpositive lid shoulder/tongue dimensions block;
- insufficient engagement blocks;
- insufficient lower/upper/stop web blocks;
- shortened front cannot form three segments blocks;
- oversized pull blocks;
- unknown pull, open-end, dimension, and lid modes block;
- outside width at `4t`, depth at `2t`, and height at `U + 2t + Cv` block.

**Regression**

- retain the existing 82 assertions with identical names and results;
- snapshot exact current default SVG strings or fixed hashes for all four legacy non-box templates;
- snapshot current default open-top and loose-lid `finger-box` SVG strings or fixed hashes;
- verify `buildBoxModel()` output is unchanged before and after adding the new template;
- verify legacy layout order remains unchanged;
- verify input drafts, panels, patterns, and rails are not mutated.

Add boundary fixtures on either side of finger-count transitions for the shortened front wall. The independent audit identified this as a useful gap in the current standard-box coverage.

## 13. Deferred features

Defer all of the following until the basic sliding box is cut and physically validated:

- semicircular or curved thumb notches;
- alternate open ends or user-selectable slide direction;
- removable top rails;
- tab-indexed or slotted rails;
- partial-depth engraved grooves;
- hidden finger joints;
- tab-and-slot corners;
- rabbet/lap-style construction;
- mitered appearance;
- keyed or decorative joints;
- a generic joinery/lid framework;
- sheet nesting or optimization;
- DXF, LightBurn-native, or G-code export;
- kerf compensation or inheritance;
- saved Designs presets;
- persistent fit learning;
- Library, Project, Inventory, or Pricing integration;
- automatic material/cut-setting selection;
- network access or external dependencies.

The optional combined fit coupon should also be deferred to a small Phase 3.1. A useful coupon needs both a complementary finger pair and a representative captured rail/lid section, likely at multiple clearances. Adding it now would complicate layout, labels, and fixture count before the primary construction is proven. The first implementation should instead recommend a reduced-depth prototype or manually isolated rail/lid sample cut from the same sheet.

## 14. Risks and required manual tests

### Primary risks

1. **Rail alignment:** top/back flush references are simple but glue can creep. Use a thickness-plus-clearance spacer while clamping.
2. **Plywood thickness variation:** nominal 3 mm stock can vary enough to bind or rattle. Measure the actual sheet.
3. **Char and soot:** channel bearing surfaces may need cleaning or light sanding, changing fit.
4. **Panel bow:** a long lid or rail can curve and bind even when 2D dimensions are correct.
5. **Stop strength:** the integral `S` web must survive repeated closure without splitting along a veneer.
6. **Front-wall stiffness:** shortening the front removes upper corner support. The minimum height and three-segment rules are important.
7. **Glue squeeze-out:** adhesive in the channel will bind the lid; assembly instructions must call this out.
8. **Shoulder/tongue transition:** sharp internal corners can concentrate stress. This is acceptable for a prototype but must be observed physically.
9. **Fit defaults:** the existing body test found slightly loose joints, and no lid clearance has physical evidence. Defaults remain experimental.

### Required manual validation after implementation

1. Open normal `index.html` directly by `file://` and confirm no self-tests run.
2. Confirm all legacy template previews and downloads are unchanged.
3. Generate both golden boxes and inspect dimensions/results.
4. Import the new SVG into LightBurn; verify millimeter scale, one cut layer, eight pieces, no duplicate cuts, and correct orientation.
5. Cut a short rail/lid sample from the actual sheet at the selected `Cs` and `Cv`.
6. Cut a reduced-depth full body before a production-size box.
7. Dry-fit the body and confirm the shortened-front mating phases.
8. Glue rails with a spacer; verify they are parallel, top-flush, and back-flush.
9. Confirm the lid enters only from Front, cannot lift while captured, reaches both stops, and closes to Back with the tongue.
10. Cycle the lid repeatedly and inspect stop wear, rail splitting, binding, rattle, and front-wall flex.
11. Test the optional rectangular pull for finger access and remaining edge strength.
12. Record actual material thickness and successful clearances manually; do not persist them automatically in this phase.

Passing browser fixtures will establish deterministic 2D relationships, not manufacturability. A physical sliding prototype is mandatory before calling the template production-ready.

## 15. Recommended bounded implementation sequence

1. Capture fixed legacy SVG regression signatures before changing dispatch or layout.
2. Add template defaults and `normalizeDesignDraft()` parsing for `sliding-lid-box` only.
3. Implement and fixture `slidingLidBoxDimensions()` using the two hand-derived goldens.
4. Implement pure rail and lid outlines; test bounds, stop, engagement, and insertion relationships.
5. Implement shortened-front and composite side-front edges with independent complementary patterns.
6. Assemble `buildSlidingLidBoxModel()` without changing `buildBoxModel()`.
7. Add row-driven layout underneath the exact legacy wrapper.
8. Add actual-path AABB and intersection fixtures.
9. Add the minimal UI fields, results, warnings, and assembly summary.
10. Verify exact preview/download identity and all legacy hashes.
11. Run Designs and full self-tests, HTML parsing, syntax validation, normal offline startup, storage/backup probes, and `git diff --check`.
12. Perform LightBurn inspection and the required scrap/physical prototype before any fit claims or alternative joints.

Keep each step reviewable. If shortened-front complementarity or shoulder/stop geometry cannot be proven with pure fixtures, stop before UI work and cut a geometry prototype.

## 16. Final `git status -sb`

At the end of the review, tracked application files remained unchanged. The only newly created file is this explicitly authorized untracked report:

```text
docs/SLIDING_LID_BOX_ARCHITECTURE_REVIEW_2026-07-15.md
```

All unrelated pre-existing untracked files remain present and untouched. No staged paths were introduced. A final status/diff verification is recorded in the completion response; if the status command lists other untracked paths, they predate and are outside this review.

## 17. Change and repository-safety confirmation

- `index.html` was inspected but not edited.
- No existing report, README, source, data, LightBurn project, or Python file was edited.
- The authorized Markdown report is the sole file created by this review.
- Nothing was staged.
- Nothing was committed.
- Nothing was pushed.
- No stash, reset, clean, move, or delete operation was performed.
- Designs remain session-only: `designDraft` is module-level, absent from `freshState()`, omitted by `persist()`, and omitted by `backupObject()`.
- `STORAGE_KEY` remains `genmitsu-l8-tracker-v1`.
- `SCHEMA_VERSION` remains `2`.
- No network access or external dependency was introduced.

READY FOR BOUNDED IMPLEMENTATION
