# Sliding-Lid Phase 3.1 — Second-Pass Review (Independent Check of Grok's Audit)

**Reviews:** `docs/SLIDING_LID_PHASE3_1_INDEPENDENT_AUDIT_2026-07-15.md` (Grok's audit)
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `df54da0` — *Add sliding-lid box generator*
**Reviewed:** Uncommitted changes in `index.html`, `README.md`
**Date:** 2026-07-15
**Mode:** Read-only. No browser runtime here, so the live 280/812 run couldn't be reproduced. Every geometric claim was verified by reading `mirrorDesignPanel`, `slidingRailGuideSegments`, `buildSlidingCouponModel`, and `buildSlidingLidDesignResult` directly and hand-tracing coordinates — and, critically, by tracing the guide marks' coordinate frame against the panel's *actual* physical dimensions rather than only against the "intended" corner table the task prompt supplied.

---

## Verdict on Grok's audit

**Mostly accurate — but I found a real, physically significant defect that neither the architecture review, the implementation report, nor Grok's audit identified: the rail placement guide marks are drawn Front-flush, not Back-flush, contradicting the explicit assembly instructions printed next to them.** Everything else — mirroring mechanics, guide segment count/shape, coupon formulas, fixture count (280, recomputed exactly), storage isolation, legacy signatures — checks out. Because of the guide-mark finding, I'm downgrading the recommendation from Grok's unconditional "SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP" to the same label but with a **specific, code-level correction required before the guide-marks feature is trusted for actual cutting** (the underlying box body/lid/rail geometry, which doesn't depend on the marks, remains sound and separately committable).

---

## Repository inspection (re-run independently)

| Check | Result |
|---|---|
| HEAD | `df54da0`, matches |
| Staged files | None |
| Tracked changes | `README.md`, `index.html` only |
| `git diff --numstat` | `README.md` 2/2, `index.html` **87/9** |

Ninth review in this chain where my `--numstat` doesn't match Grok's reported figure (Grok: +96/−11). No functional impact, consistent pattern.

---

## The guide-mark offset — an independently discovered finding not in any prior report

### What the code does

`slidingRailGuideSegments(dimensions, thickness)` (line 1885) computes eight L-mark segment coordinates in a purely **local** frame: `xFront = inset`, `xBack = dimensions.railLength - inset` (80 − 1.5 = 78.5 for Golden A), i.e., the marks span **0 to `railLength` (80mm)**.

`buildSlidingLidDesignResult` (lines 2076–2083) takes these segments and, after optionally mirroring the Right copy, hands them to the layout stage. The only positioning applied afterward is:

```javascript
placedScorePaths = scorePaths.map(score => ({...score, x: placedById[score.panelId].x, y: placedById[score.panelId].y}));
```

This places the guide path's local origin `(0,0)` at the **panel's own layout origin** — i.e., the marks are drawn starting exactly at the panel's Front edge (x=0 in panel-local space), spanning to x=80. **No horizontal offset is ever applied to shift the marks toward the Back.**

### Why that's wrong

The side panel's own width is `outsideDepth` (86mm for Golden A), not `railLength` (80mm) — a 6mm (=`2t`) difference, because the panel spans the full outside depth while the rail spans only the inside depth (the rail can't occupy the space where the Front/Back wall material itself sits). The UI text is explicit about the intended convention, in **two places**:

- Template note (line 1508): *"Glue the rails **top-flush and back-flush** with both channels facing inward."*
- Assembly summary (line 2148): *"Glue the two identical rails inside the side walls, **top-flush and back-flush**... Insert the lid from Front toward Back; the shoulders stop at Back."*

"Back-flush" means: push the rail's closed stop-end against the panel's actual Back edge until they touch. Since the rail is 80mm long and the panel is 86mm wide, a back-flush rail occupies **x = 6 to x = 86** on the panel (i.e., offset by `outsideDepth − railLength = 2t = 6mm` from the Front), not **x = 0 to x = 80** as the code currently draws the marks.

