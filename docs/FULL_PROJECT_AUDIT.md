# Genmitsu L8 Tracker — Full Project Audit & Roadmap

**Date:** 2026-07-08
**Commit audited:** `a6248df` — *Address audit QOL fixes*
**Supersedes:** the version of this file audited at `94f59c4` (predates the audit QOL pass that addressed 40W focus-helper wording, Inventory sort controls, low-stock tab count, Inventory Ctrl+N behavior, and merge-import summary counts)
**Scope:** Focused verification of `index.html` standalone app plus documentation refresh
**Auditor constraints:** Offline single-file design must remain intact

---

## Executive summary

At `a6248df`, the Genmitsu L8 Tracker is a mature **seven-tab offline workshop notebook**: Log, Library, Test Grids, Reference, Projects, Inventory, and Pricing. What started as a settings logger now covers the full loop from "what should I try" through "what did I make, what did it cost, and what do I have on the shelf."

**Architecture is intact:** one self-contained `index.html` (~3,100+ lines, up from ~2,242 at the last audit), no CDN/scripts/modules, no `fetch`/XHR, system fonts only. `Genmitsu L8 Tracker.dc.html` remains untouched (legacy x-dc artifact).

**Strengths today:** every new top-level data structure (Inventory included) was wired correctly into `persist()`, `backupObject()`, `loadState()`, `mergeData()`/`replaceData()`, and undo-delete — the exact class of bug that had to be fixed for Pricing/Projects earlier did **not** recur for Inventory. CSV exports for Inventory correctly respect active filters from the start, learning from the earlier Projects CSV fix.

**Main risks:** the single-IIFE file is now large enough that per-tab boilerplate (search/tag/sort/undo/CSV wiring) is duplicated five-plus times with only minor variation — a maintainability watch-item that's more pressing now than at the last audit, not less. Storage/photo volume risk is mitigated (try/catch + warning) but not solved. The remaining practical gaps are conservative follow-ups: no auto-deduction from Projects, silent corrupt-storage failure, merge conflict detail beyond the new summary counts, and the broader maintainability cost of continued feature growth in one file.

**Verdict: Safe to keep building.** No critical blockers. The single largest non-feature risk is `index.html`'s growing size and repeated wiring patterns — worth a light internal refactor before a possible 8th tab, not before this one.

---

## What changed since the last audit (`2f8d0f7` → `a6248df`)

| Commit | Change |
|--------|--------|
| `628b1bf` | Project modal now uses collapsible `<details>` sections (Details / Photos / Settings snapshot / Accounting / Notes) — fixes the old "one long form" finding (I6) |
| `1ee7b45` | `.gitignore` addition for a local image scratch folder |
| `0b5cb83` | **Inventory tracking MVP** — raw materials + finished batches, quantities, costs, low-stock threshold |
| `088286b` | Hardened inventory starter/seed data |
| `022727d` | Linked material name fields to Inventory (autocomplete awareness) |
| `c23cc34` | Material suggestions now show Inventory stock status |
| `06e42cb` | Fixed a duplicate-dropdown bug in material autocomplete |
| `94f59c4` | **Project material line items** — itemized material costs on a Project, with an explicit one-time "copy inventory unit cost in" action, and a total that can be copied into the Project's material cost field |
| `a6248df` | **Audit QOL fixes** — hides/rewords the 20W focus helper for 40W, adds Inventory raw/batch sort controls, surfaces low-stock count on the Inventory tab, improves Inventory Ctrl+N behavior, and shows merge-import added/updated summary counts |

---

## Current feature inventory

### Views (7 tabs)

