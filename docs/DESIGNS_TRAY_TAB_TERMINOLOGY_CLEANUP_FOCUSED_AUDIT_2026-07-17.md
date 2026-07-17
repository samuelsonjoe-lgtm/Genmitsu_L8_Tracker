# Dice/Divider Tray Tab Terminology Cleanup — Focused Independent Audit

**Date:** 2026-07-17  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Baseline:** `2ac5080` — *Document joint system architecture*  
**Implementation report:** `docs/DESIGNS_TRAY_TAB_TERMINOLOGY_CLEANUP_IMPLEMENTATION_2026-07-17.md`  
**Planning context:** joint-system architecture review + challenge review (Phase A: terminology + dead-helper only)  
**Review type:** read-only focused audit. No application files were modified.

---

## Git baseline (actual)

| Item | Value |
|---|---|
| **HEAD** | `2ac508008e4e4c1a93178818700838aa36f6c707` (`2ac5080`) |
| **Branch** | `main`…`origin/main` |
| **Working tree** | Modified: `index.html`, `README.md`; untracked docs / other non-app artifacts |
| **`git diff --check`** | Clean (CRLF warnings only) |
| **`git diff --stat` vs `2ac5080`** | `README.md` 4 lines; `index.html` +15/−26 (net −11); **17 insertions, 28 deletions** |
| **Diff scope** | Tray form label/options/help; four compact tray fixtures (+ two strengthened FV wording asserts); delete `designEdgePoints`; README suite totals + tray honesty wording |

---

## Runtime totals (independent)

Isolated headless **Edge** (`channel='msedge'`) against `file://` `index.html`.

| Group | Expected | Observed | Notes |
|---|---:|---:|---|
| Tray model / live production | 248 / 0 | **248 / 0** | `window.runTrayModelFixtures()` |
| Designs geometry | 876 / 0 | **876 / 0** | `?selftest=all` console; **includes** nested tray 248 |
| Designs production application | 118 / 0 | **118 / 0** | |
| Test Grid machine identity | 18 / 0 | **18 / 0** | |
| Evidence Promotion | 58 / 0 | **58 / 0** | |
| Production Settings | 66 / 0 | **66 / 0** | |
| Storage recovery | 8 / 0 | **8 / 0** | |
| **Complete suite (logical)** | **1668 / 0** | **1668 / 0** | See below |

**Double-count rule:** Designs geometry **876** already embeds tray **248**. Tray is **not** added again when summing the suite. Own Designs-geometry-only count remains **628** (876 − 248), unchanged from the pre-Phase-A own count (872 − 244).

**Console caveat:** `?selftest=all` emits intermediate Material Browser `passed` lines (12 / 23 / 49) before the final Material Browser **57**. Naïve sum of every `N passed` line is **1752**; subtracting the three intermediate Material Browser totals (12+23+49) yields **1668**, matching README and the implementation report.

Default production goldens (from tray fixtures, live generation):

| Template | Length / hash | Result |
|---|---|---|
| Dice Tray default | **1726 / `51a55721`** | Pass |
| Divider Tray default | **1965 / `a55dda6e`** | Pass |

---

## 1. User-facing terminology — Verified

### Source

```1883:1883:C:\Genmitsu L8 Tracker\index.html
${box || sliding || cabinet || jointCoupon ? '' : tray ? designSelect('jointStyle','Wall-to-base tab profile',d.jointStyle,[['finger','Four-tab walls (wall-to-base)'],['tab-slot','Two-tab walls (wall-to-base)']]) : '<div></div>'}
```

### Browser-rendered (Edge, Designs tab)

| Check | Dice Tray | Divider Tray |
|---|---|---|
| Field label | **Wall-to-base tab profile** | same |
| Option values | `finger`, `tab-slot` | same |
| Visible option text | Four-tab / Two-tab walls (wall-to-base) | same |
| Default selected | `finger` → Four-tab… | same |

### Internal / validation

| Check | Result |
|---|---|
| Default `designDefaults().jointStyle` | still `'finger'` (`designDefaults` byte-identical to baseline) |
| `normalizeDesignDraft` allow-list | still `['finger','tab-slot']` only (function identical to baseline) |
| Invalid style e.g. `dovetail` | still rejected via existing “recognized tray joint style” path |
| Aliases / capability registry | **none** (`designJointCapabilities` absent) |
| Non-tray joint labels | unchanged (box/sliding/cabinet/coupon help and fields untouched in diff) |

---

## 2. Help-text honesty — Verified

### Source and live DOM text (identical)

> Tabs on each wall bottom mate with through-slots in the base. Adjacent walls meet as butt joints and need glue after a dry fit. This is not corner finger joinery. Fit clearance is a starting value; test on scrap or with a future fit coupon.

