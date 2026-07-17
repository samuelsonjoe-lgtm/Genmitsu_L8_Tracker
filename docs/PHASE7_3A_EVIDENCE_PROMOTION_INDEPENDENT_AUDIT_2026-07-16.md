# Phase 7.3A — Explicit Evidence Promotion Core
## Independent Read-Only Audit — 2026-07-16

## Executive summary

Conclusion: **NOT SAFE TO COMMIT**.

The current Phase 7.3A implementation has a coherent session/review/save shape, preserves the protected persistence and model helpers inspected here, and passes the available automated suites. Direct browser probes also show that opening, editing, reviewing, and cancelling promotion flows are inert before Save. Those results are useful, but they do not overcome several source-level contract violations in the promotion path.

The highest-risk issue is that the selected Grid-cell entry path replaces the source machine identity with the current global machine preference. Other commit-blocking issues allow crafted selections to pass Create/Update validation without selecting a compatible field, overwrite unselected Update context, infer physical completion from weak Material Test text, accept inappropriate verification checks, collide coupon session identities, and expose promotion for incomplete Projects.

No application files were edited during this audit. This document is the sole audit artifact created by this pass.

## Scope and baseline

The audit used the Phase 7.3A implementation brief as the contract and inspected the current uncommitted implementation in `index.html`, its adjacent documentation changes, the protected boundaries, the source adapters, transaction flow, UI entry points, fixtures, and runtime behavior.

Baseline observed:

- Repository: `C:\Genmitsu L8 Tracker`
- HEAD: `a9af90c Add drawer cabinet finished front view`
- Branch state: `main...origin/main`
- Existing tracked changes: `README.md`, `index.html`
- Existing unrelated untracked files/reports were preserved.
- The implementation report claims the Phase 7.3A suite is ready for independent audit; this report is the independent assessment of that claim.

## Independent validation performed

The following checks passed:

- `git diff --check`
- `python -m html.parser index.html`
- Direct startup with the promotion, production, design-production, design, and all self-test query variants; each loaded with the expected application title.
- Bundled Chromium runtime startup with no page errors.
- Complete reported fixture total: **1329 passed / 0 failed**.
- Evidence promotion fixtures: **26 passed / 0 failed**.
- Production settings fixtures: **66 passed / 0 failed**.
- Designs production settings fixtures: **118 passed / 0 failed**.
- Designs geometry fixtures: **587 passed / 0 failed**.
- Storage recovery fixtures: **8 passed / 0 failed**.

Focused browser probes also passed:

- Material Test: open review, edit fields, cancel; `localStorage` remained byte-equivalent.
- Grid: open a selected cell and promotion review; `localStorage` remained unchanged.
- Project: open promotion and cancel; `localStorage` remained unchanged.
- Coupon: open the physical-winner flow, confirm no default winner, cancel; draft text remained unchanged.
- A real Material Test promotion reached Save and created one setting while the pre-Save storage snapshot remained unchanged.

These checks establish good baseline behavior for startup, syntax, broad regression coverage, and pre-commit inertness. They do not exercise all required source-specific and adversarial paths.

## Findings

### 1. Blocker — Grid promotion silently substitutes the current global machine

Location: `promoteSelectedGridCell()` around line 9656.

The selected-cell path calls:

```js
startPromotion(buildGridPromotionCandidate(grid, activeCellKey, cell, {
  machine: wizardMachineLabel()
}));
```

`wizardMachineLabel()` is the current application machine preference, not the machine recorded by the Grid source. This violates the requirement that promotion must not invent or substitute source identity. A Grid result with a missing, stale, or different machine can therefore be promoted as a current 20W/40W result.

Consequence: machine-dependent production settings can appear source-proven even though the source did not establish that machine. This is especially dangerous when the current preference differs from the machine under which the Grid was run.

Recommended correction: pass only the explicitly recorded Grid machine into the candidate. If the source lacks a machine, preserve it as missing and require an explicit reviewed value under the Phase 7.3A machine rules; never populate it from the current global preference.

Detection: not detected by the current 26 promotion fixtures. The fixture constructs a Grid candidate with an explicit machine and does not exercise the selected-cell UI path under a conflicting current preference.

