# Designs Gift Box G1 implementation report

Date: 2026-07-19  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `769069d5a3c8bc2e74d2da912cd2257c10288f96` (`main`)

## 1. Scope and protected boundaries

Implemented the isolated `gift-box` Designs template requested by the G1 brief. No storage key, schema version, backup format, Project schema, machine identity behavior, existing template contract, or external dependency was changed. No stage, commit, push, reset, clean, stash, checkout, move, rename, or delete operation was performed. Existing unrelated untracked files, including the Gift Box architecture review, remain untouched.

## 2. Initial repository state

The initial inspection confirmed branch `main`, HEAD `769069d5a3c8bc2e74d2da912cd2257c10288f96`, `origin/main...main = 0 0`, no tracked modifications, no staged files, and a clean `git diff --check`. The unrelated untracked inventory was preserved. Release constants remained `genmitsu-l8-tracker`, `Genmitsu L8 Tracker`, `0.9.0`, `2026-07-19`, `genmitsu-l8-tracker-v1`, schema `2`, and backup format `genmitsu-l8-tracker-backup-v1`.

## 3. Changed files

- `index.html` — Gift Box defaults, registry, form, normalization, geometry, validation, guide layers, Finished Views, results, filename, handoff, Help copy, fixtures, and route registration.
- `README.md` — Gift Box description, focused route/function, and reconciled fixture totals.
- `CHANGELOG.md` — 0.9.0 Gift Box G1 entry.
- This report.

## 4. Registry and defaults

Registered `gift-box` under Boxes and cabinets as `Gift Box`. The template has explicit session-only defaults for units, internal dimensions, one measured thickness, kerf, panel spacing, joint adjustment, finger target, lid type/appearance/clearance, overhangs, locating frame, hinge dimensions, screw pattern, coupon, front clasp, magnets, and assembly labels. Finger Box was not given a hinged mode.

## 5. Form and accessibility

The Gift Box form has Units, internal/external finished dimension mode, finished length/width/height, material thickness, kerf, joint fit, preferred finger width, panel spacing, lid type, lid appearance, lid clearance, overhang controls, locating-frame controls, assembly labels, and hinged-only hardware controls. Lift-off hides hinge controls; flush hides overhang controls; hinged mode exposes measured hinge and screw fields plus coupon/front-guide options. Help text explains measurement, scrap testing, blue guide semantics, manual drilling, LightBurn layer assignment, framing, ventilation, and fire watch.

## 6. Dimension contract

Internal mode preserves usable inside length, width, and height and derives outside shell dimensions from the measured wall/base thickness. External mode treats the entered closed shell dimensions as literal outside dimensions. The lid adds its structural stack to reported closed height. Lid footprint is reported separately. Frame clear opening and the per-side cavity reduction are exposed in metrics and the result summary. G1 remains single-thickness; dual thickness is deferred.

## 7. Structural geometry

The generator reuses `buildBoxModel`, `designBoxDimensions`, `buildFingerPattern`, `buildFingerPanel`, panel geometry validation, existing row layout/nesting, score-path serialization, and assembly-label helpers. It emits BASE, FRONT, BACK, LEFT, RIGHT, and LID panels with exposed finger joints, deterministic ordering, closed contours, explicit panel spacing, and optional four separate FRAME-FRONT/BACK/LEFT/RIGHT rails. Structural red paths remain separate from blue guide paths.

## 8. Lid modes

Lift-off lids support flush and front/left/right overhang. Rear overhang is fixed at zero. The locating frame is four independently selectable rails and is blocked when its opening is nonpositive, rails are too narrow, or its geometry is otherwise invalid. Hinged lids are surface-mounted only; G1 does not create mortises, pockets, recesses, barrel cuts, or screw holes.

## 9. Hinge and hardware guides

One, two, and three hinges are supported. One hinge is centered; two and three use symmetric corner-inset placement with a conservative minimum inter-leaf gap. Blue guides contain leaf outlines and screw-center crosses on the rear wall and lid. Screw patterns are centered within each leaf. Screw centers warn below 6 mm edge clearance and block below 3 mm. Invalid counts, lengths, bounds, patterns, nonpositive gaps, and locating-frame/hinge conflicts block output. Entered barrel/gap relationships produce a conservative warning where exact hardware collision cannot be proven.

## 10. Coupon and optional front guides

