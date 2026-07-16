# Phase 7 Material Profiles Architecture Review

Date: 2026-07-16
Repository: `C:\Genmitsu L8 Tracker`
Reviewed baseline: `c93ebd1` — `Add production joint-fit tuning`
Review type: read-only architecture, schema, workflow, and compatibility review

## 1. Executive recommendation

Phase 7 should add an optional array of typed production-setting records to each existing Library profile. The recommended persisted field is:

```javascript
profile.productionSettings = []
```

Each entry should represent one bounded, reproducible production condition: one material profile, one machine, one operation, one measured material condition, one settings snapshot, optional fit values, and one or more evidence references. This is safer than putting more “winning” fields directly on the material, overloading Material Tests, or introducing a separate top-level collection.

The existing Library profile should remain the parent material/thickness/operation record and retain every current field unchanged. Existing primary settings remain a useful saved recipe, but they should not become proven merely because they are populated. Existing Material Tests remain raw and interpreted test evidence. The new production-setting record becomes the explicit promoted, reviewable result that connects evidence to a specific machine and material condition.

Phase 7.1 should be deliberately bounded:

- Add and normalize `profile.productionSettings`.
- Provide manual add, edit, archive/supersede, and delete controls inside the existing Library material editor.
- Support cut-production records first, including measured thickness, machine, cut settings, signed LightBurn kerf offset, finger-joint clearance, and Drawer Cabinet total lateral clearance.
- Store typed evidence references without attempting automatic physical-result detection.
- Preserve old Library fields, tests, IDs, notes, unknown fields, and backup compatibility.
- Add fixtures for editor round-tripping, normalization, duplicate handling, merge/replace import, and unknown-field preservation.

Designs integration should wait for Phase 7.2. Test/coupon promotion should wait for Phase 7.3. This sequencing keeps the first implementation focused on trustworthy storage before the app begins consuming the data.

No `STORAGE_KEY` change is recommended. A `SCHEMA_VERSION` bump is not required for an optional nested field if normalization and import behavior remain backward-compatible. The more important work is preserving the new field through material edits, unit conversion, duplication, merge import, and unknown-field round-trips.

Current verification baseline:

- `git diff --check`: passed before report creation.
- HTML parser: passed.
- Direct `file://` startup: passed.
- Complete built-in fixture suite: 1091 passed / 0 failed.
- No external script, stylesheet, or runtime dependency is used by the app.

## 2. Existing material and test architecture

### Library profile structure

Library materials are stored in the top-level `state.profiles` array. A profile is created from `formDefaults('profile')` and `normalizeSetting()`. Its established fields are:

```javascript
{
  id,
  material,
  thicknessValue,
  thicknessUnit,
  jobType,
  minPower,
  maxPower,
  speed,
  passes,
  lineInterval,
  airAssist,
  airPressure,
  overscanPercent,
  kerfOffset,
  ditherMode,
  focusValue,
  focusUnit,
  materialHeightValue,
  materialHeightUnit,
  software,
  notes,
  tags,
  tests
}
```

The form requires a nonblank material name. Most numeric setting fields are optional in practical use, although `normalizeSetting()` converts an empty minimum power and speed through the existing numeric helper and ensures at least one pass. Existing data can therefore contain mixtures of blank, zero, and populated setting fields.

The profile `id` is the record identity. Browser grouping uses normalized material, thickness, and operation labels, but Library leaf identity and selection are ID-based. Multiple profiles may legitimately have the same display material, thickness, operation, or title.

### Thickness representation

Library thickness is stored as a value/unit pair:

```javascript
thicknessValue
thicknessUnit // "mm" or "in"
```

The editor help currently calls this actual material thickness. In the rest of the application, however, it also serves as:

- the Library Browser thickness grouping value;
- the closest-setting lookup value;
- the Inventory-to-Library matching value;
- the Project Wizard cutting-compatibility value;
- a copied Project snapshot value.

It therefore behaves as both a material/profile thickness and an assumed tested thickness. It does not record when the measurement occurred, whether it was nominal or measured, whether it applied to one sheet or a batch, or which measuring method was used. It is not an adequate authoritative location for Phase 7 measured-thickness evidence by itself.

### Primary settings

The top-level Library setting fields form one primary recipe. The Library status helpers correctly distinguish:

- starter/unverified;
- Best Known when a recommended Material Test exists;
- Experimental when tests exist but none are recommended;
- Saved setting when primary settings exist without tests;
- Unverified when neither settings nor tests are meaningful.

The Project Wizard treats a primary Library recipe as `Saved setting`, not `Best Known`, because it has no machine provenance. This is the correct precedent for Phase 7.

### Material Test structure

Each Library profile has an optional `tests` array normalized by `normalizeMaterialTests()`. Known fields include:

```javascript
{
  id,
  date,
  machine,
  testType,
  status,
  notes,
  winningSpeed,
  winningPower,
  winningInterval,
  winningPasses,
  airAssist,
  confidence,
  attachments,
  overallResult,
  observedTraits,
  winnerLocation,
  winnerComment,
  resultSummary,
  intendedOperation,
  adaptiveMode,
  optimizationGoal,
  recommendationRationale,
  baselineSource,
  baselineLabel,
  baselineTestId
}
```

