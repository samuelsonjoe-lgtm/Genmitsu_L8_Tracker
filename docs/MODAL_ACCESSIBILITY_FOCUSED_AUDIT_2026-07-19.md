# Modal Accessibility — Focused Implementation Audit

**Date:** 2026-07-19  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Auditor mode:** Read-only (no product source edits; no stage/commit/push)  
**Implementation report reviewed:** `docs/MODAL_ACCESSIBILITY_IMPLEMENTATION_2026-07-19.md`  
**Expected committed baseline:** `efd5b81` — Add first-run onboarding and backup guidance  
**Expected pre-phase fixture baseline:** `1928 passed / 0 failed`

---

## 1. Repository state and actual baseline

| Check | Result |
| --- | --- |
| `git status -sb` | `## main...origin/main` with tracked modifications only on `index.html`, `README.md`, `CHANGELOG.md` |
| `git log -1 --oneline` | `efd5b81 Add first-run onboarding and backup guidance` |
| `git rev-parse HEAD` | `efd5b817f04b128750e78d3d560e4bfb91221629` |
| `git rev-list --left-right --count origin/main...main` | `0	0` (not behind, not ahead) |
| `git diff --check` | Clean (no conflict markers / whitespace errors reported as failures) |
| `git diff --stat` | `CHANGELOG.md` 1 line; `README.md` ~9 lines; `index.html` ~105 insertions / 10 deletions |
| `git diff --cached` | Empty — nothing staged |
| Commit / push during phase | None — phase remains uncommitted working-tree changes on top of `efd5b81` |

**Baseline confirmation**

- Actual HEAD matches the expected committed baseline subject and hash prefix `efd5b81`.
- The committed tree at HEAD is the first-run onboarding baseline; Modal Accessibility exists only as uncommitted working-tree changes.
- Branch is `main`, synchronized with `origin/main` (0/0).
- Staging area is empty; no commit or push occurred for this phase.

**Modified tracked files (phase delta)**

1. `index.html` — modal accessibility helpers, open/close wiring, fixtures, startup order.
2. `README.md` — fixture totals and modal selftest documentation.
3. `CHANGELOG.md` — 0.9.0 modal-accessibility bullet.

**Relevant new untracked file from the phase**

- `docs/MODAL_ACCESSIBILITY_IMPLEMENTATION_2026-07-19.md`

**Unrelated untracked material left untouched**

- `LightBurn Projects/` (many `.lbrn2` / `.lbt` files)
- `debug.log`
- Historical `docs/*` review/audit/verification reports
- `parametric_qr_stand_generator.py`

**Conclusion:** The pre-phase committed baseline is clean at `efd5b81`. The Modal Accessibility work is confined to the three tracked files plus the implementation note. Unrelated LightBurn projects, utilities, debug files, and historical reports remain untouched.

---

## 2. Complete modal inventory

### Markup containers (source)

Exactly **eleven** `.modal` containers exist in `index.html`:

| # | ID |
| --- | --- |
| 1 | `entryModal` |
| 2 | `profileModal` |
| 3 | `projectModal` |
| 4 | `inventoryModal` |
| 5 | `gridModal` |
| 6 | `cellModal` |
| 7 | `testModal` |
| 8 | `productionSettingModal` |
| 9 | `productionEvidenceModal` |
| 10 | `materialTestWizardModal` |
| 11 | `projectWizardModal` |

This matches the implementation report.

### Workflow coverage of those containers