| Required meaning | Covered? |
|---|---|
| Tabs on each wall bottom | Yes |
| Mate with through-slots in the base | Yes |
| Adjacent walls are butt joints | Yes |
| Glue after dry fit | Yes |
| Not corner finger joinery | Yes |
| Clearance only a starting value | Yes |
| Scrap or coupon testing | Yes (“scrap or with a future fit coupon”) |

| Forbidden claim | Present in tray form? |
|---|---|
| Interlocking tray corners | No |
| Finger-jointed tray corners | No |
| Self-locking assembly | No |
| Glue-free construction | No |
| Secure / proven / production-tested fit | No |

Browser check: `Finger-tab` absent; `interlocking` absent from form text.

---

## 3. Internal behavior preservation — Verified

Byte-identical to `2ac5080` (function-body extract compare):

- `trayTabProfile`, `buildTrayModel`, `designWallPath`, `designTabPositions`, `serializeTrayCompatibilitySvg`
- `normalizeDesignDraft`, `designDefaults`
- `designSafePatternEdgePoints`, `buildSlidingLidBodyPanel`
- Box / Sliding Lid / Cabinet / Joint Coupon / FV builders listed in §8

Behavioral implications:

- `finger` → count **4**; `tab-slot` → count **2** (`trayTabProfile` unchanged)
- Slot width, tab width clamps, component IDs, divider spacing formulas unchanged
- Both styles remain valid through `buildDesignResult` (fixture + matrix cases still exercise both)

---

## 4. Finished View wording and isolation — Verified

| Item | Result |
|---|---|
| Dice / Divider FV builders vs baseline | **Identical** |
| Wording | Four-tab / Two-tab wall profile; “Walls align with base slots” (Dice); “Dividers align with base slots” (Divider) |
| viewBox / IDs / orientation / divider order | Unchanged (builder identity) |
| Corner finger joinery claims | Still negated in fixtures; not present in FV strings |
| Production SVG isolation | Default goldens unchanged; FV class names / wall profile strings not part of production hash contract |

Metrics panel still uses “Four-tab wall profile” / “Two-tab wall profile” (honest, not “finger joint”).

---

## 5. Dead-helper removal — Verified

### `designEdgePoints` repository search

| Location | Classification |
|---|---|
| `index.html` ~3451 fixture | **Live-source absence assertion only** (`typeof designEdgePoints === 'undefined'`) |
| Historical `docs/*` reviews | Documentation only (architecture / whisker history) |
| **Function definition** | **Removed** (present at baseline; absent now) |
| Call sites / dynamic maps | **None** |

### `designSafePatternEdgePoints`

| Check | Result |
|---|---|
| Definition | Present; **identical** to baseline |
| `buildSlidingLidBodyPanel` still calls it | Yes (body identical; fixture asserts `.toString()` includes name) |
| Finger Box / Sliding Lid / Cabinet / Coupon | Still flow through `buildFingerPanel` → `buildSlidingLidBodyPanel` (builders identical) |
| Second edge generator introduced | **No** |

---

## 6. Production-byte preservation — Verified

| Contract | Observed |
|---|---|
| Dice default | **1726 / `51a55721`** |
| Divider default | **1965 / `a55dda6e`** |
| Geometry / layout helpers | Unchanged |
| Serializer | `serializeTrayCompatibilitySvg` identical → still single anonymous red group, no panel IDs/titles |
| Download path | `downloadCurrentDesignSvg` identical |

**No production-byte drift.** Blocker criterion not met.

---

## 7. Fixture quality

### Net new assertions (244 → 248)

1. Tray form field + option labels + internal values  
2. Tray help completeness + forbidden-claim regex  
3. Both internal styles valid + `tab-slot` selected label  
4. `designEdgePoints` absent + safe helper present/used  

Plus **strengthened** (not net-new count drivers alone) Dice/Divider FV asserts requiring “align with base slots” and corner-joinery negatives.

### Coverage vs brief

| Required coverage | Covered by |
|---|---|
| Field label | Assert 1 |
| Visible option labels | Assert 1 |
| Unchanged internal values | Assert 1 (`value="finger"` / `value="tab-slot"`) |
| Complete help wording | Assert 2 |
| Both valid internal styles | Assert 3 + existing matrices |
| Finished View honesty | Strengthened FV asserts + existing profile strings |
| Dead-helper absence | Assert 4 |
| Continued safe-helper use | Assert 4 (string presence in panel builder) |

### Independence classification

| Check | Class | Notes |
|---|---|---|
| Form HTML substring labels | **Independent** | Against `renderDesigns()` output |
| Help substrings + negative regex | **Independent** | Live copy; regex phrase `secure or proven fit` is slightly narrower than “secure, proven, or production-tested” as separate tokens — **acceptable** for this phase |
| `buildDesignResult(…).valid` for both styles | **Partially circular** | Reuses production validation; still valuable regression guard |
| Golden length/hash | **Independent oracle** | Pinned literals |
| `typeof designEdgePoints === 'undefined'` | **Independent** | |
| `buildSlidingLidBodyPanel.toString().includes('designSafePatternEdgePoints')` | **Partially circular / weak** | Proves name still referenced, not full call-graph; adequate for wording-only phase given body identity vs baseline |

