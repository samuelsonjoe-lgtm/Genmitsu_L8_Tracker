# Designs Phase 6.2 - Production Joint-Fit Tuning Implementation

Date: 2026-07-16  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `9079183 - Add finger-joint fit coupon generator`  
Status: **READY FOR INDEPENDENT REVIEW**

## Summary

Phase 6.2 is implemented within the approved production-only boundary.

The Designs tab now:

- accepts Finger Box joint clearance down to `-0.10 mm`;
- accepts Drawer Cabinet cabinet joint clearance down to `-0.10 mm`;
- accepts Drawer Cabinet drawer joint clearance down to `-0.10 mm`;
- evaluates production feature-web safety using the magnitude of signed clearance;
- warns clearly that negative clearance creates geometric interference and requires matching physical prototype conditions;
- retains non-negative Sliding Lid and drawer running clearances;
- defaults Drawer Cabinet total lateral clearance to `0.45 mm`;
- preserves all existing joint-clearance defaults;
- keeps kerf compensation separate from generated geometry;
- remains session-only and direct-file compatible.

No shared geometry helper, SVG serializer, storage key, schema, persisted state, backup, import, or export behavior was changed.

## Repository state before editing

The required inspection reported:

| Check | Result |
|---|---|
| Branch | `main...origin/main` |
| HEAD | `9079183` |
| Commit | `9079183 Add finger-joint fit coupon generator` |
| Tracked changes | none |
| Staged files | none |
| `git diff --check` | passed |

The working tree contained numerous pre-existing untracked files and reports, including LightBurn work and earlier documentation. They were preserved and were not edited, staged, moved, or deleted.

The Phase 6.2 architecture review was read completely before implementation:

```text
docs\DESIGNS_PRODUCTION_JOINT_FIT_TUNING_ARCHITECTURE_REVIEW_2026-07-16.md
```

## Files changed

Application/documentation changes:

- `index.html`
- `README.md`
- `docs/DESIGNS_PRODUCTION_JOINT_FIT_TUNING_IMPLEMENTATION_2026-07-16.md`

No unrelated tracked file was changed.

Nothing was staged, committed, or pushed.

## Exact validation change

### Scalar production validator

Added:

```javascript
function designJointClearanceNumber(draft, key, label, errors) {
  return designRequiredNumber(draft, key, label, errors, -.1, true);
}
```

The helper:

- requires a value through the existing scalar validator;
- parses through `Number()`;
- rejects non-finite values;
- rejects values below `-0.10 mm`;
- preserves the existing positive scalar domain;
- adds no positive hard maximum;
- does not impose Joint Fit Coupon list syntax or fixed two-decimal text.

It is used only for:

- Finger Box `jointClearance`;
- Drawer Cabinet `cabinetJointClearance`;
- Drawer Cabinet `drawerJointClearance`.

It is not used for:

- Sliding Lid body joint clearance;
- Sliding Lid lid side clearance;
- Sliding Lid lid vertical clearance;
- Sliding Lid Front insertion clearance;
- Drawer Cabinet lateral clearance;
- Drawer Cabinet vertical clearance;
- Drawer Cabinet rear clearance.

### Symmetric feature-web validation

`buildBoxModel()` changed from:

```javascript
pattern.actualWidth - parameters.clearance
```

to:

```javascript
pattern.actualWidth - Math.abs(parameters.clearance)
```

This applies to:

- Finger Box width;
- Finger Box depth;
- Finger Box vertical;
- Drawer width;
- Drawer depth;
- Drawer vertical.

`buildDrawerCabinetModel()` changed cabinet shell safety from:

```javascript
pattern.actualWidth - parameters.cabinetClearance
```

to:

```javascript
pattern.actualWidth - Math.abs(parameters.cabinetClearance)
```

This applies to:

- cabinet width;
- cabinet depth;
- cabinet height.

The required web remains unchanged:

```javascript
Math.max(0.5, materialThickness * 0.25)
```

