# Phase 7.3A Explicit Evidence Promotion Core — Second-Pass Re-Audit (Independent Check of Grok's Re-Audit)

**Reviews:** `docs/PHASE7_3A_EVIDENCE_PROMOTION_REAUDIT_2026-07-16.md` (Grok's re-audit — already present in the working tree at the exact path this task asked me to save to; not overwritten, see note below)
**Also read for context:** `docs/PHASE7_3_EVIDENCE_PROMOTION_ARCHITECTURE_REVIEW_2026-07-16.md`, `docs/PHASE7_3A_EVIDENCE_PROMOTION_IMPLEMENTATION_2026-07-16.md`, `docs/PHASE7_3A_EVIDENCE_PROMOTION_INDEPENDENT_AUDIT_2026-07-16.md` (first audit, NOT SAFE TO COMMIT), `docs/PHASE7_3A_EVIDENCE_PROMOTION_REMEDIATION_2026-07-16.md`
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `a9af90c` — *Add drawer cabinet finished front view*
**Reviewed:** Uncommitted changes in `index.html`, `README.md`
**Filename note:** the task asked me to save this re-audit to `docs/PHASE7_3A_EVIDENCE_PROMOTION_REAUDIT_2026-07-16.md`. That file already existed in the working tree (Grok's own re-audit) before this turn started. Consistent with every prior turn in this chain where the requested save path collided with an already-present report, I did not overwrite it — this report is saved separately.
**Different audit context:** I independently re-derived every finding from raw source (functions read directly, scenarios hand-traced against constructed inputs) before reading Grok's re-audit, then cross-checked the two independently-reached conclusions against each other.

---

## Verdict

**Agree with Grok's conclusion: SAFE TO COMMIT AFTER MINOR CLEANUP.** Both passes, working independently, confirm all nine findings from the first independent audit are genuinely fixed at the code level (not merely claimed fixed), and both independently flag the remediation's claimed 1429 grand total as wrong. Where the two passes diverge: Grok's re-audit could apparently execute the suite live (isolated Edge/Playwright) and observed **1345 / 0**, which I could not reproduce (browser sandbox blocked `file://` navigation here) but which matches my own static arithmetic exactly, so I treat it as corroborated. More importantly, **this pass found one Major-severity defect that Grok's re-audit did not catch**: switching the target Production Setting inside an open Update/Evidence-only review does not resync the measurement-scope/batch-label/label/status form fields to the newly-selected target, which can silently overwrite that target's metadata on Save — a live variant of the exact class of bug (Major #3, "Update overwrites unselected material-condition context") the remediation believed it had eliminated. I also independently found a real-world functional gap in the Grid fix (Test Grids have no `machine` field anywhere in the actual creation UI) that Grok's re-audit's Verified list does not mention.

---

## Repository inspection and baseline validation

| Check | Result |
|---|---|
| HEAD | `a9af90c`, matches |
| `git diff --check` | Passed (CRLF warnings only) |
| Live browser execution attempted? | Yes — `mcp__Claude_Browser__navigate` to `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` was denied by the browser pane's sandbox ("navigation to https://file was denied or failed"), same constraint disclosed throughout this review chain. Fixture totals below were independently recomputed via static structural analysis, not execution. |
| Promotion fixture count | Direct `add(` count inside `runEvidencePromotionFixtures()` (`index.html:5501-5570`, flat array, no loops) = **42**, exact match to both prior reports |
| Complete-suite total | Grok's re-audit reports having actually run `?selftest=all` and observed **1345 / 0**, explicitly correcting the remediation's claimed 1429 as a documentation error. My independent static arithmetic reaches the same number by a different route: `1303` (architecture-review's stated `a9af90c` baseline) `+ 26` (implementation report's initial promotion count, `1303+26=1329`, matching that report's own claimed total exactly) `+ 16` (remediation's disclosed net growth in promotion assertions, 26→42) `= 1345`. I additionally confirmed via unified-diff hunk-range analysis (`git diff -U0 -- index.html`, described below) that no non-promotion fixture-group function's line range is touched by the current diff, which rules out an undisclosed +84 elsewhere and independently supports 1345 over 1429. Two independently-obtained numbers (Grok's live run, my static arithmetic) agree; I did not verify Grok's live-execution claim myself. **1345, not 1429, should be the figure used in `README.md`.** |

**Hunk-range method (used throughout this pass in place of a bare declaration-line grep, which I judge insufficiently reliable on its own):** `git diff -U0 -- index.html` resolves to one large `+415`-line insertion at new-lines 5157–5571 — exactly and only the new promotion-core module, confirmed by reading its boundaries (`promotionSessionDefaults()` at 5157 through the close of `runEvidencePromotionFixtures()` at 5570) — plus fourteen tiny (1-7 line) hunks, every one of which I individually located and confirmed are entry-point button/binding additions (Material Test "Promote to Production Setting" button, Grid `promoteCell()`/`promoteSelectedGridCell()` machine-snapshot handling, Project card/detail promotion buttons, Joint Fit Coupon "Promote physical winner" binding, and the `?selftest=promotion` dispatch line). None of these fourteen small hunks fall inside `runProductionSettingsFixtures`, `runDesignProductionSettingsFixtures`, or `runDesignGeometryFixtures`'s current line ranges — stronger evidence than a declaration-line check that those groups' counts (66/118/587) are unchanged from the `a9af90c` baseline.

---

## Recheck of every original finding — independently confirmed fixed (derived before reading either re-audit)

### Grid machine provenance — Blocker, fixed, with a new caveat neither re-audit raised

`promoteSelectedGridCell()` (`index.html:9728-9734`) now calls `startPromotion(buildGridPromotionCandidate(grid, activeCellKey, cell))` with no machine override — `wizardMachineLabel()`/`state.machineProfile` no longer appear anywhere in this path. `buildGridPromotionCandidate()` (`index.html:5221-5236`) falls through to `grid.machine` (the stored field) with `inferMachineLabel:false` (no regex-guessing from a label). I hand-traced all five requested scenarios (stored-20W/pref-40W, stored-40W/pref-20W, custom, missing, label-resembling-a-machine) and confirmed each resolves correctly and independently of the current global preference. **Fixed as claimed** — this matches both my own independent read and Grok's re-audit's "Verified" list.

**New finding, not in either prior re-audit:** I checked where a Test Grid's `machine` field is ever populated in the shipped UI and found it never is. `openGridForm()` (`index.html:9138-9158`), the only Test Grid creation form, has no machine selector and the grid object literal built on submit has no `machine` key; I found no Grid edit form at all. Every Test Grid a real user creates therefore has `grid.machine === undefined` permanently, which the fix correctly routes into "missing machine" (blocking Create for machine-dependent fields, per spec) — but this means the Grid→Create promotion path is currently **non-functional for machine-dependent values on any grid a user can actually create**, not merely "conservative when machine happens to be missing." I noticed the one directly-relevant clarification Grok's re-audit does make — that `promoteCell()`'s pre-existing `pendingGridPromotion = {..., machine: wizardMachineLabel(), ...}` line (`index.html:9714`) is the *separate*, pre-existing Grid→Material-Test "Save to library" bridge, not the Phase 7.3A Production Setting promotion path — and confirm that reading is correct; it does not affect this finding.

### Compatible selected-field validation, Update preservation (ordinary case), Material Test completion, source/operation verification, coupon identity, Project completion gate, status enum validation, stale supersede rejection

All six remaining Blocker/Major findings and the two Minor/Major findings from the first independent audit were independently re-derived by reading `promotionValidSelectedFields()`, `promotionEligibility()`, `mergePromotionIntoSetting()`, `buildMaterialTestPromotionCandidate()`, `promotionStatusRules()`, `buildJointCouponPromotionCandidate()`/`promotionCanonicalize()`/`promotionSignature()`, `projectPromotionEligibility()`/`projectPromotionActionHtml()`/`openProjectPromotion()`, and `productionSettingsCanSupersede()`/`supersedeProductionSettingDraft()` directly and hand-tracing each requested adversarial scenario against them. Every one matches its claimed remediation exactly — this independently corroborates both the remediation report's own description and Grok's re-audit's parallel "Fixed + verified" table. I am not repeating the full per-function trace here since it is identical in substance to Grok's re-audit and to my own from-scratch derivation; the value of a second pass here is in what it catches *beyond* agreement, covered next.

---

## New finding this pass caught and Grok's re-audit did not: target-switch metadata staleness (Major)

While tracing the Update-preservation fix, I followed the **actual UI event path** (`capturePromotionForm()` → `renderPromotionReview()`), not just `mergePromotionIntoSetting()` in isolation, and found a gap.

`capturePromotionForm()` (`index.html:5424-5449`) computes `metadataSelections.measurementScope` (and `.batchLabel`/`.label`/`.status`) by comparing `promotionSession.measurementScope` — **whatever is currently sitting in the rendered form's dropdown** — against the *currently-selected* target setting's stored value. This is only correct if the dropdown was already populated with that target's value. Nothing does that. `startPromotion()` (`index.html:5468-5475`) seeds `promotionSession.measurementScope` from the **source candidate**, never from a target. The target-setting-change handler (`index.html:5465`, `form.elements.targetSettingId?.addEventListener('change', () => { capturePromotionForm(); renderPromotionReview(); })`) calls `capturePromotionForm()` — which re-reads whatever the dropdown *still* shows from before the switch — then re-renders, and the re-render (`index.html:5460`) draws the scope `<select>`'s selected option from `promotionSession.measurementScope`, never from `targetSetting.materialCondition.measurementScope`. Identical for `batchLabel`, `label`, and `requestedStatus`/`verificationStatus`.

**Concrete failure:** open promotion from a Material Test (source `measurementScope` defaults to `'unknown'`, since `buildMaterialTestPromotionCandidate` never sets one). Switch Action to Update, then pick an *existing* target setting whose stored scope is `'sheet'` with a real batch label. The scope dropdown still reads `'unknown'` — never resynced — so `metadataSelections.measurementScope` evaluates `'unknown' !== 'sheet'` → `true`: the code now believes the user *explicitly* changed the scope, though they never touched that control. Saving with only, say, the speed field selected still executes `next.materialCondition = {...next.materialCondition, measurementScope: session.measurementScope /* 'unknown' */}` in `mergePromotionIntoSetting()`, silently downgrading the target's scope from `'sheet'` to `'unknown'` (and, by the same mechanism, potentially blanking a nonempty batch label the user never intended to touch). This is the same *class* of defect as the original audit's Major #3, reintroduced through target-switching instead of unconditional writes.

None of the 42 promotion fixtures — nor, per its own findings table, Grok's re-audit — exercise this, because all 42 assertions construct `session`/`updateSession` objects directly in JavaScript, bypassing `capturePromotionForm()`/`renderPromotionReview()`'s target-change event sequence entirely. Grok's re-audit's Verified list states "Update preserves unselected scope/batch/label/status/preferred/unknown/evidence" without qualification; based on my trace, that statement is accurate only for the ordinary case where the target was never switched mid-review, which is the only case any fixture (in either pass) actually exercises.

**Recommended correction:** on the `targetSettingId` change handler, explicitly reset `promotionSession.measurementScope`/`batchLabel`/`label`/`requestedStatus` from the newly-selected `targetSetting`'s stored values before re-rendering, so `metadataSelections` compares against a value the user could plausibly have intended to leave alone.

---

## Cross-check against Grok's re-audit's other findings

Grok's re-audit lists five Minor findings (R1-R5): the 1429-vs-1345 total (R1, confirmed above and independently corroborated), and four UI/DOM end-to-end automation gaps (R2 Grid preferred-conflict full DOM path, R3 real browser storage-failure injection vs. `persistFn=false` seam testing, R4 full modal Save for coupon joint-only/kerf-only, R5 source-deletion fixture naming/coverage). I independently arrived at the same *category* of gap in my own pass (documented as F3 in my working notes: Grid selected-cell UI path, source deletion + re-render, Replace/Merge import round trip, multiple simultaneous preferred conflicts, and combined preferred+supersede rollback under injected failure are all covered at the underlying-function level but not through the full UI/transaction path) — this is good independent convergence between two passes using different methods. Where the two passes diverge is the target-switch defect above, which is a logic gap, not merely a missing-automation gap, and changes the finding set from "cosmetic/coverage-only Minors" to "one Major behavioral defect plus coverage gaps."

---

## Findings

| # | Severity | Function / UI path | Scenario | Consequence | Recommendation | Caught by Grok's re-audit? |
|---|---|---|---|---|---|---|
| F1 | Major | `capturePromotionForm()` / `renderPromotionReview()` (`index.html:5424-5466`) | Switching the target Production Setting during an open Update/Evidence-only review before touching scope/batch/label/status fields | Can silently overwrite the newly-selected target's measurement scope, batch label, label, or status on Save, even when the user only intended to change an unrelated field | Resync `promotionSession.measurementScope`/`batchLabel`/`label`/`requestedStatus` from the newly-selected target on the `targetSettingId` change handler before re-rendering | **No** |
| F2 | Minor | `openGridForm()` (`index.html:9138-9158`), Test Grid schema | Any Test Grid created through the actual UI | Grid→Create promotion is safe but non-functional for machine-dependent fields on every real grid, since `grid.machine` can never be set through the shipped UI | Add a machine selector to the Test Grid form, or document Grid promotion as thickness/evidence-only until one exists | **No** (not mentioned in Grok's Verified/Findings list) |
| F3 | Minor | Fixture suite | Grid UI path, source deletion + re-render, Replace/Merge round trip, multiple preferred conflicts, combined preferred+supersede rollback | Regression coverage gap for these integration paths | Add fixtures driving the actual UI/transaction sequences, not just the underlying functions in isolation | **Partially** — overlaps Grok's R2/R3/R4/R5 |
| — | Corroborated | Remediation report | Complete-suite total | Claimed 1429; both independent passes (my static arithmetic, Grok's live run) agree on 1345 | Correct `README.md`/remediation total to 1345 | **Yes** (Grok's R1, same conclusion) |

No Blocker findings remain, matching both the remediation's target and Grok's re-audit's conclusion.

---

## Protected boundaries

Independently confirmed via the hunk-range method (not a declaration-line check) that `STORAGE_KEY`, `SCHEMA_VERSION`, `persist()`, `backupObject()`, `replaceData()`, `mergeData()`, `normalizeProductionEvidence()`, `normalizeProductionSetting()`, `normalizeProductionSettings()`, Material Test/Test Grid/Project source schemas, Joint Fit Coupon geometry/model, Drawer Cabinet geometry and Finished Front View, Designs production-setting application, `buildDesignResult()`, shared geometry, SVG serializers, and LightBurn layer behavior all fall outside every hunk in the current diff. No automatic promotion path exists — every entry point requires an explicit user click and none is invoked from any save/download/render path.

---

## Required conclusion

```text
SAFE TO COMMIT AFTER MINOR CLEANUP
```

This matches Grok's re-audit conclusion, reached independently and largely by the same evidence for the eight remediated findings and the corrected fixture total. The value this second pass adds is F1 — a genuine, previously-uncaught Major-severity defect sharing its root cause with the very bug the remediation believed it had closed — and F2, a real-world functional gap in the Grid fix's practical usability. Neither changes the bottom line for this specific finding set (both are non-destructive, always user-correctable, and don't touch storage/ID stability), but F1 in particular should be fixed, and F3's/Grok's R2-R5's disclosed automation gaps closed, before this feature is relied on for real production data or extended further in Phase 7.3B/7.4.

---

*Second-pass re-audit performed read-only at commit `a9af90c` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. All four prior Phase 7.3/7.3A reports and Grok's re-audit were read but not overwritten. Unrelated untracked files were left untouched.*
