# T4-Status — Workshop-ready Implementation

Date: 2026-07-22  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `1d2d860 Add T3B complete tray evidence workflow`

## Repository state and scope

Started on synchronized `main...origin/main` at `1d2d860d46a55bbdd95ae70906eff066ab24b1cc`, with no staged or tracked changes. Existing unrelated untracked reports, LightBurn files, utilities, and debug artifacts were preserved. No real browser profile, saved evidence, or physical record was accessed.

Changed files are `index.html`, `README.md`, `CHANGELOG.md`, and this report only. Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## Delivered maturity contract

`tabletopTemplateMaturityRegistry` is frozen source metadata keyed by all three required identity fields:

- `templateId`: `tabletop-storage-tray`
- `generatorVersion`: `tabletop-storage-tray-t1`
- `construction`: `t2-floor-owned-shell-plus-bottom-supported-four-rail-removable-panel-t1`

Only that exact identity returns `workshop-ready` / `Workshop-ready`; an altered or future construction returns the safe default `prototype` / `Prototype`. The lookup neither reads nor writes localStorage, raw results, Library evidence, backups, imports, or user-editable state.

The template selector now contains five native optgroups. T1, T2, and T3A remain in `Coupons and prototypes`; only T3B is in `Workshop-ready`. All 15 existing option values remain present once.

## T3B presentation and evidence boundaries

The T3B summary displays Workshop-ready with a scoped explanation: it covers review of this construction, validation, and evidence workflow, while the four evidence rows still describe the current exact configuration. A fresh session therefore shows Workshop-ready plus four Unproven rows; exact complete-tray evidence shows only `Complete Tray-proven — Exact`; a mismatch or revoked evidence returns that row to Unproven without changing maturity.

The four independent boundaries remain outer shell, storage mechanism, complete tray, and cork fit. Cork wording now explains that cork lining is not part of the base template and requires separate fit evidence. No cork geometry, input, export, or T3C behavior was added.

T3B no longer exposes `metrics.prototypeOnly`. Source inspection found that field was neither persisted, exported, rendered, nor consumed by production behavior; its consumers were fixture assertions. T3B now has `metrics.requiresPhysicalVerification: true`. T1, T2, and T3A prototype/coupon contracts remain unchanged.

The T3B result includes the requested download caution, Help now defines Prototype/coupon, Workshop-ready, Exact, Unproven, configuration mismatch, Complete Tray-proven, and the template-maturity/physical-proof separation. Project handoff adds presentation-only Workshop-ready text in existing notes; it adds no Project field or schema data.

## Protected boundaries

No geometry calculation, panel ordering, production serializer, filename, SVG MIME type, evidence matcher, promotion rule, storage/schema/backup/import/merge behavior, or Project schema changed. T2, T3A, and T3B evidence meanings remain unchanged. T3B production output remains `2860` bytes / FNV-1a `3e256fad`.

## Validation

- `git diff --check`: passed.
- Python `html.parser`: passed.
- Browser runtime execution parsed inline JavaScript without page errors in focused and interactive checks.
- Duplicate IDs: none; malformed-markup parser errors: zero.
- Focused T4 maturity group: `15 / 0`.
- T3B generator: `37 / 0` (one replacement-metadata assertion added).
- T3B-E1 complete-tray evidence: `66 / 0`.
- Designs geometry/production goldens: `1093 / 0`.
- Modal Accessibility: `37 / 0`; Designs-to-Projects handoff: `17 / 0`; Storage recovery: `15 / 0`.
- All 46 unique registered fixture groups: `3003 / 0`.

Correction (post-implementation verification): this report originally published `3048 / 0` by arithmetic reconciliation (`3032` baseline + `15` new T4 assertions + `1` T3B metadata-contract assertion). Independent post-implementation verification recalculated the unique registered-group total as `3003 / 0` across 46 groups. `3048` was a documentation arithmetic miscount, not a changed fixture result; the focused T4 `15 / 0`, T3B `37 / 0`, T3B-E1 `66 / 0`, and Designs geometry `1093 / 0` totals remain correct.

Direct disposable headless Edge `file:///.../index.html?selftest=all` reached `document.readyState === "complete"`. Direct UI interaction verified the five selector groups, T3B placement only in Workshop-ready, fresh unproven rows, maturity explanation, cork wording, export caution, Help text, functional Project-handoff notes, reload, and no duplicate IDs. The normal all-route emitted four existing Edge console diagnostics, `Error: <svg> attribute height: Expected length, "auto"`; these are screen-preview warnings already present before this T4 change. Focused and UI-only runs had no console errors or page errors.

## Production goldens

All registered golden coverage passed unchanged, including the required pins: legacy pocketed shell `2181 / 2ef9606b`; T1 coupon `2992 / 4f543f95`; T2 shell `2337 / ed5d6f6e`; T3A `897 / b5c549ee`; T3B `2860 / 3e256fad`; Dice default `1726 / 51a55721`; alternate Dice `1054 / 41697123`; Divider default `1965 / a55dda6e`; and wall-to-base coupon `1551 / d9ffc278`.

Additional registered goldens exercised by the complete Designs suite include the Dice/Divider boundary matrix, Dice underside-cover `1778 / 8e2ea3f4`, J2 concealed cleat `4457 / 88721533`, J2.5 full-box cleat `9701 / 2316430b`, finger-box and sliding-lid variants, and Drawer Cabinet one/two/three-row plus guide variants. No production byte changed.

## Unverified physical boundaries and readiness

No new physical cutting was performed. The exact reference build does not prove other sizes, materials, machines, thicknesses, clearances, load capacity, long-term durability, adhesive suitability, or cork fit; users still need material checks, appropriate tests, and a dry fit before gluing.

The implementation is ready to commit: tests are green, the maturity state is static and identity-specific, evidence remains independent, and protected production output is byte-identical.
