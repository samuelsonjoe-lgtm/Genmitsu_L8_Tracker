# Concealed Cleat Corner Prototype (J2) — Implementation

## Scope and baseline

Implemented from `77c2395 Fix Drawer Cabinet custom row assembly labels` on synchronized `main` (`0 0` against `origin/main`). The tracked tree was clean and staging empty; the authoritative corrected review remained untracked and unchanged, as did unrelated reports, LightBurn Projects, `.claude/`, `debug.log`, and utilities.

Changed tracked files: `index.html`, `README.md`. This report is new and untracked. Nothing was staged, committed, or pushed.

## Implementation

Added the dedicated `concealed-cleat-corner-prototype` Designs template with session-only raw fields `materialThickness`, `cleatLegLength`, and `cleatWallHeight`. Normalization derives `O`, `H`, `t`, `cleatLayers = clamp(round(6 / t), 1, 3)`, `B = layers × t`, `W = max(3t, 10)`, `tickLength = max(6, 2t)`, and `guideMinRun = 3 × tickLength`.

`buildConcealedCleatCornerDesignResult()` and its local semantic helpers create deterministic rectangular panels in the required order: Base, Wall A, Wall B, Base-cleat-A layers, Base-cleat-B layers, then Seam-cleat layers. Wall A owns the corner; Wall B yields. Base cleat B uses the corrected `O - t - W` length. The model records installed 2-D footprints, zero base-cleat intersection, seam z-range `[B, H - t]`, Wall-B descent `x ∈ [0, t]`, area, intrusion, and ownership values.

Production remains one combined `l8-concealed-cleat-corner-prototype-<date>.svg` with MIME `image/svg+xml;charset=utf-8`. Its J2-local layer serialization emits blue `cleat-placement`, green `assembly-labels`, then red `cut`. Base/Wall/Cleat labels are generated through the existing label path builder; only the final Base-cleat-A, Base-cleat-B, and Seam-cleat layers receive `CA`, `CB`, and `SC`. Added missing deterministic glyph definitions for `S`, `W`, and space so the specified local labels render.

The screen-only `buildConcealedCleatFinishedViewSvg()` reads semantic values only; it never parses or changes production SVG bytes. It shows Base, both walls, both base-cleat positions, seam cleat, inward guide/label faces, and the visible exterior butt seam. It explicitly states: “Assembly and registration prototype - not a strength test.”

## Literal results and validation

- 3 mm / O 80 / H 50: two layers, B 6, W 10; Wall B 77 × 50; Base-cleat A 77 × 10; Base-cleat B 67 × 10; Seam cleat 41 × 10; 9 pieces; union area 1440 mm² (22.5%); intersection 0.
- 6 mm / O 80 / H 50: one layer, B 6, W 18; Wall B 74 × 50; Base-cleat A 74 × 18; Base-cleat B 56 × 18; Seam cleat 38 × 18; 6 pieces; union area 2340 mm² (36.5625%); intersection 0.
- Added 20 focused Designs assertions: independent dimensions/formulas, topology/order, 2-D/3-D non-overlap, guide edges, outer-layer labels, SVG order/validation, filename/MIME, preview byte isolation, invalid inputs, warnings, session-only storage/backup behavior, and one post-structural default golden.
- Default golden: 4171 bytes, FNV-1a `ca0cfb8e`; labels present and all six fit.
- `python -m html.parser index.html`: passed.
- Direct `file://` isolated Edge: readyState `complete`, no fixture failures. Designs geometry: 1034 / 0. Tray-model: 264 / 0.

## Protected boundaries and limitations

No changes were made to storage/schema/import-export, Production Settings, generic serializer/layout, Finger Box geometry, faux-dovetail behavior, Drawer Cabinet geometry or label-prefix behavior, existing production goldens, Library, Projects, Inventory, Test Grid, or promotion. The new template is not a Finger Box joint option and performs no physical verification. Cutting, glue performance, squareness, fit, strength, durability, LightBurn import, and usefulness relative to finger joints remain unverified physical work.

## Final totals

All 15 groups reconcile to `1826 / 0`: baseline 20, normalization 12, production 66, promotion 58, design-production 118, grid 23, grid-machine 18, grid-browser 67, materials 57, library-browser 56, project-browser 61, metadata 12, storage 8, project-wizard 216, Designs 1034. Arithmetic: `792 + 1034 = 1826`.

## Post-audit cleanup

The README complete-suite narrative now correctly states `1806 + 20 = 1826`; final totals are unchanged. The former tautological J2 storage assertion was replaced with a behavioral session-only check that captures `localStorage` and `backupObject()` bytes before generation, builds the production SVG and screen-only Finished View, then verifies unchanged bytes and no cleat draft/template leakage before restoring fixture session variables.

An isolated Sliding-Lid reproducibility probe built its default production SVG before and after the complete available suite and compared both to the existing fixture golden: both were `2800` bytes with hash `4a7ab718`. No Sliding-Lid production change or golden update was made. Post-cleanup validation retained Tray-model `264 / 0`, Designs `1034 / 0`, and complete suite `1826 / 0`.

## LightBurn compatibility correction

LightBurn Pro 2.1.03 imported the original J2 blue score layer as Fill and reported “Open shapes skipped” because open `cleat-placement` guide paths and closed `assembly-labels` paths shared the same blue color. J2 now serializes the open no-fill guides as blue `#0000ff` and the deterministic no-fill label paths as green `#00ff00`, followed by the unchanged red `#ff0000` cut group. The intentional default production SVG change is `4171` bytes / `ca0cfb8e` to `4099` bytes / `c9bfd05c`; the red cut group remains `1606` bytes / `c16a9ef5`, and all non-J2 production goldens passed unchanged.

Focused layer/open-path/label/order/cut-stability checks, Tray-model (`264 / 0`), Designs (`1037 / 0`), and the complete available suite (`1829 / 0`) passed. Joe must still re-export and re-import the corrected SVG in LightBurn Pro 2.1.03 to confirm no warning, separate Line-mode guides, visible engravable labels, and correct red cuts.
