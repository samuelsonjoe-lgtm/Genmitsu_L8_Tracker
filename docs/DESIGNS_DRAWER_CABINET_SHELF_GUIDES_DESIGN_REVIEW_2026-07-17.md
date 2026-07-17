# Drawer Cabinet Shelf-Position Score Guides — Focused Design Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-17
**Review type:** read-only design and architecture review. No file was edited, staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## 0. Repository and verification state

- `git status -sb`: `## main...origin/main`, tracked working tree **clean** (no modified tracked files). A long-standing, pre-existing set of untracked files (`LightBurn Projects/`, `debug.log`, numerous prior `docs/*.md` reports, `parametric_qr_stand_generator.py`) is present and was **not** touched.
- `git log -1 --oneline`: **`a36d7ea Add wall-to-base tab clearance coupon`** — this matches the baseline stated in the task exactly; the committed HEAD was independently confirmed rather than assumed.
- `git diff --stat` (all tracked files) and `git diff --stat -- index.html README.md`: both empty. There is no working-tree diff to audit; this review evaluates a **not-yet-implemented** feature against the current committed source only.

## 1. Files and functions inspected

- `README.md` — current Designs paragraph, fixture-total paragraph (887 Designs geometry / 1679 complete suite / 248 Tray-model), Finished View paragraph, promotion-deferral paragraph confirming Drawer Cabinet fit promotion is already deferred.
- `docs/DESIGNS_DRAWER_CABINET_ARCHITECTURE_REVIEW_2026-07-16.md` — full read. Establishes the cabinet's phased roadmap; §4.3, §9.3, and §9.4 specifically anticipate a Phase 5.3 "placement guides and assembly labels" feature and explicitly flag Right-panel handing as a requirement for it.
- `docs/DRAWER_CABINET_PHASE5_1_IMPLEMENTATION_2026-07-16.md`, `docs/DRAWER_CABINET_FINISHED_FRONT_VIEW_IMPLEMENTATION_2026-07-16.md`, `docs/DESIGNS_PHASE4_1_HIDDEN_ASSEMBLY_LABELS_IMPLEMENTATION_2026-07-15.md` — full read, as the two closest proven precedents (cabinet geometry itself, and the existing blue-score assembly-label mechanism).
- `index.html` — read and traced directly, not summarized from prior reports:
  - `drawerCabinetDimensions()` (`:2354`), `namespaceDesignPanel()` (`:2367`), `buildDrawerCabinetModel()` (`:2370-2413`), `buildDrawerCabinetFrontElevation()` (`:2414-2447`), `buildDrawerCabinetDesignResult()` (`:3111-3132`), `buildDrawerCabinetFrontElevationSvg()` (`:3180-3191`), `drawerCabinetFrontLegendHtml()` (`:3192-3195`).
  - `mirrorDesignPanel()` (`:2631-2634`), `designScoreSegmentsPath()` (`:2635-2641`), `designAssemblyLabelSpecs()` (`:2693-2718`), `designAssemblyLabelGeometry()` (`:2719-2727`), `designMinimumSegmentClearance()` (`:2728-2731`), `buildAssemblyLabelPaths()` (`:2732-2763`), `designSerializedPathGroup()` (`:2764-2766`), `slidingRailGuideSegments()` (`:2767-2778`), `serializeDesignSvg()` (`:2877-2883`), `designSerializedAssemblyLabelValidation()` (`:2954` region).
  - `buildSlidingLidDesignResult()` (`:3019-3061`), specifically the `rightPanelMustBeHanded` / `mirrorDesignPanel` / `slidingRailGuideSegments` / `scorePaths` block (`:3023-3041`), used as the only existing precedent for a handed panel carrying blue-score guide marks.
  - `buildBoxModel()` (`:2240-2276`) and `buildDrawerCabinetModel()`'s side-panel construction (`:2399-2401`), read specifically to compare their edge-pattern symmetry (see §5 below).
  - `layoutDesignPanelRows()` (`:2855-2874`), `designBoxesOverlap()`/`designBoxesSeparatedBy()` (`:2850-2854`).
  - `renderDesigns()` cabinet field block (`:1835-1846`) and the `drawer-cabinet` branch of `normalizeDesignDraft()` (`:2127-2139`), read to identify the exact insertion point and naming convention for the new field.
  - `designResultsHtml()` cabinet metrics block (`:3243-3258`) and the sliding-lid guide-marks metric line (`:3275`), read for wording precedent.

No file was edited during this review.

## 2. Executive summary

The feature is buildable as a small, self-contained addition entirely inside `buildDrawerCabinetDesignResult()` (plus one small, cabinet-owned helper and one new session-draft field), reusing the same layered contract the Sliding-Lid Box already established for its blue-score rail-placement guides (`normalizeDesignDraft()` → boolean field → guide-segment helper → `scorePaths` → `serializeDesignSvg()`). It does **not** require a capability registry, a general joint-engine refactor, or any change to `buildDrawerCabinetModel()`'s dimension or joint math — the shelf elevations it needs (`model.metrics.shelfBottomElevations`) already exist and require no new formula.

