# Phase 7.3A — Evidence Promotion Final Target-Switch Cleanup
## Cleanup and Validation Report — 2026-07-16

## Final readiness

**READY FOR FINAL RE-AUDIT**

This cleanup fixes stale promotion metadata when the effective target profile, action, or Production Setting changes; adds rendered-form transition and Save regression coverage; corrects the inaccurate complete-suite documentation total; and documents the current Test Grid machine limitation without adding a Grid selector or changing the Grid schema.

No commit or push was performed.

## Scope and files changed

Application/documentation changes in this pass:

- `index.html`
- `README.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_REMEDIATION_2026-07-16.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_FINAL_CLEANUP_2026-07-16.md`

Unrelated modified and untracked files were preserved. No reset, clean, stash, delete, move, stage, commit, or push operation was performed.

Protected source schemas, storage/import paths, normalization, Design geometry, Drawer Cabinet geometry/Finished Front View, SVG generation, and LightBurn behavior were not changed.

## Helpers and functions

Added:

- `syncPromotionMetadataFromTarget(session, targetSetting)`
- `runPromotionTargetSwitchFixtures()`

Modified:

- `renderPromotionReview()` — adds the missing-Grid-machine warning and transition wiring.
- `capturePromotionForm()` — continues to capture explicit edits, now after target state is synchronized.
- Promotion action-change handler.
- Target-profile change handler.
- Target-setting change handler.
- `startPromotion()` — synchronizes Evidence-only startup to its selected target.
- `savePromotionTransaction()` — validates status only when it is newly requested or explicitly changed; unchanged verified target status is preserved without requiring a new verification check.
- `runEvidencePromotionFixtures()` — incorporates the rendered transition fixture results.
- `README.md` and the remediation report totals/limitation wording.

## Target-switch correction

`syncPromotionMetadataFromTarget()` safely clones the session and resets metadata selection flags. For a selected Update or Evidence-only target it loads:

- setting label;
- measurement scope;
- batch label;
- verification status;
- verified date;
- preferred display state.

For no effective target, it resets the review metadata to conservative candidate defaults without changing the source candidate or selected compatible value fields.

The helper is used for all relevant transitions:

- target Production Setting change;
- target Library profile change, which clears the now-invalid setting target;
- Create → Update;
- Create → Evidence-only;
- Update ↔ Evidence-only;
- returning to a previously selected target.

The transition sequence is now:

1. Capture any edits made to the currently displayed form.
2. Apply the new action/profile/setting identity.
3. Load metadata from the effective target, if one exists.
4. Clear metadata-selection flags for the newly loaded values.
5. Rerender the review form.

An intentional edit after the switch is captured normally and sets the corresponding metadata-selection flag. Metadata from Target A is not carried into Target B.

Unchanged verified statuses on Update/Evidence-only are not revalidated as new verification requests. Create and explicit status changes still run the strict status/physical-check rules.

## Grid machine limitation and warning

This cleanup does not add a Test Grid machine selector and does not change the stored Grid schema.

Phase 7.3A no longer substitutes the current global machine. Machine-dependent Grid promotion requires a Grid source that actually contains an explicit machine identity. Test Grids created through the current UI do not record that identity, so those results are generally limited to Evidence-only or non-machine-dependent values until a separately reviewed Grid-machine enhancement exists.

When the selected Grid lacks a machine, the review displays exactly:

> Machine identity was not recorded for this Grid. Machine-dependent settings cannot be promoted. Evidence-only remains available.

Create/Update machine-dependent selections are blocked at Save; Evidence-only remains available. No machine is inferred from the current preference, Grid name, Material profile, notes, or power values.

The pre-existing Grid → Library Material Test bridge still uses its own pending-draft machine behavior; this cleanup does not broaden or rewrite that separate path.

## Regression fixtures

`runPromotionTargetSwitchFixtures()` uses the actual rendered promotion form and event handlers. It does not only construct final session objects. It exercises:

- Create → Update;
- target A → target B;
- rendered Target B label, scope, batch, status, verified date, and preferred state;
- metadata-selection reset after switching;
- selecting only speed and saving Update;
- Target B metadata/evidence/unknown-field preservation;
- Target A unchanged after Update;
- Update → Evidence-only on the same target;
- Evidence-only append-only preservation for Target B;
- Target A unchanged after Evidence-only;
- explicit scope change and intentional batch-label clearing after switching;
- preservation of unedited label/status/verified-date values.