| Tab | Purpose | Key capabilities |
|-----|---------|------------------|
| **Log** | Chronological job history | Stats dashboard, search/filter/sort, 1–5 rating, duplicate, promote to Library, save as Project |
| **Library** | Best-known settings per material | Tags, search/filter/sort, thickness matcher (top 4), print sheet, duplicate, save as Project |
| **Test Grids** | Power × speed matrix | Configurable ranges (capped at 60 steps, now flagged when truncated), per-cell rating/notes/best flag, promote to Library, CSV export |
| **Reference** | Official starting points | 20W/40W profile selector, embedded tables, local PDF links, 20W-only focus helper hidden/reworded for 40W, safety/maintenance notes |
| **Projects** | Finished-piece portfolio | Settings snapshot, up to 3 compressed photos, tags, status (kept/gifted/for-sale/sold), optional accounting incl. material line items, filter-aware report + copy, accounting CSV export (filtered), copy to Log, collapsible modal sections |
| **Inventory** *(new)* | Raw materials + finished batches | Quantity on hand, unit cost, estimated value, low-stock badges and tab count, canonical name + aliases, category/status filters, persisted raw/batch sort controls, search, CSV exports (filtered), starter-material seed helper |
| **Pricing** | Scratch estimator | Live totals, target margin/profit prices, saved rate defaults, print/CSV/copy, create Project draft (no auto-save) |

### Cross-cutting features

- **Units:** Global in/mm toggle with conversion of stored thickness, focus, and Z-offset fields
- **Tags:** Comma-separated on Library, Projects, and Inventory items; pill display; tag/category filters
- **Safety:** `materialSafetyWarning()` on material fields/cards (PVC, vinyl, chlorinated, unknown plastics, fiberglass, carbon fiber, coated unknown metal)
- **Help:** `?` badges (native `title`/`aria-label`) on laser, grid, pricing, and accounting fields
- **Keyboard:** `Ctrl+N` new item, `Ctrl+S` save modal, `Esc` close/back, `/` focus search (Inventory `Ctrl+N` starts raw materials by default, or finished batches when a batch status filter is active)
- **Undo:** 8-second restore toast for Log, Library, Projects, Test Grid, **and Inventory** deletes
- **Backup:** Export/import JSON with `schemaVersion: 2`; merge-by-`id` or replace; includes pricing, prefs, sorts, machine profile, library match inputs, **and Inventory (raw materials + finished batches, with aliases)**; merge imports now summarize added/updated counts
- **Photos:** Client-side resize (~1200px max, JPEG ~0.72 quality), stored in `project.photos[].dataUrl`
- **Material line items** *(new)*: itemized costs per Project, with a one-time "pull matched Inventory unit cost in" action — does not create a live link back to Inventory

### Beginner workflow supported today

1. **First material test** — Reference → pick 20W/40W → save to Library or run a Test Grid
2. **Dial in settings** — Grid cells → rate → promote best cell to Library
3. **Log real jobs** — Log entry with rating/notes → promote winners to Library
4. **Finish a piece** — Save as Project → add photos → optional accounting (with material line items)
5. **Track stock** — Inventory tab: log raw material purchases and finished batches, watch low-stock badges
6. **Price before selling** — Pricing tab estimate → optional Project draft → status `for-sale` / `sold`
7. **Repeat later** — Library matcher or Project gallery → copy to Log

---

## Findings by severity

### Critical

*None identified.* Math, persistence, and offline constraints remain sound. Inventory's addition did not reintroduce any of the storage/backup wiring gaps seen earlier in the project's history.

### Important — carried over from the last audit, still open

| # | Area | Finding |
|---|------|---------|
| I1 | **Storage / photos (partially mitigated)** | `persist()` now has try/catch with a user-facing warning (fixed in `30e2adc`), so the *crash* risk is gone. The underlying *capacity* risk is not: no proactive backup-size estimate, and Inventory adds another growing data source on top of photo-heavy Projects. |

### Important — new, from this audit

