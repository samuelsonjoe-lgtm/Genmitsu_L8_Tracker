# Phase 7.1 — Proven Production Settings Independent Audit

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Named baseline: `c93ebd1` — Add production joint-fit tuning  
Audit type: independent, read-only, data-preservation focused  
Auditor: independent agent (did not modify application files)

## Repository state inspected

| Check | Result |
|---|---|
| `git status -sb` | `## main...origin/main`; **modified** `README.md`, `index.html` (unstaged); many untracked `docs/` files |
| `git log -1 --oneline` | `c93ebd1 Add production joint-fit tuning` |
| `git diff --check` | LF/CRLF warnings only; no conflict markers |
| `git diff --stat` | `README.md` +14; `index.html` +530/-23 (521 net) |
| Committed `c93ebd1:index.html` contains `productionSettings` | **No** (0 matches) |

**Critical scope note:** Phase 7.1 lives in the **unstaged working tree** (`index.html`, `README.md`), not in the committed tree at `c93ebd1`. This audit inspected the working-tree implementation that implements Phase 7.1 on top of that commit. No files were staged, committed, pushed, reset, cleaned, stashed, moved, deleted, or rewritten.

Primary reports were read for context only; claims were not accepted without independent verification.

## Methods

1. Full read of production-setting constants, normalizers, merge/import paths, editor handlers, duplicate flow, backup/load paths, and Library render summaries.
2. Adversarial JavaScript probes injected into a **temporary copy** of `index.html` under `%TEMP%` (repository file untouched); executed headless via Microsoft Edge + Python Playwright.
3. Direct invocation of all twelve built-in fixture runners in the browser (not only `?selftest=all` console parsing).
4. Cross-check of Designs/SVG code paths for Phase 7.2 leakage.

## Live fixture totals (independently recalculated)

| Group | Passed | Failed | Total |
|---|---:|---:|---:|
| Baseline resolution | 20 | 0 | 20 |
| Material Test normalization | 12 | 0 | 12 |
| **Production settings** | **47** | **0** | **47** |
| Test Grid promotion | 23 | 0 | 23 |
| Grid browser | 67 | 0 | 67 |
| Material browser | 57 | 0 | 57 |
| Library browser | 56 | 0 | 56 |
| Project browser | 61 | 0 | 61 |
| Wizard metadata | 12 | 0 | 12 |
| Storage recovery | 8 | 0 | 8 |
| Project Wizard | 216 | 0 | 216 |
| Designs geometry | 559 | 0 | 559 |
| **Complete suite** | **1138** | **0** | **1138** |

Implementation report claimed 47 production / 1138 total — **confirmed independently**.

Prior baseline (pre–Phase 7.1 committed tree) was 1091/0; delta **+47** production assertions only.

## Production-settings fixture classification (47)

| Class | Count | Notes |
|---|---:|---|
| **Independent** | 14 | Direct `strictOptionalNumber`, `buildLibraryProfileRecord`, `saveProfileRecord`, source JSON equality, `backupObject` serialization, `convertStoredUnits`, `STORAGE_KEY`/`SCHEMA_VERSION`, summary HTML helpers, filter-based delete isolation |
| **Partially circular** | 33 | Exercise real adversarial payloads through `normalizeProductionSetting`, `mergeLibraryProfiles`, `replaceData`, `mergeData`, `duplicateProductionSettings`, `upsertProductionSettingDraft`, `supersedeProductionSettingDraft` — same helpers under test, but non-trivial constructed oracles (incoming-wins, unknown fields, order independence, signed numerics) |
| **Circular** | 0 | No assertion relies solely on `f(x) === f(x)` without an external oracle |

**Fixture gaps (high value, not covered):** UI modal cancel without save; nested evidence editor accidental parent submit; persistence rollback through real `openProfile` DOM path; supersede chain with deleted replacement; import of profile where `recognized` spread could clobber fields if `normalizeSetting` ever emitted `tests`/`productionSettings`; repeated `productionSettingsSummaryHtml` render ID churn (probed separately — none observed).

