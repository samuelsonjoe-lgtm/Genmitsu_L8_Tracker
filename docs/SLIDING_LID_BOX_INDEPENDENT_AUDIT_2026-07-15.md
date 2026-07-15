# Sliding-Lid Box Phase 3 — Independent Architecture and Geometry Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Baseline commit:** `0a304d8` — Add parametric finger-box generator  
**Audit date:** 2026-07-15  
**Auditor:** Grok (read-only independent review)  
**Reports reviewed (not trusted as proof):**
- `docs/SLIDING_LID_BOX_ARCHITECTURE_REVIEW_2026-07-15.md`
- `docs/SLIDING_LID_BOX_PHASE3_IMPLEMENTATION_2026-07-15.md`
- Prior finger-box architecture, implementation, and audit reports

**Note:** The implementation report states `index.html` already contained uncommitted sliding-lid work when Phase 3 verification began. This audit inspected the **entire current working-tree diff** against `0a304d8`, not a single clean authoring session.

---

## 1. Executive conclusion

**Recommendation: `SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP`**

The sliding-lid template is a well-bounded extension: separate dispatch branch (`buildSlidingLidDesignResult`), dedicated dimension model (`slidingLidBoxDimensions`), asymmetric shortened-Front body geometry, composite side edges, laminated rail channels, shouldered lid, four-row layout with path-AABB checks, and a substantially expanded fixture suite. Live execution confirms **797 / 0** total fixtures and **265 / 0** Designs geometry assertions.

Independent hand-calculation of Golden A and Golden B dimension tables matches the production `slidingLidBoxDimensions()` output and the hardcoded fixture expectations. No Critical or High defects were found in code review. Residual risk is physical: scrap cut, rail glue alignment, and LightBurn import were not performed in this audit.

---

## 2. Baseline verification

### Commands run

```powershell
git status -sb
git rev-parse --short HEAD
git log -1 --oneline
git diff --check
git diff --stat
git diff --cached --name-only
```

### Results

| Check | Result |
|-------|--------|
| `HEAD` | `0a304d8` ✓ |
| `git log -1` | `0a304d8 Add parametric finger-box generator` ✓ |
| Staged files | None ✓ |
| Tracked modifications | `index.html`, `README.md` only ✓ |
| `git diff --stat` | `README.md` 4 lines; `index.html` +476/−22 (458 insertions net) |
| `git diff --check` | Clean (LF→CRLF warnings only) |
| Unrelated untracked files | Preserved |

This audit file was created after review completion at user request.

---

## 3. Findings by severity

### Medium

#### M1 — 3D fold and scrap assembly remain unproven

| Field | Detail |
|-------|--------|
| **Region** | Entire sliding-lid model; fixtures ~L2314–2322 |
| **Scenario** | Golden A default; Front insertion; shoulder/stop closure |
| **Expected** | Lid inserts from Front, captures in channels, stops at Back with tongue closing center gap |
| **Actual** | Algebraic consistency checks pass (e.g. `Hf − t = Uh − Ci`, `shoulderStop + S = Di`, `tongueWidth < railGap`). No folded 3D simulation or physical prototype |
| **Consequence** | Opposite 2D phases and formula locks are necessary but not sufficient for plywood assembly |
| **Correction** | None before commit; mandatory scrap prototype |
| **Fixture required?** | Optional golden 3D assembly reference (out of scope Phase 3) |

#### M2 — Many sliding fixtures still mirror production helpers

| Field | Detail |
|-------|--------|
| **Region** | `runDesignGeometryFixtures()` sliding block (~L2216–2386) |
| **Scenario** | Clearance deltas, mating pairs, layout positions |
| **Expected** | Independent falsification |
| **Actual** | Golden A/B tables are hand-derived (good). Many other assertions call `buildDesignResult`, `slidingLidBoxDimensions` outputs, or re-run the same generators |
| **Consequence** | Strong regression locks; weaker independent proof |
| **Correction** | Add more hand-derived coordinate tables for one rail + one composite side corner |
| **Fixture required?** | Yes (incremental) |

### Low

#### L1 — Download-handler bypass not DOM-tested

