# Phase 7.2 Designs Production Settings Independent Audit

Date: 2026-07-16
Repository: `C:\Genmitsu L8 Tracker`
Committed baseline: `d54b05d` - `Add proven production settings`
Audit scope: uncommitted Phase 7.2 working-tree implementation

## Executive result

The core Phase 7.2 implementation is conservative and its safety boundaries held under source inspection, existing fixtures, normal direct-file UI exercise, and additional adversarial browser probes.

No Blocker or Major finding was found. In particular:

- selection is inert;
- explicit target application is template-bounded;
- machine mismatch cannot apply mechanical-fit values;
- a crafted checked mismatch target did not bypass the applicability result;
- Library, evidence, backup, and localStorage isolation held;
- kerf and cut references did not enter SVG geometry;
- stale records were not substituted;
- protected storage, normalization, geometry, and serializer functions are byte-identical to the committed baseline;
- all 1,250 current assertions passed.

The audit did find three Minor implementation/ordering concerns and one Minor fixture-quality concern. The fixture group especially overstates several behaviors with circular checks. These are not evidence of a current persisted-data or geometry defect, but correcting them before commit would make the phase substantially safer to maintain.

## Repository state

Before the audit:

- branch: `main`, tracking `origin/main`;
- HEAD: `d54b05d`;
- modified tracked files: `index.html`, `README.md`;
- staged files: none;
- implementation report and architecture report: untracked;
- unrelated untracked reports, LightBurn work, and `parametric_qr_stand_generator.py`: present and preserved;
- `git diff --check`: passed.

This audit did not edit application files, stage files, commit, push, reset, clean, stash, move, or delete anything. Its only repository write is this requested audit report.

## Findings

### Finding 1 - Minor - circular and overclaimed Phase 7.2 fixtures

- Severity: Minor
- Affected function: `runDesignProductionSettingsFixtures()`
- Exact scenario: 11 of the 93 assertions are circular, and several more exercise only pure helper output while their names claim UI, import, preview/download, or unit behavior.
- Expected behavior: fixture names should correspond to the code path actually exercised and critical policies should be tested through an independent path.
- Observed behavior:
  - `Selection can be represented without changing Design draft` constructs a session object but does not select through the chooser.
  - `Simulated reload recreates fresh production defaults` compares the default helper with itself.
  - `Replace and Merge import shapes cannot create production selection` does not call Replace or Merge.
  - `Reset defaults can clear draft and production session independently` does not invoke the Reset handler.
  - `Template switch model preserves IDs and clears checks` constructs an object rather than switching templates.
  - selection/checked SVG assertions generate the same unchanged draft without performing selection/checking.
  - kerf-only and cut-reference-only assertions do not change kerf or cut reference source data.
  - preview/download identity calls `buildDesignResult()` twice rather than comparing the preview and download paths.
  - the mm/in fixture compares the same expression with itself and never changes `state.unit`.
- Consequence: 93/0 is numerically true but overstates independent coverage. A regression in event binding, import/reset behavior, unit switching, or preview/download integration could remain green.
- Recommended correction: replace the 11 circular checks with focused event-level or helper-plus-independent-oracle checks. Rename any remaining helper-only assertion so it does not claim a UI path it does not run.
- Existing fixtures detect it: No. The issue is within the fixture design itself.

### Finding 2 - Minor - evidence reordering produces a false changed-record warning

- Severity: Minor
- Affected function: `designProductionFingerprint()`
- Exact scenario: the same evidence records in a different array order produce a different fingerprint because evidence is serialized in current array order.
- Expected behavior: the fingerprint should be deterministic for semantically identical evidence collections unless evidence order is itself intended to be review-relevant.
- Observed behavior: object property order is stable because the helper constructs explicit objects, but evidence array order is not canonicalized.
- Consequence: an import/merge or future UI operation that reorders unchanged evidence can clear checked targets and require an unnecessary review. It does not apply data, mutate Library, or create an unsafe fit.
- Recommended correction: sort the fingerprint-only evidence snapshots by stable evidence ID, with deterministic secondary fields for malformed duplicate/missing IDs. Do not reorder stored evidence.
- Existing fixtures detect it: No. There is no evidence-reordering fingerprint fixture.

