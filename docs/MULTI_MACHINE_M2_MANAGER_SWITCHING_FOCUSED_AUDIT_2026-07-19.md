# Multi-machine Phase M2 — Machines Manager and Seamless Switching — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer:** Claude Sonnet 5
**Type:** read-only implementation audit. No file was edited, staged, committed, pushed, or otherwise changed. Nothing was implemented.

---

## 1. Repository state and actual baseline

- `git status -sb`: `## main...origin/main`. Modified (unstaged): **`CHANGELOG.md`**, **`README.md`**, **`index.html`**. Untracked: `docs/MULTI_MACHINE_M2_MANAGER_SWITCHING_IMPLEMENTATION_2026-07-19.md` plus the long-standing unrelated set — all preserved untouched.
- HEAD: **`e402b8cd3f9eb9c4ddc7c3e49b5e9d4452c4b784`** — "Add multi-machine storage groundwork" (matches expected baseline `e402b8c`).
- Branch `main`; ahead/behind origin/main `0 0`. `git diff --cached` empty (nothing staged). `git diff --check` clean (only benign LF→CRLF warnings).
- Diff confined to exactly the expected files (`index.html` +357/−40, `README.md`, `CHANGELOG.md`) — no unrelated file touched.
- **Fixture arithmetic independently reconciled by live runtime execution** (headless Edge, real `file://` load): all 28 dynamically-discovered `run*Fixtures` groups total **2545/0**. Excluding the two standalone groups not registered under `selftest=all` (`runTrayModelFixtures`=264, `runPromotionTargetSwitchFixtures`=16): `2545 − 264 − 16 = 2265`, and a direct count of `selftest === 'all'` registrations in source = **26** — exactly matching the claimed post-phase baseline (2265/0, 26 groups). `runMultiMachineStorageFixtures` (M1) = **29/0**; `runMachineManagerFixtures` (M2) = **30/0**; `runMachineSetupFixtures` = **50/0**; `runModalAccessibilityFixtures` = **28/0** — all match the implementation report's claims exactly, verified by live execution, not merely trusted.
- Constants confirmed unchanged: `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.

## 2. Files and functions reviewed

Full `index.html` diff read line-by-line (not summarized from the report). New: `ownedMachineDisplayName`, `machineTypeLabel`, `machineWorkAreaLabel`, `machineReferencePaths`, `machineModalHasUnsavedChanges`, `machineFormSnapshot`, `renderHeaderMachineControls`, `filteredWorkshopMachines`, `machineSummaryHtml`, `renderWorkshopMachinesManager`, `setActiveWorkshopMachine`, `openMachineForm`, `saveMachineForm`, `openMachineConfirmation`, `performMachineConfirmation`, `restoreWorkshopMachine`, `runMachineManagerFixtures`, `runMachineSetupCompatibilityFixtures`. Re-verified-unchanged M1 functions: `normalizeOwnedMachine`, `normalizeMachines`, `machineById`, `resolveActiveMachineId`, `machineIdentityForActiveMachine`, `machineRootFromData`, `synchronizeMachineIdentity`, `positiveMachineNumber` (confirmed absent from the M2 diff by direct grep). `render()`, `bindPage()`, `closeModal()`, `renderReference()`, `openHelp()`, import-mode/replace-confirmation modals, and the `selftest` registry were all inspected. README.md and CHANGELOG.md diffs reviewed in full.

## 3. M1 regression findings

**None found.** `runMultiMachineStorageFixtures()` runs at **29/0** both standalone and as the final step inside `runMachineManagerFixtures()` (which explicitly re-invokes it, `:194` of the diff). Live-reproduced independently: legacy migration to `legacy-workshop-machine`, blank-legacy migration, explicit-empty-`machines` no-fabrication, same-ID Merge local-wins, Merge local-active preservation, Replace restoration — all pass via the same M1 functions, confirmed byte-identical to the M1 audit's contract (M1 functions are untouched by this diff). No M1 storage/migration/merge/replace/recovery behavior was altered.

## 4. Header selector findings

Verified via source and live interaction: native `<label>`-wrapped `<select id="activeMachineSelect" aria-label="Active machine">`; empty state shows exactly `"Active machine: No workshop machine configured"` + a real `<button id="manageMachinesHeader">Manage machines</button>`; a blank owned-machine name would render as `""` for that option's text via `ownedMachineDisplayName` (returns `'Unnamed workshop machine'` when blank, confirmed by fixture and source — `:573-575`); the header **never** falls back to Genmitsu wording for an owned machine (live-confirmed: `headerHasGenmitsuFallback:false` after adding "L8 Workshop"). Archived machines are excluded from ordinary choices (`choices = machines.filter(machine => !machine.archived || machine.id === state.activeMachineId)`) — live-confirmed after archiving. Names are escaped via `esc()`. The control does not interfere with Import/Export/Help/unit/tab controls (confirmed by responsive checks, §21). `machineProfile` is never touched by header switching (live-confirmed at every step).

## 5. Switching findings — **BLOCKER identified**

**Finding M2-B1 (BLOCKER): the header Active-machine `<select>` silently stops responding to user input after any blocked switch attempt, until an unrelated full re-render happens to occur.**

Root cause, confirmed by direct source read (`setActiveWorkshopMachine`, `:5918-5928`):
```js
function setActiveWorkshopMachine(machineId) {
  if (machineModalHasUnsavedChanges()) { showInfoToast(...); renderHeaderMachineControls(); return false; }
  const machine = machineById(machineId, state.machines);
  if (!machine || machine.archived) { showInfoToast(...); renderHeaderMachineControls(); return false; }
  ...
  render();
  return true;
}
```
Both blocked branches call `renderHeaderMachineControls()` directly. That function (`:594-604` in the diff) **rebuilds `#activeMachineControls`'s innerHTML**, creating a **brand-new `<select id="activeMachineSelect">` DOM node** — but it does **not** re-attach the `onchange` handler. Only `bindPage()` (invoked exclusively by the full `render()`) does `activeMachineSelect.onchange = e => setActiveWorkshopMachine(e.target.value)`. The result: the very act of *blocking* an invalid/dirty switch **destroys the header control's own event binding**, and every subsequent header switch attempt is a complete no-op (no error, no toast, no state change — the browser visually shows the newly-picked option, but nothing happens) until some other action triggers a full `render()`.