The normalizer spreads the source object before setting known normalized fields, which preserves unknown future fields. The test model already carries useful extension metadata from wizard and Test Grid paths.

Test Grid promotion also adds provenance such as:

```javascript
{
  sourceKind: "test-grid",
  sourceGridId,
  sourceCellKey,
  sourceGridName
}
```

This demonstrates a working pattern: preserve a raw test result and add stable source references rather than flattening all provenance into notes.

### Material Test creation and promotion

Material Tests can be:

- manually added or edited inside a Library material;
- created by the Material Test Wizard;
- recorded while promoting a Test Grid cell;
- duplicated when a Library profile is duplicated.

Test Grid promotion can copy a cell’s settings into a new Library draft and optionally append a structured Material Test. Duplicate promotion is detected by `sourceGridId` plus `sourceCellKey`.

The Material Test Wizard can resolve and save a winner and can explicitly apply winner values to the Library primary settings. This write is user-controlled. It is an important precedent: evidence capture and primary-setting mutation are separate actions.

### Machine behavior

The app has one global preference:

```javascript
state.machineProfile // "20W" or "40W"
```

It drives Reference content and the current wizard machine label. It is not a machine record, immutable machine ID, lens record, or calibration history.

Material Tests store `machine` as text. The Project Wizard canonicalizes a small set of 20W/40W aliases and requires an exact known machine for production test copying. It performs no wattage conversion. Primary Library settings have no machine field and are treated as machine-unknown.

### Inventory and batches

Inventory has two top-level arrays:

```javascript
inventory.rawMaterials
inventory.finishedBatches
```

Raw materials have ID, name, aliases, category, quantity, unit, cost, supplier, purchase date, low-stock threshold, tags, and notes. Thickness is inferred from name/notes for browser matching; it is not a dedicated raw-material field.

Finished batches represent finished products linked optionally to a Project. They are not raw sheet lots. The current Inventory model cannot authoritatively distinguish individual sheets or raw-material purchase batches.

### Creation, editing, copying, deletion, and import

- `openProfile()` creates and edits Library profiles.
- `normalizeSetting()` rebuilds the recognized top-level setting fields.
- The submit path then adds normalized `tests`.
- `duplicateProfile()` creates a new profile ID and duplicates Material Tests with new test IDs.
- Generic delete removes a profile by ID with the existing undo behavior.
- `loadState()` normalizes Library profiles and tests.
- `backupObject()` includes `profiles` unchanged.
- Replace import normalizes incoming profiles.
- Merge import replaces an existing entire profile when the incoming profile has the same ID.

The edit path is the most immediate Phase 7 compatibility hazard: because it rebuilds a profile from recognized form fields, a new nested field will be lost on edit unless the implementation explicitly carries forward unknown/unrelated fields.

## 3. Current sources of truth and conflicts

The application currently has several related but intentionally different setting sources:

1. Official Reference rows are manufacturer starting points.
2. Library primary fields are saved settings without machine provenance.
3. Material Tests are raw or interpreted experimental evidence.
4. A recommended Material Test can make the Library profile display Best Known.
5. Test Grid cells are matrix observations that can be promoted into tests.
6. Project settings are immutable-at-save production snapshots but are not automatically evidence of success.
7. Designs fields are session-only dimensional inputs and fit assumptions.
8. The global machine preference selects Reference/wizard context but is not persistent machine identity for each recipe.

The main conflicts are:

- The Library top-level thickness is both a grouping value and an implied actual value.
- Library primary settings and Material Test winners can both describe the “current” recipe.
- A recommended test communicates recommendation but does not prove an assembled result.
- A Project snapshot may represent a physically successful job, but rating/status alone does not prove which setting was verified.
- `kerfOffset` exists on Library, Log, and Project settings, while Designs correctly treats kerf as external LightBurn setup.
- Designs fit fields are independent session inputs and have no material or machine provenance.
- Inventory can identify a purchased item and supplier but not a raw-material lot or individual measured sheet.

Phase 7 should not choose one of these existing objects and declare it universally authoritative. It should introduce one explicit promoted record that:

- references evidence;
- snapshots the relevant settings;
- states the machine and material condition;
- records verification level;
- can be superseded without deleting source evidence.

The new record should be authoritative only for its own declared applicability. It should not silently rewrite Material Tests, Projects, Inventory, or Designs.

## 4. Recommended profile data model

### Placement decision

Recommended:

```javascript
profile.productionSettings = []
```

Alternatives considered:

- **Direct fields on the material:** too limited for multiple machines, batches, and historical results.
- **Only inside Material Tests:** conflates raw observations with promoted production decisions and cannot cleanly represent assembled-project verification.
- **A promoted-test object only:** too test-centric for manual, coupon, and Project evidence.
- **Separate top-level collection:** adds referential-integrity, deletion, merge, and orphan-management complexity without a current cross-material use case.
- **Hybrid top-level index plus nested records:** premature for the single-file application.

Nesting under the Library profile gives clear ownership, automatic backup inclusion, straightforward deletion semantics, and a natural Library UI location. Each nested record still needs its own immutable ID.

### Proposed Phase 7.1 shape

