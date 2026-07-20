# Tabletop Accessories T1.5 — Physical Coupon Results and Evidence Reuse

Date: 2026-07-20  
Baseline: `0b7f89f275cc19e4be763c10e1e9753d5666ff28`  
Scope: additive T1.5 implementation in the standalone offline `index.html`.

## Outcome

T1.5 is implemented as a raw physical-result workflow for Tabletop Accessories T1. The app can record generated or manual coupon results, retain the complete test context, review and edit saved results, explicitly promote an eligible result into the existing Library production-setting path, and reuse the resulting evidence through a pure matching/ranking helper. No automatic promotion, product card, recommendation flow, schema migration, or T2 behavior was added.

## Changed files

- `index.html` — raw result model, normalization, persistence, import/export/merge/replace/recovery, recorder/list/promotion UI, Help, matching helper, and fixtures.
- `README.md` — T1/T1.5 workflow, focused routes, and measured totals.
- `CHANGELOG.md` — implementation and validation note.
- This report.

## Raw record and identity

The new root collection is `tabletopCouponResults`; existing storage key, app id, schema version, and backup format remain unchanged. Records normalize machine snapshots, material profile/free-text snapshots, nominal and measured thickness, construction identity, candidate geometry, C1/C2/C3 observations, fit classification, self-support classification, preferred candidate, tighter alternate, notes, timestamps, and promotion references.

Construction identity is explicit: wall-join type, floor-join type, horizontal/floor finger count, horizontal/floor actual width, vertical/corner finger count, vertical/corner actual width, and generator version. Generated records derive all four pattern fields from real generated candidate data; missing generated pattern data fails closed instead of inventing counts or widths. Manual records remain readable when identity is incomplete but are ineligible for promotion or evidence reuse until the required identity is present. Measured thickness and exactly one preferred candidate are required for eligibility; a tighter alternate is optional, but when present it must exist, differ from the preferred candidate, and have a smaller clearance.

## Workflow and boundaries

The Designs Tabletop Accessories area exposes Record Physical Results and View Saved Results. The recorder supports the current generated candidate set and previous/manual entry, with visible candidate labels and preferred selection. Saved results support view, edit, delete, and explicit Promote to Library. Deleting a promoted result requires confirmation and does not delete the Library setting.

Promotion uses the existing Library transaction and existing material profile. It creates a tested `tabletop-coupon` production setting containing only coupon-proven fields plus a frozen `fitContext` snapshot: evidence stage, construction identity, thickness values, clearance step, material snapshot, preferred candidate, optional tighter alternate, source result id, and test time. The raw back-reference is written only after the existing promotion transaction and raw persistence succeed; failed promotion leaves the raw result unpromoted. Coupon-proven settings cannot be marked verified or treated as independently verified production evidence.

The matching helper is pure and does not mutate state. It prefers exact construction identity, then material profile identity, then strongest/closest thickness tier, with a conservative legacy fallback. It is available for later reuse but does not auto-apply settings or alter recommendation/product behavior.

## Storage and recovery

The collection is included in normal state load/persist, backup export, import replace, merge de-duplication, and recovery normalization. Existing persistence rollback behavior remains the failure boundary. Existing state, SVG bytes, paths, layers, filenames, geometry, Library records, localStorage key, and unrelated production evidence are not migrated or rewritten.

## Help and accessibility

Help documents physical cutting, C1/C2/C3 observations, preferred/tighter alternate selection, measured thickness, Coupon-proven status, explicit promotion, frozen evidence context, matching limits, shell-gate and safety boundaries. Controls have labels, modal titles, cancel paths, and explicit destructive-action confirmation.

## Validation

Direct file validation passed with `python -m html.parser index.html` and `git diff --check` (CRLF warnings only). Browser fixture validation was run from the standalone `file://` page with no network dependency:

| Route/group | Passed | Failed |
| --- | ---: | ---: |
| Tabletop engine | 27 | 0 |
| Tabletop coupon SVG/engine | 76 | 0 |
| Raw coupon results | 33 | 0 |
| Evidence promotion | 22 | 0 |
| Evidence matching | 21 | 0 |
| Help | 43 | 0 |
| Gift Box G1 | 69 | 0 |
| Dice Tray DT1 | 92 | 0 |
| Designs geometry | 1093 | 0 |
| Complete suite | 2642 | 0 |

The retained protected Tabletop coupon SVG pin is 2992 bytes with FNV hash `4f543f95`. Existing protected legacy pins remained unchanged, including Dice Tray, alternate, divider, and joint coupon fixtures. The complete suite reports 34 groups and no fixture failures or page exceptions; the browser emitted four existing non-fatal `svg height="auto"` preview warnings.

## Physical evidence status and next boundary

The implementation makes the current physical result enterable and preserves evidence for reuse, but it does not claim that a physical coupon has been cut or measured. T1.5 is ready for the focused audit. T2 remains outside scope until the complete shell/physical validation gate is satisfied.

No stage, commit, push, reset, clean, stash, checkout, move, rename, delete, or network dependency action was performed.
