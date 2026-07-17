# Designs Drawer Cabinet Shelf-placement guides — Focused Audit (2026-07-17)

**Mode:** Read-only adversarial audit (no file edits, no git writes, no staging, no LightBurn / debug / utility changes).  
**Primary app path:** `C:\Genmitsu L8 Tracker`  
**Feature under audit:** Optional **Shelf-placement guides** for Drawer Cabinet (session-only, off by default).  
**Primary reports (not trusted alone):**  
- `docs/DESIGNS_DRAWER_CABINET_SHELF_GUIDES_DESIGN_REVIEW_2026-07-17.md`  
- `docs/DESIGNS_DRAWER_CABINET_SHELF_GUIDES_IMPLEMENTATION_2026-07-17.md`  
**This document:** Independent verification of working-tree implementation vs baseline `a36d7ea`.

---

## 1. Actual HEAD, baseline, working-tree state

| Check | Result |
| --- | --- |
| `git log -1 --oneline` | **`a36d7ea` Add wall-to-base tab clearance coupon** |
| Matches expected baseline `a36d7ea`? | **Yes** |
| `git status -sb` | `## main` · `M index.html` · `M README.md` · many **untracked** docs/reports, LightBurn, debug, `.claude/`, utilities (pre-existing; not part of this feature scope) |
| Staged files | **None** (`git diff --cached --name-only` empty) |
| Tracked modified (feature scope) | **`index.html`**, **`README.md` only** — no unexpected tracked file changed |
| Unrelated untracked reports / LightBurn / debug / utilities | **Untouched by this audit** (left as-found) |
| `git diff --check` | **Clean** (no whitespace errors) |
| `git diff --stat` | `README.md` 6 ± · `index.html` 67 ± · **2 files changed, 61 insertions(+), 9 deletions(-)** |
| `git diff --numstat` | README `3 3` · index `58 6` · **totals 61 / 9** — **reconciles** with `--stat` |
| Read-only status | **Confirmed** — this audit only **created** this report file under `docs/` |

**Expected new untracked report:** `docs/DESIGNS_DRAWER_CABINET_SHELF_GUIDES_IMPLEMENTATION_2026-07-17.md` — present among untracked docs.  
**This audit report:** `docs/DESIGNS_DRAWER_CABINET_SHELF_GUIDES_FOCUSED_AUDIT_2026-07-17.md` (new).

---

## 2. Complete diff-hunk enumeration

### 2.1 `README.md` (numstat 3 / 3)

| Hunk | Classification | Content |
| --- | --- | --- |
| README ~L19 | **README documentation** | Feature blurb: optional **Shelf-placement guides** on Left / Right / Back; blue score; edge glue still required; first cabinet confirms face + top/bottom. |
| README ~L89 | **README documentation** | Fixture totals **906** Designs geometry / **1698** complete suite (was 887 / 1679). |
| README ~L103 | **README documentation** | “including Shelf-placement guides” in validation sentence. |

No unexpected README topics (no storage/schema/promotion claims).

### 2.2 `index.html` (numstat 58 / 6)

Classified against full `git diff -- index.html` (hunks in approximate source order):

| # | Region (approx.) | Classification | Summary |
| --- | --- | --- | --- |
| 1 | `designDefaults` | **Drawer Cabinet form/UI (defaults)** | `drawerShelfGuides: false` |
| 2 | Drawer Cabinet form HTML | **Drawer Cabinet form/UI** | Checkbox + help: marks on Left/Right/Back interiors; upper-shelf elevations; score-before-cut; edge glue; positioning aids; first-cabinet orientation confirm |
| 3 | `normalizeDrawerCabinetDesign` | **normalization** | `shelfGuides` only for `true` / `'true'` / `'on'` |
| 4 | New `drawerCabinetShelfGuideSegments` | **guide helper** | Elevations from `model.metrics.shelfBottomElevations`; ticks on Left/Right/Back; no second shelf formula |
| 5 | `buildDrawerCabinetDesignResult` | **Drawer Cabinet result/model integration** | When `shelfGuides`: scorePaths + `scoreGroupId: 'shelf-guides'`; metrics line; one-row warning |
| 6 | `serializeDesignSvg` | **serializer compatibility** | `designSerializedPathGroup(scorePaths, result.scoreGroupId \|\| 'rail-guides')` |
| 7 | Sliding-Lid preview fixture | **Sliding-Lid test-harness correction** | Isolate live `#designResults` id during temporary fixture DOM |
| 8 | Form change handler | **Drawer Cabinet form/UI** | `drawerShelfGuides` in live re-render name list |
| 9–N | Self-tests | **fixtures** | Shelf-guides block + goldens + Sliding-Lid serializer pin + suite totals 906 / 1698 |
| metrics | `result.metrics` | **result metrics/warnings** | `Shelf-placement guides: Disabled` or `Enabled - N marks` |

