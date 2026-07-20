# Designs Dice Tray DT1 — Focused Independent Audit

**Date:** 2026-07-20  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit type:** Read-only focused verification of uncommitted Dice Tray DT1  
**Committed baseline:** `294cc06` — *Add parametric Gift Box generator*  
**Phase under review:** Dice Tray DT1 (working tree only; not committed)  
**Architecture reference:** `docs/DESIGNS_DICE_TRAY_SYSTEM_ARCHITECTURE_REVIEW_2026-07-20.md`  
**Implementation report:** `docs/DESIGNS_DICE_TRAY_DT1_IMPLEMENTATION_2026-07-20.md`  

---

## Exact final verdict

# **CHANGES REQUIRED BEFORE COMMIT**

| Severity | Count |
|----------|------:|
| BLOCKER | 1 |
| IMPORTANT | 4 |
| POLISH | 3 |
| NOT A DEFECT | several (documented) |

**DT1 may not be committed** until the blocker and the locked 20 mm storage contract are corrected and narrowly re-verified.  
**DT2 must not begin** until DT1 is corrected, committed, and scrap-tested.  
**Claude Sonnet is not required** for the correction; Grok-level re-verification is sufficient.

---

## 1. Repository state

| Check | Result |
|-------|--------|
| `git status -sb` | `## main...origin/main` with **M** `index.html`, `README.md`, `CHANGELOG.md` |
| `git log -1 --oneline` | `294cc06 Add parametric Gift Box generator` |
| `git rev-parse HEAD` | `294cc064f82020ce02ca95e2ced997f003715ba9` |
| `origin/main...main` | **0 0** |
| Staged | **Nothing staged** |
| DT1 commit/push | **None** |
| Tracked DT1 files | Intended: `index.html`, `README.md`, `CHANGELOG.md` only |
| Architecture + implementation reports | Present as untracked docs |
| Unrelated untracked | LightBurn projects, historical docs, `debug.log`, utilities — **untouched** |
| `git diff --check` | CRLF warnings only; no whitespace error markers |
| Diff size | ~177 lines in `index.html` (+170 / −15 across three tracked files) |

### Application constants (unchanged)

| Constant | Value |
|----------|--------|
| `APP_ID` | `genmitsu-l8-tracker` |
| `APP_NAME` | `Genmitsu L8 Tracker` |
| `APP_VERSION` | `0.9.0` |
| `BUILD_DATE` | `2026-07-19` |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | `2` |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` |

---

## 2. Files / functions reviewed

| Area | Locations |
|------|-----------|
| Defaults | `designDefaults()` — `diceStorageMode`, `diceStorageWidth`, `diceFixedDivider`, `diceInsertType`, `diceInsertClearance`, `diceInsertThickness`, `diceWoodInsertPanel` |
| Form / conditional UI | Dice Tray form section, Help DT1 section, insert safety copy |
| Normalize | `normalizeDesignDraft()` dice-tray branch |
| Validation | `trayModelValidationErrors()` |
| Model | `buildTrayModel()` DT1 branch |
| Serialize / labels | `serializeTrayCompatibilitySvg()` |
| Projection | `trayFinishedViewProjection()` |
| Finished View SVG | `buildDiceTrayFinishedViewSvg()` DT1 branch |
| Design result | `buildTrayDesignResult()` |
| Results UI / assembly | `designResultsHtml()` DT1 metrics and assembly message |
| Project handoff | `projectDraftFromDesign()` dice-tray DT1 notes |
| Fixtures | `runDiceTraySystemFixtures()` (51 asserts), interaction with `runTrayModelFixtures()` |
| Route | `?selftest=dice-tray-system` and `?selftest=all` registration |
| Docs | README DT1 bullet, CHANGELOG DT1 entry, implementation report |

No product files were modified by this audit. Temporary probe HTML was written only under `%TEMP%\dt1-audit-out\` outside the repository.

---

## 3. Legacy-output comparison

Independent runtime probe (same process as the app, disposable Edge profile, `file://`):

| Case | Expected pin | Actual | Result |
|------|--------------|--------|--------|
| Default Dice Tray SVG length | 1726 | 1726 | **PASS** |
| Default Dice Tray hash | `51a55721` | `51a55721` | **PASS** |
| Default has no `dt1` projection | — | true | **PASS** |
| Default has no DT1 `assembly-labels` | — | true | **PASS** |
| Tab-and-slot default | 1054 / `41697123` | match | **PASS** |
| Divider Tray default | 1965 / `a55dda6e` | match | **PASS** |
| Underside cover default | 1778 / `8e2ea3f4` | match | **PASS** |
| Stale storage width with mode `none` | legacy path | length 1726, no dt1 | **PASS** |

