# Designs Dice Tray DT1 — Correction Re-Verification (2026-07-20)

**Scope:** Narrow read-only re-verification that focused-audit corrections are complete and did not alter protected legacy geometry, storage, schemas, or existing Designs behavior.

**Not in scope:** Full DT1 architecture re-audit (unless new high-impact shared-helper or production-output issues appear).

**Method:** Independent inspection of uncommitted working tree against committed baseline `294cc06`; direct runtime probes of model layout coordinates, validation thresholds, assembly-label gating, and Finished View geometry; headless Edge `file://` fixture re-runs; documentation and protected-boundary checks.

**Final verdict:** **APPROVED TO COMMIT**

| Severity | Count |
|----------|------:|
| BLOCKER | 0 |
| IMPORTANT | 0 |
| POLISH | 0 |

---

## 1. Repository state

Commands run (read-only):

```text
git status -sb
git log -1 --oneline
git rev-parse HEAD
git rev-list --left-right --count origin/main...main
git diff --check
git diff --stat
git diff --cached
git ls-files --others --exclude-standard
```

| Check | Result |
|-------|--------|
| Branch | `main` tracking `origin/main` |
| HEAD | `294cc064f82020ce02ca95e2ced997f003715ba9` |
| `git log -1` | `294cc06 Add parametric Gift Box generator` |
| Ahead/behind `origin/main...main` | `0 0` |
| Staged changes | None (`git diff --cached` empty) |
| `git diff --check` | Clean (CRLF warnings only; no conflict markers / whitespace errors) |
| DT1 commit or push | **None** — work remains uncommitted on `main` |

**Tracked modifications (intended DT1 + docs surface only):**

| File | Diff stat |
|------|-----------|
| `index.html` | ~244 lines changed (net +235 / −17 across the three tracked files) |
| `README.md` | suite totals / DT1 notes |
| `CHANGELOG.md` | one DT1 bullet |

**Untracked (historical / local; not DT1 product code):** architecture/implementation/audit/correction reports under `docs/`, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, plus this verification report when written.

**Confirmed present:**

- `docs/DESIGNS_DICE_TRAY_SYSTEM_ARCHITECTURE_REVIEW_2026-07-20.md`
- `docs/DESIGNS_DICE_TRAY_DT1_IMPLEMENTATION_2026-07-20.md`
- `docs/DESIGNS_DICE_TRAY_DT1_FOCUSED_AUDIT_2026-07-20.md`
- `docs/DESIGNS_DICE_TRAY_DT1_CORRECTION_2026-07-20.md`

Historical untracked LightBurn projects, `debug.log`, and utility scripts were not modified by this verification.

---

## 2. Files and functions reviewed

| Area | Symbols / locations |
|------|---------------------|
| Form controls | Dice Tray DT1 storage/insert/cover fields; `diceStorageWidth` `min="20"`; help “below 20 mm is blocked” |
| Normalization / validation | `normalizeDesignDraft()` Dice Tray branch; storage width `< 20` block; insert thickness `>= wall height` block |
| Model / layout | `buildTrayModel()`; `dt1PhysicalExtra`; `dt1PanelBottomMm`; underside cover Y; `materialComponentIds` |
| Labels | `trayAssemblyLabelResult()`; `serializeTrayCompatibilitySvg()` assembly-label group (`#00ff00`) |
| Finished View | `trayFinishedViewProjection()`; `buildDiceTrayFinishedViewSvg()` |
| Fixtures / routes | `runDiceTraySystemFixtures()` (92 `add(`); `runTrayModelFixtures()`; `if (selftest === 'dice-tray-system'` **exactly once** under `all`) |
| Docs | README suite line; implementation report thresholds and correction notes; CHANGELOG DT1 bullet |

---

## 3. Storage-threshold verification

Independent `buildDesignResult` cases (tray 150×150×35, storage right, fixed divider):

| Width (mm) | Valid | Blocked | Narrow-retrieval warning |
|-----------:|:-----:|:-------:|:------------------------:|
| 17.9 | no | yes — “at least 20 mm” | no |
| 18.0 | no | yes | no |
| 19.0 | no | yes | no |
| 19.99 | no | yes | no |
| 20.0 | yes | no | **yes** |
| 27.9 | yes | no | **yes** |
| 28.0 | yes | no | **no** |

**Contract surfaces:**

