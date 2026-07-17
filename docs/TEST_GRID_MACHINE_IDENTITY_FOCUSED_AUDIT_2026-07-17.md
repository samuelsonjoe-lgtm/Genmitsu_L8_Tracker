# Test Grid Machine-Identity — Focused Independent Audit

**Date:** 2026-07-17  
**Target:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `4dd7a6b` — Add explicit evidence promotion workflow  
**Primary implementation report:** `docs/TEST_GRID_MACHINE_IDENTITY_IMPLEMENTATION_2026-07-17.md`  
**Mode:** Read-only audit (no application edits; no git write operations)

---

## Pre-audit git snapshot

| Check | Result |
|---|---|
| `git status -sb` | `main...origin/main`; modified `README.md`, `index.html`; many untracked docs (including this enhancement’s implementation report) |
| `git log -1 --oneline` | `4dd7a6b Add explicit evidence promotion workflow` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --stat` (working tree vs HEAD) | `README.md` small; `index.html` +173 / −21 (net +152 lines in the machine-identity change set vs `4dd7a6b`) |

**Scope note:** The machine-identity enhancement is **working-tree only** relative to `4dd7a6b`. This audit evaluates that working tree against the baseline commit and the implementation report. Claims in the report were not accepted without independent source and runtime checks.

---

## Runtime baseline (fresh isolated Edge / Chromium `file://`)

Self-tests emit totals via console groups and function return values (no DOM `selftest-group` panel). Authoritative observed totals are **top-level fixture return values**, not nested console sums.

| Run | Observed |
|---|---|
| `index.html` (bare) | Loads; title `Genmitsu L8 Tracker`; no self-test panel |
| `index.html?selftest=grid-machine` | **18 passed / 0 failed** (`runTestGridMachineIdentityFixtures`) |
| `index.html?selftest=promotion` | **58 passed / 0 failed** |
| `index.html?selftest=production` | **66 passed / 0 failed** |
| Complete suite (each top-level runner once after `?selftest=all` load) | **1379 passed / 0 failed** |

### Complete suite group breakdown (observed)

| Group | Passed | Failed |
|---|---:|---:|
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| production | 66 | 0 |
| promotion | 58 | 0 |
| design-production | 118 | 0 |
| grid | 23 | 0 |
| **grid-machine** | **18** | **0** |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| metadata | 12 | 0 |
| storage | 8 | 0 |
| project-wizard | 216 | 0 |
| design | 587 | 0 |
| **Total** | **1379** | **0** |

Matches expected suite total **1379 / 0**. Independent behavioral probes (temp HTML inject of closure APIs only; application files untouched): **57 / 57** additional checks passed.

---

## Findings

### Verified — New Grid automatic machine snapshot

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `openGridForm` → `gridMachineControls` / `gridMachineFromForm` → create submit; `activeMachineSnapshot` |
| **Scenario** | Active machine Genmitsu L8 20W; open New Test Grid; create; change global preference to 40W; reopen; create another with active 40W |
| **Expected** | Form shows `Machine: Genmitsu L8 20W`; saved `machine: { key: "genmitsu-l8-20w", label: "Genmitsu L8 20W" }`; after preference change original Grid still 20W and not rewritten; 40W create stores canonical 40W snapshot |
| **Observed** | Form summary and stored snapshot match; reopen after preference change still 20W; stored object byte-stable; 40W create correct; prior 20W unchanged |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Covered: form display 20W, store 20W/40W, global preference non-rewrite |

### Verified — Advanced machine override

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | Advanced `machineAction` / `machineLabel` in `gridMachineControls`; `gridMachineFromForm`; `gridMachineSnapshot` |
| **Scenario** | Explicit 20W, 40W, custom label; blank custom; confirm global preference unchanged |
| **Expected** | Canonical keys/labels for standards; custom key + exact label; blank/malformed custom rejected; no global preference mutation; no internal keys in ordinary UI |
| **Observed** | Custom stores `{ key: "custom", label: "Joe custom diode" }`; blank custom alerts and does not create; global `machineProfile` unchanged; card/detail/cell ordinary text uses labels only |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Custom identity covered; blank custom rejection **not** in the 18 fixtures (verified by independent probe) |