These pins match the existing Designs geometry / tray-model goldens. They were **not** rewritten by DT1 fixtures; `runDesignGeometryFixtures()` still asserts the same hashes and reports **1093 / 0**.

Additional smoke (valid generation, not full byte pins in this probe): Gift Box, Finger Box, Sliding-lid Box all remain valid. Full Designs suite green implies Gift Box / Finger Box / Sliding / Drawer / coupon goldens inside that suite remain stable.

**Conclusion:** When storage = none, insert = none, no wood panel, the default Dice Tray production path is **byte-stable** against baseline pins. Highest production boundary for legacy open tray is **intact**.

---

## 4. Product-contract threshold finding

### Locked implementation prompt

- **Block** clear storage widths **below 20 mm**  
- **Warn** below **28 mm**

### Implementation report claim

- Hard minimum **18 mm**, warning below 28 mm  

### Independent source + runtime

| Storage clear width (mm) | Valid? | Behavior |
|--------------------------|--------|----------|
| 17.9 | **No** | Error: “Storage clear width must be at least **18** mm.” |
| 18.0 | **Yes** | Warning: narrower than 28 mm |
| 19.0 | **Yes** | Warning: narrower than 28 mm |
| 19.99 | **Yes** | Warning: narrower than 28 mm |
| 20.0 | **Yes** | Warning: narrower than 28 mm |
| 27.9 | **Yes** | Warning: narrower than 28 mm |
| 28.0 | **Yes** | No 28 mm retrieval warning |

Source anchors:

- Form: `min="18"` and help text “below 18 mm is blocked”  
- `trayModelValidationErrors`: `values.storageWidth < 18`  
- `normalizeDesignDraft`: `designRequiredNumber(..., 18, true)`  
- Fixtures: block only `17` (not 19.99); encode 18 mm in error match  

**Classification:** **IMPORTANT — product-contract deviation.**  
Source permits **18–19.999 mm**. This is **not** an inaccurate report alone; the **implementation** diverged from the locked prompt. Fixtures passing does **not** authorize the change. **Correction required before commit** unless Joe explicitly re-locks 18 mm with a documented workshop reason (none found in architecture review as a safety requirement to lower the floor).

Implementation report’s “18 mm” text matches source but **conflicts with the locked prompt**; it must be updated when the threshold is corrected to 20 mm.

---

## 5. Dimension-model findings

Verified formulas (default 220×160×35, t=3, storage 42):

| Quantity | Formula / expectation | Actual |
|----------|----------------------|--------|
| `rollingClearWidth` | insideW − storage − t = 220 − 42 − 3 | **175** |
| `rollingClearDepth` | insideD | **160** |
| `storageClearDepth` | insideD | **160** |
| `baseOutsideWidth` | insideW + 2t | **226** (unchanged with storage) |
| `baseOutsideDepth` | insideD + 2t | **166** (unchanged) |
| Left/right outside footprint | same | **PASS** |
| Divider thickness counted once in rolling width | yes | **PASS** |
| Storage width = clear bay (not centerline span) | yes (slot center at clear edge + t/2) | **PASS** |

Slot centers (fit clearance 0.15 → slot width 3.3):

- Right storage 42: slot x = **187.925** (matches formula)  
- Left storage 42: slot x = **54.925** (matches formula)  

Units remain mm in the tray model; Designs unit conversion for other templates is unchanged. DT1 form fields are mm-labeled.

---

## 6. Storage left / right findings

| Check | Result |
|-------|--------|
| Right storage valid | PASS |
| Left storage valid | PASS |
| Modes mirror in metrics (`storageMode`) | PASS |
| Finished View storage rect X differs left vs right | PASS (coordinate-level) |
| Outside footprint invariant | PASS |
| One fixed divider required (unchecked → blocked) | PASS |

---

## 7. Divider geometry

