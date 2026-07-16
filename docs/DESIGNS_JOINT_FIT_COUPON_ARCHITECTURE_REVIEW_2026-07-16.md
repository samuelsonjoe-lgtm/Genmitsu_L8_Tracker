# Designs Phase 6.1 — Finger-Joint Fit Coupon Architecture Review

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Reviewed baseline: `b1ba69e — Fix finger-panel SVG corner retracing`  
Review type: read-only architecture and implementation planning

## Executive summary

The proposed `Joint Fit Coupon` template is ready for a bounded implementation without changing shared finger-joint semantics, persisted state, backup formats, or the corrected panel-composition engine.

The existing geometry already supports the required physical behavior:

- `buildFingerPattern()` supplies one deterministic odd-count boundary pattern.
- Two ordinary rectangular panels can share that pattern on one edge and use complementary phases.
- `designPatternEdge()` and the corrected `buildFingerPanel()` produce the same finger/recess geometry used by Finger Box and Drawer Cabinet panels.
- Positive clearance creates relief; zero is nominal; a small negative clearance reverses the relief into geometric interference.
- Current shared validation rejects negative joint clearance before geometry generation, but the low-level edge composer accepts signed clearance. Negative support can therefore remain coupon-specific.

Recommended defaults:

| Field | Default |
|---|---:|
| Measured material thickness | `3.00 mm` |
| Mating-edge length | `45 mm` |
| Coupon body depth | `22 mm` |
| Preferred finger width | `9 mm` |
| Clearance values | `0.06, 0.04, 0.02, 0.00, -0.02, -0.04` |
| Coupon-to-coupon spacing | `10 mm` |
| Pair-to-pair spacing | `10 mm` |
| Clearance labels | enabled by default, user may disable |

With two pairs per row, the six-pair default set is approximately `230 × 106 mm`, including the existing `10 mm` document margin.

Current verification before this report:

- tracked working tree: clean;
- branch: `main`, tracking `origin/main`;
- expected baseline: confirmed at `b1ba69e`;
- `git diff --check`: passed;
- current direct `file://` startup: passed;
- current built-in fixtures: `977 passed / 0 failed`;
- current Designs fixtures: `445 passed / 0 failed`;
- the requested report did not exist before this review;
- numerous unrelated untracked files and prior reports were present and were not modified.

## Current architecture summary

### Session-only Designs state

`designDraft` is initialized from `designDefaults()` as a module-local variable. It is not a member of `state`.

The storage boundary is already appropriate:

- `freshState()` contains application records and preferences, but no Designs draft.
- `persist()` explicitly destructures and serializes persisted state fields; it does not serialize `designDraft`.
- `backupObject()` excludes `designDraft`.
- `replaceData()` and `mergeData()` operate only on backed-up application collections and preferences.
- import/export uses `backupObject()` and the existing storage decoder.

The new template can remain session-only by adding only draft defaults and normalization fields. It requires no change to:

- `STORAGE_KEY`, currently `genmitsu-l8-tracker-v1`;
- `SCHEMA_VERSION`, currently `2`;
- `state`;
- `freshState()`;
- `loadState()`;
- `persist()`;
- `backupObject()`;
- merge or replace import behavior;
- existing localStorage data.

### Designs render and dispatch path

The current flow is:

1. `renderDesigns()` renders the template selector and template-specific fields.
2. `updateDesignDraft()` copies form values into the session draft.
3. `normalizeDesignDraft()` validates and converts the selected template's values.
4. `buildDesignResult()` dispatches to a template result builder.
5. The result builder constructs panels, lays them out, creates optional blue score geometry, serializes one SVG, and validates it.
6. `designResultsHtml()` previews that exact serialized SVG.
7. the download handler calls the same result path and downloads the same SVG bytes.

A new explicit branch in these existing decision points is sufficient. A generalized template framework is unnecessary.

### Shared geometry

The relevant shared path is:

