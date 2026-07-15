# Designs Phase 2 Finger-Box Implementation

Date: 2026-07-14  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `6a27579` (`Add inventory-guided project wizard`)

## Baseline Verification

Before editing:

- `HEAD`, `main`, `origin/main`, and `origin/HEAD` were synchronized at `6a27579`.
- The tracked working tree was clean.
- Existing untracked architecture/review documents, `LightBurn Projects/`, and `parametric_qr_stand_generator.py` were present and intentionally preserved.
- Nothing was staged.
- `git diff --check`, `git diff --stat`, and `git diff` were empty.
- The established fixture baseline was 532 passed / 0 failed.

The retained `Genmitsu L8 Tracker.dc.html` reference artifact was not modified.

## Initial Legacy SVG Capture

Before the Designs code was refactored, the four default legacy SVG outputs were captured in memory and fingerprinted:

| Template | Bytes/characters | SHA-256 |
|---|---:|---|
| `qr-stand` | 359 | `34b50585d4c427b2d9d5a1107432a0e9fbc43f4f74edb4fa7383375f6fa4e499` |
| `hanging-sign` | 341 | `38300ab2033bb12023b76bf3c78173eaed632da38ca689df1ee4fd50c46e3c3c` |
| `dice-tray` | 1726 | `2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf` |
| `divider-tray` | 1965 | `91a5fcaaa54a63d9c5f9b3e9851de70b62fdd98798d356d50977ed4f46f1d924` |

No temporary capture file was added to the repository.

## Changed Files

- `index.html`
  - shared Designs result/validation/layout/SVG pipeline;
  - exact inline preview and focused preview refresh;
  - `finger-box` geometry and UI;
  - Designs fixture group and self-test dispatch.
- `README.md`
  - bounded feature description, limitations, session-only statement, fixture query/function, and verified total.
- `docs/BOX_GENERATOR_PHASE2_IMPLEMENTATION_2026-07-14.md`
  - this requested untracked implementation report.

No other repository file was edited.

Final tracked diff summary before this untracked report:

```text
README.md  |   8 +-
index.html | 457 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++---
2 files changed, 444 insertions(+), 21 deletions(-)
```

## Architecture Implemented

The Designs tab now uses one structured result for preview and download. The implementation separates these responsibilities:

- `normalizeDesignDraft()` validates raw form values without substituting geometry defaults.
- `buildDesignResult()` routes normalized input into the legacy or box result pipeline.
- `designBoxDimensions()` performs inside/outside conversion.
- `buildFingerPattern()` chooses deterministic odd segment counts and exact segment widths.
- `designEdgePoints()` generates complementary cleared edge transitions.
- `buildFingerPanel()` creates one closed orthogonal panel contour.
- `buildBoxModel()` defines body dimensions, edge parity, warnings, metrics, and five/six panel models.
- `layoutDesignPanels()` applies the fixed three-row non-nesting layout and verifies panel boxes.
- `serializeDesignSvg()` emits the standalone millimeter SVG.
- `designSvgValidation()` parses the SVG, validates document dimensions/viewBox, finite coordinates, and geometry bounds.
- `designResultsHtml()` renders calculated metrics, errors, warnings, and the exact serialized SVG through an inline data-URL preview.
- `refreshDesignPreview()` updates only the results/preview region for numeric input and updates the download-disabled state.

The existing `designWallPath()` remains limited to legacy tray output. It is not used by the finger-box engine.

The SVG contains one predictable red `#ff0000` cut group. Each box panel is one closed path in a named metadata group with a `<title>`; no visible text is exported as cut geometry. Preview and download use the same SVG string.

## Box Dimensions and Construction

The new `finger-box` template defaults to inside dimensions and accepts:

- width, depth, and height in millimeters;
- measured material thickness;
- nonnegative joint clearance;
- preferred finger-width target;
- open top or optional loose flat lid.

For material thickness `t`:

```text
Wo = Wi + 2t
Do = Di + 2t
Ho = Hi + t

Wi = Wo - 2t
Di = Do - 2t
Hi = Ho - t
```

Outside body height is the underside of the bottom panel to the top of the walls. Inside height is the inside face of the bottom to the top of the walls. The loose lid is a separate plain `Wo × Do` panel and adds one material thickness to overall closed height.

The open-top body contains:

1. Bottom
2. Front
3. Back
4. Left
5. Right

The loose-lid variant adds a sixth panel, Lid.

All four vertical corner joints and all four bottom-to-wall joints use matching nominal segment boundaries with opposite phases. Deterministic parity keeps one finger-side and one recess-side at each panel corner. The box uses one joint style only: alternating rectangular fingers.

## Finger Width and Clearance Semantics

For each outside width, outside depth, and outside height edge family:

- the preferred width is a target;
- the generator selects an odd segment count of at least three;
- actual segment width is exact edge length divided by segment count;
- minimum actual segment width is `max(2 × material thickness, 4 mm)`;
- every mating edge shares the same nominal boundaries and uses the opposite phase.

