# Inventory Organization and Material Cleanup — Architecture and Workflow Review

Date: 2026-07-23
Repository: `C:\Genmitsu L8 Tracker`
Reviewer role: read-only architecture and workflow review (Claude Sonnet 5)
Product files modified by this review: **none**
Authorized output only: this report

---

## Executive summary

Inventory already has more organizational infrastructure than a first read of the tab suggests: a full "Material Browser" (Browse mode) groups raw materials by category → base name → parsed size/thickness, shows aliases, tags, safety warnings, and related Library/Project records, and offers a non-destructive "Use canonical name" suggestion elsewhere in the app. What it is **missing** is much smaller than it looks: Manage mode has no thickness-aware sort, no duplicate/near-duplicate visibility, and no warning when deleting a raw material that other records still refer to by name.

Every reference from Projects, Test Grids, and Designs-to-Project handoffs into Inventory is a **name-string snapshot taken at creation time**, not a live foreign key — so renaming or deleting a raw material cannot corrupt those records structurally. The one place a true persisted ID reference exists (`materialCondition.inventoryItemId` on Library production settings) already degrades gracefully when the target is missing (a `dangling` flag and a frozen last-known name), which is the existing pattern this phase should extend rather than replace.

**Recommended first phase** (detailed in §12): add a numeric/derived thickness sort to Manage mode, surface "referenced by N Projects / N Library settings" before a raw-material delete, and add a read-only, suggestion-only near-duplicate panel — all computed from data already in memory, none of it requiring a schema change, a migration, or a change to stored or backup bytes.

---

## 1. Actual HEAD and working-tree state

