# Genmitsu L8 Tracker — Full Project Audit & Roadmap

**Date:** 2026-07-08  
**Commit audited:** `2f8d0f7` — *Add field help tooltips*  
**Scope:** Read-only audit of `index.html` standalone app  
**Auditor constraints:** No file edits; offline single-file design must remain intact

---

## Executive summary

At `2f8d0f7`, the Genmitsu L8 Tracker is a mature **six-tab offline workshop notebook**: Log, Library, Test Grids, Reference, Projects, and Pricing. Recent work has turned it from a settings logger into a credible beginner workflow tool — dial-in grids, official reference starting points, portfolio projects with photos, scratch pricing, optional project accounting, safety warnings, portable JSON backups, and field-level help tooltips.

**Architecture is intact:** one self-contained `index.html` (~142 KB, ~2,242 lines), no CDN/scripts/modules, no `fetch`/XHR, system fonts only. `Genmitsu L8 Tracker.dc.html` was **not modified** since `7417509` (legacy x-dc artifact, ~109 KB).

**Strengths today:** repeatable settings capture, promotion paths (Log → Library → Project), thickness-aware library matching, machine profile selector (20W/40W), undoable deletes, keyboard shortcuts, CSV/print/copy exports, and beginner-oriented terminology help on dense forms.

**Main risks:** `localStorage` size limits — especially with project photos embedded as JPEG data URLs in JSON — and uneven quota-error handling. Import **replace** mode does not fully reset pricing/preferences when importing older backups. A few search UX inconsistencies and one long Project modal remain the biggest usability friction points.

**Verdict: Safe to keep building**, with a short **fix-soon** pass on storage/search/import edge cases before adding more photo- or data-heavy features. No critical blockers; pause for broad QA only if you plan to rely heavily on photos or multi-device backup workflows.

---

## Current feature inventory

### Views (6 tabs)

| Tab | Purpose | Key capabilities |
|-----|---------|------------------|
| **Log** | Chronological job history | Stats dashboard, search/filter/sort, 1–5 rating, duplicate, promote to Library, save as Project |
| **Library** | Best-known settings per material | Tags, search/filter/sort, thickness matcher (top 4), print sheet, duplicate, save as Project |
| **Test Grids** | Power × speed matrix | Configurable ranges (capped at 60 steps), per-cell rating/notes/best flag, promote to Library, CSV export |
| **Reference** | Official starting points | 20W/40W profile selector, embedded tables, local PDF links, 20W focus helper, safety/maintenance notes |
| **Projects** | Finished-piece portfolio | Settings snapshot, up to 3 compressed photos, tags, status (kept/gifted/for-sale/sold), optional accounting, filter-aware report + copy, accounting CSV export, copy to Log |
| **Pricing** | Scratch estimator | Live totals, target margin/profit prices, saved rate defaults, print/CSV/copy, create Project draft (no auto-save) |

### Cross-cutting features

- **Units:** Global in/mm toggle with conversion of stored thickness, focus, and Z-offset fields
- **Tags:** Comma-separated on Library + Projects; pill display; tag filters + clear actions
- **Safety:** `materialSafetyWarning()` on material fields/cards (PVC, vinyl, chlorinated, unknown plastics, fiberglass, carbon fiber, coated unknown metal)
- **Help:** `help()` / `labelText()` `?` badges on laser, grid, pricing, accounting, and photo fields (native `title` + `aria-label`)
- **Keyboard:** `Ctrl+N` new item, `Ctrl+S` save modal form, `Esc` close modal / exit grid detail, `/` focus search
- **Undo:** 8-second restore toast for Log, Library, Projects, and Test Grid deletes
- **Backup:** Export/import JSON with `schemaVersion: 2`; merge-by-`id` or replace; includes pricing, prefs, sorts, machine profile, library match inputs
- **Photos:** Client-side resize (~1200px max, JPEG ~0.72 quality), stored in `project.photos[].dataUrl`, ride in localStorage and JSON backup

### Beginner workflow supported today

1. **First material test** — Reference tab → pick 20W/40W → save row to Library or run a Test Grid  
2. **Dial in settings** — Grid cells → rate → promote best cell to Library  
3. **Log real jobs** — Log entry with rating/notes → promote winners to Library  
4. **Finish a piece** — Save as Project (from Log/Library/Pricing) → add photos → optional accounting  
5. **Price before selling** — Pricing tab estimate → optional Project draft → status `for-sale` / `sold`  
6. **Repeat later** — Library matcher or Project gallery → copy to Log  

---

## Findings by severity

### Critical

*None identified.* Math, persistence, and offline constraints are sound for normal use. Storage limits are a **capacity risk**, not a logic crash, and project save already surfaces a user-facing message on quota failure.