**Finished View note:** **Not present** in this diff (no Finished View production dependency).  
**Unexpected / unrelated production hunks:** **None.**

**Insertion/deletion reconciliation:** 58 insertions + 6 deletions in `index.html` match `git diff --numstat`. Net feature is additive (helper + UI + fixtures + optional score path); production cut path for disabled Drawer Cabinet is structurally unchanged (empty `scorePaths` → no score group).

---

## 3. Files / functions inspected

| Area | Items |
| --- | --- |
| App core | `index.html` (working tree + `git show a36d7ea:index.html` for key symbols) |
| Docs | README; design review; implementation report; prior Drawer Cabinet architecture / implementation / Finished View / assembly-label reports (context) |
| Defaults / UI | `designDefaults`, Drawer Cabinet form checkbox, help text |
| Normalize | `normalizeDrawerCabinetDesign` (`shelfGuides`) |
| Model (unchanged) | `buildDrawerCabinetModel`, `drawerCabinetDimensions`, `drawerCabinetCabinetOutsideHeight`, shelf elevation formula in model metrics |
| Guides | `drawerCabinetShelfGuideSegments` |
| Result | `buildDrawerCabinetDesignResult` |
| Serializer | `serializeDesignSvg`, `designSerializedPathGroup`; all `scorePaths` / `scoreGroupId` call sites |
| Sliding Lid | `buildSlidingLidBoxDesignResult`, rail guides, preview fixture isolation |
| Finished View | `buildDrawerCabinetFinishedViewModel` — no scorePaths / guides |
| Storage isolation | `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState`, `loadState`, `persist`, backup/replace/merge — **absent from diff** |
| Fixtures | Shelf-guides suite; goldens; Sliding-Lid serializer; suite total counting |

---

## 4. Implementation contract (verified)

| Contract item | Verdict |
| --- | --- |
| User-facing name | **Shelf-placement guides** |
| Raw draft field | **`drawerShelfGuides`** |
| Normalized / metric key | **`shelfGuides`** |
| Score subgroup | **`shelf-guides`** via `scoreGroupId` |
| Helper | **`drawerCabinetShelfGuideSegments`** |
| Session-only / off by default | **Yes** (`false` default; not in storage/schema) |
| Enable values | **only** `true`, `'true'`, `'on'` → true; else false |
| One-row + enabled | Valid result; **0** entries; **0** segments; warning *“1-row cabinet has no upper shelf to mark”*; metric **`Enabled - 0 marks`** (honest with warning; not “misleading geometry”) |

---

## 5. Independent geometry derivation

### 5.1 Source of truth — no duplicated shelf math

Helper uses **only**:

```text
elevations = model.metrics.shelfBottomElevations
localY = cabinetOutsideHeight - thickness - shelfBottomElevation
```

Model still computes (unchanged from baseline logic):

```text
cellHeight = drawerOpeningHeight + 2 * drawerVerticalClearance
shelfBottomElevation = thickness + i * (cellHeight + thickness)   // i = 1 .. rows-1
cabinetOutsideHeight = thickness + rows * cellHeight + (rows - 1) * thickness + thickness
```

**No second** cell-height / row-pitch / support-shelf formula was added in the guide helper. Arithmetic involving `drawerRows` / openings in the **new** code is limited to reading model metrics and panel bounds.

### 5.2 Default two-row numbers (hand derivation)

Defaults: thickness **3**, drawer **100×70×30**, clearances **0.45 / 0.30 / 0.50**, rows **2**.

