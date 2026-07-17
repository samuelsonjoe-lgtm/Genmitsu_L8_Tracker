# Phase 7.3A — Evidence Promotion Remediation
## Implementation and Validation Report — 2026-07-16

## Final readiness

**READY FOR INDEPENDENT RE-AUDIT**

The independent audit findings were remediated within the requested Phase 7.3A scope. The expanded promotion fixtures pass, the complete in-page fixture suite passes, and targeted isolated Chromium workflows confirm the corrected Grid machine provenance and Project completion gate. No commit or push was performed.

The implementation is ready for a fresh independent audit. It is not presented as a commit approval because the requested source-deletion/re-render, full Replace/Merge round trips, and persistence-failure browser injection workflows were not all independently exercised in this pass.

## Scope and preserved boundaries

Only the Phase 7.3A findings were addressed. The promotion architecture remains session-only until explicit Save; no automatic promotion or redesign was introduced.

Preserved:

- `STORAGE_KEY` and `SCHEMA_VERSION`.
- `persist()`, `backupObject()`, `replaceData()`, and `mergeData()`.
- Production evidence and setting normalizers.
- Material Test, Test Grid, and Project source schemas.
- Joint Fit Coupon geometry and result generation.
- Drawer Cabinet geometry and Finished Front View.
- Design production-setting application.
- SVG serializers, export behavior, and LightBurn layers.
- No external dependencies, storage migration, or new source schema.

## Files changed

- `index.html`
- `README.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_REMEDIATION_2026-07-16.md`

Unrelated modified and untracked files were preserved. No file was staged, committed, pushed, reset, cleaned, stashed, deleted, moved, or rewritten outside this scoped change and report.

## Functions added or modified

Added or materially extended:

- `promotionCanonicalize()`
- `promotionValidSelectedFields()`
- `projectPromotionEligibility()`
- `projectPromotionActionHtml()`
- `promotionSessionDefaults()` metadata-selection state

Modified:

- `promotionMachineSnapshot()`
- `promotionCandidateBase()` machine inference option
- `buildMaterialTestPromotionCandidate()`
- `buildGridPromotionCandidate()`
- `buildJointCouponPromotionCandidate()`
- `buildProjectPromotionCandidate()` vicinity for Project gating
- `promotionEligibility()`
- `promotionStatusRules()`
- `promotionSignature()`
- `mergePromotionIntoSetting()`
- `savePromotionTransaction()`
- `capturePromotionForm()`
- `openProjectPromotion()`
- Grid selected-cell promotion path
- Project card and browser-detail promotion entry points
- `productionSettingsCanSupersede()`
- `runEvidencePromotionFixtures()`

Protected persistence, import, normalization, geometry, and export helpers were not modified.

## Remediation details

### Grid source machine provenance

The selected-cell UI path no longer calls `wizardMachineLabel()` or any current global preference when building a promotion candidate. It carries the stored Grid `machine` field through the selected-cell snapshot and passes it to `buildGridPromotionCandidate()`.

Grid machine handling is now conservative:

- An explicitly stored `{ key, label }` is preserved.
- A custom stored machine remains custom with its exact label.
- A missing machine remains missing.
- Grid labels do not infer 20W or 40W keys.
- Create still requires an exact/explicit source machine.
- Evidence-only remains available when machine identity is missing.
- Machine-dependent Create/Update fields remain blocked without an exact compatible machine.
- Measured thickness remains the non-machine-dependent exception.

The current Test Grid creation UI does not record a machine field. Therefore ordinary UI-created Grids generally cannot create machine-specific Production Settings in this phase; they remain evidence-only or limited to non-machine-dependent values until a separately reviewed Grid-machine enhancement. The promotion review displays: “Machine identity was not recorded for this Grid. Machine-dependent settings cannot be promoted. Evidence-only remains available.”

