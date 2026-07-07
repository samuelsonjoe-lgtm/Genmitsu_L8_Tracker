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

### 1.2 Next up (agreed priority — biggest workshop value before Tauri)

These three were voted as the next pass:

1. **Duplicate entry button** — clone a Log entry (or Library profile) into a new draft with the same settings, dated today, ready for a quick tweak-and-resave. Removes re-typing power/speed/passes/etc. for "one more pass at the same material."
2. **Print-friendly Library view** — a `@media print` stylesheet (or a dedicated print route) that renders Library cards as a clean cheat sheet: material, thickness, job type, power/speed/passes/line interval/focus — no buttons, no chrome. Tape it up next to the laser.
3. **Import merge mode** — today's import replaces `entries`/`profiles`/`grids` wholesale. Add a choice on import: **Replace** (current behavior) vs **Merge** (append/upsert by `id`, skip or overwrite duplicates). Needed once there's more than one machine/computer touching the data.

### 1.3 Still open from the original backlog

| Item | Priority (orig. audit) | Status |
|------|------------------------|--------|
| Sort options (Log by date, Library by material/rating) | High | Not started |
| Material presets/autocomplete seeded from the **Reference** table specifically | High | Partial — datalist exists but only draws from Log/Library, not the SainSmart reference rows |
| Keyboard shortcuts (`Ctrl+N`, `Ctrl+S`, `Esc`, `/`) | High | Not started |
| Version/schema stamp in exported JSON (`schemaVersion`) | Medium | Not started — matters once merge-import and the Project Gallery (below) add new fields |
| True desktop packaging (Tauri) — app-data file persistence, auto-backups | High (blocks real durability) | Not started; still on `localStorage` |
| Thickness-aware Library matching ("what do I use for 3mm birch?") | Medium | Not started |
| Tag system (`gift`, `commission`, `failed`, etc.) | Medium | Not started — would also benefit the Project Gallery below |
| Test grid export (CSV / printable HTML) | Medium | Not started |
| Unit conversion on in↔mm toggle | Medium | Not started (toggle exists, values aren't converted) |
| Undo delete (toast) | Medium | Not started |
| Machine profile selector (20W default, hide 40W table) | High | Not started |

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
| 1 | Data model + `schemaVersion: 2`, `projects: []` added to state/load/save, migration for old exports (missing `projects`/`schemaVersion` defaults to `[]`/`1`) | Nothing — can start immediately |
| 2 | Projects tab: card grid, search/filter/sort, create/edit/delete, "Save as Project" from Log & Library | Phase 1 |
| 3 | Photo capture: file input → canvas downscale → base64 `dataUrl`, primary photo display on cards | Phase 2 |
| 4 | Print/export niceties: include Projects in JSON export (already automatic once in `state`), optional single-project print view | Phase 2 |
| 5 | (Later, post-Tauri) Swap `dataUrl` storage for on-disk file storage + one-time migration | Tauri desktop packaging from the standalone-offline audit |

Phases 1–4 need nothing beyond what the app already has today (still just `index.html` + `localStorage`). Phase 5 is explicitly gated on the desktop packaging work already planned in `docs/STANDALONE_OFFLINE_AUDIT.md`.
