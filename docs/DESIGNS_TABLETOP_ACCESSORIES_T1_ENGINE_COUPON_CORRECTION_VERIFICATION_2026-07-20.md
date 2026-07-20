# Tabletop Accessories T1 Engine + Coupon — Correction Verification

**Date:** 2026-07-20  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Mode:** Strictly read-only (no edit, stage, commit, push, reset, clean, stash, checkout, move, rename, or delete of product files; this report is the sole deliverable write).  
**Committed baseline:** `7876551` — Add Dice Tray storage and insert options  
**Scope:** Re-verify that the focused-audit blocker and important findings were corrected, and that the coupon is safe for controlled exploratory scrap cutting only.

**Authority / history:**
- `docs/DESIGNS_TABLETOP_ACCESSORIES_ARCHITECTURE_REVIEW_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_CONSTRUCTION_CONTRACT_REVIEW_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_IMPLEMENTATION_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_FOCUSED_AUDIT_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_CORRECTION_2026-07-20.md`

---

## Exact final verdict

**APPROVED FOR CONTROLLED PHYSICAL COUPON CUT**

| Severity | Count |
|----------|------:|
| BLOCKER | 0 |
| IMPORTANT | 1 |
| POLISH | 2 |

| Question | Answer |
|----------|--------|
| Form blocker fixed? | **Yes** |
| Controlled exploratory scrap cutting permitted? | **Yes** (same measured stock; dry-fit; not production/sold use) |
| Production / sold-product use? | **No** |
| May T1 be committed? | **Yes, after optional pre-commit fix of the IMPORTANT summary NaN** (or commit with known disclosure issue documented) |
| T2 blocked until physical results? | **Yes** |
| Claude full audit needed? | **No** |
| Files edited/staged/committed/pushed by this verification? | **No** (report only) |

---

## 1. Repository state

