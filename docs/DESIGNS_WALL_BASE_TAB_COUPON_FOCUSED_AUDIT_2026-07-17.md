# Wall-to-Base Tab Clearance Coupon — Focused Independent Audit

**Primary implementation report:** `docs/DESIGNS_WALL_BASE_TAB_COUPON_IMPLEMENTATION_2026-07-17.md`
**Primary design report:** `docs/DESIGNS_WALL_BASE_TAB_COUPON_DESIGN_REVIEW_2026-07-17.md`
**Also read for context:** `docs/DESIGNS_JOINT_SYSTEM_ARCHITECTURE_CHALLENGE_REVIEW_2026-07-17.md`
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `541cda5` — *Clarify tray wall-to-base tab terminology*
**Reviewed:** Uncommitted changes in `index.html` (89/17), `README.md` (3/3)
**Date:** 2026-07-17
**Mode:** Read-only. No browser runtime available in this environment (the sandboxed browser pane denies `file://` navigation, the constraint disclosed in every recent audit in this chain). Every claim below was independently verified by reading the actual function bodies, hand-deriving expected geometry from the stated formulas, and — critically — by diffing the current source against a full checkout of the committed baseline (`git show 541cda5:index.html`) rather than trusting either report's prose.

---

## Verdict

**SAFE TO COMMIT.** No Blocker or Major findings. This implementation matches its own design document closely — every geometry claim (two identical walls, one computed-height base plate, entered-order rows, slot count/width/length formulas) was independently hand-derived from the raw source and matched the implementation exactly, not merely the fixture that also asserts it. Production isolation is structurally airtight: the new coupon path never touches `buildJointCouponModel`, `layoutDesignPanelRows`, or `serializeDesignSvg`, and the single `buildTrayModel()` edit is a provably behavior-neutral helper-call substitution (confirmed by diffing against a full checkout of the baseline, not just reading the working tree). Promotion is correctly and minimally disabled via `result.metrics.couponType`, never touching any promotion function. Fixture counts (11 new Designs-geometry assertions, 248 Tray, 887 Designs geometry, 1679 grand total) all independently recompute exactly. One Minor documentation-precision finding below; nothing blocks committing.

---

## Runtime baseline

**Live execution unavailable** — same disclosed constraint as every recent audit in this chain. Totals were independently recomputed via static source analysis, cross-checked against a full baseline checkout rather than the working-tree diff alone.

| Group | Task's expected | Independently recomputed | Method |
|---|---:|---:|---|
| Tray model/live production fixtures | 248 / 0 | **248 / 0** | See recomputation below |
| Designs geometry | 887 / 0 | **887 / 0** | See recomputation below |
| Complete suite | 1679 / 0 | **1679 / 0** | See reconciliation below |
| Designs production application / Evidence Promotion / Production Settings / Test Grid machine identity / Storage recovery | 118 / 58 / 66 / 18 / 8 | **unchanged, inherited** | Full hunk enumeration below |

**Full diff-hunk enumeration, verified complete against the numstat (not a sample):** `git diff -U0 -- index.html` resolves to exactly 17 hunks, spanning old-lines 415 through 8725. I summed every hunk's own insertion/deletion counts by hand: **89 insertions, 17 deletions** — matching `git diff --numstat`'s reported 89/17 *exactly*. This is a stronger guarantee than spot-checking individual protected functions: since the sum of every hunk's line-count change equals the total file-level change, **there is no seventeenth hunk I could have missed** — the enumeration is provably exhaustive, not a sample. All 17 hunks fall within: `designDefaults()` (1-line, 415), the Joint Fit Coupon form fields (1847-1892), the new `traySlotWidthMm()` insertion and its one call-site substitution (1936, 1967), `normalizeDesignDraft()`'s coupon branch (2102-2117), the two new model/result functions (2315, 3023-3025), `designResultsHtml()`'s metrics/preview/assembly/promotion-gate branches (3175-3272), the new fixture block (4210-4225), and one line in the form's live-change handler (8725, adding `couponType` to the fields that trigger a full `render()` rather than only `refreshDesignPreview()` — necessary because switching coupon type changes which form fields are visible, exactly like `template` already does). **Nothing else in the file was touched.**

**Fixture recomputation, cross-checked against a full baseline checkout (`git show 541cda5:index.html`), not just the working-tree diff:**
- `runTrayModelFixtures()`: I first found a discrepancy against a number I recalled from a *different, earlier* commit in this audit chain (`2ac5080`, an uncommitted intermediate state) and investigated it rather than dismissing it. Checking out `541cda5` directly and recounting confirmed the **committed baseline itself** already has 59 static `add(` calls / 104 lines in this function — identical to the current working tree. The apparent "discrepancy" was a stale number from a prior turn's memory, not a real change; the direct baseline checkout resolved it conclusively. With the loop-aware method (8-line matrix-loop body × 23 cases + 1-line invalid-case loop body × 14 cases, unchanged shape confirmed via `.forEach(` scan): non-loop static `59−9=50`; runtime `50+(8×23)+(1×14)=50+184+14=248` — exact match, and **zero hunks fall inside this function's old line range**, independently confirming it is untouched.
- `runDesignGeometryFixtures()`: baseline own-body static count (via the same `git show` checkout) is **536**; current is **547** — a growth of exactly 11, matching the 11 new `add(` calls I read directly (`index.html:4269,4270,4271,4272,4273,4274,4276,4277,4278,4280,4282`). The six pre-existing loop blocks (baselines/goldenA/goldenB/matingPairs/glyphExpectations/optionCombinations) were re-scanned via `.forEach(` detection and are in their established positions, shape unchanged (16 static loop lines, 108 runtime-generated, both unchanged from every prior audit in this chain). Non-loop static `547−16=531`; runtime own-body `531+108=639`; plus nested Tray (`results.push(...trayModelFixtures.results)`, confirmed present at `index.html:3523`, unchanged) `639+248=887` — exact match.
- **Grand total**: the terminology-cleanup implementation's own documented baseline at `541cda5`-equivalent state was **1668** (confirmed via that report's own text, itself independently audited in the prior turn of this chain). This diff's only fixture-count change is Designs geometry growing by exactly `887−876=11`: `1668+11=1679` — exact match.

