# Dice Tray Finished View — Focused Independent Audit

**Primary implementation report:** `docs/DESIGNS_DICE_TRAY_FINISHED_VIEW_IMPLEMENTATION_2026-07-17.md`
**Also read for context:** `docs/DESIGNS_FINISHED_VIEWS_ARCHITECTURE_REVIEW_2026-07-17.md`, `docs/DESIGNS_TRAY_MODEL_EXTRACTION_ARCHITECTURE_REVIEW_2026-07-17.md`, `docs/DESIGNS_TRAY_MODEL_PHASE2_PRODUCTION_SWITCH_IMPLEMENTATION_2026-07-17.md`, `docs/DESIGNS_TRAY_MODEL_PHASE2_PRODUCTION_SWITCH_FOCUSED_AUDIT_2026-07-17.md`
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `edd07d5` — *Switch trays to structured production model*
**Reviewed:** Uncommitted changes in `index.html` (81/9), `README.md` (2/2)
**Date:** 2026-07-17
**Mode:** Read-only. No browser runtime available (the sandboxed browser pane denies `file://` navigation, the constraint disclosed in every recent audit in this chain). Every claim below was checked by reading the actual function bodies directly and hand-deriving expected values from `buildTrayModel()`'s own formulas, not by trusting the implementation report's narrative or re-running the fixtures that assert the same behavior.

---

## Verdict

**SAFE TO COMMIT.** No Blocker or Major findings. Production isolation is structurally airtight — `buildDiceTrayFinishedViewSvg()` and `trayFinishedViewProjection()` are never referenced anywhere in `serializeTrayCompatibilitySvg()`, `downloadCurrentDesignSvg()`, or the `svg` field computed inside `buildTrayDesignResult()`; the `finishedView` property is attached to the result object via a conditional spread that happens *after* `svg` is already fully computed, and only for the `dice-tray` template. The projection function (`trayFinishedViewProjection`) reads exclusively from the model's explicit `orientation`/`inputs`/`dimensions` objects — never from `layout` (sheet-packing x/y) or any component's path/SVG data — so Front/Back/Left/Right orientation is provably not inferred from Cut Layout coordinates. Divider Tray is excluded by an explicit `model.template !== 'dice-tray'` guard at the top of the projection function, confirmed structurally in three independent places (the projection guard itself, the selector's `dice`/`validForView` check, and `designResultsHtml`'s `dice` flag requiring both the template *and* a truthy `result.finishedView`). Fixture counts (11 new Dice-Finished-View assertions, 233 Tray-suite total, 861 Designs geometry, 1653 grand total) all recompute exactly. No findings rise above Minor.

---

## Runtime baseline

**Live execution unavailable** — same disclosed constraint as every recent audit in this chain. Totals were independently recomputed via static source analysis.

| Group | Task's expected | Independently recomputed | Method |
|---|---:|---:|---|
| Tray model/live production fixtures | 233 / 0 | **233 / 0** | See recomputation below |
| Designs geometry | 861 / 0 | **861 / 0** | See recomputation below |
| Complete suite | 1653 / 0 | **1653 / 0** | See reconciliation below |
| Designs production application / Test Grid machine identity / Evidence Promotion / Production Settings / Storage recovery | 118 / 18 / 58 / 66 / 8 | **unchanged, inherited** | Hunk-range method below |

**Hunk-range method** (`git diff -U0 -- index.html`): the diff resolves to twelve hunks, every one falling between old-line 2015 and 3291 (new-lines 2016–3392) — the new `trayFinishedViewProjection()` function, the eight-line `buildDiceTrayFinishedViewSvg()` insertion point (the function itself is a single very long line, confirmed by reading it directly), five small edits to `designPreviewModeForTemplate()`/`designPreviewSelectorHtml()`/`designResultsHtml()` (adding the `dice-tray` case alongside the existing `finger-box` one, not rewriting them), and two new-fixture insertion blocks (`diceFinishedViewBrowserFixture()` at old-line 3291/new-line 3330, and the 11 Dice-Finished-View assertions plus supporting fixture helper at old-line 3320/new-line 3376). Summing every hunk's insertion/deletion count reproduces the numstat's 81/9 exactly, confirming this enumeration is complete. This proves Finger Box, Sliding Lid Box, Drawer Cabinet, `buildTrayModel()`'s production geometry, `serializeTrayCompatibilitySvg()`, persistence, backup/import, Production Settings, Evidence Promotion, and Test Grid machine-identity code are all byte-identical to `edd07d5`.