No terminal-segment production rule was added. Fixtures verify that terminal transitions remain ordered and positive.

## Exact default change

Changed:

```javascript
drawerLateralClearance: '0.40'
```

to:

```javascript
drawerLateralClearance: '0.45'
```

Unchanged defaults:

| Field | Default |
|---|---:|
| Finger Box joint clearance | `0.10 mm` |
| Drawer Cabinet cabinet joint clearance | `0.10 mm` |
| Drawer Cabinet drawer joint clearance | `0.10 mm` |
| Sliding Lid body joint clearance | `0.05 mm` |
| Drawer vertical clearance | `0.30 mm` |
| Drawer rear clearance | `0.50 mm` |

The current default Drawer Cabinet dimensions now include:

- drawer outside width: `106 mm`;
- total lateral clearance: `0.45 mm`;
- per-side lateral clearance: `0.225 mm`;
- cabinet inside width: `106.45 mm`;
- cabinet outside width: `112.45 mm`.

One-row and three-row deterministic layout widths remain `368 mm`.

## UI changes

The following fields now use:

```html
min="-0.10" step="0.01"
```

- Finger Box joint clearance;
- Drawer Cabinet cabinet joint clearance;
- Drawer Cabinet drawer joint clearance.

All Sliding Lid and drawer running-clearance controls remain at `min="0"`.

### Finger Box help

```text
0.00 mm is nominal. Positive values add joint relief; negative values create geometric interference. Kerf compensation is separate.
```

### Cabinet joint help

```text
Adjusts only the five-panel cabinet shell joints. Negative values create geometric interference; kerf compensation is separate.
```

### Drawer joint help

```text
Adjusts only the removable drawer joints. Negative values create geometric interference; kerf compensation is separate.
```

The existing template-specific kerf guidance remains unchanged. No Designs kerf field was added.

## Warnings added

Finger Box negative values emit:

```text
Joint clearance is negative. Negative joint clearance creates geometric interference. Prototype with the same material, measured thickness, focus, cut settings, and kerf compensation. Excessive interference can crush, split, or permanently lock the joint.
```

Drawer Cabinet uses field-specific warnings:

```text
Cabinet joint clearance is negative. Negative joint clearance creates geometric interference. Prototype with the same material, measured thickness, focus, cut settings, and kerf compensation. Excessive interference can crush, split, or permanently lock the joint.
```

```text
Drawer joint clearance is negative. Negative joint clearance creates geometric interference. Prototype with the same material, measured thickness, focus, cut settings, and kerf compensation. Excessive interference can crush, split, or permanently lock the joint.
```

When both Drawer Cabinet fields are negative, both warnings remain visible.

The wording does not claim:

- guaranteed press fit;
- universal safety;
- glue-free production;
- material-independent correctness.

## Shared-helper boundary

The implementation did not modify:

- `buildFingerPattern()`;
- `designPatternEdge()`;
- `designSafePatternEdgePoints()`;
- `buildFingerPanel()`;
- `buildSlidingLidBodyPanel()`;
- `serializeDesignSvg()`.

It also did not modify:

- `STORAGE_KEY`;
- `SCHEMA_VERSION`;
- `state`;
- `freshState()`;
- `loadState()`;
- `persist()`;
- `backupObject()`;
- import merge behavior;
- import replace behavior.

## Fixture coverage

Added 44 focused Designs assertions.

### Signed scalar validation

Verified:

- Finger Box accepts `-0.04`;
- Finger Box accepts `-0.10`;
- Finger Box rejects `-0.11`;
- Finger Box rejects blank, malformed, `NaN`, and `Infinity`;
- existing positive values remain accepted when geometry permits;
- Drawer Cabinet cabinet joint accepts `-0.04`;
- Drawer Cabinet drawer joint accepts `-0.04`;
- both Drawer Cabinet fields accept `-0.10`;
- both reject `-0.11`;
- both reject blank, malformed, `NaN`, and `Infinity`;
- positive Drawer Cabinet joint values remain accepted.

