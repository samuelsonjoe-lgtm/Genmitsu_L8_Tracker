# Designs SVG Whisker Fix

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `c7c6845` — `Add drawer cabinet design generator`

Investigation: `docs/DESIGNS_SVG_WHISKER_INVESTIGATION_2026-07-16.md`

## Outcome

The verified finger-panel corner-stitching defect is corrected.

Finger Box body panels and Drawer Cabinet Drawer panels now use one terminal-offset-aware join at each corner. Generated paths no longer travel to a raw rectangle corner and then retrace one material thickness.

The correction is limited to the shared rectangular finger-panel composition path. Drawer Cabinet formulas, shell and shelf geometry, Sliding Lid geometry, layout, IDs, clearances, labels, SVG serialization, persistence, and offline behavior remain unchanged.

## Initial repository state

The requested repository checks were run before editing.

- Branch: `main`, tracking `origin/main`
- HEAD: `c7c6845 Add drawer cabinet design generator`
- Tracked working tree: clean
- Staging area: empty
- `git diff --check`: passed
- Existing unrelated untracked files and reports: preserved

The known defective baseline had:

| Output | Panels with overlap | Overlapping runs |
|---|---:|---:|
| Default Finger Box | 5 | 16 |
| Each Drawer | 5 | 16 |
| Three-row cabinet Drawers | 15 | 48 |
| Drawer Cabinet shell | 0 | 0 |
| Drawer Cabinet support shelves | 0 | 0 |
| Sliding Lid Box | 0 | 0 |

## Exact root fix

### Shared corner composition

`buildFingerPanel()` previously concatenated four independent `designEdgePoints()` results at raw rectangle corners. A patterned edge beginning or ending recessed by `edge.depth` could therefore retrace part of an adjacent edge.

`buildFingerPanel()` now delegates its rectangular four-edge composition to the existing `buildSlidingLidBodyPanel()` implementation:

```javascript
function buildFingerPanel(definition) {
  return buildSlidingLidBodyPanel(definition);
}
```

That existing composer:

1. obtains each previous-edge terminal offset with `designEdgeTerminalOffset()`;
2. obtains each current-edge starting offset;
3. combines both offsets at one shared corner join;
4. trims patterned runs through `designSafePatternEdgePoints()`.

No finger-pattern count, boundary, rounding, clearance, phase, dimension, or serialization logic was changed.

### Positive-length collinear-overlap diagnostic

Two pure helpers were added:

- `designCollinearSegmentOverlapLength()`
- `designPositiveCollinearOverlaps()`

They detect positive-length overlap between collinear segments, including:

- exact duplicate traversal;
- partial overlap;
- adjacent immediate reversal;
- non-adjacent overlap across closure.

They do not flag:

- endpoint-only contact;
- parallel separated segments;
- legitimate short segments.

The epsilon is used only for floating-point comparisons. No geometry is deleted by length.

### Validation coverage

`designPanelGeometryErrors()` now reports positive-length collinear overlap.

Full panel validation is applied to:

- all Finger Box panels from `buildBoxModel()`;
- all Drawer Cabinet shell panels;
- all support shelves;
- all namespaced Drawer panels;
- all Sliding Lid physical panels, as before.

## Focused fixtures

Seventeen Designs assertions were added.

Corner composition:

1. patterned edge meeting patterned edge;
2. plain edge meeting a recessed patterned edge;
3. recessed patterned edge meeting a plain edge;
4. two recessed patterned edges;
5. final closure into the first edge.

The expected join is independently calculated from:

- the raw corner;
- the previous edge's terminal offset;
- the current edge's starting offset;
- the two inward vectors.

Overlap diagnostics:

6. adjacent immediate backtracking;
7. non-adjacent closure overlap;
8. partial collinear overlap;
9. exact duplicate traversal;
10. endpoint-only and separated parallel contact remain valid;
11. legitimate short finger geometry remains valid.

Template regressions:

12. all five Finger Box body panels are overlap-free;
13. every Drawer panel is overlap-free for rows 1, 2, and 3;
14. Drawer Cabinet shell and shelves remain overlap-free;
15. Sliding Lid pieces remain overlap-free;
16. cabinet shell/shelf path signature remains unchanged;
17. corrected Drawer Cabinet SVG signatures remain stable.

## Intentionally changed signatures

