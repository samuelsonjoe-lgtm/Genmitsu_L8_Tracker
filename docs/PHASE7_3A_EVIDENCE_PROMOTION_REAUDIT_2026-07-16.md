# Phase 7.3A — Explicit Evidence Promotion Core
## Independent Re-Audit — 2026-07-16

## Executive summary

Conclusion: **SAFE TO COMMIT AFTER MINOR CLEANUP**

Every Blocker and Major finding from the prior independent audit has been remediated in the current uncommitted working tree and independently re-verified by source inspection plus isolated Edge/Playwright probes. Protected persistence, normalization, Designs geometry, and Drawer Cabinet boundaries remain unchanged versus `a9af90c`.

Residual issues are documentation accuracy (complete-suite total) and remaining high-value fixture/UI-automation gaps. They do not restore incorrect machine provenance, empty Create/Update persistence, unselected Update mutation, weak verified upgrades, coupon identity collision, or incomplete-Project promotion.

No application files were edited during this re-audit. This document is the sole re-audit artifact created by this pass.

## Scope and baseline

| Check | Observed |
|---|---|
| Repository | `C:\Genmitsu L8 Tracker` |
| `git log -1 --oneline` | `a9af90c Add drawer cabinet finished front view` |
| `git status -sb` | `main...origin/main`; modified `README.md`, `index.html`; many untracked docs |
| `git diff --check` | CRLF warnings only |
| `git diff --stat` | `README.md` +6/-?; `index.html` +443/-8 (441 net insertions) |
| Phase 7.3A location | **Unstaged working tree** on top of `a9af90c` (not in the committed tree alone) |

Primary reports read for contract context only; claims were not accepted without independent verification:

- `docs/PHASE7_3_EVIDENCE_PROMOTION_ARCHITECTURE_REVIEW_2026-07-16.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_IMPLEMENTATION_2026-07-16.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_INDEPENDENT_AUDIT_2026-07-16.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_REMEDIATION_2026-07-16.md`

## Independent fixture totals (fresh Edge, direct `file://`)

Observed via `?selftest=all` console capture and direct runner invocation:

| Group | Expected (remediation) | Observed | Match |
|---|---:|---:|---|
| Evidence promotion | 42 / 0 | **42 / 0** | Yes |
| Production settings | 66 / 0 | **66 / 0** | Yes |
| Designs production application | 118 / 0 | **118 / 0** | Yes |
| Designs geometry | 587 / 0 | **587 / 0** | Yes |
| Storage recovery | 8 / 0 | **8 / 0** | Yes |
| Baseline resolution | — | 20 / 0 | — |
| Material Test normalization | — | 12 / 0 | — |
| Test Grid promotion | — | 23 / 0 | — |
| Grid browser | — | 67 / 0 | — |
| Material browser | — | 57 / 0 | — |
| Library browser | — | 56 / 0 | — |
| Project browser | — | 61 / 0 | — |
| Wizard metadata | — | 12 / 0 | — |
| Project Wizard | — | 216 / 0 | — |
| **Complete suite** | **1429 / 0** | **1345 / 0** | **No — remediation overstated** |

Individual URL checks:

- `?selftest=promotion` → 42 passed / 0 failed  
- `?selftest=production` → 66 / 0  
- `?selftest=design-production` → 118 / 0  
- `?selftest=design` → 587 / 0  
- `?selftest=all` → **1345 / 0** (14 groups; no page errors)

Reconciliation: prior independent audit reported **1329 / 0** with **26** promotion fixtures. Current promotion group is **42** (+16). Expected complete total: **1329 + 16 = 1345**. The remediation claim of **1429** is **+84 too high** and is a documentation error, not a missing test group.

---

## 1. Recheck of every original audit finding

### Finding 1 — Grid machine provenance (was Blocker)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `promoteSelectedGridCell()`, `buildGridPromotionCandidate()`, `promotionMachineSnapshot(..., inferLabel=false)` for grids |
| Prior defect | Selected-cell path called `wizardMachineLabel()` (current preference) |
| Observed | `promoteSelectedGridCell` is `startPromotion(buildGridPromotionCandidate(grid, activeCellKey, cell))` — **no** `wizardMachineLabel`. Grid path sets `inferMachineLabel: false`. |
| Probes | Grid 20W with app pref 40W → candidate 20W; reverse 40W with pref 20W → 40W; missing machine → empty key; custom label exact; string label without key → **not** inferred to 20W/40W |
| Create / Evidence-only | Create without machine key blocked; Evidence-only still allowed |
| Fixtures | Exact/missing/custom Grid machine assertions present and pass |
| Residual | Full DOM open-cell with conflicting pref not re-driven in this re-audit; source path is unambiguous |

