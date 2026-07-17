# Phase 7.3 Evidence Promotion Architecture Review

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline requested: `a9af90c` — `Add drawer cabinet finished front view`  
Review type: read-only architecture and implementation planning

## 1. Executive recommendation

**READY AFTER DESIGN DECISION**

The existing architecture can support Phase 7.3 without changing `STORAGE_KEY`, `SCHEMA_VERSION`, backup shape, shared geometry, SVG serialization, or the Designs production-setting application path. The safest implementation is a session-only promotion review workflow that creates a pure, normalized candidate and persists only after explicit field selection and confirmation.

Implementation should not begin until the product decisions in Sections 4, 5, and 8 are accepted, especially:

1. how Drawer Cabinet `cabinetJointClearance` and `drawerJointClearance` map to the single persisted `fingerJointClearanceMm` field;
2. whether a Joint Fit Coupon session result is promoted as a fully snapshotted manual source or deferred until coupon results have a saved record;
3. whether completed Projects may promote only cut settings/evidence or may also enter explicit fit outcomes;
4. whether `verified` is permitted only after the review records an applicable physical confirmation and date.

The current baseline is healthy: `git status -sb` shows the branch aligned with `origin/main` with only unrelated untracked artifacts, `git log -1 --oneline` reports `a9af90c`, and `git diff --check` passes. The current isolated Chromium fixture run reports **1303 passed / 0 failed**, including 66 production-setting checks, 118 Designs-application checks, 587 Designs-geometry checks, and 8 storage-recovery checks.

## 2. Current source and target data architecture

### Persistence and ownership

The app stores all live data under `STORAGE_KEY = 'genmitsu-l8-tracker-v1'` with `SCHEMA_VERSION = 2` (`index.html:262-263`). Library profiles are nested in `state.profiles`; each profile owns `tests` and `productionSettings` arrays. `persist()` serializes the top-level arrays and preferences (`index.html:468-477`), while `backupObject()` includes profiles and their nested production records (`index.html:514+`).

This ownership model is suitable for promotion: a promoted record belongs to an explicitly chosen target Library profile, while its evidence contains copied source snapshots rather than a required live relationship. A top-level promotion collection is not needed and would introduce unnecessary orphan, merge, and deletion handling.

### Production Setting model

`normalizeProductionSetting()` currently preserves unknown top-level and nested fields while normalizing machine, material condition, cut settings, LightBurn kerf, fit settings, verification status, preferred state, dates, evidence, and supersession (`index.html:5076-5147`). The existing target model already has:

- exact machine key/label;
- nominal and measured thickness in millimeters;
- measurement scope, date, batch, supplier, and product snapshots;
- speed, power range, passes, air assist, air pressure, focus, and software;
- signed LightBurn kerf;
- general finger-joint clearance and Drawer Cabinet lateral clearance;
- `untested`, `estimated`, `tested`, `verified`, and `superseded` states;
- preferred and superseded history;
- nested evidence records.

The current production editor is cut-only (`index.html:6006-6011`) and its evidence editor stores only kind, date, source ID/profile ID, source label, result, and notes (`index.html:5935-5966`). Unknown evidence fields survive normalization, so additive snapshot metadata is possible without a schema version change.

### Existing Library lifecycle

Production settings are edited in a draft array and committed with the parent material profile. `openProfile()` loads draft tests/settings, `openProductionSetting()` edits only the draft, and `saveProfileRecord()` rolls the profile back if `persist()` fails (`index.html:7881-7895`, `index.html:7897-7925`). Preferred conflicts are explicitly confirmed and only same-operation/same-machine records are considered (`index.html:5877-5907`). Supersede is explicit, preserves the old record/evidence, and requires the same machine and operation (`index.html:5909-5922`, `index.html:6065-6079`).

These are good primitives but are not sufficient by themselves for promotion because they do not provide source-specific eligibility, selected-field merge semantics, immutable source snapshots, promotion duplicate detection, or a multi-record atomic transaction.

