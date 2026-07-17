# Designs Tray Model Extraction Architecture Review

Date: 2026-07-17  
Baseline reviewed: `16128e0 Add sliding lid box finished views`

## 1. Executive conclusion

The Dice Tray and Divider Tray are good candidates for a shared structured Tray model, but neither can safely receive a Finished View directly from the current SVG. Both templates flow through a single legacy, flattened generator and expose only anonymous `Shape N` items after validation. The production geometry is small and well-characterized, so extraction is feasible without redesigning the output.

The safest route is a temporary, test-only dual path: build an authoritative semantic model in parallel, serialize it with a compatibility serializer, and prove the generated SVG is byte-identical for a defined valid-input matrix before routing production through it. The temporary legacy path must be removed when the switch is complete; it must not become a permanent second generator.

The only material product decision still needed is a canonical assembled orientation. Current Dice geometry is symmetric and does not establish an inherent Front/Back. Divider geometry establishes a physical axis, but not a user-facing Front/Back. The recommended convention is documented in section 7 and must be accepted before a Finished View can make orientation claims.

**Final status: READY AFTER DESIGN DECISION**

## 2. Current implementation and shared-generator assessment

Both templates use the same legacy branch:

```text
buildDesignResult(draft)
  -> buildLegacyDesignResult(draft)
    -> legacyDesignSvg(normalized.draft)
      -> designTraySvg(draft) for dice-tray and divider-tray
```

`designTraySvg()` derives the entire layout directly into a single ordered array of SVG fragments. `designSvgDocument()` wraps that array in one anonymous red-stroke group. `buildLegacyDesignResult()` subsequently validates the SVG and represents each parsed element as a generic `Shape N` panel. No stable component ID, panel role, assembly orientation, operation metadata, or divider-to-slot relationship survives that process.

The templates are one generator with one conditional divergence:

| Concern | Dice Tray | Divider Tray |
| --- | --- | --- |
| Base, four walls, material, clearance, wall tabs, layout | Shared | Shared |
| `jointStyle` behavior | Shared: `finger` uses four tabs; `tab-slot` uses two | Same |
| Base slots for walls | Shared | Shared |
| Interior divider slots | None | One row across the base per divider |
| Divider panels | None | One rectangular panel per divider |
| Input unique to template | None | `dividerCount` (1–6 integer) |

The UI calls the first option “Finger” and the helper text says “Finger-tab / Tab-and-slot tray layout.” In the current production code, the distinction is only the tab count (`4` versus `2`); it is not a separate structural finger-joint algorithm. The extracted model must preserve that behavior and terminology without implying a capability that the SVG does not currently have.

## 3. Current production geometry contract

For valid normalized inputs, the common tray geometry is:

- `t = materialThickness`; `c = fitClearance`.
- Requested `trayWidth` and `trayDepth` are inside dimensions; `trayHeight` is wall height.
- Base outside dimensions are `insideWidth + 2t` by `insideDepth + 2t`.
- Wall-slot width is `t + c`.
- The base is a rectangle. Four wall-slot runs are cut into it: two runs parallel to width and two parallel to depth.
- Two full-width wall paths use the base outside width. Two side-wall paths use the inside depth. Each wall has tabs on its lower long edge only.
- Tab width is clamped to 8–16 mm independently on width and depth runs. The validation rule rejects tray edges too short for the current tab count and tab width.
- The document uses one red cut group. It emits no score paths, labels, operation IDs, layers, or panel IDs.

At the current defaults (`220 × 160 × 35 mm`, `3 mm` material, `0.15 mm` clearance, Finger), Dice Tray emits one base rectangle, sixteen wall-slot rectangles, and four wall paths. Divider Tray at its default count of two additionally emits two divider-slot rectangles and two divider-panel rectangles. The currently committed default golden SVGs are 1,726 characters / hash `51a55721` for Dice and 1,965 characters / hash `a55dda6e` for Divider.

### Divider-specific behavior

