# Tabletop Accessories T1 Engine + Coupon Implementation

Date: 2026-07-20  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `7876551 Add Dice Tray storage and insert options`  
Authority: `DESIGNS_TABLETOP_ACCESSORIES_CONSTRUCTION_CONTRACT_REVIEW_2026-07-20.md`

## 1. Request and acceptance boundary

Implemented the bounded T1 shared engine groundwork and the user-facing Tabletop Corner + Floor Coupon. Existing production templates remain outside the new engine path.

## 2. Source of truth

The construction-contract review governs the rectangle, hex groundwork, authoritative cavity, joint terminology, physical-readiness, and protected-output decisions.

## 3. Repository baseline

The working tree started at synchronized HEAD `7876551`; no tracked edits were present before T1 implementation.

## 4. Scope implemented

Added one registry entry, prefixed tabletop helpers, one coupon builder, one screen-only Finished Assembled view, two focused fixture routes, Help/README/CHANGELOG wording, and this report.

## 5. Scope intentionally not implemented

No complete premium rectangle tray, user-facing hex product, lid, divider, insert, rebate, groove, pocket, layered channel, schema migration, cloud path, or production claim was added.

## 6. User-facing template

Template ID: `tabletop-corner-floor-coupon`. Display name: `Tabletop Corner + Floor Coupon`. It is grouped with the existing coupon/prototype selector while its option identifies Tabletop Accessories; the registry retains the explicit family name without changing the four-optgroup selector contract.

## 7. Inputs

The form exposes units, measured material thickness, center clearance, clearance step, preferred finger width, requested interior leg, wall height, panel spacing, assembly labels, and a kerf reference. The first focused audit found that the runtime ternary selected a note-only branch, so this statement was initially inaccurate; the correction pass wires the actual controls and adds direct form-presence coverage.

## 8. Defaults

Defaults are 3 mm material, 0.10 mm center clearance, 0.05 mm step, 9 mm preferred finger width, 40 mm interior leg, 22 mm wall height, 15 mm panel spacing, and zero kerf reference. Clearance step and panel spacing must now be positive.

## 9. Candidate order

Exactly three candidates are emitted in deterministic order: center-step, center, and center+step. With defaults they are 0.05, 0.10, and 0.15 mm.

## 10. Physical coupon composition

Every candidate contains exactly WALL-A, perpendicular WALL-B, and FLOOR. Across three candidates the production result contains exactly nine physical pieces.

## 11. Semantic IDs

Panel IDs are `C1-WALL-A`, `C1-WALL-B`, `C1-FLOOR` through `C3-*`. Finished-view identifiers do not enter production SVG.

## 12. Rectangle contract

The coupon uses existing box geometry for a square rectangular corner: four-wall-compatible finger mathematics is reused, two perpendicular walls are selected, and the bottom panel provides the finger-jointed floor connection.

## 13. Corner construction

The result metadata records `finger-90` wall-to-wall construction and opposite vertical wall phases. No generic non-90-degree finger solver was introduced.

## 14. Floor construction

The result metadata records `finger-jointed-floor` and both selected wall-to-floor connections. The floor is a single flat through-cut panel.

## 15. Existing finger math

`buildFingerPattern` remains the selection authority. T1 does not rewrite odd-count selection, width calculation, or terminal-web math.

## 16. Manufacturing limits

T1 applies `max(2t, 4)` as the minimum segment and retains a positive minimum web. Preferred widths below the manufacturable minimum and impossible patterns are blocked.

## 17. Shared rectangle outline

`tabletopAccessoryRectangleOutline` returns deterministic ordered vertices and dimensions.

## 18. Shared regular hex outline

`tabletopAccessoryHexagonOutline` records side = flat-to-flat / sqrt(3) and point-to-point = 2 * flat-to-flat / sqrt(3), with deterministic flat-oriented vertices.

## 19. Wall loop descriptor

`tabletopAccessoryWallLoop` returns wall ID, ordinal, endpoints, length, direction, outward normal, previous/next links, angle reference, height, and join metadata.

## 20. Corner descriptors

`tabletopAccessoryCornerDescriptors` supports `finger-90` and the groundwork `floor-registered-butt-120` reference without pretending to solve a 120-degree finger joint.

## 21. Hex status

Hex geometry and six-edge ordered-loop groundwork are present only as shared non-user-facing primitives. No hex product selector or renderer is exposed.

## 22. Dimension descriptor

`tabletopAccessoryDimensions` separates requested interior, finished cavity, outer envelope, panel dimensions, and cut bounds. In interior-primary mode it blocks a smaller finished cavity unless explicit allowed-intrusion metadata is supplied; it no longer silently shrinks the cavity.

