# Sliding-Lid Phase 3.1 Guide Offset Correction

Date: 2026-07-15  
Baseline: `df54da0` — `Add sliding-lid box generator`

## 1. Baseline verification

Before editing:

- `HEAD`: `df54da0`
- branch: `main`, tracking `origin/main`
- tracked working files already modified: `index.html`, `README.md`
- nothing staged
- `git diff --check`: passed
- existing reported baseline: Designs `280/0`, complete suite `812/0`

Existing unrelated untracked files were left untouched. No reset, clean, stash, move, delete, stage, commit, or push operation was performed.

## 2. Confirmed defect

`slidingRailGuideSegments()` generated guide X coordinates from `0` through `dimensions.railLength` and the score paths were translated only to the side panel layout origin. Golden A has:

| Dimension | Value |
| --- | ---: |
| side-panel width / outside depth | 86 mm |
| rail length / inside depth | 80 mm |
| material thickness | 3 mm |
| missing horizontal offset | 6 mm |

The existing guide overlay therefore described a Front-flush footprint (`0..80`) even though the documented rail installation is Back-flush. A Back-flush 80 mm rail on an 86 mm side panel occupies `6..86`.

The defect was isolated to optional guide placement. Body panels, rails, lid dimensions, stop geometry, and the guides-off output were not changed.

## 3. Correction

`slidingRailGuideSegments()` now accepts the actual side-panel width and derives:

```text
railOffsetX = panelWidth - railLength
```

The corrected local guide coordinates use:

```text
xFront = railOffsetX + guideInset
xBack  = panelWidth - guideInset
guideInset = max(1, thickness / 2)
guideLeg   = min(6, max(3, thickness))
```

For Golden A, `railOffsetX = 6`, `guideInset = 1.5`, `guideLeg = 3`, and `railHeight = 11.2`.

## 4. Golden A corrected coordinates

Left-side guide endpoints are:

```text
(7.5,1.5)-(10.5,1.5)
(7.5,1.5)-(7.5,4.5)
(84.5,1.5)-(81.5,1.5)
(84.5,1.5)-(84.5,4.5)
(7.5,9.7)-(10.5,9.7)
(7.5,9.7)-(7.5,6.7)
(84.5,9.7)-(81.5,9.7)
(84.5,9.7)-(84.5,6.7)
```

The Right panel is still a complete horizontal mirror across its 86 mm local width. Independently derived Right guide endpoints are:

```text
(78.5,1.5)-(75.5,1.5)
(78.5,1.5)-(78.5,4.5)
(1.5,1.5)-(4.5,1.5)
(1.5,1.5)-(1.5,4.5)
(78.5,9.7)-(75.5,9.7)
(78.5,9.7)-(78.5,6.7)
(1.5,9.7)-(4.5,9.7)
(1.5,9.7)-(1.5,6.7)
```

The correction preserves four L marks per side, eight score segments per side, and sixteen score segments total. The marks remain inside the corrected rail footprint, away from side-panel cut edges, with the intentional Front gap equal to `panelWidth - railLength`.

## 5. Mirroring and mating behavior

- Guides off: the existing Right panel and disabled SVG remain unchanged.
- Guides on: the complete Right cut panel and Right guide are mirrored together.
- Left remains unmirrored.
- Separate rail pieces remain identical and unmirrored.
- Right layout position and document bounds remain unchanged.
- Mirroring is horizontal only; there is no vertical flip or negative coordinate.
- No model or draft mutation occurs.

Guided fixtures also check the Bottom/Right, Back/Right, and shortened Front/Right lower-field mating metadata. The underlying mating topology remains the Phase 3 model; generated mirrored output still requires physical confirmation.

## 6. Help and documentation corrections

The Designs form now places dedicated muted help beneath both optional checkboxes:

- guide help explains four concealed corner marks per side, blue score before red cuts, top-flush and Back-flush rails, and marked faces inward;
- coupon help explains the six-piece glue-up test and curing before judging fit.

The assembly message now states that shortened edges face Front, stop ends face Back, and—when guides are enabled—the visible Front gap is intentional and represents the wall allowance from `panelWidth - railLength`. README wording now documents the corrected Back-flush footprint and operation order.

