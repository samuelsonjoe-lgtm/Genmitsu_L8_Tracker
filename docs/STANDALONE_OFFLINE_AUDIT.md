# Genmitsu L8 Tracker — Standalone Offline Audit Report

**Prepared for:** Pearl (main PC, no internet required)
**Audit date:** July 7, 2026
**Audited source:** `C:\Genmitsu L8 Tracker` (local install)
**Upstream repo:** https://github.com/samuelsonjoe-lgtm/Genmitsu_L8_Tracker (currently empty — nothing pushed yet)
**Scope:** How Codex can convert this into a standalone, offline-accessible application; plus improvement and QOL recommendations.
**Constraint:** This document is planning only — no code changes were made.

---

## Executive summary

The Genmitsu L8 Tracker is a well-designed, feature-complete **single-page web app** (~1,450 lines) for logging laser settings, curating a material library, running power/speed test grids, and browsing official L8 reference data. The business logic, UI, and data model are already solid.

**It is not yet a standalone offline app.** Today it depends on:

1. A missing runtime file: `support.js` (x-dc / DCLogic framework)
2. Google Fonts loaded over the network
3. Browser `localStorage` for persistence (fragile, browser-specific, not a real file on disk)
4. Opening via a browser, with no desktop launcher, installer, or pinned data location

**Recommended path for Pearl:** Port the app off the x-dc framework into a self-contained HTML + vanilla JavaScript (or lightweight framework) build, bundle fonts locally, persist data to a JSON file on disk via a thin desktop wrapper (**Tauri** preferred on Windows for small footprint), and ship a one-click shortcut. Estimated effort: **2–4 focused Codex sessions** depending on how polished the desktop packaging should be.

---

## 1. Current architecture

### 1.1 File inventory

| File | Role | Offline-ready? |
|------|------|----------------|
| `Genmitsu L8 Tracker.dc.html` | Entire UI + application logic | Partial — needs runtime |
| `support.js` | x-dc framework (`DCLogic`, `React.createRef`, template compilation) | **Missing from repo** |
| `README.md` | Documentation | Yes |
| `docs/Genmitsu_L8_Laser_Engraving_Machine_User_Manual.pdf` | Reference manual | Yes |
| `L8_Laser_Machine_SD_Files-1/` | SD card contents (gitignored) | Yes (local only) |

### 1.2 Technology stack

- **Framework:** x-dc (custom single-file app platform)
  - Custom elements: `<x-dc>`, `<sc-if>`, `<sc-for>`
  - Template bindings: `{{ variable }}`, `onClick="{{ handler }}"`
  - Logic class: `class Component extends DCLogic`
- **State:** In-memory React-style state via `this.setState()` + `renderVals()`
- **Persistence:** `localStorage` key `genmitsu-l8-tracker-v1`
- **Backup:** Manual JSON export/import (download/upload)
- **Styling:** Inline CSS with oklch colors; Space Grotesk + Public Sans via Google Fonts CDN

### 1.3 Data model (already well-structured)

```json
{
  "entries": [],   // Log tab — dated job records with ratings
  "profiles": [],  // Library tab — curated best-known settings
  "grids": [],     // Test Grids tab — power×speed matrices with per-cell results
  "unit": "in"     // Global imperial/metric preference
}
```

Each record uses a client-generated `id` (`uid()`), and fields are consistent across entries, profiles, and grids. Export/import already uses this schema — a strong foundation for file-based persistence.

### 1.4 Feature completeness (what already works)

| Feature | Status | Notes |
|---------|--------|-------|
| Log entries with rich fields | Complete | Power min/max, dither, kerf, overscan, focus, air assist |
| Search + material filter | Complete | Text search across notes/material/software |
| Library profiles | Complete | Promote from log, reference, or grid cells |
| Test grid planner | Complete | Up to 60 cells; rating + "best" marking |
| Reference tables (20W/40W) | Complete | SainSmart chart data embedded |
| Reference notes | Complete | Manual-derived troubleshooting/safety |
| Unit toggle (in/mm) | Complete | Does not auto-convert existing values |
| JSON export/import | Complete | Import replaces all data (with confirm) |
| PDF manual | Present locally | Not linked/opened from inside the app |

