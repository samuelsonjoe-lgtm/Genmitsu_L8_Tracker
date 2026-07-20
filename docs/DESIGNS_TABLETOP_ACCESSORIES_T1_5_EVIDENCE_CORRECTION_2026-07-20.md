# Tabletop Accessories T1.5 — Evidence Correction Report

Date: 2026-07-20  
Baseline: `0b7f89f275cc19e4be763c10e1e9753d5666ff28`  
Scope: bounded correction of the remaining T1.5 focused-audit findings.

## 1. Repository state

Initial inspection confirmed `main`, HEAD `0b7f89f275cc19e4be763c10e1e9753d5666ff28`, and `origin/main...main` at `0 0`. Nothing was staged. Existing tracked T1.5 edits remained uncommitted. The architecture, implementation, and focused-audit reports existed. Existing LightBurn Projects, historical reports, `debug.log`, and `parametric_qr_stand_generator.py` were preserved and not edited.

No stage, commit, push, reset, clean, stash, checkout, move, rename, delete, or network-dependent action was performed.

## 2. Findings corrected

The implementation now documents and directly tests the locked behavior: exactly one preferred candidate is required; the tighter alternate is optional. When supplied, it must exist, differ from the preferred candidate, and have a smaller clearance. Existing rejection coverage for missing preferred, equal/missing alternate, and looser alternate remains intact.

The raw-results fixture now saves a completed generated record with preferred `C2` and no alternate, confirms eligibility, retains `preferredCandidateId: C2`, retains the canonical empty alternate, and confirms no automatic promotion. The promotion fixture separately explicitly promotes a no-alternate record and verifies C2 clearance, `preferredCandidate: C2`, `tighterAlternate: null`, tested status, and Coupon-proven evidence.

The dead metrics fallback in `tabletopCouponConstructionIdentityFromResult` was removed. The function now reads all four required fields only from `finishedView.candidates[0].patterns`: horizontal/floor finger count and actual width, and vertical/corner finger count and actual width. `buildTabletopCouponResultRecord` fails closed when that generated pattern is unavailable. Fixtures prove incomplete generated pattern data cannot create a complete identity or generated raw record; manual incomplete records remain readable but ineligible.

## 3. Files changed

- `index.html` — fail-closed generated identity path and localized no-alternate/fallback fixtures.
- `README.md` — corrected optional-alternate wording and measured totals.
- `CHANGELOG.md` — bounded corrected aggregate and focused totals.
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_5_EVIDENCE_IMPLEMENTATION_2026-07-20.md` — corrected history/report wording and totals.
- This correction report.

The architecture review and focused-audit report were not edited. Unrelated modified and untracked files were preserved.

## 4. No-alternate raw-save and promotion runtime

The successful no-alternate raw fixture uses the real generated Tabletop result shape, measured thickness `2.88`, preferred `C2`, and blank alternate. It passes eligibility, saves through `tabletopCouponRecordSave`, retains C2 and an empty alternate, and remains eligible. Saving leaves Library production settings and evidence untouched.

The separate promotion fixture uses an existing disposable Library material profile and requires explicit promotion. The promoted setting receives `fitSettings.fingerJointClearanceMm` from C2; frozen evidence stores `preferredCandidate.id = C2`, `tighterAlternate = null`, `verificationStatus = tested`, and evidence kind/stage `tabletop-coupon` / `coupon-proven`. Raw back-reference finalization remains a separate successful step. Failed promotion still leaves the raw record unpromoted; editing raw data does not mutate frozen evidence; deleting raw data does not delete the Library setting.

## 5. Physical-result regression

The existing physical-result shape remains covered without seeding user storage: measured thickness `2.88 mm`, C1 `-0.075 mm`, C2 `-0.050 mm`, C3 `-0.025 mm`, preferred C2, tighter alternate C1. It remains eligible and maps the preferred clearance correctly. No record was inserted into Joe's production localStorage.

## 6. Exact fixture totals

Direct `file://` Edge/Playwright execution of the exposed fixture functions reported zero failures:

| Group/route | Passed | Failed |
| --- | ---: | ---: |
| Tabletop engine | 27 | 0 |
| Tabletop corner/floor coupon | 76 | 0 |
| Raw coupon results | 33 | 0 |
| Evidence promotion | 22 | 0 |
| Evidence matching | 21 | 0 |
| Help | 43 | 0 |
| Designs geometry | 1093 | 0 |
| Dice Tray DT1 | 92 | 0 |
| Gift Box G1 | 69 | 0 |
| Machines M1 | 29 | 0 |
| Machines M2 | 31 | 0 |
| Machines M3 | 27 | 0 |
| Existing production Evidence Promotion | 58 | 0 |
| Storage recovery | 15 | 0 |
| Production settings | 66 | 0 |
| Tray model standalone | 264 | 0 |
| Promotion-switch standalone | 16 | 0 |

The complete `?selftest=all` registration contains 34 groups exactly once and the independently measured complete total is **2642 passed / 0 failed**. The focused total rose from the audit's 2632 to 2642 because the correction adds localized assertions; the browser runtime, not expected arithmetic, is authoritative.

## 7. Output protection

The Tabletop coupon remains pinned at **2992 bytes / FNV `4f543f95`**. Existing protected legacy output pins remain green: Dice Tray **1726 / `51a55721`**, alternate Dice Tray **1054 / `41697123`**, Divider Tray **1965 / `a55dda6e`**, and Joint Fit Coupon **7764 / `db7ea7e9`**. Existing Gift Box, Finger Box, Sliding-lid Box, Drawer Cabinet, and wall/base tab coupon fixture coverage remained green; no golden was updated to accept unexplained drift.

## 8. Storage, matching, and protected boundaries

Existing storage, import/export, merge, replace, recovery, matching, promotion, production evidence, machine identity, Library, Material Tests, Test Grids, Projects, accounting, Inventory, Pricing, schemas, localStorage key, legacy backup handling, SVG contracts, filenames, LightBurn colors, kerf conventions, and offline behavior remain unchanged. Old backups without `tabletopCouponResults`, round trips, malformed records, local-wins merge, replace, and recovery remain covered by the existing raw/storage fixtures. No schema migration, legacy backfill, automatic apply, global default, product card, recommendation, Shell-proven, Production-proven, or T2 behavior was added.

## 9. Direct file and accessibility validation

`python -m html.parser index.html` passed, `git diff --check` passed with only existing line-ending warnings, and inline JavaScript executed in disposable headless Edge with no page exceptions and no network dependency. The recorder, saved-result view, and promotion review use the explicit `None` alternate state rather than blank punctuation or a misleading required label. The existing direct UI path remains intact; the new no-alternate behavior is directly fixture-covered through the same save and promotion helpers.

## 10. Decisions and readiness

- Optional-alternate documentation: corrected in README, CHANGELOG, and the implementation report.
- Direct successful no-alternate fixture: present in raw-results and evidence-promotion groups.
- Dead fallback: removed; generated identity now fails closed on missing pattern data.
- Production SVG bytes: protected pins remain unchanged.
- T1.5: ready to commit after user review; nothing was committed here.
- Joe may record the real `2.88 mm` physical result after the T1.5 correction is committed, using the real machine/material context; the app still does not claim the physical cut has happened.
- T2 remains blocked until the complete physical shell validation gate is satisfied.
- No additional Claude or Grok review is necessary: the corrections are local, optional alternate behavior is directly covered, documentation matches runtime, all groups are green, and protected outputs remain unchanged.

## 11. Remaining polish

The deferred promoted-button relabeling polish was not changed because it is not required for the audit correction and could widen the promotion-surface diff. Existing non-fatal browser preview warnings are outside this bounded correction.