| Check | Result |
|-------|--------|
| Exactly one divider when storage on | PASS |
| No divider when storage off | PASS |
| Full-depth through-cut base slot | PASS (`heightMm === insideDepth`) |
| Slot uses tray slot width (thickness + 2×fit) | PASS |
| Divider panel uses wall path + depth tab profile | PASS |
| Panel thickness = stock thickness | PASS (tabs into slot) |
| No fake pocket | PASS |
| Glue-after-dry-fit warning | PASS |
| Dice migration past divider ends | Butt to front/back interior faces by length = inside depth; residual gaps are kerf/fit, not intentional large openings |

**Wall-slot intersection:** Axis-aligned overlap test between the full-depth divider slot and front/back wall tab slots **fails** (slots share the wall-band region where wall slots protrude into the floor). This matches the existing **divider-tray** pattern (divider slots already cross wall-slot bands).  

**Classification:** **NOT A DEFECT** relative to established tray-model joinery, but scrap dry-fit remains mandatory; no new “no-overlap” guarantee should be claimed.

---

## 8–10. Insert contract, non-wood safety, wood insert

| Mode | Measurement | Structural wood cut | Safety warning | Notes |
|------|-------------|---------------------|----------------|-------|
| Disabled | — | no | — | Legacy path |
| Cork / leather / felt / generic | dimensions reported | **no INSERT panel / no INSERT label** | measurement-only + composition | PASS |
| Wood + panel + same thickness | dims + panel | red rect + INSERT label | — | PASS (219.2×159.2 at 0.4 clearance) |
| Wood + panel + different thickness | blocked | — | match structural thickness error | PASS |
| Wood measurement-only (panel off) | dims | no panel | — | PASS |

Formulas confirmed:

- `insertWidth = rollingClear − 2×clearance`  
- `insertDepth = rollingClearDepth − 2×clearance`  
- `usableWallAboveInsert = wallHeight − insertThickness`  

Clearance: 0 allowed; negative blocked; oversized clearance blocked.  

**No base insert score outline** in DT1 production SVG (no blue score). Absence is acceptable; documentation does not claim a recess.

Help / form / warnings mention unknown leather, vinyl-backed felt, mystery coatings, adhesives, separate process, ventilation, fire watch. Adequate for DT1; not a universal material-safety certificate.

### BLOCKER — wood insert + underside cover layout overlap

When **Generate wood insert panel** and **Add underside cover plate** are both enabled:

| Panel | xMm | yMm | widthMm | heightMm |
|-------|-----|-----|---------|----------|
| rolling-insert-panel | 10 | 307 | 219.2 | 159.2 |
| tray-bottom-cover | 10 | 312 | 226 | 166 |

Independent geometry probe: **`overlaps: true`**.  

Both are red cut panels on the same sheet. Nested/overlapping cut paths are a **production SVG defect**.

Root cause: `bottomCoverYmm` is still computed from `existingLayoutHeightMm` as if no DT1 strip exists, while insert (and divider strip height) are placed at `existingLayoutHeightMm` / `dt1ExtraHeight` without pushing the cover below that band.

Storage + cover **without** wood insert does **not** overlap the divider (divider is placed to the right of the base column). The failure mode is specifically **insert panel vs cover**.

**Classification:** **BLOCKER.**

---

## 11. Finished View findings

Coordinate-level checks (DOM-parsed rects):

| Scenario | Finite coords | Storage/divider | Insert | Mirror |
|----------|---------------|-----------------|--------|--------|
| Legacy (no dt1) | uses pre-existing path; suite green | n/a | n/a | n/a |
| Storage right | PASS | PASS | none | — |
| Storage left | PASS | PASS | — | X differs from right |
| Wood insert only | PASS | none | positive rect | — |
| Storage + liner | suite + probe finite | PASS | insert in rolling region | — |

No NaN/Infinity in production or DT1 Finished View samples. Screen-only; production SVG lacks Finished View ids. No lid.  

**Gaps:** DT1 Finished View is a simplified top-down schematic (not the richer legacy isometric cues). Caption “FRONT / rolling area” is slightly ambiguous for left storage (POLISH). No automated assert that insert rect is strictly outside storage rect in fixtures (probe checked combo case separately for containment intent).

---

## 12. Labels and warnings

| Behavior | Finding |
|----------|---------|
| DT1 production always injects green `assembly-labels` when any DT1 feature is on | Independent of global `assemblyLabels` checkbox |
| Labels: BASE, FRONT, BACK, LEFT, RIGHT, DIVIDER (if storage), INSERT (if wood panel) | Matches code |
| Non-wood liners never get INSERT | PASS |
| BOTTOM-COVER label | **Not** in DT1 label list (cover still cut; unlabeled) — POLISH |
| Narrow storage warning &lt; 28 mm | PASS |
| Non-wood measurement-only warning | PASS |
| Shallow usable wall (&lt; 12 mm above insert) | PASS |
| Insert thickness &gt; wall height | **Allowed**; usable height can be **negative** (−5 for 40 mm insert / 35 mm wall) — IMPORTANT |