| Field | Detail |
|-------|--------|
| **Region** | `downloadDesignSvg.onclick` (~L4951–4956) |
| **Scenario** | Invalid geometry with disabled button |
| **Expected** | No download |
| **Actual** | Handler rebuilds, re-validates, returns early; button `disabled` when `!result.valid`. No automated forced-click test |
| **Consequence** | Low risk; logic is sound |
| **Correction** | Optional integration fixture |
| **Fixture required?** | Optional |

#### L2 — Finger-box layout still uses nominal envelopes only

| Field | Detail |
|-------|--------|
| **Region** | `layoutDesignPanels()` wrapper (~L1954) |
| **Scenario** | Standard `finger-box` template |
| **Expected** | Path AABB gutters (sliding uses `layoutDesignPanelRows` with path bounds) |
| **Actual** | Finger-box path unchanged; sliding adds path-AABB checks. No regression in finger-box |
| **Consequence** | Pre-existing Phase 2 limitation; sliding is improved |
| **Correction** | None for Phase 3 |
| **Fixture required?** | No (pre-existing) |

### Informational

#### I1 — Implementation landed as incremental uncommitted `index.html` work

Per implementation report, sliding-lid code predated the README/report pass. Diff inspection shows additive architecture on committed finger-box baseline without replacing `buildBoxModel()`.

#### I2 — Kerf correctly excluded; four clearance concepts separated in UI

`Cj` (`slidingJointClearance`), `Cs` (`lidSideClearance`), `Cv` (`lidVerticalClearance`), `Ci` (`frontInsertionClearance`) are distinct fields with distinct code paths.

#### I3 — No fit-coupon row (Phase 3 non-goal)

Fixture confirms four rows, eight pieces, no coupon geometry.

#### I4 — LightBurn import and physical cut not performed

---

## 4. Shortened-Front joint analysis

### Architecture

| Component | Implementation |
|-----------|----------------|
| Front height | `Hf = t + Uh − Ci` (`slidingLidBoxDimensions`, ~L1763) |
| Front vertical pattern | `buildFingerPattern(Hf, …)` as `frontVertical` — **not** cropped from full `verticalPattern` (~L1898) |
| Front panel | `buildSlidingLidBodyPanel`, height `frontHeight`, edges `[plain, frontPattern↑, width↓, frontPattern↑]` (~L1919) |
| Side lower Front | Composite edge: `compositeSpan: frontHeight`, opposite phase `frontPattern↓` (~L1852–1854, 1921) |
| Side upper Front | Plain continuation from `Hf` to full `outsideHeight` (~L1807–1811) |

### Eight body mating pairs (fixture-verified)

Same topology as standard box, with shortened Front substituting for full-height Front on width-edge bottom joint:

| Joint | Panels | Pattern |
|-------|--------|---------|
| Bottom ↔ Back | bottom[0] ↔ back[2] | width, opposite |
| Bottom ↔ Right | bottom[1] ↔ right[2] | depth, opposite |
| Bottom ↔ shortened Front | bottom[2] ↔ front[2] | width, opposite |
| Bottom ↔ Left | bottom[3] ↔ left[2] | depth, opposite |
| shortened Front ↔ Right (lower) | front[1] ↔ right[3] | frontVertical, opposite |
| shortened Front ↔ Left (lower) | front[3] ↔ left[3] | frontVertical, opposite |
| Back ↔ Left | back[1] ↔ left[1] | vertical, opposite |
| Back ↔ Right | back[3] ↔ right[1] | vertical, opposite |

### Composite transition

- `designSafePatternEdgePoints()` trims pattern to `[startTrim, patternEnd]` on composite edges (~L1786–1814).
- `designEdgeTerminalOffset()` handles corner joins between patterned and plain segments (~L1780–1800).
- Fixture: side upper plain run at `y = 7.4` (= `outsideHeight − Hf` = 50.2 − 42.8 for Golden A).
- Fixture: no cropped final finger (`boundaries[last] === length` for all patterns).
- Fixture: Hf transition segment-count tests at 42.1 mm (3 segments) vs 42.3 mm (5 segments).

**Assessment:** Shortened-Front logic is internally consistent. Back corners retain full-height vertical patterns. Structural web above sill remains 7.4 mm on Golden A defaults.

---

## 5. Rail analysis

`buildSlidingLidRail()` (~L1856–1860) — closed orthogonal 9-point path:

| Feature | Golden A (t=3, U=L=S=4, Cv=0.2) |
|---------|-----------------------------------|
| Local size | 80 × 11.2 mm (`Rl × Rh`) |
| Lower web L | y: 0 → 4 |
| Channel Ch | y: 4 → 7.2 (4 + 3.2) |
| Upper web U | y: 7.2 → 11.2 |
| Open Front | x = 0 open at channel depths |
| Back stop S | x = 76 → 80 (Rc = Di − S = 76) |
| Left/Right paths | Identical local geometry |
| Tabs/slots | None (9 points, no extras) |

Fixtures: self-intersection false, zero-length segments false, bounds contained, finite coordinates.

**Placement (formula):** `railBottom = Uh − L` (= 36 mm for Golden A); `railBottom + railHeight = insideHeight` (36 + 11.2 = 47.2). Top-flush with inside wall top per metrics.

**Assessment:** Rail geometry matches documented U-channel model. Stop web length is exactly `S` (80 − 76 = 4). Both rails exported as separate pieces with stable IDs.

---

## 6. Lid and stop/tongue analysis

`buildSlidingLidPanel()` (~L1862–1870):

| Parameter | Golden A | Formula |
|-----------|----------|---------|
| Max width `Lw` | 105.8 | `Wi − Cs` |
| Tongue width `Tw` | 99.8 | `G − Cs` where `G = Wi − 2t` |
| Engagement `E` | 2.9 | `(Lw − G) / 2` = `t − Cs/2` when Cs=0.2, t=3 |
| Total length | 83 mm | `Di + t` |
| Shoulder stop plane | y = 79 | `t + Rc` = 3 + 76 |
| Tongue end | y = 83 | `t + Di` = 3 + 80 |

**Coordinate origin:** Front edge at y=0; shoulders end at `t + Rc` (not y=Rc alone) — physically intentional: first `t` mm accounts for front-wall thickness region in the lid length budget. The extra `t` in `lidLength = Di + t` closes to outside Front face (fixture: `lidLength − insideDepth === t`).

**Capture:** Shoulders wider than tongue; engagement into each channel = E. Upper rail web captures vertically; lower web supports at `Uh`.

**Pull notch:** Optional rectangular; depth `max(t, 6)`; rejects crossing shoulder transition; centered (fixture at x ≈ 40.9–64.9 for width 105.8).

---

## 7. Insertion-path analysis

Algebraic checks (fixtures + independent verification for Golden A):

| Check | Calculation | Result |
|-------|-------------|--------|
| Sill below lid underside | `Hf − t = 42.8 − 3 = 39.8` | |
| Usable height − Ci | `40 − 0.2 = 39.8` | Match ✓ |
| Channels open at Front | Rail x=0 channel open | ✓ |
| Tongue fits between stops | `Tw 99.8 < G 100` | ✓ |
| Closure at Back | `Rc + S = 76 + 4 = 80 = Di` | ✓ |
| Tongue reaches Back plane | `tongueEnd = Di` | ✓ |
| No permanent rear opening | Stop webs at x ≥ Rc | ✓ |

**Assessment:** Front insertion path is coherent in the dimension model. Physical verification still required for rail glue, char, and squareness.

---

## 8. Clearance analysis

### Independent verification

| Symbol | Field | Effect in code | Verified |
|--------|-------|----------------|----------|
| `Cj` | `slidingJointClearance` | `designPatternEdge` clearance on body fingers only | Changes body paths, not lid/rails/dims ✓ |
| `Cs` | `lidSideClearance` | `lidWidth = Wi − Cs`, `tongueWidth = G − Cs` | Total Cs removed from width; `(railGap − tongueWidth)/2 = Cs/2` ✓ |
| `Cv` | `lidVerticalClearance` | `Ch = t + Cv`; drives `insideHeight` stack | Single addition ✓ |
| `Ci` | `frontInsertionClearance` | `Hf = t + Uh − Ci` only | Fixture: only `frontHeight` changes ✓ |

**Engagement:** `E = (Lw − G)/2`. For default t=3, Cs=0.2: E = 2.9 = t − Cs/2. Not doubled.

**Blocking thresholds:**