### Finding 3 - Minor - grouped chooser can weaken the stated global rank order

- Severity: Minor
- Affected functions: `designProductionSettingChoices()`, `designProductionPanelHtml()`
- Exact scenario: choices are globally sorted, then regrouped by profile. If profile A has a top-ranked exact-machine record plus a lower-ranked mismatch record, and profile B has another exact-machine record, all profile A options render before profile B. The lower-ranked mismatch can therefore appear before profile B's exact match in final DOM order.
- Expected behavior: the visible chooser order should either preserve the documented global rank or explicitly define ranking as profile-group rank followed by child rank.
- Observed behavior: output is deterministic, but grouping changes the flattened priority order.
- Consequence: record discoverability can differ from the stated machine-first order. Record identity and application safety are unaffected.
- Recommended correction: define a group-order contract using each profile's best child rank, then sort children inside each group; document that exact records are ranked within profile groups. If strict global record rank is required, a grouped `<select>` cannot fully preserve it.
- Existing fixtures detect it: No. The reversed-profile fixture uses data that does not expose this mixed-rank group case.

### Finding 4 - Minor - nominal thickness sorting compares raw values across units

- Severity: Minor
- Affected function: `designProductionSettingChoices()`
- Exact scenario: sort position uses `Number(profile.thicknessValue)` without converting `thicknessUnit`. A `0.125 in` profile sorts numerically before `3 mm`, even though it is physically thicker.
- Expected behavior: nominal thickness rank should compare a common canonical unit or be documented as display-value ordering.
- Observed behavior: sorting is stable but not physical-thickness-aware across mixed Library units.
- Consequence: low-risk chooser ordering inconsistency only; no value mapping or geometry impact.
- Recommended correction: derive a pure canonical millimeter sort value with the existing unit conversion helper, without modifying stored profiles.
- Existing fixtures detect it: No. No mixed-unit chooser fixture exists.

## Verified behavior

### Selection inertia

- Severity: Verified
- Affected functions: chooser handler, `designProductionPanelHtml()`, `renderDesigns()`
- Scenario: select active records, toggle Show superseded, and select a superseded record.
- Expected: no Design, SVG, Library, evidence, storage, or backup mutation.
- Observed: selection only updated module-local session state and rerendered. `renderDesigns()` generated SVG from the unchanged `designDraft`. The browser workflow confirmed Design values and saved Library data were unchanged.
- Consequence: the central inert-selection rule is satisfied.
- Existing fixtures: partially; independent browser exercise supplied the stronger verification.

### Explicit checked-value application

- Severity: Verified
- Affected functions: `productionSettingDesignApplicability()`, `applyDesignProductionValues()`, Apply handler
- Scenario: zero, one, multiple, repeated, and independent Drawer target combinations.
- Expected: only checked enabled mapped fields change; sources remain immutable.
- Observed: zero targets returned without render; one and multiple targets copied deterministic millimeter strings; unchecked fields remained unchanged; repeated application was idempotent; profile, setting, evidence, and applicability inputs were unchanged.
- Consequence: no silent multi-field application was found.
- Existing fixtures: partial helper coverage plus browser coverage.

### Template mappings

- Severity: Verified
- Affected function: `productionSettingDesignApplicability()`
- Observed mapping:

| Template | Verified targets |
|---|---|
| Finger Box | `materialThickness`, `jointClearance` |
| Drawer Cabinet | `materialThickness`, independent `cabinetJointClearance`, independent `drawerJointClearance`, `drawerLateralClearance` |
| Joint Fit Coupon | `materialThickness`; winner is reference-only |
| Sliding Lid Box | `materialThickness` only |
| QR stand | `materialThickness` only |
| Dice tray | `materialThickness` only |
| Divider tray | `materialThickness` only |
| Hanging sign | no targets and no panel |