---

## 13. Result summary

DT1 metrics UI reports mode, outside footprint, rolling/storage clears, divider text, insert type/dims/clearance, usable wall above insert, wood panel inclusion, piece count.  

Legacy summary path remains when `!dt1`. No evidence of mislabeled legacy metrics when storage is off.

Piece counts (probe): storage only → 6; wood insert only → 6; storage + wood → 7 (plus cover adds another when enabled).

---

## 14. Design-to-Project

- Still uses generic `projectDraftFromDesign`  
- No Project schema / storage changes  
- DT1 notes: storage side/size + through-cut divider; insert report + measurement-only for non-wood  
- Name still `Dice Tray …`  
- Fixtures assert handoff notes for storage case  

Legacy handoff baseline note for plain dice tray remains. No machine/accounting/Inventory/Pricing mutation observed.

---

## 15. Validation findings

| Case | Behavior |
|------|----------|
| Nonfinite / zero / negative storage | Blocked via required number + min |
| Hard min | **18 mm** (contract issue) |
| Storage consumes rolling | Blocked |
| Divider optional unchecked | Blocked when storage on |
| Negative insert clearance | Blocked |
| Insert consumes cavity | Blocked |
| Wood thickness mismatch + panel | Blocked |
| Stale storage width with mode none | Ignored; legacy path |
| Insert over wall height | **Not blocked** |

---

## 16. Fixture-quality findings

`runDiceTraySystemFixtures()` recount: **51** `add(...)` assertions. Live: **51 passed / 0 failed**.

| Coverage need | Status in 51 |
|---------------|--------------|
| Nested tray-model group green | Indirect (`runTrayModelFixtures().failed === 0`) — strong |
| Explicit legacy hash `51a55721` in DT1 group | **Absent** (relies on nested + design suite) |
| Left/right + formulas | Direct |
| Divider slot placement (magic `187.925`) | Weak but present |
| Wall-slot non-overlap | **Absent** (and not required if convention-documented) |
| Non-wood measurement-only | Direct |
| Same-thickness wood panel | Direct (label-based; panel id not in cut markup) |
| Different-thickness block | Direct |
| FV finite strings | Direct (not coordinate geometry) |
| FV side mirror coords | **Absent** |
| Insert containment in FV | **Absent** |
| Insert + bottom cover non-overlap | **Absent** — missed the blocker |
| 20 mm contract | **Absent** (encodes 18) |
| Project handoff | Direct |
| Suite registration | Present once under `all` |

**Classification:** **IMPORTANT** — fixtures are useful but insufficient alone for highest-risk layout paths; they did not catch insert/cover overlap or the 20 mm contract.

---

## 17. Full-suite arithmetic

### Pre-phase baseline (given / architecture)

2362 / 0 across 28 groups.

### Independent live `file://` Edge (disposable profiles)

| Route | Result |
|-------|--------|
| `?selftest=dice-tray-system` | **51 / 0** |
| `?selftest=design` | **1093 / 0** |
| `?selftest=gift-box` | **69 / 0** |
| `?selftest=machines-m3` | **27 / 0** |
| `?selftest=machines-m1` | **29 / 0** |
| `?selftest=machines-m2` | **31 / 0** (plus nested re-runs in log) |
| `?selftest=promotion` | **58 / 0** (evidence promotion group; not the old 16-count switch subset name) |
| `runTrayModelFixtures` via probe | **264 / 0** |
| `?selftest=all` | All observed groups **0 failed**; DT1 registered **once** after Gift Box, before Design |

Expected complete arithmetic **2362 + 51 = 2413 / 0 across 29 groups** is **consistent** with top-level group addition. Console logs also show nested re-invocation of some groups (pre-existing pattern); do not sum nested console lines as the official total.

Implementation report correctly stated full suite was not yet verified; **this audit ran it**. After commit-blocking fixes, re-run `?selftest=all` again.

Benign console noise (pre-existing): `<svg> attribute height: Expected length, "auto"` during some Finished View fixtures — does not fail assertions.

---

