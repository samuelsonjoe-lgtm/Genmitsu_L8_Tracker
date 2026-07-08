# Genmitsu L8 Tracker — QOL Status & Project Gallery Plan

**Date:** 2026-07-07
**Scope:** (1) Where the QOL backlog from the original Grok/Codex audit stands today, and (2) a design plan for a new "Project Gallery" feature — save a full settings snapshot plus a photo of the finished piece, so a past project can be found and replicated later.

---

## 1. QOL status

### 1.1 Done (shipped in `index.html`)

| Item | Notes |
|------|-------|
| Standalone offline baseline | `index.html` has no `support.js`/x-dc or Google Fonts dependency — opens straight from disk |
| Open Manual button | Reference tab links to the local PDF in `docs/` |
| Library search/filter | Library now has the same search/filter treatment as Log (was flagged as an inconsistency in the original audit) |
| 20W focus helper | `focusSuggestion()` — enter job type + thickness, get a fixed-focus recommendation (line ~270 in `index.html`) |
| Material autocomplete (partial) | `<datalist id="materials">` seeded from `materialNames()`, which pulls distinct values from existing Log entries + Library profiles |
| README/docs cleanup | Storage model, file roles, and data fields documented |
| Pushed to GitHub | Repo is live and clean |
| **Duplicate entry button** | Clone a Log entry or Library profile into a new draft with the same settings, ready for a quick tweak-and-resave (`e481745`) |
| **Print-friendly Library view** | Print Library button + dedicated print stylesheet renders a clean shop cheat sheet (`e481745`) |
| **Import merge mode** | Import now prompts: OK = merge into current data by `id`, Cancel = replace everything (with a second confirmation) (`e481745`) |
| **Sort controls** | Log, Library, and Projects now have sort dropdowns with locally remembered preferences |
| **Reference-seeded material autocomplete** | Material fields now suggest both saved user materials and official SainSmart reference materials |
| **Keyboard shortcuts** | `Ctrl+N`, `Ctrl+S`, `Esc`, and `/` cover new-item, save-form, close/back, and search-focus flows |
| **Full local 20W chart** | `docs/L8_Engraving_Speed_Power_Reference_Chart_20W.pdf` is linked from Reference and its rows are reflected in the 20W table |
| **Unit conversion toggle** | Switching between `in` and `mm` converts stored thickness, focus, and material-height/Z-offset values |
| **Thickness-aware Library matching** | Library can surface the closest saved profiles for a material and thickness query |
| **Undo delete** | Log, Library, Projects, and Test Grid deletes now show a short restore window |
| **Test grid CSV export** | Recorded Test Grid cells can be exported with grid metadata, power/speed, rating, best flag, and notes |
| **Machine profile selector** | Reference view defaults to L8 20W and can switch to L8 40W without showing both tables at once |
| **Pricing calculator MVP** | Dedicated Pricing view estimates total revenue, direct costs, machine/labor/fee costs, profit, margin, and break-even price without changing Project records |
| **Pricing polish pack 1** | Pricing now includes target margin/profit sale-price helpers, a subtle profitability badge, a short assumptions note, and copyable plain-text summaries |
| **Pricing polish pack 2** | Pricing now has saved rate/fee defaults, compact print summaries, and one-row CSV export for the current estimate |
| **Tag system polish** | Projects and Library now have visible tag pills, tag search/filtering, clear-tag actions, and comma-separated tag entry guidance |
| **Phase 6 Project accounting MVP** | Projects now support optional saved sale/cost/profit fields and computed accounting summaries, while Pricing remains a separate scratch estimator |

### 1.2 Next up (agreed priority — biggest workshop value before Tauri)

**Next candidate:** Project photo capture or another narrow Project Gallery polish pass. The first tag-system pass is now complete without expanding into inventory management.

### 1.3 Still open from the original backlog