| Surface | Value |
|---------|-------|
| Form `min` | `20` (`diceStorageWidth`) |
| Model validation | `storageWidth < 20` → error |
| Help copy | “below 20 mm is blocked” / retrieval-warning territory below 28 mm |
| Default enabled storage width | **42 mm** |
| Fixtures encoding 18 as **valid** | **No** — `diceStorageWidth:'18'` appears only as **blocked** case (`18.0 mm storage is blocked`) |
| Fixtures encode 17.9 / 19.99 | Yes (blocked boundaries) |

---

## 4. Insert/cover non-overlap evidence

Physical panel boxes from `model.layout.materialComponentIds` + component geometry (not source-string checks).

### Prior blocker: wood insert + underside cover

| Panel | x | y | w | h | maxX | maxY |
|-------|--:|--:|--:|--:|-----:|-----:|
| `rolling-insert-panel` | 10 | 297 | 149.2 | 149.2 | 159.2 | 446.2 |
| `tray-bottom-cover` | 10 | 461.2 | 156 | 156 | 166 | 617.2 |

- **AABB overlap:** none  
- **Vertical gap:** `461.2 − 446.2 = 15` mm = `layout.gapMm`  
- Cover sits **after** the lowest occupied physical DT1 strip (insert bottom 446.2)

### Other layout cases

| Case | Overlaps | min pair gap | Notes |
|------|:--------:|-------------:|-------|
| wood + cover | none | 15 | coordinates above |
| storage-right + wood + cover | none | 15 | divider + insert strip, cover below |
| storage-left + wood + cover | none | 15 | same physical strip pattern |
| storage divider + cover (no wood) | none | 15 | cover Y=350 after divider maxY=335 |
| wood insert, no cover | none | 15 | insert only extra strip |
| cover, no insert | none | 15 | cover after base layout |
| cork + cover | none | 15 | **no** insert panel in inventory |
| leather + cover | none | 15 | measurement-only |
| felt + cover | none | 15 | measurement-only |
| generic-liner + cover | none | — | measurement-only; no empty insert strip |

**Measurement-only liners:** `materialComponentIds` for cork+cover =  
`tray-base`, walls, `tray-bottom-cover` only — **no** `rolling-insert-panel`. Empty physical strip is **not** reserved.

**Cut path quality (wood+storage+cover representative):** unique cut item SVGs; closed paths (`Z"/>` terminator); deterministic repeat (`samePanels` / `sameIds` true across two builds).

**Silent scaling:** not observed; panel sizes match clear dimensions (e.g. insert 149.2 = 150 − 2×0.4 clearance).

---

## 5. Insert-height verification

Wall height **35 mm**.

| Material / mode | t=34.99 | t=35 | t=40 |
|-----------------|:-------:|:----:|:----:|
| cork | valid + shallow warn; usable ≈ 0.01 mm | **blocked** | **blocked** |
| leather | valid + shallow; usable > 0 | blocked | blocked |
| felt | valid + shallow; usable > 0 | blocked | blocked |
| generic-liner | valid + shallow; usable > 0 | blocked | blocked |
| wood measurement-only | valid + shallow; usable > 0 | blocked | blocked |
| wood panel (`diceWoodInsertPanel`) | blocked first by **thickness must match material** (3 mm) when t≠3; height rule still present in validator | blocked (match and/or height) | blocked |

**Disabled insert, stale thickness 40 mm:** valid; no insert-thickness error; legacy/no-insert behavior.

**Usable wall height on valid shallow cases:** positive (never zero or negative). Equal/above wall height never produce valid output.

---

## 6. Assembly-label verification

DT1 features enabled (storage + wood insert panel + cover). Label stroke color in production SVG: **`#00ff00`** (via `serializeConcealedCleatPathGroup(..., 'assembly-labels', '#00ff00')`).

### Assembly Labels **disabled**

| Check | Result |
|-------|--------|
| `assembly-labels` group | absent |
| Green label paths | none |
| BASE / FRONT / BACK / LEFT / RIGHT | absent |
| STORAGE-DIVIDER / INSERT / BOTTOM-COVER | absent |
| Missing-label warnings | none |
| `metrics.labelCount` | 0 |

### Assembly Labels **enabled** (full DT1)

| Check | Result |
|-------|--------|
| Group present | yes |
| BASE, FRONT, BACK, LEFT, RIGHT | present |
| STORAGE-DIVIDER | present when storage exists |
| INSERT | present only with generated wood insert panel |
| BOTTOM-COVER | present when cover exists |
| Non-wood liner (cork) | **no** INSERT label |
| Path count vs `labelCount` | 8 = 8 (parity) |
| Green outside red cut group | labels appended as separate `assembly-labels` group; not inside cut paths |