The direct browser path was tested with a Grid recorded as L8 20W while the global preference was 40W. The review showed 20W and did not show 40W. The reverse behavior is covered by the same source-preserving implementation and the expanded candidate fixtures.

### Compatible selected-field validation

`promotionValidSelectedFields(candidate, session, target)` now derives the valid selection after checking the candidate field map, requested checkbox value, field value, machine compatibility, and action context. It reports:

- valid selected keys;
- unknown keys;
- blocked or machine-incompatible keys;
- malformed keys/values.

Both `promotionEligibility()` and `savePromotionTransaction()` use this validation. Create and Update require at least one valid compatible field. Evidence-only can save with zero selected value fields. Unknown, unavailable, malformed, and blocked selections no longer disappear silently and continue to Save.

The UI remains a convenience layer; Save-time validation is authoritative.

### Update context preservation

The session now carries explicit `metadataSelections` flags for label, measurement scope, batch label, and status. `capturePromotionForm()` marks a metadata value as selected only when it is explicitly changed relative to the target, while Create treats its reviewed metadata as intentional.

`mergePromotionIntoSetting()` now preserves, on ordinary Update:

- measurement scope;
- batch label;
- status and verified date;
- label;
- preferred state unless explicitly requested;
- unknown top-level and nested fields.

Explicit scope changes and intentional batch-label clearing still apply when their metadata selection is marked. Evidence-only does not mutate these target values.

### Conservative Material Test completion

Material Test candidates no longer infer physical completion from date, notes, result summary, rating, or winning values. A candidate is physically complete only when an existing structured source flag is explicitly true, such as `physicalRunCompleted`, `completed`, or an existing completed result-state marker; archived and failed tests remain excluded.

Without an explicit source completion signal, a Cut candidate defaults conservatively and requires a reviewed `cutCompleted` confirmation before `tested` may be saved. Kerf/non-Cut tests do not gain Cut verification from prose or a winner alone.

No Material Test schema was changed.

### Source- and operation-specific verification

`promotionStatusRules()` now validates the active status enum, check-key applicability, notes, date, candidate policy, and source/operation rule.

Rules implemented:

- Joint Fit Coupon verified requires an explicit winner, `piecesFit`, verified date, and meaningful notes.
- Cut Material Test and Grid verified require `cutCompleted`, verified date, Cut operation, and meaningful notes.
- Appearance-only checks cannot verify a Cut setting or mechanical fit.
- Assembly-only, drawer-movement-only, and coupon-only checks cannot verify a Cut setting.
- Project remains `canVerify: false` and evidence-first.
- Unknown physical-check keys are rejected.
- Whitespace-only notes are rejected for verification.
- `superseded`, arbitrary, empty, whitespace, and case-altered requested statuses are rejected rather than silently downgraded.

### Coupon source identity and signature

Joint Fit Coupon source IDs now use a versioned `coupon-session-v1-` namespace and a canonical identity containing the source kind, exact machine, profile/material context, nominal/measured thickness, stable candidate list, geometry inputs, operation, cut settings, signed kerf reference, result context, and source date.

`promotionCanonicalize()` sorts object keys recursively while preserving meaningful array order. `promotionSignature()` now includes canonical source identity/context, target profile and target setting IDs, action, valid selected fields and values, requested status, scope, and source date. The signature remains an evidence duplicate/provenance signature and is not used as a setting ID or evidence ID.

Identical inputs remain stable; different machine context produces a different coupon source ID; reordered selected-field keys produce the same signature.

### Project completion gate

`projectPromotionEligibility()` uses existing Project status/date/operation data without adding a completion schema. Valid existing finished-piece states (`kept`, `gifted`, `for-sale`, and `sold`) with a valid date and operation may expose the evidence-only action. Cancelled, failed, archived, unknown, missing-date, or missing-operation Projects are conservatively blocked.

Both Project card/detail rendering and `openProjectPromotion()` enforce the gate. A crafted handler call cannot bypass it. Project candidates remain evidence-first, do not infer fit fields, and do not parse notes for clearances.

