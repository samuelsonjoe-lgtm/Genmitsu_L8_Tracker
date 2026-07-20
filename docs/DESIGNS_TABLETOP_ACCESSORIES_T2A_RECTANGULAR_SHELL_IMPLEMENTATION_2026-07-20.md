# Tabletop Accessories T2A — Rectangular Shell Prototype Implementation

**Repository:** `C:\Genmitsu L8 Tracker`  
**Date:** 2026-07-20  
**Baseline:** `d0a89e4afd3bba4de47e924032b3aa64182d8eb7` (`Add Tabletop coupon evidence workflow`)  
**Scope:** bounded T2A implementation only; no commit, push, reset, clean, stash, or destructive file operation.

## Delivered

- Added `tabletop-rectangular-shell-prototype`, displayed as **Tabletop Rectangular Shell Prototype**, under Tabletop Accessories.
- Added session-only interior-primary inputs: 80 × 60 × 25 mm recommended prototype, material thickness, shared signed clearance, preferred finger-width target, panel spacing, kerf reference, optional material context, and optional assembly labels.
- Reused `buildBoxModel` unchanged in open-top mode and retained exactly five production panels in stable order: `FLOOR`, `FRONT`, `BACK`, `LEFT`, `RIGHT`.
- Added explicit finished cavity, outer envelope, measured pattern widths/counts, four wall-corner pairs, four floor pairs, manufacturing limits, role-grouped layout, and no-rotation metrics.
- Added a screen-only assembled Finished View. Its `tabletop-shell-*` IDs are separate from production SVG output.
- Added a separate `tabletopJointFitCompatible` seam. It preserves exact machine identity, generator/joint/sign identity, reused thickness tiers, and bounded actual-width tolerance; finger count is not treated as an equality requirement.
- Added Compatible Coupon-proven recommendation cards with the required “Not an exact full-shell validation” wording. Apply is explicit and current-draft/session-only; manual edits are tracked; Keep and Run a new coupon remain available.
- Extended the existing descriptive Design-to-Project handoff without changing Project schema or creating a live foreign key.
- Added focused routes: `tabletop-rectangular-shell`, `tabletop-rectangular-shell-evidence`, and `tabletop-layout`.

## Protected boundaries

No new root collection, schema version, storage key, migration, backup field, shell-result recorder, T2B workflow, Shell-proven status, Production-proven status, product record, network behavior, or default evidence application was added. Existing T1 coupon and other protected production outputs were not intentionally changed. Existing `tabletopCouponConstructionMatches` remains unchanged.

The protected pins remain the audit acceptance boundaries: T1 coupon `2992 / 4f543f95`; Dice Tray `1726 / 51a55721`; alternate Dice Tray `1054 / 41697123`; Divider Tray `1965 / a55dda6e`; and Joint Fit Coupon `7764 / db7ea7e9`. The new T2A recommended default has its own pin: **2181 bytes / FNV `2ef9606b`**, layout `206.52 × 171.52 mm`, with labels disabled.

## Focused validation

Direct local Playwright `file://` runs passed:

- Rectangular shell: **22 passed / 0 failed** (including the new `2181 / 2ef9606b` production SVG pin).
- Shell evidence: **15 passed / 0 failed**.
- Role layout: **6 passed / 0 failed**.
- `?selftest=all` startup: **4592 passed / 0 failed** across 60 nested fixture invocations. The authoritative comparable flat suite is **2790 passed / 0 failed across 37 unique registered groups**; the nested total is not directly comparable to the earlier **2642 / 0 across 34 groups**.

Additional checks passed: `python -m html.parser index.html` and `git diff --check`. The existing template-selector fixture was updated from 12 to 13 registered template values to cover the authorized T2A registry entry; the complete Designs geometry group returned **1093 passed / 0 failed** after that correction.

## Physical validation still required

