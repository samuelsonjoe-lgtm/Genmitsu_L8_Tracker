# Hex Tray 120 Degree Joint Coupon — Implementation

Date: 2026-07-24  
Starting HEAD: `8f468db Add tabletop token and dice tray generator` (`main...origin/main`, synchronized)

## Scope and repository state

The initial tracked worktree and index were clean. Pre-existing untracked reports, `LightBurn Projects/`, `debug.log`, and a utility script were left untouched. The controlling document was [HEX_TRAY_120_DEGREE_JOINT_COUPON_ARCHITECTURE_REVIEW_2026-07-24.md](HEX_TRAY_120_DEGREE_JOINT_COUPON_ARCHITECTURE_REVIEW_2026-07-24.md).

Implemented one additive Designs template only:

- Name: `Hex Tray 120° Joint Coupon`
- Template ID: `hex-tray-120-joint-coupon`
- Generator version: `hex-tray-120-joint-coupon-j1`
- Status: Prototype / physical test coupon

No Project handoff, evidence flow, storage record, schema, import/export behavior, or promotion behavior was added.

## Geometry approach

`buildBoxModel` was inspected and left unchanged. Its edge helper binds finger depth to physical material thickness, so it cannot safely accept a separate compensated tab depth. The coupon instead uses a small generator-local wall-panel function built from the existing, unmodified `buildFingerPattern`, `designPatternEdge`, and `buildFingerPanel` helpers. Physical material thickness and candidate tab depth are therefore distinct values; no false thickness is passed into an existing box builder.

The fixed interior reference is 120°. Candidates share one clearance and differ only in tab-depth compensation:

| Candidate | Factor | 3 mm default depth |
| --- | ---: | ---: |
| C1 | `1.0000000000` | `3.000000 mm` |
| C2 | `1 / sqrt(3)` (`0.5773502692`) | `1.732051 mm` |
| C3 | `0.7000000000` | `2.100000 mm` |

The default form values are wall length `55 mm`, wall height `30 mm`, material thickness `3 mm`, preferred finger width `7 mm`, clearance `0.00 mm`, panel spacing `5 mm`, gauge enabled, and labels enabled. The compact form is grouped into coupon dimensions, material and fit, and layout and identification. It explains that the output tests one wall-to-wall joint only and contains no floor or full hexagonal shell.

Six cardinal-only wall panels are generated as three explicit handed pairs:

`C1-WALL-LEFT`, `C1-WALL-RIGHT`, `C2-WALL-LEFT`, `C2-WALL-RIGHT`, `C3-WALL-LEFT`, and `C3-WALL-RIGHT`.

Each pair has complementary edge phase and a distinct cut path. With the option enabled, `ANGLE-GAUGE-120` is the seventh panel; disabling it leaves only the six wall panels and leaves their paths unchanged.

## Gauge-only diagonal exception

The normal `designPointsPath` helper remains untouched and no global diagonal-edge capability was added. The only production panel with diagonal path segments is `ANGLE-GAUGE-120`. Its coupon-local path uses `M`, `L`, and `Z` to create two long 120° reference edges and a relieved, blunt apex; all finger-jointed wall panels continue to use cardinal `H`/`V` paths.

The generator validates finite, closed geometry, non-fragile segment lengths, and a calculated `120°` reference angle. When labels are enabled, the gauge receives a small, contained green `120` label. The red cut layer, green label layer, and existing output conventions are retained.

## Output and screen-only guide

The production SVG is one exact-scale millimeter layout with adjacent LEFT/RIGHT pairs. It contains no floor, rails, false bottom, liner, glue guide, full hexagon, or network dependency.

The `Finished / Assembly Guide` preview is screen-only. It draws all three plan-view candidate relationships, names each candidate and calculated depth, identifies the intended 120° relationship, and states that actual angle retention and closure require physical testing. It has an image role and useful accessible label, returns an empty string for invalid or mismatched results, and is absent from `result.svg`.

## Production regression signatures

New template signatures are pinned only for this template:

| Scenario | Bytes | FNV-1a |
| --- | ---: | --- |
| Default 3 mm, gauge and labels enabled | 5,985 | `3543f233` |
| Gauge disabled | 5,534 | `481feafb` |
| Labels disabled | 1,432 | `88e2ec43` |
| 4 mm stock / 8 mm preferred finger width | 5,859 | `330aa05c` |
| `+0.05 mm` clearance | 6,081 | `f1f8d5d5` |

Focused regression checks reconfirmed unchanged existing signatures: T1 corner coupon `2992 / 4f543f95`, T2 shell `2337 / ed5d6f6e`, Tabletop Token & Dice Tray `3036 / a86aea09`, and Dice Tray `1726 / 51a55721`.

## Validation

- `git diff --check`: passed.
- `python -m html.parser index.html`: passed.
- Inline JavaScript/runtime parsing: passed through direct `file://` Edge loading with no coupon runtime errors.
- Hex Tray 120° Joint Coupon fixtures: `32 passed / 0 failed`.
- Designs geometry fixtures: `1095 passed / 0 failed`.
- T1 corner coupon: `76 / 0`; T2 shell: `93 / 0`; T3A storage fit: `23 / 0`; T3B storage tray: `37 / 0`; Tabletop Token & Dice Tray: `35 / 0`.
- Normal direct `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` completion: `3240 passed / 0 failed` across 51 registered groups. The dispatcher itself reports completion with zero failures; an in-memory instrumentation of that same dispatcher was used only to expose its aggregate passed count. The final normal and instrumented runs had no page errors.
- Direct-file Edge checks at 1440, 1024, 768, and 480 px confirmed the template registration, default valid result, seven-piece default output, guide preview, gauge/label toggles, invalid-value blocking, no outbound network requests, no horizontal overflow, and successful rendering of the existing Finger Box template.

The focused fixture also covers default/legacy normalization, real-form field reading and booleans, factor math at 2/3/4 mm, shared clearance, handed pair roles, closed finite outlines, overlap avoidance, gauge-only diagonal isolation, optional gauge behavior, exact 120° geometry, screen-only isolation, invalid boundaries, protected generator bytes, persistence isolation, and storage/security constants.

## Compatibility and remaining physical validation

`STORAGE_KEY`, `SCHEMA_VERSION`, backup formats, vault keys, import/export behavior, record schemas, existing builders, existing Finished View builders, existing generator IDs, and existing production goldens remain unchanged. Nothing in this implementation establishes a mechanically correct 120° joint or selects a best candidate.

Physical testing remains pending: measure the intended scrap, cut C1/C2/C3 and the gauge, dry-fit each pair without glue, compare against the gauge, inspect angle retention, interior/exterior gaps, force, damage, twist, and reversibility, then select any follow-up depth/clearance investigation solely from observations. Full tray and floor-corner work remain out of scope until that evidence exists.

## Final state

Changed files are limited to `index.html` and this report. Nothing was staged, committed, or pushed.
