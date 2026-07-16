# Designs SVG Whisker Investigation

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Reviewed baseline: `c7c6845` — `Add drawer cabinet design generator`

Mode: read-only investigation; this report is the only repository file created

## Executive conclusion

The Drawer Cabinet cut whiskers are real SVG geometry defects. They originate before layout and SVG serialization in the shared legacy finger-panel corner composition path:

- `designEdgePoints()`
- `buildFingerPanel()`
- `buildBoxModel()`

When one edge reaches a raw panel corner and the next patterned edge begins recessed by one material thickness, `buildFingerPanel()` concatenates the two independently generated edge paths through the raw corner. The result contains a collinear segment that is immediately or eventually retraced. At 3 mm material thickness, each artifact is exactly 3 mm long. A separate 2.85 mm probe produced exactly 2.85 mm retraces, proving that these are material-thickness corner excursions rather than floating-point noise.

The Drawer Cabinet shell does not use the defective composer. Its five shell panels use `buildSlidingLidBodyPanel()`, which calculates a single corner join from both adjacent terminal offsets. Those shell paths contain no overlapping retraces. Support shelves and Sliding Lid Box pieces also contain no such overlap.

The defect affects:

- all five Drawer panels in every row;
- all five Finger Box body panels;
- no Drawer Cabinet shell panels;
- no Drawer Cabinet support shelves;
- no inspected Sliding Lid Box panels, lid, or rails.

The narrowest root-cause correction is to make `buildFingerPanel()` use the already-existing terminal-offset-aware corner stitching employed by `buildSlidingLidBodyPanel()`. Do not delete arbitrary short segments. The malformed segments are one full material thickness long, while legitimate finger geometry may also be short.

## Repository state

The requested inspection commands were run before the investigation.

- Branch: `main`, tracking `origin/main`
- HEAD: `c7c6845 Add drawer cabinet design generator`
- Tracked working tree: clean
- Staging area: empty
- `git diff --check`: passed
- Existing unrelated untracked files and reports: preserved
- No reset, clean, stash, staging, commit, push, move, or deletion was performed

No exported Drawer Cabinet `.svg` was present in the repository or under `LightBurn Projects`. The investigation therefore reproduced the issue from the current deterministic generator using direct `file://` execution in Microsoft Edge. The exact physically cut draft values were not available, but the defect reproduces with the current 3 mm defaults and with a separate 2.85 mm measured-thickness case.

The current complete built-in suite still reports:

| Fixture group | Passed | Failed |
|---|---:|---:|
| Designs | 428 | 0 |
| Complete suite | 960 | 0 |

The green suite does not disprove the defect; it currently lacks partial collinear-overlap and corner-stitch validation.

## Geometry path trace

### Finger-pattern generation

`buildFingerPattern()` calculates an odd count and stable rounded boundaries. Its output is not the cause. The observed retrace length is the material thickness, not the finger width or boundary-rounding error.

`designEdgePoints()` generates each edge independently from the raw rectangle corner:

1. start at the raw corner;
2. if the first segment is recessed, move inward by `edge.depth`;
3. traverse the pattern;
4. if the final segment is recessed, return from the recessed endpoint to the raw corner.

That edge-local behavior is not sufficient to compose two adjacent edges. The terminal state of both edges must be considered together.

### Legacy panel composition

`buildFingerPanel()` initializes the path at the raw top-left corner and appends each independent edge with:

```javascript
designEdgePoints(...).slice(1)
```

The first point of every new edge is discarded, but the previous edge has already reached the raw corner. If the new edge begins recessed, its next point travels backward over part of the preceding boundary. At closure, the same overlap can occur between the last edge and the first edge even when it is not adjacent in the serialized command list.

### Safe panel composition already present

`buildSlidingLidBodyPanel()` avoids this failure:

1. `designEdgeTerminalOffset()` calculates each edge's offset at the corner.
2. The join point combines the previous edge's terminal offset and the current edge's starting offset.
3. `designSafePatternEdgePoints()` trims each patterned edge to the shared join.
4. The path never visits the raw corner when the actual joined geometry belongs one material thickness inward.

The Drawer Cabinet shell and Sliding Lid body use this safe composer. Their raw SVG paths do not contain the retraces found in Drawer and Finger Box paths.

