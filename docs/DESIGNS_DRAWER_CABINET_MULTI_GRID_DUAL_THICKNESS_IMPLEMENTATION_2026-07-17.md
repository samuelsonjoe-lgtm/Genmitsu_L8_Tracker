# Drawer Cabinet multi-grid and dual-thickness implementation

## Scope and baseline

- Repository: `C:\Genmitsu L8 Tracker`
- Actual starting HEAD: `8bcfbbc Add Dice tray underside cover plate`
- Expected baseline match: yes.
- Starting tracked tree: clean. No staged files.
- Pre-existing untracked content included `.claude/`, `LightBurn Projects/`, `debug.log`, historical reports, the two supplied primary design reports, and `parametric_qr_stand_generator.py`. It was left untouched.

This pass implements only the Drawer Cabinet multi-grid and dual-thickness feature. It does not change storage, schemas, imports, backups, Production Settings schemas, evidence/promotion, other templates, LightBurn projects, or utility files. Nothing was staged, committed, or pushed.

## Changed files

- `index.html`
- `README.md`
- This implementation report

The supplied design review and design challenge files remain unchanged.

## UI and normalized contract

Drawer Cabinet now retains `drawerRows` and adds the `drawerColumns` select with values 1 through 3. `materialThickness` is labelled **Shell thickness (mm)** only for this template. The form adds `drawerUseSeparateInteriorThickness` and conditionally exposes `drawerInteriorThickness` as **Drawer and interior thickness (mm)**.

The raw fields are `materialThickness`, `drawerRows`, `drawerColumns`, `drawerUseSeparateInteriorThickness`, and `drawerInteriorThickness`. Normalization produces `rows`, `columns`, `shellThickness`, `useSeparateInteriorThickness`, and `interiorThickness`.

- Missing columns normalize to 1; rows and columns accept only integer values 1 through 3.
- Separate mode is true only for `true`, `'true'`, or `'on'`.
- Linked mode is the default. It ignores the raw interior value completely and makes `interiorThickness` equal `shellThickness` at every normalization.
- Separate mode validates a finite positive explicit interior value.
- The form explains that equal drawers are retained, partitions are manually positioned and glued, and separate mode requires separate stock and laser settings.

Production Settings still writes only `materialThickness`. This updates both semantic roles in linked mode, while it updates the shell role only in separate mode and leaves the explicit interior value intact.

## Model and construction

`drawerCabinetDimensions()` remains the sole dimension owner. With shell thickness `S`, interior thickness `I`, rows `R`, and columns `C`:

- drawer outside: `inside width + 2I`, `inside depth + 2I`, `inside height + I`
- cell: drawer outside width plus lateral clearance; drawer outside height plus vertical clearance
- cabinet inside width: `C * cellWidth + (C - 1) * I`
- cabinet inside depth: drawer outside depth plus rear clearance
- cabinet inside height: `R * cellHeight + (R - 1) * I`
- cabinet outside: inside width plus `2S`, inside depth plus `S`, inside height plus `2S`

`drawerCabinetCellOrigin(rowIndex, columnIndex, dimensions)` owns the shared bottom-up row / left-to-right column offset used by both the semantic model and Finished Front View.

The five mutually finger-jointed shell panels use shell thickness. Every drawer remains a five-panel open-top Finger Box submodel using interior thickness. Horizontal shelves are full-width, full-depth plain glued rectangles using interior thickness. Each internal column boundary has one plain row-height, full-depth glued partition using interior thickness metadata; its physical cut face is exactly `cellHeight × cabinetInsideDepth` (the representative case is `33.3 × 76.5 mm`). These deterministic IDs use `cabinet-partition-rNN-cNN`. Partitions have no tabs, slots, fingers, dados, notches, keys, or score marks.

Panel order is shell, shelves, partitions ordered by row then boundary column, then drawers ordered by row then column. At one column the existing drawer IDs, titles, panel order, layout rows, score behavior, filename, MIME, shelf guides, and SVG bytes are preserved: `drawer-rNN-*` is retained with no `c01` suffix.

## Production outputs and preview

Linked thickness remains the existing one-file result contract: one `result.svg`, one Cut Layout preview, and the existing `l8-drawer-cabinet-${today()}.svg` filename with `image/svg+xml;charset=utf-8` MIME.

Separate mode has a Drawer-Cabinet-local `productionOutputs` array with exactly two independently laid out files:

| ID | Label | Filename | Representative 6 mm shell / 3 mm interior bounds |
| --- | --- | --- | --- |
| `shell` | Cabinet shell | `l8-drawer-cabinet-shell-${today()}.svg` | 490.8 x 199.1 mm |
| `interior` | Drawers, shelves, and partitions | `l8-drawer-cabinet-interior-${today()}.svg` | 368 x 835.5 mm |