```javascript
{
  id: "stable-production-setting-id",
  label: "Basswood 3 mm - L8 20W verified cut",
  operation: "cut",

  machine: {
    key: "genmitsu-l8-20w",
    label: "Genmitsu L8 20W",
    lens: "",
    focusMethod: ""
  },

  materialCondition: {
    nominalThicknessMm: 3,
    measuredThicknessMm: 3.02,
    measurementScope: "sheet",
    measuredDate: "2026-07-16",
    inventoryItemId: "",
    inventoryItemName: "",
    supplier: "",
    product: "",
    batchLabel: "",
    finishOrCoating: ""
  },

  cutSettings: {
    speedMmPerMin: 650,
    powerMinPercent: 100,
    powerMaxPercent: 100,
    passes: 1,
    airAssist: true,
    airPressurePsi: "",
    focusValue: "",
    focusUnit: "mm",
    software: "LightBurn"
  },

  lightBurn: {
    kerfOffsetMm: 0.095
  },

  fitSettings: {
    fingerJointClearanceMm: -0.02,
    drawerTotalLateralClearanceMm: 0.45
  },

  verificationStatus: "verified",
  preferred: true,
  confidence: 5,
  verifiedDate: "2026-07-16",
  notes: "",

  evidence: [
    {
      id: "stable-evidence-id",
      kind: "joint-coupon",
      sourceId: "",
      sourceProfileId: "parent-profile-id",
      date: "2026-07-16",
      result: "winner",
      notes: "Preferred coupon fit."
    },
    {
      id: "stable-evidence-id-2",
      kind: "assembled-project",
      sourceId: "project-id-if-available",
      sourceProfileId: "parent-profile-id",
      date: "2026-07-16",
      result: "verified",
      notes: "Small open-top box held without glue."
    }
  ],

  supersededById: "",
  createdDate: "2026-07-16",
  updatedDate: "2026-07-16"
}
```

### Why typed subobjects are justified

This is not a generic settings blob. Each subobject has a distinct meaning:

- `machine` defines hardware applicability.
- `materialCondition` defines what was physically tested.
- `cutSettings` snapshots the laser operation.
- `lightBurn` contains software-layer settings that must not alter SVG geometry.
- `fitSettings` contains mechanical allowances.
- `evidence` explains why the record is trusted.
- verification fields describe confidence and lifecycle.

Keeping these concerns separate prevents a Drawer Cabinet running clearance from being mistaken for a finger-joint clearance or a LightBurn kerf offset from being treated as generated geometry.

### Phase 7.1 bounds

The data structure may be future-capable, but the first editor should expose only:

- cut operation;
- known L8 20W/L8 40W machine choices plus a custom label;
- measured thickness and scope;
- speed, min/max power, passes, air assist, air pressure, focus, and software;
- signed kerf offset;
- finger-joint clearance;
- Drawer Cabinet total lateral clearance;
- verification status, preferred flag, confidence, dates, evidence type/reference, and notes.

Score, engraving, detailed lens catalogs, machine maintenance state, and supplier catalogs should be deferred.

### Record identity and preferred selection

The nested `id` is the only record identity. No composite of material/machine/thickness should replace it.

Applicability can be compared using parent profile, machine key, operation, and material condition, but multiple records for the same condition must remain legal so evidence history is retained.

Use:

```javascript
verificationStatus: "untested" | "estimated" | "tested" | "verified" | "superseded"
preferred: boolean
supersededById: string
```

At most one non-superseded preferred record should exist for the same parent profile, operation, machine key, and intended fit context. Marking a new record preferred should explicitly ask whether to demote the old preferred record. Demotion must not delete it or its evidence.

## 5. Evidence and verification model

Use a small status enum plus optional numeric confidence and typed evidence records.

### Recommended statuses

- `untested`: manually entered information with no test claim.
- `estimated`: calculated, manufacturer-derived, or copied starting point.
- `tested`: a Material Test or coupon produced a selected result, but no assembled production verification is recorded.
- `verified`: physically confirmed in an assembled or production-representative result.
- `superseded`: retained history that should not be suggested by default.

### Evidence kinds

Phase 7.1 should recognize:

- `manufacturer`
- `material-test`
- `test-grid`
- `joint-coupon`
- `assembled-project`
- `manual`

Unknown future kinds should be preserved and displayed as an unrecognized evidence type rather than deleted.

### Confidence

Retain optional confidence from 1–5 because the existing Material Test model and UI already use that scale. Do not derive verification status automatically from confidence:

- confidence 5 on a coupon can still be `tested`;
- an assembled result can be `verified` with lower confidence if material variation is suspected;
- a manufacturer starting point remains `estimated` regardless of confidence.

The UI should explain status first and display confidence second. Notes remain essential for physical observations.

### Evidence rules

- Evidence references should be additive.
- Source deletion should create a dangling-reference display, not delete the production setting.
- Store a short snapshot label/date alongside `sourceId` so the evidence remains understandable if the source is later deleted.
- Never replace or mutate the source Material Test, Test Grid, or Project when editing a production setting.
- Promotion should copy numeric values into the production record while retaining the source link.

## 6. Machine/material/thickness identity rules

### Material identity

The parent Library profile ID is the authoritative material-profile owner. Material names and aliases are matching aids, not identity.

Do not key a production record by normalized material text. Duplicate names and punctuation variants are already legal and supported.

### Machine identity

Phase 7.1 should introduce a stable machine key inside each production record. It does not require a full top-level machine registry yet.