| Item | Priority (orig. audit) | Status |
|------|------------------------|--------|
| Sort options (Log by date, Library by material/rating) | High | Done — Log, Library, and Projects have dropdown controls |
| Material presets/autocomplete seeded from the **Reference** table specifically | High | Done — autocomplete draws from saved records plus official 20W/40W reference rows |
| Keyboard shortcuts (`Ctrl+N`, `Ctrl+S`, `Esc`, `/`) | High | Done |
| Version/schema stamp in exported JSON (`schemaVersion`) | Medium | Done — `schemaVersion: 2` is now included in storage/export with `projects: []` |
| True desktop packaging (Tauri) — app-data file persistence, auto-backups | High (blocks real durability) | Not started; still on `localStorage` |
| Thickness-aware Library matching ("what do I use for 3mm birch?") | Medium | Done — Library has a closest saved setting helper |
| Tag system (`gift`, `commission`, `failed`, etc.) | Medium | Done — Projects and Library support comma-separated tags, visible pills, and tag filtering |
| Test grid export (CSV / printable HTML) | Medium | Done — CSV export is available from grid detail |
| Unit conversion on in↔mm toggle | Medium | Done — converts stored unit-bearing fields when toggled |
| Undo delete (toast) | Medium | Done |
| Machine profile selector (20W default, hide 40W table) | High | Done |

### 1.4 Lower priority / later

Dark mode, statistics dashboard, cut/engrave color coding in Log cards, multi-machine support, G-code/job filename field, lens-cleaning reminder counter, material safety warnings surfaced from Reference notes, SD-card material test cross-link, tray icon/minimize, portable `data.json`-next-to-exe mode.

None of these block anything above — pull from this list opportunistically.

---

## 2. New feature: Project Gallery (save + replicate finished projects)

### 2.1 The problem

Library already answers *"what's the best known setting for this material?"* It does not answer *"what did I actually make, what did I run to make it, and what did it look like when it was done?"* Right now that lives in memory or scattered folders. The ask: pick a past project by name/material/tag, see the exact settings used **and** a photo of the finished result, so it can be rebuilt without re-deriving anything.

This is distinct from Library (which is normalized "best known settings per material/thickness") and from Log (which is a chronological run record). A Project is a **portfolio entry**: one finished thing, its recipe, and what it looked like.

### 2.2 Relationship to existing data

- A Project can be created from scratch, or **"Save as Project"** from an existing Log entry or Library profile (pre-fills the settings block, same pattern as the existing Library-promote flow).
- A Project optionally keeps a `sourceEntryId` / `sourceProfileId` back-reference so you can jump to "the log run this came from," but it does **not** require one — some projects start life directly as a gallery save.
- Projects get their own tab (5th tab: Log / Library / Test Grids / Reference / **Projects**) rather than being bolted onto Library, so the mental model stays clean: Library = recipe book, Projects = portfolio.

### 2.3 Data model

Adds a new `projects` array to the existing schema (`entries`, `profiles`, `grids`, `unit`), plus a `schemaVersion` bump (this is also the natural moment to add the version stamp from item 1.3):

```json
{
  "schemaVersion": 2,
  "unit": "in",
  "entries": [],
  "profiles": [],
  "grids": [],
  "projects": [
    {
      "id": "uid-string",
      "name": "Walnut coaster set",
      "material": "Walnut",
      "thicknessValue": 4, "thicknessUnit": "mm",
      "jobType": "engrave",
      "dateCreated": "2026-07-07",
      "tags": ["gift", "coasters"],
      "rating": 5,
      "notes": "Ran 2x to darken the grain lines without scorching edges.",

      "settings": {
        "minPower": 20, "maxPower": 45, "speed": 3000, "passes": 2,
        "lineInterval": 0.1, "airAssist": true, "airPressure": 20,
        "overscanPercent": 5, "kerfOffset": 0, "ditherMode": "jarvis",
        "focusValue": 7, "focusUnit": "mm",
        "software": "LightBurn", "settingsFile": "walnut-coaster.lbrn2"
      },

      "sourceEntryId": null,
      "sourceProfileId": "profile-uid-123",

      "photos": [
        { "id": "photo-uid-1", "isPrimary": true, "caption": "Finished set, natural light", "dataUrl": "data:image/jpeg;base64,..." }
      ]
    }
  ]
}
```