**Fixture recomputation:** `runTrayModelFixtures()` (`index.html:3347-3421`) has 44 static `add(` occurrences total, of which 8 sit inside the 23-entry `matrix.forEach` loop body (`index.html:3355-3363`) and 1 sits inside the 14-entry `invalidCases.forEach` loop body (`index.html:3409`) — non-loop static `44−9=35`; runtime total `35 + (8×23=184) + (1×14=14) = 233`, exact match. `runDesignGeometryFixtures()` (`index.html:3423-4180`) has 536 static `add(` occurrences in its own body (a *disjoint* text range from `runTrayModelFixtures`, confirmed — the two functions don't overlap), of which 16 sit inside its six pre-existing loop bodies (unchanged from the prior audit's count, confirmed via the hunk-range method showing none of those loops were touched); non-loop static `536−16=520`; runtime (excluding Tray) `520+108=628`. `runDesignGeometryFixtures()` calls `runTrayModelFixtures()` once and `results.push(...trayModelFixtures.results)`s the 233 in directly (`index.html:3426-3427`) — so its full return is `628+233=861`, exact match, and this **628** figure independently cross-validates against the *same* 628 total I derived in the prior (Sliding-Lid) audit in this chain, at a point before any Tray fixtures existed at all — strong corroboration from an entirely separate audit session. **Complete suite**: the Phase 2 report's own documented baseline was 1642/0 (with Designs geometry at 850, itself `628+222` under the old 222-count Tray suite). This diff's only fixture-count change is Designs geometry growing by exactly `861−850=11` (matching the 11 new assertions, confirmed the sole growth via the hunk-range method showing no other fixture-group function touched): `1642+11=1653`, exact match to the task's expected total.

---

## 1. Authoritative model projection — independently confirmed

Read `trayFinishedViewProjection(model)` (`index.html:2016-2036`) in full. It receives only `model` (the object `buildTrayModel()` returns) and returns `null` immediately if `!model?.valid || model.template !== 'dice-tray'` — this is the Divider Tray exclusion, confirmed structurally at the very top of the function, not inferred elsewhere. It reads exclusively from `model.orientation` (`.frontWallId`/`.backWallId`/`.leftWallId`/`.rightWallId`/`.interiorFace`), `model.inputs` (`.insideWidthMm`/`.insideDepthMm`/`.wallHeightMm`/`.materialThicknessMm`/`.fitClearanceMm`/`.jointStyle`), `model.dimensions` (`.baseOutsideWidthMm`/`.baseOutsideDepthMm`/`.widthTabProfile`/`.depthTabProfile`), and `model.metrics.compartmentCount` — grepped the entire function body for `.layout`, `.operations`, `.svg`, and `.path` and found zero occurrences; it never touches the sheet-packing `layout` object or any component's geometry/path data. The only per-component lookup is `components[wallId].baseSlotIds` and `.kind` — an ID array and a type string, never geometry. Every returned field is a defensive copy (`{...wallIds}`, `{...model.dimensions.widthTabProfile}`, `[...components[wallId].baseSlotIds]`) — the function never returns a live reference into `model`, and never assigns anything back onto `model` — confirmed no mutation is possible even by an accidental aliasing bug. The function does not reference `designDraft`, any DOM form element, `localStorage`, `state.machineProfile`, or `designPreviewMode` anywhere — it is a pure function of its single `model` argument. An additional safety guard (`index.html:2019`) rejects the projection if any of the four wall IDs doesn't resolve to an actual `kind:'wall'` component, returning `null` rather than a malformed projection.

The projection's field set exactly matches the required minimal-semantics list: inside width/depth, wall height, material thickness, fit-clearance, base outside width/depth, joint style, tab-profile metadata, the four canonical wall IDs, and each wall's base-slot-ID relationships — no more, no less.

---

## 2. Live result integration — independently confirmed

