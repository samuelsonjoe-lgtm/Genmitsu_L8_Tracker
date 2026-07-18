# Drawer Cabinet Multi-Grid + Dual-Thickness — Design Challenge (2026-07-17)

**Mode:** Read-only adversarial design challenge (no file edits, no git writes).  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Primary proposal:** `docs/DESIGNS_DRAWER_CABINET_MULTI_GRID_DUAL_THICKNESS_DESIGN_REVIEW_2026-07-17.md`  
**This document:** Independent challenge of production-critical contracts before any implementation.

---

## 1. Repository state

| Check | Result |
| --- | --- |
| `git log -1 --oneline` | **`8bcfbbc` Add Dice tray underside cover plate** |
| Matches expected baseline `8bcfbbc`? | **Yes** |
| `git status -sb` | `## main...origin/main` — **tracked tree clean** |
| `git diff` / staged | **Empty** |
| Unexpected tracked modifications | **None** |
| Untracked | Pre-existing docs / LightBurn / `debug.log` / utility — **untouched** |
| Read-only status | **Confirmed** (this challenge only **created** this report under `docs/`) |

### Baseline fixture totals (committed)

From `README.md` at `8bcfbbc` and prior independent Edge runs on this baseline family:

| Suite | Total |
| --- | --- |
| Tray-model | **264** |
| Designs geometry | **922** |
| Complete suite | **1714** |

Do not use older 887/906/1698-era totals for this design.

---

## 2. Files and functions inspected

- `index.html` (committed): `drawerCabinetDimensions`, `buildDrawerCabinetModel`, `namespaceDesignPanel`, `drawerCabinetShelfGuideSegments`, `buildDrawerCabinetFrontElevation` / Svg, `buildDrawerCabinetDesignResult`, `serializeDesignSvg`, `designSerializedPathGroup`, `layoutDesignPanelRows`, `normalizeDesignDraft` (drawer-cabinet), `renderDesigns` cabinet UI, `downloadCurrentDesignSvg`, `refreshDesignPreview`, production-settings mapping for `drawer-cabinet`, fixtures for cabinet IDs/goldens/shelf-guides  
- `README.md`  
- Primary design review (full)  
- Context: architecture review, shelf-guide / assembly-label patterns (as cited by primary review)  
- Search for multi-download / `productionOutputs` / alternate SVG exports: **none exist** in the app today  

---

## 3. Part A — Geometry foundation (verified)

| Claim | Source evidence | Verdict |
| --- | --- | --- |
| Five finger-jointed shell panels | `cabinet-bottom/top/left/right/back` via `buildSlidingLidBodyPanel` + `cabinetEdge` | **Confirmed** |
| Shelves plain glued rectangles | `designPanelFromPoints` only; no finger pattern | **Confirmed** |
| Drawers = Finger Box open-top submodel | `buildBoxModel({…, lid:'open'})` then `namespaceDesignPanel` | **Confirmed** |
| Rows 1–3 only | `drawerRows` select; validation | **Confirmed** |
| Single shared thickness | `t = parameters.materialThickness` for shell, drawers, shelves, guides | **Confirmed** |
| No vertical partitions | No partition IDs/panels | **Confirmed** |
| No mixed-thickness direct joint | One `t` everywhere | **Confirmed** |

### Role split (shell vs interior)

| Role | Pieces | Finger-jointed to |
| --- | --- | --- |
| **shellThickness** | Bottom, Top, Left, Right, Back | Each other only |
| **interiorThickness** | Drawer five panels, shelves, new partitions | Drawer-to-drawer only; shelves/partitions **unglued joinery** (glue only) |

**No unequal-thickness finger-jointed pair** exists under this split. That part of the primary review is **sound**.

---

## 4. Part B — Multi-grid formulas

### Current source (single column, one `t`)