### 2. Major — Crafted `selectedFields` can bypass the compatible-field requirement

Locations: `promotionEligibility()` around lines 5274–5284 and `savePromotionTransaction()` around line 5330.

Create/Update validation checks only whether the requested `selectedFields` object has any keys. Save later filters those keys through `candidate.fields`. Consequently, a crafted request such as `selectedFields: { bogus: true }` passes the non-empty check, filters to zero compatible fields, and can still proceed. Create can produce an essentially empty Production Setting carrying only metadata/evidence; Update can write evidence or metadata without a selected compatible value.

This violates the requirement that unavailable or blocked fields cannot be forced through crafted input and that Create requires at least one compatible selected field.

Recommended correction: derive the valid selection from `candidate.fields` before eligibility and require at least one valid selected field for Create/Update. Reject unknown keys, blocked fields, and empty post-filter selections explicitly rather than silently dropping them.

Detection: not detected. The fixture named “Create applies only selected fields” verifies ordinary valid selections, not unknown or blocked keys.

### 3. Major — Update overwrites unselected material-condition context

Location: `mergePromotionIntoSetting()` around lines 5308–5316.

The session defaults contain `measurementScope: "unknown"` and `batchLabel: ""`. The merge function writes both values whenever the session contains them, independent of the selected fields. An Update that selects only speed can therefore change an existing target from, for example, `measurementScope: "sheet"` and a nonempty batch label to `unknown` and an empty label.

This violates the byte-preservation requirement for unchecked values and changes target context without an explicit user selection.

Recommended correction: separate review/session defaults from target mutations. Only write measurement scope or batch label when the user explicitly changed or selected them; otherwise preserve the target values exactly on Update.

Detection: not detected. Current promotion fixtures check selected speed and several unrelated fields but do not seed and assert non-default scope and batch context.

### 4. Major — Material Test physical completion is inferred from weak text/date heuristics

Location: `buildMaterialTestPromotionCandidate()` around line 5203.

The candidate treats a test as physical when it has a date, is not archived/failed, and has a winning speed, winning power, result summary, or notes. Notes or a summary are not proof that a physical run was completed. A dated manufacturer/reference record or incomplete note can consequently default to `tested`.

The brief requires incomplete/reference sources to default to estimated/untested and requires tested status only when a physical run is explicitly recorded.

Recommended correction: use an explicit structured completion signal if one exists. If the current schema lacks one, use the conservative default (`untested`/estimated) and require reviewed physical checks or an explicit completion action before allowing `tested`.

Detection: not detected. No notes-only, reference-style, dated-incomplete, or missing-winner Material Test fixture is present.

### 5. Major — Verified status accepts checks that do not establish the relevant claim

Location: `promotionStatusRules()` around lines 5286–5293.

For non-coupon candidates, the implementation accepts any of `cutCompleted`, `assemblyCompleted`, or `appearanceApproved` as enough contextual evidence for `verified`. This permits a Cut Material Test to become verified based on assembly or appearance alone. For a joint coupon, the code correctly requires `piecesFit`, but the non-coupon branch is too broad.

The contract specifically says appearance approval must not verify cutting or mechanical fit. Project candidates also need to remain evidence-only and must not gain verification through unrelated checks.

Recommended correction: define verification checks per source kind and operation. A cut Material Test should require `cutCompleted`; a joint coupon should require `piecesFit`; appearance-only evidence should support appearance claims only. Validate that the selected checks are applicable to the candidate operation and reject whitespace-only notes where notes are required.

Detection: not detected. The existing fixture tests that verification needs contextual evidence, but does not submit wrong-operation checks or whitespace-only notes.

### 6. Major — Coupon source identity and promotion signature can collide

Locations: `buildJointCouponPromotionCandidate()` around line 5237 and `promotionSignature()` around line 5300.

The deterministic coupon source ID hashes only material thickness, clearances, edge length, and body depth. It omits exact machine identity, material/profile identity, operation, and the rest of the source session/result context. Identical geometry from materially different 20W and 40W sessions can therefore share a source identity. The promotion signature also omits machine, material, operation, and a complete source snapshot.

