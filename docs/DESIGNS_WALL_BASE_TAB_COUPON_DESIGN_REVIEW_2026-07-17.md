# Wall-to-Base Tab Clearance Coupon — Focused Design Review

**Date:** 2026-07-17
**Repository:** `C:\Genmitsu L8 Tracker`
**Committed baseline:** `541cda5` — *Clarify tray wall-to-base tab terminology*
**Working-tree state:** clean — `git status -sb` shows no modified tracked files (only pre-existing untracked docs/`LightBurn Projects/`/`debug.log`). `git diff --check` clean (nothing to check). This review is planning only; the source below was read at `541cda5` exactly as committed.
**Primary planning documents:** the joint-system architecture review, its adversarial challenge review, and the Phase A terminology-cleanup implementation + focused audit — all read in full before this design. This document implements **Phase B** of the challenge review's revised roadmap (§11 of that document): *"Wall-to-base tab fit coupon (geometry only)."*
**Review type:** read-only design and planning. No application file was modified.
**Method:** every function named in the task's inspection list was read directly in the current `index.html` (10,4xx lines) at this commit; no claim below is taken from either prior report without a corresponding line-level source citation.

---

## Executive recommendation

Add **one new session-only field**, `couponType`, to the existing `joint-fit-coupon` template, with two values: `finger-edge` (default, byte-identical to today) and `wall-base-tab` (new). The wall-to-base type **does not reuse `buildJointCouponModel`/`layoutDesignPanelRows`/`serializeDesignSvg`** — it cannot, without reimplementing tab geometry, because those functions assume the `buildFingerPanel` panel shape (`{path, points, bounds}`) and the tray tab system produces a structurally different component shape (`{geometry, svg}`). Instead it is built the same way `buildTrayModel()` builds a wall and its base slots: by calling `trayTabProfile()`, `designTabPositions()`, `trayWallComponent()`, and `trayRectComponent()` directly, laid out with the same margin/gap hand-placement style `buildTrayModel()` already uses, and serialized through the **same anonymous single-red-group contract** trays already use (`designSvgDocument()`). This makes the new coupon type structurally a **third tray-family output** (wall + slot pieces, no labels, no score layer) rather than a second Joint Fit Coupon geometry engine.

One genuine formula is currently un-named and inlined twice-in-waiting: the tray slot-width rule `t + fitClearance`. This review recommends extracting it into a two-line helper, `traySlotWidthMm(thicknessMm, clearanceMm)`, called from both `buildTrayModel()` (pure refactor, byte-identical, re-pin goldens to prove it) and the new coupon. This is the only change to existing tray code this design requires.

Promotion, evidence identity, and Production Settings integration are **explicitly excluded** from this implementation, consistent with the architecture-challenge review's finding that the winner field is hard-wired to `fitSettings.fingerJointClearanceMm` and would be dishonest for a tab clearance. The "Promote physical winner" button is gated off for the new coupon type at its single existing render condition — a one-line change, not new promotion code.

**Verdict: DESIGN READY FOR IMPLEMENTATION.**

---

## 1. Source findings

Findings are grouped by the function each answers; every line number was read directly at `541cda5`, not carried over from a prior report.

**`designDefaults()` (`:415`)** — current coupon defaults: `jointCouponEdgeLength:'45', jointCouponBodyDepth:'22', jointCouponPreferredFingerWidth:'9', jointCouponClearances:'0.06, 0.04, 0.02, 0.00, -0.02, -0.04', jointCouponLabels:true`. No `couponType` exists. `jointStyle:'finger'` is the tray-template default and is unrelated to the coupon (separate field, separate template branch).

**Joint Fit Coupon form (`renderDesigns()`, `:1806-1897`)** — `jointCouponFields` (`:1847-1852`) is inserted at `${jointCoupon ? jointCouponFields : ...}` (`:1885`), directly after the shared `materialThickness` field. The tray `jointStyle` selector (`:1883`) is rendered only when `tray` is true and explicitly suppressed for `jointCoupon` (`box || sliding || cabinet || jointCoupon ? '' : tray ? … : '<div></div>'`) — the coupon currently has **no access to the tray tab-profile selector at all**, confirming a new UI surface is needed if the coupon should offer a profile choice.

**`parseJointCouponClearances(raw, errors)` (`:2056-2093`)** — token split on `,`/newline, regex `/^[+-]?\d+\.\d{1,2}$/`, range `hundredths` `-10..30` (i.e. **-0.10 to +0.30 mm inclusive**), duplicate rejection after normalizing `-0.00`→`0.00`, count `3..8`. Fully generic — nothing in it references "finger" semantics. **Directly reusable for wall-to-base candidates with no changes.**

**`buildJointCouponModel(parameters)` (`:2262-2315`)** — validates `edgeLength>0`, `bodyDepth>=minimumBodyDepth=max(4t,16)`, `preferredFingerWidth>0`, `3<=clearances.length<=8`; builds one `buildFingerPattern` shared by all pairs; per clearance builds two `buildFingerPanel` calls with `edges:[designPatternEdge(pattern,phase,t,clearance.value),plain,plain,plain]` (**one patterned edge, three plain — width=edgeLength, height=bodyDepth**). This is 100% finger-pattern-specific; **nothing here is reusable for tab/slot geometry** because `designPatternEdge`/`buildFingerPanel` only know how to draw interlocking fingers, not a wall's tab-row-on-one-edge shape.

**`buildJointCouponDesignResult(normalized)` (`:3024-3065`)** — lays out panels via `layoutDesignPanelRows(model.panels, rows, 10, 10)` (4-per-row), serializes via `serializeDesignSvg(partial)`, optionally adds clearance labels via `buildAssemblyLabelPaths`. **`layoutDesignPanelRows` requires each panel to carry `.width`, `.height`, `.points`, and (for `serializeDesignSvg`) `.path`** — the exact shape `buildFingerPanel`/`designPanelFromPoints` produce. Tray components do not have this shape (see below). This confirms the A-pipeline cannot receive tray components without an adapter that would itself re-derive tab geometry into points — precisely the "second engine" risk the task warns against.