Joint Fit Coupon candidates, Sliding Lid fit fields, and tray `fitClearance` remained unchanged.

### Machine mismatch and bypass resistance

- Severity: Verified
- Affected functions: `designProductionMachineKey()`, `productionSettingDesignApplicability()`, Apply handler
- Scenarios:
  - exact 20W and exact 40W;
  - 20W record under 40W app profile;
  - 40W record under 20W app profile;
  - custom/current unknown and missing/unknown record keys by source analysis;
  - a browser-side crafted attempt that removed the disabled attribute and checked the blocked fit target.
- Expected: thickness remains available with warning; fit remains blocked unless both machine keys are the same known profile.
- Observed: exact records allowed fit; both known mismatch directions blocked fit; thickness remained available; the crafted checked target produced no joint change; custom/missing/unknown paths have `exactMachine === false`; no scaling path exists.
- Consequence: the high-risk mismatch bypass condition was not found.
- Existing fixtures: exact 20W, one mismatch direction, custom, and missing are partial; the independent browser probe added exact 40W, reverse mismatch, and crafted-check coverage.

### Status and operation blocking

- Severity: Verified
- Affected function: `productionSettingDesignApplicability()`
- Observed: verified, tested, estimated, and untested can apply; verified without date warns; superseded, unknown status, and non-Cut operation produce disabled targets. Display helpers do not change status.
- Existing fixtures: direct helper checks; source inspection confirmed the Apply helper honors `target.enabled`.

### Thickness policy

- Severity: Verified
- Affected functions: `productionSettingDesignApplicability()`, `designProductionTarget()`
- Observed:
  - source is only `materialCondition.measuredThicknessMm`;
  - parent and production nominal values are display/fingerprint context only;
  - Inventory names are not parsed;
  - missing measured thickness disables only that target;
  - zero is retained as a numeric proposal;
  - any nonblank numeric difference is rounded through the existing deterministic Design formatter;
  - blank current values do not invent a delta;
  - no tolerance exists;
  - global unit is not consulted by apply helpers.
- Existing fixtures: partial; browser unit switching independently confirmed the applied millimeter string remained stable.

### Drawer target independence

- Severity: Verified
- Affected functions: mapping table and `applyDesignProductionValues()`
- Observed: cabinet-only, drawer-only, both, neither, lateral-only, and combined checks operate through separate keys and draft fields. One source finger value is never implicitly copied to both targets.
- Existing fixtures: good helper coverage; browser independently exercised cabinet-only while drawer and lateral stayed at their prior values.

### Reference-only isolation

- Severity: Verified
- Affected functions: fingerprint/reference renderer and unchanged geometry pipeline
- Observed: speed, power, passes, air, pressure, focus, software, kerf, and evidence are not mappings. They appear only in reference/fingerprint rendering. Filenames still use only template/date. SVG metadata, titles, IDs, score paths, and cut paths receive no production object.
- Existing fixtures: kerf/cut fixture names are circular; source-path inspection and unchanged geometry functions provide the actual verification.

### Stale and changed selection behavior

- Severity: Verified
- Affected functions: `resolveDesignProductionSelection()`, `syncDesignProductionSession()`, Review/Apply handlers
- Observed:
  - missing parent, missing setting, wrong parent, and invalid IDs never substitute;
  - superseded records remain selected when Show superseded is turned off;
  - superseded Apply remains disabled and `supersededById` is not followed;
  - editing values without relying on `updatedDate` changes the fingerprint;
  - evidence and notes participate in the fingerprint;
  - checked targets clear on stale/change;
  - Review updates fingerprint/check/session notice only;
  - already applied ordinary Design values remain independent.
