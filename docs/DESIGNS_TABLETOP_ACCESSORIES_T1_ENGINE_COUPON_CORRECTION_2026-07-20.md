# Tabletop Accessories T1 Engine + Corner/Floor Coupon Correction

Date: 2026-07-20  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `7876551 Add Dice Tray storage and insert options`

## 1. Repository state

HEAD remains `78765512f58d5a16eea89073958252bd5171b6a7`, branch `main`, synchronized `origin/main...main` is `0 0`, and nothing is staged.

## 2. Original blocker reproduction

The focused audit reproduced the blocker in `renderDesigns`: the first `tabletop` arm of the `formFields` ternary returned a plain note string, making the later real tabletop form arm unreachable. Selecting the coupon therefore removed the template selector and controls.

## 3. Actual files changed

Tracked: `index.html`, `README.md`, and `CHANGELOG.md`. Documentation: this correction report and the existing T1 implementation report were updated. Existing untracked LightBurn projects, historical reports, `debug.log`, and the utility script were preserved.

## 4. Form-wiring correction

The tabletop arm now renders template selector, material thickness, units, center clearance, clearance step, preferred finger width, interior leg, wall height, panel spacing, assembly labels, kerf reference, preview controls, download, and Project handoff actions.

## 5. Dead-branch cleanup

The note-only tabletop arm was removed. There is one reachable tabletop form branch and one tabletop-specific note branch.

## 6. Tabletop-specific form note

The note states that T1 tests one 90-degree wall corner and both wall-to-floor joints together, requires actual-sheet thickness, dry-fit before glue, treats clearances as starting points, keeps kerf separate, and does not prove full-product strength.

## 7. In-app Help addition

Help now appends an accessible-styled Tabletop Accessories T1 section covering separation from legacy Dice Tray, rectangular finger-jointed walls/floor, future non-90-degree hex strategy, dry fitting, clearances, thickness, kerf, prohibited machining features, flat material, focus, framing, ventilation, fire watch, suppression, and unattended-laser prohibition.

## 8. README and selftest documentation

README now records the measured complete-suite baseline after correction as 2551 passed / 0 failed across 31 groups and lists `?selftest=tabletop-engine` and `?selftest=tabletop-corner-floor-coupon`. It states T1 is shared groundwork plus a construction coupon, not a Premium Rectangular Tray or Hex Dice Tray, and that physical cutting remains required.

## 9. Implementation-report correction

The implementation report now preserves the historical form blocker, records the Help omission and fixture gaps, documents the correction, replaces expected arithmetic with measured totals, and records remaining physical uncertainty.

## 10. Actual finger-width metrics

Coupon result metrics now expose preferred width, actual horizontal/floor width, actual vertical/corner width, and minimum manufacturable segment. Values are derived from each candidate’s generated patterns; the representative 2.88 mm case derives approximately 9.152 mm horizontal/floor, 8.293 mm vertical/corner, and 5.76 mm minimum.

## 11. Silent-cavity-shrink validation

`tabletopAccessoryDimensions` preserves requested and finished fields and blocks an interior-primary finished cavity below requested dimensions unless explicit allowed-intrusion metadata is supplied.

## 12. Clearance-step validation

Clearance step must be greater than zero. Candidates are checked for distinctness after rounding; zero, negative, or rounded-duplicate candidate sets are blocked.

## 13. Panel-spacing validation

Panel spacing must be positive. Zero and negative spacing now block production.

## 14. Expanded engine fixture inventory

The engine group directly checks rectangle and hex winding, outward normals, previous/next wrapping, 120-degree hex metadata, no 90-degree hex finger geometry, dimension-field distinction, silent-shrink invalidation, finished-cavity divider derivation, half-thickness truthfulness, physical insert intent, opaque tags, finite/deterministic projection, malformed points, arbitrary edge counts, premium corner metadata, and rejection of floor-only registration as the rectangle strategy.

## 15. Expanded coupon fixture inventory

The coupon group directly checks form wiring, selectors, all required control IDs, note truthfulness, 2.88 mm validity, candidates, nine pieces, IDs, roles, derived widths, cavity/envelope/height disclosures, wall and floor phase mating, terminal webs, no fourth wall, layout gaps, deterministic SVG, closed/unique cuts, colors, labels, Finished View geometry, invalid inputs, inch mode, handoff, and cleanup.

## 16. Form-presence assertions

The focused coupon fixture parses `renderDesigns()` for the tabletop template and asserts editable controls for `materialThickness`, units, center clearance, clearance step, preferred finger width, interior leg, wall height, panel spacing, assembly labels, and kerf reference. It also asserts the template selector and tabletop note.

## 17. 2.88 mm representative evidence