**Cover-only (no storage, no insert):** not a DT1 metrics case (`dt1` requires storage or insert type). Labels correctly omitted — **NOT A DEFECT**.

**Global safe-label algorithm for other templates:** unchanged path (`buildAssemblyLabelPaths`); DT1 only gates via `trayAssemblyLabelResult` when `diceTrayDt1` and `assemblyLabels` are true. Complete Designs geometry suite still **1093 / 0**.

---

## 7. Fixture assertion inventory

`runDiceTraySystemFixtures()` body: **92** `add(` calls.

Independent runtime: **`passed: 92`, `failed: 0`, `total: 92`**.

Direct behavioral coverage verified (fixture names + independent probes):

| Theme | Covered |
|-------|:-------:|
| Legacy Dice Tray length 1726 | yes |
| Legacy FNV `51a55721` | yes |
| Alternate-joint pin 1054 / `41697123` | yes |
| Divider Tray 1965 / `a55dda6e` | yes |
| 19.99 blocked / 20 valid+warn / 27.9 warn / 28 no warn | yes (+ 17.9–19) |
| Wood+cover non-overlap; L/R storage+wood+cover | yes (coordinates) |
| Physical panel gap | yes (15 mm) |
| Unique closed cut paths | yes |
| Insert below / equal / above wall height | yes |
| Stale disabled insert thickness | yes |
| Label toggle + conditional divider/insert/cover | yes |
| Label-count parity | yes |
| Finished View finite coords / positive rects | yes |
| Storage L/R mirror; insert in rolling; no storage overlap | yes |
| No lid; FV IDs absent from production SVG | yes |
| Exact-one `selftest=dice-tray-system` under all | yes |

---

## 8. Fixture cleanup

- Hard minimum **20 mm** is what fixtures assert; **18 mm is only used as a blocked case** (correct).
- Focused suite grew from prior **51** to **92** without leaving the old overlap gap unasserted.
- No failing or contradictory fixture names observed in the 92-result table (zero fails).

---

## 9. Finished View coordinate verification

Parsed `<rect>` attributes from `buildDiceTrayFinishedViewSvg` (finite + positive width/height for all rects in each case).

| Scenario | Finite | Positive | Lid | Storage / divider / insert notes |
|----------|:------:|:--------:|:---:|----------------------------------|
| Storage right | yes | yes | no | storage right of divider |
| Storage left | yes | yes | no | storage left of divider; mirrored vs right |
| Storage right + wood insert | yes | yes | no | insert x left of divider; inside rolling |
| Storage left + cork liner | yes | yes | no | insert x right of divider; inside rolling |
| No storage + wood insert | yes | yes | no | insert only; no storage/divider rects |

**Production SVG:** no `Finished View` / `dice-tray-assembled` identifiers (`svgHasFv: false`).

**Semantic metrics sample (150×150, storage 42 right, wood insert 3 mm panel):**  
rolling clear width 105; storage 42; insert 104.2 × 149.2; usable wall above insert 32 mm.

---

## 10. Legacy-output comparison

Independent generation with fixture defaults (220×160×35, finger joint, no storage/insert):

| Output | Bytes | FNV / fixture hash | SHA-256 |
|--------|------:|--------------------|---------|
| Legacy Dice Tray | **1726** | **`51a55721`** | **`2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf`** |
| Alternate-joint Dice Tray | **1054** | **`41697123`** | (not required; length+FNV match) |
| Divider Tray | **1965** | **`a55dda6e`** | **`91a5fcaaa54a63d9c5f9b3e9851de70b62fdd98798d356d50977ed4f46f1d924`** |

All match the expected pins.

---

## 11. Existing-template comparison

| Suite / probe | Result |
|---------------|--------|
| Designs geometry (`?selftest=design` re-run) | **1093 / 0** |
| Gift Box fixtures | **69 / 0** |
| Tray model standalone `runTrayModelFixtures()` | **264 / 0** |

Representative default SVG byte/FNV probes for Gift Box / Finger Box / Sliding-lid / Drawer Cabinet / coupons were sampled with broad defaults; **do not treat raw byte drift vs historical pin tables as defects** without matching exact fixture inputs and label settings. Authoritative protection is the green Designs geometry suite and Gift Box suite above, plus legacy tray pins in §10.