One real architectural hazard exists and must be resolved deliberately, not by copying the Sliding-Lid precedent verbatim: Sliding-Lid resolves left/right handing by mirroring the **cut geometry** of the Right panel (`mirrorDesignPanel`), which is the correct technique for that feature but would violate this task's explicit "red cut geometry must not change" constraint if reused as-is. §5 below shows that this can be avoided entirely for shelf guides by choosing tick geometry that is inherently invariant to the same left/right mirror transform, so no cut-panel mirroring, no new "handed panel" concept, and no red-geometry change are needed anywhere in this feature.

A second, smaller but real hazard is a **vertical (top/bottom) orientation ambiguity** that is already latent in the cabinet's existing panel geometry (not introduced by this feature) — see §5.3. It is resolvable in software with one explicit, documented, fixture-verified convention, but it is the one part of this feature that a physical prototype must specifically confirm before the marks are trusted.

## 3. Current production contract (baseline, before this feature)

Traced directly from `buildDrawerCabinetDesignResult()` (`index.html:3111-3132`) and `serializeDesignSvg()` (`index.html:2877-2883`):

- The cabinet SVG is built by: `layoutDesignPanelRows()` over 5 shell panels + `shelfCount` shelves + `5 × rows` drawer panels, in the fixed row order `[[cabinet-bottom,cabinet-top],[cabinet-left,cabinet-right,cabinet-back],[shelves...],[drawer rows...]]`.
- `buildDrawerCabinetDesignResult()` currently constructs `partial = {widthMm, heightMm, panels: layout.panels, labelPaths: labelResult.paths}` and calls `serializeDesignSvg(partial)` — **`scorePaths` is never set for this template today.** This means the cabinet's existing blue `<g id="score">` group, when it appears at all, currently contains only an `assembly-labels` subgroup, never a `rail-guides` subgroup — the cabinet has never emitted the "rail-guides" bucket that `serializeDesignSvg` already supports generically.
- `serializeDesignSvg()`'s score group is emitted only when `scorePaths.length || labelPaths.length`; when both are empty (the current default: `assemblyLabels` off), **no `<g id="score">` element is emitted at all**, and the SVG is exactly the red `<g id="cut">` group plus the outer `<svg>`/XML wrapper.
- Filename/MIME/preview/download identity, storage isolation, and draft-only session state for the cabinet are unchanged by anything in this review and were confirmed present (not re-derived) via the same download/backup fixture pattern already used by every other Designs template.
- `cabinet-left` and `cabinet-right` are generated by the exact same `side(id, name)` closure (`index.html:2399-2400`) with identical `width`, `height`, and `edges` arguments — **their cut geometry is byte-identical apart from `id`/`name`**, and neither is currently mirrored anywhere in the cabinet path (confirmed: `buildDrawerCabinetDesignResult` never calls `mirrorDesignPanel`, unlike `buildSlidingLidDesignResult`).

## 4. Recommended terminology

| Concept | Recommendation | Rationale |
|---|---|---|
| User-facing option name | **"Shelf-placement guides"** | Matches the existing established vocabulary exactly: the app already calls its one other guide feature "Rail-placement guide marks" (`index.html:3275`, and the Sliding-Lid help text at `:1830`). Reusing the same "`<noun>`-placement guide(s)" pattern keeps the vocabulary consistent across templates rather than introducing a third phrase. |
| Rejected: "shelf-position guides" | Not recommended | Serviceable but breaks the established "-placement" naming already in the UI; no functional reason to diverge. |
| Rejected: "shelf registration guides" | Not recommended | "Registration" is a precise woodworking term but reads as more technical/mechanical than the feature is; risks implying a locating **function** rather than a visual aid, which cuts against the explicit constraint to avoid implying structural/mechanical behavior. |
| Internal session-draft (raw) field | **`drawerShelfGuides`** | Follows the existing cabinet-specific raw-field convention (`drawerRows`, `drawerJointClearance`, `drawerLateralClearance`, …) and the Sliding-Lid precedent of a template-prefixed raw key (`slidingGuideMarks`) rather than the shared unprefixed `assemblyLabels` key, since this option's meaning is cabinet-specific. |
| Internal normalized/metrics field | **`shelfGuides`** (boolean) | Mirrors the normalized name `guideMarks` used for the Sliding-Lid feature; keeps `result.metrics.shelfGuides` self-describing in `designResultsHtml()` and fixtures. |
| New score subgroup id | **`shelf-guides`** | Distinct from the existing hardcoded `rail-guides` subgroup id (see §7) so the SVG accurately names what the marks are, without touching Sliding-Lid's existing output. |