The representative case uses 2.88 mm thickness, 0.10 mm center, 0.05 mm step, 9 mm preferred width, 40 mm leg, 22 mm wall height, and 15 mm spacing. It is valid with candidates 0.05, 0.10, 0.15 and nine pieces.

## 18. Wall-to-wall phase evidence

The fixture compares WALL-A and WALL-B vertical edge descriptors directly and requires opposite phase, equal count, and equal actual width.

## 19. Wall-to-floor phase evidence

The fixture compares each selected wall’s bottom edge to its matching FLOOR edge and requires opposite phase, equal count, and equal actual width for both walls.

## 20. Terminal-web evidence

Candidate metadata carries a positive manufacturing minimum and junction terminal-web values; the fixture requires the minimum and derived junction values to remain nonnegative/positive as applicable.

## 21. Layout AABB evidence

All nine translated bounds are checked for overlap, and pairwise separation is checked against the requested 15 mm gap.

## 22. Label toggle and count evidence

Labels-off output has zero label count and no label group. Labels-on output has green paths, nine emitted paths, parity with `labelCount`, role/candidate-identifiable labels, and a label group separate from red cuts.

## 23. Finished View coordinate evidence

The fixture parses the screen-only SVG and checks finite coordinates, nine positive-area polygons, floor-before-wall ordering, non-coplanar schematic faces, readable C1/C2/C3 identifiers, and no nonexistent groove/rebate/channel/bevel/lid/divider/insert implication.

## 24. Fixture cleanup evidence

The coupon fixture snapshots and restores draft, preview mode, active tab, and storage state. It uses a parsed form document rather than mutating the live form, creates no Project, and leaves no synthetic record.

## 25. Exact focused totals

Measured direct-file browser totals: engine **27 passed / 0 failed**; coupon **67 passed / 0 failed**; Help and Safety **40 passed / 0 failed**; Designs geometry **1093 passed / 0 failed**.

## 26. Exact complete-suite total and group count

The measured unique complete suite is **2551 passed / 0 failed across 31 groups**. The previous post-T1 total was 2478; the correction added 15 engine, 55 coupon, and 3 Help assertions.

## 27. Legacy-output byte/hash comparison

Legacy builders and serializers were not rewritten. Existing Dice Tray System, Gift Box, Designs geometry, and related suite goldens remain passing; no legacy golden was updated. The prior retained baseline pins remain the comparison boundary, including Dice Tray 1726 bytes / FNV `51a55721`, Divider Tray 1965 / `a55dda6e`, and Joint Fit Coupon 7764 / `db7ea7e9`.

## 28. Direct file:// results

`python -m html.parser index.html` exited 0 and `git diff --check` was clean. Direct browser selection confirmed the tabletop controls are visible and editable, candidates update for the 2.88 case, the Finished View control remains available, and switching away and back restores the form.

## 29. Disposable-browser workflow

A disposable direct-file Chromium profile opened Designs, selected the tabletop template, verified all required controls, entered 2.88 / 0.10 / 0.05, observed 0.05/0.10/0.15 in the summary, switched to Dice Tray, and returned to the tabletop form. Help was opened and its heading/construction/safety phrases were verified.

The focused workflows produced no page exceptions. The aggregate `selftest=all` startup also completed with zero fixture failures; it emitted four pre-existing SVG console warnings about `height="auto"` in an unrelated preview path, which were not introduced or changed by T1.

## 30. Documentation updates

Updated `README.md`, `CHANGELOG.md`, and the implementation report. Added this correction report. The existing normative architecture and construction-contract reports were not edited.

## 31. Protected-boundary comparison

APP identity, storage/recovery, machines M1/M2/M3, promotion, Project schema, accounting, Inventory, Pricing, legacy templates, kerf conventions, legacy colors/filenames, backups, and offline/no-network behavior were not intentionally changed. Existing suites for Dice Tray System (92/0), Gift Box (69/0), Tray standalone (264/0), M1 (29/0), M2 (31/0), M3 (27/0), and promotion-switch (16/0) remain green.

## 32. Remaining unverified physical areas

Real kerf, fit, triple-junction behavior, squareness after dry fit/glue, clamp access, strength, material/coating safety, and LightBurn import/framing remain physically unverified.

## 33. Physical-cut decision

The form blocker and important audit findings are corrected. The coupon is permitted only for controlled exploratory scrap cutting with actual measured stock, ventilation, fire watch, and independent LightBurn review; it is not cleared for production or sold-product use.

## 34. Commit decision

T1 may be considered for a later commit only after human review of this correction and the physical scrap result. No commit is authorized or performed in this pass.

## 35. Further verification

No additional Grok re-verification or Claude audit is required for this local correction set, unless physical testing exposes a new shared geometry issue.

## 36. Change hygiene confirmation

Nothing was staged, committed, pushed, reset, cleaned, stashed, checked out, moved, renamed, or deleted. Unrelated untracked files remain untouched.