---

## Findings by review area

### 1. Profile-edit data preservation

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 1.1 | **Verified** | `buildLibraryProfileRecord`, `openProfile` | Edit recognized material field on profile with tests, production settings, evidence, unknown top-level fields | Unrelated nested data and unknown fields preserved | `buildLibraryProfileRecord` spreads `previous` then `recognized`; `normalizeSetting` omits `tests`/`productionSettings`; drafts attached explicitly. Probe: `future` + `futurePs` survived material rename. | Safe ordinary edits | — | Partial (`Profile record overlay…`, `Profile build does not mutate…`) |
| 1.2 | **Verified** | `saveProfileRecord` | Simulated `persist()` failure on profile save | Roll back list slot | Failed persist restores prior object or removes unsaved insert | No silent list corruption | — | `Persist failure restores previous profile` |
| 1.3 | **Verified** | `openProfile` cancel path (code review) | Close modal without submit | Drafts discarded | `profileDraftProductionSettings` only committed on submit; cancel uses `data-close` without save handler | Persisted data unchanged | — | Not UI-tested |
| 1.4 | **Verified** | `normalizeSetting` | Recognized overlay during save | Must not embed empty `productionSettings: []` | `normalizeSetting` returns only scalar profile fields + optional tags | Prevents spread-wipe of nested arrays | — | Indirect |

### 2. Normalization purity and ID stability

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 2.1 | **Verified** | `normalizeProductionSetting`, `normalizeLibraryProfiles` | Missing/non-array/malformed entries; repeated normalization | Non-mutation; stable IDs; parent retained | Source JSON unchanged; `stable` ID unchanged across two passes; malformed children dropped, parent kept | No backup churn from load-time re-ID | — | Multiple fixtures |
| 2.2 | **Verified** | `productionSettingsSummaryHtml` | Render-time normalization | No new IDs when ID present | Uses `setting.id \|\| uid()` only when missing | Stable references in UI | — | Not directly fixture-tested; code review OK |
| 2.3 | **Verified** | `productionKnownValue` | Unknown enum strings | Preserved when non-empty string | Unknown `verificationStatus`, machine key, measurement scope, evidence kind preserved | Forward compatibility | — | `Unknown status and enum values remain recoverable` |

### 3. Strict optional numeric handling

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 3.1 | **Verified** | `strictOptionalNumber` | `''`, spaces, null, undefined, 0, `'0'`, `-0`, `±0.095`, `-0.02`, malformed, NaN, Inf, leading/trailing spaces | Blank stays blank; meaningful values preserved; garbage → `''` | All probe cases passed except exponent (below) | Kerf/fit signed values safe | — | 4 fixtures + probe |
| 3.2 | **Minor** | `strictOptionalNumber` (`index.html` ~4583) | Exponent notation `'1e-2'` | Fixed-decimal UX (architecture emphasis) | Parses to `0.01` (regex allows `(?:e[+-]?\d+)?`) | Unexpected scientific notation in stored evidence if user pastes it | Tighten regex or reject exponent in production fields | **No** |
| 3.3 | **Minor** | `normalizeProductionSetting` | `powerMinPercent: 150`, `measuredThicknessMm: -1` | Sensible bounds (0–100 power; non-negative thickness in UI) | Values stored as `150` and `-1` (`min="0"` HTML only) | Implausible persisted evidence | Clamp in normalizer to match UI intent | **No** |
| 3.4 | **Verified** | `normalizeProductionSetting` | `passes`, `confidence` malformed | Blank not zero | Malformed → `''`; valid passes/confidence clamped | No accidental zero passes | — | `Malformed production numbers become blank…` |

