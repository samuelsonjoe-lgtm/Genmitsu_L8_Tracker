# Designs Gift Box G1 focused-audit correction report

Date: 2026-07-19  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `769069d5a3c8bc2e74d2da912cd2257c10288f96` (`main`)

## 1. Repository state

Initial inspection confirmed `main`, HEAD `769069d`, `origin/main...main = 0 0`, no staged files, and existing Gift Box work uncommitted. `git diff --check` was clean before correction. The architecture review, implementation report, and focused audit were present. Existing unrelated LightBurn Projects, historical reports, debug.log, utility scripts, and other untracked files were preserved.

## 2. Confirmed defects reproduced

The audit defects were reproduced in source: `lidPts` used `[x,y]` arrays while `poly()` required `{x,y}` objects; the open state uniformly shrank/repositioned the lid and claimed an approximately 90-degree opening; fixtures only checked captions/attributes; the Gift Box label list unconditionally attempted HINGE-COUPON and frame side labels; and the dead duplicate Gift Box form branch remained.

## 3. Lid point-shape correction

`buildGiftBoxFinishedViewSvg()` now passes object-shaped points to `poly()`. The focused fixtures parse both states, reject NaN/Infinity/undefined, require at least four finite points, and require positive polygon area. Live UI extraction produced finite closed points `72,164.2032 220.3968,164.2032 258.815712,125.784288 110.418912,125.784288` and finite open points `302,116 450.3968,116 488.815712,77.581088 340.418912,77.581088`.

## 4. Lift-off Closed behavior

Closed lift-off lids are now full-size horizontal projected panels seated over the shell footprint. Flush mode follows the shell footprint; overhanging mode changes the projected lid bounds. The LID label is computed from the visible lid bounds. Frame rails use the same lid projection and are rendered beneath the closed lid so they do not float independently.

## 5. Lift-off Open behavior

Open lift-off lids remain full-size and are shown aside using a separate horizontal projection. They have no hinge axis and are labeled “Lift-off lid removed aside.” The caption and description no longer imply a 90-degree opening. All four locating rails use the lid’s aside projection and move with it.

## 6. Hinged Closed behavior

Closed hinged lids are horizontal and seated over the shell. The rear edge is the conceptual hinge axis, with rear overhang still fixed at zero. Schematic hinge markers are generated from the same projected lid axis and remain spatially near the rear wall/lid relationship.

## 7. Hinged Open rotation

Open hinged lids now project a real rotation about the rear top hinge axis using a bounded approximately 78-degree schematic rotation. The hinge-side edge remains fixed while the opposite edge moves and changes orientation. The lid is not uniformly scaled or translated away. The view caption reads “Hinged lid approximately 90 degrees open,” while the description identifies the rear-axis rotation as schematic.

## 8. Locating-frame visualization

Frame rails are represented as four separate polygons derived from the lid’s own projected coordinate system. Closed rails are mostly hidden under the lid; open lift-off rails visibly travel with the removed lid. Hinged frame-plus-hardware validation remains blocked and was not broadened.

## 9. Label positioning

The LID label is centered from the actual rendered lid bounds. FRONT and BACK remain tied to understandable shell surfaces. Hinge markers use the projected axis/leaf points rather than the former front-top shortcut. No leader lines or unsafe exterior labels were introduced.

## 10. Conditional assembly-label specs

Gift Box label specs now always include only the structural panels and LID, add FRAME-FRONT/BACK only when the frame is enabled, and add HINGE-COUPON only for hinged mode with the coupon enabled. FRAME-LEFT and FRAME-RIGHT are intentionally omitted from attempted long labels because the rails are identical and interchangeable; the general safe-label algorithm was not changed.

## 11. Warning/message changes

The default lift-off configuration no longer produces a missing HINGE-COUPON warning or FRAME-LEFT/FRAME-RIGHT omission warnings. Genuine missing/unsafe label warnings remain available for other required labels. Gift Box assembly guidance now explains that the frame side rails are identical and interchangeable, while the result’s emitted label count remains authoritative.

## 12. Closed-height summary change

Gift Box result summaries now show `Closed overall height` using `metrics.closedHeight` and the existing millimeter formatter. No geometry or dimension formula changed.

## 13. Dead-branch cleanup

Removed the confirmed unreachable duplicate Gift Box ternary branch in `renderDesigns()`. The broader form-selection chain was not refactored.

## 14. Fixture assertions added

Added coordinate-level checks for finite/nondegenerate closed and open lids, closed lift-off overlap, full-size lift-off removal, closed hinged seating, fixed-axis hinged opening, open/closed caption semantics, frame movement with the lift-off lid, conditional label targets, warning absence, label count, and production/Finished View separation.

## 15. Fixture cleanup

The fixture restores the session draft around summary rendering and uses actual DOMParser coordinate data rather than source-string-only checks. It does not write storage or mutate persisted records.

## 16. Exact focused and complete totals

- Previous Gift Box focused total: **60 passed / 0 failed**.
- Final Gift Box focused total: **69 passed / 0 failed**.
- Existing Designs geometry: **1093 passed / 0 failed**.
- Complete suite: **2362 passed / 0 failed**, reconciled as the prior 2353 plus 9 correction assertions, across **28 groups**.
- M1: **29 / 0**.
- M2: **31 / 0**.
- M3: **27 / 0**.
- Tray standalone: **264 / 0**.
- Promotion-switch standalone: **16 / 0** (the promotion route remains green; its nested target-switch assertions are not double-counted).