## 3. Promotion candidate architecture

Use three layers:

1. **Pure source adapters** convert one source type into a reviewable candidate. They must not mutate source objects, Library profiles, `state`, or storage.
2. **Session-only promotion draft** holds source snapshot, target profile ID, create/update/evidence-only choice, selected fields, proposed values, status review, preferred/supersede choice, and duplicate warnings.
3. **One explicit save transaction** merges the reviewed candidate into a target profile and appends evidence. Only this transaction may call the existing Library save/persist path.

Recommended responsibilities, following existing naming conventions:

```javascript
promotionSessionDefaults()
buildPromotionCandidate(sourceContext, sourceType)
promotionEligibility(candidate, targetProfile, targetSetting)
promotionFieldRows(candidate, target)
promotionEvidenceSnapshot(candidate, selectedFields)
promotionStatusRules(candidate, review)
promotionSignature(candidate, selectedFields)
mergePromotionIntoSetting(target, candidate, selectedFields)
savePromotionTransaction(draft)
```

`buildPromotionCandidate()` should return a plain object containing source type/ID/label/date, source profile ID, source snapshots, machine/material/operation context, available fields, limitations, default status, and an evidence snapshot. It should not return a live object reference to the source.

The session state should be module-local, analogous to `materialTestWizard`, `projectWizard`, and `designProductionSession`. It must be cleared on cancel, reset, template/tab changes where appropriate, and reload. It must never be added to `state`, `backupObject()`, or source records.

The candidate builder should be reusable and pure; a modal/panel should only render its result and collect explicit choices. A direct modal that writes a new Production Setting while the user is reviewing would violate the required cancel/inertness boundary.

## 4. Source-specific eligibility and mapping

### Material Test

Material Tests are the strongest immediately available source for a tested laser run but are not automatically production-proven. The source record has machine, date, test type, status, winning speed/power/interval/passes, air assist, confidence, result text, and intended operation (`index.html:5016-5038`). The Wizard records the setup and result, then optionally changes primary Library fields only when the user checks Apply Winner (`index.html:7198-7223`).

Recommended mapping:

| Source condition | Available promotion | Default status |
| --- | --- | --- |
| `testType: cut`, intended operation cut | measured thickness, cut speed/power/passes/air, focus/Z-offset only if present in a retained source field, notes/date/machine | `tested` |
| `testType: kerf` | cut settings and/or evidence; signed kerf only if the user explicitly enters/confirms its LightBurn meaning | `tested` |
| `testType: engrave_matrix`, `photo`, or `interval` | evidence-only, or a future operation-specific target; never silently populate a Cut Production Setting | `tested` |
| manufacturer/reference or incomplete manual source | evidence-only or explicitly selected values | `estimated` or `untested` |

The existing Material Test schema does not consistently retain every requested field as structured data. Focus/Z-offset and some setup fields are often embedded in notes, and line interval is meaningful only for applicable test types. The review must show unavailable fields as unavailable, not parse prose heuristically.

### Test Grid winner

Grid cells are observations with rating, `best`, and notes. `buildGridCellMaterialTestRecord()` copies a selected cell into a Material Test with stable `sourceGridId` and `sourceCellKey` (`index.html:5167-5179`). The cell's `best` flag and rating are evidence that the user selected a candidate, not physical assembly proof.

Eligibility requires an explicit selected cell. A grid with no cell or only a selected cell lacking an explicit review cannot create a production candidate. The review must show grid ID/name, cell key, operation, machine, material, thickness, winning speed/power, interval, passes, air assist, rating, notes, and any limitation.

The user may choose `estimated`, `tested`, or `verified` in the review, but `verified` must be blocked unless applicable physical confirmation is recorded. A winning cell alone defaults to `tested` when the grid was physically run and inspected; it never auto-verifies.

### Joint Fit Coupon

The current generator is a Designs/session feature. Its candidate list and selected physical winner are not currently a persisted source record with a stable result ID. Therefore a direct promotion entry point cannot safely claim a saved coupon source unless Phase 7.3 adds an explicit session snapshot.

