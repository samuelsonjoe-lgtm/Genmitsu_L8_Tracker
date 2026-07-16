# Designs Phase 6.2 - Production Joint-Fit Tuning Architecture Review

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Reviewed baseline: `9079183 - Add finger-joint fit coupon generator`  
Review type: architecture and implementation planning only

## Executive summary

Designs Phase 6.2 is ready for a narrow implementation.

The current geometry engine already supports signed joint clearance. Negative clearance changes transition positions within the existing nominal finger pattern; it does not change pattern count, nominal boundaries, edge depth, mating phase, panel envelope, piece identity, layout architecture, or SVG serialization. The production templates reject negative values only because their form constraints and normalization currently require values at least zero.

The recommended implementation is:

- allow joint clearance down to `-0.10 mm` for Finger Box;
- allow cabinet joint clearance down to `-0.10 mm` for Drawer Cabinet;
- allow drawer joint clearance down to `-0.10 mm` for Drawer Cabinet;
- retain the existing positive-clearance domain and dynamic geometry checks for backward compatibility;
- make production feature-web validation symmetric by checking `actualWidth - Math.abs(clearance)`;
- keep every Sliding Lid Box clearance non-negative in this phase;
- change the Drawer Cabinet default total lateral clearance from `0.40 mm` to `0.45 mm`;
- retain the current default joint clearances;
- add explicit interference, kerf, material-specificity, and physical-prototype warnings;
- make no shared geometry, storage, schema, backup, import/export, or SVG serializer changes.

The physical result of `-0.04 mm` joint clearance with a `+0.095 mm` LightBurn red-layer kerf offset is credible evidence for enabling that value as a user-entered production prototype setting. It is not sufficient evidence to make `-0.04 mm` a global default. The tested fit remains specific to the measured 3 mm basswood, machine, focus, cutting parameters, and kerf compensation used for that coupon.

Current verification performed for this review:

| Check | Result |
|---|---|
| Branch and tracking | `main...origin/main` |
| HEAD | `9079183 Add finger-joint fit coupon generator` |
| Tracked working tree | clean before this report |
| Staged files | none |
| `git diff --check` before report | passed |
| Direct `file://` startup | passed in headless Microsoft Edge |
| Designs fixtures | `515 passed / 0 failed` |
| Complete fixture suite | `1047 passed / 0 failed` |

Numerous pre-existing untracked LightBurn files and review documents were present. They were not edited, staged, moved, or deleted.

## 1. Current validation architecture

### 1.1 Session-only draft and dispatch

`designDraft` is initialized from `designDefaults()` and remains outside the persisted `state` object.

The current generation path is:

1. `renderDesigns()` renders the selected template and its session-only fields.
2. `updateDesignDraft()` copies form values into `designDraft`.
3. `normalizeDesignDraft()` validates and converts template fields.
4. `buildDesignResult()` dispatches to the relevant template result builder.
5. The model builder creates panel geometry and diagnostics.
6. The result builder lays out panels and optional score paths.
7. `serializeDesignSvg()` creates the SVG.
8. preview and download use the same generated SVG.

Phase 6.2 can stay within the first four stages plus model validation and warning text. It does not require a new template or a new result-builder branch.

### 1.2 Current defaults

Relevant values in `designDefaults()` are:

| Field | Current default |
|---|---:|
| Finger Box joint clearance | `0.10 mm` |
| Sliding Lid body joint clearance | `0.05 mm` |
| Drawer Cabinet cabinet joint clearance | `0.10 mm` |
| Drawer Cabinet drawer joint clearance | `0.10 mm` |
| Drawer Cabinet total lateral clearance | `0.40 mm` |
| Drawer Cabinet vertical clearance | `0.30 mm` |
| Drawer Cabinet rear clearance | `0.50 mm` |
| Joint Fit Coupon list | `0.06, 0.04, 0.02, 0.00, -0.02, -0.04` |

Only the Drawer Cabinet total lateral default should change in this phase.

### 1.3 Current form constraints

`renderDesigns()` currently renders these production joint fields with `min="0"` and `step="0.01"`:

- Finger Box `jointClearance`;
- Sliding Lid Box `slidingJointClearance`;
- Drawer Cabinet `cabinetJointClearance`;
- Drawer Cabinet `drawerJointClearance`.

