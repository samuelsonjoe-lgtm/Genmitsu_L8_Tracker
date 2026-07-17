# Wall-to-base Tab Clearance Coupon Implementation

**Date:** 2026-07-17
**Baseline:** `541cda5` — Clarify tray wall-to-base tab terminology
**Scope:** Session-only second Joint Fit Coupon mode; no commit or push

## Delivered

- Added a session-only `couponType` select: **Finger-edge clearance** (`finger-edge`) and **Wall-to-base tab clearance** (`wall-base-tab`). Missing or blank values default to Finger-edge; unknown nonblank values are rejected.
- Wall-to-base defaults are four-tab walls, 80 mm test-wall length, and `0.20, 0.15, 0.10, 0.05, 0.00` mm slot-clearance candidates. Its candidate list is separate from the Finger Coupon list and preserves entered top-to-bottom order.
- Wall-to-base geometry contains two identical 80 mm test walls with a named fixed 22 mm wall height, plus one base plate. The plate has 6 mm top/bottom margins, 10 mm clear material gaps between candidate rows, and one production-profile slot per wall tab in every row. The two walls and base plate use the existing 10 mm layout margin/gap convention.
- Added `traySlotWidthMm(thicknessMm, clearanceMm)` and changed only `buildTrayModel()`'s former inline slot-width calculation to call it. The coupon reuses `trayTabProfile()`, `designTabPositions()`, `trayWallComponent()`, `trayRectComponent()`, `traySlotWidthMm()`, and `designSvgDocument()` directly.

## Coupon contract

- The new coupon serializes as one anonymous red-cut group with no score group, IDs, titles, metadata, or labels.
- Candidate rows correspond to the displayed top-to-bottom list; use the loosest row first. The second wall is a backup.
- The result explains that this is a local feature-clearance test and that any winner must be recorded manually. Finger-edge promotion remains available; wall-to-base promotion is intentionally hidden based on `result.metrics.couponType`.
- Wall-to-base default golden: **1551 characters / `d9ffc278`**.

## Validation

- Direct browser mode switch: Finger Coupon fields and promotion appeared; Wall-to-base fields replaced them, rendered a preview, hid promotion, and showed the manual-recording note; switching back restored the Finger Coupon controls and promotion.
- Browser download interception: the preview and downloaded wall coupon SVG matched exactly and retained the established filename and MIME type.
- Finger Coupon missing/explicit `finger-edge` path remains byte-identical to the current Finger Coupon output.
- Dice Tray default production SVG: **1726 characters / `51a55721`** — unchanged.
- Divider Tray default production SVG: **1965 characters / `a55dda6e`** — unchanged.
- `git diff --check` — clean
- `python -m html.parser index.html` — passed
- Isolated headless Edge direct `file://` startup and browser fixture execution — passed
- Tray model/live production fixtures — **248 passed / 0 failed**
- Designs geometry fixtures — **887 passed / 0 failed**
- Designs production application — **118 passed / 0 failed**
- Evidence Promotion — **58 passed / 0 failed**
- Production Settings — **66 passed / 0 failed**
- Test Grid machine identity — **18 passed / 0 failed**
- Storage recovery — **8 passed / 0 failed**
- Complete suite — **1679 passed / 0 failed**

## Protected boundaries and physical limits

No storage/schema, existing Finger Coupon geometry/serializer, promotion implementation, production-setting mapping, tray product geometry, Finished Views, download naming/MIME, or LightBurn grouping for existing templates changed. Protected-function comparison against `541cda5` is limited to the approved `buildTrayModel()` helper extraction and the new wall-to-base branch; all other protected functions are untouched.

This software validation does not prove insertion feel, kerf/char behavior, wall or corner strength, glue strength, full-tray squareness, warping, cumulative four-wall error, Divider retention, or production safety. Test on the intended sheet or batch with intended cutting settings.

**Final status: READY FOR FOCUSED AUDIT**