---

## 2. Offline blockers (must fix)

### Blocker A — Missing `support.js` (critical)

The HTML begins with:

```html
<script src="./support.js"></script>
```

Without this file, **the app does not run at all**. It is not in the repository or on disk at `C:\Genmitsu L8 Tracker`. This file is almost certainly provided by the x-dc authoring environment (e.g. Grok/Claude design canvas) at build/preview time, not shipped with the export.

**Codex action:** Either vendor a copy of `support.js` into the project (if licensing/source is available), or **remove the dependency entirely** by porting to standard HTML/JS. Porting is the safer long-term choice.

### Blocker B — Google Fonts CDN (medium)

```html
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk...&family=Public+Sans...">
```

Without internet, the app falls back to system fonts. It still works, but typography degrades.

**Codex action:** Download WOFF2 font files into `assets/fonts/`, add `@font-face` rules, remove CDN links.

### Blocker C — `localStorage` persistence (medium–high)

Data lives inside the browser profile, not in `C:\Genmitsu L8 Tracker\`. Risks:

- Cleared when browser cache/storage is wiped
- Different data per browser (Edge vs Chrome vs Firefox)
- `file://` protocol may restrict or isolate storage depending on browser policy
- No automatic backup — user must remember to export

**Codex action:** Persist to `%APPDATA%\GenmitsuL8Tracker\data.json` (or similar) via a desktop wrapper, with optional auto-save every N seconds.

### Blocker D — No desktop entry point (low–medium)

User must manually open an HTML file in a browser. No Start Menu shortcut, no taskbar pin workflow, no version display, no "data lives here" transparency.

**Codex action:** Package with Tauri or Electron; add `Genmitsu L8 Tracker.lnk` to Start Menu / Desktop.

### Blocker E — `file://` protocol limitations (low)

Some browsers restrict module loading, storage, or file pickers under `file://`. Serving from `localhost` (even offline) avoids this.

**Codex action:** Desktop wrapper serves app from `127.0.0.1` internally — user never needs to think about it.

---

## 3. Paths to standalone offline (ranked)

### Path 1 — Tauri desktop app (recommended)

