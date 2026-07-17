# Designs Dice Tray Underside Cover Plate — Concealment Correction Audit (2026-07-17)

**Mode:** Read-only final verification (no application edits, no git writes, no staging).  
**Primary app path:** `C:\Genmitsu L8 Tracker`  
**Feature:** Optional Dice Tray **Underside cover plate** (session-only, off by default).  
**Prior defect:** Fixed **3 mm inset** left a radial strip of every wall slot exposed at typical **3 mm** material (`t/2 = 1.5 mm` outer edge vs 3 mm inset).  
**Approved correction:** Full-size plate, **`bottomCoverInsetMm = 0`**, matching structural-base outside footprint.  
**Primary documents (context only; source is authority):**  
- `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_DESIGN_REVIEW_2026-07-17.md` (historical; unchanged)  
- `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_IMPLEMENTATION_2026-07-17.md` (updated for full-size)  
- `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_FOCUSED_AUDIT_2026-07-17.md` (historical fail; unchanged)  
- `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_CONCEALMENT_CORRECTION_2026-07-17.md`  

**This document:** Independent verification that the Major concealment defect is resolved and the corrected tree is safe to commit.

---

## 1. Actual HEAD, baseline, working-tree state

| Check | Result |
| --- | --- |
| `git log -1 --oneline` | **`c064896` Add drawer cabinet shelf-placement guides** |
| Matches baseline `c064896`? | **Yes** |
| `git status -sb` | `## main...origin/main` · `M index.html` · `M README.md` · pre-existing untracked docs / LightBurn / debug / utility |
| Staged | **None** |
| Tracked modified | **`index.html`**, **`README.md` only** |
| Unrelated untracked | **Untouched** by this audit |
| `git diff --check` | **Clean** (CRLF notices only) |
| `git diff --stat` | README 5 ± · index 56 ± · **2 files, 47 insertions(+), 14 deletions(-)** |
| `git diff --numstat` | README `3 2` · index `44 12` · **totals 47 / 14** — reconciles with `--stat` |
| Read-only status | Application sources **not edited**; this audit **only created** this report under `docs/` |

---

## 2. Complete diff-hunk enumeration

### 2.1 `README.md` (numstat 3 / 2)

| Hunk | Classification | Content |
| --- | --- | --- |
| Feature blurb | **README** | Full-size plate; manual edge alignment; cosmetic only; FV deferred — **no 3 mm reveal** |
| Fixture totals | **README** | Complete **1714**, Tray **264**, Designs **922** |
| Validation sentence | **README** | Designs **922** / complete **1714** |

### 2.2 `index.html` (numstat 44 / 12)

| # | Region | Classification | Summary |
| --- | --- | --- | --- |
| 1 | `designDefaults` | **Dice-only defaults/UI** | `trayBottomCover: false` |
| 2 | Dice form checkbox + help | **Dice-only defaults/UI** | “Add underside cover plate”; full-size; align edges; cosmetic |
| 3 | `normalizeDesignDraft` | **normalization** | Dice-only `bottomCover` for `true` / `'true'` / `'on'` |
| 4 | `buildTrayModel` cover block | **cover geometry correction** + **layout** + **metrics/warnings** | `inset=0`, width/depth = base outside; height `+ gap + coverDepth`; warnings |
| 5 | `buildTrayDesignResult` | **metrics/warnings** | Spreads model metrics/dimensions; model warnings |
| 6 | Dice result metrics UI | **metrics/warnings** | Bottom cover + base thickness lines |
| 7 | Tray fixtures | **fixtures** | 16 cover assertions incl. real 2D containment |
| 8 | Form handler | **form event handling** | `trayBottomCover` live re-render |

**Unexpected production hunks:** **None.**  
**Serializers:** `designSvgDocument` / `serializeTrayCompatibilitySvg` **unchanged** vs `c064896`.  
**Obsolete inset:** **gone** (`bottomCoverInsetMm=0`; no `designRound(base−6)`).

---

## 3. Files / functions inspected