| Workflow | Container | Open path |
| --- | --- | --- |
| New/Edit Log Entry | `entryModal` | `openEntry` → `openModal` |
| New/Edit Library profile | `profileModal` | `openProfile` → `openModal` |
| New/Edit Project | `projectModal` | `openProject` → `openModal` |
| Inventory raw / finished | `inventoryModal` | `openRawMaterial` / `openFinishedBatch` → `openModal` |
| New/Edit Test Grid | `gridModal` | `openGridForm` → `openModal` |
| Grid cell result | `cellModal` | `openCell` → `openModal` |
| Project photo viewer | `cellModal` | `openProjectPhotoViewer` → `openModal` + `bindModal` |
| Material test add/edit | `testModal` | `openMaterialTest` → `openModal` |
| Production setting add/edit/supersede | `productionSettingModal` | `openProductionSetting` / `openSupersedeProductionSetting` → `openModal` |
| Evidence / promotion review | `productionEvidenceModal` | `openProductionEvidence`, promotion openers → `openModal` |
| Material Test Wizard + recommendation | `materialTestWizardModal` | `openMaterialTestWizard` / `openNextRecommendedTest` → `openModal` |
| Project Wizard | `projectWizardModal` | `renderProjectWizard` / `openProjectWizard` → `openModal` |

### Search results (bypass risk)

| Pattern | Finding |
| --- | --- |
| `classList.add('open')` | **One** production site — inside `openModal` |
| `classList.remove('open')` | **One** production site — inside `closeModal` |
| `openModal(` | 27 call sites (helpers + workflows + fixtures) |
| `closeModal(` | 38 call sites |
| `bindModal(` | Shared Cancel/× wiring via `[data-close]` |
| Backdrop-click dismissal | **None** present; none added |
| Document `keydown` for shortcuts | Single `document.addEventListener('keydown', handleShortcuts)` |
| Extra keydown | One local input listener (~line 11052) unrelated to modal trapping |
| Photo viewer | Reuses `cellModal` through shared open/close |
| Role/dialog markup in static HTML | Containers are empty shells; semantics applied at open time |

**Inventory completeness:** The implementation report’s eleven-container list is **complete**. No separate modal-like overlay bypasses the shared infrastructure. Photo viewing is modal-adjacent but correctly reuses `cellModal`.

---

## 3. Files and functions reviewed

**Files**

- `index.html` (implementation + surrounding openers, wizards, forms, startup, fixtures)
- `README.md`
- `CHANGELOG.md`
- `docs/MODAL_ACCESSIBILITY_IMPLEMENTATION_2026-07-19.md`
- `git diff` vs `efd5b81` for protected-boundary comparison

**Core functions**

- `modalFocusOrigins`, `modalOpenOrder` (page memory)
- `modalDialog`, `modalHeadingId`, `modalFocusableElements`
- `focusModalInitial`, `prepareModalAccessibility`, `restoreModalFocus`, `trapModalFocus`
- `openModal`, `closeModal`, `activeModal`, `bindModal`
- `handleShortcuts` (Tab trap + Escape)
- `initializeModalAccessibility`
- Startup block: `initializeModalAccessibility` → `renderReleaseIdentity` → `render` → optional selftests
- `runModalAccessibilityFixtures` + `?selftest=modal`
- Project Wizard: `renderProjectWizard`, `openProjectWizard`, `cancelProjectWizard`, `handoffProjectWizard`
- Material Test Wizard: open/render/close paths
- Entry/profile/project/inventory/grid/cell/test/production openers
- `backupObject`, `freshState`, storage identity constants
- Selftest registry at end of `index.html`

**Independent runtime harness** (temp hook copy only; product tree not modified for execution): Playwright Chromium against `file:///…/index.html`.

---

## 4. Implementation summary

The phase centralizes dialog accessibility in the shared open/close path:

1. **Open:** capture focus origin (per modal, only when not already open), inject HTML, mark open, push `modalOpenOrder`, apply dialog ARIA on `.dialog`, set container `aria-hidden="false"`, move focus inside.
2. **Close:** clear open class, set `aria-hidden="true"`, clear `innerHTML`, drop open-order entry, restore focus to origin or active tab fallback.
3. **Keyboard:** existing single document listener now traps Tab while a modal is active and continues to Escape-close the topmost modal through `closeModal`.
4. **Fixtures:** 25 assertions under `runModalAccessibilityFixtures`, registered as `?selftest=modal` and included in `?selftest=all`.
5. **Startup:** render (including release identity) before optional selftests so fixtures observe the same first-paint surfaces as normal use.
6. **Docs:** README totals → 1953; CHANGELOG 0.9.0 notes modal accessibility without claiming full a11y completion.

