# Drawer Cabinet Phase 5.1 — Independent Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Baseline commit:** `67860a6` — Refine sliding-lid guides and add assembly labels  
**Audit date:** 2026-07-16  
**Mode:** Read-only independent review (this report is the only file written)

---

## Executive summary

**No blockers found.**

The Drawer Cabinet template is a bounded, session-only extension with dedicated geometry (`drawerCabinetDimensions`, `buildDrawerCabinetModel`, `buildDrawerCabinetDesignResult`), correct open-front depth semantics, independent cabinet/drawer clearances, deterministic piece counts (10 / 16 / 22), score-before-cut SVG serialization, and unchanged Finger Box / Sliding Lid signatures. Live execution confirms **Designs 424 / 0**.

Residual risk is physical: plain edge-glued support shelves, drawer running fit, and cabinet squareness remain unproven until scrap is cut and assembled.

---

## 1. Baseline verification

### Commands run

```powershell
git status -sb
git rev-parse --short HEAD
git log -1 --oneline
git diff --check
git diff --stat 67860a6
git diff 67860a6 -- index.html README.md
```

### Results

| Check | Result |
|-------|--------|
| `HEAD` | `67860a6` ✓ |
| Tracked modifications | `index.html`, `README.md` only ✓ |
| Staging area | Empty ✓ |
| `git diff --stat 67860a6` | `README.md` 4 lines; `index.html` +216 / −12 (204 net) ✓ |
| `git diff --check` | Clean (LF→CRLF warnings only) ✓ |
| Unrelated untracked files | Preserved ✓ |
| Destructive Git actions | None performed ✓ |

---

## 2. Scope reviewed

- `index.html`: drawer-cabinet template, geometry, layout, labels, fixtures (+204 net lines vs `67860a6`)
- `README.md`: Designs description and fixture total update
- `docs/DRAWER_CABINET_PHASE5_1_IMPLEMENTATION_2026-07-16.md` (read for context; not trusted as proof)
- `docs/DESIGNS_DRAWER_CABINET_ARCHITECTURE_REVIEW_2026-07-16.md` (read for context)

No changes to `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`, `persist()`, or `backupObject()` in the diff.

---

## 3. Findings

### Blocker

None.

---

### Major

#### M1 — Upper-row support shelves are software-correct but physically under-constrained

| Field | Detail |
|-------|--------|
| **Severity** | Major |
| **Region** | `buildDrawerCabinetModel()` shelves (~L1836); assembly text (~L2513) |
| **Explanation** | Shelves are plain `cabinetInsideWidth × cabinetInsideDepth` rectangles with no dados, ledges, tabs, pins, or alignment marks. Support relies on edge glue to side walls and back at computed elevations. |
| **Consequence** | SVG and formulas can be correct while shelf sag, slip during glue-up, or rack still ruin a multi-row build. |
| **Recommended correction** | None before commit; mandatory scrap prototype for 2–3 rows. Future phase may add cleats, shallow dados, or jig marks. |
| **Fixture detection** | **No** — warnings only; no mechanical registration test exists |

#### M2 — Most dimension fixtures mirror production formulas

| Field | Detail |
|-------|--------|
| **Severity** | Major (test-quality, not geometry defect) |
| **Region** | `runDesignGeometryFixtures()` cabinet block (~L2944–2999) |
| **Explanation** | 48 new assertions largely compare `buildDesignResult()` outputs to hardcoded values derived from the same formulas in `drawerCabinetDimensions()`. |
| **Consequence** | A systematic formula error could pass many tests together. |
| **Recommended correction** | Add hand-derived elevation table and one independent panel-coordinate spot-check per shell piece. |
| **Fixture detection** | Partial — mating-pair and clearance-isolation tests are stronger |

#### M3 — No 3D assembly or physical cut verification

| Field | Detail |
|-------|--------|
| **Severity** | Major (process gap) |
| **Region** | Entire template |
| **Explanation** | Implementation report explicitly defers cutting, LightBurn import, and assembly. |
| **Consequence** | Commit is architecturally sound; production use still needs measured-sheet prototype. |
| **Recommended correction** | One-row cabinet first, then two-row with shelf glue-up. |
| **Fixture detection** | **No** |

---

### Minor

#### N1 — Cabinet results UI shows mojibake multiplication signs

| Field | Detail |
|-------|--------|
| **Severity** | Minor |
| **Region** | `designResultsHtml()` cabinet metrics (~L2456–2464) |
| **Explanation** | Serialized strings contain `Ã—` and `â€”` instead of `×` and `—`. |
| **Consequence** | Cosmetic display corruption in Design results panel only; SVG geometry unaffected. |
| **Recommended correction** | Replace with UTF-8 `×` / `—` or ASCII `x` / `-`. |
| **Fixture detection** | **No** |