## 7. Fixtures added or corrected

The existing Phase 3.1 fixtures remain in place. The previous guide-position expectation was not independently physical and was replaced with corrected Back-flush expectations. New assertions cover:

- offset and panel-width references;
- all corrected Golden A Left endpoints;
- independently mirrored Right endpoints;
- guide footprint bounds, edge clearance, Front gap, and unchanged viewBox;
- four option combinations: guides off/coupon off, guides on/coupon off, guides off/coupon on, guides on/coupon on;
- deterministic output, cut count, score-group count, preview/download identity, and first-eight layout positions for each combination;
- dedicated disabled sliding SVG signature;
- guided mating metadata spot checks;
- complete coupon numeric values, `Cs/2` lateral clearance, single `Cv` addition, `Cj` isolation, stable IDs, positions, viewBox, and insertion/capture/stop relationships;
- invalid guide-envelope rejection.

## 8. Option-combination results

All four combinations are valid and deterministic. The first eight pieces retain the existing positions.

| Combination | Cut pieces | Score groups | Document size |
| --- | ---: | ---: | --- |
| guides off / coupon off | 8 | 0 | 259 × 262.6 mm |
| guides on / coupon off | 8 | 1 | 259 × 262.6 mm |
| guides off / coupon on | 14 | 0 | 336.8 × 320.6 mm |
| guides on / coupon on | 14 | 1 | 336.8 × 320.6 mm |

The score group is emitted before the red cut group. Guides do not alter the document viewBox. Coupon geometry remains unchanged apart from the additional regression coverage.

## 9. Disabled-output and legacy signatures

The dedicated disabled Golden A sliding output remains:

- SVG length: `2800`
- fixture hash: `4a7ab718`
- no score group
- no coupon pieces
- eight existing piece positions unchanged

The other six existing signatures remain unchanged:

| Template | SVG length | Hash |
| --- | ---: | --- |
| QR stand | 359 | `fe737a09` |
| Hanging sign | 341 | `656e633d` |
| Dice tray | 1726 | `51a55721` |
| Divider tray | 1965 | `a55dda6e` |
| Finger box open | 2559 | `4e2a6f4b` |
| Finger box loose lid | 2691 | `c202cef2` |

## 10. Fixture totals

The Designs group now reports **322 passed / 0 failed**.

All groups report **854 passed / 0 failed**:

| Group | Passed | Failed |
| --- | ---: | ---: |
| Baseline resolution | 20 | 0 |
| Material Test normalization | 12 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Project Wizard | 216 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Designs | 322 | 0 |
| **Total** | **854** | **0** |

## 11. Validation

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Inline JavaScript executed successfully in isolated headless Edge.
- Normal direct `file://` startup rendered all eight application tabs with no runtime errors.
- Designs fixture function: `322/0`.
- Existing fixture groups: `854/0` total.
- Runtime error collection from the browser harness was empty.
- Preview/download identity was checked through the shared SVG result path.
- Guide score/cut ordering, guide-off absence, invalid guide rejection, dimensions, positions, and signatures were checked.

The isolated browser harness executed the same `runDesignGeometryFixtures()` and all existing fixture functions used by the `?selftest=design` and `?selftest=all` startup paths. A separate console transcript for those query URLs was not retained.

## 12. Storage, backup, and physical status

The isolated browser profile had no application storage writes from fixture generation; `backupObject()` contained no Designs draft. No external dependencies, network calls, persisted fields, JSON schema, import/export behavior, or application storage code were changed.

The corrected guide overlay has not been physically cut and test-fitted in this pass. The existing user-provided Phase 3 physical test supports the Back-flush and handed-face convention, but the corrected Phase 3.1 guide marks still require a scrap cut and LightBurn verification.

## 13. Final working-tree state

Final tracked changes are limited to:

- `index.html`
- `README.md`
- `docs/SLIDING_LID_PHASE3_1_GUIDE_OFFSET_CORRECTION_2026-07-15.md`

Existing unrelated untracked files remain untouched. Nothing was staged, committed, or pushed.
