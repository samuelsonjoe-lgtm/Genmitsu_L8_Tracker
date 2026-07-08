# Project Accounting Bridge + Export Pack — Implementation Report

**Date:** July 8, 2026  
**Base commit:** `f7aba47` — Add project accounting MVP  
**Status:** Implemented locally, not committed

---

## Pre-implementation audit (Project Accounting MVP)

### Current behavior found

| Area | Finding |
|------|---------|
| **Render/edit/save** | `openProject()` merges `formDefaults('project')` with existing record; accounting section renders via `projectAccountingFields()` with live preview on `input`/`change`. |
| **Old projects** | Records without accounting fields load safely — defaults applied on open; `projectQuantity()` falls back via `num(p.quantity, 1)`; `status` defaults to `kept`. |
| **Calculations** | `projectAccountingTotals()` mirrors Pricing math; `soldPrice` is gross batch revenue; fees on revenue; material per-item/total modes work. |
| **JSON export/import** | Projects (including accounting fields) ride in `backupObject()` / import paths unchanged. Pricing draft still localStorage-only (unchanged). |
| **Small fix applied** | `normalizeProjectAccounting()` now treats `null` quantity like empty (`f.quantity == null`). |

No Critical issues found. No schema version bump required.

---

## What changed

### 1. Create Project from Pricing Estimate

**Pricing tab** — new button: **Create project from estimate**

- Syncs current Pricing form to `state.pricing` before mapping.
- Opens **New project** modal prefilled via `projectDraftFromPricing()` — does **not** auto-save.
- Field mapping:
  - Pricing name → Project name
  - quantity → Project quantity
  - sale price per item × quantity → Project `soldPrice` (gross revenue)
  - material cost + mode, hardware, packaging, laser minutes, rates, fees → matching Project accounting fields
  - Pricing notes → Project `accountingNotes` (with draft reminder appended)
  - Status → `for-sale` if sale price entered, else `kept`
- Yellow draft banner in modal + brief info toast (skipped if undo toast active).
- Material and settings snapshot still require user input before save (existing validation unchanged).

### 2. Project accounting CSV export

**Projects tab** — new button: **Export accounting CSV**

- One row per project (all projects, sorted by current Projects sort preference).
- Includes project metadata, settings snapshot fields, accounting inputs, and calculated outputs.
- Uses existing `csvValue` / `csvRows` / `downloadTextFile` helpers.
- Empty accounting fields export as blank cells.

### 3. Projects status filter

- New dropdown: All statuses / Kept / Gifted / For sale / Sold
- Works alongside existing search, tag filter, and sort.
- **Clear status** button when a status filter is active.
- Session-only filter (not persisted to `localStorage`, same as tag/search filters).

### 4. Documentation

- `README.md` — bridge + CSV export noted.
- `docs/QOL_AND_PROJECT_GALLERY_PLAN.md` — Phase 6 bridge/export marked done.

---

## Changed files

| File | Changes |
|------|---------|
| `index.html` | Bridge functions, CSV export, status filter, toast helper, UI wiring (+129 lines) |
| `README.md` | Feature notes (+4 lines) |
| `docs/QOL_AND_PROJECT_GALLERY_PLAN.md` | Status updates (+5 lines) |

`Genmitsu L8 Tracker.dc.html` — not touched.

---

## Validation

| Command | Result |
|---------|--------|
| `git diff --check` | **Pass** (LF/CRLF warnings only) |
| `python -m html.parser index.html` | **Pass** (exit 0) |
| `Select-String` external dependency scan | **Pass** (no matches) |

---

## Skipped / risky / known limits

| Item | Notes |
|------|-------|
| **Material not mapped from Pricing** | Pricing has no material field — user must enter material before Save project. |
| **Settings snapshot empty from Pricing** | Bridge fills accounting only; laser settings still entered manually or via Log/Library promote flows. |
| **Pricing in JSON Export** | Updated later in Backup/Import Completeness Pack 1 — Pricing draft/defaults now export with the main JSON backup. |
| **CSV exports all projects** | Not limited to current filter; uses full `state.projects` list. |
| **Status filter not persisted** | Resets on reload (by design, matches tag filter behavior). |
| **Invoice/tax/inventory/customers** | Out of scope per constraints. |

---

## Suggested git commands

```powershell
cd "C:\Genmitsu L8 Tracker"
git status
git diff
git add index.html README.md docs/QOL_AND_PROJECT_GALLERY_PLAN.md docs/PROJECT_ACCOUNTING_BRIDGE_IMPLEMENTATION.md
git commit -m "Add Pricing-to-Project bridge and accounting CSV export"
git push origin main
```

---

## Suggested commit message

```
Add Pricing-to-Project bridge and accounting CSV export

- Open a prefilled Project draft from the Pricing tab without auto-saving
- Export per-project accounting inputs and calculated totals as CSV
- Filter Projects by sale/status (kept, gifted, for-sale, sold)
- Document bridge and export in README and QOL plan
```

---

*End of report.*
