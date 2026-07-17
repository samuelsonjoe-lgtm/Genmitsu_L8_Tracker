# Phase 7.3A — Metadata Cleanup Final Focused Audit

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Committed baseline: `a9af90c` — Add drawer cabinet finished front view  
Audit type: independent, read-only, focused on final metadata cleanup  
Primary cleanup report: `docs/PHASE7_3A_EVIDENCE_PROMOTION_METADATA_CLEANUP_2026-07-16.md`

## Executive summary

Conclusion: **SAFE TO COMMIT**

The final metadata cleanup correctly tracks explicit Update edits for verified date and Preferred, applies them only when dirty, blocks invalid date/status pairs, allows Preferred true→false without a demotion prompt, runs conflict handling only on explicit false→true Preferred requests, preserves Evidence-only targets, and resets metadata flags on target/action switches. All expected fixture totals match independent runtime measurement. No Blockers or Majors were found.

No application files were edited during this audit.

## Repository state

| Check | Observed |
|---|---|
| `git log -1 --oneline` | `a9af90c Add drawer cabinet finished front view` |
| `git status -sb` | `main...origin/main`; modified `README.md`, `index.html`; untracked docs and unrelated files preserved |
| `git diff --check` | CRLF warnings only |
| `git diff --stat` | `README.md` +8/-?; `index.html` +582/-9 (581 net) |
| Phase 7.3A + metadata cleanup | Unstaged working tree on top of `a9af90c` |

Context reports were read for contract history only; claims were not accepted without independent verification.

## Independent runtime totals

Fresh Microsoft Edge headless, direct `file://`, **no nested-console double-counting**.

| Group / URL | Expected | Observed |
|---|---:|---:|
| `?selftest=promotion` / Evidence promotion | 58 / 0 | **58 / 0** |
| Target-switch helper (`runPromotionTargetSwitchFixtures`) | 16 / 0 | **16 / 0** (nested into promotion group as 16 of 58) |
| Production settings | 66 / 0 | **66 / 0** |
| Designs production application | 118 / 0 | **118 / 0** |
| Designs geometry | 587 / 0 | **587 / 0** |
| Storage recovery | 8 / 0 | **8 / 0** |
| `?selftest=all` complete suite | 1361 / 0 | **1361 / 0** |

Complete suite breakdown (independent runner invocation, same totals as `?selftest=all` console groups):

| Group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material Test normalization | 12 | 0 |
| Production settings | 66 | 0 |
| Evidence promotion | 58 | 0 |
| Designs production application | 118 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid browser | 67 | 0 |
| Material browser | 57 | 0 |
| Library browser | 56 | 0 |
| Project browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs geometry | 587 | 0 |
| **Total** | **1361** | **0** |

Page errors: none observed in probe/selftest runs.

---

## 1. Metadata-selection state

| Check | Expected | Observed | Severity |
|---|---|---|---|
| Keys | label, measurementScope, batchLabel, status, verifiedDate, preferred | All six present | **Verified** |
| Fresh session | All false | `promotionSessionDefaults()` all false | **Verified** |
| `syncPromotionMetadataFromTarget` | Flags false; loads target metadata | Flags false; label/scope/batch/status/date/preferred loaded | **Verified** |
| Target not mutated during sync | Byte-stable target | JSON equality before/after sync | **Verified** |
| Null target | Clears preferred/date; flags false | Confirmed | **Verified** |
| Evidence-only | Suppresses metadata edits | Capture forces `verifiedDate`/`preferred` false; evidence path does not call merge | **Verified** |

---

## 2. Verified-date-only Update

Rendered form path (start promotion → Update → select target → change date only + min field → Save):

| Check | Result |
|---|---|
| Dirty flag | `metadataSelections.verifiedDate === true`, status not dirty |
| Save | Succeeds without new physical checks while status remains `verified` |
| Date | `2026-07-01` → `2026-07-15` |
| Status / preferred | Remain `verified` / `true` |
| Scope, batch, label, unknown fields, prior evidence | Preserved |
| Source | Unmutated |
| Evidence append | One new promotion evidence record |

Invalid combinations:

| Scenario | Result |
|---|---|
| Clear date while status verified | Save blocked; target date unchanged |
| Date with non-verified status (explicit date edit) | Save blocked: “Verified date requires verified status.” |
| `promotionVerifiedDateRules('verified','')` | Error |
| `promotionVerifiedDateRules('tested','2026-07-01')` | Error |
| `promotionVerifiedDateRules('verified','2026-07-01')` | Clean |

**No Major verified-date corruption.**

---

## 3. Preferred true-to-false

Rendered Update: uncheck Preferred + min compatible field → Save.

| Check | Result |
|---|---|
| Target preferred | `false` |
| Demotion confirm | **Not** invoked (confirm callback never called) |
| Same-machine other preferred | Unchanged |
| Other-machine preferred | Unchanged |
| Status, date, unknown fields | Survive |

**Inability to clear Preferred: not observed.**

---

## 4. Preferred false-to-true

Seeded same-machine conflict, different-machine preferred, different-operation preferred.

| Scenario | Result |
|---|---|
| Accept demotion | Target preferred; same-machine conflict demoted only; 40W and engrave preferred intact |
| Decline demotion | Target saves non-preferred; conflict remains preferred |
| Conflict gate | Runs only when `metadataSelections.preferred && session.preferred` (create or explicit update) |

---

## 5. Unchanged Preferred across target switches

| Step | Result |
|---|---|
| Edit date/preferred on B, switch to A | A loads (non-preferred, empty date); all metadata flags false |
| Switch back to B | B preferred + original date reloaded; flags false |
| Save without touching Preferred | B remains preferred; `metadataSelections.preferred` false |