Scope is intentionally bounded: no storage/schema change, no Designs geometry change, no new backdrop-click behavior, no full-page `inert` adoption.

---

## 5. Dialog-semantics findings

**Contract on open (confirmed on production Entry and all eleven containers via fixtures):**

| Attribute | Target | Value |
| --- | --- | --- |
| `role` | `.dialog` panel | `dialog` |
| `aria-modal` | `.dialog` | `true` |
| `tabindex` | `.dialog` | `-1` |
| Accessible name | `.dialog` | `aria-labelledby` → `{modalId}-heading` when an `h1–h3` exists |
| Exposure | `.modal` container | `aria-hidden="false"` when open |

**Closed state**

- Startup + post-close: every container has `aria-hidden="true"`.
- Closed containers have empty `innerHTML` after `closeModal`, so stale dialog roles do not remain in the DOM.
- Independent probe: closed inventory shows `role`/`aria-modal` as null (no `.dialog` present) and `aria-hidden="true"`.

**Semantics placement**

- Correctly applied to the **panel** (`.dialog`), not only the backdrop/container.

**Fallback name**

- If no heading exists, `aria-label="Dialog"` is set. Production workflows inspected always supply an `h2`. Generic label is a safety net, not the production naming path.

**Verdict:** Dialog semantics meet the phase contract. No inaccessible or incorrectly named production dialogs found.

---

## 6. Dialog-name and heading-ID findings

- Heading ID scheme: `{modalId}-heading` via `modalHeadingId(id)`.
- Assignment uses the first `h1,h2,h3` inside the dialog and rewrites its `id` on each open/`prepareModalAccessibility` call.
- Production Entry probe: heading text `New log entry`, id `entryModal-heading`, `aria-labelledby` matches.
- Project Wizard probe: `projectWizardModal-heading` / “Project Wizard”.
- Wizard step rerenders call `openModal` again, which re-prepares accessibility; heading id is reassigned cleanly rather than duplicated across detached nodes (prior content is replaced via `innerHTML`).
- Photo viewer uses `cellModal` with a project/photo `h2` title — receives `cellModal-heading`.
- Recommendation UI on `materialTestWizardModal` uses `h2` “Next Recommended Test”.
- Duplicate-ID fixture after modal lifecycle: **pass**.

**Risks considered and not confirmed as defects**

- Targeting non-heading elements: not observed; selector is heading-only.
- Stale headings: mitigated by `innerHTML` replacement on open/close.
- Multiple headings: first heading wins; current templates use a single primary title heading.

---

## 7. Focus-origin findings

- Stored in `modalFocusOrigins` (`Map`), keyed by modal id.
- Captured in `openModal` only when the modal is **not already open** (`if (!m.classList.contains('open'))`).
- Page-memory only; absent from `freshState()`, `backupObject()`, and localStorage (probe confirmed).
- Not serialized; Map holds live element references for the session only.
- Production Entry probe: origin was the audit trigger button.
- Repeated open after close updates origin again.
- Re-open while already open does not overwrite origin (safe; avoids thrashing during internal re-renders that might call open paths incorrectly). Project Wizard re-opens via full `openModal` replace while already open — origin retention is acceptable because the original opener remains the correct restore target.

---

## 8. Initial-focus findings

Priority implemented:

1. First focusable with `autofocus`
2. Else first focusable from helper
3. Else dialog panel (`tabindex="-1"`)

Fixtures cover all three paths (**pass**). Production Entry focuses a control inside the dialog (button/control, not destructive Delete as sole target when safer controls exist — Entry form has many earlier fields/controls).

`focus({ preventScroll: true })` is used to avoid jumpy scrolling.

**No evidence** of initial focus altering values, submitting forms, or triggering validation on open.

---

## 9. Focusable-element-helper findings

Selector:

`a[href], button, input, select, textarea, [tabindex]`

Exclusions:

- `disabled`
- `input[type="hidden"]`
- `[tabindex="-1"]`
- Descendants of `[hidden]` or `[aria-hidden="true"]`
- Elements with `getClientRects().length === 0`

