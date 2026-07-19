# M2 — Machines Manager and seamless active-machine switching

Date: 2026-07-19  
Baseline: `e402b8c Add multi-machine storage groundwork`  
Scope: UI and workflow only on the approved M1 root model.

## 1. Repository state and baseline

Implementation started on `main` at `e402b8cd3f9eb9c4ddc7c3e49b5e9d4452c4b784`, synchronized with `origin/main` (`0 0`). Tracked files were clean; only the pre-existing untracked local settings, LightBurn projects, debug log, historical reports, and utility were present and were left untouched. The pre-phase suite was `2234 / 0`.

## 2. Files changed

M2 changes `index.html`, `README.md`, `CHANGELOG.md`, and this implementation report only. No unrelated file was staged, moved, renamed, deleted, reset, cleaned, stashed, committed, or pushed.

## 3. M1 storage contract verification

The root `machines` array, `activeMachineId`, compatibility `machineIdentity` mirror, own-property legacy detection, fixed `legacy-workshop-machine`, normalization, active fallback, ID-keyed Merge, local-active Merge preservation, Replace behavior, backup validation, recovery behavior, and existing constants are retained unchanged. M1 fixtures remain `29 / 0`.

## 4. Header selector

The header now contains a native, labelled **Active machine** selector plus **Manage machines**. It shows an honest no-machine state, friendly names, or `Unnamed workshop machine`; it never treats the Genmitsu Reference profile as an owned-machine name. Only live machines are ordinary choices; an active archived record is labelled if all imported records are archived.

## 5. No-machines and unnamed machines

An empty workspace shows `No workshop machine configured` in the header and an actionable Reference empty state. Blank migrated machines remain blank in storage and display as `Unnamed workshop machine` until a user edits and names them.

## 6. Reference Machines manager

Reference now has a separate **Workshop machines** manager, clearly distinct from the Genmitsu L8 Reference-profile selector. It lists friendly name, optional manufacturer/model, type, optional nominal power, paired work area, notes, Active/Archived text, search, Show archived, counts, and actions. It remains usable for large lists without persisted indexes or pagination.

## 7. Add behavior

Add uses the existing modal system and `uid()` for a new stable ID. New records receive only the structured fields, default to live, require a friendly name, validate optional positive power and paired positive work area, and become active automatically only when they are the first machine. A clear checkbox supports an explicit active choice later.

## 8. Edit behavior

Edit preserves ID, archive state, and unknown additive fields while updating only supported structured fields. Editing the active machine synchronizes `machineIdentity`; editing another machine does not. Once a migrated unnamed machine is opened for Edit, saving requires a friendly name.

## 9. Duplicate behavior

Duplicate pre-fills the structured and unknown additive capability fields, supplies a distinct `… copy` name for confirmation, creates a new `uid()`, resets archive status to false, and does not make the duplicate active unless explicitly requested.

## 10. Active switching

Header and manager switching reject archived targets, set `activeMachineId`, synchronize the compatibility mirror, persist once, and rerender the header/manager. It does not change `machineProfile`, historical records, open Designs draft/preview, or production SVG bytes.

## 11. Unsaved-form guard

Switching is blocked while a machine Add/Edit/Duplicate form differs from its opening values. The form stays open and the user is told to save or cancel it; no draft, state, or storage is discarded.

## 11a. Focused correction

The focused audit found that a blocked header switch rerendered the active-machine select without restoring its event handlers. Header binding is now centralized in `bindHeaderMachineControls()`, which binds both the active-machine `onchange` handler and **Manage machines** action; blocked dirty-form and invalid/archived-target branches call the same helper after rerendering. A real DOM regression fixture now proves that a dirty switch is blocked, the draft and storage survive, Cancel leaves a newly rendered selector operational, and the next header switch updates `activeMachineId` and the compatibility mirror without changing `machineProfile`.

The unreachable retired singleton-editor fixture body, `saveMachineSetup(form)`, and guarded `#machineSetupForm` binding were removed after reference inspection. The active compatibility fixture remains 50 / 0. Final M2 is 31 / 0 and the complete suite is 2266 / 0.

## 12. Archive behavior

Archive is confirmed, keeps the record, hides it from ordinary header choices, and never changes `machineProfile` or historical records. The active machine cannot be archived; the user must switch first.

## 13. Restore behavior

Restore clears `archived`, persists once, makes the machine selectable again, and does not activate it automatically.

## 14. Delete-unused behavior

Delete requires confirmation and is allowed only for a non-active machine with no exact `machineId` references. It removes only the machine record and never cascades into records.

## 15. Machine-reference detection

`machineReferencePaths(machineId)` safely walks actual current record roots (Grids, Library nested data including tests/settings/evidence, Projects, Logs, and Inventory) for exact optional `machineId` equality. It ignores names, labels, and legacy `{key,label}` snapshots, tolerates malformed structures, and excludes the machine collection itself.

## 16. Reference-profile separation

`machineProfile` remains the separate Genmitsu 20W/40W table selector. Owned-machine switching does not alter it; changing it does not alter the active owned machine. Existing Reference copy attribution remains unchanged and no Reference copy gains a `machineId`.

## 17. Current workshop editor transition

