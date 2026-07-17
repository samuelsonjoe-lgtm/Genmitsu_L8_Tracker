# Designs Joint & Assembly System — Architecture Review

**Date:** 2026-07-17
**Repository:** `C:\Genmitsu L8 Tracker`
**HEAD at review time:** `5233609` — *Add dice tray finished view*
**Working-tree state:** uncommitted changes in `index.html` (+66/−18 per stat; 17 hunks) and `README.md` (+2/−2) — the Divider Tray Finished View plus its post-audit `1 divider` pluralization cleanup, both already documented in the working-tree README (totals 1664 / 872 / 244). `git diff --check` clean (CRLF warnings only). Untracked: prior reports under `docs/`, `LightBurn Projects/`, `debug.log`, and other non-application files, all left untouched.
**Review type:** read-only architecture and planning. No application file was modified.
**Method:** direct source inspection of the current `index.html` (10,462 lines) — every function named below was read in this pass or verified byte-identical to a version read in a prior focused audit in this chain via diff-hunk-range analysis; fixture totals were recomputed with the established loop-aware counting script (Tray 244 = 46 non-loop + 8×23 matrix + 1×14 invalid; Designs geometry 872 = 628 own + 244 nested Tray; suite 1664 — all reconcile exactly with the working-tree README). Report summaries were used only as leads, never as evidence.

---

## 1. Executive conclusion

The Designs system is in an unusually good position to grow a joint-style architecture, because the load-bearing abstractions already exist in embryo and have been hardened by eleven consecutive audited phases:

1. **A joint-edge descriptor already exists.** `designPatternEdge(pattern, phase, depth, clearance)` is a small tagged object consumed by one shared panel builder (`buildSlidingLidBodyPanel`, aliased as `buildFingerPanel`). Every structural finger joint in the app — Finger Box, Sliding Lid body, Drawer Cabinet shell and drawers, Joint Fit Coupon — flows through this single descriptor→builder pipeline. A joint-strategy contract does not need to be invented; it needs to be *named and bounded*.
2. **A decorative-overlay architecture already exists.** Assembly labels and rail guides are score-layer-only paths (`scorePaths`/`labelPaths` in `serializeDesignSvg`'s blue `g#score` group) with fixtures proving cut-path byte-identity when toggled. Decorative joinery (faux dovetails included) is the same pattern with different path content.
3. **A coupon-to-evidence lifecycle already exists.** Joint Fit Coupon → explicit physical winner → Phase 7.3A promotion → Phase 7.1 production settings → Phase 7.2 explicit application back into Designs. New joint styles need new coupon *types*, not a new evidence system.
4. **The single biggest honesty problem is two words.** The tray `jointStyle` values `finger` and `tab-slot` are both the *same* wall-to-base tab construction differing only in tab count (4 vs 2, `trayTabProfile()`, `index.html:1933-1936`). Neither produces interlocking corner fingers, and tray walls have no corner joinery at all. This must be fixed at the label/help layer before any new joint style ships, or "finger" will mean three different things in one app.
5. **The single biggest hidden constraint is rectilinearity.** The entire panel-path pipeline (`designPointsPath`, `index.html:2229-2240`) serializes only H/V segments and returns an empty path for any diagonal. True structural dovetails — angled in-plane pins/tails — cannot pass through the current engine at all. That is not a small extension; it gates the dovetail roadmap far more than laser physics does.

**Final verdict: ARCHITECTURE READY FOR CHALLENGE REVIEW** (§23), with a wall-to-base tab fit coupon as the recommended first implementation (§20) after a terminology cleanup phase.

---

## 2. Current architecture map

Data flow for every template follows one of three pipelines:

```
A. Structured named-layer pipeline (Finger Box, Sliding Lid, Drawer Cabinet, Joint Fit Coupon):
   designDraft → normalizeDesignDraft → build<X>Model → panels[{id,name,path,points,edges,bounds}]
     → layoutDesignPanelRows → serializeDesignSvg  (g#score blue + g#cut red, per-panel <g id="panel-…"><title>)
     → designSvgValidation → result{svg, panels, metrics, validation}

B. Structured anonymous-compat pipeline (Dice Tray, Divider Tray — Phase 2 tray switch):
   designDraft → normalizeDesignDraft → validateLegacyDesign → buildTrayModel
     → semantic components[{id,name,kind,role,geometry,svg}] with layout baked into the model
     → serializeTrayCompatibilitySvg → designSvgDocument  (ONE anonymous red group, no ids/titles)
     → result{svg, panels:"Shape N", metrics, finishedView projection}

C. Legacy string pipeline (QR stand, Hanging sign):
   designDraft → normalizeDesignDraft → validateLegacyDesign → buildLegacyDesignResult
     → legacyDesignSvg → designStandSvg | designHangingSignSvg  (geometry and SVG in one function)
     → designSvgDocument (anonymous red group) → result{svg, panels:"Shape N"}
```

Screen-only layer (all pipelines): `designPreviewMode` (module `let`, reset on init/template change, never persisted) → `designPreviewModeForTemplate` whitelist → `designResultsHtml` → one of five Finished View renderers consuming semantic projections only. Download (`downloadCurrentDesignSvg`) reads `result.svg` and never touches preview state — verified end-to-end in the five Finished View audits in this chain.

Persistence boundary: Designs drafts are session-only and **excluded from backups** (README §storage; `designDraft` is a module variable, absent from `backupObject()`). The only Designs-adjacent persisted data is Library production settings (`fitSettings.fingerJointClearanceMm`, `fitSettings.drawerTotalLateralClearanceMm`, `lightBurn.kerfOffsetMm`, measured thickness) plus promotion evidence snapshots. This is a critical planning fact: **there are no saved Designs records containing `jointStyle`**, so internal enum choices are far less locked-in than they would first appear.

---

## 3. Template-by-template joint inventory

| # | Template (key) | Pipeline | Model | Joint/assembly behavior today | Finished View | Joint-expansion suitability |
|---|---|---|---|---|---|---|
| 1 | Finger-jointed box (`finger-box`) | A | `buildBoxModel` (`:2244`) | True edge-to-edge interlocking fingers on all mating edges; Bottom↔4 walls + wall↔wall corners; signed clearance ≥ −0.10; odd-count pattern via `buildFingerPattern`; optional plain loose lid (pure adjacency, no retention — explicitly warned) | Yes (`finished-view`) | **Primary structural expansion target** — fully structured, shared builder, coupon-backed |
| 2 | Sliding-lid box (`sliding-lid-box`) | A | `buildSlidingLidBoxModel` (`:2786`) | Finger-jointed 5-panel body (4 patterns incl. shortened-Front composite side edges); lid = shoulder/tongue profile riding in channels formed by **glued-on laminated rails** (positional sliding fit, not cut grooves); 4 independent clearances; optional 6-piece fit coupon | Yes (Closed/Open) | Suitable but already the most complex template; extend last |
| 3 | Drawer Cabinet (`drawer-cabinet`) | A | `buildDrawerCabinetModel` | Finger-jointed open-front shell (8 mating pairs) + nested `buildBoxModel` drawers (separate clearance) + **plain edge-glued shelves with no registration feature** (audited known gap) + drawer running clearances | Yes (Front elevation) | Suitable; shelf registration is itself a joint-improvement candidate |
| 4 | Dice Tray (`dice-tray`) | B | `buildTrayModel` (`:1963`) | Wall-to-base tabs only: `trayTabProfile` → `designWallPath` tabs protruding from wall bottom into through-slots in base (`slotWidth = t + fitClearance`); **adjacent walls have no corner connection whatsoever** — butt adjacency, glue implied | Yes | **Primary tab-system expansion target** — cleanest semantic model in the app |
| 5 | Divider Tray (`divider-tray`) | B | same | Same wall-to-base tabs + full-width divider slots in base (`base-divider-slot-NN`); dividers are plain rectangles, `intendedAssembly:'removable'`, `retention:'unspecified'` — honestly modeled | Yes (uncommitted) | Same as Dice Tray |
| 6 | Joint Fit Coupon (`joint-fit-coupon`) | A | `buildJointCouponModel` (`:2284`) | Test artifact: A/B panel pairs sharing one nominal finger pattern across a signed clearance list (−0.10…+0.30, fixed-decimal parser); same `designPatternEdge` machinery as the boxes; promotion entry point | No (deliberate) | **The coupon seed** — generalization target, not a product |
| 7 | QR / sign stand (`qr-stand`) | C | none | One slot-together connection: upright drops into base slot, `slotWidth = thickness + fitClearance`; geometry inlined in `designStandSvg` | No | Not suitable — would need model extraction first; low value |
| 8 | Hanging sign (`hanging-sign`) | C | none | No joints (flat panel + 2 holes) | No | Not applicable |

Classification: **fully structured/semantic-model driven:** 1–6. **Legacy flattened:** 7–8. **Production-serializer split:** A-pipeline templates use named layers; B-pipeline trays use the pinned anonymous contract (Dice `1726/51a55721`, Divider `1965/a55dda6e`). **Not suitable for joint expansion:** 7, 8 (and 6 is a meta-template).

Inputs/validation per template are centralized in `normalizeDesignDraft` (`:2094-2180`) — a genuine asset: one function knows every template's parameter surface, so capability gating has a single enforcement point.

---

## 4. Geometry ownership map

Classification of every load-bearing function (production geometry **PG**, model semantics **MS**, compatibility serialization **CS**, screen rendering **SR**, UI orchestration **UI**, fixture-only **FX**, legacy compatibility **LC**, storage/normalization **SN**):

**Shared structural-joint core (PG)** — the de-facto joint engine:
- `buildFingerPattern` (`:2187`) — odd-segment-count optimizer; sole owner of finger counts/boundaries
- `designPatternEdge` (`:2198`) — the joint-edge descriptor `{pattern, phase, depth, clearance}`
- `designSafePatternEdgePoints` — trimmed edge-point generator (whisker-fix era); **the only live edge-point generator**
- `designEdgePoints` (`:2199`) — **dead code.** Defined, never called (verified by grep this pass). Pre-whisker-fix variant left in place. Should be deleted in a cleanup commit before any joint work builds nearby, so nobody extends the wrong function.
- `designEdgeTerminalOffset` — corner-join inset logic (incl. the composite-edge special case)
- `buildSlidingLidBodyPanel` / alias `buildFingerPanel` (`:2241`) — the shared corner-stitching panel builder; consumed by Finger Box, Sliding Lid, Drawer Cabinet, Joint Fit Coupon
- `buildSlidingLidSidePanel`, `buildSlidingLidPanel`, `buildSlidingLidRail` — Sliding-Lid-specific panel shapes (template-owned PG)
- `designPanelFromPoints` — plain-panel constructor
- Dimension calculators: `designBoxDimensions`, `slidingLidBoxDimensions`, `drawerCabinetDimensions` (template-owned PG)

**Tray tab system (PG, parallel to the finger core, tray-owned):**
- `trayTabProfile` (`:1933`) — tab count/width policy (4 vs 2, width clamped 8–16mm)
- `designTabPositions` (`:1905`) — even spacing formula (also reused by divider slots)
- `designWallPath` (`:1908`) — wall outline with bottom-edge tabs (H/V path string)
- `trayRectComponent` / `trayWallComponent` (`:1953-1959`) — **MS+PG mixed**: semantic component objects that also carry their own `svg` string. This is the one place in the structured pipelines where geometry and serialization co-habit one object; acceptable under the pinned-bytes contract but should not be the pattern for new joint work.
- `buildTrayModel` (`:1963`) — tray semantic model incl. slots, orientation, layout

**Validation/diagnostics (PG-adjacent, shared):** `designPanelGeometryErrors`, `designPathSelfIntersects`, `designPositiveCollinearOverlaps`, `designCollinearSegmentOverlapLength`, `designSegmentsContainedInPolygon`, `designSvgValidation`. All assume orthogonal geometry.

**Layout (PG):** `layoutDesignPanelRows`/`layoutDesignPanels` (A-pipeline sheet packing, gap/overlap enforcement); tray layout is baked into `buildTrayModel` (B-pipeline).

**Serialization (CS):** `serializeDesignSvg` (`:2861`) — named `g#score` (blue 0.1) + `g#cut` (red 0.1) with per-panel groups/titles; `designSvgDocument` (`:1902`) — anonymous single red group; `serializeTrayCompatibilitySvg` (`:2012`) — thin wrapper emitting model component `svg` strings through `designSvgDocument`.

**Score-layer content (PG for score ops):** `slidingRailGuideSegments`, `designAssemblyGlyphs`/`buildAssemblyLabelPaths`/`designAssemblyWordGeometry`/`designAssemblyLabelSpecs` + serialized-label validation. This is the proven decorative/annotation channel.

**Screen rendering (SR):** `trayFinishedViewProjection` (`:2016`), `buildDiceTrayFinishedViewSvg`, `buildDividerTrayFinishedViewSvg`, `buildFingerBoxFinishedViewSvg`, `buildSlidingLidFinishedViewSvg`, `buildDrawerCabinetFrontElevationSvg`. All consume model/result semantics; none parse production SVG (each independently audited).

**UI orchestration (UI):** `normalizeDesignDraft`, `buildDesignResult` dispatcher (`:3020`-region), `designResultsHtml`, `designPreviewModeForTemplate`/`designPreviewSelectorHtml`/`bindDesignPreviewActions`, `downloadCurrentDesignSvg`, `updateDesignDraft`, Phase 7.2 production-application panel (`productionSettingDesignApplicability` `:1641`, `applyDesignProductionValues`).

**Legacy (LC):** `designStandSvg`, `designHangingSignSvg`, `legacyDesignSvg`, `buildLegacyDesignResult`, `validateLegacyDesign` (still first in the tray route for validation parity — intentional).

**Fixture-only (FX):** `trayLivePreviewDownloadFixture`, `diceFinishedViewBrowserFixture`, `dividerFinishedViewBrowserFixture`, `designFixtureHash`, the `run*Fixtures` families.

**Storage/normalization (SN):** `STORAGE_KEY`/`SCHEMA_VERSION`, `persist`, `backupObject`, `loadState`/`decodeStoredState`, `replaceData`/`mergeData`, `normalizeProductionSetting(s)`/`normalizeProductionEvidence` (unknown-field-preserving spreads — the additive-schema mechanism), promotion transaction stack.

**Duplication findings:**
1. `designEdgePoints` is dead (above) — delete.
2. Two tab/finger systems exist (finger core vs tray tab system). This is **acceptable, not accidental** duplication — they model genuinely different joints (edge-to-edge interlock vs wall-to-base tab row) and were deliberately kept separate in the tray-extraction review. The joint contract should *name* both rather than merge them.
3. Two serializers exist by design (named vs pinned-anonymous). Do not unify; §16.
4. `designNumber` (legacy clamp-default parser, used by `designStandSvg`) vs `designRequiredNumber` (validating parser) — legacy-only residue, contained.
5. The two coupon models (`buildJointCouponModel`, `buildSlidingCouponModel` `:2763`) both correctly reuse production builders (`buildFingerPanel`, `buildSlidingLidRail`, `buildSlidingLidPanel`) — the anti-second-engine precedent to preserve.

---

## 5. Terminology findings

**What the tray values actually mean** (verified in source, not docs):

- `finger` → `trayTabProfile(length,'finger')` → `{count: 4, widthMm: clamp(8..16, length/8)}` → four rectangular tabs on the wall's bottom edge, mating with four through-slots in the base (`slotWidth = thickness + fitClearance`).
- `tab-slot` → the same construction with `count: 2`.

**Neither is a finger joint in the woodworking sense**, and neither joins wall corners — tray corners are pure butt adjacency held by the base slots and glue. Meanwhile the *same word* `finger` in Finger Box / Sliding Lid / Drawer Cabinet / Joint Fit Coupon means true alternating interlocking edge fingers via `buildFingerPattern`. The Finished View layer already had to paper over this with the honest-but-awkward labels "Four-tab wall profile" / "Two-tab wall profile" (both tray renderers), and the tray audits repeatedly flagged corner-joinery disclaimers. The UI select for trays, however, still presents `finger` vs `tab-slot` labels.

**Recommendation (three layers, in order of urgency):**
1. **UI labels + help text: change now** (Phase 0). Tray select options become "Four-tab walls (wall-to-base)" and "Two-tab walls (wall-to-base)", help text stating explicitly: tabs join walls to the base only; tray corners are glued butt joints; this is not corner finger joinery. Zero geometry/byte risk (labels live in form HTML, not in the SVG or model).
2. **Internal enum values: keep for now, rename cheaply later if the enum grows.** Because Designs drafts are never persisted and `jointStyle` appears in no backup, no Library record, and no promotion snapshot field (coupon identity hashes cover coupon drafts, which have no `jointStyle`), an internal rename (`finger`→`four-tab`, `tab-slot`→`two-tab`) touches only: `designDefaults`, form options, `normalizeDesignDraft` validation, `trayTabProfile`, `trayModelValidationErrors`, Finished View guards, and the fixture matrix. The pinned production goldens do **not** depend on the value names. Risk is confined to fixture churn. Alias handling (`'finger'→'four-tab'`) in `normalizeDesignDraft` would make even an in-flight session save harmless. Verdict: safe but not urgent — do it as part of Phase 1's capability declarations, where the enum must be touched anyway, not as a standalone commit.
3. **Docs/README: already partially honest** (the tray sections disclaim corner joinery); tighten wording in the same commit as (1).

Do not preserve the misleading labels merely because they shipped; equally, do not rush the internal rename ahead of the capability work that gives it a reason.

---

## 6. Proposed joint-system contract

**Principle: descriptors in, panels out.** The contract formalizes what already works rather than inventing an engine.

### 6.1 Edge-treatment descriptors (structural joints)

Today `buildSlidingLidBodyPanel` consumes an `edges` array where each entry is either `null` (plain) or a `designPatternEdge` descriptor. Formalize this as a **tagged descriptor union**:

```
{ kind:'plain' }                                            (today: null)
{ kind:'finger', pattern, phase, depth, clearance }         (today: designPatternEdge output)
{ kind:'tab-row', count, tabWidthMm, depth, clearance }     (future: formalized tray tabs / new box options)
{ kind:'rabbet-step', ... }                                 (future, if ever — see §9)
```

Rules:
- The **template model builder** owns which edges get which descriptor and all dimension math (as today).
- A **joint strategy** owns exactly two things: producing its descriptor from validated parameters, and generating that edge's point run inside the shared panel builder (as `designSafePatternEdgePoints` does for `finger`). It never owns panels, layout, serialization, or persistence.
- The **shared panel builder** dispatches on `kind`, stitches corners via a per-kind terminal-offset rule (generalizing `designEdgeTerminalOffset`), and runs the existing geometry diagnostics unchanged.
- **Mating is a template concern**: complementary `phase`/slot pairing stays in the model builder, checked by mating fixtures (the existing `matingPairs` pattern).

This is a refactor-by-naming: step one is literally replacing `null` with `{kind:'plain'}` and adding `kind:'finger'` to `designPatternEdge`'s return — behavior-neutral, golden-pinned.

### 6.2 Wall-to-base tabs stay template-owned

The tray tab system (`trayTabProfile`/`designWallPath`) should *not* be forced into the edge-descriptor union immediately — the B-pipeline's pinned anonymous bytes make any refactor there pure risk with no user value. Instead, the capability registry (§7) names it as a distinct joint family (`wall-base-tab`) with its parameters, and the tab **coupon** (§11) reuses `designWallPath`/`designTabPositions` directly. Merge the representations only if a tray ever needs a second wall-joint option.

### 6.3 Decorative overlays

Decorative joinery is **score/engrave path generation only**, flowing through the existing `scorePaths`/`labelPaths` channel of `serializeDesignSvg`. A decorative strategy receives the placed panels and semantic model and returns `{paths, warnings}` exactly as `buildAssemblyLabelPaths` does, with the same containment/clearance validation. It can never touch `panels[].path`. See §8.

### 6.4 Where shared helpers end

Shared: `buildFingerPattern`-class parameter math, the panel builder + diagnostics + layout + serializers, glyph/label machinery, coupon framing. Template-owned: dimension formulas, edge assignments, mating topology, orientation semantics, Finished View renderers. A joint strategy that wants to own more than its descriptor and point-run is over-scoped — that's the line that prevents both a universal geometry engine and per-template forks of the same joint.

---

## 7. Capability-registry recommendation

**Yes — but as one small in-code constant, transient, read-only.** Data-driven-in-code, not persisted, not pluggable:

```
const designJointCapabilities = {
  'finger-box': [
    { id:'finger-edge', label:'Finger-jointed edges', category:'structural', roles:['panel-edge','corner'],
      params:['jointClearance','preferredFingerWidth'], status:'production', coupon:'joint-fit-coupon',
      finishedViewNote:'Interlocking edge fingers', evidence:'coupon+physical' } ],
  'dice-tray': [
    { id:'wall-base-tab-4', label:'Four-tab walls (wall-to-base)', category:'structural', roles:['wall-base'],
      params:['fitClearance'], status:'production', coupon:null /* Phase 2 target */,
      finishedViewNote:'Walls align with base slots; corners are glued butt joints', evidence:'physical' },
    { id:'wall-base-tab-2', … } ],
  …
};
```

Determinations requested by the brief:
- **In code, yes** — a const object next to `designDefaults()`. A separate data file would break the single-file constraint; localStorage persistence would create migration surface for pure code truth.
- **Never persisted.** Capabilities are facts about the running build. Saved records reference joint IDs only if/when a joint parameter ever needs to live in a production setting (§13), and those are plain strings.
- **Enforcement:** `normalizeDesignDraft` validates the draft's joint selection against the registry entry for the template (single choke point, already validates everything else); the form select is *populated from* the registry, so unsupported combinations are unreachable by UI and rejected if crafted.
- **Anti-overengineering rule:** entries are plain data — no functions, no hooks, no registration API, no dynamic loading. If an entry needs behavior, the behavior lives in the template model builder or a named strategy function, and the registry stores only its string ID. The registry earns its existence by powering three consumers (UI select/badges, validation, Finished View wording) from one source of truth; if it ever grows past that, it has become the plugin framework this review recommends against.

Aliases/deprecation metadata (`aliases:['finger']`, `deprecated:false`) belong here too, giving §5's eventual internal rename a home.

---

## 8. Structural versus decorative separation

**Structural** (changes cut paths / assembly): finger edges, wall-base tabs, base slots, divider slots, sliding channels (as laminated-rail assemblies), any future rabbet-like or captured-panel geometry, real dovetails. Owned by model builders; emitted in `g#cut` (A-pipeline) or the anonymous red group (B-pipeline); validated by `designPanelGeometryErrors` + layout checks; changes always re-pin goldens.

**Decorative** (appearance only): faux dovetail engravings, corner accent lines, simulated joinery marks, engraved fastener details, decorative face overlays (which are *cut* parts but structurally inert skins — see below). Owned by overlay generators; emitted in `g#score` (or a future dedicated engrave group, §16); validated by the containment/clearance machinery labels already use.

**Enforcement mechanisms (all with existing precedent):**
1. **Layer rule:** decorative generators can only append to `scorePaths`/`labelPaths`/future `engravePaths`. The fixture pattern "cut group byte-identical with option on/off" (already proven for labels, guides, and the sliding coupon toggle) becomes mandatory for every decorative option.
2. **Naming rule:** decorative option IDs carry the `deco-` prefix and `category:'decorative'` in the registry; UI renders them in a separate "Appearance" group with wording like "Engraved decoration — does not join or strengthen anything."
3. **Finished View rule:** decorative features render only as labeled overlays or text ("decorative engraving"), never drawn with the same visual weight as structural geometry.
4. **Edge case — cut-but-inert parts:** a decorative outer skin is a *cut* panel that is structurally a veneer. It goes in the cut layer (it must be cut), but the registry categorizes it decorative and the assembly text states "glued-on decorative layer; not a joint." The category is about *claimed function*, not which laser operation produces it.
5. **Warning rule:** any decorative option whose appearance mimics a joint (faux dovetail above all) must carry an always-on UI note and Finished View label: "Decorative only — this corner is joined by [actual joint]."

---

## 9. Dovetail feasibility assessment (20W diode, ~3mm plywood)

Candid assessments, in increasing order of ambition:

| Concept | Feasibility | Notes |
|---|---|---|
| **Faux engraved dovetails** | **Feasible now; primarily decorative** | Score/engrave-layer line art of pin/tail outlines near corners. Identical machinery to assembly labels (vector glyph paths, containment validation). Zero structural risk; honest if labeled per §8. The obvious Phase 5 candidate. |
| **Applied dovetail overlays** (thin cut appliqué glued to a face) | **Feasible with careful testing; decorative** | Cut contrasting-veneer or thin-ply shapes; glue to visible faces. Costs material and assembly labor; alignment is manual. Kerf makes sharp interior corners slightly rounded — acceptable for decoration. |
| **Layered plywood dovetail appearance** (stacked laminations with alternating profiles) | **Feasible with testing; decorative-to-marginal-structural; likely uneconomical** | Building wall thickness from 2–3 laminated layers whose end profiles alternate to *look* like through-dovetails from the side. Multiplies cut area, glue-up time, and tolerance stack (each layer's kerf and char). Visually striking; poor effort-to-value for production items. |
| **Through-dovetail flat cut geometry** (angled pins/tails cut in-plane — "box dovetails") | **Poor fit for the current engine; physically feasible-with-testing; defer** | Geometrically real: laser-cut flat dovetails do resist pull-apart in one axis. But: (a) **the entire path pipeline is rectilinear** — `designPointsPath` emits only `H`/`V` and returns `''` for any diagonal, and every diagnostic (`designPathSelfIntersects`, overlap detection, terminal offsets) assumes orthogonal runs. Supporting angled fingers means adding `L`-segment support across the serializer, validators, and corner-stitching logic — a versioned second path pipeline, not a tweak. (b) In 3mm ply, pins narrow enough to look like dovetails approach the `requiredWeb` floor and cross veneer grain — fragile, chip-prone at the char line. (c) Diode kerf taper and char make angled fit *more* clearance-sensitive than straight fingers, for a joint whose mechanical advantage over well-fitted straight fingers in glued ply is marginal. Verdict: do not build until a versioned non-orthogonal pipeline is independently justified; even then, expect it to be aesthetic. |
| **Half-blind appearance** | **Unsafe/misleading to offer as real; decorative-only via engraving** | Real half-blind joints require partial-depth material removal. A 20W diode cannot pocket 3mm ply to controlled depth reliably (char, veneer burn-through variance). Only honest form: engraved *suggestion* of half-blind tails, clearly decorative. |
| **True structural dovetails in sheet goods** | **Poor fit** | Everything from the flat-cut row, plus the claim of structural superiority that thin ply cannot honor (glue line dominates strength; short-grain pins fail first). Offering it would imply strength the material doesn't deliver. |
| **Hidden dovetails** | **Not feasible** | Requires internal material removal invisible from both faces — not a through-cutting operation. |
| **Sliding dovetail** | **Not feasible as cut; partially mimicable by lamination** | A true angled groove is impossible; a laminated three-layer wall *can* form a trapezoidal channel (the rail/channel lamination pattern generalized), but that is a new laminated-assembly concept, not a dovetail — name it honestly if ever pursued. |

Cross-cutting physics honestly stated: kerf 0.1–0.25mm with taper; char weakens and discolors mating faces (the app already warns to sand sliding surfaces); minimum web `max(0.5, t/4)` already enforced; plywood veneer chips at sharp external corners; every fit claim requires same-sheet coupons — all of which the existing warning/coupon culture already handles correctly. The architecture should channel "dovetail desire" into the faux-engraved option plus the cleaner-joint alternatives below, not into the real thing.

---

## 10. Cleaner and hidden joint options

Ranked by fit for this app and equipment:

| Option | Templates | Purpose / appearance | Complexity | Verdict |
|---|---|---|---|---|
| **Decorative outer skins / doubled walls** | Finger Box, trays | Full-coverage glued outer panel hides all finger/tab ends → genuinely "hidden joints" from outside | Cut duplicate outline panels (no joint cutouts); 2× wall material; alignment during glue-up; kerf-match outer size slightly proud then sand, or slightly shy | **Strongest hidden-joint candidate.** Structurally inert (decorative category), massive visual payoff, trivial geometry (plain outlines already exist). Needs a skin-alignment note + coupon for overhang preference |
| **Layered base (slot-hiding bottom skin)** | Dice/Divider Tray | Second unslotted base layer glued beneath the slotted base hides through-slot ends from below | One extra rect per tray; material cost; must warn about doubled base thickness in wall-tab depth (tab depth = t, unchanged; only appearance) | **Feasible now**; smallest possible "cleaner" win on the newest templates |
| **Internal corner blocks** | Finger Box, trays, cabinet | Square glue blocks inside corners; invisible outside; adds rigidity and glue area; also a shelf-registration aid for the Drawer Cabinet's known gap | Trivial rectangles; assembly text; no joint geometry change | **Feasible now**; also directly answers the audited shelf-registration weakness |
| **Interrupted/alternating/keyed tabs** | Trays | Vary tab rhythm (asymmetric key prevents backwards assembly — a real usability win) | Extends `designTabPositions`/`trayTabProfile` with a pattern parameter; B-pipeline byte changes → new goldens; keyed = one tab offset | **Good Phase 6 candidate** — "configurable tab-and-slot" from the brief, with the keyed variant as the honest headline feature |
| **Captured bottom via wall slots** | Future box variant | Bottom tabs into through-slots in walls (tab ends visible as small rectangles on outside faces instead of full finger rhythm) | New edge treatment (`tab-row` on box walls) through the §6 contract; A-pipeline; couponable | Medium-term; the first real second structural joint style for the box family |
| **Slot covers** | Trays | Small glued caps over slot ends | Fiddly tiny parts, easily lost/burned | Low value vs layered base; skip |
| **Face frames** | Drawer Cabinet | Applied frame hides shell edges | Real appearance win but new part set + reveal math | Later; after skins prove the decorative-part workflow |
| **Sacrificial alignment features / glue-only aids** | All glue-ups | E.g., score-line glue boundaries for rail placement — **already shipped** (rail guides) | — | Extend the existing pattern (e.g., shelf-position score guides in the cabinet — direct M1 remediation from the cabinet audit) |
| **Offset/underside slots** | Trays | Blind slots impossible (through-cut); offsetting changes nothing visible | — | Not viable; superseded by layered base |

Every promising option above is either decorative-category (skins, layered base) or a parameterization of an existing tab system (keyed/interrupted) — none requires the dovetail-class engine change. That is the practical "cleaner joints" story for this machine.

---

## 11. Test-coupon architecture

**Recommendation: one coupon template, selectable coupon types, mandatory production-builder reuse.**

- Keep `joint-fit-coupon` as the single user-facing coupon template. Add a `couponType` select whose options come from the capability registry (`coupon:` references). Current behavior becomes `couponType:'finger-edge'` (default, unchanged bytes for the default draft → golden-pinned).
- **First new type: `wall-base-tab`** — a strip of base material with a row of slots at stepped clearances + one tab wall segment per clearance (or one wall + multiple slot strips), reusing `designWallPath`/`designTabPositions` verbatim. Tests the tray `fitClearance`, which today has *no* test path at all (the existing coupon only tests finger edges).
- Future types map 1:1 to registry joint families: captured-panel tabs, skin-overhang preference, divider insertion, sliding-channel fit (already exists as the Sliding Lid's own `slidingFitCoupon` — leave it embedded; do not migrate it, it is template-specific by design and already reuses `buildSlidingLidRail`/`buildSlidingLidPanel`).
- **The anti-second-engine rule, made testable:** every coupon type must generate its mating geometry by calling the same production builder functions the real templates call (`buildFingerPanel`, `designWallPath`, …). Enforce with a fixture per coupon type asserting shared-function output identity (the existing coupon suite already does this for finger edges via shared `pattern` object identity — extend the pattern).
- **Parameters:** coupon types consume the same parameter names as their production joints (`fitClearance`, `jointClearance`, tab width) so a coupon result maps 1:1 onto a Design field and a production-setting field without translation tables.
- **Evidence:** results flow through the existing explicit promotion (Phase 7.3A) — the coupon promotion candidate gains `couponType` in its canonical identity hash (additive; new sessions only). Promotion writes production settings; nothing writes back to Designs; old designs never silently change (Designs aren't persisted, and Phase 7.2 application remains explicit-checkbox-only). This already satisfies the brief's "promote without silently changing old designs" requirement by construction.

---

## 12. Physical-evidence lifecycle

The existing stack already implements most of the requested lifecycle; the recommendation is to *name* the stages and extend fields only where a real gap exists:

- **theoretical/generated** → a Designs draft + generated SVG (session-only; correctly unrecorded)
- **cut / fit observed** → Joint Fit Coupon physical winner (explicit selection; no auto-detection — correct)
- **tested** → promoted production setting with `physicalChecks` (`piecesFit` for coupons, `cutCompleted` for cut tests), machine identity, measured thickness, scope (`sheet`/`batch`/`material-general`), batch label, date, notes
- **verified** → explicit applicable check + date + notes (source/operation-specific rules, audited)
- **approved starting point / proven** → `preferred` flag within machine+operation scope; superseded history preserved

**Record (already supported):** machine key/label, material profile, measured + nominal thickness, batch, speed/power/passes/air, focus, signed kerf reference, clearance values, result text, notes, snapshot evidence. **Worth adding when joint work ships (additive, optional):** `fitSettings.tabFitClearanceMm` (wall-base tabs) and — resolving a collapse the Phase 7 architecture review already flagged — `fitSettings.cabinetJointClearanceMm` / `drawerJointClearanceMm` so the Drawer Cabinet's two clearances stop sharing `fingerJointClearanceMm` (today Phase 7.2 maps that one stored field onto *three* draft fields: `jointClearance`, `cabinetJointClearance`, `drawerJointClearance` — verified at `productionSettingDesignApplicability`, `index.html:1663-1673`). **Do not record:** photos-in-record beyond the existing Project photo mechanism (reference Projects as evidence instead — already supported), glue brand/sanding as structured fields (notes suffice), or any auto-captured result.

**Interaction rules (all current, all correct — preserve):** Test Grids → Material Tests → promotion; coupons → promotion; Projects → evidence-only; no automatic overwrite of preferred settings (explicit demotion prompt); no auto-promotion from any save/download.

---

## 13. Storage and backward-compatibility analysis

**Finding: nothing about joint styles needs to persist today.** Designs drafts are session-only and excluded from backups; the capability registry is code; coupon evidence flows through the existing production-setting schema whose normalizers (`normalizeProductionSetting`, spread-then-overwrite) preserve unknown fields — the proven additive mechanism (Phase 7.1 fixtures cover unknown-field round-trips at every nesting level).

Per-proposal justification:
- `couponType` in promotion evidence snapshots: **transient-derived is impossible** (evidence must describe what was physically cut after the session is gone) → persist as a plain string inside the already-snapshotted evidence blob. No schema change — evidence snapshots are free-form and unknown-field-safe.
- New `fitSettings.*Mm` fields (§12): **cannot be derived** (they are measured user decisions) → additive optional numbers via `strictOptionalNumber`, exactly like the two existing fit fields. No `SCHEMA_VERSION` bump (matches the Phase 7 rule: bump only for reinterpretation/moves, never addition).
- Joint capability data, joint IDs on drafts, geometry models, production SVG: **never persist.** SVG-as-source-of-truth is explicitly rejected — the tray Phase 2 switch just spent three audited phases establishing models as truth with SVG as pinned projection.
- Existing saved data containing `jointStyle`: **does not exist** (no Designs persistence). Existing production settings containing `fingerJointClearanceMm`: untouched; if the cabinet-specific fields are added, old records keep working through the current mapping until a record carries the newer field (Phase 7.2's mapping table gains two rows; old field remains the fallback — additive, no rewrite).
- Destructive migration, alias rewriting of stored values, casual rename of stored keys: none required, none recommended.

---

## 14. UI and wording recommendations

- **Joint select per template**, populated from the capability registry; hidden entirely for templates with one fixed joint (Finger Box today) until a second option exists — no speculative dropdowns.
- **Grouped options:** "Construction" (structural) vs "Appearance" (decorative), with decorative entries carrying the always-on inertness note (§8).
- **Badges from registry `status`:** `production` (no badge), `experimental` ("Experimental — cut a coupon first"), plus a per-choice evidence line when a matching production setting exists ("Tested 2026-07-15 on Basswood 3.02mm, L8 20W" — data already available via the Phase 7.2 panel; reuse its lookup, don't duplicate it).
- **Honest replacements (from §5):** "Four-tab walls (wall-to-base)" / "Two-tab walls (wall-to-base)"; anywhere "finger" appears for trays, it goes. Keep "Finger-jointed edges" only where `buildFingerPattern` geometry is actually produced.
- **Coupon shortcut:** a "Cut a fit coupon for this joint" link on each structural joint's help row, pre-selecting the matching `couponType` (session-only draft change; precedent: Pricing → Project draft flow).
- **Theoretical-value warnings:** already strong (negative-clearance warning, scrap-test warnings); extend the same voice to new options rather than inventing a new warning style. Language stays LightBurn-user-level: "tabs", "slots", "clearance", "snug/loose" — no "kinematic", no "tolerance stack".
- **Unsupported combinations:** unreachable via registry-driven selects; rejected in `normalizeDesignDraft` if crafted (fixture-enforced).

---

## 15. Finished View integration

The five shipped renderers define the contract; codify it for joints:

- Renderers consume **semantic projections only** (`trayFinishedViewProjection`, `metrics.slidingFinishedView`, `frontElevation`, model dimensions) — never `result.svg`, never production paths, never layout x/y (all five independently audited to this rule).
- **Per-joint representation policy** lives in the registry's `finishedViewNote`: structural joints are drawn *simplified* (solid wall bands, no tooth rhythm — the current, correct choice: the tray FVs draw plain bands and state the relationship in text; the sliding FV draws rails as bands with data attributes) and *described* in text with model-sourced counts/IDs. Do **not** start drawing accurate finger/tab rhythms in FVs — that recreates production geometry in the screen layer, the exact duplication §4 warns against. Draw accurately only what orientation requires; label the rest.
- Decorative features: rendered as clearly-labeled overlays or omitted with a text line ("Decorative dovetail engraving on Front — not structural"). Never rendered indistinguishably from structure.
- Never imply verified fit — the existing disclaimer sentence pattern ("does not prove fit, strength, kerf accuracy…") becomes a required element for any new joint's FV text, sourced from the registry note so wording stays consistent.
- Screen-only/download isolation: keep the established structural guarantees (renderer unreferenced in the serialize/download chain; `finishedView` attached after `svg` is computed) and the established fixture trio (deterministic render, absent-from-production-SVG, download identity) for every new representation.

---

## 16. Production SVG and LightBurn implications

Two contracts exist; keep both, and version forward instead of retrofitting:

- **A-pipeline (named layers):** `g#score` (blue, guides+labels) before `g#cut` (red), per-panel groups with titles, 0.1mm strokes. LightBurn maps colors to operations; score-before-cut ordering is deliberate (score ops first). **All new joint styles and coupon types on A-pipeline templates keep this contract.**
- **B-pipeline (anonymous compat):** single anonymous red group, no IDs/titles, byte-pinned. **Structural tray changes (keyed tabs, tab-count config) must change these bytes — that is expected and handled by re-pinning goldens in the same commit**, exactly as the Phase 2 switch established. What must *not* happen casually: switching trays to the named-layer serializer. If tray options ever need a score layer (e.g., skin-alignment guides), that is the moment to introduce a **versioned tray output contract** (new option ⇒ named-layer serializer for that option only, default drafts still byte-identical) — mirror of the "labels/guides off ⇒ legacy bytes" pattern the Sliding Lid already proved.
- **Decorative engraving layer:** faux-dovetail *line* work fits the existing blue score layer. If filled/shaded engraving ever ships, add a third named group (`g#engrave`, distinct color) — additive, options-gated, never emitted when the option is off (fixture: default bytes unchanged). Do not overload blue score with fill semantics.
- **No metadata, no named LightBurn layers-by-name, no embedded settings** in SVG — kerf and cut settings remain reference data in Library records (the audited kerf boundary: never baked into geometry). Multi-stage jobs (engrave→score→cut) are expressed purely by group order and color, as today.
- Filenames/MIME: unchanged pattern `l8-<template>-<date>.svg`, `image/svg+xml;charset=utf-8` — coupon types stay under the `joint-fit-coupon` filename.

---

## 17. Fixture strategy

Per new joint strategy (structural), required minimum — all patterns already exist in the suite and are cited as precedent:

1. Deterministic model + byte-stable default SVG golden (legacy-baselines pattern)
2. Parametric matrix over thickness {2, 3, 4.5}, clearance {−0.05, 0, +0.15, max}, size extremes — pinned length/hash/element-signature per cell (tray 23-case matrix pattern)
3. Minimum-feature enforcement (web/tab floors) rejecting with errors, not clamping silently
4. Mating complementarity traced from raw edge descriptors, not from a fixture that mirrors the formula (matingPairs pattern; the Drawer Cabinet audit's hand-derivation standard)
5. Symmetry where claimed (mirrored panels, alternating patterns)
6. Assembly-direction semantics (keyed tabs: assert the key breaks 180° symmetry)
7. Serializer isolation: option-off bytes identical to pre-change goldens (labels/guides/coupon-toggle pattern)
8. Finished View isolation trio (§15) + storage isolation (localStorage/backup snapshots around generation — existing pattern)
9. Filename/MIME + browser download-identity fixture (the `*FinishedViewBrowserFixture` DOM-click pattern)
10. Coupon↔production parameter agreement: coupon geometry produced by the same builder call, asserted by shared-object identity or output equality
11. Unsupported-template blocking: crafted draft with a joint the registry doesn't allow → `normalizeDesignDraft` error

**Circularity discipline** (this chain's standing vocabulary): goldens and hand-derivable literals are independent; "call it twice and compare" determinism checks are partially circular and acceptable only alongside independent oracles; assertions that re-evaluate the production formula are circular and must be replaced by literal expected values (the `literalPositiveA` coupon-fixture precedent). **Avoiding dual engines:** never keep an old geometry implementation alive for comparison — pin its output as data and delete the code (the Phase 2 tray transition is the canonical example: shadow comparison replaced by pinned oracles, `designTraySvg` deleted). The same rule applies to any future edge-treatment refactor: pin, refactor, assert against pins, never ship both.

---

## 18. Risk register

| # | Risk | Sev | Likelihood | Templates | Detection | Mitigation | Blocks? |
|---|---|---|---|---|---|---|---|
| R1 | Geometry duplication (coupon or FV re-implements joint math) | High | Medium | all | Shared-builder fixtures (§17.10); audit greps for parallel formulas | Contract §6; mandatory builder reuse | Yes — reject at review |
| R2 | Misleading terminology ("finger" trays; future "dovetail" claims) | High (trust) | Certain if unaddressed | trays; future deco | §5 findings; wording audits | Phase 0 relabel; §8 naming rules | Yes for new joints; No for existing |
| R3 | Unnecessary storage migration | High | Low | n/a | Schema review per phase | §13: additive-only; nothing persists now | Yes if proposed |
| R4 | Invalid old saved records | Low | Very low | n/a | — | No Designs persistence; fit fields additive | No |
| R5 | Overgeneralized joint engine / plugin framework | High | Medium (drift) | all | Registry stays data-only (§7); contract line (§6.4) | Smallest-architecture rule; challenge review Q1 | Yes — reject at review |
| R6 | Renderer/production coupling (FV draws production geometry) | Medium | Low | FVs | Isolation fixtures; §15 "simplify, don't replicate" | Established audit trio | Yes |
| R7 | Insufficient physical testing before "production" status | High (physical) | Medium | all structural | Registry `status` + evidence lifecycle | Coupons first; `experimental` badge until tested | Yes for status promotion only |
| R8 | Material variation invalidating stored fits | Medium | High (inherent) | all | Scope warnings (sheet/batch) already shipped | Keep measured-thickness/scope discipline; never imply nominal fit | No (managed) |
| R9 | Minimum-feature failures (tiny tabs/pins) | Medium | Medium | trays, future dovetail-ish | `requiredWeb`-class floors + matrix fixtures | Enforce floors in validation, not clamps | Yes per-joint |
| R10 | Fragile plywood fingers/pins (short grain, char) | Medium | High for dovetail-class | box family | Physical coupons | §9: defer real dovetails; prefer straight fingers + decoration | Yes for dovetails |
| R11 | Slot overburn/taper making clearances lie | Medium | Medium | trays, boxes | Coupon results diverge from theory | Coupon-first culture; kerf stays a LightBurn concern | No |
| R12 | LightBurn layer/contract drift | High | Low | trays esp. | Pinned goldens; §16 versioning rule | Never retrofit B-pipeline; option-gated contracts | Yes |
| R13 | Fixture circularity hiding regressions | Medium | Medium | suite | §17 classification discipline (already standing practice) | Literal oracles; independent derivations in audits | No |
| R14 | UI complexity creep (joint pickers everywhere) | Medium | Medium | all | UX review | Hide single-option selects; registry-driven minimalism | No |
| R15 | Single-file performance (10.5k lines growing) | Low-Med | Medium long-term | app | Startup/self-test timing | Fixture code is the main growth driver; keep matrices data-compact; acceptable headroom today | No |
| R16 | Long-term maintainability of parallel tab/finger systems | Low | Low | trays vs boxes | §4 map | Deliberate separation documented in registry; merge only on demonstrated need | No |

---

## 19. Phased roadmap

Each phase is one commit-sized, independently auditable unit. "Bytes change" refers to default-draft production SVG.

**Phase 0 — Terminology and capability inventory.** Purpose: honesty floor. Scope: tray UI labels/help ("Four-tab walls (wall-to-base)" etc.), README wording, delete dead `designEdgePoints`, one wording fixture. Files: `index.html`, `README.md`. Storage: no. Bytes: **no** (labels/help only; verify goldens unchanged). Physical testing: none. Audit: focused, light. Rollback: single commit. Suggested message: `Clarify tray joint terminology and remove dead edge helper`.

**Phase 1 — Joint capability declarations.** Purpose: single source of truth. Scope: `designJointCapabilities` const; tray/box selects populated from it; `normalizeDesignDraft` cross-check; internal tray enum rename with aliases (optional but natural here); ~10 fixtures (registry-UI-validation agreement, unsupported-combination blocking, alias acceptance). Storage: no. Bytes: **no**. Audit: focused. Message: `Add joint capability declarations`.

**Phase 2 — Coupon architecture: couponType + wall-to-base tab coupon.** Purpose: give trays a fit-test path; prove the coupon contract. Scope: `couponType` on `joint-fit-coupon` (default `finger-edge`, bytes pinned-unchanged); new `wall-base-tab` type reusing `designWallPath`; promotion identity gains `couponType`; new goldens + matrix + builder-reuse fixtures. Storage: evidence snapshots only (free-form, additive). Bytes: default unchanged; new type = new pinned output. **Physical testing: required** (cut tab coupons on 3mm stock). Audit: full focused + physical follow-up note. Message: `Add wall-to-base tab fit coupon`.

**Phase 3 — First structural enhancement (one template).** Purpose: exercise the edge-descriptor contract on real product value. Recommended content: **keyed tab option for trays** (one asymmetric tab prevents backwards wall assembly) *or* layered slot-hiding base — chosen after Phase 2 coupon results. Scope: one template, one option, off-by-default with legacy bytes pinned when off. Storage: no. Bytes: only with option on. Physical: required. Audit: full. Message: `Add keyed tray wall tabs` (or equivalent).

**Phase 4 — Finished View representation for new options.** Scope: FV wording/overlays for Phases 2–3 options via registry notes; isolation trio fixtures. Bytes: no. Audit: focused. Message: `Represent tray tab options in Finished Views`.

**Phase 5 — Decorative faux dovetail (score-layer).** Scope: `deco-dovetail-engrave` option on Finger Box; score-path generator reusing label machinery; §8 warnings; cut-identity fixtures. Storage: no. Bytes: cut group unchanged; score group additive when on. Physical: one scrap engrave pass (cosmetic check). Audit: focused. Message: `Add decorative engraved dovetail option`.

**Phase 6 — Second structural joint style.** Scope: formalize `{kind:'plain'|'finger'}` descriptors (behavior-neutral refactor, all goldens pinned) then add `tab-row` captured-bottom variant for the box family, coupon-backed. Storage: possible additive `fitSettings.tabFitClearanceMm`. Bytes: refactor none; new option gated. Physical: required. Audit: full + challenge review of the refactor. Two commits (refactor, then feature). Messages: `Formalize edge treatment descriptors`; `Add captured-bottom box option`.

**Phase 7 — Broader adoption.** Cabinet-specific fit fields (`cabinetJointClearanceMm`/`drawerJointClearanceMm`, resolving the Phase 7.2 collapse), corner blocks/shelf guides, skins. Each its own audited commit; no omnibus.

---

## 20. First implementation recommendation

Candidates compared:

| Candidate | User value | Risk | Teaches architecture? | Physical coupon? |
|---|---|---|---|---|
| (a) Terminology/help cleanup only | Honesty, no capability | ~zero | No | No |
| (b) Capability declarations only | None visible | Low | Registry only | No |
| (c) **Wall-to-base tab fit coupon** | **High — trays currently have no fit-test path at all; `fitClearance` is pure guesswork on the two newest production templates** | Low (new coupon type; default bytes pinned; no storage change) | **Yes — coupon contract, registry reference, builder-reuse rule, promotion extension in one small feature** | **Yes — directly cuttable on 3mm ply** |
| (d) Concealed/keyed tray tabs | Medium | Medium (production byte changes before any tray fit evidence exists) | Partially | Needs (c) first |
| (e) Layered decorative dovetail sample | Low-medium, cosmetic | Low | Decorative channel only | Cosmetic only |
| (f) Finger Box joint-capability refactor | None visible | Medium (touches the most-shared code first) | Yes, but with maximal blast radius | No |

**Firm recommendation: (c) — the wall-to-base tab fit coupon** (Phase 2), with Phase 0+1 as its two small prerequisite commits. It is the only candidate that simultaneously delivers something Joe can cut and measure this week, closes a real gap (tray clearance has no evidence path; the finger coupon doesn't cover it), exercises every architectural seam this review proposes (registry reference, coupon contract, builder reuse, promotion identity) at minimum blast radius, changes no stored schema and no existing production byte, and generates the physical data needed before any Phase 3 structural tray change is worth making. Candidates (d)–(f) all sequence *after* it naturally; (a)/(b) are its warm-up commits, not alternatives.

---

## 21. Grok-review recommendation

**Yes — one challenge review of this architecture before Codex implements Phase 0/1**, consistent with this project's two-reviewer discipline and genuinely valuable here because the main failure modes are judgment calls, not bugs. Grok should be asked to attack, specifically:

1. Is the edge-descriptor contract (§6) overengineered for a codebase that may only ever add two more joint kinds — should Phase 6's refactor be dropped entirely until a second box joint is actually approved?
2. Does the capability registry (§7) create hidden duplication with `normalizeDesignDraft`'s existing validation, and who wins when they disagree?
3. Is the claim that **no storage change is needed** (§13) actually airtight — especially coupon `couponType` inside evidence snapshots and the deferred cabinet fit fields?
4. Is the structural/decorative separation (§8) enforceable by fixtures alone, or does it need a serializer-level guard?
5. Is Phase 2 (tab coupon) truly smaller than Phase 0+1+2 combined suggests — could the coupon ship *without* the registry?
6. Can the coupon share `designWallPath` without eventually forcing tray-model internals into public contract status (second-engine risk from the opposite direction)?
7. Are keyed tabs (Phase 3) and captured bottoms (Phase 6) realistic on charred 3mm ply at the stated web floors, or do the minimum-feature numbers need physical revision first?
8. Is the rectilinear-pipeline finding (§9) correct that flat dovetails are blocked by `designPointsPath` — or is there a cheaper path this review missed?

---

## 22. Open questions

1. Should the tray internal enum rename land in Phase 1 (natural home) or be dropped entirely, keeping `finger`/`tab-slot` as permanent internal legacy IDs behind honest labels? (Cost is fixture churn only; benefit is grep-ability.)
2. Does Joe want the Sliding Lid's embedded `slidingFitCoupon` eventually surfaced in the coupon registry for discoverability, even though it stays template-embedded?
3. When skins/layered bases ship (decorative cut parts), should material-usage estimates surface in the Designs metrics panel (they double specific panel areas), and should Pricing get a hook?
4. Is a third `g#engrave` layer (§16) ever needed, or is line-score decoration sufficient for the faux-dovetail ambition? (Defers a color-mapping decision in LightBurn workflows.)
5. Phase 7's cabinet fit-field split: should Phase 7.2's mapping prefer the new specific fields with the old shared field as fallback, or require re-promotion? (Leans fallback; needs its own mini-review.)

---

## 23. Final verdict

```text
ARCHITECTURE READY FOR CHALLENGE REVIEW
```

The system's existing seams — the edge descriptor, the score-layer overlay channel, the coupon→promotion→application lifecycle, the pinned-golden fixture culture, and the session-only Designs boundary — support incremental joint-style growth without a rewrite, without persistent-schema risk, and without a universal geometry engine. The two mandatory correctives before any new joint ships are the tray terminology fix and the deletion of the dead edge helper; the one hard technical gate to respect is the rectilinear path pipeline, which rules out real dovetails until independently revisited. Recommended sequence: Grok challenge review of this document (§21), then Phase 0 → Phase 1 → Phase 2 (wall-to-base tab fit coupon) as the first implementation, each as its own audited commit.

---

*Architecture review performed read-only at HEAD `5233609` with the uncommitted Divider Tray Finished View working tree recorded above. No application files were modified, staged, committed, or pushed.*
