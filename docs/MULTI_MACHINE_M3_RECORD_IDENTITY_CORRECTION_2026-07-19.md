# M3 focused-audit bounded corrections

Date: 2026-07-19  
Baseline: `7a9ec6d Add multi-machine manager and switching`  
Scope: the two IMPORTANT findings from the M3 focused audit only.

## 1. Repository state

- Branch: `main`
- HEAD: `7a9ec6d65755ab4a17d24b530c94ec3a9800d15e`
- `origin/main...main`: `0 0`
- Nothing staged, committed, or pushed.
- Existing M3 edits remain in `index.html`, `README.md`, `CHANGELOG.md`, and M3 reports. The unrelated Gift Box architecture report and historical untracked files were preserved untouched.

## 2. Correction 1 — Test Grid wording

Only visible wording changed:

- `Use active machine` → `Use active workshop machine`
- `Assign active machine` → `Assign active workshop machine`

Option values and all Grid selection, fallback, normalization, creation, edit, and import behavior remain unchanged. A focused DOM assertion checks both new/edit controls and rejects the old wording.

## 3. Correction 2 — focused fixture coverage

`runMachineRecordIdentityFixtures()` now has 27 assertions and covers:

- Candidate-ID/target-no-ID rejection and candidate-no-ID/target-ID rejection as separate checks.
- Same-name/key with different IDs rejection.
- Same ID with renamed labels acceptance.
- Promotion-time switch from Machine A to Machine B while the candidate, created setting, and evidence remain frozen to Machine A.
- Real Project form hidden-field values, unrelated notes edit, actual submit handler, and identity-preserving save.
- Real Reference Copy handler: SainSmart attribution remains, no owned ID or production setting/evidence is created.
- Individual exact-ID references for Grid, Material Test, production setting, evidence, and Project settings.
- Label-only snapshots do not count as references.
- Referenced non-active machines can be archived; permanent deletion remains blocked; records are byte-stable through the archive attempt.

## 4. Fixture cleanup

The fixture restores state and storage, active tab and machine fields, Grid/Profile/Project/entry/Inventory records, draft tests/settings/evidence, Project drafts, import state, first-run state, Design draft/preview, machine-manager filters, modal contents/open state, modal focus origins, modal open order, and final rendered DOM/focus. No synthetic machine or ID remains after cleanup.

## 5. Exact totals

- M3 focused: **27 passed / 0 failed**
- M2 focused: **31 / 0**
- M1 focused: **29 / 0**
- Machine compatibility: **50 / 0**
- Modal accessibility: **28 / 0**
- Non-Design: **1200 / 0**
- Designs: **1093 / 0**
- Complete suite: **2293 passed / 0 failed**
- Complete-suite groups: **27**
- Tray standalone: **264 / 0**
- Established promotion-switch standalone: **16 / 0**

Arithmetic: `1200 non-Design assertions + 1093 Designs assertions = 2293`.

## 6. Direct `file://` validation

Headless Edge opened the app directly from `file:///C:/Genmitsu%20L8%20Tracker/index.html` for `?selftest=machines-m1`, `?selftest=machines-m2`, `?selftest=machines-m3`, and `?selftest=all`.

- M1 and M2 completed with no page errors.
- M3 completed at **27 / 0** with no page errors.
- The complete route completed at **2293 / 0** by the established suite arithmetic.
- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.

The broad route still emits four pre-existing SVG `height="auto"` browser warnings; there were no page exceptions or fixture failures. This correction does not address those unrelated warnings.

## 7. Documentation updates

Updated `README.md` with the explicit workshop-machine wording, M3 focused total, and complete-suite total. Updated the M3 implementation report with the correction note and final totals. `CHANGELOG.md` was not changed because it does not record fixture totals.

## 8. Protected-boundary comparison

Unchanged: application identity/version/build constants; storage key, schema, backup format, machine root, `activeMachineId`, compatibility mirror, `machineProfile`, M1 migration/import behavior, M2 manager/switch/archive/restore/delete semantics, `machineReferencePaths`, active snapshot helpers, identity normalization and matching truth table, record `machineId` placement, promotion eligibility, Project accounting, Inventory, Pricing, Designs builders/geometry/SVG bytes/downloads/Finished Views, and offline `file://` behavior.

## 9. Files changed and diff summary

Changed for this correction pass:

- `index.html` — wording plus targeted M3 fixture assertions and cleanup coverage.
- `README.md` — wording and totals.
- `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_IMPLEMENTATION_2026-07-19.md` — correction note and totals.
- `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_CORRECTION_2026-07-19.md` — this report.

No unrelated file was staged, committed, pushed, reset, cleaned, stashed, checked out, moved, renamed, or deleted. `git diff --check` is clean.

## 10. Remaining verification and handoff

Another full audit is not needed for these bounded corrections; the focused audit’s two findings are addressed. A narrow re-verification remains worthwhile after any future M3-related source change, especially the `machines-m3` route and the complete `file://` route. Physical laser testing is not required: this pass changes wording and offline fixture coverage only, with no laser settings, production SVG, or machine-control behavior changed.