Joint clearance behavior:

- `0.00 mm` is nominal material-thickness fit;
- positive values make the fit looser;
- negative values are rejected;
- clearance shifts internal finger/recess transitions by half the clearance on each mating side;
- exterior panel envelopes do not change;
- values that collapse a finger below the required web are rejected.

Kerf is deliberately not baked into the SVG. The UI and README explain that the SVG is nominal centerline geometry, clearance adjusts assembly fit, and material-specific kerf must be tested on scrap and applied separately in LightBurn.

## Validation and Warnings

Blocking validation now reports explicit errors for:

- blank, nonnumeric, non-finite, zero, or negative required dimensions;
- invalid material thickness or negative clearance;
- outside width/depth `<= 2t` or outside height `<= t`;
- nonpositive calculated inside dimensions;
- edge lengths unable to support three minimum-width segments;
- clearance that collapses the required finger web;
- unknown template, dimension mode, lid mode, or tray joint mode;
- non-finite/non-orthogonal panel coordinates;
- overlapping panel boxes or geometry outside the document bounds;
- malformed SVG, invalid millimeter dimensions, or viewBox mismatch.

Nonblocking warnings cover unusually loose clearance, unusually small preferred fingers, large boxes/layouts, and the loose lid's lack of retention.

The download button is disabled while invalid. The click handler rebuilds the result and parses the SVG again before creating a download.

## Deterministic Panel Layout

The box layout uses fixed 10 mm margins and 15 mm gaps:

- Row 1: bottom and optional lid;
- Row 2: front and back;
- Row 3: left and right.

The generated document bounds come from the actual translated panel envelopes. Fixtures verify no overlap, no negative coordinates, all geometry inside the viewBox, stable panel order, byte-stable repeated output, and no duplicate translated cut segment.

## Legacy SVG Comparison

After implementation, all four default legacy outputs remained byte-for-byte stable:

| Template | Final length | Final SHA-256 | Result |
|---|---:|---|---|
| `qr-stand` | 359 | `34b50585d4c427b2d9d5a1107432a0e9fbc43f4f74edb4fa7383375f6fa4e499` | exact match |
| `hanging-sign` | 341 | `38300ab2033bb12023b76bf3c78173eaed632da38ca689df1ee4fd50c46e3c3c` | exact match |
| `dice-tray` | 1726 | `2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf` | exact match |
| `divider-tray` | 1965 | `91a5fcaaa54a63d9c5f9b3e9851de70b62fdd98798d356d50977ed4f46f1d924` | exact match |

All four were also downloaded through the updated UI flow and parsed successfully. No physical legacy geometry was redesigned.

## Designs Fixture Results

`runDesignGeometryFixtures()` is exposed on `window`, runs through `?selftest=design`, and participates in `?selftest=all`.

Direct result:

```text
Design geometry fixtures
82 passed
0 failed
82 unique fixture names
```

Coverage includes:

- legacy SVG parsing, positive dimensions, finite tokens, viewBox containment, and captured-output stability;
- inside/outside formulas, round trip, and lid closed height;
- odd segment counts, minimum widths, all eight mating joint pairs, opposite phases, deterministic parity, clearance behavior, and collapse rejection;
- five/six panel counts, names, closed orthogonal paths, finite points, expected envelopes, and all required patterned edges;
- deterministic non-overlapping layout, margins, gaps, document containment, and duplicate-segment detection;
- invalid blank/nonnumeric/zero/negative/short/unknown-mode cases;
- SVG parsing, millimeter dimensions, viewBox equality, one cut group, panel groups, and absence of cut-layer labels;
- warnings and source-draft nonmutation.

## Complete Fixture Suite

Direct `file://` execution of `index.html?selftest=all` returned:

| Group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Project Wizard | 216 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Designs geometry | 82 | 0 |
| **Total** | **614** | **0** |

The existing Project Wizard browser regression also remained green at 31 passed / 0 failed with no runtime errors.

## HTML, JavaScript, Offline, and UI Validation

- `python -m html.parser index.html`: passed.
- Inline JavaScript parsed and executed in Brave through direct `file://` startup.
- `?selftest=design`: 82/0.
- `?selftest=all`: 614/0.
- No external script, stylesheet, `fetch`, or `XMLHttpRequest` dependency was found.
- Direct offline startup still renders all eight tabs without a server.
- Live Designs interaction checks: 24 passed / 0 failed.
- Runtime/console errors during the focused Designs interaction run: none.
- At a 585 px viewport, the preview layout stacked into one column and remained present.
- Numeric input changed the calculated preview while the edited `boxWidth` input retained focus.
- The preview data decoded to the exact same SVG string captured from the download Blob.
- Blank width produced the specific `Width is required` error, removed the preview, and disabled download.
- Correcting the value restored preview and download.

## Representative SVG Verification