### Symmetric web safety

An independent literal model fixture uses:

- 3 mm material;
- 18 mm pattern lengths;
- three 6 mm segments;
- required web `0.75 mm`;
- exact boundary magnitude `5.25 mm`.

It verifies:

```text
6.00 - abs(5.25) = 0.75
```

Both `+5.25` and `-5.25` are accepted at the exact boundary.

Both `+5.250001` and `-5.250001` are rejected.

This proves positive and negative feature-web validation is symmetric rather than relying only on production helper output.

### Literal Finger Box transition coordinates

A 45 mm outside edge with five 9 mm nominal segments and `-0.04 mm` clearance is checked against literal transitions:

```text
9.02, 17.98, 27.02, 35.98
```

The fixture also verifies:

- ordered transitions;
- positive first terminal feature;
- positive last terminal feature.

### Signed Finger Box geometry

Verified:

- valid output;
- deterministic SVG;
- five pieces;
- stable IDs;
- stable dimensions;
- stable panel envelopes;
- stable phases;
- stable nominal boundaries;
- stable layout;
- paths differ from zero and positive settings;
- no zero-length segments;
- no duplicate local cuts;
- no self-intersections;
- no positive collinear overlap;
- no panel overlap;
- valid SVG;
- warning content;
- draft nonmutation;
- localStorage nonmutation.

### Drawer Cabinet isolation

Four cases are compared:

| Cabinet joint | Drawer joint |
|---:|---:|
| `0.10` | `0.10` |
| `-0.04` | `0.10` |
| `0.10` | `-0.04` |
| `-0.04` | `-0.04` |

Verified:

- cabinet-only negative changes shell paths only;
- drawer-only negative changes drawer paths only;
- both negative change both path groups;
- support-shelf paths remain unchanged;
- dimensions remain unchanged;
- IDs remain unchanged;
- three-row piece count remains 22;
- rows and shelf elevations remain unchanged;
- shell phases and nominal boundaries remain unchanged;
- every repeated drawer receives the signed drawer setting;
- separate drawer IDs remain stable;
- lateral, vertical, and rear running clearances remain unchanged;
- SVG remains valid;
- no self-intersections;
- no positive collinear overlap;
- warnings identify the correct affected fields;
- draft and localStorage remain unchanged.

### Continued non-negative behavior

Verified rejection of:

- negative drawer lateral clearance;
- negative drawer vertical clearance;
- negative drawer rear clearance;
- negative Sliding Lid body joint clearance;
- negative Sliding Lid side clearance;
- negative Sliding Lid vertical clearance;
- negative Sliding Lid Front insertion clearance.

### Storage and schema isolation

Verified:

- signed Finger Box generation does not write localStorage;
- signed Drawer Cabinet generation does not write localStorage;
- signed drafts are not mutated;
- production Designs fields remain absent from `backupObject()`;
- storage key remains `genmitsu-l8-tracker-v1`;
- schema remains `2`.

## Intentionally changed signatures

The new `0.45 mm` Drawer Cabinet default independently produced:

| Output | Length | Hash |
|---|---:|---|
| One-row full SVG | 4456 | `a6dd23dc` |
| Three-row full SVG | 9153 | `8c286797` |
| Three-row cabinet shell and shelf path signature | 2315 | `4e5dc2cf` |

These values match the architecture review's measured projections.

## Preserved compatibility signatures

An explicit Drawer Cabinet `drawerLateralClearance: '0.40'` preserves:

| Output | Length | Hash |
|---|---:|---|
| One-row full SVG | 4436 | `7494c326` |
| Three-row full SVG | 9124 | `b158a794` |
| Three-row cabinet shell and shelf path signature | 2288 | `63d818d0` |

Other preserved signatures:

| Output | Length | Hash |
|---|---:|---|
| Open-top Finger Box positive default | 2483 | `a892f91c` |
| Loose-lid Finger Box positive default | 2615 | `6181bc75` |
| Sliding Lid fixture output | 2800 | `4a7ab718` |
| QR stand | 359 | `fe737a09` |
| Hanging sign | 341 | `656e633d` |
| Dice tray | 1726 | `51a55721` |
| Divider tray | 1965 | `a55dda6e` |

Joint Fit Coupon default and signed endpoint fixtures also remain green.

## Validation results

### `git diff --check`

Passed.

### HTML parsing

Command:

```powershell
python -m html.parser index.html
```

Result: passed.

### Direct local-file startup

Verified with headless Microsoft Edge:

```text
file:///C:/Genmitsu%20L8%20Tracker/index.html
```

Results:

- `document.readyState`: complete;
- tabs visible;
- Designs form visible;
- direct `file://` operation: confirmed;
- storage unchanged.

### Complete built-in fixture suite

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
| Designs geometry | 559 | 0 |
| **Complete suite** | **1091** | **0** |

### Live signed Finger Box

Input:

```text
Joint clearance: -0.04 mm
```

Result:

- valid: yes;
- SVG parser valid: yes;
- pieces: 5;
- warning visible: yes;
- storage changed: no.

The signed output remains 2483 bytes but has hash `21d08a84`, confirming that the same panel/document structure is retained while finger transitions intentionally change.

### Live Drawer Cabinet isolation

Three-row live generation:

| Case | Valid | Shell changed | Drawers changed |
|---|---|---|---|
| Cabinet `-0.04`, drawer `0.10` | yes | yes | no |
| Cabinet `0.10`, drawer `-0.04` | yes | no | yes |
| Cabinet `-0.04`, drawer `-0.04` | yes | yes | yes |

Field-specific warnings appeared in the correct cases.

### Live Sliding Lid negative checks

All remain invalid:

- `slidingJointClearance: -0.01`;
- `lidSideClearance: -0.01`;
- `lidVerticalClearance: -0.01`;
- `frontInsertionClearance: -0.01`.

Each displayed its existing at-least-zero validation error.

### External dependency scan

No external script, stylesheet, font, fetch, XHR, WebSocket, or dynamic import was added.

The only matched URL-like text remains embedded SVG namespace output, not a network dependency.

### Persistence scan

No diff occurred in:

- storage constants;
- `persist()`;
- `backupObject()`;
- replace import;
- merge import.

## README changes

The Designs description now states:

- Finger Box and Drawer Cabinet joint fields accept small negative interference values down to `-0.10 mm`;
- negative values require matching physical prototyping;
- negative values do not guarantee a press fit;
- kerf remains separate;
- Drawer Cabinet total lateral clearance defaults to `0.45 mm`.

Built-in fixture totals were updated to:

```text
1091 passed / 0 failed
559 Designs geometry checks
```

## Final diff review

Confirmed:

- no unrelated tracked file changed;
- no shared geometry helper changed;
- no SVG serialization changed;
- no storage key or schema changed;
- no state or persistence behavior changed;
- no backup/import/export behavior changed;
- no external dependency was added;
- no files were staged;
- no commit was created;
- nothing was pushed.

## Unverified physical assumptions

No plywood was cut during this implementation.

Software validation cannot establish:

- insertion force;
- plywood crushing or delamination;
- actual glue-free retention;
- effect of grain direction;
- effect of char cleanup;
- whether a complete five-panel assembly remains square;
- whether `-0.04 mm` remains correct with a different sheet, focus, power, speed, passes, or kerf offset.

The recommended next physical confirmation remains one small open-top Finger Box using:

- the same 3 mm basswood;
- `-0.04 mm` joint clearance;
- `+0.095 mm` LightBurn kerf offset on the red cut layer;
- the same focus and cutting settings as the successful Joint Fit Coupon.

An independent code review should occur before spending material on that prototype.

# READY FOR INDEPENDENT REVIEW