Implementation report still lists historical representative pins for Finger Box, Sliding-lid, Drawer Cabinet, coupons; those templates were not re-pinned in this narrow correction pass beyond full-suite green status.

---

## 12. Full-suite arithmetic

Independent re-invocation of all 29 registered selftest groups (same set as `selftest=all`):

| Group | Passed | Failed |
|-------|-------:|-------:|
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| production | 66 | 0 |
| promotion | 58 | 0 |
| design-production | 118 | 0 |
| grid | 23 | 0 |
| grid-machine | 18 | 0 |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| design-project | 17 | 0 |
| metadata | 12 | 0 |
| storage | 15 | 0 |
| first-run | 19 | 0 |
| machine | 50 | 0 |
| machines-m1 | 29 | 0 |
| machines-m2 | 31 | 0 |
| machines-m3 | 27 | 0 |
| help | 37 | 0 |
| modal | 28 | 0 |
| accessibility | 36 | 0 |
| responsive | 45 | 0 |
| empty-state | 60 | 0 |
| beginner | 22 | 0 |
| project-wizard | 216 | 0 |
| gift-box | 69 | 0 |
| **dice-tray-system** | **92** | **0** |
| design | 1093 | 0 |
| **TOTAL** | **2454** | **0** |
| **Groups** | **29** | |

Reconciliation: prior verified baseline **2362 + 92 = 2454**.

| Expected (if runtime confirms) | Actual |
|--------------------------------|--------|
| Dice Tray System 92 / 0 | **92 / 0** |
| Complete suite 2454 / 0 | **2454 / 0** |
| Groups 29 | **29** |
| Designs geometry 1093 / 0 | **1093 / 0** |
| Gift Box 69 / 0 | **69 / 0** |
| Tray standalone 264 / 0 | **264 / 0** |
| M1 / M2 / M3 | **29 / 0**, **31 / 0**, **27 / 0** |
| Promotion-switch standalone | **`runPromotionTargetSwitchFixtures`: 16 / 0** (16 `add(`; nested under evidence promotion, not a separate `all` group) |

**DT1 under `selftest=all`:** registered exactly once  
`if (selftest === 'dice-tray-system' || selftest === 'all') runDiceTraySystemFixtures();`

---

## 13. Direct file:// results

| Check | Result |
|-------|--------|
| `git diff --check` | clean |
| `python -m html.parser index.html` | exit 0 |
| Inline JS load (headless Edge dump-dom) | app IIFE executes; fixtures runnable |
| `?selftest=dice-tray-system` | 92 / 0 |
| `?selftest=design` | 1093 / 0 |
| `?selftest=gift-box` | 69 / 0 |
| `?selftest=machines-m1/m2/m3` | 29 / 0, 31 / 0, 27 / 0 |
| Complete 29-group sequence (all) | 2454 / 0 |
| Tray / promotion-switch standalone | 264 / 0, 16 / 0 |

Broader browser UX groups (responsive, modal, accessibility, beginner, Help, storage/recovery, Design-to-Project, M1–M3, etc.) are included in the **2454 / 0** aggregate above. No page exceptions observed in headless fixture runs. Duplicate-ID / console polish beyond suite green was not separately re-scanned in this narrow pass (see §18).

---

## 14. Documentation verification

| Claim | Status |
|-------|--------|
| README focused **92 / 0** | Present |
| README complete **2454 / 0** across **29** groups | Present |
| Implementation report hard minimum **20 mm** | Present |
| Implementation report full-suite verification | Present (2454 / 29 groups) |
| Implementation report insert/cover overlap correction | Present |
| CHANGELOG DT1 bullet | Additive open tray + storage/divider/insert/cover/FV/fixtures; **does not** claim sliding lid, removable dividers, finger-jointed outer walls, recesses, or physical proof |
| Non-wood liner safety copy | Intact in form help |
| Physical validation claims | **None** in implementation report / CHANGELOG |

---

## 15. Protected-boundary comparison