Wording to avoid, confirmed against the actual generated content plan in §6: "dado," "groove," "slot," "locking," "captured," "structural support," "strengthens," or any phrase implying the score line participates in load transfer. The marks are a **positioning aid only**; the shelf is still edge-glued exactly as it is today, with the same "prototype shelf alignment and glue strength" warning `buildDrawerCabinetModel()` already emits (`index.html:2392`) staying in place unchanged.

## 5. Geometry analysis (answers to the 12 required questions)

### 5.1 Which panels receive guides

**Cabinet-left, cabinet-right, and cabinet-back.** Not cabinet-top, cabinet-bottom, or any drawer panel.

Reasoning: each support shelf (`designPanelFromPoints(... width:cabinetInsideWidth, height:cabinetInsideDepth ...)`, `index.html:2402`) is a flat rectangle sized to the cabinet's full interior footprint and is edge-glued directly to the interior faces of left, right, and back — those are the only three panels the shelf's edge actually contacts. Cabinet-top and cabinet-bottom never touch a shelf edge in this construction (Phase 5.1/5.2 uses no vertical dividers), so a guide there would have no physical referent.

### 5.2 Which face, and is it normally concealed

The **interior face** — the same face `cabinet-shelf-rNN` sits against. Once a shelf (and later, the drawers that ride above/below it) are glued in and the cabinet is in normal use, this face is visible only by looking directly into the open front with the relevant drawer removed — the same concealment level the existing Hidden Assembly Labels feature already relies on for `LEFT`/`RIGHT`/`BACK` text (`docs/DESIGNS_PHASE4_1_HIDDEN_ASSEMBLY_LABELS_IMPLEMENTATION_2026-07-15.md`). Using short, inset tick marks rather than a full continuous line (see §5.4) further reduces the amount of interior surface actually marked, minimizing any chance of a mark peeking past a shelf's front edge or being visible from the open front.

### 5.3 Does the existing layout encode panel handing strongly enough?

**Not on its own — this is the central design hazard, and it has two independent components.**

**(a) Left/right (front-to-back) axis.** `cabinet-left` and `cabinet-right` are generated by the identical `side()` closure and are byte-identical cut shapes (§3). Comparing their edge construction against `buildBoxModel()`'s Left/Right panels (`index.html:2268-2269`) is instructive:

- Finger Box Left/Right: `edges:[plain, edge(verticalPattern,false), edge(depthPattern,true), edge(verticalPattern,false)]` — the **front and back edges use the identical pattern and phase** (`verticalPattern,false` on both), so the panel is symmetric under the left/right mirror; the one asymmetric (`plain`) edge is the **open top**, a top/bottom asymmetry, not a left/right one. This is why Finger Box has never needed `mirrorDesignPanel` for its Left/Right labels — mirroring a self-symmetric shape is a no-op.
- Drawer Cabinet and Sliding-Lid Left/Right: the **plain (open-front) edge sits at local `x=0`**, opposite a genuinely different, jointed edge at local `x=width`. This shape is **not** symmetric under the left/right mirror — mirroring it swaps which local edge carries the finger teeth. This is exactly the asymmetry `slidingRailGuideSegments` + `mirrorDesignPanel` + `rightPanelMustBeHanded` were built to solve (`index.html:3023-3041`), and it is exactly the asymmetry `docs/DESIGNS_DRAWER_CABINET_ARCHITECTURE_REVIEW_2026-07-16.md` §9.4 anticipated for this cabinet: *"The Right cabinet panel must be handed when physical guides or concealed-face labels require a specific inward face."*

So: the existing layout genuinely does **not** encode this on its own, and the Sliding-Lid precedent's own fix (mirroring the Right panel's cut geometry) is the wrong tool here, because it changes red cut bytes whenever the option is enabled — directly forbidden by this task's constraints and by fixture-plan item 2. **§5.4 shows a geometry choice that sidesteps this entirely, without any cut-panel mirroring.**

**(b) Top/bottom axis.** A second, independent ambiguity exists and is **not solvable by geometry alone**: `cabinet-left`'s top edge (`y=0`) and bottom edge (`y=height`) both use `cabinetDepthPattern,true` (`index.html:2399`) — the identical pattern and phase on both — and `cabinet-top`/`cabinet-bottom` are themselves generated by identical edge sets to each other (`index.html:2397-2398`). Nothing in the finger-pattern math distinguishes "this local edge mates to the physical top" from "this local edge mates to the physical bottom" — that distinction is pure assembly convention today (established only by which physical piece the builder chooses to install where, optionally reinforced by the `TOP`/`BOTTOM` assembly labels when enabled). This ambiguity already exists in the shipped cabinet; this feature is simply the first one whose correctness depends on resolving it, because a shelf-elevation mark is only useful if "up" in the mark's local coordinate frame matches "up" in the physically assembled cabinet.

