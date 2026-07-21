# Tabletop Accessories T3A-E1 Evidence Implementation

## Scope and state

Implemented from `b71cfc1 Add Tabletop storage fit coupon` on `main`, synchronized with `origin/main` before work. Long-standing untracked files were preserved; no file was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## Record and data safety

`SCHEMA_VERSION` is now 3. The additive `tabletopStorageResults` collection stores generated T3A identity, machine/material snapshots, nominal and measured thickness, signed shell clearance, requested 80 x 60 x 25 cavity identity, actual floor-finger geometry, rail/panel values, recomputed derived dimensions, tri-state observations, result/notes, promotion back-reference, and a frozen outer-shell snapshot. Missing data loads as `[]`; malformed records are filtered without startup failure; unknown root fields survive record normalization. Version-1/2 data remains importable, version-3 backups round-trip, and later versions are rejected by the existing future-format guard.

The snapshot keeps linked T2 result/setting/evidence IDs when available while retaining the T2 construction identity, Floor ownership, dimensions, measured thickness, clearance, material, machine, and shell stage if the linked record disappears.

## Evidence contract

The dedicated recorder supports contemporaneous and retrospective-manual entry, exact generated facts, a selected T2 shell result, all required unknown-by-default observations, raw pass/fail/incomplete records, list/edit/delete, eligibility review, and explicit promotion. Promotion requires the exact T3A generator/construction, Library profile, exact machine, valid date, linked snapshot, matching linked material ID, pass, and every required true/false observation. It writes the additive `tabletop-storage-fit-coupon` kind with `storage-fit-coupon-proven`; duplicate raw promotion is blocked.

The dedicated matcher uses 1e-6 numeric equality. It requires the exact Floor-owned T2 identity, dimensions, material, thicknesses, signed clearance, actual width/depth floor fingers, rail height, panel thickness, total clearance, machine, generator, and construction. It has no compatible path. T3A never proves T2, cork, complete-tray, or Production-proven status.

## Validation

- HTML parser and `git diff --check` passed.
- Disposable headless `file://` Edge: focused `tabletop-storage-results` passed **14 / 0**; it exercised a synthetic retrospective 80 x 60 x 25 result, 2.88 / -0.075 measurements, raw save, backup round trip, promotion, exact matching, and mismatch gates.
- The existing T3A SVG is unchanged: **897 bytes / FNV-1a `b5c549ee`**. Broader protected golden and full-suite revalidation remain to be run before commit.

## Joe's procedure and remaining validation

Record or import the associated promoted T2 shell first; regenerate the exact T3A coupon with the matching Library material and machine; choose the shell result; enter the true test date and retrospective-manual mode for the completed test; record all observations (leave unmeasured post-cure diagonals unknown); save the raw result; review every gate; then explicitly promote only if the pass criteria are true. An independent review is recommended before committing because this phase adds persisted evidence and schema v3.
