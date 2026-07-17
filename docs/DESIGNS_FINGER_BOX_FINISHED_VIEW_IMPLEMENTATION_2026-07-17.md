# Finger-jointed Box Finished View — Implementation Report

Date: 2026-07-17  
Baseline: `c46fb40 Record machine identity on test grids`  
Architecture source: `docs/DESIGNS_FINISHED_VIEWS_ARCHITECTURE_REVIEW_2026-07-17.md`

## Result

Implemented the Finger-jointed box Finished View as a transient, screen-only preview. The existing flattened production SVG remains the only preview/download source for Cut Layout and the only download source while Finished View is selected.

**Status: READY FOR FOCUSED AUDIT**

## Files changed

- `index.html`
- `README.md`
- `docs/DESIGNS_FINGER_BOX_FINISHED_VIEW_IMPLEMENTATION_2026-07-17.md`

Pre-existing unrelated untracked files and reports were preserved. No commit or push was performed.

## Implementation

- Added `buildFingerBoxFinishedViewSvg(result)`, which consumes the structured Finger Box result dimensions and material thickness supplied by the existing box model/result path.
- Added Finger Box support to the session-only preview mode normalization, selector, result renderer, and preview action binding.
- The selector offers `Cut Layout` and `Finished View`; Cut Layout remains the default. Invalid dimensions reset the mode to Cut Layout, and template switching cannot carry Finished View to another template.
- The assembled view is a restrained top-down open box with a cavity and Front, Back, Left, and Right orientation labels. Outside width, depth, and height are shown.
- A loose lid is emitted only for the loose-lid configuration as a separate flat panel labeled `Loose lid`. No hinge, rail, retention, or sliding geometry is emitted.
- The Finished View is inserted only into the result-area markup. It is not added to `result.svg`, serialization, downloads, backups, persistence, production settings, or any application record.

## Fixture coverage

Added 18 focused assertions to the existing `runDesignGeometryFixtures()` group covering:

- deterministic finite SVG parsing with title and description;
- orientation labels, cavity/open-top behavior, outside dimensions, and positive viewBox bounds;
- separate loose-lid behavior and absence of hinge/rail/retention/sliding geometry;
- draft nonmutation, production SVG isolation, extreme valid dimensions, and invalid-result handling;
- selector accessibility, screen-only result markup, invalidation reset, and template-switch isolation.

The independently executed fixture totals are:

| Group | Passed | Failed |
| --- | ---: | ---: |
| Test Grid machine identity | 18 | 0 |
| Evidence Promotion | 58 | 0 |
| Production Settings | 66 | 0 |
| Designs production application | 118 | 0 |
| Designs geometry | 605 | 0 |
| Complete deduplicated suite | 1397 | 0 |

The other independently executed groups also remained green: baseline 20, normalization 12, Grid Promotion 23, Grid Browser 67, Material Browser 57, Library Browser 56, Project Browser 61, Project Wizard 216, wizard metadata 12, and storage recovery 8.

## Validation

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Fresh headless Microsoft Edge direct-open execution: all listed fixture groups passed, including `runDesignGeometryFixtures()` at 605 / 0 and the complete deduplicated total at 1397 / 0.
- The focused result-area checks confirmed the Finished View is separate from the production SVG and that the loose lid is separate and non-mechanical in its markup.
- The final diff is limited to the Finger Box preview, its focused fixtures, README documentation, and this report. The existing Drawer Cabinet Finished Front View fixture coverage remains present.

## Protected-boundary comparison

Compared with `c46fb40`, this pass does not alter storage keys/schema, persistence/recovery, backup/import/replace/merge behavior, production normalizers or application, Material Test, Test Grid, Project schemas, `buildFingerPanel()`, `buildSlidingLidBoxModel()`, Drawer Cabinet geometry, Sliding Lid geometry, LightBurn layer serialization, `serializeDesignSvg()`, or the download filename/path. The additive box-result metadata is limited to transient `boxLid` and `materialThickness` values needed by the screen renderer.

## Remaining audit scope

Focused audit should inspect the final `index.html` diff and perform a visual browser walkthrough for open-top and loose-lid states. Physical fit, strength, kerf accuracy, glue quality, and production safety remain outside the Finished View's claims and require material-specific testing.