Recommended decision: allow **Promote physical winner** from the current coupon result only if the review captures a complete immutable snapshot containing coupon source ID, material, measured thickness, machine, cut settings, candidate list, selected clearance, date, and notes. The user must explicitly identify the physical winner. Otherwise defer the entry point until coupon results become saved records.

Mapping rules:

- joint clearance may be selected independently;
- cut settings may be selected independently;
- measured thickness may be selected independently;
- signed LightBurn kerf may be selected independently only as a reference value;
- joint clearance and signed kerf remain separate fields;
- no candidate value is promoted merely because it appears in the list;
- no kerf sign is inferred from fit success.

### Drawer Cabinet Project

Projects persist material, thickness, job type, date, rating, notes, source IDs, photos, and a settings snapshot (`index.html:8062-8084`). They do not currently have structured cabinet-joint, drawer-joint, lateral-clearance, dry-fit, movement, or glued-assembly result fields.

A generic completed Project is therefore not sufficient to infer Drawer Cabinet fit. Phase 7.3 should either add explicit promotion-review inputs for those physical outcomes or restrict the Project entry point to values actually present in the project snapshot. It must not infer assembly success from `status`, `rating`, photos, or project completion.

For Drawer Cabinet-specific fields:

- measured thickness and cut settings may be promoted when present;
- Drawer lateral clearance may be promoted independently only from an explicit source value;
- cabinet and drawer joint clearances must remain separate during review;
- because the target currently has only `fingerJointClearanceMm`, do not silently choose or average the two values;
- recommended Phase 7.3 behavior is to offer “general finger-joint clearance source” with an explicit Cabinet or Drawer choice, while showing the other value as not selected;
- if the values differ, saving both into one target field must be blocked;
- a future schema extension for separate cabinet/drawer production values is preferable if both need to be retained as production data.

### Other completed Projects

Generic Projects may provide a material, thickness, machine-independent cut snapshot, date, notes, rating, and accounting data. They do not prove fit or machine compatibility. The conservative entry point is **Add as Production evidence**, with cut-setting fields available only when the Project is a cut project and its snapshot is complete. Fit controls, signed kerf, and verification must remain unavailable unless explicit applicable evidence is collected in the review.

## 5. Status and verification rules

Use the existing status enum, but make transitions explicit:

- **Estimated:** manufacturer guidance, copied/reference data, inference, incomplete physical evidence, or a source without a completed run.
- **Tested:** a physical laser run was completed and inspected, but final assembly or applicable outcome was not documented.
- **Verified:** the user explicitly confirms an applicable physical outcome, records what was checked, and records a verified date.
- **Untested:** manually entered or incomplete information without a physical-run claim.
- **Superseded:** historical record retained after explicit replacement.

Verification must be contextual, not a generic checkbox. The review should ask **What was physically verified?** and store additive evidence metadata such as:

```javascript
physicalChecks: {
  cutCompleted: false,
  piecesFit: false,
  assemblyCompleted: false,
  drawerMovedFreely: false,
  appearanceApproved: false
}
```

These checks should be interpreted by source type. For a Joint Fit Coupon, `piecesFit` can support verification of the selected joint clearance. For a Drawer Cabinet, `assemblyCompleted` and/or `drawerMovedFreely` can support applicable fit verification. `appearanceApproved` must not verify cutting or assembly. A verified date and notes are required; source labels, confidence, or a “winner” label alone cannot upgrade status.

If the user selects `verified` without required checks/date, block Save with a precise message. Do not silently downgrade to `tested`; the user should correct the review.

## 6. Target profile and machine rules

Target selection must use stable Library profile IDs, never material names. The default target may be the source's existing profile when its ID exists, but the user must be able to select a different existing profile explicitly. Creating a new Library profile should remain a separate action, not an implicit side effect of promotion.

If the source profile is deleted, retain the source material/profile label and snapshot in the candidate and require an explicit target. Duplicate display names must remain distinguishable by ID and context.

