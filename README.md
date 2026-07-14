# Genmitsu L8 Tracker

A single-file web app for logging and organizing laser settings for a Genmitsu L8 20W diode laser engraver, so results are repeatable across sessions.

## Files to use

- `index.html` - standalone offline baseline. This is the preferred file to open locally because it has no `support.js`, x-dc, or Google Fonts dependency.
- `Genmitsu L8 Tracker.dc.html` - original Claude/Grok artifact export kept for reference and comparison.

## Why this exists

Getting clean, repeatable cuts and engraves out of a diode laser depends on a pile of interacting numbers - power, speed, passes, line interval, focus height, air assist, and more - that vary by material and thickness. Without a system, that knowledge lives in scattered notes (or nowhere), and every new sheet of material means re-guessing settings from scratch. This tool gives that data a permanent, searchable home.

## What it does

The app has eight views:

- **Log** - a quick-entry record of every job you run: material, thickness, job type (cut/engrave), power (with optional min/max ramp), speed, passes, line interval, air assist + pressure, overscan %, kerf offset, dither mode, focus height/material Z-offset, software used, settings file name, a 1-5 result rating, and free-form notes. Searchable, sortable, and filterable by material.
- **Library** - a curated set of "best known settings" per material/thickness, promoted from log entries once you've dialed something in. Manage mode preserves the searchable/sortable/taggable card reference and remains the legacy-safe default. Browse mode adds a material -> thickness -> operation tree with a selected profile detail, suggested Inventory matches, and related Projects. Its Material Test Wizard guides matrix, interval, cut, kerf, and manual tests into each material's test history, with an explicit option to apply winners to primary settings.
- **Test Grids** - a power x speed test matrix planner for dialing in a brand-new material: set ranges and step sizes, get a grid, then click each cell to record a rating/notes and mark winning combinations. Manage mode is the compatibility default and preserves the existing cards and matrix workflow. Browse mode adds a material -> operation -> Test Grid tree, search and operation filtering, selected-grid completion and best-cell summaries, and completed Library Material Test promotion history. Any cell can still be promoted straight to a new Library draft; optional structured Material Test recording and profile-local provenance deduplication remain in the matrix workflow. Recorded results can be exported as CSV.
- **Designs** - offline SVG generators for freestanding QR/sign stands, hanging signs, dice trays, and divider trays. Dimensions, material thickness, joint style, and fit clearance can be adjusted before downloading a LightBurn-ready SVG; test fits on scrap are still recommended. The Designs form is session-only and is not included in JSON backups.
- **Reference** - official Genmitsu L8 20W and 40W speed/power starting points from SainSmart's resource center, selected by machine profile, plus focus, safety, maintenance, offline, and software notes pulled from the user manual. Any reference setting can be copied into the Library and tuned from there.
- **Projects** - a searchable/taggable gallery of finished pieces with a saved settings snapshot, small offline photos, tags, notes, optional sale/cost/profit accounting, filtered accounting summaries, status filtering, accounting CSV export, and a copy-to-log action for repeating a past project. Manage mode is the legacy-safe default and preserves the sortable Project cards. Browse mode adds a status -> primary material -> Project tree with selected Project details, source-record context, suggested Inventory matches, and related finished batches. The accounting report remains above both modes and continues to follow the current Project filters. Projects can also be opened as drafts from Pricing estimates.
- **Inventory** - a lightweight manual tracker for raw materials on hand and simple finished-item batches, with low-stock badges, estimated raw-material value, remaining finished counts, starter-material helpers, canonical material aliases, and CSV exports. It includes Manage cards and a collapsible Browse mode with search, in-stock filtering, stock details, suggested Library matches and Material Tests, related Projects, and draft actions. Manage is the safe default for legacy data; Finished Batches remain separate and nothing auto-deducts stock.
- **Pricing** - a compact cost/profit calculator for estimating revenue, material and consumable costs, machine time, labor, fees, margin, break-even price, target sale prices, saved rate/fee defaults, printable summaries, CSV export, and a copyable summary before committing to a price.

Other features:

- Toggle between imperial and metric units, with stored thickness/focus/Z-offset values converted when switching.
- Export/import your data and portable app preferences, including the Library, Test Grids, Projects, and Inventory Manage/Browse preferences, as a JSON backup file with merge or replace import modes.
- Storage failures are caught without overwriting unreadable browser data. The recovery panel can download the damaged raw data, import a valid backup, or explicitly start fresh. Photo-heavy JSON exports also show an approximate size warning before download.
- Material suggestions include saved materials, official SainSmart reference materials, and raw Inventory material names with stock status when available.
- Inventory raw materials can store comma-separated aliases so Amazon-style listing titles or old shorthand can match a clean canonical material name without blocking free typing.
- Material safety warnings flag obvious risky/unknown terms such as PVC, vinyl, chlorinated plastics, unknown plastics, fiberglass, and carbon fiber without blocking saves. Inventory Browse suppresses laser-action drafts for clear unsafe or mystery cases until composition is verified.
- Laser, Project, and Pricing fields include short hover/focus help for beginner-friendly terminology.
- Library includes a closest-saved-setting helper for matching material and thickness.
- Keyboard shortcuts: `Ctrl+N` creates a new item in the active view, `Ctrl+S` saves the open form, `Esc` closes modals or exits grid detail, and `/` focuses the active search field.
- Delete actions show a short undo window before the removal becomes final.
- Duplicate Log entries and Library profiles for quick tweak-and-resave runs.
- Print a simplified Library cheat sheet for shop use. Printing from Browse still uses the complete filtered Library card summary rather than only the selected profile.
- Pricing calculator drafts/defaults persist locally between sessions and are included in the main JSON backup/export.
- **Create project from estimate** on the Pricing tab opens a prefilled Project draft (user must still click Save project).
- Project accounting can copy a matched raw Inventory unit cost into the current Project as an explicit one-time action.
- Project accounting can track optional material line items and copy their total into the Project material cost field when you choose.
- Project photos are resized/compressed before storage and ride inside Project records; image-heavy backups can get larger.
- **Accounting report** on the Projects tab summarizes filtered project revenue, cost, profit, margin, quantity, and status counts.
- **Export accounting CSV** on the Projects tab exports one row per project with accounting inputs and calculated totals, including a material line count and material line total.
- **Inventory CSV exports** save filtered raw-material stock and finished-batch counts as plain CSV files.
- All data persists automatically in the browser (`localStorage`) between sessions. Pricing results update immediately while browser writes are briefly debounced to avoid serializing the full tracker on every keystroke.
- Log, Projects, Library, Test Grids, and Inventory searches update only their result areas, preserving search focus and avoiding unnecessary full-tab rebuilds.

## Current storage model

The standalone `index.html` app saves live data in the browser's `localStorage` under `genmitsu-l8-tracker-v1`. That means data is tied to the browser/profile that opened the app. Use Export regularly for portable JSON backups until a future desktop build writes to a real app data file.

JSON backups include Log entries, Library profiles, Test Grids, Projects, Project photos, Inventory records including raw-material aliases, unit preference, saved sort preferences, the Library, Test Grids, Projects, and Inventory Manage/Browse preferences, Library match helper inputs, selected machine profile, Pricing draft, and Pricing defaults. Session-only browser selections and expanded nodes, search text, tag/status filters, active tab, and open modal state are intentionally not backed up. Missing or invalid `gridViewMode` values safely default to Manage.

If the saved `localStorage` value is malformed, the app opens with temporary empty data and blocks normal saving so the damaged value is not silently replaced. Use the recovery controls before continuing.

## Fields tracked

Log entries, Library profiles, Test Grids, and Projects track material, thickness, job type, power (min/max), speed, passes, line interval, air assist (on/off + pressure), overscan %, kerf offset, dither mode (for photo engraves), focus height / material Z-offset, software (e.g. LightBurn), settings file reference, result rating, and notes where applicable. Inventory tracks raw material stock counts/costs and finished-batch quantities manually, and its raw-material names/aliases can help keep material names consistent; it does not auto-deduct from Projects.

## Official source material

- Local manual: `docs/Genmitsu_L8_Laser_Engraving_Machine_User_Manual.pdf`
- Local 20W chart: `docs/L8_Engraving_Speed_Power_Reference_Chart_20W.pdf`
- SainSmart L8 resources: https://docs.sainsmart.com/article/1wosao83f2-genmitsu-l-8-resources
- SainSmart L8 speed/power chart: https://docs.sainsmart.com/article/389gfcep9u-l-8-engraving-speed-power-reference-chart

## Built-in checks

The standalone page includes browser-console fixture tests for storage recovery, material-test normalization, Test Grid promotion, Grid Browser behavior, Material Browser behavior, Library Browser behavior, Project Browser behavior, wizard metadata, and operation-aware baseline selection.

- Open `index.html?selftest=all` to run every fixture group at startup.
- Individual query values are `baseline`, `normalization`, `grid`, `grid-browser`, `materials`, `library-browser`, `project-browser`, `metadata`, and `storage`.
- The same functions are exposed in the console as `runBaselineResolutionFixtures()`, `runMaterialTestNormalizationFixtures()`, `runGridPromotionFixtures()`, `runGridBrowserFixtures()`, `runMaterialBrowserFixtures()`, `runLibraryBrowserFixtures()`, `runProjectBrowserFixtures()`, `runWizardMetadataFixtures()`, and `runStorageRecoveryFixtures()`.

## Status

Actively being built out as the laser is set up and put through its first material tests. The repo is pushed to GitHub for version history and backup: https://github.com/samuelsonjoe-lgtm/Genmitsu_L8_Tracker
