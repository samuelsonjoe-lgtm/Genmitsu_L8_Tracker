# Tray Tab Terminology Cleanup Implementation

**Date:** 2026-07-17  
**Baseline:** `2ac5080` — Document joint system architecture  
**Scope:** Phase A terminology-only cleanup and verified dead-helper removal; no commit or push

## Delivered

- Changed the Dice Tray and Divider Tray field label to **Wall-to-base tab profile**.
- Kept the internal `jointStyle` values unchanged: `finger` and `tab-slot`.
- Changed their visible option labels to **Four-tab walls (wall-to-base)** and **Two-tab walls (wall-to-base)**.
- Replaced the tray help text with: “Tabs on each wall bottom mate with through-slots in the base. Adjacent walls meet as butt joints and need glue after a dry fit. This is not corner finger joinery. Fit clearance is a starting value; test on scrap or with a future fit coupon.”
- Preserved the existing honest Dice and Divider Finished View profile wording, including four-/two-tab wall profiles and alignment with base slots; no Finished View markup or geometry changed.
- Removed the unused `designEdgePoints()` helper after confirming zero live `index.html` definition or call sites. The sole remaining runtime textual reference is the new `typeof designEdgePoints === 'undefined'` absence assertion; `buildSlidingLidBodyPanel()` continues to use `designSafePatternEdgePoints()`.

## Fixture coverage

Added compact Tray-model assertions for the field label, visible option labels, unchanged internal values, complete help wording, both valid internal styles, Finished View wording, dead-helper absence, and continued safe-helper use. Existing default Dice and Divider golden assertions remain in place.

## Production and protected boundaries

- Dice Tray default production SVG: **1726 characters / `51a55721`** — unchanged.
- Divider Tray default production SVG: **1965 characters / `a55dda6e`** — unchanged.
- No change to `buildTrayModel()`, `trayTabProfile()`, `designWallPath()`, `designTabPositions()`, `serializeTrayCompatibilitySvg()`, Finished View markup, storage, backups, schemas, production settings, promotion evidence, download filename/MIME/bytes, LightBurn grouping, or non-tray geometry.
- Comparison against `2ac5080` is limited to tray display wording, focused fixture coverage, removal of the dead helper, and README/report documentation. Historical review documents retain descriptive references to the retired helper; no live application function definition, call site, dynamic function map, or browser path references it.

## Validation

- `git diff --check` — clean
- `python -m html.parser index.html` — passed
- Isolated headless Edge direct `file://` startup and JavaScript fixture execution — passed
- Tray model/live production fixtures — **248 passed / 0 failed**
- Designs geometry fixtures — **876 passed / 0 failed**
- Designs production application — **118 passed / 0 failed**
- Test Grid machine identity — **18 passed / 0 failed**
- Evidence Promotion — **58 passed / 0 failed**
- Production Settings — **66 passed / 0 failed**
- Storage recovery — **8 passed / 0 failed**
- Complete suite — **1668 passed / 0 failed**

## Unverified areas

This wording-only phase does not provide physical proof of tab fit, glue strength, material behavior, kerf behavior, safe assembly, or production safety. Physical fit testing remains required before treating any clearance as proven.

**Final status: READY FOR FOCUSED AUDIT**
