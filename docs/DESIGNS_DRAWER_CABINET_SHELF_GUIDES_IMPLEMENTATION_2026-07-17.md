# Drawer Cabinet Shelf-placement Guides Implementation

**Date:** 2026-07-17
**Baseline and actual HEAD before editing:** `a36d7ea` — Add wall-to-base tab clearance coupon
**Scope:** Optional session-only Drawer Cabinet blue-score marks; no commit or push

## Working tree and files

Before editing, tracked files were clean. Pre-existing untracked material included `.claude/`, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, the supplied Shelf-guides design review, and prior documentation reports; all remain untouched.

Changed tracked files:

- `index.html`
- `README.md`

Added implementation report:

- `docs/DESIGNS_DRAWER_CABINET_SHELF_GUIDES_IMPLEMENTATION_2026-07-17.md`

No files were staged, committed, pushed, reset, cleaned, stashed, renamed, moved, or deleted.

## Delivered behavior

- Added the session-only raw checkbox field `drawerShelfGuides`, normalized as `shelfGuides`. Only `true`, `'true'`, and `'on'` enable it; absent, false, malformed, and other values disable it.
- Added **Shelf-placement guides** beside the Drawer Cabinet assembly-label option. The help text identifies the Left, Right, and Back interior faces, actual upper-shelf elevations, blue-score-before-red-cut order, edge-glue requirement, existing prototype/glue-strength warning, intended interior orientation, and first-cabinet top/bottom confirmation.
- One-row cabinets remain valid with zero guide entries and a warning that there is no upper shelf to mark.
- Each upper shelf creates three score-path entries — one on each of `cabinet-left`, `cabinet-right`, and `cabinet-back` — and every entry contains two short horizontal tick segments. Thus 1/2/3 cabinet rows produce 0/3/6 guide entries and 0/6/12 line segments.
- Result metrics expose `shelfGuides`, `shelfGuideCount`, `shelfGuideEntries`, `shelfGuideEdgeInset`, `shelfGuideJoineryInset`, and `shelfGuideTickLength`. The result panel reports `Disabled` or `Enabled - N marks`, where N is guide entries, not individual line segments.

## Geometry convention

`drawerCabinetShelfGuideSegments()` is Drawer-Cabinet-owned and uses only the production model’s `model.metrics.shelfBottomElevations`, current cabinet dimensions, current material thickness, and production panel-local geometry.

- Panel-local `y = 0` is the cabinet’s physical top.
- Shelf elevations are measured upward from the inside floor. Their top-down bottom-edge coordinate is:

  `localY = cabinetOutsideHeight - materialThickness - shelfBottomElevation`

- Tick dimensions are `edgeInset = max(2 mm, materialThickness)` and `tickLength = max(6 mm, 2 × materialThickness)`.
- Cabinet side panels have a toothed local end, so the tick origins also apply a `joineryInset = materialThickness`; this leaves the requested edge inset from retained material instead of touching a tooth. The two-end pattern remains mirror-invariant and does not mirror or alter any cut panel.

For the default two-row case, `cabinetOutsideHeight = 75.6 mm`, `materialThickness = 3 mm`, and `shelfBottomElevation = 33.3 mm`, yielding `localY = 39.3 mm`. The Left panel’s hand-derived ticks are `(6,39.3)–(12,39.3)` and `(67.5,39.3)–(73.5,39.3)`.

## Serialization and protected boundaries

- `serializeDesignSvg()` now honors an optional `result.scoreGroupId`, retaining `rail-guides` as the default for existing callers. Drawer Cabinet supplies `shelf-guides`.
- When enabled, the blue `shelf-guides` subgroup serializes before `assembly-labels` and before the red `cut` group. Disabled Drawer Cabinet SVGs remain byte-identical.
- No Drawer Cabinet dimension, shelf, drawer, finger-pattern, clearance, nesting, or red cut path changed. No cabinet panel is mirrored. Finished Front View receives no guide path or coordinate data.
- Preview and download remain the same production SVG; filename and MIME remain `l8-drawer-cabinet-YYYY-MM-DD.svg` and `image/svg+xml;charset=utf-8`.
- No storage, schema, state, persistence, backups, import/export, Production Settings, evidence, promotion, Library, Projects, Inventory, Pricing, Test Grid, Material Test, or other template behavior changed.

## Regression pins and fixtures

Disabled Drawer Cabinet production SVG goldens remain:

- One row: `4456 / a6dd23dc`
- Two rows: `6802 / 36a41b07`
- Three rows: `9153 / 8c286797`

Enabled default two-row Shelf-placement guides are structurally checked and pinned at `7534 / dd3ff0bd`.

Nineteen Designs assertions were added: `887 + 19 = 906`. The complete suite changes from `1679` to `1698` assertions: `1679 + 19 = 1698`.

The focused coverage verifies normalization; disabled byte identity; all row goldens; unchanged red paths and panel order; score ordering; subgroup selection; guide and segment counts; panel ownership; literal elevation/tick coordinates; finite containment and cut clearance; Left/Right mirror invariance; Back elevation agreement; row regeneration; metric wording; Finished Front independence; Sliding-Lid `rail-guides` compatibility; assembly-label coexistence; and preview/download, filename, MIME, localStorage, and backup isolation.

## Validation performed

- `git status -sb`
- `git log -1 --oneline` — `a36d7ea Add wall-to-base tab clearance coupon`
- `git diff --check` — passed
- `python -m html.parser index.html` — passed
- Isolated headless Microsoft Edge direct `file://` startup — passed
- Browser fixture groups — Designs `906 / 0`, Production Settings `66 / 0`, Evidence Promotion `58 / 0`, Designs production `118 / 0`, Test Grid machine identity `18 / 0`, Storage recovery `8 / 0`, and complete suite `1698 / 0`
- Browser form exercise confirmed the Drawer Cabinet checkbox renders, enables the score-layer preview, and retains the normal offline direct-open path.

The only non-feature fixture correction isolates the existing Sliding-Lid preview test’s temporary `designResults` element before clicking its controls. It fixes a duplicate-ID test harness collision and does not change production behavior or SVG output.

## Physical limits and unverified areas

These score marks are positioning aids only. They do not establish shelf squareness, glue strength, long-term support, plywood flatness, laser score visibility/depth, actual face orientation, or correct physical top/bottom orientation. The first physical cabinet using the option must confirm the marked interior face and top/bottom convention before relying on the marks; shelves still require edge glue and the existing shelf alignment/glue-strength prototype warning remains applicable.

No physical cut, LightBurn import, score-depth check, or cabinet assembly was performed in this software pass.
