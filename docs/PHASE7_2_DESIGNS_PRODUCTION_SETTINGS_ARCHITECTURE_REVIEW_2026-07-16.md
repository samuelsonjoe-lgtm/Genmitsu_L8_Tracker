# Phase 7.2 Designs Production Settings Architecture Review

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Reviewed baseline: `d54b05d` — `Add proven production settings`  
Review type: read-only architecture and implementation planning  
Proposed phase: Phase 7.2 — Explicit Production Settings Application in Designs

## 1. Executive recommendation

Phase 7.2 is ready to implement with one central rule:

> A saved production setting is a reviewable source record, not a live Design preset. It may copy only explicitly checked compatible values into ordinary session-only Designs fields.

The safest implementation is a compact, collapsed Production setting section near the top of Designs. Use one grouped production-setting chooser rather than a two-step profile/setting workflow. Each option should resolve to both a parent Library profile ID and a production-setting ID, while the session object stores those IDs separately.

Selection must only display:

- parent material and nominal Library thickness;
- production-setting label;
- machine and operation;
- preferred and verification status;
- measured thickness and scope;
- sheet/batch label;
- verified date;
- evidence summary;
- cut and LightBurn reference settings;
- compatible application targets and warnings.

Selection alone must not change `designDraft`. Applying values must require checked target rows and an explicit **Apply checked values** button.

Recommended template mappings:

| Template | Apply measured thickness | Apply finger-joint value | Apply Drawer lateral value |
|---|---:|---:|---:|
| Finger Box | Yes | Yes, to `jointClearance` | No |
| Drawer Cabinet | Yes | Yes, as two independent targets: `cabinetJointClearance` and `drawerJointClearance` | Yes, to `drawerLateralClearance` |
| Joint Fit Coupon | Yes | Reference only in Phase 7.2 | No |
| Sliding Lid Box | Yes | Defer | No |
| QR stand | Yes | No | No |
| Dice tray | Yes | No | No |
| Divider tray | Yes | No | No |
| Hanging sign | No production application panel | No | No |

For Drawer Cabinet, the single stored `fingerJointClearanceMm` should be offered twice as two independently checked targets:

```text
[ ] Cabinet joint clearance: -0.02 mm
[ ] Drawer joint clearance: -0.02 mm
```

Checking both is an explicit request to apply the same stored value to both fields. Do not add an implicit combined mapping and do not silently apply one value to both.

Known 20W/40W machine mismatch should block fit-value application. It should not block measured-thickness application because the physical measurement itself is not wattage-dependent, but the UI must still show that the setting's production result belongs to another machine. Custom or missing machine identity should also block fit-value application in this phase; no cross-machine override or scaling is recommended.

All Designs inputs are currently expressed and stored in the session draft as millimeter strings regardless of the app's global `in`/`mm` preference. Phase 7.2 should preserve that architecture. Production evidence is already canonical millimeters, so application should copy deterministic millimeter text directly. Do not add a second Designs unit system.

No changes are required to:

- `STORAGE_KEY`;
- `SCHEMA_VERSION`;
- `backupObject()`;
- `persist()`;
- import merge/replace;
- production-setting normalization;
- `normalizeDesignDraft()`;
- `buildDesignResult()`;
- shared finger geometry;
- `serializeDesignSvg()`;
- preview/download SVG generation.

The integration should end before geometry normalization: explicit application mutates only ordinary `designDraft` fields, after which the existing pipeline behaves exactly as if the user typed those values manually.

Current baseline verification:

- tracked working tree clean;
- `git diff --check` passed;
- normal direct `file://` startup passed with no page errors;
- complete fixture suite passed: **1157 / 0**;
- Designs geometry passed: **559 / 0**;
- production settings passed: **66 / 0**.

## 2. Existing Designs session architecture

### Session-only draft

Designs currently uses one module-local variable:

```javascript
let designDraft = designDefaults();
```

`designDraft` is not part of `state`, `persist()`, or `backupObject()`. Reloading the page recreates it from `designDefaults()`. The Reset defaults button does the same:

```javascript
designDraft = designDefaults();
render();
```

This is the correct session boundary for applied numeric Design values.

### Form update path

`renderDesigns()` builds the complete form from `designDraft`. Event binding uses:

```javascript
updateDesignDraft(designForm);
```

That helper copies every named form element into `designDraft`. Text/number/select values remain strings; checkboxes become booleans.

Input changes refresh only the Design result where possible. Template and structural-option changes perform a full render. No Designs input handler calls `persist()`.

### Geometry pipeline

The path is:

```text
designDraft
  -> normalizeDesignDraft()
  -> buildDesignResult()
  -> template model/layout
  -> serializeDesignSvg() or legacy SVG helper
  -> designResultsHtml() preview
  -> downloadTextFile() download
```