No physical shell was cut or inspected by this implementation. The evidence recommendation remains Coupon-proven only. For the Joe-equivalent test, use measured 2.88 mm plywood, explicitly Apply `-0.075 mm`, import at 100%, frame and fire-watch the cut, dry-fit FLOOR → FRONT → LEFT → RIGHT → BACK, and inspect cumulative tightness, squareness, veneer crushing, split tabs, floor seating, cavity dimensions, and non-destructive disassembly. If that value risks damage, test the already Coupon-proven `-0.050 mm` fallback; do not skip to an untested clearance. Do not call the shell Shell-proven or Production-proven until a separate physical evaluation passes.

## T2A focused-audit correction pass

The six IMPORTANT findings from the focused audit were corrected locally without changing geometry, evidence compatibility semantics, layout coordinates, serializer behavior, or protected SVG bytes:

1. Project handoff notes now include descriptive machine label/ID or key when available, material label/profile ID or free-text material label, measured thickness, tested date, evidence stage, compatibility, and source Setting/Evidence IDs as plain text. Missing identity is omitted honestly; no live lookup or foreign key is created.
2. All four wall corners and all four wall-floor pairs now have named, failure-localizing assertions covering IDs, phases, pattern counts/widths, joint depth, axis/pattern selection, terminal web, closed contours, and duplicate-path protection. A width/depth axis-swap regression is explicit.
3. Width-axis, depth-axis, and vertical tolerance cases now cover just-inside, exact-boundary, just-outside, axis-specific reasons, count-vs-width behavior, requested-width non-substitution, Coupon-width percentage reference, minimum-web floor, and epsilon behavior. `tabletopCouponConstructionMatches` remains unchanged.
4. Evidence fixtures now cover explicit Apply immutability against raw results, Production Setting/Evidence, defaults, and another template; Apply → manual override → restoration provenance; session template switching; free-text and missing material identity; and source deletion after handoff creation.
5. Conflicting compatible entries are both returned and rendered, sorted newest-first within equal rank, never averaged or auto-applied; explicit Apply targets only the selected entry. Keep current value records `Manual value retained` session-only.
6. Recommendation cards now show discrete Evidence stage `Coupon-proven` and Compatibility `Compatible` rows. Help explains why automatic rotation is disabled: plywood grain, veneer appearance, strength, label readability, and user expectations.

## Correction validation

The direct focused totals are: Rectangular Shell **93/0**, Shell Evidence **49/0**, and Role Layout **6/0**. The authoritative comparable flat suite is **2790 passed / 0 failed across 37 unique registered groups**, with every route registered exactly once. The nested `?selftest=all` diagnostic is **4592 passed / 0 failed across 60 invocations**; it contains repeated regression calls and is not comparable to the earlier `2642/0 across 34 groups`.

The representative case remains 2.88 mm material, 80 × 60 × 25 mm interior, 9 mm target, labels off, role-grouped, and no rotation. The T2A production SVG remains **2181 bytes / FNV `2ef9606b`**, with layout bounds **206.52 × 171.52 mm**. Protected T1/Dice/Divider/Joint Coupon pins and the existing Finger Box, Gift Box, Sliding-lid Box, Drawer Cabinet, and wall/base-tab regression groups remain green.

`python -m html.parser index.html` and `git diff --check` pass. Direct `file://` Playwright routes passed with no page exceptions for the required Tabletop, Help, Designs, Dice Tray, Gift Box, M1/M2/M3, and storage/evidence checks; the UI paths covered startup, T2A generation, Cut Layout/Finished Assembled, labels, evidence cards, Apply, Keep, override/restoration, conflict cards, source deletion, and Project handoff text. Responsive and keyboard behavior continue to use the existing audited form/layout surfaces; no network or storage/schema migration was introduced.

T2A is ready to commit after user review, and Joe may perform the controlled physical shell cut using the approved procedure. T2B remains deferred. No additional Grok audit is required because all corrections remained local, the compatibility formula and geometry were untouched, and all direct fixtures and output pins pass; a second audit would only be worthwhile if those protected contracts change. Nothing was staged, committed, or pushed.
