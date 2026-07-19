# In-app About, Help, and Safety Implementation — 2026-07-19

## Baseline and repository state

- Actual HEAD: `2f8639d60f6338d460fe4536f824733af6e5d79f` — `Improve modal keyboard accessibility`.
- Branch: `main`; `origin/main...main` was `0 / 0` behind/ahead.
- The tracked tree and staging area were clean before this phase. Pre-existing untracked `.claude/`, `LightBurn Projects/`, `debug.log`, historical reports, and `parametric_qr_stand_generator.py` were preserved exactly.
- Baseline fixture total: `1953 / 0`.

## Files changed

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/ABOUT_HELP_SAFETY_IMPLEMENTATION_2026-07-19.md`

## Existing surfaces inspected

The static header already provided units, Import, Export, and the compact release footer About disclosure. The Reference tab already included official starting-point tables, the bundled manual/chart links, a material-safety reminder, and a starting-points warning. Designs already warns that production SVGs must remain at 100% scale in LightBurn and that physical validation is required. Storage recovery, Import validation, Export photo-size warning, localStorage handling, and the accessible shared-modal helpers were already present.

Before this change there were eleven modal containers. Afterward there are twelve: `entryModal`, `profileModal`, `projectModal`, `inventoryModal`, `gridModal`, `cellModal`, `testModal`, `productionSettingModal`, `productionEvidenceModal`, `materialTestWizardModal`, `projectWizardModal`, and `helpModal`.

## Help entry points and content

Added a real header `Help` button and a compact normal-flow Reference callout with `Open Help & Safety`. Both invoke one `openHelp()` helper, which creates `helpModal` through the existing `openModal()` and `bindModal()` path.

The modal has six visible sections:

1. About this Tracker — renders `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `SCHEMA_VERSION`, and `BACKUP_FORMAT` from existing constants; explains direct `file://` use, local-browser records, and absence of accounts/cloud synchronization.
2. Your data and backups — says Export creates a JSON backup and is not automatic; describes Import merge/Replace, newer-schema refusal, browser-data loss, photo storage, and existing recovery tools.
3. Using Designs with LightBurn — gives the Generate/Download/Import/100%-scale/unit-and-dimension/layer/framing workflow, says Tracker does not send laser jobs, and explains physical-result variables.
4. Laser safety — requires active ventilation/fume extraction, fire watch, no unattended running, suitable nearby suppression, focus, secure material, framing, clean optics/work area, and controlled air assist. It visibly prohibits PVC, vinyl, chlorinated plastics, mystery plastics, unidentified materials, and unknown coatings; it calls out fiberglass, carbon fiber, reflective surfaces, coated metals, foams, leather, adhesives, and composites.
5. Starting points and physical validation — describes Reference/manufacturer values as starting points rather than guarantees; directs users to material testing and actual-material validation of cut, fit, strength, and finish.
6. Documentation — links only to verified bundled user documentation.

The Reference callout remains compact and does not change the selector or source tables. The release footer remains compact and uses the same identity constants.

## Offline documentation links

- `README.md` — verified at the repository root; linked as `README.md` with `target="_blank" rel="noopener"`.
- `docs/Genmitsu_L8_Laser_Engraving_Machine_User_Manual.pdf` — verified bundled file; linked with the same offline-compatible attributes.

No temporary report, development audit, internet URL, or invented documentation path was added. The 20W chart and existing manual links in Reference were preserved.

## Accessibility and persistence

`helpModal` receives the shared panel semantics: `role="dialog"`, `aria-modal="true"`, `tabindex="-1"`, a visible-heading-derived stable `aria-labelledby`, and accurate `aria-hidden` state. Initial focus enters Help, Tab/Shift+Tab are contained by the existing document listener, Escape and visible Close controls use `closeModal()`, and focus returns to the originating header or Reference button.

Help content, modal origin tracking, and modal order are page-session-only. No Help preference, acknowledgement, storage key, schema field, migration, backup field, or import/export behavior was added.

## Fixtures and totals

- Added `runHelpSafetyFixtures()` and `?selftest=help`: `37 / 0` assertions.
- Help fixtures use the real header and Reference buttons, real Help DOM, synthesized browser keyboard events for Tab/Escape, real focus checks, duplicate-ID checks, relative-link checks, identity consistency, no-persistence checks, and content checks.
- Modal accessibility fixture inventory automatically includes `helpModal` exactly once; its total increased from `25 / 0` to `26 / 0`.
- Complete suite: `1991 / 0` = `898` non-Design assertions + `1093` Designs assertions. The baseline `1953` increased by `37` Help assertions and one modal-inventory assertion.
- Designs remains `1093 / 0`. Tray-model remains `264 / 0` as a subset of Designs and is not double-counted.

## Validation performed

- `git diff --check` — clean.
- `python -m html.parser index.html` — passed.
- Direct headless Edge `file:///.../index.html` — application, tabs, footer, and direct-file startup rendered without page exceptions.
- Direct headless Edge `file:///.../index.html?selftest=all` — complete suite `1991 / 0`; browser console and exception monitoring were clean.
- Focused Help fixture — `37 / 0`; modal accessibility fixture — `26 / 0`; first-run — `19 / 0`; storage recovery — `15 / 0`; production settings — `66 / 0`; Designs production settings — `118 / 0`; Designs — `1093 / 0`.
- Browser automation used actual clicks for header Help and Reference Help, synthesized Tab and Escape events, and verified focus restoration to both triggers. No screen reader was used.
- At 320 px, Help had six readable sections, a 274 px dialog width, and no horizontal page overflow. At 1024 px, the dialog was 720 px wide with no horizontal overflow; the Reference callout remained normal flow.
- Verified the README and bundled-manual targets before linking.

## Protected boundaries

No changes were made to `STORAGE_KEY`, `SCHEMA_VERSION`, `APP_VERSION`, `BUILD_DATE`, `freshState()`, `loadState()`, normalization, `backupObject()`, import validation, merge/replace, `persist()`, corruption recovery, first-run eligibility/storage, release footer identity values, or modal form values/defaults/validation.

No Designs geometry, serializer, download, Finished View, dimensions, kerf, clearance, panel layout, filename, or production bytes changed. Machine-profile semantics, Library promotion/evidence, Project accounting, Inventory, Pricing, dependencies, and network behavior are unchanged.

## Remaining verification

This pass did not use a physical screen reader or real laser/material test. General contrast, touch-target, and tab-role work remains separate. Help intentionally does not replace template-specific experimental or physical-validation warnings.

## Final state

Only the four files listed above were changed by this phase. Nothing was staged, committed, or pushed.