| Rule | Implementation |
|------|----------------|
| `Omin = max(1, t/3)` | `minimumEngagement` (~L1881) |
| `E ≥ Omin` | Error if engagement < Omin |
| `Rc > max(3t, 12)` | `minimumRun`; channelLength ≤ minimumRun blocks |
| Excessive Cs | Blocks at Cs=4.01 on Golden A (engagement collapse) |

**Invalid inputs tested in fixtures:** blank, text, NaN, Infinity, negative clearances, boundary outside dimensions, collapsed insertion, short channel, undersized Hf pattern, oversized pull.

**Kerf:** Absent from draft and SVG (fixture + code inspection).

---

## 9. Path and layout analysis

### Eight pieces (closed orthogonal paths)

Bottom, Sliding lid, Front (open), Back, Left, Right, Left rail, Right rail — all checked via `designPanelGeometryErrors()` and fixture self-intersection / duplicate-segment tests.

### Layout (`layoutDesignPanelRows`, ~L1933–1952)

| Row | Pieces |
|-----|--------|
| 1 | bottom, sliding-lid |
| 2 | front-open, back |
| 3 | left, right |
| 4 | rail-left, rail-right |

Golden A (independently verified against fixture):

| Piece | Position |
|-------|----------|
| bottom | 10, 10 |
| sliding-lid | 137, 10 |
| front-open | 10, 111 |
| back | 137, 111 |
| left | 10, 176.2 |
| right | 111, 176.2 |
| rail-left | 10, 241.4 |
| rail-right | 105, 241.4 |

Document: **259 × 262.6 mm**; margin 10; gap 15.

**Path AABB checks (Phase 3 improvement):** `translatedPathBounds` — no overlap, ≥15 mm separation, nonnegative, inside viewBox. Duplicate translated cut segments rejected in `buildSlidingLidDesignResult`.

---

## 10. SVG readiness

| Requirement | Status |
|-------------|--------|
| mm width/height | Yes |
| viewBox match | Validated |
| One `g#cut` stroke `#ff0000` | Yes |
| Eight `panel-*` groups | Yes |
| One path per piece | Yes |
| `<title>` metadata only | No `<text>` in cut layer |
| Byte-stable repeat | Fixture pass |
| Preview = download | Same `result.svg` string |
| Invalid → no SVG | Errors block serialization |

LightBurn: **not opened**.

---

## 11. Legacy regressions

| Template | Length | FNV hash | Status |
|----------|-------:|----------|--------|
| qr-stand | 359 | fe737a09 | Stable ✓ |
| hanging-sign | 341 | 656e633d | Stable ✓ |
| dice-tray | 1726 | 51a55721 | Stable ✓ |
| divider-tray | 1965 | a55dda6e | Stable ✓ |
| finger-box open | 2559 | 4e2a6f4b | Stable ✓ |
| finger-box loose | 2691 | c202cef2 | Stable ✓ |

`buildBoxModel()` unchanged in role; `buildSlidingLidBoxModel()` is separate. Finger-box byte-stable through `buildDesignResult` fixture.

---

## 12. Fixture-quality assessment

### Live totals (Edge headless `?selftest=all`)

| Group | Passed | Failed |
|-------|--------|--------|
| Baseline | 20 | 0 |
| Material test normalization | 12 | 0 |
| Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Project Wizard | 216 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| **Designs geometry** | **265** | **0** |
| **Total** | **797** | **0** |

`?selftest=design`: **265 / 0** (same Designs group).

### Strengths (new sliding coverage)

- Hand-derived Golden A/B dimension tables (36 keys × 2)
- Shortened Front separate pattern; composite side edges
- Eight body joint boundary/phase tests
- Rail geometry (stop S, open channel, 9-point path)
- Lid shoulder/tongue coordinates
- Clearance isolation (Cj/Cs/Cv/Ci)
- Path self-intersection, duplicate segments, zero-length
- Actual path AABB layout gutters
- Exact Golden A layout positions and document size
- Invalid-input matrix; pull validation
- Legacy six-template stability

### Remaining gaps

1. Independent golden path coordinates (not only dimension metrics)
2. Forced-download DOM bypass test
3. 3D assembly / insertion ray simulation
4. LightBurn import behavior

Existing fixtures were not weakened or removed.

---

## 13. Dimensional model — independent hand verification