```text
buildFingerPattern()
  -> designPatternEdge()
  -> buildFingerPanel()
  -> buildSlidingLidBodyPanel()
  -> designSafePatternEdgePoints()
  -> designPanelFromPoints()
  -> designPanelGeometryErrors()
```

`buildFingerPanel()` now deliberately delegates to the corrected, terminal-offset-aware `buildSlidingLidBodyPanel()`. That is the correct builder for a coupon with one patterned edge and three plain edges.

### Layout, SVG, and diagnostics

The reusable output path is:

```text
layoutDesignPanelRows()
  -> optional buildAssemblyLabelPaths()
  -> serializeDesignSvg()
  -> designSvgValidation()
  -> optional designSerializedAssemblyLabelValidation()
```

Existing diagnostics already cover:

- closed orthogonal panel paths;
- non-finite coordinates;
- zero-length segments;
- self-intersection;
- positive collinear retracing;
- duplicate cut segments;
- panel-bound containment;
- translated panel overlap;
- required layout gap;
- viewBox containment;
- invalid SVG numeric tokens;
- label polygon containment;
- label-to-cut clearance;
- blue score before red cut.

The new result builder should also perform the same translated-cut duplicate check used by Sliding Lid and Drawer Cabinet results.

## Clearance-semantics analysis

### Exact effect of clearance

For each internal nominal boundary, `designSafePatternEdgePoints()` uses:

```javascript
distance = nominal + (currentFinger ? -clearance / 2 : clearance / 2);
```

For a complementary pair:

- the panel whose current segment is a finger shifts that transition by `-clearance / 2`;
- its mate, whose current segment is a recess, shifts the corresponding transition by `+clearance / 2`.

The two realized transitions therefore differ by exactly the requested signed clearance.

For a typical internal segment of nominal width `w`:

| Clearance | Finger width | Recess width | Physical meaning |
|---|---:|---:|---|
| `c > 0` | `w - c` | `w + c` | relief / looser fit |
| `c = 0` | `w` | `w` | nominal geometry |
| `c < 0` | `w + abs(c)` | `w - abs(c)` | interference / tighter press fit |

Terminal segments use only one shifted internal boundary, so their width changes by half the clearance. The internal full segments are the limiting case.

### Answers to the clearance questions

- Smaller positive values do create a tighter fit than larger positive values.
- `0.00 mm` is currently the tightest value accepted by the existing Finger Box, Sliding Lid Box, and Drawer Cabinet form/normalization paths.
- The low-level geometry supports negative values meaningfully: negative clearance widens fingers and narrows their complementary recesses.
- Negative clearance changes transition positions along the mating edge. It does not change the material-thickness recess depth.
- Existing template validation rejects negatives because the relevant numeric inputs use a minimum of zero and call `designRequiredNumber(..., 0, true)`.
- Supporting negative values does not require changing `buildFingerPattern()`, `designPatternEdge()`, `designSafePatternEdgePoints()`, `buildFingerPanel()`, or existing box-template validation.
- The coupon should use its own signed-clearance parser and its own symmetric feature-width check.

### Safe geometric bound

Existing positive-clearance validation checks:

```text
actual segment width - clearance >= required web
```

where:

```text
required web = max(0.5 mm, material thickness × 0.25)
```

That check is one-sided because current templates reject negative values. For signed coupon values, the narrow feature may be either a finger or a recess. The conservative coupon rule should be:

```text
actual segment width - abs(clearance) >= required web
```

Equivalently, the geometry-derived signed range is:

```text
required web - actual segment width
    <= clearance <=
actual segment width - required web
```

For the recommended defaults:

- actual segment width: `45 / 5 = 9 mm`;
- required web at `3 mm` material thickness: `0.75 mm`;
- geometry-only lower bound: `-8.25 mm`.

That mathematical lower bound is not a sensible physical setting. It only identifies where an internal recess would collapse below the structural web. A much tighter product-policy range is appropriate.

Recommended accepted input range for Phase 6.1:

```text
-0.10 mm through +0.30 mm
```

Every value must additionally pass the dynamic symmetric web check. This bounded range:

- contains the proposed test set;
- supports useful slight interference tests;
- avoids presenting extreme negative values as reasonable press-fit settings;
- remains far inside the geometric-collapse limit for practical coupon dimensions.

### Independent coordinate check

With a `45 mm` edge and five `9 mm` nominal segments, nominal internal boundaries are:

```text
9, 18, 27, 36
```

At `+0.04 mm`, complementary realized boundaries are:

```text
Coupon A: 8.98, 18.02, 26.98, 36.02
Coupon B: 9.02, 17.98, 27.02, 35.98
```

At `-0.04 mm`, the offsets reverse:

```text
Coupon A: 9.02, 17.98, 27.02, 35.98
Coupon B: 8.98, 18.02, 26.98, 36.02
```

This confirms that `-0.02` and `-0.04` are valid, meaningful interference tests with the recommended dimensions. It does not prove the resulting physical fit in a particular plywood sheet or laser setup.

### Important fixture terminology

The pair must share identical **nominal pattern boundaries** by reusing one `buildFingerPattern()` result. For a nonzero clearance, its final cut-transition coordinates must not be literally identical; they must be symmetrically offset by the requested clearance. Fixtures should distinguish these two facts.

## Recommended physical coupon design

### Panel form

Each clearance pair should contain two independent rectangular panels:

- Coupon A: one patterned edge, phase `true`;
- Coupon B: the same patterned-edge length and same pattern object, phase `false`;
- remaining three edges: plain;
- recess depth: measured material thickness;
- clearance: the pair's signed test value.

Recommended local panel definition:

```javascript
{
  width: matingEdgeLength,
  height: couponBodyDepth,
  edges: [
    designPatternEdge(sharedPattern, phase, materialThickness, clearance),
    null,
    null,
    null
  ]
}
```

The exact edge index may be changed for convenient sheet orientation, but both pieces must use the same local mating edge and complementary phase.

### Why this tests the real 90-degree joint

This is not a flat puzzle joint. The two flat-cut coupons are intended to be assembled with their panel planes perpendicular:

1. keep the patterned edges aligned;
2. rotate one panel 90 degrees out of the other's plane;
3. mate the alternating material-thickness-deep fingers and recesses.

The relevant cut geometry is the same as the Finger Box and Drawer Cabinet edge geometry:

- same nominal odd-count pattern generation;
- same material-thickness recess depth;
- same complementary phases;
- same signed boundary offsets;
- same terminal-offset-aware corner stitching.

The UI and result instructions must explicitly say “assemble A and B at 90 degrees.” Without that instruction, a user could incorrectly treat the pieces as a coplanar puzzle test.

### Practical defaults

| Parameter | Recommendation | Reason |
|---|---:|---|
| Material thickness | `3.00 mm` | matches the current measured plywood context |
| Mating-edge length | `45 mm` | enough length for representative fingers without excessive material |
| Coupon body depth | `22 mm` | sufficient grip and label area without becoming box-sized |
| Preferred finger width | `9 mm` | produces exactly five segments on a 45 mm edge |
| Piece gap | `10 mm` | reuses the proven deterministic row layout safely |
| Pair gap | `10 mm` | permits the existing single-gap layout helper with no custom placer |
| Margins | existing `10 mm` | reuses current layout behavior |

At these defaults, `buildFingerPattern()` chooses five `9 mm` segments. The minimum permitted segment width at `3 mm` thickness is `6 mm`, so the default remains structurally conservative.

### Layout

Recommended deterministic order:

```text
row 1: p01-a, p01-b, p02-a, p02-b
row 2: p03-a, p03-b, p04-a, p04-b
row 3: p05-a, p05-b, p06-a, p06-b
```

Using the existing `10 mm` margin and `10 mm` gap:

```text
width  = 20 + (4 × 45) + (3 × 10) = 230 mm
height = 20 + (3 × 22) + (2 × 10) = 106 mm
```

This uses approximately one quarter of the panel area of repeated full boxes while keeping the parts large enough to handle.

## Recommended UI fields and defaults

Add one template option:

```text
Joint Fit Coupon
```

Recommended session-only fields:

| Draft field | UI label | Default |
|---|---|---:|
| existing `materialThickness` | Measured material thickness (mm) | `3.00` |
| `jointCouponEdgeLength` | Mating-edge length (mm) | `45` |
| `jointCouponBodyDepth` | Coupon grip depth (mm) | `22` |
| `jointCouponPreferredFingerWidth` | Preferred finger width (mm) | `9` |
| `jointCouponClearances` | Clearance values (mm) | `0.06, 0.04, 0.02, 0.00, -0.02, -0.04` |
| `jointCouponLabels` | Add clearance labels (blue score layer) | checked |

Use a text input or compact textarea for an explicit comma/newline-separated list. Do not use start/end/step or center/increment generation in Phase 6.1.

An explicit list is preferable because it:

- represents the exact values the user intends to cut;
- allows asymmetrical or reordered test sets;
- avoids cumulative floating-point sequence errors;
- makes duplicate detection straightforward;
- avoids silently adding an unwanted endpoint.

### Decimal parsing

Parse each token as a bounded decimal string, not as a generated floating sequence.

Recommended rules:

- separators: commas and/or line breaks;
- optional leading sign;
- one or two decimal places for Phase 6.1;
- no exponent notation;
- convert to integer hundredths of a millimeter;
- normalize `-0.00` to `0.00`;
- detect duplicates after integer normalization;
- preserve the user's normalized list order;
- format every metric and label with exactly two decimals.

Example canonical representation:

```javascript
{
  hundredths: -4,
  value: -0.04,
  text: '-0.04'
}
```

Using integer hundredths for identity and formatting prevents values such as `0.019999999`.

### Result metrics and instructions

Show:

- tested clearance list;
- pair count;
- physical piece count;
- mating-edge length;
- coupon body depth;
- segment count and actual segment width;
- measured thickness;
- label count and any omissions;
- final layout dimensions;
- explicit “assemble each A/B pair at 90 degrees” guidance;
- explicit reminder that laser kerf is separate and must be established with material testing.

## Labels and LightBurn layers

### Current glyph capability

The current `designAssemblyGlyphs` map contains only a limited uppercase-letter set. It includes `A` and `B`, but it does not include:

- digits `0`–`9`;
- decimal point `.`;
- minus sign `-`;
- plus sign `+`;
- space.

Current code rejects an unsupported glyph and emits a deterministic warning rather than a partial word.

### Recommended label format

Use compact labels without spaces or a plus sign:

```text
A0.04
B0.04
A-0.02
B-0.02
```

Add deterministic vector glyphs for:

```text
0 1 2 3 4 5 6 7 8 9 . -
```

No space or plus glyph is required for this format. Do not use SVG `<text>` or external fonts.

The existing `4.5 mm` cap height should fit within the recommended `45 × 22 mm` coupon body, but placement must still use `buildAssemblyLabelPaths()` and its polygon/cut-clearance checks. If a label cannot fit:

- omit only that score label;
- retain the valid cut output;
- emit the existing visible deterministic warning naming the label and panel.

### Layer order

Reuse `serializeDesignSvg()` unchanged:

```text
score
  assembly-labels
cut
```

When labels are disabled, no score group is needed. Preview and download continue to consume the same serialized SVG string.

## IDs and deterministic ordering

Use ordinal IDs independent of punctuation in the clearance:

```text
joint-coupon-p01-a
joint-coupon-p01-b
joint-coupon-p02-a
joint-coupon-p02-b
...
```

Reasons:

- every physical coupon remains separate;
- IDs contain no decimal point or minus sign;
- two values that format differently cannot accidentally alter record identity;
- ordering remains stable even if future display formatting changes.

Keep the exact clearance in metrics:

```javascript
{
  pairIndex: 1,
  clearanceHundredths: 6,
  clearance: 0.06,
  clearanceText: '0.06',
  couponAId: 'joint-coupon-p01-a',
  couponBId: 'joint-coupon-p01-b'
}
```

