# Accessibility Polish Implementation — 2026-07-19

## Scope and baseline

Implemented the bounded release-readiness accessibility polish on baseline `23b4706` (`Add optional machine setup and work area`). The work is limited to primary navigation semantics and keyboard behavior, focus visibility, bounded touch-target sizing, documentation, and focused fixtures. It does not alter storage, schema, release identity, Designs geometry, production SVG output, import/export, machine setup behavior, or application data semantics.

## Primary navigation

The existing primary navigation is now a labelled `role="tablist"` (`Tracker sections`). Each rebuilt primary control is a real `button` with a stable `tab-*` ID, `role="tab"`, `aria-selected`, `aria-controls="app"`, and roving `tabindex`. The already-shared `#app` region is now the single `role="tabpanel"`; its `aria-labelledby` follows the active tab. No empty or duplicate tab panels were added.

When actual primary-tab focus is present, Left/Right Arrow wraps and activates the adjacent section, Home/End activates the first/last section, and Enter/Space retain activation on the focused tab. Keyboard-triggered navigation restores focus to the newly rendered active tab. Ordinary programmatic `setTab()` calls do not move focus. The implementation stays inside the existing `handleShortcuts()` document listener, does not intercept controls in inputs/selects/textareas or dialogs, and leaves existing `/`, `Ctrl+N`, `Ctrl+S`, modal Tab trapping, and Escape behavior intact.

## Focus, targets, and contrast

The global `:focus-visible` style is a 3 px `#075985` outline with a 2 px offset, with an `@supports not selector(:focus-visible)` fallback. Disabled controls retain native disabled behavior and are not made focusable. Primary tabs, header actions, common action groups, modal actions, and disclosure summaries have bounded 40 px minimum heights; dense table/grid buttons remain scoped to 34 px so dense data UI is not broadly inflated.

Reviewed normal-text color pairs with WCAG relative-luminance contrast calculations:

| Foreground | Background | Contrast |
| --- | --- | --- |
| `#222831` | `#f5f6f8` | 13.72:1 |
| `#667085` | `#ffffff` | 4.97:1 |
| `#236d9c` | `#ffffff` | 5.62:1 |
| `#2f7d55` | `#eef8f1` | 4.62:1 |
| `#9a641d` | `#fff4db` | 4.56:1 |
| `#b14242` | `#fbeeee` | 4.98:1 |

## Fixtures and validation

Added `runAccessibilityPolishFixtures()` and `?selftest=accessibility`. The focused group verifies tablist/tab/tabpanel semantics, selection and roving tabindex, rendered visual state, focus-visible and target CSS contracts, Arrow/Home/End wrapping, Enter activation, programmatic-focus preservation, input/select/modal isolation, slash-search behavior, machine setup labels, Help and first-run controls, duplicate IDs, offline dependency absence, storage/backup stability, constants, and the existing first-run, machine, Help, and modal fixture groups.

Observed validation results:

- `python -m html.parser index.html`: passed.
- `git diff --check`: passed.
- Direct `file:///` Headless Edge fixture run: no exceptions.
- Accessibility polish: `36 / 0`.
- First-run onboarding: `19 / 0`; machine setup: `50 / 0`; Help and safety: `37 / 0`; modal accessibility: `26 / 0`.
- Designs geometry: `1093 / 0`; complete suite: `2077 / 0`.

The browser fixture uses synthetic DOM keyboard events and structural DOM/CSS assertions. It does not claim screen-reader testing or substitute for manual keyboard and assistive-technology validation across supported browsers.

## Preserved boundaries

No external dependency, storage key, schema version, release/version identity, persisted preference, import/export format, production SVG, production geometry, download behavior, machine identity semantics, or Design template was changed. The app remains directly openable from `file://` and offline.