#### N2 — Global kerf note still references lid clearances on cabinet template

| Field | Detail |
|-------|--------|
| **Severity** | Minor |
| **Region** | `renderDesigns()` kerf paragraph (~L1520) |
| **Explanation** | Cabinet form inherits the sliding-lid-oriented kerf sentence for all templates. |
| **Consequence** | Slightly confusing UX; does not alter geometry. |
| **Recommended correction** | Template-conditional kerf help for cabinet. |
| **Fixture detection** | **No** |

#### N3 — Support shelves are intentionally unlabeled

| Field | Detail |
|-------|--------|
| **Severity** | Minor |
| **Region** | `designAssemblyLabelSpecs()` cabinet branch (~L2064–2073) |
| **Explanation** | Shelves omitted from label specs. |
| **Consequence** | Glue-up relies on numeric shelf elevations in results/assembly text. |
| **Recommended correction** | Optional shelf labels or row markers in a later pass. |
| **Fixture detection** | N/A (by design) |

---

### Verified

#### V1 — Cabinet shell geometry and open-front depth

For thickness `t`, drawer inside `Wi × Di × Hi`, lateral `Cl`, vertical `Cv`, rear `Cr`:

```text
drawer outside width/depth/height = Wi+2t, Di+2t, Hi+t
cell width/height                 = drawer outside width + Cl, drawer outside height + Cv
cabinet inside width/depth/height = cell width, drawer outside depth + Cr, rows×cellHeight + (rows-1)×t
cabinet outside width/depth/height = inside width + 2t, inside depth + t, inside height + 2t
```

**Default Golden A** (`t=3`, `100×70×30`, `Cl=0.4`, `Cv=0.3`, `Cr=0.5`, 1 row) — independently verified live:

| Quantity | Expected | Actual |
|----------|----------|--------|
| Drawer outside | 106 × 76 × 33 | ✓ |
| Cabinet inside | 106.4 × 76.5 × 33.3 | ✓ |
| Cabinet outside | 112.4 × 79.5 × 39.3 | ✓ |
| Outside depth − inside depth | `t` = 3 | ✓ |

Open front adds **one** back-panel thickness to outside depth, not two. Correct.

#### V2 — Five-panel open-front shell topology

| Panel | Role | Front edge |
|-------|------|------------|
| `cabinet-bottom` | Floor | Plain (open front) |
| `cabinet-top` | Lid rail | Plain (open front) |
| `cabinet-left` / `cabinet-right` | Sides | Plain at front |
| `cabinet-back` | Closed back | Fully fingered |

Panel dimensions: bottom/top `outsideW × outsideD`; sides `outsideD × outsideH`; back `outsideW × outsideH`. Physically buildable open-front carcass.

#### V3 — Eight cabinet mating boundaries

Live independent check (default 1-row): all eight fixture pairs show **opposite phases** and **matching pattern boundaries**.

Edge assignment follows proven open-top box convention: bottom/top width fingers mate back; bottom/top depth fingers mate sides; side height fingers mate back vertical edges; front remains plain on bottom, top, and sides.

#### V4 — Cabinet vs drawer joint clearance independence

Fixtures confirm `cabinetJointClearance` changes only `cabinet-*` paths and `drawerJointClearance` changes only `drawer-*` paths. Verified in diff (~L2980–2982).

#### V5 — Drawer geometry and namespacing

- Each drawer via `buildBoxModel()` open-top (`lid:'open'`) — five panels.
- `namespaceDesignPanel()` produces `drawer-r01-*`, `drawer-r02-*`, `drawer-r03-*`.
- Three-row cabinet: **15 drawer panels**, **5 unique local path shapes** (front≡back, left≡right — expected symmetry), **15 unique IDs**.
- Drawer inside dimensions preserved in output metrics.

#### V6 — Multi-row height accumulation

| Rows | Inside height formula | Default value | Shelf bottom elevations (mm) |
|------|----------------------|---------------|------------------------------|
| 1 | `1×cellHeight` | 33.3 | — |
| 2 | `2×cellHeight + 1×t` | 69.6 | 33.3 |
| 3 | `3×cellHeight + 2×t` | 105.9 | 33.3, 69.6 |

Vertical stacking interpretation (verified):

- Row *n* cell occupies `cellHeight` above prior structure.
- Each inter-row gap includes shelf material thickness `t`.
- Row 2 drawer floor = `cellHeight + t` = 36.3 mm above inside cabinet floor.
- No omitted or double-counted `t` in height formula.

#### V7 — Support shelves

- Count: `rows − 1` (`cabinet-shelf-r01`, `cabinet-shelf-r02`).
- Size: `cabinetInsideWidth × cabinetInsideDepth` — fits between side inner faces and spans to back.
- Elevations reported in metrics and assembly guidance.
- Bottom drawer rests on cabinet bottom (no shelf).