Read `buildTrayDesignResult(normalized)` (`index.html:2977-2984`) in full. `const svg = serializeTrayCompatibilitySvg(model)` is computed and the function's `svg:errors.length?'':svg` return field is fixed *before* the `finishedView` property is even considered — the conditional spread `...(errors.length ? {} : model.template === 'dice-tray' ? { finishedView: trayFinishedViewProjection(model) } : {})` (`index.html:2983`) is appended afterward and has no path back into the `svg` computation. This means: (a) `svg` has zero dependency on `finishedView`/`trayFinishedViewProjection` at the source level — not merely "happens not to include it," structurally cannot; (b) an invalid result (`errors.length` truthy) gets `{}`, i.e. no `finishedView` key exists on the object at all, not even a `null` one — confirmed this is a stricter guarantee than "empty projection," it's "key absent"; (c) Divider Tray (valid, but `model.template !== 'dice-tray'`) also gets `{}` — no `finishedView` key, confirmed at this exact call site, independently of the projection function's own internal Divider guard (defense in depth: two independent places both exclude Divider). Anonymous `Shape N` panel projection (`panels:validation.elements.map(...)`) is computed identically to the pre-existing Phase 2 code, confirmed unchanged by the hunk-range method — this diff's only edit to `buildTrayDesignResult` is the trailing `finishedView` spread, appended after every other field.

---

## 3. Preview-mode behavior — independently confirmed

`designPreviewModeForTemplate()` (`index.html:3159-3163`) whitelists `['finger-box','dice-tray'].includes(template) && mode==='finished-view'` — `dice-tray` was added alongside the existing `finger-box` entry; `divider-tray` does not appear anywhere in this function, confirmed by reading it in full. `designPreviewSelectorHtml()` (`index.html:3165-3178`) computes `dice = designDraft.template === 'dice-tray'` (never `'divider-tray'`) and gates `validForView` for Dice specifically on `!!result.finishedView` (`index.html:3168`) — a second, independent Divider-exclusion mechanism, since a Divider Tray result never carries a `finishedView` key at all (confirmed in §2). `designResultsHtml()` (`index.html:3185`) computes `dice = designDraft.template === 'dice-tray' && result.finishedView` — a *third* independent check requiring both conditions. Mode-reset-on-invalid (`index.html:3170-3172`, `designPreviewMode='cut-layout'` when the current mode is a finished-mode but `validForView` is false) is the same mechanism already verified in the prior Sliding-Lid audit, now confirmed to also cover Dice Tray. Finger Box, Sliding Lid Box, and Drawer Cabinet's own branches in all three functions are unmodified lines (confirmed via the hunk-range method — the diff only *adds* the Dice-Tray case to existing conditionals, never rewrites the Finger/Sliding/Cabinet branches). Mode changes only ever call `refreshDesignPreview()`, which rebuilds from the same `designDraft` and replaces only `designResults.innerHTML` — `designPreviewMode` is a plain module-level `let` (confirmed unreferenced by `persist()`/`backupObject()` via the hunk-range method, since those functions aren't in the diff at all).

---

## 4. Canonical orientation — independently confirmed against structural model data, not labels

`buildTrayModel()` (`index.html:1963-2011`) constructs `orientation:{interiorFace:'top', frontWallId:'wall-front', backWallId:'wall-back', leftWallId:'wall-left', rightWallId:'wall-right', ...}` (`index.html:2005`) as an **explicit, separate object from `layout`** — the sheet-packing `x`/`y` coordinates live only inside `layout.items[].geometry`, a structurally distinct part of the model the projection function never touches (confirmed in §1). This is a stronger guarantee than "the renderer happens to use the right labels" — orientation and sheet-position are different fields in the authoritative model itself, so there is no code path by which the renderer could accidentally read a layout position and mislabel it as Front/Back/Left/Right. `buildDiceTrayFinishedViewSvg()` renders the four walls at fixed screen positions (top=Back, bottom=Front, left=Left, right=Right, `index.html:3130`) keyed off `view.wallIds.{front,back,left,right}` from the projection — the actual on-screen position of a wall never depends on which sheet row `layoutDesignPanelRows`-equivalent packing placed it in (Dice Tray uses `layout.items`, not the panel-row system, and the renderer never reads `layout` at all). Repeated renders are deterministic (see §8/§10) so Front/Back cannot swap between calls. Finger (`jointStyle:'finger'`) and Tab-and-slot (`'tab-slot'`) settings both flow through the same `orientation` object unchanged — joint style only affects `tabProfiles`/label wording, never the wall-ID-to-screen-position mapping. Square (`trayWidth===trayDepth`) and long-narrow configurations use the same fixed four-position layout, scaled by `scale=min(274/width,142/depth)` (`index.html:3127`) — a uniform scale factor applied to both axes, so orientation cannot invert or rotate at extreme aspect ratios (confirmed the formula never swaps width/depth roles).

