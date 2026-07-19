# Designs-to-Projects Handoff — Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer model:** Claude Opus 4.8
**Review type:** read-only architecture review across Designs, Projects, storage, backup, import/export. No file was edited, staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## 1. Repository state

- `git status`: on `main`, "Your branch is up to date with 'origin/main'." Only untracked files present (the long-standing `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, and prior `docs/*.md` reports). **No tracked file is modified.**
- HEAD full hash: **`b35ed48356b3ba190951e2e58dde2647ae99f04e`** — "Organize Designs form controls" — matches the stated baseline `b35ed48`.
- Branch: `main`. Ahead/behind origin/main: `0 0` (**fully synchronized**).
- `git diff --stat` (unstaged): empty. `git diff --cached --name-only` (staging): empty. Tracked working tree is **byte-identical to `b35ed48`**.
- Untracked count: 40 files/dirs, all unrelated to this review and preserved untouched.
- Baseline fixture totals (264 / 1,093 / 1,885) taken as given; not re-executed (forward-looking architecture review, not a correction audit).

Note: the previously-recommended Designs form-organization phase has landed at this baseline — `designTemplateSelect()` (optgroups), `designFormSectionLabel()`, and `designAdvancedSection()` (native `<details class="project-section">`) now exist at `:1484-1494`. This is relevant because the same native-`<details>` and existing-prefill precedents make the present handoff low-risk.

## 2. Files and functions reviewed

State/storage/backup: `freshState` (`:433`), `loadState` (`:436`), `persist` (`:470`), `backupObject` (`:516`), `replaceData` (`:10865`), `mergeData` (`:10875`), `mergeList`/`mergeListStats` behavior. Project model: `normalizeProject` (`:9762`), `normalizeProjectAccounting` (`:9789`), `normalizeProjectMaterialLines` (`:5688`), `projectQuantity` (`:5682`), `formDefaults('project')` (`:948-956`), `settingsFromSetting` (`:923`). Project form/handoff: `openProject` (`:9645`), its `onsubmit`/`upsert` path (`:9742-9757`), `projectDraftFromSource` (`:9810`), `projectFromEntry`/`projectFromProfile` (`:9827-9834`), `projectToEntry` (`:9835`), `createProjectFromPricing` (`:5926`), `projectDraftFromPricing` (`:5898`), `projectAccountingFields` (`:5976`), `openModal`/`closeModal`/`bindModal` (`:9430-10511`). Inventory-guided wizard (to distinguish it): `projectWizard*` / `openProjectWizard` (`:7834-8186`), `runProjectWizardFixtures` (`:8201`). Designs: `renderDesigns` (`:1816`), `designTemplateSelect` (`:1484`), `buildDesignResult` (`:3376`) and every `build*DesignResult`/finished-view builder, `designResultsHtml` (`:3488`), `refreshDesignPreview` (`:3650`), `downloadCurrentDesignSvg` (`:3668`), `downloadDrawerCabinetProductionOutput` (`:3675`), `updateDesignDraft` (`:3682`), `normalizeDesignDraft` (`:2142`). Fixture runners inventory (`:494-10330`) to model the fixture plan. Docs: the recent Designs result-summary and form-organization reviews, and the New Project Wizard / Inventory-guided Project Wizard reports.

## 3. Project model inventory (current record shape)

Written by `normalizeProject(f, existingId)` + `normalizeProjectAccounting(f)`. Field-by-field:

| Key | Meaning | Type | Default | Req? | Normalization | Rendered in | Editable | Backup/import | Older records may omit | Fixture-protected |
|---|---|---|---|---|---|---|---|---|---|---|
| `id` | Stable identity | string | `uid()` | yes (auto) | generated or preserved | list/detail | no | merges by id | no | project browser fixtures |
| `name` | Piece/batch name | string | `''` | **yes** (submit blocks if empty) | `.trim()` | list/detail header | yes | plain | rare | project browser |
| `material` | Material species/name | string | `''` | **yes** (submit blocks if empty) | `.trim()` | list/detail, material match | yes | plain | rare | material match fixtures |
| `thicknessValue` | Material thickness | number\|`''` | `''` | no | `num()` or `''` | Details | yes | plain | yes | — |
| `thicknessUnit` | Unit for thickness | `'in'`/`'mm'` | `state.unit` | no | passthrough | Details | yes | plain | yes | — |
| `jobType` | engrave/cut | enum | `'engrave'` | no | passthrough | Details | yes | plain | yes | — |
| `dateCreated` | Creation date | date str | `today()` | no | `|| today()` | Details/CSV | yes | plain | yes | — |
| `tags` | Filter labels | string[] | `[]` | no | `tagArray()` | list/detail chips | yes | plain | yes | project browser |
| `rating` | 0–5 stars | number | `0` | no | `num(...,0)` | Details | yes | plain | yes | — |
| `notes` | Free notes | string | `''` | no | `.trim()` | Notes section | yes | plain | yes | — |
| `sourceEntryId` | Origin log entry | string\|null | `null` | no | `|| null` | hidden input | via source | plain | yes | — |
| `sourceProfileId` | Origin library profile | string\|null | `null` | no | `|| null` | hidden input | via source | plain | yes | — |
| `photos` | Compressed images | object[] | `[]` | no | `normalizeProjectPhotos` | Photos section | yes | plain (large) | yes | photo fixtures |
| `settings` | Laser settings snapshot | object | `settingsFromSetting({})` | no | nested normalize | Settings snapshot | yes | plain | yes | — |
| `status` | kept/gifted/for-sale/sold | enum | `'kept'` | no | `|| 'kept'` | list/detail | yes | plain | yes | accounting fixtures |
| `quantity` | Item count | number\|`''` | `'1'` | no | `projectQuantity(f)` or `''` | Accounting | yes | plain | yes | accounting |
| `soldPrice` | Gross revenue | money\|`''` | `''` | no | `optionalMoney` | Accounting | yes | plain | yes | accounting |
| `saleDate` | Sale date | date str | `''` | no | passthrough | Accounting | yes | plain | yes | — |
| `materialCost`, `materialCostMode`, `hardwareCost`, `packagingCost`, `machineMinutes`, `machineRate`, `laborMinutes`, `laborRate`, `feePercent`, `fixedFee`, `fixedFeeMode` | Cost/accounting inputs | money/enum | `''`/mode defaults | no | `optionalMoney`/passthrough | Accounting | yes | plain | yes | accounting |
| `materialLines` | Itemized material rows | object[] | `[]` | no | `normalizeProjectMaterialLines` | Accounting | yes | plain | yes | material-line fixtures |
| `accountingNotes` | Accounting notes | string | `''` | no | `.trim()` | Accounting | yes | plain | yes | — |

**Fields that could honestly accept Design-derived information:** `name` (from template display name + finished dimensions), `notes` (compact dimension/warning/prototype summary), `thicknessValue`/`thicknessUnit` (from the Design's measured `materialThickness`, which is always mm), `jobType` (Designs are cut → `'cut'`), `quantity` (a single finished object → `'1'`), `status` (`'kept'`). **No new field is needed.** There is no Project field that maps to Design piece counts, production-output counts, joint clearances, or SVG geometry, and none should be invented.

## 4. Project workflow inventory (the real "New Project" path)

There are two distinct things named "wizard" in the source. They must not be confused:

- **Inventory-guided Project Wizard** (`openProjectWizard`, `projectWizard*`, `:7834-8467`) — a multi-step *material-and-settings recommendation* flow that helps pick a laser recipe from Inventory/Library evidence. Its transient state lives in the module-level `projectWizard` object and is cleared by `closeModal('projectWizardModal')` (`:9438`). **This is not the Project-record creation form and is not the handoff target.**
- **The New/Edit Project form** (`openProject(project, options)`, `:9645`) — the actual Project-record editor. This *is* the "New Project workflow" for record creation.

Trace of `openProject`:
- **Opens** via `openModal('projectModal', …)` — a single modal form (`#projectForm`) with `<details>` sections (Details, Photos, Settings snapshot, Accounting, Notes). It is *not* a stepper; all fields exist at once.
- **Temporary state** lives entirely in the modal DOM plus two module-level drafts (`projectPhotosDraft`, `projectMaterialLinesDraft`) seeded from the passed object. **None of this is persisted** — no `localStorage`, no `backupObject` entry.
- **Defaults** come from `{ ...formDefaults('project'), ...(project || {}) }` — i.e. a caller may pass a partial prefill object and it overlays the defaults. This is the prefill seam.
- **`editingProjectId`** is set to `project.id` if present, else `null`. A `null` id ⇒ header reads "New project" and submit creates a brand-new record.
- **Validation** occurs only at submit: `if (!f.name.trim() || !f.material.trim()) return alert(...)`. Nothing is written before that.
- **A Project record is written** only in `onsubmit` → `normalizeProject(f, editingProjectId)` → `upsert('projects', projectObj)` → `closeModal`. Cancel (`data-close`) closes the modal and writes nothing.
- **Cancellation** creates no record; the modal DOM is discarded.
- **Editing an existing Project shares this same code** (`openProject(existingProject)` with an id).
- **`options.draftNote`** renders a highlighted banner at the top of the form (`:9651-9653`) — already used to say "review before saving." **`options.pricingDraft`** adds a second explanatory note.

**Existing prefill precedents (three of them) already use exactly this seam:**
- `projectFromEntry` / `projectFromProfile` → `openProject(projectDraftFromSource(source, type))` (`:9827-9834`).
- `createProjectFromPricing` → `openProject(projectDraftFromPricing(state.pricing), { draftNote, pricingDraft })` (`:5926-5934`), plus an info toast "Project draft opened from Pricing. Click Save project to keep it."

**Smallest supported entry point for a Design handoff:** build a partial project-draft object from the current valid Design result and call `openProject(draft, { draftNote })`. This reuses the exact, already-tested Pricing pattern and duplicates no Project-creation logic.

## 5. Design-side semantic data inventory

`buildDesignResult(draft)` (`:3376`) returns a `result` with, across templates: `result.valid`, `result.errors[]`, `result.warnings[]`, `result.svg` (production SVG string), `result.widthMm`/`result.heightMm` (production sheet size), `result.panels[]`, `result.metrics` (rich per-template semantic object), `result.finishedView` (trays/cleats), `result.productionOutputs[]` and `result.requiresMultipleProductionOutputs` (Drawer Cabinet separate-thickness only). `designDraft` holds the raw inputs; `normalizeDesignDraft` produces the validated values. **All Design linear values are in millimeters.** Display names come from `designTemplateSelect` groups (`:1486-1489`).

Reliable structured values available **before** serialization (per template):

| Template / mode | Reliable semantic values (source) |
|---|---|
| `qr-stand` | `metrics.template`, `metrics.finishedWidth`/`finishedHeight`, `metrics.materialThickness`, `metrics.fitClearance`, `panels.length` |
| `hanging-sign` | `metrics.template`, `metrics.finishedWidth`/`finishedHeight`, `metrics.materialThickness`, `panels.length` |
| `dice-tray` | `finishedView.insideWidthMm`/`insideDepthMm`/`wallHeightMm`/`baseOutsideWidthMm`/`baseOutsideDepthMm`/`materialThicknessMm`, `metrics.pieceCount`, bottom-cover flags |
| `divider-tray` | `finishedView.*` dims, `dividerCount`, `compartmentCount`, `metrics.pieceCount` |
| `finger-box` | `metrics.dimensions.outside/inside W/D/H`, `metrics.closedHeight`, `metrics.materialThickness`, `metrics.boxLid`, `metrics.cornerDecoration`, `panels.length` |
| `sliding-lid-box` | `metrics.dimensions.*` (outside/usable/inside, rails, lid), `metrics.materialThickness`, clearances, `metrics.fitCoupon`, `panels.length` |
| `drawer-cabinet` linked | `metrics.dimensions.cabinetOutside/inside/drawerInside/outside`, `metrics.rows`/`columns`/`drawerCount`, `metrics.shellThickness`, `metrics.pieceCount`, `metrics.productionFileCount` (1) |
| `drawer-cabinet` separate | as linked, plus `metrics.useSeparateInteriorThickness=true`, `metrics.interiorThickness`, `requiresMultipleProductionOutputs=true`, `productionOutputs[]` (2), `metrics.productionFileCount` (2) |
| `drawer-cabinet` custom-row | as linked, plus `metrics.layoutMode='custom-row'`, `metrics.rowDrawerCounts`, `metrics.rowLayouts`, `metrics.maximumColumns` |
| `joint-fit-coupon` finger-edge | `metrics.jointFitCoupon`, `metrics.clearances[]`, `metrics.pairCount`, `metrics.pieceCount`, `metrics.materialThickness`, `metrics.edgeLength`/`bodyDepth` |
| `joint-fit-coupon` wall-to-base | `metrics.couponType='wall-base-tab'`, `metrics.wallJointStyle`, `metrics.wallLengthMm`/`wallHeightMm`, `metrics.clearances[]`, `metrics.wallPieceCount`/`basePlateCount`/`pieceCount` |
| `concealed-cleat-corner-prototype` | `metrics.legLength`, `metrics.wallHeight`, `metrics.materialThickness`, `metrics.semantic.*`, `metrics.pieceCount` |
| `concealed-cleat-full-box-prototype` | `metrics.Ow`/`Od`/`H`, `metrics.internalWidth/Depth/Height`, `metrics.semantic.clearInterior.*`, `metrics.cleat*`, `metrics.pieceCount` |

Classification:
- **Reliable semantic data (safe to reference):** template identity/display name, finished/outside dimensions, measured material thickness, prototype/coupon status, validity, warnings.
- **Presentation-only text:** metric labels, assembly-instruction paragraphs, the Finished-View SVG markup and its captions.
- **Production-only data (never copied):** `result.svg`, `productionOutputs[].svg`, `widthMm`/`heightMm` sheet sizes, filenames, panel path geometry.
- **Values that must not be copied into Projects:** `panels.length`/`pieceCount`, drawer counts, coupon-row counts, `productionFileCount`, joint clearances, finger-pattern segment counts — none maps to a Project field, and several (piece/row/file counts) would be actively misleading as Project quantity.
- **Values that would require a new schema field:** any Design-source identity, template id, or clearance recipe. **None is worth a schema change in Phase 1** (see §11, §16).

**The handoff must not parse `result.svg` to construct anything** — every value it needs is already available as structured `result.metrics` / `result.finishedView` data.

## 6. Handoff boundary

The action should be offered/enabled only when **`result.valid === true`** (which guarantees a production SVG exists and all required inputs normalized). Warnings may still be present and do **not** block the handoff (they are advisory and are surfaced in the copied note). Concretely:

| Situation | Behavior |
|---|---|
| Design invalid (`!result.valid`) | Action **disabled** (mirrors the Download button's disabled state at `:1935`). |
| Design valid with warnings | Action **enabled**; a concise warning summary is included in the Project notes. |
| Coupon (finger-edge or wall-to-base) | Action **enabled**, but the draft name/notes label it a test coupon; quantity `'1'`; no coupon-row count copied. |
| Experimental prototype (either cleat) | Action **enabled**; name/notes label it an assembly/registration prototype, "not a strength test." |
| Drawer Cabinet 2 outputs (`requiresMultipleProductionOutputs`) | Action **enabled** (result is valid even though the single Download is disabled); notes record shell/interior thickness and "two production files required." |
| Shell ≠ interior thickness | Shell thickness → `thicknessValue`; interior thickness recorded in notes only. |
| User edits the Design after opening the Project modal | No effect — the draft is a **point-in-time snapshot** captured at click; the open modal is independent. |
| User cancels the Project modal | No record created; nothing persisted. |
| User already has unsaved Project-modal work | The action opens the Project modal; see §14 — because the button lives on the Designs tab and the modal opens in place, there is no *other* open Project form to overwrite. If one were somehow open, `openModal` replaces its content, so a confirmation guard is warranted only if a project modal is already open (rare; addressed in §14). |

**No Project may be created merely by rendering or downloading a Design.** The only write path remains the Project modal's explicit Save.

## 7. Candidate comparison

| Approach | Workshop value | Impl size | Storage/schema | Import/export | Back-compat | Cancel-safety | Stale risk | Dup-project risk | A11y | Fixture burden | Extensibility | Scope-creep |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **A. Copy summary to clipboard** | Low–med | Tiny | none | none | full | n/a | n/a | none | ok | small | dead-ends | low |
| **B. Prefill existing New Project form (session-only draft), Save is explicit** | **High** | **Small** | **none** | **none** | **full** | **inherent** | **none (snapshot)** | **none (Save-gated)** | **reuses modal** | **moderate** | **high** | **low** |
| C. Unsaved Project draft object → regular editor | High | Small | none | none | full | inherent | none | none | reuses modal | moderate | high | low |
| D. Immediately create record after confirm | High | Small–med | none | none | full | **weak** (record exists) | med | **higher** | ok | high | med | med |
| E. Permanent Design snapshot/source object in Project schema | Med | Large | **new field + migration** | **changes** | **risk** | inherent | high | none | ok | large | high | **high** |
| F. Saved Design presets + Project links | Med | Large | **new store + links** | **changes** | **risk** | inherent | high | none | ok | large | high | **high** |

**In this codebase, B and C are the same implementation:** the "regular Project editor" *is* `openProject(draft)`, and its prefill object is inherently a session-only, in-memory draft passed by value — there is no separate persistent draft store to build. So Candidate B is selected and is realized precisely as the existing `createProjectFromPricing` pattern. D is rejected (creates a record before the user reviews it, risking silent/duplicate projects). E and F are rejected (schema/migration/import-export changes that the preferred outcome explicitly wants to avoid, with no Phase-1 need).

## 8. Selected bounded phase

**Phase 1 — "Start Project from Design" prefill.** A button in the Designs actions row, enabled only for a valid Design, builds a point-in-time partial project draft from `result`/`designDraft` and calls `openProject(draft, { draftNote })`, exactly mirroring `createProjectFromPricing`. The user reviews the prefilled New Project modal and saves through the existing Save action. Nothing is persisted until Save; cancel writes nothing; Inventory, Pricing, storage schema, and production output are untouched.

## 9. Exact field mapping (`projectDraftFromDesign(result, draft)`)

| Source value | Destination field | Transformation | Fallback | User-editable before save | Honest? |
|---|---|---|---|---|---|
| Template display name (+ finished/outside dims when finite) | `name` | e.g. `"Finger-jointed box 180 × 120 × 80 mm"`; coupons/prototypes use their descriptive display name | display name alone | yes | Yes — names what was designed |
| — | `material` | **left blank** (`''`) | `''` | yes (**required at Save**) | Yes — Design knows thickness, never species; forcing the user to supply material is correct |
| `metrics.materialThickness` (mm; for cabinet use `metrics.shellThickness`) | `thicknessValue` + `thicknessUnit='mm'` | numeric copy; unit pinned to `mm` | `''`/`state.unit` | yes | Yes — it is a measured mm thickness |
| (constant) | `jobType` | `'cut'` | `'cut'` | yes | Yes — these templates are cut designs |
| (constant) | `quantity` | `'1'` | `'1'` | yes | Yes — one finished object |
| (constant) | `status` | `'kept'` | `'kept'` | yes | Yes — neutral default |
| Compact summary: template, outside/finished dims, prototype/coupon status, separate-output note, top 1–2 ordered warnings | `notes` | short human sentence(s), **not** a metric dump | template name only | yes | Yes — descriptive, non-authoritative |

**Explicitly avoided:** copying every metric into notes; claiming a Material or Inventory match from thickness; auto-selecting Inventory by name; recording warnings as completed production steps; using piece/panel/drawer/coupon/file counts as `quantity`; treating prototypes/coupons as production-ready or saleable. `sourceEntryId`/`sourceProfileId` remain `null` (Designs are neither); no material lines, accounting, or settings snapshot are prefilled.

## 10. Material and Inventory findings

Design material thickness may safely prefill only the **plain Project `thicknessValue`** (as mm). It must **not** prefill a selected Material record or Inventory item, and Inventory must **not** be matched on thickness alone (`materialInventoryMatchHtml` already keys off the material *name* string the user types — leaving `material` blank means no false match is asserted). The existing Project form's material field and its inventory-match helper remain the manual selection point and are preserved unchanged. For Drawer Cabinet **separate-thickness** mode, represent shell thickness in `thicknessValue` and describe the interior thickness in `notes` ("Interior/drawer stock N mm; two production files required"); do not attempt to encode two thicknesses in one field or add a schema field. The handoff must not reserve, deduct, create, or alter Inventory, infer sheet count/cost/usable stock, or claim a matching material exists — none of those paths is touched.

## 11. Quantity findings

Project `quantity` today means **items this Project represents** (help text: "How many items this Project represents," `:5978`; default `'1'`). Do not conflate it with: one generated object, physical cut-piece count (`panels.length`/`pieceCount`), number of SVG files (`productionFileCount`), number of drawers (`drawerCount`), coupon sample count, or Inventory sheet count. **Recommended default: `'1'`** (one finished design), which matches current semantics. Coupons and prototypes also default to `'1'` with a descriptive note rather than a sample-count-derived quantity.

## 12. Naming findings

Default names use the template display name, optionally suffixed with compact finished/outside dimensions when finite, e.g.:
- `Finger-jointed box 180 × 120 × 80 mm`
- `Sliding-lid finger box 150 × 90 × 60 mm`
- `Drawer Cabinet 2 × 3` (or `… — custom rows`)
- `Joint Fit Coupon — Finger Edge` / `Joint Fit Coupon — Wall-to-Base`
- `Concealed Cleat Full-Box Prototype 180 × 150 × 90 mm`

No timestamps or customer names (none are available; `dateCreated` already records the date). Including compact outside dimensions keeps repeated projects distinguishable without unwieldy names; the field is fully editable before Save.

## 13. Snapshot vs. live link

**Phase 1 copies a point-in-time prefill only.** The current Project schema has no field for a stable Design link, and adding one raises questions about later Design edits, template removal, schema evolution, import/export of a dangling reference, duplicated drafts, and reverse navigation. A live link is explicitly deferred. `designDraft` is itself session-only (absent from `persist`/`backupObject`), so there is nothing durable to link to anyway.

## 14. Navigation and cancellation

The button lives in the Designs actions row (`:1935`), so — exactly like `createProjectFromPricing` from the Pricing tab — it opens the Project modal **in place** via `openModal`; no cross-tab route is required. Flow: valid Design → click → `openProject(projectDraftFromDesign(...), { draftNote })` → modal opens prefilled with a "review before saving; nothing is saved until you click Save project" banner → user reviews/edits → **Save** writes exactly one record via `upsert` → **Cancel** writes nothing. `openModal` moves the form into the DOM and `bindModal` wires the close controls; focus handling matches every other modal opened from a tab action. Because the Project modal is opened fresh from Design's own DOM, there is no *separate* unsaved Project form to clobber; a lightweight guard ("A project form is already open") is only warranted if `#projectModal` is already `.open`, and even then the safe choice is to no-op or confirm rather than silently replace. Unsaved Design inputs are untouched (the Designs tab remains behind the modal).

## 15. Accessibility

Use an ordinary `<button type="button">` in the existing `.actions` row with existing button styling (secondary, next to the primary Download). Disabled state via the same `${result.valid ? '' : 'disabled'}` idiom already used for Download — so validity is conveyed non-color-only through the native disabled semantics and label. Keyboard activation is native. Focus after activation is handled by the shared modal path. A status toast ("Project draft opened from Design. Click Save project to keep it.") mirrors the Pricing precedent and is announced like the existing info toast. No modal-inside-modal is introduced (the Project modal *is* the destination). Duplicate activation is harmless: each click builds a fresh snapshot; no record exists until Save.

## 16. Storage and schema contract

Phase 1 requires **no schema change, no new Project field, no new storage key, no migration, no backup change, no import/export change.** Justification: the draft is an in-memory function argument consumed by `openProject`; it never enters `state`, `persist()`, or `backupObject()`. A saved Project is a normal id'd record, so `mergeData`/`replaceData` (both merge/replace `projects` by id, `:10869`/`:10888`) remain byte-compatible and older records still load. If a future phase wants a durable Design link, that is the point at which old-record compatibility, missing-field defaults, import/export/merge behavior, fixture updates, and rollback must be specified — **explicitly out of scope here.**

## 17. Production-output contract

The handoff must not change Design normalization, geometry builders, production SVG bytes, `productionOutputs`, filenames, Finished Views, download behavior, warning behavior, or `designDraft` persistence. It only **reads** `result` (already built for the preview) and `designDraft`. Fixtures: for representative drafts per template, assert `buildDesignResult(draft).svg` and each `productionOutputs[].svg` (and `widthMm`/`heightMm`/filenames) are **byte-identical before and after** invoking the handoff builder, and that calling `projectDraftFromDesign` does not mutate `designDraft` or `result`.

## 18. Project-data safety contract

Fixtures must prove:
- Building the draft / opening the modal creates **no** Project record (`state.projects` length unchanged).
- Cancel creates **no** record.
- Save creates **exactly one** record via the existing `upsert` path.
- Repeated activation before Save cannot create duplicates (no record until Save; each click rebuilds the snapshot).
- Existing Projects are unchanged (deep-equal before/after for all non-target records).
- Existing unsaved Project-form input is not silently overwritten (guard when `#projectModal` already open).
- Imported older Projects still render (no new required field).
- `backupObject()` output is byte-identical for a schema-neutral change with no new projects.
- `mergeData`/`replaceData` behavior is unchanged.
- Inventory and Pricing state are unchanged by opening/cancelling/saving the handoff draft.

## 19. Fixture plan

Add `runDesignToProjectHandoffFixtures()` (registered on `window`, matching the existing runner convention at `:494-10330`), behavioral and table-driven (build real `result` objects and real draft objects; interact with the real modal DOM for the open/cancel/save lifecycle rather than string-searching source):
1. Action **visible/enabled** exactly when `result.valid`.
2. Action **disabled/absent** for an invalid Design (missing thickness, etc.).
3. Warning-bearing valid Design: action enabled; note contains a concise warning summary.
4. One case per template producing the expected `name`, `thicknessValue`, `thicknessUnit='mm'`, `jobType='cut'`, `quantity='1'`, blank `material`.
5. Both coupon modes: descriptive name, no coupon-row count used as quantity.
6. Drawer Cabinet linked / separate / custom-row: separate → shell thickness in `thicknessValue`, interior thickness in notes, "two files" note.
7. Field-mapping exactness (destination values equal the mapping table).
8. Project-name defaults (with/without finite dimensions).
9. Quantity always `'1'`.
10. Material always blank; Save blocked until user supplies material (existing validation).
11. Temporary-state lifecycle: draft not in `state`, `persist()`/`backupObject()` unaffected.
12. Cancellation: no record, nothing persisted.
13. Duplicate activation: no duplicate records.
14. Unsaved-wizard/modal conflict guard.
15. Navigation/focus: modal opens, close controls bound, focus reachable.
16. Storage isolation and backup isolation (byte-identical `backupObject()`).
17. Import/export and existing-Project compatibility (older record without new fields still renders/normalizes).
18. Inventory/Pricing neutrality.
19. Production-byte and Finished-View neutrality (§17).
20. Direct `file://` behavior (feature is pure DOM/JS; no network).
21. 320 px responsive: button and opened modal usable (reuses existing `.actions`/modal responsive rules).
22. Machine-profile independence: handoff identical regardless of `state.machineProfile`.

## 20. Candidate bounded implementation (exact scope)

- **Files changed:** `index.html` only.
- **Functions added:** `projectDraftFromDesign(result, draft)` (pure; builds the partial draft per §9) and a thin `startProjectFromDesign()` handler that guards `result.valid`, calls `openProject(projectDraftFromDesign(...), { draftNote: 'Draft from a generated Design. Add the material you will actually use, review the fields, and click Save project. Nothing is saved until then.' })`, and shows the info toast. Plus `runDesignToProjectHandoffFixtures()`.
- **Functions changed:** `renderDesigns()` — add one button to the actions row (`:1935`) with the same `${result.valid ? '' : 'disabled'}` idiom; `bindDesignPreviewActions()` **or** the Designs event binding — wire the button's `onclick`. (No change to `buildDesignResult`, builders, serializers, or `downloadCurrentDesignSvg`.)
- **Functions protected (must not change behavior):** `normalizeDesignDraft`, all `build*DesignResult`, all finished-view builders, `serializeDesignSvg` and the concealed-cleat local serializer, `productionOutputs`, filenames, `downloadCurrentDesignSvg`, `downloadDrawerCabinetProductionOutput`, `normalizeProject`, `openProject`'s save/validation, `persist`, `backupObject`, `replaceData`, `mergeData`.
- **Temporary-state location:** the draft object is a function argument passed to `openProject`; it lives only in the modal DOM until Save. **No module-level variable, no storage key.**
- **Action placement:** Designs `.actions` row beside Download.
- **Field mapping:** §9.
- **Navigation behavior:** modal opens in place (§14).
- **Conflict behavior:** guard if `#projectModal` already open (§14/§18).
- **Cancellation cleanup:** none required (nothing persisted); existing `closeModal('projectModal')` suffices.
- **README:** small addition documenting the button and that it prefills a reviewable, unsaved Project.
- **Help text:** the `draftNote` banner text above; no other help changes.
- **Exact non-goals:** §21.

## 21. Non-goals (deferred)

Saved Design presets; Project→Design reverse navigation; live Design↔Project synchronization; storing SVG bytes/filenames/Blob URLs in Projects; automatic downloads on handoff; automatic Inventory selection; Inventory reservation/deduction/creation; automatic Pricing records; cost/material-use/sheet-count/cut-time estimation; automatic nesting; multi-sheet export; Project attachments; cloud sync; schema migration; customer/order integration; production-completion-tracking changes; new templates or joints; copying the universal 100%-scale scale-safety notice into Projects.

## 22. Post-implementation audit recommendation

The selected phase is **fully schema-neutral** (no Project field, storage key, migration, or import/export change). Therefore: **Codex implements; Grok performs a focused adversarial audit.** A Claude Opus 4.8 implementation audit is **not** required (it would be required only if the implementation ends up touching the Project schema, import/export, merge, or persistent storage — none of which this phase does). **Fable 5 is unnecessary.**

## Final verdict

**READY FOR BOUNDED IMPLEMENTATION**

- **Recommended implementation report filename:** `docs/DESIGNS_TO_PROJECTS_HANDOFF_IMPLEMENTATION_2026-07-19.md`
- **Recommended implementation audit filename:** `docs/DESIGNS_TO_PROJECTS_HANDOFF_FOCUSED_AUDIT_2026-07-19.md`
- **Should Codex implement directly?** Yes — scope is small, fully enumerated (§20), reuses the existing `openProject(draft, options)` prefill seam (the same pattern as `createProjectFromPricing`), and touches no production/storage/schema path.
- **Is Grok sufficient afterward?** Yes — a focused adversarial audit is appropriate for this schema-neutral change.
- **Should Claude Opus 4.8 audit the implementation?** No — only if the implementation deviates into Project schema, import/export, merge, or persistent storage changes.
- **Is Fable 5 unnecessary?** Yes.