**Recommendation:** adopt one explicit, documented convention — *local `y=0` is the cabinet's physical top* (matching the existing mental model already used by `buildDrawerCabinetFrontElevation`'s screen-space math, independently re-derived for the raw panel rather than by calling that function) — verify it with a hand-derived fixture for a concrete numeric case (§8), and require the physical prototype to specifically confirm the mark lands at the correct real-world height before treating it as reliable (§9). This is a **Major**, not Blocker, risk: getting it backward produces a cosmetically wrong (but harmless — it doesn't touch cut geometry) mark, not a structural failure, and is fully containable by explicit wording plus one prototype check.

### 5.4 Guide pattern: recommend short endpoint tick marks, not a continuous line

Rejecting the other three options in the prompt for concrete, source-grounded reasons:

- **Two parallel lines for shelf thickness**: doubles the marked area and the validation/containment surface for no assembly benefit — a single elevation is sufficient to align a shelf that already has a known, fixed thickness.
- **One continuous horizontal line spanning the full interior depth/width**: this is the version most exposed to the physical risk explicitly raised in the task (§5.8) — a continuous score mark directly under the entire length of a glued joint chars/roughens the surface exactly where glue needs to bond, for the full contact length.
- **Corner marks** (L-shaped, like `slidingRailGuideSegments`): reasonable, but a shelf's registration only needs one datum (its elevation) at two points (front end + back end), not a full corner outline; a single short straight tick per end is simpler and covers less surface.
- **Recommended — short endpoint ticks**: two short straight horizontal segments per panel per shelf, one inset from the panel's front-ish local edge, one inset from the back-ish local edge, both at the shelf's elevation. This marks only the two ends of the glue line, leaving the interior span between them completely unmarked (full original surface, no char), while still giving the builder two physical reference points — enough to align the shelf square and level.

Placing a tick at **both** local ends (rather than only one) is also what makes §5.3(a)'s left/right hazard moot without any mirroring: whichever local edge (`x≈0` or `x≈width`) turns out to be "front" versus "back" for a given panel's actual installed orientation, a correctly elevated tick already exists at both ends, generated from the exact same unmirrored coordinate formula on both `cabinet-left` and `cabinet-right`. The two-tick pattern is therefore invariant to the very mirror transform (`x → width − x`) that made Sliding-Lid's Right panel require special handling — no `mirrorDesignPanel` call, no `rightPanelMustBeHanded`-style flag, and no red cut geometry change are needed anywhere in this feature.

### 5.5 Span: stop short of exposed/jointed edges, aligned to the real glue-contact ends

Each tick is **inset** from the panel's local edges by a fixed clearance (recommend reusing the same order-of-magnude margin `slidingRailGuideSegments` already uses, e.g. `inset = Math.max(2, materialThickness)`), so it never overlaps the plain front edge, the finger teeth at the jointed edges, or a panel corner. This is validated the same way rail guides and labels already are, via the existing `designSegmentsContainedInPolygon()`/`designMinimumSegmentClearance()` helpers (§7) — no new containment algorithm is needed.

### 5.6 Back panel: yes, using the same two-tick pattern

`cabinet-back` is jointed on all four local edges (no plain edge), and its left/right (local x) edges use the identical pattern+phase (`cabinetHeightPattern,false` on both, `index.html:2401`) — the same left/right interchangeability as §5.3(a), resolved the same way: one tick inset from each local x-edge, at the shelf's elevation. This gives the shelf three-sided registration (left, right, back) during glue-up, matching how it is actually edge-glued.

### 5.7 Visibility on the finished cabinet

Not expected to be visible in normal use, for the same reason as §5.2, reinforced by the short-tick geometry marking only a small fraction of the interior surface near each panel's side edges rather than a full span.

### 5.8 Weakening plywood / soot / interference with the glue joint

This is the specific reason §5.4 rejects a continuous line. Two short ticks, together spanning only a few times the material thickness, leave the great majority of the shelf's actual glue-contact length completely unscored. The existing warning already emitted by `buildDrawerCabinetModel()` when `rows>1` ("Upper drawers use plain full-width edge-glued support shelves. Prototype shelf alignment and glue strength before production use.", `index.html:2392`) already covers glue-strength verification generally and needs no change; the new option's own help text (§6) should additionally note that the marks are placement aids only and do not replace that prototype step.

### 5.9 Reusing existing model data, no duplicated formula

`model.metrics.shelfBottomElevations` (`index.html:2408`, `shelfBottomElevations=Array.from({length:shelves.length},(_,index)=>designRound((index+1)*dimensions.cellHeight+index*t))`) already exists and is exactly the value needed — one entry per shelf, in the same order as `cabinet-shelf-rNN`. The guide feature should **read this array directly**; it must not recompute cell height, row pitch, or elevation with a second formula anywhere.

### 5.10 Deriving all coordinates from the same production geometry

Yes: the guide-generating code should take its `x`/`y` inputs only from (a) `model.dimensions` (for panel width/height and material thickness) and (b) `model.metrics.shelfBottomElevations` — the same values `buildDrawerCabinetDesignResult()` already has in scope after calling `buildDrawerCabinetModel()`. It must **not** read from `model.frontElevation` (which is explicitly a separate, screen-only projection — reusing it here would violate "Finished View behavior must remain production-independent" in the opposite direction, making production depend on a screen-only artifact) and must not read from any display/layout coordinate (`layout.panels[...].x/y` are sheet-placement offsets applied only after the guide's own panel-local coordinates are computed, exactly as `slidingRailGuideSegments`' output is translated by `placedById[...].x/y` only at the very end, `index.html:3052`).

### 5.11 Is a new reusable/shared helper justified?

**No — keep it cabinet-owned.** A small, local helper (recommend `drawerCabinetShelfGuideSegments(dimensions, thickness, shelfBottomElevation)`, structurally similar in *style* to `slidingRailGuideSegments` but not shared with it) is justified and sufficient. This deliberately does **not** generalize into a shared cross-template "guide" abstraction: the architecture review already recommended against a joint/guide capability registry until a second physically-validated guide family exists (`docs/DESIGNS_DRAWER_CABINET_ARCHITECTURE_REVIEW_2026-07-16.md` §8), and this task's own constraints explicitly forbid one. A small amount of structural duplication between this helper and `slidingRailGuideSegments` is the correct trade here, not a shared framework.

### 5.12 Finished View: no geometry change, one optional text note

**Recommend no change to the rendered geometry of `buildDrawerCabinetFrontElevationSvg()`.** The Finished Front View already shows each shelf as a distinct band with a "Shelf N" label (`index.html:3188`) — it already communicates shelf position without needing the raw guide marks, and per the explicit constraint, production and Finished View must stay independent. The one optional, low-risk addition — directly mirroring an existing, already-shipped precedent — is a one-line informational text note, exactly like `buildSlidingLidFinishedViewSvg()`'s existing `guideNote=result.metrics.guideMarks?'Rail-placement guides enabled (Cut Layout score marks).':''` (`index.html:3177`). A parallel `result.metrics.shelfGuides` boolean check and a static string (e.g. `"Shelf-placement guides enabled (Cut Layout score marks)."`) would cost nothing in production-independence, since it reads only the same boolean metric already computed for the results panel, not any guide geometry. This is optional, not required, for a first commit.

## 6. Proposed UI

Insert the new control in `renderDesigns()`'s `cabinetFields` (`index.html:1845-1846`), directly **after** the existing `assemblyLabels` checkbox/help-text pair and before the closing of `cabinetFields` — grouping the two blue-score options together, matching how Sliding-Lid groups its guide checkbox immediately before its own label checkbox (`index.html:1830-1832`). It does not belong next to `drawerRows`, since it is a downstream consequence of row count (only relevant once `rows>1`), not a dimension input itself.

```html
<label class="row"><input name="drawerShelfGuides" type="checkbox" style="width:auto" ...> Add shelf-placement guides (blue score layer)</label>
<div class="small muted" style="grid-column:1/-1">Adds a short positioning mark near each end of every upper shelf's glue line, on the Left, Right, and Back interior faces at the shelf's actual elevation. These are placement aids only, not a structural joint; shelves still require edge glue and the existing shelf-prototype warning. Run the blue score layer before cutting, and confirm the marked face and elevation on a scrap prototype before gluing.</div>
```

Behavior:
- **Default off**, matching `assemblyLabels`/`guideMarks`.
- Toggling it, like `assemblyLabels`/`guideMarks`/`fitCoupon` today, should trigger a full `render()` (not just a preview refresh) since — like labels — it changes which score subgroup(s) can appear and changes `result.metrics`/`designResultsHtml()` output, consistent with the existing live-field trigger list already handling `assemblyLabels` the same way.
- If `rows === 1` (so `shelfCount === 0`) and the box is checked, add a warning (not an error) — e.g. *"No upper shelves exist at 1 row; shelf-placement guides have nothing to mark."* — rather than silently doing nothing, consistent with how the app generally surfaces no-op option states rather than staying silent.
- Switching `template` away from `drawer-cabinet` and back, or changing `drawerRows`, does not need any special reset logic beyond what already exists: `drawerShelfGuides` simply stays in `designDraft` (same as `assemblyLabels` does today) and the guide count naturally recomputes from the current `shelfBottomElevations` length on every rebuild — there is no stale per-shelf state to invalidate, because guides are derived fresh from `model.metrics` on every `buildDrawerCabinetDesignResult()` call rather than cached.

`designResultsHtml()`'s cabinet metrics block (`index.html:3243-3258`) should gain one line, directly after the existing `Assembly labels` metric (`:3256`), following the exact existing phrasing pattern: `designMetric('Shelf-placement guides', result.metrics.shelfGuides ? \`Enabled - ${result.metrics.shelfGuideCount || 0} marks\` : 'Disabled')`.

## 7. Normalization and compatibility

- **Default when absent from older drafts**: `false` — identical pattern to `assemblyLabels`/`guideMarks`: `values.shelfGuides = draft.drawerShelfGuides === true || draft.drawerShelfGuides === 'true' || draft.drawerShelfGuides === 'on';`, added to the `drawer-cabinet` branch of `normalizeDesignDraft()` (`index.html:2127-2139`) alongside the existing `values.assemblyLabels` line.
- **Accepted values**: boolean-equivalent only (`true`/`'true'`/`'on'`), same as every other checkbox field in this codebase. A boolean is sufficient; no enum or numeric variant is needed for a first phase.
- **Malformed values**: any other value normalizes to `false` (no error raised), identical to how `assemblyLabels` already behaves — consistent, not a new validation posture.
- **Session-only `designDraft` handling is sufficient.** No new field belongs in `state`, `freshState()`, `loadState()`, `persist()`, `backupObject()`, or import/export — this exactly mirrors the already-confirmed boundary for every other Designs option (§12 of the architecture review, reconfirmed directly by reading `buildDrawerCabinetDesignResult()` and finding no reference to `state`/`persist`/`backupObject` anywhere in the cabinet path).
- **No storage migration is needed.** Nothing in `STORAGE_KEY`, `SCHEMA_VERSION`, or any persisted record changes; this was confirmed by source, not assumed — the entire feature's state lives in one new key on the existing session-only `designDraft` object, exactly like `assemblyLabels`.

## 8. Production-output contract

- **Current contract (unchanged parts)**: as described in §3. When `drawerShelfGuides` is absent, `false`, or any falsy-equivalent value, `buildDrawerCabinetDesignResult()` must take **exactly** the same code path it takes today (no new function call executes, `scorePaths` stays absent from the `partial` object passed to `serializeDesignSvg`), so the resulting SVG is provably byte-identical to the currently committed golden.
- **Bytes that must remain unchanged when disabled**: every panel's `<g id="panel-...">`/`<path d="...">` in the red `<g id="cut">` group (unconditionally, regardless of the new option's state — see next point), plus the entire document when the option is off, including the presence/absence and contents of any `assembly-labels` score subgroup exactly as they exist today.
- **Bytes that must remain unchanged even when enabled**: the entire `<g id="cut">` group. This feature reuses the existing `layout.panels` untouched — no panel's cut points are mirrored, translated, or re-derived, unlike Sliding-Lid's Right-panel handling (§5.3/§5.4). This is the one deliberate divergence from the Sliding-Lid precedent, and it is required by this task's explicit constraint.
- **New blue score elements when enabled**: one `scorePaths` entry per (panel × shelf) combination — up to `3 × shelfCount` entries (e.g. 3 for a 2-row cabinet, 6 for a 3-row cabinet), each containing 2 short line segments (the two end ticks). Each entry gets the same automatic `id`/`<title>` wrapping every existing `scorePaths`/`labelPaths` entry already gets from `designSerializedPathGroup()` (`index.html:2764-2766`) — **not anonymous**, matching the existing contract for guides and labels alike (only the red cut geometry serialization is anonymous-by-panel-group; score elements have always carried per-item ids/titles in this app). Recommended ids follow the existing `guide-rail-left`-style convention, e.g. `shelf-guide-r01-cabinet-left`, titled e.g. `"Shelf 1 placement guide - Left"`.
- **Required group order**: unchanged outer contract — `score` (if present) before `cut`, exactly as `serializeDesignSvg()` already enforces structurally (score markup is concatenated before the cut group in the template string, `index.html:2882`). Within `score`, keep the existing sub-order `rail-guides` bucket, then `assembly-labels` bucket (`index.html:2880`) — see next point for how the new shelf-guide bucket fits into this without disturbing it.
- **Serializer change required, and why it is the smallest safe option**: `serializeDesignSvg()` currently hardcodes the id `'rail-guides'` for whatever is in `result.scorePaths` (`index.html:2880`), a name inherited from the one existing caller (Sliding-Lid). Feeding cabinet shelf-guides through the same bucket verbatim would mislabel them (`<g id="rail-guides">` on a cabinet with no rails). The recommended fix is a **single additive, default-preserving change**: `designSerializedPathGroup(scorePaths, result.scoreGroupId || 'rail-guides')`. No existing caller sets `scoreGroupId`, so Sliding-Lid's output — including its already-shipped, physically-tested golden bytes when guides are enabled — is provably unchanged. `buildDrawerCabinetDesignResult()` is the only new caller that would set `scoreGroupId:'shelf-guides'`. This is a one-line, purely additive change with no refactor and no risk to any other template's output; it does not touch `layout`, `panels`, `labelPaths`, or the `cut` group in any way.
- **Preview and downloaded SVG must remain byte-identical** — no change needed to enforce this; the cabinet already reuses one `result.svg` for both, exactly like every other template, and this feature adds no new preview-only code path.
- **Normal flat (Cut Layout) preview**: shows the guide marks, since they are part of `result.svg` exactly like assembly labels already are.
- **Finished View**: does not show the guide geometry (§5.12) — only an optional text note referencing that guides exist in Cut Layout.
- **Filename and MIME**: unchanged. The template key stays `drawer-cabinet` regardless of `shelfGuides`; no new branch is needed in the filename generator.