### Important

| # | Area | Finding |
|---|------|---------|
| I1 | **Storage / photos** | Photos inflate `localStorage` and JSON backups quickly (base64 JPEG in every project). No proactive size indicator or pre-save estimate. Only project **Save** wraps `upsert` in try/catch; other `persist()` calls (Log, Library, Grids, Pricing keystrokes) can throw `QuotaExceededError` uncaught once storage is full. |
| I2 | **Import replace semantics** | `replaceData()` replaces `entries`, `profiles`, `grids`, `projects`, and `unit`, but **pricing / pricingPrefs persist** unless the import file includes those keys (`applyBackupPreferences` only overwrites when present). Replacing with an older v1 backup can leave a stale Pricing draft while records were wiped — surprising for “replace everything.” |
| I3 | **Search UX inconsistency** | Log search uses `oninput` (live). **Library and Projects search use `onchange`** — filtering only updates on blur/Enter, which feels broken on long lists. |
| I4 | **Accounting CSV vs report** | Accounting **report** respects current Projects filters; **Export accounting CSV** exports **all** projects (sorted), ignoring active search/tag/status filters. Easy to export the wrong slice. |
| I5 | **Pricing → Project bridge vocabulary** | Bridge maps per-item `salePrice` × `quantity` into Project `soldPrice` (gross batch revenue). Correct for accounting math, but beginners may not notice the field meaning shifted from per-item (Pricing) to total revenue (Project). Draft banner helps; inline note on `soldPrice` after bridge would help more. |
| I6 | **Project modal density** | Single form combines identity, photos, full settings snapshot, and accounting (~30+ fields). Tooltips help, but the modal is long on laptop and very long on mobile (`@media max-width: 700px` only collapses grid to one column). |
| I7 | **40W focus helper** | Focus helper panel is labeled “20W focus helper” and always uses 20W manual table logic, even when machine profile is 40W. Misleading for 40W owners. |
| I8 | **Maintainability** | ~2,242 lines in one IIFE is still navigable (clear `render*` / `open*` sections) but event binding in `bindPage()` + modal setup duplicates patterns. Further features increase regression risk without light extraction (see Cleanup). |

### Minor

| # | Area | Finding |
|---|------|---------|
| M1 | **Ctrl+N gap** | `openActiveNewItem()` skips Pricing (and Reference). Intentional or not, undocumented. |
| M2 | **Help tooltips on touch** | `?` badges rely on `title` tooltips — weak on mobile (no tap popover). |
| M3 | **Info toast vs undo** | `showInfoToast()` no-ops while `pendingUndo` is active — Pricing→Project draft notice can be swallowed if user recently deleted something. |
| M4 | **Grid size cap** | `range()` stops at 60 steps silently — large power/speed ranges truncate without warning. |
| M5 | **Reference PVC rows** | Official table includes PVC cut rows with “Confirm material safety” notes; safety helper only fires when material **name** matches patterns (e.g. `pvc`), not on promote unless user keeps the name. |
| M6 | **Merge import conflicts** | Merge overwrites same `id` without listing conflicts — last imported wins, no preview. |
| M7 | **loadState failure** | Corrupt `localStorage` JSON fails silently to empty defaults — user may not realize data was lost until they notice empty tabs. |
| M8 | **Clipboard under file://** | Copy summary/report uses `navigator.clipboard` with textarea fallback — correct pattern; some browsers still block clipboard on `file://`. |
| M9 | **Quantity zero** | `pricingQuantity` / `projectQuantity` clamp to ≥0; quantity `0` yields $0 revenue and NaN-ish break-even displays (“Not available”) without explaining why. |
| M10 | **Project card speed display** | `projectCard` shows `${s.speed || 0} mm/min` — blank speed reads as `0 mm/min` instead of `-`. |

### Nice-to-have

| # | Area | Finding |
|---|------|---------|
| N1 | Dark mode, stats dashboard, cut/engrave color coding on Log cards |
| N2 | Tauri / desktop packaging with file-based persistence and scheduled backups |
| N3 | Separate photo storage or optional “photos excluded” export for slim backups |
| N4 | G-code / job filename field, lens-cleaning reminder, SD-card material test cross-link |
| N5 | Richer help (expandable glossary panel vs `title` only) |
| N6 | Accounting CSV column for photo count / has-photo flag |
| N7 | Print stylesheet for Projects gallery or accounting report |

---

## Data / storage risks

### localStorage

