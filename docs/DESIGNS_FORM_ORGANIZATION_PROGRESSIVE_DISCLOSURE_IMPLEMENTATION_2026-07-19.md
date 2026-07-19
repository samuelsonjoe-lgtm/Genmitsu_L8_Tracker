# Designs Form Organization and Progressive Disclosure Implementation

## Scope and baseline

- Actual HEAD: `e4bb285314f5ada3d7e0e5ee2c557bf397359915` (`e4bb285`, **Improve Designs result summaries**).
- Branch: `main`; `origin/main` was synchronized at `0 ahead / 0 behind` before editing.
- The initial tracked tree and staging area were clean. Pre-existing untracked content (`.claude/`, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, the review, and historical `docs/` reports) was inspected and preserved.
- Changed files: `index.html`, `README.md`, and this report. Nothing was staged, committed, or pushed.

This is a markup-and-presentation pass. It does not alter defaults, normalization, draft keys, validation, builders, layout, production SVGs, serializers, downloads, Finished Views, storage, schema, import/export, or backups.

## Form organization

The Template select now uses native optgroups:

- **Boxes and cabinets:** Finger-jointed box, Sliding-lid finger box, Drawer Cabinet.
- **Trays:** Dice tray, Divider tray.
- **Signs and stands:** Freestanding QR / sign stand, Hanging sign.
- **Coupons and prototypes:** Joint Fit Coupon, Concealed Cleat Corner Prototype, Concealed Cleat Full-Box Prototype.

`designTemplateSelect()` is local to this selector; the shared `designSelect()` contract was not changed. `designAdvancedSection()` emits the established native `<details class="project-section span2">` / `<summary>` / `.project-section-body.formgrid` pattern, with no new CSS or disclosure JavaScript. It is closed on each render and has no persisted state.

### Sliding-Lid Box

**Box size and lid** remains visible: dimension basis, width, depth, height, Finger pull, and conditional Pull width. **Advanced fit and production options** contains finger-joint clearance, preferred finger width, lid side/vertical clearance, Front insertion clearance, rail-placement guides, assembly labels, and the six-piece coupon. Pull width stays outside Advanced and remains dormant-but-preserved when Finger pull is `None`.

### Drawer Cabinet

**Drawer size and layout** remains visible: inside width/depth/height, rows, layout mode, uniform columns or the active custom-row selectors. **Material** remains visible: shell thickness, separate-thickness toggle, and either interior thickness or the existing linked-thickness note. **Advanced fit, joints, and markings** contains cabinet/drawer joint clearance, preferred finger width, lateral/vertical/rear clearance, shelf guides, and assembly labels. Uniform/custom-row and separate-thickness conditional behavior is unchanged.

All other template forms, including Finger Box, remain structurally compact with no Advanced disclosure.

## Behavioral protection

New DOM fixtures exercise the real `renderDesigns()` output. They verify optgroups and option uniqueness; simple-template compactness; exact Advanced field placement; closed-default disclosures; native form-control inclusion in `FormData`; wrapped labels and unique names; Sliding-Lid Pull-width preservation; Drawer Cabinet custom-row and interior-thickness preservation after visible-field edits; disclosure toggling with no draft/storage/backup change; and production bytes remaining identical after open/close presentation changes.

The existing form control names, values, types, minimums, steps, checkbox state, and select options remain unchanged. No IDs, ARIA relationships, tabindex overrides, custom `aria-expanded`, or live regions were added. Native summary focus and keyboard behavior remain available.

## Validation

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Focused form-organization/Designs fixtures: `1093 / 0` (up from `1086 / 0`).
- Tray-model fixtures: `264 / 0`.
- Complete available suite: `1885 / 0` (`792` non-Design + `1093` Designs).
- Direct `file://` Edge loaded Designs and the grouped select/disclosures without external dependencies or console errors.
- A real Edge keyboard check focused the Sliding-Lid summary: Enter opened it, Space closed it, focus remained on summary, and no navigation occurred.
- The 320 px and 1280 px Edge checks both reported no page overflow; the Advanced section remained one native disclosure and the existing responsive grid handled the inner fields.
- Existing production and Finished View golden fixtures remained green. The direct harness re-observed the J2 production golden at `4457` bytes / FNV-1a `88721533` and the Sliding-Lid golden at `2800` bytes / `4a7ab718`; no golden constants changed. Existing download, storage/backup, and machine-profile fixtures also passed.

The production-byte evidence is the established committed golden suite plus review of this markup-only diff: no builder, serializer, output, or download function changed. A separate executable temporary checkout of baseline `e4bb285` was not created during this no-worktree pass. Physical cutting is intentionally unverified and remains outside this presentation-only phase.