```text
drawerOutsideWidth  = insideW + 2t
drawerOutsideDepth  = insideD + 2t
drawerOutsideHeight = insideH + t          // open-top: bottom only
cellWidth  = drawerOutsideWidth + lateralClearance
cellHeight = drawerOutsideHeight + verticalClearance
cabinetInsideWidth  = cellWidth
cabinetInsideDepth  = drawerOutsideDepth + rearClearance
cabinetInsideHeight = rows·cellHeight + (rows−1)·t
cabinetOutsideWidth  = cabinetInsideWidth + 2t
cabinetOutsideDepth  = cabinetInsideDepth + t   // back contributes ONE thickness
cabinetOutsideHeight = cabinetInsideHeight + 2t
```

### Proposed multi-grid dual-thickness

Formulas in the primary review match the natural extension:

- replace grid-gap and drawer/shelf thickness terms with **interiorThickness**;
- replace shell outside padding with **shellThickness**;
- `cabinetInsideWidth = columns·cellWidth + (columns−1)·interiorThickness`.

| Check | Verdict |
| --- | --- |
| Back depth contribution | **One** shell thickness — matches source; proposal correct |
| Drawer outside height | **+ one** material thickness (open top) — matches `buildBoxModel` open-top; proposal correct |
| Each shelf consumes one interior thickness | Yes (gap term `(rows−1)·interiorThickness`) |
| Each partition consumes one interior thickness in width | Yes (gap term `(columns−1)·interiorThickness`) |
| Clearances per opening | `cellWidth/Height` include lateral/vertical once per cell — correct; not per-cabinet |
| 3×3 equal openings | Equal cells by construction; no cumulative pitch drift if one origin formula is used |
| Formula substitution `columns=1`, equal thickness | Collapses **exactly** to current `drawerCabinetDimensions` |

### Hand-derived representative numbers (default drawer 100×70×30, clearances 0.45/0.30/0.50, t=3)

| Config | Outside W×D×H (approx) |
| --- | --- |
| 2-row × 1-col | 112.45 × 79.5 × 75.6 |
| 3×3 equal 3 mm | 331.35 × 79.5 × 111.9 (width already triggers existing large-layout warnings) |
| 3×3 shell 6 / interior 3 | 337.35 × 82.5 × 117.9 |

### Source-of-truth plan (required)

| Concern | Single owner |
| --- | --- |
| Shell outside/inside | `drawerCabinetDimensions()` only |
| Cell size | same function |
| Cell origin (r,c) | **one** helper used by model layout *and* Finished View |
| Shelf elevations | derived only from dimensions + interiorThickness |
| Partition positions | same cell-origin helper |
| Drawer submodel | one `buildBoxModel` call with `interiorThickness` |

**Do not** re-derive outside dimensions in guide, Finished View, or layout code.

---

## 5. Part C — Legacy byte identity

### Feasible when

- `columns` absent or `1`;
- separate-interior mode **off** (linked thicknesses);
- `shellThickness === interiorThickness ===` current `materialThickness` path;
- layout row builder is a **strict superset** of today’s row list;
- drawer IDs stay `drawer-r0N-*` (no `-c01`) when `columns===1`;
- shelf IDs stay `cabinet-shelf-r0N`;
- score subgroup still `shelf-guides` when only shelf guides fire;
- filename `l8-drawer-cabinet-${today()}.svg`, MIME `image/svg+xml;charset=utf-8`.

### Hard fixtures that require conditional IDs

Pinned ID strings and goldens in source include:

- `drawer-r01-bottom|…` (no column token);
- goldens **4456/a6dd23dc**, **6802/36a41b07**, **9153/8c286797**;
- Finished View `data-panel-prefix="drawer-r0N"`.

A naive always-`drawer-r01-c01-*` scheme would **break** those fixtures and any saved LightBurn name selections. Conditional naming is **mandatory**, not optional polish.

### Normalization defaults

Absent new fields must normalize to:

- `columns = 1`;
- linked thicknesses from existing `materialThickness`;
- current row/clearance/guide behavior.