- Existing fixtures: resolver/fingerprint helpers cover part; browser independently exercised changed, deleted, and selected-superseded cases.

### Session, reset, events, and keyboard safety

- Severity: Verified
- Affected functions: module initialization, template handler, Reset handler, `bindDesignProductionActions()`
- Observed:
  - fresh load creates an empty session;
  - Reset replaces both session drafts and clears the notice;
  - template changes preserve IDs/Show superseded and clear checks;
  - controls are outside `designForm`;
  - buttons have `type="button"`;
  - `designForm` retains `onsubmit="return false"`;
  - rerender replaces DOM nodes before rebinding, so handlers do not accumulate;
  - target keys and option data use stable identifiers rather than indexes;
  - zero-target Apply returns before render/notice;
  - pressing Enter in the production panel has no enclosing form to submit.
- Existing fixtures: several claims are circular; source and browser inspection provide the verification.

### Data/storage isolation

- Severity: Verified
- Affected functions: all Phase 7.2 helpers and handlers
- Observed: no new helper calls `persist()` or `localStorage.setItem()`. `backupObject()` has no selection fields. Browser selection/apply/edit/template/reset workflows left saved Library production data unchanged, excluding intentional existing unit-preference/profile-unit conversion behavior; production evidence remained unchanged across unit switches.
- Existing fixtures: partial, strengthened by browser snapshots and protected-function comparison.

### SVG and geometry isolation

- Severity: Verified
- Affected functions: existing Design pipeline
- Observed:
  - selection and checking do not enter `designDraft`;
  - application writes only ordinary mapped draft strings;
  - preview and download continue to use `buildDesignResult(designDraft).svg`;
  - 559 geometry assertions and byte signatures pass;
  - no serializer, template model, finger helper, score/cut layer, or SVG metadata function changed.
- Existing fixtures: geometry suite is strong; several new Phase 7.2 SVG assertions are circular and should be replaced.

### Front assembly preview boundary

- Severity: Verified
- Affected area: Drawer Cabinet rendering and SVG generation
- Observed: Phase 7.2 adds no Finished Front View, assembly projection, or screen-only cabinet preview. Drawer Cabinet model and exported SVG functions are unchanged.
- Future boundary: a later screen-only Front Assembly Preview can consume the existing `buildDrawerCabinetModel()` result in `designResultsHtml()` or a sibling display helper without changing panel layout or `serializeDesignSvg()`.

## Chooser determinism review

The underlying choice array is deterministic for reversed input because every record receives explicit rank fields and stable profile/setting ID tie-breakers. Identity resolution requires both IDs and refuses same-ID records under another parent.

Important gaps:

- no shuffled setting-array fixture;
- no duplicate setting-ID-under-different-parent chooser interaction fixture;
- no mixed exact/mismatch children across profile groups;
- no mixed-unit nominal thickness fixture.

No wrong-source selection was observed, but the two ordering findings above should be cleaned up or explicitly documented.

## Fingerprint review

The fingerprint includes all minimum required fields and more:

- parent and setting IDs;
- label and operation;
- machine key/label/lens/focus method;
- status, preferred, confidence, dates, supersede link, notes;
- nominal/measured thickness, scope, measured date, Inventory/material snapshots, batch, finish;
- finger and Drawer lateral fit values;
- kerf;
- all displayed cut references;
- evidence IDs, kinds, sources, dates, results, and notes;
- `updatedDate`.

Object key order is explicit and stable. The one identified stability concern is evidence array order.

## Fixture quality classification

Numbering below follows source order in `runDesignProductionSettingsFixtures()`.

### Independent - 36

IDs: `1, 3, 4, 9-15, 17-24, 30-31, 55-56, 58-61, 63, 70, 76-79, 90-93`.

These use an external invariant or compare source/output state meaningfully: session/state/backup absence, machine mapping, chooser inclusion/order, resolver identity/staleness, fingerprint changes, signed values, warning text, source immutability, object identity, and schema constants.

