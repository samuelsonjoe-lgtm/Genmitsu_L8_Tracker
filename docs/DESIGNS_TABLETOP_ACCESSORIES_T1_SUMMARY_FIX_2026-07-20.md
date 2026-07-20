# Tabletop Accessories T1 Result-Summary Fix

Date: 2026-07-20  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `7876551 Add Dice Tray storage and insert options`

## 1. Repository state

Initial inspection confirmed branch `main`, HEAD `78765512f58d5a16eea89073958252bd5171b6a7`, and `origin/main...main` at `0 0`. Nothing was staged. Existing uncommitted T1 edits were retained. Tracked edits remained limited to the intended T1 files; existing LightBurn projects, historical reports, `debug.log`, and utility scripts were left untouched.

## 2. Reproduction

The representative coupon case used measured thickness 2.88 mm, requested interior leg 40 mm, wall height 22 mm, center clearance 0.10 mm, step 0.05 mm, preferred finger width 9 mm, and panel spacing 15 mm. Before this correction, the result summary rendered `Finished cavity: NaN × NaN mm`.

## 3. Root cause

The builder stores the authoritative finished cavity as `result.metrics.finishedCavity.clearBounds`, while the summary read nonexistent `result.metrics.finishedCavity.width` and `.depth` fields. The production geometry and serializer were not the source of the defect.

## 4. Exact correction

The summary now reads:

```js
result.metrics.finishedCavity.clearBounds.width
result.metrics.finishedCavity.clearBounds.depth
```

No fallback to `requestedInterior` was added. The displayed metric therefore remains tied to the authoritative finished-cavity descriptor.

## 5. Optional material metric

Added a dedicated `Material thickness` result metric derived from `result.metrics.materialThickness`. The representative display is `2.88 mm`; no value is hard-coded.

## 6. Fixture additions

Added eight separate, failure-localizing coupon assertions covering: Finished cavity metric presence; representative 40 × 40 mm display; no NaN; no undefined; use of `clearBounds`; separation of requested and finished metrics; dedicated material thickness; and unchanged production SVG for identical inputs.

## 7. Representative result

The direct browser result summary now displays `Finished cavity: 40 × 40 mm`, `Requested interior: 40 mm legs`, and `Material thickness: 2.88 mm`. Candidates remain 0.05 / 0.10 / 0.15 mm and the coupon remains nine pieces.

## 8. Production SVG comparison

The correction is confined to `designResultsHtml()` plus fixtures and documentation. The tabletop builder, panel geometry, layout, label generation, Finished View builder, and production SVG serializer were not changed. The new fixture compares the production SVG from the result with a repeat build from identical inputs and passes byte-for-byte. Deterministic repeated generation also remains green.

## 9. Legacy-output comparison

No golden was updated. Retained pins remain passing: Dice Tray 1726 bytes / FNV `51a55721`; Alternate Dice Tray 1054 / `41697123`; Divider Tray 1965 / `a55dda6e`; Joint Fit Coupon 7764 / `db7ea7e9`. Existing Dice Tray, Gift Box, Tray standalone, Designs geometry, machine, promotion, and other legacy groups remained green.

## 10. Exact focused totals

Measured direct browser totals after the fix:

- Tabletop engine: 27 passed / 0 failed
- Tabletop coupon: 75 passed / 0 failed
- Help and Safety: 40 passed / 0 failed
- Designs geometry: 1093 passed / 0 failed
- Dice Tray System: 92 passed / 0 failed
- Gift Box: 69 passed / 0 failed
- Tray standalone: 264 passed / 0 failed
- M1: 29 passed / 0 failed
- M2: 31 passed / 0 failed
- M3: 27 passed / 0 failed

## 11. Exact complete-suite total

The measured unique complete suite is **2559 passed / 0 failed across 31 groups**. The eight new coupon assertions are included in that total; this is a measured post-change result, not an expected-only arithmetic claim.

## 12. Direct `file://` validation

`python -m html.parser index.html` exited 0 and `git diff --check` passed. A disposable direct-file Edge run opened Designs, selected Tabletop Corner + Floor Coupon, entered the representative values, confirmed all controls were editable, and verified the Finished cavity and material-thickness metrics. No NaN appeared, candidates remained visible, and no page errors occurred.

The focused routes `?selftest=tabletop-engine`, `?selftest=tabletop-corner-floor-coupon`, `?selftest=help`, `?selftest=design`, `?selftest=dice-tray-system`, `?selftest=gift-box`, `?selftest=machines-m1`, `?selftest=machines-m2`, `?selftest=machines-m3`, and `?selftest=all` remain the validation routes. The standalone Tray and promotion-switch checks also remained green.

## 13. Documentation updates

Updated `README.md` and the T1 implementation report with the new measured coupon and complete-suite totals. Added this report. The prior focused audit and prior correction-verification report were not edited.

## 14. Protected-boundary comparison

APP identity, storage/recovery, machines M1/M2/M3, promotion, Project schema, accounting, Inventory, Pricing, legacy templates, existing coupons, kerf conventions, LightBurn colors, filenames, import/export/backup, and offline/no-network behavior were not intentionally changed.

## 15. Remaining physical uncertainty

Actual kerf, fit, triple-junction behavior, squareness, clamp access, glue strength, material/coating safety, and LightBurn import/framing remain physically unverified.

## 16. Controlled-cut decision

Controlled exploratory scrap cutting remains approved only with actual measured stock, ventilation, active fire watch, appropriate suppression, and independent LightBurn review. The coupon is not cleared for production or sold-product use.

## 17. Commit readiness

The NaN summary bug is fixed and all registered groups are green, but no commit is approved or performed in this pass. Human review remains required before committing the broader uncommitted T1 work.

## 18. T2 gate

T2 remains blocked until controlled physical coupon results are recorded and reviewed.

## 19. Further-review decision

No additional Grok or Claude review is necessary for this bounded correction because the Finished cavity path is correct, the new fixtures pass, production bytes remain unchanged, and all registered groups are green.

## 20. Change hygiene

Nothing was staged, committed, pushed, reset, cleaned, stashed, checked out, moved, renamed, or deleted.