**Live reproduction (headless Edge, real `file://`, no synthetic state injection):**
1. Added two machines ("L8 Workshop" active, "CO2 Bench" non-active).
2. Opened Edit on the active machine, changed the name (dirty), then attempted to switch to "CO2 Bench" via the header `<select>` → correctly **blocked** (`dirtyBlockedActive:true`, form value preserved, state unchanged) — this call triggered the bug.
3. Clicked Cancel (closes the modal, clears `machineModalMode`).
4. Attempted to switch to "CO2 Bench" via the **same header control** again → **`switchWorksAfterCancel:false`** — the switch silently failed to apply (`activeMachineId` remained unchanged).
5. Diagnosed directly: `document.getElementById('activeMachineSelect').onchange` was no longer a function after the blocked attempt; it became a function again only after an unrelated full `render()` (e.g., a tab click) occurred, at which point the header worked normally again (`headerWorksAfterRecovery:true`).

This is a genuine, reproducible defect in the **primary deliverable of this phase** ("seamless active-machine switching"), triggered by an entirely ordinary interaction (attempting to select an archived machine, or having any dirty machine form open — both are easy, plausible first actions). It fails silently, with no error and no visible explanation, directly contradicting the phase's own "Save or cancel the open machine form before switching machines" toast promise (the toast fires once, correctly, on the *first* blocked attempt, but the control is then broken for any *subsequent* attempt). **Note the manager's own per-row "Set … active" buttons are unaffected** (they are rebuilt by the next full `render()` of the manager section and are not touched by the out-of-band `renderHeaderMachineControls()` call), so an escape hatch exists via the Reference-tab manager — but the header control, which is the seamless/always-visible switching surface this phase was built to deliver, is genuinely broken until the user takes an unrelated action.

**Smallest bounded correction:** in `setActiveWorkshopMachine`'s two blocked branches, replace the bare `renderHeaderMachineControls();` call with either (a) a full `render()` call, or (b) re-binding the handler immediately after the rebuild, e.g.:
```js
renderHeaderMachineControls();
const select = document.getElementById('activeMachineSelect');
if (select) select.onchange = e => setActiveWorkshopMachine(e.target.value);
```
This is a tiny, storage/schema-neutral, single-function fix. One new fixture assertion should confirm: after a blocked switch (either branch), a subsequent **real** `change` event on `#activeMachineSelect` still applies successfully.

## 6. Dirty-form guard findings