For `dividerCount = n`, the generator places `n` equally spaced slot rows at:

```text
insideDepth * (index + 1) / (n + 1)
```

Each slot runs across the inside width, and each divider is an individual rectangle of `insideWidth × wallHeight`. The result is a one-direction divider system: dividers span Left-to-Right and separate the depth axis into `n + 1` compartments. There are no cross-dividers, notches, divider tabs, score marks, labels, or distinct cut-operation semantics.

The UI’s “Removable divider slots” wording supports a removable intent, but the output alone does not prove retention, insertion direction, a stop, or a production-safe fit. The model should record the intended removable relationship and its source, not assert unverified mechanical performance.

## 4. Recommended model boundary

Introduce one pure builder after validation/normalization:

```js
buildTrayModel(values)
```

`values` should be the validated normalized numeric values, not a raw UI draft. That prevents fallback parsing and validation from diverging between model and export. The public result builder can retain the existing UI contract:

```text
normalizeDesignDraft(draft)
  -> validateLegacyDesign(normalized)
  -> buildTrayModel(normalized.values)
  -> buildTrayDesignResult(model, normalized)
  -> serializeTrayCompatibilitySvg(model)
```

Use shared helpers for tray dimensions, tab profile, base wall slots, wall components, divider components, and sheet layout. Do not parse the old SVG into a model, and do not retain a second independently-calculated tray geometry implementation.

### Proposed authoritative model

```text
TrayModel
  template: 'dice-tray' | 'divider-tray'
  source: { generatorVersion, normalizedInputVersion }
  inputs:
    materialThicknessMm, fitClearanceMm, jointStyle
    insideWidthMm, insideDepthMm, wallHeightMm
    dividerCount (Divider only)
  dimensions:
    baseOutsideWidthMm, baseOutsideDepthMm, slotWidthMm
    widthTabProfile, depthTabProfile
  orientation:
    interiorFace: 'top'
    frontWallId, backWallId, leftWallId, rightWallId
    dividerSpan: 'left-to-right' (Divider only)
    dividerOrder: 'front-to-back' (Divider only)
  components:
    base { id, name, role, cutGeometry, wallSlots[], dividerSlots[] }
    walls[] { id, name, role, widthMm, heightMm, cutGeometry, tabProfile, baseSlotIds[] }
    dividers[] { id, name, index, widthMm, heightMm, cutGeometry, baseSlotId, intendedAssembly }
  operations:
    cutPaths[]
    scorePaths: []
    labels: []
  layout:
    orderedItems[] { componentId, geometry, xMm, yMm }
    widthMm, heightMm, marginMm, gapMm
  metrics:
    pieceCount, wallCount, dividerCount, compartmentCount
```

The model owns production geometry and semantics. The result builder owns validation presentation, existing download metadata, the `panels` projection used by the UI, and any future screen-only preview payload. The serializer owns only the model-to-SVG projection. A Finished View must consume the model or result’s explicit semantic projection, never infer parts from flattened cut-layout coordinates.

### Field ownership and applicability

| Field group | Dice Tray | Divider Tray | Source / unit | Production or screen-only |
| --- | --- | --- | --- | --- |
| Template, material thickness, clearance, joint style | Yes | Yes | Current form; mm except enum | Production |
| Inside width, depth, wall height | Yes | Yes | Current form; mm | Production |
| Base outside dimensions, slot width, tab profiles | Derived | Derived | Model calculation; mm | Production |
| Base, four named walls, wall slots | Yes | Yes | Model calculation | Production and semantic screen input |
| Divider count, slots, named divider panels | No | Yes | Current form / model; count and mm | Production and semantic screen input |
| Canonical orientation and divider order | Yes | Yes | Product convention, not layout inference | Semantic screen input; stable model metadata |
| Finished-view camera, colors, shadows, annotations | Future | Future | Renderer | Screen-only |
| Labels and score paths | None currently | None currently | Existing SVG contract | Production empty collections; future explicit capability |

## 5. Production-equivalence strategy

