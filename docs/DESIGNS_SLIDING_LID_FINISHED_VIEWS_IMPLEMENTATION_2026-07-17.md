# Sliding-lid Finger Box Finished Views — Implementation Report

Date: 2026-07-17  
Baseline: `5a3f0f3 Add finger box finished view`  
Architecture source: `docs/DESIGNS_FINISHED_VIEWS_ARCHITECTURE_REVIEW_2026-07-17.md`

## Result

Implemented session-only Sliding-lid finger box `Finished Closed` and `Finished Open` views beside the existing `Cut Layout`. Both views are screen-only; the production cut SVG remains the only download source.

**Status: READY FOR FOCUSED AUDIT**

## Files changed

- `index.html`
- `README.md`
- `docs/DESIGNS_SLIDING_LID_FINISHED_VIEWS_IMPLEMENTATION_2026-07-17.md`

All unrelated untracked files were preserved. No reset, clean, stash, delete, move, stage, commit, or push was performed.

## Helpers and functions modified

- Added `buildSlidingLidFinishedViewSvg(result, state)` for the template-owned Closed/Open pseudo-isometric renderer.
- Added minimal transient `metrics.slidingFinishedView` identity metadata plus `materialThickness` in `buildSlidingLidDesignResult()`.
- Extended `designPreviewModeForTemplate()`, `designPreviewSelectorHtml()`, `designResultsHtml()`, and preview button binding for the two Sliding Lid modes only.
- Retained the existing Finger Box and Drawer Cabinet mode handling unchanged.

## Authoritative inputs and layout

The renderer consumes the validated Sliding Lid result only: outside and usable-cavity dimensions; material thickness; Front height; Back-stop/channel values; rail length, height, and position; lid width, length, and engagement; the authoritative Front/Back/rail/lid component IDs; guide state; and coupon state. It does not parse the production SVG, panel paths, rendered coordinates, or create a second geometry model.

Closed places the lid with its back edge at the model's inside Back plane, adjacent to the Back stop. Open uses a documented screen-only offset toward Front: the smaller of 42% of the modeled channel length and the larger of two material thicknesses or 35% of usable depth. It remains less than the modeled channel length and never describes itself as an exact mechanical travel limit.

Front is always labeled `Front — lid entry`; Back is always labeled `Back — stop`; the arrow is always `Front → Back`. Exactly two internal rails derive from the two authoritative rail IDs. Guide marks are only noted when enabled; the optional fit coupon is omitted from the assembly and labeled `Separate fit coupon: Cut Layout only` when enabled.

## Production isolation

Finished View markup is used only in the result-area HTML. It never enters `result.svg`, `serializeDesignSvg()`, LightBurn cut/score layers, download data, filenames, backups, saved Designs, Projects, Production Settings, Material Tests, or any persisted record. Downloading while Finished Open is selected returns the unchanged production Cut Layout SVG and original filename pattern.

## Fixture coverage

Added 23 focused Designs geometry assertions covering:

- deterministic Closed/Open documents, distinct states, finite SVG, Front/Back identity, exactly two rails, dimensions, travel arrow, and prohibited mechanism absence;
- closed Back-stop position, restrained Open Front translation, guide annotation, separate coupon behavior, invalid-result safety, and extreme valid proportions;
- nonmutation of draft/model panels/production SVG, selector defaults, state switching, storage/backup isolation, download identity, invalidation reset, template switching, and button interaction in the result area.

The existing Finger Box Finished View and Drawer Cabinet Finished Front View assertions remain in the same fixture group.

## Validation

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Fresh isolated headless Edge direct `file://` startup had no page errors for `?selftest=design`, `design-production`, `production`, `promotion`, `grid-machine`, `storage`, and `all`.
- Actual Designs-form interaction confirmed Cut Layout is the default, Closed and Open render correctly, the Open lid receives a positive Frontward offset, guide/coupon annotations behave as scoped, and no coupon geometry is inserted.
- Independently invoked fixture totals: Test Grid machine identity 18 / 0; Evidence Promotion 58 / 0; Production Settings 66 / 0; Designs production application 118 / 0; Designs geometry 628 / 0; complete deduplicated suite 1420 / 0.

## Protected-boundary comparison

The `git diff --unified=0 5a3f0f3 -- index.html` review shows only the additive Sliding Lid result metadata, screen renderer, template-scoped preview UI, and focused fixtures. It contains no hunks in `STORAGE_KEY`, `SCHEMA_VERSION`, persistence/recovery, backup/import/merge/replace, production normalizers/application, Material Test/Test Grid/Project schemas, `buildFingerPanel()`, Finger Box geometry, Drawer Cabinet geometry, `serializeDesignSvg()`, download path/filename, or LightBurn serialization.

## Unverified areas

The renderer does not prove smooth travel, physical fit, rail alignment, glue strength, kerf accuracy, material behavior, or production safety. Those remain material-specific physical validation work; the Open offset is a restrained screen presentation, not a measured travel specification.