Preview and download both call `buildDesignResult(designDraft)`. The preview embeds the exact `result.svg` in a data URL; download writes the same `result.svg` text.

This architecture already supports the desired Phase 7.2 boundary. Once checked production values are copied into `designDraft`, no downstream code needs to know their source.

### Current Designs units

All Designs form labels and normalizers use millimeters:

- `materialThickness`;
- finger-joint clearances;
- Drawer Cabinet clearances;
- dimensions;
- Joint Fit Coupon values;
- LightBurn kerf guidance.

The global app unit toggle converts persisted Log, Library, Test Grid, and Project value/unit pairs. It does not convert `designDraft`.

Therefore:

- the Designs page remains millimeter-based even when `state.unit === "in"`;
- production values are already stored canonically in millimeters;
- applying a production value should not call `convertUnitValue()`;
- changing the global unit must not rewrite production evidence or applied Design values.

Introducing an inch representation inside Phase 7.2 would create parallel conversion logic and precision risk without matching the current Designs UI.

### Reset and template switching

Reset should clear:

- the ordinary Design draft back to defaults;
- selected profile ID;
- selected production-setting ID;
- checked target fields;
- show-superseded preference;
- stale/review warnings.

Template switching should:

- preserve the selected source record for convenience;
- clear all checked target fields;
- recalculate applicability for the new template;
- change no ordinary Design values beyond the current existing template-switch behavior.

Preserving the selected source while clearing checks makes cross-template work efficient without carrying an accidental apply intention from Finger Box into Drawer Cabinet.

## 3. Production-setting lookup architecture

### Existing persisted ownership

Production settings are nested under Library profiles:

```javascript
profile.productionSettings = []
```

Each record has:

- immutable nested ID;
- operation;
- machine key/label;
- material condition;
- cut settings;
- LightBurn kerf reference;
- fit settings;
- verification/preferred status;
- evidence;
- supersede history.

The parent Library profile ID remains the material-profile owner.

### Recommended lookup record

Add a pure flattening helper:

```javascript
designProductionSettingChoices(profiles, options)
```

It should return lightweight resolved choices:

```javascript
{
  profileId,
  settingId,
  profile,
  setting,
  sortKey,
  disabledReason
}
```

The helper must not mutate or duplicate records into session state.

Default inclusion rules:

- include only valid object records with stable parent and nested IDs;
- include cut operation records;
- exclude superseded records by default;
- include verified, tested, estimated, and untested records;
- allow a session-only **Show superseded** toggle;
- never treat preferred as proof of compatibility.

Recommended deterministic sort:

1. exact current machine before mismatch/custom/missing;
2. non-superseded before superseded;
3. preferred before non-preferred;
4. verification rank: verified, tested, estimated, untested, unknown;
5. parent material;
6. parent nominal thickness in millimeters;
7. production-setting label;
8. profile ID;
9. setting ID.

Stable IDs must be the final tie-breakers so order does not depend on `state.profiles` input order.

### Do not copy full records into session state

Store only IDs and a small review token. Resolve current Library data on each render:

```javascript
resolveDesignProductionSelection(selection, state.profiles)
```

This ensures:

- Library edits are visible;
- deleted records become stale safely;
- selected objects are never mutated through the Designs UI;
- import/replace cannot leave an obsolete copied object silently active;
- no full persisted object leaks into geometry models.

## 4. Recommended selection model

### Use one grouped chooser

Recommended UI:

```text
Production setting
[ Select saved setting...                         ]
```

The `<select>` should group choices by parent Library profile:

```text
Basswood plywood — 3 mm — Cut
  Preferred / Verified — L8 20W — Batch A
  Tested — L8 20W — Sheet 2

Baltic birch — 1/8 in — Cut
  Estimated — L8 40W
```

This is preferable to selecting a profile first because:

- it requires one action instead of two;
- the actual actionable entity is the production-setting record;
- grouped labels retain parent-material context;
- duplicate Library profiles remain distinguishable through thickness, operation, and IDs;
- a compact collapsed panel remains practical.

Store:

```javascript
profileId
settingId
```

Do not parse record identity from the visible label. A safe encoded composite option value may be used for the DOM, but resolving must still verify both IDs.

### Display requirements

After selection show:

- parent material;
- parent nominal thickness and unit;
- production-setting label;
- machine;
- operation;
- verification status;
- preferred status;
- measured thickness;
- measurement scope;
- batch label or Inventory snapshot;
- verified date;
- evidence count and concise entries;
- reference-only cut/LightBurn settings;
- compatibility warnings;
- applicable checkbox rows.

### Superseded records