**Any source-machine substitution: not observed. Blocker cleared.**

### Finding 2 — Compatible selected-field validation (was Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `promotionValidSelectedFields()`, `promotionEligibility()`, `savePromotionTransaction()` |
| Probes | `{ bogus: true }` Create rejected; valid+bogus rejected with Unknown error; empty Create/Update rejected; Evidence-only with `{}` allowed; Save revalidates independently of UI |
| Fixtures | Unknown field, zero valid Create/Update, evidence-only zero fields |

**Empty/incompatible Create/Update that persists: not observed. Major cleared.**

### Finding 3 — Update preservation (was Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `mergePromotionIntoSetting()` + `metadataSelections` via `capturePromotionForm()` |
| Seed | `measurementScope: sheet`, batch `Batch-A`, custom label, verified+preferred, unknown top/nested, existing evidence |
| Update only speed/power/passes/measured thickness | Scope, batch, label, status, verifiedDate, preferred, unknown fields, old evidence preserved; selected values updated |
| Explicit metadata flags | Scope/batch/label/status change only when `metadataSelections.*` true |
| Evidence-only | Values/status/preferred intact (fixture + probe) |

**Unselected context mutation: not observed. Major cleared.**

### Finding 4 — Material Test completion (was Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `buildMaterialTestPromotionCandidate()` physical flag; `promotionStatusRules()` tested gate |
| Physical rule | Only `physicalRunCompleted` / `completed` / `completionStatus|resultState|testStatus === 'completed'` and not archived/failed |
| Probes | Notes-only, summary-only, winner-only → default **not** `tested`; explicit completion → `tested`; archived excluded; `tested` without completion requires reviewed `cutCompleted` |
| Prose parsing | None observed |

**Notes-only/reference silently becoming tested: not observed. Major cleared.**

### Finding 5 — Source-specific verification (was Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `promotionStatusRules()` |
| Coupon | `piecesFit` + date + notes + winner field succeeds; `cutCompleted` / appearance / assembly alone fail |
| Cut MT/Grid | `cutCompleted` required; appearance/assembly rejected |
| Project | `canVerify: false`; verified rejected |
| Status enum | Only `untested|estimated|tested|verified`; `superseded`, blank, case variants, arbitrary text rejected with errors (no silent downgrade at request boundary) |
| Unknown checks | Rejected |

**Inappropriate verified upgrade: not observed. Major cleared.**

### Finding 6 — Coupon identity / signatures (was Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `promotionCanonicalize()`, `buildJointCouponPromotionCandidate()`, `promotionSignature()` |
| Identity | Versioned `coupon-session-v1-` + canonical machine/material/geometry/settings/result |
| Probes | Identical sessions stable; 20W vs 40W different `sourceId`; selected-field key order stable signatures; IDs not derived from signature string as setting/evidence IDs |
| Fixtures | Coupon machine separation; reordered selected-field stability |

**Materially different machine collision: not observed. Major cleared.**

### Finding 7 — Project completion gate (was Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `projectPromotionEligibility()`, `projectPromotionActionHtml()`, `openProjectPromotion()` |
| Eligible | `kept`, `gifted`, `for-sale`, `sold` with valid date + cut/engrave |
| Ineligible | planned, in-progress, cancelled, failed, archived, missing date/op, missing project |
| Handler | `openProjectPromotion` re-checks eligibility (cannot bypass via crafted call when project ineligible) |
| Fit inference | Project candidate has no finger/drawer/lateral fields from notes |
| Verify | Blocked |

**Incomplete Project opening promotion: not observed. Major cleared.**

### Finding 8 — Requested status validation (was Minor/Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Accepted | `untested`, `estimated`, `tested`, `verified` only |
| Rejected | `superseded`, arbitrary, blank, whitespace, case variants without silent repair |

### Finding 9 — Supersede validation (was Minor/Major)

| Field | Detail |
|---|---|
| Severity now | **Verified remediated** |
| Path | `productionSettingsCanSupersede()`, `supersedeProductionSettingDraft()`, Save transaction |
| Probes | Valid same machine/op OK; self/missing/cross-machine/already-superseded/stale `supersededById` rejected; no chain traversal |

---

## 2. End-to-end Save workflows (transaction layer)

Independent probes exercised Save via `savePromotionTransaction` with real `state.profiles` mutation (not review-only):