## 17. Production SVG before/after comparison

The correction is isolated to Finished View projection, Gift Box label-spec selection, presentation text, and fixtures. A reconstructed pre-correction source copy with the original unconditional Gift Box label-spec list was loaded in a disposable Edge page and compared byte-for-byte with the corrected page. Every representative output matched: Gift lift-off `5302a64cba33b05e03ea646f9119f6545e4bebf15685c6f7a95df58b6c6894d9` (3210 bytes), Gift hinged `558b9af6e1898dc023b742a4e60a418000d4675496c4de003fa92a58b085dda4` (5490), Finger Box `62cc24a3055852797a4f23c9cb224da5f9d3c76fdc5cefc428e90df3ea0d410e` (2483), Sliding Lid `5eb28ea7dfcaeb02fa3ecd9c5a4e8e6169453e2b1869fc92e8306e2d284bdb15` (2818), Drawer Cabinet `21739161b3f8719f711f95ca9331943ef6b7dec92c577620963661fd0aec6a28` (4456), Tray `2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf` (1726), Joint Fit Coupon `e4f03deb042a52e5f08389fcd1e780bb5fc6bee511a1a7600ae2f09a03f7263d` (7764), and Wall/base coupon `e5758d7104573489428651b0dcfda3f2281ae2dc1aa5292974a30c0aaf8b3ae0` (1551). All eight `same` comparisons were true; no output contained Finished View IDs.

The label-spec filtering removes only attempted labels for absent or intentionally interchangeable panels; the final emitted production label paths and bytes are unchanged.

## 18. Existing-template regression comparison

The existing Designs geometry group remained **1093/0**. Direct UI captures for Finger Box, Sliding-lid Box, Drawer Cabinet, Dice Tray, Joint Fit Coupon, and Wall/base coupon all produced valid red/blue production SVGs without Finished View IDs. No existing filename, storage, import/export, machine, Project, accounting, Inventory, Pricing, or kerf code was modified.

## 19. Direct file:// results

Headless Edge direct `file://` runs completed without page exceptions for `?selftest=gift-box` (69/0), `?selftest=design` (1093/0), `?selftest=machines-m3` (27/0), and `?selftest=all` (all groups zero failures; 2362 reconciled total). `git diff --check` and `python -m html.parser index.html` passed.

## 20. Live user-scenario reproduction

A disposable Edge page used the Designs DOM workflow with Gift Box, internal 120 × 90 × 50 mm, 2.88 mm material, lift-off, flush, frame enabled, and labels enabled. Cut Layout, Finished Closed, and Finished Open rendered without page errors. Closed/open screenshot buffers were captured in memory (64,845 and 64,853 bytes); the extracted lid points were finite and distinct. The live result contained no HINGE-COUPON or FRAME-LEFT warnings. Hinged flush/overhanging and one/two/three-hinge/coupon combinations were covered by the focused generator fixtures; hinged open showed fixed-axis point behavior.

## 21. Documentation changes

Updated README totals and Gift Box wording, corrected the existing implementation report with a follow-up note, and added this correction report. CHANGELOG now records the Finished View, coordinate-fixture, and conditional-label corrections. The focused audit report was not edited.

## 22. Protected-boundary comparison

APP_ID, APP_NAME, APP_VERSION, BUILD_DATE, STORAGE_KEY, SCHEMA_VERSION, BACKUP_FORMAT, storage, machines, machine identity, promotion, Project schema, accounting, Inventory, Pricing, existing Design geometry, kerf convention, LightBurn colors, existing filenames, import/export, backup behavior, and offline/no-network behavior remained outside the correction scope.

## 23. Actual files changed

Tracked files: `index.html`, `README.md`, and `CHANGELOG.md`. New documentation: this report. The existing G1 implementation report was updated. All other untracked files were preserved.

## 24. Git diff summary

At handoff, the tracked diff was limited to the three intended application/documentation files; the correction report and prior reports remain untracked documentation artifacts. `git diff --stat` reported the existing phase plus this correction as 309 insertions and 28 deletions across the three tracked files. No staged diff exists.

## 25. Remaining unverified areas

No physical cut, glue-up, hinge drilling, frame registration, material-strength, screw blowout, or LightBurn-machine execution was performed. The schematic projection is not a physical fit or hardware guarantee. A persisted pre-correction SVG snapshot was unavailable, as noted in §17.

## 26. Physical prototype readiness

The correction is ready for the next physical validation step, not a claim of production proof. Start with measured-stock finger-fit scrap, a locating-frame corner scrap, and the existing hinge placement coupon before cutting a complete box.

## 27. Whether physical laser testing is required

Physical laser testing is not required as a consequence of these corrections because they are screen-only rendering, warning-copy, and fixture changes. Physical testing remains required before normal Gift Box production under the pre-existing G1 safety plan.

## 28. Whether another full audit is needed

No second full Gift Box audit is needed for this bounded correction. The original audit’s non-correction findings were preserved.

## 29. Whether narrow re-verification is sufficient

Yes. The focused route, existing Designs route, M1/M2/M3, complete suite, direct UI workflow, coordinate parsing, page exceptions, and production isolation checks provide sufficient narrow re-verification.

## 30. Whether Gift Box G1 may be committed

Yes, the corrected G1 phase is eligible for a deliberate commit after user review. This run did not create a commit.

## 31. Final git boundary

Nothing was staged, committed, or pushed. No reset, clean, stash, checkout, move, rename, or delete operation was performed.