This violates the requirement for deterministic session identity that does not collide across materially different coupon sessions and weakens duplicate detection and evidence provenance.

Recommended correction: include a versioned source-kind namespace plus canonical machine key/label, material/profile snapshot, operation, all source settings, winner/result identity, and candidate/result signature. Keep field ordering canonical and test same-geometry/different-machine and same-geometry/different-material cases.

Detection: not detected. Current fixtures verify deterministic output but do not compare materially different sessions with identical geometry.

### 7. Major — Incomplete Projects receive a promotion entry point

Locations: `projectCard()`, `projectBrowserDetailHtml()`, and `openProjectPromotion()`.

Project cards/details render “Add as Production evidence” for Projects without gating on completed status or meaningful completion data. `openProjectPromotion()` also has no completion eligibility check and can build a candidate for an incomplete Project.

The brief requires a completed Project entry point and explicitly excludes inappropriate promotion affordances for incomplete Projects.

Recommended correction: gate the button and handler on an explicit completed/valid completion condition. Hide or disable the promotion action for incomplete Projects, and add a UI test for both completed and incomplete states. Preserve the Project evidence-only restriction after the gate.

Detection: not detected. Browser coverage opened a Project promotion path but did not assert the action is absent for incomplete Projects.

### 8. Minor/Major — Requested status is not explicitly validated as an enum

Location: `promotionStatusRules()` and `savePromotionTransaction()`.

The UI offers known statuses, but crafted sessions with arbitrary status strings are not rejected. Unknown values are normalized or silently downgraded rather than producing a clear validation error.

Recommended correction: validate the requested status against the supported enum before transaction work and reject malformed values. Keep normalization for legacy stored data separate from validation of a new promotion request.

Detection: not detected by current fixtures.

### 9. Minor/Major — Supersede replacement does not reject stale supersession metadata

Location: `supersedeProductionSettingDraft()`.

The replacement is rejected if it is already marked `superseded`, but an otherwise active replacement carrying a stale `supersededById` is not explicitly rejected. The implementation does not traverse a chain, which is good, but it should reject inconsistent active-chain metadata rather than accept it.

Recommended correction: validate that an active replacement has no `supersededById` before applying supersede, and add a stale-metadata fixture. Preserve the no-chain-traversal rule.

Detection: not detected.

## Source-specific assessment

Material Test has a usable source adapter, deep-copied source snapshot, and a real browser promotion path. Its completion and verification semantics remain too inferential for safe commit.

Grid has a working selected-cell review entry point, but the machine substitution in the final selected-cell path is a blocker. Grid promotion needs adversarial tests for missing machine, conflicting current preference, and exact machine-dependent field behavior.

Joint Fit Coupon correctly exposes a physical-winner step with no default winner and keeps kerf independent from finger-joint clearance. Its deterministic identity is insufficiently specific, and final Save/verified paths lack independent coverage.

Project is correctly modeled as evidence-only in the candidate (`canVerify: false`) and excludes fit fields, but the UI exposure is not limited to completed Projects.

## Session, inertness, and mutation boundaries

The module-local `promotionSession` plus review/cancel flow is a sound architectural boundary. The focused browser probes support that opening, editing, and cancelling do not persist changes. Source adapters tested here use cloned snapshots rather than mutating source records.

That is not equivalent to transaction safety for all Save paths. The validation and merge findings above occur after the inert review boundary and can still produce incorrect durable records when Save is accepted.

## Create, Update, and evidence-only audit

Create correctly requires a target profile and exact machine handling at the transaction layer, and machine-dependent fields are blocked across machines except for the measured-thickness exception. However, Create is not safe until post-filter compatible-field validation prevents crafted empty selections.

Update preserves the target ID and nested unknown fields through the existing merge/normalization structure, but currently mutates unselected measurement scope and batch label. This is a direct violation of the Update contract.

Evidence-only promotion preserves target setting values, status, and preferred state in the tested ordinary path. It still requires duplicate and malformed-input tests and source-specific status validation before it can be accepted as complete.

## Preferred, Supersede, duplicate, and rollback behavior