| Quantity | Formula | Value |
| --- | --- | --- |
| `cellHeight` | 30 + 2×0.3 | **30.6** |
| `cabinetOutsideHeight` | 3 + 2×30.6 + 1×3 + 3 | **75.6** |
| `shelfBottomElevation` (one upper shelf) | 3 + 1×(30.6+3) | **33.3** |
| `localY` | 75.6 − 3 − 33.3 | **39.3** |
| Side panel width `W` | `cabinetOutsideDepth` = 70 + 2×0.5 + 2×3 + 2×1.25 | **79.5** |
| `edgeInset` | max(2, 3) | **3** |
| `joineryInset` (sides) | thickness | **3** |
| `tickLength` | max(6, 6) | **6** |
| Left tick starts | joinery+edge = 6; W−joinery−edge−tick = 67.5 | — |
| Left ticks | (6,39.3)–(12,39.3) and (67.5,39.3)–(73.5,39.3) | **Confirmed** |

Report claims for 75.6 / 3 / 33.3 / 39.3 and Left tick endpoints: **accepted after independent derivation**.

### 5.3 Orientation convention (two claims, kept separate)

| Claim class | Assessment |
| --- | --- |
| **Software coordinate consistency** | Front elevation panel uses **y = 0 at top**, increasing downward (`drawerCabinetFrontElevationPanel`). Cabinet side/back outer paths use the same vertical extent and the same `localY` formula. Guide y matches that convention. |
| **Physical top/bottom certainty** | **Not proven by software.** Help text correctly requires first physical cabinet to confirm marked face and top/bottom. Software may be consistent while physical assembly orientation remains a shop check. |

---

## 6. Guide shape and mirror-invariance proof

### 6.1 Helper I/O

- **Inputs:** `model`, `panels`, `materialThickness`  
- **Output:** array of `{ id, title, ownerPanelId, segments: [{x1,y1,x2,y2}, ×2] }`  
- **Owners:** Left, Right, Back only (not Top/Bottom/Front)  
- **Rounding:** `roundDesign(n, 2)` — deterministic  
- **Finite:** insets and tick length from max(constants, thickness) — finite for valid thickness  

### 6.2 Containment / clearance

For side panels: ticks start at `joineryInset + edgeInset` and end `tickLength` later; right tick ends at `W - joineryInset - edgeInset`. With W=79.5, t=3: ticks stay **inside** panel and clear toothed ends by **joineryInset**. Back uses `joineryInset = 0` (full width with edge inset only) — appropriate for back panel tooth layout as implemented.

Left and Right share the **same** `localY` (same elevation list) — same physical shelf height.

### 6.3 Mirror invariance (set equality, not helper-vs-helper)

Panel width **W = 79.5**. Transform every x by `x → W − x`:

| Original segment | Transformed |
| --- | --- |
| x 6 → 12 | 73.5 → 67.5 | **same segment reversed** |
| x 67.5 → 73.5 | 12 → 6 | **same segment reversed** |

Unordered pair of segments is **identical** under mirror. Fixture that only re-calls the helper is weaker; **literal arithmetic** above is independent.

---

## 7. Red-cut identity and disabled byte identity

### 7.1 Source isolation

- `buildDrawerCabinetModel(normalized)` — **no** `shelfGuides` input; panels identical.  
- Guides attach only as `scorePaths` after model build.  
- No `mirrorDesignPanel` on cabinet.  
- Layout / filename / MIME unchanged.  
- Finished View does not consume score paths.

### 7.2 Disabled goldens

| Case | Size / hash | Baseline pin? | Working tree |
| --- | --- | --- | --- |
| One row | **4456 / a6dd23dc** | **Yes** (`a36d7ea` fixture) | Still asserted; fixtures also cover absent + explicit false |
| Two rows | **6802 / 36a41b07** | **Not in baseline fixtures** as a literal pin | New disabled golden + structural path equality tests |
| Three rows | **9153 / 8c286797** | **Yes** (`a36d7ea`) | Still asserted |

**Analysis:** With guides off, `scorePaths` is empty → serializer emits **no** score group → SVG equals baseline cut-only output for the same model. One- and three-row **pre-existing** pins confirm disabled identity against committed baseline. Two-row pin is **new** but consistent with empty-scorePaths path identity; structural fixtures compare red `id:path` joins enabled vs disabled.

Malformed normalize → `shelfGuides: false` → same disabled path.

### 7.3 Enabled default two-row

Claimed golden **7534 / dd3ff0bd** is supported by:

- structural fixtures (score-before-cut, subgroup name, 3 entries / 6 segments, owners, assembly-label coexistence, red path equality);  
- plus size/hash pin.

Not hash-only circularity.

---

