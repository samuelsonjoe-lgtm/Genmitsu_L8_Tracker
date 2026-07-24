# Inventory Organization Phase 1 — Material Cleanup Implementation

Date: 2026-07-23  
Starting HEAD: `a4d4c06 Add encrypted local vault enrollment`

## Starting state and design boundary

The tracked worktree was clean at the start, with no staged changes. Pre-existing untracked reports, LightBurn project material, utilities, and debug files were present and protected; none were edited, moved, staged, or removed. The controlling design document was [Inventory Organization Material Cleanup Architecture Review](INVENTORY_ORGANIZATION_MATERIAL_CLEANUP_ARCHITECTURE_REVIEW_2026-07-23.md), especially sections 5, 10, 12, 14, and 15.

This pass changes only `index.html` and this report. It adds four derived-only Inventory **Manage**-mode behaviors:

1. thickness sorting;
2. thickness-aware search;
3. raw-material delete reference information; and
4. read-only possible-cleanup suggestions.

Inventory Browse mode, its category/material/size tree, related-record display, and safety behavior were not changed.

## Implementation

### Thickness sort and search

`inventoryRawSearchHaystack()`, `inventoryRawMatchesSearch()`, and `sortRawMaterialsByMode()` reuse the existing `parseInventorySizeMetadata()` result. The new stored preference value is `thickness-asc`, labelled **Thickness: Thin to Thick**.

Confident parsed numeric thicknesses sort first by normalized `thicknessValue`, then by material name. Missing, ambiguous, and unparseable thicknesses follow and sort by material name. Existing sort values retain their prior comparisons. The derived search haystack preserves name, category, unit, supplier, notes, aliases, and tags, and adds the parser's display label plus its compact form so supported `3 mm` / `3mm` formatting variations match without rewriting stored names.

No raw-material thickness field, parser, migration, or schema field was added. Selecting the new preference may persist `inventoryRawSort: "thickness-asc"`; rendering alone does not persist or rewrite any record.

### Delete reference information

`rawMaterialReferenceSummary()` and `rawMaterialDeleteWarning()` run only for raw-material deletion. They reuse `libraryProfilesForInventory()`, `projectsForInventory()`, and exact production-setting `materialCondition.inventoryItemId` comparison to report distinct counts for:

- matching Library profiles;
- matching Projects; and
- exact Library production-setting links.

Unreferenced material deletion keeps the concise existing-style warning. Referenced deletion explains that saved Library and Project records are not modified, name-based suggestions may stop matching, and exact production-setting links become unavailable while retaining their frozen last-known material information. The existing delete and eight-second undo behavior remains intact. `productionInventoryLinkState()` was not changed.

### Possible material cleanup

`possibleRawMaterialCleanupGroups()` is a pure, non-mutating detector. The Manage panel is a native collapsed `<details>` element and is omitted when there are no candidate groups. Each group gives only existing Edit actions and states that users must review composition, finish, dimensions, supplier, and notes before changing anything.

Candidate connections are deliberately conservative:

- exact normalized name identity, including case/spacing/punctuation differences and sheet/sheets forms;
- exact normalized name-to-established-alias identity; or
- equal `browserMaterialBaseName()` plus equal confident parsed thickness.

It does not use fuzzy distance or substring matching. Distinct short names, compositions, and finish variants remain separate unless one of those exact rules applies. There are no merge, normalization, deletion, archival, or dismissal actions.

## Protected boundaries

Unchanged: `STORAGE_KEY`, `SCHEMA_VERSION`, backup and encrypted-backup formats, vault keys/enrollment behavior, Inventory normalizers and record fields, import/merge/replace behavior, CSV export columns, Project/Library/Test Grid/Design/Pricing schemas, production Inventory dangling-link behavior, and all Browse-mode helpers.

No raw-material or finished-batch stored bytes change merely from rendering. Backups and exports remain structurally unchanged. The only permitted stored difference is the existing preference storing the new `inventoryRawSort` value after a user selects it. No Designs geometry, production SVG serialization, filenames, or output bytes were changed.

## Fixtures and validation

`runInventoryOrganizationFixtures()` is one additional cumulative layer on the established Material Browser chain; the `materials` route and dispatcher remain unchanged. It adds 36 assertions for sort ordering, parser-aware search, conservative candidate groups, composition/color false-positive protection, non-mutation, reference counts and wording, real delete/undo with dangling-link retention, legacy compatibility, Merge/Replace preservation, preference reload, and disposable in-memory derived behavior.

Commands and checks performed:

- `git diff --check` — pass.
- `python -m html.parser index.html` — pass.
- Browser JavaScript parse/execution through direct `file://` Edge — pass; no page exceptions.
- Focused direct `file://` Edge routes, each in a disposable profile:

| Group | Result |
| --- | --- |
| Material Browser / Inventory Organization | 93 passed / 0 failed |
| Empty state and dangling reference | 60 passed / 0 failed |
| Storage recovery | 15 passed / 0 failed |
| Security S1 crypto | 21 passed / 0 failed |
| Security S2 encrypted backup | 17 passed / 0 failed |
| Security S3 local vault | 37 passed / 0 failed |
| Designs geometry and protected production goldens | 1093 passed / 0 failed |

The unmodified direct `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` route completed normally with **0 failures**. A second, external in-memory response observer added only a temporary pass counter (the repository file was not written or changed) and measured the normal runner at **3160 passed / 0 failed across 49 registered groups**.

The normal Inventory direct-file startup had no console or page errors. The full-suite run emitted four SVG `height="auto"` browser diagnostics while the Designs fixtures executed; it had no page exception and still completed 0-failure. That diagnostic is outside this Inventory-only scope.

## Direct-file Manage-mode checks

A disposable headless Edge profile seeded raw materials, one finished batch, one matching Library profile and exact production setting, and one matching Project before startup. It verified:

- parsed thicknesses sort before an unspecified record;
- `3 mm` search finds stored `3mm` material;
- the cleanup panel appears collapsed, toggles with native keyboard Enter, and its Edit action opens the existing raw-material form;
- an unreferenced material gets concise delete copy;
- a referenced material reports one Library profile, one Project, and one Library production setting, then deletes and restores through Undo;
- `thickness-asc` persists across reload;
- raw materials and the finished batch return byte-identically after Undo; and
- Browse mode still renders its existing material-browser layout.

The browser profile and its localStorage were disposable. No real user record or vault data was used.

## Remaining validation boundaries

No real user Inventory was altered. Physical material identity, composition, finish, dimensions, supplier, and notes remain a human review decision; the panel intentionally does not establish duplication. Cross-browser manual testing beyond headless Edge was not performed.

## Final state

Expected changed files are only:

- `index.html`
- `docs/INVENTORY_ORGANIZATION_MATERIAL_CLEANUP_IMPLEMENTATION_2026-07-23.md`

Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.
