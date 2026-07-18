# Cross-Template Assembly Label Focused Audit

**Date:** 2026-07-18  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit type:** Independent, adversarial, read-only — assembly labels and Drawer Cabinet custom-row target-ID correction  
**Expected baseline:** `d838041` — Add faux dovetail engraving to Finger Box  
**Feature under audit:** Drawer Cabinet custom-row assembly-label target-ID correction  

This audit does not trust the implementation report alone. Findings come from committed baseline source, working-tree source, complete diff enumeration, independent prefix/target derivation, committed-baseline bug reproduction, and direct `file://` headless Microsoft Edge suite runs.

---

## 1. Repository and synchronization state

| Check | Result |
| --- | --- |
| `git status -sb` | `## main...origin/main`; tracked `M README.md`, `M index.html` |
| `git log -1 --oneline` | `d838041 Add faux dovetail engraving to Finger Box` |
| `git rev-parse HEAD` | `d8380414af75cd8c9307fb7c6307751078bbb2e1` |
| Matches expected `d838041` | **Yes** |
| Branch | `main` |
| `HEAD...origin/main` | `0 0` (fully synchronized) |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --stat` | `README.md 2 ±`; `index.html 31 ±`; **2 files, 30 insertions, 3 deletions** |
| `git diff --numstat` | `1 1 README.md`; `29 2 index.html` — reconciles with `--stat` |
| Staged | **Empty** |
| Tracked modified only | **`index.html`, `README.md` only** |
| Read-only audit | No application files edited; only this audit report created under `docs/`; no stage/commit/push/reset/clean/stash |

### Expected untracked materials

| Path | Status |
| --- | --- |
| `docs/DESIGNS_DRAWER_CABINET_CUSTOM_ROW_ASSEMBLY_LABEL_FIX_2026-07-18.md` | Present, untracked |
| `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md` | Present, untracked, **not modified by this audit** (`git status` shows untracked only; no working-tree edit) |

### Untouched out-of-scope materials

`LightBurn Projects/*`, `debug.log`, `parametric_qr_stand_generator.py`, historical `docs/*` reports, and any `.claude/` content were not opened for edit and were not moved/renamed/deleted.

---

## 2. Complete working-tree diff-hunk enumeration

Source: `git diff` vs `HEAD` / `d838041`.

### `index.html`

| Hunk | Classification | Assessment |
| --- | --- | --- |
| `@@ -2732,8 +2732,8 @@` inside `designAssemblyLabelSpecs()` cabinet branch | **Drawer Cabinet assembly-label specification correction** | Replaces bare `drawer-rNN` loop with nested row×column loop calling `drawerCabinetDrawerPrefix(row, column, values.maximumColumns)`. **Sole production correction.** |
| `@@ -4367,6 +4367,33 @@` Designs fixtures | **regression fixtures** | Adds **20** focused assertions covering prefixes, model membership, screenshot case, separate outputs, containment omission, goldens, storage, cut identity. |

No other `index.html` hunks.

### `README.md`

| Hunk | Classification | Assessment |
| --- | --- | --- |
| Built-in checks totals | **README** / **fixture totals** | `1806` complete; Designs `1014`; Tray `264`; arithmetic `1786 + 20 = 1806`. Matches runtime. |

### Unexpected / unrelated

**None.** No edits to panel generation, panel IDs, geometry, `serializeDesignSvg`, layout helpers, storage, Production Settings, Finger Box / Sliding-Lid / tray / coupon builders, or `drawerCabinetDrawerPrefix` itself.

---

## 3. Files and functions inspected

- Working-tree and `git show d838041:` for `index.html`, `README.md`
- Implementation report; Concealed Cleat review presence only (untouched)
- Prior context: assembly-label architecture, Finger Box faux-dovetail, Sliding-Lid guides, Drawer Cabinet custom-row / shelf-guide contracts

**Functions (working-tree line anchors are approximate):**

- `designDefaults()` — session `assemblyLabels: false`
- Template UI: Finger Box / Sliding-Lid / Drawer Cabinet assembly-label checkboxes; coupons use separate `jointCouponLabels`
- `normalizeDesignDraft()` — `assemblyLabels`, cabinet `rowDrawerCounts` / `maximumColumns`
- `drawerCabinetDrawerPrefix()` — authoritative prefix policy
- `buildDrawerCabinetModel()` — drawer ID emission via prefix helper
- `designAssemblyLabelSpecs()` — **corrected** cabinet loop
- `designAssemblyLabelGeometry()`, `buildAssemblyLabelPaths()` — containment, clearance, missing-panel warning, `label-${panelId}-${index}` IDs
- `designSegmentsContainedInPolygon()`, `designMinimumSegmentClearance()`
- `namespaceDesignPanel()`, `serializeDesignSvg()`
- `buildBoxDesignResult()`, `buildSlidingLidDesignResult()`, `buildDrawerCabinetDesignResult()` (linked + separate membership filter)
- Tray result path (no `designAssemblyLabelSpecs` for trays)
- Coupon path (`jointCouponLabels` — separate from assembly-label specs)
- Preview/download fixtures (`designSvg`, `trayLivePreviewDownloadFixture` patterns)
- Metrics/warnings in `designResultsHtml()`
- Designs fixtures (existing labels + 20 new); complete-suite dispatcher

---

## 4. Baseline runtime reproduction (`d838041`)

Method: temp file of `git show d838041:index.html`; isolated headless Edge `channel=msedge`; direct `file://…?selftest=all`; established group aggregation (Material browser final cumulative pair only).

| Group | Passed | Failed |
| --- | --- | --- |
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Production settings | 66 | 0 |
| Evidence promotion | 58 | 0 |
| Design production settings | 118 | 0 |
| Grid promotion | 23 | 0 |
| Test Grid machine identity | 18 | 0 |
| Grid browser | 67 | 0 |
| Material browser (final) | 57 | 0 |
| Library browser | 56 | 0 |
| Project browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Design geometry | **994** | 0 |
| **Complete suite** | **1786** | **0** |

- `readyState: complete`; no page exceptions.
- All 15 groups connected; none skipped or double-counted.
- Matches expected baseline: Tray **264** (nested + separately callable), Designs **994**, complete **1786**.

### Independent baseline bug reproduction (screenshot layout)

Draft: custom top 2 / bottom 1, labels on.

| Field | Baseline `d838041` result |
| --- | --- |
| `rowDrawerCounts` | `[1, 2]` |
| `maximumColumns` | `2` |
| Model drawer IDs | `drawer-r01-c01-*`, `drawer-r02-c01-*`, `drawer-r02-c02-*` |
| Label specs (drawer) | bare `drawer-r01-*`, `drawer-r02-*` only |
| Missing targets | **all 10 bare drawer specs** miss the model |
| Missing-panel warnings | **10** (`missing panel drawer-…`) |
| Emitted labels | **5** (shell only) |
| Metric count | **5** |

Root cause confirmed: specs used bare row prefixes; model always used `cNN` when `maximumColumns > 1`.

---

## 5. Working-tree runtime reproduction

Same method on product-tree `index.html`.

| Group | Passed | Failed | vs baseline |
| --- | --- | --- | --- |
| (all non-Designs groups) | same as baseline | 0 | unchanged |
| Design geometry | **1014** | 0 | **+20** |
| **Complete suite** | **1806** | 0 | **+20** |

### Arithmetic

```text
20 + 12 + 66 + 58 + 118 + 23 + 18 + 67 + 57 + 56 + 61 + 12 + 8 + 216 + 1014 = 1806
```

Verified:

- Only Designs grew (+20).
- Tray-model alone: **264 / 0**.
- No runtime exceptions; no group removal.
- Source, README, and implementation report totals reconcile: **264 / 1014 / 1806**.
- `python -m html.parser index.html` passed.

---

## 6. Assembly-label applicability matrix

| Template | Assembly labels | UI | Other blue score | Production files | Notes |
| --- | --- | --- | --- | --- | --- |
| Finger Box | **Optional** supported | checkbox | Faux dovetail engraving | One SVG | Specs: `bottom/front/back/left/right` (+ `lid` if loose) |
| Sliding-Lid Box | **Optional** supported | checkbox | Rail-placement guides | One SVG | Specs use `front-open`, `sliding-lid`; Right panel mirrored when labels or guides on |
| Dice Tray | **Unsupported** | no checkbox | (tray score N/A for assembly labels) | One SVG | `designAssemblyLabelSpecs` returns `[]`; forcing flag does not change SVG |
| Divider Tray | **Unsupported** | no checkbox | — | One SVG | Same as Dice Tray |
| Drawer Cabinet | **Optional** supported | checkbox | Shelf-placement guides | One or two (separate thickness) | Specs: 5 shell + 5×drawer via prefix helper |
| Joint Fit Coupon | **Separate system** | `jointCouponLabels` | Clearance labels (not `designAssemblyLabelSpecs`) | One SVG | Not assembly-label specs; default coupons emit coupon score paths |
| Wall-to-base coupon | Coupon mode | coupon labels path | Coupon-specific | One SVG | Outside shared assembly-label template matrix |

Shared helpers exist for all templates, but **only Finger Box, Sliding-Lid Box, and Drawer Cabinet** expose and generate the shared assembly-label contract via `designAssemblyLabelSpecs`.

---

## 7. Root-cause verification and correction

### Former (baseline) cabinet loop

```javascript
for (let row=1; row<=values.rows; row++) {
  const prefix = `drawer-r${String(row).padStart(2,'0')}`;
  // five faces per row only
}
```

### Model ownership

`buildDrawerCabinetModel` names drawers with `drawerCabinetDrawerPrefix(row, column, maximumColumns)`:

- `maximumColumns === 1` → `drawer-rNN`
- `maximumColumns > 1` → `drawer-rNN-cNN` for **every** drawer, including one-drawer rows

### Correction (working tree)

```javascript
for (let row=1; row<=values.rows; row++)
  for (let column=1; column<=values.rowDrawerCounts[row-1]; column++) {
    const prefix = drawerCabinetDrawerPrefix(row, column, values.maximumColumns);
    // five faces per actual drawer
  }
```

Verified:

- Iterates every drawer in `rowDrawerCounts`
- Calls `drawerCabinetDrawerPrefix` only — **no duplicated prefix rule**
- Prefix helper remains sole policy owner
- **No** global suppression of missing-panel warnings (`buildAssemblyLabelPaths` still emits `Assembly label … omitted: missing panel …` for genuine missing IDs — probe confirmed with `does-not-exist`)

---

## 8. Global label-target contract

For each **supported** template, independent probes:

| Template | Every target exists in panels? | Deterministic IDs? | False missing-panel warnings? |
| --- | --- | --- | --- |
| Finger Box | Yes (5 open / 6 loose) | `label-<panelId>-<index>` | No on valid defaults |
| Sliding-Lid | Yes (6 body/lid; coupons excluded) | Yes | No |
| Drawer Cabinet (all 9 cases below) | Yes after fix | Yes; repeated FRONT/BOTTOM unique by panel id | No after fix |

Trays: zero specs — no false targets. Coupons: outside assembly-label specs.

---

## 9. Finger Box (labels ± faux dovetail)

| Check | Labels only | Labels + faux dovetail |
| --- | --- | --- |
| Label targets exist | 5 | 5 |
| Score paths (decoration) | 0 | 20 |
| Both serialize | labels in `assembly-labels` | `corner-decoration` then `assembly-labels` under one `score` |
| Unique IDs | Yes | Yes |
| Score before cut | Yes | Yes |
| Red cut vs structural peer | Equal to undecorated/unlabeled structural cut as applicable | Cut equal to decoration-only; labels-only cut equal to plain open box |
| Shared-face warning | N/A | Present |
| Open golden | **2483 / `a892f91c`** unchanged | — |
| Faux golden | — | **7269 / `d9512d1c`** unchanged |

**Drawer Cabinet fix does not alter Finger Box production bytes.** Hierarchy:

```text
score → corner-decoration → marks; assembly-labels → labels; cut → panels
```

---

## 10. Sliding-Lid Box

| Check | Result |
| --- | --- |
| Label targets | `bottom`, `front-open`, `back`, `left`, `right`, `sliding-lid` — all exist |
| Labels + rail guides | Coexist: `score` children `rail-guides`, `assembly-labels` |
| Unique IDs | Yes |
| Score before cut | Yes |
| Missing-panel warnings | None on valid defaults |
| Label-disabled golden | Unchanged suite pin (**2800 / `4a7ab718`** in probe for default sliding A-style draft) |
| Coupon pieces unlabeled when labels on | Existing fixtures still pass |

**Cut-byte note (established, not introduced by this fix):** enabling assembly labels (or guides) sets `rightPanelMustBeHanded` and **mirrors the Right panel**, so label-on vs label-off cut markup is **intentionally not equal**. Label-**disabled** bytes remain the production golden. This is not a regression of the Drawer Cabinet correction.

---

## 11. Dice Tray and Divider Tray

- No assembly-label UI; `designAssemblyLabelSpecs` returns `[]`.
- Forcing `assemblyLabels: true` leaves SVG byte-identical to off.
- Drawer Cabinet correction has **no effect** on tray output.
- Finished View independence unchanged (no tray label path).

---

## 12. Drawer Cabinet prefix policy

Authoritative helper (unchanged):

| `maximumColumns` | Prefix form |
| --- | --- |
| `1` | `drawer-rNN` |
| `> 1` | `drawer-rNN-cNN` for every drawer |

### Independent case matrix

| Case | `rowDrawerCounts` | `maxCols` | Drawer prefixes | Specs all in model? | Labels (linked) | Missing-panel warn |
| --- | --- | --- | --- | --- | --- | --- |
| Uniform 1×1 | `[1]` | 1 | `drawer-r01` | Yes | 10 | 0 |
| Uniform 1×3 | `[1,1,1]` | 1 | bare r01–r03 | Yes | 20 | 0 |
| Uniform 2×3 | `[2,2,2]` | 2 | all `c01`/`c02` | Yes | 35 | 0 |
| Custom top 2 / bottom 1 | `[1,2]` | 2 | r01-c01; r02-c01; r02-c02 | Yes | **20** | 0 |
| Custom top 1 / bottom 2 | `[2,1]` | 2 | r01-c01/c02; r02-c01 | Yes | 20 | 0 |
| Custom top 1 / bottom 3 | `[3,1]` | 3 | cNN throughout | Yes | 25 | 0 |
| Custom 2/1/3 | mixed | 3 | cNN throughout | Yes | 35 | 0 |
| Custom all-one 1/1/1 | `[1,1,1]` | 1 | bare r01–r03 **no c01** | Yes | 20 | 0 |
| Separate custom 1/2/3 | 6 drawers | 3 | cNN | Yes | shell **5** + interior **30** | 0 cross-output |

Old bare-spec simulator still **would** miss all multi-column and mixed custom cases — correction is necessary and sufficient.

---

## 13. Screenshot regression case (top 2 / bottom 1)

| Expectation | Actual (working tree) |
| --- | --- |
| `rowDrawerCounts = [1, 2]` | Yes |
| `maximumColumns = 2` | Yes |
| Prefixes | `drawer-r01-c01-*`, `drawer-r02-c01-*`, `drawer-r02-c02-*` |
| Drawer targets | **15** |
| Shell targets | **5** |
| Safe labels | **20** (all fit) |
| Metric HTML | `Enabled - 20 safe labels` |
| Missing `drawer-` panel warnings | **None** |
| Red cut vs labels-off | **Byte-identical** |
| Score before cut | Yes |

Baseline behavior was 5 shell labels + 10 false missing-panel warnings — fixed.

---

## 14. All-one legacy and uniform multi-column

- Custom **1/1/1**: `maximumColumns = 1`; labels use bare `drawer-r0N-*`; **no `c01` token**; cut/layout identity vs labels-off holds.
- Uniform **2×3**: all drawers `cNN`; 35 labels; no false warnings; deterministic; cut-byte identity vs labels-off holds.

---

## 15. Linked vs separate Drawer Cabinet output

### Linked

- One score group with shell + drawer labels; score before cut.
- Metric count = emitted paths = `5 + 5 × drawerCount` when all fit.
- Repeated words (FRONT, BOTTOM, …) unique via `label-<panelId>-<index>`.
- No production geometry change (panel paths/IDs/layout equal labels-off).

### Separate-thickness custom 1/2/3

| Output | Labels | Membership |
| --- | --- | --- |
| Shell | **5** shell only | All panelIds in shell output |
| Interior | **30** drawer only | All panelIds in interior output |
| Cross-output missing warnings | **None** (membership filter before `buildAssemblyLabelPaths`) | Intentional, not suppression of real missing IDs |
| Generic mixed download | Remains suppressed for multi-output | Unchanged |

Filenames/MIME/production dual-output contract unchanged by this fix.

---

## 16. ID uniqueness and SVG structure

- Label IDs: `label-${spec.panelId}-${index}` — namespaced by panel identity, not visible word.
- Multi-drawer FRONT count equals drawer count; full ID set unique in all probed cases.
- Representative hierarchies:

**Finger Box + faux + labels:**  
`score` → `corner-decoration`, `assembly-labels` → `cut`

**Sliding + guides + labels:**  
`score` → `rail-guides`, `assembly-labels` → `cut`

**Drawer Cabinet linked labels:**  
`score` → `assembly-labels` (+ optional `shelf-guides`) → `cut`

**Separate shell / interior:** each SVG has its own score/cut pair; labels only for panels in that file.

Blue score / red cut separation preserved; markup valid under suite + `designSvgValidation`.

---

## 17. Containment omission and missing-panel mechanism

| Test | Result |
| --- | --- |
| Compact cabinet (`drawerInsideWidth: 12`) + labels | Valid; **7** labels; geometric omit warnings; **no** `missing panel drawer-` |
| Direct `buildAssemblyLabelPaths` with nonexistent target | Warning: `Assembly label FRONT omitted: missing panel does-not-exist.` — mechanism **active** |

Correction fixed ID generation; it did **not** mute warnings.

---

## 18. Cut-byte identity (critical gate)

| Template | Labels-off vs labels-on structural cut |
| --- | --- |
| Finger Box | **Equal** (paths, layout, full `g#cut` markup) |
| Drawer Cabinet linked cases | **Equal** `g#cut`; panel IDs/paths/x/y equal |
| Drawer Cabinet separate | Model IDs/paths unchanged; only blue labels added per output |
| Sliding-Lid | **Not equal when labels on** — intentional Right mirror (pre-existing); labels-off remains golden |

Drawer Cabinet correction: only newly resolved blue label paths differ from broken baseline behavior; model geometry unchanged.

---

## 19. Production goldens (label-disabled pins)

Verified present and unchanged (suite + probe):

| Golden | Length / hash |
| --- | --- |
| Finger Box open | 2483 / `a892f91c` |
| Finger Box faux dovetail | 7269 / `d9512d1c` |
| Drawer Cabinet default 1-row | 4456 / `a6dd23dc` |
| Custom linked 1/3 | 13606 / `500396df` |
| Separate custom interior 1/2/3 | 14777 / `69b0ce8b` |
| Sliding / tray / coupon suite goldens | Suite green (994→1014 only from new label fixtures) |

**Current production goldens disable assembly labels** (or pin label-off bytes). Enabling labels is additive score content for Finger Box and Drawer Cabinet; Sliding also mirrors Right when labels are on.

No hash updates performed by this audit.

---

## 20. Preview/download identity

- Existing fixtures: Finger Box labels preview/download; Sliding guides+labels; decoration live helper; Drawer Cabinet live patterns.
- Suite green implies `designSvg(...)` / live preview helpers remain aligned with `result.svg`.
- Disabling labels restores label-free production bytes (Finger Box / Sliding / Cabinet fixtures).
- Separate dual-output UI still uses per-output downloads (unchanged contract).

---

## 21. Metrics and warnings

| Template | Metric wording | Count integrity |
| --- | --- | --- |
| Drawer Cabinet | `Enabled - N safe labels` | N = emitted paths on linked cases; screenshot **20** |
| Finger / Sliding | `Enabled — N intended-face labels` | Matches `labelPaths.length` |
| Omissions | Geometric wording | Compact case uses omit, not missing-panel |
| Legitimate cabinet warnings | Shelf glue, custom partitions, dry-fit, sheet size | Still present (suite) |

No stale cross-template counts observed.

---

## 22. Storage and Production Settings

| Boundary | Status |
| --- | --- |
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged |
| `freshState` / load / persist / backup / import-export | Unchanged |
| localStorage / backup around Finger labels, faux+labels, linked cabinet labels, separate labels | **Byte-identical** |
| Production Settings schema / applicability | Unchanged |
| Assembly labels as Production Setting | **Not** introduced |
| Linked thickness / separate shell vs interior thickness apply rules | Unchanged |
| Finger/faux/Sliding/tray/coupon geometry | Unchanged |

---

## 23. Fixture-quality assessment (20 new assertions)

| Classification | Coverage |
| --- | --- |
| **Independent** | Normalize drawer counts; bare one-column prefixes; multi-column `cNN`; mixed custom `c01`; screenshot full ID list; custom 1/3 model membership; all-one bare; every target in model; no false missing-panel; unique IDs + FRONT count; shell words unchanged; metric = emitted; score-before-cut + cut-byte identity; panel path/layout identity; separate shell/interior membership; compact geometric omit; label-disabled goldens; storage isolation |
| **Partially circular** | None material — cases build model and specs independently then compare |
| **Circular** | None |
| **Irrelevant** | None |

No fixture name overstates a weak boolean: cut identity, membership, and missing-panel absence are separate assertions.

---

## 24. Findings table

| ID | Severity | Location | Consequence | Recommendation | Fixture coverage |
| --- | --- | --- | --- | --- | --- |
| — | — | — | No Blocker or Major defects in the working-tree fix | Safe to commit as-is | Full suite 1806/0 |
| F1 | Informational | Sliding-Lid `rightPanelMustBeHanded` | Labels-on intentionally changes Right cut path (mirroring) | Document for operators; not a defect of this fix | Existing Sliding label fixtures |
| F2 | Informational | Physical engraving | Label legibility on wood unverified | Scrap check | N/A |
| F3 | Informational | Concealed Cleat review | Deferred structural work; review untouched | Out of scope | N/A |

No Minor corrections required for commit safety.

---

## 25. Unverified physical areas

This audit does **not** claim:

- Physical engraving readability or contrast
- Assembled label orientation on finished cabinets/boxes
- LightBurn import
- Machine cutting, fit, or strength
- Concealed Cleat prototype implementation

---

## 26. Final conclusion

The production change is a minimal, correct fix to `designAssemblyLabelSpecs()` so Drawer Cabinet label targets use `drawerCabinetDrawerPrefix` and actual `rowDrawerCounts`. The baseline bug (bare `drawer-rNN` vs model `drawer-rNN-cNN` when `maximumColumns > 1`) was independently reproduced and is resolved without muting missing-panel warnings, without geometry/serializer/storage drift, and without regressing Finger Box, Sliding-Lid, tray, or coupon contracts. Complete suite **1806 / 0** (Designs **+20** only) matches claimed arithmetic.

**SAFE TO COMMIT**