| Workflow | Result |
|---|---|
| Material Test Create | One new setting + one evidence; selected fields only; source object unmutated |
| Material Test Update | Unselected context preserved (see Finding 3) |
| Grid Create candidate | Machine snapshot exact; Best limitation present; Create blocked without machine |
| Coupon joint-only field map | Winner field present without kerf field when kerf not supplied |
| Coupon kerf independent | Separate field paths; no auto preselect of winner |
| Project evidence-only | Append-only evidence path; target values/status/preferred intact |
| Verified gates | Coupon/Cut rules as above |

**Note:** Full multi-click DOM Save through visible modals for every coupon/grid scenario was not exhaustively UI-automated here; transaction + entry-path source review covers the durable mutation surface. Classified as residual **Minor** coverage gap, not a failed behavior.

---

## 3. Source-deletion resilience

| Source | Probe | Result |
|---|---|---|
| Material Test | Create promotion, clear `profile.tests`, re-read evidence | Snapshot + `sourceId` + `selectedValues` remain readable without live test |
| Grid / Project / Coupon | Snapshot design clones `sourceSnapshot` at Save | Same self-contained pattern; fixtures retain snapshot checks |
| Target profile deletion | Out of scope (owning profile removal) | Not claimed |

Prior overclaimed fixture name (“Evidence remains readable after source deletion”) remains partially circular at fixture level; **independent probe did perform deletion**. Major “evidence requires live source” **not observed**.

---

## 4. Backup, Replace, Merge

| Step | Result |
|---|---|
| `backupObject()` after Create | Promoted settings + evidence snapshots present; `promotionSession` absent |
| `replaceData` of exported profiles | Settings/evidence/snapshots survive |
| `mergeData` into state with local-only profile | Both profiles retained; local evidence ID kept |
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged (`genmitsu-l8-tracker-v1`, `2`) |

No destructive loss observed. Full multi-history preferred+supersede matrix not exhaustively re-imported; residual **Minor** fixture gap.

---

## 5. Atomic rollback

| Scenario | Result |
|---|---|
| Create with `persistFn = () => false` | `ok: false`; `state.profiles` JSON byte-identical to pre-Save |
| Fixture `Persist failure rolls back the complete target` | Pass |

Clone-then-`saveProfileRecord` pattern prevents partial durable writes. Combined preferred+supersede under real `localStorage` quota failure not browser-injected; code path shares same rollback seam — residual **Minor** gap.

---

## 6. Preferred conflicts

| Scenario | Result |
|---|---|
| Decline demotion | Save succeeds non-preferred; same-machine preferred conflict **not** demoted; other-machine preferred intact |
| Accept path | Existing demotion via `confirmDemotion` + conflict filter (same op + machine key) |
| Default preferred | Off (`!!session.preferred`) |

Multiple-conflict listing uses `productionPreferredConflictMessage` with all conflicts. Residual **Minor**: multi-conflict UI message not separately fixture-asserted for label/scope/batch detail completeness.

---

## 7. Duplicate handling

| Scenario | Result |
|---|---|
| Deterministic signature | Pass |
| Reordered selected-field keys | Same signature |
| Duplicate evidence ID | Rejected before mutation |
| Duplicate source warning | Non-destructive warning path |

Signature is not used as setting/evidence ID (`uid()` at Save).

---

## 8. Session inertness

| Check | Result |
|---|---|
| `promotionSession` in `backupObject` / export | Absent |
| Cancel / defaults | `cancelPromotion()` resets session |
| Source mutation during build | Fixture + probe: source JSON unchanged |

Opening A then B: `startPromotion` replaces session from defaults + new candidate clone — no stale field leak by design. Full dual-open UI cycle not re-automated (**Minor** gap).

---

## 9. Fixture quality (42 promotion assertions)

Classification (approximate, independent):

| Class | Count | Notes |
|---|---:|---|
| Independent | ~12 | Status rules pure checks; eligibility errors; machine snapshot pure; project gate pure |
| Partially circular | ~28 | Constructed sessions through `savePromotionTransaction` / adapters with external oracles (IDs, values, errors) |
| Circular | ~2 | “Duplicate signature is deterministic” is pure self-equality of helper (still useful smoke) |

Remediation added coverage for: Grid machine exact/missing/custom; unknown fields; zero valid Create/Update; evidence-only zero fields; Update scope preservation/explicit change; verification ops; whitespace notes; status validation; signature order; coupon cross-machine; Project gate; stale supersede; duplicate evidence ID.