**Byte-identical legacy output is achievable** if the above are treated as hard acceptance tests, not hope.

---

## 6. Part D — Single-thickness linking contract (**Major correction**)

### Why the primary review’s UI is not safe

Primary review recommends two always-visible fields (shell + interior), both defaulting to 3 mm, with **no** mode toggle.

**Failure mode:**

1. Today, user changes **Material thickness** 3 → 6 mm → shell, drawers, and shelves all become 6 mm.  
2. Under two independent fields, “Shell thickness” becomes 6 while “Drawer and interior thickness” remains 3 → **silent mixed thickness** without any intentional dual-stock decision.  
3. Production Settings apply **only** `materialThickness` for Drawer Cabinet (`productionSettingDesignApplicability` mapping). After a naive split, applying measured thickness would update shell only unless linking is explicit — another silent mixed path.

That is an **accidental behavior change** of the existing single-thickness workflow. Severity: **Major**.

### Rejected options

| Option | Why not |
| --- | --- |
| 1 Always-visible independent values | Silent mixed thickness (above) |
| 3 Blank means match shell | Easy to misunderstand; hard to show “linked” state in a plain number input |
| 4 Linked-until-edit | Hidden state, fragile event handling, poor fixture clarity |

### Selected contract — **OPTION 2 (checkbox gate)**

| Item | Contract |
| --- | --- |
| Raw fields | `materialThickness` (existing; shell stock when cabinet), `drawerUseSeparateInteriorThickness`, `drawerInteriorThickness` |
| Normalized | `shellThickness`, `useSeparateInteriorThickness`, `interiorThickness` |
| Enable separate mode | only `true` / `'true'` / `'on'` |
| When separate **false or absent** | `interiorThickness = shellThickness` always (re-sync every normalize) |
| When separate **true** | `interiorThickness` = validated `drawerInteriorThickness` (positive finite, same min as material thickness) |
| UI | Checkbox: **Use separate drawer/interior thickness** (default **off**). Interior thickness field **visible only when checked** (or disabled + ignored when unchecked). Shell field label for this template: **Shell thickness (mm)** (or keep “Measured material thickness” with help that it is the shell stock when separate mode is off — both thicknesses). |
| Defaults | `drawerUseSeparateInteriorThickness: false`; `drawerInteriorThickness: '3'` (or mirror shell default) but **ignored while unchecked** |
| Changing shell while linked | Updates both roles together |
| Session-only | No storage / schema / backup |
| Production Settings | Apply measured thickness to `materialThickness` / shell; while linked, interior follows automatically |

This preserves the mental model: mixed stock is an **explicit** action, not a side effect of editing one number.

---

## 7. Part E — Production output safety (**Major correction**)

### What the real serializer does

`serializeDesignSvg()` emits:

1. Optional blue **score** group (guides / labels).  
2. Exactly one red **`g#cut`** containing **every** panel, each as `g#panel-{id}` with a `<title>` and path.

There is:

- **one** `result.svg`;
- **one** Download LightBurn SVG button (`downloadCurrentDesignSvg` → `result.svg`);
- **no** per-group download, hide, or multi-file API;
- **no** thickness-colored cut layers (score is blue; cut is all red).

SVG titles do **not** create separate LightBurn cut layers. On import, the whole red cut group is one operation unless the operator carefully selects subsets by name every time.

### Why “one combined SVG + titles” is not operator-safe for mixed thickness

| Risk | Assessment |
| --- | --- |
| Same layer settings for shell and interior | **Yes** if user cuts all red |
| Cut thick shell geometry from thin stock | **Credible** |
| Cut thin drawers from thick stock | **Credible** |
| Combined sheet larger than either physical sheet | **Yes** — layout width/height warning is about a **virtual** mixed sheet, not either real sheet |
| Shell and interior cannot share one physical pass | **True** when thicknesses differ |
| Documentation-only mitigation | **Insufficient** for a production download path |

The primary review’s OUTPUT A is therefore **not acceptable** as the sole production contract for mixed-thickness mode.