I verified the vertical axis does *not* have this problem — `railBottom = usableHeight − lowerWeb` (line 1767) is an explicit, independently-computed placement formula that correctly positions the rail against the top of the interior (`railBottom + railHeight === insideHeight`, confirmed by an existing fixture at line 2381). **There is no equivalent horizontal placement formula anywhere in the codebase** — `railLength` is used only for the rail's *own* local geometry, never combined with `outsideDepth` to derive where along the panel's depth the rail (or its guide marks) should sit.

### Consequence

As currently coded, the guide marks show a builder where to glue the rail **flush with the Front edge**, not the Back edge — the opposite of the documented and dimensionally-load-bearing convention. If a builder follows the marks literally:
- The rail's closed stop lands 6mm short of the actual Back wall, leaving a 6mm gap instead of a firm stop.
- The lid's travel — which the dimensional model assumes ends when the tongue reaches the Back plane (`tongueEnd = Di`, verified in the Phase 3 audit) — would in the *physically built* box stop 6mm early, and the tongue would not reach Back.
- This directly undermines the one thing the guide-marks feature exists to do: a builder who ignores the marks and eyeballs "push it against the actual Back wall" (the Phase 3, no-guides approach) would build a **more correct** box than one who follows the new marks.

This is scoped narrowly: it's a bug in the **optional, defaults-off guide-marks overlay only**. The underlying box body, rail, and lid geometry (Phase 3, and Phase 3.1 with guides disabled) is unaffected — those dimensions don't reference the guide marks at all. `buildSlidingLidDesignResult`'s dimensional model and the guide-mark drawing are two independent code paths that happen to disagree about where the rail sits.

### Why this passed all 280 fixtures

I checked what the Phase 3.1 fixtures actually assert about guide position (per the mating-pair and mirror-compare style seen in `runDesignGeometryFixtures`): they compare the *generated* segment coordinates against a hand-derived table — but that table (and the one supplied in this audit's own task prompt) was evidently derived the same way the code computes it (`0` to `railLength`), not independently derived from the "back-flush" physical requirement. Both the fixture and the task-prompt's "intended" reference table encode the *same* Front-flush assumption the code has, so a self-consistent bug reproduces itself in the check. This is precisely the kind of gap the task's own coverage-map exercise is meant to surface ("fixture merely calls production helper and checks its own result" vs. "independently falsifies") — and it's the deepest form of that trap: even a hand-written "expected" table can inherit the same wrong assumption if it isn't cross-checked against the plain-English assembly instructions.

### Recommended correction (not implemented — read-only audit)

Add a horizontal offset `railOffset = dimensions.outsideDepth − dimensions.railLength` (= 2t) to every guide-mark X coordinate before placement, so `xFront = railOffset + inset` and `xBack = dimensions.outsideDepth − inset`. A new regression fixture should assert `xBack (guide) === panelWidth − inset` for the *panel's actual width*, not `railLength − inset`, precisely because the current fixture's blind spot is comparing the code to itself rather than to the panel dimension.

**Severity: Medium.** Not a crash, not a validity-gate failure, doesn't affect the underlying box's own correctness — but it makes the specific new feature actively counterproductive if used as intended (guiding rail placement by eye/laser-etch), and no existing fixture can catch it because the "expected" data was derived the same way as the code under test.

---

## Everything else — independently confirmed, consistent with Grok's audit

### Mirroring — confirmed exact

`mirrorDesignPanel`: `x' = designRound(panel.width − x)`, `y' = y` (line 1877) — a pure horizontal mirror, no vertical flip, applied only to the `right` panel's cut path when `guideMarks` is true (line 2074). Guide segments for the Right side are mirrored using `rightPanel.width` (the already-mirrored panel's own width, 86mm) — Grok's "important detail" that this differs from the 80mm rail-local width used to *generate* the marks is correct and appropriately flagged, though it's a separate, correct design choice (mirror axis must be the panel's own width) from the Front/Back offset bug above (which exists in both the mirrored and unmirrored copies equally, since mirroring doesn't fix a Front/Back positioning error — it just reflects it).