Build pairs in normalized input order and always emit A before B. Layout rows must be derived from these IDs, not from display labels or numeric sorting.

## Proposed geometry flow

Recommended bounded flow:

```text
normalizeDesignDraft()
  -> parseJointCouponClearances()
  -> buildJointCouponDesignResult()
       -> buildFingerPattern(edgeLength, preferredWidth, thickness)
       -> for each clearance:
            validate abs(clearance) against structural web
            build Coupon A with phase true
            build Coupon B with phase false
            run designPanelGeometryErrors() for both
       -> layoutDesignPanelRows()
       -> check translated duplicate cuts
       -> optional buildAssemblyLabelPaths()
       -> serializeDesignSvg()
       -> designSvgValidation()
       -> optional designSerializedAssemblyLabelValidation()
```

Recommended small new helpers:

- `parseJointCouponClearances(raw, errors)`;
- `buildJointCouponModel(parameters)`;
- `buildJointCouponDesignResult(normalized)`;
- optionally `jointCouponLabelText(pair, side)`.

Do not:

- create a second finger-pattern algorithm;
- change the meaning of shared clearance;
- permit negative clearance in existing box templates;
- refactor current builders into a generic framework;
- persist the draft.

## Validation rules

### Blocking input validation

1. Material thickness must be present, finite, and at least the existing `0.1 mm` minimum.
2. Mating-edge length must be finite and greater than zero.
3. Coupon body depth must be finite and greater than zero.
4. Preferred finger width must be finite and greater than zero.
5. Clearance list must contain between `3` and `8` values.
6. Each clearance token must be a valid signed fixed decimal with no more than two decimal places.
7. Each clearance must be within `-0.10` through `+0.30 mm`.
8. Canonically equal values, including `0.00` and `-0.00`, must be rejected as duplicates.
9. `buildFingerPattern()` must produce a valid odd-count pattern of at least three segments.
10. Coupon body depth must leave sufficient plain grip material after accounting for material thickness and label clearance.

Recommended minimum body-depth rule:

```text
coupon body depth >= max(4 × material thickness, 16 mm)
```

The `22 mm` default passes comfortably at `3 mm` thickness.

### Blocking geometry validation

For every tested clearance:

```text
actual segment width - abs(clearance) >= required web
```

Then require:

- finite panel points;
- closed orthogonal paths;
- no zero-length segments;
- no self-intersections;
- no positive collinear overlap/retracing;
- no duplicate segment within a panel;
- panel geometry within its envelope;
- exactly two panels for each value;
- exactly one patterned edge on each panel;
- complementary phases;
- shared nominal pattern boundaries;
- corresponding realized transitions separated by the signed clearance;
- no translated duplicate cuts;
- no panel or actual-path overlap;
- all geometry within SVG bounds;
- valid parsed SVG.

### Label validation

Labels remain non-blocking:

- unsupported glyph or unsafe placement omits only that label;
- warning names the exact omitted label and coupon;
- cut output remains valid;
- every emitted label stays inside its coupon;
- minimum label-to-cut clearance is `max(1 mm, material thickness / 3)`;
- score geometry precedes cut geometry.

### Maximum pair count

Use a maximum of `8` pairs. This limits the result to:

- `16` cut panels;
- a predictable number of score paths;
- a compact browser preview;
- a manageable fixture surface;
- a layout that remains practical on common laser beds.

The minimum should be `3` pairs because a one- or two-value output does not adequately bracket a fit trend and conflicts with the stated purpose of testing several clearances.

## Fixture plan

Add focused fixtures inside `runDesignGeometryFixtures()` without rewriting the fixture infrastructure.

### Parsing and validation

- default list parses to integer hundredths `[6, 4, 2, 0, -2, -4]`;
- canonical text is exactly `0.06|0.04|0.02|0.00|-0.02|-0.04`;
- blank list rejects;
- non-finite/malformed token rejects;
- exponent notation rejects;
- more than two decimal places rejects;
- fewer than three values rejects;
- more than eight values rejects;
- duplicate value rejects;
- `0.00` and `-0.00` are duplicate;
- `-0.10` and `+0.30` boundary values accept;
- values beyond either bound reject;
- zero clearance accepts;
- negative `-0.02` and `-0.04` accept;
- draft object remains byte-for-byte unchanged.

