# Designs-to-Projects Handoff Phase 1 — Implementation

## Baseline and working tree

- Actual HEAD: `b35ed48356b3ba190951e2e58dde2647ae99f04e` (`b35ed48 — Organize Designs form controls`).
- Branch: `main`, synchronized with `origin/main` (`0` ahead / `0` behind).
- Initial tracked working tree and staging area were clean. Existing untracked reports, `.claude/`, `LightBurn Projects/`, `debug.log`, and `parametric_qr_stand_generator.py` were preserved.
- Changed by this phase: `index.html`, `README.md`, and this implementation report. Nothing was staged, committed, or pushed.

## Handoff behavior

`projectDraftFromDesign(result, draft)` is a pure, structured-data-only mapper. It returns an in-memory partial Project draft, and `startProjectFromDesign()` passes it to the established `openProject(draft, { draftNote })` seam. No second editor, save path, ID creation, Design link, storage key, or persistence state was introduced.

- Valid Designs show **Start Project from Design** beside the download action. It uses a native disabled state only when the current Design is invalid; valid warnings and valid multi-output Drawer Cabinets remain eligible.
- The modal is a point-in-time snapshot. It is only saved by the existing **Save project** submit path; Cancel creates nothing.
- If a Project modal is already open, the action leaves that form untouched and asks the user to save or close it first.

## Field mapping

- `name`: concise, editable template name with reliable finished/outside dimensions where available; cabinets retain linked grid or custom-row context.
- `material`: intentionally blank; Save continues to require user confirmation.
- `thicknessValue` / `thicknessUnit`: a reliable measured thickness in `mm`; Drawer Cabinet uses shell thickness. Separate interior/drawer stock is notes-only.
- `jobType`: `cut`; `quantity`: `1` finished Design; `status`: `kept`.
- `notes`: concise fit, loose-lid, decorative, coupon, prototype, and separate-output context only. Both concealed-cleat templates state that they are assembly/registration prototypes and not strength or load-capacity tests.
- No material lines, accounting values, settings snapshot, Inventory selection, Pricing data, source IDs, SVG text, production filename, or Design identity is prefilled.

Separate-thickness Drawer Cabinets state the interior/drawer-stock thickness and that two production SVG files are required; they still create one reviewable Project draft with blank material.

## Neutrality and validation

The production builders, serializers, `productionOutputs`, download functions, Finished View helpers, storage/schema/import/export functions, Project normalization, Project submit behavior, Inventory matching, and Pricing behavior were not modified. The dedicated fixture exercises pure mapping for all templates/modes, invalid activation, native action state, real modal open/cancel/save behavior, conflict protection, material validation, point-in-time behavior, localStorage/backup isolation before save, production and Finished View byte stability, and 20W/40W machine-profile independence.

- Designs-to-Projects handoff fixtures: `17 / 0`.
- Tray-model fixtures: `264 / 0`.
- Designs geometry: `1093 / 0`.
- Complete suite: `1902 / 0` (`809` non-Design + `1093` Designs assertions).
- `git diff --check` and `python -m html.parser index.html` pass.
- Direct `file://` Edge exercised the action and existing Project modal lifecycle. Desktop and 320 px checks confirmed the action row wraps without horizontal page overflow; the existing modal, draft note, fields, Save, and Cancel remained reachable. Keyboard activation uses the native button and existing modal focus behavior.

`git diff b35ed48 -- index.html` confirms that no builder, production-output, serializer, download, or Finished View helper changed. A detached `b35ed48` baseline worktree was also run directly in Edge: its committed complete suite was `1885 / 0` and its Designs production-golden group was `1093 / 0`. The working tree retained `1093 / 0` Designs production goldens while adding only the `17 / 0` handoff group, for `1902 / 0` overall.

No physical cutting is required; production geometry and production bytes remain outside this phase.