### Selected contract — **OUTPUT D (conditional dual production files)**

Bounded, template-local, not a generic multi-document framework:

| Mode | Production behavior |
| --- | --- |
| **Linked / single thickness** (`useSeparateInteriorThickness === false`) | **Exactly current contract**: one `result.svg`, one preview cut layout, one download, one width/height. **Legacy goldens apply here.** |
| **Separate interior thickness** (`true`) | **Two production outputs only** (Drawer Cabinet-owned): **shell** and **interior**. Each has its own layout, sheet bounds, SVG, filename, MIME, thickness label. Download UI shows **two** cards/buttons. Combined virtual sheet is **not** the production download. |

#### Minimal result shape (evaluate; implement only for drawer-cabinet)

```text
result.svg                 // linked mode: full production SVG
                           // separate mode: either omit as production, or set to shell SVG
                           //   with explicit metrics flag that dual downloads are required
result.productionOutputs   // only when useSeparateInteriorThickness
  [
    {
      id: 'shell',
      label: 'Cabinet shell',
      thicknessMm,
      svg, filename, mime: 'image/svg+xml;charset=utf-8',
      widthMm, heightMm,
      panelIds: [...]
    },
    {
      id: 'interior',
      label: 'Drawers, shelves, and partitions',
      thicknessMm,
      svg, filename, mime,
      widthMm, heightMm,
      panelIds: [...]
    }
  ]
```

**Filename examples (session, no schema):**

- `l8-drawer-cabinet-shell-${today()}.svg`
- `l8-drawer-cabinet-interior-${today()}.svg`

**UI when separate mode on:**

- Cut Layout shows tabs or stacked previews: Shell sheet / Interior sheet (or two side-by-side cards).  
- Two download buttons, each wired to the matching `productionOutputs[].svg`.  
- Warning: pieces are **not** cut from one stock; do not use a combined file.  
- Optional screen-only “all parts” assembly diagram remains Finished View, not a cut file.

**All other templates:** continue to use only `result.svg` and the existing single download path — no shared multi-doc framework.

### Why not pure OUTPUT B always

Always forcing two files would change legacy single-thickness UX and risk byte/fixture churn. Conditional dual files keep **linked** mode byte-compatible.

### Why not titles-only OUTPUT A

Fails the audit’s own safety questions: no per-group download, one red group, same LightBurn layer settings, dishonest combined sheet size.

---

## 8. Part F — Multi-material layout

When separate mode is on:

| Rule | Contract |
| --- | --- |
| Shell panels | Layout only on shell sheet (margin/gap as today) |
| Drawers, shelves, partitions | Layout only on interior sheet |
| Overlap checks | Per-output only |
| Sheet warnings (`>400` layout, large-cabinet notes) | Per-output dimensions |
| Combined coordinate system | **Not** used for production downloads |

When linked: single layout as today (shell + interior pieces same thickness, one sheet is honest).

---

## 9. Part G — Vertical partition construction

### Selected: primary review Option A (row-height plain rectangles)

| Property | Decision |
| --- | --- |
| Count | `(columns − 1) × rows` |
| Thickness / width in plan | `interiorThickness` |
| Depth | `cabinetInsideDepth` |
| Height | **`cellHeight`** (full bay height including vertical clearance band) |
| Joinery | Glue only — no tabs into shell, no interlocking grid |
| Placement | Rest on bottom or shelf; capped by shelf or top |

**Why `cellHeight`:** Front elevation already models each opening height as `cellHeight`. Distance between successive shelf bottom faces is `cellHeight + interiorThickness`; the clear bay for a partition between floor/shelf and the next shelf underside is the opening span of one cell. Using full cabinet height would force shelf notches (new joint type) — out of phase-one scope.

### Physical honesty (no strength claims)