Default chooser: exclude.

Optional session-only control:

```text
[ ] Show superseded records
```

If a superseded record is deliberately selected:

- display it as reference;
- show its replacement ID if available;
- disable all application targets;
- require selection of an active record before applying.

If an active selected record becomes superseded while selected, retain the selected IDs and show the stale-status warning. Do not silently switch to `supersededById`.

## 5. Applicability and mismatch rules

### Pure helper

Add a pure helper approximately shaped as:

```javascript
productionSettingDesignApplicability({
  profile,
  setting,
  draft,
  currentMachineKey
})
```

Recommended result:

```javascript
{
  validSelection,
  blockReasons: [],
  warnings: [],
  references: [],
  targets: [
    {
      key,
      draftKey,
      label,
      currentValue,
      proposedValue,
      resultingValue,
      enabled,
      blockReason,
      warning
    }
  ]
}
```

The helper must:

- normalize for reading without mutating the source;
- never write `designDraft`;
- never write Library data;
- never call `persist()`;
- never generate SVG;
- provide deterministic output.

### Operation

Phase 7.2 should require:

```text
setting.operation === "cut"
```

Unknown or non-cut operation:

- allow display as unavailable reference only if it somehow appears;
- disable all application targets;
- show `This production setting is not a cut record.`

### Machine

Map current preference:

```javascript
"20W" -> "genmitsu-l8-20w"
"40W" -> "genmitsu-l8-40w"
```

Rules:

| Condition | Thickness apply | Fit apply | Reference display |
|---|---:|---:|---:|
| Exact known machine | Yes | Yes | Yes |
| Known 20W/40W mismatch | Yes with warning | Block | Yes |
| Custom machine | Yes with warning | Block | Yes |
| Missing/unrecognized machine | Yes with warning | Block | Yes |

Do not scale power, speed, kerf, focus, or fit between machines.

Recommended mismatch text:

```text
Machine mismatch: this record was verified for Genmitsu L8 20W; the current app machine is Genmitsu L8 40W. Mechanical fit values cannot be applied across known machine profiles.
```

The measured material thickness may still be applied because it is a physical measurement, not a converted machine setting. Its sheet/batch scope warning remains visible.

### Material identity

Designs currently has no material-name field. It cannot independently prove that the selected Library profile matches the physical blank.

The UI must state:

```text
Designs does not track material identity. Selecting this record confirms that you intend to use the shown Library material and condition.
```

Do not invent automatic matching from dimensions alone.

### Thickness

Use measured thickness when present:

```javascript
setting.materialCondition.measuredThicknessMm
```

Do not fall back silently to:

- parent Library nominal thickness;
- production nominal thickness;
- Inventory name parsing.

If measured thickness is missing:

- show `Measured thickness is not recorded`;
- omit or disable the thickness checkbox;
- continue displaying other compatible values.

Comparison policy:

- exact numeric equality: `Current thickness matches the measured value`;
- any difference: show current, proposed, and signed delta;
- blank Design thickness: show `Current value is blank`;
- do not invent a universal safe tolerance;
- do not classify nominal match as proven compatibility.

Example:

```text
Current: 3.00 mm
Proposed: 3.02 mm
Change: +0.02 mm
```

The checkbox itself is the explicit overwrite confirmation. No second confirmation dialog is necessary if the row clearly shows current/proposed/resulting values.

### Measurement scope

Warnings:

- `sheet`: `Verified for one measured sheet; confirm this is the same sheet or remeasure.`
- `batch`: `Verified for one batch; variation may exist within or between sheets.`
- `material-general`: `Recorded as material-general evidence; it does not guarantee this sheet's thickness or fit.`
- `unknown`: `Measurement scope is unknown; treat the result as limited evidence.`

A scope warning does not by itself block explicit application.

### Verification status

Recommended wording:

- `verified`: `Physically verified under the recorded condition. Fit is still material- and setup-specific.`
- `tested`: `Tested result; assembled production verification is not recorded.`
- `estimated`: `Estimated or copied starting point; physical verification is required.`
- `untested`: `Untested saved record; use only as a starting point.`
- `superseded`: `Superseded historical record; application is disabled.`
- unknown: `Unrecognized verification status; application is disabled until reviewed in Library.`

Missing verified date on `verified`:

```text
Verified status has no verified date.
```

This is a warning, not an automatic rewrite.

### Partial and malformed records

The existing production normalizer already converts malformed known numeric fields to blank. Applicability should treat missing values independently:

- one missing field must not invalidate all other targets;
- malformed machine/operation/status should block only where necessary;
- reference display should use `-` or `Not recorded`;
- zero must remain a valid explicit value;
- negative signed joint values remain valid for Finger Box/Drawer Cabinet within their existing Designs validation.