`index.html` (working tree + baseline), `README.md`, all four primary docs, `buildTrayModel`, `trayRectComponent`, `trayTabProfile`, `designTabPositions`, `traySlotWidthMm`, `trayModelValidationErrors`, `designSvgDocument`, `serializeTrayCompatibilitySvg`, `normalizeDesignDraft`, `buildTrayDesignResult`, Dice metrics/warnings, cover fixtures (`wallSlotContainment`), preview/download helper, storage/backup isolation, Finished View projection/renderer, suite counting via Edge selftest.

---

## 4. Corrected geometry contract

| Requirement | Actual source | Verdict |
| --- | --- | --- |
| `bottomCoverInsetMm = 0` | `bottomCoverInsetMm=0` | **Pass** |
| Width = base outside width | `bottomCoverWidthMm=baseOutsideWidthMm` | **Pass** |
| Depth = base outside depth | `bottomCoverDepthMm=baseOutsideDepthMm` | **Pass** |
| No second base-outside derivation | Uses already-computed `baseOutside*` only | **Pass** |
| Finite / positive | Guard + defaults | **Pass** |
| No user-editable inset / no thickness-aware inset / no reveal option | None present | **Pass** |
| Plain rect only | `trayRectComponent` — no scores/holes/labels/radii | **Pass** |

---

## 5. Independent default geometry derivation

Inputs: inside **220×160**, thickness **3**, height **35**, `marginMm=10`, `gapMm=15`.

| Quantity | Formula | Value |
| --- | --- | --- |
| `baseOutsideWidthMm` | 220 + 2×3 | **226** |
| `baseOutsideDepthMm` | 160 + 2×3 | **166** |
| Cover W×D | = base outside | **226 × 166** |
| `wallY` | 10 + 166 + 15 | **191** |
| Disabled layout height | 191 + (35+3+15)×2 + 10 | **307** |
| Cover `x` | margin | **10** |
| Cover `y` | 307 − 10 + 15 | **312** |
| Enabled layout height | 307 + 15 + 166 | **488** |
| Layout width | max(10+226+15+160+10, 226+20) | **421** (unchanged vs disabled) |

All claimed default numbers **confirmed by hand**.

---

## 6. Slot-containment audit (primary correction boundary)

### 6.1 Placement formulas (source)

- Front: base-local `y = t/2`, height `slotWidth = t + clearance`  
- Back: symmetric from far edge  
- Left/right: outer edge at `t/2`, span `slotWidth`  
- Lateral positions: `designTabPositions`  

### 6.2 Default front slot (hand derivation)

`t = 3`, clearance `0.15` → `slotWidth = 3.15`  
Front band: **`y = 1.5` through `4.65`**  
Cover / base footprint: **`[0, 226] × [0, 166]`**  
⇒ fully contained (strict interior on y; lateral first tab starts at **42**).

Correction report claim **1.5..4.65**: **confirmed**.

### 6.3 Required cases (independent rectangle containment)

Containment rule: for every wall-slot base-local rect,  
`left≥0`, `right≤W`, `top≥0`, `bottom≤D` with full-size cover `W×D = baseOutside`.

| Case | Profile | Base | Issues |
| --- | --- | --- | --- |
| Default 3 mm | Finger (4-tab) | 226×166 | **0** (16 slots) |
| Default 3 mm | Tab-slot (2-tab) | 226×166 | **0** (8 slots) |
| Near-min 42×48 | Finger | 48×54 | **0** |
| Near-min 42×48 | Tab-slot | 48×54 | **0** |
| Thin 0.1 mm, 48×48 inside | Finger | 48.2×48.2 | **0** |

All four wall families audited in each case. Edge equality is allowed; full-size plate shares base edges. Prior Major (radial exposure under 3 mm inset) is **eliminated** because inset is zero and cover equals the full base that already hosts every slot.

---

## 7. Fixture quality

Helper `wallSlotContainment(model)`:

- Finds real `base-wall-slot` components  
- Converts sheet coords → base-local via `slot − base` origin  
- Tests **full rectangles** (min/max both axes)  
- Aggregates **front/back/left/right** families  