Machine identity must be copied exactly from the source. No 20W/40W scaling, power translation, or current-machine substitution is allowed. If source and target machine keys differ:

- block machine-dependent cut, fit, and kerf selections;
- allow measured thickness or evidence-only promotion with a visible warning where appropriate;
- require a separate machine-specific target for machine-dependent values.

Custom machines require a nonblank source snapshot label. Missing or free-text-only machine identity should not be silently normalized to the current global `state.machineProfile`.

Measurement scope must be explicit: `sheet`, `batch`, `material-general`, or `unknown`. Defaults may be suggested from the source, but the review must show the scope and never upgrade sheet evidence to material-general.

## 7. Create/update/evidence-only behavior

The review must present three explicit target actions:

1. create a new Production Setting;
2. update an existing Production Setting;
3. add evidence only.

An optional separate supersede action may be selected, but ordinary updates must not require supersession. A candidate with zero selected fields must not save a Production Setting. Evidence-only promotion may still save one evidence record when the user explicitly chooses that action and confirms the evidence snapshot.

For create:

- generate a new stable record ID and evidence ID at save time;
- preserve all source snapshots;
- do not mark preferred by default;
- default status conservatively and require explicit confirmation for stronger status;
- do not copy irrelevant operation fields into a cut setting.

For update:

- preserve the target ID, unknown fields, unrelated evidence, and unchecked values;
- merge only selected fields;
- incoming selected values win;
- show source/current/proposed values and every changed field before save;
- preserve target machine and operation unless the review explicitly creates a compatible replacement rather than mutating identity.

For evidence-only:

- append one immutable snapshot evidence record;
- do not change target values, preferred state, status, or source object;
- permit this path for non-cut Material Tests, incomplete Projects, and coupon observations that are useful but not target-compatible.

## 8. Preferred and supersede behavior

The existing preferred helper already requires confirmation before demoting same-machine/same-operation conflicts (`index.html:5895-5907`). Phase 7.3 must make the conflict review richer: show record ID/label, machine, operation, measurement scope, batch, status, and evidence date. Declining demotion must not discard the promotion draft. The recommended behavior is to continue saving the reviewed record as non-preferred after a declined preferred change, provided all other validation succeeds.

Preferred selection must never silently demote an existing record and must not cross machine boundaries.

Supersede should remain optional and explicit. Require:

- distinct old and replacement IDs;
- same machine and operation;
- replacement not superseded;
- no cross-machine supersede;
- old record retained with evidence;
- `supersededById` set on the old record;
- old preferred cleared;
- no automatic traversal of prior supersede chains.

The current helper enforces same machine/operation and rejects self/missing/superseded replacements, but Phase 7.3 should add explicit UI review and a fixture for replacement records that already carry stale chain metadata. Supersede and preferred demotion must be included in the same rollback transaction as the target save.

## 9. Evidence snapshot strategy

Existing evidence is additive and unknown-field-safe, but its current structured fields are not enough for Phase 7.3 to make the evidence readable after source deletion. Additive metadata is recommended rather than a schema version bump:

```javascript
{
  id,
  kind,
  sourceId,
  sourceProfileId,
  sourceLabel,
  sourceDate,
  createdDate,
  sourceStatus,
  sourceSnapshot: {
    machine,
    material,
    thickness,
    operation,
    settings,
    result,
    notes
  },
  selectedValues,
  physicalChecks,
  promotionSignature,
  result,
  notes
}
```

The snapshot must contain copies, not live references. Source IDs are useful for display and duplicate warnings but are never required for reading the evidence. Deleting a Material Test, Grid, Coupon session, Project, or source profile must leave the target record and snapshot readable. A target Library profile deletion naturally deletes records owned by that profile; the workflow must not promise otherwise for the target owner.

## 10. Duplicate detection

Add a stable, deterministic promotion/source signature over source kind/ID, source profile ID, target profile ID, selected fields, source revision/date, and normalized selected values. The signature is warning metadata, not record identity.