| Boundary | Status |
|----------|--------|
| `APP_ID` | `genmitsu-l8-tracker` |
| `APP_NAME` | `Genmitsu L8 Tracker` |
| `APP_VERSION` | `0.9.0` |
| `BUILD_DATE` | `2026-07-19` |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | `2` |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` |
| Machines / M1 / M2 / M3 suites | Green (29/31/27) |
| Promotion (incl. switch 16/0) | Green |
| Project schema / accounting / Inventory / Pricing | Not modified in DT1 tracked diff surface; suites green |
| Divider Tray semantics + bytes | Protected (1965 / `a55dda6e` / SHA match) |
| Legacy Dice Tray bytes | Protected (1726 / `51a55721` / SHA match) |
| Gift Box suite | 69 / 0 |
| Kerf convention / LightBurn colors / filenames | No evidence of change; Designs suite green |
| Import/export/backup / offline | Unchanged constants; storage fixtures in full suite green |

Tracked diff remains limited to `index.html`, `README.md`, `CHANGELOG.md` for product-facing DT1 work.

---

## 16. Findings by severity

### BLOCKER
None.

### IMPORTANT
None. All six correction themes from the focused audit are satisfied with independent coordinate/threshold evidence.

### POLISH
None required for commit.

### NOT A DEFECT

| Item | Rationale |
|------|-----------|
| Cover-only assembly labels absent | `dt1` requires storage or insert type; cover alone is not DT1 metrics mode |
| Wood **panel** thickness must match material | Separate contract; height rule still applies; measurement-only covers height boundaries |
| Label color `#00ff00` vs older `#00A000` probes | Production uses `#00ff00` for DT1 assembly-label group; fixtures pass |
| `18` string in fixtures | Only as **blocked** storage width case |
| Promotion-switch not its own `selftest=all` group | Nested 16 assertions inside evidence promotion; standalone 16 / 0 confirmed |

---

## 17. Exact final verdict

# APPROVED TO COMMIT

---

## 18. Remaining unverified areas

- Interactive manual click-through of every Designs form control in a headed browser (headless dump-dom + API probes used instead).
- Exhaustive duplicate-ID DOM scan and long-session browser console beyond suite runs.
- Physical laser cut of DT1 panels (explicitly out of software verification).
- Re-pinning every non-tray Designs template to historical byte tables under every label toggle combination (full Designs suite stands in as regression gate).

---

## 19. Physical prototype readiness

**Software side:** ready for scrap testing of wall-to-base tabs, storage divider through-slot, optional wood insert panel, and underside cover alignment.

**Not proven by this verification:** kerf fit, glue-up strength, insert friction, cover cosmetic alignment on real stock, or retrieval ergonomics of narrow storage bays.

---

## 20. Whether physical laser testing is required

**Yes.** Digital fixtures and coordinate probes do not replace scrap cuts on the Genmitsu L8. Recommended before selling or locking Hex Dice Tray architecture: wall/base coupon, small legacy tray, then one DT1 storage + divider, then wood insert ± cover.

---

## 21. Whether DT1 may be committed

**Yes.** Verdict is **APPROVED TO COMMIT**. Nothing is staged yet; commit remains an operator action. No push was performed during this verification.

---

## 22. Whether Hex Dice Tray architecture may begin after commit and scrap testing

**Yes — after** (1) DT1 commit and (2) successful scrap/laser smoke tests of DT1 geometry. Architecture review already defers sliding lid to DT2 and treats Hex as a later system; do not start Hex implementation before those two gates.

---

## 23. Whether Claude capacity remains reserved for DT2

**Yes.** No new high-impact shared-helper or production-output defect was found that would require a second full Claude architecture audit before commit. Reserve deeper Claude capacity for DT2 (e.g. sliding lid / next phase), not for re-litigating closed DT1 correction themes.

---

## 24. Confirmation: no product mutation during verification

This verification:

- Did **not** edit, stage, commit, push, reset, clean, stash, checkout, move, rename, or delete product source files.
- Used **temporary** HTML copies under the OS temp directory for headless probes only.
- Wrote **only** this report file:  
  `docs/DESIGNS_DICE_TRAY_DT1_CORRECTION_VERIFICATION_2026-07-20.md`
- At end of verification, HEAD remains **`294cc06`**, branch **`main`**, ahead/behind **`0 0`**, nothing staged by this process.

---

## Correction-theme scorecard (summary)

| # | Theme | Status |
|---|-------|:------:|
| 1 | Wood insert + underside cover no longer overlap | **PASS** (coordinates) |
| 2 | Storage hard minimum exactly 20 mm | **PASS** |
| 3 | Insert thickness ≥ wall height blocked | **PASS** |
| 4 | DT1 assembly labels honor control | **PASS** |
| 5 | Focused fixtures protect layout/threshold/label/FV | **PASS** (92 / 0) |
| 6 | Measurement-only inserts do not reserve empty strip | **PASS** |

---

*End of correction re-verification report.*