The drawer lateral, vertical, and rear running clearances also use `min="0"`.

The Joint Fit Coupon uses a separate fixed-decimal list parser and already accepts signed values from `-0.10` through `+0.30 mm`.

### 1.4 Current normalizer

`normalizeDesignDraft()` sends all production joint and running clearances through:

```javascript
designRequiredNumber(..., 0, true)
```

That means all production values must currently be at least zero.

The Joint Fit Coupon does not use that validation path for its list. `parseJointCouponClearances()`:

- requires signed decimal text with one or two decimal places;
- converts each value to integer hundredths;
- normalizes negative zero;
- rejects duplicates;
- accepts only `-0.10` through `+0.30 mm`;
- requires three to eight values.

That parser is appropriate for a deterministic coupon list, but it should not be directly reused for one production number field. Doing so would unnecessarily introduce list rules, fixed textual formatting, a `+0.30 mm` maximum, and possible compatibility changes for existing positive production input.

### 1.5 Current model validation

`buildBoxModel()` validates every width, depth, and vertical pattern with:

```javascript
pattern.actualWidth - parameters.clearance >= requiredWeb
```

`buildDrawerCabinetModel()` applies the same form to cabinet shell patterns:

```javascript
pattern.actualWidth - parameters.cabinetClearance >= requiredWeb
```

Drawer panels are built through `buildBoxModel()`, so drawer joint validation inherits the Finger Box rule.

`buildSlidingLidBoxModel()` also uses the one-sided form for body patterns.

The shared minimum feature web is:

```javascript
Math.max(0.5, materialThickness * 0.25)
```

At 3 mm material thickness, this is `0.75 mm`.

### 1.6 Current live fixture baseline

The current direct-file browser harness reported:

| Fixture group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material-test normalization | 12 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs geometry | 515 | 0 |
| **Complete suite** | **1047** | **0** |

Normal `file:///C:/Genmitsu%20L8%20Tracker/index.html` reached `document.readyState === "complete"`, rendered the tabs and Designs form, and did not change the storage value during the verification.

## 2. Signed-clearance safety analysis

### 2.1 What clearance changes

`buildFingerPattern()` creates a nominal pattern:

- odd segment count;
- at least three segments;
- common `actualWidth`;
- nominal boundaries from zero to the complete edge length.

`designPatternEdge()` attaches four properties to an edge:

- pattern;
- phase;
- depth;
- clearance.

`designSafePatternEdgePoints()` shifts each internal transition using:

```javascript
nominal + (currentFinger ? -clearance / 2 : clearance / 2)
```

Therefore:

- positive clearance narrows finger regions and widens recess regions;
- zero clearance leaves transitions on nominal boundaries;
- negative clearance widens finger regions and narrows recess regions.

This is the expected geometric behavior for interference tuning.

### 2.2 Mating phases do not change

Negative clearance does not change:

- the `phase` boolean;
- which edge receives which phase;
- nominal pattern boundaries;
- finger depth, which remains the measured material thickness;
- panel dimensions;
- panel IDs;
- panel count;
- edge pairing.

It only reverses the direction of the existing transition displacement.

The two mating phases still share the same nominal pattern and remain complementary. A `-0.04 mm` value moves each complementary transition by `0.02 mm` in the interference direction, producing a `0.04 mm` geometric difference at that transition.

### 2.3 Existing production validation is asymmetric

The current production rule:

```javascript
actualWidth - clearance
```

is safe for positive values because positive clearance narrows one member of each alternating feature pair.

It is not safe as a signed rule. If clearance is negative, subtracting it increases the calculated web:

```text
actualWidth - (-0.04) = actualWidth + 0.04
```

That reports more available web even though the complementary recess has become narrower.

The coupon implementation correctly uses:

```javascript
actualWidth - Math.abs(clearance)
```

Production templates that accept negative joint clearance need the same symmetric safety rule.

### 2.4 Minimum feature-width rule

The existing rule should remain:

```javascript
requiredWeb = Math.max(0.5, materialThickness * 0.25)
```

For each affected finger pattern, require:

```javascript
pattern.actualWidth - Math.abs(clearance) >= requiredWeb
```

This rule is appropriate because the smallest full internal segment under either phase is the nominal segment width minus the magnitude of the signed clearance.

The rule should be applied to:

- Finger Box width pattern;
- Finger Box depth pattern;
- Finger Box vertical pattern;
- Drawer Cabinet cabinet width pattern;
- Drawer Cabinet cabinet depth pattern;
- Drawer Cabinet cabinet height pattern;
- Drawer Cabinet drawer width, depth, and vertical patterns through `buildBoxModel()`.

### 2.5 Terminal segments

A separate terminal-segment rejection rule is not required.

The first and last outside boundaries remain fixed at the panel corners. A terminal feature can lose at most half the clearance magnitude because it has only one shifted internal boundary. Internal features can lose the full clearance magnitude because both adjacent transitions move.

With an odd count of at least three, the symmetric internal rule:

```javascript
actualWidth - Math.abs(clearance) >= requiredWeb
```

is stricter than the terminal requirement:

```javascript
actualWidth - Math.abs(clearance) / 2 >= requiredWeb
```

Terminal behavior should still receive fixtures that confirm:

- the first and last feature widths stay positive;
- transitions remain ordered;
- no transition crosses a corner trim;
- corner stitching remains free of retracing and nonfunctional stubs.

That is regression coverage, not a second production limit.

### 2.6 Shared geometry does not need modification

No change is recommended to:

- `buildFingerPattern()`;
- `designPatternEdge()`;
- `designSafePatternEdgePoints()`;
- `buildFingerPanel()`;
- `buildSlidingLidBodyPanel()`;
- `serializeDesignSvg()`.

The Joint Fit Coupon has already exercised the shared edge composer at negative values, including `-0.10 mm`, without requiring a parallel geometry engine.

The narrow production change belongs in:

- template field constraints;
- production numeric validation;
- model feature-web validation;
- warnings;
- fixtures.

## 3. Recommended template scope

### 3.1 Finger Box: enable

Enable signed joint clearance for Finger Box.

Reasons:

- it uses the same corrected finger-panel geometry as the coupon;
- all its patterned edges represent assembly joints rather than motion gaps;
- the tested coupon was expressly designed to model these 90-degree joints;
- a small open-top Finger Box is the best next physical confirmation.

### 3.2 Drawer Cabinet cabinet joints: enable

Enable signed `cabinetJointClearance`.

The shell uses the same pattern and edge-composition primitives. Negative values should affect only the five shell panels' patterned transitions.

### 3.3 Drawer Cabinet drawer joints: enable

Enable signed `drawerJointClearance`.

Each drawer is generated through `buildBoxModel()` as an open-top five-panel box. The physical joint semantics match Finger Box and the coupon.

### 3.4 Sliding Lid body joints: leave unchanged

The Sliding Lid body finger joints are geometrically capable of using the same signed rule. However, no implementation need justifies broadening Phase 6.2 to them.

Reasons to defer:

- the requested physical evidence came from the coupon and Drawer Cabinet work;
- the Sliding Lid template combines corner joints with a more complex rail and lid system;
- its existing fixtures explicitly require negative body joint clearance to be rejected;
- preserving it avoids changing a third production template before a dedicated physical validation;
- this phase can meet its goal without modifying `buildSlidingLidBoxModel()`.

The Sliding Lid body can be reconsidered later using the same signed validator and absolute web rule after a representative body prototype.

### 3.5 Sliding Lid motion clearances: keep non-negative

Do not allow negative values for:

- lid side clearance;
- lid vertical clearance;
- Front insertion clearance.

These values create free space for motion and insertion. A negative value would intentionally overlap nominal solids or remove a required gap, not tune a complementary glued or friction-fit finger joint.

### 3.6 Drawer running clearances: keep non-negative

Do not allow negative values for:

- total drawer lateral clearance;
- drawer vertical clearance;
- rear clearance.

They control running and insertion space, not finger mating transitions.

## 4. Recommended accepted ranges

### 4.1 Do not copy the coupon range wholesale

The Joint Fit Coupon range of `-0.10` through `+0.30 mm` is a good compact test-list policy. It is not the best production-field policy.