Three options were considered:

| Option | Assessment |
| --- | --- |
| A. Require byte-identical SVG | Strongest initial guard for existing valid configurations; preserves element ordering, numeric formatting, anonymous group structure, and downstream import expectations. |
| B. Require geometry-equivalent SVG only | Necessary as a complementary structural assertion, but too weak as the sole first gate because element ordering, grouping, and formatting may be observable to downstream tooling. |
| C. Temporary dual path | Safest transition mechanism when constrained to tests/fixtures and removed after the switch. It is not acceptable as a permanent production architecture. |

Recommendation: use **C to prove A**, with B as a diagnostic assertion. First extract the model and an intentionally compatibility-oriented serializer while `designTraySvg()` remains the production source. Fixture code compares old and model SVG output for a defined matrix. Once byte equality passes, switch the two tray templates to the model serializer and delete the legacy tray dispatch. Keep geometry-level assertions to make future intentional serializer changes understandable.

The compatibility serializer must initially preserve the legacy output contract:

- one anonymous red cut group produced by `designSvgDocument()`;
- no `id="cut"`, score group, titles, labels, or panel grouping;
- current shape order: base, wall slots, wall paths, then Divider-only divider slots and panels;
- existing `designRound`/string formatting, margins, gaps, viewBox sizing, and dimensions;
- existing `l8-dice-tray-YYYY-MM-DD.svg` and `l8-divider-tray-YYYY-MM-DD.svg` download filenames.

Do not route this first extraction through the newer generic `serializeDesignSvg()`: it emits named panel groups and cut/score layers, which would change the exported SVG even if the physical paths match.

## 6. Component inventory and semantics

### Dice Tray

| Component | Required semantic identity | Current physical evidence |
| --- | --- | --- |
| Base | `tray-base` | One rectangular panel with four wall-slot runs. |
| Front wall | `wall-front` | First full-width wall path in current emission order. |
| Back wall | `wall-back` | Second full-width wall path. |
| Left wall | `wall-left` | First depth-length wall path. |
| Right wall | `wall-right` | Second depth-length wall path. |
| Wall slots | Stable IDs such as `base-slot-front-01` | Rectangular cuts in the base. |

The design is open-top and currently has no liner, lid, retention feature, labels, or score marks. It is safe for a future Finished View to show an open tray with base and four walls, while explicitly stating that the view is screen-only and does not prove fit, strength, clearance, glue quality, or production safety. It must not depict or imply a liner, fasteners, glued corners, or verified interchangeability.

Opposing Dice walls are geometrically symmetric under the current generator. “Interchangeable” should therefore be recorded narrowly, if at all: equivalent cut geometry for the current settings, not an assembly or verification claim. Future labels, asymmetrical accessories, or altered joints could invalidate the equivalence.

### Divider Tray

In addition to the Base and four walls, the model needs:

| Component | Required semantic identity | Current physical evidence |
| --- | --- | --- |
| Divider slot | `base-divider-slot-01` … | One base cut per configured divider. |
| Divider | `divider-01` … | One separate rectangular panel per configured divider. |
| Divider relationship | Divider ID references its slot ID | Equal depth-axis placement and matching count. |
| Compartment metadata | `dividerCount + 1` | Derived from a one-direction divider system. |

The current layout supports only one divider direction. The model should call it `left-to-right` span and `front-to-back` ordering only after the canonical orientation decision below. Do not invent cross-divider, grid, dovetail, fixed, retained, or captive-divider semantics. The UI describes the divider slots as removable, but a Finished View should state the divider placement as an intended assembly arrangement, not as proof of a safe removable mechanism.

## 7. Orientation decision required before Finished Views

Current cut-layout placement is not an assembled orientation. It must not be used as the source of semantic names. The current generator merely happens to emit two full-width walls before the two side walls; the base is symmetric, and no existing user-facing Front/Back label selects an edge.

Recommended product convention, pending approval:

```text
Interior is viewed from above.
Front = the first full-width wall / near width edge in the canonical assembled model.
Back  = the opposing second full-width wall.
Left and Right = the first and second depth-length walls when facing Front.
For Divider Tray, dividers span Left-to-Right and are numbered Front-to-Back
by increasing modeled divider position: Divider 01 is nearest Front.
```

This convention is deliberately independent of sheet placement. It changes no production geometry and gives future views, labels, accessibility text, and documentation a deterministic reference frame. It is a product decision because the present Dice Tray has no physical feature that makes the selected edge inherently front. If a different convention is preferred, select it once in the model contract and preserve it in fixtures.

## 8. Joint, wall, and divider compatibility boundaries

Keep the following concerns separate in the model and UI language:

| Concern | Current evidence | Correct treatment |
| --- | --- | --- |
| Structural compatibility | Base wall slots are sized `materialThickness + clearance`; wall tabs are generated from current tab profile. | Represent geometry and mating relationships. Continue existing validation. Do not claim strength, fit, or safe assembly. |
| Appearance compatibility | No labels, score marks, veneers, liners, or finished-view metadata exist. | Screen renderer may add visual cues only as screen-only decoration, never as cut geometry. |
| Decorative features | None in current trays. | Keep absent from the production model until a separately specified feature has a material, operation, and validation contract. |
| Divider compatibility | Same count and depth-axis positions link base slots to divider panels. | Model an intended removable placement. Do not assert retention, fixed seating, or cross-grid compatibility. |

The current validation permits `fitClearance >= 0` and guards against undersized edges for the active two- or four-tab profile. The extracted model should retain those exact validation rules in the first switch. It should not silently expand the allowed dimensions, reinterpret clearance as kerf compensation, or change the joint strategy.

## 9. Persistence, import, evidence, and application boundaries

The current tray form state is part of the shared Designs draft and uses the existing template string plus current form keys (`jointStyle`, `materialThickness`, `fitClearance`, `trayWidth`, `trayDepth`, `trayHeight`, and Divider-only `dividerCount`). A model extraction should remain ephemeral at first: build it from the normalized draft every refresh and do not add a stored `trayModel` object.

This preserves the following boundaries for the extraction phase:

- no local-storage schema migration;
- no change to saved Designs records, Test Grid records, evidence promotion, production-setting records, backups, imports, or exports;
- no new machine identity, verification, or production-safe claim;
- no change to the download action or current filenames;
- no change to LightBurn-visible layer/group behavior; the legacy anonymous red cut group remains the exported contract;
- no new inputs, defaults, dimensions, labels, score settings, or helper text unless separately specified.

When a later feature needs persistence (for example, a user-selected divider orientation or non-default finished-view option), store only its source input. Rebuild derived model geometry on load. Version any newly persisted semantic enum and preserve absent values as legacy defaults rather than guessing from old SVGs.

## 10. Fixture and validation plan

Retain the existing default Dice and Divider byte-stability fixtures as protected baseline assertions. Add model-specific fixtures before switching production routing:

1. **Golden compatibility matrix.** Include both templates, both joint-style values, normal dimensions, minimum valid dimensions near the current edge rule, zero and default clearance, varied material thickness, wall-height boundaries, and Divider counts 1, 2, and 6. For each valid case, assert legacy and model compatibility SVG are byte-identical.
2. **Semantic component assertions.** Assert unique stable IDs; exactly one base; four walls; expected wall-slot runs; zero Divider components for Dice; and exactly `dividerCount` slots/panels plus `dividerCount + 1` compartments for Divider.
3. **Geometry assertions.** Assert finite dimensions; slot width `t + c`; base outside dimensions; tab profile/count; expected divider position formula; divider-to-slot linkage; valid layout bounds; no overlaps introduced by the projected layout; and empty score/label operation collections.
4. **Serializer assertions.** Parse the output and compare element type/order/attributes as well as full bytes. Assert one legacy-style red group, no score group, no titles, and no accidental layer or filename change.
5. **Invalid-input parity.** Cover all existing tray validation failures, including too-short edges and invalid Divider counts. Invalid results must remain non-downloadable and must not construct misleading finished-view data.
6. **Browser-level checks.** Exercise the current Designs form, template switch, default preview, download callback, and direct `file://` startup. Verify no persisted-state or import/backup behavior changes.

