# Phase 7.3A Evidence Promotion Metadata Cleanup

Date: 2026-07-16  
Scope: final metadata-edit cleanup for the standalone offline app  
Status: **READY FOR FINAL FOCUSED AUDIT**

## Result

The final cleanup adds explicit Update handling for verified dates and Preferred state while preserving the Phase 7.3A append-only evidence and target-switch rules. Create, Update, and Evidence-only remain session-only until the existing explicit Save transaction.

The focused rendered-form and Save coverage now passes:

- Evidence Promotion fixtures: **58 passed / 0 failed**.
- Complete deduplicated fixture suite: **1361 passed / 0 failed**.
- New target-switch rendered-form checks: **16 passed / 0 failed**.

## Source changes

Only the requested implementation and documentation files were changed by this cleanup:

- `index.html`
  - `promotionSessionDefaults()` and `syncPromotionMetadataFromTarget()` now track `verifiedDate` and `preferred` as explicit metadata-selection fields.
  - `capturePromotionForm()` sets the Preferred form value before comparing it with the selected target, detects verified-date and Preferred changes, and suppresses both metadata edits for Evidence-only actions.
  - `promotionVerifiedDateRules()` enforces the verified/date pair for explicit date edits.
  - `mergePromotionIntoSetting()` applies a verified-date edit only when that metadata field changed and applies Preferred only when explicitly changed during Update.
  - `savePromotionTransaction()` validates explicit date edits without requiring new physical checks when the target remains verified, and invokes Preferred conflict handling only for an explicit request to mark the target Preferred.
  - `runPromotionTargetSwitchFixtures()` adds rendered-form Save paths for verified-date-only edits, Preferred true-to-false clearing, and accepted and declined Preferred conflicts.
- `README.md`
  - Updated the documented totals to 58 Evidence Promotion checks and 1361 complete checks.
  - Documented verified-date editing, Preferred clearing, and Evidence-only preservation.
- This report.

No prior Phase 7.3A report was modified. Unrelated dirty and untracked worktree items remain untouched.

## Behavioral guarantees checked

### Verified date

An Update can change only the verified date while the target remains `verified`. The Save path accepts that change without requiring a new physical confirmation. A verified target still requires a nonblank date, and a date cannot be saved with a non-verified status when the date is explicitly edited.

### Preferred state

An Update can explicitly clear a previously Preferred target. Clearing does not invoke the Preferred conflict prompt. Explicitly setting a target Preferred retains the existing same-machine conflict flow: accepting demotes the conflicting Preferred setting, while declining leaves the target non-Preferred and preserves the existing conflict.

### Evidence-only preservation

Evidence-only captures reset verified-date and Preferred metadata selections to false. The merge path therefore preserves both target fields, along with existing target context and unknown evidence fields, while appending the new evidence snapshot.

### Target switching

Switching to Target B reloads its label, scope, batch, status, verified date, and Preferred state, and clears all metadata-selection flags before rerender. The rendered-form fixtures then verify that unchecked Update metadata remains unchanged and that Target A is not mutated.

## Validation evidence

Executed from `C:\Genmitsu L8 Tracker`:

- `python -m html.parser index.html` — passed.
- `git diff --check` — passed.
- Isolated headless Chromium startup — loaded without application errors.
- `?selftest=all` — all fixture groups reported zero failures; aggregate was 1361 passed / 0 failed.
- Direct `window.runPromotionTargetSwitchFixtures()` — 16 passed / 0 failed.
- Protected-boundary body comparison against baseline commit `a9af90c` — unchanged:
  `persist`, `backupObject`, `replaceData`, `mergeData`, `normalizeProductionEvidence`, `normalizeProductionSetting`, `normalizeProductionSettings`, `buildDesignResult`, `buildJointCouponModel`, `buildDrawerCabinetModel`, `buildDrawerCabinetDesignResult`, `serializeDesignSvg`, and `downloadTextFile`.
- Offline dependency scan found no runtime `fetch`, `XMLHttpRequest`, or module import introduced by this cleanup. README contains only the existing official-source and repository links.

The browser run still emits the pre-existing Chromium SVG warning for the Finished Front View's `height="auto"` attribute during design/all startup; the relevant fixtures pass, and this cleanup did not change the SVG/geometry boundary.

## Intentionally unverified or unchanged

- No physical Grid machine was available for hardware validation; the existing explicit-machine and missing-machine limitation remains documented.
- Full browser deletion/import and storage-failure injection flows remain outside this focused metadata-edit cleanup.
- Schemas, storage, import/normalization, geometry, SVG serialization, downloads, and the Grid limitation were not changed.
- No commit or push was performed.

