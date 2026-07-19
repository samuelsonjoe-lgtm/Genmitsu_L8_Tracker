# Designs Result Summary and Warning Consistency Implementation

## Scope and baseline

- Actual HEAD: `fea4b90105a85dd60ede79e57853bd266a648c3f` (`fea4b90`, **Add SVG scale safety guidance**).
- Branch: `main`; `origin/main` synchronization was `0 ahead / 0 behind` before editing.
- Initial tracked working tree and staging area were clean. Pre-existing untracked content included `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, the authoritative review, and historical reports under `docs/`; it was preserved untouched.
- Changed files: `index.html`, `README.md`, and this report. Nothing was staged, committed, or pushed.

This pass is limited to the generated Designs result presentation and its behavioral fixtures. It does not change design geometry, production SVG generation, serializers, downloads, Finished View SVG helpers, persisted state, storage schema, import/export, or template inputs.

## Full-Box correction

The Full-Box instructions existed in an unreachable duplicate `fullCleat` branch in the metrics ternary. The actual rendered assembly ternary had no Full-Box branch, so the result fell into the generic assembly-label fallback, which incorrectly described Finger Box cut geometry.

The unreachable branch was removed and the real assembly path now has a dedicated Concealed Cleat Full-Box message. It covers layer lamination, blue-guide placement of base stacks BA/BB/BC/BD, seam stacks, the Wall A/Wall C then Wall D/Wall B sequence, square checks, clamping, curing, and the existing assembly-and-registration/not-a-strength-test boundary. The single-corner message was not changed.

## Result presentation

- Valid results now use lightweight **Result summary**, **Messages**, and conditional **Assembly** groups. The replaced result panel uses `role="status"`, `aria-live="polite"`, and keeps visible `Error:` and `Warning:` prefixes.
- `designDimensionsText()` formats comparable multi-axis values as `width × depth [× height] mm`. It is presentation-only and does not feed SVG serialization.
- Separate-thickness Drawer Cabinet output sizes are explicitly **Shell production SVG size** and **Interior production SVG size**; linked output remains **Production SVG size**.
- QR Stand and Hanging Sign now show their known flat finished size, material thickness, and physical-piece count. QR Stand also reports its known slot fit clearance. No Finished View or assembled-depth claim was added.
- Common total-piece labels use **Physical pieces** while retaining specialized pairs and wall/base-plate details.
- Each valid result includes a screen-only **Operations** metric: `Cut: N · Score: N · Labels: N`. Counts come only from structured `panels`, `scorePaths`, and `labelPaths`; multi-output cabinets sum those arrays across their production outputs. It is not a runtime, material-use, SVG-path, or vector-node estimate.
- Assembly-label status now follows the same Enabled/Disabled/emitted/omitted wording, with retained safe-label or intended-face context.
- Warnings are stably ordered for presentation only: construction/material limitations, fit/movement/retention cautions, workflow guidance, then the existing large-layout advisory. The exact-scale information remains non-blocking inside Messages, followed by Assembly when present. Builder warning arrays are not modified.

## Fixtures and validation

Six behavioral Designs fixtures were added, increasing Designs geometry from `1080 / 0` to `1086 / 0`. They exercise real builders and `designResultsHtml()` for the Full-Box message, operations/count semantics, grouping/status semantics, legacy flat summaries, Drawer Cabinet separate outputs, multiplication-sign formatting, warning ordering, and invalid-output suppression.

Completed checks:

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Direct `file://` Edge check: passed; tab, form, and result UI loaded, and storage remained unchanged.
- Focused Full-Box summary/assembly fixtures and all Designs fixtures: `1086 / 0`.
- Tray-model fixtures: `264 / 0`.
- Complete available suite: `1878 / 0` (`792` non-Design + `1086` Designs).
- Edge confirmed the known J2 production golden at `4457` bytes / FNV-1a `88721533`, including its unchanged blue, green, and red group contract; the Sliding-Lid golden remained `2800` bytes / `4a7ab718` before and after complete-suite execution. All committed Designs production-golden assertions, including the registered QR, Hanging, tray, Finger Box, Sliding-Lid, Drawer Cabinet, coupon, and both concealed-cleat variants, passed without updating golden constants.
- Existing fixture coverage continued to verify deterministic production, Finished View screen-only behavior, downloads, storage/backup isolation, and machine-profile independence. This presentation-only diff does not touch their builders, serializers, download path, or state functions.
- A direct `file://` 320 px Edge check reported `overflow: false`, with Result summary, Messages, Assembly, and Operations present, the Full-Box text correct, download enabled, and the preview selector usable.

No production golden was changed. The historical baseline was protected by its committed golden assertions plus source-diff review; a separate temporary executable checkout of `fea4b90` was not created for this no-worktree pass. Physical cutting and Full-Box dry fitting remain required to validate clamp access, squareness, fit, strength, durability, and production safety.