### Guide segment shape and count — confirmed exact, hand-derived independently

Computed all 8 Golden A Left-guide segment endpoints directly from `slidingRailGuideSegments`'s formula (`inset = max(1, t/2) = 1.5`, `leg = min(6, max(3,t)) = 3`, using the function's own `xFront/xBack/yTop/yBottom` = 1.5/78.5/1.5/9.7): all 8 match the task prompt's "intended" table and Grok's report exactly. 4 L-marks × 2 segments = 8 per side, 16 total, one compound path per side subgroup — confirmed by reading the segment array literal directly (8 entries) and `designScoreSegmentsPath`'s join-without-separator behavior (produces one continuous multi-`M` compound path string, not 8 separate `<path>` elements).

### Coupon formulas — confirmed exact via direct hand calculation

Recomputed every Golden A coupon quantity from `buildSlidingCouponModel`'s formula line (1897) independently: Gc=32, Dc=max(40, 4+12+5)=40 (wait: `stopLength + max(3t,12) + 5` = 4+9+5=18, max(40,18)=40 ✓), Wi,c=38, Wb=44, B=max(6,6)=6, Ch,c=3.2, Rh,c=11.2, Rc,c=36, Lw,c=37.8, Tw,c=31.8, Ec=2.9, lid length=43, shoulder end=39, tongue end=43 — all match Grok's table and the task's reference values exactly.

### Fixture count — 280 confirmed via loop-aware recount

Static `add(` sites: 218 (up from 203 at the Phase 3 baseline, +15 exactly). Loop structure unchanged from Phase 3 (legacy-baselines ×4, Golden A ×21 keys, Golden B ×21 keys, mating-pairs ×8). Runtime total: (218−8 loop-body sites) + (5×4 + 21 + 21 + 8) = 210 + 70 = **280**, exact match. Grand total 20+12+23+67+57+56+61+216+12+8+280 = **812** ✓.

### Storage isolation and legacy stability — strongest-evidence confirmation

`git diff | grep` for `STORAGE_KEY`, `SCHEMA_VERSION`, `function freshState`, `function persist(`, `function backupObject` — zero hunks. All six legacy template signatures (`qr-stand` 359/`fe737a09` through `finger-box loose` 2691/`c202cef2`) confirmed present unchanged in fixture source.

---

## Findings

| # | Severity | Status |
|---|---|---|
| **New** | **Medium** | Guide marks drawn Front-flush; documented convention and lid-travel model require Back-flush. Independently discovered, not in Grok's report — see above. |
| Grok M1 (physical unverified) | Medium | Confirmed, applies equally |
| Grok M2 (fixture coverage gaps) | Medium | Confirmed, and the new finding is a concrete instance of exactly this risk |
| Grok M3 (report wording) | Medium | Confirmed via direct code read (4 L-marks, 8 segments/side, 16 total) |
| Grok M4 (missing checkbox help) | Medium | Confirmed by reading `renderDesign()`'s sliding fields |
| Grok L1–L3 | Low | Confirmed consistent with source |

---

## Unverified areas

Live 280/812 execution, LightBurn import, physical scrap cutting — same constraints as Grok. The guide-mark finding above is derived entirely from source arithmetic and does not require live execution to be true, but a physical scrap test would make it immediately obvious (measure 6mm from where the marks say to glue vs. where "push against the Back wall" actually lands).

---

## Required conclusion

```text
SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP
```

The underlying box, lid, rail, and coupon geometry are sound and independently verified. The guide-marks feature specifically should be flagged to the user as **not yet reliable for its stated purpose** until the Front/Back offset is corrected — recommend disabling or clearly caveat-labeling guide marks in the UI/README until fixed, since the feature currently guides installation to a position 6mm off from the documented and load-bearing back-flush convention.

---

*Second-pass review performed read-only at commit `df54da0` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. Unrelated untracked files were left untouched.*