## 18. Direct `file://` results

| Check | Result |
|-------|--------|
| HTML parse (`python -m html.parser index.html`) | **OK** |
| Startup load | **OK** (no page exception) |
| Focused / design / gift / m3 / all | **0 failed** as above |
| Uncaught JS in DT1 paths | **None** observed |
| Offline | No network dependency for generation |

Disposable profiles only; no real user profile or records touched.

---

## 19. Existing-output regression

| Template / area | Evidence |
|-----------------|----------|
| Dice Tray legacy | Hash pins + design suite |
| Divider Tray | Hash pin + suite |
| Gift Box | 69 / 0 |
| Finger / Sliding / Drawer / coupons | Inside design 1093 / 0 |
| Kerf / LightBurn red cut | Tray still red cut group; DT1 adds green label stroke group only when dt1 |
| Filenames | Unchanged pattern `l8-dice-tray-…` in existing fixtures |

---

## 20. Documentation findings

| Doc | Finding |
|-----|---------|
| README DT1 | Accurate: open tray, storage, fixed divider, measurement-only liners, no lid/recess; selftest route; 51 / 2413 arithmetic |
| README sliding lid on DT1 | **Not claimed** |
| CHANGELOG | Bounded; no finger outer walls / removable dividers / physical proof overclaim |
| Implementation report | Correctly notes full suite pending; **states 18 mm hard min** (matches code, conflicts with locked 20 mm prompt) |
| Implementation report after this audit | Must be corrected for threshold and full-suite status once fixes land |

---

## 21. Protected-boundary comparison

| Boundary | Status |
|----------|--------|
| APP_* / STORAGE_KEY / SCHEMA / BACKUP | Unchanged |
| machines M1/M2/M3 | Green; no DT1 coupling |
| promotion | Green |
| Project schema | Unchanged |
| accounting / Inventory / Pricing | Unchanged |
| Divider Tray semantics | Preserved |
| Legacy Dice Tray bytes | Preserved |
| Gift / Finger / Sliding / Drawer / coupons | Suite-green |
| Offline / no new deps | Preserved |

---

## 22. Findings by severity

### BLOCKER (1)

1. **Wood insert panel + underside cover cut layout overlap** — production panels share sheet space (`buildTrayModel` cover Y vs DT1 insert Y). Affects production SVG bytes for that option combination.

### IMPORTANT (4)

1. **Storage hard minimum 18 mm vs locked 20 mm** — source, UI, fixtures, and implementation report.  
2. **Fixture gaps** — no insert/cover layout assert; no 20 mm contract; weak FV geometry; no explicit legacy hash in DT1 group.  
3. **Insert thickness may exceed wall height** — negative usable wall allowed without hard block.  
4. **DT1 always emits green assembly labels** when any DT1 feature is on, independent of `assemblyLabels`, and uses a different label path than other templates (LightBurn layer discipline risk if green is cut).

### POLISH (3)

1. Non-wood insert-only still expands layout height (`dt1ExtraHeight`) with empty sheet band.  
2. DT1 Finished View wording “FRONT / rolling area” under left storage.  
3. Bottom cover unlabeled under DT1 label set.

### NOT A DEFECT

- Divider slot crossing wall-slot bands (divider-tray convention).  
- No insert score guide / no recess / no lid (bounded DT1).  
- Measurement-only non-wood (correct).  
- Different-thickness wood panel blocked (correct).  
- Full suite arithmetic expectation once top-level groups are counted.  
- Implementation report honesty that full suite was pending (now run).

---

## 23. Exact final verdict (restated)

**CHANGES REQUIRED BEFORE COMMIT**

Not **APPROVED TO COMMIT** and not **APPROVED WITH POLISH DEFERRED**, because of (a) a production cut-layout overlap blocker and (b) a locked product-contract threshold violation.

---

## 24. Minimal correction plan

Do **not** start DT2. Keep changes inside the existing `dice-tray` / tray-model path.