### Verified — Editing an identified Grid

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | Edit submit with `machineAction=keep`; explicit `machineAction=40W` |
| **Scenario** | Seed Grid with machine, cells, ratings, Best, notes, dates, IDs, unknown top-level and nested fields; rename only; then change machine only |
| **Expected** | Unrelated edit preserves machine, cells, IDs, unknown fields; no preference substitution; machine change only on selected Grid; other Grids unchanged |
| **Observed** | Rename keeps machine and unknowns; explicit 40W change only on selected Grid; other Grid JSON unchanged in probe |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Unrelated edit + selected machine change covered; fixture does not assert full byte-identity of sibling Grids (probe did) |

### Verified — Old machine-less Grid + explicit assign

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `normalizeTestGrid`, edit form (`Leave not recorded` / `Assign active machine`), `backupObject` / `replaceData` / `mergeData` |
| **Scenario** | Seed Grid without `machine`; load/edit; change global machine; backup/import; Assign active; Cancel assign |
| **Expected** | Loads; `Machine: Not recorded`; no automatic assignment; global change does not alter; backup/import do not add machine; assign previews and applies only that Grid; Cancel leaves machine-less |
| **Observed** | All of the above; assign confirm text includes `Assign Genmitsu L8 40W…`; Cancel returns false leaves machine-less; replace/merge retain machine-less and identified states |
| **Consequence** | None — silent backfill **not** present |
| **Recommended correction** | None |
| **Fixture coverage** | Display + assign + cell/unknown preserve + replace/merge covered; **Cancel assign** not in fixtures (probe verified) |

### Verified — Selected-cell Production Setting promotion provenance

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `promoteSelectedGridCell` → `buildGridPromotionCandidate` (`inferMachineLabel: false`) → `startPromotion` |
| **Scenario** | Grid stored as L8 20W while global preference is 40W; promote selected; cancel; repeat patterns for 40W/custom; machine-less |
| **Expected** | Review shows stored machine only; machine-dependent fields tied to candidate machine; no preference substitution; open/cancel no Grid mutation; machine-less warning + blocked machine-dependent Create/Update; Evidence-only available |
| **Observed** | Candidate machine is grid snapshot under 40W preference; review text includes `Genmitsu L8 20W`; cancel leaves source JSON unchanged; machine-less candidate has empty key and review warning path; eligibility blocks cross-machine / missing-machine machine-dependent fields (existing Phase 7.3A rules) |
| **Consequence** | None — preference substitution **not** observed |
| **Recommended correction** | None |
| **Fixture coverage** | Stored-machine candidate + review + source immutability covered; 40W/custom promotion paths covered by probe, not all by the 18 fixtures |

### Verified — Grid-to-Material-Test bridge

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `promoteCell` (requires `g.machine`); `buildGridCellMaterialTestRecord` |
| **Scenario** | Stored 20W under 40W preference; save path builders; machine-less Save to Library |
| **Expected** | MT records Grid label + additive `machineKey`; no preference fallback; source Grid unchanged; machine-less blocked with clear message; no MT created |
| **Observed** | `machine === "Genmitsu L8 20W"`, `machineKey === "genmitsu-l8-20w"`; 40W/custom analogous; machine-less `cellToLibrary` alerts (`no recorded machine` / assign) and leaves `pendingGridPromotion` unset |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Bridge key/label + machine-less non-substitution covered |

Both promotion and MT bridge use the same Grid `machine` snapshot shape (`key` + `label`); MT additionally stores legacy string `machine` plus additive `machineKey` — compatible, not competing formats.

### Verified — Normalization and import

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `normalizeTestGrid`, `gridMachineSnapshot`, `replaceData`, `mergeData` (bodies unchanged vs baseline) |
| **Scenario** | Clone purity; valid/missing/invalid machine; unknown fields; preference change; replace/merge identified and machine-less |
| **Expected** | Non-mutating clone; valid survive; missing stays missing; invalid dropped conservatively; unknowns/cells survive; no active-machine lookup; idempotent; import preserves machine state |
| **Observed** | Input not mutated; valid kept; missing deleted on clone only; unknown key / blank custom → null; known key without label gets default label; no preference use; replace/merge preserve machine-less and snapshots |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Replace/merge + unknown cell field after merge covered |