Warn for:

- exact same source promoted twice to the same target;
- same source promoted into a different target profile;
- same source evidence attached twice;
- changed source promoted again;
- a matching production-setting ID on import;
- an older result promoted over a newer target;
- promotion into an existing preferred record.

Do not silently block ordinary duplicates. An exact duplicate evidence ID should be rejected or ignored safely because it would violate evidence identity; the UI should explain the reason without deleting or replacing existing evidence.

## 11. Session and persistence boundaries

Opening promotion, selecting fields, changing target, reviewing status, canceling, and closing the modal must not change `state.profiles`, source records, Library drafts, or localStorage. The candidate should remain in a module-local session object only.

At Save, use a transaction over a cloned target profile/list:

1. build and validate the complete next profile in memory;
2. apply only selected target values;
3. append snapshot evidence;
4. apply explicit preferred/supersede changes;
5. call the existing profile save/persist seam once;
6. restore the full previous target/list if persistence fails.

No source `promoted` flag is required. Do not edit Material Tests, Grids, Coupons, Projects, or source profiles as a side effect. Existing backup/import paths should retain the final nested record through current `backupObject()`, `replaceData()`, and `mergeData()` behavior; the promotion session itself must be absent.

## 12. Failure and rollback behavior

`saveProfileRecord()` already demonstrates list-level rollback when `persist()` fails (`index.html:7889-7895`). Phase 7.3 must extend that principle to every affected production record:

- target setting create/update rolls back;
- appended evidence rolls back;
- preferred demotion rolls back;
- supersede status/link rolls back;
- source records remain byte-identical;
- localStorage remains at the prior valid value if the write fails;
- the modal remains open with the review draft recoverable where practical.

Do not mutate the live target and then attempt to reconstruct it field by field. Clone the profile and nested arrays, validate the clone, and commit through one rollback-aware operation. Add failure injection fixtures for create, update, evidence-only, preferred conflict, and supersede paths.

## 13. UI entry points

Recommended conservative entry points:

- completed Material Test: **Promote to Production Setting**;
- reviewed Test Grid cell: **Promote selected result**;
- Joint Fit Coupon: **Promote physical winner** only after the session snapshot decision is accepted;
- completed Project: **Add as Production evidence**, with a Production Setting path only when compatible cut values are present.

Do not add controls to every cell, every Project card, or every Designs screen. Designs should remain a consumer of saved settings, not a promotion source. A source entry point should explain why the source is eligible and what remains unverified before opening the review.

The review panel should show Source, Target, field-selection rows, limitations, status questions, duplicate warnings, and a complete changed-field summary. Every row should show source value, current target value, proposed value, replacement behavior, and confidence/limitation text. Save is disabled for zero selected fields and for unresolved machine/status/target conflicts.

## 14. Fixture plan

Add a separate `runEvidencePromotionFixtures()` group and include it in the complete suite. Keep all current groups unchanged; the current baseline is 1303/0.

### Candidate creation

- Material Test cut, kerf, engrave, interval, incomplete, and missing source;
- Test Grid selected cell, unselected cell, physical-review notes, and missing Grid;
- Joint Coupon explicit physical winner, no winner, signed kerf, and session snapshot;
- Drawer Cabinet Project with equal/different joint values, lateral value, and explicit movement/assembly checks;
- generic Project with only settings and no fit evidence;
- every candidate leaves source objects and storage unchanged;
- candidate remains session-only until Save.

### Field selection and mapping

- zero selected fields saves nothing;
- one and multiple selected fields change exactly those fields;
- unchecked target values and unknown fields survive;
- cut-only mapping excludes engraving/interval fields;
- signed zero and negative values survive;
- joint clearance and kerf remain distinct;
- Cabinet and Drawer values never collapse silently;
- lateral clearance is independent;
- malformed or unavailable values cannot be selected.

### Status and physical evidence