Recommended initial keys:

```text
genmitsu-l8-20w
genmitsu-l8-40w
custom
```

For `custom`, require a nonblank display label. Do not infer compatibility between 20W and 40W. Do not scale settings across machines.

Lens and focus method can remain optional text in Phase 7.1. A future machine registry can replace or supplement this without changing record identity.

### Nominal versus measured thickness

- Parent profile `thicknessValue`/`thicknessUnit`: existing nominal/reference/grouping thickness.
- Production record `materialCondition.nominalThicknessMm`: snapshot of the expected thickness when verified.
- Production record `materialCondition.measuredThicknessMm`: actual measurement used for the result.
- `measurementScope`: `sheet`, `batch`, `material-general`, or `unknown`.
- `measuredDate`: when the measurement was taken.

Store new production-condition dimensions canonically in millimeters. The editor may accept current display units and convert on save. This avoids changing historical evidence when the global display unit toggles.

### Inventory and batch links

`inventoryItemId` may link to a raw Inventory item, but also store `inventoryItemName`, supplier/product, and batch label snapshots. The current Inventory record can be renamed or deleted, and it is not a true raw-material batch entity.

Phase 7.1 should not add raw-sheet entities. Use:

- optional Inventory link;
- measurement scope;
- optional supplier/product/batch text;
- warning text when applying a sheet- or batch-scoped result elsewhere.

### Mismatch handling

Machine mismatch must block automatic application.

Material mismatch must require explicit confirmation.

Measured-thickness mismatch should use the existing exact-thickness concepts where possible. The current Project Wizard groups equivalent thickness values and blocks cutting when reliable thickness is missing or mismatched. Phase 7.2 should reuse that policy initially rather than introduce a second independent tolerance system.

For Designs:

- no measured target thickness: show “target thickness not measured” and require review;
- same canonical thickness group: eligible;
- different group: warning and no default application;
- sheet- or batch-scoped record applied to another item: warning even if thickness matches.

### Supplier, finish, and coating

These fields affect real production but should be optional context in Phase 7.1, not matching keys. Matching on them automatically would create brittle records and false incompatibility. Phase 7.4 can add richer filtering if actual usage demonstrates the need.

## 7. Library UI proposal

Keep the existing material editor intact and add one collapsed section after Material Tests:

```text
Proven Production Settings
--------------------------
1 preferred / 2 saved

Preferred:
Genmitsu L8 20W — Cut
Measured 3.02 mm — Verified 2026-07-16
650 mm/min / 100% / 1 pass / Air On
Kerf +0.095 mm
Joint -0.02 mm / Drawer lateral 0.45 mm

[Add setting] [Edit] [View evidence] [Supersede]
```

The Add/Edit dialog should use small subsections:

```text
Identity
  Label
  Machine
  Operation

Material condition
  Nominal thickness
  Measured thickness
  Applies to: sheet / batch / material generally / unknown
  Measured date
  Optional Inventory item / supplier / product / batch

Cut settings
  Speed / power min / power max / passes
  Air assist / pressure
  Focus / software

LightBurn
  Signed kerf offset
  Help: apply only to the red through-cut layer in LightBurn

Fit settings
  Finger-joint clearance
  Drawer Cabinet total lateral clearance

Verification
  Status / preferred / confidence / verified date
  Evidence kind / source / notes
```

Library cards and Browse detail should show only a concise summary initially. Do not add another dashboard or restructure the Browser tree in Phase 7.1.

Suggested badges:

- `Verified production setting`
- `Tested production setting`
- `Estimated setting`
- `Superseded`

The existing Library status badge should remain based on current Material Test rules until a separate, carefully reviewed change decides whether verified production settings should influence it. Do not silently redefine `Best Known` in Phase 7.1.

## 8. Designs integration proposal

Phase 7.2 should add a session-only selector near material thickness for templates that use production settings:

```text
Material production setting
[3 mm Basswood — L8 20W — Verified ▼]

Applicability
Machine: exact match
Nominal thickness: 3.00 mm
Measured thickness: 3.02 mm
Scope: one sheet
Verified: 2026-07-16

Apply to this design:
[ ] Measured thickness: 3.02 mm
[ ] Finger-joint clearance: -0.02 mm
[ ] Drawer total lateral clearance: 0.45 mm

Production reference only:
650 mm/min / 100% / 1 pass / Air On
LightBurn cut-layer kerf offset: +0.095 mm

[Apply checked values]
```

Rules:

- Profile selection is session-only.
- Selection alone changes no design value.
- Only explicitly checked compatible values are applied.
- Applied fields remain ordinary editable Design fields.
- User overrides after application are allowed and visibly marked as modified if practical.
- Changing the selected profile must not overwrite already edited values without another Apply action.
- Cut speed, power, passes, air assist, and kerf are displayed as production references but not embedded in SVG.
- SVG generation performs no Library write.
- Download and preview remain byte-identical.
- Reset Designs returns to existing defaults, not to the selected profile.
- No selected profile ID belongs in `backupObject()` unless a later phase deliberately makes Designs persistent.

Template applicability:

- Finger Box: measured thickness and finger-joint clearance.
- Drawer Cabinet: measured thickness, cabinet/drawer joint clearance only if separately recorded, and total drawer lateral clearance.
- Joint Fit Coupon: measured thickness and optional starting clearance list guidance, but not automatic winner selection.
- Sliding Lid Box: defer profile fit application until its separate lid and joint clearances are represented explicitly in production settings.
- Other templates: no fit application in the first Designs integration.

## 9. Test and coupon integration proposal

### Material Tests

Phase 7.3 may add:

```text
[Promote to production setting]
```

for eligible cut and kerf tests.

Promotion should:

- create a new production-setting record with a new ID;
- copy the test’s winning numeric settings;
- copy parent profile material/thickness context;
- require machine confirmation;
- require measured-thickness confirmation;
- set evidence kind/source ID;
- default verification status to `tested`;
- never mark `verified` automatically;
- never delete or modify the source test;
- ask before replacing an existing preferred record.

### Test Grids

Test Grids should continue promoting first to Material Tests. Direct Test Grid-to-production promotion should be deferred. The current grid result records only a cell rating/best flag and is not sufficient evidence of production verification.

### Joint Fit Coupon

The app can know automatically:

- generated clearance candidates;
- selected Design material thickness input;
- generated geometry;
- whether labels were enabled;
- the date when saving occurs.

It cannot know automatically:

- which physical pair won;
- actual measured sheet thickness unless entered;
- machine used;
- LightBurn kerf offset;
- speed/power/passes/air assist;
- whether the fit survived assembly;
- whether glue was needed;
- whether the result is repeatable.

The future Save Result flow should require manual winner selection and production context:

```text
Winning clearance: [-0.02 mm]
Measured thickness: [3.02 mm]
Machine: [Genmitsu L8 20W]
Cut settings: [...]
LightBurn kerf offset: [+0.095 mm]
Physical result: [preferred / too tight / too loose / damaged]
Notes: [...]
[Save as tested production setting]
```

Status should default to `tested`, not `verified`.

### Drawer Cabinet

The proven 0.45 mm value belongs in:

```javascript
fitSettings.drawerTotalLateralClearanceMm
```

inside a machine/material/thickness production setting. It is a Drawer Cabinet running-fit result, not a general joint-clearance value and not equivalent to finger-joint clearance.

An assembled Drawer Cabinet can add `assembled-project` evidence and elevate the record to `verified` only through explicit user action.

### Superseding

Promoting a newer result should offer:

```text
This condition already has a preferred setting.
[Keep both] [Make new preferred and supersede old] [Cancel]
```

The old record and all source evidence must remain intact.

## 10. Kerf offset representation

Store:

```javascript
lightBurn.kerfOffsetMm
```

as a signed numeric millimeter value.

The record must also retain the cut settings and machine/material condition under which it was measured. A kerf value without those conditions is not a portable material constant.

UI help should state:

> Signed LightBurn kerf offset used for the red through-cut layer under these exact material, machine, focus, air-assist, power, speed, and pass conditions. The sign is preserved exactly as entered. Verify the direction in LightBurn preview. This value is not applied to blue score/engrave geometry and is not baked into generated SVG dimensions.

The sign convention needs explicit help because positive and negative effects can be misunderstood when path direction, inner/outer contours, or software offset modes differ. The app should not reinterpret the sign. It should store and display the exact signed value the user verified.

Do not:

- change panel dimensions;
- offset SVG paths;
- attach hidden LightBurn-specific metadata to ordinary SVG;
- apply the value to blue score geometry;
- infer the value from finger-joint clearance;
- combine kerf and fit clearance into one field.

Future export formats may include a human-readable production sheet or LightBurn-specific integration, but that is outside Phase 7.

## 11. Schema and migration assessment

### STORAGE_KEY

Keep:

```javascript
genmitsu-l8-tracker-v1
```

Changing the key would strand existing local data and is unnecessary.

### SCHEMA_VERSION

Keep `SCHEMA_VERSION === 2` for Phase 7.1 if implementation is strictly additive:

- `productionSettings` is optional;
- missing fields normalize to `[]`;
- old records remain valid;
- no existing field changes meaning or representation;
- import does not require destructive migration.

A version bump would be justified only if an existing field is reinterpreted, data is moved, or older data requires a one-way migration. That is not recommended.

### Load normalization

Add a pure helper:

```javascript
normalizeProductionSettings(settings)
```

and extend:

```javascript
normalizeLibraryProfiles()
```

to produce:

```javascript
{
  ...profile,
  tests: normalizeMaterialTests(profile.tests),
  productionSettings: normalizeProductionSettings(profile.productionSettings)
}
```

Requirements:

- source objects and arrays are not mutated;
- missing/non-array value becomes `[]`;
- malformed non-object entries are excluded;
- known fields receive conservative defaults;
- unknown fields are preserved;
- known nested unknown fields are preserved through object spreads;
- valid IDs remain unchanged;
- normalization is stable when repeated;
- numeric zero and negative fit/kerf values remain valid;
- malformed numeric strings do not silently become zero.

The last point requires more care than existing broad `num()` use. For production evidence, blank and invalid are materially different from zero. Use a strict optional-number parser.

### Unit conversion

New production-setting dimensional evidence should be stored canonically in millimeters and should not be modified by `convertStoredUnits()`. The global unit toggle may change editor/display formatting only. Historical evidence should not be repeatedly converted.

