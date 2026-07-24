# Tabletop Token & Dice Tray implementation

Date: 2026-07-24

## Scope and repository state

- Starting HEAD: `d1a06e7` — Clarify focus and Z offset terminology.
- Starting branch state: `main...origin/main`; no tracked modifications or staged files.
- Pre-existing untracked LightBurn Projects, debug artifacts, utilities, and historical reports were preserved.
- Controlling review: `docs/FINISHED_VIEWS_PRACTICAL_GENERATOR_ARCHITECTURE_REVIEW_2026-07-23.md`.
- Changed files for this phase: `index.html` and this report only.
- Final tracked `git diff --stat`: `index.html | 185` with 163 insertions and 22 deletions; this new untracked report is the second intended changed file.

## Delivered generator

- User-facing name: **Tabletop Token & Dice Tray**.
- Template ID: `tabletop-token-dice-tray`.
- Generator version: `tabletop-token-dice-tray-t1`.
- Status: Prototype / starting point.

The deferred stacking/nesting concept is not represented in the name, summary, Finished View, handoff, or output. This phase adds no lip, nesting interface, lid, divider, hardware hole, lift notch, finger hole, second bay, insert, or liner cut geometry.

The generator is additive under the existing **Trays** selector group. It is deliberately absent from `tabletopAccessoryProductRegistry`, Workshop-ready maturity, evidence/promotion matching, and all storage schemas.

## Construction and defaults

The new top-level builder composes, without changing, `buildTabletopRectangularShellDesignResult` for the T2 closed-corner shell and `buildTabletopStorageFitCouponDesignResult` for the T3A rails and removable false bottom. It combines and re-lays out their returned panels through `layoutDesignPanelRows`, then serializes through `serializeTabletopAccessorySvg`.

The exact starting defaults are:

- Interior: 120 × 90 × 32 mm.
- Material thickness: 3 mm.
- Joint clearance: 0.00 mm.
- Preferred finger width: 10 mm.
- Rail height: 12 mm.
- False-bottom thickness: 3 mm.
- Total false-bottom clearance: 0.80 mm.
- Panel spacing: 5 mm.
- Liner: cork, 0.50 mm per side allowance, 2 mm thickness.
- Assembly labels: off, matching the existing Tabletop Accessories default.

All fourteen additive preferences are prefixed `tabletopTokenTray`. A legacy token-tray draft missing every one normalizes to these defaults without changing existing-template normalization.

The exact physical production panel list is:

1. `FLOOR` — 126 × 96 mm
2. `FRONT` — 126 × 35 mm
3. `BACK` — 126 × 35 mm
4. `LEFT` — 96 × 35 mm
5. `RIGHT` — 96 × 35 mm
6. `RAIL-FRONT` — 100.615385 × 12 mm
7. `RAIL-BACK` — 100.615385 × 12 mm
8. `RAIL-LEFT` — 68.666667 × 12 mm
9. `RAIL-RIGHT` — 68.666667 × 12 mm
10. `FALSE-BOTTOM` — 119.2 × 89.2 mm

The result reports 0.80 mm total / 0.40 mm per-side false-bottom clearance, 17 mm visible wall above the false bottom, and 12 mm hidden-storage height. The exterior envelope is 126 × 96 × 35 mm.

## Liner and output contract

Supported liner preferences are `none`, `cork`, `felt`, `leather`, and `generic-liner` (shown as “Other verified liner”). An active liner is reported only as a separate measurement allowance: default cork is 118.2 × 88.2 × 2 mm. It rests on the false bottom and affects usable-height reporting only.

No liner panel, cut path, score/engraving path, label, SVG document, or download is created. Active-liner results display this advisory:

> The liner is a measurement allowance only and is not included in the production SVG. Verify its composition, backing, adhesive, ventilation requirements, and fire safety before placing it in or near the laser.

With liner type `none`, inactive clearance and thickness preferences do not affect validation, summary, Finished View, or production bytes.

One exact-scale millimeter SVG is produced. The default row layout is 277 × 324.2 mm with ten panels on one sheet. It is below the existing generic 400 mm warning threshold, but it exceeds a nominal 305 mm stock height; no dimension was reduced and no scaling was introduced, so default 305 mm stock fit remains unverified.

