# Designs Phase 2 Finger-Box Generator — Independent Architecture and Geometry Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Baseline commit:** `6a27579` — Add inventory-guided project wizard  
**Audit date:** 2026-07-15  
**Auditor:** Grok (read-only independent review)  
**Implementation reports reviewed:**
- `docs/DESIGNS_BOX_GENERATOR_ARCHITECTURE_REVIEW_2026-07-14.md`
- `docs/BOX_GENERATOR_PHASE2_IMPLEMENTATION_2026-07-14.md`

---

## 1. Executive conclusion

**Recommendation: `SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP`**

The finger-box generator is architecturally sound, stays inside approved session-only boundaries, preserves legacy Designs outputs byte-for-byte, and passes **614 / 0** live fixtures including **82 / 0** Designs geometry assertions. Independent code review confirms the dimension formulas, eight mating joint pairings, opposite finger phases, half-per-side clearance semantics, deterministic layout, and shared preview/download SVG path.

The implementation is **not** proven by fixtures alone for physical cut-and-assemble success. Fixtures validate 2D complementarity (shared boundaries + opposite phases + orthogonal paths) rather than folded 3D assembly. Scrap cut, LightBurn import, and real-material fit remain mandatory manual follow-up before production use.

No Critical or High defects were found. Medium findings are fixture-coverage and layout-bounding limitations, not demonstrated geometry failures in the default representative case.

---

## 2. Repository and baseline verification

### Commands run

```powershell
git status -sb
git rev-parse --short HEAD
git diff --check
git diff --stat
git diff --cached --name-only
```

### Results

| Check | Result |
|-------|--------|
| `HEAD` | `6a27579` ✓ |
| Staged files | None (`git diff --cached --name-only` empty) ✓ |
| Tracked modifications | `index.html`, `README.md` only ✓ |
| `git diff --stat` | `README.md` +8/−?; `index.html` +457/−21 (444 insertions, 21 deletions) |
| `git diff --check` | Clean (LF→CRLF warnings only) |
| Unrelated untracked files | Preserved (docs reviews, `LightBurn Projects/`, `parametric_qr_stand_generator.py`, etc.) |

This audit file was written after review completion at user request and was not part of the implementation diff.

---

## 3. Findings ordered by severity

### Medium

#### M1 — Layout overlap checks use nominal envelopes, not swept path bounds

| Field | Detail |
|-------|--------|
| **Region** | `layoutDesignPanels()` (~L1718–1733), fixture “Panel layout has no overlaps” (~L1907) |
| **Scenario** | Panels placed using `panel.width` / `panel.height` rectangles; overlap detection uses `translatedBounds` from those nominal sizes |
| **Expected** | Layout should ensure no cut-path protrusion crosses into another panel’s space |
| **Actual** | Finger/recess geometry is drawn inward from outer edges and remains inside each panel’s nominal rectangle for reviewed cases; final SVG validation parses actual path bounds. Layout itself does not compute path AABBs |
| **Why it matters** | A future edge-orientation or clearance bug could pass nominal-box layout while paths intrude into gutter gaps |
| **Recommended correction** | Add a fixture that computes each panel path’s axis-aligned bounds (post-translation) and asserts pairwise separation ≥ layout gap, or minimum gutter width |
| **New fixture required?** | Yes |

#### M2 — Fixtures prove 2D complementarity, not folded 3D assemblability

| Field | Detail |
|-------|--------|
| **Region** | `runDesignGeometryFixtures()` mating-pair assertions (~L1881–1886); `designEdgePoints()` / `buildBoxModel()` |
| **Scenario** | Representative boxes (120×90×50 mm, t=3, clearance 0.10, preferred finger 12 mm) and corner joints |
| **Expected** | Complementary 2D profiles should correspond to a physically assemblable open-top five-panel box |
| **Actual** | All eight joint pairs share identical `pattern.boundaries` and opposite `phase`; wall/bottom edge assignment is internally consistent (wall joint on patterned `edges[2]`, open rim on plain `edges[0]`). Conceptual 3D review supports coherence; no fold simulation exists |
| **Why it matters** | Opposite SVG phases are necessary but not sufficient for scrap-less assembly; material thickness, squareness, and kerf still matter |
| **Recommended correction** | None in code before commit; document mandatory scrap assembly verification |
| **New fixture required?** | Optional golden-vector or hand-verified assembly reference for one canonical box |

