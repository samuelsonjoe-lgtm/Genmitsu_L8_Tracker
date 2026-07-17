# Dice Tray Bottom Cover Concealment Correction

**Date:** 2026-07-17
**Actual HEAD:** `c064896` - Add drawer cabinet shelf-placement guides

## State and finding

Before correction, the only tracked modifications were the uncommitted `index.html` and `README.md` Dice-cover implementation; no files were staged. Pre-existing untracked reports and project files were preserved.

The read-only focused audit found that the original fixed 3 mm inset was not safe: its lateral 8 mm tab-spacing analysis did not test the radial wall-slot band. At 3 mm material, each slot's outer edge is 1.5 mm from the structural-base perimeter, so a 3 mm inset left about 1.5 mm exposed.

## Selected correction

The cover now uses the smallest simple rule that contains every current wall slot: a full-size plate with 0 mm inset. It directly reuses the structural-base outside dimensions; no new geometry system, user setting, alignment marks, nesting, serializer, or Divider Tray support was added.

| Property | Rejected inset plate | Final plate |
| --- | --- | --- |
| Inset per side | 3 mm | 0 mm |
| Default size | 220 x 160 mm | 226 x 166 mm |
| Default layout height | 482 mm | 488 mm |
| Enabled golden | `1778 / ec9b12bc` | `1778 / 8e2ea3f4` |

Default derivation: `220 + 2(3) = 226` mm and `160 + 2(3) = 166` mm; with the existing disabled height 307 mm and existing 15 mm gap, enabled height is `307 + 15 + 166 = 488` mm. The cover remains at `x = 10`, `y = 307 - 10 + 15 = 312`.

## Verification and boundaries

- Base-local containment fixtures verify every front, back, left, and right wall-slot rectangle lies inside the full cover footprint for default Finger and two-tab profiles, near-minimum Finger/two-tab dimensions, and a valid 0.1 mm material case. The default front slot spans `y = 1.5..4.65` mm inside the full `[0, 166]` footprint.
- Existing components remain byte-identical and ordered; the cover is the final component, final material ID, and final anonymous red SVG rectangle. The tray serializer was not changed.
- Disabled Dice and Divider production bytes remain `1726 / 51a55721` and `1965 / a55dda6e`. Preview/download identity, filename, MIME type, localStorage, backup isolation, and Finished View independence remain covered.
- Parser validation and isolated headless Edge `file://` execution passed. Tray-model fixtures are `264 / 0`, Designs geometry is `922 / 0`, and the complete suite is `1714 / 0` (`248 + 16`, `906 + 16`, and `1698 + 16`).
- Only `index.html` and `README.md` are modified tracked files. The implementation report was updated; this correction report was added untracked. The design review and focused audit remain unchanged. Nothing was staged, committed, or pushed.

The full-size plate is still cosmetic. Correct concealment depends on manual edge alignment and a real tray with fully seated flush tabs, clean glue surfaces, and a flat clamp-up; no physical build or production import was performed.