### Overstated names

- “Both supported internal tray jointStyle values remain valid **and select their visible wall-to-base labels**” — primarily proves `tab-slot` selected option text; finger label already covered by assert 1. Slight overstatement, not a failure.

No large fixture framework required; compact additions are appropriate.

---

## 8. Protected boundaries vs `2ac5080`

| Boundary | Status |
|---|---|
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged |
| `persist` / `backupObject` / `replaceData` / `mergeData` | Identical |
| Production evidence/settings normalizers | Identical |
| Production-setting application fixtures | 118/0 |
| Material Test / Test Grid / Project schemas | Untouched in diff |
| `buildTrayModel` / `trayTabProfile` / wall path helpers / tray serializer | Identical |
| `downloadCurrentDesignSvg` / filenames / MIME | Identical |
| Box / Sliding Lid / Cabinet / Coupon geometry | Identical |
| QR stand / Hanging sign | Untouched |

**Intentional app changes only:** tray visible labels, tray help, compact fixtures, dead-helper removal, README wording/totals.

---

## 9. Documentation accuracy

| Claim (README / implementation report) | Audit |
|---|---|
| Four-tab / two-tab wall-to-base profiles | Accurate |
| Wall-bottom tabs → through-slots | Accurate |
| Adjacent walls butt joints; glue after dry fit | Accurate |
| Not corner finger joinery | Accurate |
| Clearance theoretical until physical test | Accurate |
| Internal IDs and production geometry unchanged | Accurate |
| `designEdgePoints` removed only after unused | Accurate (baseline had definition, zero calls; now only absence fixture) |
| Suite 1668 / Tray 248 / Geometry 876 | Accurate when tray is not double-counted |

Fixture scenario **names** still say “Tab-and-slot” / “Finger” (`~3360–3361`) — console/fixture-only, not UI. Optional rename later; not user-facing.

---

## Findings

### Blocker
*None.*

### Major
*None.*

### Minor

1. **Fixture scenario titles still use legacy “Finger” / “Tab-and-slot” names**  
   - **Path:** `runTrayModelFixtures` matrix labels ~3360–3361  
   - **Scenario:** Console table / failure names  
   - **Expected:** Optional alignment with UI wording  
   - **Observed:** Internal IDs correct; UI correct  
   - **Consequence:** Mild grepping confusion only  
   - **Correction:** Rename fixture titles when convenient  
   - **Fixture coverage:** N/A  

2. **Help negative-regex does not explicitly ban “production-tested” as a standalone phrase**  
   - **Path:** new help fixture ~3446  
   - **Expected:** Full forbidden list from audit brief  
   - **Observed:** Live help does not claim production-tested fit  
   - **Consequence:** None in current copy  
   - **Correction:** Optional regex broaden  
   - **Fixture coverage:** Partial  

3. **Safe-helper “used” check is `.toString().includes`**  
   - **Path:** ~3451  
   - **Expected:** Call-site proof  
   - **Observed:** Adequate with identical `buildSlidingLidBodyPanel` vs baseline  
   - **Consequence:** Weak alone; strong with function identity  
   - **Correction:** None required for Phase A  
   - **Fixture coverage:** Partially circular  

### Verified

1. **UI labels and option values** match Phase A / challenge-review recommendation (browser + source).  
2. **Help text** is honest on tabs, slots, butt joints, glue, non-finger corners, provisional clearance.  
3. **Internal `jointStyle` IDs and all tray geometry pipelines unchanged.**  
4. **Production goldens 1726/`51a55721` and 1965/`a55dda6e` preserved.**  
5. **`designEdgePoints` fully removed from live code; safe helper path intact.**  
6. **No registry, aliases, storage, or non-tray geometry regressions.**  
7. **Fixture totals match expected 248 / 876 / 1668 with correct non-double-count of tray.**  
8. **Finished View builders unchanged; wording remains honest.**

---

## Alignment with planning documents

| Challenge-review Phase A requirement | Status |
|---|---|
| UI/help terminology only | Met |
| Delete dead `designEdgePoints` | Met |
| No registry / no enum rename / no coupon | Met |
| No production-byte change | Met |
| Light fixtures | Met |

Implementation report claims match independent source and runtime checks.

---

## Required conclusion

```text
SAFE TO COMMIT
```

Phase A is a clean, reviewable wording + dead-code cleanup. No geometry, storage, or production-byte risk was introduced. Optional minor fixture-title polish may wait for a later housekeeping commit.

---

*Audit performed read-only at baseline `2ac5080` with uncommitted Phase A changes in the working tree. No application files were modified, staged, committed, pushed, reset, cleaned, stashed, moved, deleted, renamed, or rewritten.*
