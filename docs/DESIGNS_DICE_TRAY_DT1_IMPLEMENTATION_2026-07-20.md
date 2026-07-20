# Designs Dice Tray DT1 Implementation — 2026-07-20

DT1 remains an additive extension of `dice-tray`. Legacy no-storage/no-insert Dice Tray output continues through the original path; Divider Tray semantics and `runTrayModelFixtures()` remain unchanged.

## Current contract

- Storage clear width hard minimum: **20 mm**.
- 20 mm through below 28 mm is valid with a retrieval warning; 28 mm and above has no narrow-retrieval warning.
- Storage is left/right full-depth only, with one fixed divider and a through-cut base slot.
- Insert footprint is rolling clear width/depth minus twice per-side clearance.
- Insert thickness must be below wall height. Equal or greater thickness blocks; positive remainder below 12 mm warns.
- Cork, leather, felt, and generic liners remain measurement-only. Same-thickness wood panels may be generated.
- DT1 labels now honor the Assembly Labels setting and use the established green non-cut label group. The emitted count is the actual emitted label-path count.
- The underside cover is placed after the lowest physical DT1 panel strip plus the normal panel gap. Measurement-only liners do not reserve an empty physical strip.

## Validation before correction

The focused audit reproduced overlapping production bounding boxes when a generated wood insert and underside cover were both enabled. It also identified the 18 mm threshold, missing insert-height block, unconditional DT1 labels, and insufficient direct focused coverage.

Repository state at inspection: `main`, `HEAD=294cc064f82020ce02ca95e2ced997f003715ba9` (`294cc06`), `origin/main...main=0 0`, no staged changes. Existing untracked LightBurn projects, historical reports, `debug.log`, and utility script were preserved.

## Correction results

The cover layout now derives `dt1PanelBottomMm` from the actual insert/divider strip and places the cover at that edge plus the existing gap. Validated combinations include wood plus cover, right/left storage plus wood plus cover, divider plus cover, wood without cover, cover without insert, and measurement-only liner plus cover. Physical material-component bounding boxes are non-overlapping and spacing remains honored.

The storage form, normalization, model validation, help copy, and focused fixtures all use 20 mm. Direct cases 17.9, 18, 19, and 19.99 block; 20 and 27.9 warn; 28 does not warn. The default enabled width remains 42 mm.

Insert height validation applies to every enabled insert type. `34.99 < 35` remains valid with the shallow warning; `35 == 35` and `40 > 35` block. Disabled inserts normalize thickness away, so stale disabled thickness does not invalidate a tray.

DT1 label candidates are generated only when `assemblyLabels` is enabled. BASE, FRONT, BACK, LEFT, RIGHT, STORAGE-DIVIDER, INSERT, and BOTTOM-COVER are conditional on the corresponding physical component and safe glyph placement. Non-wood liners never get an INSERT label. Green labels are outside the red cut group.

## Focused fixtures

`runDiceTraySystemFixtures()` now reports **92 passed / 0 failed**. Direct assertions cover legacy pins, all threshold boundaries, actual physical bounding-box non-overlap and gap spacing, closed/unique cut paths, insert height, stale-disabled thickness, label toggle/count/conditionality, parsed Finished View coordinates and mirror behavior, no-lid semantics, production/Finished View ID separation, deterministic output, handoff, and one-time `selftest=all` registration. `runTrayModelFixtures()` was not altered.

## Protected output comparison

Representative production outputs were captured before and after correction. The following remain byte-identical:

| Template | Length | FNV fixture hash | SHA-256 |
|---|---:|---|---|
| Legacy Dice Tray | 1726 | `51a55721` | `2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf` |
| Alternate-joint Dice Tray | 1054 | `41697123` | pinned by focused fixture path |
| Divider Tray | 1965 | `a55dda6e` | `91a5fcaaa54a63d9c5f9b3e9851de70b62fdd98798d356d50977ed4f46f1d924` |
| Gift Box | 3210 | `06adafd3` | `5302a64cba33b05e03ea646f9119f6545e4bebf15685c6f7a95df58b6c6894d9` |
| Finger Box | 2483 | `a892f91c` | `62cc24a3055852797a4f23c9cb224da5f9d3c76fdc5cefc428e90df3ea0d410e` |
| Sliding-lid Box | 2818 | `468e9fd8` | `5eb28ea7dfcaeb02fa3ecd9c5a4e8e6169453e2b1869fc92e8306e2d284bdb15` |
| Drawer Cabinet | 4456 | `a6dd23dc` | `21739161b3f8719f711f95ca9331943ef6b7dec92c577620963661fd0aec6a28` |
| Joint Fit Coupon | 7764 | `db7ea7e9` | `e4f03deb042a52e5f08389fcd1e780bb5fc6bee511a1a7600ae2f09a03f7263d` |
| Wall/base tab coupon | 1551 | `d9ffc278` | `e5758d7104573489428651b0dcfda3f2281ae2dc1aa5292974a30c0aaf8b3ae0` |

The alternate-joint Dice Tray FNV and length remain pinned by the focused fixture and existing tray suite; its SHA-256 was not retained in the pre-correction capture. DT1-only layouts changed only where the new physical panel placement or label toggle requires a change.

## Verification

- `?selftest=dice-tray-system`: **92/0**.
- `?selftest=all`: all registered groups completed with zero failures; complete-suite total is **2454 passed / 0 failed across 29 groups** (prior independently verified 2362 plus the measured 92-assertion correction group).
- Designs geometry: **1093/0**.
- Gift Box: **69/0**.
- Tray standalone: **264/0**.
- M1: **29/0**; M2: **31/0**; M3: **27/0**.
- Promotion-switch standalone: **16/0**.
- Direct `file://` routes `production`, `designs`, `gift-box`, `machines-m3`, `dice-tray-system`, and `all`: no page exceptions.
- HTML parser, `git diff --check`, browser console, duplicate-ID scan, responsive, modal, general accessibility, beginner clarity, Help, storage/recovery, handoff, and the protected template fixture paths passed.

## Files changed

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/DESIGNS_DICE_TRAY_DT1_IMPLEMENTATION_2026-07-20.md`
- `docs/DESIGNS_DICE_TRAY_DT1_CORRECTION_2026-07-20.md`

No unrelated files were edited. Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted. Physical laser proof remains required; use the wall/base coupon, a small legacy tray, divider dry-fit, 100%-scale wood insert/cover inspection, measured liner sizing, and composition verification before any non-wood laser process. DT2 remains deferred. A narrow Grok re-verification is sufficient before commit; DT1 may be committed after review, and Hex Dice Tray architecture may begin only after that commit and physical scrap testing.

