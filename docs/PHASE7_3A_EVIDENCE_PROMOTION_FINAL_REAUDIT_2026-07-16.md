# Phase 7.3A — Explicit Evidence Promotion Core
## Final Independent Read-Only Re-Audit — 2026-07-16

## Conclusion

**SAFE TO COMMIT AFTER MINOR CLEANUP**

No Blocker or Major defect was found in the current target-switch implementation. The stale-target metadata defect is fixed in source and passes a fresh rendered-form transition fixture. The remaining items are Minor: several high-value workflows are still covered at the helper/fixture seam rather than by complete browser persistence/import/deletion automation, and explicit verified-date/preferred-state edits are not independently represented as separate metadata-selection flags.

No application files were modified during this audit. The only file created by this read-only pass is this report.

## Audit baseline and scope

Observed baseline:

- Repository: `C:\Genmitsu L8 Tracker`
- HEAD: `a9af90c Add drawer cabinet finished front view`
- Branch: `main...origin/main`
- Tracked uncommitted changes: `README.md`, `index.html`
- Numerous unrelated untracked files and reports were preserved.
- `git diff --check`: passed, with only the existing LF/CRLF warnings.

The seven Phase 7.3A reports were inspected, including the second-pass re-audit that identified the target-switch defect and the final-cleanup report that documented its intended correction.

## Independent runtime results

Fresh isolated direct-file Chromium contexts produced:

- `index.html`: loaded with no page errors.
- `index.html?selftest=promotion`: **51 passed / 0 failed**.
- `index.html?selftest=production`: **66 passed / 0 failed**.
- `index.html?selftest=design-production`: **118 passed / 0 failed**.
- `index.html?selftest=design`: **587 passed / 0 failed**.
- `index.html?selftest=all`: no page errors; reconciled top-level complete total **1354 passed / 0 failed**.
- Storage recovery group: **8 passed / 0 failed** within the complete run.

The `?selftest=all` console emits nested/chained fixture subtotals, so raw console-line summation overcounts. The current total is independently reconciled as the prior observed **1345** plus the nine new target-switch assertions: **1345 + 9 = 1354**.

Additional direct browser calls:

- `window.runPromotionTargetSwitchFixtures()`: **9 passed / 0 failed**.
- HTML parser: `python -m html.parser index.html` passed.
- No page errors were observed in any fresh context.

## Target-switch synchronization review

### Implementation inspected

The relevant current functions and handlers are:

- `promotionSessionDefaults()`
- `syncPromotionMetadataFromTarget()`
- `startPromotion()`
- `capturePromotionForm()`
- `renderPromotionReview()`
- the action-change handler
- the target-profile change handler
- the target-setting change handler
- `mergePromotionIntoSetting()`
- `savePromotionTransaction()`
- `runPromotionTargetSwitchFixtures()`

`syncPromotionMetadataFromTarget()` clones the session, resets `metadataSelections`, and loads the selected target’s label, measurement scope, batch label, verification status, verified date, and preferred state. With no effective target it resets to candidate defaults without changing the source candidate or selected value fields.

The helper is wired into:

- target Production Setting changes;
- target Library profile changes;
- Create → Update;
- Create → Evidence-only;
- Update ↔ Evidence-only;
- return to a previously selected target.

The target-setting handler captures the current form, resolves the newly selected target from the newly effective profile, updates the target ID, synchronizes metadata, and rerenders. Therefore the rendered controls are loaded from Target B before the next Save-time capture.

### Fresh transition result

The nine transition assertions use the actual rendered `#promotionReviewForm`, dispatch the action and target-setting change events, inspect the rerendered controls, call `capturePromotionForm()`, and call `savePromotionTransaction()`.

Observed passing behavior:

- Target B label, batch, scope, verified status/date, and preferred state appear after switching.
- All metadata-selection flags are false immediately after synchronization.
- Selecting only speed updates speed and appends one evidence record.
- Target B’s unedited metadata, unknown fields, and existing evidence survive.
- Target A remains unchanged in the tested Update and Evidence-only paths.
- Explicit scope change and batch-label clearing apply while untouched label/status/date remain unchanged.
- An unchanged verified target can be updated or receive evidence without new physical checks, notes, or date.

### Coverage limitation

The implementation is correct for the tested target-switch sequence, but the nine assertions do not cover every scenario requested by the brief. In particular, the fixture does not independently run all of:

- a literal Target A → Target B transition after first selecting Target A;
- target-profile switch with a different profile and invalidated setting;
- deleted target while the review remains open;
- return to a previously selected target after several transitions;
- full persisted localStorage before/after snapshots for every transition.