The `settings` sub-object intentionally mirrors the field names already used on Log/Library (`minPower`, `maxPower`, `lineInterval`, `overscanPercent`, `kerfOffset`, `ditherMode`, `focusValue`/`focusUnit`, `airAssist`, `airPressure`, `software`, `settingsFile`) so the "save as project" copy step is a straight field mapping, not a translation layer.

**Kerf note convention:** for inlay/interlocking projects, `kerfOffset` alone isn't self-explanatory — a kerf value tuned for a tight inlay fit and one tuned for general single-part dimensional accuracy can legitimately differ. When saving a project of that kind, the `notes` field should say the kerf was *inlay-calibrated* (and ideally which LightBurn kerf-test result it came from), so a future lookup doesn't assume it's a general-purpose value.

### 2.4 Photo storage — phased, because this app has no desktop backend yet

This is the one part of the feature that's constrained by where the app is architecturally right now (static HTML/JS + `localStorage`, no file system access). Two phases:

**Phase A — now, browser/`localStorage` era:**
- Store photos as compressed base64 `dataUrl` strings, generated client-side:
  - Read the file via `<input type="file" accept="image/*">`
  - Draw to an off-screen `<canvas>`, downscale to a max dimension (e.g. 1200px)
  - Export as JPEG at ~0.7 quality before storing
  - This keeps a typical photo to well under 200KB instead of multi-MB, so `localStorage`'s ~5-10MB ceiling doesn't get eaten by a handful of projects
- One primary photo per project for MVP; multiple photos can follow once the pattern is proven
- Export/import already round-trips arbitrary JSON, so this works with the existing backup flow immediately — no new plumbing needed there

**Phase A guardrails before implementation:**
- Treat one primary photo per project as a hard MVP limit.
- Add a client-side file size/downscale check and catch `localStorage` quota errors during save.
- Show a clear warning if the browser cannot store the compressed photo; keep the project saveable without the image.
- Keep photo capture behind the completed `schemaVersion: 2` / `projects: []` migration so image work does not destabilize existing Log/Library/Grid backups.

**Phase B — once the Tauri desktop wrapper lands (per the standalone-offline audit):**
- Switch photo storage to real files on disk: `%APPDATA%\GenmitsuL8Tracker\photos\<project-id>-<photo-id>.jpg`, with the JSON storing a relative filename instead of a `dataUrl`
- Migration step on first desktop launch: any project with a `dataUrl` gets it decoded and written out to the photos folder, then the field is swapped to a filename reference — old exported JSON backups keep working
- This is the same "paths not base64" guidance the original audit gave for photo attachments generally (§5.2 item 14) — Phase A is a deliberate, temporary exception because there's no file system to point paths at yet

### 2.5 UI plan

- **Projects tab**: gallery/card grid, each card = primary photo thumbnail + name + material/thickness + rating stars + tags
- **Search/filter bar**: text search across name/material/notes/tags (same pattern as Log/Library search), plus a tag filter dropdown
- **Sort**: newest first (default), by material, by rating — ties into the "sort options" QOL item in §1.3, so build that shared sort control once and reuse it across Log/Library/Projects
- **New Project** button: blank form, or launched from Log entry / Library profile via a "Save as Project" action (mirrors the existing "promote to Library" button pattern)
- **Detail/edit view**: full settings block (read-only metric tiles like Log/Library cards already use) + photo(s) + notes + tag editor + "Copy settings to new Log entry" button (the reverse direction — replicate it)

### 2.6 Implementation phases