- **Key:** `genmitsu-l8-tracker-v1`
- **Schema:** `schemaVersion: 2`
- **Persisted:** `entries`, `profiles`, `grids`, `projects` (with embedded photos), `unit`, sort prefs, library match fields, `machineProfile`, `pricing`, `pricingPrefs`
- **Session-only (not persisted):** active tab, all search strings, tag/status filters, focus helper inputs, modal state, grid detail navigation

### JSON backup / import

| Behavior | Status |
|----------|--------|
| Export includes portable preferences | ✅ Since `fa101c7` |
| Photos in projects export/import | ✅ `normalizeProjectPhotos()` accepts legacy string URLs or objects |
| Old backups without `projects` | ✅ Defaults to `[]` |
| Old backups without `pricing` | ⚠️ Merge OK; **replace leaves old pricing** (I2) |
| `schemaVersion` migration | Minimal — `loadState` uses `data.schemaVersion \|\| 1`; no explicit upgrade path beyond defaults |
| Import validation | JSON shape only; no photo size check, no duplicate-id preview |

### Photo storage model

- Max **3** photos per project; JPEG data URLs at ~1200px / quality 0.72
- Rough order-of-magnitude: **~150–400 KB stored per photo** after compression (varies by image)
- **10 projects × 3 photos** can approach **several MB** — within or beyond typical **5 MB** `localStorage` quotas depending on browser
- JSON backup size scales linearly — email/cloud sync of backups becomes painful before hard quota errors

### Recommendations (storage)

1. Add **try/catch on all `persist()`** with a single user-facing “storage full” message and link to Export  
2. Show **approximate backup size** on Export (or warn when projects contain photos)  
3. On import, optionally **strip photos** or warn when backup JSON exceeds a threshold  
4. Long-term (not compatible with pure single-file without careful design): **Tauri file persistence** or sidecar photo folder — mark as future desktop chunk  

---

## UX / beginner workflow notes

### What works well

- **Terminology help** on `2f8d0f7` is the right granularity: short, field-local, non-blocking  
- **Safety warnings** are warning-only (don’t block saves) — appropriate for a personal shop tool  
- **Pricing assumptions note** (“taxes/shipping only if you enter them”) sets honest expectations  
- **Margin definition** repeated in UI (“% of sale price, not markup on cost”) — reduces a common beginner mistake  
- **Promotion paths** are discoverable from card actions (Log → Library → Project)  
- **Reference + Library matcher** answers “what should I try first?” without leaving the app  

### Where beginners may stumble

| Scenario | Risk | Mitigation today | Gap |
|----------|------|------------------|-----|
| Enter power as watts not % | Medium | Tooltip says “power percent” | No hard validation |
| Mix per-item vs total material cost | Medium | `materialCostMode` select + help | Easy to miss if left default |
| Pricing `salePrice` vs Project `soldPrice` | Medium | Help text differs | Bridge changes semantics silently |
| Dense Project form | High | Tooltips, section headings | Still one long scroll |
| Library/Project search “not working” | Medium | — | `onchange` vs `oninput` |
| PVC in reference table name | Low | Note in table + safety on typed `pvc` | Promoted profile may not trigger if renamed |
| 40W user uses focus helper | Medium | — | Labeled 20W only |

### Forms density assessment

- **Log / Library / Grid:** Acceptable for target user; help badges add clarity  
- **Pricing:** Dense but live feedback makes it learnable  
- **Project:** Heaviest form — consider collapsible sections (Settings / Accounting / Photos) in a future polish pass, not a rewrite  

---

## Recommended next 5 small QOL tasks

1. **Fix Library + Projects search to use `oninput`** (match Log behavior) — low risk, high perceived fix  
2. **Wrap `persist()` in try/catch** with one shared `storageFullAlert()` — protects all tabs, not just Project save  
3. **Align accounting CSV with active filters** (or add “Export filtered CSV” vs “Export all”) — prevents wrong exports  
4. **On import replace, reset `pricing` / `pricingPrefs`** to defaults when absent from backup — makes replace semantically honest  
5. **Ctrl+N on Pricing** → reset/focus pricing form, or document skip in a Help line on the Pricing toolbar  

---

## Recommended next 3 bigger feature chunks

1. **Storage & backup hardening pack**  
   Backup size indicator, photo count warning, export option to omit photos, import merge summary (counts added/updated), global quota handling, optional “compact backup” — *stays offline, no new dependencies*

2. **Project modal UX pack**  
   Collapsible sections (Details / Photos / Settings / Accounting), sticky Save bar, “per-item vs total” explainer on bridged drafts, 40W focus note or hide helper when 40W selected — *incremental HTML/CSS/JS only*

