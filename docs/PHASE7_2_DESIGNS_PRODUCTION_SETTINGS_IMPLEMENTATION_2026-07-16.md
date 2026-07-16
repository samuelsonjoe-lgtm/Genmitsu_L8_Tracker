# Phase 7.2 Designs Production Settings Implementation

Date: 2026-07-16
Repository: `C:\Genmitsu L8 Tracker`
Baseline: `d54b05d` - `Add proven production settings`
Status: implementation complete; independent audit recommended

## Repository state before editing

- Branch: `main`, tracking `origin/main`.
- HEAD: `d54b05d` - `Add proven production settings`.
- Tracked working tree: clean.
- Staged files: none.
- `git diff --check`: passed.
- Pre-existing untracked LightBurn work, reports, the Phase 7.2 architecture review, and `parametric_qr_stand_generator.py` were preserved without modification.

## Implemented behavior

Designs now has an optional collapsed **Production setting** panel for every supported thickness-consuming template except Hanging sign. The panel:

- resolves saved Cut production records from Library profiles;
- groups choices by the owning Library profile;
- stores only profile and setting IDs in module-local session state;
- changes no Design field when a record is selected;
- presents current, proposed, signed change, and resulting values;
- copies only enabled targets whose checkboxes are explicitly selected;
- leaves copied values as ordinary editable millimeter strings;
- displays cut settings, signed LightBurn kerf, and evidence as read-only reference data;
- never writes to Library records or evidence;
- never persists the selection or application metadata;
- never places production metadata or kerf compensation in SVG output.

The apply notice deliberately describes copied values as editable and not a fit guarantee.

## Session-state implementation

Added module-local state separate from both `state` and `designDraft`:

```javascript
{
  profileId: "",
  settingId: "",
  checkedTargets: {},
  showSuperseded: false,
  reviewFingerprint: "",
  staleReason: ""
}
```

Behavior:

- page load starts with fresh defaults;
- Reset Designs clears `designDraft`, the production session, and its notice;
- switching templates preserves source IDs and Show superseded, but clears checked targets;
- changing templates applies no production value;
- no full profile or setting object is retained in session state;
- import/backup/state schemas have no production-selection field.

## Helpers added

- `designProductionSessionDefaults()`
- `designProductionMachineKey()`
- `designProductionFingerprint()`
- `designProductionSettingChoices()`
- `resolveDesignProductionSelection()`
- `designProductionScopeWarning()`
- `designProductionVerificationText()`
- `designProductionValuePresent()`
- `designProductionTarget()`
- `productionSettingDesignApplicability()`
- `applyDesignProductionValues()`
- `syncDesignProductionSession()`
- `designProductionChoiceLabel()`
- `designProductionStaleMessage()`
- `designProductionReferenceHtml()`
- `designProductionPanelHtml()`
- `bindDesignProductionActions()`
- `runDesignProductionSettingsFixtures()`

Lookup, fingerprint, applicability, and apply helpers return new data and do not mutate their profile, setting, evidence, draft, or state inputs. UI synchronization is isolated in the session/render helpers.

## Existing functions modified

- `renderDesigns()` inserts the Production setting panel outside `designForm`.
- The central render binding adds separate Production setting controls, clears checks on template changes, and resets both session drafts.
- `productionSettingsEditorHtml()` replaces the obsolete statement that Designs cannot consume records with accurate explicit-application wording.
- Startup self-test routing recognizes `?selftest=design-production` and includes the group in `?selftest=all`.

`designDefaults()`, `updateDesignDraft()`, and all protected normalization, geometry, serialization, storage, and import functions remain unchanged.

## Chooser and deterministic ordering

The chooser includes Cut records and excludes superseded records by default. A session-only toggle reveals superseded history. The selected superseded record remains visible even when the toggle is subsequently cleared so selection cannot silently move.

Ordering is deterministic by:

1. exact current machine;
2. active before superseded;
3. preferred before non-preferred;
4. verified, tested, estimated, untested, superseded, then unknown;
5. material;
6. nominal thickness;
7. setting label;
8. profile ID;
9. setting ID.