### 4. Nested merge-import correctness

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 4.1 | **Verified** | `mergeProductionSettingRecord`, `mergeNestedRecordsById` | Same setting ID with conflicting nested scalars and unknown subfields on both sides | Incoming-wins conflicts; both unknown keys survive; sources not mutated | Independent probe: `localOnly`+`incomingOnly` on machine; `localCut`+`incomingCut`; evidence `localEv`+`incomingEv`; `result:'new'` | No shallow-merge data loss | — | 7 merge fixtures + probe |
| 4.2 | **Verified** | `mergeLibraryProfiles` | Divergent local/incoming profiles, array order reversed | Deterministic ID-sorted output | `JSON.stringify(merged) === JSON.stringify(mergedReversed)` | Stable imports | — | `Merge output is independent…` |
| 4.3 | **Verified** | `mergeData` | Runtime merge import path | Nested library merge semantics | 3 tests, 3 settings, 3 evidence on shared setting after `mergeData` | Real import path OK | — | `Merge import path uses nested…` |

### 5. Replace-import behavior

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 5.1 | **Verified** | `replaceData` | Full collection replace with production settings + null child | Whole replace; malformed child omitted | `replaceResult` has 1 normalized setting; null dropped | Correct replace semantics | — | `Replace import path reproduces…` |
| 5.2 | **Verified** | `normalizeLibraryProfiles` | Old backup without `productionSettings` | `[]` default | Confirmed | Old backups load safely | — | `Old profile without production settings…` |

### 6. Backup round-trip and recovery

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 6.1 | **Verified** | `backupObject`, `loadState`, `persist` | Nested production + evidence + unknown fields + signed values | Survive JSON round-trip and normalization | IDs, `±0.095`, `-0.02`, `futureTop`, `futureEvidence` preserved | Backup integrity | — | 4 backup fixtures |
| 6.2 | **Verified** | `STORAGE_KEY`, `SCHEMA_VERSION` | Schema stability | Unchanged key; schema 2 | `genmitsu-l8-tracker-v1`, `2` | No migration break | — | Fixture + probe |
| 6.3 | **Verified** | `decodeStoredState` / `loadState` | Corrupt top-level JSON | Recovery path unchanged | Storage recovery fixtures 8/0; no new whole-state wipe for nested malformation | Unrelated data protected | — | Storage recovery group |

### 7. Preferred-record behavior

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 7.1 | **Verified** | `upsertProductionSettingDraft`, `productionSettingConflict` | Same machine+operation conflict; different machine untouched | Decline cancels; accept demotes only conflict | Probe + fixtures confirm | No deletion; evidence intact | — | 2 fixtures + probe |
| 7.2 | **Minor** | `productionSettingConflict` | Two active preferred records: same machine+operation, different `measurementScope` (sheet vs batch) | Phase 7.1 may allow both | Conflict key is `operation` + `machine.key` only — sheet/batch scopes **do not** isolate preference | Marking second preferred demotes first even across scopes | Narrow key documented for 7.1; widen in future phase | Partial (`Different sheet and batch…` tests coexistence, not demotion boundary) |

### 8. Supersede behavior

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 8.1 | **Verified** | `supersedeProductionSettingDraft` | Valid supersede | Old record kept, status `superseded`, `preferred` false, `supersededById` set, evidence intact | Fixture confirmed | History preserved | — | `Superseding preserves old record…` |
| 8.2 | **Verified** | `supersedeProductionSettingDraft` | Self-supersede or missing replacement ID | Reject safely | Returns `null` in probe | No broken chain | — | Probe only |
| 8.3 | **Minor** | `openSupersedeProductionSetting` | Replacement from different machine/operation | Should be constrained | UI lists any non-superseded setting except self | User may link semantically unrelated supersession | Filter choices by machine/operation | **No** |

### 9. Delete isolation

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 9.1 | **Verified** | Draft delete handlers | Delete one production setting / evidence | Only targeted ID removed | Filter on stable ID in draft arrays | Isolated draft edits | — | `Deleting one draft record…` (setting only) |
| 9.2 | **Verified** | `del()` profile delete | Delete parent profile | Removes profile only | Standard `del('profiles', id)` — no cascade into inventory/projects | No cross-collection wipe | — | Not production-specific fixture |