Wrap the app in [Tauri 2](https://tauri.app/) — a small native Windows shell around a WebView2 view.

**Prereqs to verify on Pearl before choosing this path:**

- Rust/Cargo installed and available on `PATH`
- Node.js or another frontend build runner available, if the port uses one
- Microsoft Visual Studio Build Tools / Windows SDK available for native builds
- WebView2 Runtime present or bundled by the installer
- Ability to build without internet after dependencies are initially cached

**Why this fits Pearl:**

- Fully offline after install
- Tiny installer (~5–15 MB vs Electron's ~150 MB)
- Native Windows WebView2 (already on Windows 10/11)
- Rust backend can read/write `data.json` reliably
- Can open PDF manual, export backups to user-chosen folders
- Single `.exe` + shortcut — feels like real software

**Codex implementation outline:**

```
Genmitsu L8 Tracker/
├── app/                          # Ported frontend (no x-dc)
│   ├── index.html
│   ├── css/styles.css            # Extracted from inline styles (optional)
│   ├── js/
│   │   ├── app.js                # State, persistence hooks, render
│   │   ├── data.js               # Reference tables, constants
│   │   └── components/           # Tab views, modals
│   └── assets/fonts/
├── src-tauri/                    # Tauri Rust backend
│   ├── src/main.rs               # File I/O, window, auto-save
│   └── tauri.conf.json
├── data/                         # Default/template data
├── docs/
└── scripts/
    └── build.ps1                 # One-command build for Pearl
```

**Persistence upgrade:** Frontend calls `invoke('load_data')` / `invoke('save_data', { json })` instead of `localStorage`. Rust writes to `%APPDATA%\GenmitsuL8Tracker\data.json`. Export/import still available as user-facing backup.

**Effort:** ~2–3 Codex sessions (port + Tauri scaffold + build script)

---

### Path 2 — Vanilla HTML port, no framework (minimum viable offline)

Remove x-dc entirely. Rebuild rendering with plain DOM manipulation or a ~5 KB micro-library. Open via local static server or double-click HTML.

**Pros:** Simplest codebase, no Rust/Node toolchain for daily use
**Cons:** Still browser-dependent storage unless you add a tiny local server; less "app-like"

**Codex implementation outline:**

1. Extract `renderVals()` output into a `render()` function that builds DOM with `document.createElement` or `innerHTML` templates
2. Replace `<sc-if>` / `<sc-for>` with helper functions `showIf(cond, el)` and `forEach(list, fn)`
3. Replace `handleInput` with standard `addEventListener('input', ...)`
4. Bundle fonts locally
5. Add `launch.bat` that runs `python -m http.server 8765` and opens `http://localhost:8765` (Python is often already installed; otherwise bundle a tiny server exe)

**Effort:** ~1–2 Codex sessions

---

### Path 3 — Electron wrapper (alternative desktop shell)

Same as Tauri but using Electron. Faster to scaffold if Node.js is already on Pearl, but heavier install and memory use.

**Choose Electron if:** Pearl already has Node tooling and you want the fastest path to a tray icon + auto-updater later.
**Choose Tauri if:** Small footprint and native feel matter more.

**Effort:** ~2 Codex sessions

---

### Path 4 — Keep x-dc, vendor `support.js` (not recommended)

If the original authoring tool can export `support.js`, drop it next to the HTML file and open locally.

**Why not recommended:**

- Dependency on an opaque runtime you don't control
- Unclear license and update path
- Still stuck with `localStorage` and browser UX
- `.dc.html` tooling may break silently on browser updates

Only use this as a **temporary bridge** while the port is in progress.

---

### Path 5 — Progressive Web App (PWA)

Add a service worker and `manifest.json` for offline caching.

**Why not recommended for Pearl:**

- PWAs still run inside a browser
- `localStorage` problem remains
- Installing a PWA on Windows is less intuitive than a `.exe`
- Good as a *secondary* distribution channel, not primary for an offline workshop PC

---

## 4. Recommended Codex execution plan

### Phase 0 — Commit docs and choose the runtime path (same day)

| Task | Owner | Deliverable |
|------|-------|-------------|
| Commit the moved docs/manual | Codex | `docs/` is tracked cleanly |
| Verify whether `support.js` is available | Codex | App opens in browser OR blocker is documented |
| Decide bridge vs port | Codex + Pearl | Prefer port; vendored `support.js` only as a temporary bridge if source/licensing is clear |
| Document current data location | Codex | README section for browser `localStorage` + JSON export |
| Pick first implementation path | Codex + Pearl | Vanilla offline port first, then Tauri wrapper if toolchain is available |

### Phase 1 — Port off x-dc (core refactor)

| Task | Detail |
|------|--------|
| Split monolith | Separate HTML structure, CSS, JS modules — keep behavior identical |
| Replace DCLogic | Plain class or module with `state`, `setState()`, `render()` |
| Replace template tags | Standard DOM rendering; preserve all four tabs and modals |
| Preserve data schema | Keep `genmitsu-l8-tracker-v1` format for backward-compatible import |
| Migration on first launch | If `localStorage` has data and file is empty, import once then clear |

**Acceptance criteria:** Every flow in the README works identically: log, library, grids, reference, export, import, unit toggle.

### Phase 2 — Desktop packaging (Pearl deliverable)

| Task | Detail |
|------|--------|
| Tauri scaffold | Window title, icon, single-instance |
| File persistence | `%APPDATA%\GenmitsuL8Tracker\data.json` |
| Auto-save | Debounced save on every state change (300 ms) |
| Auto-backup | Daily copy to `%APPDATA%\GenmitsuL8Tracker\backups\` |
| Installer | `Genmitsu-L8-Tracker-Setup.exe` or portable zip |
| Desktop shortcut | "Genmitsu L8 Tracker" on Desktop + Start Menu |
| Open manual | Button in Reference tab → opens PDF via system default viewer |

### Phase 3 — QOL pass (see Section 5)

Pick high-value items from the improvement backlog below.

---

## 5. Improvements, additions, and QOL recommendations

### 5.1 High priority (workshop usefulness)

| # | Improvement | Rationale |
|---|-------------|-----------|
| 1 | **Auto-save to disk + daily backup** | Never lose a month of dialing-in data |
| 2 | **"Duplicate entry" button** | Small tweaks between test runs are common |
| 3 | **Library search/filter** | Log has search; Library does not — inconsistency |
| 4 | **Sort options** | Log by date (newest first), Library by material name, rating |
| 5 | **Focus height helper** | Reference tab lists 3mm/5mm/7mm rules — add a calculator: "I am cutting 6mm → suggests 5mm focus" |
| 6 | **Link to local PDF manual** | File is already in `docs/` — one click from Reference tab |
| 7 | **Machine profile selector** | You have a 20W; 40W table is collapsed noise — let user set "My machine: 20W" in settings |
| 8 | **Keyboard shortcuts** | `Ctrl+N` new entry, `Ctrl+S` save modal, `Esc` close modal, `/` focus search |
| 9 | **Print-friendly Library view** | Printable cheat sheet for the laser area |
| 10 | **Import merge mode** | Current import *replaces* everything — add "merge" option |

### 5.2 Medium priority (repeatability)

| # | Improvement | Rationale |
|---|-------------|-----------|
| 11 | **Material presets / autocomplete** | datalist exists for software but not materials — seed from reference table |
| 12 | **Thickness-aware Library matching** | "What do I use for 3mm birch?" quick lookup |
| 13 | **Tag system** | e.g. `gift`, `commission`, `failed` — filter beyond material |
| 14 | **Photo attachments** | Store path to result photo per entry (file path, not base64 — keep DB small) |
| 15 | **LightBurn profile path field** | Click to open `.lbrn2` if path still valid |
| 16 | **Test grid export** | Export grid as CSV or printable HTML for marking at the machine |
| 17 | **Test grid cell notes in Library promote** | Already partially there — surface source grid name in profile notes |
| 18 | **Unit conversion on toggle** | Switching in↔mm could offer to convert thickness/focus values |
| 19 | **Undo delete** | 5-second toast "Undo" after deleting entry/profile/grid |
| 20 | **Version stamp in export JSON** | `"schemaVersion": 2` for future migrations |

### 5.3 Lower priority (nice to have)

| # | Improvement | Rationale |
|---|-------------|-----------|
| 21 | Dark mode | Workshop lighting varies |
| 22 | Statistics dashboard | Avg rating per material, most-tested materials chart |
| 23 | Cut vs engrave color coding | Visual scan in Log cards |
| 24 | Multi-machine support | If you add a second laser later |
| 25 | G-code / job name field | Tie log entry to `001.nc` offline file |
| 26 | Lens cleaning reminder | Counter by runtime hours (manual input) |
| 27 | Material safety warnings | PVC/chlorine, coated metals — from reference notes |
| 28 | SD card material test files | Cross-link `L8_Laser_Machine_SD_Files-1/08_ Material Test` |
| 29 | Tray icon + minimize to tray | Keep running during long engraving sessions |
| 30 | Portable mode | `data.json` next to exe for USB stick use |

### 5.4 Code / project hygiene (for Codex maintainability)

| Item | Current state | Recommendation |
|------|---------------|----------------|
| Filename with spaces | `Genmitsu L8 Tracker.dc.html` | Rename to `index.html` in packaged app |
| Monolithic file | 1,450 lines | Split into modules once ported off x-dc |
| Inline styles | Every element | Extract to CSS variables for theming (`--color-power`, etc.) |
| Magic strings | `'genmitsu-l8-tracker-v1'` | Centralize in `config.js` |
| GitHub repo | Empty remote | Push after Phase 0 for backup |
| Large PDF in git | 12 MB | Keep in `docs/`; consider Git LFS if repo grows |
| Missing `support.js` | Not tracked | Do not commit opaque binaries — port instead |
| No tests | — | Add JSON schema validation tests for import/export |
| No CI | — | Optional: GitHub Action that builds Tauri on tag |

---

## 6. Pearl-specific deployment guide (target end state)

After Codex completes Phase 2, this is what Pearl should look like:

### Install layout

```
C:\Program Files\Genmitsu L8 Tracker\
├── Genmitsu L8 Tracker.exe
├── resources\
│   ├── app\                  # Frontend assets
│   └── manual.pdf
└── uninstall.exe

%APPDATA%\GenmitsuL8Tracker\
├── data.json                 # Live database
├── backups\
│   ├── 2026-07-07.json
│   └── 2026-07-08.json
└── settings.json             # Machine profile, theme, window size
```

### User workflow (offline)

1. Double-click **Genmitsu L8 Tracker** on Desktop
2. App opens — no browser chrome, no internet
3. Log a job → auto-saved to `data.json`
4. Before a new material → check **Library** or **Reference**
5. Unknown material → **Test Grids** → promote winner to Library
6. Monthly → **Export** still available for USB backup to another machine

### First-run migration (if coming from browser version)

1. Open old browser version once (if still accessible)
2. Export JSON backup
3. In desktop app → Import → verify entries
4. Desktop app stores to `%APPDATA%` going forward

---

## 7. Risk register

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| `support.js` unavailable | High | App won't run today | Port off x-dc (Phase 1) |
| Port introduces UI regressions | Medium | Trust loss | Side-by-side checklist against README flows |
| WebView2 missing on Pearl | Low | Won't launch | Tauri installer bundles WebView2 bootstrapper |
| Data loss on migration | Medium | High | Import merge + auto-backup before first save |
| Font licensing | Low | Legal | Google Fonts are OFL — keep license files in `assets/fonts/` |
| Scope creep | High | Delayed delivery | Ship Phase 2 first; QOL in Phase 3 |

---

## 8. What Codex should NOT do

- Do not require internet for core functionality
- Do not store photos as base64 in JSON (paths only)
- Do not switch to a cloud database or account system
- Do not rewrite in React/Vue unless there's a strong reason — vanilla JS matches the current simplicity
- Do not remove export/import — it's the escape hatch for data portability
- Do not break the existing JSON backup format without a migration path

---

## 9. Suggested prompt for Codex (copy-paste ready)

Use this to kick off implementation on Pearl:

> **Task:** Convert Genmitsu L8 Tracker from x-dc single-file HTML to a standalone offline Windows desktop app.
>
> **Source:** `C:\Genmitsu L8 Tracker`
>
> **Requirements:**
> 1. Remove dependency on `support.js` / x-dc — port all logic to vanilla HTML/CSS/JS
> 2. Bundle Space Grotesk and Public Sans fonts locally
> 3. Package with Tauri 2 for Windows (WebView2)
> 4. Persist data to `%APPDATA%\GenmitsuL8Tracker\data.json` with debounced auto-save
> 5. Daily auto-backup to `%APPDATA%\GenmitsuL8Tracker\backups\`
> 6. Preserve existing JSON export/import format (`entries`, `profiles`, `grids`, `unit`)
> 7. Add Desktop + Start Menu shortcut via installer
> 8. Add "Open Manual" button in Reference tab for the local PDF
> 9. Add machine profile setting (20W default, 40W optional) to de-emphasize irrelevant reference table
> 10. Do not require internet after install
>
> **Acceptance:** All four tabs work identically to current app. Export from old app → import into new app → data intact. App launches offline from Desktop shortcut.

---

## 10. Summary verdict

| Question | Answer |
|----------|--------|
| Is the app conceptually ready for standalone use? | **Yes** — data model and features are mature |
| Can it run offline on Pearl today? | **No** — missing `support.js`, CDN fonts, no desktop package |
| Best path forward? | **Port off x-dc + Tauri desktop wrapper + file persistence** |
| Estimated Codex effort? | **2–4 sessions** (MVP in 2, polished desktop in 4) |
| Highest-value QOL wins? | Auto-save/backup, Library search, focus helper, PDF link, machine profile |

The project is closer to "finished product" than "prototype" on the feature side. The gap is entirely **packaging and persistence** — exactly the kind of work Codex can execute systematically without redesigning the app from scratch.

---

*End of audit report.*