The visible singleton Current workshop machine form is replaced by the manager; there are not two competing user-facing editors. The internal M1 compatibility helper remains for old migration/fixture seams, while the 50-assertion machine-setup group was deliberately updated to exercise manager compatibility rather than the retired DOM form.

## 18. First-run behavior

Machines remain optional. First-run eligibility still ignores them, the onboarding state remains session-only, and users can visit Reference to configure equipment when ready.

## 19. Help updates

Help now explains owned machines, active context, the separate Reference profile, historical snapshots, duplicate physical units, archive context, Merge/Replace at a high level, and that M2 does not yet link existing records to owned-machine IDs.

## 20. Accessibility

The header uses a labelled native select and named button. Manager actions are native buttons with text, status is textual as well as visual, cards have descriptive labels, forms use the existing modal dialog/focus trap/Escape/focus-restoration infrastructure, and duplicate-ID checks pass.

## 21. Responsive behavior

The header/manager use wrapping flex and min-width-safe controls. Disposable browser measurements at 320, 390, 768, and 1280 px all had `documentElement.scrollWidth === innerWidth`, with both active-machine context and manager controls reachable.

## 22. Large-list behavior

The manager has session-only search across name, manufacturer, model, type, and notes plus a Show archived toggle. A focused fixture renders 50 additional synthetic machines with long names and confirms the compact list remains operable.

## 23. Fixtures and exact counts

`?selftest=machines-m2` / `runMachineManagerFixtures()` reports `31 / 0`. M1 remains `29 / 0`; the intentionally refreshed machine-setup group remains `50 / 0`; modal accessibility is `28 / 0`; Designs geometry is `1093 / 0`; Tray remains separately callable at `264 / 0`. The promotion-switch helper remains standalone/excluded as before.

## 24. Complete-suite arithmetic

There are 26 complete-suite groups. The exact direct run is `1173` non-Design assertions plus `1093` Designs assertions: **`2266 passed / 0 failed`**. The corrected M2 route reports 31 assertions and the complete suite increases by one assertion over the prior implementation total.

## 25. Direct `file://` validation

Disposable headless Edge/Playwright runs directly opened `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=machines-m2`, `?selftest=machines-m1`, and `?selftest=all`. They completed with no page exceptions; focused M2 was `31 / 0`, M1 `29 / 0`, and the full suite `2266 / 0`.

## 26. Disposable UI workflow validation

In a disposable browser profile: cleared local storage, added first and second machines, switched through the header, edited the active machine, duplicated it, archived/restored a non-active machine, deleted an unused duplicate, reloaded persisted state, and confirmed the Reference selector stayed separate. The real profile and real records were not used.

## 27. README and CHANGELOG

README now describes header switching, Reference management, action rules, deferred record linkage, M1 import behavior, unchanged storage identity, route `machines-m2`, and exact totals. CHANGELOG records M2 without claiming M3 linkage, machine-aware preferred settings, fit checking, pass-through, multi-sheet output, or product renaming.

## 28. Protected-boundary comparison

Actual source and fixture comparison retain `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, M1 storage semantics, `machineProfile`, `productionMachineKeys`, `activeMachineSnapshot`, `gridMachineSnapshot`, Test Grid/Material Test/production-setting/evidence/Project schemas, promotion eligibility, Project Wizard matching, Library matching, first-run eligibility, accounting, Inventory, Pricing, Designs builders, geometry, kerf/clearance formulas, SVG colors, serialization, filenames, downloads, Finished Views, and offline/no-network behavior.

## 29. Git diff summary

The implementation diff is confined to the four M2 files named above. It adds manager UI, modal and fixtures in `index.html`, documents the phase in README/CHANGELOG, and adds this report. `git diff --check` is clean.

## 30. Remaining unverified areas

The validation is synthetic/disposable-browser coverage. Real long-lived backup exchange between application versions, user comprehension of the new terminology, and future record-linking behavior remain for later review; no real user storage was exercised.

## 31. Physical laser testing

Physical laser testing is not required for M2 because it changes management UI and storage selection only. Existing material, fit, safety, and production validation boundaries remain unchanged.

## 32. Storage/schema/backup semantics

The M1 root storage model is used but not redesigned: storage key, schema version, backup format, validation, migration, Merge, Replace, and recovery semantics are unchanged. M2 adds no persisted manager search/filter preference and no new record-level `machineId`.

## 33. Production SVG bytes

No production builder, serializer, preview geometry, filename, download, color layer, or Finished View changed. Focused switching checks and the full Designs geometry suite confirm production SVG bytes remain unchanged.

## 34. M3 readiness

M3 may proceed only after a focused audit of M2. M2 intentionally stops before attaching IDs to new or historical Test Grids, Material Tests, production settings, evidence, Projects, or Designs.

## 35. Recommended focused-audit scope

Audit active-switch persistence, dirty-form guard, archive/delete exact-reference detection, M1 Merge/Replace regression, Reference-profile separation, modal/focus behavior, narrow header layout, large-list filtering, and proof that no record/schema/SVG machine linkage was introduced.

## 36. Git hygiene confirmation

Nothing was staged, committed, or pushed. No unrelated tracked or untracked file was changed.