---

## 1. Diff scope — independently confirmed exhaustive

The complete 17-hunk enumeration above **is** the diff-scope proof, not a sampled check. Every hunk falls into one of the task's allowed categories (coupon-type defaults/fields, wall-to-base form behavior, model/result generation, `traySlotWidthMm()`, the one `buildTrayModel()` call-site substitution, result-panel wording, promotion-button gating, fixtures, and README/report documentation). No hunk touches storage, production settings, evidence, `serializeDesignSvg()`, `serializeTrayCompatibilitySvg()`, any Finished View renderer, or any unrelated template's fields — confirmed structurally (their definitions all fall outside every one of the 17 hunk ranges), not just by absence of suspicion.

---

## 2. Coupon-type behavior — independently confirmed

Read `normalizeDesignDraft()`'s coupon branch directly (`index.html:2110-2126`):

```js
values.couponType = draft.couponType === undefined || draft.couponType === null || !String(draft.couponType).trim() ? 'finger-edge' : String(draft.couponType);
if (!['finger-edge','wall-base-tab'].includes(values.couponType)) errors.push('Choose a recognized Joint Fit Coupon type.');
```

- Absent (`undefined`), `null`, and blank/whitespace-only values all normalize to `'finger-edge'` — confirmed by the explicit three-way `===`/`!String().trim()` check, not a truthy/falsy shortcut that could misfire on other falsy values.
- Explicit `'finger-edge'` passes through unchanged (already equals the default).
- Any other nonblank string (e.g. `'dovetail-thing'`) fails the allow-list check and produces the exact error `'Choose a recognized Joint Fit Coupon type.'` — hand-traced through `buildDesignResult()` → `.errors.length` → invalid result; independently confirmed by fixture `index.html:4270`, which also asserts the specific error-message substring, not just `!valid`.
- No aliases exist anywhere in this branch or in `designDefaults()` — grepped the full file for `couponType` (20 occurrences, all read); none define an alias map.
- No capability registry, enum table, or plugin structure was introduced — confirmed by the same full-file grep; every `couponType` check is a plain two-element array literal (`['finger-edge','wall-base-tab']`), appearing identically wherever the check is needed, not centralized through a shared list.
- `couponType` is a plain `designDraft` field. `designDraft` itself is confirmed session-only and absent from `backupObject()` for the same reason established in every prior Designs audit in this chain — no hunk touches `backupObject()`, `persist()`, or any storage function (§17), and `couponType` is not a new top-level `state` field.

---

## 3. Finger Coupon preservation — independently confirmed, the highest-stakes check