**Production Entry:** 43 focusable controls collected under headless Chromium — helper is not empty and not over-filtered for real forms.

**Includes** help tips with `tabindex="0"` (intentional app pattern).

**Does not include** the dialog panel itself in the focusable list (correct; panel is fallback only).

**Theoretical edge (not a confirmed app bug):** zero-rect but legitimately focusable offscreen controls would be excluded. No real control in this app was found missing under headless/file:// probes.

---

## 10. Tab / Shift+Tab containment findings

| Method | Result |
| --- | --- |
| Fixture synthetic `KeyboardEvent` Tab/Shift+Tab/outside | Pass (combined assertion) |
| Independent synthetic on production Entry | Tab last→first, Shift+Tab first→last, outside→inside: **pass** |
| Playwright `page.keyboard` Tab / Shift+Tab | **Native automation key input** — wrap to first / last confirmed |
| Direct helper-only claims | Not used as sole evidence |

Trap applies only when `activeModal()` is non-null. Middle-of-list Tab does not `preventDefault` (browser natural order). No second document-level modal trap listener.

**Key property:** `event.key === 'Tab'` / `'Escape'` (modern, reliable).

---

## 11. Multiple-modal / topmost findings

- `modalOpenOrder` tracks open sequence; `activeModal()` uses `findLast` open id, with DOM-order fallback.
- Re-open moves id to end without unbounded duplicates.
- Independent stack probe: Entry under Profile → Escape closes only Profile → Entry remains open, order `['entryModal']`, focus returns inside Entry.
- Topmost Tab trap fixture: **pass**.
- Nested production modals are uncommon; stacking remains safe without over-engineering.

**Focus restoration when closing topmost:** `restoreModalFocus` always runs. When the origin is inside the underlying modal (typical when the second modal opened while focus was in the first), focus correctly returns into the still-open dialog. Confirmed by stack probe (`focusInEntry: true`).

---

## 12. Escape findings

- Escape closes **only** `activeModal()` via `closeModal`.
- One keypress → one close (stack probe).
- No residual per-modal Escape listeners found that would double-fire.
- Focus restoration runs as part of `closeModal`.
- Escape with no modal still supports pre-existing Test Grid drill-out behavior.
- Native Playwright Escape: closed Entry and restored trigger.

**Pre-existing note (NOT A DEFECT of this phase):** promotion-specific cancel helpers are not always invoked on Escape; Escape uses shared `closeModal`. That behavior predates this phase’s Escape routing through the same shared close path. No new double-close or focus lockout introduced.

---

## 13. Close-path findings

| Path | Routes through shared close? |
| --- | --- |
| `[data-close]` Cancel / × via `bindModal` | Yes → `closeModal` |
| Successful form saves (Entry, etc.) | Yes → `closeModal` |
| Programmatic close | Yes |
| Escape | Yes |
| Project Wizard Cancel / completion | Yes (`cancelProjectWizard` / `handoffProjectWizard` → `closeModal`) |
| Material Test Wizard cancel/save | Yes |
| Photo viewer close | Yes via `bindModal` |
| Backdrop click | **Not supported** (confirmed absent; not classified as a defect) |

`closeModal` always updates ARIA, open order, and focus.

---

## 14. Focus-restoration findings

| Scenario | Result |
| --- | --- |
| Opener still connected/visible | Restored (Escape, Cancel, native Esc probes) |
| Opener removed | Falls back to `.tab.active` (fixture **pass**) |
| Fallback missing | Guarded (`if (target)`); no throw |
| After successful Entry save | Modal closes; restore runs; fixture asserts save path still works |
| Stack close | Restores into remaining modal when origin is there |

**Ordering:** close clears modal HTML then restores focus. Post-save `render()` may replace list rows; fallback to active tab covers detached openers. No focus-lockout observed.

---

## 15. ARIA / background-exposure findings

- Closed: `aria-hidden="true"` on all containers at startup and after close.
- Open: container `aria-hidden="false"`; dialog panel not aria-hidden.
- No permanent aria-hidden applied to the main application shell.
- Phase relies on `aria-modal="true"` + focus containment rather than document-wide `inert`.