| Assertion | Class |
| --- | --- |
| Normalize true/on; false/null/whitespace/malformed | **Independent** |
| Divider isolation | **Independent** |
| Disabled goldens 1726/51a55721, 1965/a55dda6e | **Independent** (baseline pins) |
| Existing component JSON identity + cover last | **Independent** |
| Full-size 226×166 @ (10,312), inset 0 | **Independent** (hand-derived) |
| Layout height 488, width stable, non-overlap | **Independent** |
| Default Finger containment + literal 1.5/4.65 | **Independent** (geometry + literals) |
| Default two-tab / near-min / thin containment | **Independent** (real slots vs footprint) |
| Anonymous SVG last rect; golden **1778/8e2ea3f4** | Structural **independent** + hash **partially circular** (acceptable) |
| Metrics/warnings; preview/download/storage/FV | **Independent** |

**Prior misleading fixture** (“three millimeter cover reveal inside the model slot margin” checking only `base−6`) is **removed/replaced**. No fixture claims full containment while only testing positivity.

Note: with a full-size cover equal to the base, containment is the expected model invariant (slots are cut into the base). Fixtures still **inspect real slot rectangles** and multi-case coverage — not helper-vs-self geometry reimplementation.

---

## 8. Disabled production identity

| Pin | Baseline `c064896` | Working tree |
| --- | --- | --- |
| Dice | **1726 / 51a55721** | Still asserted; path with cover off adds no component |
| Divider | **1965 / a55dda6e** | Still asserted |

Absent / false / malformed → disabled. No disabled layout, piece-count, order, filename, or MIME drift.

**Obsolete enabled golden `1778 / ec9b12bc`:** **not asserted**. Current pin **`1778 / 8e2ea3f4`** only (same length; hash changed with full-size rect dimensions).

---

## 9. Divider isolation

No checkbox; cannot normalize `bottomCover`; never adds `tray-bottom-cover`; piece count / components / layout / SVG golden / Finished View / filename / MIME unchanged.

---

## 10. Existing component identity

Enabled vs disabled Dice models: every pre-existing component retains id, name, kind, role, geometry, SVG string, relative order (fixture `JSON.stringify` equality).

Added only:

- id `tray-bottom-cover`, name `Tray bottom cover`, kind/role `bottom-cover`  
- last in `components`, last in `materialComponentIds`, +1 piece, last SVG shape  

---

## 11. Layout

| Check | Verdict |
| --- | --- |
| margin 10 / gap 15 | **Pass** |
| cover x = margin; y = oldH − margin + gap | **Pass** (312) |
| height += gap + full cover depth | **Pass** (488) |
| width unchanged | **Pass** (421) |
| Existing items unmoved; no overlap; finite; in bounds | **Pass** |

---

## 12. Anonymous SVG contract

Serializers **byte-identical** to `c064896`. Enabled output: same wrapper, one anonymous red group, old shapes unchanged, one final plain rect, no ids/titles/metadata/score/named groups/text/new layer. Golden **1778 / 8e2ea3f4** matches runtime after structural checks.

---

## 13. Result metrics and UI

| Metric | Observed |
| --- | --- |
| `bottomCoverInsetMm` | **0** |
| Width / depth | **226 / 166** (full base) |
| Thickness / total visible | **3 / 6** |
| Added piece count | **1** |

Checkbox: **Add underside cover plate** (no “hides wall slots” / no inset claim).  
Help + warnings: full-size, manual edge alignment, cosmetic only, no structural replacement, flush tabs, clean surfaces, added thickness/time, first-tray verify.

**Stale 3 mm reveal / inset / reinforcement / captured / stronger joint:** **not found** in UI, metrics, README feature blurb, or updated implementation/correction reports.

---

## 14. Finished View

No cover in `trayFinishedViewProjection`; no production SVG dependency; docs correctly defer representation.

---

## 15. Storage and protected boundaries