During the first implementation pass, the current baseline to preserve is **1,420 passed / 0 failed** for the complete self-test suite and **628 passed / 0 failed** for the Design geometry group. The implementation must update those totals only from observed test output, never from planned fixture counts.

## 11. Incremental implementation plan, estimates, and gates

| Phase | Scope | Estimate | Main risk | Gate |
| --- | --- | --- | --- | --- |
| 0. Orientation decision and characterization | Accept canonical orientation; capture compatibility matrix from the current generator. | S | Ambiguous Front/Back becomes accidental API. | Written convention and baseline fixture set accepted. |
| 1. Shared model extraction | Add `buildTrayModel(values)` and shared geometry helpers; project model to compatibility SVG in fixtures only. | M | Duplicate math or changed numeric/order behavior. | All old-vs-model valid fixtures byte-identical; invalid parity holds. |
| 2. Production switch | Route Dice and Divider through model result/compatibility serializer; remove tray dispatch from legacy generator. | M | User-visible SVG or LightBurn grouping drift. | Full self-test, Design geometry, browser preview/download, and direct-open checks pass. |
| 3. Dice Finished View | Add a screen-only open-tray view from semantic model data. | S–M | Visual rendering implies unproven fit/assembly. | Model fixture plus focused UI and accessibility audit. |
| 4. Divider Finished View | Add screen-only divider view and compartment indication from model data. | M | Divider direction/count semantics or removable claim becomes misleading. | Focused model/view audit plus UI, direct-open, and regression checks. |

Run a focused independent audit after Phase 1 before the production switch, after Phase 2 because it crosses the production SVG boundary, and after each Finished View phase. The Phase 3/4 audit should specifically inspect semantic wording, orientation, accessibility labels, and separation between visual depiction and production claims.

## 12. Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| A refactor changes red-cut SVG grouping or element order while paths appear visually similar. | Compatibility serializer and byte-for-byte matrix before production switch. |
| Model semantics are guessed from sheet placement. | Accept one explicit canonical orientation; prohibit renderer inference from layout coordinates. |
| “Finger” language is mistaken for a true finger-joint implementation. | Preserve current two-versus-four-tab behavior as the production definition; defer a new joint system to a separate feature. |
| A Finished View overstates divider retention, wall strength, fit, or safe production use. | Keep current geometry facts separate from screen-only visual explanation and safety disclaimer. |
| Divider model grows into unsupported cross-grid or liner features. | Represent only current one-direction, individually cut divider panels; require new inputs/specification for any expansion. |
| New model data leaks into storage/import/records unnecessarily. | Keep model derived and ephemeral for this extraction; version only future user-authored semantic fields. |
| Generic structured SVG serializer changes LightBurn behavior. | Use a tray compatibility serializer initially; defer intentional layer/ID redesign to a separately reviewed migration. |

## 13. Recommended next Codex task

> Implement Phase 0–1 of the Tray model extraction in `C:\Genmitsu L8 Tracker`: after recording the approved canonical tray orientation in code comments/tests, add a pure `buildTrayModel(normalizedValues)` and shared helpers for the current Dice/Divider geometry. Keep `designTraySvg()` as the production source during this pass. Add only fixture/shadow serialization coverage proving the model compatibility serializer is byte-identical to the legacy tray SVG across the agreed valid matrix and has invalid-input parity. Do not add Finished Views, new inputs, persistence/schema changes, labels, score paths, LightBurn layer changes, or commits. Run the focused model audit before switching the production route.

This task deliberately stops before changing production output. Phase 2 should be authorized only after the orientation decision and Phase 1 fixture evidence are reviewed.