Still thin or partial: full selected-cell DOM with preference conflict; browser localStorage failure injection; full Replace/Merge history matrix; dual-session UI inertness; multi-preferred detail strings; coupon joint-only vs kerf-only **Save** (field map covered).

None of these gaps reintroduce a Major behavior failure under independent probing.

---

## 10. Protected boundaries vs `a9af90c`

Function-body comparison (working tree vs `a9af90c`):

| Symbol | Result |
|---|---|
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged (`genmitsu-l8-tracker-v1`, `2`) |
| `backupObject` | Unchanged |
| `replaceData` | Unchanged |
| `mergeData` | Unchanged |
| `normalizeProductionEvidence` | Unchanged |
| `normalizeProductionSetting` | Unchanged |
| `normalizeProductionSettings` | Unchanged |
| `buildDesignResult` | Unchanged |
| `serializeDesignSvg` | Unchanged |
| `buildJointCouponModel` | Unchanged |
| `buildDrawerCabinetModel` | Unchanged |

Promotion is opt-in entry points only; `buildDesignResult` does not call `savePromotionTransaction` / `startPromotion`. No automatic promotion path found.

**Note:** Legacy Test Grid “Save to library” Material Test bridge still uses `wizardMachineLabel()` for **Material Test** promotion records — that is pre-existing Grid→Library Test behavior, **not** the Phase 7.3A Production Setting selected-cell path.

---

## Findings table (this re-audit)

### Blockers

None.

### Majors

None remaining from the prior audit set.

### Minors

| ID | Severity | Path | Scenario | Expected | Observed | Consequence | Correction | Fixtures |
|---|---|---|---|---|---|---|---|---|
| R1 | **Minor** | Remediation report | Complete suite total | Accurate total | Claimed **1429**; independent **1345** | Misleading readiness metrics | Correct README/report totals to 1345 | N/A (docs) |
| R2 | **Minor** | Fixture suite | Full DOM Grid pref conflict | UI E2E | Code path fixed; candidate/UI path inspected; DOM not re-driven here | Residual confidence gap only | Optional UI fixture | Partial |
| R3 | **Minor** | Fixture suite | Browser localStorage failure | Inject real storage failure | Transaction seam tested via `persistFn=false` | Same rollback path; less browser proof | Optional injection fixture | Partial |
| R4 | **Minor** | Fixture suite | Coupon joint-only vs kerf-only Save | Full Save asserts | Field maps + independence verified; full modal Save not UI-automated | Coverage gap | Optional Save fixtures | Partial |
| R5 | **Minor** | Fixtures | Source deletion re-render | Delete + UI re-render | Data-layer deletion verified; fixture name still soft | Naming/coverage clarity | Strengthen fixture to delete owning source collection | Partial |

### Verified (selected)

- Grid source machine never substituted from current preference on Phase 7.3A path  
- Post-filter field validation blocks unknown/blocked/empty Create/Update  
- Update preserves unselected scope/batch/label/status/preferred/unknown/evidence  
- Material Test completion is structured/conservative  
- Verification is source- and operation-specific  
- Coupon identities separate machines; signatures order-stable  
- Project promotion gated to completed states; no verify  
- Supersede rejects stale metadata; rollback restores full profile  
- Replace/Merge retain promoted evidence snapshots  
- Session not exported; no auto-promotion; protected helpers unchanged  
- Fixture groups: promotion 42/0; complete suite **1345/0**

---

## Comparison to prior audit conclusion

| Prior (NOT SAFE TO COMMIT) | Re-audit |
|---|---|
| Grid machine substitution | Fixed + verified |
| Crafted empty Create/Update | Fixed + verified |
| Update scope/batch wipe | Fixed + verified |
| Weak MT physical inference | Fixed + verified |
| Broad verification checks | Fixed + verified |
| Coupon identity collision | Fixed + verified |
| Incomplete Project entry | Fixed + verified |
| Status / stale supersede gaps | Fixed + verified |
| Fixture total 26 → 42 | Confirmed |
| Complete suite claim | **Must use 1345, not 1429** |

---

## Required conclusion

**SAFE TO COMMIT AFTER MINOR CLEANUP**

Minor cleanup means correcting the overstated complete-suite total in docs/README and optionally hardening remaining E2E fixtures. The Phase 7.3A promotion implementation itself is free of the prior Blockers and Majors and is suitable to commit once documentation totals match the independently observed **1345 / 0**.

---

*Read-only re-audit. Temporary probe HTML under `%TEMP%` only; repository application files unmodified. No stage/commit/push/reset/clean/stash/move/delete.*