Each output records `id`, `label`, `thicknessMm`, `svg`, `filename`, `mime`, `widthMm`, `heightMm`, and `panelIds`. The shell output contains only the five shell panels and shell-associated shelf guide or assembly-label content. The interior output contains shelves, partitions, and drawer panels only. Score and label entries are filtered by included panel identity. Existing serialization is reused separately, preserving score-before-cut ordering and existing colors.

The compatibility fallback `result.svg` is the shell SVG only when separate mode is active. The generic download is disabled and relabelled; only the two output cards provide production download actions. Every card previews the exact SVG bytes it downloads, presents label, thickness, and independent sheet bounds, and warns that the two sheets require distinct material and laser settings. It does not offer a combined mixed-thickness cut file.

## Finished Front View, metrics, and warnings

The bounded Finished Front View remains screen-only and now shows the rows-by-columns drawer openings/fronts, horizontal shelf bands, and vertical partition bands from semantic model data. It does not read serializer output or either production SVG. In separate mode it displays a screen-only shell/interior thickness and two-file note.

Metrics now identify grid/count, shelf and partition counts, thickness mode and roles, cabinet and drawer dimensions, production-file count, and independent sheet sizes when applicable. Warnings retain existing clearance and laser-safety guidance and add the manual glued-partition limitation, equal-bay dry-fit requirement, and separate-material / Shell-and-Interior-cut warning. No warning promises fit, squareness, strength, verification, or production safety.

## Compatibility and deterministic checks

Legacy linked one-column Drawer Cabinet goldens remain pinned:

- one row: `4456 / a6dd23dc`
- two rows: `6802 / 36a41b07`
- three rows: `9153 / 8c286797`

The focused audit found two production blockers and one metrics issue: partition generation had incorrectly used interior stock thickness as a face dimension; separate one-column output rows always requested `cNN` drawer IDs even though the model preserves legacy `drawer-rNN-*`; and Required pieces read the shell fallback panel array. The correction uses `drawerCabinetDrawerPrefix(rowIndex, columnIndex, columns)` as the single namespace source of truth, changes partition faces to `cellHeight × cabinetInsideDepth`, and reads `metrics.pieceCount` for the result metric. New deterministic checks use a representative linked 2-column by 3-row cabinet and a representative 6 mm shell / 3 mm interior cabinet:

- linked 2x3 corrected: `16429 / 0814ca2e`
- separate shell: `2779 / 956ad870`
- separate interior corrected: `8609 / bad8b97f`

The obsolete linked and interior goldens are historical audit references only. The one-column separate fixture verifies valid two-output membership, legacy drawer IDs, zero partitions, complete shell/interior accounting exactly once, finite bounds, valid SVGs, and semantic Required pieces.

Focused fixtures cover normalization/linking, valid and invalid columns, exact enable flags, deterministic multi-column IDs, equal cells, partitions, role thicknesses, separate-output membership, independent SVG validation and bounds, filename/MIME, the output-card UI, Finished View partition metadata, storage/backup isolation, and the new goldens. Existing Drawer Cabinet fixtures continue to cover legacy bytes, shelf-guide behavior, filenames, MIME, live preview/download identity, storage/backup boundaries, and Finished View behavior.

## Validation

Run successfully from the repository root:

- `python -m html.parser index.html`
- direct `file://` headless Microsoft Edge fixture harness for the default groups
- direct `file://` headless Microsoft Edge complete fixture run

The direct `?selftest=all` runtime result is **1737 passed / 0 failed** across all 15 complete-suite fixture groups, with every baseline group connected and passing. Designs geometry is **945 passed / 0 failed**; the separately callable structured Tray-model group remains **264 passed / 0 failed**. Arithmetic is `1714 - 922 + 945 = 1737`; the prior `1679` manual sum omitted the 58-assertion Evidence Promotion group, so runtime coverage was not reduced. Browser checks used isolated headless Microsoft Edge against direct `file://` startup, including one-column separate and multi-column partition cases. No LightBurn import, machine run, material-cut, glue-up, fit, strength, squareness, or shop validation was performed.

The direct-file runtime reported `readyState: complete`, visible Designs UI, and no storage mutation from the exercised flows. No LightBurn import, machine run, material-cut, glue-up, fit, strength, squareness, or shop validation was performed. Real material still requires correct focus, speed, power, passes, air assist, ventilation, a fire watch, clean optics, flat stock, nearby fire suppression, and an attended physical prototype.

## Final repository state

`git diff --check` is clean. The only intended tracked modifications are `README.md` and `index.html`; this report is a new untracked documentation file. No files are staged. No commit or push was created. Pre-existing unrelated untracked files and reports remain untouched.
