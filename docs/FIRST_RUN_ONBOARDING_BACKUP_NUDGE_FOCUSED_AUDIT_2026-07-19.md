# First-Run Onboarding and Backup Nudge — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer model:** Claude Sonnet 5
**Type:** read-only implementation audit. No file was edited, staged, committed, pushed, reset, cleaned, stashed, checked out, moved, renamed, or deleted.

---

## 1. Repository state and actual baseline

- `git status -sb`: `## main...origin/main`. Modified (unstaged): **`CHANGELOG.md`**, **`README.md`**, **`index.html`**. Untracked: `docs/FIRST_RUN_ONBOARDING_BACKUP_NUDGE_IMPLEMENTATION_2026-07-19.md` and the long-standing unrelated set (`LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, older `docs/*.md`) — **all untouched by this phase**.
- `git log -1 --oneline`: **`4d51d29 Add release identity and import safeguards`**.
- `git rev-parse HEAD`: **`4d51d29f35f965fc0762149b6262ac42da0f80a2`** — matches the expected baseline `4d51d29`.
- `git rev-list --left-right --count origin/main...main`: **`0 0`** (synchronized; no local commits, nothing pushed).
- `git diff --check`: clean (only benign LF→CRLF warnings; no whitespace/conflict markers).
- `git diff --stat`: `CHANGELOG.md` +1, `README.md` +9/-4 (wait: reported as `+9 -4`? actual reported `9` changed lines total across insert/delete), `index.html` +117/-4.
- `git diff --cached`: **empty — nothing staged.**
- **Tracked baseline was clean** before this implementation (only these three tracked files carry uncommitted changes, all attributable to this phase). No commit or push occurred.

## 2. Files and functions reviewed

`index.html`: `.first-run-onboarding` CSS (`:106-112`); `firstRunOnboardingDismissed` (`:378`); new `runFirstRunOnboardingFixtures()` (`:550-613`) and its `window` export; `isFirstRunWorkspaceEmpty()` (`:1133-1136`); `firstRunOnboardingHtml()` (`:1137-1156`); `bindFirstRunOnboardingActions()` (`:1157-1165`); modified `logResultsHtml()` (`:1167-1173`); modified `bindLogResultActions()` (one added line); `normalizeInventory()` (unchanged, read for safety); `freshState()`/`loadState()`/`backupObject()`/`persist()` (all unchanged, read to confirm no touch); `render()`/`renderLog()`/`bindPage()`/`setTab()`/`openEntry()`/`openGridForm()`/`empty()` (all unchanged, read to confirm integration points); selftest registration (`:11401`). `README.md` and `CHANGELOG.md` diffs reviewed in full. Protected Designs functions confirmed absent from the diff by targeted grep.

## 3. Implementation summary

The phase adds a page-session boolean `firstRunOnboardingDismissed`; a pure helper `isFirstRunWorkspaceEmpty(source)` that reports true only when `entries`, `profiles`, `grids`, `projects`, and both `normalizeInventory(source.inventory)` arrays are all empty; `firstRunOnboardingHtml()`, appended only inside `logResultsHtml()`'s zero-entries branch (never inside the filtered-miss branch), rendering a `<section>` with a heading, explanatory paragraphs, a three-item list, and four real buttons; `bindFirstRunOnboardingActions()`, wired from the existing `bindLogResultActions()` call site so it rebinds on every full render and every targeted search-triggered partial render; and 19 new fixture assertions in `runFirstRunOnboardingFixtures()`, registered once under `selftest=first-run`/`all`.

## 4. First-run eligibility findings

`isFirstRunWorkspaceEmpty(source = state)` (`:1133`):
```js
function isFirstRunWorkspaceEmpty(source = state) {
  const inventory = normalizeInventory(source?.inventory);
  return !((source?.entries || []).length || (source?.profiles || []).length || (source?.grids || []).length || (source?.projects || []).length || inventory.rawMaterials.length || inventory.finishedBatches.length);
}
```
- **Side-effect free:** reads only; no assignment to `source`, `state`, or module-level variables. `normalizeInventory()` (unchanged, `:827-832`) itself only returns new arrays/references — it does not mutate its argument or `state`.
- **Safe with missing fields:** every access uses `?.` and `|| []`; a `source` missing `entries`/`profiles`/`grids`/`projects`/`inventory` entirely still evaluates correctly (falls through to `[]`.length === 0).
- **Safe with malformed array-like values:** `normalizeInventory` already guards non-array `rawMaterials`/`finishedBatches` via `Array.isArray(...) ? ... : []`; a non-array `entries`/`profiles`/`grids`/`projects` (e.g. `null`) is caught by `(source?.entries || [])` — `null || []` → `[]`. Only a **truthy non-array** (e.g. a plain object) would bypass the `||` fallback and could throw on `.length` access being `undefined` (not an array) rather than throwing outright — `{}.length` is `undefined`, and `undefined` is falsy, so the OR-chain still resolves safely without a thrown error. No crash path identified.
- **Independent of live global state when a source is supplied:** confirmed — the function only ever reads `source`, never falls back to `state` internally except via the default parameter itself. Fixtures call it with constructed objects (`{ ...freshState(), entries:[...] }`) and it correctly evaluates those, not live `state`.
- **No permanent "completed onboarding" flag:** `firstRunOnboardingDismissed` is a plain `let` at module scope, initialized once per page load (`:378`) and never read from or written to `localStorage`, `freshState()`, or `backupObject()`. Confirmed by direct diff inspection — it appears nowhere else in the codebase except its declaration, the dismiss handler, and the eligibility check inside `firstRunOnboardingHtml()`.

## 5. Inventory and legacy-state handling

The six checked collections (`entries`, `profiles`, `grids`, `projects`, `inventory.rawMaterials`, `inventory.finishedBatches`) are exactly the complete set of persisted user-record arrays in `freshState()` — no other array of discrete user-created records exists in the state shape (pricing/pricingPrefs/unit/sort/view-mode/search/tab/machineProfile/release-identity are all configuration scalars or objects, not record collections). This was independently re-confirmed against `freshState()`'s full literal (unchanged in this diff). `normalizeInventory(source.inventory)` called from inside `isFirstRunWorkspaceEmpty` does not mutate the supplied `source` (it returns freshly-constructed plain objects referencing the same array instances, never reassigning `source.inventory`), does not mutate live `state` when called with a constructed test object, creates no persisted records, and does not discard any record — it only reports counts. A malformed-but-non-empty record (e.g., `{ id:'r1', name:'Basswood' }` with no other fields) is still counted as an existing record because the helper only checks array **length**, never record content/validity — correctly treating any array entry, however malformed, as evidence the workspace is not fresh.

## 6. Display and Log empty-state findings

- The onboarding panel is appended **only** inside `logResultsHtml()`'s `state.entries.length === 0` branch, and only after the ordinary `empty('No entries yet', …)` call — so the standard empty-state text is always present when onboarding is present. **Live-verified** (§11): a genuinely fresh page shows both.
- The filtered/searched zero-match branch (`state.entries.length` truthy, `filtered.length` falsy) returns only `empty('No log entries match', 'Adjust the search or material filter.')` — the onboarding call is **structurally unreachable** from that branch (it lives in the other arm of the `if`). **Live and fixture confirmed**: a search yielding zero matches against a non-empty `entries` array shows only "No log entries match" and no onboarding markup.
- Clearing a search when `entries` is empty is a no-op for onboarding (an empty-`entries` filter is always `[]` regardless of query text), so eligibility is governed solely by the workspace-emptiness check, not by search state — correct.
- **Does not appear on other tabs:** `firstRunOnboardingHtml()` is only invoked from `logResultsHtml()`, which is only invoked from `renderLog()` (gated by `state.activeTab === 'log'` in `render()`, unchanged) and from `refreshLogSearchResults()` (only bound while the Log search input exists, i.e., while Log is rendered). No other tab renderer calls either function. Confirmed by full-file grep — this is an architectural guarantee, not merely an untested assumption.
- **Normal-flow content, no overlay:** the panel is a `<section class="panel first-run-onboarding">` inserted into `#logResults` alongside the ordinary empty-state `<div>`; no `position:fixed`/`position:absolute`, no modal, no toast. Confirmed via the added CSS (`margin:14px auto 0`, no positioning override).
- **No duplicate panel/handlers on repeated render:** every render path replaces `#logResults`' (or `#app`'s) `innerHTML` wholesale before rebinding via `.onclick =` (assignment, not `addEventListener`), so repeated renders cannot stack duplicate DOM nodes or duplicate handlers. Confirmed structurally and **live-verified** (zero duplicate IDs across the entire rendered document, §11).
- **Tab-switch consistency:** switching away from Log destroys `#app`'s content (including any onboarding markup) and switching back re-renders `renderLog()` fresh from current `state`/`firstRunOnboardingDismissed` — no stale panel can persist across tabs.
- **Record creation naturally suppresses the panel:** since eligibility is recomputed from live `state` on every render with no cached/memoized result, saving any qualifying record (e.g., a Log entry, a Test Grid) makes the next render's `isFirstRunWorkspaceEmpty()` call return false. Not independently fixture-tested end-to-end via a real save, but this follows directly from the function being called fresh on every invocation with no memoization — a structural guarantee.
- **Deleting all records re-enables eligibility:** same reasoning — the helper has no "already shown once" memory; only current record counts and `firstRunOnboardingDismissed` (session-only) gate it.
- **Release identity footer unaffected:** `render()` never touches `#releaseIdentity` (only `#tabs`, `#unitIn`/`#unitMm`, `#storageRecoveryPanel`, `#app`); the footer is rendered once at startup by `renderReleaseIdentity()` and never re-rendered. Confirmed structurally and **live-verified** (§11: footer text and About content intact after multiple renders/tab switches).

## 7. Copy and backup-honesty findings

Rendered copy (verbatim from source): *"This Tracker works offline. Your Log, Library, Test Grids, Projects, and Inventory are stored locally in this browser—not in the cloud."* and *"Browser data can be cleared or become unavailable, so use **Export** in the header regularly to save a JSON backup file. Restore a backup anytime with **Import**."* This accurately states offline operation, local-browser-only storage, explicit non-cloud-storage, that browser data can be lost, and the Export/Import backup-and-restore relationship. It does **not** imply automatic backups, cloud sync, accounts, remote recovery, telemetry, automatic updates, guaranteed recovery, or 1.0/paid-release completeness — none of that language appears. The warning is two short sentences plus a three-item action list — proportionate, not alarming or repetitive, and consistent in tone with the rest of the app's existing honest-but-brief caveats.

## 8. Starter-action findings

**1. Log your first job** — `logJob.onclick = () => openEntry();` calls the existing `openEntry()` unmodified, with no argument, so `editingEntryId = null` and the form seeds from `formDefaults('entry')` — the same defaults as the ordinary "New Entry" button. No record is written until the form's own `onsubmit` (which still requires a material name and calls `upsert`). **Live-verified indirectly** via the analogous "Plan a Test Grid" click test (§11); source-confirmed identical for `openEntry`.

**2. Plan a Test Grid** — `planGrid.onclick = () => { state.activeTab = 'grids'; render(); openGridForm(); };`. This is functionally **identical** to `setTab('grids')` followed by `openGridForm()`, because `setTab(tab) { state.activeTab = tab; render(); }` (unchanged, `:652`) does exactly those same two statements — there is no additional behavior in `setTab` being bypassed. Classified as a **stylistic** duplication, not a functional risk (see §14, POLISH). The sequence is safe because `openModal`/`render()` are both synchronous; `render()` finishes rewriting `#app` to the Grids tab before `openGridForm()` runs, and `openGridForm()` targets the always-present, tab-independent `#gridModal` element (a sibling of `#app`, not inside it), so there is no DOM-readiness race. **Live-verified**: clicking the real button moved `document.body.dataset.activeTab` to `"grids"` and opened `#gridModal` with `/New test grid/i` in its content (§11). Closing the modal (Cancel or Save) leaves `state.activeTab` at `'grids'` because `closeModal()` never touches `activeTab` — the user is left on Grids, matching the contract. No grid is created before Save; `openGridForm()`'s only side effects on open are building the modal DOM (confirmed by reading through to its `onsubmit`, where the only `state.grids`/`persist()` writes occur). Defaults come from unmodified `formDefaults('grid')` — no invented material/machine/speed/power/interval values beyond the app's existing standard defaults.

**3. Open Reference** — `openReference.onclick = () => setTab('reference');` — the literal existing navigation primitive, unmodified. **Live-verified** via the fixture's real-click assertion (`state.activeTab === 'reference'`); does not touch `machineProfile` or any Reference data (not present in `setTab`).

**4. Hide this introduction** — `dismiss.onclick = () => { firstRunOnboardingDismissed = true; render(); };` — sets only the page-lifetime variable and re-renders. No `state`/`localStorage`/backup effect (§9). It re-renders via the normal `render()` path, so it does not reset on unrelated ordinary renders (only an explicit assignment back to `false`, which occurs nowhere outside a fresh page load, can undo it) — and correctly resets after a full page reload since it is a plain `let`, not persisted anywhere.

## 9. Session-dismissal and persistence findings

- `firstRunOnboardingDismissed` is declared once at module scope (`:378`) and thus initialized exactly once per page load — confirmed no re-initialization elsewhere.
- **Not part of `freshState()`** — confirmed by reading the unchanged `freshState()` literal; the flag is not one of its properties.
- **Not normalized** — no `normalize*` function references it.
- **Not passed to `persist()`** — `persist()`'s destructured field list (unchanged) does not include it.
- **Not emitted by `backupObject()`** — unchanged; confirmed by direct diff (no lines touching `backupObject()` in this phase) and by a dedicated fixture that regex-scans `Object.keys(backupObject())` for `/onboard|firstRun|dismiss/i` and asserts no match.
- **Not restored by imports** — since it is absent from both `freshState()` and every backup shape, `loadState()`/`mergeData()`/`replaceData()` (all unchanged) cannot populate or touch it.
- **No new storage surface** — no new `localStorage`/`sessionStorage`/cookie/IndexedDB key, and no URL parameter was added by this phase (the pre-existing `?selftest=` mechanism is unrelated and unchanged).
- `STORAGE_KEY` remains exactly `'genmitsu-l8-tracker-v1'`; `SCHEMA_VERSION` remains exactly `2` — confirmed unchanged in source and by a fixture assertion, and **live-verified** at runtime (§11).
- No schema migration was introduced; release-identity constants (`APP_ID`/`APP_NAME`/`APP_VERSION`/`BUILD_DATE`/`BACKUP_FORMAT`) and the `backupObject()` metadata fields from the prior phase are untouched by this diff.

## 10. Accessibility findings

The panel is a semantic `<section aria-labelledby="firstRunOnboardingHeading">` with a real `<h2 id="firstRunOnboardingHeading">`, ordinary `<p>`/`<ul>`/`<li>` markup, and four real `<button type="button">` elements with clear, descriptive accessible names ("Log your first job," "Plan a Test Grid," "Open Reference," "Hide this introduction") — no clickable `<div>`s, no icon-only controls, no color-only signaling, and no removed `:focus` styles (the diff touches no `:focus` rule). Heading hierarchy: the page's single `<h1>Genmitsu L8 Tracker</h1>` precedes this `<h2>` in document order with no skipped level. `aria-labelledby` references an ID that exists exactly once (confirmed live: zero duplicate IDs anywhere in the rendered document, §11). Button DOM order (Log job → Plan Grid → Reference → Hide) matches the visual/reading order (plain `flex` row with no `order` overrides). Keyboard activation follows automatically from using native `<button>` elements (not independently verified via a synthesized keydown event in this session — see §16). Focus after Dismiss: the panel's containing `#app`/`#logResults` subtree is replaced via a full `render()`, which — consistent with every other full-render-triggered state change elsewhere in this app (filter changes, sort changes, tab switches) — does not explicitly preserve or redirect focus; this is a pre-existing app-wide pattern, not a regression introduced or worsened by this phase, and is correctly out of scope per the existing modal-accessibility roadmap item. No new modal/dialog accessibility burden is introduced (the panel is normal-flow, not a dialog).

## 11. Responsive findings

**Runtime-verified**, not merely inspected: a real headless Microsoft Edge instance was launched (`--headless=new`, its own scratch `--user-data-dir`, i.e., a genuinely fresh, empty profile) and driven via the Chrome DevTools Protocol to open the actual repository file directly via **`file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all`** (confirmed a true `file://` load, not a data URI, not a copied temp file, not an HTTP server — the CDP tab's reported `url` was exactly that `file://` path).

- At the default headless viewport (`innerWidth: 756`): `document.body.scrollWidth: 741` ≤ `756` → **no horizontal overflow**.
- At an emulated **320 px** mobile viewport: `document.body.scrollWidth: 305` ≤ `320` → **no horizontal overflow**; the onboarding panel measured `269px` wide with its right edge at `287px` (well inside the 320px viewport) — **no clipping**; all **4** action buttons were found inside `.actions` (which already has `flex-wrap: wrap` from the pre-existing base rule, reused unmodified).
- The release-identity footer measured a non-zero bounding-box width at 320px (`footerVisible: true`) — **remains visible and unobstructed**.
- No new global CSS rule was added; `.first-run-onboarding`'s rules are scoped to that class and its descendants only (verified by diff — no bare-tag or `:root`-level selectors were touched), so no other tab's layout is affected.
- Existing styles were reused where practical: `.panel` (shared with cards/panels app-wide) and `.actions` (shared button-row wrapping) are both reused unmodified; only container/typography rules specific to `.first-run-onboarding` were added.

Tablet/large-desktop widths were not separately emulated in this session, but given the panel's `max-width:640px` centered block with no fixed pixel widths and full reliance on already-responsive shared classes, no defect is expected at those widths and none is claimed as directly verified.

## 12. Fixture quality and cleanup findings

19 assertions were added (independently counted from the diff's `add(...)` calls, not trusted from the report), each exercising real seams rather than reproducing implementation logic in isolation:
- 7 eligibility assertions call the real `isFirstRunWorkspaceEmpty()` against constructed state objects covering each of the 6 record collections individually plus a configuration-only control case (unit, machineProfile, pricing, pricingPrefs, entrySort, libraryViewMode, search — confirmed none suppress eligibility).
- 4 rendering assertions call the real `logResultsHtml()`/`filteredEntries()` pipeline: genuine empty-Log render (checks both ordinary empty copy and onboarding markers together), unique-ID presence, filtered-zero-match suppression (the single most important behavioral guarantee in this phase), and empty-Log-with-other-records suppression.
- 3 dismissal assertions exercise session-only behavior against `state`, `localStorage`, and `backupObject()` directly, including a broad key-name regex scan of the backup object.
- 3 action assertions **click the real, rendered DOM buttons** (`document.getElementById('firstRunLogJob')?.click()`, etc.) after a genuine `render()` call, then assert on real modal state (`classList.contains('open')`, `textContent` heading match) or real `state.activeTab` — these are true integration-style fixtures, not stub/helper-only calls.
- 2 protected-boundary assertions check the release-identity footer and `STORAGE_KEY`/`SCHEMA_VERSION` constants.
- 1 backup/freshState-shape assertion confirms no onboarding-related key exists in either.

**Fixture cleanup:** the function's `finally` block calls a `restore()` helper that reassigns `state` from a `JSON.stringify` snapshot taken at entry, restores `localStorage` (including correctly handling the "was absent" case via `removeItem`), restores `firstRunOnboardingDismissed` and `state.activeTab`, and explicitly restores both `entryModal` and `gridModal`'s `innerHTML` and `open` class from pre-fixture snapshots (closing them first via `closeModal` to reset any modal-specific globals, then reinstating exact prior content/state) — followed by a final `render()` to reconcile the visible DOM with restored state. No `persist()` call exists anywhere in the new function (confirmed via targeted grep restricted to the function body), so running this fixture group can never write real fixture data into a user's actual `localStorage`. No hidden fixture side effect was found that could bias a later-running group.

## 13. Fixture-total reconciliation

Independently reconciled by **counting the actual `add(...)` calls in the diff**: exactly **19**. Only one runner (`runFirstRunOnboardingFixtures`) was added; no existing runner's assertion count changed in this diff (confirmed — the diff touches no other `run*Fixtures` function body). Prior verified baseline was `1909 / 0` (16 groups); this phase's arithmetic (`1909 + 19 = 1928`) is consistent with that.

**This was independently confirmed by live execution**, not only by arithmetic: the headless-Edge session (§11) called all 17 registered runner functions directly (`window.run*Fixtures()` for each) and summed their real return values:

| Group | Result |
|---|---|
| runBaselineResolutionFixtures | 20/0 |
| runMaterialTestNormalizationFixtures | 12/0 |
| runProductionSettingsFixtures | 66/0 |
| runEvidencePromotionFixtures | 58/0 |
| runDesignProductionSettingsFixtures | 118/0 |
| runGridPromotionFixtures | 23/0 |
| runTestGridMachineIdentityFixtures | 18/0 |
| runGridBrowserFixtures | 67/0 |
| runMaterialBrowserFixtures | 57/0 |
| runLibraryBrowserFixtures | 56/0 |
| runProjectBrowserFixtures | 61/0 |
| runProjectWizardFixtures | 216/0 |
| runDesignToProjectHandoffFixtures | 17/0 |
| runWizardMetadataFixtures | 12/0 |
| runStorageRecoveryFixtures | 15/0 |
| **runFirstRunOnboardingFixtures** | **19/0** |
| runDesignGeometryFixtures | 1093/0 |

**Total: 1928 passed / 0 failed.** Sum of non-Design groups: `20+12+66+58+118+23+18+67+57+56+61+216+17+12+15+19 = 835`; `835 + 1093 (Design) = 1928` — **exactly matches the README's claimed arithmetic**, confirmed both analytically and by genuine runtime execution. `runFirstRunOnboardingFixtures` executes exactly once under `selftest=all` (confirmed by source: registered once, no duplicate `if` line). No existing fixture was found deleted or weakened to preserve the total — every prior group's count matches its pre-phase value exactly (e.g., `runStorageRecoveryFixtures` remains 15/0, unchanged from the prior phase's audit).

## 14. Runtime/`file://` validation results

**Actual `file://` startup was performed**, not simulated and not approximated. A headless Microsoft Edge process (`msedge.exe --headless=new --disable-gpu --no-sandbox --remote-debugging-port=9333 --user-data-dir=<fresh scratch dir>`) was launched, and a new tab was opened via the Chrome DevTools Protocol's `PUT /json/new?<url>` endpoint with `url = file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` — the CDP-reported tab `url` confirmed this exact `file://` path was loaded (not a data URI, not a copied temporary file, not `localhost`). A WebSocket connection to the tab's `devtoolsFrontendUrl` was used to send `Runtime.evaluate` calls directly against the live page.

Results obtained by direct runtime evaluation (not claimed from static review):
- `document.readyState === "complete"`, `document.title === "Genmitsu L8 Tracker"` — the file parsed and executed with no fatal script error blocking `window`-level function exposure (all 17 `run*Fixtures` functions resolved successfully rather than throwing).
- **All 17 fixture groups executed, totaling 1928 passed / 0 failed** (table in §13), matching the README's claim exactly.
- **Zero duplicate IDs** across the entire rendered document (`document.querySelectorAll('[id]')` scanned for repeats — none found).
- **Release-identity footer** rendered with the exact expected text: `"Genmitsu L8 Tracker v0.9.0"` + `"About"` disclosure containing `"Build 2026-07-19 · Storage schema 2 · Backup format genmitsu-l8-tracker-backup-v1. Works offline and stores records locally in this browser."`
- On the **genuinely fresh** profile (empty `localStorage`), the **live, non-fixture** page showed `firstRunOnboarding` present on the real Log tab (`activeTabBefore: "log"`).
- A **real click** on the live `#firstRunPlanGrid` button moved `document.body.dataset.activeTab` to `"grids"` and opened `#gridModal` containing `/New test grid/i` — full end-to-end confirmation of the second starter action outside the fixture harness.
- **No horizontal overflow** at the default headless viewport (`741 ≤ 756`) nor at an emulated **320px** viewport (`305 ≤ 320`); the onboarding panel measured 269px wide, fully inside the 320px viewport, with all 4 buttons present.

Not performed in this session: a dedicated console-error/exception listener during initial page load (the successful, exception-free execution of all 17 fixture calls is strong indirect evidence of a clean load, but no `Log.enable`/`Runtime.consoleAPICalled` event stream was captured); direct keyboard-event (as opposed to `.click()`) activation of the buttons; tablet/large-desktop emulated widths. The headless Edge process was terminated after validation (`taskkill /F /PID 67480` succeeded).

`git diff --check` was run and is clean. HTML/inline-JS validity is additionally corroborated by the fact that the live page executed 1928 real assertions with zero thrown exceptions.

## 15. Documentation findings

`README.md`: the new bullet accurately states the five/six-collection eligibility rule (grouping `rawMaterials`+`finishedBatches` under "Inventory records," an accurate summary, not a discrepancy), accurately describes offline local-browser storage and the three starter actions, and explicitly states dismissal "lasts only for the current page session and is not saved to `localStorage` or JSON backups." The Built-in Checks section correctly updated to 17 groups / `1928 passed / 0 failed`, added `Storage recovery 15/0` and `first-run onboarding 19/0` breakdowns, corrected arithmetic to `835 + 1093 = 1928`, added `first-run` to the query-value list, and added `runFirstRunOnboardingFixtures()` to the exposed-functions list — all independently confirmed accurate against source and live execution (§13/§14). `CHANGELOG.md` adds one accurate bullet under the existing `0.9.0` entry describing the onboarding panel; **version remains 0.9.0** (correct per this phase's own contract, which explicitly expects retention, not a bump) with no 1.0/paid-release-complete claim anywhere. No stale fixture totals or contradictory backup instructions were found; the two backup-related sentences (header Export/Import tooltips from the prior phase, and this phase's onboarding copy) are consistent with each other.

## 16. Protected-boundary comparison

Confirmed by full diff review and a targeted grep for Designs-function identifiers that this implementation touches **none** of: `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`'s persisted shape, `loadState()`, `normalizeInventory()`'s semantics (read-only, unmodified), `backupObject()`, `validateBackupImport()`, `mergeData()`, `replaceData()`, `persist()`, corrupt-storage recovery, release-identity constants, the release footer/About content (its own render function is untouched; only a new `render()`-cycle guarantee was verified around it), Designs geometry/SVG serialization/downloads/Finished Views/panel dimensions/kerf-clearance calculations/production filenames, machine-profile semantics, Library promotion/evidence behavior, Project accounting, Inventory calculations, Pricing behavior, or external-dependency/network behavior (no new network/CDN/script-src construct was introduced). The Designs fixture group total (1093) is unchanged and was independently re-confirmed via live execution (§13). The entire diff is confined to: bounded CSS, the session-only dismissal variable, the pure eligibility helper, Log onboarding markup/bindings, fixtures, README, and CHANGELOG — exactly the expected change set.

## 17. Findings classified by severity

**BLOCKER:** none.

**IMPORTANT:** none.

**POLISH (safe to defer):**
- **P-1 — `firstRunOnboardingHtml()`'s trailing `|| state.entries.length` check is redundant.** `isFirstRunWorkspaceEmpty()` (no-arg, defaults to `state`) already requires `state.entries.length === 0`, so the explicit `|| state.entries.length` clause can never independently change the outcome. Harmless, purely a minor code-clarity nit. Boundary: `firstRunOnboardingHtml()` (`:1137`).
- **P-2 — "Plan a Test Grid" inlines `state.activeTab = 'grids'; render();` instead of calling `setTab('grids')`.** Confirmed functionally identical (§8); calling `setTab('grids')` directly would be marginally more maintainable (one call site instead of duplicated two-line logic) but is not materially safer. Boundary: `bindFirstRunOnboardingActions()` (`:1160`).
- **P-3 — Focus is not explicitly preserved/redirected after Dismiss.** Consistent with this app's existing full-render focus behavior elsewhere; not a regression introduced by this phase, and correctly out of scope of this bounded phase per the existing modal-accessibility roadmap item.
- **P-4 — Keyboard-specific (non-click) activation of the four buttons was not independently runtime-verified** (native `<button>` semantics make this exceptionally low-risk, but it was not directly exercised via a synthesized keydown in this session).

**NOT A DEFECT (deliberate, contract-endorsed):**
- Version remains `0.9.0` (explicitly expected by this phase's own contract).
- README's "Inventory records" phrasing summarizing both `rawMaterials` and `finishedBatches`.
- Onboarding recomputing eligibility fresh on every render rather than caching a "seen" flag (this is the correct, contract-required behavior — no permanent dismissal, re-eligibility after deletion).

## 18. Final verdict

**APPROVED TO COMMIT**

The implementation delivers the bounded First-run Onboarding and Backup Nudge phase completely and safely: eligibility detection is pure, side-effect-free, and covers exactly the six persisted record collections with no configuration value able to suppress it; the panel renders only in the genuine empty-Log-and-empty-workspace state and never for a search/filter miss or when other collections hold records; dismissal is a page-session-only variable with zero persistence/backup footprint, independently confirmed by both fixtures and direct inspection; all four starter actions route through existing, unmodified workflows with no invented defaults and no premature record creation; accessibility markup is semantic and native-control-based with no new burden; responsive behavior was live-verified at 320px and default width with no overflow; the fixture suite is genuinely integration-style (real clicks, real render calls) rather than logic-only stubs, its cleanup is thorough, and its reported 1928/0 total was independently reproduced by direct runtime execution against the actual `file://` path; and no protected boundary (storage, schema, backup, Designs, or release identity) was touched. Only minor, optional POLISH items remain, none of which affects correctness, data safety, or release readiness.

## 19. Remaining unverified areas

- Tablet (~700px) and large-desktop emulated-width visual checks were not separately run (no defect is expected given reliance on already-responsive shared classes, but this is not directly confirmed).
- A dedicated browser console-error/event listener during page load was not attached (the exception-free execution of all 17 fixture groups is strong indirect evidence, not a direct console-log capture).
- Direct keyboard (non-`.click()`) activation of the four buttons was not independently exercised.
- Creating an actual qualifying record (e.g., saving a real Log entry through the full form) and observing the onboarding panel disappear on the very next render was not independently runtime-exercised end-to-end in this session (it follows structurally from the no-memoization eligibility check, and is fixture-covered at the eligibility-helper level, but not as one continuous save→re-render live sequence).

## 20. Physical laser testing

**Not required.** This phase changes no geometry, dimensions, serializers, downloads, filenames, or production output; the protected-boundary comparison (§16) confirms zero Designs/production impact.

## 21. Whether Codex may proceed to commit

**Yes.** No blocking or important finding exists. The optional POLISH items (§17) may be applied now or deferred without any risk to data safety, persistence, or release readiness.

## 22. Audit integrity confirmation

This audit made **no edits** to any source or documentation file other than creating this report, and did **not** stage, commit, push, or otherwise alter git state. A temporary headless-Edge browser process was launched solely for read-only runtime verification against the repository's own `index.html` and was terminated at the end of the session; no repository file was modified by that process. The working tree remains exactly as the implementation left it (HEAD `4d51d29`, nothing staged, nothing pushed).