### Profile editor preservation

Current `openProfile()` submission reconstructs the profile. Phase 7.1 must merge the normalized form result into the existing profile:

```javascript
const previous = existing profile;
const profileObj = {
  ...(previous || {}),
  ...normalizeSetting(form, id),
  tests: normalizeMaterialTests(profileDraftTests),
  productionSettings: normalizeProductionSettings(profileDraftProductionSettings)
};
```

This preserves unknown and future profile fields. Tests and production settings should be maintained in separate modal draft arrays.

### Storage recovery

No storage-recovery architecture change is required. Corrupt top-level JSON should continue to trigger the existing recovery path. Partially malformed production settings should normalize safely without making the whole backup unreadable.

## 12. Backup/import/export compatibility

### JSON backup

Because profiles are already included in `backupObject()`, nested production settings will export automatically. Add fixtures proving exact round-trip preservation.

### Older backups

Profiles without `productionSettings` load as an empty array. No migration prompt or mutation is needed.

### Replace import

Replace import may replace the complete Library collection by design. It should normalize production settings and preserve unknown fields.

### Merge import

Current `mergeList()` replaces an entire existing profile when IDs match. With nested evidence, that can erase locally added Material Tests or production settings when an older copy of the same profile is imported.

Phase 7.1 should add a Library-specific merge helper:

```javascript
mergeLibraryProfiles(existingProfiles, incomingProfiles)
```

For a matching profile ID:

- incoming recognized scalar fields may retain current merge semantics;
- tests should be merged by test ID, not replace the entire array;
- production settings should be merged by production-setting ID;
- evidence arrays inside matching production settings should be merged by evidence ID;
- deterministic ordering must not depend on input array order;
- unknown fields from both sides should be preserved where they do not conflict;
- direct same-field conflicts should follow a documented incoming-wins rule;
- merge statistics should still count the parent profile as updated.

Changing test-array merge behavior is a compatibility improvement but must receive dedicated fixtures because it is broader than merely adding the new field. If that change is considered too broad for 7.1, the UI must warn that merge import replaces matching Library profiles. Silent nested evidence loss is not acceptable.

### Newer backups into older app versions

The current normalizer spreads unknown profile fields, so an older app may load and export `productionSettings` unchanged. However, editing that material in an older version will rebuild the profile and discard the unknown field. Full backward edit compatibility cannot be guaranteed without changing old releases.

Document:

- newer backups should not be edited in an older application version;
- keep a copy of the original backup;
- importing into the current/newer app is the supported path.

### Duplicate materials

`duplicateProfile()` currently duplicates Material Tests with fresh IDs. For production settings, the safest Phase 7.1 behavior is:

- duplicate records with new production-setting and evidence IDs;
- preserve values and source snapshots;
- set `preferred: false`;
- downgrade `verificationStatus` to `estimated`;
- add a note that the record was copied from another Library profile.

This prevents verified evidence for one material profile from automatically becoming verified evidence for the duplicate. The UI should state this behavior.

### Corrupted nested records

- Non-object array entries: omit and count as normalization issues.
- Missing ID: generate a new ID during import/load normalization and preserve it on the next save.
- Invalid status: default to `untested`.
- Invalid machine key: preserve label and use `custom`/unknown.
- Invalid numeric field: use blank, not zero.
- Unknown operation/evidence kind: preserve raw value and display as unrecognized.
- Dangling source ID: keep the evidence snapshot and show source unavailable.

A future import review screen could report normalization issues, but Phase 7.1 can remain bounded if fixtures prove no crash and no unrelated data loss.

## 13. Data-preservation risks

### Blocker-level risks for implementation

1. **Profile edit drops nested fields.**

   Current form save reconstructs the profile. The implementation must preserve the existing object and explicitly carry production settings.

2. **Merge import replaces nested arrays.**

   Current profile-level incoming-wins replacement can erase tests and production evidence.

3. **Invalid numeric values become zero.**

   Existing `num()` fallback behavior would turn malformed evidence into a meaningful zero clearance or kerf. Use strict optional parsing.

4. **Existing preferred record is overwritten.**

   New preferred selection must demote or supersede explicitly, never mutate another record silently.

### Major risks

- Reusing parent `thicknessValue` as measured-sheet truth.
- Treating a recommended Material Test as assembled verification.
- Copying a verified record to a duplicate material unchanged.
- Matching machines by display label alone.
- Applying sheet-scoped evidence to another sheet without warning.
- Allowing a production record to mutate its source test/project.
- Applying kerf to SVG geometry.
- Conflating Drawer Cabinet lateral clearance with joint clearance.
- Generating new IDs during every normalization pass.
- Sorting by mutable display labels and destabilizing selection.

### Safeguards

- Immutable IDs at every persisted nested level.
- Pure normalizers with source-mutation fixtures.
- Unknown-field preservation fixtures.
- Explicit preferred/supersede workflow.
- Source snapshots next to optional IDs.
- Parent profile ownership by ID.
- Exact machine compatibility for application.
- Explicit checkbox-based Designs application.
- No write-back from SVG generation.
- Rollback on `persist()` failure, matching existing save patterns.
- Backup before any implementation-time manual migration test.

## 14. Fixture plan

### Normalization

