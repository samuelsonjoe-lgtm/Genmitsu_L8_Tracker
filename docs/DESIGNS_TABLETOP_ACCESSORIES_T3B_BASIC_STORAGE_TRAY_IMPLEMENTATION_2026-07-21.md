# Tabletop Accessories T3B — Basic Storage Tray Implementation

Date: 2026-07-21  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `12c4304 Add T3A storage fit evidence workflow`

## Scope and repository state

The implementation began on `main` at `12c4304`, synchronized with `origin/main`. There were no staged or tracked changes. Existing unrelated untracked reports, LightBurn project files, utility files, and debug artifacts were preserved without staging, moving, deleting, or otherwise modifying them.

This phase adds only the dedicated T3B Designs template and its documentation. It does not alter T1, T2, or T3A component generators; `buildBoxModel`; the generic serializers/layout utilities; shell-result or storage-result workflows; evidence matchers or promotion; storage/schema/backup/import/export; or any non-Tabletop template.

## Delivered template

- Template identity: `tabletop-storage-tray`
- User-facing name: **Tabletop Storage Tray**
- Whole-generator identity: `tabletop-storage-tray-t1`
- Construction identity: `t2-floor-owned-shell-plus-bottom-supported-four-rail-removable-panel-t1`
- Composition: the existing Floor-owned T2 shell plus the existing T3A bottom-supported four-rail removable-panel mechanism.

The T3B builder creates adapted component drafts and calls the existing T2 shell and T3A builders. It reuses their generated panel paths unchanged, then lays out the coordinated ten-piece result in these stable role rows:

1. `FLOOR`
2. `FRONT`, `BACK`
3. `LEFT`, `RIGHT`
4. `RAIL-FRONT`, `RAIL-BACK`
5. `RAIL-LEFT`, `RAIL-RIGHT`
6. `FALSE-BOTTOM`

The standard default is valid at 80 × 60 × 25 mm requested interior, 2.88 mm measured shell thickness, 3 mm nominal shell thickness, -0.075 mm joint clearance, 10 mm rails, 2.88 mm false bottom, and 0.40 mm total panel clearance. It reports a 85.76 × 65.76 × 27.88 mm finished exterior, 79.6 × 59.6 mm false bottom, 2.68 mm rail overlap, 12.12 mm remaining visible wall, and a 206.52 × 296.12 mm production layout.

The form exposes the bounded T3B dimensions, material identity, optional labels, and machine context. Invalid numeric, undersized shell, inadequate rail, excessive-bottom, and invalid-clearance configurations block output. It intentionally offers no complete-tray physical-result recorder or promotion action.

## Output and presentation boundaries

The production file remains a single exact-scale red-cut SVG with ten closed paths. The pinned T3B output is:

| Output | Length | FNV-1a |
| --- | ---: | --- |
| T3B Tabletop Storage Tray | 2,860 bytes | `3e256fad` |

The interactive browser check intercepted the actual download and reconfirmed the same length, hash, ten paths, and absence of `tabletop-storage-tray-shell` / `storage-tray-false-bottom` screen identifiers.

`Finished Assembled` is a separate screen-only SVG. It shows the T2 shell, all four floor-supported rails, the removable false bottom, and the storage cavity. The preview resolver was explicitly updated so the new template reaches this view instead of falling back to Cut Layout. Neither preview markup nor its identifiers enter the downloaded production SVG.

Existing golden coverage remains green through the Designs geometry fixtures. Relevant component pins remain T1 coupon `2992 / 4f543f95`, legacy pocketed shell `2181 / 2ef9606b`, and T2 closed-corner shell `2337 / ed5d6f6e`; the broader fixture group verifies every existing production golden without changing their bytes.

## Evidence and Project handoff

T3B adds no aggregate tray evidence. Its result summary and Project handoff make the scope explicit:

- **Outer shell:** `T2 Shell-proven — Exact` only for an exact qualifying shell result and frozen evidence identity.
- **Storage mechanism:** `Storage Fit Coupon-proven — Exact` only for an exact qualifying T3A result and frozen evidence identity.
- **Cork fit:** always `Unproven` in T3B.
- **Complete Tabletop Storage Tray:** always `Unproven` in T3B.

Focused mismatches cover requested width/depth/height, measured thickness, joint clearance, preferred finger width, material profile, shell generator/construction identity, rail height, false-bottom thickness, panel clearance, nominal thickness, and Floor ownership. A T2 match cannot stand in for T3A proof, and a T3A match cannot stand in for T2 proof.

The generic Project handoff remains a reviewable, unsaved draft. It records the two component identities, dimensions, measured and nominal thickness, clearance, rail and false-bottom facts, machine/material snapshots, and separately scoped evidence statements in notes; it adds no Project fields or schema.

The focused fixture compares the evidence-bearing state, `localStorage`, and `backupObject()` before and after generation, Finished View creation, and Project handoff. They remain byte-identical. `SCHEMA_VERSION` remains 3 and no T3B Evidence kind or persisted collection was added.

## Validation

Completed checks:

- `git diff --check`
- `python -m html.parser index.html`
- disposable direct `file://` Edge checks of the T3B form, native minimum validation, Finished Assembled selector, and intercepted production download
- focused T3B fixtures: **36 passed / 0 failed**
- T2 rectangular shell: **93 / 0**
- T2 closed corners: **13 / 0**
- T3A storage-fit coupon: **23 / 0**
- T3A storage results: **14 / 0**
- T2 shell evidence promotion: **16 / 0**
- T2 shell evidence matching: **16 / 0**
- Designs-to-Projects handoff: **17 / 0**
- Designs geometry and production-golden coverage: **1,093 / 0**
- normal fresh disposable Edge `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` runner: **2,964 passed / 0 failed across 44 registered groups**

The normal application runner was used rather than treating a post-run external fixture re-invocation as the suite result. The latter can change fixture setup ordering; in particular, it made the pre-existing T1 matching fixture appear to fail, while the fresh normal runner recorded that group at 21 / 0 and recorded no failed terminal fixture logs.

## Changed files

- `index.html`
- `README.md`
- `CHANGELOG.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T3B_BASIC_STORAGE_TRAY_IMPLEMENTATION_2026-07-21.md`

Nothing was staged, committed, or pushed.

## Remaining physical validation and recommendation

This is a prototype-generation feature, not a completed fit or safety claim. Physical work remains to assemble and square the T2 shell, verify actual material/focus/kerf behavior, dry-fit and bond the four rails, confirm the false bottom rests flat and lifts/reseats cleanly after cure, and defer cork fit to a later T3C phase. Component proof must not be read as proof of the complete tray.

A focused independent T3B audit is recommended before committing. After that audit, Joe may cut a first **prototype** T3B tray for the stated physical checks; the implementation does not authorize a Production-proven or production-safe claim.
