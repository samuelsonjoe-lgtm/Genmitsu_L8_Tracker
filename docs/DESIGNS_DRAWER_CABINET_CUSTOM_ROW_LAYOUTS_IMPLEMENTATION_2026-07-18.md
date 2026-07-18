# Drawer Cabinet custom-by-row layouts implementation

## Scope and baseline

- Repository: `C:\Genmitsu L8 Tracker`
- Starting HEAD: `eca1500 Add multi-grid dual-thickness drawer cabinets`
- Baseline match: yes; `main` and `origin/main` were synchronized (`0 0`).
- Starting tracked tree: clean, no staged files.
- Pre-existing untracked files included `.claude/`, `LightBurn Projects/`, `debug.log`, historical reports, the authoritative custom-row review, and `parametric_qr_stand_generator.py`. They were left untouched.

This pass changes only the Drawer Cabinet implementation in `index.html`, the concise README description, and this new report. It does not alter storage, schema, imports, backups, Production Settings schema, evidence/promotion, Test Grids, Library, Inventory, Projects, Pricing, shared serializer, shared layout helper, or other templates. Nothing was staged, committed, or pushed.

## UI and normalization

The Drawer Cabinet now provides **Drawer layout** with exact values `uniform` (**Uniform grid**) and `custom-row` (**Custom by row**). Uniform retains Drawer rows and Drawer columns. Custom retains Drawer rows, hides Drawer columns, and shows selectors visually top-to-bottom:

- three rows: Top row drawers, Middle row drawers, Bottom row drawers;
- two rows: Top row drawers, Bottom row drawers;
- one row: Drawers in row.

New raw session-only fields are `drawerLayoutMode`, `drawerBottomColumns`, `drawerMiddleColumns`, and `drawerTopColumns`; `drawerRows` and `drawerColumns` remain. Normalization exposes `layoutMode`, bottom-up `rowDrawerCounts`, and `maximumColumns` alongside the current thickness fields.

- Missing or blank layout mode is Uniform; nonblank unrecognized values fail with `Choose a recognized drawer layout.`
- Uniform validates `drawerColumns`, ignores every dormant custom field, and emits `Array(rows).fill(columns)`.
- Custom validates only active row controls and ignores inactive stale values and dormant `drawerColumns`.
- The visual top-to-bottom to semantic bottom-up mapping occurs only in normalization. For example, top-to-bottom `1 / 3` with two rows becomes `[3, 1]`.

The custom-mode width help makes clear that the row with the most drawers uses the entered drawer inside width and rows with fewer drawers derive a wider equal division.

## Semantic model

`drawerCabinetDimensions()` now accepts the normalized row counts and remains the single owner of shared cabinet, height, depth, and role-thickness dimensions. `maximumColumns` replaces the uniform column count in the cabinet-width formula:

```text
baseDrawerOutsideWidth = drawerInsideWidth + 2 * interiorThickness
baseCellWidth          = baseDrawerOutsideWidth + lateralClearance
cabinetInsideWidth     = maximumColumns * baseCellWidth
                       + (maximumColumns - 1) * interiorThickness

rowCellWidth(n)          = (cabinetInsideWidth - (n - 1) * interiorThickness) / n
rowDrawerOutsideWidth(n) = rowCellWidth(n) - lateralClearance
rowDrawerInsideWidth(n)  = rowDrawerOutsideWidth(n) - 2 * interiorThickness
```

`drawerCabinetRowLayout(rowIndex, dimensions)` is the bounded Drawer Cabinet helper that owns each row's count, derived widths, drawer origins, partition origins, and closure. It calculates before six-decimal output rounding and validates row closure to `1e-5`. `drawerCabinetCellOrigin()` delegates to it for the horizontal origin and retains the established bottom-up vertical pitch.

At current defaults, the maximum-three interior width is `325.35 mm`; a custom top-to-bottom `1 / 2 / 3` cabinet has bottom-up inside widths `100`, `154.725`, and `318.9 mm`.

## Construction, IDs, layout, and production