**`serializeDesignSvg(result)` (`:2861-2867`, read again this pass)** — named `g#score` (blue, only if `scorePaths`/`labelPaths` present) then `g#cut` (red), each cut panel emitted as `<g id="panel-…"><title>…</title><path d="${panel.path}"/></g>`. **Requires `panel.path`, a precomputed path string** — tray components store their shape in `.svg` (a complete `<rect .../>` or `<path d="…"/>` element string), not a bare path-data attribute value.

**Joint Fit Coupon fixtures (`runDesignGeometryFixtures()`, coupon-specific block around `:4150-4210`)** — hand-derived literal expectations for signed transitions (e.g. `couponPair04` at clearance `0.04`), golden SVG length/hash for the default draft, label-omission behavior, and a `localStorage` before/after identity check around coupon generation (`:4208`). This block establishes the exact fixture *idiom* (literal-derived expected transitions, not re-run formulas) this design's fixture plan (§10) follows.

**`trayTabProfile(lengthMm, jointStyle)` (`:1933-1936`)** — `count = jointStyle==='finger' ? 4 : 2`; `widthMm = clamp(8, 16, lengthMm/(count*2))`. Pure, two inputs, no side effects. **Directly reusable.**

**`designTabPositions(length, count, tabWidth)` (`:1905-1907`)** — `(length - tabWidth) * (index+1) / (count+1)` for `index` in `0..count-1`; the single even-spacing formula used for both wall tabs and (from `buildTrayModel`) base slot positions. **Directly reusable, must not be re-derived.**

**`designWallPath(x, y, width, height, tabCount, tabWidth, tabDepth)` (`:1908-1919`)** — builds a closed H/V-only wall outline with `tabCount` rectangular tabs of `tabWidth` protruding `tabDepth` below the wall's bottom edge, using `designTabPositions` internally. **Directly reusable via `trayWallComponent`.**

**`trayWallComponent(id, name, xMm, yMm, widthMm, wallHeightMm, tabProfile, thicknessMm, role)` (`:1956-1959`)** — thin wrapper: calls `designWallPath(xMm,yMm,widthMm,wallHeightMm,tabProfile.count,tabProfile.widthMm,thicknessMm)` and returns `{id,name,kind:'wall',role,geometry:{type:'path',xMm,yMm,widthMm,heightMm:wallHeightMm+thicknessMm},cutGeometry:{type:'path',d},tabProfile:{...tabProfile},svg:'<path d="${d}"/>'}`. **Tab depth is hard-coded to `thicknessMm`** — confirming tab depth must never be an independent coupon parameter; it is always the material thickness, exactly as production.

**`trayRectComponent(id, name, kind, xMm, yMm, widthMm, heightMm, extra)` (`:1953-1955`)** — returns `{id,name,kind,geometry:{type:'rect',...},svg:'<rect .../>',...extra}`. Used for both the tray base and, in `buildTrayModel`, for each individual slot rectangle.

**`buildTrayModel(values)` (`:1963-2011` region — walked in full this pass)** — the load-bearing example this design mirrors:
- `slotWidthMm = thicknessMm + fitClearanceMm` is computed **inline**, not a named function — the one formula this design extracts (see §3).
- `addWallSlots(wallId, role, profile, horizontal, xMm, yMm)` iterates `designTabPositions(...)` and, for each tab position, builds one `trayRectComponent` slot sized `horizontal ? {profile.widthMm, slotWidthMm} : {slotWidthMm, profile.widthMm}` (width×height depending on wall orientation) — **slots are separate closed rectangles layered inside the base's footprint in the same red-stroke group**, which is how laser cutting represents a cutout: a closed path inside another closed path is cut as a hole, no boolean/compound-path support needed. This is the exact mechanism the coupon reuses for its base strip.
- Walls are placed by hand-computed `wallY`/`sideX`/`secondWallY` (margin/gap arithmetic), **not** through `layoutDesignPanelRows`. This confirms tray-family layout is "hand-placed rows," precisely what §4 needs to mirror.
- `trayModelValidationErrors(values)` (`:1937-1952`) enforces (among tray-only checks like divider count) the **non-overlap inequality**: `width + 2*thickness < widthProfile.widthMm*(widthProfile.count+2)` (or the depth analogue) → error `'Tray edges are too short for non-overlapping wall tabs.'`. This inequality is the minimum-feature rule this design's coupon validation must mirror (§6) — but `trayModelValidationErrors` itself is tray-template-specific (validates `trayWidth`/`trayDepth`/`dividerCount`, none of which a coupon has) and must **not** be called by the coupon directly.

**`buildTrayDesignResult(normalized)`** and **`serializeTrayCompatibilitySvg(model)` (`:2012-2015`)** — `serializeTrayCompatibilitySvg` is a one-line wrapper: `designSvgDocument(model.layout.widthMm, model.layout.heightMm, model.layout.items.map(item=>item.svg))`. **`designSvgDocument(width,height,shapes)` (`:1902-1904`)** itself is generic and template-agnostic — one anonymous `<g fill="none" stroke="#ff0000" stroke-width="0.1">` wrapping raw shape strings, no IDs/titles/text. This is the exact serializer the coupon should call directly (not through the tray-specific compatibility wrapper, since the coupon is not a tray).

**Coupon promotion (`buildJointCouponPromotionCandidate`, `:5624-5637`)** — `couponIdentity` object has no `couponType`/`sourceType` distinguishing field beyond the fixed literal `'joint-fit-coupon-session'`; winner is **always** written via `promotionAddField(candidate,'fingerJointClearanceMm', …, 'fitSettings.fingerJointClearanceMm')` (`:5628`) — hard-coded field path, confirmed independently of the challenge review's citation. **`promotionStatusRules` verified-coupon rule** (read at `:5710` region) requires `physical.piecesFit && !!candidate.fields?.fingerJointClearanceMm` for a coupon to verify — again finger-specific by construction, confirming a wall-to-base winner cannot honestly reach `verified` today even if promotion were wired up.

**Promotion entry point (`:3272`, `:3290`, `openJointCouponPromotion` `:5896-5901`)** — the "Promote physical winner" button is rendered by exactly one condition: `jointCoupon && result.valid` (`:3272`). This is the single choke point to gate for the new coupon type — no other code path can reach `openJointCouponPromotion`.