---

## 5–7. Visual assembly honesty, wall/slot relationship, joint-style wording — independently confirmed

Read the complete SVG-template string in `buildDiceTrayFinishedViewSvg()` (`index.html:3130`) character by character for prohibited content. Confirmed present: exactly four wall `<rect>`s (`dice-tray-wall-back`/`-front`/`-left`/`-right`, one each), one interior cavity rect, one base rect plus two decorative "shallow depth cue" edge polygons (`dice-tray-base-front-edge`/`-right-edge` — a restrained isometric-style visual hint, not a claim of measured depth), four orientation labels plus "Open top," inside-usable dimensions, wall height, outside footprint, and a "Screen-only assembly preview" notice. Confirmed **absent** (grepped the full template string for each): any `lid`/`liner`/`felt`/`leather`/`dice`/`divider`/`compartment`/`magnet`/`hinge`/`handle`/`fastener`/`glue`/`dovetail` reference, and any element class/ID suggesting hidden joinery.

**No visual tab rhythm is rendered at all** — the four walls are drawn as plain solid rectangular bands (`class="dice-tray-wall"`) with no tooth/notch path geometry; the wall-to-slot relationship is exposed only as a `data-base-slot-ids` attribute (text data, not a redrawn visual pattern) sourced directly from the projection's `wallSlotRelationships` (itself sourced from the model's real `wall.baseSlotIds`, confirmed populated in `buildTrayModel()` at `index.html:1986`: `wall.baseSlotIds = base.wallSlotIds.filter(slotId => slotId.includes(`-${wall.role}-`))`, a real filter over the model's own generated slot IDs, not a fabricated list). Since no visual tab pattern exists, there is no "second tab algorithm" to independently verify — the task's conditional check ("if present, prove it is supplied by semantic model metadata") resolves by absence.

Joint-style wording (`index.html:3129`): `jointLabel = view.jointStyle==='finger'?'Four-tab wall profile':'Two-tab wall profile'`. I confirmed the literal substring `'finger-jointed'` never appears anywhere in the renderer's output, and the corresponding fixture (`index.html:3382`) explicitly asserts `!/finger-jointed corner|interlocking corner/i.test(diceFinishedSvg)` — both settings are distinguishable (different label text) without ever claiming interlocking corners, dovetails, or verified joinery. "Walls align with base slots" (`index.html:3129`) is a factually accurate description given the real, model-derived `baseSlotIds` relationship — not an assembly-quality or fit claim.

---

## 8. Dimensions and determinism — independently confirmed

Every dimension label in `buildDiceTrayFinishedViewSvg()` reads from the `view` object destructured from `result.finishedView` — `view.insideWidthMm`/`.insideDepthMm`/`.wallHeightMm`/`.baseOutsideWidthMm`/`.baseOutsideDepthMm` — the same values used for the geometric scale computation (`scale=min(274/width,142/depth)`, `index.html:3127`), so there is no separate "display" code path that could drift from the geometry driving the drawing. The overall `viewBox` is a fixed `"0 0 480 330"` (`index.html:3130`) regardless of tray size — only the internal `scale` factor changes to fit content within that fixed canvas, and the *numeric labels* always show the true `mm` values from `view`, never a value back-derived from the rendered pixel size. The function has no reference to `Math.random()`, `Date`, or any session variable — confirmed by reading its full body; it is a pure function of `result.finishedView`. The extensive validation guard (`index.html:3126-3128`) rejects non-finite, negative-or-zero, or malformed inputs before any geometry is computed, returning `''`.

---

## 9–10. Boundary/proportion cases and production isolation — independently confirmed