**Assessment:** Sufficient for this **bounded** phase. Lack of `inert` is **NOT A DEFECT** absent a confirmed interaction bug. Screen-reader background reading was not AT-tested (see remaining verification).

---

## 16. Project Wizard and Material Test Wizard findings

**Project Wizard**

- Opens through `openModal` → shared prepare/focus.
- `openProjectWizard` still performs an additional post-open focus query:

  `document.querySelector('#projectWizardModal input:not([disabled]), …')?.focus()`

  This is a **second focus jump** after `focusModalInitial`. Probe still shows focus **inside** the dialog with correct role/name. Classified as **POLISH** (redundant, not broken).
- Cancel → `closeModal`.
- Step changes re-call `renderProjectWizard` → `openModal` (full replace); focus re-enters via shared path.
- Fixture: initial focus inside accessible dialog — **pass**.

**Material Test Wizard**

- Same shared open/close infrastructure and heading pattern.
- Covered by eleven-container semantics fixtures; not given a dedicated step-by-step focus fixture (coverage gap → polish / residual verification, not a confirmed defect).

---

## 17. Form / workflow regression findings

| Workflow | Evidence |
| --- | --- |
| Invalid Entry save | Fixture: blocked; state/storage unchanged — **pass** |
| Valid Entry save | Fixture: persists then cleanup restores — **pass** |
| Cancel without save | Probe restore — **pass** |
| Project Wizard open | Probe + fixture — **pass** |
| Production settings / promotion groups | `66/0`, `58/0` |
| First-run actions opening modals | First-run fixtures `19/0` including “Log your first job opens New log entry” |
| Designs production outputs | Design geometry `1093/0`; sample digests generated without error |

No indication that heading ids or `tabindex="-1"` on panels broke form handlers. Sticky action regions present at 320 px.

---

## 18. Startup / selftest-order findings

Order now:

1. `document.addEventListener('keydown', handleShortcuts)`
2. `initializeModalAccessibility()`
3. `renderReleaseIdentity()`
4. `render()`
5. Optional `?selftest=…` runners

**Rationale:** Fixtures that assert release footer / onboarding need a normal render first.

**Checks**

- Direct `file:///index.html` (no selftest): tabs present, footer present, all modals closed with `aria-hidden="true"`, no page exceptions.
- Selftests still gated on query param.
- Fresh-profile first-run banner + About footer both present.
- No storage-recovery behavior change observed (`storage` fixtures `15/0`).

Unintended flash/regression: **not observed** in headless direct startup.

---

## 19. Session / storage-safety findings

