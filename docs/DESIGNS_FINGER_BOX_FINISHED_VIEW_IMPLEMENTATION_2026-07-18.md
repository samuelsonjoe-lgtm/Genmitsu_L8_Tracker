# Finger Box Finished View Implementation - 2026-07-18

## Scope

Implemented the bounded Finger Box Finished View upgrade from baseline `0ce2f5bb8f6dad66ee0fd013514a5f993598a26c` (`Add concealed cleat full-box prototype`). The branch was synchronized with `origin/main` before work; the initial tracked tree was clean, with pre-existing unrelated untracked files preserved.

## Implementation

`buildFingerBoxFinishedViewSvg(result)` now produces a responsive, screen-only isometric SVG from semantic result data only: outside width, depth, and height; material thickness; width/depth/vertical finger-pattern counts; lid mode; closed height; and optional faux-dovetail decoration. It adds an accessible SVG title, description, `role="img"`, and label; a visible screen-only notice; an open cavity; simplified visible count-based finger markers; and a separate, plain outside-size loose-lid panel without retention semantics.

The faux-dovetail variant adds only a visibly labelled decorative cue. It neither describes nor creates a structural dovetail. The preview does not parse `result.svg`, alter the production result, or affect cut-layout downloads.

## Protected boundaries

The production box model, production builder, serializer, cut geometry, output bytes, storage/schema, import/export, and non-Finger templates were not changed. The preview remains session-only and does not mutate drafts, localStorage, or backup serialization.

## Fixtures and validation

Focused Designs assertions now cover semantic envelope values, responsive accessibility markup, open and loose-lid states, count-based markers on visible structural edges, decorative-only faux cues, semantic-only generation with an empty production SVG, deterministic construction, production-byte neutrality, and localStorage/backup isolation.

- Finger Box production goldens remain unchanged: open `2483 / a892f91c`, loose `2615 / 6181bc75`, faux-dovetail `7269 / d9512d1c`.
- Direct `file://` Edge validation passed.
- Tray-model fixtures: `264 / 0`.
- Designs geometry fixtures: `1072 / 0`.
- Complete available suite: `1864 / 0`.
- `python -m html.parser index.html` and `git diff --check` passed.

No files were staged, committed, or pushed. Remaining validation is physical: dry-fit and use the real material/cutting settings before treating any previewed box as fit, strong, kerf-correct, or production-safe.