### Supersede and duplicate safeguards

Supersede replacement validation now rejects an active replacement carrying stale `supersededById`, while retaining same-machine, same-operation, distinct-ID, active-replacement checks and no chain traversal.

An explicitly supplied duplicate evidence ID is rejected before mutation. Existing duplicate-source/signature detection remains warning-only and non-destructive; existing evidence is never silently replaced.

## Fixture changes and actual totals

`runEvidencePromotionFixtures()` was expanded from the previous 26 assertions to **51 passed / 0 failed**. Added focused assertions cover:

- exact Grid machine preservation;
- missing and custom Grid machines;
- unknown selected fields;
- zero post-filter valid Create/Update fields;
- evidence-only with zero value fields;
- Update scope preservation and explicit scope change;
- operation-specific verification and appearance rejection;
- whitespace-only verification notes;
- requested status validation;
- stable selected-field signature ordering;
- cross-machine coupon identity;
- incomplete Project gating;
- stale supersede metadata;
- duplicate evidence ID rejection.
- target-switch metadata synchronization, Update preservation, Evidence-only preservation, and explicit post-switch metadata edits through the rendered review form.

The complete direct-page fixture run reported:

- Promotion: **51 / 0**
- Production settings: **66 / 0**
- Designs production application: **118 / 0**
- Designs geometry: **587 / 0**
- Complete suite: **1354 / 0**

Other groups in the complete run also passed, including storage recovery (**8 / 0**); no page errors were observed.

## Browser validation performed

Using a fresh isolated Chromium context:

- `index.html?selftest=promotion`: loaded, **51 / 0**, no page errors.
- `index.html?selftest=all`: loaded, **1354 / 0**, no page errors.
- Grid UI path: seeded a Grid with stored 20W identity, set current preference to 40W, opened the Test Grid, opened a cell, selected “Promote selected result,” and confirmed the review displayed 20W and not 40W.
- Project UI path: seeded a completed `kept` Project and a cancelled Project; confirmed the completed Project had one promotion action, the cancelled Project had none, and the unavailable explanation was rendered.

Direct syntax validation also passed:

- `python -m html.parser index.html`
- `git diff --check`

## Protected-boundary comparison

Static comparison against baseline `a9af90c` confirmed the protected persistence, import, normalization, Design geometry/model, and export helper bodies remain unchanged. The remediation was kept in the Phase 7.3A candidate, validation, transaction, UI-entry, supersede, and fixture layers.

## Unverified or intentionally deferred areas

The following were not fully exercised end-to-end in this remediation pass and remain explicit independent re-audit targets:

- Actual browser source deletion followed by re-render for Material Test, Grid, Project, and Coupon evidence.
- Full backup, Replace import, and Merge import round trips for Create, Update, Evidence-only, preferred history, superseded history, unknown metadata, and absent sources.
- Browser-injected persistence failure during Create, Update, Evidence-only, preferred demotion, supersede, and combined preferred-plus-supersede transactions.
- Full visible Coupon final Save/verified workflow under multiple machine/material contexts.
- Reverse 40W Grid UI scenario and a live Grid with genuinely missing stored machine were not repeated after the warning addition; the missing-machine warning and Create block were directly exercised once.
- Full visible Material Test notes-only/reference/incomplete workflow.
- Full visible Project handler bypass attempt after an incomplete Project is rendered.

These are verification gaps, not claims of completion. They are the reason the handoff status is `READY FOR INDEPENDENT RE-AUDIT` rather than a stronger commit recommendation.

## Documentation result

`README.md` now records the corrected 1354/0 total and documents exact Grid provenance, the current UI-created Grid machine limitation and warning, conservative Material Test completion, source-specific verification, completed-Project gating, canonical coupon identity context, source snapshots, no automatic promotion, and deferred Drawer Cabinet fit promotion.

## Final status

**READY FOR INDEPENDENT RE-AUDIT**