| # | Area | Finding |
|---|------|---------|
| I9 | **No Inventory ↔ Project auto-deduction** | Material line items can pull a one-time cost snapshot from a matched Inventory row, but using a material on a Project never decrements `quantityOnHand`. This is explicit, documented behavior (README says so), not a bug — but it means stock accuracy is entirely manual going forward, which is easy to let drift. |
| I11 | **Maintainability debt is compounding** | The file is now ~3,100+ lines in one IIFE (was ~2,242 at the last audit). Inventory alone added ~890 lines, and a large share of it re-implements the same search/tag/sort/undo/CSV-export wiring already present four other times for Log/Library/Projects/Grids. Not a bug, but the cost of the next new tab is higher than the cost of this one was. |

### Minor — carried over, still open

| # | Area | Finding |
|---|------|---------|
| M1 | **Ctrl+N gap** | `openActiveNewItem()` still skips Pricing and Reference. Inventory now has an explicit mapping: raw material by default, finished batch when a batch status filter is active. |
| M2 | **Help tooltips on touch** | Still `title`-only; weak on mobile (no tap popover). |
| M5 | **Reference PVC rows** | Safety helper still fires on material **name** pattern match, not automatically on promoted/renamed items. |
| M6 | **Merge import conflicts** | Merge import now shows added/updated counts, but still has no pre-import conflict preview or per-record list of overwritten items. |
| M7 | **`loadState` failure is silent** | Corrupt `localStorage` JSON still fails silently to empty defaults; no user-facing message that data may have been lost. |
| M9 | **Quantity zero** | Still clamps to ≥0 with a "Not available" break-even display rather than an explanatory message. |

### Minor — resolved since last audit

M3 (info toast swallowed by undo), M4 (grid range truncation — now shows a "range capped" pill), M10 (blank project speed showed `0 mm/min` — now shows `-`), I2 (import replace left stale pricing — now resets), I3 (Library/Projects search was `onchange` — now live `oninput`), I4 (accounting CSV ignored filters — now filtered, and the new Inventory CSVs got this right from day one), I6 (dense Project modal — now collapsible sections), I7 (20W focus helper hidden/reworded while 40W is selected), I10 (Inventory sort controls added), M6 partial (merge import summary counts added), and the low-stock count is now visible on the Inventory tab label.

### Nice-to-have (mostly unchanged)

Dark mode, stats dashboard, cut/engrave color coding, Tauri desktop packaging, separate/optional photo-free export, G-code/job filename field, lens-cleaning reminder, richer glossary panel, print stylesheet for Projects/accounting report.

---

## Data / storage risks

### localStorage

- **Key:** `genmitsu-l8-tracker-v1`
- **Schema:** `schemaVersion: 2`
- **Persisted:** `entries`, `profiles`, `grids`, `projects` (with embedded photos), **`inventory`** (raw materials + finished batches, with aliases), `unit`, sort prefs, library match fields, `machineProfile`, `pricing`, `pricingPrefs`
- **Verified:** `inventory` is correctly present in `persist()`, `backupObject()`, `loadState()`, `mergeData()`, and `replaceData()` — no gap here despite being the newest top-level array

### JSON backup / import

| Behavior | Status |
|----------|--------|
| Export includes portable preferences | Yes |
| Photos in projects export/import | Yes — `normalizeProjectPhotos()` accepts legacy string URLs or objects |
| Inventory export/import (raw materials + finished batches + aliases) | Yes — merges by `id` like every other list |
| Old backups without `inventory` | Defaults to empty via `normalizeInventory()` |
| `schemaVersion` migration | Still minimal — `data.schemaVersion || 1`, no explicit upgrade path beyond defaulting |
| Import validation | Still JSON shape only; no size/photo check, no merge-conflict preview; merge imports do summarize added/updated counts |

### Photo storage model (unchanged since last audit)

Max 3 photos/project, ~150–400 KB each after compression. 10 projects × 3 photos can approach several MB — still worth watching if you lean heavily on photos, now with Inventory data added to the same backup file.

---

## UX / beginner workflow notes

### What's improved since the last audit

