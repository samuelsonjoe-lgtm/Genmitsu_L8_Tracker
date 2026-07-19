# Modal Accessibility Implementation — 2026-07-19

## Baseline and repository state

- Actual HEAD: `efd5b817f04b128750e78d3d560e4bfb91221629` — `Add first-run onboarding and backup guidance`.
- Branch: `main`; `origin/main...main` was `0 / 0` (behind/ahead).
- Before editing, the tracked tree was clean and the index was empty. The pre-existing untracked `.claude/`, `LightBurn Projects/`, `debug.log`, historical `docs/` reports, and `parametric_qr_stand_generator.py` were left untouched.
- The baseline source contained the release-identity, import-safety, and first-run onboarding work. Its complete fixture baseline was `1928 / 0`.

## Changed files

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/MODAL_ACCESSIBILITY_IMPLEMENTATION_2026-07-19.md`

## Modal inventory and existing paths

The application has eleven modal containers: `entryModal`, `profileModal`, `projectModal`, `inventoryModal`, `gridModal`, `cellModal`, `testModal`, `productionSettingModal`, `productionEvidenceModal`, `materialTestWizardModal`, and `projectWizardModal`.

All normal modal workflows populate their `.dialog` panel through the shared `openModal(id, html)` helper and dismiss through `closeModal(id)`; ordinary close and Cancel controls are wired by `bindModal(id)`. This covers Log entries, Library/material profiles and tests, Projects, Inventory, Test Grids and cells, production settings/evidence/promotion, material-test recommendations, and the photo viewer. The Project Wizard intentionally owns its Cancel controls and rerenders its body, but still opens through `openModal()` and closes through `closeModal()`.

No separate modal bypass or direct modal open/close class mutation was found outside the shared helper path. There is no existing backdrop-click dismissal handler, so none was added or changed.

## Accessibility behavior

`openModal()` now records the current focused element in page memory, installs dialog semantics on the actual `.dialog` panel, exposes the containing modal with `aria-hidden="false"`, records it as active, and moves focus inside. Every panel receives `role="dialog"`, `aria-modal="true"`, `tabindex="-1"`, and a stable heading-derived `aria-labelledby` value (`<modal-id>-heading`). All current modal content has a visible heading; no fallback `aria-label` is used by production workflows.

Focusable elements are collected by one shared helper. It includes links with `href`, buttons, inputs, selects, textareas, and non-negative `tabindex` elements, while excluding disabled controls, hidden inputs, negative-tabindex controls, hidden/aria-hidden descendants, and elements without rendered client rects.

Initial focus prefers `autofocus`, then the first eligible control, then the dialog panel itself. The document's existing single keyboard listener traps Tab only while an active dialog exists: Tab wraps from last to first, Shift+Tab wraps from first to last, and focus that escapes is returned to the active dialog. `modalOpenOrder` ensures that only the most recently opened visible modal traps focus.

Escape retains its existing behavior, now closing only that active/topmost dialog through the shared close path. Closing restores focus to the recorded opener when it is still connected, visible, and enabled; otherwise it uses the current selected tab button. Origins and open order are page-session `Map`/array data only and are neither persisted nor included in backups. The Project Wizard's old post-render focus jump is reconciled by the shared focus setup and remains inside its dialog.

Startup rendering now happens before optional self-test execution, so first-run fixture checks observe the release footer just as a normal file-open does. Normal application startup behavior is unchanged.

## Fixtures and validation

Added `runModalAccessibilityFixtures()` and `?selftest=modal`. Its 25 assertions cover all eleven container semantics/names, closed-dialog exposure, autofocus/first-control/panel fallback, Tab and Shift+Tab wrapping, escaped focus, Escape, Cancel, removed-opener fallback, topmost-modal trapping, invalid and valid existing Entry form paths, Project Wizard focus, release/storage identity, duplicate IDs, and fixture state/storage restoration.

Validation completed:

- `git diff --check` — clean.
- `python -m html.parser index.html` — passed.
- Direct headless Edge `file:///.../index.html` — rendered tabs, Designs, and release footer; no page exceptions.
- Focused headless Edge modal check at 320 px — `25 / 0`; actual Entry and Project Wizard focus/restoration passed; Entry dialog width was 274 px with no horizontal overflow.
- Complete headless Edge `file:///.../index.html?selftest=all` — `1953 / 0`: `860` non-Design assertions plus `1093` Designs assertions. Browser console reported no exceptions and the duplicate-ID check passed.
- Existing tray-model result — `264 / 0` (within Designs coverage); complete Designs geometry — `1093 / 0`.
- Existing production-settings and Designs production-output fixture groups remained green (`66 / 0` and `118 / 0` respectively).

The fixture temporarily exercises a successful form save but restores state, localStorage, modal contents, classes, ARIA state, focus origin/order, and focus before returning. Direct before/after checks confirmed the fixture leaves storage unchanged.

## Protected boundaries

No changes were made to `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`, `loadState()`, normalization, `backupObject()`, import validation, merge/replace, `persist()`, corruption recovery, release identity contents, or first-run eligibility/storage behavior. Modal field values, validation rules, saves, records, machine semantics, Library/promotion behavior, Project accounting, Inventory, Pricing, external dependencies, and network behavior are unchanged.

No Designs geometry, SVG serializer, download, filename, Finished View, production-output, or production-byte generation code changed. Existing production and Designs fixtures remained green; production bytes are unchanged by this phase.

## Remaining verification

The browser harness validates keyboard events and direct-file behavior, but this pass did not use a physical screen reader or manual assistive-technology session. General tab-role semantics, global focus styling, contrast, and touch-target work remain separate roadmap items.

## Final state

Only the four files listed above were changed by this phase. Nothing was staged, committed, or pushed.