**Defaults:** t=3, U=L=S=4, Cv=0.2, Cs=0.2, Ci=0.2, Cj=0.10 (Golden A override)

### Golden A (usable 100×80×40)

| Quantity | Hand calc | Expected | Match |
|----------|----------:|---------:|:-----:|
| Wi | 100+2t=106 | 106 | ✓ |
| Di | 80 | 80 | ✓ |
| Hi | 40+t+Cv+U=47.2 | 47.2 | ✓ |
| Wo | 112 | 112 | ✓ |
| Do | 86 | 86 | ✓ |
| Ho | 50.2 | 50.2 | ✓ |
| G | 100 | 100 | ✓ |
| Lw | 105.8 | 105.8 | ✓ |
| E | 2.9 | 2.9 | ✓ |
| Tw | 99.8 | 99.8 | ✓ |
| Ch | 3.2 | 3.2 | ✓ |
| Rh | 11.2 | 11.2 | ✓ |
| Rc | 76 | 76 | ✓ |
| Hf | 42.8 | 42.8 | ✓ |
| lid length | 83 | 83 | ✓ |

### Golden B (outside 132×102×63)

| Quantity | Hand calc | Expected | Match |
|----------|----------:|---------:|:-----:|
| Wi | 126 | 126 | ✓ |
| Uw | 120 | 120 | ✓ |
| Uh | 52.8 | 52.8 | ✓ |
| G | 120 | 120 | ✓ |
| Lw | 125.8 | 125.8 | ✓ |
| Rc | 92 | 92 | ✓ |
| Hf | 55.6 | 55.6 | ✓ |

Round-trip usable↔outside fixture passes for Golden A keys.

---

## 14. Storage and integration

| Item | Changed? |
|------|----------|
| `STORAGE_KEY` | No |
| `SCHEMA_VERSION` | No |
| `freshState()` / `persist()` / `backupObject()` | No (diff inspection) |
| Merge/replace import | No |
| Library / Projects / Inventory / Pricing / Test Grids | No |
| `designDraft` | Session-only module variable |
| Designs in JSON backup | Excluded (unchanged policy) |

---

## 15. UI audit (code inspection)

| Behavior | Status |
|----------|--------|
| Sliding fields only for `sliding-lid-box` | ✓ |
| Fixed Front open end / slide direction documented | ✓ |
| Usable vs wall-to-wall vs outside metrics distinct | ✓ |
| Four clearance concepts labeled separately | ✓ |
| Kerf exclusion stated | ✓ |
| Invalid → preview unavailable, download disabled | ✓ |
| Numeric `oninput` → `refreshDesignPreview()` only | ✓ |
| Template/mode change → full `render()` | ✓ |
| Pull width fields tied to pull mode | ✓ |
| Assembly summary in results | ✓ |
| No fit-coupon option | ✓ |

Narrow viewport and keyboard: not interactively tested.

---

## 16. Remaining LightBurn and physical tests

1. Import Golden A SVG into LightBurn; confirm 259 × 262.6 mm, eight groups, one cut layer.
2. Cut scrap; assemble five body panels; verify finger joints.
3. Glue rails top-flush and back-flush with spacer; keep glue out of channels.
4. Insert lid from Front; verify capture, slide, and Back stop.
5. Test Cs/Cv/Ci at 0 and 0.25 mm on scrap.
6. Apply kerf in LightBurn separately after nominal fit is understood.

---

## 17. Final `git status -sb`

```text
## main...origin/main
 M README.md
 M index.html
?? docs/SLIDING_LID_BOX_ARCHITECTURE_REVIEW_2026-07-15.md
?? docs/SLIDING_LID_BOX_PHASE3_IMPLEMENTATION_2026-07-15.md
?? docs/SLIDING_LID_BOX_INDEPENDENT_AUDIT_2026-07-15.md
(+ other pre-existing untracked docs/, LightBurn Projects/, parametric_qr_stand_generator.py)
```

---

## 18. Read-only safety statement

Confirmed:

- Implementation files (`index.html`, `README.md`) were **not** modified by the auditor
- Nothing staged, committed, or pushed
- Unrelated untracked files left untouched
- Only this audit report was created at user request

---

## Required conclusion

```text
SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP
```