- manufacturer/reference defaults estimated;
- physical run defaults tested;
- selected Grid winner never auto-verifies;
- physical assembly plus explicit confirmation can verify;
- verified requires applicable checks, notes, and date;
- status never upgrades from label, rating, confidence, or source kind alone;
- appearance approval does not verify cut or fit.

### Machine, profile, and scope

- exact 20W, exact 40W, custom, missing, and cross-machine cases;
- no scaling;
- fit/kerf/cut values blocked on cross-machine targets;
- measured thickness remains available with warning;
- stable profile ID targeting with duplicate names;
- deleted source profile requires explicit target;
- deleted target while modal is open blocks Save;
- sheet, batch, material-general, and unknown scopes remain visible.

### Create/update/evidence-only

- create new record with stable IDs;
- update preserves unrelated values, evidence, and unknown fields;
- evidence-only changes neither target values nor status;
- selected direct conflicts use incoming values;
- unchecked direct conflicts retain target values;
- source snapshots remain readable after source deletion;
- backup/replace/merge round trips preserve nested records.

### Preferred, supersede, and duplicates

- save non-preferred;
- preferred with no conflict;
- conflict accepted and declined;
- declined demotion still permits non-preferred save;
- same-machine valid supersede;
- cross-machine, self, missing, superseded, and chain-follow attempts blocked;
- old record/evidence retained;
- exact duplicate warning is non-destructive;
- changed source produces a different signature/warning.

### Failure and regression

- persist failure on create, update, evidence-only, preferred demotion, and supersede;
- no partial evidence or source mutation;
- current complete 1303/0 baseline retained;
- Designs production 118/0, production settings 66/0, storage 8/0, Material Tests, Test Grid, Projects, backup/import, and Drawer Cabinet Finished Front fixtures retained.

## 15. Manual browser plan

Use direct `file://` workflows in a fresh browser profile:

1. Open a completed cut Material Test and open promotion; confirm Library/storage are unchanged.
2. Select measured thickness and cut fields only; create a tested, non-preferred record.
3. Inspect evidence snapshots and delete the source test; confirm the promoted record remains readable.
4. Select a Test Grid cell, confirm its rating/best flag is not treated as verified, and review the physical-result status.
5. Open a Joint Fit Coupon, identify a physical winner explicitly, promote joint clearance without kerf, then promote kerf separately and confirm the fields remain distinct.
6. Use a one- or two-row Drawer Cabinet Project with differing cabinet/drawer clearances; confirm no silent collapse and promote lateral clearance independently.
7. Add a generic Project as evidence only and confirm absent fit fields are not offered.
8. Create a preferred conflict, decline demotion, and confirm the reviewed record can still save non-preferred.
9. Repeat with accepted demotion and then explicit same-machine supersede; confirm old records and evidence remain.
10. Simulate persistence failure for create/update/supersede and confirm full rollback with the review still available.
11. Export, replace-import, and merge-import the result; confirm snapshots, unknown fields, preferred state, and supersede history survive.
12. Run `?selftest=promotion` and `?selftest=all`.

## 16. Expected files and functions affected

### Expected files

- `index.html`: candidate adapters, session review UI, transaction helpers, source entry bindings, and fixtures.
- `README.md`: update current fixture totals and promotion wording only after implementation validation. It currently contains a stale older suite statement at line 73 (`1275/559`) alongside the newer Drawer Front note at line 79 (`1303/587`); this is documentation debt, not a reason to edit during this read-only review.
- `docs/PHASE7_3_EVIDENCE_PROMOTION_ARCHITECTURE_REVIEW_2026-07-16.md`: this report.

No new runtime dependency or file is required.

### Existing functions likely to be reused or minimally modified

- `normalizeProductionEvidence()` / `normalizeProductionSetting()` for additive unknown-field-safe normalization;
- `buildLibraryProfileRecord()` and `saveProfileRecord()` for parent ownership and rollback;
- `upsertProductionSettingDraft()` for preferred conflict primitives, after promotion-specific validation;
- `supersedeProductionSettingDraft()` for explicit same-machine history;
- `backupObject()`, `replaceData()`, `mergeData()`, and `persist()` only for verification and transaction integration;
- source render/bind functions for conservative entry points;
- existing `materialTestLabel()`, machine labels, status labels, and evidence display helpers.

