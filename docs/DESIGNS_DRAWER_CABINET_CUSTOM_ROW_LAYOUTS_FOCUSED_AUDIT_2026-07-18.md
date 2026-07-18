# Drawer Cabinet Custom-by-Row Layouts — Focused Implementation Audit (2026-07-18)

**Mode:** Read-only adversarial audit (no application edits, no git writes).  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Feature:** Drawer Cabinet custom-by-row layouts (Uniform retained; Custom by row)  
**Authoritative design review:** `docs/DESIGNS_DRAWER_CABINET_CUSTOM_ROW_LAYOUTS_DESIGN_REVIEW_2026-07-17.md`  
**Implementation report (not trusted alone):** `docs/DESIGNS_DRAWER_CABINET_CUSTOM_ROW_LAYOUTS_IMPLEMENTATION_2026-07-18.md`

---

## 1. Repository state

| Check | Result |
| --- | --- |
| `git log -1 --oneline` | **`eca1500` Add multi-grid dual-thickness drawer cabinets** |
| Matches expected baseline `eca1500`? | **Yes** (`eca150067273b12f86585a878430c6a2db00bb07`) |
| `main` vs `origin/main` | **Synchronized** (same SHA) |
| `git status -sb` | `M README.md`, `M index.html` |
| Staged | **Empty** |
| `git diff --check` | **Clean** (CRLF warnings only) |
| `git diff --numstat` | `README.md` **4 / 1**; `index.html` **80 / 35** (stat: **84 / 36**) |
| Unexpected tracked modifications | **None** |
| Design review present | **Yes** |
| Implementation report present | **Yes** (untracked) |
| Unrelated untracked | LightBurn, `debug.log`, historical docs, utilities — **untouched** |
| Read-only status | **Confirmed** for product sources; this audit only **creates this report** |

---

## 2. Complete diff-hunk enumeration

### 2.1 Totals

| File | Insertions | Deletions |
| --- | --- | --- |
| `README.md` | 4 | 1 |
| `index.html` | 80 | 35 |
| **Combined (stat)** | **84** | **36** |

### 2.2 `README.md` (3 logical regions)

| Hunk | Classification | Summary |
| --- | --- | --- |
| Designs feature bullet | README | Custom-by-row description; width semantics; no spans; production dual-file retained |
| Fixture totals | README | **1752** complete / **960** Designs / **264** Tray; arithmetic `1737 + 15` |
| Finished View note | README | Custom-row semantic Finished Front View |

### 2.3 `index.html` hunks (16)

| Hunk (approx.) | Classification |
| --- | --- |
| defaults line | defaults (`drawerLayoutMode`, row column fields) |
| cabinet form fields | UI fields; mode-dependent field rendering |
| material thickness help | UI fields (unchanged role wording path) |
| `normalizeDesignDraft` drawer-cabinet | normalization; top/bottom mapping |
| `drawerCabinetDimensions` | width/dimension model |
| `drawerCabinetRowLayout` + cell origin + prefix | row-layout helper; cell-origin delegation; drawer-prefix behavior |
| `buildDrawerCabinetModel` | row-specific drawer submodels; partition generation |
| `buildDrawerCabinetFrontElevation` | Finished View |
| `buildDrawerCabinetDesignResult` | linked layout; separate layouts |
| metrics block | metrics |
| warnings (in model) | warnings |
| fixture block (+15 asserts) | fixtures |
| form change re-render list | event handling |

**No unexplained changes** to shared serializers, shared `layoutDesignPanelRows` body, storage, suite dispatcher registration, or unrelated templates.

---

## 3. Files and functions inspected

- Working tree `index.html` / `README.md`
- `git show eca1500:index.html` and baseline Edge suite on a temp checkout of that file
- Authoritative custom-row design review + implementation report
- Prior multi-grid dual-thickness contracts (committed at `eca1500`)
- `designDefaults`, cabinet UI, `normalizeDesignDraft` (drawer-cabinet)
- `drawerCabinetDimensions`, `drawerCabinetRowLayout`, `drawerCabinetCellOrigin`, `drawerCabinetDrawerPrefix`
- `buildDrawerCabinetModel`, shelf guides, Finished View builders
- `buildDrawerCabinetDesignResult` (linked + separate)
- `designResultsHtml` metrics, download handlers, fixtures, form event list

---

## 4. Baseline runtime reproduction (`eca1500`)

Isolated headless Microsoft Edge, direct `file://` on `git show eca1500:index.html`, `?selftest=all`, same aggregation as working tree (Material browser final pass only once).

