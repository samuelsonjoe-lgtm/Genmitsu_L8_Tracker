# Concealed Cleat Full-Box Prototype implementation — 2026-07-18

## Scope and baseline

- Baseline HEAD: `30ab1f9` (`Fix concealed cleat placement guides`), synchronized with `origin/main` before this work.
- The pre-existing working tree contained only unrelated untracked user files; they were preserved.
- Changed files: `index.html`, `README.md`, and this report. Nothing was staged, committed, or pushed.

## Dedicated experiment

`concealed-cleat-full-box-prototype` is a new session-only Designs template. It is separate from `concealed-cleat-corner-prototype`; the J2 single-corner production golden remains `4457 / 88721533` before and after this change.

The raw session fields are measured material thickness plus outside width, outside depth, and wall height. Defaults are 3 mm, 180 mm, 150 mm, and 90 mm. No storage, schema, backup, import/export, Production Settings, Library, Inventory, Projects, Test Grid, or promotion behavior changed.

## Geometry and ownership

The implementation uses the review’s outside-dimension convention:

- `internalWidth = Ow - 2t`, `internalDepth = Od - 2t`, `internalHeight = H - t`.
- `layers = clamp(round(6 / t), 1, 3)`, `B = layers * t`, `W = max(3t, 10)`, and `guideMinRun = 3 * max(6, 2t)`.
- Base, Wall A, Wall C, Wall B, and Wall D are cut in the required deterministic order, followed by four base and four seam laminated cleat stacks.
- Wall A and Wall C own their two seam cleats. Wall B and Wall D yield at both ends.

At the 3 mm defaults, the file has 21 physical pieces: five panels plus two layers for each of eight cleat stacks. The independent fixture calculations verify all base footprints have zero positive-area overlap, seams meet the base stack only at `z = B`, and Wall B/D descent boundaries remain clear.

## Production output and preview

One deterministic SVG is emitted as `l8-concealed-cleat-full-box-prototype-<date>.svg` with MIME `image/svg+xml;charset=utf-8`. The default golden is `9701 / 2316430b` (length / FNV-1a hash). Its top-level group order is blue `cleat-placement`, green `assembly-labels`, then red `cut`; labels are deterministic paths and no SVG text is emitted.

All eight installed footprints receive four open blue L-corner marks. The results and form state that SVG colors do not select LightBurn modes: both blue and green layers must be assigned to Line mode after import.

The Finished View is semantic and screen-only. It reads the derived model, shows all four base and seam-cleat positions, ownership walls, visible butt seams, internal and clear-floor dimensions, and carries the prototype disclaimer. It does not parse or alter production SVG bytes.

## Validation and limits

Focused fixtures cover formulas, panel order, 3 mm and 6 mm cleat runs, independent overlap checks, ownership, all eight guides, labels, colors, no-text output, filename/MIME, deterministic bytes, preview neutrality, invalid input and minimum-run rejection, warnings, storage isolation, J2 stability, and existing-template goldens.

The final local checks were `git diff --check`, `python -m html.parser index.html`, direct `file://` Edge checks, focused J2.5/J2 fixtures, Tray fixtures, Designs geometry, and the complete available suite: Tray `264 / 0`, Designs `1069 / 0`, complete `1861 / 0`.

No physical claim is made. Required remaining work is physical dry fitting, confirmation of all guide faces and positions after import, final-wall clamp access, squareness, glue-up, fit, strength, durability, and production suitability.

## Post-audit cleanup

- Corrected README arithmetic to `1832 + 29 = 1861`; totals remain Designs `1069 / 0` and complete suite `1861 / 0`.
- Corrected the occupancy warning to describe base-cleat occupancy of the nominal interior floor area; the formula and 30% trigger are unchanged.
- Replaced the static J2.5 persistence assertion with a behavioral before/after comparison around production generation and the semantic preview. It verifies byte-identical `localStorage` and `backupObject()` output, no J2.5 field/template leakage, and restoration of fixture session state.
- Final checks remain green: Tray `264 / 0`, Designs `1069 / 0`, complete `1861 / 0`; J2.5 remains `9701 / 2316430b`, J2 remains `4457 / 88721533`, and production bytes are unchanged.