- Alignment is manual; thin partitions can lean or bow.  
- Glue contact is edge/face glue, not structural interlocking.  
- Misalignment can narrow a drawer bay or scrape drawers.  
- Row-height pieces avoid shelf notching but require **per-row** placement accuracy (errors do not auto-correct across rows).  
- Full-height interlocking grid is better for squareness but is a **later** joint-system feature.

---

## 10. Part H — Partition placement guides

### Decision: **GUIDE OPTION 4 for phase one** (defer partition guides)

| Mode | Behavior |
| --- | --- |
| `columns === 1` | Existing **shelf-placement guides** unchanged; disabled goldens preserved |
| `columns > 1` | Generate parts + grid geometry; **no** partition score marks in phase one |
| Warnings when `columns > 1` | Explicit: measure partition positions; dry-fit; optional future partition guides |
| Shelf guides when multi-column | May still mark **horizontal** shelf elevations on Left/Right/Back using **shell** thickness for tooth clearance math; must not invent partition ticks until a dedicated design |

**Why not force guides into phase one:** score geometry on bottom/top/shelf faces for vertical lines is a new orientation problem (today’s ticks are horizontal on side/back panels). Doing it wrong is worse than honest manual measurement.

**Follow-up (not phase one):** GUIDE OPTION 3 — separate **Partition-placement guides** checkbox; define faces (e.g. Bottom interior, underside of shelves, Top interior) and tick pairs spanning partition thickness; never alter red cuts; preserve shelf-guide bytes when partition guides off and columns=1.

---

## 11. Part I — Finished View

| Decision | Detail |
| --- | --- |
| Scope | **Bounded multi-grid front elevation** in the same phase is acceptable |
| Data | Semantic model only (`dimensions`, cell origins, rows/columns) — **not** production SVG |
| Content | Grid of drawer fronts; horizontal shelf bands; vertical partition bands; optional note when thicknesses differ |
| Dual downloads | Finished View still one assembly preview; production remains shell/interior files when separate |
| Defer color-coding shell vs interior | Optional cosmetic |

Finished View must remain production-independent (existing download fixtures already enforce this pattern).

---

## 12. Part J — Normalization and validation

| Field | Rule |
| --- | --- |
| `rows` | integer 1–3 (existing) |
| `columns` | integer 1–3; absent → 1 |
| `shellThickness` | from `materialThickness`; positive finite ≥ 0.1 |
| `useSeparateInteriorThickness` | checkbox triad only |
| `interiorThickness` | equals shell when linked; else validated like material thickness |
| Drawer / cell / cabinet dims | existing positivity and clearance checks, using role thicknesses |
| Min openings | existing drawer finger-pattern validity (one drawer covers all equal cells) |
| Sheet bounds | per production output when dual; single layout when linked |
| Storage | **no** migration, schema, import/export, evidence, promotion, Production Settings schema changes |

Production Settings remain allowed to write `materialThickness` only; linking keeps interior in sync.

---

## 13. Part K — Fixture plan (corrected)

Start from **922 Designs / 1714 complete**. Add representative (not Cartesian) asserts:

1. Missing new fields → current output (linked, columns=1).  
2. Separate mode absent/false → `interiorThickness === shellThickness`.  
3. Change shell while linked → both roles change; single SVG still.  
4. Enable separate → explicit interior value used.  
5. Malformed checkbox/interior → disabled separate or validation errors per contract.  
6. Legacy 1/2/3-row goldens byte-identical at linked defaults.  
7. `columns=1` IDs remain `drawer-r0N-*`.  
8. Multi-column IDs deterministic, collision-free (`drawer-r0N-c0M-*`).  
9. Shelves = `rows−1`.  
10. Partitions = `(columns−1)×rows`.  
11. 2×3 → 6 drawers; 3×3 → 9 drawers.  
12. Shell panels use shellThickness in finger inputs/outside dims.  
13. Drawer/shelf/partition use interiorThickness.  
14. No unequal-thickness finger pair.  
15. Hand-derived cell/origin for one multi-column case.  
16. Linked mode: single layout finite/non-overlapping.  
17. Separate mode: **two** layouts finite/non-overlapping; each output sheet bounds correct.  
18. Linked: one download = `result.svg`.  
19. Separate: two downloads; each preview/download identity; filenames/MIME/thickness labels.  
20. Other templates still single-`result.svg` only.  
21. Shelf guides unchanged at legacy inputs.  
22. No partition score geometry in phase one; multi-column warning present.  
23. Red paths independent of score.  
24. Finished View multi-grid semantic + production independence.  
25. localStorage / backupObject unchanged by generation.  
26. Unrelated templates unchanged.  
27. New multi-grid / dual-thickness goldens only after structural asserts.  
28. Suite totals reconcile from 922 / 1714.