The boundary-scenario fixture (`index.html:3383-3384`) covers finger, tab-slot, square (`160×160`), long-narrow (`80×360`), wide-shallow (`500×60`), minimum-valid (`42×48`), large (`500×420`), shallow-wall (`height:10`), tall-wall (`height:180`), zero-clearance, and thicker-material (`4.5mm`) — eleven configurations, each asserted finite with exactly four rendered walls. This is a genuinely broader spread than the prior Sliding-Lid audit's boundary fixture (which I flagged as lacking a literal square-plan case) — this one includes it directly.

**Production isolation (the blocking-severity boundary), independently confirmed via full call-chain tracing:** `downloadCurrentDesignSvg()` (unchanged by this diff, confirmed via hunk-range) calls `refreshDesignPreview()` then downloads `result.svg` — it has no reference to `designPreviewMode` or `buildDiceTrayFinishedViewSvg` anywhere. `buildDiceTrayFinishedViewSvg`'s only caller in the entire file is inside `designResultsHtml()`'s `preview` variable (`index.html:3258`), which flows only into the on-screen `results.innerHTML` — never toward `download()`. `serializeTrayCompatibilitySvg()` (confirmed unmodified) only ever serializes `model.layout.items[].svg` — it has no `finishedView`/renderer reference at all, since it doesn't even receive `result` as an argument, only `model`. The dedicated fixture at `index.html:3388` (`diceFinishedViewBrowserFixture`) drives this end-to-end through real DOM: it builds real `designForm`/`designResults`/`downloadDesignSvg` elements, calls the real `refreshDesignPreview()`, **clicks the real rendered button** (`results.querySelector('[data-design-preview-mode="finished-view"]')?.click()`, a genuine simulated user click on a button bound by the real `bindDesignPreviewActions()`, not a synthetic session-variable assignment), and then calls the real `downloadCurrentDesignSvg()` — confirming `downloaded===diceBrowser.cutSvg` and `!diceBrowser.downloaded.includes('dice-tray-finished-view')` while Finished View is the active on-screen mode at the moment of download, matching the task's exact "while Finished View is active, click Download" scenario.

---

## 11. Tray production regression — independently confirmed

Read the full 23-case matrix (`index.html:3350-3351`) and its per-case assertions (`index.html:3355-3363`). The pinned Dice default golden (`1726` / `51a55721`) and Divider default golden (`1965` / `a55dda6e`) are the exact literal values documented in the Phase 2 report — confirmed byte-identical, not just "similar," by direct string comparison of the two report excerpts and the current source. Every matrix case's assertion set (live validity, pinned byte signature, pinned dimensions/viewBox, pinned element order/attribute hash, anonymous single red group with `fill=none|stroke=#ff0000|stroke-width=0.1`, no NaN/Infinity/undefined, live preview/download/filename identity, source non-mutation) is unchanged from the Phase 2 implementation, confirmed via the hunk-range method — none of `index.html:3349-3364` (the matrix definition and its forEach body) falls inside any hunk in this diff. This directly confirms the Finished View work did not weaken or touch the Phase 2 production-protection matrix.

---

## 12. Fixture quality

All 11 new assertions (`index.html:3377-3391`) were read directly. Classification:

- **Independent**: the "minimal semantic model data" check (`index.html:3377`) includes `!/<(?:svg|path|rect|polygon)\b/.test(JSON.stringify(diceView))` — a genuinely strong structural proof that the projection object's serialized form contains no SVG-markup substrings anywhere, not merely "the fields I expect are present." The dimension/wall-ID cross-check (`index.html:3381`) compares the rendered SVG's `data-wall-id`/`data-base-slot-ids` attributes against `dice.orientation`/`dice.components` **directly** (the original model object, not the projection's own copy) — closing the loop independently rather than checking the projection against itself. The joint-style anti-claim regex (`index.html:3382`) and the cross-template isolation check (`index.html:3389`, verifying Divider has no `finishedView` *and* the other three templates' existing modes are untouched) are both independent, external-oracle assertions. The browser-workflow fixture (`index.html:3388`) drives real DOM/button-click/download code, not a reimplementation.
- **Partially circular**: the determinism check (`index.html:3379`) calls the renderer twice and compares — proves stability, not correctness of a single call. The boundary-finiteness check (`index.html:3384`) is a broad sweep rather than per-case exact-value verification.
- **Circular**: none identified among the 11 new assertions.