The helper adds **9** promotion assertions. Promotion coverage therefore increased from **42** to **51 passed / 0 failed**. The assertions are integration-style fixture coverage for the in-page form/session transition, not full browser automation; the visible Chromium workflows below provide the separate runtime confirmation.

## Actual validation results

Fresh isolated direct-file Chromium runs:

- `index.html?selftest=promotion`: **51 passed / 0 failed**, no page errors.
- `index.html?selftest=production`: **66 passed / 0 failed**, no page errors.
- `index.html?selftest=design-production`: **118 passed / 0 failed**, no page errors.
- `index.html?selftest=design`: **587 passed / 0 failed**, no page errors.
- `index.html?selftest=all`: loaded and completed with no page errors; the corrected top-level suite arithmetic is **1354 passed / 0 failed**.
- Storage recovery remains **8 passed / 0 failed** within the complete run.

The complete total is reconciled from the independently observed pre-cleanup total of **1345** plus the **9** new target-switch assertions: **1345 + 9 = 1354**. The earlier remediation report’s **1429** claim was an overcount caused by summing nested/chained fixture console groups; it is now corrected to 1354 in `README.md` and the remediation report. The earlier implementation report’s historical **1329** result remains unchanged.

Static/syntax checks:

- `python -m html.parser index.html`: passed.
- `git diff --check`: passed.
- Protected-function comparison against `a9af90c`: all inspected protected functions unchanged.

Protected comparison covered `persist`, `backupObject`, `replaceData`, `mergeData`, `normalizeProductionEvidence`, `normalizeProductionSetting`, `normalizeProductionSettings`, `buildDesignResult`, `buildJointCouponModel`, `buildDrawerCabinetModel`, `buildDrawerCabinetDesignResult`, `serializeDesignSvg`, and `downloadTextFile`.

## Browser workflows performed

### Target-switch form/session workflow

The fresh Chromium promotion fixture opened a real review form, changed Create to Update, changed the selected Production Setting from Target A to Target B, inspected the rerendered controls, selected only speed, captured the form, and saved the transaction. Target B retained its scope, batch, label, verified status/date, preferred state, unknown field, and existing evidence; Target A remained unchanged.

The same rendered path changed Update to Evidence-only, switched to Target B, saved, and confirmed exactly one evidence append on Target B with both target records otherwise preserved.

The same path then explicitly changed scope and cleared the batch label after switching. Those intentional changes applied while unedited label/status/date values remained unchanged.

### Missing-machine Grid workflow

A fresh Chromium context opened a real Grid created without a machine, opened a selected cell, and opened Promotion review. The exact missing-machine warning was present. Selecting a machine-dependent speed field and attempting Create produced a Save-time block:

> Select at least one compatible value before saving.
> Blocked or machine-incompatible promotion field(s): speedMmPerMin.

Switching to Evidence-only remained available and displayed the zero-value-field Evidence-only guidance.

## Documentation result

`README.md` now reports:

- Promotion: **51 / 0**.
- Complete suite: **1354 / 0**.
- Target-switch metadata synchronization.
- The current UI-created Grid machine limitation and exact warning behavior.
- No current-machine substitution or inferred Grid machine.
- Evidence-only/non-machine-dependent limitation for machine-less Grids.

The prior remediation report now reports **51 / 0** promotion and **1354 / 0** complete, and no longer claims 1429.

## Unverified areas

The following remain suitable for final independent re-audit rather than being claimed as fully covered here:

- Full browser automation across every action/profile transition with persisted storage snapshots before and after.
- A reverse 40W stored-machine Grid UI scenario; the source-level behavior remains covered, but the current creation UI cannot create stored machine identity.
- Full Replace/Merge import round trips after target-switch promotions.
- Browser-injected persistence failure during each target-switch Save variant.
- Source deletion followed by rerender of every promoted evidence source.
- Full visible Coupon final Save/verified workflow under multiple contexts.

These are remaining depth/automation items, not observed target-switch metadata failures.

## Final status

**READY FOR FINAL RE-AUDIT**