- Project modal density (I6) is resolved — collapsible sections instead of one long scroll.
- Inventory's search is live (`oninput`) from the start — didn't repeat the Library/Projects mistake.
- Inventory now has persisted raw/batch sort controls and a low-stock count on the tab label.
- Material autocomplete now cross-references Inventory stock, so "do I have this on hand" is answered while typing a material name anywhere in the app.

### Where beginners may still stumble

| Scenario | Risk | Gap |
|----------|------|-----|
| Assume using a material line item updates stock | Medium | It doesn't — one-time cost copy only, no live link (I9) |
| Import conflicts during merge | Low | Added/updated counts are shown after merge, but there is no preview of what's about to be overwritten (M6) |

---

## Recommended next 4 small QOL tasks

1. **Optional "deduct from Inventory" button per material line item** (I9) — explicit user-triggered action (matching the existing "copy unit cost in" pattern), not automatic/live-linked, so it stays consistent with the app's philosophy of deliberate one-time syncs.
2. **Corrupt localStorage warning** (M7) — surface a clear warning if saved browser data cannot be parsed instead of silently falling back to empty defaults.
3. **Merge-import conflict preview** (M6) — the added/updated summary helps after merge; a preview before merge would make overwritten records less surprising.
4. **Storage/photo capacity UX** (I1) — optional compact/photo-free export, clearer backup-size messaging, or stronger photo-volume warnings.

## Recommended next 2 bigger chunks

1. **Light internal refactor pass** (I11) — extract shared helpers for search-binding, tag-filter rendering, sort dropdowns, undo-delete, and filtered-CSV-export, since the same ~30-line pattern now exists five times across Log/Library/Projects/Grids/Inventory. No framework, no build step — just de-duplication within the existing IIFE. Do this before adding an 8th tab, not after.
2. **Desktop durability (Tauri)** — unchanged from the last audit: app-data JSON file, automatic backups, no browser quota. Explicitly gated on separate Tauri packaging work, not compatible with "open `index.html` only" distribution until wrapped.

---

## What should NOT be built yet

Unchanged from the last audit: full inventory *system* beyond a personal shop tracker (multi-location/SKU-level), cloud sync/accounts, npm/React/Vite split, unbounded photo galleries, Etsy/Shopify auto-listing integrations, merging Pricing and Project accounting into one view, broad rewrite into multi-file ES modules. **Also avoid:** auto-deducting Inventory on every Project save without an explicit trigger — silent stock mutation from a settings-snapshot save would be surprising and hard to undo cleanly.

---

## Architecture / safety checklist

| Check | Result |
|-------|--------|
| Standalone offline (`index.html` opens directly) | Yes |
| No external CDN/scripts/fonts | Yes |
| No network calls in app code | Yes |
| `Genmitsu L8 Tracker.dc.html` unchanged | Yes |
| Local PDF links (`docs/*.pdf`) | Yes — relative paths |
| `index.html` maintainability | Watch closely — see I11 |
| Single-file constraint preserved | Yes |
| New Inventory feature wired into storage/backup correctly | Yes — verified directly against `persist()`/`backupObject()`/`loadState()`/`mergeData()`/`replaceData()` |

---

## Final verdict

**Safe to keep building.** The Inventory and material-line-item additions integrate cleanly with the existing storage/backup/undo pipeline — the exact class of bug that had to be retrofitted for Pricing and Projects earlier did not recur here, which is a good sign of accumulated project discipline.

**Suggested next move:** keep the next feature pass conservative: either add an explicit user-triggered Inventory deduction button for material line items, or do the corrupt-storage/merge-preview polish. Then schedule the internal refactor (I11) before the app grows an 8th tab — the duplication cost is the main thing quietly getting more expensive with each new feature, not any single bug.

---

*Audit refreshed at `C:\Genmitsu L8 Tracker` commit `a6248df`. Application source was inspected; this pass updates the audit document to reflect QOL fixes already present in `index.html`.*