## 9. Fixture plan (representative, not a combinatorial matrix)

Recommend roughly 12–16 new `runDesignGeometryFixtures()` assertions, sized similarly to the Wall-to-base Tab Clearance Coupon's 11-assertion addition (`docs/DESIGNS_WALL_BASE_TAB_COUPON_IMPLEMENTATION_2026-07-17.md`) rather than a full per-row-per-panel combinatorial grid:

1. **Disabled byte identity** — assert the current committed golden SVG (1-row, 2-row, and 3-row default cabinets) is unchanged both when `drawerShelfGuides` is absent from the draft and when explicitly `false`.
2. **Red cut path identity when enabled** — build a 2-row cabinet with guides off and on; assert `result.panels.map(p=>p.path).join('|')` is identical between the two (proves §8's "cut group never changes" claim directly, not just by absence of a mirror call).
3. **Score-before-cut ordering** — `svg.indexOf('<g id="score"') < svg.indexOf('<g id="cut"')` and `svg.includes('<g id="shelf-guides"')` when enabled.
4. **Guide entry count matches shelf count** — for 1-row (0 shelves → 0 entries), 2-row (1 shelf → 3 entries), 3-row (2 shelves → 6 entries).
5. **Elevation hand-derivation, not a restated formula** — for the default 2-row and 3-row cabinets, independently hand-compute the expected `shelfBottomElevations` from the documented cell-height formula (already established and reused verbatim, not recomputed by the guide feature) and assert the guide's local `y` matches `cabinetOutsideHeight − t − shelfBottomElevation − t` for a concretely chosen numeric example, per the documented top-down convention in §5.3(b) — this is the one fixture that specifically nails down the orientation convention rather than trusting it.
6. **Containment** — every tick segment satisfies `designSegmentsContainedInPolygon` against its owning panel's points, for both Left/Right and Back.
7. **Left/right mirror invariance** — directly assert that the tick segments generated for `cabinet-left` and `cabinet-right` at the same shelf row are related by the exact `x → width − x` transform applied to the *set* of two ticks (not per-tick), independently confirming §5.4's "no mirroring needed" claim rather than assuming it.
8. **Back-panel placement** — ticks inset from both local left/right edges of `cabinet-back`, at the same elevation as the corresponding Left/Right ticks for that shelf.
9. **Drawer-count/row-count change updates guides correctly** — regenerate at 1, 2, and 3 rows from the same base draft and assert guide count and elevations track `shelfBottomElevations` exactly, with no stale entries carried over (proves there is no cached per-shelf state).
10. **Finite, non-overlapping with unrelated panels** — every tick coordinate finite; ticks stay within their owning panel's translated bounds after layout and do not intersect any drawer or shelf panel's translated bounds.
11. **Preview/download identity** — reuse the existing intercepted-download fixture helper pattern with `drawerShelfGuides:true`.
12. **Filename/MIME unchanged** — same helper, asserting the existing `l8-drawer-cabinet-...` pattern (or whatever the current literal filename convention is) and MIME are untouched.
13. **Storage/backup isolation** — snapshot `localStorage`/`backupObject()` before and after generating, previewing, and downloading a guide-enabled cabinet; assert byte identity, mirroring the existing pattern used for every other Designs option.
14. **Finished View independence** — assert `buildDrawerCabinetFrontElevationSvg()`'s output for a guide-enabled and guide-disabled cabinet differs by at most the one optional text note (if implemented) and contains no guide-tick geometry.
15. **Existing Drawer Cabinet and complete-suite totals reconciled** — after adding the fixtures above, the Designs geometry total (887 → 887+N) and complete-suite total (1679 → 1679+N) must be recomputed and the README updated to match exactly, following the same static-count-plus-loop-aware reconciliation method used throughout this project's recent audits.

## 10. Physical-workflow assessment

What this feature **would** prove, purely in software: that the guide coordinates are internally consistent with the cabinet's own declared shelf elevations, that they stay within their panel's boundary with the documented clearance, that they do not alter any red cut path, and that they serialize in the correct blue-before-red order with correct ids.

What it explicitly **would not** prove, and must not be worded to imply: actual shelf squareness once glued, glue bond strength, whether the tick marks survive typical sanding/finishing without becoming illegible, long-term shelf load support, plywood warping over time, or that the laser's actual focus/power/speed produces a visible-but-shallow score (as opposed to a through-cut or an invisible mark) on the user's specific material. None of this is new — it is the same category of limitation the existing shelf warning and every other Designs "assembly aid" feature in this app already discloses.

**Physical verification timing**: the left/right mirror-invariance property (§5.4) is a pure geometry claim, fully verifiable in software via fixture 7 above, and does not need physical cutting to trust. The top/bottom orientation convention (§5.3(b)), by contrast, **should** be confirmed on the very first physical prototype specifically built with this option enabled — not deferred to "the next cabinet build" — because it is the one part of this design that depends on an assembly convention not otherwise enforced anywhere in the existing geometry. A wrong convention produces a harmless-but-useless mark (it does not affect the cut, the joints, or the ability to complete the cabinet), so this does not block committing the software, but it does mean the first physical cabinet built with guides enabled is the actual proof, not the fixtures alone.

## 11. Risk register

| # | Severity | Risk | Mitigation already designed in above |
|---|---|---|---|
| 1 | Major | Top/bottom orientation convention (§5.3b) is not derivable from geometry alone; a wrong convention silently places marks at the mirrored elevation | Explicit documented convention + hand-derived fixture (§9 item 5) + mandatory first-prototype confirmation (§10) |
| 2 | Major (avoided, not merely mitigated) | Reusing Sliding-Lid's `mirrorDesignPanel`-on-cut-geometry pattern would change red cut bytes whenever guides are enabled, violating an explicit hard constraint | §5.4's two-tick, mirror-invariant geometry choice avoids this at the design level; no mitigation needed at implementation time because the hazard is designed out |
| 3 | Minor | A continuous full-span guide line could char/rough the surface directly under the shelf's entire glue line | Short endpoint ticks only (§5.4/§5.8) |
| 4 | Minor | `serializeDesignSvg()`'s hardcoded `'rail-guides'` id would mislabel cabinet shelf guides if reused verbatim | One-line additive `scoreGroupId` parameter, default-preserving for every existing caller (§8) |
| 5 | Minor | Checking the option at `rows===1` has nothing to mark and could look like a silent no-op | Explicit warning message, not silence (§6) |
| 6 | Informational | `cabinet-right`'s existing `LEFT`/`RIGHT` assembly labels (already shipped, unrelated to this task) are never mirrored via `mirrorDesignPanel`, unlike Sliding-Lid's equivalent labels — a pre-existing gap noted by the architecture review's own §9.4 guidance, not introduced by and not in scope for this feature | None recommended as part of this task; flagged only so it is not mistaken for a new defect introduced here |
| 7 | Informational | Guide tick count (`3 × shelfCount`) grows the SVG and fixture surface modestly (up to 6 entries at 3 rows) | Well within existing per-template fixture scale (comparable to the 11-assertion Wall-to-base coupon addition); no nesting/matrix growth |

No Blocker-severity risk was found: nothing in this design requires touching storage, promotion, evidence, Production Settings, Inventory/Library/Project/Pricing/Test Grid/Material Test, import/export, sheet nesting, kerf compensation, or any other template's output, and the one genuine hazard identified (§5.3a, red-cut-geometry-safe handing) is designed out rather than merely mitigated.

## 12. Protected boundaries (confirmed unaffected by this design)

`STORAGE_KEY`, `SCHEMA_VERSION`, `persist()`, `backupObject()`, `replaceData()`, `mergeData()`, `normalizeProductionEvidence()`, `normalizeProductionSetting(s)`, `productionSettingDesignApplicability()`, every promotion function, `buildDrawerCabinetModel()`'s dimension/joint/finger-pattern math, `buildBoxModel()` (the drawer's own model), `layoutDesignPanelRows()`, `mirrorDesignPanel()` (referenced for comparison only, not called by this feature), Finished Views for every other template, QR stand, Hanging sign, Dice/Divider Tray, Finger Box, Sliding-Lid Box (including its own existing guide/label output), Joint Fit Coupon (both modes), and all non-Designs subsystems (Library, Log, Test Grid, Project, Inventory, Pricing) — none of these require or receive any change under this design. The only files touched by an eventual implementation would be `index.html` (one new normalized field, one small cabinet-owned helper, a small addition inside `buildDrawerCabinetDesignResult()`, one additive parameter on `serializeDesignSvg()`, one new UI field/metric line) and `README.md`/fixture totals.

## Final verdict

**READY FOR IMPLEMENTATION**

The feature has a clear, minimal implementation path that reuses `model.metrics.shelfBottomElevations` without a duplicated formula, reuses the existing containment/serialization helpers without a new abstraction, and — critically — resolves its one real geometric hazard (left/right handing) by choosing mirror-invariant guide geometry rather than by copying Sliding-Lid's cut-geometry-mirroring technique, which this task's constraints correctly forbid. The remaining open item (§5.3b top/bottom convention) is a normal, boundable design decision requiring one explicit convention, one hand-derived fixture, and one specific physical-prototype confirmation — not a reason to withhold implementation.
