# Designs Dice Tray DT1 Focused-Audit Correction — 2026-07-20

## 1. Repository state

Inspection confirmed `main`, `HEAD=294cc064f82020ce02ca95e2ced997f003715ba9` (`294cc06`), `origin/main...main=0 0`, no staged changes, and existing DT1 edits uncommitted. `git diff --check` was clean before and after. Architecture, implementation, and focused-audit reports were present. Existing untracked LightBurn projects, historical reports, `debug.log`, and `parametric_qr_stand_generator.py` were preserved.

## 2. Blocker reproduction

Before correction, enabling a wood insert and underside cover placed both red physical panels in the same strip. The cover's old position used the legacy `existingLayoutHeightMm` calculation and did not account for the DT1 insert/divider strip. DT1 labels were also serialized whenever DT1 was active, even when Assembly Labels was disabled.

## 3. Insert/cover layout correction

The model now computes the lowest occupied physical DT1 panel edge, then places the underside cover after that edge plus the existing 15 mm panel gap. The legacy cover position is untouched when DT1 physical additions are disabled. Measurement-only insert modes do not reserve an empty panel strip. Repeated output is deterministic.

Direct fixture checks inspect `model.layout.materialComponentIds` bounding boxes rather than only SVG strings. Wood+cover, right/left storage+wood+cover, storage divider+cover, wood without cover, cover without insert, and non-wood liner+cover all pass non-overlap checks; panel gap checks also pass.

## 4. Storage threshold correction

Every DT1 storage-width threshold now uses 20 mm: form `min`, guidance, normalization, model validation, and fixtures. 17.9, 18, 19, and 19.99 mm block. 20 and 27.9 mm are valid with the below-28 mm retrieval warning. 28 mm is valid without that warning. The default remains 42 mm.

## 5. Insert-height correction

Enabled insert types now block when `insertThickness >= wallHeight`. A 34.99 mm insert in a 35 mm wall remains valid with the existing shallow-positive warning; 35 and 40 mm block. Disabled inserts continue to ignore stale insert-thickness values.

## 6. Assembly-label correction

DT1 now reads the same `assemblyLabels` setting as other Designs. Disabled output has no assembly-label group or paths. Enabled output conditionally labels existing BASE, walls, STORAGE-DIVIDER, wood INSERT, and BOTTOM-COVER components. Non-wood liners receive no INSERT label. Labels use the green non-cut group and remain outside the red structural cut group. `metrics.labelCount` equals parsed emitted label paths.

## 7. Fixture assertions added

`runDiceTraySystemFixtures()` grew from 51 to **92 passed / 0 failed**. New direct coverage includes:

- legacy Dice Tray length/hash pins and alternate/Divider pins;
- 20 mm threshold boundaries;
- actual physical panel bounding-box overlap and spacing;
- unique closed cut paths;
- insert height below/equal/above wall height;
- stale disabled insert thickness;
- label toggle, conditional labels, non-wood behavior, and count parity;
- parsed Finished View rectangle finiteness, positive dimensions, left/right mirroring, insert containment, no lid, and production-ID separation;
- exact-one `selftest=all` route registration.

`runTrayModelFixtures()` was not altered.

## 8. Fixture cleanup

The focused group uses pure generated drafts/models and disposable DOM parsing. It does not modify `designDraft`, preview mode, active tab, units, forms, modal state, localStorage, project records, or final rendered DOM. No synthetic tray data remains. A temporary debug hook used during diagnosis was removed before validation.

## 9. Exact totals

- Dice Tray System: **92/0**.
- Complete suite: **2454/0 across 29 groups**; all route groups completed with zero failures. This is the independently verified 2362 baseline plus the measured 92-assertion focused group.
- Designs geometry: **1093/0**.
- Gift Box: **69/0**.
- Tray standalone: **264/0**.
- M1: **29/0**.
- M2: **31/0**.
- M3: **27/0**.
- Promotion-switch standalone: **16/0**.

## 10. Layout non-overlap evidence

