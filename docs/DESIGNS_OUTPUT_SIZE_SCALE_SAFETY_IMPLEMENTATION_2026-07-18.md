# Designs Output-Size and Scale-Safety Disclosure - 2026-07-18

## Baseline and scope

Implemented from `ad4532cfa7d5c112a25da4cbd17ac5cefd164e1c` (`Add Finger Box finished view`) on synchronized `main` (`origin/main...HEAD: 0 0`). The initial tracked tree and staging area were clean; pre-existing untracked reports, LightBurn projects, utilities, and debug files were preserved.

This is a screen-only disclosure phase. It does not change production geometry, panel layout, root SVG dimensions, serialization, downloads, storage, schema, import/export, or Finished View helpers.

## Architecture

`designLayoutSizeWarning(widthMm, heightMm)` is a small pure helper. It returns one advisory only when either finite positive output axis is greater than 400 mm; exactly 400 mm does not warn. The advisory states the actual document size and asks the user to compare it with the real material sheet and machine travel. It does not infer a bed size from `machineProfile`, claim fit, block output, split sheets, or nest parts.

`buildDesignResult()` applies that helper once after every registered builder returns a valid result: QR stand, Hanging sign, Dice Tray, Divider Tray, Finger Box, Sliding-Lid Box, Drawer Cabinet, both Joint Fit Coupon modes, Concealed Cleat Corner, and Concealed Cleat Full-Box. The three prior duplicated layout warnings in Finger Box, Sliding-Lid Box, and Drawer Cabinet were removed.

`designResultsHtml()` now renders one exact `Production SVG size` metric from `result.widthMm` and `result.heightMm`, and replaces the redundant per-template Layout/Production-sheet rows. It also renders one always-present, text-based exact-scale notice for valid results: import at 100%, move or rotate complete parts with their cut outlines, guides, score lines, engravings, and labels, but do not resize or scale them; use zoom/pan instead. Large layouts may require manual arrangement or more than one material sheet.

## Verification

The expanded behavioral fixtures call the real builders and rendering path for all registered templates and both coupon modes. They cover normal and large layouts, one-axis and both-axis overflow, the 400 mm boundary, invalid results, exact SVG root dimensions, one metric/notice per result, no fit claims, production-byte neutrality after result rendering, localStorage and backup isolation, 20W/40W machine-profile independence, and downloads with both a large advisory and a Finger Box Finished View selected.

Existing pinned production goldens remain unchanged, including QR stand, Hanging sign, Dice/Divider Tray, Finger Box open/loose/faux, Sliding-Lid Box, Drawer Cabinet, both coupon modes, and both Concealed Cleat prototypes. The phase adds no SVG-disclosure markup to downloaded bytes.

- Previous fixtures: Tray-model `264 / 0`, Designs `1072 / 0`, complete suite `1864 / 0`.
- Current fixtures: Tray-model `264 / 0`, Designs `1080 / 0`, complete suite `1872 / 0`.
- `python -m html.parser index.html`, `git diff --check`, direct `file://` Edge validation, and a 320 px headless-Edge viewport check passed; the Production SVG size metric, one advisory, and one scale-safety notice wrapped without horizontal overflow.

No physical cutting was required because production bytes are unchanged. Physical LightBurn import and material cutting remain outside the automated checks. Nothing was staged, committed, or pushed.