| Check | Result |
| --- | --- |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` (unchanged) |
| `SCHEMA_VERSION` | `2` (unchanged) |
| `APP_VERSION` | `0.9.0` |
| `BUILD_DATE` | `2026-07-19` |
| Origins / order in `freshState` | Absent |
| Origins / order in `backupObject` | Absent |
| New storage keys | Only existing key in localStorage during probes |
| Migration | None added |
| Fixture storage restoration | Modal fixture restores `localStorage` + state |

Diff does not modify `freshState`, `loadState`, normalization, `backupObject`, import validation, merge/replace, or `persist` semantics beyond reading/restoring keys inside fixtures.

---

## 20. Fixture-quality findings

`runModalAccessibilityFixtures` — **25 assertions**, all **pass**:

| # | Coverage area | How encoded |
| --- | --- | --- |
| 1–11 | Complete inventory + dialog roles/names/aria-hidden open | One assertion per modal id |
| 12 | Closed aria-hidden | Single all-modal check |
| 13 | Trigger capture + autofocus priority | Combined |
| 14 | First-focusable fallback | Dedicated |
| 15 | Dialog-panel fallback | Dedicated |
| 16 | Tab + Shift+Tab + escaped focus | Combined synthetic keyboard |
| 17 | Escape close + restore | Dedicated |
| 18 | Cancel restore | Dedicated |
| 19 | Removed-trigger fallback | Dedicated |
| 20 | Topmost Tab trap | Dedicated |
| 21 | Invalid Entry validation | Production form |
| 22 | Valid Entry save | Production form (restored after) |
| 23 | Project Wizard focus | Production wizard |
| 24 | Release footer + storage identity | Dedicated |
| 25 | No duplicate IDs | Dedicated |

**Meaningful combinations:** The suite packs more than 25 micro-points into 25 assertions (e.g., Tab/Shift+Tab/escape-focus together; 11 modal semantics loops). That is acceptable and still maps to the contract.

**Weaknesses (quality, not failures)**

- Trap tests use **synthetic dialog HTML** rather than every production form layout.
- Keyboard assertions use **synthesized `KeyboardEvent`** inside the fixture (still hit the real document listener).
- Independent audit **also** exercised production Entry + Playwright native keyboard — mitigating the above.
- Material Test Wizard step navigation not deeply fixture-covered.
- No assertion that Escape leaves an underlying modal’s form values intact beyond stack open-state checks.

**Self-fulfilling risk:** Low for Entry validation/save and wizard paths (real functions). Higher for pure synthetic trap markup — mitigated by external probes.

---

## 21. Fixture-cleanup findings

`restore()` in the modal fixture:

- Restores `state` from JSON snapshot
- Restores or removes `localStorage` item for `STORAGE_KEY`
- Restores `modalFocusOrigins` and `modalOpenOrder`
- Calls `render()`
- Restores each modal’s HTML, open class, and aria-hidden snapshot
- Removes synthetic trigger; restores prior `document.activeElement` when connected

Valid-save residue is cleared by state/storage restore. No evidence of leftover open modals, persisted fixture entries, or onboarding suppression after the group completes (subsequent groups in the audit run still passed).

---

## 22. Fixture-total reconciliation

### Independent group runs (hooked page)

| Group | Passed | Failed |
| --- | --- | --- |
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| production | 66 | 0 |
| promotion | 58 | 0 |
| design-production | 118 | 0 |
| grid | 23 | 0 |
| grid-machine | 18 | 0 |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| design-project | 17 | 0 |
| metadata | 12 | 0 |
| storage | 15 | 0 |
| first-run | 19 | 0 |
| **modal** | **25** | **0** |
| project-wizard | 216 | 0 |
| design | 1093 | 0 |
| **Complete 18-group sum** | **1953** | **0** |
| tray-model (callable subset) | 264 | 0 |

Arithmetic:

- `1928 + 25 = 1953` ✓  
- `860 non-Design + 1093 Design = 1953` ✓  
- Tray `264` is **inside** Design geometry coverage, not an extra complete-suite addend.

### `?selftest=all` console

- Modal selftest alone: `25 passed / 0 failed`.
- Full console listed every expected fixture group header including **Modal accessibility fixtures** and **Design geometry fixtures** with `0 failed` lines throughout.
- Naïve sum of all `N passed` console lines was **2037** because Material Browser intermediate fixture functions also log when their modules load; the **final** materials total is **57**, and stripping intermediate material logs yields **1953**.

### README

- Documents 18 groups, `1953 passed / 0 failed`, modal `25 / 0`, Designs `1093 / 0`, tray `264 / 0`, `860 + 1093 = 1953` — **consistent with source and independent runs**.

### Registry

- Modal runs exactly once under `selftest=all` (single `if (selftest === 'modal' || selftest === 'all')` line).
- No prior group deleted or weakened in the diff; only modal added and README arithmetic updated.

---

## 23. Runtime / file:// validation

| Check | Result | Method class |
| --- | --- | --- |
| `git diff --check` | Clean | Source |
| HTML parse (`html.parser`) | OK | Source |
| Inline JS load | No page exceptions on startup | Browser |
| Modal fixtures | **25 / 0** | Direct function + `?selftest=modal` |
| Complete suite groups | **1953 / 0** | Independent runners + console for `all` |
| Tray-model | **264 / 0** | Direct function |
| Designs geometry | **1093 / 0** | Direct function |
| Direct `file:///index.html` | Tabs, footer, closed aria-hidden | Browser |
| Direct `?selftest=all` | Groups green; modal 25 | Browser console |
| Duplicate IDs | Fixture pass | DOM |
| Page exceptions | None in audit probes | Browser |
| Console | Pre-existing SVG `height="auto"` noise only | Browser |
| Tab wrap | Pass | Synthetic + **Playwright keyboard** |
| Shift+Tab wrap | Pass | Synthetic + **Playwright keyboard** |
| Focus escape recovery | Pass | Synthetic |
| Escape close | Pass | Synthetic + **Playwright keyboard** |
| Cancel restore | Pass | DOM click |
| Save path | Pass | Fixture |
| Repeated open/close | Exercised by fixtures/probes | Mixed |
| Simple modal (Entry) | Pass | Production opener |
| Wizard modal | Pass | Production opener |
| Multi-modal topmost | Pass | Probe |
| 320 px layout | Dialog ~304 px, no overflowX | Browser |
| Release footer / About | Present | Browser |
| First-run fresh profile | Banner + footer | Browser + cleared storage |
| Protected Designs outputs | Geometry fixtures green; sample builds valid | Function |