New production-golden scenarios:

| Scenario | Bytes | FNV-1a |
| --- | ---: | --- |
| Default cork liner | 3036 | `a86aea09` |
| Liner none | 3036 | `a86aea09` |
| 4 mm material | 2980 | `94e12bcf` |
| 0.10 mm joint clearance | 3060 | `b2c3b6ab` |
| Assembly labels enabled | 8617 | `7ac9570d` |

Only the new generator has new golden pins. Focused comparisons confirmed that T2, T3A, and T3B SVG output remains byte-stable before and after token-tray generation; no existing builder or Finished View body was changed.

## User surfaces

The compact form is ordered as interior size, material/joint fit, hidden-storage construction, liner allowance, and layout/labels. Liner clearance and thickness fields are hidden when liner type is `none`.

`buildTabletopTokenDiceTrayFinishedViewSvg` uses `tabletopAccessoryProjectPoint` and actual `result.finishedView` metadata. It shows the closed-corner shell, four rails, removable false bottom, hidden-storage region, and active thin liner; it is deterministic, screen-only, aria-labelled, mismatch-safe, finite-coordinate checked, and excluded from the downloaded SVG.

The generic preview selector offers Cut Layout and Finished Assembled. Changing preview mode does not alter production bytes.

`projectDraftFromDesign` now produces an actual-dimension name, for example `Tabletop Token & Dice Tray — 120 × 90 × 32 mm`, with notes covering prototype status, construction, hidden storage, liner measurement-only status, thickness, clearance, finger width, and coupon/physical-prototype requirements. It does not save a Project or create evidence.

## Fixtures and validation

The dedicated `runTabletopTokenDiceTrayFixtures` layer has 35 assertions covering registration, defaults, live form readback, legacy defaults, exact panel IDs/dimensions, finite closed/non-overlapping layout, composition/output stability, parameter boundaries, liner isolation, Finished View behavior, handoff non-mutation, storage/schema constants, and the five new production pins. The Designs selector and T4 grouping fixtures were updated only to recognize the additive sixteenth template option.

Fresh observed results:

- Tabletop Token & Dice Tray: **35 passed / 0 failed**.
- Gift Box regression group after restoring its pre-existing defaults: **69 passed / 0 failed**.
- Designs geometry and production-golden group: **1094 passed / 0 failed**.
- Normal `file://...?selftest=all` route: **3207 passed / 0 failed**.

Commands/methods run:

- `git diff --check`
- `python -m html.parser index.html`
- Inline source execution/parsing through direct `file://` Edge sessions (the local `node --check` executable is unavailable).
- Focused token-tray, Gift Box, T4, and complete Designs fixture routes.
- The normal complete registered suite through its own scheduling, with in-memory-only aggregate observation.
- Disposable direct `file://` Edge checks at 1440, 1024, 768, and 480 px.

Direct-file checks confirmed the template appears once under Trays; defaults are present; Cut Layout and Finished Assembled render; hidden storage and false bottom are communicated; cork is screen-only; none hides the liner fields and Finished View layer; felt updates the reported allowance; only liner type/thickness leaves cut SVG bytes identical; invalid width blocks output; no horizontal overflow was introduced at the tested widths; no network request or page error occurred during those checks.

The complete-suite browser run completed without fixture failures. Its console still reported four pre-existing `<svg>` `height="auto"` browser messages; they produced no page exception or failed assertion. The direct new-generator UI checks had no console error.

## Compatibility and remaining validation

`STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, encrypted/vault formats, synchronous persistence, backups, imports, Projects, Library, Inventory, Test Grid, promotion, existing generator IDs, evidence identities, and existing production filenames remain unchanged. No data is persisted by generating, viewing, or drafting a Project from this template.

No physical validation was performed or implied. Remaining work is physical: measure actual material, run the applicable joint-fit coupon, validate the shell/corners, verify rail seating and glue behavior, confirm false-bottom flatness/removal/reseating, cut the full prototype, then separately test the chosen liner and record results through the ordinary workflow.

No files were staged, committed, or pushed.