### New helpers recommended

```javascript
promotionSessionDefaults
promotionSourceSnapshot
buildPromotionCandidate
buildMaterialTestPromotionCandidate
buildGridPromotionCandidate
buildJointCouponPromotionCandidate
buildProjectPromotionCandidate
promotionEligibility
promotionFieldRows
promotionStatusRules
promotionEvidenceSnapshot
promotionSignature
mergePromotionIntoSetting
savePromotionTransaction
runEvidencePromotionFixtures
```

Do not modify `buildDesignResult()`, shared geometry, SVG serializers, or the Drawer Cabinet Finished Front renderer. Do not add session fields to `state` or backup output. If implementation requires changing production normalization, source schemas, import semantics, or persistence behavior beyond additive compatible metadata, stop for another architecture review.

## 17. Risks and deferred behavior

### Primary risks

1. **Verified status becomes a proxy for “winner.”** Block without explicit applicable physical checks, notes, and date.
2. **Drawer Cabinet values collapse.** Keep Cabinet and Drawer choices separate; block ambiguous general-field writes.
3. **Machine context is lost.** Copy exact machine snapshots and block fit/cut/kerf cross-machine promotion.
4. **Evidence depends on live source IDs.** Store readable snapshots and preserve unknown fields.
5. **Promotion partially saves.** Clone and commit one rollback-aware target transaction.
6. **Preferred/supersede history is silently rewritten.** Require explicit conflict/replacement review.
7. **Non-cut tests populate cut settings.** Filter by operation and offer evidence-only when incompatible.
8. **Generic Project completion is mistaken for fit evidence.** Expose only fields with semantic support.
9. **Duplicate promotions create misleading evidence.** Warn with deterministic signatures without destructive blocking.
10. **Current docs drift.** Reconcile the stale README fixture totals after implementation, not during this read-only pass.

### Deferred

- automatic promotion from any save, winner selection, download, Project completion, Designs opening, or setting application;
- cross-machine scaling or compatibility inference;
- automatic profile creation or name-based target matching;
- direct promotion of an unselected coupon candidate;
- automatic kerf-sign inference;
- generalization of sheet/batch evidence;
- automatic verification from ratings, confidence, photos, status, or source labels;
- broad operation-specific Production Setting schema beyond the current cut target;
- separate persisted Drawer Cabinet cabinet/drawer fit fields unless a follow-up schema decision approves them;
- live foreign-key dependencies or cascade deletion;
- source `promoted` flags;
- automatic supersede-chain following;
- LightBurn file generation, SVG kerf offsets, or geometry changes.

## 18. Independent-audit recommendation

An independent audit is required after implementation and before relying on promoted settings physically. Audit priority is **Blocker** if the implementation changes:

- `normalizeProductionSetting()` or evidence normalization;
- Library profile save/rollback;
- preferred conflict or supersede behavior;
- backup/import/merge;
- Material Test, Grid, Coupon, or Project source schemas;
- `persist()` or source mutation paths.

The audit must verify inert candidate review, exact field selection, machine isolation, conservative status rules, source deletion resilience, snapshot readability, duplicate warnings, preferred/supersede rollback, unknown-field preservation, backup/import round trips, and retention of all current 1303/0 regression groups.

## 19. Explicit conclusion

**READY AFTER DESIGN DECISION**

The app has a suitable nested Library target model, explicit preferred/supersede primitives, rollback-backed profile persistence, and a 1303/0 fixture baseline. Phase 7.3 should proceed only after the Drawer Cabinet mapping, Joint Coupon source-record decision, Project eligibility boundary, and contextual verification rules are explicitly approved. Once those decisions are fixed, the recommended session-only candidate/review/transaction architecture can be implemented without changing storage keys, backup format, Designs geometry, or SVG serialization.
