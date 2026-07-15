# Sliding-Lid Phase 3.1 Implementation

Date: 2026-07-15  
Baseline: `df54da0` — `Add sliding-lid box generator`

## 1. Baseline

The tracked working tree was clean before this pass. Existing Designs fixtures were 265 passed / 0 failed and the existing full-suite baseline was 797 passed / 0 failed.

## 2. Changed files

- `index.html`
- `README.md`
- `docs/SLIDING_LID_PHASE3_1_COUPON_AND_GUIDES_IMPLEMENTATION_2026-07-15.md`

No storage, JSON schema, backup, import, export, Python, LightBurn, or unrelated untracked files were changed.

## 3. Session-only options

The sliding-lid Designs draft now has two opt-in, session-only fields:

- `slidingGuideMarks`, default `false`;
- `slidingFitCoupon`, default `false`.

Checkbox state is read as a boolean by the existing Designs draft updater. The options are not added to application state, persisted data, or JSON backups. Disabled options preserve the existing sliding-lid output.

## 4. Rail-placement guide marks

When enabled, the generator creates eight short L-shaped corner marks for each side panel. They are generated from material thickness, rail length, and rail height using an inset of `max(1, t / 2)` and a leg length clamped from 3 to 6 mm. Guide geometry is rejected if the available side-panel envelope cannot contain the marks.

Guide marks are emitted in a separate blue `score` group with `guide-rail-left` and `guide-rail-right` subgroups. They do not enter the red cut layer, panel count, or layout envelope.

## 5. Handed Right-side orientation

With guide marks enabled, the complete Right side-panel geometry is horizontally mirrored as a new panel value, preserving its cut path, points, bounds, and panel envelope without mutating the model or draft. The Right guide segments are mirrored across that same Right panel width. This implements the physical convention established by the LightBurn orientation test: both side panels present their inside faces upward while the guide marks remain attached to the correct handed panel.

The normal disabled path remains unchanged, including the existing byte-stable sliding SVG fixture behavior.

## 6. Guide geometry checks

The new fixtures cover:

- valid guide-enabled generation;
- two score paths;
- eight guide segments per side;
- guides excluded from cut panels;
- blue score-layer serialization;
- complete Right-panel mirroring;
- Right guide mirroring;
- draft immutability.

## 7. Six-piece sliding-fit coupon

When enabled, the coupon appends six stable pieces after the existing eight sliding-box pieces:

1. `coupon-base` — Coupon base / spacer
2. `coupon-wall-left` — Coupon left wall
3. `coupon-wall-right` — Coupon right wall
4. `coupon-rail-left` — Coupon left rail
5. `coupon-rail-right` — Coupon right rail
6. `coupon-lid` — Coupon sliding lid

The coupon uses a bounded starter geometry rather than cropping or reusing full-box body panels. It contains a full-width base/spacer, two plain wall strips, two C-channel rails, and a dedicated shouldered/tongued sliding lid. No coupon tabs or slots are added.

For the default 3 mm starter values (`Cs = 0.20`, `Cv = 0.20`, `U = L = S = 4`), the principal derived values are:

| Value | Result |
| --- | ---: |
| Coupon gap `Gc` | 32 mm |
| Coupon depth `Dc` | 40 mm |
| Base/spacer thickness `B` | 6 mm |
| Base width `Wb` | 44 mm |
| Coupon rail height `Rh,c` | 11.2 mm |
| Coupon wall height `Hc` | 17.2 mm |
| Coupon lid width `Lw,c` | 37.8 mm |
| Coupon tongue width `Tw,c` | 31.8 mm |
| Coupon lid length | 43 mm |

The coupon receives a fifth layout row, expands the SVG viewBox only when enabled, and remains red cut geometry. It does not add score geometry unless rail guides are also enabled.

## 8. Layout and validation behavior

The original four sliding-box rows and their positions remain unchanged when the coupon is disabled. The optional coupon row is laid out with the existing deterministic panel-row helper. Cut segment duplication, panel geometry, translated bounds, SVG parsing, finite coordinates, and viewBox containment continue to be validated.

## 9. Fixture totals

The Designs group now reports **280 passed / 0 failed**. Existing signatures remain unchanged:

| Template | SVG length | Fixture hash |
| --- | ---: | --- |
| QR stand | 359 | `fe737a09` |
| Hanging sign | 341 | `656e633d` |
| Dice tray | 1726 | `51a55721` |
| Divider tray | 1965 | `a55dda6e` |
| Finger box open | 2559 | `4e2a6f4b` |
| Finger box loose lid | 2691 | `c202cef2` |

The full manually collected fixture total is **812 passed / 0 failed**:

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
| Designs | 280 | 0 |
| **Total** | **812** | **0** |

## 10. Validation performed

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Browser parsing and execution in isolated headless Edge: passed.
- Normal direct `file:///` startup: tabs and application content rendered; no startup exception was observed.
- Designs fixture harness: 280 / 0.
- All existing fixture groups: 812 / 0 total.
- Existing disabled sliding SVG and all legacy template signatures remained byte-stable.
- Isolated browser storage remained empty and `backupObject()` did not contain a Designs draft.

The direct `?selftest=design` and `?selftest=all` console presentation was not separately screen-captured; the same fixture functions were executed through the isolated headless browser harness against the local file, including every group listed above.

## 11. Physical verification status

The user-provided physical verification establishes that the Phase 3 sliding body, lid, rails, and current fit-clearance approach assembled and imported correctly, and that Right-side guide orientation must mirror the complete side geometry. This pass implements that resolved orientation rule.

The newly added guide-mark and six-piece coupon outputs have not yet been cut and physically test-fitted. The UI continues to label the fit as experimental and recommends scrap testing before production.

## 12. Final status

Phase 3.1 is implemented as a conservative, offline, opt-in extension. Existing defaults, persisted application data, JSON schema, backups, imports, unrelated templates, and the existing Designs workflow are preserved.

No files were staged, committed, or pushed.