The ordinary preferred-conflict prompt and demotion path exist, and the implementation preserves preferred state when an Update does not explicitly request preferred. Supersede preserves the old evidence and does not traverse a chain in the normal path.

Coverage is incomplete for multiple conflicts, declined preferred conflict, combined Preferred plus Supersede rollback, duplicate evidence IDs, stale superseded metadata, and failed Save after partial transaction work. Those gaps are material because these paths affect durable setting identity and provenance.

## Evidence snapshots, source deletion, and backup/import

The evidence snapshot contains source identifiers, source/profile labels, dates, status, source snapshot, selected values, physical checks, promotion signature, result, and notes. The design is directionally correct for source deletion resilience, and the snapshot is cloned.

The current fixtures do not actually delete each source through the application and re-render the evidence, nor do they perform a full Replace/Merge import round trip and verify promoted settings and snapshots. The “source deletion” and “backup retains” assertions are therefore partial unit-level checks, not end-to-end proof.

## Fixture quality and independence

The 26/0 evidence fixture result is a useful smoke signal but should be classified as partial coverage, not complete safety proof. Many tests call the same helpers that implement the behavior and omit the UI-to-transaction path. Missing adversarial or end-to-end cases include:

- Grid current-machine substitution and missing source machine.
- Crafted unknown/blocked `selectedFields` for Create and Update.
- Update preservation of non-default measurement scope and batch label.
- Notes-only/reference/incomplete Material Tests.
- Wrong verification checks and whitespace-only notes.
- Same-geometry coupon sessions with different machine/material/profile identity.
- Completed versus incomplete Project button exposure.
- Final Grid, Coupon, and Project Save flows.
- Actual source deletion followed by evidence re-render.
- Replace/Merge import round trips.
- Multiple preferred conflicts, combined Preferred plus Supersede rollback, duplicate evidence IDs, and signature order stability.

Some fixture names overstate their independence: “Source and profile inputs remain unmutated” compares original objects but not complete live state/storage for all source types; “Evidence remains readable after source deletion” checks a copied snapshot without deleting and re-rendering; and “Backup retains promoted settings and snapshots” does not perform Replace/Merge import.

## Protected boundaries

Static comparison against the baseline showed unchanged function bodies for the inspected protected functions and models, including:

- `persist`, `replaceData`, and `mergeData`.
- `normalizeProductionSetting` and `normalizeProductionSettings`.
- `buildDesignResult`, `buildJointCouponModel`, `buildDrawerCabinetModel`, and `buildDrawerCabinetDesignResult`.
- `serializeDesignSvg` and `downloadTextFile`.

The audit found no evidence that Phase 7.3A intentionally altered storage schema, import/export structure, Design geometry, Drawer Cabinet mappings, or the export helpers. This boundary result is positive but does not cure the promotion-specific findings.

## Deferred Drawer Cabinet behavior

The Phase 7.3A implementation does not add Drawer Cabinet finished-front fit mappings, front-thickness parsing, or collapse/expand behavior. The audit found no evidence that those deferred behaviors were pulled into this phase. They should remain deferred unless separately authorized.

## Required remediation order

1. Remove Grid current-machine substitution and preserve source machine provenance.
2. Validate post-filter compatible selections for Create/Update and reject crafted unknown/blocked keys.
3. Preserve unselected Update context, especially measurement scope and batch label.
4. Replace Material Test physical heuristics with an explicit/conservative completion rule.
5. Make verification checks source- and operation-specific.
6. Expand coupon source identity/signature inputs to prevent materially different-session collisions.
7. Gate Project promotion on completion state.
8. Add explicit status and stale supersession metadata validation.
9. Add the missing adversarial, UI, deletion, import, duplicate, conflict, and rollback fixtures.
10. Re-run the complete suite and browser probes, then perform a fresh independent audit.

## Final conclusion

**NOT SAFE TO COMMIT**

The current implementation is not ready for commit because the identified findings can create incorrect machine provenance, accept empty or incompatible promotions, mutate unchecked target context, overstate physical/verified evidence, collide provenance identities, and expose promotion for incomplete Projects. The broad tests and inertness probes should be retained as useful regression coverage, but the remediation and independent re-audit must precede commit.
