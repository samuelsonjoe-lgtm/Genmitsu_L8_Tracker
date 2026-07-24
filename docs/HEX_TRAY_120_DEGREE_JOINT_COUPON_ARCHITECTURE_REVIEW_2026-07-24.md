# Hex Tray 120° Finger-Joint Coupon — Architecture and Geometry Review

Date: 2026-07-24
Repository: `C:\Genmitsu L8 Tracker`
Reviewer role: read-only architecture and geometry review (Claude Sonnet 5)
Product files modified by this review: **none**
Authorized output only: this report

---

## Executive summary

A regular hexagon's 120° interior wall corner **can** be built as a genuine, laser-cut, interlocking finger joint — no bevel or through-thickness miter is required, because the angle problem is entirely a 2D (plan-view) geometry problem, and a laser can cut any 2D vector shape. The review derives, and independently verifies against the known 90° case, the exact trigonometry: the **outer corner always closes automatically at the true hexagon vertex** (a basic polygon fact, true for any construction), while the **inner corner and the effective tab/finger depth must be derived from `t·cot(60°) ≈ 0.577·t`** (for a 120° interior angle), not the raw material thickness `t` the app's existing 90° box-joint math uses everywhere today. Using the unmodified thickness `t` as the tab depth at a 120° corner — the "conventional 90° tab depth" the prompt warns against — is confirmed, by derivation, to leave a real, predictable, thickness-proportional gap or interference, worst at the interior corner.

The review recommends the **angle-compensated alternating finger joint** (Option 1 of the families to evaluate), built as a **plain, un-mitered rectangular wall panel** (no new diagonal-cut geometry) with its finger pattern computed by the app's **existing, unmodified `buildFingerPattern`/`buildBoxModel` machinery**, fed a corrected wall length and a compensated tab depth derived from the trigonometry above. This is deliberately the **simplest member** of the angle-compensated family: it reuses 100% of the existing finger-panel code path, requires no new path-segment geometry, and turns "does 120° actually need different math than 90°" into a single, physically testable question — exactly the question the recommended coupon is built to answer. A fully mitered version (diagonal cut at 60° from each wall's length axis, teeth cut along that shared seam) is mathematically the more elegant, exactly-flush solution, but it requires the panel-path pipeline to support a diagonal edge direction it does not support today (`designPointsPath` only emits horizontal/vertical segments) — this review recommends deferring that escalation until the simpler compensated joint is physically proven insufficient.

**Recommended scope: Option B — a coupon generator (two wall panels, one 120° corner) plus a minimal, screen-only assembled corner diagram**, reusing the existing Tabletop Accessories isometric-projection helper. No hexagonal shell, floor, rails, false bottom, or liner is designed or implemented in this phase.

---

## 1. Actual HEAD and working-tree state