Using `+0.30 mm` as a new hard production maximum would reject positive values that the current production templates can accept when their actual finger width and required web permit them. That would be an avoidable backward-compatibility change.

### 4.2 Recommended production domain

For the three newly signed production fields:

| Field | Recommended domain |
|---|---|
| Finger Box joint clearance | `>= -0.10 mm`, plus dynamic absolute web validation |
| Drawer Cabinet cabinet joint clearance | `>= -0.10 mm`, plus dynamic absolute web validation |
| Drawer Cabinet drawer joint clearance | `>= -0.10 mm`, plus dynamic absolute web validation |

The effective upper limit remains dynamic:

```text
abs(clearance) <= pattern.actualWidth - requiredWeb
```

for every applicable pattern.

This creates:

- a hard, conservative interference lower bound;
- no new rejection of previously valid positive values;
- a geometry-dependent limit in both directions;
- the existing unusually-loose warning for large positive values.

### 4.3 Precision and parsing

Use a focused scalar validator rather than `parseJointCouponClearances()`.

Recommended shape:

```javascript
designJointClearanceNumber(draft, key, label, errors)
```

It should:

- require a value;
- convert with `Number`;
- reject non-finite values;
- reject values below `-0.10 mm`;
- otherwise preserve the current scalar behavior.

The UI should use:

```html
min="-0.10" step="0.01"
```

Do not newly require fixed two-decimal textual syntax for production fields. The existing scalar fields accept ordinary numeric values, and preserving that behavior is safer than imposing the coupon list parser's formatting policy.

### 4.4 Recommended warning thresholds

Keep the current positive warning:

```javascript
clearance > materialThickness * 0.25
```

Add an unconditional warning for any negative joint clearance.

Do not label `-0.04 mm` universally safe or guaranteed. The hard lower bound limits the supported UI domain; the dynamic web rule protects geometry; neither predicts material strength or insertion force.

## 5. Default-value recommendation

### 5.1 Change Drawer Cabinet lateral clearance to 0.45 mm

Change:

```javascript
drawerLateralClearance: '0.40'
```

to:

```javascript
drawerLateralClearance: '0.45'
```

Rationale:

- `0.45 mm` has been physically tested in an actual Drawer Cabinet context;
- the resulting drawer stayed retained when inverted and released with a gentle shake;
- this is a running-clearance default, separate from finger-joint interference;
- Designs state is session-only, so there is no persisted-data migration;
- users can still explicitly enter `0.40 mm`.

The UI and warnings must still describe it as a starting point tied to material and cutting conditions, not a universal fit.

### 5.2 Keep joint defaults unchanged

Retain:

- Finger Box joint clearance: `0.10 mm`;
- Drawer Cabinet cabinet joint clearance: `0.10 mm`;
- Drawer Cabinet drawer joint clearance: `0.10 mm`;
- Sliding Lid body joint clearance: `0.05 mm`.

The `-0.04 mm` coupon result should become an accepted manual value, not a global default, until it has been validated on a complete production joint set.

This is especially important because the observed success also used a `+0.095 mm` red-layer kerf offset. Joint clearance and kerf compensation are separate controls whose combined physical effect depends on the exact material and cutting process.

### 5.3 Future material-specific defaults

If the tracker later offers material-aware Designs presets, the appropriate long-term source is Library material evidence, preferably a recommended Material Test that records:

- material identity;
- measured thickness;
- machine;
- focus;
- cutting settings;
- kerf result;
- tested joint clearance;
- confidence and notes.

That is a future data-model and workflow phase. Phase 6.2 should not add persistence, schema fields, automatic Library lookup, or global material profiles.

## 6. UI and warning wording

### 6.1 Finger Box field help

Recommended help text:

> 0.00 mm is nominal. Positive values add joint relief; negative values create geometric interference. Kerf compensation is separate.

### 6.2 Drawer Cabinet field help

Cabinet joint:

> Adjusts only the five-panel cabinet shell joints. Negative values create geometric interference; kerf compensation is separate.

Drawer joint:

> Adjusts only the removable drawer joints. Negative values create geometric interference; kerf compensation is separate.

### 6.3 Generated warning for negative clearance

Recommended concise warning:

> Negative joint clearance creates geometric interference. Prototype with the same material, measured thickness, focus, cut settings, and kerf compensation. Excessive interference can crush, split, or permanently lock the joint.

For Drawer Cabinet, identify the affected field:

- `Cabinet joint clearance is negative...`
- `Drawer joint clearance is negative...`

If both are negative, either emit two field-specific warnings or one combined warning that names both. Deduplication should not hide which subsystem is affected.

### 6.4 Kerf guidance

Keep the current template-specific kerf guidance. It already states that:

- joint clearance and laser kerf are separate;
- kerf must be established through material testing;
- kerf is not automatically included in generated dimensions.

The implementation should not add a Designs kerf field or automatically apply `+0.095 mm` to generated coordinates. That offset belongs to the LightBurn red cut operation used for the validated material and settings.

### 6.5 Avoid press-fit guarantees

Use:

- `geometric interference`;
- `tested interference setting`;
- `prototype required`;
- `may produce a firm friction fit`.

Avoid:

- `guaranteed press fit`;
- `glue-free joint`;
- `production safe`;
- `universal basswood setting`.

The coupon result proves one successful test condition, not a universal material property.

## 7. Fixture plan

Add focused assertions to `runDesignGeometryFixtures()` without changing the fixture harness.

### 7.1 Scalar validation

1. Finger Box accepts `-0.04`.
2. Finger Box accepts lower boundary `-0.10`.
3. Finger Box rejects `-0.11`.
4. Drawer cabinet joint accepts `-0.04`.
5. Drawer joint accepts `-0.04`.
6. Both Drawer Cabinet joint fields accept `-0.10`.
7. Both reject `-0.11`.
8. Blank, malformed, `NaN`, and `Infinity` remain rejected.
9. Previously accepted positive scalar input remains accepted when dynamic geometry permits it.

Do not reuse the coupon's three-to-eight-list fixtures for scalar fields.

### 7.2 Dynamic web safety

Use independently constructed small-pattern model inputs to prove:

1. positive clearance is evaluated by magnitude;
2. negative clearance of the same magnitude reaches the same minimum web;
3. a signed value that would narrow a finger or recess below `requiredWeb` blocks generation;
4. values exactly on the web boundary are accepted within the existing tolerance;
5. first and last transitions remain ordered and leave positive terminal features.

The expected calculation should be literal in the fixture rather than computed by the same helper under test.

### 7.3 Finger Box `-0.04 mm`

Assert:

- valid result;
- deterministic SVG;
- SVG parser validity;
- five open-top pieces;
- unchanged IDs;
- unchanged dimensions and panel envelopes;
- unchanged edge phases and nominal pattern boundaries;
- changed patterned paths relative to `0.00` and `+0.10`;
- no self-intersections;
- no positive collinear overlap;
- no zero-length or duplicate cut segments;
- no layout overlap;
- interference warning present;
- no draft mutation;
- no localStorage mutation.

Add a direct transition-coordinate fixture modeled after the coupon's literal `+0.04`, `0.00`, and `-0.04` checks.

### 7.4 Drawer Cabinet cabinet joint `-0.04 mm`

Assert:

- valid result;
- cabinet shell paths change;
- drawer paths do not change;
- shelf paths do not change from joint clearance alone;
- dimensions, IDs, rows, piece count, and shelf elevations remain unchanged;
- mating shell phases and shared boundaries remain unchanged;
- no self-intersections, retracing, duplicate cuts, or layout overlap;
- warning identifies cabinet joints.

### 7.5 Drawer Cabinet drawer joint `-0.04 mm`

Assert:

- valid result;
- drawer paths change;
- cabinet shell and support-shelf paths do not change;
- all repeated drawers use the same signed setting while retaining separate IDs;
- dimensions, rows, piece count, and running-clearance metrics remain unchanged;
- no geometry diagnostics;
- warning identifies drawer joints.

### 7.6 Independent cabinet and drawer behavior

Add a four-way comparison:

| Cabinet joint | Drawer joint | Expected changed paths |
|---:|---:|---|
| `0.10` | `0.10` | baseline |
| `-0.04` | `0.10` | cabinet shell only |
| `0.10` | `-0.04` | drawers only |
| `-0.04` | `-0.04` | both groups |