### 10. Duplicate-profile behavior

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 10.1 | **Verified** | `duplicateProductionSettings`, `duplicateProfile` | Verified preferred + superseded history + evidence | New IDs; `preferred` false; `estimated`; `supersededById` cleared; values/snapshots kept; reverify note | All fixture checks pass | Safe duplication | — | 5 duplicate fixtures |
| 10.2 | **Minor** | `duplicateProductionSettings` | Evidence `sourceProfileId` | Informational remapping or clear | Copied evidence retains original `sourceProfileId` (`orig-parent` in probe) | Misleading provenance on copy | Remap to new profile on save or clear with note | **No** |

### 11. Unknown-field preservation

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 11.1 | **Verified** | Normalizers + merge + backup | Unknown at profile, setting, machine, material, cut, lightBurn, fit, evidence | Survive load/normalize/edit overlay/merge/backup | Confirmed at all levels in fixtures and probe | Forward compatibility | — | Multiple fixtures |
| 11.2 | **Verified** | `productionSettingFromForm` | Form edit of known fields | Unknown nested keys on `data` preserved via spread in `productionSettingFromForm` | `...data`, `...data.machine`, etc. before overrides | Editor does not strip future keys | — | Code review |

**Survival summary:** Unknown fields survive at all listed levels through normalize, profile build, merge (non-conflicting keys), backup round-trip, and duplicate (shallow copy). Dropped only when same key conflicts on merge (incoming wins) or user deletes parent record.

### 12. Unit stability

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 12.1 | **Verified** | `convertStoredUnits` | mm ↔ in display toggle with nested production mm fields | Production settings byte-stable | `productionBeforeUnits === productionAfterUnits` | Canonical mm evidence not rewritten | — | `Unit conversion does not rewrite…` |
| 12.2 | **Verified** | `convertStoredUnits` | Library profile thickness/focus | Legacy behavior | Production path not added to `settingPairs`; profile thickness still converted | No regression to Library unit behavior | — | Implicit via unchanged pairs list |

### 13. UI binding and modal-draft isolation

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 13.1 | **Verified** | `openProductionSetting`, `productionEvidenceDraft` | Open/edit production setting | Draft copy; persist only on profile save | Evidence loaded into `productionEvidenceDraft`; setting upsert updates `profileDraftProductionSettings` only | No premature persist | — | Code review |
| 13.2 | **Verified** | Evidence modal | `event.preventDefault()` on evidence form | No parent profile submit | Separate modal + submit handler | Nested edit isolated | — | Code review |
| 13.3 | **Minor** | UI lifecycle | Cancel production-setting modal after edits | Discard draft | `data-close` closes modal; draft array already mutated if user saved setting into draft — cancel on **parent** profile modal discards all drafts on close without save | Expected; full DOM cancel path not fixture-tested | Add UI fixture | **No** |
| 13.4 | **Verified** | Action buttons | Stable `data-*` IDs vs indices | ID-targeted handlers | `data-edit-production-setting`, `data-delete-production-setting`, etc. | Safe rerender | — | Code review |

### 14. Library summary and status semantics

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 14.1 | **Verified** | `productionSettingsSummaryHtml` | Preferred summary | Count preferred excluding superseded | `preferred.filter(item => item.preferred && item.verificationStatus !== 'superseded')` | Superseded not shown as active | — | Summary fixture |
| 14.2 | **Verified** | `libraryProfileTestSummary`, `libraryProfileStatusLabel` | Profile with production settings but no tests | Best Known unchanged | Status still uses `hasPrimary` from profile speed/power — **not** production settings | Wizard/Browse classification unchanged | — | `Production summary exposes preferred status without redefining Best Known` |

### 15. Inventory references

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 15.1 | **Verified** | `materialCondition.inventoryItemId/Name` | Inventory selection | Store ID + snapshot fields | `onchange` copies name/supplier snapshot; IDs stored as strings | Understandable after rename/delete | — | Code review |
| 15.2 | **Minor** | Inventory lifecycle | Deleted or missing inventory on merge/import | Snapshot remains | No live re-link; snapshots informational by design | User must re-link manually | Documented behavior | **No** |

### 16. Regression safety