### Pair structure and identity

- exact pair count equals parsed value count;
- exact piece count equals `2 × pair count`;
- every expected stable ID appears once;
- every pair emits A before B;
- reversing or changing unrelated source-object property order does not change output;
- every physical panel is a separate SVG group;
- no clearance punctuation appears in panel IDs;
- metrics preserve exact hundredths, numeric value, and two-decimal text.

### Independent mating geometry

Do not verify only by comparing two outputs produced by the same helper.

For a literal `45 mm`, five-segment test pattern, independently assert:

- nominal boundaries are exactly `[0, 9, 18, 27, 36, 45]`;
- at `+0.04`, A and B internal transitions match the literal arrays documented above;
- at `-0.04`, those offset arrays reverse;
- at zero, A and B transitions coincide with nominal boundaries;
- phases are complementary;
- both pieces use equal mating-edge length;
- both pieces reference equal nominal pattern boundaries;
- each corresponding nonzero transition separation is exactly `abs(clearance)`;
- the direction of the separation matches the clearance sign;
- recess depth equals measured material thickness.

### Panel and SVG geometry

- each coupon has exactly one patterned edge and three plain edges;
- all panel paths are closed and orthogonal;
- no zero-length segment;
- no self-intersection;
- no positive collinear retracing;
- no duplicate local or translated cut segment;
- no layout overlap;
- required gap is retained;
- geometry remains inside document bounds;
- SVG parses and contains no invalid numeric token;
- default result is deterministic and byte-stable;
- preview and download generation return identical SVG;
- cut layer follows score layer when labels are enabled;
- label-disabled output contains no score group;
- label-disabled output is byte-stable.

### Labels

- all required digit, decimal, and minus glyphs are valid deterministic vector geometry;
- no SVG `<text>` exists;
- exact labels for every default pair are emitted in A/B order;
- negative labels contain the vector minus glyph;
- label paths stay inside coupon polygons;
- label paths preserve minimum cut clearance;
- an intentionally undersized coupon omits only unsafe labels, emits named warnings, and retains valid cut SVG;
- labels do not change cut-panel paths.

### Isolation and regression

- generation does not mutate the draft;
- generation does not write `localStorage`;
- backup output has no coupon draft fields;
- existing Finger Box negative-clearance rejection remains;
- existing Sliding Lid Box negative-clearance rejection remains;
- existing Drawer Cabinet negative-clearance rejection remains;
- existing Finger Box signatures remain unchanged;
- existing Sliding Lid Box signatures remain unchanged;
- existing Drawer Cabinet signatures remain unchanged;
- all existing legacy-template signatures remain unchanged;
- current total suite remains green.

## Storage and compatibility assessment

The feature can be fully isolated from persistent data.

Implementation should:

- add coupon defaults only to `designDefaults()`;
- keep all values in `designDraft`;
- avoid calling `persist()`;
- avoid reading or writing `state`;
- avoid adding any backup key;
- avoid schema migration;
- avoid changing import merge/replace handling.

Compatibility risk is low if the implementation retains the current explicit persistence whitelist. A fixture should still compare `localStorage.getItem(STORAGE_KEY)` before and after coupon generation, and another should assert that `backupObject()` has no coupon-draft property.

## Files likely to change

Expected implementation changes:

1. `index.html`
   - template option and fields;
   - session draft defaults;
   - coupon-specific normalization/parser;
   - coupon model/result builder;
   - compact numeric vector glyphs;
   - metrics and assembly guidance;
   - Designs fixtures.
2. `README.md`
   - add the Joint Fit Coupon to the Designs description;
   - explain that it tests 90-degree finger-joint clearances;
   - mention signed coupon-only clearances and optional deterministic score labels;
   - update verified fixture totals after implementation.