| # | Change | Exact touch points | Production SVG? | Schema? |
|---|--------|--------------------|-----------------|---------|
| 1 | Raise storage hard min **18 → 20** mm | Form `diceStorageWidth` `min` + help text; `trayModelValidationErrors` (`< 20`); `normalizeDesignDraft` `designRequiredNumber` min; `runDiceTraySystemFixtures` invalid case + new 19.99/20 cases; implementation report line; any README threshold if added | Only for previously illegal 18–19.99 designs (now blocked) | No |
| 2 | Place underside cover **below** DT1 strip | `buildTrayModel`: compute `bottomCoverYmm` after `dt1ExtraHeight` (or max of DT1 panel bottoms + gap) so cover never overlaps insert/divider bands; fixture: wood panel + cover → no AABB overlap | **Yes** for cover+DT1 combinations | No |
| 3 | Block or hard-warn insert thickness ≥ wall height | `trayModelValidationErrors` / normalize; fixture | Validation only unless previously emitted | No |
| 4 | Labels: either gate on `assemblyLabels` **or** document forced DT1 labels and keep green non-cut guidance | `serializeTrayCompatibilitySvg` + Help | Possible label presence change | No |
| 5 | Fixture upgrades | Legacy hash `51a55721` in DT1 group; insert+cover non-overlap; 20 mm contract; optional FV storage/insert AABB | No (tests) | No |
| 6 | Docs | Fix implementation report threshold + “full suite verified after correction”; README only if threshold text is added | No | No |

**After correction:** narrow re-verify `?selftest=dice-tray-system`, `?selftest=design`, `?selftest=all`; re-probe insert+cover non-overlap and 17.9/19.99/20 thresholds. **Grok verification sufficient.** Claude not needed.

**Physical testing after correction:** still required before sale/production (coupon → small open tray → storage dry-fit → measure insert from cavity). Not software-proven.

---

## 25. Remaining unverified areas

- Interactive multi-step UI click tour in a headed browser (form toggles) — partially covered by fixtures + model probe  
- Inch display path for DT1 fields (model is mm-centric)  
- Exhaustive multi-sheet nesting for very large trays  
- Physical laser cut of any DT1 configuration  
- Exact promotion-switch 16-count historical alias (promotion group is 58 today)  
- Byte comparison of Gift/Sliding against a separate 294cc06 worktree export (suite goldens used instead)

---

## 26. Physical prototype readiness

**Coupon / scrap testing only** — not small-prototype-ready for combined wood-insert + cover until layout fix; not normal production ready.

Recommended progression after corrections:

1. Existing wall/base fit coupon  
2. Small no-storage open tray  
3. Storage tray with divider dry-fit  
4. Measure insert from assembled cavity  
5. Verify liner composition before any laser on non-wood  

---

## 27. Whether physical laser testing is required

**Yes**, after software corrections, before any sold or gift-critical unit. Software cannot prove fit, kerf, glue, or liner safety.

---

## 28. Whether DT1 may be committed

**No** — not until blocker #1 and contract threshold #1 are fixed and re-verified.

---

## 29. Whether DT2 may begin after commit and physical tests

**Only after:** DT1 corrections land, commit, scrap validation of storage + divider, and Joe accepts DT1 for workshop use. Sliding lid remains a **separate** phase (architecture review DT2).

---

## 30. Whether Claude capacity should be reserved for DT2

**No for DT1 correction.** Reserve capacity for DT2 sliding-lid geometry if desired later; DT1 fix is local and Grok-verifiable.

---

## 31. Confirmation — audit hygiene

| Action | Status |
|--------|--------|
| Edited product tracked files | **No** |
| Staged | **No** |
| Committed | **No** |
| Pushed | **No** |
| Reset / clean / stash / checkout | **No** |
| Temp probe files | Only under `%TEMP%\dt1-audit-out\` (outside repo) |
| This report written | `docs/DESIGNS_DICE_TRAY_DT1_FOCUSED_AUDIT_2026-07-20.md` (audit deliverable only) |

---

## Appendix A — Independent probe summary

```
AUDIT_SUMMARY 58 passed / 2 failed / total 60
FAIL slot_no_wall_overlap  → classified NOT A DEFECT (tray convention)
FAIL insert_cover_overlap_BUG → BLOCKER (real)
legacy_hash 51a55721 PASS
focused_51 51/0 PASS
tray_264 264/0 PASS
contract_19_allowed PASS (documents deviation)
```

## Appendix B — Selftest registration (excerpt)

```text
if (selftest === 'gift-box' || selftest === 'all') runGiftBoxFixtures();
if (selftest === 'dice-tray-system' || selftest === 'all') runDiceTraySystemFixtures();
if (selftest === 'design' || selftest === 'designs' || selftest === 'all') runDesignGeometryFixtures();
```

DT1 appears **exactly once** under `all`.

---

*End of focused DT1 audit.*