- Missing `productionSettings` becomes `[]`.
- Non-array value becomes `[]`.
- Malformed entries are removed without crashing.
- Known valid record remains stable through repeated normalization.
- Unknown top-level and nested future fields survive.
- Valid IDs remain unchanged.
- Missing IDs are generated once and do not mutate source input.
- Zero and negative kerf/joint values survive.
- Invalid numbers become blank, not zero.
- Invalid status defaults safely.
- Unknown evidence kind is preserved.
- Evidence arrays normalize without source mutation.

### Library editor

- Creating a production setting appends one record.
- Editing one production field preserves profile tests, notes, tags, unknown fields, and unrelated production records.
- Editing ordinary material fields preserves production settings.
- Cancel leaves state unchanged.
- Persist failure restores the previous profile.
- Delete removes only the chosen nested record.
- Supersede preserves the old record and evidence.
- Making a new record preferred demotes only the conflicting applicability record.
- Duplicate profile creates new nested IDs and downgrades copied verification.

### Multiple applicability contexts

- Two machines remain separate.
- Same machine with two measured sheets remains separate.
- Cut and future engrave records remain separate.
- Supplier/finish differences remain metadata and do not collapse identity.
- Reversed input produces deterministic display ordering.

### Designs selection and application for Phase 7.2

- Selection is retained only for the session.
- Selection alone changes no field.
- Checked measured thickness applies exactly.
- Checked joint clearance applies exactly.
- Drawer lateral clearance applies only to Drawer Cabinet.
- User override after apply is retained.
- Switching profiles does not overwrite without Apply.
- Machine mismatch blocks apply.
- Thickness/scope mismatch warns.
- Kerf and cut settings display but do not alter SVG.
- Preview/download remain identical.
- SVG generation does not change Library or localStorage.
- Backup excludes Design selection.

### Promotion for Phase 7.3

- Cut test promotion copies winner values and source ID.
- Kerf test promotion requires manual kerf confirmation if not represented.
- Coupon promotion requires manual winner selection.
- Project evidence can elevate status only explicitly.
- Source records are not mutated.
- Existing preferred record is not silently overwritten.
- Deleted source leaves readable snapshot evidence.

### Backup/import

- Old backup without production settings loads unchanged.
- New backup round-trips all production/evidence fields.
- Replace import reproduces normalized records.
- Merge import unions tests and production settings by nested ID.
- Matching nested IDs follow documented conflict precedence.
- Distinct nested IDs are never dropped.
- Malformed production settings do not discard the parent profile.
- Unknown future fields survive load/export.
- Storage recovery remains unchanged for corrupt top-level JSON.

### Legacy regressions

- Existing profile creation/edit/delete/duplicate.
- Material Test editing and normalization.
- Test Grid promotion and duplicate detection.
- Library Manage and Browse.
- Material Browser matching.
- Project Wizard classification and exact-machine policy.
- Project snapshots and accounting.
- Designs deterministic signatures.
- All current 1091 fixtures.

## 15. Manual verification plan

1. Open normal `index.html` through `file://`.
2. Confirm no self-tests run and all tabs render.
3. Export a pre-Phase-7 backup.
4. Open an existing Library material with tests and notes.
5. Add one production setting and save.
6. Reopen it and verify every current field/test remains unchanged.
7. Edit only measured thickness; verify IDs and evidence remain stable.
8. Add a second machine record and verify both display independently.
9. Make one preferred; then choose the other and verify explicit demotion behavior.
10. Duplicate the Library material and verify copied settings are not still verified/preferred.
11. Export JSON and inspect the nested structure and signed negative/positive values.
12. Import the old backup with Replace and confirm profiles load with no production settings.
13. Restore the new backup with Replace and confirm exact records return.
14. Create divergent local/imported nested records and test Merge without data loss.
15. Corrupt one nested numeric/status/evidence field in a backup and verify safe normalization.
16. Trigger a simulated storage quota failure if the existing harness supports it and confirm rollback.
17. In Phase 7.2, select a profile in Designs and verify nothing changes until Apply.
18. Apply measured thickness/joint clearance, override one manually, and generate preview/download.
19. Verify cut settings and kerf remain reference text only and SVG bytes do not contain or reflect them.
20. Reload the page and confirm Designs selection/application was session-only.
21. Run `index.html?selftest=all`.
22. Run `git diff --check`, HTML parsing, JavaScript runtime validation, and direct startup.

Physical verification remains necessary after Phase 7.2/7.3. Software fixtures cannot prove kerf sign, fit, glue requirements, batch repeatability, or machine setup.

## 16. Recommended implementation phases

### Phase 7.1 — Proven settings storage and Library UI

- Add constants/enums and pure normalizers.
- Add optional `profile.productionSettings`.
- Preserve unknown profile fields through normal edits.
- Add modal draft state for production settings.
- Add collapsed Library editor section and concise card/detail summary.
- Add manual add/edit/delete/supersede/preferred workflow.
- Add strict optional-number parsing.
- Define duplicate-profile behavior.
- Add Library-specific merge import or an explicit merge warning if nested union is deferred.
- Add normalization, editor, backup, merge, and preservation fixtures.
- Update README storage/model wording and fixture total after verification.

### Phase 7.2 — Explicit Designs consumption

