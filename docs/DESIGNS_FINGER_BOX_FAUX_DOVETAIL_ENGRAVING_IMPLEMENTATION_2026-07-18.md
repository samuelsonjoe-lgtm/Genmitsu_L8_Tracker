# Finger Box Faux Dovetail Engraving — Implementation

**Date:** 2026-07-18  
**Baseline:** `0fa1d37 Add custom row drawer cabinet layouts`  
**Scope:** bounded Finger Box decorative-score option only; no commit or push.

## Starting and final state

- Start: `main`, `HEAD...origin/main = 0 0`, no staged files, and a clean tracked tree. The pre-existing untracked reports, `.claude/`, `LightBurn Projects/`, `debug.log`, and utility files were left untouched.
- Finish: modified tracked files are `index.html` and `README.md`; this report is new and untracked. The authoritative architecture review was not edited.
- No storage key, schema, import/export, backup, Production Settings, evidence/promotion, Library, Inventory, Projects, Test Grid, or Material Test code was changed.

## User contract

Finger Box now has a session-only **Corner decoration** select:

- `None` (`none`)
- `Faux dovetail engraving (decorative only)` (`faux-dovetail-engrave`)

`boxCornerDecoration` defaults to `none`. Normalization exposes `cornerDecoration`; missing, blank, and `none` normalize to `none`; only `faux-dovetail-engrave` enables output; unknown nonblank input returns `Choose a recognized corner decoration.` The field is part of `designDraft` only and is absent from persisted state and backups.

The help text states that the marks follow actual vertical finger positions, are decorative only, leave the finger joint/cut geometry/fit/strength unchanged, and require blue score before red cut. Enabled output warns that engraved faces should face outward, the appearance has not been physically verified, a scrap or small first result should be inspected, and cutting must not be left unattended. When assembly labels are also enabled, the result honestly warns that labels and decoration use the same engraved sheet face and may both appear outward.

## Geometry and production contract

`fingerBoxCornerDecorationSegments()` is a Finger-Box-local helper. It reads each existing wall panel's actual vertical `designPatternEdge()` descriptor and decorates only its outer finger intervals (`isFinger`), never a separate ornamental rhythm. It uses the existing boundary/phase/clearance and terminal-trim calculations, so phase ownership matches the visible finger/end-grain positions on all four wall panels. Panel order is Front, Back, Left, Right; each panel emits its left edge before its right; marks follow the source pattern index.

Each mark is an open three-segment trapezoid impression: two diagonal flanks and an interior base, with no score line on the red cut edge. Constants are deterministic: 0.5 mm edge and end inset, 0.4 mm dedicated decoration-to-cut minimum clearance, 1.5 mm minimum inner width, 5 mm minimum source interval, depth `min(3, max(1.5, 0.75 × thickness))`, and taper `min(1.5, max(0.25, 0.35 × thickness))`. A region that cannot meet span, containment, finite-coordinate, or clearance rules is omitted without changing the finger pattern. The small valid fixture produces 8 marks and 4 accurate omissions; zero generated marks produce a validation error rather than a silent enabled state.

Each record retains its panel, edge, pattern index, source interval, local segments, translated panel position, clearance, and deterministic ID such as `corner-decoration-front-left-01`. The serializer's existing score support is used without serializer changes. The enabled hierarchy is:

```text
score (blue)
├─ corner-decoration
│  └─ corner-decoration-<panel>-<left|right>-<NN>
└─ assembly-labels (when enabled)
cut (red)
```

The Finger Box still emits one production SVG with the established filename and MIME. Disabled output does not pass empty score properties or emit score groups, and the legacy Finger Box bytes remain unchanged. Enabled output adds only the blue `corner-decoration` subgroup before the red `cut` group. Its pinned default SVG is **7269 bytes / `d9512d1c`**. The direct fixture comparison proves ordered panel IDs, titles, dimensions, paths, order, and the entire red cut-group markup are byte-identical between enabled and disabled states. No diagonal enters a panel cut path; diagonals exist only in blue score paths. Finished View remains screen-only and does not render or read production engraving geometry.

## Validation

- `python -m html.parser index.html` passed.
- Isolated headless Edge direct `file://` startup reached `readyState: complete` with no page errors. The Finger Box UI was exercised with decoration off, enabled, labels + decoration, and the small omission case; preview/download identity, existing filename/MIME, and score-before-cut ordering passed.
- Focused Designs geometry: **994 / 0** (the prior 960 plus 34 focused assertions).
- Tray-model: **264 / 0** (unchanged).
- All 15 complete-suite groups: **1786 / 0**. Arithmetic: `20 + 12 + 66 + 58 + 118 + 23 + 18 + 67 + 57 + 56 + 61 + 12 + 8 + 216 + 994 = 1786`.
- `git diff --check` passed. No LightBurn import, physical engraving, cutting, fit, corner alignment, or strength claim was made or verified.
