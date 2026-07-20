# Tabletop Accessories T2A — Focused-Audit Correction Report

**Repository:** `C:\Genmitsu L8 Tracker`  
**Baseline:** `d0a89e4afd3bba4de47e924032b3aa64182d8eb7` — `Add Tabletop coupon evidence workflow`  
**Date:** 2026-07-20  
**Status:** T2A remains uncommitted; no staging, commit, push, reset, clean, stash, checkout, move, rename, or delete was performed.

## 1. Repository state

Initial inspection confirmed branch `main`, HEAD exactly `d0a89e4afd3bba4de47e924032b3aa64182d8eb7`, `origin/main...main` = `0 0`, empty index, and no staged changes. Tracked changes before correction were limited to the existing T2A work in `index.html`, `README.md`, and `CHANGELOG.md`; the existing historical reports, LightBurn Projects, `debug.log`, utility script, and other untracked files were preserved. The architecture, implementation, and focused-audit reports were present. Final `git diff --check` is clean apart from normal line-ending warnings.

## 2. Findings corrected

All six IMPORTANT findings were corrected. No production geometry, edge phase, compatibility formula, layout coordinate, rotation policy, serializer, or protected output changed.

### 3. Project handoff provenance

T2A Project notes now carry frozen descriptive text for machine label plus ID/key, material label plus profile ID or free-text label, measured thickness, evidence tested date, evidence stage, compatibility, and source Setting/Evidence IDs. The values are copied into the generated handoff text; they are not live references. Missing identity is omitted rather than replaced with `undefined`, `null`, or an invented machine/material.

### 4. Four named wall-corner assertions

`FRONT-LEFT`, `FRONT-RIGHT`, `BACK-LEFT`, and `BACK-RIGHT` each separately assert panel IDs, opposite phases, equal mating count, equal actual finger width, equal joint depth, vertical-pattern selection, positive terminal-web condition, closed contours, and unique structural paths.

### 5. Four named wall-floor assertions

`FLOOR-FRONT`, `FLOOR-BACK`, `FLOOR-LEFT`, and `FLOOR-RIGHT` each separately assert IDs, phases, count, width, depth, axis assignment, positive terminal condition, no cross-axis substitution, and closed contours. A deliberate width/depth swap regression confirms FRONT/BACK use the width-axis pattern and LEFT/RIGHT use the depth-axis pattern.

### 6. Finger-width tolerance coverage

The approved `tabletopJointFitCompatible` formula is unchanged. Fixtures now cover just-inside, exact, and just-outside boundaries for width-axis, depth-axis, and vertical-axis comparisons, including axis-specific failure reasons. They also prove that counts may differ when actual widths match, matching counts cannot rescue incompatible widths, requested/preferred width is not substituted for generated width, the Coupon actual width supplies the 15% reference, minimum-web/2 is honored, and the floating-point epsilon only affects the boundary. `tabletopCouponConstructionMatches` remains unchanged.

### 7. Apply immutability and override provenance

The evidence fixture begins with a manual value, verifies that recommendations do not alter it, explicitly applies `-0.075 mm`, and verifies that only the current T2A draft changes. Raw coupon results, Production Setting/Evidence, global defaults, and another template draft remain unchanged. A real form input event changes the value to `-0.050 mm`, records “Evidence-applied then manually overridden” in the result and handoff, and restoring `-0.075 mm` does not erase that provenance. Template switching follows the existing session-only Design-draft contract.

### 8. Conflict and Keep behavior

Two compatible entries with different preferred clearances are both returned and rendered, newest-first within equal rank. No average or silent selection occurs; selected-entry Apply targets only that entry. Keep current value is a real session-only acknowledgement with source text `Manual value retained`; it does not change clearance, defaults, persistence, or future recommendations.

### 9. Recommendation-card and Help polish

The existing heading remains exactly **Compatible Coupon-proven evidence — Not an exact full-shell validation.** Cards now include discrete `Evidence stage: Coupon-proven` and `Compatibility: Compatible` rows. Help now explains that automatic rotation is disabled to protect plywood grain direction, veneer appearance, strength, label readability, and user expectations.

