# Drawer Cabinet blocker correction — 2026-07-17

## Scope and state

- Repository: `C:\Genmitsu L8 Tracker`
- Actual HEAD: `8bcfbbc Add Dice tray underside cover plate`
- Audit verdict corrected: `NOT SAFE TO COMMIT`
- Before: tracked modifications were `index.html` and `README.md`; nothing was staged. Historical design review, challenge, focused audit, and implementation reports were present and left intact.
- After: `index.html`, `README.md`, the updated implementation report, and this correction report are the only task files changed or added. Unrelated untracked files remain untouched.
- Nothing was staged, committed, or pushed.

## Corrections

F1 root cause was partition construction using `interiorThickness` as the visible rectangle width. The implementation now creates every `cabinet-partition-rNN-cNN` as a closed plain rectangle with face dimensions `cellHeight × cabinetInsideDepth`; the representative literal is `33.3 × 76.5 mm`. `thicknessMm` remains `interiorThickness`, `thicknessRole` remains `interior`, `rowHeightMm` remains `cellHeight`, and `depthMm` remains `cabinetInsideDepth`. Partition count remains `(columns - 1) × rows`; no tabs, slots, fingers, dados, notches, keys, score geometry, or placement guides were added.

F2 root cause was the separate interior layout always constructing `drawer-rNN-cNN-*`. The model and every Drawer Cabinet layout now use `drawerCabinetDrawerPrefix(rowIndex, columnIndex, columns)`: one column produces exact legacy `drawer-r01-*`, `drawer-r02-*`, and `drawer-r03-*`; multiple columns produce `drawer-rNN-cMM-*`. The one-column separate case is valid, has a non-empty interior output, zero partitions, and preserves all drawer panels.

F4 root cause was the result panel reading `result.panels.length`, which is the shell fallback in separate mode. Required pieces now reads `result.metrics.pieceCount`, the semantic full-model count.

## Accounting and goldens

The one-column separate fixture compares the shell/interior output union with the semantic full-model piece count, requires every physical panel exactly once, checks output panel IDs against resolved and serialized panels, and validates finite bounds and SVG structure. The corrected multi-column goldens are:

- linked 2-column × 3-row: `16429 / 0814ca2e`
- separate shell: `2779 / 956ad870`
- separate interior: `8609 / bad8b97f`

The prior linked `16420 / 250efbb6` and separate interior `8603 / 41162757` values are obsolete because they encoded F1 geometry. Legacy one-, two-, and three-row linked goldens remain unchanged: `4456 / a6dd23dc`, `6802 / 36a41b07`, and `9153 / 8c286797`.

## Validation

- `git diff --check`: clean.
- Direct `file://` isolated headless Microsoft Edge runtime: all 15 complete-suite fixture groups, `1737 passed / 0 failed`; every baseline group remained connected and all groups passed.
- Arithmetic correction: `1714 - 922 + 945 = 1737`. The prior `1679` manual sum omitted the 58-assertion Evidence Promotion group; runtime coverage was not reduced.
- Designs geometry: `945 passed / 0 failed`.
- Separately callable structured Tray-model group: `264 passed / 0 failed`.
- Broader exposed fixture groups, including production settings, evidence, design-production, machine identity, browser flows, storage, and project flows: zero failures.
- Direct-file startup reached `readyState: complete`; Designs UI was available; storage and backup isolation checks passed.
- HTML/inline JavaScript were exercised by the browser harness.

These checks do not claim LightBurn import, machine cutting, physical partition dimensions, drawer fit, glue-up, squareness, strength, or production success. Physical verification remains required.

## Protected boundaries

No storage or schema change, generic multi-output framework, automatic nesting, kerf compensation, partition guides, custom-by-row layout, unrelated template change, LightBurn project change, debug-log change, utility change, staging, commit, or push was performed. The design review, design challenge, and focused audit remain historical records unchanged.