| Phase | Work | Depends on |
|-------|------|------------|
| 1 | Data model + `schemaVersion: 2`, `projects: []` added to state/load/save, migration for old exports (missing `projects`/`schemaVersion` defaults to `[]`/`1`) | Done in `d73220f` |
| 2 | Projects tab: card grid, search/filter/sort, create/edit/delete, "Save as Project" from Log & Library, "Copy to Log" from Projects | Done across `d73220f` and the sort-control follow-up |
| 3 | Photo capture: file input → canvas downscale → base64 `dataUrl`, primary photo display on cards | Phase 2 |
| 4 | Print/export niceties: include Projects in JSON export (already automatic once in `state`), optional single-project print view | Phase 2 |
| 5 | (Later, post-Tauri) Swap `dataUrl` storage for on-disk file storage + one-time migration | Tauri desktop packaging from the standalone-offline audit |
| 6 | Expense/sale tracking on Projects — see §2.7 | MVP implemented: Project records can store optional sale/cost/profit accounting fields and compute summaries. Pricing-to-Project bridge remains a follow-up. |

Phases 1–2 are now the working Projects foundation. Phases 3–4 need nothing beyond what the app already has today (still just `index.html` + `localStorage`). Phase 5 is explicitly gated on the desktop packaging work already planned in `docs/STANDALONE_OFFLINE_AUDIT.md`. Phase 6 can slot in any time now that the Projects form/cards exist — it doesn't need photos or Tauri.

### 2.7 Phase 6 — expense/sale tracking (optional, per project)

**Why on Projects, not a new tab:** a Project already represents "one finished thing" — the natural place to ask "what did this cost and what did it sell for" is right next to the settings snapshot that made it, not a separate ledger disconnected from the piece.

**Goal:** answer "am I actually making money on this?" at a glance. Not a bookkeeping system — no invoices, no tax categories, no multi-currency, no customer CRM. Once sales are real and regular, a proper accounting tool (or even just the CSV export below fed into one) handles that better than a bolt-on tracker would.

**New optional fields on the Project record** (all default to empty/zero — invisible in the form and on cards until used):

```json
{
  "status": "kept",
  "qty": 1,
  "materialCost": 0,
  "otherCosts": 0,
  "timeHours": 0,
  "soldPrice": 0,
  "saleDate": "",
  "buyerNote": ""
}
```

- `status`: `kept` / `gifted` / `for-sale` / `sold` — lets Projects be filtered down to "actual inventory" without cluttering personal pieces that were never for sale
- `qty`: supports batches (e.g. one "Walnut coaster set" project = 10 units) instead of one record per identical item
- `materialCost`, `otherCosts` ($): raw material + consumables (glue, finish, packaging, shipping supplies)
- `timeHours`: optional — only meaningful if a labor rate is set (see below)
- `soldPrice`, `saleDate`, `buyerNote`: what it sold for, when, and where/to whom (e.g. "Etsy," "craft fair," a name)

**A single global setting**, not per-project: `hourlyLaborRate` ($/hr), stored alongside `unit` in top-level state. Lets time-cost be optional — leave it at 0 and `timeHours` just becomes a log, not a cost input. Because this is top-level state, it must be added to `loadState()`, `persist()`, `backupObject()`, and both import paths — not just `normalizeProject()`.

**Computed at render time, never stored** (so nothing can drift out of sync with the raw fields):
- `costPerUnit = (materialCost + otherCosts + timeHours * hourlyLaborRate) / max(qty, 1)`
- Treat `soldPrice` as the **total sale amount for the whole project/batch**, not per-unit.
- `profit = soldPrice - (costPerUnit * qty)`, only shown when `status === 'sold'`
- `marginPercent = profit / soldPrice`, only when `soldPrice > 0`

**UI:**
- A collapsible **"Sale & cost"** section at the bottom of the existing project form, collapsed by default — keeps the form uncluttered for anyone just using Projects as a settings/photo log
- A one-line **shop totals strip** in the Projects tab header, computed live from `state.projects`: e.g. *"Revenue $420 · Costs $85 · Profit $335 across 6 sold"* — no new nav, no new tab
- `status` badge on project cards (reuse the existing pill style used for "BEST KNOWN" on Library cards)

