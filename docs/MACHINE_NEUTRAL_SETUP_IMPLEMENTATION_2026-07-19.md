# Machine-neutral Setup and Advisory Work Area — Implementation Report

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Phase:** Machine-neutral Setup and Advisory Work Area (release-readiness)

---

## 1. Actual HEAD hash and subject

`ea5465ac95e7351b90d48b3fee2241a7c22b106c` — **"Add in-app help and laser safety guidance"** (matches the expected baseline `ea5465a`). No commit was created by this phase; changes are uncommitted working-tree edits.

## 2. Branch and ahead/behind state

Branch `main`; `git rev-list --left-right --count origin/main...main` → `0 0` (fully synchronized). Nothing was committed or pushed.

## 3. Initial git status and diff state

At start: tracked tree **clean** (`git diff`, `git diff --cached`, `git diff --stat` all empty), `git diff --check` clean. Untracked files were the long-standing set (`LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, historical `docs/*.md`). Pre-phase fixture baseline confirmed as stated: **1991 / 0** (verified by runtime run before and reconciled after).

Constants confirmed at start and unchanged after: `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.

## 4. Unrelated files preserved

All unrelated untracked content (LightBurn projects, `debug.log`, `parametric_qr_stand_generator.py`, prior reports) was left exactly as found — not modified, moved, renamed, deleted, or staged. Final `git status` shows only `CHANGELOG.md`, `README.md`, `index.html` as modified tracked files plus this new report.

## 5. Files changed

- `index.html` — additive `machineIdentity` state/normalization, backup inclusion, deliberate merge/replace handling, `currentMachineDisplayName` helper, Reference "Current workshop machine" setup UI + Genmitsu attribution, narrow first-run and Help wording, new `runMachineSetupFixtures()` group + selftest registration. (+158 / −11 lines.)
- `README.md` — Reference/machine-setup description, backup/merge/replace documentation, fixture totals and registry.
- `CHANGELOG.md` — one accurate bullet under 0.9.0.
- `docs/MACHINE_NEUTRAL_SETUP_IMPLEMENTATION_2026-07-19.md` — this report (new, untracked).

## 6. Existing machineProfile behavior found

`machineProfile` is a persisted top-level string (`'20W'`/`'40W'`, default `'20W'`) selecting the bundled Genmitsu L8 Reference tables (`reference20w`/`reference40w`) and the 20W focus helper. It is written by the Reference `#machineProfile` selector (`bindPage`), read by `renderReference`, `designProductionMachineKey`, `activeMachineSnapshot`, `wizardMachineLabel`, and production-setting defaults. It is included in `freshState`, `loadState`, `persist`, `backupObject`, `applyBackupPreferences` (so it merges on import), and reset in `resetBackupPreferences`. **This field's name, meaning, values, storage, and merge behavior were left entirely unchanged.**

## 7. Exact new machineIdentity shape

```js
{ name: string, workAreaWidthMm: (positive finite number) | null, workAreaHeightMm: (positive finite number) | null }
```
Default: `{ name: '', workAreaWidthMm: null, workAreaHeightMm: null }`. No manufacturer/model/serial/firmware/wattage/lens/origin/camera/accessory/multi-machine fields were added.

## 8. Normalization rules

`normalizeMachineIdentity(value)` (side-effect free; does not mutate input; returns a new object):
- Non-object/`null`/array input → treated as `{}` → default shape.
- `name`: kept as-is if a string; `null`/`undefined` → `''`; otherwise `String(value)`. Never HTML-interpreted (escaped at display via existing `esc`).
- `workAreaWidthMm`/`workAreaHeightMm`: numbers used directly; non-empty numeric strings converted via `Number()`; blank/whitespace, `0`, negative, `NaN`, `Infinity`, objects, arrays, and nonnumeric text → `null`. No default dimensions are invented and none are inferred from the Reference profile.
- Unknown additive properties are preserved (`...source` then the three known keys overridden), matching the project's forward-compatibility convention; repeated normalization is stable.

Integrated into `loadState()` as `machineIdentity: normalizeMachineIdentity(data.machineIdentity)` with no rename or rewrite of any existing field.

## 9. Legacy-state behavior

A stored state or backup lacking `machineIdentity` normalizes to the empty default (fixtures + runtime confirmed). `loadState` reads only known keys, so unknown top-level keys are ignored; `freshState` supplies the default so older data always yields a valid shape.

## 10. Exact merge behavior

**Merge import never applies an incoming `machineIdentity`.** It is treated as local workshop configuration, not a mergeable record collection, so `mergeData()` does not touch `state.machineIdentity` (a code comment documents this), and `machineIdentity` was deliberately **not** added to `applyBackupPreferences` (which both merge and replace call). Incoming records still merge through existing id-based `mergeList` behavior. `machineProfile`, records, and other preferences retain their existing merge behavior unchanged. Runtime-verified: merging a backup whose `machineIdentity.name='Should Not Apply'` left the local `'Local Only'` intact while the incoming entry merged in.

## 11. Exact replace behavior

`replaceData()` sets `state.machineIdentity = normalizeMachineIdentity(data.machineIdentity)` — restoring the backup's identity, or the empty default for a legacy backup lacking it. Existing replace confirmation and recovery protections are unchanged. Runtime-verified: replace restored `{name:'Imported', workAreaWidthMm:500, workAreaHeightMm:350}`; a legacy replacement yielded the empty default.

## 12. Backup behavior

`backupObject()` additively includes `machineIdentity: normalizeMachineIdentity(state.machineIdentity)` after `machineProfile`. All prior metadata (`app`, `appVersion`, `backupFormat`, `schemaVersion`) and every existing field remain present and unchanged. Round-trip preservation and additive metadata stability were fixture- and runtime-verified. Future-version, foreign-app, and foreign-backup-format guards are unchanged and still refuse before mutation.

## 13. Current-machine display fallback

`currentMachineDisplayName(source = state)` returns the trimmed `machineIdentity.name` when non-empty; otherwise an honest Genmitsu fallback (`'Genmitsu L8 20W'` / `'Genmitsu L8 40W'`) derived from `machineProfile`. It never writes the fallback into `machineIdentity`, never modifies records, and never implies a custom machine is a Genmitsu.

## 14. Reference setup UI

A compact inline `<form id="machineSetupForm">` panel (normal-flow, existing `.panel`/`.formgrid` visual language, no modal) near the top of the Reference tab, containing: a heading "Current workshop machine"; an advisory-only description; an optional `#machineName` text input (`span2`, `maxlength=120`, placeholder example only); `#machineWorkAreaWidth` and `#machineWorkAreaHeight` number inputs (mm, `step="any"`, `min="0"`, `inputmode="decimal"`); an mm/both-or-neither explanatory line; an inline `role="alert"` error paragraph; and a "Save machine setup" submit button. Every input is wrapped in a native `<label>` (existing labeling pattern). The existing 20W/40W selector was relabeled with a visible native label **"Genmitsu L8 Reference profile"** (options now read "Genmitsu L8 20W" / "Genmitsu L8 40W", plus `aria-label`), left in its existing toolbar location to avoid regression risk.

## 15. Validation and clear behavior

`saveMachineSetup(form)`: both dimensions must be entered together or both left blank; a single filled dimension shows an inline error and saves nothing (runtime-verified the previous value is retained). Entered dimensions must each be finite `> 0` (mm, decimals allowed, no manufacturer maximum imposed); invalid input shows an understandable inline message and does not save partial dimensions. Both blank clears the configured work area (width/height → `null`) while keeping the name. On success it updates only `state.machineIdentity`, persists via the existing safe `persist()`, shows the normal info toast, and re-renders. It never creates a new storage key, changes `machineProfile`, or creates records.

## 16. Genmitsu Reference attribution

The toolbar subtitle now states the 20W/40W tables are "bundled Genmitsu L8 manufacturer starting points, not universal settings for your machine"; the selector is labeled "Genmitsu L8 Reference profile"; the table heading is "Genmitsu L8 {20W/40W} official starting points"; the existing "Official values are starting points" panel is retained. Selecting a profile changes Reference content only — runtime-verified it does not change the custom machine name or work-area dimensions and does not prove the current machine's wattage. Underlying Reference values were not modified.

## 17. First-run integration

The existing onboarding panel and its three starter actions are unchanged except one narrowed list item: "Open Reference for Genmitsu L8 starting points and optional current-machine setup." The session-only dismissal behavior is unchanged. `machineIdentity` is not counted as a workshop record: `isFirstRunWorkspaceEmpty` is untouched, so populating `machineIdentity` does not suppress onboarding (fixture + runtime confirmed the panel still shows and "Open Reference" still navigates).

## 18. Help integration

The Help & Safety "Starting points and physical validation" section gained one paragraph: the optional Current workshop machine name and usable work area are advisory only — they do not resize SVGs, perform nesting, guarantee bed fit, control the machine, or change Reference settings — and the 20W/40W tables are specifically Genmitsu L8 starting points. No existing Help text was changed; Help persistence/modal behavior is unchanged; the full setup form is not duplicated in Help.

## 19. Existing machine-field defaults changed, with exact workflows

**None.** No existing new-record workflow was changed to prefill `currentMachineDisplayName()`.

## 20. Existing machine-field defaults deliberately left unchanged, with reasons

Inspected: New Test Grid (`openGridForm`/`gridMachineControls`/`activeMachineSnapshot`), Material Test records, production settings/evidence (`normalizeProductionSetting` machine key), Designs-to-Projects handoff, and promoted-evidence machine keys. All of these derive machine identity from the **Genmitsu canonical key/label system** (`genmitsu-l8-20w` / `genmitsu-l8-40w`, labels like "Genmitsu L8 20W") tied to `machineProfile`, and multiple fixtures assert those exact keys/labels. A free-text custom `machineIdentity.name` is explicitly **not** a Genmitsu Reference identity and must not imply one; injecting it into these provenance records would alter established machine-provenance semantics and break fixture bytes, and would broaden the phase into a machine-provenance redesign (out of scope). Therefore all existing machine-identity defaults were left unchanged. A fixture proves a custom name does **not** leak into Test Grid machine identity (`activeMachineSnapshot().label` stays "Genmitsu L8 20W") while `currentMachineDisplayName()` still returns the custom name for display, and another proves changing `machineIdentity` does not rewrite existing records.

## 21. Work-area use and explicitly deferred fit behavior

The phase stores and displays advisory usable work-area dimensions only. No automatic nesting, arrangement, multi-sheet splitting, SVG clipping, geometry resizing, production-output change, fit guarantee, automatic rejection, manufacturer bed presets, rotary/camera/origin logic was added. The optional read-only "Configured work area … advisory only" statement in **Designs** was **deferred**: the app does not expose a reliable exact exported-layout bounding box uniformly across every template and multi-file output, and adding markup to the Designs tab risked interacting with the 1093 Designs geometry/byte assertions. Deferring keeps Designs builders, serializers, and production bytes untouched. A fixture proves `machineIdentity` does not change Designs production SVG bytes (identical SVG with empty vs. populated identity).

## 22. Accessibility and responsive behavior

Real `<label>`-wrapped native inputs and buttons; the error uses `role="alert"`; the panel uses `aria-labelledby` to a unique heading id; no clickable divs, no new modal or focus trap. Keyboard users can edit and submit the form (native submit). Runtime-verified at **320 px**: no horizontal page overflow (`scrollWidth 320 = innerWidth 320`), the panel is present and within the viewport (right edge 302), inputs remain wrapped in labels, and the release footer stays visible; at **1280 px** desktop no overflow. Long names wrap safely (escaped text in normal-flow inputs/`.panel`). Existing Reference selector/tables, Help, and first-run surfaces remain usable. No global CSS rule was added (reused `.panel`/`.formgrid`/`.span2`/`.actions`, which already collapse to one column at the 700 px breakpoint).

## 23. Confirmation: no new storage key or migration

`machineIdentity` is the only new persisted field; it lives inside the existing `genmitsu-l8-tracker-v1` object. No second `localStorage`/`sessionStorage`/cookie/IndexedDB key was created (runtime-verified `Object.keys(localStorage)` stays `["genmitsu-l8-tracker-v1"]` after save; fixture asserts no `…-machine` key). No schema migration was introduced.

## 24. Confirmation: STORAGE_KEY and SCHEMA_VERSION unchanged

`STORAGE_KEY = 'genmitsu-l8-tracker-v1'` and `SCHEMA_VERSION = 2` — unchanged (fixture-asserted and source-verified).

## 25. Confirmation: APP_VERSION and BUILD_DATE unchanged

`APP_VERSION = '0.9.0'` and `BUILD_DATE = '2026-07-19'` — unchanged (fixture-asserted and source-verified).

## 26. Fixtures added or updated

Added `runMachineSetupFixtures()` (selftest query `machine`, registered once in `selftest=all`), **50 assertions**, self-restoring (state, localStorage, `firstRunOnboardingDismissed`, `activeTab`, `machineProfile`, `machineIdentity`, helpModal contents/open/aria, records, followed by a final `render()`). Coverage includes: default shape; legacy → default; side-effect-free and stable normalization; name/whitespace/decimal/blank/zero/negative/NaN/Infinity/nonnumeric/object/array handling; unknown-field survival; display-helper custom-name and Genmitsu fallback; real Reference-form save, partial-input rejection, non-positive rejection, and clear; `machineProfile` independence on save and on profile change; no custom-name leak into Test Grid identity; existing records not rewritten; first-run non-suppression + Open Reference; `backupObject` inclusion and round trip; additive metadata unchanged; **real-seam** merge-preserves / replace-restores / legacy-replace-default / rejected-import-no-mutation with unchanged future/app/format guards; STORAGE_KEY/SCHEMA_VERSION/APP_VERSION/BUILD_DATE unchanged; single storage key; Reference panel rendering and Genmitsu attribution; advisory/mm wording; Help wording + dialog accessibility; no duplicate IDs; Designs byte neutrality; and four assertions that the storage-recovery, first-run, Help/Safety, and modal-accessibility groups still report `0` failed. No existing assertion was weakened or deleted. (One machine-fixture regex was corrected during runtime validation to match the panel's actual wording before the suite reached 0 failures.)

## 27. Exact fixture totals: passed, failed, and total

Runtime `?selftest=all` (headless Edge, `file://`): **2041 passed / 0 failed** across **20** groups.
- Machine setup: **50 / 0** (new).
- Designs geometry: **1093 / 0** (unchanged).
- Non-Design: **948** (898 baseline + 50).
- Storage recovery 15/0, first-run 19/0, Help/Safety 37/0, modal accessibility 26/0, Designs-to-Projects handoff 17/0 — all unchanged.
- Tray-model group (separately callable, a subset not double-counted in the complete suite) remains 264/0.
- Reconciliation: 1991 baseline + 50 new = **2041**; only the new machine group changed the count; no existing group's count changed.

## 28. Every validation command and browser check actually run

- `git status -sb`, `git log -1 --oneline`, `git rev-parse HEAD`, `git rev-list --left-right --count origin/main...main`, `git diff --check` (clean), `git diff --stat`, `git diff`, `git diff --cached` (empty), `git ls-files --others --exclude-standard` — **filesystem/git checks**.
- HTML/inline-JavaScript validity — **corroborated at runtime**: the page loaded to `document.readyState==='complete'` and all 20 `window.run*Fixtures` functions existed and executed with zero thrown exceptions (a parse/syntax error would have prevented this).
- Complete browser fixture suite via **browser automation** (headless Microsoft Edge `--headless=new`, own scratch `--user-data-dir`, CDP `Runtime.evaluate`) against **actual `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all`** — 2041/0, 20 groups, zero duplicate IDs.
- Focused groups (machine, storage, first-run, Help, modal, Designs geometry) individually reported within that run.
- **Browser-automation live interaction** (distinct from the fixture harness, on a fresh empty profile): real Reference save, partial-dimension rejection, clear, and profile-change-preserves-identity, plus storage-key inspection.
- **Responsive**: emulated 320 px and 1280 px device metrics via CDP, measuring overflow and panel bounds.
- `git diff --check` re-run after edits — clean.

Distinctions: normalization/display assertions are **direct helper invocations**; save/validate/clear/profile tests use **synthetic DOM events / automation clicks** on the real rendered form; merge/replace/backup use **direct function invocation** of the real `applyBackupImport`/`mergeData`/`replaceData`/`backupObject` seams; duplicate-ID/responsive/console checks are **browser automation**; git/constant checks are **filesystem/source inspection**. No real OS file-download dialog or OS-level save was exercised (export uses the existing unchanged Blob path).

## 29. Actual direct file:// startup result

Loaded directly from `file:///C:/Genmitsu%20L8%20Tracker/index.html` (and `…?selftest=all`) in headless Edge: `readyState` complete, title "Genmitsu L8 Tracker", release footer intact, Reference machine-setup panel renders, all fixtures pass. No network/dependency was introduced (offline preserved).

## 30. Actual setup save/clear/validation results

Live (automation, fresh profile): saving name `"  Ortur LM3  "` + 400×300 stored `{name:'Ortur LM3', workAreaWidthMm:400, workAreaHeightMm:300}` (trimmed) with `machineProfile` still `20W`; entering width only showed the inline error and retained the previous width (no partial save); blanking both cleared width/height to `null` while keeping the name; changing the Reference profile to `40W` preserved `machineIdentity.name='Ortur LM3'`. Only `genmitsu-l8-tracker-v1` exists in `localStorage`.

## 31. Actual merge/replace results

Via the real `applyBackupImport` seam: **Replace** restored the backup's `machineIdentity` (and legacy replace yielded the empty default); **Merge** preserved the local `machineIdentity` while still merging incoming records; rejected imports (future schema / foreign app / foreign format) left `machineIdentity` unmutated and remained refused.

## 32. git diff --check result

Clean (only benign LF→CRLF informational warnings; no whitespace or conflict errors).

## 33. Protected-boundary comparison

Unchanged (verified by diff/source): `STORAGE_KEY`, `SCHEMA_VERSION`, `APP_VERSION`, `BUILD_DATE`, `APP_ID`, `BACKUP_FORMAT`, the `machineProfile` field name/meaning/merge behavior, Reference values, existing record IDs/meanings, Library promotion/evidence semantics, Test Grid history, Project accounting, Inventory behavior, Pricing behavior, Designs geometry, SVG serializers, SVG downloads, Finished Views, dimension/kerf/clearance/panel-layout calculations, production filenames/bytes, import future-version/app/format guards, corruption-recovery architecture, first-run dismissal persistence, and Help/modal accessibility infrastructure. No external dependency or network behavior was added. Changes are limited to the additive `machineIdentity` state/normalization, backup inclusion, deliberate merge/replace handling, the display helper, Reference setup UI + attribution, narrow first-run and Help wording, the machine fixtures, README, and CHANGELOG.

## 34. Whether Designs production bytes changed

**No.** No Designs builder, serializer, download path, Finished View, or production byte was modified. A fixture confirms identical production SVG with empty vs. populated `machineIdentity`, and the Designs geometry group remains 1093/0.

## 35. Remaining unverified areas

- The optional Designs "Configured work area" advisory line was intentionally deferred (see §21); no automated fit/bed-fit comparison exists or was implied.
- No real OS-level file download/import dialog was exercised (export/import Blob and FileReader paths are unchanged; import decision logic was tested via the real `applyBackupImport` seam).
- Tablet-specific (~700–1024 px) layout was not separately emulated beyond 320 px and 1280 px; the panel reuses already-responsive shared classes with no fixed widths.
- Load-time console output was not captured as a discrete error stream (a websocket hiccup during a reload aborted that capture); clean load is instead evidenced by exception-free execution of all 20 fixture groups and `readyState==='complete'`.

## 36. Final git status

Modified tracked: `CHANGELOG.md`, `README.md`, `index.html`. New untracked: `docs/MACHINE_NEUTRAL_SETUP_IMPLEMENTATION_2026-07-19.md`. All prior unrelated untracked files preserved. HEAD remains `ea5465a`. `git diff --cached` empty.

## 37. Confirmation that nothing was staged, committed, or pushed

Nothing was staged (`git diff --cached` empty), committed (HEAD unchanged at `ea5465a`), or pushed (`0 0` vs origin/main). No `git add`, reset, clean, stash, checkout, move, rename, or delete of unrelated files was performed.