| Check | Result |
|-------|--------|
| `git status -sb` | `## main...origin/main`; `M CHANGELOG.md`, `M README.md`, `M index.html` |
| `git log -1 --oneline` | `7876551 Add Dice Tray storage and insert options` |
| `git rev-parse HEAD` | `78765512f58d5a16eea89073958252bd5171b6a7` |
| `origin/main...main` | `0 0` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --cached` | Empty |
| `git diff --stat` | `CHANGELOG.md` +2; `README.md` +7/−; `index.html` +289/−19 (≈279/+19 net) |
| HEAD remains `7876551` | **Yes** |
| Branch `main` | **Yes** |
| T1 uncommitted | **Yes** |
| Tracked delta limited to T1 intent | **Yes** (`index.html`, `README.md`, `CHANGELOG.md`) |
| Architecture / construction / implementation / audit / correction reports | **Present** (untracked under `docs/`) |
| LightBurn tree, historical docs, `debug.log`, utility script | Untracked; untouched by T1 |

---

## 2. Files / functions reviewed

| Area | Symbols / paths |
|------|-----------------|
| Form | `renderDesigns`, `tabletopFields`, `formFields` ternary, tabletop `note` |
| Normalize | `normalizeDesignDraft` tabletop branch (step/spacing `> 0`) |
| Engine | `tabletopAccessoryDimensions` (silent-shrink), divider/insert helpers, manufacturing limits, projection |
| Coupon | `buildTabletopCornerFloorCouponDesignResult`, mating descriptors, terminal webs, SVG serialize |
| Summary | tabletop `designMetric` block in results HTML |
| Help | `openHelp` / `tabletopHelpSection`; Help fixtures |
| Fixtures | `runTabletopAccessoryEngineFixtures` (27), `runTabletopCornerFloorCouponFixtures` (67) |
| Selftest | `tabletop-engine`, `tabletop-corner-floor-coupon`, `help`, `all` |
| Docs | README, CHANGELOG, implementation + correction reports |

---

## 3. Form blocker re-verification

**Original blocker:** first `tabletop` arm of `formFields` returned a note-only string, making controls unreachable.

**Current code:** single reachable arm:

```text
tabletop ? `${templateField}${materialThicknessField}${tabletopFields}` : …
```

**Runtime proof (in-app fixture, not a false `window.designDraft` mutation):**  
`runTabletopCornerFloorCouponFixtures` asserts form presence and passes **67 / 0**, including:

- tabletop form branch renders actual controls  
- template selector remains present  
- measured thickness, units, center clearance, clearance step, preferred finger width, interior leg, wall height, panel spacing, assembly labels, kerf reference  

Controls named in the fixture:  
`template`, `materialThickness`, `tabletopCouponUnits`, `tabletopCouponCenterClearance`, `tabletopCouponClearanceStep`, `tabletopCouponPreferredFingerWidth`, `tabletopCouponInteriorLeg`, `tabletopCouponWallHeight`, `tabletopCouponPanelSpacing`, `assemblyLabels`, `tabletopCouponKerf`.

Download and Project handoff actions remain on the Designs form shell (`downloadDesignSvg`, `startProjectFromDesign`). Preview mode for Finished View is driven by the Designs results chrome (tabletop finished-view routing remains registered).

**2.88 mm entry via builder with form-equivalent draft fields:** valid; candidates **0.05 / 0.10 / 0.15**.

**Dice Tray → Tabletop round-trip:** fixture path restores draft/template; form branch re-renders controls (fixture cleanup + form assertions).

**Form blocker status:** **CLEARED** (not a BLOCKER).

---

## 4. Tabletop note verification

`note` is tabletop-first:

> “This three-piece construction coupon tests one 90-degree wall corner and both wall-to-floor joints together…”

Fixture asserts that text is present and freestanding-sign fallback text is **absent**.

Muted in-form copy still states three-piece WALL-A / WALL-B / FLOOR, finger-jointed floor, no lid/divider/insert/rebate/groove/pocket.

**Status:** **CLEARED**.

---

## 5. Help verification

Help modal includes section **Tabletop Accessories T1** with required concepts:

| Concept | Present |
|---------|---------|
| Separate from legacy Dice Tray | Yes |
| Rectangular finger-jointed wall corners + floor | Yes |
| Future hex uses different non-90° strategy | Yes |
| Coupon tests wall corner + both floor joints | Yes |
| Clearance starting point; measured thickness; kerf separate; dry-fit | Yes |
| No groove/rebate/bevel/dado/pocket/CNC | Yes |
| Flat material, focus, framing, ventilation, fire watch, suppression, unattended | Yes (section + global laser safety) |

Help and Safety fixtures: **40 passed / 0 failed** (includes three Tabletop-specific assertions). Modal accessibility / Help identity fixtures remain green in the complete suite.

**Status:** **CLEARED**.

---

## 6. Representative 2.88 mm evidence

| Quantity | Measured |
|----------|----------|
| Valid | `true` |
| Candidates | **0.05, 0.10, 0.15** mm |
| Pieces | **9** |
| IDs | `C1-FLOOR|C1-WALL-A|C1-WALL-B|…|C3-*` unique |
| Requested interior | 40 × 40 mm |
| Finished cavity clearBounds | 40 × 40 mm; usable height **22** |
| Outer envelope | **45.76 × 45.76 × 24.88** |
| Wall blank height | **24.88** (= 22 + t); floor blank **45.76** |
| Preferred finger | 9 mm |
| Actual horizontal/floor | **9.152** mm (derived; all candidates) |
| Actual vertical/corner | **≈8.293** mm (derived) |
| Minimum segment max(2t,4) | **5.76** mm; all actual ≥ minimum |
| Sheet | **187.28 × 187.28** mm |
| Deterministic SVG | yes; FNV `4f543f95`; 2992 bytes |

---

## 7. Wall-to-wall mating

C2 (center clearance 0.10) mating descriptors:

| Pair | Phases | Count | Actual width |
|------|--------|------:|-------------:|
| WALL-A vertical ↔ WALL-B vertical | true vs false (**opposite**) | 3 = 3 | 8.293… = 8.293… |

---

## 8. Wall-to-floor mating

| Pair | Phases | Count | Actual width |
|------|--------|------:|-------------:|
| WALL-A bottom ↔ FLOOR matching edge | opposite | 5 = 5 | 9.152 = 9.152 |
| WALL-B bottom ↔ FLOOR matching edge | opposite | 5 = 5 | 9.152 = 9.152 |

No fourth wall required; only WALL-A, WALL-B, FLOOR per candidate.

---

## 9. Terminal web

| Field | Value |
|-------|------:|
| `terminalWebs.minimum` | **2.88** mm (> 0) |
| `terminalWebs.wallWall` | 0 |
| `terminalWebs.floor` | 0 |

Fixture requires minimum &gt; 0 and junction values ≥ 0. No zero-area blanks; closed unique cut paths. Junction zeros are non-negative edge-terminal offsets from the shared finger helper, not negative webs.

---

## 10. Dimension and cavity contract

Distinct metrics fields present: `requestedInterior`, `finishedCavity`, `outerEnvelope`, `panelDimensions`, `cutBounds`.

Interior-primary silent shrink with requested 100 and finished 96.73:

- `valid === false`
- Error: *Finished cavity cannot be smaller than requested interior in interior-primary mode without explicit allowed intrusion metadata.*

Coupon interior-primary path keeps finished clear bounds equal to requested leg (40).

---

## 11. Divider / insert regression

| Check | Result |
|-------|--------|
| Divider span from 96.73 − 2×1 | **94.73** |
| Source | `finishedCavity` |
| Insert region from cavity − clearance | 95.93 × 79.2 |
| physical true → intent `cut-panel` | Yes |
| physical false → intent `measurement-only` | Yes |
| wood vs felt region equality | Yes (tag opaque) |
| Half-thickness as clear cavity | Not reported as cavity |

---

## 12. Validation boundaries

| Case | Outcome |
|------|---------|
| Zero / negative clearance step | **Blocked** |
| Zero / negative panel spacing | **Blocked** |
| Short leg 10 / short height 4 | **Blocked** |
| Borderline leg 18 / height 14.4 | **Valid** |
| Preferred width &lt; 5.76 | **Blocked** |
| Preferred width = 5.76 | **Valid** |
| Candidate clearances outside −0.10…0.30 | **Blocked** (all three checked) |
| Nonfinite interior | **Blocked** |
| Inch-mode 2.88 equivalent | **Valid** |
| Rounded-duplicate candidates | Blocked by distinctness check when step collapses |

---

## 13. Result-summary disclosure

Measured summary content for 2.88 case includes:

- Candidates 0.05 / 0.10 / 0.15  
- Requested interior 40 mm legs  
- Usable wall height 22  
- Outer envelope 45.76 × 45.76 × 24.88  
- Wall/floor blanks honest  
- Finger widths: Preferred 9 / horizontal-floor **9.152** / vertical-corner **≈8.293** / minimum **5.76**  
- Physical pieces 9 / sheets 1  
- Assembly labels count  
- Production SVG size 187.28 × 187.28  
- Exact-scale / dry-fit style messages  

**IMPORTANT defect:**  
`Finished cavity` is rendered as **`NaN × NaN mm`** because the summary uses `finishedCavity.width` / `.depth` while the stored object exposes dimensions under **`finishedCavity.clearBounds`**. Authoritative metrics and fixtures use `clearBounds` correctly; only the operator-facing summary line is wrong. Requested interior and outer envelope remain correct, and production SVG is unaffected.

There is no separate labeled “Material thickness” metric line (thickness is present in metrics and implicit in blanks); preferred/actual finger disclosure is present.

---

## 14. Production SVG

| Check | Status |
|-------|--------|
| Exact mm width/height/viewBox | 187.28 |
| Nine panel groups | Yes |
| Red `#ff0000` cut group | Yes |
| No blue without guides | Yes |
| No Finished View IDs | Yes |
| Closed contours | Yes |
| No NaN/Infinity in production SVG | Yes |
| Deterministic | Yes |