**Production-setting Designs application (`productionSettingDesignApplicability`, `:1641-1691`, mapping table `:1663-1679`)** — `'joint-fit-coupon': [['materialThickness','materialThickness','Material thickness',material.measuredThicknessMm,'thickness']]` — **only thickness is ever offered for the coupon template today; no fit-clearance mapping exists to gate.** Confirms no change is needed here for this design.

---

## 2. Coupon type and UI contract

### Decision

Add `couponType` as a new session-draft field on `joint-fit-coupon` only, with exactly two values:

| Internal ID | Visible label | Default |
|---|---|---|
| `finger-edge` | **Finger-edge clearance** | Yes (matches today's only behavior) |
| `wall-base-tab` | **Wall-to-base tab clearance** | No |

**Byte-identity guarantee:** `normalizeDesignDraft`'s `joint-fit-coupon` branch (`:2101-2109`) gains one line reading `couponType` with a default of `'finger-edge'` when absent/unrecognized; `buildJointCouponDesignResult` gains a single early branch `if (normalized.values.couponType === 'wall-base-tab') return buildWallBaseTabCouponDesignResult(normalized);` before its first existing line. Since `couponType` defaults to `'finger-edge'` and every existing fixture/draft omits the field entirely, **every existing code path, golden byte, and fixture is unreachable-changed** — confirmed by reading `buildJointCouponDesignResult`'s existing body, which has zero dependency on any new field.

### Fields that stay, hide, or reword per type

| Field | Finger-edge (unchanged) | Wall-to-base tab |
|---|---|---|
| `materialThickness` | shown, shared | shown, shared (same field — both coupon types measure the same sheet) |
| `jointCouponEdgeLength` | "Mating-edge length (mm)" | **hidden** — replaced by a new `jointCouponWallLength` (§4) |
| `jointCouponBodyDepth` | "Coupon grip depth (mm)" | **hidden** — wall-to-base coupon has no analogous "grip depth"; base-strip depth is derived (§4) |
| `jointCouponPreferredFingerWidth` | "Preferred finger width (mm)" | **hidden** — tab width/count come from `trayTabProfile`, not a preferred-width search |
| `jointCouponClearances` | signed **finger** clearance list | **same field, reworded label**: "Slot clearance candidates (mm)" — same parser, reinterpreted as signed extra clearance added to `thickness + candidate` for the slot width (§3); still `-0.10`..`+0.30`, 3–8 values |
| `jointCouponLabels` (score-layer text labels) | shown | **hidden** — wall-to-base output uses the anonymous tray-style serializer, which has no label/score capability (§8) |
| New: `jointCouponWallJointStyle` | n/a | **shown only for wall-to-base** — reuses the tray's existing two values `finger`/`tab-slot` with their already-cleaned-up labels "Four-tab walls (wall-to-base)" / "Two-tab walls (wall-to-base)" (§3) |
| New: `jointCouponWallLength` | n/a | **shown only for wall-to-base** — representative test-segment length (§4) |

### Understandability

One template, two types, is the right scope for a first ship. The form already branches heavily per template (`jointCoupon ? jointCouponFields : cabinet ? … `); adding one more internal branch inside the existing `jointCoupon` branch (keyed on `couponType`) is the same pattern already used for `hanging` inside the legacy branch (`:1860-1865`) — no new UI paradigm. A `couponType` select at the top of `jointCouponFields`, with the rest of the fields conditionally shown beneath it, keeps the form legible without inventing a second template or a registry.

---

## 3. Geometry-reuse contract

### What the coupon calls, in order

```
1. trayTabProfile(wallLengthMm, jointStyle)              → {count, widthMm, lengthMm}
2. designTabPositions(wallLengthMm, profile.count, profile.widthMm)   [implicitly, inside step 3 and step 4]
3. trayWallComponent(id, name, xMm, yMm, wallLengthMm, wallHeightMm, profile, thicknessMm, role)
     → one wall piece, tabs protruding thicknessMm below its bottom edge
4. traySlotWidthMm(thicknessMm, candidateClearanceMm)     [NEW two-line helper, extracted from buildTrayModel]
     → slotWidthMm for this specific candidate
5. designTabPositions(wallLengthMm, profile.count, profile.widthMm)   → tab/slot x-offsets (same call as step 3 makes internally; the coupon calls it a second time only to place slot rectangles, exactly mirroring buildTrayModel's own addWallSlots)
6. trayRectComponent(slotId, name, 'base-wall-slot', xMm, yMm, profile.widthMm, slotWidthMm, {...})   [one per tab position]
7. trayRectComponent(baseId, name, 'base', xMm, yMm, baseStripWidthMm, baseStripDepthMm)   → the base strip itself, placed under/around its slots
```

Nothing in this list recomputes tab count, tab width, tab spacing, or tab depth — every one of those values is produced by `trayTabProfile`/`designTabPositions`/`designWallPath` (via `trayWallComponent`), called with the same argument shapes `buildTrayModel` already uses. **The only new arithmetic is `traySlotWidthMm`, and it is a two-line extraction of an existing inline expression, not new geometry logic:**

```js
// NEW — extracted verbatim from buildTrayModel's existing inline expression
function traySlotWidthMm(thicknessMm, clearanceMm) { return thicknessMm + clearanceMm; }
```

`buildTrayModel` changes its one occurrence of `thicknessMm + fitClearanceMm` to `traySlotWidthMm(thicknessMm, fitClearanceMm)` — arithmetically identical, so the Dice `1726/51a55721` and Divider `1965/a55dda6e` production goldens **must** still pass unchanged; this is the fixture proof that the extraction is behavior-neutral (§10).

The coupon's own slot width for candidate `i` is `traySlotWidthMm(thicknessMm, candidateClearance_i)`, where `candidateClearance_i` **may be negative** (down to −0.10, per `parseJointCouponClearances`'s existing range) — unlike production trays, where `trayModelValidationErrors` requires `fitClearance >= 0` (`:1941`). This is intentional and safe: the coupon has its **own** validation surface (§6), never calls `trayModelValidationErrors`, and a negative candidate simply means "test a slot narrower than the tab" (an interference fit) — the same signed-clearance concept the Finger Coupon already uses for its own candidates, just applied to slot width instead of finger width.

