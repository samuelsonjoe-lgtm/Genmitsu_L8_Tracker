# Genmitsu L8 Tracker

A single-file web app for logging and organizing laser settings for a Genmitsu L8 20W diode laser engraver, so results are repeatable across sessions.

## Why this exists

Getting clean, repeatable cuts and engraves out of a diode laser depends on a pile of interacting numbers - power, speed, passes, line interval, focus height, air assist, and more - that vary by material and thickness. Without a system, that knowledge lives in scattered notes (or nowhere), and every new sheet of material means re-guessing settings from scratch. This tool gives that data a permanent, searchable home.

## What it does

The app has four views:

- **Log** - a quick-entry record of every job you run: material, thickness, job type (cut/engrave), power (with optional min/max ramp), speed, passes, line interval, air assist + pressure, overscan %, kerf offset, dither mode, focus height/material Z-offset, software used, settings file name, a 1-5 result rating, and free-form notes. Searchable and filterable by material.
- **Library** - a curated set of "best known settings" per material/thickness, promoted from log entries once you've dialed something in. This is the reference sheet you check before starting a new job.
- **Test Grids** - a power x speed test matrix planner for dialing in a brand-new material: set ranges and step sizes, get a grid, then click each cell to record a rating/notes and mark the winning combination. Any cell can be promoted straight to the Library.
- **Reference** - official Genmitsu L8 20W and 40W speed/power starting points from SainSmart's resource center, plus focus, safety, maintenance, offline, and software notes pulled from the user manual. Any reference setting can be copied into the Library and tuned from there.

Other features:

- Toggle between imperial and metric units.
- Export/import your data as a JSON backup file, so it can move between devices or be backed up outside the browser.
- All data persists automatically in the browser (localStorage) between sessions.

## Fields tracked (per entry/profile/grid)

Material, thickness, job type, power (min/max), speed, passes, line interval, air assist (on/off + pressure), overscan %, kerf offset, dither mode (for photo engraves), focus height / material Z-offset, software (e.g. LightBurn), settings file reference, result rating, and notes.

## Official source material

- Local manual: `Genmitsu_L8_Laser_Engraving_Machine_User_Manual.pdf`
- SainSmart L8 resources: https://docs.sainsmart.com/article/1wosao83f2-genmitsu-l-8-resources
- SainSmart L8 speed/power chart: https://docs.sainsmart.com/article/389gfcep9u-l-8-engraving-speed-power-reference-chart

## Status

Actively being built out as the laser is set up and put through its first material tests. Intended to be pushed to GitHub for version history and backup: https://github.com/samuelsonjoe-lgtm/Genmitsu_L8_Tracker