3. **Desktop durability (Tauri) — when ready**  
   App-data JSON file, automatic timestamped backups, open/export without browser quota — *explicitly marked: requires Tauri build; not compatible with “open index.html only” distribution until packaged*

---

## Suggested cleanup / refactor areas (no broad rewrites)

| Area | Suggestion | Avoid |
|------|------------|-------|
| `persist()` | Centralize + quota handling | Splitting into modules requiring a bundler |
| `bindPage()` | Small bind helpers (`onInput(id, fn)`) | Framework adoption |
| Search handlers | Shared `bindSearch(inputId, stateKey)` | — |
| Accounting | Shared field defs between Pricing and Project accounting | Merging Pricing into Projects (intentionally separate today) |
| CSV export | Shared “filtered project list” helper used by report + CSV | — |
| Constants | Extract `STORAGE_KEY`, tab defs, CSV headers to top block comments/sections | Multiple files |
| Tests | Manual test checklist in docs (open file, import, photo save, CSV) | Adding npm test harness to repo unless user wants it |

---

## Architecture / safety checklist

| Check | Result |
|-------|--------|
| Standalone offline (`index.html` opens directly) | ✅ |
| No external CDN/scripts/fonts | ✅ |
| No network calls in app code | ✅ |
| `Genmitsu L8 Tracker.dc.html` unchanged since `7417509` | ✅ |
| Local PDF links (`docs/*.pdf`) | ✅ Relative paths; work when repo folder structure intact |
| `index.html` maintainability | ⚠️ Large but coherent; watch binding duplication |
| Single-file constraint preserved | ✅ |

---

## Feature interaction matrix (recent work)

| Feature | Interacts with | Notes |
|---------|----------------|-------|
| Pricing calculator | localStorage, JSON backup, Project bridge | Draft persists on every input; bridge opens modal only |
| Project accounting | Pricing bridge, CSV export, report summary | `soldPrice` = gross revenue; fees subtracted in totals |
| Pricing → Project bridge | Pricing form, Project modal, toast | Maps `laserMinutes` → `machineMinutes`; does not copy material name from pricing (pricing has no material field) |
| Photos | Projects, JSON backup, localStorage quota | Compressed; max 3; viewer uses `cellModal` |
| Tags/search/sort/filter | Log, Library, Projects | Filters session-only; sorts persist |
| Material safety | All material fields + cards | Live update via `bindMaterialSafetyWarning` |
| Tooltips | Modals, Pricing, accounting | `2f8d0f7` commit |
| Keyboard shortcuts | Modals, grids, Log/Library/Projects | Pricing/Reference excluded from Ctrl+N |
| Undo deletes | Log, Library, Projects, Grids | Blocks info toast while active |
| Machine profile | Reference table, PDF link visibility | 20W chart link hidden on 40W |

---

## What should NOT be built yet

- **Full inventory / SKU / multi-location stock** — scope creep beyond workshop notebook  
- **Cloud sync / accounts** — violates offline constraint  
- **npm/React/Vite split** — breaks open-`index.html` distribution unless carefully optional  
- **Unbounded photo galleries or original-resolution storage** — will break `localStorage`  
- **Auto-listing Etsy/Shopify integrations** — network + API dependencies  
- **Merging Pricing and Project accounting into one view** — intentional separation per Phase 6 vocabulary decision  
- **Broad rewrite into multi-file ES modules** — high risk for modest gain at current size  

---

## Roadmap summary

### Fix soon (before more photo/data features)

- I1–I4, M3, import replace pricing reset  
- Manual QA pass: 3 photos × several projects → export → import merge → replace on another browser profile  

**Follow-up:** Storage/Search/Import Fix-Soon Pack 1 addresses the core I1–I4 items, import replace optional-pref reset, filtered accounting CSV, live Library/Projects search, blank Project speed display, and visible Test Grid range caps. M3 remains a minor toast/undo interaction.

### Later nice-to-have

- Tauri desktop, dark mode, dashboard, compact backup, richer glossary  
- Project print view, grid printable HTML  

### Living plan reference

See `docs/QOL_AND_PROJECT_GALLERY_PLAN.md` for historical QOL tracking (mostly shipped through `2f8d0f7`).

---

## Final verdict

**Safe to keep building** — the app fulfills its offline single-file purpose, recent features integrate cleanly, and beginner support improved materially with tooltips and safety warnings.

**Suggested discipline:** spend one focused session on **fix-soon storage/search/import items** (especially I1–I4) before adding photo-heavy or sync-adjacent features. No need for a full development pause unless you are about to depend on large photo libraries or cross-machine backup without manual JSON export.

---

*Audit performed read-only at `C:\Genmitsu L8 Tracker` commit `2f8d0f7`. No application files were modified.*