Diff does not alter storage/schema/persist/backup/import/export, Production Settings, evidence, promotion, Library, Inventory, Projects, Pricing, Test Grid, Material Test, QR Stand, Hanging Sign, Finger Box, Sliding-Lid, Drawer Cabinet, Joint Fit Coupon, serializers, or unrelated Finished Views.

Preview/download and storage fixtures exercise the **enabled** real path.

---

## 16. Fixture totals

| Suite | Baseline | Claimed | Runtime (this audit) |
| --- | --- | --- | --- |
| Tray-model | 248 | 264 (=248+16) | **264 / 0** |
| Designs geometry | 906 | 922 (=906+16) | **922 / 0** |
| Complete suite | 1698 | 1714 (=1698+16) | **1714 / 0** |

16 new static `add(` assertions in the cover block (no multi-assert loops). Nested Tray results are included once in Designs. Complete-suite reconciliation uses the **final** Material-browser total only (avoids intermediate double-count):

`20+12+66+58+118+23+18+67+57+56+61+12+8+216+922 = 1714`.

README totals **match** execution.

---

## 17. Runtime / browser checks

| Check | Result |
| --- | --- |
| `git diff --check` | Pass |
| Headless **Microsoft Edge** `file://…?selftest=all` | Pass |
| Tray 264 / Designs 922 / complete 1714 | **Reproduced** |
| Containment fixtures | All pass (incl. thin material) |
| Independent Python containment | All five required cases **0 issues** |

---

## 18. Documentation accuracy

| Document | Assessment |
| --- | --- |
| README | Full-size, alignment, cosmetic, totals 1714/264/922 — **accurate** |
| Implementation report (updated) | 0 mm inset, 226×166, 488 height, golden `8e2ea3f4`, 16 asserts — **accurate** |
| Concealment correction report | Explains radial defect, full-size fix, validation — **accurate** |
| Original design review + failed focused audit | Left as historical records — **not rewritten** (correct) |
| Physical success claims | **None** |

---

## 19. Findings table

| ID | Severity | Location | Consequence | Recommendation | Fixture coverage |
| --- | --- | --- | --- | --- | --- |
| — | **Blocker** | — | — | **None** | — |
| — | **Major** | — | Prior radial exposure **resolved** | **None** | Full 2D family containment + literals |
| F1 | **Informational** | Physical assembly | Edge alignment, flush tabs, flat clamp still shop-only | Keep first-tray verify (present) | Warnings / help |
| F2 | **Informational** | Finished View | Cover still not drawn | Optional future note/geometry | FV independence fixture |
| F3 | **Informational** | Containment fixtures | Full-size cover makes base-slot containment the model invariant | Acceptable; fixtures still inspect real rects multi-case | Strong |

---

## 20. Physical limitations (explicit)

Software proves **mathematical** full-footprint coverage of wall-slot cutouts when the plate is perfect edge-aligned. It does **not** prove physical flush tabs, glue quality, char cleanup, clamp flatness, or shop alignment. No physical cut or LightBurn import was performed in this audit.

---

## 21. Highest-risk question scorecard

1. `bottomCoverInsetMm === 0` — **Yes**  
2. Cover matches full structural-base outside — **Yes**  
3. Every F/B/L/R wall-slot rect contained — **Yes**  
4. Both four-tab and two-tab — **Yes**  
5. Default, near-min, thin 0.1 mm — **Yes**  
6. Existing components byte-identical — **Yes**  
7. Divider fully isolated — **Yes**  
8. One final anonymous red rect — **Yes**  
9. Fixtures test true 2D containment — **Yes**  
10. Totals reconcile — **Yes (1714)**  
11. Reports accurate — **Yes**  
12. Unrelated production/storage/serializer/FV — **Unchanged**  

---

## 22. Conclusion

The approved full-size (**0 mm inset**) correction removes the prior Major radial-exposure defect. Independent derivation matches 226×166 @ (10, 312) and layout height 488 mm; all required slot families and profiles are fully contained; disabled goldens and Divider isolation hold; serializers and protected boundaries are untouched; fixtures and complete suite **1714 / 0** were reproduced on Edge `file://`.

### SAFE TO COMMIT