**Keyboard method legend (explicit):**

- **Direct helper invocation:** focusable enumeration, open/close.
- **Synthesized `KeyboardEvent`:** fixture trap tests + several probes.
- **Playwright `page.keyboard`:** native automation key input for Tab / Shift+Tab / Escape (not CDP raw; not OS-level hardware).
- **Source inspection:** listener wiring, inventory, storage.

This audit does **not** claim physical keyboard hardware testing or screen-reader AT testing.

---

## 24. Responsive findings (bounded)

At **320×720**:

- Modal width 320; dialog width ~304
- No horizontal document overflow detected
- Heading visible
- Action region present

Desktop width not fully re-profiled beyond normal headless default; no modal regression indicators from fixtures.

This is **not** a general responsive-polish audit.

---

## 25. Documentation findings

| Claim | Assessment |
| --- | --- |
| Dialog semantics / focus containment / Escape / restore described | Accurate vs source |
| Screen-reader testing claimed | **Not claimed** — good |
| General accessibility completion claimed | **Not claimed** — good |
| Version remains 0.9.0 | Yes |
| BUILD_DATE unchanged inappropriately | Still `2026-07-19` with release identity |
| 1.0 / paid-release readiness claimed | No |
| Fixture totals | Match 1953 / modal 25 / Designs 1093 |

CHANGELOG 0.9.0 bullet accurately scopes the change.

---

## 26. Protected-boundary comparison

From `git diff` vs `efd5b81` and runtime checks, the phase **did not** alter:

- `STORAGE_KEY` / `SCHEMA_VERSION` values
- `freshState()` / `loadState()` / normalization / `backupObject()` structure
- `validateBackupImport` / merge / replace / `persist` behavior
- Corruption recovery logic
- Release identity content constants (footer still renders About)
- First-run eligibility rules (fixtures still green)
- Modal field defaults/validation semantics (Entry invalid/valid paths green)
- Designs geometry builders, SVG serialization, downloads, Finished Views
- Machine-profile semantics, Library promotion rules, Project accounting, Inventory math, Pricing
- External network/dependencies

Diff touches: modal helpers, `openModal`/`closeModal`/`activeModal`/`handleShortcuts`, startup order, modal fixtures, README/CHANGELOG text.

`classList.add/remove('open')` remains solely inside shared helpers.

---

## 27. Findings classified by severity

### BLOCKER

*None.*

### IMPORTANT

*None confirmed.*

### POLISH

1. **Project Wizard redundant focus** — `openProjectWizard` re-focuses after shared `focusModalInitial`. Still inside dialog; safe to defer.
2. **Generic `aria-label="Dialog"` fallback** — vague if a future modal lacks a heading; production paths currently always provide `h2`.
3. **Fixture trap markup is synthetic** — acceptable for unit-style checks; production Entry covered by independent probes. Optional future expansion: trap assertions on a real Entry form.
4. **Material Test Wizard step-focus** — not deeply fixture-covered beyond container semantics.
5. **No document `inert` / main-content aria-hidden** — optional future enhancement; not required for this phase’s stated contract.

