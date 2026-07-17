# Phase 7.3A Explicit Evidence Promotion Core

Date: 2026-07-16  
Baseline: `a9af90c` — `Add drawer cabinet finished front view`  
Final readiness: **READY FOR INDEPENDENT AUDIT**

## Scope and changed files

Implemented the approved Phase 7.3A core in the standalone offline app. No commit, push, staging, reset, clean, stash, delete, move, or schema migration was performed.

Changed files:

- `index.html`
- `README.md`
- `docs/PHASE7_3A_EVIDENCE_PROMOTION_IMPLEMENTATION_2026-07-16.md`

Existing unrelated untracked files and reports were preserved. The tracked diff is limited to the promotion core, its entry points/fixtures, and documentation totals/behavior notes.

## Helpers added

The new module-local promotion core includes:

- `promotionSessionDefaults()`
- `promotionClone()`
- `promotionMachineSnapshot()`
- `promotionCandidateBase()` / `promotionAddField()`
- `buildMaterialTestPromotionCandidate()`
- `buildGridPromotionCandidate()`
- `buildJointCouponPromotionCandidate()`
- `buildProjectPromotionCandidate()`
- `promotionEligibility()`
- `promotionFieldRows()`
- `promotionStatusRules()`
- `promotionEvidenceSnapshot()`
- `promotionSignature()`
- `mergePromotionIntoSetting()`
- `promotionSourceDuplicateWarnings()`
- `savePromotionTransaction()`
- review-panel rendering and binding helpers
- `runEvidencePromotionFixtures()`

Final setting/evidence IDs are allocated only inside the explicit Save transaction. Candidate adapters do not call `persist()`, mutate sources, or allocate record IDs.

## Existing functions modified

The following existing UI paths received entry-point bindings only:

- `materialTestsEditorHtml()` / `bindMaterialTestsEditor()` — eligible Material Tests expose **Promote to Production Setting**.
- `openCell()` — selected Grid cells expose **Promote selected result** while retaining the existing Save to library behavior.
- `projectCard()` / `projectBrowserDetailHtml()` and their binders — Projects expose **Add as Production evidence**.
- `designResultsHtml()` / `bindDesignPreviewActions()` — Joint Fit Coupon results expose **Promote physical winner**.
- self-test query dispatch — `promotion` runs the new group and `all` includes it.

No source Material Test, Test Grid, Project, or Joint Fit Coupon schema was changed.

## Session-only review behavior

Promotion state is held in module-local `promotionSession`, initialized from `promotionSessionDefaults()`. It is not part of `state`, `backupObject()`, localStorage, or any source record.

The review panel shows:

- source type, label, date, source ID/profile snapshot, material, operation, and exact machine;
- target Library profile by stable ID;
- Create, Update, or Evidence-only action;
- target Production Setting for Update/Evidence-only;
- status, measurement scope, batch label, label, preferred, optional supersede, date, and review notes;
- physical-check confirmation rows;
- source/current target values and replacement limitations for each compatible field.

Cancel discards the session. Opening the review, changing fields, changing target, and canceling do not write Library data, sources, state persistence, or backups.

## Source mapping

### Material Tests

Cut tests expose only structured compatible values: measured thickness when explicitly present, speed, power, passes, air assist, air pressure, focus/Z-offset when structured, and software. A physical dated run defaults to `tested`; incomplete/reference material becomes `estimated` or `untested`.

Kerf tests may expose compatible cut values and evidence. Signed LightBurn kerf is never inferred from a measured gap or fit result; it requires explicit entry/confirmation.

Engrave, photo, and interval tests are evidence-only in this phase and cannot populate irrelevant Cut fields.

### Test Grid cells

Only an explicitly opened/selected cell can be promoted. The review carries Grid ID/name, cell key, operation, material, thickness context, speed, power, passes, interval, air assist, rating, Best marker, and notes. Best/rating does not imply verification. A selected physical cut cell defaults to `tested`; non-Cut cells are evidence-only.

### Joint Fit Coupon

The current session requires the user to choose a physical winner from the candidate list; no value is preselected. The source snapshot includes a deterministic session ID, candidate list, result/draft snapshot, machine, date, and notes. The review can select joint clearance, measured thickness, cut settings, and signed kerf independently.

`fingerJointClearanceMm` and `lightBurn.kerfOffsetMm` remain separate. Selecting the winner never selects kerf, and selecting kerf never selects the winner. Coupon fit may become verified only with explicit `piecesFit`, date, and meaningful notes.

### Projects

Projects are evidence-first. The entry point offers structured settings actually present in the Project snapshot and never parses notes, photos, rating, or completion status for fit, assembly, drawer movement, or kerf direction. Generic and Drawer Cabinet Projects cannot become verified through this phase. Drawer Cabinet cabinet joint, drawer joint, and lateral fit promotion is deliberately not implemented.

## Status behavior

- `estimated`: reference, copied, inferred, or incomplete source.
- `tested`: physical test run and inspection without applicable final physical confirmation.
- `verified`: requires source eligibility, an applicable physical check, a verified date, and meaningful review notes.
- `untested`: manually entered or insufficiently evidenced source.
- `superseded`: existing retained-history status, not selectable as an active replacement.

Project candidates explicitly cannot be verified in Phase 7.3A. Appearance approval cannot verify cutting or mechanical fit. A selected Grid Best marker cannot verify a result.

## Target actions

### Create

Create requires at least one selected compatible value and an exact or explicitly labeled source machine. It creates a new setting and one self-contained evidence snapshot, does not prefer the record by default, and applies only selected values.