## 10. Files changed

- `index.html` — bounded handoff, session-state, recommendation, Help, and focused-fixture corrections.
- `README.md` — authoritative unique/nested totals and focused totals.
- `CHANGELOG.md` — correction summary and totals.
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T2A_RECTANGULAR_SHELL_IMPLEMENTATION_2026-07-20.md` — correction-pass evidence.
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T2A_RECTANGULAR_SHELL_CORRECTION_2026-07-20.md` — this report.

No unrelated modified or untracked file was changed.

## 11. Runtime totals

| Group | Passed | Failed |
| --- | ---: | ---: |
| Rectangular shell | 93 | 0 |
| Shell evidence | 49 | 0 |
| Role layout | 6 | 0 |

The comparable flat suite is **2790 passed / 0 failed across 37 unique registered groups**. Each top-level route is registered exactly once. The nested diagnostic is **4592 passed / 0 failed across 60 invocations** and includes repeated regression calls. It must not be compared directly with the earlier **2642 / 0 across 34 groups**.

Required standalone focused routes also passed: Engine `27/0`; Corner/Floor Coupon `76/0`; Raw Results `33/0`; Evidence Promotion `22/0`; Evidence Matching `21/0`; Help `43/0`; Designs `1093/0`; Dice Tray `92/0`; Gift Box `69/0`; Machines M1 `29/0`; M2 `31/0`; M3 `27/0`; Tray model standalone `264/0`; promotion-target switch `16/0`; storage/recovery `15/0`.

## 12. Representative case and protected outputs

The controlled case remains 2.88 mm material, 80 × 60 × 25 mm interior, 9 mm target, `-0.075 mm` applied only explicitly, labels off, role-grouped layout, and no rotation. The requested cavity remains 80 × 60 × 25 mm; outer envelope remains 85.76 × 65.76 × 27.88 mm. T2A remains **2181 bytes / FNV `2ef9606b`**, with bounds **206.52 × 171.52 mm** and five structural panels.

Protected output comparisons remain unchanged: T1 Coupon `2992 / 4f543f95`; Dice Tray `1726 / 51a55721`; alternate Dice Tray `1054 / 41697123`; Divider Tray `1965 / a55dda6e`; Joint Fit Coupon `7764 / db7ea7e9`. Finger Box, Gift Box, Sliding-lid Box, Drawer Cabinet, and wall/base-tab coupon regression groups remain green. No golden was updated to accept drift.

## 13. Direct `file://` validation

Disposable local Playwright testing confirmed clean startup and no page exceptions on the required routes. The focused fixtures exercised T2A form generation, representative Cut Layout and Finished Assembled output, labels off/on, compatible evidence, discrete status rows, explicit Apply, Keep, manual override/restoration, conflict cards, selected Apply, source-deletion-safe handoff text, template switching, and no auto-application. Parser validation passed with `python -m html.parser index.html`; `git diff --check` passed. No network dependency, storage write, schema migration, or new root collection was introduced.

Responsive and keyboard behavior continue to use the existing audited Design form/result surfaces; they were not redesigned in this correction. Physical cutting, material behavior, squareness, strength, glue quality, and production safety remain unverified.

## 14. Final decision

- All six IMPORTANT findings: **corrected**.
- Production geometry changed: **no**.
- T2A focused totals: **93/0, 49/0, 6/0**.
- Unique flat suite: **2790/0 across 37 groups**.
- Nested suite: **4592/0 across 60 invocations**, separately reported.
- T2A golden `2181 / 2ef9606b`: **unchanged**.
- T2A ready to commit: **yes, after user review**.
- Joe may perform the controlled physical shell cut: **yes**, using the approved evidence-guided procedure; this is not Shell-proven or Production-proven.
- T2B: **deferred**.
- Additional Grok audit: **not required** while the compatibility formula, geometry, layout, and golden remain unchanged; it becomes worthwhile if any protected contract changes.
- Nothing was staged, committed, or pushed.