| Item | Observed |
| --- | --- |
| HEAD | `a4d4c069ea21947692bd9f8bf0250fac8f843bd2` |
| Subject | `Add encrypted local vault enrollment` |
| Branch | `main` |
| Upstream | `main...origin/main`, **0 / 0** (synchronized) |
| Tracked working-tree diff | **None** — `git diff --stat` and `git diff --check` both empty |
| Staged changes | None |
| Untracked files | All pre-existing (historical `docs/*.md` reports, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`) — none touched by this review, none related to Inventory |

The stated baseline (`a4d4c06`) matches the actual repository state exactly; the working tree is genuinely clean, not merely assumed clean. Security development (S1 → S2 → S3-1 foundation → S3-2 enrollment) is fully committed and out of scope for this review, consistent with the prompt's framing that security work stops here.

---

## 2. Files and functions inspected

All inspection was read-only (`Read`/`Grep`/`Bash` with no write flags) against the single `index.html` (15,194 lines). No other product file exists for this app (single-file architecture); `README.md` and `CHANGELOG.md` were not modified.

Functions and regions read in full or in relevant part:

- **Schema/state**: `inventoryDefaults`, `normalizeInventory`, `freshState`, `liveStateFromData`, `backupObject`, `persist` (inventory fields in the persisted envelope)
- **Raw material record**: `normalizeRawMaterial`, `rawMaterialCategories`, `rawMaterialCategoryOptions`, `categoryDisplayLabel`, `inventoryStarterMaterials`
- **Finished batch record**: `normalizeFinishedBatch`, `batchStatuses`, `batchRemaining`
- **Identity/matching**: `rawMaterialIdentityKeys`, `materialMatchKey`, `aliasesOf`, `tagsOf`, `tagArray`/`tagString`, `materialInventoryMatch`, `materialInventoryMatchHtml`, `materialSuggestionOptions`
- **Size/thickness parsing**: `parseInventorySizeMetadata`, `inventoryDualUnitThicknessIsConsistent`, `browserMaterialBaseName`, `enrichInventoryBrowserItem`
- **Browse mode (Material Browser)**: `buildMaterialBrowserTree`, `materialBrowserResultsHtml`, `materialBrowserDetailHtml`, `bindMaterialBrowserActions`, `filterInventoryBrowserItems`, `findInventoryBrowserItem`, `resolveMaterialBrowserSelection`/`resolveBrowserSelection`, `materialBrowserSafetyWarning`/`materialBrowserBlocksLaserActions`, `materialBrowserTestSummary`
- **Manage mode**: `renderInventory`, `inventoryManageHtml`, `inventoryBrowseHtml`, `filteredRawMaterials`, `filteredFinishedBatches`, `rawInventoryResultsHtml`, `finishedBatchResultsHtml`, `rawMaterialCard`, `finishedBatchCard`, `refreshInventorySearchResults`, `refreshInventoryBatchResults`, `bindInventoryResultActions`
- **CRUD**: `upsertInventory`, `delInventory`, `restoreDeletedItem`, `inventoryList`
- **Cross-references**: `libraryProfilesForInventory`, `projectsForInventory`, `projectRelatedFinishedBatches`, `libraryInventorySuggestions`, `projectInventorySuggestions`, `productionInventoryLinkState`, `productionInventoryOptions`, `productionInventorySelection`, `projectWizardSelectedInventory`, `projectWizardInventoryEligibility`, `projectWizardInventoryThickness`, `projectDraftFromWizard`
- **Import/export/backup**: `applyBackupImport`, `mergeData`, `mergeListStats`, `mergeList`, `replaceData`, `applyBackupPreferences`, `resetBackupPreferences`, `rawInventoryCsv`, `finishedBatchInventoryCsv`, `exportRawInventoryCsv`, `exportFinishedBatchInventoryCsv`
- **Fixtures**: `runMaterialBrowserFixtures` (original), `runMaterialBrowserMatchingFixtures`, `runMaterialBrowserRelatedFixtures`, `runMaterialBrowserCorrectionFixtures`, `runEmptyStateDanglingReferenceFixtures`
- **Storage keys/constants**: `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT` (read only for confirmation that nothing Inventory-specific has its own key)

---

## 3. Current Inventory architecture

Inventory is not a single flat list — it is two independent record collections under one `state.inventory` object:

```javascript
inventory: { rawMaterials: [...], finishedBatches: [...] }
```

This whole object lives inside the single top-level `state` document that is JSON-serialized under the one existing `STORAGE_KEY` (`genmitsu-l8-tracker-v1`, unchanged since `SCHEMA_VERSION` was introduced) — there is no separate storage key for Inventory, and no separate schema version for it. `normalizeInventory` (`index.html:1946`) is the sole shape guard: it defaults each collection to `[]` if the incoming value is not an array, and otherwise passes every item through **unchanged** (no per-item field validation at load time — that happens lazily, at edit time, via `normalizeRawMaterial`/`normalizeFinishedBatch`).

Inventory has **two independent, coexisting UI modes**, both reading the same underlying data:

1. **Manage mode** (`inventoryManageHtml`, `index.html:8283`) — a flat, filterable, sortable card grid. This is the "administrative" view: add/edit/delete raw materials and finished batches, category filter, a handful of sort keys, free-text search.
2. **Browse mode** ("Material Browser", `inventoryBrowseHtml` + `materialBrowserResultsHtml`, `index.html:8282`/`8299`) — a grouped tree (category → base material name → parsed size) with a detail panel showing stock, aliases, tags, safety warnings, matching Library settings, and related Projects. This mode already implements most of what "grouped navigation" would otherwise require building from scratch.

The active mode is a persisted preference (`state.inventoryViewMode`, normalized by `normalizeInventoryViewMode` to `'browse'`/`'manage'`), toggled by two buttons in the shared Inventory toolbar (`renderInventory`, `index.html:8317`, line 8342).

---

## 4. Current record schema and storage boundaries

### Raw material (`normalizeRawMaterial`, `index.html:13246`)

| Field | Type | Notes |
| --- | --- | --- |
| `id` | string | `uid()` on create; never reassigned by normalization |
| `name` | string | free text; trimmed only |
| `category` | string | defaults to `'other'`; **not restricted to the known list** — an imported/legacy category string is kept and displayed via `categoryDisplayLabel`'s fallback (title-cased passthrough), confirmed by fixture `'Imported category remains visible'` |
| `quantityOnHand` | number or `''` | |
| `unit` | string | must be one of `inventoryUnits`, else forced to `'other'` |
| `unitCost` | number or `''` | |
| `supplier` | string | free text |
| `purchaseDate` | string | free text/date |
| `lowStockThreshold` | number or `''` | drives low-stock badge via `rawIsLow` |
| `aliases` | **string** (comma-separated), read via `aliasesOf`/`tagArray` | see §7 — this is the existing alias mechanism |
| `notes` | string | free text |
| `tags` | array (normalized via `tagArray`) | |

`normalizeRawMaterial` spreads `...(f || {})` first, so **any additional field an import or a future version adds survives untouched** — confirmed live by fixture `'Unknown fields survive normalization'` and `'Repeated normalization stable'` (re-normalizing an already-normalized record is a no-op / idempotent).

**There is no dedicated thickness field.** Thickness/size is never parsed out into its own stored property on the record — it lives entirely inside the free-text `name` (and secondarily `notes`), and is **re-derived at render time** by `parseInventorySizeMetadata` (`index.html:1969`) whenever Browse mode needs to group or display it. This is a deliberate, already-existing "derived, display-only" pattern, not a gap that needs a schema field to close.

### Finished batch (`normalizeFinishedBatch`, `index.html:13784`)

Separate collection; status enum `batchStatuses = ['planned','made','for-sale','sold-out','archived']` — **finished batches already have an archive concept; raw materials do not.**

### Storage/compatibility boundaries confirmed unchanged and relevant to this phase

- `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, `ENCRYPTED_BACKUP_FORMAT`, vault keys — none are Inventory-specific; Inventory rides inside the same single JSON document as everything else.
- `inventory.rawMaterials`/`inventory.finishedBatches` merge on **import** the same way every other list-shaped collection does: `mergeList` (`index.html:14213`), an ID-based upsert (`index.html:14358-14359`), with `mergeListStats` (`index.html:14346-14347`) separately reporting added/updated counts for the merge-summary toast. Replace mode wholesale-assigns `normalizeInventory(data.inventory)` (`index.html:14325`). Both paths already exist, are already tested at the generic-list level, and require no change for this phase.
- `rawInventoryCsv`/`finishedBatchInventoryCsv`/`exportRawInventoryCsv`/`exportFinishedBatchInventoryCsv` (`index.html:12467-12492`) already provide a **read-only CSV export** of Inventory (not import) — worth noting only because it shows CSV plumbing already exists for export; CSV *import* remains correctly out of scope per the prompt's non-goals.

---

## 5. Cross-tab / cross-feature dependencies (the most important finding for cleanup safety)

This is the single most load-bearing fact for the whole review: **almost nothing outside Inventory holds a live pointer into Inventory.**

| Consumer | Reference mechanism | Confirmed live/frozen |
| --- | --- | --- |
| Log entries / Library profiles / Projects (`material` field) | Free-text string, matched loosely by `materialInventoryMatch`/`materialMatchKey` for **display/suggestion purposes only** | Not a stored link. Renaming or deleting the Inventory item has **zero structural effect** on these records; they simply stop matching a suggestion the next time someone looks. |
| New Project Wizard (`projectWizard.inventoryId`) | In-memory-only draft field, alive only for the duration of the wizard session | **Never persisted.** `projectDraftFromWizard` (`index.html:11311-11338`) copies only `inventoryItem?.name` into the resulting saved Project's `material` field (plus a human-readable `"Inventory: <name> (<id>)"` line inside the free-text `notes`, for audit/traceability only — not machine-read anywhere). |
| Library "production settings" material condition (`materialCondition.inventoryItemId`) | **Genuine persisted ID reference** | This is the one real foreign key in the app. `productionInventoryLinkState` (`index.html:10933-10937`) already computes `dangling: !!selectedId && !item` and the option-list renderer (`productionInventoryOptions`) already shows `"Unavailable: <frozen inventoryItemName>"` for a dangling reference. **This is the existing, working pattern for "protection against deleting referenced inventory" — it degrades gracefully today, with no crash and no silent data loss.** |
| Test Grids / promoted settings / Tabletop evidence | No Inventory reference of any kind found (`grep` for `inventoryId`/`inventoryItemId` across the file returns only the two mechanisms above) | Not coupled to Inventory at all. |
| Designs → Project handoff | Copies a plain `material`/`thicknessValue`/`thicknessUnit` snapshot into the new Project draft (same `openProject({...})` pattern used everywhere else) | Same as Projects above — a one-time string/number snapshot, not a link. |
| Pricing | `state.pricing`/`pricingPrefs` are their own top-level state slices; no Inventory ID reference found | Independent. |

**Conclusion:** because Projects/Library-`material`/Grids/Designs consumers are all decoupled name snapshots, and the one true ID reference already tolerates dangling targets, **this phase can add stronger cleanup/organization tooling without any risk of breaking a downstream record** — the only genuine risk category is a raw material a user is *actively relying on* for identification (its name), which argues for warning-before-destructive-action rather than any structural safeguard.

---

## 6. Current organization, sorting, filtering, and search behavior

### Manage mode

- Filter: category (`state.inventoryCategoryFilter`, single-select against the fixed `rawMaterialCategories` list plus whatever a legacy/imported category value already exists via `rawMaterialCategoryOptions`) + free-text search (`state.inventorySearch`).
- Search haystack (`filteredRawMaterials`, `index.html:8158-8172`): `name`, `category`, `unit`, `supplier`, `notes`, `aliases` (raw string) **and** `aliasesOf(item)`/`tagsOf(item)` (parsed arrays) — joined and lower-cased. **Does not include the parsed size/thickness display text**, so searching "3mm" only works if that literal substring happens to already be in the name/notes (which it usually is, by convention, but is not guaranteed).
- Sort (`inventoryRawSort`): exactly five options — `low-stock` (default), `name-asc`, `category-asc`, `value-desc`, `quantity-asc`. **No thickness-based sort of any kind exists in Manage mode.** Batches (`inventoryBatchSort`) similarly have four options, none thickness-related (batches don't carry thickness).
- Grouping: **none** in Manage mode — it is a flat card grid regardless of sort key. "Grouping by category" only exists as a *filter*, not a visual grouping/heading structure.

### Browse mode (Material Browser)

- Groups by **category → base material name (thickness/dimension stripped via regex) → parsed size bucket**, built fresh on every render by `buildMaterialBrowserTree` (`index.html:2001`) from `enrichInventoryBrowserItem`, which calls `parseInventorySizeMetadata` per item. This is already exactly the "grouped inventory tree" and "grouping by category/material/thickness" candidate feature the prompt asks about — **it already exists**, just only in Browse mode, not Manage mode.
- Selection state (`inventoryBrowserSelectedId`) persists across re-renders within a session and gracefully handles a selection that's been filtered out (`resolveMaterialBrowserSelection` → `hidden: true`, shows a "hidden by current filters" note) or deleted (falls back to first visible item).
- Uses native `<details>` elements for the tree (`data-browser-node`), which gives keyboard/screen-reader disclosure semantics for free without custom ARIA work.
- Detail panel shows: stock status, unit cost, inventory value, supplier, purchase date, aliases, tags, notes, a safety warning if applicable, matching Library settings (with test-evidence summary), and up to 5 related Projects with an "open Project" action — this is already "better display of records referenced elsewhere," just not present in Manage mode.

### What is genuinely missing (confirmed, not assumed)

1. No numeric/thickness-aware sort anywhere in Manage mode.
2. No grouping (only filtering) in Manage mode.
3. No duplicate/near-duplicate detection or warning anywhere.
4. No "referenced elsewhere" signal before deleting a raw material in either mode (Browse mode's detail panel shows related records only when you've already selected the item to look, not as a delete-time warning).
5. No archive status for raw materials (only finished batches have one).
6. Manage-mode search does not include parsed size/thickness text.

---

## 7. Material naming and category inconsistencies

### How the app currently handles the exact examples in the prompt

- **"Cork," "cork sheet," "Cork Sheets"** — three separate raw-material records today if entered separately; nothing merges them. `materialMatchKey` (`index.html:1858`) would normalize all three to overlapping-but-not-identical keys (`cork`, `cork sheet`, `cork sheets`) — close but not equal, so `materialInventoryMatch`'s exact-match path would treat them as distinct, while its **loose substring fallback** (`nameKey.includes(key) || key.includes(nameKey)`, gated at `key.length >= 3`) would surface "Cork" as a loose match candidate for "cork sheet" in the *typing-suggestion* UI (Log/Project material field), but this is advisory only — it never merges records or forces a single canonical spelling.
- **"Baltic Birch," "birch plywood," "Baltic birch plywood"** — same situation. The `aliases` field is the app's existing, intentional tool for this: one record's name stays authoritative, and the other spellings become entries in its `aliases` string, which is then searchable (`filterInventoryBrowserItems`, `materialSuggestionOptions`) and matchable (`materialInventoryMatch`'s alias branch) without ever rewriting anyone's already-saved `material` string elsewhere. `inventoryStarterMaterials` (`index.html:2016-2032`) uses exactly this pattern already (e.g., the Baltic birch/basswood starter item's aliases include `"birch basswood plywood, baltic birch plywood 3mm, craft plywood"`).
- **Materials under an unsuitable category** — fully manual today, fixed by editing the record's `category` field (a free-select dropdown seeded with `rawMaterialCategories` plus whatever legacy value is already present, per `rawMaterialCategoryOptions`). No bulk reassignment exists.
- **Thickness variants (3, 3.0, "3 mm," 0.118 in, 1/8 in)** — `parseInventorySizeMetadata` already recognizes a specific "dual-unit" convention (`"3 mm / 1/8 in"`) as one canonical display bucket, validates the two values are mutually consistent within a 0.2 mm tolerance (`inventoryDualUnitThicknessIsConsistent`), and falls back to `"Unspecified size"` grouping for anything ambiguous (a range, a `+`/`-` prefix, multiple metric numbers, a comma-decimal, etc. — all explicitly guarded against false-positive parsing). This is a **display/grouping heuristic only**; it does not normalize or rewrite the stored `name`.
- **Nominal vs. measured thickness** — Inventory has no concept of this distinction at all today (it belongs to the Tabletop Accessories evidence system, which tracks *measured* dimensions on physical test coupons, a completely separate data model from Inventory stock records). Inventory's parsed thickness is whatever number is in the name — effectively "nominal, as the user typed it."
- **Missing thickness** — `parseInventorySizeMetadata` returns `{ displayLabel: 'Unspecified size', groupingKey: 'unspecified', thicknessConfidence: false }`, which groups all such items together in Browse mode rather than scattering them or crashing.
- **Duplicate or near-duplicate records** — no detection exists anywhere today. Two records with identical or near-identical names simply both appear (Browse mode's tree would place them in the same base-name/size bucket if the names parse to the same base+size, which is a form of *visual* grouping, but there is no explicit "these look like duplicates" signal).
- **Same material name, different finish/color/vendor/composition** — this is exactly what separate records + separate names/aliases already exist to express; it is by design, not a defect. The review should not recommend collapsing these.
- **Records already referenced elsewhere** — see §5: nothing breaks structurally; the "protection" question is really "should the user be warned before deleting," not "will data become inconsistent."

### Explicit determination on cleanup mechanism (per the prompt's checklist)

| Option | Recommendation |
| --- | --- |
| Change stored values | **No**, not as an automatic/bulk action. Manual per-record edits (already possible) remain the only value-changing path. |
| Add canonical fields | **No** for this phase — no new "canonicalName"/"canonicalThickness" field. The existing `aliases` mechanism already serves this role adequately. |
| Derived, display-only normalization | **Yes** — this is already the established pattern (`parseInventorySizeMetadata`, `categoryDisplayLabel`, `materialMatchKey`) and this phase should extend it, not replace it. |
| Preserve original user-entered value | **Yes** — required, and already the existing behavior everywhere in Inventory. |
| Use aliases | **Yes** — already exists; this phase's cleanup features should surface/encourage it more (e.g., a duplicate-suggestion panel that offers "add as alias" as one non-destructive action), not introduce a second mechanism. |
| Require schema migration | **No** — see §11 for why this phase does not need one. |
| Remain completely manual | **Partially** — detection/surfacing should be automatic (computed, not stored); the actual edit remains a manual, explicit user action. |
| Offer safe suggestions without auto-modifying records | **Yes** — this is the correct model, consistent with the existing "Use canonical name" button pattern (`materialInventoryMatchHtml`, `index.html:1911`), which suggests but never forces a rewrite. |

No silent rewriting, automatic merging, or destructive deduplication is recommended anywhere in this review.

---

## 8. Thickness and unit-handling findings

- Thickness is **never a first-class stored field** on a raw material; it is always re-derived from `name`/`notes` text at render time via regex-based heuristics in `parseInventorySizeMetadata`.
- The parser already distinguishes a validated "dual-unit" bucket (mm + fractional inch, cross-checked for consistency) from a single-metric bucket from a "dimension" bucket (e.g. `90mm x 3mm`) from an explicitly-ambiguous/unparseable bucket (`"Unspecified size"`), and is deliberately conservative — it prefers "unspecified" over a wrong guess (confirmed by its guards against ranges, signs, thousands-separators, and multiple-metric-number strings).
- **Sorting is where the gap actually is.** `filteredRawMaterials`'s five sort modes never touch thickness. Adding a numeric thickness sort is straightforward because the parsing function (`parseInventorySizeMetadata`) already exists and already returns a numeric `thicknessValue` (in mm) when confidence is true — the sort would reuse it directly rather than inventing new parsing logic, and would need only a stable, sensible position for "unspecified size" items (recommendation: last, or grouped separately, never silently dropped).
- Unit display elsewhere in the app (Log/Library/Project thickness fields) already has its own established mm/in conversion (`convertUnitValue`/`thicknessIn`) and dual-unit consistency tolerance pattern reused by `inventoryDualUnitThicknessIsConsistent` — this phase should reuse that existing tolerance constant/logic rather than inventing a new one.

---

## 9. Duplicate and cleanup risks

- **Risk of false-positive merging**: any "near-duplicate" detector based on fuzzy string similarity risks conflating genuinely different materials (e.g., "Walnut plywood 3mm" vs. "Walnut veneer plywood 3mm" — different composition, similar name). The existing `materialMatchKey`/loose-substring approach already tolerates this ambiguity by only ever *suggesting*, never merging — any new duplicate detector must keep that same discipline (surface, don't act).
- **Risk of hiding real distinctions**: grouping by parsed base-name+size (as Browse mode already does) can visually cluster two materials that are meaningfully different (different finish/vendor) under one tree node if their names happen to share a base string — this is a display-grouping side effect, not a data-integrity risk, but worth calling out in any "possible duplicate" UI copy (e.g., "these share a name pattern — review before treating as duplicates").
- **Risk of a delete that surprises a user**: the real cleanup risk in this app is not data corruption (see §5) but a user permanently deleting a record whose name they were actively relying on, without being told it's still referenced. `delInventory` (`index.html:14489-14505`) already has an 8-second undo window (`showUndoToast`/`restoreDeletedItem`), which is a reasonable existing safety net, but the confirm dialog itself is generic (`"${message} You can undo for a few seconds."`) and does not mention reference counts today.
- **Risk of category-string drift**: because `category` is free-text-compatible (an imported/legacy value is preserved and displayed, not coerced into the fixed list), a large inventory could accumulate near-duplicate category strings (e.g., "wood," "Wood," "wood stuff") the same way material names can. The same suggestion-only philosophy applies here as to material names — no automatic coercion recommended.

---

## 10. Data-safety findings

1. **No schema-breaking risk found.** `normalizeInventory` is permissive (array-shape only); `normalizeRawMaterial`/`normalizeFinishedBatch` are idempotent and preserve unknown fields. Any additive change (a new derived/computed field, a new sort-mode string, a new UI-only warning) is safe against legacy records that predate it.
2. **No live foreign-key risk found** for Projects/Grids/Designs (decoupled name snapshots, per §5).
3. **One live foreign-key exists** (`materialCondition.inventoryItemId`) and already has dangling-reference handling — this phase should not weaken or bypass that handling, and any new delete-time warning should include this reference type alongside Projects/Library-by-name matches.
4. **Merge/Replace import already ID-based and tested at the generic-list level** for both `rawMaterials` and `finishedBatches` — no change needed or recommended.
5. **No inventory-specific dispatcher fixture route exists** — Material Browser fixtures are registered under the shared key `selftest === 'materials'` (`index.html:15113`), which currently resolves to the cumulative `runMaterialBrowserCorrectionFixtures` (itself built by chaining `runMaterialBrowserFixtures` → `runMaterialBrowserMatchingFixtures` → `runMaterialBrowserRelatedFixtures` → `runMaterialBrowserCorrectionFixtures`, each reassigning the same `window.runMaterialBrowserFixtures` binding to the newest superset). This is consistent with the app's existing "corrections layer on top of the previous fixture function" pattern seen elsewhere in the codebase, not a defect — but it means **any new fixtures for this phase should be added as one more layer in that same chain**, not a second competing dispatcher key, to avoid the group being silently skipped or double-counted under `?selftest=all`.
6. **No accessibility regression risk identified in Browse mode** (native `<details>` disclosure, existing modal/focus infrastructure reused for edit forms) — Manage mode's card grid uses the same shared button/form patterns as every other tab, so no Inventory-specific accessibility gap was found beyond the general absence of thickness sort/grouping already noted.

---

## 11. Options considered

| Option | Verdict |
| --- | --- |
| Add a first-class `thicknessValue`/`thicknessUnit` field to the raw-material schema, migrating existing name-embedded thickness into it | **Rejected for this phase.** Would require a migration step, a decision about handling values `parseInventorySizeMetadata` can't confidently parse (which exist by design — "Unspecified size" is a deliberate fallback, not a bug), and would create two sources of truth (the new field vs. the name text) that could drift. The derived/display-only approach already in place avoids all of this and is proven (fixtures already assert its behavior). |
| Auto-merge near-duplicate records | **Rejected**, per the prompt's own instruction and the risk in §9 (false-positive merges of genuinely different materials). |
| Bulk category reassignment tool | **Deferred**, not rejected — useful, but a genuinely separate, larger UI surface (multi-select + bulk-apply) than this "small phase" should take on; nothing here blocks adding it later. |
| Rebuild Manage mode as a grouped tree (replacing Browse mode's role) | **Rejected.** Two purpose-built modes already exist and already serve different needs (Manage = flat administrative CRUD grid; Browse = grouped read-oriented navigation with related-record context). Duplicating Browse mode's grouping logic inside Manage mode's flat grid is unnecessary; the correct fix is smaller — add sort, not restructure. |
| Add numeric thickness **sort** to Manage mode, reusing `parseInventorySizeMetadata` | **Recommended.** Additive, derived-only, no schema change. |
| Add a "referenced elsewhere" count/warning at delete time, reusing `libraryProfilesForInventory`/`projectsForInventory`/`productionInventoryLinkState` | **Recommended.** Reuses existing, already-tested matching functions; changes only the confirm-dialog copy and (optionally) the Manage-mode card. |
| Add a read-only "possible duplicates" panel, reusing `materialMatchKey`/`browserMaterialBaseName` clustering already used by the tree builder | **Recommended**, scoped strictly to suggestion display — no merge action beyond linking to the existing edit form and (if useful) a "copy as alias" convenience action that still requires an explicit save. |
| Extend Manage-mode search to include parsed size/thickness display text | **Recommended**, small and additive (`parseInventorySizeMetadata(item).displayLabel` joined into the existing search haystack). |
| Archive status for raw materials (mirroring `batchStatuses`) | **Deferred.** Useful parity with finished batches, but a status-enum change plus filter/sort updates is a second, separable unit of work; not required to make Inventory "easier to navigate, organize, search, sort, and clean up" in this first pass. |

---

## 12. Recommended bounded implementation ("Inventory Organization Phase 1")

### 1. Exact user problem being solved

Today, a shop owner with a growing raw-material list can filter and search Manage mode, but cannot sort by thickness, cannot see at a glance whether a material name might be a duplicate of another, and can delete a raw material without being told anything else in the Tracker still refers to it by name. Browse mode already solves grouping/related-record visibility, but a user who prefers the administrative Manage grid gets none of that.

### 2. Exact included behaviors

- **Numeric thickness sort** in Manage mode: a new `inventoryRawSort` option (e.g. `'thickness-asc'`) that sorts by `parseInventorySizeMetadata(item).thicknessValue` (mm-normalized) when `thicknessConfidence` is true, with all "unspecified size" items placed in a stable, clearly-labeled trailing group — never silently interleaved or dropped.
- **Referenced-elsewhere signal at delete time**: `delInventory('rawMaterials', ...)` gains a pre-delete lookup (reusing `libraryProfilesForInventory`, `projectsForInventory`, and a new `productionInventoryLinkState`-based count for Library production settings) and includes the counts in the confirm-dialog message (e.g., "This material is referenced by 2 Library settings and 3 Projects by name. Deleting it will not change those records, but they will no longer suggest this Inventory item. You can undo for a few seconds."). No behavior change to the delete/undo mechanism itself.
- **Read-only "possible duplicates" panel** in Manage mode: computed from `rawInventoryMaterials()` by clustering on `browserMaterialBaseName`/`materialMatchKey`, surfaced as a dismissible, non-blocking panel or badge ("2 materials share a similar name — review") linking to the existing edit form for each; no automatic action.
- **Extend Manage-mode search** to include `parseInventorySizeMetadata(item).displayLabel` in the existing search haystack, so "3mm" or "1/8 in" finds items even when that exact substring formatting differs slightly from the stored name.

### 3. Explicit deferred behaviors

- Bulk category reassignment
- Canonical-name rewriting/auto-merge of any kind
- A first-class thickness schema field or any migration
- Nominal-vs-measured thickness distinction (not applicable to Inventory's data model)
- Archive status for raw materials
- CSV import
- Any change to Browse mode's existing tree/grouping logic (it is already correct and should be reused, not modified)
- Any change to Library `materialCondition.inventoryItemId` dangling-reference handling (already correct)

### 4. Files and functions likely to be touched

All within `index.html`: `filteredRawMaterials` (add sort case), `inventoryManageHtml` (add sort `<option>`), `delInventory` (add reference lookup + message), `renderInventory`/`rawInventoryResultsHtml` (optional new duplicates-panel markup and a mount point), a new small pure function (e.g. `possibleDuplicateInventoryGroups()`) alongside the existing `materialMatchKey`/`browserMaterialBaseName` helpers, and `filteredRawMaterials`'s search-haystack line (extend, don't restructure).

### 5. Protected functions and storage boundaries

Do not modify: `normalizeInventory`, `normalizeRawMaterial`'s field set, `mergeData`/`mergeList`/`replaceData`, `STORAGE_KEY`/`SCHEMA_VERSION`/`BACKUP_FORMAT`, `parseInventorySizeMetadata`'s parsing rules (extend by calling it, not by changing its heuristics), `productionInventoryLinkState`'s dangling-reference contract, `buildMaterialBrowserTree`/Browse mode generally.

### 6. Whether stored inventory bytes will change

**No.** Every included behavior is either a derived computation (thickness sort, duplicate clustering, search haystack) or a new persisted **preference** string (`inventoryRawSort` gaining one more valid value, exactly like every existing sort-mode addition in this app's history) — not a change to any raw-material or finished-batch record's own fields.

### 7. Whether backup/export bytes will change

**No**, for the same reason: `backupObject()` already includes `inventoryRawSort` as a plain string preference; a new valid value for that string is not a structural change. No raw-material/finished-batch field is added, renamed, or removed.

### 8. Whether a schema or migration is required

**No.** `SCHEMA_VERSION` stays at its current value. Legacy records (missing thickness-parseable names, missing categories, missing aliases) are already tolerated by every function this phase touches or adds (confirmed by existing fixtures like `'Unknown fields survive normalization'` and the "Unspecified size" fallback).

### 9. Compatibility behavior for legacy records

A record with no parseable thickness sorts into the trailing "unspecified" group under the new sort mode rather than erroring or sorting arbitrarily. A record with a legacy free-text category not in `rawMaterialCategories` continues to display via `categoryDisplayLabel`'s existing fallback and is unaffected by the duplicate panel (which clusters on name, not category).

### 10. Handling of referenced inventory records

No new blocking behavior — deletion remains possible (this app does not lock users out of removing their own data), but the confirm dialog becomes informative rather than generic. The existing undo window remains the actual safety net.

### 11. Validation behavior

No new validation rules on raw-material fields. The duplicate panel and reference-count lookup are read-only computations over existing, already-normalized data — they must not mutate `state.inventory` or any other collection, and should be covered by a "does not mutate" fixture assertion (see §14).

### 12. Accessibility requirements

- The duplicate-suggestion panel/badge must be reachable and dismissible via keyboard, with any dismiss/expand control using a real `<button>`, and any grouping using either native `<details>` (matching Browse mode's existing pattern) or an appropriately-labeled `<section aria-label="…">` — not a div-only collapsible.
- The reference-count text added to the delete confirmation must be included in the same `confirm()`-dialog message string (or, if moved to a custom modal, must follow the existing modal accessibility pattern already covered by the Modal Accessibility fixture group) so it is read by assistive technology before the destructive action is confirmed.
- No new keyboard trap; the new sort `<option>` uses the existing `<select>` element already wired for `inventoryRawSort`.

### 13. Performance considerations

`parseInventorySizeMetadata` and `materialMatchKey` are already called once per item per render in Browse mode; reusing them for a Manage-mode sort adds the same per-item cost to Manage mode's render, which is the same order of work Browse mode already performs today for the same inventory size. Duplicate clustering is an additional `O(n)` pass with an `O(n)` (or small-`k`) grouping map — negligible until inventories reach the thousands of items, well beyond any realistic single-shop scale. No new network, storage, or async work is introduced.

### 14. Required fixtures and regression tests

Add to the existing Material Browser fixture chain (append as one more layer, per §10 finding 5, rather than a new dispatcher key):

- Legacy raw-material record (no `aliases`/`tags`/parseable thickness) sorts into "unspecified" under the new thickness sort without error.
- Numeric thickness sort orders mm-equivalent values correctly across mixed mm/in-labeled names (reusing existing `parseInventorySizeMetadata` test fixtures as a baseline).
- Millimeter and inch display values in the sorted list match the existing Browse-mode display formatting exactly (no new formatting logic introduced).
- Nominal-vs-measured: **not applicable** — assert (rather than silently skip) that Inventory has no measured-thickness concept, to catch any future accidental coupling with the Tabletop evidence system.
- Category reassignment: confirm an item's category change is reflected in both Manage's filter option list and Browse's category grouping without requiring a reload.
- Search/filter: confirm the extended search haystack matches a query like `"3 mm"` against a record whose name uses `"3mm"` (formatting-tolerant, reusing `parseInventorySizeMetadata`'s already-normalized `displayLabel`).
- Near-duplicate detection: confirm two records with names differing only by whitespace/case/pluralization ("Cork," "cork sheet," "Cork Sheets") are grouped in the suggestion panel, and confirm two genuinely different materials that happen to share a short substring are **not** falsely flagged (reusing the existing `key.length < 3` guard pattern from `materialInventoryMatch`).
- Referenced-record protection: confirm the delete confirmation message includes accurate Library/Project reference counts for a seeded fixture item, and confirm deleting it anyway leaves those other records' `material` strings and `materialCondition.inventoryItemId` (now correctly `dangling: true`) unchanged.
- Import/export round trip: confirm a backup created after this phase re-imports (Merge and Replace) with identical `rawMaterials`/`finishedBatches` content and an unchanged `inventoryRawSort` preference string.
- Backup/restore round trip: confirm the encrypted-backup (S1/S2) and local-vault (S3) paths are unaffected — Inventory data flows through the same generic `state`/`persist()` path already covered by those phases' own fixtures; this phase adds no new storage key or write path for those systems to interact with.
- Vault-enabled and vault-disabled behavior: confirm the new sort/search/duplicate features behave identically whether `vaultMode` is `'plaintext'` or `'vault-unlocked'` (they operate purely on in-memory `state`, same as every existing Inventory feature).
- Persistence after reload: confirm the new `inventoryRawSort` value round-trips through `persist()`/`loadState()` like every existing sort preference.
- Direct `file://` operation: confirm no new fixture or feature introduces a network reference, external script, or `fetch`.
- Empty inventory: confirm the duplicate panel and new sort option degrade to the existing empty-state messaging (`'No raw materials yet'`) rather than rendering an empty/error panel.
- Large synthetic inventory (e.g., 500+ synthetic raw materials): confirm sort/search/duplicate-clustering complete without a noticeable UI stall in a direct-file browser check (manual, not a numeric fixture assertion, per the "do not claim runtime results not executed" rule below).
- Accessibility/keyboard: confirm the new duplicate panel and the extended confirm dialog are keyboard-reachable and, where a custom modal is used instead of `confirm()`, pass the existing Modal Accessibility fixture group.
- No changes to unrelated production SVG or generator output: rerun the full Designs geometry/production-golden fixture group and confirm the count is unchanged, since this phase touches no Designs code.

### 15. Manual browser checks (to be performed at implementation time, not claimed here)

Direct `file://` load in a disposable browser profile, seeded with synthetic legacy-shaped raw materials (missing thickness, missing aliases, legacy category string), verifying: the new sort option orders them sensibly, the duplicate panel flags an intentionally-seeded near-duplicate pair without flagging an intentionally-seeded false-positive-risk pair, the delete confirmation shows accurate reference counts for a seeded cross-referenced item, and a reload preserves the new sort preference. None of this was executed as part of this review — this is a review, not an implementation or verification pass, and no runtime/browser/fixture result should be inferred from this report.

### 16. Clear acceptance criteria

1. `?selftest=materials` (or the chain it currently resolves to) passes with all prior assertions unchanged plus the new ones from §14, 0 failed.
2. `?selftest=all` shows no regression in any other group's count, especially Designs geometry, Storage recovery, Empty state, Modal accessibility, and the Security S1/S2/S3 groups (none of which this phase should touch).
3. A backup exported before this phase imports (Merge and Replace) correctly after this phase, and vice versa (forward/backward compatibility of the `inventory` shape and the `inventoryRawSort` preference string).
4. No raw-material or finished-batch field is added, renamed, or removed; `git diff` for this phase touches only the functions listed in §4 above.
5. Deleting a referenced raw material still succeeds (not blocked), still offers undo, and now names what else references it.
6. A legacy inventory with no thickness-parseable names, no aliases, and an unrecognized category string loads, sorts, searches, and displays with no error and no data loss.

### 17. Risks and rollback strategy

- **Risk**: the reference-count lookup at delete time double-counts or under-counts if it doesn't reuse the exact same matching functions Browse mode already uses (`libraryProfilesForInventory`/`projectsForInventory`) — mitigation is to call those functions directly rather than re-implementing matching logic.
- **Risk**: a near-duplicate panel that is too aggressive becomes noise a user learns to ignore — mitigation is the same conservative-guard philosophy `materialInventoryMatch` already uses (a minimum key length, exact-then-loose tiering) rather than a raw string-distance threshold.
- **Rollback**: because nothing here changes stored record shape, `SCHEMA_VERSION`, or storage keys, rollback is a plain code revert — no data migration to reverse, no `-pending` staging key, no forward-incompatible state to unwind. This is materially simpler than the S3 vault rollback story precisely because this phase adds no persisted structure beyond one more valid string value for an existing preference field.

---

## 13. Unverified areas

- No fixture or browser check was executed as part of this review (explicitly out of scope — this is a review, not an implementation or verification pass).
- Real-world inventory size/performance beyond the order-of-magnitude reasoning in §12.13 was not measured.
- Whether users would actually want a "possible duplicates" panel surfaced proactively vs. on-demand (a UX preference judgment, not an architecture fact) — recommend implementing it collapsed/opt-in by default and letting real usage inform whether it should be more prominent.

---

## 14. Suggested follow-up independent audit boundary

After implementation, an independent read-only verification pass should confirm, at minimum: (a) the new sort/search/duplicate features perform zero mutation of `state.inventory` or any other collection; (b) the delete-confirmation reference counts are accurate against a variety of seeded cross-reference shapes (name match, alias match, `materialCondition.inventoryItemId` match); (c) the fixture chain in §10 finding 5 was extended correctly (one more layer, same dispatcher key, no duplicate/competing registration); (d) backup/import/vault round trips are unaffected; (e) Designs geometry and all unrelated fixture groups are unchanged. This mirrors the same "focused post-implementation verification" pattern already used for the Tabletop Accessories and Security phases in this repository's `docs/` history.

---

## 15. Concise implementation handoff (for a later Codex prompt)

> Implement Inventory Organization Phase 1 in `C:\Genmitsu L8 Tracker\index.html`, per `docs/INVENTORY_ORGANIZATION_MATERIAL_CLEANUP_ARCHITECTURE_REVIEW_2026-07-23.md` §12. Add exactly four additive, derived-only behaviors to Manage mode: (1) a new `'thickness-asc'` option in `inventoryRawSort`/`filteredRawMaterials`, sorting by `parseInventorySizeMetadata(item).thicknessValue` with unparseable items grouped last; (2) an extended search haystack in `filteredRawMaterials` including `parseInventorySizeMetadata(item).displayLabel`; (3) a reference-count lookup added to `delInventory('rawMaterials', ...)`'s confirmation message, reusing `libraryProfilesForInventory`, `projectsForInventory`, and a `materialCondition.inventoryItemId`-based count, worded to make clear deletion does not alter those other records; (4) a new read-only, dismissible "possible duplicates" panel in Manage mode, clustering on `browserMaterialBaseName`/`materialMatchKey` with the same exact-then-loose, minimum-key-length discipline as `materialInventoryMatch`, linking each flagged item to its existing edit form and offering no automatic merge action. Do not add a thickness schema field, do not migrate any data, do not change `SCHEMA_VERSION`, `STORAGE_KEY`, or `BACKUP_FORMAT`, do not touch Browse mode's existing tree/grouping code, and do not implement CSV import, bulk category reassignment, or an archive status for raw materials — all explicitly deferred. Add fixtures as one more layer in the existing `runMaterialBrowserFixtures` chain (same dispatcher key `'materials'`), covering the cases enumerated in §14 of the architecture review, and re-run the full `?selftest=all` suite plus a manual direct-`file://` browser check with seeded legacy/duplicate/referenced synthetic data before calling the phase complete.