3. a future implementation report in `docs/`, if requested.

No other application file, storage file, dependency manifest, or schema artifact should change.

## Risks and unverified physical assumptions

### Physical fit remains material-specific

The geometry proves the direction and magnitude of signed boundary offsets. It does not prove that `-0.02` or `-0.04 mm` will produce a usable press fit on the current laser and plywood.

Physical results still depend on:

- measured sheet thickness;
- laser kerf and focus;
- glue layers and plywood density;
- char and cleanup;
- grain direction;
- humidity;
- machine calibration;
- assembly angle and force.

Negative clearance can increase insertion force and may crush or split thin basswood. The UI should call it “interference” or “tighter than nominal,” not promise a press fit.

### Coupon orientation

The coupons test the real joint only when assembled at 90 degrees. The result instructions must be prominent enough to prevent a coplanar puzzle-fit interpretation.

### One-edge sample versus a full box

The coupon reproduces the local edge fit but not:

- accumulated dimensional error around four corners;
- panel bow;
- squareness;
- glue-up stress;
- large-panel flex;
- full-box grain interactions.

A successful coupon should select the next full prototype clearance, not replace final box validation.

### Label scoring

Label visibility depends on score settings and material. Omitted-label warnings are non-destructive, but unlabeled loose coupons can be confused after removal from the sheet. The generated result should recommend keeping pairs organized if labels are disabled or omitted.

### Shared-helper change risk

No shared helper change is required for the recommended plan. Risk rises materially if implementation instead:

- changes `designRequiredNumber()` to accept negatives globally;
- changes existing `buildBoxModel()` web semantics;
- changes pattern boundary calculations;
- introduces a separate finger algorithm.

Those alternatives should be rejected for Phase 6.1.

## Drawer Cabinet lateral-clearance scope

Do not change the Drawer Cabinet default lateral clearance from `0.40 mm` to `0.45 mm` in this phase.

The physical result is useful and should be recorded, but drawer running clearance is independent from finger-joint clearance. Combining the change would:

- mix a verified Drawer Cabinet tuning change with a new coupon template;
- alter existing Drawer Cabinet default geometry and SVG signatures;
- complicate regression attribution;
- expand acceptance criteria beyond the coupon.

Make `0.45 mm` a separate, small, explicitly reviewed default update.

## Recommended implementation phases

### Phase A — Coupon model and signed parsing

- add template and session-only fields;
- implement fixed-decimal list parser;
- implement coupon-specific signed range and symmetric web validation;
- build A/B panels with one shared nominal pattern and complementary phases;
- add deterministic IDs, metrics, layout, SVG, and geometry validation;
- keep labels disabled internally until the cut path is fully covered by fixtures.

### Phase B — Numeric vector labels

- add digit, decimal, and minus glyphs;
- add exact A/B clearance labels;
- route labels through existing safe-placement and warning behavior;
- verify score-before-cut and unchanged cut paths.

### Phase C — Regression and documentation closeout

- add independent literal-coordinate fixtures;
- add storage/nonmutation fixtures;
- preserve every existing template signature;
- run direct `file://` startup and the complete fixture suite;
- update README and verified totals.

These are implementation checkpoints within Phase 6.1, not separate schema or product migrations.

## Independent-review recommendation

The recommended plan does not require changing shared clearance semantics or shared finger-joint helpers. Under the stated review criterion, an additional Grok or Claude architecture review is not necessary before implementation.

An independent code audit becomes worthwhile only if implementation departs from this boundary, particularly if it modifies:

- `buildFingerPattern()`;
- `designSafePatternEdgePoints()`;
- `buildFingerPanel()`;
- existing template validation;
- shared clearance meaning.

## Explicit conclusion

**READY FOR IMPLEMENTATION**

The implementation should proceed with an explicit fixed-decimal clearance list, coupon-only bounded negative values, a symmetric feature-width safety check, ordinary one-patterned-edge finger panels assembled at 90 degrees, deterministic ordinal IDs, and reuse of the current layout, label, SVG, and validation systems.