## Three representative corner traces

All coordinates below are local panel coordinates from the generated SVG before layout translation.

### 1. Patterned edge meeting patterned edge

Panel: `drawer-r01-bottom`

Top-right excerpt:

```text
... H 94.272222 V 0 H 106 H 103 V 10.907143 ...
```

Relevant segments:

```text
(94.272222, 0) -> (106, 0)
(106, 0)       -> (103, 0)
```

The second segment retraces 3 mm of the first segment. The top patterned edge reaches the raw corner `(106, 0)`, then the right patterned edge begins recessed at `(103, 0)`.

The correct terminal-offset join for these phases is `(103, 0)`. The top edge should stop there; the path should not travel to `(106, 0)` and back.

The same panel has four 3 mm overlapping corner runs:

- top-left, across path closure;
- top-right;
- bottom-right;
- bottom-left.

### 2. Plain edge meeting patterned edge

Panel: `drawer-r01-left`

Top-right opening:

```text
M 0 0 H 76 H 73 V 11.05 ...
```

Relevant segments:

```text
(0, 0)  -> (76, 0)
(76, 0) -> (73, 0)
```

The plain top edge reaches the raw corner, then the patterned right edge immediately retraces 3 mm. This proves the problem is not limited to patterned-patterned corners. It occurs whenever independent edge composition ignores the next edge's starting terminal offset.

### 3. Final path closure

Panel: `drawer-r01-bottom`

Path opening:

```text
M 0 0 H 11.727778 ...
```

Path ending:

```text
... H 3 V 0 H 0 Z
```

The final explicit segment `(3, 0) -> (0, 0)` overlaps the first segment `(0, 0) -> (11.727778, 0)` by 3 mm. This overlap is non-adjacent in the command stream, so an adjacent-backtracking test alone will miss it.

The explicit `H 0` already returns to the path start. The following `Z` therefore contributes a redundant zero-length close operation. That redundant close is shared more broadly and is not the cause of the visible 3 mm whisker, but it should be covered by a separate serialization-quality fixture.

## Raw-coordinate diagnostics

### Default 3 mm, three-row Drawer Cabinet

Generated physical pieces: 22

| Piece family | Pieces | Pieces with collinear overlap | Overlapping runs |
|---|---:|---:|---:|
| Cabinet shell | 5 | 0 | 0 |
| Support shelves | 2 | 0 | 0 |
| Drawer bottoms | 3 | 3 | 12 |
| Drawer fronts/backs | 6 | 6 | 12 |
| Drawer left/right sides | 6 | 6 | 24 |
| **Total** | **22** | **15** | **48** |

Every overlap is exactly 3 mm.

Per Drawer:

| Panel | Overlapping runs |
|---|---:|
| Bottom | 4 |
| Front | 2 |
| Back | 2 |
| Left | 4 |
| Right | 4 |
| **Total per Drawer** | **16** |

### Measured 2.85 mm probe

A one-row Drawer Cabinet generated with:

- material thickness: 2.85 mm;
- drawer joint clearance: 0.15 mm.

Result:

- shell overlap count: 0;
- Drawer overlap count: 16;
- every overlap length: 2.85 mm.

The overlap follows `edge.depth` / material thickness exactly. It does not follow joint clearance and is not a near-zero rounding artifact.

### Zero-length segments

Every inspected serialized path has one zero-length close operation because the point list explicitly returns to its start and `designPointsPath()` also appends `Z`.

Observed:

- three-row Drawer Cabinet: 22 zero-length closes across 22 pieces;
- Finger Box: 5 across 5 pieces;
- Sliding Lid Box: 8 across 8 pieces.

This is shared serialization redundancy, not the source of the physical Drawer whiskers. `designSvgPathSegments()` filters zero-length segments from its returned diagnostics, so current tests cannot see them.

### Very short nonzero segments

No nonzero segment under 1 mm was found in the inspected defaults. The shortest nonzero cuts were:

- 3 mm in the default Drawer Cabinet;
- 2.85 mm in the measured-thickness probe;
- 3 mm in the explicit 3 mm Sliding Lid comparison.

Therefore:

- the defect is not a sub-millimeter floating-point stub;
- deleting all short segments would be unsafe;
- the correct detector is positive-length collinear overlap/backtracking, not a broad length threshold.