## 23. Authoritative finished cavity

`tabletopAccessoryFinishedCavity` is the source descriptor for clear bounds, polygon, usable height, source dimension mode, wall thickness, floor relation, and intrusions.

## 24. Divider primitive

`tabletopAccessoryDividerFromCavity` derives span from finished cavity clear bounds and explicit end clearance. It does not read raw nominal dimensions.

## 25. Insert primitive

`tabletopAccessoryInsertFromCavity` derives the insert region from finished cavity bounds. Physical true/false is the only behavior branch; material tags remain opaque. Physical true records cut-panel intent and false records measurement-only intent.

## 26. Physical regression

The focused engine fixture uses a 96.73 mm finished cavity against a 100 mm nominal source and confirms the derived divider and insert do not use 100 mm.

## 27. Projection helper

`tabletopAccessoryProjectPoint` validates x/y/z and projection parameters, converts x/y/z deterministically, and rejects malformed points before SVG assembly-view markup.

## 28. Production serializer

T1 uses a new serializer wrapper with red cut paths and optional green assembly-label paths. Legacy `serializeDesignSvg` behavior is untouched.

## 29. Production SVG contract

The coupon output uses millimeter width/height/viewBox, closed panel paths, deterministic panel transforms, semantic panel IDs, no finished-view IDs, and no NaN/Infinity values.

## 30. Labels

Labels are optional, concise, deterministic vector paths and are emitted in `assembly-labels` with green stroke. They are assembly aids, not decorative or structural geometry.

## 31. Finished Assembled view

The screen-only view displays all three candidate corners using the shared projection helper. It is not persisted, downloaded, or interpreted as physical proof.

## 32. Project handoff

Generic handoff names the coupon and records candidate clearances and the construction-coupon caveat without adding a schema field.

## 33. Help and safety wording

The form, Help modal, and README describe same-stock testing, dry fitting, squaring, manual recording, exact-scale import, kerf separation, and the non-production nature of T1. Help now has a discoverable Tabletop Accessories T1 section with construction, future-hex, and laser-safety boundaries.

## 34. Fixture routes

`?selftest=tabletop-engine` runs the shared engine group. `?selftest=tabletop-corner-floor-coupon` runs the coupon group. Each route is registered once, and both run once under `?selftest=all`.

## 35. Engine fixture coverage

The engine group covers rectangle and hex formulas, ordered loops, corner types, dimensions, cavity authority, divider/insert derivation, the 96.73 regression, manufacturing minimums, projection finiteness, and malformed-point rejection.

## 36. Coupon fixture coverage

The coupon group covers registry/defaults, candidate ordering, nine pieces, semantic IDs, construction metadata, colors, absence of Finished View IDs, SVG validation, invalid preferred width, impossible dimensions, and handoff naming.

## 37. Direct focused validation

Focused engine result after correction: 27 passed, 0 failed. Focused coupon result after the result-summary correction: 75 passed, 0 failed. Help and Safety result: 40 passed, 0 failed.

## 38. Existing Designs validation

Designs geometry result after T1 selector registration update: 1093 passed, 0 failed.

## 39. Complete validation

The aggregate browser run completed with all existing groups passing plus the corrected T1 groups. The measured unique-suite total is 2559 passed, 0 failed across 31 groups. The summary correction added 8 coupon assertions to the previously measured 2551 total.

## 40. Direct-open validation

`index.html` was parsed with Python's HTML parser and opened through the local browser fixture harness. No network, module, cloud, or Tauri dependency was added.

## 41. Diff hygiene

`git diff --check` is clean. Existing untracked LightBurn projects, historical reports, debug.log, and the utility generator were preserved.

## 42. Protected templates

Existing Finger Box, Sliding-Lid Box, Drawer Cabinet, Gift Box, Dice Tray, Divider Tray, Joint Fit Coupon, and legacy stand/sign paths do not dispatch through the T1 builder.

## 43. Protected SVG boundary

No existing template source geometry, production SVG serializer, filenames, storage keys, localStorage schema, or historical record is intentionally changed by T1.

## 44. Physical readiness

T1 is ready for bounded scrap-coupon testing only. It is not proof of fit, square, strength, glue quality, kerf accuracy, material safety, clamp access, or production success.

## 45. Commit boundary

No commit, push, reset, clean, stash, or destructive operation was performed.

## 46. T2 recommendation

T2 should consume the authoritative finished-cavity descriptor and validated coupon results before any complete rectangle product is considered. A divider/insert product should remain separate and bounded.

## 47. Hex recommendation

Keep hex as groundwork until a separate construction contract defines `floor-registered-butt-120`, floor registration, layout, validation, and physical testing. Do not add a generic 120-degree finger solver by implication.
