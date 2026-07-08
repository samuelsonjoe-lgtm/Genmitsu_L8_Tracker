# Genmitsu L8 Tracker

A single-file web app for logging and organizing laser settings for a Genmitsu L8 20W diode laser engraver, so results are repeatable across sessions.

## Files to use

- `index.html` - standalone offline baseline. This is the preferred file to open locally because it has no `support.js`, x-dc, or Google Fonts dependency.
- `Genmitsu L8 Tracker.dc.html` - original Claude/Grok artifact export kept for reference and comparison.

## Why this exists

Getting clean, repeatable cuts and engraves out of a diode laser depends on a pile of interacting numbers - power, speed, passes, line interval, focus height, air assist, and more - that vary by material and thickness. Without a system, that knowledge lives in scattered notes (or nowhere), and every new sheet of material means re-guessing settings from scratch. This tool gives that data a permanent, searchable home.

## What it does

The app has six views:

- **Log** - a quick-entry record of every job you run: material, thickness, job type (cut/engrave), power (with optional min/max ramp), speed, passes, line interval, air assist + pressure, overscan %, kerf offset, dither mode, focus height/material Z-offset, software used, settings file name, a 1-5 result rating, and free-form notes. Searchable, sortable, and filterable by material.
- **Library** - a curated set of "best known settings" per material/thickness, promoted from log entries once you've dialed something in. This is the searchable/sortable/taggable reference sheet you check before starting a new job.
- **Test Grids** - a power x speed test matrix planner for dialing in a brand-new material: set ranges and step sizes, get a grid, then click each cell to record a rating/notes and mark the winning combination. Any cell can be promoted straight to the Library, and recorded results can be exported as CSV.
- **Reference** - official Genmitsu L8 20W and 40W speed/power starting points from SainSmart's resource center, selected by machine profile, plus focus, safety, maintenance, offline, and software notes pulled from the user manual. Any reference setting can be copied into the Library and tuned from there.
- **Projects** - a searchable/sortable/taggable gallery of finished pieces with a saved settings snapshot, tags, notes, optional sale/cost/profit accounting, status filtering, accounting CSV export, and a copy-to-log action for repeating a past project. Projects can also be opened as drafts from Pricing estimates.
- **Pricing** - a compact cost/profit calculator for estimating revenue, material and consumable costs, machine time, labor, fees, margin, break-even price, target sale prices, saved rate/fee defaults, printable summaries, CSV export, and a copyable summary before committing to a price.

Other features:

- Toggle between imperial and metric units, with stored thickness/focus/Z-offset values converted when switching.
- Export/import your data as a JSON backup file, with merge or replace import modes.
- Material autocomplete includes both your saved materials and official SainSmart reference materials.
- Library includes a closest-saved-setting helper for matching material and thickness.
- Keyboard shortcuts: `Ctrl+N` creates a new item in the active view, `Ctrl+S` saves the open form, `Esc` closes modals or exits grid detail, and `/` focuses the active search field.
- Delete actions show a short undo window before the removal becomes final.
- Duplicate Log entries and Library profiles for quick tweak-and-resave runs.
- Print a simplified Library cheat sheet for shop use.
- Pricing calculator drafts/defaults persist locally between sessions, but are not part of the main JSON backup/export yet.
- **Create project from estimate** on the Pricing tab opens a prefilled Project draft (user must still click Save project).
- **Export accounting CSV** on the Projects tab exports one row per project with accounting inputs and calculated totals.
- All data persists automatically in the browser (localStorage) between sessions.

## Current storage model

The standalone `index.html` app saves live data in the browser's `localStorage` under `genmitsu-l8-tracker-v1`. That means data is tied to the browser/profile that opened the app. Use Export regularly for portable JSON backups until a future desktop build writes to a real app data file.

## Fields tracked

Log entries, Library profiles, Test Grids, and Projects track material, thickness, job type, power (min/max), speed, passes, line interval, air assist (on/off + pressure), overscan %, kerf offset, dither mode (for photo engraves), focus height / material Z-offset, software (e.g. LightBurn), settings file reference, result rating, and notes where applicable.

## Official source material

- Local manual: `docs/Genmitsu_L8_Laser_Engraving_Machine_User_Manual.pdf`
- Local 20W chart: `docs/L8_Engraving_Speed_Power_Reference_Chart_20W.pdf`
- SainSmart L8 resources: https://docs.sainsmart.com/article/1wosao83f2-genmitsu-l-8-resources
- SainSmart L8 speed/power chart: https://docs.sainsmart.com/article/389gfcep9u-l-8-engraving-speed-power-reference-chart

## Status

Actively being built out as the laser is set up and put through its first material tests. The repo is pushed to GitHub for version history and backup: https://github.com/samuelsonjoe-lgtm/Genmitsu_L8_Tracker