- Add session-only production-setting selection.
- Add applicability/mismatch helper.
- Add per-field checkboxes and explicit Apply.
- Display laser/LightBurn settings as reference only.
- Keep all Designs values overrideable.
- Add no-write-back and unchanged-SVG fixtures.

### Phase 7.3 — Evidence promotion

- Promote eligible cut/kerf Material Tests.
- Save Joint Fit Coupon winner manually.
- Save Drawer Cabinet running-clearance result manually.
- Link Projects as assembled evidence.
- Add explicit verified/superseded transitions.

### Phase 7.4 — Richer machine and material history

Only if real usage requires it:

- top-level machine registry;
- lens/focus presets;
- raw-material lots/sheets;
- supplier/product catalogs;
- richer evidence timelines;
- stale-calibration policies;
- batch comparison and variance reporting.

## 17. Expected files and functions affected

The app should remain single-file/offline.

### `index.html`

Likely new or changed areas:

- top-level production status, machine, scope, and evidence constants;
- `formDefaults('profile')`;
- `normalizeLibraryProfiles()`;
- new `normalizeProductionSettings()` and nested helpers;
- strict optional-number parser;
- `openProfile()` preservation and submit path;
- new `profileDraftProductionSettings`;
- `modalFields('profile')`;
- production-setting editor HTML/binding helpers;
- Library card/detail summary helpers;
- `duplicateProfile()`;
- `convertStoredUnits()` decision boundary;
- `mergeData()` through a Library-specific merge helper;
- merge statistics only if nested details are reported;
- `backupObject()` fixtures, not necessarily implementation;
- new Phase 7 fixture group and `selftest` query gating;
- full-suite invocation.

Phase 7.2 additionally affects:

- `designDraft` session fields;
- `designDefaults()` only for session selection defaults, not production numeric defaults;
- `normalizeDesignDraft()`;
- `renderDesigns()`;
- Design form event binding;
- `buildDesignResult()` only to consume already-applied ordinary values, not profile objects;
- Design fixtures.

Phase 7.3 additionally affects:

- Material Test editor/wizard save paths;
- Test/coupon result capture;
- Project evidence linkage helpers.

### `README.md`

Update only after implementation is verified:

- optional proven production settings;
- evidence/status semantics;
- session-only explicit Designs application;
- kerf remains a LightBurn through-cut-layer setting;
- backup inclusion;
- fixture total.

### `docs/`

Implementation and independent-audit reports may be added. No new runtime files or dependencies are needed.

## 18. Risks and deferred features

### Future sheet-aware layout

Do not place general layout preferences in production settings.

Belongs primarily to Inventory or future stock/layout records:

- blank width and height;
- stock unit;
- sheet quantity;
- grain direction tied to a sheet/product;
- available remnants.

Belongs to future layout/export preferences:

- rotation permission/preference;
- sheet margin;
- part gap;
- nesting strategy;
- layout quantity.

A production setting may snapshot a material condition relevant to testing, but it should not become a stock or nesting record.

### Deferred machine system

A top-level machine registry is not required for Phase 7.1. Stable initial keys plus custom label are enough. Avoid introducing machine CRUD, maintenance logs, lens inventories, and calibration schedules before multiple-machine usage proves the need.

### Deferred operation breadth

Phase 7.1 should focus on cutting and mechanical fit. Engraving/photo settings already have meaningful Material Test and Project Wizard paths, but a generalized proven-production model for every operation would expand the UI and validation surface significantly.

### Staleness

Phase 7.1 can display verified date and scope. Automatic “stale” expiration should be deferred until the user has enough records to establish useful policies. Machine maintenance, lens replacement, and supplier changes may later invalidate evidence, but the first version should warn through context rather than invent arbitrary expiration.

### Repository naming

Material Profiles move the app further from a simple log toward a broader offline laser workshop tool, but Phase 7 should not rename the repository, app title, storage key, or backup format.

A safe later rename boundary is a documentation/branding release after:

- the feature model is stable;
- import/export compatibility is fixture-backed;
- storage key remains unchanged or has an explicit compatibility alias;
- links and GitHub repository migration are planned separately.

Branding can change later without changing persisted data identity.

## 19. Independent-audit recommendation

Require an independent focused audit after Phase 7.1 and before Phase 7.2 consumes the records.

The audit should concentrate on:

- existing-profile edit preservation;
- nested unknown-field preservation;
- merge import behavior;
- duplicate-profile verification downgrade;
- strict numeric parsing for zero and negative values;
- ID stability and deterministic ordering;
- preferred/superseded transitions;
- backup round-trip;
- corrupt nested-record handling;
- no changes to existing Material Test or Project Wizard semantics.

Require a second independent review after Phase 7.2 before physical reliance. It should verify:

- explicit-only application;
- machine/thickness mismatch warnings;
- no SVG kerf application;
- no Library write-back;
- session-only selection;
- no generated-SVG changes when no profile values are applied.

The physical prototype remains the final authority for fit and kerf.

## 20. Explicit conclusion

**READY FOR PHASE 7.1 IMPLEMENTATION**

The existing Library ownership model, nested Material Test precedent, offline backup structure, and fixture infrastructure support a conservative additive implementation. Phase 7.1 should proceed only with the preservation safeguards identified above, especially profile-editor round-tripping, strict optional numeric parsing, nested-ID stability, and safe merge-import behavior.