Apart from the header-rebinding defect above, the dirty-guard logic itself is correct and precisely scoped:
- `machineModalHasUnsavedChanges()` compares a `FormData`-based snapshot (`machineFormSnapshot`) captured at `openMachineForm` time against the live form — correctly stable across no-op renders/focus (nothing else touches the form).
- Confirmed live: opening a form does not mark it dirty; editing a field does; the guard blocks **both** header switching (verified) and, per source, would identically block any other `setActiveWorkshopMachine` caller including the manager's "Set active" buttons (they call the same function) — so the guard is not header-specific, it correctly protects "switching" as a concept regardless of entry point.
- Cancel (`data-close` → `closeModal('machineModal')`) resets `machineModalMode=''` and correctly clears the guard (confirmed: `document.getElementById('machineModal').classList.contains('open')` was `false` immediately after Cancel, and `machineModalHasUnsavedChanges()`'s first clause short-circuits false). The **only** reason "switching after cancel" appeared broken in live testing was the header-rebinding defect (§5), not the guard itself — the guard was already correctly cleared; it was the DOM handler that was missing.
- The guard is scoped only to the machine form (`machineModalMode === 'form'`) — it does not block unrelated actions (archive/delete confirmations use `machineModalMode = 'confirm-…'`, a different value, so they are unaffected by the dirty check, and the dirty check does not fire for them since they don't call `setActiveWorkshopMachine`).
- localStorage, `activeMachineId`, `machineIdentity`, and `machineProfile` were all confirmed unchanged during a blocked attempt (`stateUnchangedDuringBlock:true`).

## 7. Manager information architecture

The manager's toolbar copy (`:367` of the diff) explicitly states: "Workshop machines are equipment you actually own or use. The active machine supplies current workshop context. The Genmitsu L8 Reference profile below only selects bundled manufacturer starting-point tables. Switching the active machine does not rewrite historical records; later phases may attach owned-machine IDs while preserving their original snapshots." This accurately covers all five required distinctions (owned equipment / active-context / Reference-profile-separate / no-rewrite / M3-deferred) in accessible, non-technical language. Confirmed present live and via fixture (`:193` of the diff checks for all these fragments in Help too).

## 8. Machine-list findings

Verified via live interaction and source: friendly name (or honest "Unnamed workshop machine" placeholder); manufacturer/model shown when present (`machineSummaryHtml`); laser type always shown (readable label via `machineTypeLabel`, incl. `CO₂` glyph); nominal power shown only when present ("20 W nominal"); work area shown only when both dimensions are present (`machineWorkAreaLabel` returns `''` otherwise — no fabricated summary, confirmed by fixture); "Active machine" and "Archived" are **textual** pills, not color-only (confirmed: text content includes the literal words); notes and names use `overflow-wrap:anywhere` styling (confirmed present in CSS diff, `.machine-meta span { overflow-wrap: anywhere; }`); IDs are never used as visible labels (search fixture explicitly asserts the raw ID string is absent from rendered text, `:189`). Counts: "available machine(s)" / "archived" / active-name line are computed directly from `normalizeMachines(state.machines)` with `!machine.archived` filtering — verified correct via live add/archive sequence (counts matched at each step, not independently re-derived here beyond the passing fixture, which is adequate given the trivial arithmetic). Empty state offers a clear `data-machine-action="add"` button (confirmed present and functional).

## 9. Add findings

Live-verified: name required (blank name → inline error, no mutation); type restricted to the four-item allowlist (native `<select>`, only `diode|co2|fiber|other` options present — confirmed by fixture and by reading the option list); power must be blank or a positive finite number (source: `!(Number.isFinite(power) && power > 0)` guards); work-area pairing enforced (`(widthRaw==='') !== (heightRaw==='')` → error, live-confirmed "Partial work area is rejected without Add mutation"); Cancel and Escape make no state/storage change (fixture + Escape specifically tested at `:144-145` of the diff, dispatching a real `KeyboardEvent`); new IDs come from `uid()` (confirmed distinct from `legacy-workshop-machine` and from each other, live-confirmed two distinct IDs); `archived` defaults `false`; first machine becomes active automatically (checkbox forced `checked disabled` when `isFirst`, live-confirmed); a second Add does **not** replace the active machine by default (live-confirmed `activeAfterSecondAdd === firstId`); explicit "Set active" checkbox works only when checked (live-confirmed for a third machine with `setActive.checked=true`); persist occurs once per successful submit (single `persist()` call in `saveMachineForm`, no loop or duplicate call site).

## 10. Edit findings

Live and fixture confirmed: stable ID preserved through edit (`id:mode==='edit' ? source.id : uid()`); unknown additive fields survive (`{ ...(mode==='add'?{}:source), ... }` spread preserves e.g. `futureField`/`futureCapability`, confirmed by both the M2 and compatibility fixture groups and not independently re-tested live since the fixture evidence is direct and unambiguous); `archived` state survives edit unless explicitly changed elsewhere (edit form has no `archived` field — confirmed by fixture `:269` "Edit modal preserves the archived state outside form fields"); editing the **active** machine re-synchronizes `machineIdentity` (confirmed); editing a **non-active** machine leaves `machineIdentity` unchanged (live-confirmed: `mirrorBeforeNonActiveEdit` unchanged after editing the non-active "L8 Backup Unit"); clearing optional fields (power/work-area) works and clears only the machine's own fields, not unrelated records (fixture `:227`); invalid submission does not partially update the record (whole-record `normalizeOwnedMachine(...)` construction is atomic — either the function returns before mutating `state.machines`, or it doesn't run at all); rename does not create a duplicate (same ID, just updated fields) and does not alter historical records (fixture explicitly checks `grids`/`profiles`/`projects` snapshot equality before/after, `:711` continuing from M1, and separately in M2 fixtures); edit does not change active selection unless the edited machine already was active.

## 11. Duplicate findings

Live-verified end-to-end: `openMachineForm('duplicate', secondId)` produced a form pre-filled with name `"CO2 Bench copy"` (distinguishable, includes "copy" per the fixture's own bar); submitting created a **new, distinct ID**, left the original ("CO2 Bench") completely unchanged, copied manufacturer/model/type/power/work-area/notes (via the same-mode object spread) including a synthetic additive field (`futureCapability`) in the fixture; `archived` reset to `false` on the duplicate even though the source might have been archived (source always constructs with explicit `archived:false` for non-edit modes, confirmed); the duplicate is **not** automatically active (live: `duplicateNotActive:true`); no shared-reference/relationship metadata is created (a plain independent record). Persist occurs once.

## 12. Archive findings

Live-verified: attempting to archive the **active** machine is blocked before any modal opens (`openMachineConfirmation` returns early with a toast, no `machineModal` opens) — confirmed `archiveActiveBlockedNoModal:true` and the machine's `archived` flag unchanged. Archiving a **non-active** machine opens a real confirmation naming the machine ("Archive CO2 Bench?"), and clicking confirm sets `archived:true` **only** (no other field touched, no `activeMachineId`/`machineProfile`/`machineIdentity` change — confirmed by source: `performMachineConfirmation`'s archive branch does exactly `{ ...item, archived:true }`). Archived machines correctly disappear from the header's ordinary choices (confirmed) while remaining visible/manageable via "Show archived" in the manager (confirmed live and by fixture). Persist occurs once per action.

## 13. Restore findings

Live-verified: restoring an archived machine sets `archived:false` **without** activating it (`restoreDidNotAutoActivate` equivalent confirmed: `state.activeMachineId` unchanged after restore), does not touch `machineIdentity`/`machineProfile`, and returns the machine to the header's ordinary choice list. Persist occurs once.

## 14. Delete findings

Live and source confirmed: the **active** machine's Delete button is rendered `disabled` with an explanatory `title` ("Active machines cannot be deleted.") — confirmed live (`deleteActiveButtonDisabled:true`); a direct call attempt (`openMachineConfirmation('delete', activeId)`) is also blocked defensively before any modal opens (belt-and-suspenders, both the UI disabled-state and the underlying guard agree). A **referenced** non-active machine's Delete is blocked with a distinct message ("This machine is referenced and can only be archived.") and no modal opens; confirmed via the fixture's synthetic-reference test (`machine-reference-grid` with `machineId:secondId`) which correctly reports exactly one reference path and blocks deletion with **zero record mutation** (`state.grids.length===1` preserved). An **unreferenced** non-active machine deletes cleanly with a single confirmation step stating "Permanent deletion cannot be undone through the normal UI... does not rewrite records," removes only the machine record, and does not cascade into any other collection (fixture: `state.grids.length===1` after deleting an unreferenced duplicate, i.e., the pre-existing unrelated grid is untouched). Persist occurs once per deletion. No double-click/duplicate-delete race was found (the confirmation button click handler is a single `onclick` assignment, not additive, and the modal closes immediately on success, removing the button from the DOM).

## 15. Machine-reference scanner inventory

`machineReferencePaths(machineId)` (`:75-86` of the diff) is a **fully generic recursive deep-scanner**, not a hand-enumerated per-collection list:
```js
function machineReferencePaths(machineId) {
  const matches = [], seen = new Set();
  const scan = (value, path) => {
    if (!value || typeof value !== 'object' || seen.has(value)) return;
    seen.add(value);
    if (!Array.isArray(value) && value.machineId === machineId) matches.push(path || 'record');
    if (Array.isArray(value)) value.forEach((item, index) => scan(item, `${path}[${index}]`));
    else Object.entries(value).forEach(([key, item]) => { if (key !== 'machines' && key !== 'machineIdentity') scan(item, path ? `${path}.${key}` : key); });
  };
  scan({ grids:state.grids, profiles:state.profiles, projects:state.projects, entries:state.entries, inventory:state.inventory }, '');
  return matches;
}
```

| Scanned root | Reaches (via generic recursion) | Exact-`machineId`-only? |
|---|---|---|
| `state.grids` | every grid + nested `cells` | yes |
| `state.profiles` | every Library profile + nested `tests[]` (Material Tests) + nested `productionSettings[]` + nested `evidence[]` (production settings' own array) | yes |
| `state.projects` | every project + nested `settings` | yes |
| `state.entries` | every Log entry | yes |
| `state.inventory` | raw materials + finished batches | yes |

Verified: exact-equality only (`value.machineId === machineId`, strict `===`, not name/label matching); names/manufacturer/model never match (there is no code path comparing them); **legacy `machine.key`/`machine.label` snapshots do not count** — live-confirmed via the fixture that seeded a grid with `machine:{key:'custom',label:'L8 Backup Edited'}` (matching a real machine's *name*) and proved `machineReferencePaths` returns **zero** matches for it (`:186-187`); blank `machineId` cannot match (an object without a `machineId` property has `undefined !== machineId` for any real id string); malformed arrays/objects cannot throw (`typeof value !== 'object'` guard, plus a `seen` Set guards against cycles); the `machines` collection and `machineIdentity` mirror are explicitly excluded from the walk so a machine can never "reference itself." No false positive from unrelated strings was found (equality is against the specific property name `machineId` only). Because the scan walks the **entire nested graph** under these five roots generically, any future M3 addition of a `machineId` field anywhere within Grids/Library-nested-Tests/Library-nested-ProductionSettings/Library-nested-Evidence/Projects/Entries/Inventory will be caught **without code changes** to the scanner — this materially reduces the risk the audit brief specifically flagged ("classify an incomplete reusable scan as IMPORTANT if it would allow deletion of a future-referenced machine"). **This is correctly implemented and is NOT classified as a defect.**

## 16. Reference-profile separation

Live-confirmed: `document.getElementById('machineProfile').value='40W'` + change event → `state.machineProfile==='40W'` while `state.activeMachineId` unchanged (source-confirmed the `machineProfile` `onchange` handler, `:9603-9604`-equivalent, is untouched by this diff and contains no reference to `activeMachineId`/`machineIdentity`). Conversely, switching the active machine never touches `machineProfile` (confirmed repeatedly across every live switch test). Reference tables still derive purely from `machineProfile` (`activeRows = state.machineProfile === '40W' ? reference40w : reference20w` — unchanged). No owned machine is fabricated from a Reference-profile selection. A CO₂-typed owned machine can still view the Genmitsu tables (nothing gates the Reference table on `machine.type`) without the UI claiming those values belong to that machine — the toolbar subtitle explicitly says the tables are "bundled Genmitsu L8 manufacturer starting points, not universal settings for another machine," which is accurate for exactly this scenario.

## 17. Existing-editor transition

The old `#machineSetupForm` singleton editor is confirmed **not rendered** anywhere (`renderReference()` now calls `renderWorkshopMachinesManager()` in its place; a fixture and a live check both confirm `!document.getElementById('machineSetupForm')`). One clear owned-machine management surface exists (the Workshop machines manager). `machineIdentity` remains correctly mirrored via `synchronizeMachineIdentity()` on every relevant mutation. **However, this transition left behind meaningful dead code** (see §24 — not user-facing, but a genuine cleanliness regression this phase introduced).

## 18. Search, filtering, and large-list findings

`filteredWorkshopMachines()` searches `name`, `manufacturer`, `model`, `type`, `notes` case-insensitively (`.toLowerCase()` on both query and haystack) — confirmed by source and by the live/fixture search test (`machineManagerSearch='backup edited'` correctly found "L8 Backup Edited" without exposing its raw ID in the rendered text). Search and "Show archived" are plain module-level `let` variables (`machineManagerSearch`, `machineManagerShowArchived`), confirmed **not** persisted (a fixture explicitly checks they appear in neither `freshState()` nor `backupObject()` keys, `:272` of the diff) and not mutating `state`/storage on input. The 50-synthetic-machine stress fixture (`:190-191`) confirms the manager renders ≥52 `.machine-card` elements, retains the manually-added "L8 Workshop" card, and preserves the `.machine-list` grid container — i.e., it remains structurally sound at this scale; no separate live 50-machine timing measurement was performed (the fixture's synchronous pass is adequate evidence at this population size; a live wall-clock measurement was not additionally taken and is noted as unverified in §33). Archived machines remain discoverable via the always-visible "Show archived" checkbox (native, labelled).

## 19. First-run and Help findings

Confirmed by fixture and source: `isFirstRunWorkspaceEmpty({ ...freshState(), machines:[{id:'one'}], activeMachineId:'one' })` returns `true` — machines alone do not suppress first-run eligibility (the onboarding check was never touched by this diff, and machines/activeMachineId are not among the collections it inspects). Onboarding dismissal remains session-only and unaffected. Help's new "Workshop machines" section (verified present, `:311` of the diff) and the amended "Starting points and physical validation" paragraph accurately describe: owned equipment, header switching, Reference-profile separation, "does not rewrite historical records," and explicitly "**M2 does not yet link** existing Tests, settings, evidence, Projects, or Designs to owned-machine IDs" — confirmed present verbatim and checked by a dedicated fixture (`:193`) requiring exactly this fragment set. Help does **not** claim machine-specific preferred settings or Designs fit-checking exist (no such language was found in the diff). No unnecessary compatibility-mirror jargon (`machineIdentity` is never mentioned by name in the user-facing Help text).

## 20. Accessibility findings

Header selector: native, labelled (`aria-label="Active machine"` plus a visible `<label>` wrapper). Manager buttons: all native `<button>` elements (confirmed by fixture: every `[data-machine-action]`/`#addWorkshopMachine` element `.tagName === 'BUTTON'`). Action labels include the machine's name (e.g., "Archive CO2 Bench", constructed via `${esc(ownedMachineDisplayName(machine))}` interpolation) for unambiguous context when multiple rows exist. The machine form/confirmation modals reuse the shared modal-accessibility infrastructure (`openModal`/`bindModal`/`modalDialog`) — confirmed `role="dialog"`/`aria-modal="true"` present on the form dialog via a dedicated fixture and live `modalDialog(...)` check. Escape cancels (confirmed live via a real `KeyboardEvent`). Active/Archived status is textual (not color-only, §8). No clickable `<div>`s or icon-only actions were found. **No duplicate HTML IDs** — confirmed both by a dedicated fixture and by an independent live scan of `document.querySelectorAll('[id]')` across the full 28-group fixture run (0 duplicates reported). Search field is a labelled `<input>`; "Show archived" is a labelled checkbox with visible text. Archive/delete confirmations use ordinary focusable buttons inside the shared dialog (keyboard accessible via the existing Tab-trap infrastructure, unmodified by this diff).

## 21. Responsive findings

Live-measured (headless Edge, real `file://`, emulated viewports): at **320px**, **390px**, and **768px** the Reference tab (header + Workshop machines manager) showed **no document-level horizontal overflow** (`document.body.scrollWidth ≤ window.innerWidth` at every width, e.g. 305≤320, 375≤390), the header (`.topbar`) never overflowed, the manager (`#workshopMachines`), the "Manage machines" button, and the active-machine `<select>` all remained present and reachable at every width. The CSS diff shows the expected narrow-width rule (`@media (max-width:700px) { .active-machine-control { width:100%; } .active-machine-control select { width:100%; } }`) matching the fixture's own regex check (`:195` of the diff: `/\.active-machine-control \{ width: 100%/`). Long names/manufacturer/model/notes rely on `overflow-wrap:anywhere` (confirmed present) rather than fixed widths. Modal width/reachability at narrow widths was not independently re-measured beyond the existing shared modal-responsive fixtures (already passing, unmodified infrastructure) — treated as adequately covered rather than re-verified from scratch.

## 22. Historical-record protection

Deep-compared before/after switching, editing (active and non-active), archiving, restoring, and deleting: `state.grids`, `state.profiles`, and `state.projects` snapshots were confirmed **byte-identical** (via `JSON.stringify` equality checks, both in the M2/compatibility fixtures and independently in live testing) across every one of these operations. No record anywhere received a `machineId` field as a side effect of M2 (the only place `machineId` is written is the fixtures' own synthetic test setup, never by production code). `activeMachineSnapshot()`, `gridMachineSnapshot()`, `productionMachineKeys`, promotion eligibility, evidence identity, and Project Wizard machine matching are all **absent from the diff** — confirmed untouched by direct grep against the full diff.

## 23. Designs-output protection

Confirmed by both fixture and live reproduction: `buildDesignResult(designDefaults()).svg` is **byte-identical** before and after switching the active machine (fixture `:159-162`, and independently reconfirmed live via the compatibility-fixture equivalent). No Design builder, serializer, filename, download path, or Finished View appears in the diff. `runDesignGeometryFixtures` remains **1093/0** and `runTrayModelFixtures` remains **264/0** (both independently confirmed via live dynamic execution, §1).

## 24. Fixture-quality findings

`runMachineManagerFixtures` (30 assertions) and `runMachineSetupCompatibilityFixtures` (the new body of `runMachineSetupFixtures`, verified 50/0) are **genuinely behavioral**: they drive the real `openMachineForm`/`saveMachineForm`/`setActiveWorkshopMachine`/`openMachineConfirmation`/`performMachineConfirmation`/`restoreWorkshopMachine` functions through real DOM form submission and real button clicks, not source-string checks or logic-only stubs. The one meaningful **gap**: no assertion in either fixture group reproduces the exact BLOCKER sequence in §5 (dirty-form-block → Cancel → attempt a **second** header switch via a dispatched `change` event) — every fixture that exercises the blocked path (`:164`, `:173`, `:177`, `:181`, `:184-185`) stops at confirming the *block itself* worked, never re-attempting a subsequent valid switch through the header afterward. This gap in test depth is exactly why the defect went undetected before this audit, and is the direct explanation for why source review plus live reproduction — not fixture trust alone — was necessary here.

**Dead code left behind by this phase (confirmed by direct source read, not merely the diff):**
- `function runMachineSetupFixtures() { return runMachineSetupCompatibilityFixtures(); ` followed immediately by **~74 lines of the old, now-unreachable fixture body** (referencing the retired `machineSetupForm`/`machineName`/`machineWorkAreaWidth` element IDs) still present in the file (confirmed at `index.html:911-985`-ish region).
- `function saveMachineSetup(form) { ... }` (the old M1-era singleton-editor save handler) still fully defined (`index.html:6059`), now called by nothing live.
- `bindPage()` still contains `const machineSetupForm = document.getElementById('machineSetupForm'); if (machineSetupForm) machineSetupForm.onsubmit = ...` (`index.html:10631-10632`) — harmless (the `if` guard prevents any error since the element no longer exists) but dead.

None of this dead code executes or affects behavior (confirmed: the early `return` makes the old fixture body unreachable; `saveMachineSetup` is never invoked since no element with `id="machineSetupForm"` is ever rendered). It does **not** violate the audit brief's literal "no dead handler refers to removed DOM in a way that throws" bar. It is, however, a real maintainability regression newly introduced by this phase (roughly 100 lines of orphaned code referencing a retired feature), and leaving it risks confusing a future M3 implementer working in this exact area. Classified **IMPORTANT** (correctable before commit, not a functional/data risk).

## 25. Fixture-realism findings

Reference-detection, delete-blocking, and archive-blocking assertions all operate on **real, synthetically-seeded records** (a real grid object with a real `machineId`, a real grid with only a name-matching legacy snapshot) rather than calling `machineReferencePaths` in isolation with hand-built inputs disconnected from actual record shapes — this is realistic and directly relevant to the scanner's real-world behavior. The 50-machine stress test uses `normalizeOwnedMachine(...)` to construct each synthetic machine (not raw literals), ensuring the stress data itself passes through the same normalization path as real user input.

## 26. Fixture-cleanup findings

`runMachineManagerFixtures`'s `restore()` (and the parallel one in `runMachineSetupCompatibilityFixtures`) correctly restores: `state` (full `JSON.parse`/`Object.assign` snapshot restore, covering `machines`/`activeMachineId`/`machineIdentity`/`machineProfile`), `localStorage` (including the "was absent" `removeItem` case), `firstRunOnboardingDismissed`, `designDraft`/`designPreviewMode` (M2 group only — confirmed present), `machineManagerSearch`/`machineManagerShowArchived`, all four `machineModal*` module variables, `modalFocusOrigins`/`modalOpenOrder`, the `machineModal` element's exact `innerHTML`/`open`-class/`aria-hidden` snapshot, and `document.activeElement` (with a `try/catch` fallback). A final `render()` call reconciles the visible DOM with restored state. No synthetic machine, grid, or profile created during either fixture group's execution was found surviving after `restore()` — confirmed live: after running the full 28-group suite, `localStorage`'s `genmitsu-l8-tracker-v1` payload was independently inspected via the manual live-scenario scripts run immediately afterward and contained only the machines this auditor's own live scripts had deliberately added (not any fixture-internal synthetic data), consistent with correct fixture cleanup.

## 27. Fixture-total reconciliation

Independently reconciled (not trusted from the report):
- M1 committed baseline: **29/0** (live-confirmed).
- New M2 group: **30/0** (live-confirmed, and independently recounted from the diff's `add(...)` call sites — 30 distinct calls).
- `runModalAccessibilityFixtures`: **28/0** (live-confirmed; the diff shows exactly one assertion changed in place — the old `machineSetupForm`-labelling check replaced by an active-machine-selector check, `:287-290` — not a net addition, so the group's total count is unchanged from its pre-M2 value plus whatever the prior phase already established; live execution is the authoritative confirmation here).
- **Dynamic total across all 28 registered `run*Fixtures` functions: 2545/0.**
- Excluding standalone `runTrayModelFixtures` (264) and `runPromotionTargetSwitchFixtures` (16): **2265/0**.
- `selftest === 'all'` registrations counted directly in source: **26** (exact match).
- Non-Design = `2265 − 1093 = 1172` (matches the claimed 1172 exactly).
- No existing assertion was found weakened or deleted — every prior group's live count matches its expected pre-M2 value (15, 19, 50, 37, 36, 45, 60, 22, 16, 58, 12, 66, 118, 23, 18, 12, 216, 20, 57, 56, 61, 17, 67, 264).

## 28. Runtime/`file://` validation

**Actual `file://` startup performed**, not simulated: headless Microsoft Edge (`--headless=new`, fresh scratch `--user-data-dir`) opened `file:///C:/Genmitsu%20L8%20Tracker/index.html` directly (confirmed via the CDP tab's reported URL), and separately with `?selftest=` variants exercised through direct `window.run*Fixtures()` invocation (equivalent to the query-string dispatch, since both paths call the identical functions). `git diff --check` clean. HTML/inline-JS validity corroborated by 2545 real assertions executing without a single thrown exception across all 28 groups. Live UI reproductions performed via real DOM clicks and dispatched `change`/`submit`/`keydown` events (not internal function calls) for: empty state, first/second Add, header switch, dirty-guard block, Cancel, **the post-cancel switch failure (§5)**, diagnosis of the exact broken binding, recovery after an unrelated render, active-archive block, non-active archive + confirm, restore, duplicate, delete-active-disabled, and responsive checks at 320/390/768px. Reference-detection and delete-blocking-by-reference were verified via the passing fixture evidence (source-confirmed as genuinely behavioral, §25) rather than re-driven through the full Log/Grid creation UI, given time constraints and that the underlying scanner logic was independently source-audited (§15). No real user profile or data was touched at any point; the disposable Edge profile was terminated at the end of the session.

## 29. README/CHANGELOG findings

Both accurately describe: multiple owned machines, header switching, the Reference-tab manager, Add/Edit/Duplicate/Archive/Restore/permanent-Delete-of-unused-non-active-machines-only, the active-vs-Reference-profile distinction, "switching does not rewrite historical records," and explicitly "**M2 does not yet link** Tests, settings, evidence, Projects, or Designs to owned-machine IDs" (verbatim in both files). Merge/Replace machine semantics are described precisely and match source behavior (Merge unions by ID with local/active winning; Replace restores the backup's model). The exact `2265/0`, `26` groups, `29/0`/`30/0` sub-totals, and the new `machines-m1`/`machines-m2` selftest query values are all present and independently confirmed accurate (§27). No overclaiming found: no claim that Test Grids/production settings/promotion/Designs are already machine-ID-aware, no large-format claim, no product-renaming claim.

## 30. Protected-boundary comparison

Confirmed unchanged by direct diff/grep and live execution: `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`; all M1 functions (§3); `machineProfile` semantics and Reference tables; `productionMachineKeys`; `activeMachineSnapshot`/`gridMachineSnapshot`; every historical record schema (§22); evidence identity and promotion eligibility; Project Wizard matching; Library matching; first-run eligibility; Project accounting; Inventory calculations; Pricing calculations; every Designs builder/geometry/kerf-clearance/SVG-color/serializer/filename/download/Finished-View path (§23); no network/dependency behavior introduced.

## 31. Findings by severity

**BLOCKER (1):**
- **M2-B1** — The header Active-machine selector's `onchange` binding is destroyed (not re-attached) by `renderHeaderMachineControls()` on both blocked-switch branches of `setActiveWorkshopMachine`, making the primary "seamless switching" control silently non-functional after any blocked attempt until an unrelated full re-render occurs. Confirmed by live reproduction with exact root cause and a minimal, storage-neutral, single-function fix (§5).

**IMPORTANT (1):**
- **M2-I1** — This phase leaves ~100 lines of dead code referencing the retired singleton editor (`saveMachineSetup`, the unreachable body of the old `runMachineSetupFixtures`, and a dangling `bindPage()` reference to `machineSetupForm`). Zero runtime/data risk, but a genuine cleanliness regression that should be removed before commit given M3 will work in this exact area (§24).

**POLISH (0 additional):** none beyond what is already covered above (the fixture-depth gap in §24 is folded into M2-I1's remediation, not counted separately since fixing M2-B1 requires adding the missing assertion anyway).

**NOT A DEFECT:** the machine-reference scanner's generic recursive design (§15) — correctly implemented, not incomplete; manager escape-hatch via per-row "Set active" buttons remaining functional during the header's broken window (unintentional but harmless mitigation, not a designed feature); all Add/Edit/Duplicate/Archive/Restore/Delete rules exactly as specified; Reference-profile separation; responsive behavior; accessibility semantics; Help/README accuracy.

## 32. Exact final verdict

**CHANGES REQUIRED BEFORE COMMIT**

**Minimal bounded correction plan:**
1. **Exact function:** `setActiveWorkshopMachine` (`index.html:5918-5928`).
2. **Exact failure mode:** on either blocked branch (dirty machine form open, or target machine archived/missing), `renderHeaderMachineControls()` rebuilds `#activeMachineControls`'s `<select id="activeMachineSelect">` without re-attaching its `onchange` handler, silently disabling the header switch control for all subsequent attempts until an unrelated full `render()` occurs.
3. **Storage/schema implications:** none — this is a pure DOM-event-binding fix; no state shape, persistence, or import/export logic is touched.
4. **Required fix:** immediately after each `renderHeaderMachineControls();` call inside `setActiveWorkshopMachine`'s two blocked branches, re-bind the freshly-rendered select's `onchange` handler (either inline, or by re-invoking the same one-line binding used in `bindPage()`), so the control remains live after a block.
5. **Required fixture:** extend `runMachineManagerFixtures` (or add one assertion) to reproduce exactly the sequence in §5 — trigger a blocked switch, clear the guard (Cancel or make the target machine available again), then dispatch a **real** `change` event on `#activeMachineSelect` and assert `state.activeMachineId` updates correctly. This closes the test-depth gap identified in §24.
6. **Recommended, not required for the blocker:** delete the ~100 lines of dead code identified in §24 (`saveMachineSetup`, the unreachable tail of `runMachineSetupFixtures`, the dangling `bindPage()` reference) as part of the same correction pass, since it is in the same functional area and trivial to remove alongside the binding fix.
7. **Whether Codex can implement it directly:** yes — both the binding fix and the dead-code removal are small, mechanical, single-file, storage-neutral changes confined to functions already identified exactly.
8. **Whether a second focused audit is necessary:** a **narrow** re-verification is warranted — re-run the full fixture suite (confirm the new assertion passes and the total becomes 2266/0 or as adjusted), and re-reproduce the exact §5 sequence live to confirm the header now survives a blocked attempt. A full re-audit of the entire M2 phase is **not** necessary; everything outside M2-B1/M2-I1 was independently verified clean in this pass.

## 33. Remaining unverified areas

- Live wall-clock rendering-time measurement at 50+ machines (fixture-confirmed structurally sound; no separate timing benchmark was taken).
- Full end-to-end reference-detection/delete-blocking driven through the real Log/Test-Grid creation UI (verified instead via the passing, genuinely-behavioral fixture plus independent source audit of the scanner, §15/§25).
- Modal width/reachability at 320px specifically for the machine form dialog (the shared modal-responsive infrastructure was not modified by this diff and its own fixtures pass; not independently re-measured pixel-by-pixel in this session).
- Whether any other, unexercised code path also calls `renderHeaderMachineControls()` directly without a subsequent full render (only the two branches in `setActiveWorkshopMachine` were found via grep of the diff; no other call site of `renderHeaderMachineControls` outside `render()` itself was found in the diff, so this appears to be the only occurrence of the pattern).

## 34. Physical laser testing required?

**No.** Every finding concerns JavaScript event-binding and code cleanliness; nothing depends on physical laser behavior, and Designs production bytes are confirmed unchanged (§23).

## 35. Whether M2 may be committed

**Not yet, as-is.** M2-B1 must be fixed (and ideally M2-I1 cleaned up in the same pass) before commit, per §32.

## 36. Whether M3 may begin after commit

Once M2-B1 is fixed, fixture-verified, and committed, **M3 may begin** — the underlying M1 storage contract and M2's reference-detection scanner (§15) are both sound and forward-compatible with M3's planned `machineId` back-references, requiring no architectural change.

## 37. Confirmation

This audit made **no edits** to any source or documentation file other than creating this report, and did **not** stage, commit, or push. HEAD remains `e402b8c`, and the working-tree diff is exactly as the M2 implementation left it. A temporary headless-Edge profile was used solely for read-only runtime verification and was terminated at the end of the session; no repository file or real user data was modified by it.
