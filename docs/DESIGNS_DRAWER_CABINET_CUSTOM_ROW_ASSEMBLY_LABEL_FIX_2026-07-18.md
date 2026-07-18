# Drawer Cabinet Custom-Row Assembly Label Fix

**Date:** 2026-07-18  
**Baseline:** `d838041 Add faux dovetail engraving to Finger Box`  
**Scope:** bounded blue assembly-label target-ID correction; no commit or push.

## Initial state

- `HEAD` was `d838041` on `main`; `HEAD...origin/main` was `0 0`; no files were staged and the tracked tree was clean.
- The corrected untracked [Concealed Cleat review](DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md) was present and was not edited, renamed, staged, deleted, or replaced.
- Existing untracked reports, `.claude/`, `LightBurn Projects/`, `debug.log`, and utility files were left untouched.

## Root cause and correction

`designAssemblyLabelSpecs()` was generating one legacy bare prefix per drawer row (`drawer-rNN-*`). `buildDrawerCabinetModel()` generates each real drawer through `drawerCabinetDrawerPrefix(row, column, maximumColumns)`, which correctly includes `-cNN` for **every** drawer whenever `maximumColumns > 1`, including a one-drawer row in a mixed custom layout. The label specifications therefore requested nonexistent targets such as `drawer-r01-bottom`, while the model owned `drawer-r01-c01-bottom`.

The sole production correction makes `designAssemblyLabelSpecs()` iterate each active row's actual `rowDrawerCounts` and call `drawerCabinetDrawerPrefix()` for every drawer. The prefix helper remains the sole policy owner:

- `maximumColumns === 1` → legacy `drawer-rNN-*`.
- `maximumColumns > 1` → `drawer-rNN-cNN-*` for all drawers, including `c01` in one-drawer rows.

No panel IDs, geometry, paths, ordering, layout, serializer, Finished Front View, filenames, MIME type, storage, schemas, or Production Settings applicability changed.

## Observed behavior

The screenshot-style custom layout (top 2 / bottom 1; internal `rowDrawerCounts = [1, 2]`) now requests and resolves the fifteen drawer targets `drawer-r01-c01-*`, `drawer-r02-c01-*`, and `drawer-r02-c02-*`, plus the five unchanged shell targets. The live direct-open UI reports **Assembly labels: Enabled - 20 safe labels**, emits 20 blue assembly-label paths, retains score-before-cut ordering, and has no `missing panel drawer-` warning.

Linked output keeps shell and drawer labels in one score group. In separate-thickness custom 1 / 2 / 3 output, the shell SVG has only its five shell labels and the interior SVG has all 30 drawer labels; membership filtering excludes cross-output labels intentionally without a missing-panel warning. Label IDs remain deterministic and unique because `buildAssemblyLabelPaths()` namespaces them with the resolved panel ID.

## Validation

- `python -m html.parser index.html` passed.
- Isolated headless Edge direct `file://` startup reached `readyState: complete` with no runtime exceptions. The UI exercised the screenshot case with labels enabled: correct `c01`/`c02` panel IDs, 20 labels, blue score before red cut, and no false missing-panel warning.
- Added **20** focused Designs assertions covering uniform one/multi-column, custom 2/1, 1/2, 1/3, 2/1/3, all-one, separate custom 1/2/3, authoritative prefixes, model-target resolution, label IDs, label counts, shell/interior membership, genuine containment omission, storage, and cut-byte identity.
- Designs geometry: **1014 / 0** (was 994 / 0). Tray-model: **264 / 0** unchanged.
- Complete 15-group suite: **1806 / 0**. Arithmetic: `20 + 12 + 66 + 58 + 118 + 23 + 18 + 67 + 57 + 56 + 61 + 12 + 8 + 216 + 1014 = 1806`.
- Existing label-disabled Drawer Cabinet goldens remain unchanged, including default linked `4456 / a6dd23dc`, custom linked 1/3 `13606 / 500396df`, and separate custom 1/2/3 interior `14777 / 69b0ce8b`. Focused comparisons prove red cut-group markup, semantic IDs, paths, and layout coordinates remain identical with labels enabled versus disabled.
- `git diff --check` passed. No LightBurn import, physical engraving readability, physical cabinet assembly, fit, or strength was claimed or verified.

## Final state

Tracked edits are `index.html` and `README.md`; this report is new and untracked. Nothing was staged, committed, or pushed.