Reversing the input Library profile array produces the same choice order. Identity is always resolved with the stable parent profile ID and nested setting ID, never a display label or array index.

## Machine behavior

Current preferences map as follows:

- `20W` -> `genmitsu-l8-20w`
- `40W` -> `genmitsu-l8-40w`

An exact known-machine match allows measured thickness and mapped fit values. Known mismatch, custom, missing, and unknown machine conditions:

- allow measured thickness with a warning;
- block every mechanical-fit target;
- display cut, kerf, and evidence references;
- perform no scaling or conversion.

The machine warning identifies the record machine and current app machine and states that mechanical fit cannot transfer across profiles.

## Template mappings

| Template | Explicitly applicable values |
|---|---|
| Finger Box | measured thickness -> `materialThickness`; finger joint -> `jointClearance` |
| Drawer Cabinet | measured thickness -> `materialThickness`; finger joint -> independently checked `cabinetJointClearance` and `drawerJointClearance`; drawer lateral -> `drawerLateralClearance` |
| Joint Fit Coupon | measured thickness only; stored finger joint is reference-only |
| Sliding Lid Box | measured thickness only |
| QR stand | measured thickness only |
| Dice tray | measured thickness only |
| Divider tray | measured thickness only |
| Hanging sign | no panel and no targets |

No similarly named field is inferred. In particular:

- Joint Fit Coupon candidates and winner status are untouched;
- Sliding Lid joint/lid/insertion clearances are untouched;
- tray `fitClearance` is untouched;
- Drawer Cabinet's two joint targets remain independent despite sharing one source value.

## Thickness, scope, and status behavior

Only `materialCondition.measuredThicknessMm` is eligible. Parent nominal thickness, production nominal thickness, and Inventory-name parsing are never used as fallback values.

Any numeric difference is shown exactly; no universal tolerance is invented. Blank current values are shown as blank without inventing a delta. Zero and negative signed values remain valid inputs to the existing Designs validator.

Sheet, batch, material-general, and unknown measurement scopes display separate warnings without blocking an otherwise valid explicit apply. Verified, tested, estimated, and untested records can apply. Superseded and unknown statuses cannot apply. A verified record without a verified date displays a warning.

Designs also states that it cannot independently verify material identity. Retained Inventory/material-condition snapshots remain visible, and a dangling Inventory link displays a warning without modifying Inventory.

## Reference-only production data

The panel displays recorded:

- speed;
- minimum/maximum power;
- passes;
- air assist and pressure;
- focus and unit;
- software;
- signed LightBurn kerf offset;
- evidence count, kind, date, source label, result, and notes.

None has an Apply checkbox. Kerf guidance explicitly says it must be applied to the red through-cut layer in LightBurn only after direction verification and is not part of generated SVG geometry.

## Review fingerprint and stale selections

The deterministic fingerprint covers IDs, label, operation, machine context, verification/preferred state, verification metadata, measured and nominal condition data, Inventory snapshots, fit values, cut/reference values, kerf, update date, notes, and evidence snapshots.

When current Library data differs from the reviewed fingerprint:

- selected IDs remain;
- current data is displayed;
- checked targets clear;
- Apply is disabled;
- the user must choose **Review current values** before checking/applying again.

Deleted profiles, deleted settings, wrong-parent IDs, incomplete IDs, and superseded records retain a visible stale selection with Apply disabled. No replacement or `supersededById` record is followed automatically. Already applied ordinary Design values remain unchanged.

## Unit behavior

Production dimensions remain canonical millimeters. Applied values use the existing deterministic `designNumberText()` formatting, preserving zero and signed negative values without negative zero.

The global `mm`/`in` preference is not consulted during application. A direct browser workflow confirmed that switching global units leaves the applied millimeter Design value and normalized production evidence unchanged.

## Fixture coverage

Added `runDesignProductionSettingsFixtures()` and the `design-production` self-test query. The group contains 93 assertions covering:

- default/reload/reset/template-switch session behavior;
- backup/state/localStorage exclusion;
- deterministic chooser identity and ordering;
- exact, mismatched, custom, and missing machine handling;
- every approved template mapping and excluded mapping;
- independent Drawer Cabinet targets;
- zero, negative, blank, and missing values;
- scope and verification warnings;
- stale IDs and fingerprint changes;
- explicit one/multiple/zero-target application;
- source/draft nonmutation and idempotence;
- SVG isolation and signed reference precision;
- storage key and schema stability.

## Verified totals

Fresh direct-file Microsoft Edge runs produced:

| Fixture group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Production settings | 66 | 0 |
| Designs production application | 93 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs geometry | 559 | 0 |
| **Complete suite** | **1250** | **0** |

Both `file://...?selftest=design-production` and `file://...?selftest=all` were opened in the Edge harness. Normal direct `file://` startup rendered the header, tabs, and main application content.

## Browser workflow validation

A fresh isolated Edge profile created a Library profile and verified Cut production setting through the normal UI, then confirmed:

- selection changed no Design value;
- measured thickness applied only after its checkbox and Apply button;
- template changes preserved selection and cleared checks;
- cabinet joint, drawer joint, and lateral targets remained independent;
- Joint Fit Coupon candidates remained unchanged;
- Sliding Lid fit fields remained unchanged;
- Hanging sign had no Production setting panel;
- cut and kerf references had no Apply controls;
- global unit switching preserved the applied millimeter value;
- Library data stayed byte-identical through selection and application;
- production data stayed byte-identical through unit switching;
- an edited setting required explicit review;
- a deleted setting remained visibly stale without substitution;
- reset cleared both drafts;
- persisted JSON contained no Designs production selection.

## SVG, Library, and storage isolation

- Existing Designs geometry fixtures remained 559/0, including established byte signatures and preview/download identity checks.
- Selection and checked-but-unapplied targets preserve SVG bytes.
- Reference-only cut and kerf changes do not enter the draft or SVG.
- Applying a mapped value affects output only through the same ordinary Design field path as manual entry.
- Profile, setting, evidence, and applicability inputs remained unmutated in fixtures.
- Browser UI selection/application left the saved Library profile byte-identical.
- No Phase 7.2 helper calls `persist()` or writes localStorage.
- `STORAGE_KEY` remains `genmitsu-l8-tracker-v1`.
- `SCHEMA_VERSION` remains `2`.

## Protected-boundary verification

A function-level byte comparison against `HEAD:index.html` confirmed these functions are unchanged:

- `persist()`
- `backupObject()`
- `replaceData()`
- `mergeData()`
- `normalizeProductionSetting()`
- `normalizeProductionSettings()`
- `normalizeDesignDraft()`
- `buildDesignResult()`
- `serializeDesignSvg()`
- `refreshDesignPreview()`
- `designSvg()`

No shared finger geometry, template model, SVG layer, filename, SVG metadata, migration, schema, or import behavior changed.

## Validation results

- `git diff --check`: passed.
- Python HTML parser: passed.
- JavaScript runtime/startup: passed in Microsoft Edge; Node.js is not installed locally.
- Direct offline `file://` startup: passed.
- `?selftest=design-production`: 93/0.
- `?selftest=all`: 1250/0.
- Designs geometry: 559/0.
- Production settings: 66/0.
- Storage recovery: 8/0.
- Library and evidence nonmutation: passed in fixtures and UI harness.
- localStorage/backup selection exclusion: passed.
- global mm/in equivalence: passed.
- changed/deleted stale selection: passed in UI harness.
- final intended file set is limited to `index.html`, `README.md`, and this report.

## Not performed

- No physical laser/material fit test was performed.
- No human-operated visible Edge session was performed; browser validation used fresh headless Edge profiles over the direct `file://` page.
- No actual downloaded SVG file was opened in LightBurn during this pass. The unchanged preview/download path and existing byte-identity fixtures were verified instead.
- Supersede lifecycle behavior was covered by existing production fixtures and new applicability fixtures, but the final manual UI harness exercised edited and deleted records rather than clicking the supersede workflow again.

## Repository actions

- No unrelated untracked file was edited, moved, deleted, or staged.
- Nothing was staged.
- No commit was created.
- Nothing was pushed.

**READY FOR INDEPENDENT AUDIT**