---

## 15. Panel layout

Nine panels; fixture asserts non-overlapping AABBs and **15 mm** separation; sheet matches layout bounds.

---

## 16. Labels

| Mode | Group | Paths | labelCount | Red |
|------|-------|------:|-----------:|-----|
| Off | absent | 0 | 0 | present |
| On | `assembly-labels` green | **9** | **9** | present |

Parity restored (path count equals labelCount). Labels separate from cut group.

---

## 17. Finished View

- Finite coordinates; 9 polygons; min area &gt; 0  
- Floor class first; non-coplanar wall faces  
- C1/C2/C3 readable  
- No groove/rebate/channel/bevel/lid/divider/insert implication  
- Malformed projection points rejected  
- FV IDs absent from production SVG  

---

## 18. Engine assertion inventory (27 / 0)

Measured **27 passed / 0 failed**. Coverage includes:

- Rectangle positive winding, size, ordered links, unit outward normals, finger-90 corner metadata  
- Hex winding, formulas, unit normals, six-edge wrap, 120° metadata, no finger-90 on hex  
- Arbitrary edge count (pentagon loop length 5)  
- Distinct dimension fields; full thickness not clear cavity  
- Interior-primary silent shrink invalidation  
- Divider 96.73 → 94.73; clearance formula; not nominal 100  
- Insert measurement-only vs cut-panel intent; opaque tags  
- Manufacturing min 5.76; half-thickness not cavity  
- Projection finite, deterministic, malformed rejection  
- Premium rectangle remains wall-to-wall (not floor-only registration)  