The focused fixtures generated physical component boxes for wood+cover, storage+wood+cover on both sides, divider+cover, and liner+cover. Every pair remained separated; the normal 15 mm panel gap remained honored. Corrected DT1 production SVG dimensions update honestly for the added strip. No insert dimensions or storage formulas changed.

## 11. Legacy hash comparison

The pre/post representative capture is unchanged for legacy default Dice Tray: 1726 bytes, FNV `51a55721`, SHA-256 `2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf`. Alternate-joint Dice Tray remains pinned at 1054 / `41697123`; Divider Tray remains 1965 / `a55dda6e`.

## 12. Existing-template comparison

The following pre/post lengths and SHA-256 values were unchanged: Gift Box 3210 / `5302a64cba33b05e03ea646f9119f6545e4bebf15685c6f7a95df58b6c6894d9`, Finger Box 2483 / `62cc24a3055852797a4f23c9cb224da5f9d3c76fdc5cefc428e90df3ea0d410e`, Sliding-lid Box 2818 / `5eb28ea7dfcaeb02fa3ecd9c5a4e8e6169453e2b1869fc92e8306e2d284bdb15`, Drawer Cabinet 4456 / `21739161b3f8719f711f95ca9331943ef6b7dec92c577620963661fd0aec6a28`, Joint Fit Coupon 7764 / `e4f03deb042a52e5f08389fcd1e780bb5fc6bee511a1a7600ae2f09a03f7263d`, and Wall/base tab coupon 1551 / `e5758d7104573489428651b0dcfda3f2281ae2dc1aa5292974a30c0aaf8b3ae0`.

## 13. Direct `file://` results

Direct, production, designs, gift-box, machines-m3, dice-tray-system, and all routes opened in headless Edge without page exceptions. HTML parsing and `git diff --check` passed. Browser console and duplicate-ID scans were clean.

## 14. Disposable-browser workflow

The disposable browser exercised right and left storage, same-thickness wood insert, underside cover, wood without storage, threshold values, insert heights, label disabled/enabled, cork/leather/felt/generic measurement-only behavior, Finished View storage sides, no-lid behavior, cover toggling, deterministic repeated generation, Divider Tray, Gift Box, and Sliding-lid Box. No real browser profile or records were used.

## 15. Documentation updates

Updated `README.md`, the DT1 implementation report, and this correction report. `CHANGELOG.md` retained the existing accurate DT1 entry; no unrelated feature documentation was changed.

## 16. Protected-boundary comparison

APP constants, storage/schema/backup behavior, machines M1/M2/M3, promotion, Project schema, accounting, Inventory, Pricing, Divider Tray semantics, legacy Dice Tray bytes, Gift Box/Finger Box/Sliding-lid/Drawer/Coupon bytes, kerf conventions, colors, filenames, import/export, and offline behavior remain unchanged.

## 17. Actual files changed

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/DESIGNS_DICE_TRAY_DT1_IMPLEMENTATION_2026-07-20.md`
- `docs/DESIGNS_DICE_TRAY_DT1_CORRECTION_2026-07-20.md`

The working tree also contains pre-existing unrelated untracked files; they were not edited.

## 18. Git diff summary

The tracked implementation delta remains limited to the three intended source/document files. `git diff --check` passes. Nothing is staged, committed, or pushed.

## 19. Remaining unverified areas

No physical material, kerf, squareness, glue strength, liner composition, adhesive behavior, machine-bed fit, or LightBurn operation assignment was physically proven. No automatic nesting or multi-material export was added.

## 20. Physical prototype readiness

Before production, run the wall/base fit coupon, a small legacy no-storage tray, a storage divider dry-fit, and a 100%-scale wood insert+cover inspection. Measure liners from the assembled cavity. Verify composition before any non-wood laser process. This software correction does not establish physical readiness.

## 21. Audit and phase decision

Physical laser testing is required. Narrow Grok re-verification is sufficient for this bounded correction before commit. DT1 may be committed after review; Hex Dice Tray architecture may begin only after the DT1 commit and physical scrap testing. DT2 remains deferred.

## 22. Final status

The confirmed overlap blocker and four important audit findings are corrected. No stage, commit, push, reset, clean, stash, checkout, move, rename, or delete operation was performed.