The existing Design validator remains the final authority. If an applied value is outside a template's supported range, the normal preview error should appear exactly as if the user typed it.

## 6. Template-by-template field mapping

### Finger Box

Compatible targets:

```text
materialCondition.measuredThicknessMm
  -> designDraft.materialThickness

fitSettings.fingerJointClearanceMm
  -> designDraft.jointClearance
```

Do not apply:

- Drawer lateral clearance;
- speed/power/passes;
- kerf;
- focus;
- software.

Both thickness and joint clearance must have separate checkboxes.

### Drawer Cabinet

Compatible targets:

```text
materialCondition.measuredThicknessMm
  -> designDraft.materialThickness

fitSettings.fingerJointClearanceMm
  -> designDraft.cabinetJointClearance

fitSettings.fingerJointClearanceMm
  -> designDraft.drawerJointClearance

fitSettings.drawerTotalLateralClearanceMm
  -> designDraft.drawerLateralClearance
```

The single finger-joint source value must produce two independent target rows. This is the clearest representation of the current data-model limitation.

Do not add:

- an automatic “apply to both” default;
- a hidden combined mapping;
- inference that cabinet and drawer joints were separately proven.

Suggested explanation:

```text
This production record stores one finger-joint clearance. Cabinet and drawer targets are separate so you can decide explicitly whether the same tested value applies to either or both.
```

Defer distinct persisted cabinet/drawer production fields to a future data-model phase if physical evidence shows they routinely differ.

### Joint Fit Coupon

Compatible target:

```text
materialCondition.measuredThicknessMm
  -> designDraft.materialThickness
```

The stored finger-joint clearance should appear as:

```text
Reference winner: -0.02 mm
```

Phase 7.2 should not alter `jointCouponClearances`.

Reasons:

- the coupon requires 3–8 unique candidates;
- one stored winner is not a candidate set;
- replacing the list would reduce the test rather than configure it;
- inserting into the list requires ordering, deduplication, and range decisions beyond simple field application.

An explicit **Add to candidate list** action may be designed later, but should be deferred from this bounded phase.

### Sliding Lid Box

Compatible target:

```text
materialCondition.measuredThicknessMm
  -> designDraft.materialThickness
```

Defer all fit mappings:

- do not map general finger-joint clearance to `slidingJointClearance`;
- do not map it to `lidSideClearance`;
- do not map it to `lidVerticalClearance`;
- do not map it to `frontInsertionClearance`;
- do not map Drawer lateral clearance.

The current Sliding Lid body clearance rejects negative values, while the production model supports signed finger clearances. More importantly, the record does not identify a Sliding Lid body-joint context. A future field or evidence context is needed before that mapping is unambiguous.

### QR stand

Measured thickness is compatible because it directly controls the mating slot width:

```text
measuredThicknessMm -> materialThickness
```

Do not map finger-joint or Drawer values.

### Dice tray and Divider tray

Measured thickness is compatible because it controls:

- outside dimensions;
- slot widths;
- tab depth;
- layout.

Do not map production finger-joint clearance to legacy `fitClearance`; that field is slot clearance with different semantics.

### Hanging sign

Do not show the Production setting application panel.

Although the shared form currently requires `materialThickness`, hanging-sign SVG geometry does not consume it. Applying measured thickness would produce no SVG change and would add irrelevant UI.

### Other/future templates

Require an explicit mapping table entry before a new template receives any production target. Never infer compatibility because a draft happens to have a similarly named field.

## 7. Session-state proposal

Add one module-local object separate from `state` and `designDraft`:

```javascript
let designProductionSession = designProductionSessionDefaults();
```

Recommended shape:

```javascript
{
  profileId: "",
  settingId: "",
  checkedTargets: {},
  showSuperseded: false,
  reviewedUpdatedDate: "",
  staleReason: ""
}
```

`designProductionSessionDefaults()` should return a fresh object.

### Why it should be separate from `designDraft`

- production IDs are UI provenance, not geometry input;
- `updateDesignDraft()` copies all named fields and should remain geometry-focused;
- `normalizeDesignDraft()` should never receive Library IDs or production objects;
- fixture isolation is clearer;
- Reset can explicitly clear both session structures;
- backup exclusion remains obvious.

### Applied-state metadata

Do not implement per-field “modified after apply” indicators in Phase 7.2.

Once applied, values become ordinary editable Design fields. Tracking every subsequent modification would require:

- per-field snapshots;
- event-source distinctions;
- reset/template-switch rules;
- extra UI status that does not change generated geometry.

That complexity is not required for safe explicit application.

The only useful review token is `reviewedUpdatedDate`. If the selected Library record's `updatedDate` changes:

- retain selection;
- clear checked targets;
- show `This production setting changed in Library; review it again before applying.`

Do not retain a full last-applied record snapshot.

### Apply helper

Add a pure helper:

```javascript
applyDesignProductionValues(draft, applicability, checkedTargets)
```

It should return a new draft object:

```javascript
{
  ...draft,
  materialThickness: "3.02",
  jointClearance: "-0.02"
}
```

Requirements:

- no mutation of source `draft`;
- no mutation of profile/setting/applicability;
- only known enabled checked targets are applied;
- zero checked targets returns an unchanged-equivalent draft and no render side effect;
- values use deterministic millimeter text;
- unchecked fields remain byte-identical;
- repeated application is idempotent.

The UI handler may assign the returned object to `designDraft`, clear target checks, render, and show a concise summary.

## 8. Unit-handling plan

### Preserve canonical millimeters end-to-end

Production evidence:

```text
canonical millimeters
```

Designs fields:

```text
millimeter strings
```

Therefore:

- proposed values should be formatted with `designNumberText()` or a small equivalent deterministic formatter;
- do not convert through the global `state.unit`;
- do not call `convertStoredUnits()`;
- do not mutate `materialCondition`;
- do not introduce `designThicknessUnit`;
- do not display kerf without an explicit `mm` suffix.

### Global inches mode

When the app preference is inches, Designs still labels fields `(mm)`. A fixture described as “apply while UI uses inches” should mean:

1. set `state.unit = "in"`;
2. apply `3.02 mm`;
3. verify `designDraft.materialThickness === "3.02"`;
4. verify the production record remains numeric `3.02`;
5. verify the resulting SVG matches the same application under `state.unit = "mm"`.

This confirms the existing architecture rather than creating an inch-mode Designs form.

### Precision

Use the existing Designs rounding/serialization convention:

```javascript
designNumberText(value)
```

It retains meaningful signed clearances and avoids `-0`.

Do not round evidence in storage. Only format the copied Design string.

### Kerf

Always display:

```text
Signed LightBurn kerf offset: +0.095 mm
```

with:

```text
Production reference only. Apply the signed kerf offset to the red through-cut layer in LightBurn after verifying the direction in Preview. It is not included in SVG geometry.
```

No checkbox should target kerf in Designs.

## 9. UI and warning proposal

### Placement

Place a collapsed `<details>` block between the Designs heading and the ordinary `designForm`, or at the top of the form visually but outside its named form controls.

Recommended:

```html
<details id="designProductionSection" class="panel">
  <summary><b>Production setting</b> <span>optional</span></summary>
  ...
</details>
<form id="designForm">...</form>
```

Keeping it outside `designForm` prevents `updateDesignDraft()` from copying chooser/check fields into the geometry draft.

Do not show the panel for Hanging sign. For templates with thickness-only mapping, keep the detail compact.

### Selection summary

Example:

```text
Basswood plywood 3 mm
Verified basswood cut
Genmitsu L8 20W · Cut · Preferred · Verified
Measured 3.02 mm · Sheet · Batch A · 2026-07-16
Evidence: 2 records
```

### Reference-only block

```text
Production reference
Speed: 650 mm/min
Power: 100%
Passes: 1
Air assist: On
Air pressure: 15 PSI
Focus: 7 mm
Software: LightBurn
Signed kerf: +0.095 mm
```

This block has no Apply controls.

### Apply rows

Each row should show:

```text
[ ] Material thickness
Current: 3.00 mm
Proposed: 3.02 mm
Result after apply: 3.02 mm
```

For missing/blocking values:

```text
[disabled] Finger-joint clearance
Unavailable: machine mismatch
```

### Button behavior

Button:

```text
[Apply checked values]
```

Disabled when:

- no valid selected record;
- no compatible targets;
- zero targets checked;
- selected record is superseded/stale/invalid.

On success, show:

```text
Applied 2 values: Material thickness, Finger-joint clearance.
```

Do not say calibrated, guaranteed, or production safe.

### Overwrite wording

When current and proposed differ:

```text
Applying will replace the current Design value. The source record remains unchanged.
```

No automatic confirmation dialog is required if this warning and the current/proposed/result values are visible before the button click.

## 10. Stale-selection behavior

Resolve selected IDs on every `renderDesigns()`.

### Parent profile deleted

- do not substitute another profile;
- retain stale IDs long enough to show:

```text
The selected Library profile is no longer available. Already applied Design values are unchanged.
```

- disable Apply;
- offer `Clear selection`.

### Production setting deleted

Same behavior:

```text
The selected production setting is no longer available. Already applied Design values are unchanged.
```

### Record becomes superseded