## 8. Shared serializer compatibility

**Baseline:** always `designSerializedPathGroup(scorePaths, 'rail-guides')`.  
**Working tree:** `designSerializedPathGroup(scorePaths, result.scoreGroupId || 'rail-guides')`.

| Check | Verdict |
| --- | --- |
| `scoreGroupId` optional | **Yes** |
| Default remains `rail-guides` | **Yes** |
| Sliding Lid needs no change | **Yes** — still omits `scoreGroupId` |
| Other `scorePaths` callers | Only Sliding Lid + Drawer Cabinet (new) |
| Assembly-label order | Unchanged relative order (labels after paths / same serializer structure) |
| Score before cut | Unchanged (`if (scorePaths.length) …` then cut) |
| Empty score | No empty subgroup (guard on length) |
| User input as group id | **No** — fixed `'shelf-guides'` string in result builder only |
| Malformed group id from UI | **N/A** — not user-editable |

**No shared serializer regression** for existing templates.

---

## 9. Sliding-Lid fixture-correction analysis

| Question | Answer |
| --- | --- |
| Exact change | Before attaching temporary `designResults` node, rename live `#designResults` to a temp id; restore after `runSelfTests` |
| Production event binding changed? | **No** |
| Production HTML ids changed? | **No** |
| Scope | **Strictly fixture/test code** |
| Narrows to fixture DOM? | **Yes** — `getElementById('designResults')` hits temporary root during test |
| Still drives real preview/download code? | **Yes** — same `runSelfTests` / click handlers |
| Masks product duplicate-id bug? | **No.** Normal runtime has a **single** `#designResults` in the page. Collision was **self-test harness vs live render root**, not dual roots in production UI. |
| Less representative? | **No** — removes cross-contamination; still exercises real code paths |

**Classification:** Appropriate **test-harness isolation**, not a product defect cover-up. Residual note: if a future feature intentionally mounts two `designResults` nodes in production, that would be a separate product bug — not introduced by this isolation.

---

## 10. UI / wording

Help and metrics accurately state:

- marks on **Left, Right, and Back** interior faces;  
- **upper-shelf** elevations;  
- **blue score before red cut**;  
- shelves still need **edge glue**;  
- **positioning aids only**;  
- first cabinet confirms face + top/bottom.

**Prohibited implications searched:** no dado / groove / captured / locking / structural support / strengthens / guaranteed alignment / proven squareness / glue strength in user-facing shelf-guide copy.

Help text is long but clear; not classified as a defect.

---

## 11. Finished View

- `buildDrawerCabinetFinishedViewModel` does **not** take score paths.  
- No guide geometry in Finished View.  
- No production geometry derived from Finished View coordinates.  
- No optional Finished View note in this diff (boolean-only note not required for safety).

---

## 12. Storage and protected boundaries

**Diff does not touch:** `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState`, `loadState`, `persist`, `backupObject`, `replaceData`, `mergeData`, import/export, production evidence/settings normalize/applicability, promotion/evidence functions, Inventory, Library, Projects, Pricing, Test Grid, Material Test, QR Stand, Hanging Sign, Dice Tray, Divider Tray, Finger Box, Joint Fit Coupon, other Finished Views.

**`drawerShelfGuides`:** session draft normalize / UI / result / fixtures only.

---

## 13. Fixture count reconciliation

| Item | Value |
| --- | --- |
| Baseline Designs geometry total (pre-feature) | **887** |
| New static `add(` in shelf-guides suite | **19** (hand-counted in new block; no new multi-assert loops in that block beyond single `add` each) |
| Current Designs total | **906** = 887 + 19 |
| Baseline complete suite | **1679** |
| Current complete suite | **1698** = 1679 + 19 |
| Unrelated suite totals | Unchanged except Designs / complete |

**Independent Edge `?selftest=all` (this audit):**

| Group | Result |
| --- | --- |
| Designs geometry | **906 / 0** |
| Production settings | **66 / 0** |
| Evidence promotion | **58 / 0** |
| Designs production | **118 / 0** |
| Test Grid machine identity | **18 / 0** |
| Storage recovery | **8 / 0** |
| Material browser (partial lines in log) | 70 / 0, 8 / 0, … (not double-counted into complete) |
| **Complete suite (final)** | **1698 / 0** |

### Assertion quality (summary)

