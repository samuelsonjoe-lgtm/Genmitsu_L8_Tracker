# Beginner Clarity Implementation — 2026-07-19

## Scope

Implemented the bounded beginner-clarity release-readiness phase on baseline `20325e779689d7c37fabfbec1fdb23ad5c14459e` (`Improve empty states and unavailable references`). The work remains a single-file offline application change plus release documentation; no storage/schema migration, external dependency, or laser-control capability was added.

## Delivered behavior

- Import now opens an accessible three-choice application dialog: **Merge with current data**, **Replace current data**, or **Cancel import**. Replace opens a second explicit confirmation before the existing replace path runs. Merge and Replace retain their established machine-identity semantics, and Cancel/Escape/close leave data untouched.
- Reference table actions now say **Copy to Library**. The Library introduces Material Tests as confirmation/refinement/documentation of one setting and Test Grids as a matrix comparison tool; its match helper is hidden while the Library is empty.
- Help and first-run wording explain Test Grids, Material Tests, winners, evidence, promotion, preferred proven settings, reference starting points, and physical-validation limits. They also distinguish the current workshop machine, Reference profile, and Test Grid machine snapshot. The Tracker explicitly does not control the laser.
- Designs result presentation now gives text-only LightBurn layer guidance: red paths are cuts, blue paths are score/engrave guides when present, and users must review layer assignments before running. Production SVG generation and bytes are unchanged.

## Verification

- `python -m html.parser index.html` — passed.
- `git diff --check` — passed.
- Edge fixture harness — beginner clarity `22 / 0`; complete suite `2205 / 0` (24 complete-suite groups). Existing Design geometry remains `1093 / 0`; structured Tray fixtures remain `264 / 0` when called separately.
- Direct `file://` Edge run at a 320 px emulated viewport completed the `?selftest=beginner` route at `22 / 0`, with a 305 px document width and no horizontal overflow.
- The beginner fixture covers import choices and cancellation, second Replace confirmation, merge/replace machine behavior, rejected imports, Reference wording, empty-Library ordering, Test Grid/Material Test/Help copy, first-run no-control wording, machine/reference/grid distinction, and screen-only Design guidance without SVG mutation.

## Boundaries retained

No production SVG bytes, dimensions, geometry, formulas, downloads, storage keys/schema, import/export format, persistence behavior, manufacturing claims, laser communication, external URLs/dependencies, or unrelated application workflows were changed. No validation against a physical laser was performed; users still must verify material safety, scale, operations, focus, framing, and physical results before running a job.