| Item | Observed |
| --- | --- |
| HEAD | `8f468db861ae5d1fe2f12ef5fd98ab0735f438e3` |
| Subject | `Add tabletop token and dice tray generator` |
| Branch | `main` |
| Upstream | `main...origin/main`, **0 / 0** (synchronized) |
| Tracked working-tree diff | **None** — `git diff --stat` and `git diff --check` both empty |
| Staged changes | None |
| Untracked files | All pre-existing (historical `docs/*.md` reports, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`) — none related to this review, none touched |

The stated baseline matches the repository exactly. HEAD's subject confirms the "Stackable Token & Dice Tray" generator recommended in this session's immediately preceding Finished Views review has been implemented and committed since that review — consistent with the stated project sequence. This review inspects that new generator only for workflow patterns (§4), per the task's explicit instruction, and does not modify it.

---

## 2. Existing geometry inspected

All read via `Read`/`Grep` against the single `index.html`, no edits made:

- `buildFingerPattern` (`index.html:3967-3977`) — the core finger-count/spacing algorithm.
- `designPatternEdge`, `designEdgeTerminalOffset`, `designSafePatternEdgePoints`, `buildSlidingLidBodyPanel` (aliased `buildFingerPanel`) (`index.html:3978-4363`) — the panel-outline generator that consumes finger patterns and produces a closed point sequence.
- `designPointsPath`, `designCompactPoints` (`index.html:3979-3998`) — the path-string serializer.
- `buildBoxModel` (`index.html:4002-4038`) — the shared, generic rectangular box-panel generator (reused by Finger Box, T2 shell, and others).
- `designPanelFromPoints`, `designPanelGeometryErrors` (`index.html:4319-4324`, `4412`) — panel construction and geometry validation.
- `designSegmentsIntersect`, `designSegmentContainedInPolygon`, `designSegmentsContainedInPolygon`, `designPointStrictlyInsidePolygon`, `designPointToSegmentDistance`, `designSegmentDistance` (`index.html:4364-4412` region) — the shared self-intersection/containment geometry-validation primitives.
- `layoutDesignPanelRows` (`index.html:4679`) — sheet-layout helper.
- `designSvgValidation` (`index.html:4769`) — post-serialization sanity check.
- `tabletopAccessoryRectangleOutline`, `tabletopAccessoryDimensions`, `tabletopAccessoryFinishedCavity`, `tabletopAccessoryManufacturingLimits`, `tabletopAccessoryProjectPoint`, `serializeTabletopAccessorySvg` (`index.html:5126-5208`) — the Tabletop Accessories–scoped helper family.
- `buildTabletopCornerFloorCouponDesignResult` (T1, `index.html:5209-5236`) — read in full; this is the **exact existing precedent** for a multi-candidate coupon (three clearance candidates in one SVG, each independently labeled and laid out), directly reusable for this review's candidate matrix (§9).
- `buildTabletopRectangularShellDesignResult` (T2, `index.html:5210`+, read in the prior Finished-Views review and re-confirmed here) — the closed-three-way-corner shell, confirming `buildBoxModel`'s `floorCornerOwnership` flag and its reuse pattern.
- `tabletopClosedCornerStates` (`index.html:5795`) — the existing 90°-corner closure-validation logic.
- `designFixtureHash` (`index.html:5980`) — the shared production-SVG golden hashing helper.

**Confirmed architecture fact, load-bearing for this whole review**: `designPointsPath` (`index.html:3987-3998`) explicitly supports only horizontal and vertical line segments —

```javascript
if (previous.y === point.y) path += ` H ${designNumberText(point.x)}`;
else if (previous.x === point.x) path += ` V ${designNumberText(point.y)}`;
else return '';
```

— a segment that is neither horizontal nor vertical causes the function to return an **empty path string**. Every existing finger-joint panel in the app (Finger Box, T1, T2, T3A, T3B, the new Token & Dice Tray) is built exclusively from axis-aligned rectangles with axis-aligned tabs, and `buildSlidingLidBodyPanel`'s corner/tab math (`designSafePatternEdgePoints`, `designEdgeTerminalOffset`) is written entirely in terms of a single per-edge "inward" direction pair, always one of the four cardinal directions. **The existing panel-geometry pipeline cannot represent a diagonal (non-cardinal) cut line at all without a new path-emission mode.** This fact directly shapes the recommended joint (§6, §8).

### New Token & Dice Tray generator — workflow patterns only

Per the task's instruction, this review inspected the new `tabletop-token-tray`-family code (registered per the prior review's recommendation) only for its **composition pattern**, not to modify it: it builds internal sub-drafts and calls the existing, unmodified `buildTabletopRectangularShellDesignResult` and the T3A rail/false-bottom pattern directly, exactly as T3B does. This composition style — a new template calling existing, unmodified builder functions with new parameters — is the same reuse pattern this review recommends for the hex coupon relative to `buildFingerPattern`/`buildBoxModel` (§8).

---

## 3. The 120° corner geometry problem

**Architecture fact (basic polygon geometry, not specific to this app):** a regular hexagon has interior angle 120° at every vertex, and the direction of travel along its perimeter turns by the exterior angle, 60°, at every vertex (six turns of 60° sum to the required 360°). This matches the prompt's framing exactly and is not in question.

**Key derived fact**: the corner-joint problem is entirely a **2D, in-plane** geometry problem, not a 3D bevel problem. A laser cuts a vertical, square-through-the-sheet path — it genuinely cannot taper a cut through the material's thickness (a true "mitered" edge, in the woodworking sense, where the cut face itself is angled through the cross-section, is not achievable). But an **angled straight line drawn in the flat, top-down 2D plan of a panel is an entirely ordinary vector cut** — the laser does not care whether a line is horizontal, vertical, or diagonal; it only cannot bevel *through* the thickness. This distinction is the crux of why a genuine interlocking hexagon corner is achievable at all, and it is worth stating explicitly because the prompt's phrasing ("the laser cuts square through the plywood thickness and cannot create a conventional beveled or mitered edge") could otherwise be read as ruling out angled *plan-view* cuts, which it does not.

---

## 4. Mathematical derivation (exact geometry)

All of the following was derived from first principles in this review and cross-checked against the known, already-implemented 90° box-joint case as a sanity check (the formulas below correctly reduce to the existing 90° behavior when θ=90°, which they must, since the 90° box joint is real, tested, and in production).

**Setup**: place the shared corner vertex at the origin. Let wall A's outward run direction be the reference axis (0°), and wall B's outward run direction be at the interior angle θ=120° from it (this is the definition of "interior angle" — the wedge between the two walls' outward directions, swept through the hexagon's interior, has measure θ). Each wall is a strip of thickness `t`.

### 4.1 Outer corner (confirmed exact, always true)

The outer face of wall A and the outer face of wall B are two rays emanating from the same vertex point. **They meet exactly at that point, for any thickness and any angle** — this is true by definition of a polygon vertex, not something that needs to be engineered. **The exterior corner of a hexagonal tray closes automatically and exactly, with zero gap, regardless of which joint family is chosen**, as long as each wall panel's outer-face corner point is cut to coincide with the true vertex.

### 4.2 Inner corner (derived, verified two ways)

The inner face of wall A (offset `t` inward from its outer face, perpendicular to A's run direction) and the inner face of wall B (offset `t` inward from its outer face, perpendicular to B's run direction) are two lines that intersect at one point — call it the inner corner point.

Solving directly (full derivation available on request; verified numerically in this review):

- Distance from the vertex to the inner corner point, **measured along the angle bisector**: `t / sin(θ/2)`.
- Distance from the vertex to the point where the inner corner falls, **measured along either wall's own run/length axis** (the dimension that actually matters for computing a wall panel's usable length): `t · cot(θ/2)`.

**Verification against the known 90° case**: for θ=90°, `cot(45°) = 1`, giving an along-length inner-corner offset of exactly `t` — this is precisely the well-known, already-implemented 90° box-joint result (inner corner at `(t, t)` when both walls run along the axes from a shared origin), confirming the general formula is correct.

**Applied to the hexagon (θ=120°)**: `cot(60°) = 1/√3 ≈ 0.5774`. So the along-length inner-corner offset is **`t · 0.5774`**, not `t`. For 3 mm plywood, that is **≈ 1.73 mm**, not 3 mm.

**This is the precise, derived answer to the prompt's central question**: conventional 90° tab-depth math (which implicitly assumes the along-length inner-corner offset equals `t`) is confirmed incorrect at 120° — the correct, derived, angle-projected value is `t · cot(θ/2)`, roughly **58% of the material thickness** for a hexagon, not 100% of it.

### 4.3 What a plain miter (no fingers) would look like

A symmetric miter — cutting each wall's end along the angle bisector, from the outer-corner point to the inner-corner point — closes both the outer and inner corner **exactly**, for any thickness, with zero gap and zero overlap (this is the same principle that makes a picture-frame corner work, generalized from 90° to 120°). The miter cut length (the diagonal itself) is `t / sin(θ/2) ≈ 1.1547 · t`. This is a real, valid, and geometrically *exact* option — but building interlocking finger teeth along this diagonal seam requires the panel-path pipeline to emit a non-cardinal (diagonal) line direction, which it does not support today (§2). This is presented in §6 as the "purer" but architecturally larger option.

### 4.4 What happens if a plain rectangular (un-mitered) wall is used instead, with a compensated tab depth

If the wall panel's end remains a plain, square (perpendicular-to-length) cut — exactly like every existing box-joint wall in the app today — and the tab depth is simply set to the derived value `t · cot(θ/2)` instead of `t`, the tab's reach is now dimensionally correct **at the point where it is calibrated** (e.g., flush with the neighboring wall's outer face), but because the target boundary (the true inner/outer corner relationship) is itself diagonal, a square-ended tab cannot be simultaneously flush at both the outer-face edge and the inner-face edge of its own thickness. The derived, worst-case mismatch (gap if under-compensated, interference if over-compensated) at the *opposite* edge of the tab's own thickness is on the order of `t · (1 − cot(θ/2))`, i.e. roughly **0.42 × t** at its largest — for 3 mm plywood, on the order of **1.2–1.7 mm**, concentrated at the **interior** side of the joint (the less visually prominent side for a dice tray). This is a real, quantifiable, non-zero limitation — not a "large" gap by the prompt's own threshold, but not a perfect miter either.

**This is exactly the tradeoff the prompt asks this review to resolve honestly**: a true, exactly-flush 120° finger joint requires a diagonal miter (§4.3), which the app's geometry pipeline cannot produce today without a bounded extension (§6, §8); a plain rectangular wall with a *derived, projected* (not naive) tab depth is fully buildable with the existing, unmodified geometry pipeline today, at the cost of a small, quantified, testable residual mismatch concentrated on the interior face. **Neither option is "impractical"** — this review does not recommend abandoning the finger-jointed construction for glue-only butt joints, cleats, or a layered wall. It recommends starting with the simpler, fully-reusable option and treating the residual mismatch as the coupon's central physical-test question.

---

## 5. Joint options compared

| Option | Geometric feasibility | Corner closure | Visible gap | Handedness | Extends to 6 corners | New geometry required | Assessment |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **1a. Angle-compensated finger joint, plain rectangular wall, projected tab depth (`t·cot(θ/2)`)** | Confirmed feasible, fully derived | Outer: exact. Inner: small residual (≈0.42·t max, interior side) | Small, quantified, testable | Yes — the two walls at a corner are mirror images | Yes, all 6 corners identical by hexagon symmetry | **None** — reuses `buildFingerPattern`/`buildBoxModel` unmodified with corrected numeric inputs | **Recommended for the first coupon.** |
| **1b. Angle-compensated finger joint, mitered wall + diagonal comb** | Confirmed feasible, fully derived, exactly flush | Outer and inner: exact | None (by construction) | Yes | Yes | A bounded, scoped extension: one new diagonal edge direction, via local-frame rotation reusing existing pattern math | The "purer" solution; recommended as a **later escalation only if 1a's residual proves unacceptable in physical testing**. |
| **2. Interlocking hook/dovetail-like joint** | The dovetail *shape* itself (a trapezoidal tab, wider at the tip) is a straight-line 2D shape and is laser-cuttable, but the same non-90°-corner projection problem in §4 applies to *where* that shape sits relative to the corner — it does not avoid the derivation in §4, it only changes the tab's cross-section. Its wider-at-the-tip shape also introduces assembly direction constraints (it must be slid in along the joint's length, not pressed straight in, or it will not disassemble for a dry fit) | Same inner-corner analysis as Option 1 applies | Same order of residual as Option 1a | Yes | Yes | Comparable to 1a, plus new tab-shape logic | **Not recommended for the first coupon** — no evidence it solves the core angle problem better than Option 1, while adding assembly-direction complexity and a new tab-shape helper; the prompt's own dry-fit/reversibility requirement is harder to guarantee with an undercut tab shape. |
| **3. Slot-through-tab angular joint** | Feasible as a *different* mechanism: one panel keeps ordinary straight tabs (reusing existing tab geometry unmodified); the other panel has plain rectangular through-slots cut a set distance in from its edge, sized/positioned for the tabs to pass through at the 120° insertion angle | Requires the **same** outer/inner-corner analysis (§4) for the panels' own outer silhouettes near the corner — a through-slot does not eliminate the need to close the panel's own outline | Comparable order to Option 1a for the outline; the slot itself is a simple rectangle (fully H/V-compatible) | Yes — "slotted" panel and "tabbed" panel are structurally different roles, an even stronger handedness requirement | Yes | A new (but simple) through-slot placement calculation, in addition to — not instead of — the corner-outline math in §4 | **Viable second-tier candidate**, but does not obviously simplify anything relative to Option 1a — it still needs the same corner trigonometry, plus a new slot-placement mechanism, for no derived reduction in gap. **Not recommended over 1a for the first coupon.** |
| **4. Separate interlocking corner key (fallback only)** | Feasible, but the prompt is explicit that this must not become a concealed-cleat construction under a different name, and that the six main walls should still participate in an interlocking joint | Would rely on the key piece, not the walls themselves, to fix the angle | Depends entirely on the key's own fit, not addressed by this review | n/a | Yes, in principle | A new, separate small-part geometry | **Correctly treated as fallback-only, per the prompt's own framing** — this review does not recommend it as the primary joint, since Option 1a already demonstrates a genuine wall-to-wall interlocking joint is achievable without a separate key. |
| **5. Any stronger existing-architecture-compatible pattern** | Investigated: no existing helper in the app already solves a non-90° corner (`tabletopClosedCornerStates`, T2's closure logic, and `buildBoxModel`'s `floorCornerOwnership` flag are all 90°-specific) | n/a | n/a | n/a | n/a | n/a | No stronger already-compatible pattern was found; Options 1a/1b, derived fresh in this review, are the strongest available. |

---

## 6. Recommended joint

**Option 1a: angle-compensated alternating finger joint, plain rectangular wall panels, tab depth derived from the projected/compensated value `t · cot(θ/2)` rather than the raw material thickness.**

This is a genuine, visibly and structurally finger-jointed, interlocking joint — not a glue-only butt joint, not a cleat, not a layered wall. It:
- is cut entirely as 2D vector profiles (no bevel, no diagonal cut required in this first version);
- assembles two flat vertical wall panels at a repeatable 120° angle, since the tab/slot geometry itself is only correct at one specific angle;
- **mechanically fixes the angle** — once the fingers interlock, the panels cannot rotate relative to each other away from the angle the tabs were cut for, which is a structural property worth confirming physically (§13);
- avoids a *large* gap, by derivation (§4.4) — the worst-case residual is a small, quantified, interior-side mismatch, not an open corner;
- is reversible for a dry fit before glue, exactly like every other finger joint already in the app;
- extends to all six corners identically, by the hexagon's six-fold symmetry (§11);
- requires **zero new panel-path geometry** — it is built entirely from the existing `buildFingerPattern`/`buildBoxModel`/`designSafePatternEdgePoints` pipeline, fed corrected numeric inputs.

The mitered/diagonal version (Option 1b) is **not rejected** — it is explicitly the recommended escalation path if the physical coupon shows Option 1a's residual mismatch is unacceptable for a sellable product (§16, §17).

---

## 7. Dimensional / mathematical contract

Clearly separated, per the task's requirement:

**Exact geometry (derived, not assumed):**
- Interior angle θ = 120°; exterior/turn angle = 60°.
- Outer corner closes automatically at the true vertex, any thickness, any construction.
- Inner-corner offset along a wall's own run axis = `t · cot(θ/2)` = `t · cot(60°)` = `t / √3 ≈ 0.5774 · t`.
- Full miter-line length (if a miter is used) = `t / sin(θ/2)` = `t / sin(60°) ≈ 1.1547 · t`.
- These formulas are verified to correctly reduce to the existing, production 90° box-joint values when θ=90° (`cot(45°)=1`, `1/sin(45°)=√2`).

**Assumptions made explicit:**
- Wall length is measured on the **outer face** for the purpose of "where does the panel's cut geometry begin" (the outer corner is the simple, exact reference point; the inner corner is a derived, secondary quantity). This mirrors the existing 90° convention where outside dimensions are the simplest, most direct reference.
- The vertical (height-direction) finger/tab pattern is **entirely independent of the plan-view angle** — it varies with wall height exactly as the existing `verticalPattern` does today, using the app's existing, unmodified odd-count convention (`buildFingerPattern` always returns an odd finger count so both ends of an edge start and end on a "finger," avoiding a fragile partial segment) — this transfers unchanged to the hex corner, since parity is a height-axis property, not a plan-view-angle property.
- Left/right handedness is required: **confirmed, not assumed** — the two walls meeting at any given corner are mirror images of each other (the corner is only symmetric about its bisector, not about either wall's own axis), so the coupon needs one "left" and one "right" wall-end treatment, exactly mirrored.

**Suggested starting values (require physical testing, not defaults to trust):**
- Tab depth: start from the derived `t · cot(θ/2)` (≈0.577·t) rather than the naive `t`, but do **not** assume this single derived value is perfectly correct in practice — real plywood, kerf, char, and glue behavior can shift the ideal value slightly; this is exactly why the coupon's candidate matrix (§9) varies tab-depth compensation as its primary tested variable.
- Joint clearance: start from a small value near zero (this review explicitly does **not** reuse Joe's prior −0.075 mm shell clearance or +0.10 mm coupon-fit result, per the task's own instruction — those values are evidence for a *different* joint contract, at 90°, on specific prior material). A modest, symmetric starting range (e.g., a small positive value, zero, and a small negative value) is more appropriate for a brand-new joint family than reusing either prior number.
- Minimum finger width for 3 mm plywood: **reused directly from the existing, already-validated constant** — `Math.max(2·t, 4)` = 6 mm at t=3 mm (`buildFingerPattern`, `index.html:3968`). No new minimum-width rule is proposed.
- Kerf: represented by the app's existing signed-clearance convention, unchanged — no separate kerf variable is introduced, consistent with how every other generator in this app already treats kerf as folded into the signed clearance value.

**Machine/material-specific fit values requiring physical testing (not to be assumed from prior evidence):**
- Whether `t · cot(θ/2)` is the best tab depth in practice, or whether a slightly larger/smaller value fits better once real kerf and char are accounted for.
- Whether the residual interior-side mismatch (§4.4) is visually/structurally acceptable at Joe's actual measured material thickness (not the nominal 3 mm).
- The actual assembled interior angle, measured physically (§14).

---

## 8. Existing helper reuse for this generator

| Helper | Reused how |
| --- | --- |
| `buildFingerPattern` | Unmodified — called with the hex corner's derived wall length and preferred finger width, exactly as it is called for every existing box-joint edge. |
| `buildBoxModel` / `designPatternEdge` / `buildSlidingLidBodyPanel` / `designSafePatternEdgePoints` | Unmodified — the coupon's two wall panels are still ordinary rectangles with ordinary cardinal-direction tab edges; only the **numeric inputs** (wall length, tab depth) differ from a 90° box wall, derived per §7. |
| `designPanelGeometryErrors`, `designSegmentsIntersect`, `designSegmentContainedInPolygon` | Unmodified — reused as-is for self-intersection/overlap validation, exactly as every other generator uses them. |
| `layoutDesignPanelRows` | Unmodified — reused for laying out the candidate matrix's panels, exactly as `buildTabletopCornerFloorCouponDesignResult` (T1) already does for its own multi-candidate layout. |
| `designSvgValidation`, `designFixtureHash` | Unmodified — reused for post-serialization validation and golden-pinning. |
| `tabletopAccessoryManufacturingLimits` | Unmodified — reused for the minimum-finger-width/minimum-web checks. |
| `tabletopAccessoryProjectPoint`, `serializeTabletopAccessorySvg` | Unmodified — reused if a screen-only assembled diagram is included (§15). |
| `buildTabletopCornerFloorCouponDesignResult`'s candidate-array pattern | **Directly reused as a structural template** — this coupon's candidate matrix should follow the exact same `[...].map((offset,index) => ({index, id:'C'+..., label:...}))` shape T1 already uses, laying out each candidate's panels together and relabeling panel IDs with the candidate prefix (`C1-WALL-A`, `C1-WALL-B`, etc.), exactly as T1 does with `WALL-A`/`WALL-B`/`FLOOR`. |

**New, small, generator-specific code required**: one new function computing the hex-corner-specific derived values (`t·cot(θ/2)` for tab depth, the corresponding wall-length adjustment) from the interior angle and thickness — this is a handful of lines of trigonometry, not a new geometry engine, and it feeds its outputs into the existing, unmodified `buildBoxModel`/`buildFingerPattern` pipeline exactly the way every other generator already supplies its own derived numbers to that same shared pipeline.

**What must remain generator-specific and should not be generalized prematurely**: the interior-angle-to-tab-depth trigonometry is specific to this joint family; this review does not recommend generalizing `buildBoxModel` itself to accept an arbitrary corner angle parameter in this phase — that generalization, if ever justified, should wait until the mitered (Option 1b) escalation is actually needed, since forcing it in now would be exactly the kind of "abstraction for theoretical cleanliness" the task explicitly warns against.

---

## 9. Coupon dimensions

Recommended, derived (not merely copied from the suggested ranges):

| Parameter | Recommended starting value | Rationale |
| --- | --- | --- |
| Wall height | 30 mm | Middle of the suggested 25–35 mm range; comfortable for at least 3–5 fingers at a 6–8 mm finger width, per the existing odd-count algorithm. |
| Wall length (each of the two coupon walls) | 55 mm | Comfortable to hand-hold and clamp during a dry fit; well within the suggested 40–70 mm range; long enough that the corner joint's geometry is clearly visible and separated from the panel's far (uncut, plain) end. |
| Material thickness | 3 mm | Matches Joe's commonly used basswood/Baltic Birch/walnut plywood. |
| Preferred finger width | 7 mm | Comfortably above the existing 6 mm minimum for 3 mm ply; the existing algorithm will select the nearest odd count automatically. |

### A. One candidate per SVG, or B. a compact matrix?

**Recommended: B, a compact matrix of 3 candidates in one SVG**, following T1's own exact existing precedent (§8). A matrix is preferred here specifically because the single largest open question (§4.4) is **whether the derived tab-depth compensation is correct**, which is best isolated by holding clearance fixed and varying only tab-depth compensation across a small number of candidates — exactly the kind of side-by-side comparison a single labeled sheet supports well, and exactly the pattern T1 already established for its own single most important open variable (clearance).

---

## 10. Candidate matrix

**Recommended: 3 candidates, varying tab-depth compensation only, with clearance held fixed at a small, modest starting value common to all three** (this isolates the one variable this review's desk math could not fully resolve, per good experimental design, and keeps the assembly/evaluation burden small, per the task's explicit "keep the number of candidates low enough to assemble and evaluate carefully" instruction):

| Candidate | Tab depth (relative to `t`) | Purpose |
| --- | --- | --- |
| C1 | `1.00 · t` (the naive, unprojected 90°-style depth) | A deliberate negative control — expected, per §4.4, to show a visibly worse fit than C2, directly demonstrating why "conventional 90° tab depth" does not simply transfer to 120°. |
| C2 | `t · cot(60°) ≈ 0.577 · t` (this review's derived, projected value) | The primary hypothesis under test. |
| C3 | A modest step beyond C2 (e.g., roughly `0.70 · t`) | Tests whether a small amount of *extra* engagement beyond the pure derivation improves the dry-fit feel or glue surface, without assuming the pure derivation is necessarily the practical optimum. |

Clearance should be held at one small, fixed starting value across all three candidates (a modest value near zero — not zero necessarily, and explicitly not a reuse of Joe's prior −0.075 mm or +0.10 mm values, per §7) so that any difference observed between C1/C2/C3 is attributable to tab-depth compensation, not a confound with clearance. A **second, smaller follow-up matrix varying clearance around whichever tab-depth candidate performs best** is a natural next physical test, but is not required in this first coupon.

Each candidate must be clearly labeled outside the cut geometry (following T1's existing `assemblyLabels`/engraved-ID convention, e.g., "C1", "C2", "C3" plus their compensation factor), and each candidate's two walls should be positioned adjacently on the sheet so the correct pairing is unambiguous during assembly.

An optional small angle-registration gauge (a simple flat piece with a 120° notch or printed reference angle) may be included **only if** it materially helps Joe verify the assembled angle without needing a separate protractor/bevel gauge he may not have on hand — this review recommends including it, since it is a small, simple, low-risk addition to the same sheet and directly supports the pass/fail criteria in §14.

---

## 11. Future six-wall compatibility

Because a regular hexagon's six corners are all geometrically identical (by the shape's six-fold rotational symmetry), the **same** mirrored left/right edge-pair validated by this coupon applies unchanged at every corner of the eventual hex tray: each of the six walls needs one "left"-handed end and one "right"-handed end, consistently oriented around the perimeter, and the sixth wall's "right" end closes the loop against the first wall's "left" end using the exact same joint geometry as every other corner — no special-case sixth-corner logic is needed. This is a direct, confirmed consequence of the hexagon's symmetry, not an assumption requiring separate proof.

---

## 12. Future floor-corner implications (not designed in this phase)

The existing T1/T2 architecture's wall-to-floor "closed three-way corner" concept (`tabletopClosedCornerStates`, `floorCornerOwnership`) is explicitly 90°-specific today (it assumes the floor meets each wall at a right angle, which is still true for a hexagonal floor's *relationship to the vertical wall height axis*, but the *wall-to-wall* corner meeting the floor becomes a genuinely new three-way intersection at 120° in plan view, not addressed by this review). **What the wall-joint physical coupon must prove before floor-corner geometry design should begin**: (a) whether the derived tab-depth compensation (§4.4, §10) is physically correct or needs adjustment, (b) whether the residual interior-side mismatch is acceptable or demands escalation to the mitered joint (§6), and (c) the real, physically measured assembled angle and any systematic deviation from 120° — all three directly determine which wall-to-wall joint contract the eventual three-way wall-to-floor corner must be built around, so starting floor-corner design before this coupon's results are in would risk building on an unconfirmed foundation, exactly as the prompt cautions against.

---

## 13. Physical test matrix

Per §10: vary **tab-depth compensation** (3 levels) as the primary variable, holding **clearance** fixed at one small starting value across all three candidates. This directly answers the prompt's example question (a three-candidate clearance set centered on 0.00 mm) with a considered alternative: since the single largest open question this review's math could not fully resolve is tab depth (not clearance, which already has a well-understood existing convention from the 90° system), the matrix is built around tab-depth compensation instead, with clearance deliberately held constant to keep the test isolating one variable.

---

## 14. Pass/fail criteria

- Both panels fully seat, without requiring destructive force.
- The assembled interior angle is close to 120°, measured using a printed/laser-cut reference gauge (the optional small angle-registration piece, §10) checked against a bevel gauge or protractor Joe already owns, or a simple printed 120° angle template as a fallback if no gauge is available.
- The joint holds its angle unassisted during a dry fit (no clamping needed to keep the two walls at 120°) — this is the direct physical test of the "mechanically fixes the angle" claim in §6.
- No major interior gap and no major exterior gap (both assessed visually against the "small, quantified residual, not large" expectation from §4.4 — a materially larger-than-expected gap on either face is a fail signal for that candidate).
- No excessive panel twist out of the shared vertical plane.
- No veneer splitting, no fragile-tab failure, no crushed or torn fingers during assembly.
- The joint can be disassembled cleanly before glue (confirming reversibility).
- Glue surfaces (the mating tab/slot faces) are genuinely accessible and usable, not fully enclosed/inaccessible.
- The corner is visually acceptable for a sellable dice tray (a subjective, but explicit, pass/fail judgment call reserved for Joe).

**No candidate should be described as "proven" from this coupon alone** — a passing coupon result establishes a validated *starting point* for the wall-to-wall joint, not a production-proven claim, consistent with how every other Tabletop Accessories phase in this project has treated its own first physical test.

---

## 15. Screen-only diagram decision

**Recommended: yes, include a minimal, screen-only assembled 120° corner diagram (Option B).** Reasoning: the corner's assembled geometry (specifically, which face is "outer," which is "interior," and how the two mirrored walls come together) is genuinely harder to picture from a flat cut layout than an ordinary 90° corner is, precisely because it is unfamiliar — a small top-down (plan-view) diagram showing the two walls meeting at 120°, reusing the existing `tabletopAccessoryProjectPoint` projection helper (or, since this is a plan-view diagram rather than an isometric one, a simpler flat top-down `<polygon>` rendering may be even more appropriate and simpler than invoking the 3D-style projection helper), directly helps a first-time viewer confirm the intended orientation before cutting. This must follow every existing Finished View contract already established in this app: explicitly labeled screen-only, never entering `result.svg`, never altering panel layout or production dimensions, reusing the coupon's own calculated angle/dimensions rather than recalculating anything independently, deterministic, and degrading safely (empty string) for an invalid result. **No decorative rendering beyond this minimal, informative diagram is recommended.**

---

## 16. Validation rules

**Hard blocks:**
- Interior angle not equal to a valid, supported value (this coupon should hard-code 120° for the hexagon case — it is not a generic arbitrary-angle tool — so a hard block only matters if a future generalization is attempted; for this phase, the angle is a fixed constant, not a user input, eliminating this whole class of invalid input by construction).
- Zero/negative wall height, wall length, material thickness, or finger width.
- Material thickness too large relative to the coupon's short wall length (reuse `tabletopAccessoryManufacturingLimits`'s existing `minPanel = minSegment*3` style check).
- Finger width below the existing minimum (`Math.max(2·t,4)`).
- Too few usable fingers for the given wall height (reuse `buildFingerPattern`'s existing `maxOdd < 3` rejection).
- Tabs computed to extend beyond the panel's own bounds (reuse `designPanelGeometryErrors`).
- Fragile residual wall segments below the existing minimum web (`tabletopAccessoryManufacturingLimits`'s `minWeb`).
- Impossible handed-edge pairing (e.g., a configuration that would produce two identical, non-mirrored edges instead of a true left/right pair) — a new, small, explicit check specific to this generator.
- Invalid candidate values (e.g., a tab-depth-compensation factor that would exceed the wall's own thickness envelope or fall to zero/negative).
- Geometry self-intersection (reuse `designSegmentsIntersect`).
- Non-finite coordinates (reuse the existing finite-coordinate check pattern used throughout every Finished View and production-panel builder).
- Duplicate cut paths (reuse the existing per-panel uniqueness check pattern already used by `dice-tray`'s fixtures).
- Overlapping layout (reuse `layoutDesignPanelRows`'s existing error propagation).

**Warnings (not hard blocks):**
- Very short coupon wall length (comfortably assemblable but close to the practical minimum).
- Unusually tight or loose clearance relative to material thickness (reuse the existing warning pattern from `buildBoxModel`: "Joint clearance is unusually loose relative to material thickness" / the existing negative-clearance interference warning).
- A large exterior corner gap **predicted by the geometry** for a given candidate's inputs (a new, small, explicit warning specific to this generator, computed from the same §4 derivation, so a user who deliberately enters an extreme tab-depth-compensation value is told in advance that the geometry predicts a poor fit, rather than only discovering it after cutting).
- A configuration that assembles but does not mechanically fix the angle (e.g., an extremely small tab-depth-compensation value that would leave the tabs too shallow to meaningfully engage) — flagged as an advisory, not a hard block, since it may still be a legitimate exploratory candidate for testing.

**No production limitation should be hidden behind the screen-only diagram** — exactly as this session's Finished-Views review already established as a hard rule for every Finished View in this app, every hard block above prevents `result.valid` from becoming true, and the screen-only diagram's own guard (checking `result.valid` first) means an invalid configuration never produces a diagram that could look more finished than the actual cut geometry.

---

## 17. App integration

**Template ID**: `hex-tray-120-joint-coupon` (as suggested). **Display name**: "Hex Tray 120° Joint Coupon". **Status**: Prototype / test coupon — grouped with the existing "Coupons and prototypes" optgroup (`designTemplateSelect`, `index.html:2886-2891`), alongside `joint-fit-coupon` and the T1–T3A Tabletop Accessories prototypes, **not** under "Trays" (which holds finished/near-finished tray products) and **not** under "Workshop-ready" (reserved for T3B's own physically-validated maturity, per the existing `tabletopProductMaturityFor` registry).

**Minimum input fields**:
- Wall height (mm)
- Wall length (mm)
- Material thickness (mm)
- Preferred finger width (mm)
- Joint clearance (mm)
- Candidate set (fixed at the 3-candidate tab-depth-compensation matrix recommended in §10, not a freely user-editable list, to keep the coupon bounded and its purpose unambiguous — a future phase could open this up if useful)
- Panel spacing (mm)
- Assembly labels (boolean, reusing the existing shared toggle)

**Explicitly not exposed as raw user inputs**: the interior angle itself (fixed at 120°, since this is a hex-specific coupon, not a general-angle tool), and the derived trigonometric constants (`cot(60°)`, `sin(60°)`) — these are internal calculations, not user-tunable parameters, consistent with the task's instruction to avoid exposing raw geometric internals unless genuinely needed.

**Result reporting**: angle (120°, fixed), material thickness, effective/projected tab depth per candidate, finger width (actual, as computed by the existing pattern algorithm), clearance, candidate identity (C1/C2/C3), wall dimensions, expected assembled corner convention (a short, plain-language description, e.g. "two walls meeting at a 120° interior angle, mirrored left/right ends"), and an explicit starting-point status string (reusing the existing app-wide convention of labeling unproven prototypes, e.g. the same kind of language T1–T3A already use before their own physical evidence exists).

---

## 18. Panel and SVG contract

- Exact-scale millimeter SVG, produced by the existing `serializeTabletopAccessorySvg`-style pipeline, unmodified.
- Two wall panels per candidate (one "left"-handed, one "right"-handed), following T1's existing multi-candidate panel-ID convention (`C1-WALL-LEFT`, `C1-WALL-RIGHT`, etc.).
- An optional gauge panel (§10), included once (not per-candidate), if the angle-registration gauge is included.
- No Finished View required beyond the minimal screen-only diagram decided in §15 — no additional decorative rendering.
- No silent scaling: reuse the existing `designLayoutSizeWarning` mechanism verbatim for the ">400 mm, verify your sheet" warning; this coupon's dimensions should comfortably stay well under that threshold by design.
- No automatic multi-sheet behavior — six wall panels plus an optional gauge, at these dimensions, fit comfortably on a single sheet; no nesting/splitting logic is needed or recommended.
- Clear panel/candidate labels, reusing the existing `assemblyLabels` opt-in mechanism.
- No duplicate cut paths, no overlapping panels, closed finite paths — all enforced by the existing, reused validation helpers (§16).
- **One output SVG.**

---

## 19. Options considered (A–D)

| Option | Verdict |
| --- | --- |
| **A — Coupon generator only** | Viable, but §15 found a genuine, non-decorative comprehension need (understanding the unfamiliar 120° assembly orientation) that a minimal screen-only diagram directly addresses, at very low incremental cost since it reuses existing projection helpers. |
| **B — Coupon generator plus a minimal screen-only assembled diagram** | **Recommended.** |
| **C — Coupon plus full hexagonal shell generator** | **Rejected for this phase.** No physical evidence yet exists for even the two-wall corner joint (§14); building the full six-wall shell before that evidence exists would risk repeating the same mistake the task explicitly warns against — designing the floor-corner or full shell on an unconfirmed foundation. |
| **D — Full hexagonal dice tray** | **Rejected for this phase**, for the same reason as C, compounded — this option additionally assumes the floor-corner, rail, false-bottom, and liner contracts, none of which have been designed or tested at all. |

---

## 20. Recommended bounded implementation

A new Designs template, `hex-tray-120-joint-coupon` ("Hex Tray 120° Joint Coupon"), producing:
- A 3-candidate matrix (C1/C2/C3) varying tab-depth compensation (naive `1.00·t`, derived `0.577·t`, and a modest `~0.70·t` step beyond it), clearance held fixed across all three at one small starting value.
- Two mirrored ("left"/"right") wall panels per candidate, each an ordinary rectangle with an ordinary cardinal-direction finger pattern along its vertical (height) edge, built entirely from the existing, unmodified `buildFingerPattern`/`buildBoxModel` pipeline fed the derived wall-length and tab-depth values from §4/§7.
- An optional small angle-registration gauge panel.
- A minimal, screen-only, plan-view assembled-corner diagram (§15), reusing existing projection/rendering conventions.
- Full validation per §16, reusing existing shared helpers wherever possible.
- Fixtures per §21.

---

## 21. Exact non-goals

No hexagonal floor; no support rails; no removable false bottom; no liner allowance of any kind; no full hexagonal shell or six-wall assembly; no wall-to-floor three-way corner design; no mitered/diagonal-cut geometry engine (deferred, §6); no generic arbitrary-corner-angle tool (this generator is hex-specific, angle fixed at 120°); no change to any existing generator, helper, or production golden; no schema or storage change; no claim that any candidate is production-proven.

---

## 22. Storage/schema/backup impact

None. This is a new, additive Designs template following the exact existing pattern every prior template addition in this app has used — new, independently-defaulted `designDraft` fields, no `SCHEMA_VERSION` change, no new storage key, no change to `backupObject()`'s shape beyond the new draft fields appearing the same way every other template's fields already do.

## Existing-generator production-byte impact

None, by construction — no existing `build*DesignResult`/`build*FinishedViewSvg` function is modified; the new generator only calls the existing, unmodified `buildFingerPattern`/`buildBoxModel`/layout/validation helpers.

## New generator production-output contract

One production SVG; up to seven panels (six wall panels across three candidates, plus one optional gauge panel); exact-scale; single sheet; no engrave layer required beyond the existing optional assembly-label convention.

---

## 23. Fixtures

- **Template registration**: the new template appears exactly once in `designTemplateSelect` and exactly once in `buildDesignResult`'s dispatch chain.
- **Defaults**: `designDefaults()` includes valid, safe values for every new field.
- **Left/right handed geometry**: an explicit assertion that the two wall panels in any candidate are genuine mirror images (not identical), and that swapping which panel is labeled "left" vs. "right" is caught as an error if the geometry doesn't actually support the pairing.
- **Exact wall outlines**: fixed expected dimensions for the default scenario's wall panels.
- **Exact tab and slot count**: assert the existing odd-count convention holds for the chosen wall height/finger width combination.
- **Effective tab-depth calculation**: assert the derived `t · cot(60°)` value is computed correctly and distinctly from the naive `t` value, for at least two different material thicknesses (proving the formula, not just one hard-coded number).
- **Candidate clearance/tab-depth differences**: assert C1/C2/C3 produce genuinely different tab-depth values while sharing the same clearance.
- **Closed finite paths / no self-intersections / no duplicate cuts / non-overlapping layout**: reuse the existing shared assertion patterns already used by `dice-tray`'s and T1's fixtures.
- **Deterministic production SVG**: call the build function twice with identical inputs, assert identical output.
- **Exact production bounds**: `widthMm`/`heightMm` match the expected layout for the default scenario.
- **Invalid parameter boundaries**: boundary tests for every hard block in §16, following the existing boundary-testing style already used elsewhere in this app (test just-below, at, and just-above each threshold).
- **Labels**: assembly-label opt-in produces the expected label set for each candidate.
- **No storage/schema change**: assert `SCHEMA_VERSION` and `STORAGE_KEY` are unchanged.
- **Project handoff**: include only if useful for test documentation — a simple, descriptive name/notes branch in `projectDraftFromDesign`, following the existing per-template pattern, recording which candidate was cut and its derived tab-depth value, useful for Joe to log which physical coupon he actually tested.
- **All existing generator goldens unchanged**: re-run the full existing golden table and confirm every prior hash is unchanged.
- **Complete Designs suite unchanged**: the full `runDesignGeometryFixtures`/production-golden group must remain fully green.

No fixture or physical result is claimed as executed in this review.

---

## 24. Risks

- **Risk**: the derived `t·cot(60°)` tab depth, while mathematically correct for the *idealized* corner geometry, does not account for real kerf width, char, or thin-plywood compressibility — this is precisely why the physical test (§13–§14) exists, and why C1/C3 bracket C2 rather than shipping only the single derived value.
- **Risk**: the small interior-side residual mismatch (§4.4) proves larger in practice than the ~0.42·t desk estimate, due to real material tolerances — mitigated by the explicit escalation path to the mitered joint (§6) if this occurs.
- **Risk**: scope creep toward the full hexagonal shell before physical evidence exists — mitigated by this report's explicit non-goals (§21) and the Option C/D rejection (§19).
- **Risk**: handedness confusion during assembly (mixing up which wall is "left" vs. "right") — mitigated by explicit, clear labeling (§18) and by the recommended fixture explicitly proving the two panels are genuine, distinguishable mirror images (§23).

---

## 25. Rollback strategy

This phase is entirely additive (new template, new small trigonometry helper calling existing unmodified geometry functions, new fixtures) — no existing function, schema constant, or storage key is touched. Rollback is a plain code revert with no data migration to reverse and no persisted state to unwind, the same low-risk profile as every other purely-additive Designs template addition in this project's history.

---

## 26. Unverified areas

- No browser was launched for this review; all findings are geometric derivations and source/fixture-text reading.
- The exact real-world optimal tab-depth-compensation value is explicitly not resolved by this review — that is the coupon's entire purpose.
- Whether a small angle-registration gauge is genuinely necessary, versus Joe already owning a suitable bevel gauge or protractor, is a product/workshop-equipment question this review does not resolve — it is recommended as a low-cost inclusion, not asserted as strictly required.
- Whether the mitered/diagonal escalation (§6, Option 1b) will ever be needed depends entirely on this coupon's physical results, which do not yet exist.

---

## 27. Concise Codex implementation handoff

> Implement the "Hex Tray 120° Joint Coupon" Designs generator (template id `hex-tray-120-joint-coupon`) in `C:\Genmitsu L8 Tracker\index.html`, per `docs/HEX_TRAY_120_DEGREE_JOINT_COUPON_ARCHITECTURE_REVIEW_2026-07-24.md` §20. Register it once in `designTemplateSelect` under the existing "Coupons and prototypes" optgroup, and once in `buildDesignResult`'s dispatch chain. Implement a small, new trigonometry helper computing, for a fixed 120° interior angle: the along-length inner-corner offset `t · cot(60°)` and the corresponding tab-depth-compensation values for the three candidates (naive `1.00·t`, derived `≈0.577·t`, and a modest step beyond it `≈0.70·t`), all derived exactly as shown in §4/§7 of the review — do not invent different formulas. Feed these derived numbers into the existing, unmodified `buildFingerPattern`/`buildBoxModel`/`buildSlidingLidBodyPanel` pipeline to build two mirrored (left/right) plain rectangular wall panels per candidate, following `buildTabletopCornerFloorCouponDesignResult`'s (T1, `index.html:5209`) existing multi-candidate panel-ID/labeling/layout pattern exactly. Include an optional small angle-registration gauge panel and a minimal, screen-only, plan-view assembled-corner diagram (reusing `tabletopAccessoryProjectPoint`/`serializeTabletopAccessorySvg`-style conventions), following every existing Finished View contract in this app (screen-only, never in `result.svg`, deterministic, degrades safely). Do **not** implement a diagonal/mitered cut, a hexagonal floor, rails, a false bottom, a liner, or a full hexagonal shell — all explicitly deferred per §21. Add fixtures per §23 (registration, defaults, handedness, exact dimensions, tab-depth-formula correctness at multiple thicknesses, candidate differentiation, closed/non-intersecting/non-duplicate/non-overlapping geometry, determinism, invalid-parameter boundaries, labels, no schema change, all existing goldens unchanged), then re-run the full Designs/production-golden suite and confirm zero regressions before calling the phase complete. Treat every candidate as an unproven starting point requiring the physical test in §13–§14 before any "proven" claim is made anywhere in the UI, and do not reuse Joe's prior −0.075 mm shell clearance or +0.10 mm coupon-fit value as a default for this different joint contract.