### Low

#### L1 — Several fixtures mirror implementation logic

| Field | Detail |
|-------|--------|
| **Region** | `runDesignGeometryFixtures()` dimension/pattern assertions (~L1868–1879, 1900–1904) |
| **Scenario** | Inside/outside conversion, odd segment counts, envelope sizes |
| **Expected** | Independent expected outcomes |
| **Actual** | Many assertions call the same helpers under test (`designBoxDimensions`, `buildFingerPattern`, `buildDesignResult`) |
| **Why it matters** | Reduces falsification strength; good as regression locks, weak as proof |
| **Recommended correction** | Add golden numeric tables for 2–3 hand-derived boxes |
| **New fixture required?** | Yes |

#### L2 — Segment-count transition boundaries not explicitly fixture-tested

| Field | Detail |
|-------|--------|
| **Region** | `buildFingerPattern()` (~L1621–1630) |
| **Scenario** | Edge lengths where optimal count transitions 3→5, 5→7, 7→9 (e.g., widths producing near-tie candidate differences) |
| **Expected** | Deterministic odd count closest to preferred width |
| **Actual** | Algorithm scans odd counts 3..maxOdd with strict `< difference - 1e-9` tie-break (lower count wins on ties). Default 126 mm width selects 11 segments — not boundary-tested |
| **Why it matters** | Transition bugs could pick wrong count at discrete thresholds |
| **Recommended correction** | Table-driven fixture with precomputed lengths at transition points |
| **New fixture required?** | Yes |

#### L3 — Per-side clearance semantics not quantified in fixtures

| Field | Detail |
|-------|--------|
| **Region** | `designEdgePoints()` clearance term (~L1647); fixtures (~L1888–1891) |
| **Scenario** | clearance = 0.10 mm on default box |
| **Expected** | Total mating gap = 0.10 mm via ±clearance/2 on finger vs recess transitions |
| **Actual** | Code applies `nominal ± clearance/2` along edge direction; fixtures only assert path changes, envelope preservation, and rejection at clearance=20 |
| **Why it matters** | Accidental doubling/halving would pass current tests |
| **Recommended correction** | Fixture comparing nominal vs cleared boundary coordinates for one segment on a known edge |
| **New fixture required?** | Yes |

#### L4 — Legacy hash documentation mismatch (non-functional)

| Field | Detail |
|-------|--------|
| **Region** | `designFixtureHash()` (~L1842–1845) vs implementation report SHA-256 table |
| **Scenario** | Legacy byte-stability verification |
| **Expected** | Consistent fingerprint method |
| **Actual** | Fixtures use 32-bit FNV-style hash + byte length; implementation report cites SHA-256. Fixtures pass with documented lengths |
| **Why it matters** | Documentation only; no stability regression detected |
| **Recommended correction** | Align docs or record both fingerprints |
| **New fixture required?** | No |

### Informational

#### I1 — Kerf is correctly excluded from geometry

`designEdgePoints()` uses material thickness for recess depth only, not kerf. UI and README distinguish joint clearance from LightBurn kerf offset. No silent kerf embedding found.

#### I2 — `designWallPath()` remains tray-only

Diff does not modify `designWallPath()` body. Tray/sign/stand generators route through `legacyDesignSvg()` → `buildLegacyDesignResult()` with added blocking validation. No tray geometry redesign detected.

#### I3 — Storage and integration boundaries unchanged

`STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`, `persist()`, `backupObject()`, merge/replace import paths unchanged in diff. `designDraft` remains module-level session state outside `state`. No Designs fields in backup payload.

#### I4 — Manual LightBurn and physical-cut verification not performed

No LightBurn instance was opened. No physical scrap assembly was performed.

---

## 4. Physical joint and assembly analysis

### Dimension definitions (verified in code)

`designBoxDimensions()` (~L1617–1619) implements:

```text
Inside mode:  Wo = Wi + 2t,  Do = Di + 2t,  Ho = Hi + t
Outside mode: Wi = Wo − 2t,  Di = Do − 2t,  Hi = Ho − t
```