---

## 14. Protected boundaries

No changes required or allowed for: `STORAGE_KEY`, `SCHEMA_VERSION`, persist/backup/import/export, evidence/promotion, Library/Inventory/Projects/Pricing, Test Grid, Material Test, Finger Box / Sliding-Lid / Trays / Joint Fit Coupon production paths, generic multi-document framework for all templates.

Drawer Cabinet may add **template-local** dual download UI and `productionOutputs` only when separate thickness is enabled.

---

## 15. Physical limitations

Software cannot prove: glue strength, partition squareness, multi-bay racking, dual-stock kerf differences, or LightBurn operator discipline. First real multi-grid build should verify fit of every bay, shelf/partition seating, and (if used) dual-file cut workflow.

---

## 16. Findings by severity

| ID | Severity | Finding | Consequence | Required correction |
| --- | --- | --- | --- | --- |
| F1 | **Major** | Two independent thickness fields defaulting to 3 mm silently create mixed thickness when user only edits shell/material thickness | Accidental dual-stock geometry; production-settings apply only one field | **OPTION 2** checkbox linking contract |
| F2 | **Major** | One combined red-cut SVG for two stocks is not a safe production workflow (single `#cut`, single download, titles ≠ LightBurn layers, dishonest combined sheet) | Wrong-thickness cuts; operator error | **OUTPUT D**: dual production SVGs when separate mode on; single SVG when linked |
| F3 | **Major** (if ignored) | Legacy IDs/order must be conditional at `columns=1` | Golden/fixture and saved-name breakage | Explicit conditional ID + layout superset |
| F4 | **Minor** | Partition guides deferred | Manual placement harder for multi-column | Honest warnings; guides as follow-up |
| F5 | **Minor** | Fixture surface large (Phase 5.1 scale) | Review load | Representative plan in §13 |
| F6 | **Informational** | Geometry role split and grid formulas are sound | — | Proceed after F1–F3 fixed in design |
| F7 | **Informational** | Physical glue-up / squareness unverified | — | First real build |

No Blocker-level storage or direct-open risk in the **geometry** proposal itself. The unsafe path is **shipping dual-thickness with a single undifferentiated cut file and unlinked thickness fields**.

---

## 17. Agreement with primary review (what stands)

- Dual-thickness role split without unequal finger joints.  
- Equal-cell multi-grid formulas and `columns=1` collapse.  
- Glued shelves + row-height partitions (Option A).  
- Session-only fields; no schema migration.  
- Conditional legacy drawer IDs.  
- Single source of truth for dimensions/origins.  
- Finished View as semantic multi-grid extension.  

---

## 18. Required design deltas before coding

1. Replace independent dual thickness fields with **linked-by-default separate-thickness checkbox**.  
2. Replace “one combined production SVG always” with **OUTPUT D** (single file linked; two files when separate).  
3. Define template-local dual download/preview UI and per-sheet layout for separate mode.  
4. Defer partition score guides; keep shelf guides legacy-safe.  
5. Keep conditional IDs and byte-identity fixtures as hard gates.  

Until those deltas are written into the design (and this challenge’s contracts are adopted), implementation should not start.

---

## Final verdict

### NEEDS DESIGN CORRECTION
