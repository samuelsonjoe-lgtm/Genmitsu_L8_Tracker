# Dice Tray Underside Cover Plate Implementation

**Date:** 2026-07-17
**Baseline / actual HEAD before editing:** `c064896` - Add drawer cabinet shelf-placement guides

## Scope and correction

This uncommitted Dice-only feature adds session-only `trayBottomCover`, normalized as `bottomCover` only for `dice-tray`; only `true`, `'true'`, and `'on'` enable it. Divider Tray cannot enable it.

The first implementation used a 3 mm inset, producing a 220 x 160 mm default cover. The focused audit found that this lateral-only analysis missed the radial slot band: at default 3 mm material, each wall slot begins 1.5 mm from the structural-base perimeter. The inset therefore left part of each slot exposed. This report now describes the corrected implementation.

## Final geometry and contracts

- `buildTrayModel()` appends one final `trayRectComponent('tray-bottom-cover', 'Tray bottom cover', 'bottom-cover', ...)` only when enabled. It remains the last component, material-component ID, and anonymous red SVG rectangle.
- The final cover is full size: `bottomCoverInsetMm = 0`, `bottomCoverWidthMm = baseOutsideWidthMm`, and `bottomCoverDepthMm = baseOutsideDepthMm`. No thickness-dependent inset, reveal option, marks, holes, nesting, or serializer change was added.
- With the default 220 x 160 mm inside tray and 3 mm material, the structural base is 226 x 166 mm. The cover is 226 x 166 mm at `(10, 312)`; disabled layout height remains 307 mm and enabled layout height is `307 + 15 + 166 = 488 mm`. Layout width remains 421 mm.
- A base-local containment fixture checks every front, back, left, and right wall-slot rectangle against the full cover footprint for default Finger and two-tab profiles, near-minimum cases, and a valid 0.1 mm material case. The default front-slot example is `y = 1.5..4.65` mm within the `[0, 166]` mm footprint.
- Disabled Dice remains `1726 / 51a55721`; Divider remains `1965 / a55dda6e`. The corrected enabled Dice golden is `1778 / 8e2ea3f4`; the former `1778 / ec9b12bc` pin is obsolete.
- Existing tray components, the anonymous serializer, Divider Tray, Finished View geometry, downloads, filenames, MIME type, storage/schema, backup/import/export, and non-tray templates remain unchanged. Finished View representation of the cover remains intentionally deferred.

## UI and physical limits

The Dice-only control now says **Add underside cover plate**. It describes a full-size cosmetic plate that must be manually aligned to the tray edges. It does not strengthen or replace the structural joint. Before gluing, tabs must be seated flush and both surfaces cleaned of soot, char, raised fibers, glue residue, and debris; the first real tray must verify concealment, alignment, and flatness. No physical cut, LightBurn import, or assembly was performed.

## Validation

- `python -m html.parser index.html` and `git diff --check` passed.
- Isolated headless Edge opened the local `file://` page and passed focused Tray-model fixtures: `264 / 0`; Designs geometry: `922 / 0`.
- The four corrected focused assertions replace the prior 12-cover assertion block with 16 cover assertions: baseline Tray-model `248 / 0`, then `248 + 16 = 264`; baseline Designs `906 / 0`, then `906 + 16 = 922`; baseline complete suite `1698 / 0`, then `1698 + 16 = 1714`.
- Fixtures cover normalization, Divider isolation, disabled byte goldens, unchanged component order, full-size metrics and layout, two-dimensional slot containment, anonymous SVG ordering, preview/download identity, filename/MIME, storage/backup isolation, and Finished View independence.

Nothing was staged, committed, or pushed. The existing untracked project files and historical design-review and focused-audit reports were not modified.