### Update

Update requires a stable target profile and setting ID. It preserves the target ID, unknown fields, unrelated evidence, identity, and unchecked target values. Selected incoming values win. Existing preferred state remains unless the user explicitly requests Preferred.

### Evidence-only

Evidence-only requires an explicit target setting and may select zero value fields. It appends one evidence snapshot without changing target values, status, preferred state, machine, or operation.

## Machine, profile, and measurement rules

Target profiles are selected by stable ID. The source profile is used as the default only when that exact profile still exists. There is no name matching, automatic profile creation, or current-profile substitution.

Source machine key and label are preserved exactly. Cross-machine machine-dependent fields—speed, power, passes, focus, kerf, and fit—are blocked. Measured thickness remains available with a warning. Missing machine identity cannot silently become 20W or 40W; Create blocks until an exact or explicit custom machine is recorded.

Measurement scope is explicit and retained as `sheet`, `batch`, `material-general`, or `unknown`. The workflow does not upgrade sheet evidence to material-general.

## Evidence and duplicate behavior

Each saved promotion appends unknown-field-safe metadata to the existing evidence record, including source date/status, deep-copied source snapshot, selected values, physical checks, and deterministic promotion signature. The saved target does not require the source to remain present.

Duplicate source/signature matches warn without silently blocking ordinary promotion. Evidence IDs remain unique and are generated at Save. The production normalizer already preserves unknown fields, so no schema-version change was needed.

## Preferred and supersede behavior

Preferred is off by default. Requesting Preferred identifies same-machine/same-operation conflicts and uses the existing explicit confirmation pattern. Declining demotion preserves the draft and saves non-preferred when the rest of the review is valid. Accepting demotes only the identified conflicts.

Supersede is optional and requires an active same-machine/same-operation replacement. Self, missing, cross-machine, cross-operation, already-superseded, and invalid replacements are rejected. Old settings and evidence remain stored. Preferred demotion and supersede changes are part of the same cloned transaction.

## Atomic rollback

`savePromotionTransaction()` resolves and validates source/target state first, clones the target profile and nested settings, applies selected fields, appends one evidence snapshot, applies explicit preferred/supersede changes, and calls the existing rollback-aware `saveProfileRecord()` seam once.

If persistence fails, the complete target profile is restored. No partial setting, evidence, preferred demotion, supersede change, or source mutation remains. The promotion session remains available to the review caller for retry.

## Fixture validation

The new `?selftest=promotion` group passed:

**26 passed / 0 failed**

It covers candidate creation, no premature IDs, exact machine identity, selected-field create/update, evidence-only zero-field save, unknown/unrelated evidence preservation, source snapshots, coupon winner/kerf separation, Grid limitations, Project fit exclusion, status gating, machine mismatch, target/profile validation, custom machines, duplicate warnings/signatures, and persistence rollback.

The final complete isolated Chromium run passed:

**1329 passed / 0 failed**

Key totals:

- Production settings: 66/0
- Evidence promotion: 26/0
- Designs production application: 118/0
- Designs geometry: 587/0
- Storage recovery: 8/0

Direct startup passed for `?selftest=promotion`, `?selftest=production`, `?selftest=design-production`, `?selftest=design`, and `?selftest=all`. No page errors were observed.

## Browser workflow checks

Using isolated bundled Chromium and direct `file://` pages:

- Material Test promotion review opened; localStorage was unchanged before Save; a real selected-speed promotion saved one Production Setting successfully.
- Selected Grid cell opened the new **Promote selected result** path and reached Promotion review.
- Project card opened **Add as Production evidence** and reached Promotion review.
- Joint Fit Coupon opened **Promote physical winner**, showed no preselected winner, accepted an explicit winner plus machine/notes, and reached Promotion review.

The full manual source-deletion, failure-injection, and backup/Replace/Merge browser workflows remain audit follow-ups. The focused fixture verifies rollback and snapshot/backup normalization behavior; existing production/storage fixture groups continue to cover the underlying import/merge/persistence paths.

## Protected-boundary review

Static function comparison against `a9af90c` reports these protected functions unchanged:

- `persist()`
- `replaceData()`
- `mergeData()`
- `normalizeProductionSetting()`
- `normalizeProductionSettings()`
- `buildDesignResult()`
- `buildJointCouponModel()`
- `buildDrawerCabinetModel()`
- `buildDrawerCabinetDesignResult()`
- `serializeDesignSvg()`
- `downloadTextFile()`

Also unchanged: `STORAGE_KEY`, `SCHEMA_VERSION`, source schemas, backup format, Designs production application, Drawer Cabinet geometry, Finished Front View, shared geometry, SVG serializers, and LightBurn layers. No automatic promotion path was added.

Static checks passed:

- `git diff --check`
- Python `html.parser` validation
- direct file startup and JavaScript runtime validation

## Unverified areas and audit handoff

The following should be covered by the required independent audit:

- final persisted Project evidence-only save in a seeded browser profile;
- final persisted Grid promotion with a selected target profile;
- final persisted Joint Coupon promotion with a selected target profile and explicit physical checks;
- simulated browser-level persistence failure while the modal is open;
- source deletion after a persisted promotion through the visible Library/Project UI;
- complete Export, Replace Import, and Merge Import manual round trips for newly promoted evidence;
- visual wording/accessibility review of the new promotion panel.

These are validation follow-ups, not known source or protected-boundary failures. The implementation is ready for the independent audit requested by the Phase 7.3 architecture.

## Final readiness

**READY FOR INDEPENDENT AUDIT**