| Suite | Expected | Observed |
| --- | --- | --- |
| Complete | **1737 / 0** | **1737 / 0** |
| Designs geometry | **945 / 0** | **945 / 0** |
| All 15 group slots | Connected | **Yes** (same group titles as working tree) |

Registered group passes (baseline): 20+12+66+58+118+23+18+67+57 (Material final)+56+61+12+8+216+945 = **1737**.

---

## 5. Working-tree runtime reproduction

Same Edge path on live working-tree `index.html`.

| Suite | Expected | Observed |
| --- | --- | --- |
| Complete | **1752 / 0** | **1752 / 0** |
| Designs geometry | **960 / 0** | **960 / 0** |
| Arithmetic | `1737 + 15 = 1752` | **Matches** |
| `readyState` | complete | **complete** |
| Runtime exceptions | none | **None observed** |
| Group coverage | all baseline groups retained | **Yes** — only Designs grew by **15** |

`python -m html.parser index.html` succeeds.

---

## 6. Normalization and mode contract

| Requirement | Result |
| --- | --- |
| Absent / blank mode → uniform | **Pass** |
| `'uniform'` / `'custom-row'` | **Pass** |
| Unknown nonblank → clear error | **Pass** (`Choose a recognized drawer layout.`) |
| Uniform validates `drawerColumns` | **Pass** |
| Uniform ignores malformed custom fields | **Pass** |
| Custom ignores dormant `drawerColumns` | **Pass** (probe: `drawerColumns:'99'` → max from rows only) |
| Custom validates only active row fields | **Pass** (inactive middle/top ignored) |
| Active counts integers 1–3 | **Pass** |
| No linked-until-edited / event history | **Pass** — pure normalize |

Raw fields used only in UI + normalization: `drawerLayoutMode`, `drawerBottomColumns`, `drawerMiddleColumns`, `drawerTopColumns`, plus existing row/column/thickness fields. **No downstream reads** of `drawerTopColumns` / `drawerMiddleColumns` / `drawerBottomColumns` outside normalize/UI/fixtures.

---

## 7. Top-to-bottom → bottom-up mapping

| User top→bottom | Normalized `rowDrawerCounts` | Runtime |
| --- | --- | --- |
| Two-row `1 / 3` (top 1, bottom 3) | **`[3, 1]`** | **Confirmed** |
| Three-row `1 / 2 / 3` | **`[3, 2, 1]`** | **Confirmed** |
| One-row | `[bottom]` via “Drawers in row” | **Confirmed** (UI label) |

Mapping occurs **only** in `normalizeDesignDraft`. Semantic `r01` remains bottom.

---

## 8. Width model and independent derivation

Defaults: inside width **100**, interior thickness **3**, lateral clearance **0.45**, `maximumColumns = 3`.

| Quantity | Hand-derived | Runtime |
| --- | --- | --- |
| baseDrawerOutsideWidth | **106** | **106** |
| baseCellWidth | **106.45** | **106.45** |
| cabinetInsideWidth | **325.35** | **325.35** |
| 3-drawer inside | **100** | **100** |
| 2-drawer inside | **154.725** | **154.725** |
| 1-drawer inside | **318.9** | **318.9** |

Formulas match the design review. `drawerCabinetDimensions` owns shared cabinet width from **maximum** row count. Per-row widths owned by `drawerCabinetRowLayout` from unrounded division of rounded `cabinetInsideWidth`, with `designRound` on outputs.

---

## 9. Row closure

Closure check: `|n × cellWidth + (n−1) × iT − W| ≤ 1e-5` (model errors if exceeded).

| Case | Closure |
| --- | --- |
| 1-, 2-, 3-drawer rows at W=325.35 | **Exact (0 error)** |
| Custom `1 / 3`, `2 / 1`, `1 / 2 / 3` | **Pass** (runtime + fixtures) |
| Separate-thickness custom `1 / 2 / 3` | **Pass** (valid dual outputs) |

Origins: `designRound(columnIndex × (rawCellWidth + iT))` — independent from unrounded pitch, not cumulative rounded steps.

---

## 10. Row-layout helper and cell origin

`drawerCabinetRowLayout(rowIndex, dimensions)` owns count, cell/outside/inside widths, drawer origins, partition origins, and `closure`.

`drawerCabinetCellOrigin` delegates `x` to row layout and keeps bottom-up `y` pitch.

Finished View and model both call these helpers — **no duplicate width formulas** in the renderer.

Search: custom width arithmetic is not re-derived in metrics (consumes `rowLayouts`) or layout (panel dimensions only).

---

## 11. Drawer submodel cache