Representative defaults used inside dimensions `120 × 90 × 50 mm`, material thickness `3 mm`, preferred finger width `12 mm`, and joint clearance `0.10 mm`.

Calculated body:

```text
Inside:       120 × 90 × 50 mm
Outside body: 126 × 96 × 53 mm
```

Open-top result:

- valid;
- five named panel groups;
- `287 × 252 mm` layout;
- parseable SVG;
- one red cut group;
- no duplicate translated cut segments.

Loose-lid result:

- valid;
- six named panel groups;
- `287 × 252 mm` layout;
- `56 mm` overall closed height;
- parseable SVG;
- explicit no-retention warning.

Both representative SVGs were generated and parsed in memory through the actual download path. No generated SVG capture was added to the repository.

## Session-Only and Integration Boundaries

The Designs draft remains module-level session state only. A browser interaction check compared the full localStorage value before and after Designs editing/export and found no change.

The diff contains no change to:

- `STORAGE_KEY`;
- `SCHEMA_VERSION`;
- `freshState()`;
- `persist()`;
- `backupObject()`;
- merge or replace import behavior;
- Library, Inventory, Projects, Pricing, or Test Grid records.

No design presets, cross-tab links, stock deductions, kerf inheritance, schema migration, DXF, LightBurn-native, G-code, or network integration were added.

## Unverified Manual Checks

The following require the user's installed software and physical material and were not claimed as verified:

- importing the new box SVGs into LightBurn and confirming its displayed document/layer interpretation;
- cutting and assembling a physical open-top or loose-lid box;
- validating real plywood fit at multiple clearance values;
- confirming material-specific kerf compensation in LightBurn;
- checking the generated layout against the user's actual sheet dimensions and L8 workholding setup.

A scrap test is required before production use.

## Final Repository State

Final `git status -sb` after creating this report:

```text
## main...origin/main
 M README.md
 M index.html
?? "LightBurn Projects/"
?? docs/BOX_GENERATOR_PHASE2_IMPLEMENTATION_2026-07-14.md
?? docs/DESIGNS_BOX_GENERATOR_ARCHITECTURE_REVIEW_2026-07-14.md
?? docs/FULL_CODE_AUDIT_2026-07-14.md
?? docs/FULL_PROJECT_ARCHITECTURE_AUDIT_2026-07-12.md
?? docs/LIBRARY_BROWSER_FINAL_VERIFICATION_2026-07-14.md
?? docs/LIBRARY_BROWSER_FOURTH_PASS_REVIEW_2026-07-14.md
?? docs/LIBRARY_BROWSER_SECOND_PASS_REVIEW_2026-07-14.md
?? docs/LIBRARY_BROWSER_THIRD_PASS_REVIEW_2026-07-14.md
?? docs/MATERIAL_BROWSER_ARCHITECTURE_REVIEW_2026-07-14.md
?? docs/MATERIAL_BROWSER_FINAL_VERIFICATION_2026-07-14.md
?? docs/MATERIAL_BROWSER_FOURTH_PASS_REVIEW_2026-07-14.md
?? docs/MATERIAL_BROWSER_SECOND_PASS_REVIEW_2026-07-14.md
?? docs/MATERIAL_BROWSER_THIRD_PASS_REVIEW_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_CORRECTIONS_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_CORRECTIONS_VERIFICATION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_DUAL_UNIT_CORRECTION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_DUAL_UNIT_VERIFICATION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_FINAL_PARSER_CORRECTION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_FINAL_PARSER_VERIFICATION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_IMPLEMENTATION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_REPEATED_SLASH_CORRECTION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_REPEATED_SLASH_VERIFICATION_2026-07-14.md
?? docs/NEW_PROJECT_WIZARD_PHASE1_VERIFICATION_2026-07-14.md
?? docs/PROJECTS_ACCOUNTING_SEPARATION_REVIEW_2026-07-14.md
?? docs/PROJECTS_ACCOUNTING_SEPARATION_SECOND_PASS_REVIEW_2026-07-14.md
?? docs/PROJECT_BROWSER_SECOND_PASS_REVIEW_2026-07-14.md
?? docs/PROJECT_BROWSER_THIRD_PASS_REVIEW_2026-07-14.md
?? docs/TEST_GRID_BRIDGE_SECOND_PASS_REVIEW_2026-07-14.md
?? docs/TEST_GRID_BROWSER_SECOND_PASS_REVIEW_2026-07-14.md
?? docs/TEST_GRID_BROWSER_VERIFICATION_2026-07-14.md
?? docs/TEST_GRID_LIBRARY_BRIDGE_REVIEW_2026-07-14.md
?? parametric_qr_stand_generator.py
```

`git diff --check` passes with only the existing LF-to-CRLF informational warnings. `git diff --cached --name-only` is empty.

Nothing was staged, committed, pushed, stashed, reset, cleaned, moved, or deleted. Existing unrelated untracked files were preserved.