**Export:** project-level sale/cost fields ride along in the existing JSON export/import once added to `normalizeProject()`. The top-level `hourlyLaborRate` needs explicit export/import handling with the other top-level settings. A "Sold projects" CSV export (name, material, qty, cost, sold price, profit, sale date) is a nice-to-have for handing off to whoever does taxes at year-end, but isn't required for MVP.

### 2.8 Phase 6 vocabulary decision — align Project accounting with Pricing

This design note supersedes the rough Phase 6 field sketch above before any schema/code work happens. Pricing remains the scratch estimator for now; saved Project accounting should use compatible vocabulary so a future bridge does not require a rename pass.

| Question | Decision for future Phase 6 MVP |
|----------|----------------------------------|
| Quantity name | Use `quantity`, not `qty`, to match Pricing. |
| Material cost mode | Support `materialCostMode: "total" / "perItem"` plus `materialCost`, matching Pricing. |
| Hardware and packaging | Keep `hardwareCost` and `packagingCost` separate rather than merging into `otherCosts`. They are common shop costs and already separate in Pricing. |
| Machine vs labor time | Track `machineMinutes`, `laborMinutes`, `machineRate`, and `laborRate` separately. This is more useful than one generic `timeHours` field. |
| Marketplace/payment fees | Include `feePercent`, `fixedFee`, and `fixedFeeMode` so sold-project profit can reflect Etsy/PayPal/craft-market fees when desired. |
| Gross vs net sale price | Store `soldPrice` as gross total sale revenue for the whole Project/batch, then calculate fee cost from fee fields. If only net revenue is known, leave fee fields blank and enter the net amount by convention. |
| Pricing-to-Project bridge | Yes, eventually, but not in the first accounting MVP. Add a "Create/Update Project from Pricing" workflow only after these Project fields exist. |

**Recommended future MVP field set:**

```json
{
  "status": "kept",
  "quantity": 1,
  "soldPrice": 0,
  "saleDate": "",
  "materialCost": 0,
  "materialCostMode": "total",
  "hardwareCost": 0,
  "packagingCost": 0,
  "machineMinutes": 0,
  "laborMinutes": 0,
  "machineRate": 0,
  "laborRate": 0,
  "feePercent": 0,
  "fixedFee": 0,
  "fixedFeeMode": "order",
  "buyerNote": ""
}
```

**Rate defaults:** Project accounting can prefill from the same local Pricing defaults (`pricingPrefs`), but each Project should store the rate values actually used. That keeps historical sold-project profit stable if default rates change later.

**Computed at render time, never stored:**
- `directCost = materialCost + hardwareCost + packagingCost`
- `machineCost = machineMinutes / 60 * machineRate`
- `laborCost = laborMinutes / 60 * laborRate`
- `feeCost = soldPrice * feePercent / 100 + fixedFeeCost`
- `profit = soldPrice - totalCost`, shown when `status === "sold"`
- `marginPercent = profit / soldPrice`, only when `soldPrice > 0`

**Future Pricing-to-Project mapping:** `name` -> Project name, plus `quantity`, `materialCost`, `materialCostMode`, `hardwareCost`, `packagingCost`, `machineMinutes`, `laborMinutes`, `machineRate`, `laborRate`, `feePercent`, `fixedFee`, `fixedFeeMode`, and `notes` can copy directly. `salePrice` should map to `soldPrice = salePrice * quantity` only when the user confirms the Project is sold or wants a sale draft. Target-margin and target-profit suggestions stay in Pricing and should not be stored on Projects for MVP.

**Implementation status:** Phase 6 MVP accounting fields and summaries are implemented in `index.html`. A future Pricing-to-Project bridge remains intentionally out of scope.

**Privacy note:** `buyerNote` may contain customer names, marketplaces, or order details. Keep it optional, avoid storing sensitive information by default, and remember that JSON backups/exports will include it.