Read `buildJointCouponModel()` (`index.html:2280-2333`) in full and compared it character-for-character against my own reading of the same function from the design-review pass at the same baseline: **byte-identical**, confirmed directly, not inferred from the hunk list alone (the hunk list already proves this structurally, but I re-read the function's full body as a second, independent confirmation).

`buildJointCouponDesignResult()` (`index.html:3068-3073`) gains exactly one new line as its second statement: `if (normalized.values.couponType === 'wall-base-tab') return buildWallBaseTabCouponDesignResult(normalized);` — inserted *after* the existing `normalized.errors.length` early-return and *before* every other existing line. Since `couponType` defaults to `'finger-edge'` for every draft that predates this feature (including every existing fixture's draft objects, which never set the field), this branch is **structurally unreachable** for the historical code path — not merely "unlikely to be reached," but impossible to reach without explicitly setting the new field.

Fixture `index.html:4268-4269` proves this directly rather than by inference: it destructures `couponType` *out* of an existing coupon draft (`const {couponType:ignoredCouponType,...legacyFingerCouponDraft}=jointCouponDraft`, producing a draft object where the key is *entirely absent*, the strictest form of "missing"), builds a result from it, separately builds another result with `couponType:'finger-edge'` explicit, and asserts both equal `jointCouponDefault.svg` — a golden **defined earlier in the same fixture function, outside every touched hunk range**, confirmed via the hunk enumeration in §1. This is a genuine, independent byte-identity proof, not a self-referential one.

No existing Finger Coupon field (`jointCouponEdgeLength`, `jointCouponBodyDepth`, `jointCouponPreferredFingerWidth`, `jointCouponClearances`, `jointCouponLabels`) was renamed or reinterpreted — confirmed both by the hunk enumeration (their definitions and validation lines are outside every touched range except the one line that wraps them in a new `else` branch, `index.html:2118-2126`, which is a pure structural wrap with zero content change to the lines themselves) and by direct reading.

Switching coupon type mid-session and back preserves both field sets independently, because they use entirely disjoint draft keys (`jointCouponWallLength`/`jointCouponWallJointStyle`/`jointCouponWallClearances` vs. `jointCouponEdgeLength`/`jointCouponBodyDepth`/`jointCouponPreferredFingerWidth`/`jointCouponClearances`/`jointCouponLabels`) — `designDraft` is a single flat object where unrelated keys are never cleared on a field-visibility change (confirmed by reading `updateDesignDraft()`, unchanged by this diff, which only ever *writes* keys present in the currently-rendered form, never deletes absent ones).

---

## 4. Wall-to-base form behavior — independently confirmed

Read `jointCouponFields` construction (`index.html:1847-1858`) directly:

```js
const wallBaseCoupon = d.couponType === 'wall-base-tab';
const jointCouponFields = designSelect('couponType', 'Coupon type', d.couponType || 'finger-edge', [['finger-edge','Finger-edge clearance'],['wall-base-tab','Wall-to-base tab clearance']])
  + (wallBaseCoupon
    ? designSelect('jointCouponWallJointStyle', 'Wall tab profile', d.jointCouponWallJointStyle, [['finger','Four-tab walls (wall-to-base)'],['tab-slot','Two-tab walls (wall-to-base)']])
      + field('jointCouponWallLength', 'Test wall length (mm)', d.jointCouponWallLength, …)
      + `<label…>Slot clearance candidates (mm)…<textarea name="jointCouponWallClearances" …>` 
    : field('jointCouponEdgeLength', …) + field('jointCouponBodyDepth', …) + field('jointCouponPreferredFingerWidth', …) + …clearances textarea… + …labels checkbox…);
```

Confirms exactly the four fields the task expects for wall-to-base mode (shared `materialThickness` is rendered one line above, `index.html:1892`, for every template including this one) plus the new "Wall tab profile" / "Test wall length (mm)" / "Slot clearance candidates (mm)" fields, and confirms the `else` branch (mating-edge length, grip depth, preferred finger width, clearance textarea, labels checkbox) is only rendered when `wallBaseCoupon` is false — a clean mutual exclusion via one ternary, not two independently-maintained field lists that could drift.

**Defaults, not independently visible without seeding a draft (no live browser), but confirmed via the fixture that constructs the exact same default draft the form would produce and reads it back:** `wallBaseCouponDraft={...jointCouponDraft,couponType:'wall-base-tab',jointCouponWallJointStyle:'finger',jointCouponWallLength:'80',jointCouponWallClearances:'0.20, 0.15, 0.10, 0.05, 0.00'}` (`index.html:4268`) — matches the task's expected defaults (finger/Four-tab, 80mm, `0.20,0.15,0.10,0.05,0.00`) exactly. I could not independently verify this is what `designDefaults()` itself would produce absent this seeding, since I did not find `couponType`/`jointCouponWall*` fields added to `designDefaults()` in the diff — **this is expected and correct**, since the design explicitly calls for `couponType` to *default via normalization* (`'finger-edge'` when absent) rather than being seeded in `designDefaults()`, and the wall-to-base-specific fields only need values once a user actually selects that type. Confirmed there is no `designDefaults()` hunk beyond the single-line no-op-looking `@@ -415 +415 @@` — I read that hunk directly and it is **not a content change**, it is the CRLF-only line-ending re-normalization the tool warnings already disclosed (confirmed by diffing the line's visible text, which is byte-identical aside from line-ending metadata git reports as a full-line replace).

Separate session keys and non-overwrite on mode switch: confirmed in §3.

---

## 5. Geometry-helper reuse — independently confirmed, no duplicated formula found

Read `buildWallBaseTabCouponModel()` (`index.html:2334-2352`) line by line and traced every call:

- `trayTabProfile(wallLengthMm, jointStyle)` — called once, output used for both walls and every slot row; never recomputed with different logic.
- `designTabPositions(wallLengthMm, tabProfile.count, tabProfile.widthMm)` — called once per row (`index.html:2346`), always with the *same* three arguments across all rows (only `wallLengthMm`/`tabProfile` are shared, row-invariant), so every row's slots align with the same wall's tabs — confirmed this is the *same* function signature used by `buildTrayModel()`'s `addWallSlots`, not a re-derived formula.
- `trayWallComponent(...)` — called exactly twice (`primaryWall`, `backupWall`, `index.html:2347`), passing the *same* `wallLengthMm`, `wallHeightMm` (fixed `22`), `tabProfile`, and `materialThicknessMm` to both — the only differing arguments are `id`, `name`, `x`, and `role`. **No independent tab geometry is computed here at all** — `trayWallComponent` internally calls `designWallPath`, exactly as it does for production trays.
- `trayRectComponent(...)` — called for the base plate and for every slot rectangle (`index.html:2347-2348`), identically to how `buildTrayModel()` builds the tray base and its wall slots.
- `traySlotWidthMm(materialThicknessMm, clearance.value)` — called once per row (`index.html:2342`), the sole source of every row's slot width.
- `designSvgDocument(widthMm, heightMm, shapes)` — called once in `buildWallBaseTabCouponDesignResult()` (`index.html:3065`), with `model.svgShapes` (an array of `.svg` strings pulled directly from the wall/plate/slot components) as the shape list — the exact generic anonymous-serializer contract, not routed through `serializeTrayCompatibilitySvg()` (which is tray-model-shaped) or `serializeDesignSvg()` (which is panel-shaped and structurally incompatible, as established in the design review).

I specifically searched the new function bodies for any inline arithmetic resembling tab-count selection, tab-width clamping, tab spacing, tab depth, or slot-width computation *outside* the named helper calls, and found none — every one of those five concerns is delegated to the shared functions, confirming **no duplicated production formula** exists in this implementation. This satisfies the task's explicit Major-severity trigger by its absence.

---

## 6. `traySlotWidthMm` extraction — independently confirmed exact and behavior-neutral

`function traySlotWidthMm(thicknessMm, clearanceMm) { return thicknessMm + clearanceMm; }` (`index.html:1945`) — a direct two-token arithmetic expression, exactly `thickness + clearance`, with no rounding, clamping, or side effects.

`buildTrayModel()`'s only changed line (`index.html:1976`): `slotWidthMm = traySlotWidthMm(thicknessMm, fitClearanceMm)`, replacing what I independently confirmed (via `git show 541cda5:index.html`, reading the exact prior line) was the inline expression `thicknessMm + fitClearanceMm` — **arithmetically identical**, confirmed by direct comparison of the extracted function's body against the removed inline text, not by inference.

**Independently re-derived, not merely read from a fixture:** Dice Tray default (`t=3, fitClearance=0.15`) — `traySlotWidthMm(3,0.15)=3.15`, matching both the pinned `dice.dimensions.slotWidthMm===3.15` fixture (`index.html:3436`, itself unchanged by this diff per §1) and my own hand-computation. The pinned production goldens `1726/51a55721` (Dice) and `1965/a55dda6e` (Divider) are asserted at three separate points in the fixture file (the tray production matrix, `index.html:3513-3514`, and the coupon-block cross-check at `index.html:4272`'s sibling `jointCouponDefault`-independent finger-preservation check) — all pointing at the same literal values, none of which changed from the baseline I independently confirmed via the full checkout in the runtime-baseline section above. No component count, element order, dimension, viewBox, slot dimension, anonymous-group signature, filename, or MIME check for trays falls inside any of the 17 touched hunks — **tray production drift is structurally ruled out**, not merely untested for.

---

## 7. Wall-piece geometry — independently confirmed

`primaryWall=trayWallComponent('wall-base-tab-primary-wall','Primary test wall',layoutMarginMm,layoutMarginMm,wallLengthMm,wallHeightMm,tabProfile,materialThicknessMm,'coupon-primary')` and `backupWall=trayWallComponent('wall-base-tab-backup-wall','Backup test wall',layoutMarginMm+wallLengthMm+layoutGapMm,layoutMarginMm,wallLengthMm,wallHeightMm,tabProfile,materialThicknessMm,'coupon-backup')` (`index.html:2347`) — **every argument except `id`, `name`, `x`, and `role` is identical between the two calls**: same `wallLengthMm`, same `wallHeightMm` (fixed `22`), same `tabProfile` object reference, same `materialThicknessMm`. This is not a mirror, not a scale transform, and not sourced from a different candidate — it is the literal same call with a shifted `x` and different labels, confirmed by direct reading, not inference from the result. Fixture `index.html:4273` independently checks this via `component.geometry.widthMm===80&&component.geometry.heightMm===25` for *every* component of `kind==='wall'` (not just one), which I independently hand-verified: `heightMm = wallHeightMm + thicknessMm = 22+3 = 25` (confirmed from `trayWallComponent`'s own `geometry` object construction, `index.html:1959`-region, unchanged from baseline).

Tab depth equals `materialThicknessMm` by construction — `trayWallComponent` passes `materialThicknessMm` directly as `designWallPath`'s `tabDepth` argument, the same call shape used for production tray walls (confirmed identical function, not a coupon-specific wrapper). Four-tab (`jointStyle==='finger'`) produces `count=4`; two-tab (`'tab-slot'`) produces `count=2` — both are pure `trayTabProfile()` outputs, independently re-derivable and cross-checked in fixture `index.html:4274` against a direct `trayTabProfile(80,'finger')`/`trayTabProfile(80,'tab-slot')` call.

**Physical usefulness of two identical walls, independently assessed:** the design report and result wording both call the second wall a "backup," and I agree this is the honest and correct framing — it is not a second independent test condition (both walls test the *same* profile against the *same* base plate's rows), it exists so a damaged or lost primary wall doesn't require regenerating and re-cutting the whole coupon. The wording at `index.html:1874` and `index.html:3316` ("the second wall is a backup") accurately describes this; nothing in the implementation or its text overstates the second wall as an independent measurement.

---

## 8. Base-plate geometry — independently hand-derived, not accepted from the fixture alone

Read `buildWallBaseTabCouponModel()`'s row-placement loop (`index.html:2345-2349`) and hand-simulated it for the default five candidates `0.20, 0.15, 0.10, 0.05, 0.00` at `materialThickness=3` (the shared default), independently of the fixture's own assertions:

```
slotWidth_i = 3 + candidate_i → [3.20, 3.15, 3.10, 3.05, 3.00]
rowOffset starts at plateMarginMm=6
row1: offset=6.00 → next rowOffset = 6.00+3.20+10 = 19.20
row2: offset=19.20 → next = 19.20+3.15+10 = 32.35
row3: offset=32.35 → next = 32.35+3.10+10 = 45.45
row4: offset=45.45 → next = 45.45+3.05+10 = 58.50
row5: offset=58.50 → next = 58.50+3.00+0 (last row, no trailing gap) = 61.50
plateHeightMm = 61.50 + 6 (bottom margin) = 67.50 mm
```

This is a genuine independent derivation from the stated rule (6mm top/bottom margins, 10mm gaps between rows, no gap after the last row, row height = that row's own slot width) — **not a restatement of the code's own arithmetic dressed up as verification**. I could not find a fixture that pins this exact `67.5` value numerically (the fixtures check row *count*, row *order*, and per-row *slot width*, but I did not find an assertion on `plateHeightMm` itself) — see Finding F1.

Five candidate rows top-to-bottom in entered order: confirmed both by the placement loop (`rows.forEach((row,index)=>{... row.offsetMm=designRound(rowOffsetMm); ...})`, which assigns strictly increasing `offsetMm` in array order, and array order is `parameters.clearances` in the order `parseJointCouponClearances` returned them — which preserves input order, confirmed independently in §9) and by fixture `index.html:4276`'s direct check of `wallCouponRows.map(row=>row.candidateText).join('|')==='0.20|0.15|0.10|0.05|0.00'`.

Each row contains one slot per wall tab: `row.slotPositions.forEach((position,index)=>{...slotComponents.push(trayRectComponent(...))})` (`index.html:2348`) — one `trayRectComponent` per position in `designTabPositions`'s output, i.e. exactly `tabProfile.count` slots per row. For four-tab default: `4×5=20` slots; confirmed by fixture `index.html:4274`'s `twoTabWallBaseCoupon.components.filter(component=>component.kind==='base-slot').length===twoTabWallBaseCoupon.metrics.rowCount*2` (the two-tab case, `2×5=10`) — I independently re-derived the four-tab case (`20`) from the same formula rather than trusting only the two-tab fixture, since no fixture I found asserts the four-tab total slot count directly (also noted in F1).

Slot length (`tabProfile.widthMm`) equals production tab width, slot width (`row.slotWidthMm`) equals `traySlotWidthMm(thickness,candidate)` — both confirmed directly in the `trayRectComponent(slotId,...,'base-slot',...,tabProfile.widthMm,row.slotWidthMm,...)` call (`index.html:2348`). Plate width equals `wallLengthMm` — confirmed at `basePlate=trayRectComponent(...,wallLengthMm,plateHeightMm,...)` (`index.html:2347`). Top/bottom margins `6mm`, inter-row gaps `10mm` — both are named local constants (`plateMarginMm=6`, `rowGapMm=10`, `index.html:2335`) consumed by the placement loop, not hard-coded inline at multiple sites. **Plate height is a computed value derived from the loop, not a guessed fixed total** — confirmed by the absence of any literal height constant assigned to `plateHeightMm`; it is always `designRound(rowOffsetMm+plateMarginMm)` where `rowOffsetMm` is the loop's running total.

---

## 9. Candidate ordering and identification — independently confirmed, including a non-sorted case

`parseJointCouponClearances()` (unchanged, confirmed in §1) pushes parsed values in token order via `values.push(...)` inside a `tokens.forEach(...)` loop — no sort anywhere in that function, confirmed by direct re-read. `buildWallBaseTabCouponModel()`'s `rows=(parameters.clearances||[]).map((clearance,index)=>({rowIndex:index+1,...}))` (`index.html:2342`) preserves array order via `.map()`, and the placement loop (§8) assigns strictly increasing `offsetMm` in that same order — so entered order flows through unmodified to physical top-to-bottom position, confirmed by full trace, not by re-running the code.

**Non-sorted input, independently checked:** I hand-traced the task's example `0.05, 0.20, -0.02, 0.00` through `parseJointCouponClearances` (token split → four values in that literal order, none violate the `-0.10..0.30` range or duplicate) and through the row-building `.map()` — the resulting `rows` array is `[0.05, 0.20, -0.02, 0.00]` in that exact order, confirmed no sort step exists anywhere between parsing and row assignment. I did not find a dedicated fixture using this specific non-sorted input, but the code path is identical regardless of input ordering (there is no order-dependent branch), so the existing default-order fixture (`0.20,0.15,0.10,0.05,0.00`, itself already non-*numerically*-sorted in one sense — it happens to be descending, which is a coincidence of the chosen defaults, not evidence the code sorts) exercises the same order-preservation code path a scrambled input would (noted as a Minor coverage gap, F2, not a functional risk given the code trace).

Result-panel text shows the mapping: `designMetric('Candidate rows', (result.metrics.clearances||[]).map(item=>item.text).join(', ')...)` (`index.html:3231`) — prints candidates in the same array order as generated, i.e. entered order, confirmed directly. The assembly note (`index.html:3316`) instructs "Test the loosest row first," which is accurate *if* the user enters values loosest-first as the shipped default does (`0.20` first) — the note does not itself re-sort or claim the app enforces loosest-first ordering, it is user guidance consistent with the shipped default, not a code-level guarantee; this is an honest framing, not an overclaim.

Internal IDs identify candidate index and value: `wall-base-tab-row-01`, `…-row-01-slot-01` etc. (`index.html:2348`), plus rich `candidateRows` metrics (`id, rowIndex, candidate, candidateText, slotWidthMm, slotIds, slotPositions`, `index.html:2352`) and per-slot `data-*`-style metadata (`rowId, rowIndex, candidate, candidateText, tabIndex`, same line) — confirmed sufficient for fixture/audit identification without parsing SVG.

No engraved/scored labels in production SVG: confirmed by `designSvgDocument()`'s generic contract (no title/text/id support at all) and independently by fixture `index.html:4271`'s explicit regex-negative-assertion against `id=`, `<title`, `<metadata`, `<text`, `id="score"`. The lack of physical labels is disclosed honestly in both the form help text (`index.html:1852`) and the assembly note (`index.html:3316`, "Candidate rows are top-to-bottom in the entered order... Test the loosest row first").

---

## 10. Layout and non-overlap — independently confirmed, including two profile/length variants

Read the explicit pairwise overlap/separation check (`index.html:2350`): `for(let first...) for(let second...) if(designBoxesOverlap(...)||!designBoxesSeparatedBy(...,layoutGapMm)) errors.push(...)` — runs over the **three major panels** (primary wall, backup wall, base plate), using the same `designBoxesOverlap`/`designBoxesSeparatedBy` utilities used throughout the rest of the app (confirmed generic, template-agnostic, unchanged by this diff). `layoutMarginMm=10`, `layoutGapMm=10` — both confirmed as named constants (`index.html:2335`), matching the app-wide 10mm convention.

**Hand-verified bounding boxes for the default (four-tab, 80mm) configuration:**
- Primary wall: `x=[10,90]`, `y=[10,35]` (10 to 10+80; 10 to 10+22+3)
- Backup wall: `x=[100,180]` (10+80+10=100 to 100+80), `y=[10,35]` — separated from primary wall by exactly the 10mm gap on the x-axis, confirmed `100-90=10`
- Base plate: `y` starts at `plateY=designRound(layoutMarginMm+wallHeightMm+materialThicknessMm+layoutGapMm)=10+22+3+10=45`; `x=[10,90]` (`wallLengthMm=80` wide, same x-origin as the primary wall); `y=[45,45+67.5=112.5]` — separated from both walls by exactly 10mm on the y-axis (`45-35=10`), confirmed
- No overlap between the two walls (disjoint x-ranges) or between either wall and the plate (disjoint y-ranges) — confirmed by direct coordinate arithmetic, independent of the code's own overlap check

All slot rectangles are confirmed to fall within the base plate's `x`-range by construction (`layoutMarginMm+position` where `position` comes from `designTabPositions(wallLengthMm,...)`, which by definition never exceeds `wallLengthMm−tabWidth`, confirmed via the formula `(length-tabWidth)*(index+1)/(count+1)`, always `< length-tabWidth`) and within the plate's `y`-range by construction (`plateY+row.offsetMm`, where `row.offsetMm` is bounded by the same loop that computes `plateHeightMm`, so no row can exceed the plate by construction, not merely by a separate check).

**Finite geometry:** `components.some(component=>Object.values(component.geometry||{}).some(value=>typeof value==='number'&&!Number.isFinite(value)))||!Number.isFinite(widthMm)||!Number.isFinite(heightMm)` (`index.html:2351`) — checks every component, not just the three panels. Fixture `index.html:4277` independently re-checks finiteness on `translatedBounds` values for the panel set. I traced this against the two-tab profile and a non-default wall length (the validation-rejection fixture at `index.html:4278` uses `jointCouponWallLength:'20'`, which correctly *fails* validation rather than producing degenerate geometry, confirming the non-overlap machinery is never reached with invalid inputs) — no path to negative dimensions, zero-length paths, or NaN/Infinity was found in either the finger or tab-slot branch.

---

## 11. Validation boundaries — independently confirmed, including the three-layer distinction the task asks about

Traced three genuinely separate validation layers, confirmed by reading each in isolation:

1. **Candidate range/count/duplicate validity** — enforced entirely inside `parseJointCouponClearances()` (unchanged, §1), before `buildWallBaseTabCouponModel()` ever runs: `-0.10..+0.30` range, 2-decimal precision, 3–8 count, duplicate rejection. Confirmed this is the *same* parser instance, not a reimplementation, by the unchanged hunk-range proof.
2. **Positive physical slot width** — `if(!Number.isFinite(row.slotWidthMm)||row.slotWidthMm<=0) errors.push(...)` (`index.html:2343`), a distinct `if` branch from:
3. **Minimum remaining plate web** — `else if(row.slotWidthMm<requiredWeb-1e-9) errors.push(...)`, using `requiredWeb=Math.max(.5,materialThicknessMm*.25)` (`index.html:2335`), the same formula used elsewhere in the file (`buildJointCouponModel`, `buildBoxModel`, etc., confirmed by grep — all four use the identical `Math.max(.5, t*.25)` expression).

These are genuinely distinct `if`/`else if` branches with distinct error messages, confirmed by direct reading — not a single collapsed check, satisfying the task's explicit request to verify this distinction exists.

Additional validation confirmed by direct reading: thickness `<0.1` rejected (`index.html:2336`); `jointStyle` not in `['finger','tab-slot']` rejected (`index.html:2337`); `wallLengthMm<=0` rejected (`index.html:2338`); the tab non-overlap inequality `wallLengthMm+2*materialThicknessMm<tabProfile.widthMm*(tabProfile.count+2)` mirrors (does not call) `trayModelValidationErrors()`'s equivalent inequality, confirmed structurally identical in shape by direct comparison of both expressions. Every failure path returns `errors.push(...)` — I found no `Math.max`/`Math.min` clamp applied to any *user-facing geometry parameter* after validation (the only clamps in the reused helpers, e.g. `trayTabProfile`'s tab-width clamp, are the same pre-existing production clamps, not new silent-acceptance logic).

Fixture `index.html:4278` exercises exactly two of these boundary cases directly (short wall; thin material + negative candidate collapsing slot width to exactly `0`, hand-verified: `traySlotWidthMm(0.10,-0.10)=0.00`, correctly caught by the `<=0` branch, not the `requiredWeb` branch) — confirming both the "short wall" and "non-positive slot width" paths are exercised, though not the `requiredWeb`-but-still-positive collapse case specifically (noted as F2).

---

## 12. SVG contract — independently confirmed by direct structural inspection, not the hash alone

Read `buildWallBaseTabCouponDesignResult()` (`index.html:3062-3067`) and traced `designSvgDocument(widthMm,heightMm,shapes)` (unchanged, confirmed §1): produces exactly `<?xml…?>\n<svg…viewBox="0 0 W H">\n  <g fill="none" stroke="#ff0000" stroke-width="0.1">\n    <shape1/>\n    <shape2/>\n…\n  </g>\n</svg>\n` — **one** anonymous group, no conditional score-group insertion anywhere in this function (unlike `serializeDesignSvg`, which this path never calls). Fixture `index.html:4271` independently confirms this structurally via regex against the actual generated SVG string (not the model object) — I re-derived the same conclusion by reading `designSvgDocument`'s template literal directly, an independent path to the same answer.

Pinned golden `1551 / d9ffc278` (`index.html:4272`) matches both reports' claims and the task's expected value exactly — I did not recompute the FNV-style hash by hand (impractical without executing JS), but I did independently verify the *structural* claims around it (piece count, dimensions, no-IDs/no-titles) through direct source reading rather than accepting the hash as the only proof, consistent with the task's explicit instruction not to rely on the hash alone.

Filename: unchanged pattern, confirmed via fixture `index.html:4282`'s `wallCouponLive.filename===\`l8-joint-fit-coupon-${today()}.svg\`` — the template key stays `joint-fit-coupon` regardless of `couponType`, so the filename generator (unchanged, confirmed §1) needs no new branch. MIME `image/svg+xml;charset=utf-8` — same fixture, same confirmation. Preview/download identity — same fixture, drives the real `trayLivePreviewDownloadFixture()` DOM-intercept helper (confirmed unchanged, pre-existing, reused rather than reimplemented for this coupon type).

---

## 13. Promotion boundary — independently confirmed, the second-highest-stakes check

The single promotion-button gate (`index.html:3329`): `jointCoupon && result.valid && result.metrics?.couponType !== 'wall-base-tab'` — confirmed to read `result.metrics.couponType` (the metrics of the *actually-computed, actually-rendered* result object passed into `designResultsHtml(result)`), not `designDraft.couponType` (which could theoretically be stale relative to what's on screen if a render were interrupted) — this exactly matches the design document's own recommendation, and independently confirmed correct: `buildJointCouponModel()`'s finger-edge result (unchanged, §3) never sets `metrics.couponType` at all, so `undefined !== 'wall-base-tab'` evaluates `true`, correctly preserving promotion for every historical finger-edge result without that function needing any change. `buildWallBaseTabCouponModel()` explicitly sets `metrics.couponType:'wall-base-tab'` in **both** its valid and invalid-result return branches (`index.html:2344` and `2352`), so the gate is correctly closed even for an invalid wall-to-base attempt (moot, since `result.valid` is already false there, but confirms no accidental gap).

Stale-form-state exposure: since the gate reads from the `result` parameter (computed once per render from whatever `designDraft` looked like at that render call, confirmed via `buildDesignResult(designDraft)` call sites), and `designResultsHtml` always receives a freshly-built `result` alongside the form re-render (confirmed by tracing `refreshDesignPreview()`, unchanged, which calls `buildDesignResult(designDraft)` immediately before `designResultsHtml(result)` on every relevant event), there is no code path I could find where a displayed Wall-to-base result's markup could show a Finger-edge promotion button — the two are always computed and rendered together, atomically, from the same function call.

`buildJointCouponPromotionCandidate()`, `promotionStatusRules()`, `openJointCouponPromotion()`, `promotionCanonicalize()`, `promotionSignature()`, `productionSettingDesignApplicability()` — **zero hunks touch any of these**, confirmed by the exhaustive 17-hunk enumeration in §1 (all are defined well past old-line 4225, before old-line 8725, entirely within the untouched gap). Since the promotion button is the *only* rendered entry point into `openJointCouponPromotion()` (confirmed by full-file grep for that function name — one definition, one caller, at the button's `onclick`), and that button is now provably absent from wall-to-base markup, **no wall-to-base winner can reach `buildJointCouponPromotionCandidate()` through any UI path**, and consequently no value can reach `fitSettings.fingerJointClearanceMm` from this feature. No evidence, Production Settings, fingerprint, or Designs-application code changed — confirmed structurally, not by absence of suspicion.

---

## 14. Storage isolation — independently confirmed

Fixture `index.html:4281-4282` captures `localStorage.getItem(STORAGE_KEY)` and `JSON.stringify(backupObject())` both before and after generating a wall-to-base coupon, previewing it, and downloading it (via the real `trayLivePreviewDownloadFixture()`), asserting byte-identical values on both sides. I independently confirm this is a meaningful check (not merely "nothing was written because nothing was expected to write") because `backupObject()` is unchanged (§1/§17) and contains no `couponType`/`jointCouponWall*` reference by construction — those keys live only on the module-local `designDraft`, which `backupObject()` never reads (confirmed by reading `backupObject()`'s field list, unchanged, and finding no `designDraft` reference in it at all). No new schema field, storage key, draft record, evidence record, or Production Settings field is created anywhere in the diff — confirmed by the exhaustive hunk enumeration showing zero touches to any storage/schema/evidence/production-setting function.

---

## 15. Browser workflow

**Not independently verified live** — no browser runtime available in this environment, the same `file://`-navigation-denied constraint disclosed in every recent audit in this chain. All 13 steps in the task's browser-workflow checklist are instead covered by the source-level and fixture-level verification in §§2–14 above (mode default, field replacement, promotion absence, download identity, filename/MIME, session round-trip on mode-switch), which I consider strong evidence given the fixtures drive the *actual* rendering and event-binding functions (`designResultsHtml`, `refreshDesignPreview`, `bindDesignPreviewActions` via `trayLivePreviewDownloadFixture`) rather than reimplementations — but this is not a substitute for an actual mouse-driven session, and I am disclosing that gap rather than fabricating a browser observation.

---

## 16. Fixture quality

All 11 new assertions (`index.html:4269-4282`, listed in §1's runtime-baseline section) were read directly, not sampled. Classification:

- **Independent**: 4269 (byte-identity against a golden defined elsewhere, outside the diff), 4270 (error-message-specific rejection), 4271 (structural regex against the actual SVG string), 4274 (calls `trayTabProfile()` directly and compares — genuine builder-reuse proof, not visual similarity), 4276 (calls `designTabPositions()` directly and compares, plus literal hand-derivable slot widths `3.2`/`3`), 4277 (reuses the generic `designBoxesOverlap`/`designBoxesSeparatedBy` utilities, not the implementation's own logic), 4278 (validation-rejection, hand-verifiable), 4280 (drives real `designResultsHtml()` rendering and checks for real markup substrings), 4282 (drives the real DOM-intercept download helper).
- **Partially circular**: 4272 (the golden itself is self-referential on first capture — inherent to any "pin the default output" fixture, mitigated by 4273/4274/4276 independently checking structure and formulas rather than only the hash), 4273 (checks the model's own component objects against literal expected values `80`/`25`/`4` — reasonably independent since it checks *structure*, not a re-run of the *generating* formula, but shares the same model instance as the golden).
- **Circular**: none found among the 11.

**Coverage gaps found by this audit, not overclaimed by the fixtures themselves** (both Minor, §"Findings"): no fixture pins the exact default `plateHeightMm` (`67.5`) numerically, and no fixture uses a genuinely non-sorted candidate list to prove order-preservation beyond what the code trace already establishes. No fixture name I read overstates what it proves — each name (e.g. "Wall-to-base coupon uses exactly two equal wall pieces and one multi-row base plate") accurately describes the assertion body I read alongside it.

This is a right-sized set for a coupon feature — consistent with the architecture challenge review's explicit warning against a full product-style matrix for coupons, and I found no actual coverage gap large enough to justify one.

---

## 17. Protected boundaries

Every function in the task's protected list was checked against the exhaustive 17-hunk enumeration from §1 (proven complete via the numstat cross-check, not sampled): `STORAGE_KEY`, `SCHEMA_VERSION`, `persist()`, `backupObject()`, `replaceData()`, `mergeData()`, `normalizeProductionEvidence()`, `normalizeProductionSetting(s)`, `productionSettingDesignApplicability()`, `buildJointCouponPromotionCandidate()`, `openJointCouponPromotion()`, `promotionCanonicalize()`, `promotionSignature()`, `promotionStatusRules()`, `buildJointCouponModel()`, `parseJointCouponClearances()`, `trayTabProfile()`, `designTabPositions()`, `designWallPath()`, `trayWallComponent()`, `trayRectComponent()`, `trayModelValidationErrors()`, `serializeDesignSvg()`, `serializeTrayCompatibilitySvg()`, `designSvgDocument()`, every Finished View renderer, QR stand, Hanging sign, and Material Test/Test Grid/Project/Inventory/Pricing/import-export/storage-recovery code — **all fall entirely outside every one of the 17 touched hunk ranges**. The approved `buildTrayModel()` helper-call substitution (§6) is confirmed to be the *only* intentional edit to existing tray production internals, matching the task's stated expectation exactly.

---

## 18. Documentation accuracy

Read the actual `README.md` diff directly (not summarized from the implementation report). It accurately describes: the two coupon modes and their exact labels; the wall-to-base defaults' behavior (top-to-bottom entered order); "two identical test walls and one multi-row base plate"; shared tray-helper reuse is implied by "uses the selected four- or two-tab tray wall profile" (consistent with, though less explicit than, this audit's confirmed reuse chain); the anonymous SVG contract is not explicitly named in the README prose but the "no engraved labels" and "cannot be promoted yet" statements are accurate and consistent with what I confirmed in source; "any physical winner must be recorded manually" matches §13's confirmed promotion gating exactly; unchanged Finger/Dice/Divider goldens are not restated in this specific paragraph but are correctly stated as unchanged elsewhere in the same README diff; software-vs-physical limits are stated almost verbatim to what I independently confirmed the code enforces ("not proof of a full tray, corner strength, glue strength, warping, cumulative assembly error, Divider retention, or production safety"); fixture totals (`887`/`1679`) match my independent recomputation exactly. I found no statement in either the README or the implementation report that overstates physical fit evidence — every disclosed limitation in the documentation matches a corresponding, independently-confirmed limitation in the actual code (no promotion path, no labels, geometry-only validation).

---

## Findings

| # | Severity | Function / path | Scenario | Consequence | Recommendation | Fixture coverage |
|---|---|---|---|---|---|---|
| F1 | Minor | `runDesignGeometryFixtures()`, wall-to-base fixture block | No fixture pins the exact default `plateHeightMm` (`67.5`, independently hand-derived in §8) or the four-tab total slot count (`20`, independently hand-derived in §8) numerically | The underlying formula is correct (independently re-derived by hand and matches the implementation) and the row-count/per-row-slot-count are separately checked, so this is a coverage-completeness gap, not an observed defect | Add one literal assertion pinning `plateHeightMm===67.5` for the default draft, and one pinning the four-tab total slot count (`20`) alongside the existing two-tab check | Partially — row height/count and two-tab slot total are checked; the derived plate height and four-tab total are not pinned directly |
| F2 | Minor | `runDesignGeometryFixtures()`, wall-to-base fixture block | No fixture exercises a genuinely non-numerically-sorted candidate list (e.g. `0.05, 0.20, -0.02, 0.00`) to prove order-preservation beyond what the code trace establishes; separately, no fixture exercises the `requiredWeb`-collapse branch specifically (only the `<=0` branch is exercised) | The code trace (§9, §11) independently confirms both behaviors are correct by construction (no sort step exists; the two validation branches are genuinely distinct `if`/`else if`), so this is a coverage gap, not an observed defect | Add one fixture with a scrambled candidate order, and one with a candidate that produces a small-but-positive slot width below `requiredWeb` specifically | Not covered directly; covered indirectly by direct code-path tracing in this audit |

No Blocker or Major findings.

---

## Required conclusion

```text
SAFE TO COMMIT
```

Every geometric claim in the implementation report was independently re-derived from the raw source — by hand, not by re-running the fixture that also asserts it — and matched exactly: two identical wall pieces differing only in position/label, one base plate whose height is a genuine running-sum computation (independently hand-verified at 67.5mm for the default draft) rather than a guessed constant, slot counts/widths/positions all traced to the shared `trayTabProfile`/`designTabPositions`/`traySlotWidthMm` helpers with no duplicated formula found anywhere. Production isolation for both the Finger Coupon (unchanged, confirmed byte-identical by direct function-body comparison) and the tray production contract (unchanged, confirmed via a full baseline checkout, not just the working-tree diff) is airtight. Promotion is correctly and minimally gated on the actually-rendered result's own metrics, confirmed via a complete, numstat-verified enumeration of every touched line in the file to show no promotion-adjacent function was edited. The two Minor findings are fixture-completeness gaps this audit closed independently through direct hand-derivation, not observed defects — neither blocks committing.

---

*Focused audit performed read-only at commit `541cda5` with uncommitted `index.html`/`README.md` changes, cross-checked against a full baseline checkout (`git show 541cda5:index.html`). No application file was modified, staged, committed, or pushed. Unrelated untracked files were left untouched.*