---

## 19. Coupon assertion inventory (67 / 0)

Measured **67 passed / 0 failed**. Coverage includes:

- Registry; form controls and template selector; tabletop note / no freestanding fallback  
- 2.88 case; candidates; nine pieces; IDs; roles  
- Actual width manufacturability; summary preferred/actual strings  
- Requested = finished clear bounds; outer = cavity + 2t; usable vs blank height  
- Wall–wall and wall–floor opposite phases, counts, widths  
- Terminal webs; no fourth wall  
- Layout AABBs + spacing; deterministic SVG; closed unique cuts  
- Red/green; labels off/on parity; FV geometry  
- Validation matrix; inch mode; handoff; cleanup restore  

---

## 20. Fixture cleanup

Coupon fixture snapshots and restores `designDraft`, `designPreviewMode`, `state.activeTab`, and verifies `localStorage` unchanged; uses parsed `renderDesigns()` HTML rather than mutating live form permanently; finally restores original draft. Engine fixtures are pure. **Cleanup assertions pass.**

---

## 21. Selftest registration

| Route | Count in source |
|-------|-----------------|
| `tabletop-engine` under `all` | **1** |
| `tabletop-corner-floor-coupon` under `all` | **1** |

---

## 22. Exact measured totals

Independently measured (disposable Edge harness on working tree):

| Group / standalone | Passed | Failed |
|--------------------|-------:|-------:|
| tabletop-engine | **27** | 0 |
| tabletop-corner-floor-coupon | **67** | 0 |
| help | **40** | 0 |
| design | **1093** | 0 |
| dice-tray-system | **92** | 0 |
| gift-box | **69** | 0 |
| tray standalone | **264** | 0 |
| machines-m1 | **29** | 0 |
| machines-m2 | **31** | 0 |
| machines-m3 | **27** | 0 |
| promotion-switch | **16** | 0 |
| **Complete suite (31 groups)** | **2551** | **0** |

Matches correction-report claims.

---

## 23. Complete-suite arithmetic

Measured **2551** across **31** groups.  
Relative to post-T1 pre-correction **2478**: +15 engine +55 coupon +3 Help = **+73** → **2551**.  
Consistent with measured group table.

---

## 24. Legacy-output comparison

Legacy builder function bodies remain **byte-identical to HEAD** (`buildBoxModel`, tray/gift/sliding/drawer/joint/box/legacy builders, `layoutDesignPanelRows`, `buildFingerPattern`).

Working-tree FNV pins (match prior audit pins):

| Product | Bytes | FNV |
|---------|------:|-----|
| Dice Tray | 1726 | `51a55721` |
| Dice Tray tab-slot | 1054 | `41697123` |
| Divider Tray | 1965 | `a55dda6e` |
| Joint Fit Coupon | 7764 | `db7ea7e9` |
| Finger Box (labels on) | 4470 | `3aa538d3` |
| Gift Box (labels on) | 6859 | `647c8367` |
| Sliding-lid (labels on) | 5045 | `85247b6f` |
| Drawer Cabinet (labels on) | 8194 | `1c009e7f` |

No unexplained golden drift.

---

## 25. Documentation verification

| Item | Status |
|------|--------|
| README measured 2551 / 31 | **Yes** |
| README lists both T1 selftest routes | **Yes** |
| README: groundwork + coupon only; physical still required | **Yes** |
| CHANGELOG bounded + correction bullet | **Yes** |
| Implementation report records form blocker history + measured totals | **Yes** |
| Correction report matches runtime evidence | **Yes** (except residual summary NaN not called out there) |
| Finger-jointed floor terminology | Consistent |
| No full tray / hex product claim | **Yes** |

---

## 26. Direct file:// results

| Check | Result |
|-------|--------|
| `python -m html.parser index.html` | exit **0** |
| `git diff --check` | clean |
| Headless Edge probe | startup OK; fixtures OK; no probe exceptions |
| Form wiring | Correct in source + coupon fixtures |
| 2.88 geometry / SVG / labels / FV | OK |
| Help section | OK |
| Complete suite | 2551 / 0 / 31 |