- retain selection;
- display superseded status;
- clear checked targets;
- disable Apply;
- never follow `supersededById` automatically.

### Record edited

Compare current `updatedDate` with `reviewedUpdatedDate`.

If changed:

- show latest record data;
- clear checked targets;
- require review before applying;
- do not roll back or change already applied Design values.

If an imported record changes values without changing `updatedDate`, current data is still resolved and displayed. Fixtures should also compare a deterministic review fingerprint of application-relevant values if stronger protection is desired. A compact fingerprint of machine, operation, status, measured thickness, scope, fit values, and kerf is safer than trusting dates alone.

Recommended session field:

```javascript
reviewFingerprint
```

instead of or in addition to `reviewedUpdatedDate`.

### Inventory link disappears

The production record retains its snapshot fields under Phase 7.1. Designs should:

- display the snapshot;
- show `Linked Inventory item is no longer available`;
- continue normal applicability based on the production record;
- never recreate or modify Inventory.

### Unit change

Re-render and re-resolve selection. Do not change production evidence, checked targets, or applied Design values. Display remains millimeters.

## 11. Data-isolation requirements

Phase 7.2 actions must not write Library data.

Before and after each action, fixtures should compare serialized profiles:

- select setting;
- toggle Show superseded;
- check/uncheck targets;
- apply values;
- manually edit applied fields;
- refresh preview;
- download SVG;
- switch template;
- Reset defaults.

Expected:

```javascript
JSON.stringify(state.profiles) === before
```

### No persistence

Selection/application handlers must not call:

- `persist()`;
- `schedulePricingPersist()`;
- `saveProfileRecord()`;
- production-setting editor helpers;
- import/merge helpers.

Selection and apply should not change `localStorage`. Fixtures should compare the exact storage string before/after.

Reset must also remain session-only and must not write storage.

### Geometry boundary

Do not pass any of these into `normalizeDesignDraft()` or `buildDesignResult()`:

- selected profile ID;
- selected production-setting ID;
- machine;
- evidence;
- status;
- kerf;
- cut settings;
- whole production-setting object.

Only already-applied ordinary Design fields enter geometry.

### SVG isolation

Reference-only values must not appear in:

- SVG metadata;
- `<title>`;
- layer IDs;
- path offsets;
- blue score paths;
- file names.

Changing only speed/power/passes/air/focus/software/kerf in a selected source record must not change generated SVG bytes if no ordinary Design field is applied.

## 12. Fixture plan

Add a separate fixture group:

```javascript
runDesignProductionSettingsFixtures()
```

Recommended query:

```text
index.html?selftest=design-production
```

Include it in `?selftest=all`. Keep the existing 559 Designs geometry fixtures intact so geometry baseline changes remain easy to detect.

### Session isolation

- default session has no IDs or checked targets;
- selection changes only session object;
- selection is absent from `state`;
- selection is absent from `backupObject()`;
- selection/apply does not change localStorage;
- simulated reload/session-default recreation clears selection;
- Replace import does not create selection;
- Merge import does not create selection;
- Reset clears selection and draft;
- template switch preserves source IDs but clears checks.

### Explicit apply

- selection alone leaves `designDraft` byte-identical;
- zero checked targets returns unchanged draft;
- one checked target applies only that target;
- two checked targets apply exactly those targets;
- unchecked fields remain byte-identical;
- current/proposed/result values are correct;
- manual override after application remains;
- changing selected setting changes no Design value;
- repeated Apply is deterministic/idempotent;
- source objects remain unmutated.

### Finger Box

- thickness only;
- joint only;
- both;
- Drawer lateral omitted;
- signed zero and negative joint values preserved;
- known machine mismatch allows thickness but blocks joint.

### Drawer Cabinet

- thickness only;
- cabinet joint only;
- drawer joint only;
- both joint targets explicitly checked;
- lateral clearance only;
- any combination remains independent;
- unchecked cabinet/drawer fields remain unchanged;
- one stored joint value is never silently applied to both;
- negative signed values use existing validation.

### Joint Fit Coupon

- thickness applies;
- stored joint value displays as reference;
- candidate list remains byte-identical;
- no winner is declared;
- SVG changes only through applied thickness.

### Sliding Lid Box

- thickness applies;
- `slidingJointClearance` unchanged;
- `lidSideClearance` unchanged;
- `lidVerticalClearance` unchanged;
- `frontInsertionClearance` unchanged;
- Drawer lateral unchanged.

### Other templates

- QR stand receives thickness;
- Dice tray receives thickness;
- Divider tray receives thickness;
- Hanging sign exposes no applicable targets/panel;
- legacy `fitClearance` is never populated from production joint clearance.

### Machine and status