The hinged-only placement coupon uses the actual entered leaf and screw pattern on a separate cut panel. It does not change box dimensions and can be disabled. Front clasp and magnet options add simple blue center guides only; no recesses or hardware-size assumptions are generated.

## 11. Validation and warnings

Blocking validation covers nonfinite/nonpositive dimensions, invalid material thickness, invalid finger geometry, negative derived cavity, invalid hinge layout, out-of-bounds leaves, screw pattern overflow, edge-distance violations, zero hinge gap, frame conflicts/opening/rail violations, duplicate translated structural segments, open/nonfinite panel geometry, and SVG validation failures. Existing large-panel warnings and the Gift Box fit/physical-validation warnings remain non-blocking.

## 12. Production SVG and layout

Production SVG remains exact-scale, deterministic, offline, and LightBurn-importable. Existing red cut and blue score conventions are preserved; screen-only Finished View markup is never serialized into production SVG. The existing layout/nesting system handles the shell, lid, frame rails, and coupon with the entered panel spacing. Gift Box exports use `gift-box-<outside-width>-<outside-depth>-<lid-type>.svg`-style sanitized names.

## 13. Finished Views

Added session-only Finished Closed and Finished Open modes. The closed view is labeled as a three-quarter view; the open view identifies the lid as approximately 90 degrees. Both show schematic finger cues, lid appearance, frame presence, and surface hardware where applicable. They are not included in downloads and do not imply exact hardware thickness, screw depth, fit, strength, or production safety.

## 14. Result summary and Project handoff

Gift Box summaries report lid type/appearance, internal and external dimensions, lid footprint, material/fit/kerf, lid-clearance starting value, frame details and clear opening, hinge dimensions and blue-guide semantics, coupon state, piece count, and labels. The existing generic `projectDraftFromDesign` path now names Gift Box designs and adds concise lid/coupon notes without changing Project persistence or schema.

## 15. Help and documentation

In-app Help now includes a Gift Box G1 safety section. README documents the template and focused route. CHANGELOG records the bounded feature and explicitly notes that guide geometry is not mortise or screw-hole geometry.

## 16. Focused fixtures

Added `runGiftBoxFixtures()` with 60 assertions and registered it exactly once for `?selftest=gift-box` and once under `?selftest=all`. Coverage includes registration/defaults, internal/external dimensions, units, one-thickness behavior, flush/overhanging lids, frame rails/opening/cavity loss, one/two/three hinges, guide layers, coupon behavior, screw centering and safety, invalid-input blocking, deterministic geometry, production SVG separation, Finished Views, labels, optional front guides, and Project handoff.

## 17. Regression verification

- `?selftest=gift-box`: **60 passed / 0 failed**.
- `?selftest=machines-m3`: **27 passed / 0 failed**.
- `?selftest=design`: **1093 passed / 0 failed**.
- `?selftest=all`: all observed fixture groups reported zero failures; reconciled complete total **2353 passed / 0 failed** across **28 groups** (`2293` baseline + `60` Gift Box).
- Headless Edge direct `file://` loading produced no page exceptions for the focused, M3, Designs, or all routes.
- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.

The browser console may retain the repository's existing SVG `height="auto"` warning on unrelated preview markup; it did not produce page exceptions or fixture failures.

## 18. Physical validation plan and deferrals

Before production, measure actual stock and hinges, run a finger-fit scrap test, run the hinge placement coupon, verify LightBurn colors/operations at 100% scale, frame the job, and test screw drilling on scrap. G1 intentionally defers dual thickness, decorative skins, corner keys, faux dovetail overlays, hidden tabs, corner blocks, automatic nesting/bed-fit checking, automatic mortises, and hardware-specific strength claims to later phases.

## 19. Final boundary status

No unrelated file was edited. No storage or schema migration was introduced. No existing production template SVG contract was intentionally changed. Nothing was staged, committed, or pushed.

## Correction follow-up

The focused audit identified a screen-only lid point-shape defect, misleading lift-off/hinge Finished View semantics, missing coordinate-level protection, and unconditional Gift Box label attempts. Those corrections are documented in `DESIGNS_GIFT_BOX_G1_CORRECTION_2026-07-19.md`; the focused group subsequently grew from 60 to 69 assertions and the reconciled complete suite from 2353 to 2362, with the existing Designs geometry group still at 1093/0. Production SVG serialization remains separate from Finished View rendering; repeated post-correction production captures are deterministic and contain no Finished View IDs.