No fixture name overstates its coverage beyond what direct reading confirms it tests.

---

## 13–14. Existing Finished View regressions and protected boundaries — independently confirmed via the hunk-range method

Every hunk in this diff falls within `index.html:2016-3392`; `buildFingerBoxFinishedViewSvg()`, `buildSlidingLidFinishedViewSvg()`, `buildDrawerCabinetFrontElevationSvg()`, `buildBoxModel()`/`buildBoxDesignResult()`, `buildSlidingLidBoxModel()`/`buildSlidingLidDesignResult()`, `buildDrawerCabinetModel()`/`buildDrawerCabinetDesignResult()`, `STORAGE_KEY`/`SCHEMA_VERSION`, `persist()`, `backupObject()`, `replaceData()`, `mergeData()`, `normalizeProductionEvidence()`/`normalizeProductionSetting()`/`normalizeProductionSettings()`, Production Setting application, Material Test/Test Grid/Project schemas, `downloadCurrentDesignSvg()`, filename generation, and the anonymous-red-group contract are all outside every hunk range — none appear anywhere in the diff. `designPreviewModeForTemplate()`'s Drawer Cabinet and Sliding Lid branches (`index.html:3160`, `3162`) are confirmed unmodified (the diff only inserts a `dice-tray` case into the pre-existing Finger Box line, per the hunk at old-line ~3054). No automatic persistence or record creation occurs in the mode-switch path, for the same reasoning verified in §3 and the prior Sliding-Lid audit.

---

## 15. Visual review

**Not independently verified visually** — no browser runtime available in this environment (same `file://`-navigation-denied constraint disclosed throughout this chain). All findings above were derived by reading and hand-tracing source rather than rendering pixels; I'm disclosing this gap rather than fabricating a visual description.

---

## 16. Documentation accuracy — independently confirmed

Read the actual `README.md` diff (not shown verbatim here, but inspected directly): it states the totals (233 Tray / 861 Designs geometry / 1653 complete), matching my independent recomputation exactly. The implementation report's stated claims — screen-only, authoritative structured Tray model as the source, canonical Front/Back/Left/Right orientation, open-top depiction, unchanged production Cut Layout downloads, and explicit non-claims about fit/strength/kerf/glue/safety — all match what I independently verified in the source. No claim of a true finger-jointed tray corner appears anywhere in either the code or the documentation. Divider Tray Finished View is correctly documented as not implemented, matching the triple-guarded exclusion confirmed in §2/§3.

---

## Findings

| # | Severity | Function / path | Scenario | Consequence | Recommendation | Fixture coverage |
|---|---|---|---|---|---|---|
| F1 | Minor | Visual/screen rendering | Task requests actual browser-rendered visual observations | No browser runtime was available to this audit to produce them; all findings above are source-derived, not pixel-derived | A follow-up pass with real browser access should confirm the isometric depth-cue reads clearly at small tray sizes and that labels don't crowd on the extreme boundary configurations (e.g. the `42×48` minimum-valid case) | N/A — outside what static analysis can prove |

No Blocker or Major findings.

---

## Required conclusion

```text
SAFE TO COMMIT
```

Production isolation (the blocking-severity boundary) is structurally proven by tracing every function in the call chain from `downloadCurrentDesignSvg()` back to `serializeTrayCompatibilitySvg()` — the Finished View renderer and projection are referenced nowhere in that chain, and the `finishedView` field is attached to the result only after `svg` is already fully and independently computed. Divider Tray exclusion is enforced redundantly in three separate places (the projection function, the selector, and the results renderer), and orientation is derived from the model's explicit `orientation` object, structurally separate from sheet-layout coordinates. Fixture counts (233 Tray / 861 Designs geometry / 1653 grand total) all independently reconcile, cross-checked against the Phase 2 report's own documented baseline and, for the 628 non-Tray Designs-geometry figure, against an entirely separate prior audit session. The single Minor finding is a disclosed verification-depth gap (no browser access for visual confirmation), not an observed defect.

---

*Focused audit performed read-only at commit `edd07d5` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. Unrelated untracked files were left untouched.*