Each distinct active drawer count has one cached open-top Finger Box submodel, up to three. Uniform therefore remains a one-submodel degenerate case. All shell panels use shell thickness; drawers, shelves, and plain row-height partitions use interior thickness. No unequal-thickness finger joint is introduced.

Partitions remain `cellHeight x cabinetInsideDepth` plain closed rectangles with interior metadata, no joinery, no score marks, and manual glued placement. Their count is `sum(rowDrawerCount - 1)`.

Panel order is unchanged: shell, shelves, partitions bottom-to-top then left-to-right, then drawers bottom-to-top then left-to-right. `drawerCabinetDrawerPrefix()` remains the only prefix source:

- maximum count one: legacy `drawer-rNN-*` prefixes for every row;
- otherwise: every drawer has `drawer-rNN-cNN-*`, including a one-drawer row in a mixed cabinet.

Uniform linked output remains one SVG. Separate thickness remains exactly two role-based outputs, `shell` and `interior`, with independent layouts, previews, downloads, filenames, MIME, and generic fallback suppression. Custom topology changes neither output shape nor membership policy; every model panel appears exactly once across the separate outputs.

## Finished View, metrics, warnings, and boundaries

The screen-only Finished Front View uses semantic row layouts only. It renders the correct fronts per row, row-specific wider fronts, full-width shelf bands, and row-specific partition bands without reading a production SVG. Separate mode retains its two-file thickness note.

Custom metrics show Custom by row, top-to-bottom configuration, maximum drawers in a row, and one inside-width line for each active row. They retain shell/interior roles, dimensions, piece count, production-file count, and separate output-sheet bounds. Custom warnings state that widths are equal only within each row, partitions are manually positioned and glued with no marks, each row must be dry-fit, wide drawers may rack/flex, very small drawers may be fragile, and the first physical build must confirm fit and squareness. Existing practical laser-safety and separate-stock warnings remain intact.

This remains session-only. Production Settings still updates `materialThickness`: linked mode therefore updates both semantic roles, while separate mode updates only the shell role and retains explicit interior thickness.

## Goldens and fixtures

All six existing Uniform pins pass unchanged:

- legacy one, two, three row: `4456/a6dd23dc`, `6802/36a41b07`, `9153/8c286797`
- linked uniform 2x3: `16429/0814ca2e`
- separate shell/interior: `2779/956ad870`, `8609/bad8b97f`

New focused production pins were added only after structural coverage:

- linked Custom top-to-bottom `1 / 3`: `13606/500396df`
- separate-thickness Custom top-to-bottom `1 / 2 / 3` interior: `14777/69b0ce8b`

The 15 new Designs assertions cover mode normalization, dormant and inactive values, exact top/bottom mapping, `1 / 3`, `2 / 1`, and `1 / 2 / 3` counts, width model and closure, cached row submodels, IDs, partitions, Uniform/custom byte equivalence, separate-output membership, Finished View semantics, custom metrics, isolation, and the two new pins.

## Validation

Successful commands/checks:

- `git diff --check`
- `python -m html.parser index.html`
- local direct-file (`file://`) isolated headless Microsoft Edge focused harness
- all 15 registered fixture groups in isolated headless Microsoft Edge

Runtime was available and direct-open reported `readyState: complete` with no runtime exceptions. Final totals are **1752 passed / 0 failed**, with Designs geometry **960 / 0** and structured Tray-model **264 / 0**. Complete-suite arithmetic is `1737 + 15 = 1752`; every registered group remained connected.

No LightBurn import, machine cut, physical drawer-fit test, glue-strength test, partition-placement test, wide-drawer load test, or squareness validation was performed. Physical work still requires measured stock, correct focus and laser settings, ventilation, active fire watch, clean optics, flat stock, nearby fire suppression, attended operation, dry fitting, and a real prototype.

## Final state

Only `index.html` and `README.md` are modified tracked files. This report is new. The authoritative design review and historical multi-grid reports are unchanged. Unrelated untracked files remain untouched; nothing is staged, committed, or pushed.