**Note (not a defect):** `gridMachineSnapshot` intentionally returns only `{ key, label }`, so extra properties on `machine` are dropped when normalized. Cell-level and top-level Grid unknowns are preserved.

### Verified — UI consistency

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `gridCard`, `renderGridDetail`, `gridBrowserDetailHtml`, `openCell`, promotion review, MT record fields |
| **Scenario** | Identified vs machine-less display wording |
| **Expected** | `Machine: Genmitsu L8 20W` / `Machine: Not recorded`; no ordinary internal keys |
| **Observed** | Same `gridMachineLabel` helper across card, detail, browser detail, cell modal; promotion uses candidate label; ordinary panels do not show `genmitsu-l8-*` keys |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Form + cell + promotion review partially; full card/browser rendering not exhaustively fixture-covered (probe + source inspection) |

### Verified — Protected boundaries vs `4dd7a6b`

| Symbol | Body vs `4dd7a6b` |
|---|---|
| `STORAGE_KEY` | Unchanged (`'genmitsu-l8-tracker-v1'`) |
| `SCHEMA_VERSION` | Unchanged (`2`) |
| `persist` | SAME |
| `backupObject` | SAME |
| `replaceData` | SAME |
| `mergeData` | SAME |
| `normalizeProductionEvidence` | SAME |
| `normalizeProductionSetting` | SAME |
| `normalizeProductionSettings` | SAME |
| `buildDesignResult` | SAME |
| `buildJointCouponModel` | SAME |
| `buildDrawerCabinetModel` | SAME |
| `buildDrawerCabinetDesignResult` | SAME |
| `serializeDesignSvg` | SAME |
| `downloadTextFile` | SAME |

No schema migration, bulk backfill, Drawer Cabinet / Designs geometry / SVG / LightBurn production path changes observed in this diff. Phase 7.3A remains session-only until Save. Grid cell matrix directions and `gridCellValues` rating model unchanged by this feature’s machine paths.

### Minor — README Phase 7.3A paragraph still describes pre-enhancement Grid behavior

| Field | Detail |
|---|---|
| **Severity** | Minor |
| **Path** | `README.md` Phase 7.3A section |
| **Scenario** | Reader compares updated Test Grids bullet with Phase 7.3A paragraph |
| **Expected** | Docs consistent: new UI Grids record machine; machine-dependent promotion available when snapshot present; old Grids remain evidence-limited until assigned |
| **Observed** | Test Grids bullet correctly documents auto-snapshot, fixed preference, old Grids unassigned, promotion/MT provenance, provenance≠proof. Phase 7.3A still says *“current UI-created Grids show a warning and remain generally evidence-only/non-machine-dependent”* — stale after this enhancement. Finished Front View blurb still cites complete suite **1361** |
| **Consequence** | User-facing contradiction; risk of wrong operational expectation |
| **Recommended correction** | Update Phase 7.3A wording to distinguish **new** identified Grids vs **old** machine-less Grids; fix residual **1361** total to **1379**; list `grid-machine` (and `promotion` if desired) in individual `selftest` query values |
| **Fixture coverage** | N/A (docs) |

### Minor — Fixture coverage gaps and one overstated name

| Field | Detail |
|---|---|
| **Severity** | Minor |
| **Path** | `runTestGridMachineIdentityFixtures` |
| **Scenario** | Review of all 18 assertions vs required audit surface |
| **Expected** | Assertions independently exercise form, storage, preference isolation, edit, assign, promotion, MT, import |
| **Observed** | 18/0 all pass; genuine form submit paths for create/edit; promotion uses `buildGridPromotionCandidate` + `startPromotion` (not only pure helpers); machine-less library path clicks `cellToLibrary`. Gaps: blank custom rejection; Cancel on assign; exhaustive card/browser UI; byte-identity of non-selected Grids on machine edit (name *“Explicit machine change affects only selected Grid”* only checks sibling still has 40W key). Implementation report’s Material Browser **23** count does not match observed materials runner **57** (suite total 1379 still correct) |
| **Consequence** | Noncritical residual risk if future refactors bypass form paths still covered by fixtures |
| **Recommended correction** | Optional fixture additions for cancel-assign and blank custom; tighten sibling byte-identity assert; correct report breakdown |
| **Fixture coverage** | See classification table below |