### Duplicate points and closure

The SVG parser sees the start coordinate three times:

1. initial `M`;
2. explicit final point;
3. implicit endpoint from `Z`.

No unmatched start/end coordinate was found. Paths are syntactically closed and orthogonal, which explains why ordinary SVG parsing passes despite malformed traversal.

### Corner overshoot and repeated endpoints

The affected paths overshoot from the intended terminal-offset join to the raw rectangle corner, then return by one material thickness. The repeated raw corner is not an accidental numeric duplicate; it is produced by the independent edge contract.

The SVG contains no disconnected path subparts. The “dangling” appearance is caused by overlapping/retraced connected geometry, which LightBurn can expose as whiskers and which Cut Shapes can remove manually.

## Template impact matrix

| Template / piece family | Composer | Collinear retrace found | Assessment |
|---|---|---:|---|
| Finger Box body panels | `buildFingerPanel()` | Yes | Shared defect |
| Finger Box loose lid | plain rectangle through `buildFingerPanel()` | No corner retrace expected; redundant close remains | Mostly unaffected |
| Sliding Lid body panels | `buildSlidingLidBodyPanel()` | No | Verified safe in inspected defaults |
| Sliding Lid side panels | `buildSlidingLidBodyPanel()` via `buildSlidingLidSidePanel()` | No | Verified safe |
| Sliding Lid lid and rails | `designPanelFromPoints()` | No | Verified safe |
| Drawer Cabinet shell | `buildSlidingLidBodyPanel()` | No | Verified safe |
| Drawer Cabinet drawers | `buildBoxModel()` -> `buildFingerPanel()` | Yes | Physical incident source |
| Drawer Cabinet support shelves | `designPanelFromPoints()` | No | Verified safe |

## Findings

## Blocker

### B1 — Shared finger-panel corner stitching emits overlapping cut runs

- **Affected file/function:** `index.html`; `designEdgePoints()`, `buildFingerPanel()`, `buildBoxModel()`, and the Drawer creation inside `buildDrawerCabinetModel()`.
- **Exact cause:** independently generated edges are joined at raw rectangle corners. A recessed next edge moves backward by `edge.depth`, overlapping the preceding edge. Closure can overlap the first segment non-adjacently.
- **Scope:** shared by Finger Box and Drawer Cabinet Drawer panels. Not present in Drawer Cabinet shell, support shelves, or inspected Sliding Lid geometry.
- **Production consequence:** LightBurn receives overlapping cut geometry. The laser can double-cut or dwell on the overlap, leave whisker-like corner runs, increase charring, weaken corners, or require manual Cut Shapes cleanup on every export.
- **Recommended fix:** make `buildFingerPanel()` use terminal-offset-aware corner joins. The smallest existing-code route is to delegate its four-edge panel composition to the proven `buildSlidingLidBodyPanel()` logic, or move that join calculation into `buildFingerPanel()` without changing pattern generation.
- **Fixture coverage needed:** positive-length collinear-overlap detection across all panel cut outlines; explicit patterned-patterned, plain-patterned, and closure corner fixtures; full Finger Box and Drawer Cabinet panel checks.
- **Risk of changing shared helpers:** medium. Finger Box and all Drawer panel SVG bytes will intentionally change, so legacy hashes must be reviewed and updated. Panel dimensions, edge patterns, phases, IDs, layout, and clearances should remain unchanged. Sliding Lid output should remain byte-identical.

## Major

### M1 — Current geometry validation cannot detect partial overlapping traversal

- **Affected file/function:** `index.html`; `designPointSegments()`, `designPathSelfIntersects()`, `designPanelGeometryErrors()`, Drawer validation in `buildDrawerCabinetModel()`, and current Designs fixtures.
- **Exact cause:** `designPathSelfIntersects()` ignores adjacent segments, which is normally appropriate but hides immediate reversals. Duplicate detection compares whole normalized segment keys, so a 3 mm segment overlapping part of a longer segment is not an exact duplicate. Closure overlap can be non-adjacent but collinear rather than a crossing. Drawer panels are not passed through `designPanelGeometryErrors()` in `buildDrawerCabinetModel()`; only cabinet shell pieces and shelves are.
- **Scope:** validation gap is shared. The actual malformed geometry currently comes from `buildFingerPanel()`.
- **Production consequence:** syntactically valid and deterministic SVGs pass all 960 fixtures while still containing machine-visible overlapping cuts.
- **Recommended fix:** add a pure collinear-positive-overlap diagnostic and apply it to every physical panel outline. Also apply shared panel geometry validation to Drawer panels after the root-cause stitch correction.
- **Fixture coverage needed:** adjacent reversal, non-adjacent closure overlap, partial overlap, exact duplicate, and a legitimate short finger segment that must remain valid.
- **Risk of changing shared helpers:** low if added as validation only after corrected expected geometry; medium if enabled before correcting legacy Finger Box because it will correctly reject current output.

