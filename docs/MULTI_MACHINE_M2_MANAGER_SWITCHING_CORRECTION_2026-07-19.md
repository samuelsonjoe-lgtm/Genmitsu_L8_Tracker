# M2 Manager and Switching Correction Report

Date: 2026-07-19  
Baseline: `e402b8c Add multi-machine storage groundwork` (`e402b8cd3f9eb9c4ddc7c3e49b5e9d4452c4b784`)  
Scope: bounded M2 header-switching correction and retired-editor dead-code removal.

## 1. Repository state and baseline

The repository was on `main`, at `e402b8c`, with `origin/main...main` equal to `0 0`. The pre-existing M2 working changes were in `index.html`, `README.md`, and `CHANGELOG.md`; nothing was staged. Existing untracked reports, LightBurn files, `debug.log`, and utilities were preserved.

## 2. Confirmed blocker and root cause

The focused audit reproduced: two machines existed; a dirty Add/Edit/Duplicate form blocked a header switch; `setActiveWorkshopMachine()` rerendered the header; the replacement select had no `onchange`; Cancel or Save therefore left the selector inert until another full render. The same risk existed for **Manage machines** because both controls were replaced together.

## 3. Correction

Added `bindHeaderMachineControls()` to bind both `#activeMachineSelect onchange` and `#manageMachinesHeader onclick`. `bindPage()` now calls that helper, and both blocked `setActiveWorkshopMachine()` branches call it immediately after `renderHeaderMachineControls()`.

Successful switching still updates `activeMachineId`, synchronizes `machineIdentity`, persists once, and performs the existing full render. Blocked switching does not persist or mutate the active machine, mirror, machine profile, records, localStorage, Design draft/preview, or production SVG bytes.

## 4. Dirty, invalid, and archived behavior

The dirty modal remains open with its draft intact and the existing explanatory toast. After Cancel, a newly queried header select accepts a real `change` event immediately. Invalid and archived targets remain blocked and restore both header bindings. Manage machines remains bound after the same rerenders.

## 5. Regression fixture

`runMachineManagerFixtures()` now creates two live machines, opens Edit through the UI, dirties the name, dispatches a real DOM `change` event on `#activeMachineSelect`, checks blocked state/draft/storage/active ID, cancels through the modal path, re-queries the replacement select, dispatches another real change event, and verifies active-machine mirror synchronization with unchanged `machineProfile`. The fixture returns state, storage, machines, modal state, focus, filters, Design session state, and rendered DOM to their captured values.

Focused M2 result: **31 passed / 0 failed**.

## 6. Dead code removed

After reference inspection, the following clearly unreachable paths were removed:

- The unreachable body after the immediate return in `runMachineSetupFixtures()` — 75 lines; `runMachineSetupFixtures()` now delegates directly to `runMachineSetupCompatibilityFixtures()`.
- `saveMachineSetup(form)` — 29 lines; no production, fixture, import, or compatibility caller remained.
- The guarded `#machineSetupForm` submit binding — 2 lines; the retired element is no longer rendered.

`normalizeMachineIdentity`, machineIdentity synchronization, M1 migration compatibility, and the active 50-assertion compatibility fixture remain intact.

## 7. Fixture totals

Verified focused groups include M2 `31 / 0`, M1 `29 / 0`, machine setup compatibility `50 / 0`, modal accessibility `28 / 0`, accessibility `36 / 0`, responsive `45 / 0`, storage recovery `15 / 0`, first-run `19 / 0`, beginner `22 / 0`, evidence/promotion `58 / 0`, Project Wizard `216 / 0`, Designs production settings `118 / 0`, Designs geometry `1093 / 0`, and Tray standalone `264 / 0`.

The reconciled complete suite is **2266 passed / 0 failed** across 26 groups: **1173 non-Design + 1093 Designs**. Promotion-switch remains standalone and excluded.

## 8. Direct file:// and browser validation

Disposable headless Edge/Playwright opened the direct file URL for `?selftest=machines-m2`, the M1 route, and `?selftest=all`. The M2 and M1 routes completed without page exceptions at `31 / 0` and `29 / 0`; the complete route’s fixture groups reported zero failures. A separate direct file:// Playwright evaluation ran the focused M2 fixture and returned `31 / 0` with no page errors and duplicate IDs absent.

This distinguishes source inspection from fixture execution: the new regression proof is a real DOM event path in a disposable browser, not a direct call to `setActiveWorkshopMachine()`.

## 9. Live reproduction status

The exact corrected workflow was exercised with synthetic disposable machines and real DOM controls: dirty header switch blocked, draft survived, replacement selector rebound, Cancel completed, and the next header switch succeeded. The fixture also covered manager actions and profile separation. No real browser profile or user records were touched. A separate physical-laser/manual-user workflow was not performed.

## 10. Documentation and protected boundaries

Updated `README.md` and the M2 implementation report with the corrected totals and correction note. `CHANGELOG.md` was not changed by this correction; its existing M2 entry already describes the final phase accurately.

The correction does not change storage shape, `activeMachineId` semantics, `machineIdentity`, normalization, migration, Merge/Replace/recovery, constants, `machineProfile`, record schemas, evidence/promotion rules, Project/Library matching, Inventory/Pricing, Designs builders or geometry, kerf/clearance, SVG serialization/colors/filenames/downloads, Finished Views, or offline behavior. No production SVG bytes changed.

## 11. Actual files and diff summary

Correction edits are confined to:

- `index.html` — binding helper, blocked-branch rebinding, real DOM regression assertions, and retired dead-code removal.
- `README.md` — fixture totals `31 / 0` and `2266 / 0`.
- `docs/MULTI_MACHINE_M2_MANAGER_SWITCHING_IMPLEMENTATION_2026-07-19.md` — correction note and totals.
- `docs/MULTI_MACHINE_M2_MANAGER_SWITCHING_CORRECTION_2026-07-19.md` — this report.

The working tree also contains the pre-existing M2 `CHANGELOG.md` modification and unrelated untracked files listed during inspection; they were not altered by this correction. `git diff --check` is clean.

## 12. Remaining unverified areas and handoff

Real long-lived cross-version backup exchange, manual user comprehension, and physical laser output remain outside this correction. Physical laser testing is not required because M2 changes management UI and selection only. Another full audit is not required for this narrow fix; a focused re-verification of the header guard, archive/invalid branch, modal focus, and protected boundaries may proceed.

Nothing was staged, committed, or pushed. No reset, clean, stash, checkout, move, rename, or delete operation was performed on unrelated files.