#### V8 — Clearances

| Parameter | Semantics | Verified |
|-----------|-----------|----------|
| `drawerLateralClearance` | **Total** across both sides (`Cl/2` per side in UI) | ✓ |
| `drawerVerticalClearance` | Added once per row cell | ✓ |
| `drawerRearClearance` | Gap behind drawer back | ✓ |
| Blocking | `Cl ≥ drawer outside width`, etc. | ✓ |
| Warnings | Zero / large clearances | ✓ |
| Kerf | Explicitly separate in UI/README | ✓ |

Insertion consistency check (~L1840–1841):

```text
cabinetInsideWidth − Cl = drawerOutsideWidth
cellHeight − Cv = drawerOutsideHeight
```

#### V9 — Piece counts and IDs

| Rows | Physical pieces | Live count |
|------|----------------:|-----------:|
| 1 | 10 | 10 ✓ |
| 2 | 16 | 16 ✓ |
| 3 | 22 | 22 ✓ |

Stable ID strings fixture-locked for all three row counts. Every physical copy serialized separately in layout/SVG.

#### V10 — SVG and LightBurn behavior

- `serializeDesignSvg()`: blue `g#score` (labels in `g#assembly-labels`) precedes red `g#cut`.
- Preview/download share `buildDesignResult()` / `designSvg()` string.
- No SVG `<text>`; vector glyph paths only (20 labels on 3-row labeled run; 0 text nodes).
- Labels optional; omitted labels do not block valid cut export (existing label containment pipeline).
- Finite coordinates; `designSvgValidation()` passes for 1/2/3-row outputs.

#### V11 — Storage and legacy isolation

- Drawer fields remain in `designDraft` only.
- Fixture: generation with labels does not mutate `localStorage`.
- Legacy signatures unchanged (live):

| Template | Length | Hash |
|----------|-------:|------|
| Finger box open | 2559 | `4e2a6f4b` |
| Sliding lid disabled | 2800 | `4a7ab718` |

#### V12 — Fixture totals

| Metric | Value | Method |
|--------|------:|--------|
| New cabinet assertions | 48 | Static count in diff block |
| Designs runtime | **424 / 0** | Live Playwright harness |
| Full suite (reported) | **956 / 0** | 908 + 48 arithmetic; Designs verified live |

---

## 4. Coverage map (fixture quality)

| Requirement | Coverage |
|-------------|----------|
| Cabinet dimension formulas | Present — mostly circular |
| Open-front depth semantics | Present |
| Eight shell mating pairs | Present — phase/boundary only |
| Cabinet/drawer clearance isolation | Present — strong |
| Piece counts 10/16/22 | Present |
| Shelf count and elevations | Present |
| Layout overlap / SVG validity | Present |
| Score/cut order | Present |
| Label containment | Present (3-row labeled) |
| Physical shelf strength | **Absent** |
| 3D fold / scrap assembly | **Absent** |
| Independent panel coordinate table | **Absent** |

---

## 5. Physical construction assessment

### Software vs physical

| Item | Software | Physical |
|------|----------|----------|
| Open-front five-panel shell | Correct finger topology | Standard carcass; scrap fit still needed |
| Equal drawers | Correct emission | Drawer fit depends on `Cl/Cv/Cr` and kerf |
| Edge-glued shelves | Correct size/count/elevation | **High glue-up risk** without mechanical registration |
| Running clearances | Nominal geometry only | Glue squeeze-out, finish, warp, kerf not modeled |

### Assembly guidance quality

Results panel and assembly message state:

- Open front / closed back
- Bottom drawer on cabinet floor
- Upper shelves glued between sides and back at listed elevations
- Kerf excluded; scrap prototype required

Adequate for Phase 5.1 with documented manual follow-up.

---

## 6. Conclusion

| Verdict | Detail |
|---------|--------|
| **Blockers** | **None** |
| **Recommendation** | Safe to commit with **documented manual follow-up**: cut a one-row cabinet, then a two-row cabinet with shelf glue-up on measured scrap before production multi-row use. |

---

## 7. Final git status

```text
## main...origin/main
 M README.md
 M index.html
?? <pre-existing unrelated untracked files unchanged>
```

`HEAD` remains `67860a6`. Nothing was edited (except this report), staged, committed, pushed, reset, cleaned, stashed, moved, or deleted.

---

## 8. Independent validation performed

- Full `git diff 67860a6` inspection
- Hand-derived dimension verification
- Live Playwright harness on local `index.html` (temp files under `%TEMP%\drawer-cabinet-audit-2026-07-16\` only)
- Independent eight-joint mating phase check
- `python -m html.parser index.html` (parseable)