- One `buildBoxModel` per **distinct** active count (Map keyed by count) → **at most three**
- Uniform degenerates to one count / one submodel
- Depth/height remain global; width is row-derived inside width
- Namespaced panels take geometry from the matching count’s submodel
- Error/warning prefixes distinguish custom rows vs generic “Drawer”

---

## 12. Uniform byte identity

Working-tree probe vs required pins:

| Pin | Required | Observed |
| --- | --- | --- |
| 1-row | 4456 / a6dd23dc | **Match** |
| 2-row | 6802 / 36a41b07 | **Match** |
| 3-row | 9153 / 8c286797 | **Match** |
| Linked uniform 2×3 | 16429 / 0814ca2e | **Match** |
| Separate shell | 2779 / 956ad870 | **Match** |
| Separate interior | 8609 / bad8b97f | **Match** |
| Absent/blank mode = uniform | — | **SVG-identical to default** |
| One-column separate | drawers present, legacy IDs | **Valid; 5 interior drawers** |

**No Uniform production drift.**

---

## 13. Custom equivalence

| Case | Result |
| --- | --- |
| Custom `3 / 3 / 3` SVG == Uniform 3×3 | **Byte-identical** |
| Custom all-one rows SVG == Uniform 3-row one-column | **Byte-identical** |
| All-one uses bare `drawer-rNN-*` | **Pass** |

---

## 14. ID and prefix contract

`drawerCabinetDrawerPrefix` is the sole prefix source.

| Policy | Result |
| --- | --- |
| `maximumColumns === 1` → bare `drawer-rNN-*` | **Pass** |
| `maximumColumns > 1` → always `cNN`, including one-drawer rows | **Pass** |
| Top-to-bottom `1 / 3`: bottom `r01-c01..c03`, top `r02-c01` | **Pass** |
| No `drawer-r02-bottom` in mixed cabinet | **Pass** |
| No duplicate panel IDs | **Pass** |

---

## 15. Counts (metrics + physical panels)

| Top→bottom | Drawers | Shelves | Partitions | Physical check |
| --- | --- | --- | --- | --- |
| `1 / 3` | 4 | 1 | 2 | 4×5 drawer panels; 2 partitions |
| `2 / 1` | 3 | 1 | 1 | Confirmed |
| `1 / 2 / 3` | 6 | 2 | 3 | 6×5 drawers; 3 partitions |

Partition count `sum(n−1)` matches.

---

## 16. Partitions

| Check | Result |
| --- | --- |
| Face `cellHeight × cabinetInsideDepth` | **33.3 × 76.5** (defaults) |
| Interior thickness role / metadata | **Pass** |
| Plain closed rect; no joinery/scores | **Pass** |
| Row-specific horizontal origins | Bottom: **106.45, 215.9**; middle: **161.175**; top: none — **matches independent derivation** |
| Origins from row helper, not global max pitch | **Pass** |

---

## 17. Shell, shelves, production architecture

- Shelves: full `cabinetInsideWidth × cabinetInsideDepth`, interior thickness, count `rows−1`
- Shell: five finger-jointed panels at shell thickness; width from common cabinet outside
- Linked: one SVG; Separate: exactly shell + interior; fallback shell-only; generic download suppressed
- Custom topology does not change `productionOutputs` shape
- No unequal-thickness finger joints

---

## 18. Panel accounting (separate custom `1 / 2 / 3`, S=6 / I=3)

| Check | Result |
| --- | --- |
| Model panels | **40** |
| Union of production `panelIds` | **40**, no missing/extra |
| Shell leak / interior shell leak | **None** |
| Each output SVG validates | **Pass** (fixtures + probe) |

---

## 19. Layout and serializer

- Linked custom layout valid, finite, non-overlapping (suite)
- Separate layouts independent
- `layoutDesignPanelRows` not broadly rewritten
- No silent discard of requested IDs when prefixes match model (prior columns=1 separate bug remains fixed at baseline and in WT)

---

## 20. Preview / download

- Separate UI cards still bind `data-download-production-output` and encode each `output.svg`
- Generic download remains disabled when multi-output required
- Storage/backup isolation fixtures for custom generation: **byte-identical** around exercised paths
- Full live download harness for custom is thinner than for shelf-guides legacy path (**Minor fixture gap**, not a production defect observed)

---

## 21. Finished View

| Check | Result |
| --- | --- |
| Front counts per row (1/2/3 → 3+2+1) | **Pass** |
| Equal widths within row; wider for fewer drawers | **Pass** (front widths 106 / 160.725 / 324.9) |
| Partition bands row-specific | **Pass** |
| Screen-only; no production SVG consumption | **Pass** |
| Uses helper semantics, not re-derived widths | **Pass** |

---

## 22. Metrics and warnings