### Partially circular - 46

IDs: `16, 25-29, 32-54, 57, 62, 64-69, 71-75, 80, 85-86, 89`.

These generally assert the correct policy, but build applicability and then feed the same applicability into the apply helper, or test only a helper while claiming a broader integration result. They are useful unit checks but are not independent proof of UI binding, storage, or SVG pathway behavior.

### Circular - 11

IDs and names:

1. `2` - Selection can be represented without changing Design draft.
2. `5` - Simulated reload recreates fresh production defaults.
3. `6` - Replace and Merge import shapes cannot create production selection.
4. `7` - Reset defaults can clear draft and production session independently.
5. `8` - Template switch model preserves IDs and clears checks.
6. `81` - Selection alone preserves exact SVG bytes.
7. `82` - Checked but unapplied values preserve exact SVG bytes.
8. `83` - Kerf-only source changes preserve exact SVG bytes.
9. `84` - Reference-only cut changes preserve exact SVG bytes.
10. `87` - Preview and download source remain the same generated SVG.
11. `88` - Global mm and in preferences produce same applied draft.

The 93 classifications sum to 36 independent, 46 partially circular, and 11 circular.

## Missing high-value fixtures

Recommended additions or replacements:

- crafted `checkedTargets` against a real mismatch applicability result;
- evidence reordering and missing/duplicate evidence IDs in fingerprinting;
- duplicate setting IDs under different parent profiles in chooser interaction;
- Enter key behavior with focus on select, checkbox, Review, and Apply;
- edit without changing `updatedDate`, including evidence-only and notes-only changes;
- actual preview data URL versus intercepted download text after apply;
- wrong-parent stale record through rendered UI;
- selected superseded record while Show superseded is turned off;
- Reverse 20W/40W mismatch and exact 40W;
- mixed-unit and mixed-rank grouped chooser order;
- actual Replace and Merge import workflows with pre-existing and empty sessions;
- localStorage/backup snapshots before and after each individual UI action.

## Fixture and runtime totals

Independently rerun in a fresh direct-file Microsoft Edge profile:

| Group | Passed | Failed |
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

Normal direct `file://` startup displayed the header, tabs, and main content.

The independent adversarial Edge probe additionally verified:

- exact 40W fit application;
- both known mismatch directions;
- mismatch thickness availability;
- blocked mismatch fit target;
- a crafted DOM attempt to check/apply the blocked fit target;
- thickness-only apply under mismatch;
- superseded record visibility after Show superseded is turned off;
- superseded Apply disabled;
- Library production records unchanged.

## Protected-boundary comparison

Function-level byte comparison against `HEAD:index.html` confirmed unchanged:

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
- `buildFingerPattern()`
- `designPatternEdge()`
- `buildFingerPanel()`
- `buildSlidingLidBodyPanel()`
- `buildGridCellMaterialTestRecord()`
- `projectWizardCandidatesForInventory()`

Diff inspection found no change to Library save mechanics, Material Test promotion, Project Wizard classification, Drawer Cabinet geometry, shared finger geometry, SVG serialization, storage/schema/import paths, or LightBurn layer behavior.

## Validation summary

- `git diff --check`: passed.
- HTML parser: passed.
- Direct `file://` Edge startup: passed.
- Design-production fixtures: 93/0.
- Designs geometry: 559/0.
- Production settings: 66/0.
- Complete suite: 1250/0.
- Normal UI workflow: passed.
- Additional mismatch/superseded adversarial workflow: passed.
- Protected boundary byte comparison: passed.
- Application files modified by audit: none.
- Staging/commit/push: none.

No physical laser test, human-visible browser session, or LightBurn import was performed; those are not required to establish this phase's data-flow safety but remain appropriate before relying on copied production values physically.

SAFE TO COMMIT AFTER MINOR CLEANUP