### Profile choice: user-selectable, one profile per generated sheet

`trayTabProfile` already takes `jointStyle` as a parameter with two values. The coupon reuses this **exact existing field concept** (not a new enum) via a new `jointCouponWallJointStyle` select using the identical two `<option>` values and labels the terminology-cleanup phase already wrote (`:1883`). **Firm choice: one profile per generated coupon, user-selectable, default `finger` (4-tab, matching tray defaults).** Testing "both profiles on one sheet" is rejected for the first ship — it would double the piece count and layout complexity for a benefit Joe can get for free by generating the coupon twice (once per profile), exactly how the Finger Coupon itself only ever tests one nominal pattern per sheet (`buildFingerPattern` is computed once and shared by every pair, `:2268`).

---

## 4. Coupon physical layout

**One self-contained wall+base-strip pair per candidate clearance**, directly mirroring the Finger Coupon's "one A+B pair per candidate" convention (`:2272-2292`) rather than one wall tested against many base strips. This keeps every candidate independently identifiable and keep-or-discard, matching the existing coupon UX Joe already knows.

| Spec | Value | Rationale |
|---|---|---|
| Candidate count | 3–8 (same as Finger Coupon) | Reuses `parseJointCouponClearances` verbatim |
| Default candidates | `0.00, -0.02, -0.04, -0.06, 0.02` (5 values, centered on zero, biased toward interference since tray `fitClearance` default is `0.15` and a coupon's job is to find how much *less* clearance still inserts) | Mirrors the Finger Coupon default's spread-around-zero pattern (`0.06,0.04,0.02,0.00,-0.02,-0.04`) but tuned for slot fit rather than finger fit; **open decision, see §13** |
| Signed clearance? | Yes — same `-0.10..+0.30` range, same parser | No new validation surface |
| Wall pieces | 1 per candidate | Self-contained pairs |
| Base strips | 1 per candidate, each holding exactly `profile.count` slots (all identical slot width for that candidate) | A strip with all slots at one candidate's width lets Joe insert the *same* wall's full tab row into that strip in one motion — closer to the real tray insertion feel than one-slot-at-a-time |
| Labeling | **Position order, not laser-etched text** | The anonymous tray-style serializer (§8) has no text/label capability, matching how production trays themselves carry no labels today. Pairs are laid out left-to-right in the same order as the parsed candidate list (ascending or as-typed), and the result panel (§9) prints the candidate list in that same order so Joe can read positions off the on-screen layout description before cutting. |
| Assembly direction | Wall inserts tab-first into its matching base strip, same direction as production (tabs point down into slots) | No new orientation concept |
| Minimum spacing between pieces | Reuse the coupon layout's existing `gap=10mm` convention (Finger Coupon calls `layoutDesignPanelRows(model.panels, rows, 10, 10)`, `:3030`) — wall-to-base uses the same `10mm` margin/gap when hand-placing rows, for visual consistency across coupon types even though it does not call that function directly | Consistency, not reuse-by-call |
| Confusing which wall matches which slot | Each pair is placed in the same column: wall directly above (or beside) its matching base strip, both carrying the same candidate index in their internal `id` (`wall-tab-coupon-c01`, `wall-tab-coupon-base-01`) — visible only as geometry position since IDs are not printed on cut anonymous shapes, but the layout order is documented in the result panel text | No ambiguity if cut in printed order |
| Full wall vs. shortened segment | **Shortened representative segment** — new field `jointCouponWallLength`, default **80mm**. At 80mm with the `finger` (4-tab) profile, `trayTabProfile` clamps tab width to `min(16, 80/8)=10mm`, comfortably above the 8mm floor; at `tab-slot` (2-tab), `min(16,80/4)=16mm`. Both fit the non-overlap inequality (§6) with margin. Joe can increase it to match a specific tray's actual wall length if he wants a closer analogue, but the default does not require him to also enter tray width/depth/height just to generate a coupon. |
| Approximate finished size | Roughly **~120mm × ~90mm** for the default 5-candidate, 4-tab configuration (5 walls @ 80mm + gaps in one row, 5 base strips @ ~26mm width + gaps in a second row) — compact enough for a single scrap offcut, consistent with "useful on ~3mm plywood" without being so tiny it's hard to identify by hand |

Do not optimize purely for minimum material: an 80mm wall segment (vs. e.g. a 20mm stub) is deliberately large enough to hold a full representative tab row and to be handled/labeled by hand without magnification.

---

## 5. Production-fidelity limits

| Dimension | Coupon fidelity | Gap vs. full tray |
|---|---|---|
| Tab width | **Exact** — same `trayTabProfile` formula, same length-dependent clamp | None if `jointCouponWallLength` is set to match the real wall |
| Tab count | **Exact** — same formula | None |
| Tab depth | **Exact** — always `= thicknessMm`, same as production (`trayWallComponent` hard-codes this) | None |
| Slot width | **Exact per candidate**, via the shared `traySlotWidthMm` helper | None — this is the entire point of the coupon |
| Slot length (the tab-width dimension of the slot) | **Exact** — same `profile.widthMm` | None |
| Local heat accumulation | **Approximate** — a short segment near a scrap edge cuts in less time and with different surrounding thermal mass than a full 4-wall tray sheet | Real; disclose |
| Grain direction | **User-controlled, not guaranteed** — the coupon does not know or enforce sheet grain orientation relative to the tab row | Real; disclose — recommend Joe orient the coupon on the same sheet the same way he plans to orient the tray walls |
| Wall insertion direction | **Exact** — tabs point the same way (down, into slots) | None |
| Material thickness behavior (warp, batch variation) | **Only as good as "same sheet"** — a coupon cut from a different sheet/batch than the eventual tray does not represent that sheet's variation | Real; disclose |

**Recommended wording:** *"This is a feature-clearance coupon, not a full-tray proof. It tests local wall-tab and base-slot fit at the same measured thickness and cut settings you intend to use for a tray, on a short representative wall segment. It does not test full-tray squareness, corner strength, glue strength, warping, cumulative four-wall error, or Divider retention. Cut on the same sheet or batch as your tray whenever possible, and orient the wall segment the same way relative to sheet grain that your real tray walls will be."*

---

## 6. Validation rules

All rules attach to a new coupon-specific validator (not `trayModelValidationErrors`, which stays tray-template-only). Reused constants are cited explicitly.

| Rule | Formula / source | Reuse |
|---|---|---|
| Measured thickness | `>= 0.1`, same as every other template (`designRequiredNumber(draft,'materialThickness',...,0,true)`, existing shared line `:2100`) | Fully shared field, zero new code |
| Candidate clearance list | `parseJointCouponClearances(draft.jointCouponWallClearances, errors)` — identical parser, 3–8 values, `-0.10..+0.30`, 2-decimal, dedup | **Reused verbatim**, new field name to avoid confusing it with the finger-type field in fixtures/UI state, but same function |
| Wall length | `> 0`, plus the non-overlap inequality below | New `designRequiredNumber` call |
| Joint style | must be `finger` or `tab-slot` | Same allow-list check pattern as `normalizeDesignDraft`'s tray branch (`:2161`), duplicated as a one-line check since the coupon's `couponType==='wall-base-tab'` branch is a sibling of the tray branch, not the same branch |
| Non-overlap / minimum wall length for the profile | `wallLengthMm >= profile.widthMm * (profile.count + 2)` | **Mirrors** (does not call) `trayModelValidationErrors`'s inequality (`:1949`) — same shape, applied to the coupon's own `wallLengthMm` instead of a tray's `trayWidth`/`trayDepth`. A fixture (§10) asserts the coupon's threshold numerically agrees with the tray's for matched inputs, so drift is caught even though the two call sites are not the same function. |
| Minimum slot web (base-strip material around each slot) | `baseStripWidthMm >= profile.widthMm + 2*requiredWeb`, `baseStripDepthMm >= slotWidthMm + 2*requiredWeb`, where `requiredWeb = Math.max(.5, thicknessMm*.25)` | **Reuses the exact `requiredWeb` formula** already used four other places in the file (`buildJointCouponModel:2263`, `buildBoxModel:2257`, `buildSlidingLidBoxModel`, `buildDrawerCabinetModel`) — the established app-wide minimum-material convention, applied to a context (tray slot webs) that does not currently have its own explicit floor |
| Slot-width collapse (candidate too negative) | For each candidate, `traySlotWidthMm(thicknessMm, candidateClearance) >= requiredWeb` else error naming that candidate | New check, same `requiredWeb` constant, mirrors the collapse-guard pattern in `buildJointCouponModel` (`:2269`) |
| Duplicate/invalid/malformed clearances | Already handled by `parseJointCouponClearances` | Fully reused |
| Overlap prevention between pieces | Hand-placed rows with fixed `gap=10mm` between every piece (§4) make overlap structurally impossible by construction (each piece's `x`/`y` is `previous.x + previous.width + gap`, the same placement arithmetic `buildTrayModel` already uses) — no separate overlap-detection pass is required, matching how `buildTrayModel` itself does not run `designBoxesOverlap` checks on hand-placed tray pieces either | Same placement discipline as production trays |
| Sheet-layout bounds | Total width/height computed from the same hand-placement arithmetic; no explicit bound needed beyond what `designSvgValidation` already checks generically | Reuses `designSvgValidation` |

**No silent clamping anywhere** — every rule above is a hard validation error via the existing `errors.push(...)` convention, consistent with the rest of the file's "reject, never clamp" posture (explicitly confirmed in `buildJointCouponModel`, `buildTrayModel`, and every other model builder read this pass).

---

## 7. UI behavior

**Field order when `couponType === 'wall-base-tab'`:**

1. Coupon type select (`Finger-edge clearance` / `Wall-to-base tab clearance`) — always visible, top of the coupon field block
2. `materialThickness` — shared field, label unchanged ("Measured material thickness (mm)")
3. `jointCouponWallJointStyle` select — label **"Wall tab profile"**, options "Four-tab walls (wall-to-base)" / "Two-tab walls (wall-to-base)", same wording the terminology cleanup already shipped for trays
4. `jointCouponWallLength` — label **"Test wall length (mm)"**, help: *"A short representative segment, not your full tray wall. Increase it to more closely match a specific tray if you want."*
5. `jointCouponWallClearances` (reworded textarea, same widget as `jointCouponClearances`) — label **"Slot clearance candidates (mm)"**, help: *"Enter 3 to 8 signed values added to your measured thickness to size each test slot. Negative values test a tighter (interference) fit. Same range as the finger coupon: -0.10 through 0.30 mm."*

**Hidden for wall-to-base:** `jointCouponEdgeLength`, `jointCouponBodyDepth`, `jointCouponPreferredFingerWidth`, `jointCouponLabels` checkbox and its score-layer help text.

**Warning/help text block** (replaces the existing coupon "Use:" note, `:1866-1867`, for this type only): *"Each clearance candidate creates one wall piece and one matching base strip using the same wall-to-base tab construction as Dice and Divider Trays. This tests local tab/slot fit only — it does not prove full-tray squareness, corner strength, glue strength, or Divider retention. Cut on the same sheet as your tray when possible."*

**Existing "Test order" note** (`:1892`, currently rendered whenever `jointCoupon` is true) stays visible for both types — its guidance ("start with looser positive values, then move toward zero and negative interference values… stop if insertion force crushes, splits, or permanently locks the material") is equally correct for slot clearance as for finger clearance, with zero changes needed.

**Result panel** (`designResultsHtml`, extends the existing `jointCoupon` branch of the metrics table): shows wall tab profile, test wall length, candidate list in generated order, piece count (`2 × candidates`), and the same "Layout" width×height metric every template already reports. No new metric concepts.

No mechanical-engineering terms are introduced — "tab," "slot," "clearance," "snug/tight/loose" match the vocabulary the terminology-cleanup phase already established for trays.

---

## 8. Result and SVG contract

| Question | Answer |
|---|---|
| Named score/cut serializer (`serializeDesignSvg`)? | **No.** Panel shape incompatibility (§1, §3) makes this the wrong contract; using it would require reimplementing tab geometry as points, which is the exact duplication this design avoids. |
| What serializer, then? | **`designSvgDocument(widthMm, heightMm, shapes)`** — the same generic anonymous-group serializer trays already use, called directly (not through `serializeTrayCompatibilitySvg`, which is tray-model-shaped and the coupon is not a tray). |
| Cut group content | One anonymous `<g fill="none" stroke="#ff0000" stroke-width="0.1">` containing each wall's `<path d="…">` (from `trayWallComponent.svg`) and each slot/base-strip's `<rect>` (from `trayRectComponent.svg`) — no IDs, no titles, matching the tray contract exactly. |
| Score-label content | **None.** No score layer exists for this type (§4, §7) — consistent with trays having no labels either. |
| Panel names | N/A in the output SVG (anonymous, matching trays) — internal names only exist in the pre-serialization component objects for fixture readability. |
| Filename | `l8-joint-fit-coupon-YYYY-MM-DD.svg` — **unchanged pattern**, since the template key stays `joint-fit-coupon` regardless of `couponType`. |
| MIME type | `image/svg+xml;charset=utf-8` — unchanged. |
| Operation ordering | Single group, no ordering question (no score layer to sequence before cut). |
| Labels required or optional? | **Not offered** for wall-to-base in this implementation (§4/§7) — an explicit non-goal, not a bug. |
| Default Finger Coupon bytes | **Byte-for-byte unchanged** — `couponType` defaults to `'finger-edge'`, and `buildJointCouponDesignResult`'s existing body is reached exactly as today whenever it is `'finger-edge'` (§2). Pin the existing goldens as the proof fixture (§10). |
| New wall-to-base output contract | **New, pinned from its first commit** — deterministic default draft → golden length/hash, exactly like every other template's default golden. |
| Tray anonymous production SVG | **Untouched** — `buildTrayModel`/`serializeTrayCompatibilitySvg`/Dice `1726/51a55721`/Divider `1965/a55dda6e` are unaffected by anything in this design except the internal `traySlotWidthMm` extraction, which must re-prove those exact goldens unchanged (§3, §10). |

---

## 9. Promotion non-goals

**Recommended approach: hide the promotion control for wall-to-base mode; allow free-text result notes only.**

Concretely: the existing single gate at `:3272`, `jointCoupon && result.valid`, becomes `jointCoupon && result.valid && normalized.values.couponType !== 'wall-base-tab'` (reading the already-computed `result.metrics.couponType` rather than re-deriving it, so the check has one source of truth). This is a **one-line change to an existing conditional**, not new promotion code, and it is the only touch this design makes anywhere near `openJointCouponPromotion`, `buildJointCouponPromotionCandidate`, `promotionStatusRules`, or `promotionCanonicalize`.

The result panel (§7) may include a plain, non-persisted note reminding Joe that promotion is not available for this coupon type yet, and that any winning clearance should be written down manually (Library notes, or a future Test Grid/Material Test entry) rather than promoted through the existing "Promote physical winner" flow, which would incorrectly write into `fitSettings.fingerJointClearanceMm`.

**Explicitly documented for a later phase (not designed or implemented here), per the challenge review's own §5 field-classification table, independently re-confirmed against source this pass:**

1. **Coupon identity type** — `couponIdentity` (`:5625`) needs a `couponType` (or `sourceType`) discriminator before two coupon kinds can safely share promotion dedup; inserting it without versioning would silently change `sourceId` hashes for existing finger sessions (challenge review §5, confirmed: `couponIdentity` today has no type field at all).
2. **Verification rules** — `promotionStatusRules`'s coupon-verify branch (`:5710` region) hard-requires `candidate.fields?.fingerJointClearanceMm`; a wall-to-base winner would need its own applicability rule.
3. **Winner field name** — needs a decision between reusing `fitSettings.fingerJointClearanceMm` (semantically wrong) versus a new optional `fitSettings.*Mm` key (needs `strictOptionalNumber`, a production-setting form field, and fingerprint-entry work per the field-classification table in the challenge review, independently confirmed here: the production-setting form (`:1663-1679` mapping) only ever edits two known fit fields today).
4. **Production-setting normalization** — any new fit key flows through `normalizeProductionSetting`'s existing spread-then-overwrite pattern (additive, no schema bump), but the **form/UI** must explicitly add a field for it or the value is stored-but-uneditable.
5. **Fingerprinting** — `designProductionFingerprint` (cited at architecture-challenge `~1519`, not re-walked this pass since no code here touches it) enumerates known fit keys for Designs review staleness detection; a new key must be added there or it will not invalidate a stale Designs review.
6. **Tray application mapping** — `productionSettingDesignApplicability`'s mapping table (`:1663-1679`, confirmed this pass) has **no fit-clearance entry for `dice-tray`/`divider-tray` at all today**, only thickness — a later phase would need to decide whether/how a proven tab clearance ever flows back into a tray draft's `fitClearance` field.

None of the above is designed in detail here, matching the task's explicit boundary.

---

## 10. Fixture plan

Deliberately **not** a full 23-case product matrix (the challenge review's §10 explicitly warns against this for coupons, and nothing in this design's source findings shows a need for one — the coupon has no divider-count-style combinatorial surface). Recommended set, with classification:

| # | Assertion | Class |
|---|---|---|
| 1 | Default Finger Coupon (`couponType` absent/`'finger-edge'`) production SVG length/hash **unchanged** from today's pinned golden | **Independent** (literal pinned bytes) |
| 2 | `traySlotWidthMm` extraction: Dice default `1726/51a55721` and Divider default `1965/a55dda6e` **unchanged** after the refactor | **Independent** (literal pinned bytes — proves the extraction is behavior-neutral) |
| 3 | Wall-to-base default draft → deterministic SVG, new pinned length/hash | **Independent** once the golden is captured; **partially circular** on first write (the golden is generated by the code under test) — mitigated by cross-checking piece count and layout width/height against hand-computed values from `trayTabProfile`/`designTabPositions` inputs |
| 4 | Invalid `couponType` (e.g. `'dovetail-thing'`) rejected with a clear error, falls back to `'finger-edge'` behavior or errors explicitly (pick one; recommend explicit reject, matching the app's "reject, don't guess" convention) | **Independent** |
| 5 | Wall-to-base wall piece's tab count/width for a given `wallLengthMm`/`jointStyle` equals `trayTabProfile(wallLengthMm, jointStyle)` called directly — **object/value equality, not visual comparison** | **Independent** (builder-reuse proof) |
| 6 | Wall-to-base slot width for candidate `c` equals `traySlotWidthMm(thickness, c)` called directly | **Independent** |
| 7 | Four-tab vs. two-tab (`jointStyle` both values) produce differing tab counts (4 vs 2) in the generated piece, same `wallLengthMm` | **Independent** |
| 8 | Signed clearance ordering: candidates `[0.02, 0.00, -0.02]` produce slot widths `[t+0.02, t, t-0.02]` in generated order — hand-derived, not re-run through the formula under test | **Independent** |
| 9 | Piece count = `2 × candidateCount`; every generated piece's internal id maps to exactly one candidate index | **Independent** |
| 10 | Finite, non-overlapping geometry: no `NaN`/`Infinity`/`undefined` in output SVG; every piece's translated bounding box is separated from every other by at least the fixed gap (same box-overlap style check used elsewhere, e.g. `designBoxesOverlap`/`designBoxesSeparatedBy`, reused as a generic diagnostic — **this is a legitimate reuse**, those functions are geometry-agnostic utilities already used by trays and boxes alike) | **Independent** |
| 11 | Minimum-web / collapse rejection: a candidate whose `traySlotWidthMm` result falls below `requiredWeb` is rejected with a named error, using the same **literal** `requiredWeb` value the fixture computes independently (`Math.max(.5, t*.25)`), not by calling the coupon's own internal check | **Independent** |
| 12 | `localStorage` before/after identity around wall-to-base generation (mirrors the existing finger-coupon fixture at `:4208`) | **Independent** |
| 13 | Promotion button absent from rendered markup when `couponType==='wall-base-tab'` and `result.valid` (string-search the rendered HTML for the `data-promote-joint-coupon` attribute) | **Independent** |
| 14 | Filename `l8-joint-fit-coupon-<date>.svg` and MIME `image/svg+xml;charset=utf-8` for wall-to-base downloads, via the same live-preview/download DOM-intercept fixture pattern already used for trays (`trayLivePreviewDownloadFixture`/`diceFinishedViewBrowserFixture` precedent) | **Independent**, drives real DOM |
| 15 | Determinism: identical draft → byte-identical SVG across two calls | **Partially circular** (calls the function twice and compares) — acceptable only alongside the independent literal checks above, per this project's established fixture-quality discipline |

**Avoiding the two-engine trap in the fixtures themselves:** assertions 5, 6, and 11 specifically call the *shared* helper functions (`trayTabProfile`, `traySlotWidthMm`) and compare their output against the coupon's generated geometry — if a future edit made the coupon stop calling these helpers and started recomputing tab/slot math independently, these fixtures would catch the drift even though they cannot see the coupon's internals directly.

---

## 11. Physical test procedure

Distinct from every software fixture above — this is what Joe does with the laser, not what the app verifies.

1. **Measure actual thickness** of the specific sheet with calipers; enter that measured value, not the nominal (3mm) label.
2. **Choose 3–8 clearance candidates** centered around your current tray default (`0.15`) and extending into negative (tighter) territory — e.g. `0.15, 0.05, 0.00, -0.05, -0.10` as a first spread, or use this design's proposed default (§4) and adjust after one round.
3. **Use the exact focus, speed, power, passes, and air-assist settings** you intend to use for the real tray — same LightBurn profile, not a "test mode."
4. **Cut on the same sheet or the same batch** as the tray you're about to build, when practical — thickness and char behavior vary sheet to sheet.
5. **Let the coupon cool before handling** — char and slight material softening near the kerf can mislead a hot-to-the-touch fit test.
6. **Insert each wall into its matching base strip in printed/generated order** — do not force. Note **loose / snug / tight / will-not-insert** per candidate, plus any visible char buildup at the tab edges.
7. **Do not apply excessive force** to test a tight candidate — if a tab does not seat with light hand pressure, record it as too tight rather than forcing it, exactly as the app's existing coupon guidance already tells Joe for the finger coupon (`:1892`'s "stop if insertion force crushes, splits, or permanently locks the material" applies unchanged).
8. **Record the winning candidate manually** — in a Library note, a Material Test, or on paper for now; there is no promotion path for this coupon type yet (§9).
9. **Never leave the laser unattended** during the cut, matching the app's existing safety posture throughout its documentation.

Software verification stops at "the SVG is finite, deterministic, and geometrically sound." Physical verification is everything from step 6 onward and cannot be automated or inferred from the model.

---

## 12. Exact Codex implementation scope

**One commit.** No new file, no new framework, no capability registry.

### Functions added

- `traySlotWidthMm(thicknessMm, clearanceMm)` — 2-line extraction, called from `buildTrayModel` and the new coupon path
- `buildWallBaseTabCouponModel(parameters)` — builds wall + base-strip component pairs per candidate, validation per §6, returns `{valid, errors, warnings, pairs, metrics}` in the same shape family as `buildJointCouponModel` for symmetry
- `buildWallBaseTabCouponDesignResult(normalized)` — lays out pairs (hand-placed rows, §4), serializes via `designSvgDocument` (§8), returns the same result shape (`{valid,errors,warnings,svg,widthMm,heightMm,panels,metrics}`) `buildJointCouponDesignResult` returns, so `buildDesignResult`'s dispatcher and `designResultsHtml`'s generic result handling need no new branching beyond the metrics table (§7)
- `jointCouponWallFields(d)` (or inlined into the existing `jointCouponFields` builder as a conditional block) — the new form fields (§7)

### Functions modified

- `buildTrayModel` — one-line change: inline `thicknessMm + fitClearanceMm` → `traySlotWidthMm(thicknessMm, fitClearanceMm)`. **Must reproduce identical bytes.**
- `normalizeDesignDraft`'s `joint-fit-coupon` branch (`:2101-2109`) — read `couponType` (default `'finger-edge'`); when `'wall-base-tab'`, validate the new fields instead of the finger-specific ones
- `buildJointCouponDesignResult` — one new early-branch line to `buildWallBaseTabCouponDesignResult` when `couponType==='wall-base-tab'`; **zero other changes to its existing body**
- `renderDesigns`'s `jointCouponFields` construction (`:1847-1852`) — add the `couponType` select and the conditional field set (§7)
- The promotion-button gate at `:3272` — add `&& couponType !== 'wall-base-tab'` (§9)
- `designResultsHtml`'s `jointCoupon` metrics branch — add wall-to-base-specific metric rows (§7)
- `runDesignGeometryFixtures` (or a small sibling fixture function called from it, matching the `runTrayModelFixtures` pattern) — the fixture set in §10

### Expected files

`index.html` only. `README.md` gets a short new sentence describing the second coupon type, mirroring the existing coupon-paragraph style, plus the updated fixture totals.

### Protected — must remain byte-identical (verify via diff, not trust)

- `buildJointCouponModel`, `buildJointCouponDesignResult`'s existing body, `parseJointCouponClearances`, `jointCouponLabelText` — finger-coupon geometry untouched
- `buildTrayModel`'s **output bytes** (Dice `1726/51a55721`, Divider `1965/a55dda6e`) — the one internal line changes, the output must not
- `designWallPath`, `trayTabProfile`, `designTabPositions`, `trayRectComponent`, `trayWallComponent`, `trayModelValidationErrors` — called, never edited
- `serializeDesignSvg`, `serializeTrayCompatibilitySvg`, `designSvgDocument` — called (the last one, directly), never edited
- `buildJointCouponPromotionCandidate`, `promotionStatusRules`, `promotionCanonicalize`, `promotionSignature`, `openJointCouponPromotion` — **zero changes**, per §9
- `productionSettingDesignApplicability`'s mapping table — zero changes (coupon already only offers thickness)
- `STORAGE_KEY`, `SCHEMA_VERSION`, `backupObject`, `persist`, `normalizeProductionSetting(s)` — untouched; nothing here persists

### Expected fixture groups

The 15 assertions in §10, added to (or alongside) `runDesignGeometryFixtures`'s existing coupon block, plus the two `traySlotWidthMm`-non-regression checks folded into the existing tray golden assertions.

### Manual browser checks

1. Select Joint Fit Coupon → confirm default type is Finger-edge, form and output identical to before this change.
2. Switch to Wall-to-base tab clearance → confirm the finger-specific fields disappear and the new fields appear.
3. Generate a default wall-to-base coupon, confirm preview renders, confirm no "Promote physical winner" button appears.
4. Switch back to Finger-edge → confirm the promote button reappears and the finger fields return with their prior values intact (session draft round-trip).
5. Download while wall-to-base is selected → confirm filename/MIME/bytes match the previewed SVG.
6. Switch template away from Joint Fit Coupon and back → confirm `couponType` resets to session default without error.

### Unverified/physical areas

Everything in §11 — insertion feel, char behavior, grain-direction sensitivity, and whether the recommended default candidate spread (§4) is actually the right starting range for Joe's specific materials. This implementation proves the geometry is sound and reusable; it does not and cannot prove a clearance value is good.

---

## 13. Risks and open questions

| # | Item | Note |
|---|---|---|
| R1 | Default candidate spread (§4) is a first guess, not physically validated | Low risk — Joe can freely edit the textarea before the first physical cut; only matters for the out-of-the-box default experience |
| R2 | One-wall-per-strip-of-N-slots (§4) vs. one-wall-tested-against-N-single-slot-strips | This design picked the former for simplicity/consistency with the Finger Coupon's pairing convention; the latter is arguably a closer single-tab fit test but adds layout complexity for a first ship — **flagging as an open decision**, not resolved here |
| R3 | `jointCouponWallLength` default of 80mm is a judgment call, not derived from any existing constant | Low risk — purely a starting convenience; validation (§6) rejects anything too short for the chosen profile regardless of default |
| R4 | Reworded textarea field (`jointCouponWallClearances`) duplicates `parseJointCouponClearances`'s call site rather than sharing the exact same draft key as the finger type | Deliberate — keeps finger and wall-to-base candidate lists independent in the session draft so switching coupon type does not silently blend two different physical meanings under one field name; costs one extra draft key, not one extra parser |
| R5 | No score-layer labels means Joe must track candidate identity by position only | Accepted limitation for the first ship (§4, §8); matches existing tray behavior exactly, so it is not a *new* gap this feature introduces |
| R6 | Should the "Promote physical winner" gate check `result.metrics.couponType` or `normalized.values.couponType`? | Recommend `result.metrics.couponType` (set by whichever build path ran) so the check reflects what was actually generated, not merely what the form currently shows — matters if a stale `result` is ever rendered against a changed `designDraft.couponType` mid-render |
| R7 | Is `traySlotWidthMm` worth extracting for a single new caller, given the Grok challenge review's "avoid dual sources of truth for a two-option surface" caution? | Yes — unlike the capability registry (rejected in the challenge review for having no second consumer), this extraction has **two real call sites from day one** (`buildTrayModel` and the new coupon) and is two lines, the smallest possible unit of reuse, not new architecture |

No open question above blocks implementation; all are refinable in review or in a follow-up small commit.

---

## 14. Final verdict

```text
DESIGN READY FOR IMPLEMENTATION
```

The wall-to-base tab clearance coupon can ship as one reviewable commit: a new `couponType` field defaulting to today's exact behavior, a new geometry path built entirely from existing tray primitives (`trayTabProfile`, `designTabPositions`, `trayWallComponent`, `trayRectComponent`) plus one small, provably behavior-neutral extraction (`traySlotWidthMm`), serialized through the tray family's existing anonymous contract rather than a forced adapter into the incompatible Finger Coupon pipeline, with promotion explicitly and minimally disabled at its single existing gate. No registry, no internal enum rename, no schema change, and no production-setting field are required, consistent with the architecture challenge review's Phase B scope.

---

*Design review performed read-only at commit `541cda5` (clean working tree). No application file was modified, staged, committed, or pushed.*
