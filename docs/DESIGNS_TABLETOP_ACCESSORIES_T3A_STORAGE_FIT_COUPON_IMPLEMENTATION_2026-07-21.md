# Tabletop Accessories T3A - Storage Fit Coupon

## Scope

T3A adds a dedicated, session-only `tabletop-storage-fit-coupon` for retrofitting one existing physical T2 closed-three-way-corner shell. It is not a shell generator, tray product, or persistence/evidence expansion.

## Output and fit model

The production SVG contains exactly five red-cut pieces, with optional existing green assembly labels:

- `RAIL-FRONT`
- `RAIL-BACK`
- `RAIL-LEFT`
- `RAIL-RIGHT`
- `FALSE-BOTTOM`

The rails sit on the existing shell floor and glue against the interior walls. The false bottom is plain wood and rests on the rails. It contains no cork, notch, hole, groove, pocket, or permanent pull feature; the assembly instructions specify temporary masking-tape pull tabs for dry fitting.

The coupon regenerates the matching T2 `buildBoxModel(... floorCornerOwnership: true)` pattern. Front/back rail runs are the requested interior width minus two actual width-axis floor finger widths; left/right runs are requested interior depth minus two actual depth-axis floor finger widths. The false-bottom dimensions are `W - C` by `D - C`, with `C / 2` clearance on each opposing side. Rail height is blocked below `max(3 * measured shell thickness, 10 mm)` and the remaining visible wall height and false-bottom overlap must remain positive and manufacturable.

## Presentation and evidence boundary

The Finished View shows the existing shell only as screen-only context, plus the four rails and false bottom. It is absent from the downloaded SVG. Start Project from Design records the differentiating retrofit facts in editable notes without changing the Project schema.

Existing T2 shell evidence is described as outer-shell scope only. It does not verify the T3A storage mechanism. A dedicated raw-result/promotion workflow is intentionally deferred because it would require a new persisted evidence-record schema; no matcher, storage key, schema, import/export behavior, or evidence semantics changed.

## Verification

- Focused `tabletop-storage-fit-coupon`: **23 / 0**.
- All **42** registered fixture groups: **2912 / 0** in a disposable direct `file://` headless Edge probe.
- HTML parser and inline browser load passed after a corrected initial syntax issue.
- T3A coverage verifies the exact five-piece contract, actual-pattern rail insets, dimensions/clearance split, height/overlap/visible-wall blocks, deterministic red SVG, green labels, contextual-only Finished View, safe Project notes, and storage/backup isolation.

Protected production outputs remain covered by their existing fixtures: legacy T2/T1 shell and coupon pins, Dice/Divider Tray, Joint Fit Coupon, J2, and non-J2 goldens were not changed by T3A. The new default T3A production SVG is 897 bytes / FNV-1a `b5c549ee`; it is a new output, not a replacement golden.

## Remaining physical validation

Dry-fit and validate rail seating, generated end setbacks, glue/clamp behavior, false-bottom insertion/removal, racking, usable capacity, and the actual material before relying on the mechanism. T3A does not establish fit, strength, durability, evidence promotion, or production safety.