- exact 20W match;
- exact 40W match;
- known mismatch;
- custom machine;
- missing/unrecognized machine;
- no scaling;
- verified;
- verified without date;
- tested;
- estimated;
- untested;
- superseded;
- unknown status;
- non-cut operation.

### Thickness and scope

- current Design thickness blank;
- source measured thickness blank;
- exact measured match;
- nonzero delta shows exact difference;
- nominal match does not suppress measured mismatch warning;
- sheet warning;
- batch warning;
- material-general warning;
- unknown-scope warning;
- zero malformed fields remain unavailable according to current production normalizer bounds.

### Data isolation

- apply does not mutate profile;
- apply does not mutate production setting;
- apply does not mutate evidence;
- selection does not mutate anything persisted;
- preview does not mutate Library;
- download does not mutate Library;
- manual Design edits do not mutate Library;
- Reset does not mutate Library;
- no selection/application storage write;
- backup remains identical.

### SVG

- selection alone preserves exact SVG bytes;
- checking without applying preserves exact SVG bytes;
- kerf-only source changes preserve exact SVG bytes;
- cut-reference-only changes preserve exact SVG bytes;
- applied thickness changes SVG only where the template consumes thickness;
- applied fit changes the expected template geometry;
- preview/download identity remains exact;
- existing template signatures remain unchanged when no values are applied.

### Units

- production millimeter evidence unchanged;
- apply under global mm preference;
- apply under global in preference produces the same millimeter draft and SVG;
- repeated global unit switching does not drift Design values;
- signed clearance precision retained;
- kerf reference precision retained;
- no `convertStoredUnits()` interaction.

### Stale selection

- parent removed;
- setting removed;
- setting superseded;
- setting edited with new fingerprint;
- invalid profile ID;
- invalid setting ID;
- setting ID exists under a different parent;
- stale selection never substitutes another record;
- already applied values remain unchanged.

### Regression

The final implementation must retain:

- current complete 1157/0 baseline;
- Designs geometry 559/0;
- production settings 66/0;
- Material Tests;
- Project Wizard classification;
- backup/import/recovery;
- storage recovery 8/0;
- existing SVG signatures.

## 13. Manual browser plan

Use direct `file://` operation.

1. Create or use a Library profile with an active verified production setting.
2. Confirm the record includes measured thickness, one signed finger-joint value, Drawer lateral clearance, cut references, kerf, scope, and evidence.
3. Open Designs and choose Finger Box.
4. Expand Production setting and select the record.
5. Confirm no ordinary Design field changes.
6. Review material, machine, status, scope, evidence, and reference settings.
7. Check measured thickness only and apply.
8. Confirm only material thickness changes.
9. Check finger-joint clearance only and apply.
10. Confirm only joint clearance changes.
11. Manually override the applied joint value.
12. Select another production setting and confirm the manual value remains.
13. Switch to Drawer Cabinet.
14. Confirm source selection remains but all apply checks are cleared.
15. Apply cabinet joint only.
16. Apply drawer joint only.
17. Apply Drawer lateral only.
18. Confirm the three controls remain independent.
19. Switch to Joint Fit Coupon.
20. Apply thickness and confirm the candidate list is unchanged.
21. Switch to Sliding Lid Box.
22. Apply thickness and confirm all body/lid fit values remain unchanged.
23. Review kerf wording and confirm it is reference only.
24. Generate preview and download SVG.
25. Confirm preview/download bytes are identical.
26. Confirm Library record and evidence are unchanged.
27. Change global app unit to inches and repeat thickness application; verify Designs still uses the exact millimeter evidence.
28. Delete the selected setting in Library, return to Designs, and confirm stale selection is reported without losing applied values.
29. Restore/create a setting, select it, supersede it, and confirm application becomes disabled.
30. Reset Designs and confirm draft and production session return to defaults.
31. Reload and confirm selection is absent.
32. Export JSON and confirm no Designs production-selection metadata exists.
33. Run `?selftest=design-production`.
34. Run `?selftest=all`.

## 14. Expected files and functions affected

### Expected files

Implementation:

- `index.html`
- `README.md` only after verified behavior/totals make its current “Designs does not consume them yet” wording inaccurate
- implementation/audit reports under `docs/`

No new runtime file or dependency is needed.

### New functions recommended

```javascript
designProductionSessionDefaults()
designProductionSettingChoices()
resolveDesignProductionSelection()
designProductionMachineKey()
productionSettingDesignApplicability()
designProductionReferenceHtml()
designProductionApplicationHtml()
designProductionSectionHtml()
applyDesignProductionValues()
bindDesignProductionActions()
runDesignProductionSettingsFixtures()
```

Names may follow project conventions; responsibilities should remain separated.

### Existing functions that need modification