## Minor

### N1 — Serialized paths redundantly close an already closed point list

- **Affected file/function:** `index.html`; `designPointsPath()` and point-producing panel helpers.
- **Exact cause:** panel point lists include the start point as their final explicit point, then `designPointsPath()` appends `Z`. The serialized close is zero-length.
- **Scope:** shared across inspected Finger Box, Drawer Cabinet, and Sliding Lid paths.
- **Production consequence:** likely harmless in common SVG importers and not responsible for the observed 2.85–3 mm whiskers, but it adds redundant geometry and masks zero-length checks because `designSvgPathSegments()` filters it.
- **Recommended fix:** handle separately from B1. Either omit the duplicated final start point before serialization or avoid adding a redundant close operation while preserving one valid closed subpath.
- **Fixture coverage needed:** raw command-level zero-length detection, exactly one effective closure, and unchanged visible geometry.
- **Risk of changing shared helpers:** medium because it changes SVG bytes for all modern Designs templates even though visible geometry should remain identical. Do not bundle it into the minimum whisker correction unless importer testing justifies it.

## Verified

### V1 — Finger-pattern calculation and clearance rounding are not the cause

- **Affected file/function:** `buildFingerPattern()`, `designRound()`, `designNumberText()`.
- **Evidence:** overlap length follows material thickness exactly at 3 mm and 2.85 mm. It does not follow preferred finger width or clearance.
- **Scope:** verified across default and measured-thickness Drawer probes.
- **Production consequence:** no change to finger counts, boundaries, or numeric precision is needed for the root fix.
- **Recommended action:** preserve these helpers.
- **Fixture coverage needed:** retain current dimension/pattern golden values.
- **Risk of changing shared helpers:** high and unnecessary.

### V2 — SVG serialization faithfully preserves the already malformed panel path

- **Affected file/function:** `serializeDesignSvg()`.
- **Evidence:** the retracing commands are present in each panel's `path` before panel translation. Serialization only writes that path under a translated group.
- **Scope:** shared serializer is not at fault.
- **Production consequence:** changing XML formatting, transforms, or numeric output will not fix the whiskers.
- **Recommended action:** leave serialization unchanged for B1.
- **Fixture coverage needed:** preview/download identity and pre-/post-serialization segment equivalence.
- **Risk of changing shared helpers:** high and unnecessary.

### V3 — Drawer Cabinet shell and support geometry do not contain the defect

- **Affected file/function:** `buildDrawerCabinetModel()` shell calls to `buildSlidingLidBodyPanel()` and shelf calls to `designPanelFromPoints()`.
- **Evidence:** zero positive-length collinear overlaps in five shell panels and both three-row support shelves.
- **Scope:** Drawer Cabinet shell/shelves only.
- **Production consequence:** the cabinet-specific formulas, phases, shell topology, IDs, shelf dimensions, and layout do not need to change.
- **Recommended action:** preserve cabinet geometry and isolate the correction to shared box-panel corner stitching.
- **Fixture coverage needed:** shell and shelf paths remain overlap-free and byte-stable if possible.
- **Risk of changing shared helpers:** low if the fix is limited to `buildFingerPanel()`.

### V4 — Sliding Lid geometry uses the correct corner-stitching model

- **Affected file/function:** `buildSlidingLidBodyPanel()`, `buildSlidingLidSidePanel()`, `designPanelFromPoints()`.
- **Evidence:** zero positive-length collinear overlaps in all eight inspected Sliding Lid pieces.
- **Scope:** Sliding Lid Box.
- **Production consequence:** no Sliding Lid geometry correction is indicated.
- **Recommended action:** reuse its terminal-offset join approach; do not alter its existing formulas or paths.
- **Fixture coverage needed:** preserve the current Sliding Lid SVG signature and assert no overlap.
- **Risk of changing shared helpers:** low if Sliding Lid code is only reused, not modified.