No checkbox leak between targets.

---

## 6. Evidence-only preservation

Hacked form: verified date, Preferred, label, scope, batch, status altered before capture.

| Check | Result |
|---|---|
| Metadata flags for date/preferred | Forced false on Evidence-only capture |
| Save | Exactly one evidence appended |
| Target core metadata/values/unknown | Unchanged (preferred true, date, label, scope, batch, status, future fields) |
| Preferred conflict prompt | Not run |

**Evidence-only target metadata mutation: not observed.**

Note: Evidence-only protection is primarily architectural (merge not applied on evidence action) plus explicit zeroing of verifiedDate/preferred dirty flags. Label/scope/batch/status dirty flags may still compute true if the form is tampered with, but they cannot mutate the target on the evidence path. Residual **Minor** hardening opportunity only if evidence ever gains merge.

---

## 7. Target-switch reset

Independent form switches and the 16 target-switch fixtures confirm:

- Create → Update, Update → Evidence-only, target ID switch reload metadata and clear flags  
- Save after switch mutates only the selected target  
- Target A unchanged when B is saved  
- Explicit metadata edits after switch apply when dirty  

---

## 8. Save-time adversarial validation

Crafted sessions (not form-trusted):

| Crafted state | Result |
|---|---|
| `metadataSelections.verifiedDate=true`, empty date, status verified | Blocked; no mutation |
| Date edit with status `tested` | Blocked |
| Evidence-only with `metadataSelections.preferred=true` | Save OK; preferred not demoted/changed |
| Persist failure during preferred request | Full profile rollback |

---

## 9. Fixture quality

### Evidence promotion (58)

Includes prior 42-class coverage plus nested **16** target-switch/metadata assertions (`Target transition: …`).

| Class | Approx. count | Notes |
|---|---:|---|
| Independent | ~18 | Status/date rules, eligibility, machine snapshots, project gate |
| Partially circular | ~38 | Form/transaction with external oracles (values, IDs, prompts) |
| Circular | ~2 | Pure signature self-equality smokes |

New metadata assertions exercise rendered form controls, dirty flags, Preferred clear/accept/decline, verified-date-only Save, Evidence-only preservation, and target-switch flag reset — confirmed by independent re-probe.

### Target-switch helper (16)

All 16 pass; names accurately describe form load, flag clear, Update/Evidence-only isolation, verified-date, Preferred clear, and conflict accept/decline. Mostly **partially circular** (real form + `savePromotionTransaction`) with strong oracles.

Overclaim residual (pre-existing, not introduced by this cleanup): “Evidence remains readable after source deletion” remains a snapshot presence check rather than UI delete+rerender.

---

## 10. Regression and protected boundaries

Prior Phase 7.3A safety re-confirmed by suite 1361/0 and code review:

- Session-only until Save  
- Grid source machine exact / missing stays missing  
- Compatible selected-field validation  
- Unselected Update preservation  
- Conservative Material Test completion  
- Source-specific verification  
- Canonical coupon identity  
- Completed-Project gate  
- Strict status enum  
- Stale supersede rejection  
- Duplicate evidence ID rejection  
- No Drawer Cabinet fit inference / no automatic promotion  

Function-body comparison vs `a9af90c`:

| Symbol | Result |
|---|---|
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged (`genmitsu-l8-tracker-v1`, `2`) |
| `backupObject`, `replaceData`, `mergeData` | Unchanged |
| `normalizeProductionEvidence/Setting/Settings` | Unchanged |
| `buildDesignResult`, `serializeDesignSvg` | Unchanged |
| `buildJointCouponModel`, `buildDrawerCabinetModel` | Unchanged |

Pre-existing Chromium SVG `height="auto"` Finished Front View warning is outside this cleanup; geometry fixtures still pass. No evidence this diff introduced it.

---

## 11. Documentation accuracy

| Claim | Observed |
|---|---|
| README complete suite 1361/0 | Present and correct |
| README Evidence Promotion 58 | Present and correct |
| Verified-date Update / Preferred clear / Evidence-only preservation | Documented in README Phase 7.3A paragraph |
| Grid machine limitation | Still documented |
| Guaranteed fit/production safety claims | None found |

Cleanup report totals match independent measurement.

---

## Findings

### Blockers

None.

### Majors

None.

### Minors

| ID | Severity | Path | Scenario | Expected | Observed | Consequence | Correction | Fixtures |
|---|---|---|---|---|---|---|---|---|
| M1 | **Minor** | `capturePromotionForm` Evidence-only | Tampered label/scope/batch/status | All metadata dirty flags suppressed | Only verifiedDate/preferred forced false; other flags may be true but evidence path never merges | No data loss today; future merge wiring risk | Zero all metadata flags on evidence action | Evidence-only fixtures pass behaviorally |
| M2 | **Minor** | Fixtures | Source deletion naming | Full delete+rerender | Snapshot presence only | Overstated fixture name | Optional strengthen | Partial |

### Verified (selected)

- Six metadata flags with clean defaults and sync  
- Verified-date-only Update without re-verification  
- Preferred clear without demotion prompt  
- Preferred conflict accept/decline isolation  
- Evidence-only byte preservation of target metadata  
- Target-switch flag reset and no cross-target leak  
- Adversarial date validation at Save  
- Atomic rollback on persist failure  
- Complete suite **1361 / 0**; promotion **58 / 0**; switch helper **16 / 0**  
- Protected boundaries unchanged vs `a9af90c`  

---

## Required conclusion

**SAFE TO COMMIT**

---

*Read-only audit. Temporary probe HTML under `%TEMP%` only. Repository application files unmodified. No stage/commit/push/reset/clean/stash/move/delete.*