| ID | Severity | File / function | Scenario | Expected | Observed | Consequence | Fix | Fixtures |
|---|---|---|---|---|---|---|---|---|
| 16.1 | **Verified** | Full suite | All pre-existing groups | 0 failures | 1138/0 independent run | No regressions detected | — | All groups |
| 16.2 | **Verified** | Designs geometry | 559 assertions | Pass | 559/0 | Designs stable vs implementation report | — | Designs group |

### 17. Fixture quality

See classification table above. Production group is **substantive** but **helper-heavy** (33/47 partially circular). Independent probes confirmed merge non-mutation, profile overlay, exponent/bounds edge cases, and Phase 7.2 isolation beyond fixtures.

### 18. Scope confirmation (Phase 7.2 not implemented)

| ID | Severity | Check | Observed | Fixtures |
|---|---|---|---|---|
| 18.1 | **Verified** | Designs production-profile selector | None in Designs UI | — |
| 18.2 | **Verified** | SVG generation reads `productionSettings` | `buildDesignResult.toString()` has no `productionSettings` (probe) | Designs 559/0 |
| 18.3 | **Verified** | Kerf baked into SVG / Library write on download | No code path found | Designs fixtures |
| 18.4 | **Verified** | Auto-promotion from tests/coupons/projects | No automatic promotion helpers added | Grid promotion fixtures unchanged |
| 18.5 | **Verified** | Machine registry / raw-sheet entity | Not added | Code search |

---

## Blockers

**None identified.** No credible path found to whole-profile loss, unstable IDs on load/render, or merge/replace wiping valid nested data under tested scenarios.

## Majors

**None identified** for data preservation. The preferred-conflict scope limitation (7.2) is a known Phase 7.1 workflow narrowness, classified **Minor** because it does not destroy data and matches the bounded 7.1 design.

## Minors (recommended cleanup, non-blocking)

1. Reject or document exponent notation in `strictOptionalNumber` for production fields (3.2).
2. Clamp power 0–100 and optionally non-negative thickness in `normalizeProductionSetting` (3.3).
3. Widen preferred-conflict key when multiple measurement scopes must coexist as preferred (7.2).
4. Filter supersede replacement choices by machine/operation (8.3).
5. Remap or clear `sourceProfileId` on duplicate (10.2).
6. Add fixtures for UI cancel paths, supersede chains with deleted replacement, and exponent/bounds edge cases (17).

## Verified (high-confidence behaviors)

- Additive `profile.productionSettings = []` with backward-compatible load.
- Pure normalization with stable IDs and source non-mutation.
- `buildLibraryProfileRecord` + draft arrays preserve nested data on ordinary profile edit.
- `saveProfileRecord` rollback on persist failure.
- Deep nested merge with incoming-wins and unknown-field retention.
- Replace = full collection replacement with malformed child omission only.
- Backup JSON round-trip for nested production evidence.
- Signed kerf/fit values (`+0.095`, `-0.02`, `0`) preserved.
- `convertStoredUnits` does not touch production canonical mm fields.
- Preferred demotion prompt with safe decline.
- Supersede retains history and rejects invalid targets.
- Duplicate generates new IDs and downgrades verification.
- `STORAGE_KEY` / `SCHEMA_VERSION` unchanged.
- Phase 7.2 not started; Designs does not consume production settings.
- Complete suite **1138 passed / 0 failed**.

---

## Conclusion

**SAFE TO COMMIT AFTER MINOR CLEANUP**

Phase 7.1 data-preservation, merge/import, backup round-trip, and regression safety independently check out. Residual risk is limited to minor validation and workflow-clarity gaps (exponent parsing, unbounded power/thickness, narrow preferred key, supersede picker breadth, duplicate provenance IDs, and UI-level fixture holes) — not to structural data loss. Commit should include the unstaged `index.html` / `README.md` Phase 7.1 implementation; the current `c93ebd1` commit alone does not contain `productionSettings`.

---

*Audit performed read-only. Temporary probe copy only under `%TEMP%`; repository application files unchanged.*