Support shelves should remain independent of both joint-clearance fields.

### 7.7 Continued non-negative behavior

Preserve or split the current broad Drawer Cabinet negative-clearance fixture so it explicitly proves:

- negative lateral clearance rejects;
- negative vertical clearance rejects;
- negative rear clearance rejects.

Preserve Sliding Lid assertions proving:

- negative body joint clearance rejects in Phase 6.2;
- negative lid side clearance rejects;
- negative lid vertical clearance rejects;
- negative Front insertion clearance rejects.

The existing fixture named `Existing templates still reject negative joint clearance` must be replaced with more precise expectations because Finger Box and Drawer Cabinet joint fields will intentionally stop rejecting supported negative values.

### 7.8 Default Drawer Cabinet change

Update default expectations from `0.40` to `0.45 mm`:

- lateral metric: `0.45`;
- per-side metric: `0.225`;
- cabinet inside width: `106.45 mm` for current default dimensions;
- cabinet outside width: `112.45 mm`;
- drawer outside width remains `106 mm`;
- one-row and three-row layout widths remain `368 mm` in the current deterministic layout;
- piece counts and IDs remain unchanged.

Live read-only probing of the current generator with the field manually set to `0.45 mm` produced:

| Output | Proposed default signature |
|---|---|
| One-row full SVG | length `4456`, hash `a6dd23dc` |
| Three-row full SVG | length `9153`, hash `8c286797` |
| Three-row cabinet shell and shelf path signature | length `2315`, hash `4e5dc2cf` |

These values were measured without changing repository source. The implementation should independently reproduce them after changing the default and should fail loudly if its other code changes produce different bytes.

### 7.9 Explicit `0.40 mm` compatibility

Add an explicit old-default draft:

```javascript
{
  ...defaultsFor('drawer-cabinet'),
  drawerLateralClearance: '0.40'
}
```

It should preserve the current committed signatures:

| Output | Current explicit `0.40 mm` signature |
|---|---|
| One-row full SVG | length `4436`, hash `7494c326` |
| Three-row full SVG | length `9124`, hash `b158a794` |
| Three-row cabinet shell and shelf paths | length `2288`, hash `63d818d0` |

This fixture distinguishes an intentional default change from an unintended geometry change.

### 7.10 Unchanged templates and shared output

Preserve exact committed signatures for:

- open-top Finger Box at its unchanged positive default: `2483 / a892f91c`;
- loose-lid Finger Box at its unchanged positive default: `2615 / 6181bc75`;
- Sliding Lid fixture output: `2800 / 4a7ab718`;
- all legacy template baselines;
- Joint Fit Coupon default and signed endpoint outputs.

Changing accepted input range must not alter SVG bytes for existing non-negative Finger Box or Drawer Cabinet drafts with explicit values.

## 8. Storage and compatibility assessment

### 8.1 No persistence changes required

`designDraft` is not in persisted `state`.

`persist()` serializes an explicit field list that excludes all Designs fields.

`backupObject()` excludes `designDraft` and all Designs options.

`replaceData()` and `mergeData()` operate on saved application collections and preferences. They do not load Designs form state.

Therefore Phase 6.2 requires no change to:

- `STORAGE_KEY` (`genmitsu-l8-tracker-v1`);
- `SCHEMA_VERSION` (`2`);
- `freshState()`;
- `loadState()`;
- `persist()`;
- `backupObject()`;
- import replace;
- import merge;
- JSON schema behavior;
- existing localStorage values.

### 8.2 Backward compatibility

Existing explicit positive joint-clearance values should continue to normalize and generate exactly as before.

The recommended production validator preserves:

- finite scalar parsing;
- zero and positive acceptance;
- existing dynamic upper safety;
- existing positive loose-fit warnings.

The only intentional default-output change is Drawer Cabinet lateral clearance changing from `0.40` to `0.45 mm`.

Because Designs is session-only:

- no saved project is rewritten;
- no JSON backup changes;
- no migration is needed;
- no old value is silently normalized;
- current application data remains untouched.

### 8.3 Storage fixtures

Retain and extend existing storage checks:

- Finger Box signed generation does not write localStorage;
- Drawer Cabinet signed generation does not write localStorage;
- changing/resetting the Designs form remains session-only;
- backup keys remain unchanged;
- no production-clearance field appears in `backupObject()`.

## 9. Expected files and signatures affected

### 9.1 Expected implementation files

The production implementation should normally modify:

- `index.html`;
- `README.md` only if its Designs description needs a brief signed-production-clearance clarification;
- one implementation report in `docs/`.

No additional script, module, dependency, asset, schema, or migration file is needed.

### 9.2 Expected `index.html` regions

Narrow changes are expected in:

1. `designDefaults()`
   - change Drawer Cabinet lateral default to `0.45`.

2. `renderDesigns()`
   - set `min="-0.10"` on the three approved joint fields;
   - update field help;
   - leave Sliding Lid and running-clearance minima at zero.

3. validation helper area near `designRequiredNumber()`
   - add one scalar signed-joint validator, or an equivalently bounded small helper.

4. `normalizeDesignDraft()`
   - use the signed helper for Finger Box joint clearance;
   - use it for Drawer Cabinet cabinet and drawer joint clearance;
   - leave Sliding Lid and running clearances unchanged.

5. `buildBoxModel()`
   - use `Math.abs(clearance)` in feature-web validation;
   - add negative-interference warning.

6. `buildDrawerCabinetModel()`
   - use `Math.abs(cabinetClearance)` for shell web validation;
   - add field-specific negative warning.

7. `runDesignGeometryFixtures()`
   - add signed production checks;
   - update Drawer Cabinet default expectations and signatures;
   - preserve explicit `0.40 mm` signatures;
   - revise fixtures that currently expect all production negatives to reject.

No change should be necessary in `buildDesignResult()` beyond its existing dispatch.

### 9.3 Intentionally changed signatures

Changing the Drawer Cabinet lateral default changes default Drawer Cabinet geometry because:

- cabinet inside width increases by `0.05 mm`;
- cabinet outside width increases by `0.05 mm`;
- cabinet width finger `actualWidth` changes;
- cabinet bottom, top, back, and support-shelf width coordinates change;
- deterministic panel placement and serialized numeric text may change;
- default full SVG hashes change.

Drawer panel local paths do not change solely because lateral clearance changes. Their inside dimensions and joint clearance remain the same. Their translated positions or the complete document bytes may still change as part of the deterministic layout.

Expected changed default fixtures:

- Drawer lateral-clearance formula assertion;
- default cabinet inside/outside width values;
- default one-row Drawer Cabinet SVG signature;
- default three-row Drawer Cabinet SVG signature;
- three-row cabinet shell/shelf path signature;
- any fixture literal that embeds `0.40` as the default.

Expected unchanged signatures:

- explicit Drawer Cabinet `0.40 mm` output;
- Finger Box positive-default output;
- Sliding Lid output;
- Joint Fit Coupon output;
- legacy Designs templates.

## 10. Physical validation plan

### 10.1 Smallest useful confirmation

Cut one small open-top Finger Box.

This is preferable to another flat coupon because it validates:

- real 90-degree corner assembly;
- bottom-to-wall joints;
- vertical wall-to-wall joints;
- both mating phases;
- multiple edge orientations;
- corner stitching;
- cumulative assembly force across a complete five-panel object.

It uses less material than a full Drawer Cabinet and provides stronger production evidence than one standalone drawer edge or another representative two-piece corner.

### 10.2 Test conditions

Use:

- the same 3 mm basswood batch if available;
- measured actual thickness recorded before cutting;
- Finger Box joint clearance `-0.04 mm`;
- LightBurn red cut-layer kerf offset `+0.095 mm`;
- the same focus method;
- the same speed, power, passes, air assist, and cut order as the successful coupon;
- assembly labels off unless label behavior is also being checked.

### 10.3 Suggested prototype size

Use a compact box large enough to preserve ordinary finger counts and hand assembly:

- approximately `50 x 40 x 25 mm` inside dimensions;
- preferred finger width around `8-10 mm`;
- open top.

Before cutting, verify the generated patterns remain comfortably above the dynamic required web.

### 10.4 Acceptance observations

Record:

- hand insertion force;
- whether a mallet or clamp was required;
- edge crushing or whitening;
- ply separation or splitting;
- whether all five panels seat fully and squarely;
- whether the box remains assembled without glue;
- whether it can be separated without damage;
- whether char cleanup materially changes fit;
- whether the same fit occurs across grain directions;
- actual material thickness;
- LightBurn kerf offset and complete cut settings.

Stop assembly if insertion begins to crush, split, delaminate, or permanently lock the material.

### 10.5 Drawer follow-up

The existing physical result supports changing total drawer lateral clearance to `0.45 mm`. After the small Finger Box confirms complete-joint behavior, the next production prototype should be one single-row Drawer Cabinet using:

- cabinet joint `-0.04 mm`;
- drawer joint `-0.04 mm`;
- total lateral clearance `0.45 mm`;
- the same `+0.095 mm` red-layer kerf offset and cutting conditions.

That later test would verify the combined shell, drawer, and running-clearance behavior. It is not necessary before implementing the bounded input support.

## 11. Risks and unverified assumptions

### 11.1 Physical fit remains process-specific

The software can validate geometry but cannot predict:

- plywood density;
- glue-layer variation;
- grain direction;
- sheet flatness;
- moisture;
- char thickness;
- focus drift;
- laser power variation;
- mechanical squareness;
- LightBurn offset interpretation;
- assembly technique.

The `-0.04 mm` result is verified only for the reported test conditions.

### 11.2 Kerf and joint clearance are coupled physically, separate digitally

The generator's clearance changes finger transition coordinates. The LightBurn kerf offset changes the toolpath applied to the red cut layer.

They are separate controls, but their physical effects combine. A clearance that works with `+0.095 mm` kerf compensation may not work at zero offset or with a different measured kerf.

Phase 6.2 should preserve this separation and warn clearly.

### 11.3 Hard lower bound is policy, not a universal damage threshold

`-0.10 mm` is a conservative supported input boundary inherited from the coupon test envelope. The dynamic web rule protects geometric feature width, but it does not prove that `-0.10 mm` can be assembled safely in every material.

Users should move from tested values toward tighter interference gradually and stop at signs of damage.

### 11.4 Positive compatibility limits

Retaining the current dynamic positive range avoids breaking existing inputs. Very large positive values may remain mathematically valid yet physically loose. The existing loose-fit warning remains important.

Introducing a new `+0.30 mm` production maximum would simplify the UI but would be a behavioral restriction without a demonstrated need. This review does not recommend it.

### 11.5 Sliding Lid body is intentionally deferred

Its body finger joints share the geometry engine and could technically accept signed values. Deferring that field is a scope and validation decision, not a geometry limitation.

No negative value should be enabled for Sliding Lid motion clearances.

### 11.6 Default Drawer Cabinet signature change is intentional

The default lateral-clearance change modifies default cabinet shell and shelf coordinates. That is expected and should not be mistaken for a regression.

Explicit `0.40 mm` output must remain byte-identical to the current committed generator. If it does not, the implementation has changed more than the default.

### 11.7 No application changes were made in this review

This review did not implement:

- signed production inputs;
- default changes;
- warning changes;
- fixture changes;
- README changes;
- storage or schema changes.

Only this architecture report was created.

## 12. Explicit conclusion

# READY FOR IMPLEMENTATION

The current architecture supports a conservative Phase 6.2 without modifying shared geometry or persistence.

The approved implementation boundary should be:

1. Finger Box, Drawer Cabinet cabinet joints, and Drawer Cabinet drawer joints accept values down to `-0.10 mm`.
2. Every affected pattern uses `actualWidth - Math.abs(clearance) >= Math.max(0.5, thickness * 0.25)`.
3. Sliding Lid body and motion clearances remain unchanged and non-negative.
4. Drawer running clearances remain non-negative.
5. Drawer Cabinet total lateral default changes to `0.45 mm`.
6. Existing joint defaults remain unchanged.
7. Negative values produce explicit interference and physical-prototype warnings.
8. Kerf remains a separate LightBurn/material-test concern.
9. Existing positive and explicit `0.40 mm` outputs remain compatible.
10. No storage, schema, backup, import/export, SVG serializer, or shared geometry change is required.