| Class | Examples |
| --- | --- |
| **Independent** | normalize true/false/on; hand-derived elevation 39.3; literal Left ticks; red path equality; score-before-cut; subgroup name; one/two/three-row counts; containment; mirror set check; storage isolation; Sliding-Lid still `rail-guides`; preview/download identity; filename/MIME |
| **Partially circular** | Enabled size/hash golden (supported by structural asserts) |
| **Circular** | None that solely re-call helper as sole proof of geometry without literals |
| **Weak but acceptable** | Two-row **disabled** golden is new (not baseline pin); path-empty analysis + 1/3-row baseline pins compensate |

Coverage list from audit brief is **substantially met**.

---

## 14. Runtime / browser checks performed

| Check | Result |
| --- | --- |
| `git diff --check` | Pass |
| HTML parse / structure | Pass (via Edge load + suite) |
| Isolated headless **Microsoft Edge** `file://…/index.html?selftest=all` | **Pass** — complete **1698 / 0** |
| Designs 906 / 0 | **Reproduced** |
| Production settings / Evidence / Designs production / Grid MI / Storage | **Reproduced** as above |
| Form exercise (checkbox + preview) | Not separately scripted beyond suite fixtures that render/download; suite covers preview identity and form field presence |

---

## 15. Documentation accuracy

| Claim | Assessment |
| --- | --- |
| Terminology / panels / counts / score-before-cut | **Accurate** |
| Placement-aid + physical top/bottom disclosure | **Accurate** |
| No structural-support claim | **Accurate** |
| Fixture totals 906 / 1698 | **Accurate** (runtime-confirmed) |
| Goldens (1/2/3-row disabled; 2-row enabled) | Match working-tree fixtures; 1- and 3-row disabled match baseline pins |
| Changed-file scope index + README | **Accurate** |
| Implementation report Sliding-Lid isolation | **Accurate** |

---

## 16. Findings table

| ID | Severity | Location | Consequence | Recommendation | Fixture coverage |
| --- | --- | --- | --- | --- | --- |
| F1 | **Informational** | Two-row disabled golden `6802/36a41b07` | New pin not present at baseline; residual reliance on empty-scorePaths identity | Optional: accept as-is (structural red-path tests + 1/3-row baseline pins suffice) | Partial — enabled/disabled red equality + 1/3-row baseline goldens |
| F2 | **Informational** | Metric `Enabled - 0 marks` (1-row) | Slightly odd phrasing; not false | Optional: “Enabled (no upper shelf)” — not required | Warning + zero geometry fixtures |
| F3 | **Informational** | Physical top/bottom | Software cannot prove shop orientation | Keep first-cabinet confirm (already present) | Wording fixtures / help text |
| — | **Blocker** | — | — | **None found** | — |
| — | **Major** | — | — | **None found** | — |
| — | **Minor** | — | — | **None that block commit** | — |

**Audited boundaries (explicit pass):**

1. Disabled Drawer Cabinet SVGs remain byte-identical for pinned cases; empty scorePaths preserves baseline serialize for all disabled inputs.  
2. Enabled guides do not alter red cut paths or panel ordering.  
3. Elevations from `model.metrics.shelfBottomElevations` only.  
4. Top-down panel-local conversion correct under software convention.  
5. Two-tick pattern mirror-invariant (literal set proof).  
6. Shared serializer default preserves Sliding Lid and other callers.  
7. Sliding-Lid fixture correction is harness-only; does not mask production dual-root bug.  
8. Storage / schema / promotion / unrelated templates untouched.  
9. Fixture totals and runtime reconcile **exactly**.  
10. User-facing wording does not imply structural support.

---

## 17. Explicit unverified areas

- **Physical** cabinet orientation, marked face, glue strength, and squareness after assembly.  
- **LightBurn** import of blue score subgroup (not re-run in this audit).  
- **Manual** form click-through in a headed browser (suite covers equivalent paths).  
- **Every** historical Drawer Cabinet SVG golden beyond 1/2/3-row pins (not required if model + serialize path identity holds).

---

## 18. Conclusion

Implementation matches the design contract: session-only optional blue **shelf-guides** on Left/Right/Back from existing shelf elevations; red cut geometry isolated; serializer change additive and backward-compatible; Sliding-Lid fix is pure test isolation; fixtures and complete suite **1698 / 0** independently reproduced on Edge `file://`.

### SAFE TO COMMIT