Fixtures confirm default inside 120×90×50 / t=3 → outside 126×96×53 and outside round-trip stability.

### Panel model (`buildBoxModel`, ~L1705–1712)

| Panel | Nominal size | Jointed edges | Plain edges |
|-------|--------------|---------------|-------------|
| Bottom | Wo × Do | All four (width/depth patterns) | — |
| Front / Back | Wo × Ho | Vertical corners + top width edge (`edges[2]`) | Bottom rim `edges[0]` (open top) |
| Left / Right | Do × Ho | Vertical corners + top depth edge (`edges[2]`) | Bottom rim `edges[0]` |
| Lid (optional) | Wo × Do | None (all plain) | All |

**Flat-pattern convention:** Wall `edges[2]` (geometric top in SVG) is the bottom-to-wall joint when folded; `edges[0]` is the open rim. This matches the eight fixture mating pairs.

### Eight mating pairs (fixture-verified)

| Joint | Panel A | Panel B | Pattern | Phases |
|-------|---------|---------|---------|--------|
| Bottom ↔ Back (width) | `bottom.edges[0]` | `back.edges[2]` | width | opposite |
| Bottom ↔ Right (depth) | `bottom.edges[1]` | `right.edges[2]` | depth | opposite |
| Bottom ↔ Front (width) | `bottom.edges[2]` | `front.edges[2]` | width | opposite |
| Bottom ↔ Left (depth) | `bottom.edges[3]` | `left.edges[2]` | depth | opposite |
| Front ↔ Right (vertical) | `front.edges[1]` | `right.edges[3]` | vertical | opposite |
| Front ↔ Left (vertical) | `front.edges[3]` | `left.edges[1]` | vertical | opposite |
| Back ↔ Left (vertical) | `back.edges[1]` | `left.edges[3]` | vertical | opposite |
| Back ↔ Right (vertical) | `back.edges[3]` | `right.edges[1]` | vertical | opposite |

### Representative case: 120 × 90 × 50 mm inside, t=3, preferred 12 mm, clearance 0.10

| Metric | Value |
|--------|------:|
| Outside body | 126 × 96 × 53 mm |
| Width fingers | 11 segments × ~11.45 mm |
| Open-top panels | 5 |
| Layout (default) | 287 × 252 mm (per implementation report; validation confirms viewBox containment) |

**Conceptual assembly:** Five body panels sufficient. Bottom provides perimeter fingers; walls provide complementary patterns on bottom and vertical corners; open top uses plain rims. Inside cavity unobstructed by lid in open-top mode. Loose lid is separate plain Wo×Do panel; closed height adds one thickness (fixture-verified).

### Additional conceptual cases

| Case | Assessment |
|------|------------|
| Small valid box near minimums | Rejects nonpositive inside dims, outside ≤2t, edges too short for 3 min-width segments |
| Tall narrow box | Patterns scale with outside height; no separate height cap beyond warnings >200 mm |
| Wide shallow box | Same; large layout warning >400 mm |
| Outside-dimension mode | Fixture round-trip 126×96×53 outside → 120×90×50 inside |

**Corner overlap:** Front/back span Wo; left/right span Do; corners intentionally overlap by wall thickness with interlocking vertical fingers — standard finger-box convention. No evidence both panels leave a void or occupy the same solid volume in the 2D cut pattern.

**Residual risk:** Without physical assembly, squareness tolerance and kerf on real plywood remain unverified.

---

## 5. Clearance-semantics analysis

### Stated semantics (code + UI)

| Value | Behavior |
|-------|----------|
| `0.00 mm` | Nominal fit (no transition offset) |
| Positive | Looser fit; transitions shift symmetrically |
| Negative | Rejected (`designRequiredNumber`, minimum 0 inclusive) |
| Blank / text / non-finite | Rejected with specific errors |
| `Infinity` / non-finite | Rejected (“must be a finite number”) |
| Scientific notation | `Number()` parses (e.g. `1e-1` → 0.1) if finite |

### Implementation (`designEdgePoints`, ~L1647–1648)

Along each segment boundary between finger and recess:

- Finger side: `nominal − clearance/2`
- Recess side: `nominal + clearance/2`

**Total gap between mating transitions on opposite panels:** clearance (not doubled). Exterior panel envelopes unchanged (fixture: “Clearance preserves panel envelopes”).

### Collapse guard (`buildBoxModel`, ~L1695–1698)

`requiredWeb = max(0.5, t × 0.25)`. Rejects when `actualWidth − clearance < requiredWeb`. Fixture rejects clearance=20 on default box. Threshold is per-segment width minus clearance, not clearance > thickness alone.

### Kerf distinction

No kerf term in geometry. UI explicitly states kerf is separate from joint clearance. Legacy tray `fitClearance` is unrelated slot clearance for stand/tray templates.

---

## 6. Finger-width selection analysis

`buildFingerPattern()` (~L1621–1630):

| Rule | Status |
|------|--------|
| Odd segment count | Enforced (loop steps by 2 from 3) |
| Minimum 3 segments | Enforced (`maxOdd < 3` → error) |
| Minimum segment width `max(2t, 4 mm)` | Enforced via `maxOdd` |
| Closest to preferred width | Minimizes `|length/count − preferred|` |
| Deterministic | Fixed scan order; strict inequality tie-break |
| Positive actual width | `length / count` for valid length |

**Edge cases:**

| Input | Result |
|-------|--------|
| Preferred > edge length | Count=3, actualWidth=length/3 |
| Preferred < min width | Still picks valid odd count; may warn “unusually small preferred fingers” |
| Edge shorter than 3×minWidth | Error, null pattern |
| Very large preferred | Higher odd counts up to maxOdd |

Floating-point: boundaries rounded via `designRound()`; compact removes duplicate vertices.

---

## 7. Panel contour audit

`buildFingerPanel()` + `designPointsPath()`:

| Check | Result |
|-------|--------|
| Closed paths | All box paths end with `Z` (fixture) |
| Orthogonal only | H/V commands only; non-orthogonal returns `''` |
| Zero-length segments | `designCompactPoints()` deduplicates |
| Self-intersection | No explicit detector; inward recesses on rectangular panels reduce risk |
| Material-depth offset | Recess offset = `t` via `edge.depth` |
| Lid plain | Sixth panel all `plain` edges — no inherited joints |

---

## 8. SVG and LightBurn-readiness analysis

`serializeDesignSvg()` (~L1735–1736):

| Requirement | Status |
|-------------|--------|
| mm width/height attributes | Yes (`${value}mm`) |
| Matching viewBox | Validated |
| One red `#ff0000` cut group `id="cut"` | Yes |
| No CSS / external deps | Yes |
| Panel metadata via `<title>` only | Yes; no `<text>` in cut layer |
| Finite coordinates | `designNumberText`, validation rejects NaN/Infinity |
| Deterministic output | Fixture: repeated serialization byte-stable |
| Preview = download | Same `result.svg` in data-URL preview and download handler |

`designSvgValidation()` (~L1751–1773): DOMParser check, dimension/viewBox equality, finite path points, viewBox containment.

**Download path** (~L4951–4956): Rebuilds result, re-parses SVG; returns without download if invalid. Button disabled when `!result.valid`.

**LightBurn:** Not opened. SVG structure is consistent with prior proven templates (mm document, red stroke group, orthogonal paths).

---

## 9. Legacy regression results

| Template | Fixture length | FNV hash | Parse | ViewBox |
|----------|---------------:|----------|-------|---------|
| `qr-stand` | 359 | `fe737a09` | Pass | Pass |
| `hanging-sign` | 341 | `656e633d` | Pass | Pass |
| `dice-tray` | 1726 | `51a55721` | Pass | Pass |
| `divider-tray` | 1965 | `a55dda6e` | Pass | Pass |

Legacy outputs remain byte-stable per fixtures (FNV fingerprint). New blocking validation added for legacy templates without changing default SVG bytes. `designWallPath()` unchanged.

Invalid legacy inputs now block export (e.g. impossible hanging-hole inset, oversized stand slot) — behavioral improvement, not default-output change.

---

## 10. UI and validation audit (code inspection)