### NOT A DEFECT

1. **No backdrop-click dismissal** — pre-existing product choice; not introduced as a regression.
2. **Escape uses shared `closeModal` rather than promotion-specific cancel helpers** — pre-existing pattern; not a modal-a11y regression.
3. **Absence of full screen-reader / AT certification** — out of scope; docs do not overclaim.
4. **Tray-model 264 not added on top of 1953** — tray is nested under Designs geometry.
5. **SVG `height="auto"` console messages** — pre-existing noise; unrelated to modals.

---

## 28. Exact final verdict

# APPROVED WITH POLISH DEFERRED

The Modal Accessibility phase **safely and completely** delivers the bounded contract: dialog semantics, naming, focus entry, Tab containment, Escape, restoration, topmost ordering, storage safety, workflow non-regression, and fixture accounting (`1928 + 25 = 1953`). No blockers or important correctness defects require a fix before commit. Remaining items are polish or out-of-scope AT work.

---

## 29. Remaining unverified areas

- Physical keyboard on real OS focus rings with assistive tech attached
- Screen readers (NVDA/JAWS/VoiceOver) announcement quality
- Every modal workflow’s full create/edit/delete path under keyboard-only use (sample set exercised; not exhaustive manual matrix)
- Material Test Wizard multi-step focus after each Next/Back under native keyboard
- Save-then-render focus when the opener was a table row that disappears from a filtered list (fallback path unit-tested; not every browser surface)
- Touch / mobile OS browser quirks beyond 320 px layout probe
- Visual focus-ring contrast audit (separate roadmap)

---

## 30. Physical laser testing required?

**No.** This phase does not change Designs geometry, production SVG bytes, machine profiles, kerf, or cut paths. Laser testing is not required for Modal Accessibility sign-off.

---

## 31. Whether Codex may proceed to commit

**Yes.** Codex may commit the three tracked files (`index.html`, `README.md`, `CHANGELOG.md`) plus, if desired, the implementation note already present. No further code correction is required before commit for this phase.

Suggested commit scope (for the implementer; **not performed by this audit**):

- Include only the Modal Accessibility tracked changes.
- Do not add unrelated untracked LightBurn/debug/historical docs.

---

## 32. Audit hygiene confirmation

This audit:

- Inspected the real repository at `C:\Genmitsu L8 Tracker`
- Ran read-only git, HTML parse, and browser fixture/probe commands
- Used a **temporary hooked copy** of `index.html` outside the product tree for some probes
- **Did not** edit, stage, commit, push, reset, clean, stash, checkout, move, rename, or delete any product source file
- Writes only this report: `docs/MODAL_ACCESSIBILITY_FOCUSED_AUDIT_2026-07-19.md`

---

## Appendix A — Key implementation references

Shared helpers live near the prior `openModal` / `closeModal` site:

- `modalFocusOrigins` / `modalOpenOrder` — page-session only
- `prepareModalAccessibility` — role, aria-modal, tabindex, labelledby/label, container aria-hidden
- `trapModalFocus` — Tab wrap / recapture
- `handleShortcuts` — Tab trap then Escape → `closeModal(activeModal.id)`
- Startup: `initializeModalAccessibility(); renderReleaseIdentity(); render();` then selftests

---

## Appendix B — Runtime summary snapshot

```
modal:              25 passed / 0 failed
first-run:          19 / 0
storage:            15 / 0
production:         66 / 0
promotion:          58 / 0
design:           1093 / 0
tray-model:        264 / 0
complete (18):    1953 / 0
native Tab wrap:   pass
native Esc:        pass
stack Escape:      topmost only; focus remains in underlying
entry ARIA:        role=dialog, aria-modal=true, labelledby=entryModal-heading
STORAGE_KEY:       genmitsu-l8-tracker-v1
SCHEMA_VERSION:    2
page exceptions:   none
```

---

*End of focused audit report.*
