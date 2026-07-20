# Tabletop Accessories T2A — Rectangular Shell Prototype, Evidence-Guided Clearance, and Compact Cut Layout — Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-20
**Reviewer:** Claude Sonnet 5
**Type:** read-only architecture and geometry review. No product file, README, CHANGELOG, fixture, storage, application record, or existing report was edited. Nothing was staged, committed, or pushed. Only this report was written. All runtime probing described below used disposable browser profiles and synthetic data only — Joe's real browser profile and production `localStorage` were never touched.

## 1. Repository state and actual HEAD

- `git log -1 --oneline` / `git rev-parse HEAD`: `d0a89e4afd3bba4de47e924032b3aa64182d8eb7` — "Add Tabletop coupon evidence workflow." Matches the stated authoritative baseline exactly.
- Branch `main`; `git rev-list --left-right --count origin/main...main` = `0 0` (synchronized).
- `git diff --check`, `git diff`, `git diff --cached`: all empty. **Tracked working tree is fully clean** — T1.5 and its correction are now committed.
- `git ls-files --others --exclude-standard`: identical to the long-standing untracked set (LightBurn Projects, `debug.log`, historical `docs/*.md`, `parametric_qr_stand_generator.py`) — untouched.
- T1 and T1.5 reports (architecture review, construction contract review, implementation, focused audit, correction) are all present on disk and were read in full for this review.
- README confirms the current measured totals: `2642 passed / 0 failed` across 34 groups, matching the stated baseline exactly (README §"Built-in checks").
- Constants confirmed unchanged: `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.

## 2. Existing functions and systems inspected

Direct source reads (not report summaries) of: the Tabletop registry entry; `tabletopAccessoryRectangleOutline`, `tabletopAccessoryHexagonOutline`, `tabletopAccessoryWallLoop`, `tabletopAccessoryCornerDescriptors`, `tabletopAccessoryDimensions`, `tabletopAccessoryFinishedCavity`, `tabletopAccessoryDividerFromCavity`, `tabletopAccessoryInsertFromCavity`, `tabletopAccessoryManufacturingLimits`, `tabletopAccessoryProjectPoint`, `serializeTabletopAccessorySvg`; `buildTabletopCornerFloorCouponDesignResult` and `buildTabletopCornerFloorFinishedViewSvg`; `buildBoxModel` and `buildFingerPattern` (panel/edge/phase construction, terminal webs); `layoutDesignPanelRows` and `layoutDesignPanels` (the shared row-based packer already used by the generic box, Drawer Cabinet, Dice Tray, and Gift Box); `buildAssemblyLabelPaths`; `tabletopCouponRankMatches`, `tabletopCouponConstructionMatches`, `tabletopCouponMaterialMatches`, `tabletopCouponThicknessTier`; `productionMachineIdentityMatches`; the T1.5 `normalizeTabletopCouponResult`/`tabletopCouponResultEligibility`/`buildTabletopCouponPromotionCandidate`/`finalizeTabletopCouponPromotion`; `normalizeProductionEvidence`'s optional `fitContext`; `projectDraftFromDesign`; and the `?selftest=` registration block (34 groups, matching README).

## 3. Physical Coupon-proven evidence summary

Machine: Genmitsu L8 20W, existing owned-machine identity from the T1.5 workflow. Material: plywood, nominal ~3 mm, **measured 2.88 mm**, no invented species/vendor/coating. Coupon: Tabletop Corner + Floor Coupon, `finger-90` corner, `finger-jointed-floor`, preferred finger width 9 mm, interior leg 40 mm, wall height 22 mm. Candidates C1 −0.075 / C2 −0.050 / C3 −0.025 mm. **C1 (−0.075 mm) is preferred** — best friction fit, one wall self-supports alone. C2 was acceptable but looser (needs two walls to self-support); **no tighter alternate exists because the preferred candidate is already the tightest tested value** — this is a normal, correctly representable state under T1.5's existing optional-alternate model (alternate stays `null`), not a gap. C3 not selected. Already saved and explicitly promoted: Coupon-proven, Tested, exact machine identity, valid Library material profile, complete `fitContext`. Locked interpretation carried forward unmodified: −0.075 mm is an evidence-backed **starting point** for this specific machine/material/measured-thickness/construction/engine combination; a complete shell may tighten further because multiple joints engage at once; this must not be called Shell-proven before physical assembly and evaluation.

## 4. Primary compatibility problem

`tabletopCouponConstructionMatches` requires exact equality (within `1e-6`) of finger **count and actual width on both axes**, plus exact `generatorVersion`/`wallCornerJoint`/`floorJoint`. A rectangular shell has different panel lengths than the 40×40 mm coupon leg, so `buildFingerPattern`'s length÷target-width optimization will almost always select a different count and a different actual width. Direct computation confirms this is not a corner case: the coupon's own actual pattern (interior leg 40 mm, wall height 22 mm, thickness 2.88 mm, preferred width 9 mm) resolves to **horizontal/floor: count 5, actual width ≈9.15 mm; vertical/corner: count 3, actual width ≈8.29 mm** (outside dimensions 45.76×45.76×24.88 mm). Any materially different, genuinely rectangular prototype size will resolve to different counts/widths on at least one axis. An exact-identity requirement would therefore reject Joe's real, physically relevant −0.075 mm evidence for essentially any useful shell size — confirming the problem is real and must be solved before implementation, not deferred.

## 5. Compatibility options evaluated

- **Option A — exact T1.5 identity match.** Rejected. Forcing the shell's outside width/depth/height to reproduce the coupon's exact 5×9.15 mm / 3×8.29 mm pattern would require an interior leg within a very narrow band around 40 mm on every axis — i.e., essentially the same size as the coupon, square, not a genuine rectangular multi-joint prototype. This directly violates the instruction not to force an "unhelpfully tiny or distorted shell."
- **Option B — a separate, named joint-fit compatibility identity, provenance identity unchanged.** Adds a second, deliberately looser, explicitly-named comparison model alongside (never replacing) `constructionIdentity`. Reusable by T2A now and by any future Tabletop shell product later without re-deriving the concept.
- **Option C — a T2-specific compatibility adapter function only, no named identity concept.** Functionally similar to B but narrower in scope — a one-off query local to T2A rather than a documented, reusable contract.
- **Option D — weaken T1.5 identity globally.** Disfavored per the task's own instruction; `tabletopCouponConstructionMatches` is not "unnecessarily strict" for its actual purpose (exact coupon-to-coupon provenance matching, already fixture-proven correct) — weakening it would degrade provenance for every existing and future consumer to solve a problem specific to cross-product clearance transfer.
- **Option E.** Not needed; B is sufficient and bounded.

## 6. Selected evidence-compatibility model

**Option B.** Add a new, explicitly named `tabletopJointFitCompatibility` field set and a new pure function `tabletopJointFitCompatible(evidenceIdentity, targetIdentity, evidenceThicknessMm, targetThicknessMm, clearanceStepMm)`, implemented and fixture-tested independently of `tabletopCouponConstructionMatches`, which is left **completely unmodified** (protecting historical provenance exactly as instructed). The new function is reusable by T2A and by any later Tabletop shell product, so the concept is derived once rather than re-invented per product. `constructionIdentity` continues to answer "is this the same tested construction" (coupon-to-coupon); `tabletopJointFitCompatibility` answers a strictly separate question — "may this coupon's clearance be offered as a starting point for this different but related shell."

## 7. Exact versus compatible terminology (locked)

- **Exact** — passes `tabletopCouponConstructionMatches` unchanged (identical family/template/generatorVersion/wallCornerJoint/floorJoint/finger count/actual width within `1e-6`). In practice this only occurs for literal coupon-to-coupon reuse, not coupon-to-shell.
- **Compatible** — passes the new `tabletopJointFitCompatible` gate (§9-§10) but does **not** require exact finger count or exact actual width. The recommendation UI must display, verbatim: **"Compatible Coupon-proven evidence — Not an exact full-shell validation."** It must never be labeled an "exact proven shell setting."

## 8. Machine/material/thickness matching for compatibility

- **Machine — exact requirement, not merely ranked.** Reuses `productionMachineIdentityMatches` unmodified (exact `machineId` match, or legacy-key fallback only when neither side has an ID — the established M3 rule). A genuine machine mismatch **excludes** the evidence from the "compatible" recommendation set entirely (matches the task's own "blocking" framing for wrong-machine evidence); the evidence remains separately viewable via normal Library browsing, just not offered as a shell recommendation.
- **Material — ranking tier, not a hard exclusion.** Reuses `tabletopCouponMaterialMatches` unmodified: exact `materialId` is strongest, snapshot-label match is next, and an unconfirmed/mismatched material still surfaces with an explicit "material not confirmed" caveat rather than being hidden — consistent with T1.5's existing philosophy of never hiding evidence outright over material identity alone.
- **Thickness — tolerance tier, reused unmodified.** Reuses `tabletopCouponThicknessTier` exactly (strongest within one clearance step, close within three steps, excluded beyond). No new thickness tolerance is invented.

## 9. Finger-pattern compatibility (the core new rule)

**Finger count may differ safely; only actual finger width needs a tolerance check.** Reasoning: `buildFingerPattern` always optimizes for the *same* `preferredFingerWidth` target; count is a discrete, coupled side effect of panel length divided by that target, with an odd-count constraint. A longer panel simply gets more fingers at essentially the same per-finger width — the physically meaningful quantity for friction/interference behavior is the finger's width relative to the fixed absolute clearance value, not how many fingers exist. Two joints with 9.15 mm and 9.53 mm actual fingers behave nearly identically at a −0.075 mm clearance (0.82% vs. 0.79% of finger width); a joint with fingers of a genuinely different scale (e.g., 20 mm) does not.

**Locked tolerance (grounded, not casual):** for each axis independently, `|actualWidthCandidateA − actualWidthCandidateB| ≤ max(0.15 × actualWidthCandidateA, manufacturingMinimum.minWeb / 2)`. The 15% band is grounded directly in this system's own observed natural rounding variance from aiming at one shared `preferredFingerWidth` target across different panel lengths (the coupon's own two axes already differ by ~9.15 vs. ~8.29 mm — an ~11% relative difference — purely from the odd-count constraint, not from a different joint system); the `minWeb/2` floor prevents the percentage term from becoming meaninglessly tiny at very small finger widths. Both the horizontal/floor axis and the vertical/corner axis must independently pass; finger **count** is not compared at all.

## 10. Generator-version behavior

`generatorVersion` remains an **exact-match requirement**, not tolerance-based, in both `constructionIdentity` and the new compatibility model. A future engine revision (e.g., a T2.1 shell engine change) must never silently inherit today's coupon evidence — it must show "no compatible evidence yet" until re-tested, exactly as the T1.5 architecture already locked for the coupon-to-coupon case. `wallCornerJoint`/`floorJoint`/`family` remain exact-match requirements for the same reason: different joint systems are never treated as equivalent merely because both are finger joints.

**Field classification summary:**

| Field | Classification |
| --- | --- |
| family | Exact requirement |
| generatorVersion | Exact requirement |
| wallCornerJoint | Exact requirement |
| floorJoint | Exact requirement |
| clearance sign convention | Exact requirement (invariant; only one convention exists today, checked defensively) |
| machine identity | Exact requirement for "compatible" classification (ranking tier for display of non-compatible evidence elsewhere) |
| measured material thickness | Tolerance-based (reused `tabletopCouponThicknessTier`) |
| actual horizontal/floor finger width | Tolerance-based (§9, 15%/minWeb-floor rule) |
| actual vertical/corner finger width | Tolerance-based (§9, same rule) |
| material identity | Ranking context, never a hard exclusion on its own |
| horizontal/vertical finger counts | Irrelevant to the compatibility gate (may differ freely) |
| requested/target finger width | Provenance-only / informational |
| panel length, wall height | Provenance-only / informational (their effect is captured indirectly via actual width) |
| manufacturing minimum, terminal web | Irrelevant to transfer (each object already independently self-validated) |
| open-shell vs. coupon construction, number of simultaneously engaging joints | Provenance-only / mandatory UI disclosure, never a pass/fail field |

## 11. Template identity (locked)

**Display name: "Tabletop Rectangular Shell Prototype."** **Template ID: `tabletop-rectangular-shell-prototype`.** The word "prototype" is embedded directly in the ID (not only the display name) as an extra, durable safeguard against ever being confused with a future sellable "Premium Rectangular Tray," which will receive its own distinct template ID when that day comes. This ID is visibly distinct from `dice-tray`, `finger-box`, `gift-box`, and `tabletop-corner-floor-coupon`, and no future Hex product may reuse it.

## 12. Rectangular construction contract

All preferred-direction items are **confirmed as stated** in the task, locked as follows: interior-primary dimension mode; finished cavity equals requested interior absent explicitly disclosed intrusions (reuses `tabletopAccessoryDimensions`'s existing interior-primary guard, which already blocks silent shrink — no new code needed); outer width = interior width + 2×thickness; outer depth = interior depth + 2×thickness; open top; four walls; one finger-jointed floor; **one shared clearance value applied to both wall corners and floor joints**, matching the coupon's own combined test exactly; kerf remains fully separate from joint clearance (informational `kerfReference` only, never auto-subtracted); no groove, rebate, channel, pocket, dado, bevel, captured floor, or CNC-only feature of any kind.

**Geometry reuse determination:** `buildBoxModel` already produces the complete five-panel topology needed (`bottom`/`front`/`back`/`left`/`right`) using `width`/`depth`/`vertical` finger patterns and an `lid:'open'` mode that already exists and is already used in production by the legacy Finger Box template. Front/back share one edge-phase value and left/right share the opposite phase value in the existing model, which is exactly why all four corners already alternate tabs/slots correctly today — this is not new or unproven geometry; it is the same generic box math the shipped Finger Box template already relies on. **T2A therefore reuses `buildBoxModel`/`buildFingerPattern`/`tabletopAccessoryDimensions`/`tabletopAccessoryFinishedCavity`/`tabletopAccessoryManufacturingLimits`/`tabletopAccessoryProjectPoint`/`serializeTabletopAccessorySvg` completely unchanged**, through one new, small, Tabletop-specific wrapper function (analogous to `buildTabletopCornerFloorCouponDesignResult`) that keeps all five `buildBoxModel` panels instead of filtering to three. No shared-geometry extension is required, and this is not "casually routing legacy Finger Box through the new Tabletop engine" — it is the reverse: T2A (like T1 before it) reuses the same pre-existing generic box math Finger Box already uses; Finger Box's own builder and serializer are untouched.

## 13. Prototype-dimension selection

**Recommended: 80 × 60 × 25 mm interior**, using measured 2.88 mm plywood and preferred finger width 9 mm (matching the coupon's own tested target width for maximal evidence relevance).

- Outer envelope: 85.76 × 65.76 × 27.88 mm — comfortably inside a 12×12 in (304.8×304.8 mm) sheet even before compact packing.
- Computed finger patterns: width axis (85.76 mm) → count 9, actual width ≈9.53 mm; depth axis (65.76 mm) → count 7, actual width ≈9.39 mm; vertical axis (27.88 mm) → count 3, actual width ≈9.29 mm. All three comfortably exceed the minimum three-finger requirement and are all within the §9 compatibility tolerance of the coupon's own 5×9.15/3×8.29 mm pattern.
- Genuinely rectangular (width ≠ depth), unlike a 60×60 mm option, which would still read as "a bigger square coupon" rather than a true general-rectangle validation.
- Exposes real cumulative multi-joint behavior: four corners × 3 vertical fingers (12 corner-finger engagements) plus four floor edges × 7–9 fingers (28–36 floor-finger engagements) simultaneously — far beyond the coupon's single corner.
- Small and light enough for confident one-handed dry fitting; walls are rigid at this thickness/height.
- 100×70×30 mm was considered but uses more material and height for no added validation value at this stage; 60×60×25 mm was considered but reads as a same-shape repeat of the coupon rather than a genuine rectangle test.

**Safe configurable ranges for future user inputs:** interior width/depth 40–200 mm each (matches the coupon's own already-validated lower bound; upper bound keeps this a hand-dry-fittable prototype rather than requiring multi-sheet nesting the app does not have); wall height 15–100 mm; all inputs remain additionally gated by the existing, unmodified `tabletopAccessoryManufacturingLimits`/`buildFingerPattern` minimum-segment validation, so no new range-enforcement logic is required beyond the existing generic checks.

## 14. Panel topology and phases

Five panels, in a fixed, stable order matching `buildBoxModel`'s own internal definition order (bottom→front→back→left→right), renamed for the real shell: **`FLOOR`, `FRONT`, `BACK`, `LEFT`, `RIGHT`** (uppercase, deliberately distinct from T1's `WALL-A`/`WALL-B`/`FLOOR` coupon naming to avoid any confusion between coupon and shell records in Help text, saved results, or evidence). Corner joints: `FRONT`–`LEFT`, `FRONT`–`RIGHT`, `BACK`–`LEFT`, `BACK`–`RIGHT` — four `finger-90` corners total, each independently validated to alternate tabs/slots correctly by the existing, already-production-proven box-model phase assignment (§12). Floor joints: `FLOOR`–`FRONT`, `FLOOR`–`BACK`, `FLOOR`–`LEFT`, `FLOOR`–`RIGHT` — four `finger-jointed-floor` edges total. All are reused unmodified from `buildBoxModel`'s existing edge/phase construction; no new phase logic is written.

## 15. Finished-cavity truth

Reuses `tabletopAccessoryDimensions`/`tabletopAccessoryFinishedCavity` exactly as T1 already calls them, but with independent `width`/`depth` (a true rectangle) instead of the coupon's forced single `legLength`. The underlying function signature already accepts distinct width/depth — no extension needed. Finished cavity equals requested interior (80×60×25 mm) unless an explicit, disclosed intrusion is supplied (none exist in T2A); the existing interior-primary guard continues to block any silent smaller-cavity substitution.

## 16. Clearance and kerf behavior

One shared, signed clearance value (millimeters, negative = interference, matching the sole existing sign convention) applies uniformly to both the four wall corners and the four floor joints, exactly mirroring how the coupon tested both together. Kerf remains a separate, informational `kerfReference` field, never automatically subtracted from clearance or geometry — unchanged from every existing template's convention.

## 17. Recommendation UI

The shell form shows: current clearance value; compatible Coupon-proven evidence when available (machine, material, measured thickness, evidence date, evidence stage, exact-vs-compatible status); the recommended clearance; and an always-visible warning that the complete shell is not yet validated (verbatim caveat drawn from §3's locked interpretation: "a complete shell may become tighter because multiple joints engage simultaneously"). Controls: **View evidence**, **Apply recommendation**, **Keep current value**, **Run a new coupon** — directly reusing the same control set and visual language already established for T1.5's deferred recommendation-card concept, adapted to the shell.

## 18. Explicit Apply and override

Applying is explicit only (a dedicated button click, never automatic on form load or template switch). Applying fills the ordinary, still-editable clearance form field on the **current** Design draft only; it does not change any universal/global default, does not mutate the saved raw `tabletopCouponResults` record, does not mutate the promoted Evidence/Production Setting snapshot, does not mark anything Shell-proven, and does not reach into or update any other open Design form or session. This mirrors the "never silently apply, always explicit, never mutate frozen evidence" contract already established and fixture-proven in T1.5. **Override status is represented in both** the Design result summary (whether the current clearance came from an applied recommendation or manual entry) **and** the Project handoff notes (§26) — the task explicitly lists "manual override status" among the provenance the handoff should preserve, so both surfaces carry it rather than only one.

## 19. No-evidence and conflicting-evidence behavior

When no compatible evidence exists, the form shows its ordinary default clearance with a "Run a new coupon" link and no recommendation card — no error, no block. When multiple compatible entries conflict (e.g., two different preferred clearances for the same machine/material/construction), **all are shown, ranked, never averaged, never auto-selected** — directly reusing `tabletopCouponRankMatches`'s existing tie-handling behavior (sorted by rank, then by date descending), extended only with the new compatibility gate in place of the exact-identity gate.

## 20. Compact layout architecture

Joe's manual LightBurn improvement groups floor panels in one column and wall panels in a compact grid, with far less unused area than T1's current per-candidate row grouping. The existing `layoutDesignPanelRows(panels, rowIds, margin, gap)` primitive — already the shared, deterministic, overlap/spacing/bounds-validated packer used by the generic box, Drawer Cabinet, Dice Tray, and Gift Box — already supports exactly this style of layout: it accepts an arbitrary row grouping and is precisely how `layoutDesignPanels` already groups `[['bottom','lid'],['front','back'],['left','right']]` for the plain box. **Recommended strategy: Role grouped**, implemented as a small new row definition `[['FLOOR'], ['FRONT','BACK'], ['LEFT','RIGHT']]` passed into the existing, unmodified `layoutDesignPanelRows`. This is not a new geometry/packing engine — it is a new *row grouping* fed into an already-proven packer, bounded and low-risk. It automatically preserves every required property because `layoutDesignPanelRows` already enforces them: exact scale (plain millimeter geometry, no scaling step exists anywhere in the function), deterministic output (pure function of panel order and dimensions), no AABB overlap (`designBoxesOverlap` check raises an error), minimum requested spacing preserved (`designBoxesSeparatedBy` check raises an error), production bounds equal actual layout bounds (explicit bounds-containment check), stable piece ordering (array order is preserved), and reproducible SVG bytes (no randomness anywhere in the packer). Label-to-panel attachment is preserved because labels are generated from the already-placed `layout.panels` exactly as T1 already does. **T1's own coupon layout is not retrofitted** — its existing per-candidate row grouping and 2992-byte/`4f543f95` golden remain completely untouched; T2A defines its own new row grouping used only by the new template, with its own new golden approved only after this review.

Compact Packing (true bin-packing/nesting) and a fully generic Assembly Grouped mode were considered and rejected as **not needed**: Role grouped already delivers Joe's stated preference (floor alone, walls compact) using the existing packer with zero new geometry-packing logic, and building a general-purpose nesting optimizer is explicitly out of scope unless clearly justified — it is not justified for a five-panel prototype.

## 21. Rotation and grain policy

**Locked: no rotation in T2A.** The role-grouped layout already achieves good material efficiency without needing any panel rotated (front/back share identical dimensions and pack directly side by side; left/right likewise; floor stands alone), so rotation adds no material-efficiency benefit here while introducing real risk to plywood grain direction, veneer appearance, label readability, and user expectations. This is the simplest, safest, and most bounded of the three offered policies and needs no explicit-toggle UI, no rotation-transform code, and no additional fixture burden in T2A. If a future phase needs rotation (e.g., an oddly proportioned custom size that would otherwise waste material), it must be added as an explicit, off-by-default, disclosed toggle exactly as the task describes — not as part of this bounded phase.

## 22. Production SVG contract

Reuses `serializeTabletopAccessorySvg` completely unchanged (red cut paths, green assembly-label paths) — it already accepts any `result` with `panels`/`labelPaths` and requires no Tabletop-shell-specific modification. Exact millimeter scale, closed unique panel paths, and finite-value validation are inherited unchanged from the existing `designSvgValidation` pipeline every other template already uses. Production bounds come directly from the new role-grouped `layoutDesignPanelRows` call, so bounds-equals-actual-layout is guaranteed by that function's own existing invariant.

## 23. Finished View

One new, small, screen-only builder (analogous to `buildTabletopCornerFloorFinishedViewSvg`) reusing `tabletopAccessoryProjectPoint` unchanged, showing the complete open-top shell (floor + four walls) in one assembled schematic — no groove/rebate/channel/lid/divider/insert/bevel/hardware is depicted, matching the coupon's own precedent of a simple, honest schematic. **Assembled mode only** — an exploded view solves no real assembly problem for a first five-panel prototype and would add complexity, a second rendering path, and a second set of fixtures for no stated benefit; the task's own steer favors one mode unless exploded view is clearly needed, and it is not. Deterministic face ordering (floor, then front/back, then left/right, matching §14's panel order); finite coordinates only (the projection helper already rejects malformed points); understandable at small viewport sizes by following the coupon's existing simple-schematic visual style. Remains fully separate from the production SVG, exactly as T1's Finished View already is.

## 24. Result summary

Recommended exact metrics: requested interior width/depth/height; finished cavity; outer envelope; measured material thickness; current joint clearance; whether the current clearance came from applied evidence or manual entry; evidence stage (Coupon-proven, when applicable); compatibility level (Exact/Compatible/None); preferred finger width (target); actual horizontal/floor finger count and width; actual vertical/corner finger count and width; manufacturing minimum; **five** physical pieces (not nine, since this is one complete shell, not three coupon candidates); sheet/layout dimensions; layout strategy name ("Role grouped"); rotation status ("No rotation applied"); assembly-label count; the existing exact-scale-import notice; and an explicit physical-prototype warning reusing T1's own established wording style ("not proof of fit, strength, glue quality, or production readiness"). No NaN/undefined is possible given the existing, reused validation pipeline; no hidden cavity shrink is possible given the reused interior-primary guard; no "production proven" language is ever used anywhere in this summary.

## 25. Validation

**Blocks:** nonfinite/nonpositive dimensions; missing or out-of-range material thickness; wall height too short for a valid vertical finger pattern (`buildFingerPattern` returns `null`, already the existing error path); width/depth too short for a valid floor pattern; preferred finger width below the manufacturing minimum; impossible odd finger counts (`maxOdd < 3`, already existing); nonpositive panel spacing; incomplete construction identity (blocks promotion/matching only, not generation); silent cavity shrink (already blocked); NaN/Infinity (already blocked); malformed production paths (already blocked); layout overflow and overlapping panels (already blocked by the reused packer). Compatible-evidence exclusions (not generation blocks, but exclusions from the recommendation set): wrong generator version; wrong joint system (`family`/`wallCornerJoint`/`floorJoint`); wrong machine identity; missing measured thickness on the evidence side. **Warnings (non-blocking):** unsupported/unusually loose or negative clearance magnitude (reusing the coupon's existing warning thresholds); thickness at the "close" tolerance tier; material not confirmed by ID. **Informational evidence caveats (always shown, never blocking):** "a complete shell may become tighter because multiple joints engage simultaneously"; "this is Coupon-proven evidence, not Shell-proven"; "Compatible, not an exact full-shell validation."

## 26. Project handoff

**Locked: no Project schema change.** Directly extends the existing, already-zero-schema-change generic Design-to-Project handoff pattern established by T1 (`projectDraftFromDesign`), encoding as descriptive notes/name text: shell dimensions; selected clearance; whether evidence was applied or the value was manually entered; evidence stage; the source setting/evidence ID **as plain text**, not a live foreign-key reference (so a later-deleted evidence record cannot break the Project); machine and material snapshot text; measured thickness; manual override status; and the prototype-only caveat. This satisfies "do not create a live dependency that breaks if evidence is later deleted" and avoids inventing a new Project field where existing free-text handoff notes are already sufficient and already proven safe by T1's own precedent.

## 27. Physical shell-test procedure

1. Use the same measured 2.88 mm plywood.
2. Explicitly Apply the compatible Coupon-proven −0.075 mm clearance (never pre-filled automatically).
3. Import the downloaded SVG into LightBurn at 100% scale.
4. Verify dimensions in LightBurn against the result summary before cutting.
5. Frame the job on the bed.
6. Cut with active ventilation, fire watch, and suppression nearby.
7. Keep all five panels identified by their printed/etched labels (`FLOOR`/`FRONT`/`BACK`/`LEFT`/`RIGHT`).
8. Dry-fit without glue, in the recommended order (§ below).
9. Record: individual corner effort (all four, not just one); floor-joint effort per edge; whether all four walls can seat simultaneously with the floor; cumulative tightness as more walls are added; squareness; bowing; veneer crushing; split tabs; floor seating/gaps; finished cavity dimensions as actually assembled; whether disassembly damages any part.
10. Do not call the setting Shell-proven until this test passes (§28).

**Recommended assembly order:** floor first (reference plane) → attach two adjacent walls first (e.g., FRONT then LEFT), mirroring the coupon's own "one wall can self-support" observation → add the third wall (RIGHT) to FRONT's remaining corner and the floor → close the loop last with the fourth wall (BACK), since closing the final two corners simultaneously is the highest-difficulty step and should only be attempted once the other three are confirmed properly seated.

## 28. Pass/fail criteria

**Pass (eligible for a later, separate Shell-proven decision — never automatic):** all four walls and the floor seat together without cracking or splitting; friction is broadly consistent across all four corners (no single corner drastically tighter or looser than the others); the shell holds itself square by hand within ordinary workshop tolerance; no veneer crushing or split tabs at any joint; the floor is fully seated with no persistent gap; the shell can be disassembled without breaking any tab (confirming the fit is firm but not destructively tight).

**Fail / triggers the −0.050 mm fallback (§29):** any corner or floor joint requires force risking splitting; visible tab splitting or veneer crushing at any joint; the shell cannot be squared by hand (persistent racking); disassembly damages any tab.

## 29. Fallback to −0.050 mm

If the −0.075 mm test fails any criterion in §28, the next test uses **−0.050 mm** — the already-tested, evidence-backed "acceptable but looser" value from the same coupon — rather than skipping to an untested value. This preserves the principle that every clearance tried in the shell test remains grounded in existing Coupon-proven evidence.

## 30. Shell-evidence follow-up

**Recommended: a separate, later T2B phase**, not part of T2A, and not an extension of `tabletopCouponResults`. Reusing/extending the coupon-result shape would distort it exactly the way T1.5 warned against distorting Material Test's shape (it is a fixed three-candidate C1/C2/C3 shape; a shell result is a single-outcome record). Recommended, deferred T2B design: a small, genuinely new "shell result" raw record following the same proven lifecycle T1.5 already established — raw observation → explicit promotion — reusing the **existing** `fitContext`/evidence machinery with one new `sourceType:'tabletop-shell'` and one new `evidenceStage:'shell-proven'` value (already a free string field today, not a hardcoded enum requiring a schema change). This keeps T2B's eventual implementation low-risk when it is built, while T2A itself implements none of it. History is preserved: Coupon-proven evidence is never overwritten; a later Shell-proven entry coexists as a separate, higher-ranked entry for the same construction/machine/material via the existing supersede mechanism (as already recommended for the T1.5-to-T2 relationship); any later Production-proven entry follows the same additive, non-destructive pattern.

## 31. Storage/schema/import/export impact

**No new root collection, no schema bump, no migration, no backfill, no storage-key change.** T2A only *reads* the existing `state.tabletopCouponResults` and Library Evidence/Production Settings (read-only, via the new §9 compatibility function); it writes nothing new to persisted storage. The shell's clearance-applied/override state lives in the existing, already-ephemeral, session-only Design-draft mechanism every other template already uses for its own draft fields — no persistence change required. Because nothing new is persisted, existing full backup export, old-backup import, new-backup import, merge, replace, and recovery are **unaffected and require no new test surface** beyond confirming they remain green (they will, since nothing new exists to serialize). Missing promoted evidence, a deleted raw coupon result, a deleted Library material, an archived machine, and legacy evidence without `fitContext` are all handled by the compatibility function simply treating them as "no match" via the same defensive optional-chaining style `tabletopCouponRankMatches` already uses — never a crash, never a silent fabrication. No encryption implementation belongs in T2A, and none is added.

## 32. Fixture organization

Three new, focused, standalone groups, matching T1.5's own proven organizational pattern:

- **`runTabletopRectangularShellGeometryFixtures()`** — exact five-piece topology; panel IDs and roles; requested vs. finished cavity; outer envelope; wall/floor blank dimensions; all four corner phase pairs; all four floor phase pairs; matching counts/widths at the recommended 80×60×25 mm size; terminal webs; exact scale; deterministic SVG; closed unique paths; no NaN/Infinity; no unsupported machining feature; validation boundaries; Finished View geometry; Project handoff; cleanup. Re-invokes `runTabletopAccessoryEngineFixtures` and `runTabletopCornerFloorCouponFixtures` as regression guards.
- **`runTabletopRectangularShellEvidenceFixtures()`** — the Joe-equivalent synthetic Coupon-proven evidence (§ below); compatible-evidence discovery; exact-vs-compatible status; explicit Apply; no auto-apply; editable override; wrong-machine exclusion; wrong-material ranking; thickness-mismatch warning/exclusion; generator-version mismatch exclusion; wrong-joint-system exclusion; incompatible-finger-pattern rejection (beyond the §9 tolerance); ties/conflicts never auto-resolved; no-evidence state; missing/deleted source raw result; legacy evidence without `fitContext`; Project handoff provenance; cleanup. Re-invokes `runTabletopEvidencePromotionFixtures` and `runTabletopEvidenceMatchingFixtures` as regression guards.
- **`runTabletopLayoutFixtures()`** — compact deterministic Role-grouped placement; requested spacing; no AABB overlap; no path overlap; label transforms; bounds; piece order; the no-rotation default; grain/readability disclosure text; unchanged panel geometry before/after placement; **unchanged legacy and T1 coupon SVG pins** (re-asserts the existing 2992-byte/`4f543f95` pin unmodified). Re-invokes `runTabletopCornerFloorCouponFixtures` as the regression guard proving T1's own layout was not touched.

**Joe-equivalent synthetic evidence fixture (design, not seeded into production data):** `measuredThicknessMm:2.88`, preferred candidate clearance `-0.075`, `alternateCandidateId: null` (no tighter alternate — matches the real result exactly, since the preferred value is already the tightest tested), `fitContext.evidenceStage:'coupon-proven'`, `verificationStatus:'tested'`, an exact synthetic `machineId`, a disposable synthetic Library material profile, and a complete `fitContext` (construction identity, thickness fields, preferred candidate, `tighterAlternate: null`). This is built and torn down entirely within the fixture's own disposable in-memory `state`/`localStorage` snapshot-and-restore pattern, exactly as every existing Tabletop fixture already does — it is never written to Joe's real profile.

## 33. Protected outputs

At minimum retained, unmodified: T1 Tabletop coupon **2992 bytes / FNV `4f543f95`**; Dice Tray **1726 / `51a55721`**; alternate Dice Tray **1054 / `41697123`**; Divider Tray **1965 / `a55dda6e`**; Joint Fit Coupon **7764 / `db7ea7e9`**; Finger Box, Gift Box, Sliding-lid Box, Drawer Cabinet, and the wall/base tab coupon fixtures; current T1.5 storage and evidence fixtures. **T2A receives its own new, explicit golden only after this geometry and layout review is approved** — no existing golden is ever updated to hide drift, and changed panel-placement bytes are never treated as merely cosmetic.

## 34. Protected boundaries

Confirmed to remain untouched by this review's recommendations: `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`; localStorage recovery; legacy imports/backups; machines M1/M2/M3 and machine snapshots; Material Tests; Test Grids; Library profiles; existing Production Settings and Evidence (only additively touched by the deferred T2B, never by T2A); T1.5 raw coupon results; promotion history; Project schema (no change, per §26); accounting; Inventory; Pricing; legacy Designs; the current T1 coupon's production SVG bytes; kerf conventions; LightBurn layer colors; filenames; offline/no-network behavior.

## 35. First bounded Codex implementation

The task's own likely T2A MVP list is confirmed **as stated, with no reduction needed**: one new `tabletop-rectangular-shell-prototype` template; inside-dimension form; four walls + finger-jointed floor via the reused `buildBoxModel`; the new `tabletopJointFitCompatible` helper; a compatible-Coupon-proven recommendation card; explicit Apply; an editable current value; the new Role-grouped compact layout; an explicit no-rotation default; a screen-only assembled Finished View; the result summary in §24; the Project handoff in §26; the three fixture groups in §32; Help/README/CHANGELOG updates; no storage migration; no shell-result recording (deferred to T2B per §30).

## 36. Locked-decision table

| Decision | Locked value |
| --- | --- |
| Template name | Tabletop Rectangular Shell Prototype |
| Template ID | `tabletop-rectangular-shell-prototype` |
| Prototype dimensions | 80 × 60 × 25 mm interior |
| Configurable ranges | Width/depth 40–200 mm; wall height 15–100 mm; further gated by existing manufacturing-minimum checks |
| Dimension mode | Interior-primary |
| Floor relation | Finished cavity = requested interior absent disclosed intrusions |
| Wall-height meaning | Interior usable wall height; outer height = wall height + material thickness |
| Piece topology | Five pieces: one floor, four walls |
| Panel IDs | `FLOOR`, `FRONT`, `BACK`, `LEFT`, `RIGHT` |
| Wall ordering | Fixed: FLOOR, FRONT, BACK, LEFT, RIGHT (matches `buildBoxModel`'s own definition order) |
| Corner joint | `finger-90`, four corners, reusing existing box-model phase assignment unmodified |
| Floor joint | `finger-jointed-floor`, four edges, reused unmodified |
| Clearance field | One shared signed value for both corners and floor joints |
| Kerf behavior | Separate, informational, never auto-subtracted |
| Evidence-compatibility model | Option B — new, separate `tabletopJointFitCompatibility`; `constructionIdentity`/`tabletopCouponConstructionMatches` unchanged |
| Exact-vs-compatible terminology | "Exact" = passes unmodified `tabletopCouponConstructionMatches`; "Compatible" = passes new gate; UI always says "Compatible Coupon-proven evidence — Not an exact full-shell validation" |
| Machine matching | Exact requirement for compatible classification (reused `productionMachineIdentityMatches`, unmodified) |
| Material matching | Ranking tier only, never a hard exclusion (reused `tabletopCouponMaterialMatches`, unmodified) |
| Thickness matching | Tolerance tier (reused `tabletopCouponThicknessTier`, unmodified) |
| Finger-pattern compatibility | Count irrelevant; actual width per axis within `max(15% of actual width, minWeb/2)` |
| Generator-version behavior | Exact-match requirement; a version mismatch excludes the evidence from "compatible" |
| Recommendation ranking | Reuses `tabletopCouponRankMatches`'s tie/no-averaging behavior with the new compatibility gate substituted for the exact-identity gate |
| Apply behavior | Explicit button only; fills the current draft's clearance field only; never automatic |
| Override behavior | Field remains editable after Apply; override status shown in both the result summary and Project handoff notes |
| No-evidence behavior | Ordinary default clearance, "Run a new coupon" link, no card, no error |
| Conflicting-evidence behavior | All shown, ranked, never averaged or auto-selected |
| Project handoff provenance | No schema change; existing free-text handoff notes carry dimensions, clearance, evidence-applied status, stage, source ID as plain text, machine/material snapshot, measured thickness, override status, prototype caveat |
| Compact layout strategy | Role grouped: `[['FLOOR'],['FRONT','BACK'],['LEFT','RIGHT']]` fed into the existing, unmodified `layoutDesignPanelRows` |
| Alternate layout strategy | None implemented in T2A; general-purpose nesting explicitly out of scope |
| Spacing rule | User-configurable panel spacing, enforced by the existing, unmodified `layoutDesignPanelRows` overlap/spacing checks |
| Rotation policy | No rotation in T2A |
| Label policy | Labels attach to placed panels via the existing `buildAssemblyLabelPaths`, unchanged |
| Finished View | One assembled-only screen-only view; no exploded mode |
| Physical-test procedure | The ten-step procedure in §27, with the stated assembly order |
| Pass criteria | §28 pass list |
| Fallback to −0.050 mm | Triggered by any §28 failure condition |
| Shell-result recording phase | Deferred to a separate T2B phase; not implemented in T2A |
| Storage/schema behavior | No new collection, no schema bump, no migration, no backfill, no storage-key change; read-only evidence reuse; ephemeral Design-draft state |
| Fixture groups | Exactly three: geometry, evidence, layout (§32), each guarding its nearest existing dependency |
| Production golden strategy | T2A receives its own new golden only after this review is approved; all existing goldens remain unchanged |
| First Codex scope | As listed in §35, unreduced |
| Deferred features | Shell-result recording (T2B), recommendation-card reuse beyond this shell, rotation toggle, exploded view |
| Non-goals | Hex Dice Tray, lid, divider, insert, foam/token/card features, hardware, bevels/grooves/rebates/pockets/dadoes, CNC machining, general-purpose auto-nesting, silent rotation, automatic evidence application, universal defaults, Shell-proven/Production-proven promotion, encrypted backups, school edition, network/cloud features, new major tab, Material Tests redesign, legacy migration, T1 coupon geometry changes, production-ready/sellable claims |

## 37. Findings by severity

**BLOCKER TO IMPLEMENTATION:** none. The core risk this review exists to resolve — exact-identity evidence rejection — is fully resolved by the new, separate, bounded compatibility model in §6–§10, grounded in real computed geometry rather than assumption.

**IMPORTANT:** none unresolved. All fit/provenance/Project-handoff/physical-validation decisions the task required are locked in §36 with exactly one answer per row.

**POLISH:** exact wording of the "Compatible Coupon-proven evidence" card copy; whether the result summary orders metrics identically to the coupon's existing summary for visual familiarity.

**NOT A DEFECT (intentionally deferred or excluded):** shell-result recording (T2B); Hex product; premium/sellable product features; any claim of Shell-proven or Production-proven confidence prior to physical testing; exploded Finished View; rotation toggle; general-purpose nesting.

## 38. Exact final verdict

**READY FOR CODEX T2A IMPLEMENTATION**

## 39. Remaining user decision, if any

None blocking. The 80×60×25 mm prototype size and the 15%/minWeb-floor compatibility tolerance are both principled, single recommendations Joe may adjust later without altering the architecture (e.g., choosing a different but still-rectangular size within the locked configurable ranges).

## 40. Whether Claude is needed again

Yes, once, after implementation — a focused implementation audit matching this project's established cadence, specifically re-verifying: the new compatibility gate never silently exact-matches, the exact identity path (`tabletopCouponConstructionMatches`) remains byte-for-byte unchanged, no rotation code was introduced, the Role-grouped layout produces zero AABB/path overlap at the recommended size, and the T1 coupon's existing 2992-byte/`4f543f95` golden is unchanged.

## 41. Whether Grok should audit implementation

Yes, as the standard second-pass adversarial check, focused specifically on the finger-pattern compatibility tolerance (§9) and the exact/compatible terminology boundary (§7), since an independent reviewer confirming those same conclusions materially increases confidence before Joe applies evidence-guided clearance to a real physical shell cut.

## 42. Confirmation

No product source file, README, CHANGELOG, fixture, storage, application record, or existing report was edited during this review. Nothing was staged, committed, or pushed. All geometry computations in this report were performed analytically against the actual, currently committed source (`buildFingerPattern`, `buildBoxModel`, `tabletopAccessoryDimensions`, `tabletopAccessoryManufacturingLimits`) rather than assumed. Only this report file was written.