| Behavior | Status |
|----------|--------|
| Template change | `render()` on template/mode change; preserves `designDraft` fields |
| Numeric input partial refresh | `refreshDesignPreview()` updates `#designResults` only |
| Focus on numeric edit | Input outside replaced region — preserved by architecture |
| Specific errors | `normalizeDesignDraft` / `buildBoxModel` push explicit strings |
| Invalid → no preview | Error state shows “Preview unavailable” |
| Download disabled when invalid | `disabled` attribute + click guard |
| Reset defaults | `designDefaults()` + full `render()` |
| Unknown select values | Rejected (“Choose a recognized…”) |
| Errors vs warnings | Separate CSS classes `.error` / `.warning` |
| Inside/outside labels | Dynamic field labels + helper text |
| Kerf vs clearance | Separate UI paragraphs |
| Session-only | README + no persist hook on Designs |

Narrow viewport / keyboard: not interactively tested in this audit.

---

## 11. Fixture-quality assessment

### Live results (Edge headless, `--enable-logging=stderr`)

**`?selftest=design`:** 82 passed / 0 failed (via `?selftest=all` console group)

**`?selftest=all`:**

| Group | Passed | Failed |
|-------|--------|--------|
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| **Designs geometry** | **82** | **0** |
| **Total** | **614** | **0** |

### Strengths

- Eight joint pairs: boundaries + opposite phases
- Clearance envelope preservation and excessive-clearance rejection
- Closed orthogonal paths, finite coordinates
- Layout non-overlap (nominal), margins, gaps, document containment
- Duplicate translated cut-segment detection
- Legacy byte stability + parse checks
- SVG structure (one cut group, mm dims, viewBox)
- Draft nonmutation

### Gaps (recommended new fixtures)

1. Path AABB separation in layout gutters (M1)
2. Segment-count transition lengths at 3↔5↔7 boundaries (L2)
3. Quantified clearance offset on a single known boundary (L3)
4. Golden expected coordinates for one canonical box corner joint (L1/L2)
5. Self-intersection / zero-length segment detector on parsed paths
6. Download-handler integration test that invalid geometry cannot bypass disabled state via forced click simulation

Existing tests were not weakened or removed in the reviewed diff.

---

## 12. Exact validation commands and totals

| Command / check | Result |
|-----------------|--------|
| `git status -sb` | `M README.md`, `M index.html`; HEAD `6a27579` |
| `git diff --check` | Pass (LF warnings) |
| `python -m html.parser index.html` | Pass |
| `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` (Edge headless) | **614 / 0** |
| Designs subgroup | **82 / 0** |
| Runtime console errors during selftest | None observed |
| LightBurn import | **Not performed** |
| Physical scrap assembly | **Not performed** |
| Interactive UI / narrow viewport | **Not performed** |

---

## 13. Remaining manual LightBurn and physical-cut checks

Before trusting production cuts:

1. Import open-top and loose-lid default SVGs into LightBurn; confirm document size, single cut layer, and path closure.
2. Cut scrap material at t≈3 mm; dry-assemble; verify inside 120×90×50 mm cavity and square corners.
3. Repeat at clearance 0, 0.10, and 0.25 mm; measure slot fit.
4. Apply kerf offset in LightBurn only after nominal fit is understood.
5. Confirm sheet layout fits available stock for large boxes triggering layout warnings.

---

## 14. Final `git status -sb`

```text
## main...origin/main
 M README.md
 M index.html
?? docs/BOX_GENERATOR_PHASE2_IMPLEMENTATION_2026-07-14.md
?? docs/DESIGNS_BOX_GENERATOR_ARCHITECTURE_REVIEW_2026-07-14.md
?? docs/DESIGNS_BOX_GENERATOR_INDEPENDENT_AUDIT_2026-07-15.md
(+ other pre-existing untracked docs/, LightBurn Projects/, parametric_qr_stand_generator.py)
```

---

## 15. Read-only safety statement

Confirmed during this audit:

- **No** implementation files (`index.html`, `README.md`) were edited by the auditor
- **Nothing** was staged, committed, pushed, stashed, reset, or cleaned
- Unrelated untracked files were left untouched
- Only this audit report was created at user request

---

## Audit conclusion (required single recommendation)

```text
SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP
```