`renderDesigns()`

- insert the optional Production setting section;
- omit it for Hanging sign;
- continue building geometry from `designDraft` only.

Main render binding function near current `designForm` binding

- bind chooser, show-superseded, checkboxes, Clear selection, and Apply;
- handle template-switch check reset;
- do not call persistence.

Reset handler

- reset both `designDraft` and `designProductionSession`.

Self-test gating

- add `design-production`;
- include the new group in `all`.

Potential small display-helper reuse:

- `productionMachineLabel()`;
- `productionPowerLabel()`;
- `productionStatusLabel()`;
- evidence formatting logic may be extracted into a read-only helper if that avoids duplicating Library status semantics.

### Functions that should not need modification

- `designDefaults()` — ordinary numeric defaults remain unchanged;
- `updateDesignDraft()` — keep source-selection controls outside its named form;
- `normalizeDesignDraft()`;
- `buildDesignResult()`;
- `buildBoxDesignResult()`;
- `buildDrawerCabinetDesignResult()`;
- `buildJointCouponDesignResult()`;
- `buildSlidingLidDesignResult()`;
- shared finger geometry;
- `serializeDesignSvg()`;
- `designResultsHtml()`;
- `refreshDesignPreview()`;
- `downloadTextFile()`;
- `normalizeProductionSetting()`;
- `normalizeProductionSettings()`;
- production-setting Library lifecycle helpers;
- `backupObject()`;
- `persist()`;
- `replaceData()`;
- `mergeData()`;
- storage recovery.

If implementation requires changing geometry or production normalization, stop and reassess scope.

## 15. Risks and deferred behavior

### Primary risks

1. **Selection accidentally mutates the draft.**  
   Prevent by keeping chooser/check controls outside the named Designs form and using pure helpers.

2. **One stored joint value silently maps to two Drawer fields.**  
   Prevent with separate cabinet and drawer checkboxes.

3. **Known machine mismatch applies fit evidence.**  
   Block fit targets; never scale.

4. **Kerf leaks into geometry.**  
   Keep kerf in a reference-only block and fixture exact SVG byte stability.

5. **Library edits invalidate a checked proposal.**  
   Resolve by ID on render, fingerprint reviewed values, and clear checks when the record changes.

6. **Session metadata enters backup/state.**  
   Keep one module-local object and assert backup/storage exclusion.

7. **Global unit conversion drifts values.**  
   Preserve current Designs millimeter-only convention.

8. **Hanging sign receives irrelevant controls.**  
   Omit the panel because thickness does not affect its SVG.

### Explicitly deferred

- applying speed/power/passes/air/focus/software into a LightBurn file;
- SVG metadata for production references;
- automatic LightBurn layer creation;
- kerf path offsets;
- cross-machine scaling or overrides;
- mapping general joint clearance to Sliding Lid body/lid clearances;
- modifying Joint Fit Coupon candidate lists;
- declaring a stored clearance the coupon winner;
- distinct persisted cabinet/drawer joint fields;
- last-used timestamps, counters, or evidence write-back;
- persistent selected profile/setting;
- full modified-after-apply tracking;
- automatic material identity matching in Designs;
- promotion from Material Tests, coupons, or Projects.

## 16. Independent-audit recommendation

An independent focused audit is required after implementation and before physical reliance.

This phase connects persisted evidence to production geometry inputs. The audit should verify:

- selection alone is inert;
- every mapping is template-specific and explicit;
- Drawer Cabinet joint targets are independent;
- machine mismatch blocks fit application;
- no cross-machine scaling;
- kerf and cut references never change SVG;
- no Library write-back;
- no localStorage write from selection/apply/reset;
- no selection in backups/import/schema;
- stale IDs never auto-substitute;
- applied values are ordinary editable fields;
- millimeter handling is deterministic under both global unit preferences;
- existing SVG signatures remain stable when nothing is applied;
- all original and new fixtures pass.

Audit priority becomes Blocker if implementation touches:

- `normalizeDesignDraft()`;
- `buildDesignResult()`;
- shared geometry;
- `serializeDesignSvg()`;
- production normalizers;
- Library save paths;
- `persist()` or `backupObject()`.

Physical prototyping remains necessary after software audit. The application can verify explicit data flow, but it cannot prove material batch consistency, kerf direction, finger fit, drawer travel, glue behavior, focus, or machine setup.

## 17. Explicit conclusion

**READY FOR PHASE 7.2 IMPLEMENTATION**

The current architecture supports the phase without schema, persistence, import, production-model, or geometry changes. The implementation should remain a session-only UI adapter that resolves a selected Library production record, presents applicability and evidence, and copies only explicitly checked compatible millimeter values into ordinary `designDraft` fields.