The old Finger Box and Drawer signatures encoded malformed corner traversal and were intentionally replaced after reviewing corrected coordinates.

| Output | Corrected length | Corrected hash |
|---|---:|---|
| Open-top Finger Box | 2483 | `a892f91c` |
| Loose-lid Finger Box | 2615 | `6181bc75` |
| One-row Drawer Cabinet SVG | 4436 | `7494c326` |
| Three-row Drawer Cabinet SVG | 9124 | `b158a794` |
| One-row Drawer path set | 1364 | `4fc549b3` |
| Three-row Drawer path set | 4094 | `56166d5a` |

These byte changes are confined to corrected Finger Box and Drawer panel corner commands.

## Preserved signatures

The existing fixture-specific Sliding Lid signature remains:

- length: 2800
- hash: `4a7ab718`

The direct UI default Sliding Lid output also remained unchanged in the before/after path comparison.

Drawer Cabinet shell and support-shelf paths remain byte-identical to the investigation baseline:

| Path set | Length | Hash |
|---|---:|---|
| One-row shell | 1723 | `b30f5b10` |
| Three-row shell plus shelves | 2288 | `63d818d0` |

The investigated full Sliding Lid path set also remained byte-identical:

- length: 1728
- hash: `edf8cec5`

## Preserved geometry and behavior

Existing and new fixtures confirm preservation of:

- Finger Box dimensions;
- Drawer and cabinet dimensions;
- finger counts and boundary metadata;
- all mating phases;
- joint-clearance behavior;
- cabinet and Drawer clearance independence;
- piece IDs and ordering;
- piece counts;
- explicit row layout;
- deterministic output;
- score-before-cut ordering;
- optional assembly labels;
- preview/download identity;
- storage nonmutation.

The live deterministic layout positions remain:

### Finger Box

```text
bottom@10,10
front@10,121
back@151,121
left@10,189
right@121,189
```

### One-row Drawer Cabinet

```text
cabinet-bottom@10,10
cabinet-top@137.4,10
cabinet-left@10,104.5
cabinet-right@104.5,104.5
cabinet-back@199,104.5
drawer-r01-bottom@10,158.8
drawer-r01-front@131,158.8
drawer-r01-back@252,158.8
drawer-r01-left@10,249.8
drawer-r01-right@101,249.8
```

Layout uses unchanged panel envelopes and row definitions; the corrected local corner path does not alter placement.

## Live overlap validation

Fresh Microsoft Edge generation was performed directly from:

`file:///C:/Genmitsu%20L8%20Tracker/index.html`

| Generated output | Physical pieces | Panels with positive overlap | Overlapping runs | SVG validation |
|---|---:|---:|---:|---|
| Finger Box | 5 | 0 | 0 | Passed |
| Sliding Lid Box | 8 | 0 | 0 | Passed |
| One-row Drawer Cabinet | 10 | 0 | 0 | Passed |
| Three-row Drawer Cabinet | 22 | 0 | 0 | Passed |

Drawer comparison:

```text
Before: 16 overlapping runs per Drawer
After:   0 overlapping runs per Drawer
```

## Complete fixture totals

| Fixture group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material-test normalization | 12 | 0 |
| Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs | 445 | 0 |
| **Complete suite** | **977** | **0** |

## Other validation

- `git diff --check`: passed
- `index.html` Python HTML parsing: passed
- direct `file://` startup: passed
- tabs and Designs UI visible: passed
- all four requested generated SVGs parsed successfully
- no external dependency added
- README updated only for the verified total of 977 passed / 0 failed

Static diff inspection confirms no changes to:

- storage keys;
- schema;
- persistence;
- localStorage;
- backup/import/export;
- cabinet or Drawer formulas;
- layout helpers;
- SVG serialization;
- finger-pattern rounding;
- labels.

## Intentionally deferred

The redundant explicit return-to-start followed by SVG `Z` remains unchanged. It creates a zero-length close operation but was verified as separate from the material-thickness whiskers. It was not required for this root fix and remains a future bounded cleanup candidate.

## Unverified areas

No new physical plywood cut or LightBurn production import was performed after this software correction. A scrap re-cut remains the final physical confirmation that LightBurn no longer displays or cuts the corner whiskers.

Nothing was staged, committed, or pushed.

READY FOR LIGHTBURN SCRAP RE-CUT
