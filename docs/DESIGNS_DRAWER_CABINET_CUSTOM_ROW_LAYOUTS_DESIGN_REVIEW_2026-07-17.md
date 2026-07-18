# Drawer Cabinet Custom-by-Row Layouts — Deep Design and Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-17
**Review type:** read-only design and architecture review. No file was edited, staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## 0. Repository and verification state

- `git status -sb`: `## main...origin/main` with no ahead/behind marker — **synchronized with origin/main**; tracked working tree **clean**; staging area **empty** (`git diff --cached --name-only` returned nothing); `git diff --check` / `--stat` / full diff all empty.
- `git log -1 --oneline`: **`eca1500 Add multi-grid dual-thickness drawer cabinets`** — matches the expected baseline exactly. The commit contains `index.html`, `README.md`, and the six multi-grid/dual-thickness reports (design review, challenge, implementation, blocker correction, focused audit, correction audit), confirmed via `git show --stat eca1500`.
- Untracked files: the same long-standing set (67 entries: `LightBurn Projects/*`, `debug.log`, `parametric_qr_stand_generator.py`, prior `docs/*.md` reports including this session's Finger Box corner-blocks review). All untouched.
- Because the working tree is byte-identical to `eca1500` (empty diff), every function read below **is** the committed baseline — `git show eca1500:index.html` and the working tree are the same bytes.

### Verified baseline totals

Confirmed from the committed `README.md` at `eca1500` (which now correctly documents the reconciled arithmetic: *"1714 − 922 + 945 = 1737; the prior 1679 manual sum omitted the 58-assertion Evidence Promotion group"*), **and** independently confirmed by live headless-Edge CDP execution in this session's immediately preceding correction audit, which ran every fixture group against byte-identical content (the working tree from which `eca1500` was committed, unchanged since):

- Tray-model: **264 / 0**
- Designs geometry: **945 / 0**
- Complete suite (all 15 dispatched groups): **1737 / 0**

The superseded figures (922, 934, 1714, 1726, 1679) are not used anywhere in this review as current totals.

### Golden pins confirmed in committed source

All six pins verified present at `eca1500` by direct grep of the committed `index.html`: legacy one-column **4456/`a6dd23dc`** (1-row), **6802/`36a41b07`** (2-row), **9153/`8c286797`** (3-row); enabled **16429/`0814ca2e`** (linked 2×3), **2779/`956ad870`** (separate shell), **8609/`bad8b97f`** (separate interior).

## 1. Files and functions inspected

- `README.md` at `eca1500` (Designs paragraph, Built-in-checks paragraph, Finished View paragraph).
- All six committed multi-grid/dual-thickness reports, plus the original Drawer Cabinet architecture review (2026-07-16), Phase 5.1 implementation report, shelf-guide design review, and Finished Front View reports — all read in this session or in this session's immediately preceding turns.
- `index.html` (byte-identical to `eca1500`): `designDefaults()` cabinet keys; `renderDesigns()` `cabinetFields` block; `normalizeDesignDraft()` drawer-cabinet branch (`shellThickness`/`columns`/`useSeparateInteriorThickness`/`interiorThickness` normalization); `drawerCabinetDimensions()`; `drawerCabinetCellOrigin()`; `drawerCabinetDrawerPrefix()`; `buildDrawerCabinetModel()` (shell panels, shelves, partitions, per-cell drawer namespacing, insertion check, metrics); `drawerCabinetShelfGuideSegments()`; `buildDrawerCabinetFrontElevation()` / `buildDrawerCabinetFrontElevationSvg()`; `buildDrawerCabinetDesignResult()` including the linked layout-row builder and the separate-mode `output()` closure with role-based shell/interior membership; `designResultsHtml()` cabinet metrics and production-output cards; `refreshDesignPreview()` / `bindDesignPreviewActions()` / `downloadDrawerCabinetProductionOutput()`; the Drawer Cabinet fixture block (all multi-grid assertions); the `selftest` dispatcher; `designRound()` (`Math.round((v+ε)·1e6)/1e6` — six-decimal precision, load-bearing for §6's rounding analysis).

No file was edited during this review.

## 2. Current contract to preserve (from actual source)

The committed cabinet: uniform `rows × columns` grid (each 1–3); one shared `drawerInsideWidth/Depth/Height` for every drawer; `shellThickness` (= `materialThickness`) on the five finger-jointed shell panels; `interiorThickness` (linked to shell by default; separate only on explicit checkbox) on drawers, shelves, and partitions; **no unequal-thickness finger joint anywhere** (shelves/partitions are unjointed plain rectangles; shell joints shell, drawers joint themselves); partitions are plain `cellHeight × cabinetInsideDepth` rectangles with interior-role metadata and **no** tabs/slots/guides; one linked production SVG, exactly two role-partitioned SVGs in separate mode with fallback suppression; `drawer-rNN-*` IDs at `columns===1`, `drawer-rNN-cMM-*` otherwise; `cabinet-partition-rNN-cMM` where `cMM` is the within-row boundary index; Finished Front View consuming only `dimensions`/`metrics`; session-only `designDraft` state. All of this is the preservation surface for the new feature.

## 3. Scope decision — is "custom by row" the right first step?

**Yes, with the boundaries as stated, and this is not a rubber stamp.** The challenge test I applied: does any smaller feature deliver the stated use cases (1/3, 2/1, 1/2/3), and does any part of this feature secretly require the excluded machinery (spans, mixed widths, grid editor)?

- Smaller alternatives fail: a "wide top drawer" preset would be a special case of exactly the same geometry with less generality and the same validation surface. There is no cheaper path to 1/3 than per-row counts.
- The feature does **not** secretly require spans: a row with fewer drawers is not a spanning drawer — it is a row whose equal division has fewer cells. No panel crosses a shelf; the shelf model is untouched. No spanning metadata exists or is needed.
- It does not require mixed widths within a row (equal division per row is the entire width model, §6), does not require a layout engine (one 3-element array of counts is the whole configuration space — 27 custom combinations, 10 distinct up to row order), and does not require GUI redesign (two-to-three conditional selects, §11).
- Per-row counts is also the natural stopping point: every excluded feature (spans, cubbies, mixed widths) would require breaking the "one row = one equal division" invariant that makes the closure proof in §6 one line. That invariant is the boundary, and it is worth defending.

## 4. Terminology and field contract (settled)

| Concept | Decision |
|---|---|
| User-facing mode name | **"Drawer layout"** with options **"Uniform grid"** / **"Custom by row"** |
| Raw mode field | **`drawerLayoutMode`** — absent/blank → uniform; `'uniform'` → uniform; `'custom-row'` → custom; any other nonblank value → validation error *"Choose a recognized drawer layout."* (matches the established `couponType` reject-unknown-nonblank pattern) |
| Raw per-row fields | **`drawerBottomColumns`**, **`drawerMiddleColumns`**, **`drawerTopColumns`** — semantic names, not an array, because the raw draft is written by named form inputs and named fields make the stale-value rules (§10) trivially per-field |
| Normalized fields | **`layoutMode`** (`'uniform'`\|`'custom-row'`), **`rowDrawerCounts`** (bottom-up array, one entry per active row), **`maximumColumns`** (`Math.max(...rowDrawerCounts)`); `columns` retained as the uniform-mode input; `drawerCount`/`partitionCount` remain derived in `drawerCabinetDimensions()` |

**The single most important normalization decision:** in **both** modes, normalization emits `rowDrawerCounts` — uniform mode emits `Array(rows).fill(columns)`. Every downstream consumer (dimensions, model, layout, Finished View, metrics, fixtures) reads **only** `rowDrawerCounts` and `maximumColumns`. Uniform mode is then not a separate code path but the degenerate case of one representation, which is what makes the byte-identity proof (§7) and the 3/3/3 ≡ 3×3 equivalence structural rather than special-cased.

### Top-to-bottom vs bottom-to-top mapping (explicit)

- **Internal geometry is bottom-up and stays bottom-up**: row 1 is the bottom row, exactly as today (`drawer-r01` rests on the cabinet bottom; `shelfBottomElevations` measure up from the inside floor; the Finished Front View already converts to top-down screen space at render time). Changing this would break every existing ID and golden for no benefit.
- **UI is top-to-bottom** (§11), matching how a person looks at a cabinet.
- **The mapping lives in exactly one place: normalization.** `rowDrawerCounts[0]` = bottom = `drawerBottomColumns`; `rowDrawerCounts[rows−1]` = top = `drawerTopColumns`; middle exists only at `rows===3`. For `rows===2`: bottom→index 0, top→index 1, middle ignored. For `rows===1`: the single row is fed from `drawerBottomColumns` and the UI labels it plainly (§11).
- **Documentation rule:** every fixture and report states row configurations top-to-bottom with an explicit "top-to-bottom" qualifier (matching the task's own notation), while IDs remain `r01`=bottom. One fixture (§14 item 9) pins the mapping itself so it can never silently flip.

## 5. Width model — decision

**OPTION A is selected: the row with the most drawers establishes the cabinet width, and `drawerInsideWidth` keeps exactly its current meaning for that row.**

- **A vs B:** B (cabinet width as primary input) breaks the existing input contract for every current user, cannot preserve legacy bytes without a parallel legacy path, and answers the wrong question — Joe sizes drawers around contents ("what must fit in each small drawer"), which is the original architecture review's own justification for drawer-inside-first dimensioning. Rejected.
- **A vs C:** C (independent width per row) multiplies UI, validation, and metrics surface for a use case ("specific width for the wide drawer") nobody has stated; and it destroys the property that a custom layout is fully described by one small count array. Rejected as out of first-phase scope.
- **D:** unwarranted — A's semantics are clear and stable, proven below.

The one honest cost of A: `drawerInsideWidth`'s label needs a custom-mode help-text clarification (§11), and rows with fewer drawers get **derived** widths the user does not type. §12's metrics make every derived width visible, which is the correct mitigation.

## 6. Exact formulas and closure proof

With `iT = interiorThickness`, `L = lateralClearance`, `W = cabinetInsideWidth`, and per-row count `n`:

```text
baseDrawerOutsideWidth = drawerInsideWidth + 2·iT                    (unchanged)
baseCellWidth          = baseDrawerOutsideWidth + L                  (unchanged)
maximumColumns         = max(rowDrawerCounts)
W                      = maximumColumns·baseCellWidth + (maximumColumns − 1)·iT
                                                                     (current formula with columns → maximumColumns)
rowCellWidth(n)          = (W − (n − 1)·iT) / n
rowDrawerOutsideWidth(n) = rowCellWidth(n) − L
rowDrawerInsideWidth(n)  = rowDrawerOutsideWidth(n) − 2·iT
```

**Closure proof (every row consumes exactly W):**

```text
n·rowCellWidth(n) + (n−1)·iT
  = n·[(W − (n−1)·iT)/n] + (n−1)·iT
  = W − (n−1)·iT + (n−1)·iT
  = W                                                                ∎
```

**Uniform degeneracy proof (max-count rows recover the entered width exactly):** for `n = maximumColumns = c`,
`rowCellWidth = (c·baseCellWidth + (c−1)·iT − (c−1)·iT)/c = baseCellWidth`, hence `rowDrawerInsideWidth = drawerInsideWidth` exactly. Custom rows at the maximum count are byte-for-byte the current uniform geometry; uniform mode itself (all rows at `columns`) is therefore the degenerate case of the same formulas — no parallel formula system exists.

**Lateral-clearance accounting:** applied exactly once per cell (`rowDrawerOutsideWidth = rowCellWidth − L`), the precise generalization of the current `cellWidth = drawerOutsideWidth + L`. The existing insertion self-check generalizes per row: `rowCellWidth − L === rowDrawerOutsideWidth` (trivially true by construction, but kept as the model's own consistency assertion, as today).

**Worked example (current defaults: 100 mm inside width, iT = 3, L = 0.45):** `W = 3·106.45 + 2·3 = 325.35`.

| Row count | rowCellWidth | outside | inside |
|---|---|---|---|
| 3 | 106.45 | 106 | **100** (input recovered exactly) |
| 2 | (325.35−3)/2 = 161.175 | 160.725 | 154.725 |
| 1 | 325.35 | 324.9 | 318.9 |

Closure check, 2-drawer row: `2·161.175 + 3 = 325.35` ✓ exact. No negative or undersized interior at defaults; the general guards are: error when `rowCellWidth − L ≤ 0` or `rowDrawerInsideWidth ≤ 0`, plus each row's own `buildBoxModel` finger-pattern validation (§8), which is strictly *easier* to satisfy for wider drawers — the narrowest drawer in the cabinet is always the already-validated base drawer.

**Rounding note (implementation caution, not a defect):** `designRound()` works at 1e-6 precision, so division residues like `161.175` are exactly representable and closure survives rounding at physical precision. The row-layout helper (§9) should still perform the division before rounding and round only its outputs, and the model's closure assertion should use the existing `1e-6`-style epsilon — with three cells the accumulated rounding bound is ≤ 1.5e-6, so the epsilon should be `1e-5` to avoid false failures. Cumulative horizontal drift is impossible by construction because drawer and partition origins are all computed from the single unrounded `rowCellWidth`, never by summing previously-rounded positions.

## 7. Legacy byte-identity contract

When `drawerLayoutMode` is absent, blank, or `'uniform'`, normalization emits `rowDrawerCounts = Array(rows).fill(columns)` and `maximumColumns = columns`; every formula in §6 then evaluates to precisely the committed formula (shown by substitution in §6), every ID evaluates to precisely the committed ID (§10), and every layout row to the committed layout row — so the production SVG bytes, panel IDs/titles/order, score-group order, shelf-guide bytes, filename, MIME, Finished View, linked/separate behavior, and all six committed goldens are preserved **structurally**, enforced by re-asserting the existing goldens (which requires no fixture change — they already exist and already run).

**Custom 3/3/3 vs uniform 3×3:** under the single-representation model these normalize to the same `rowDrawerCounts = [3,3,3]` with the same `maximumColumns`, and the only remaining input difference (`layoutMode` itself) must not leak into geometry, IDs, or serialization. Recommendation: **assert full byte identity** (`customResult.svg === uniformResult.svg`) as a fixture — it follows naturally from one semantic model rather than being a brittle special case, and it doubles as the proof that `layoutMode` is a pure normalization concern with zero downstream footprint. (The one permitted difference is `metrics.layoutMode` itself and the layout-mode metric line — metrics HTML is not part of the byte contract.)

## 8. Height, partition, and drawer models

**Height model — unchanged, verified sound.** All rows share `cellHeight`; `cabinetInsideHeight = rows·cellHeight + (rows−1)·iT`; shelves remain `rows−1` full-width (`cabinetInsideWidth × cabinetInsideDepth`) plain rectangles; `shelfBottomElevations` unchanged. Custom row counts affect width division, drawer count, and partitions only. Physically sound: shelves span the full cabinet regardless of what the rows above/below them contain, and every partition stands between two full-width horizontal surfaces (cabinet bottom/shelf/top), exactly as today. Mixed row heights are excluded.

**Partition model.** `partitionCount = Σ(rowDrawerCount − 1)` over active rows — degenerates to `(columns−1)·rows` in uniform mode ✓. Every partition keeps the committed contract: `cellHeight × cabinetInsideDepth` cut face (row-height, full interior depth — **unchanged by row count**, since only the *x position and count* vary per row, never the face dimensions), interior thickness/role metadata, plain rectangle, no joinery, no guides, glued manually. IDs stay `cabinet-partition-rNN-cMM` with `cMM` = within-row boundary index `1..n−1` — the committed pattern already row-scopes the boundary index, so it extends without change. Partition x positions: boundary `m` of a row with count `n` sits at `m·rowCellWidth + (m−1)·iT` from the row's interior-left origin — owned by the §9 helper, nowhere else.

**Drawer model construction.** Build **one `buildBoxModel()` submodel per distinct row count** (at most 3 distinct: counts 1, 2, 3), cached by count, since rows with equal counts have identical drawer dimensions (depth and height are global; width depends only on count). This is the minimal extension of the current single-submodel pattern: uniform mode has one distinct count → exactly one submodel → identical behavior and bytes. Per-row namespaced copies via the existing `namespaceDesignPanel()`. Validity: `model.valid` requires **every** distinct-count submodel valid; errors/warnings from each submodel bubble with a row-identifying prefix (e.g. `"Row 2 drawer: …"` where multiple distinct widths exist, retaining the current bare `"Drawer: …"` prefix in uniform mode to preserve current warning text), deduplicated through the existing `Set` pass. The narrowest drawer (the base width, in the max-count row) drives finger-pattern feasibility in practice since wider edges always admit at least the same pattern count; but each submodel is validated independently regardless — no inference. The existing large-box warning inside `buildBoxModel` covers wide-drawer sizes automatically; a cabinet-level wide-drawer racking warning is added in §12. Drawer metrics must not present a single width when widths differ (§12).

## 9. Single source of truth — the row-layout helper

One new helper owns all per-row horizontal geometry:

```text
drawerCabinetRowLayout(rowIndex, dimensions)
  → { rowIndex, drawerCount, cellWidth, drawerOutsideWidth, drawerInsideWidth,
      drawerOrigins: [x per column, interior-relative],
      partitionOrigins: [x per boundary, interior-relative] }
```

`dimensions` (from `drawerCabinetDimensions()`) carries `rowDrawerCounts`/`maximumColumns`, so the helper needs no extra arguments. **`drawerCabinetCellOrigin(rowIndex, columnIndex, dimensions)` is retained with its existing signature and call sites but reimplemented to delegate**: `{ x: rowLayout(rowIndex).drawerOrigins[columnIndex], y: rowIndex·(cellHeight + iT) }`. In uniform mode the delegation reproduces the committed `columnIndex·(cellWidth + iT)` exactly (degeneracy proof, §6), so the existing model/Finished-View/fixture call sites keep working unchanged and byte-identically. This satisfies the no-parallel-formulas requirement: model generation, Finished View, layout, metrics, and fixtures all consume `drawerCabinetRowLayout` (directly or through the delegating `cellOrigin`), and no second width formula may exist anywhere — enforced by fixture 15/24 in §14.

## 10. ID and ordering contract

**Recommended prefix policy — cabinet-level, not per-row.** `drawerCabinetDrawerPrefix(row, column, columns)` keeps its single-source-of-truth role with one semantic change: its third argument becomes **`maximumColumns`** (cabinet-wide maximum). The no-`cNN` form applies **only when `maximumColumns === 1`** — i.e., precisely the cabinets that are one-column today (every row has exactly one drawer), which is exactly the set of cabinets whose IDs any existing LightBurn file or fixture can reference. Any cabinet containing a multi-drawer row uses `cNN` on **every** drawer, including single-drawer rows (`drawer-r02-c01-*` for the wide drawer in a 1/3 layout).

The task's illustrative alternative (per-row style: `drawer-r02-*` beside `drawer-r01-c01-*`) was challenged as instructed and **rejected** for three concrete reasons: (1) *prefix ambiguity* — `drawer-r02` is a string prefix of `drawer-r02-c01`, so any future prefix-based filtering (or a human typing a LightBurn selection filter) can silently match the wrong set when styles mix within one cabinet; (2) *branch proliferation* — every consumer that constructs or parses drawer IDs would need per-row conditionals instead of one cabinet-level flag; (3) *semantic honesty* — a single-drawer row genuinely is "column 1 of 1", and `c01` states that; whereas the legacy bare form's actual purpose (protecting existing saved names) applies only to all-ones cabinets, which the cabinet-level rule preserves perfectly. Panel *names* follow the same cabinet-level policy (`Drawer 2, 1` for the wide drawer) so names and IDs never disagree about structure.

**Uniform-mode identity:** `maximumColumns === columns` in uniform mode, so the changed third argument is value-identical at every existing call site — bytes preserved. Custom 3/3/3 and uniform 3×3 produce identical prefixes ✓. Collision-freedom: within any cabinet, prefixes are either all bare (all-ones) or all `cNN` — uniqueness follows from `(row, column)` uniqueness as today.

**Deterministic panel and layout order** (custom mode; uniform order is the committed order unchanged): 1. shell panels in the committed order (bottom, top, left, right, back); 2. shelves `r01…`; 3. partitions by row then boundary (`r01-c01, r01-c02, r02-c01, …` — rows **bottom-to-top**, matching the committed uniform layout-row order, which already emits partitions and drawers in ascending row order); 4. drawers by row (bottom-to-top) then column (left-to-right), each keeping the committed two-layout-row split (`[bottom, front, back], [left, right]`). Production row order is bottom-to-top ascending exactly as committed — only the *UI* presents top-first (§4).

## 11. UI contract

Smallest clear structure, in the existing `cabinetFields` block:

- **`Drawer layout`** select (`Uniform grid` / `Custom by row`) placed directly before `Drawer rows`. Added to the full-`render()` trigger list (like `couponType`/`drawerUseSeparateInteriorThickness`), since it changes which fields are visible.
- **Uniform:** `Drawer rows` + `Drawer columns` render exactly as today.
- **Custom:** `Drawer rows` stays; `Drawer columns` is **hidden** (its session value is preserved untouched, following the exact precedent of the interior-thickness field when the separate checkbox is off — dormant, not cleared, not seeding anything); per-row selects render **top-to-bottom in UI order**: `Top row drawers` (rows ≥ 2), `Middle row drawers` (rows = 3), `Bottom row drawers` (always). At `rows === 1` a single select renders labeled **`Drawers in row`** (fed by `drawerBottomColumns`). **UI ordering OPTION 1 (top-first) is selected**: a builder reads a cabinet face top-down, the task's own notation is top-down, and bottom-up UI order would put the most-commonly-customized row (the wide top drawer of 1/3) last. The bottom-up internal mapping is normalization's job alone (§4).
- Help text on the mode select: *"Custom by row keeps equal drawer heights; each row holds 1–3 equal-width drawers. The row with the most drawers uses the entered drawer inside width; rows with fewer drawers get proportionally wider drawers."* — this doubles as the `drawerInsideWidth` clarification, avoiding a second help-text block.
- Mode switching is fully predictable: values never migrate between `drawerColumns` and the per-row fields in either direction; switching modes only changes which fields are read by normalization. No hidden event-history state.
- No other GUI changes; the planned broad GUI cleanup remains explicitly out of scope.

## 12. Metrics and warnings

**Metrics (custom mode; uniform-mode metrics unchanged):**
- `Drawer layout` → `Custom by row` (uniform shows `Uniform grid`).
- `Row configuration` → top-to-bottom, e.g. `Top 1 / Middle 2 / Bottom 3`.
- Replace the single `Drawer inside` / `Drawer outside` lines with **per-row lines** whenever more than one distinct width exists, e.g. `Top row drawers: 1 × 318.9 mm inside width` / `Bottom row drawers: 3 × 100 mm inside width` (one line per row, top-to-bottom; collapse to the current single line when all rows share one count). Never display a single `drawerInsideWidth` as if it described all drawers when it does not.
- `Drawer grid / count` becomes `Drawer layout / count` sourced from `rowDrawerCounts` (`drawerCount = Σ counts`); `partitionCount`, shelf count, thickness mode/values, `Required pieces` (semantic `pieceCount`), production file count, and sheet-bound metrics all continue from existing semantic fields unchanged in shape.

**Warnings (added only in custom mode, deduplicated as today):** drawers are equal width within each row but rows may differ — mixed widths within one row are not supported; partitions remain manually positioned and glued with no placement marks — dry-fit every row and confirm equal bay widths before gluing (extends the committed multi-column warning to name per-row bays); wide drawers in low-count rows may rack or flex depending on material and load; drawers at minimum dimensions may become fragile; separate-thickness mode still requires separate stock and laser settings (existing warning, unchanged); the first physical custom-row build must verify fit and squareness. No structural-performance claims.

## 13. Production output, layout, Finished View, storage, Production Settings

- **Production output contract — no change needed, confirmed against source.** Separate-mode membership is role-based (`shellIds` set + negation) and cannot see row topology; `productionOutputs` shape, filenames (`l8-drawer-cabinet-shell/interior-<date>.svg`), MIME, per-output score/label filtering (already per-output panel-membership-filtered), fallback suppression (`requiresMultipleProductionOutputs`), and preview/download identity all carry over untouched. Linked mode remains one SVG/preview/download.
- **Flat layout.** `layoutDesignPanelRows()` is already width-agnostic per panel; each drawer keeps its two-layout-row split, partitions keep their per-row layout row, so differing drawer widths per row are just wider/narrower entries in existing row structures. Sheet dimensions grow with wide drawers (a 1-drawer row's bottom panel approaches `W` wide); the existing >400 mm sheet warning covers this. No nesting.
- **Finished Front View.** Bounded extension consuming only `drawerCabinetRowLayout` data via the existing elevation object: per-row openings of `rowCellWidth`, fronts of `rowDrawerOutsideWidth`, row-specific partition bands at `partitionOrigins`, shelves and shell bands unchanged, separate-thickness note unchanged. No production SVG reads; no independent width re-derivation (fixture-enforced). Screen row order remains the existing top-down conversion, which matches UI labels by construction.
- **Storage.** All new fields live in `designDraft` only; `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`, `loadState()`, `persist()`, `backupObject()`, `replaceData()`, `mergeData()`, import/export untouched; absent fields preserve current behavior (§7).
- **Production Settings.** No new controls. Measured-thickness application continues to write `materialThickness` (= shell), and separate-mode interior thickness remains the user's explicit field; row layout is invisible to Production Settings. Confirmed no applicability change is needed since the committed mapping is thickness-only.

## 14. Fixture plan (representative, ~28–34 assertions)

1. Missing/blank/`'uniform'` `drawerLayoutMode` normalizes to uniform; unknown nonblank value errors.
2. Uniform mode ignores stale/malformed dormant per-row fields (mirror of the committed stale-interior-thickness fixture).
3. Uniform 1×1, 2×2, 3×3 outputs byte-identical to a draft with no layout fields at all.
4. All three legacy one-column goldens (4456/`a6dd23dc`, 6802/`36a41b07`, 9153/`8c286797`) re-asserted (already present — must keep passing).
5. Linked 2×3 golden (16429/`0814ca2e`) and 6. separate shell/interior goldens (2779/`956ad870`, 8609/`bad8b97f`) unchanged in uniform mode (already present).
7. Custom mode validates only active rows; active counts reject non-integers and out-of-range.
8. Two-row custom cabinet ignores a malformed dormant middle-row value.
9. **Top/bottom mapping pinned**: a 1/3 (top-to-bottom) draft yields `rowDrawerCounts=[3,1]` bottom-up, wide drawer in `r02`.
10. 1/3 → 4 drawers, 2 partitions; 11. 2/1 → 3 drawers, 1 partition; 12. 1/2/3 → 6 drawers, 3 partitions (counts hand-derived, not read back from the formula).
13. Custom 3/3/3 SVG === uniform 3×3 SVG, byte-identical.
14. `maximumColumns` drives `cabinetInsideWidth`; a 1/3 cabinet's width equals the uniform 3-column width for the same inputs (literal expected value hand-computed: 325.35 at defaults).
15. Row closure: for each active row, `n·rowCellWidth + (n−1)·iT` equals `cabinetInsideWidth` within 1e-5 — computed from returned helper values, not by re-running the formula.
16. Equal widths within each row (all `drawerOrigins` gaps equal); 17. fewer-drawer rows produce strictly wider drawers; 18. all derived widths finite and positive, with the literal 318.9/154.725/100 expectations from §6 asserted for the default-input 1/2/3 case.
19. Every distinct-count drawer submodel is valid and each row's namespaced panels match its submodel's dimensions.
20. IDs deterministic and collision-free across a 1/3 cabinet (set size equals panel count); 21. all-ones custom cabinet (e.g. 1/1) keeps bare `drawer-rNN-*` prefixes and equals uniform 1-column bytes; 22. mixed cabinet uses `cNN` on every drawer including the single-drawer row (`drawer-r02-c01-*` present, bare `drawer-r02-bottom` absent).
23. `partitionCount = Σ(n−1)`; 24. partition x positions match `partitionOrigins` and differ per row in a 2/3 cabinet; 25. partition faces remain `cellHeight × cabinetInsideDepth` with interior metadata (re-using the committed literal-33.3×76.5-style assertion at defaults).
26. Shelves remain `rows−1` full-width; 27. shell panel paths byte-identical between uniform 3×3 and custom 1/3 at equal cabinet dimensions... (only when dimensions actually coincide — assert shell path identity between custom 3/3/3 and uniform 3×3, and separately that a 1/3 cabinet's shell equals a uniform 3-column cabinet's shell at the same rows, since W is identical by §6).
28–30. Linked, separate-shell, separate-interior layouts finite/non-overlapping for a representative custom case; 31. separate-output panel accounting exact (union = pieceCount, each panel once) for a custom 1/3 separate-mode cabinet.
32. Preview/download identity per output (live DOM fixture, custom mode); 33. generic fallback suppressed in separate custom mode.
34. Finished View shows per-row front counts/widths from semantic data and contains no production-SVG dependency; 35. Finished View front widths equal `rowDrawerOutsideWidth` (cross-checked against the helper, not re-derived).
36. Metrics render row configuration and per-row widths (HTML-string inclusion check, as the committed Required-pieces fixture does).
37. Storage/backup byte-identity around custom-mode generation and downloads; 38. Production Settings application unchanged (existing fixtures keep passing).
39. Unrelated templates unchanged (existing goldens keep passing).
40. New custom-mode goldens (e.g. linked 1/3, separate 1/2/3) pinned **only after** assertions 10–31 pass structurally; hashes are confirmation, never primary proof.

## 15. Risk register

| # | Severity | Finding |
|---|---|---|
| 1 | Major (designed out) | Ambiguity of `drawerInsideWidth` in custom mode — resolved by Option A's exact contract (§5/§6): it is always the max-count row's drawer width; per-row metrics expose every derived width; help text states the rule |
| 2 | Major (designed out) | UI top-first vs internal bottom-up reversal — confined to one normalization mapping with a dedicated pinning fixture (§4, fixture 9); no other layer may translate row order |
| 3 | Major (designed out) | Duplicated row-width formulas across model/Finished View/metrics — prevented by the single `drawerCabinetRowLayout` helper with `cellOrigin` delegating (§9) |
| 4 | Major (designed out) | Prefix collision/ambiguity in mixed row counts — resolved by the cabinet-level `maximumColumns` prefix policy (§10), which also preserves all legacy IDs verbatim |
| 5 | Minor | Rounding of the per-row division at 1e-6 precision — closure holds at physical precision; epsilon guidance in §6; enforced by fixture 15 and golden identity 13 |
| 6 | Minor | Warning-text drift: per-row prefixed submodel warnings must keep uniform-mode wording byte-compatible with current messages (uniform mode keeps the bare `"Drawer:"` prefix) |
| 7 | Minor | Metrics clutter from per-row width lines — bounded at ≤3 lines; collapse to the single current line when all rows share one count |
| 8 | Informational | Sheet growth for wide-drawer rows is real but bounded and already covered by the existing >400 mm warning |
| 9 | Informational | Custom 3/3/3 ≡ uniform 3×3 byte identity is a free consequence of the single-representation design, asserted rather than special-cased |

No Blocker-severity risk was found. The dual-output safety model, storage isolation, and every protected boundary in the task's list (serializer, other templates, import/export, Production Settings schema, evidence/promotion, Inventory/Projects/Pricing/Library/Test Grid/Material Test) require no change under this design.

## 16. Physical honesty and unavailable checks

Software review cannot verify: real drawer fit, plywood thickness variation, kerf, manual partition placement accuracy, glue strength, wide-drawer racking, load capacity, squareness, LightBurn import, or machine cutting. First physical prototypes should include a 1/3 and a 2/1 cabinet, and one mixed-thickness custom-row cabinet once implemented — but no geometry question in this review depends on a physical cut, so none is required before implementation. No live browser/runtime execution was performed *in this turn*; the 264/945/1737 baseline totals rest on the committed README plus this session's immediately prior live CDP run against byte-identical content, as disclosed in §0. LightBurn import behavior of mixed-width layouts remains unverified.

## Final verdict

**READY FOR IMPLEMENTATION**

The bounded recommendation survives its own challenge: per-row equal-division is the smallest model that delivers every stated use case; the width contract (Option A) preserves the existing input's meaning exactly and recovers uniform behavior as a proven degenerate case rather than a special case; closure is exact by construction; the ID policy extends the legacy contract at cabinet level without touching any ID that exists today; and the production-output, storage, and Production Settings contracts need no change at all. The implementation's genuinely delicate obligations — uniform byte-identity through the single-representation refactor, the one-place top/bottom mapping, and the cabinet-level prefix rule — are each pinned by specific fixtures in §14 rather than left to discipline. Scope on the order of the multi-grid phase itself; plan its implementation and audit accordingly.