This is a Minor fixture-coverage gap, not an observed stale-metadata defect. The fixture names should not be read as proof of the complete transition matrix.

## Intentional metadata edits

The tested explicit edit path changes scope and clears the batch label after Target B synchronization. `capturePromotionForm()` marks those differences as selected, and the save applies only those intentional changes.

The current `metadataSelections` object has flags for:

- `label`;
- `measurementScope`;
- `batchLabel`;
- `status`.

It does not have separate `verifiedDate` or `preferred` dirty flags. Consequently, an explicit verified-date-only edit on an already-verified target is not independently represented as a metadata edit, and unchecking an already-preferred Update does not request a preferred-state clear because merge preserves preferred unless `session.preferred` is true. The requested target-switch scenarios leave those fields untouched and pass, but this is a Minor edge-case limitation for explicit metadata editing.

## Evidence-only and existing verified targets

Evidence-only target switching passed in the rendered fixture. Exactly one evidence record is appended to Target B; Target A is unchanged; target values, label, scope, batch, status, date, preferred state, and existing evidence remain intact.

`savePromotionTransaction()` now validates status rules only when the status is newly requested or explicitly changed. This prevents an unchanged verified target from being forced through a new physical-verification requirement. Explicitly changing a status still invokes `promotionStatusRules()`; invalid verified requests remain blocked.

## Grid missing-machine limitation

Source inspection confirms the current Test Grid creation form does not write a `machine` field. The Phase 7.3A selected-cell path does not call `wizardMachineLabel()` and does not infer a machine from labels, power, notes, Grid name, or Material profile.

The review warning is present exactly as required:

> Machine identity was not recorded for this Grid. Machine-dependent settings cannot be promoted. Evidence-only remains available.

A fresh browser workflow with a real machine-less Grid confirmed:

- warning displayed;
- Create with speed selected blocked at Save with a machine-incompatible selection error;
- Evidence-only remained available.

Seeded explicit-machine Grid fixtures also preserve the exact stored machine identity. Ordinary UI-created Grids remain generally limited to evidence-only or non-machine-dependent values until a separately reviewed Grid-machine enhancement. No Grid selector or schema change was made.

The separate pre-existing Grid → Library Material Test bridge still contains `pendingGridPromotion.machine = wizardMachineLabel()`. That is not the Phase 7.3A Production Setting promotion path; no current-preference substitution was found in `promoteSelectedGridCell()`.

## Regression of original audit findings

| Finding | Severity | Independent result |
|---|---:|---|
| Grid current-machine substitution | Blocker originally | Verified fixed; selected-cell path uses stored `grid.machine` only. |
| Crafted unknown/blocked selected fields | Major originally | Verified fixed; post-filter validation blocks unknown, malformed, unavailable, and machine-incompatible selections. |
| Unselected Update scope/batch mutation | Major originally | Verified fixed for ordinary and target-switched Update paths. |
| Weak Material Test physical heuristic | Major originally | Verified fixed; date/notes/winner alone do not establish physical completion. |
| Over-broad verification checks | Major originally | Verified fixed; source/operation-specific checks and strict notes/date rules apply. |
| Coupon identity collision | Major originally | Verified fixed in source and existing cross-machine canonical-identity fixture. |
| Incomplete Project promotion | Major originally | Verified fixed by pure eligibility gate and UI/handler gate. |
| Malformed requested status | Minor/Major originally | Verified fixed; active enum is strict and `superseded` is rejected. |
| Stale superseded metadata | Minor/Major originally | Verified fixed; active replacement with `supersededById` is rejected. |
| Duplicate evidence ID | Minor originally | Verified fixed for explicitly supplied duplicate IDs; existing-source warnings remain non-destructive. |
| Drawer Cabinet fit inference/collapse | Boundary | No Phase 7.3A fit values or geometry changes observed. |

## Source deletion resilience

The evidence design stores a cloned `sourceSnapshot`, and source adapters use cloned candidate/source data. The existing promotion fixture confirms a copied snapshot remains present, but it does not actually remove a Material Test, Project, Coupon session, or Grid/profile source and rerender the Production Setting evidence.

A full source-delete/re-render workflow was not independently completed in this pass. This remains a Minor verification gap. No source-live lookup was found in the evidence rendering path inspected.

## Backup, Replace, and Merge

Protected `backupObject()`, `replaceData()`, and `mergeData()` bodies remain unchanged. Existing fixture coverage confirms broad storage/import behavior and the promotion snapshot structure, but the current promotion-specific path does not perform a complete post-target-switch Create/Update/Evidence-only backup, Replace, and Merge round trip with deleted sources, unknown fields, preferred history, and supersede history.