### Fixture classification (18)

| # | Name | Classification | Notes |
|---:|---|---|---|
| 1 | New Grid form displays active L8 20W identity | Independent | DOM summary after `openGridForm` |
| 2 | New Grid stores active L8 20W identity | Independent | Real form submit |
| 3 | New Grid stores active L8 40W identity | Independent | Real form submit |
| 4 | Explicit custom Grid override preserves exact identity | Independent | Form override fields |
| 5 | Global machine change does not rewrite stored Grids | Independent | In-memory objects; no silent rewrite path exists |
| 6 | Unrelated Grid edit preserves stored machine | Independent | Form keep path |
| 7 | Explicit machine change affects only selected Grid | Partially circular / overstated | Checks sibling still 40W + different id; not full other-Grid immutability |
| 8 | Old machine-less Grid displays missing identity | Independent | Form summary |
| 9 | Assign active machine updates only the old Grid | Independent | Confirm forced true |
| 10 | Assigning machine preserves cells and unknown fields | Independent | |
| 11 | Selected-cell promotion uses stored Grid machine | Independent | Candidate builder under 40W pref |
| 12 | Promotion review displays stored machine | Independent | Modal text |
| 13 | Opening promotion does not mutate source Grid | Independent | |
| 14 | Grid to Material Test bridge stores key and label | Independent | Builder with stored machine |
| 15 | Machine-less Grid bridge does not substitute active machine | Independent | UI click + alert suppressed |
| 16 | Replace import retains Grid machine snapshot | Independent | Machine-less remains undefined |
| 17 | Merge import retains Grid machine snapshot | Independent | |
| 18 | Machine normalization preserves unknown Grid fields | Partially circular | Asserts cell notes after merge more than pure `normalizeTestGrid` |

No fully circular “assert helper equals helper” checks that would hide preference substitution.

---

## Section checklist (1–11)

| # | Topic | Result |
|---:|---|---|
| 1 | New Grid automatic snapshot | Verified — no rewrite on preference change |
| 2 | Advanced override | Verified — including blank custom rejection (probe) |
| 3 | Edit identified Grid | Verified — preserve + selected-only machine change |
| 4 | Old machine-less Grid | Verified — no silent backfill; assign/cancel correct |
| 5 | Selected-cell promotion | Verified — no preference substitution (would be Blocker if present) |
| 6 | Grid → Material Test bridge | Verified — same snapshot authority; machine-less blocked |
| 7 | Normalization / import | Verified — pure, conservative, no active lookup |
| 8 | UI consistency | Verified — shared label helper; keys not in ordinary UI |
| 9 | Fixture quality | 18/0; mostly independent; minor gaps/overstatement |
| 10 | Regression / protected boundaries | Verified vs `4dd7a6b` bodies listed above |
| 11 | Documentation accuracy | Main Test Grids + suite total accurate; Phase 7.3A + residual 1361 stale |

---

## Severity summary

| Severity | Count | Summary |
|---|---:|---|
| Blocker | 0 | No preference substitution, wrong-machine durable write, source mutation, or storage/import corruption |
| Major | 0 | No silent backfill, snapshot loss on edit/import, or destructive normalization of Grid body |
| Minor | 2 | README internal contradictions (Phase 7.3A / 1361); fixture gap/overstatement + report breakdown nit |
| Verified | 10 | Core create/edit/assign/promote/bridge/normalize/UI/boundaries |

---

## Conclusion

**SAFE TO COMMIT AFTER MINOR CLEANUP**

Implementation behavior is sound and independently verified: automatic active-machine snapshot on create, fixed stored identity under preference changes, explicit assign for old Grids, promotion and Material Test bridges use only `grid.machine`, machine-less paths do not substitute the current preference, protected storage/geometry helpers are unchanged vs `4dd7a6b`, and the complete suite reports **1379 / 0** with machine-identity **18 / 0**. Clean up the stale Phase 7.3A / suite-count README lines (and optional fixture gaps) before treating docs as fully consistent with the code.