## Narrowest recommended correction

Do not add a generic post-serialization cleanup and do not delete segments by length.

Recommended implementation boundary:

1. Preserve `buildFingerPattern()`, `designPatternEdge()`, dimensions, phases, clearances, IDs, layout, labels, and SVG serialization.
2. Change only the four-edge composition performed by `buildFingerPanel()`.
3. Use the existing terminal-offset join calculation from `buildSlidingLidBodyPanel()` for ordinary rectangular finger panels.
4. Validate every resulting Finger Box and Drawer panel for:
   - closure;
   - orthogonality;
   - finite coordinates;
   - no zero-length explicit segments;
   - no positive-length collinear overlap;
   - no self-intersection;
   - no duplicate complete segments;
   - bounds.
5. Keep the redundant `Z` cleanup as a separate optional correction unless LightBurn testing identifies it as independently harmful.

This correction will intentionally change Finger Box and Drawer panel path bytes. It should not change:

- requested or calculated dimensions;
- finger counts or boundary metadata;
- mating phases;
- physical piece counts;
- piece IDs;
- deterministic layout positions;
- Drawer Cabinet shell/shelf paths;
- Sliding Lid paths;
- preview/download identity;
- storage or backup behavior.

## Recommended focused regression fixtures

### Corner stitching

Add pure fixtures for corner joins rather than relying only on full SVG hashes:

1. patterned `phase:true` meeting patterned `phase:false`;
2. patterned `phase:false` meeting patterned `phase:true`;
3. plain meeting recessed patterned edge;
4. recessed patterned edge meeting plain;
5. two recessed patterned edges;
6. final edge closure into the first edge.

For each corner, calculate the expected join:

```text
raw corner
+ previous-edge inward vector * previous terminal offset
+ current-edge inward vector * current starting offset
```

Assert that the path contains the expected join and does not visit/retrace the raw corner when either terminal offset is nonzero.

### Path traversal

Add a helper that detects positive-length overlap between any two collinear segments. It must detect:

- exact duplicates;
- partial overlap;
- adjacent immediate reversal;
- non-adjacent closure overlap.

Do not use a blanket “segment shorter than N millimeters” rule.

Add a separate near-zero fixture with a numerical epsilon, such as `0 < length <= 1e-6`, to catch rounding residue without rejecting legitimate short fingers.

### Dangling endpoints

For each cut-panel path:

- require one `M` and one effective `Z`;
- require a closed endpoint;
- require no open subpath;
- after overlap normalization, require no degree-one endpoint in the cut outline.

### Zero-length commands

Inspect raw path commands before filtering. Current `designSvgPathSegments()` removes zero-length segments and therefore cannot support this assertion as written.

### Template regression scope

After the correction:

- all Finger Box body panels: no collinear overlap/backtracking;
- every Drawer panel for rows 1, 2, and 3: no overlap/backtracking;
- Drawer Cabinet shell and shelves: remain overlap-free;
- Sliding Lid Box: remain overlap-free and preserve its existing SVG signature;
- piece counts, IDs, dimensions, patterns, phases, and layouts: unchanged;
- preview and download: identical;
- SVG: valid under direct `file://` operation.

Old Finger Box and Drawer hashes should not be blindly retained because they encode the defect. Replace them only after reviewing the expected corrected corner coordinates. Preserve unaffected Sliding Lid and cabinet-shell signatures.

## Final assessment

The physical report is consistent with a deterministic shared geometry bug, not a LightBurn-only artifact. The generated Drawer paths contain material-thickness-length overlapping corner traversals. Manual Cut Shapes cleanup removes the symptom because it removes the redundant overlap, but the generator must own the correction.

The safe cabinet-shell and Sliding Lid composer already demonstrates the correct architectural solution. A bounded correction to `buildFingerPanel()` plus overlap-aware fixtures is sufficient; no cabinet formula, layout, serialization, storage, or broad geometry refactor is warranted.

No fix was implemented.