**Metrics (custom):** layout mode, top-to-bottom configuration string, maximum drawers in a row, per-row inside widths, cabinet dimensions, thickness roles, production file count, sheet bounds, **Required pieces from `metrics.pieceCount`** (full model).

Example string: `Top 1 drawer / Middle 2 drawers / Bottom 3 drawers` — clear and fixture-locked.

**Warnings (custom):** equal-within-row, rows may differ, manual partitions, no marks, dry-fit, rack/flex, fragile small drawers. Separate-stock/laser settings warning retained. No strength/fit guarantees.

Detailed shop laser checklist (ventilation, fire watch, etc.) is not newly expanded in custom warnings; existing product tone is preserved (**Informational**, not a regression).

---

## 23. Storage and Production Settings

| Boundary | Result |
| --- | --- |
| STORAGE_KEY / SCHEMA / load/persist/backup/import | **Not in diff** |
| Other templates | **Untouched** |
| Production Settings still maps `materialThickness` only | **Unchanged** |
| Linked: thickness updates both roles | **Pass** (prior contract) |
| Separate: shell only; interior explicit | **Pass** |
| Custom layout does not alter applicability schema | **Pass** |

---

## 24. Fixture quality (15 new Designs asserts)

| Assertion theme | Independence |
| --- | --- |
| Mode normalize missing/blank/uniform + dormant custom | **Independent** |
| Active-only validation + top→bottom map + unrecognized | **Independent** |
| Count bounds `1/3` and `2/1` | **Independent** (metrics + structural) |
| Width literals 325.35 / 100 / 154.725 / 318.9 | **Independent** |
| Closure + monotonic wider rows | **Independent** |
| Submodels / patterns keys + thickness | **Partially circular** (name overstates width check) |
| Prefix policy mixed vs all-one | **Independent** |
| Partition face dims + plain | **Independent** |
| Uniform↔Custom byte equivalence | **Independent** |
| Separate membership once | **Independent** |
| Finished View counts/origins | **Independent** |
| Metrics HTML strings | **Independent** |
| Storage isolation + dual download markup | **Independent** |
| Golden linked `1/3` 13606/500396df | **Partially circular** (after structural asserts) |
| Golden separate interior 14777/69b0ce8b | **Partially circular** (after structural asserts) |

No circular-only suite: goldens follow structural coverage. No group removed or double-counted.

---

## 25. New golden verification

After structural checks (mapping, widths, IDs, partitions, accounting):

| Golden | Claimed | Observed |
| --- | --- | --- |
| Linked Custom top→bottom `1 / 3` | 13606 / 500396df | **Match** |
| Separate Custom `1 / 2 / 3` interior | 14777 / 69b0ce8b | **Match** |

No unnecessary custom shell pin (shell is geometry-unique at max-3 / dual thickness; not claimed).

---

## 26. README / report accuracy

Claims for Uniform/Custom modes, independent per-row counts, width semantics, equal heights, no mixed widths within a row, no spans, plain partitions, linked/separate outputs, Finished View scope, fixture totals, new goldens, Edge runtime, and no physical validation are **supported** by source and runtime.

---

## 27. Findings table

| ID | Severity | Location | Consequence | Recommendation | Fixture coverage |
| --- | --- | --- | --- | --- | --- |
| F1 | **Minor** | Fixture name: “custom drawer submodels… row-specific dimensions” | Boolean mainly checks thickness and pattern keys, not literal inside widths | Optionally assert `panel.width` / model dimensions against 100 / 154.725 / 318.9 | Partial |
| F2 | **Minor** | Custom separate live download path | Preview/download identity relies on shared card machinery + object equality more than a dedicated custom live-download fixture | Optional: mirror shelf-guide live download fixture for one custom separate case | Narrow gap |
| F3 | **Informational** | Physical fit | Wide drawers, partition glue-up, dual stock uncut | Shop prototype | N/A |

**No Blocker. No Major.**

---

## 28. Unverified physical areas

- Real material fit, kerf, and drawer travel  
- Wide-drawer racking under load  
- Manual partition placement accuracy  
- Dual-thickness focus/power between stocks  
- Squareness after glue-up  

No LightBurn import or machine run was performed.

---

## 29. Final conclusion

### SAFE TO COMMIT

The custom-by-row implementation correctly maps top-to-bottom UI into bottom-up semantics, derives maximum-row cabinet width and proportional row drawer widths with exact closure, keeps Uniform production pins byte-identical, preserves dual-thickness production architecture with complete panel accounting, and grows Designs fixtures by 15 with complete-suite **1752 / 0**. Minor fixture polish items do not block commit.