Note: an external probe that assigns only `window.designDraft` without the closure `designDraft` cannot drive `renderDesigns`; in-app fixtures exercise the real variable and pass.

---

## 27. Protected boundaries

| Item | Status |
|------|--------|
| APP_ID / VERSION 0.9.0 / BUILD_DATE 2026-07-19 | Unchanged |
| STORAGE_KEY / SCHEMA_VERSION 2 / BACKUP_FORMAT | Unchanged |
| Machines M1–M3, promotion, storage | Suites green |
| Project / accounting / inventory / pricing | Untouched |
| Legacy templates / coupons / kerf / offline | Protected |

---

## 28. Findings by severity

### BLOCKER (0)

None. Form controls render; production geometry is coherent; scale is exact mm; no open/duplicate production cuts; no legacy drift.

### IMPORTANT (1)

1. **Result summary “Finished cavity” shows `NaN × NaN mm`.**  
   Cause: `designMetric('Finished cavity', designDimensionsText(result.metrics.finishedCavity.width, result.metrics.finishedCavity.depth))` while metrics store clear size under `finishedCavity.clearBounds`.  
   Does **not** corrupt SVG or mating; **does** mis-disclose finished cavity in the UI.  
   Fix before or immediately after commit: use `clearBounds.width/depth` (or dual-field access). Prefer a fixture that asserts the summary text contains `40` for finished cavity, not `NaN`.

### POLISH (2)

1. No dedicated “Material thickness” metric line in the tabletop summary (thickness is in metrics and blanks).  
2. Junction `terminalWebs.wallWall` / `floor` report 0 while minimum web is positive — acceptable per fixture, but less informative than an explicit positive junction web if one is intended.

### NOT A DEFECT

- Unused finger profiles on non-mated floor edges (full open-box reuse).  
- No complete tray / hex product.  
- Physical fit/kerf still require scrap validation.

---

## 29. Exact final verdict (restated)

**APPROVED FOR CONTROLLED PHYSICAL COUPON CUT**

Original form blocker and the six important audit themes are corrected at implementation and fixture level. One residual IMPORTANT disclosure bug remains (Finished cavity NaN in summary only).

---

## 30. Remaining physical uncertainty

- Real kerf and joint fit on measured stock  
- Squareness after dry-fit and glue  
- Triple-junction behavior under machine kerf  
- Clamp/glue workflow  
- Long-term strength  

---

## 31. Whether controlled scrap cutting is permitted

**Yes**, for exploratory scrap on the same measured sheet:

- Use measured thickness (e.g. 2.88 mm) and the three clearance candidates  
- Import production SVG at **100%** scale  
- Dry-fit before glue  
- Do **not** treat results as production or sold-product proof  

---

## 32. Whether T1 may be committed

**Yes, T1 may be committed** as corrected construction groundwork + coupon, preferably after fixing the Finished cavity summary NaN (one-line display fix + fixture).  
Physical scrap testing may proceed before or after that fix; production SVG is already valid.

---

## 33. Whether T2 remains blocked until physical results

**Yes.** T2 (complete rectangle product / further accessories) should wait for scrap dry-fit feedback from this coupon family.

---

## 34. Whether Claude is needed

**No.** Residual issue is a local metrics-to-summary field path; shared legacy builders are unchanged.

---

## 35. Confirmation of verification hygiene

This verification did **not** edit, stage, commit, push, reset, clean, stash, checkout, move, rename, or delete any repository product file.  
Deliverable write only:

`C:\Genmitsu L8 Tracker\docs\DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_CORRECTION_VERIFICATION_2026-07-20.md`

Temporary Edge profiles under OS temp and worktree probe scripts were measurement-only.

---

## Appendix — Original finding disposition

| Original finding | Disposition |
|------------------|-------------|
| Form note-only branch (BLOCKER) | **Fixed** |
| Missing Help section | **Fixed** |
| README totals / selftest routes | **Fixed** (2551 / 31; both routes listed) |
| High-risk fixture gaps | **Fixed** (27 + 67 direct asserts) |
| Actual finger widths omitted from summary | **Fixed** (preferred / horizontal / vertical / min) |
| Implementation report overstated form | **Updated** (blocker history + correction) |
| Freestanding-sign fallback note | **Fixed** |
| Silent shrink; step/spacing; mating/layout/labels/FV/cleanup contracts | **Implemented + fixture-covered** |
| Finished cavity summary NaN | **New residual IMPORTANT** |

---

*End of correction verification.*