This is a Minor end-to-end coverage gap, not observed import corruption.

## Rollback

The existing transaction seam uses cloned target profiles and `saveProfileRecord()` rollback. The promotion fixture exercises a false `persistFn` and confirms the complete target profile is restored. The current audit did not inject a real browser persistence failure after a target switch for every requested Update, Evidence-only, preferred-demotion, and supersede combination, nor compare all localStorage/session/source snapshots through the UI.

No partial mutation was observed in the available transaction fixture. Full browser-level failure injection remains a Minor verification gap.

## Fixture quality

The 51 promotion assertions consist of the original 42 assertions plus nine target-switch assertions. The nine new assertions are partially independent integration-style fixtures because they use the rendered form, actual event handlers, `capturePromotionForm()`, and Save transaction. They are not fully independent browser persistence tests.

Classification:

- Independent: pure status, eligibility, machine, identity, Project-gate, stale-supersede, and duplicate-ID assertions.
- Partially circular: ordinary candidate/save assertions and the nine rendered transition assertions, because they exercise implementation helpers and the transaction seam in the same page.
- Missing rather than overclaimed: complete target-profile invalidation, deleted-target, source-deletion/rerender, import round-trip, and browser failure-injection scenarios.

The earlier claims that the implementation passed its target-switch fixture are accurate for the nine implemented scenarios, but should not be expanded to mean the full requested transition matrix has been automated.

## Documentation accuracy

Verified current documentation:

- `README.md`: 51 promotion assertions and 1354 complete.
- Final cleanup report: 51/0 promotion and 1354/0 complete.
- Remediation report: corrected to 51/0 and 1354/0; it documents the 1429 figure as a prior overcount.
- Implementation report: historical 1329 result remains unchanged.
- Grid limitation and exact warning are documented.
- No current documentation claims ordinary UI-created Grids can create machine-specific Production Settings.
- Documentation does not claim promoted values guarantee fit or production safety.

## Protected-boundary comparison

Static comparison against `a9af90c` showed unchanged bodies for:

- `persist()`;
- `backupObject()`;
- `replaceData()`;
- `mergeData()`;
- `normalizeProductionEvidence()`;
- `normalizeProductionSetting()`;
- `normalizeProductionSettings()`;
- `buildDesignResult()`;
- `buildJointCouponModel()`;
- `buildDrawerCabinetModel()`;
- `buildDrawerCabinetDesignResult()`;
- `serializeDesignSvg()`;
- `downloadTextFile()`.

Source/schema inspection found no Material Test, Test Grid, or Project schema change; no Joint Fit Coupon geometry/model change; no Drawer Cabinet geometry or Finished Front View change; no Designs production-setting application change; and no SVG/LightBurn layer change.

No automatic promotion path was found. Promotion begins at explicit UI actions and remains session-only until `savePromotionTransaction()` is submitted.

## Findings

### Minor — Explicit verified-date/preferred edits lack independent dirty flags

- Function/path: `capturePromotionForm()`, `mergePromotionIntoSetting()`.
- Scenario: after synchronizing an already-verified/preferred target, explicitly change only verified date or uncheck preferred.
- Expected: the explicit edit is represented and saved.
- Observed: there is no `verifiedDate` or preferred dirty flag; verified-date-only changes are not applied, and preferred is only set when true rather than explicitly cleared.
- Consequence: narrow metadata-edit affordance gap; the required untouched-field target-switch behavior remains correct.
- Recommendation: add explicit `verifiedDate` and preferred metadata-selection tracking in a future minor cleanup, with focused fixtures.
- Current coverage: not covered by the 51 assertions; scope/batch explicit edits are covered.

### Minor — End-to-end source deletion/import/failure coverage remains incomplete

- Function/path: promotion fixture and existing backup/import/rollback seams.
- Scenario: delete original source, perform Replace/Merge after target-switch promotion, or inject browser persistence failure after target switching.
- Expected: complete independent UI-level proof of snapshot resilience, import preservation, and rollback.
- Observed: source snapshots and transaction rollback are tested at helper/transaction seams, but the full requested browser workflows are not all present.
- Consequence: lower confidence in those combinations, without observed corruption.
- Recommendation: add isolated browser tests for deletion/rerender, Replace/Merge, and failure injection.
- Current coverage: partial/circular for snapshot and rollback; absent for full UI combinations.

No Blocker or Major finding remains from the original audit or the target-switch review.

## Required final conclusion

**SAFE TO COMMIT AFTER MINOR CLEANUP**
