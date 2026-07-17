# Dice Tray Underside Cover Plate ("Layered Slot-Hiding Base") — Focused Design Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-17
**Review type:** read-only design and architecture review. No file was edited, staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## 0. Repository and verification state

- `git status -sb`: `## main...origin/main`, tracked working tree **clean**.
- `git log -1 --oneline`: **`c064896 Add drawer cabinet shelf-placement guides`** — matches the stated expected baseline exactly.
- `git diff --check`, `git diff --stat`, `git diff`: all empty — no tracked-file diff exists to audit; this is a forward-looking design review against committed source only.
- `git ls-files --others --exclude-standard`: the same long-standing untracked set as every prior session in this project (`LightBurn Projects/*`, `debug.log`, `parametric_qr_stand_generator.py`, and numerous prior `docs/*.md` reports). All were left untouched; none were read as part of satisfying this task's required-reading list beyond what's cited below.

## 1. Files and functions inspected

- `README.md` — current Designs/Dice-Tray/Divider-Tray paragraphs and fixture-total paragraph (887 Designs geometry / 1679 complete suite / 248 Tray-model, confirmed unchanged from the immediately prior review in this session).
- `docs/DESIGNS_WALL_BASE_TAB_COUPON_DESIGN_REVIEW_2026-07-17.md`, `docs/DESIGNS_WALL_BASE_TAB_COUPON_IMPLEMENTATION_2026-07-17.md`, `docs/DESIGNS_WALL_BASE_TAB_COUPON_FOCUSED_AUDIT_2026-07-17.md` — the closest existing precedent for adding a new tray-adjacent geometry option that reuses `trayRectComponent`/`trayWallComponent`/`designSvgDocument` without touching production tray bytes.
- `docs/DESIGNS_DICE_TRAY_FINISHED_VIEW_IMPLEMENTATION_2026-07-17.md`, `docs/DESIGNS_DIVIDER_TRAY_FINISHED_VIEW_IMPLEMENTATION_2026-07-17.md` — read for the existing Finished-View/production-independence contract this feature must also honor.
- `docs/DESIGNS_JOINT_SYSTEM_ARCHITECTURE_REVIEW_2026-07-17.md` and `docs/DESIGNS_JOINT_SYSTEM_ARCHITECTURE_CHALLENGE_REVIEW_2026-07-17.md` — read in an earlier turn of this same session as required background; both establish the project's standing "no capability registry / no premature abstraction" posture that this review's recommendations follow.
- `index.html`, read and traced directly:
  - `designSvgDocument()` (`:1912-1914`) — the tray family's anonymous red-only SVG contract.
  - `trayTabProfile()` (`:1943-1946`), `traySlotWidthMm()` (`:1947`), `trayModelValidationErrors()` (`:1948-1963`), `trayRectComponent()` (`:1964-1966`), `trayWallComponent()` (`:1967-1970`), `trayLayoutItem()` (`:1971-1973`).
  - `buildTrayModel()` in full (`:1974-2022`) — the single shared model builder for both Dice Tray and Divider Tray.
  - `serializeTrayCompatibilitySvg()` (`:2023-2026`), `trayFinishedViewProjection()` (`:2027-2051`).
  - `normalizeDesignDraft()`'s shared `dice-tray`/`divider-tray` branch (`:2177-2186`) and the `validateLegacyDesign()` redundant pre-check for the same templates (`:3008-3011`).
  - `buildTrayDesignResult()` (`:3020-3027`) and `buildDesignResult()`'s dispatch line (`:3165`).
  - `renderDesigns()`'s shared tray field block (`legacyFields`, `:1861-1865`).
  - `designResultsHtml()`'s `dice`/`divider` metrics blocks (`:3314-3326`) and, for comparison, the `cabinet` block's already-shipped `Shelf-placement guides` metric (`:3283`) — the most recent precedent for wiring a new optional-feature metric into this same function.
  - Fixture golden references: Dice Tray default `1726 characters / 51a55721`; Divider Tray default `1965 characters / a55dda6e` (`:3502`), both re-confirmed present and unchanged in the current source.

No file was edited during this review.

## 2. Executive summary

This is buildable as one small, additive change confined almost entirely to `buildTrayModel()` and `buildTrayDesignResult()`: one new plain rectangular `trayRectComponent`, pushed onto the existing `components`/`materialComponentIds` arrays only when a new boolean is set, with a small, source-derived inset formula that needs no new geometry algorithm and touches no existing component, slot, wall, divider, or clearance calculation. The tray family's SVG contract is already fully anonymous (no per-shape ids, no score group, no metadata at all — see §5) and already supports an arbitrary list of shapes via `designSvgDocument()`, so no serializer change of any kind is required, unlike the two prior guide/label features this project has added elsewhere. The one genuine design decision — full-size versus inset cover — is answered in §6 with a derived, proven worst-case bound rather than a guess.

## 3. Physical workflow assessment

The stated five-step workflow (cut & assemble the slotted tray → confirm tabs flush → clean soot/raised ends without changing fit → glue cover underneath → clamp flat while curing) is physically reasonable for ~3 mm plywood on a Genmitsu L8 20W, with the same caveats every other glued/edge-jointed feature in this app already carries:

- **Step 2 (tabs flush)** is the load-bearing assumption for the whole feature: the *mathematical* model places every wall's tab ends exactly at the base's own plane (§6.4 below shows the model has zero tab protrusion by construction), but real assembly can leave tabs slightly proud from kerf variation, incomplete seating, or char buildup — exactly what the task's own Q7 flags. This is a **physical** tolerance question, not a software defect; the model itself is honest.
- **Step 3 (clean soot without changing fit)** is achievable with typical light sanding/scraping on 3 mm plywood, provided it is light enough not to remove the tabs' own bearing surface — the app cannot verify this and should not claim to.
- **Step 5 (clamp flat during cure)** is standard practice for a flat glue-up and is not specific to this feature.
- Whether glue adheres reliably over lightly scorched plywood (task Q9) is genuinely material- and char-depth-dependent; light sanding/scraping (already implied by step 3) is the standard mitigation, and this app should say so without inventing a specific grit, product, or cure time.

None of this workflow touches software geometry beyond what §6 defines. It is a real, physically defensible workflow for the stated material and machine, provided the wording is careful never to claim the mathematical "flush" model guarantees a physically flush result (§9, §11).

## 4. Scope decision: Dice Tray only, and why the shared model does not force otherwise

`buildTrayModel()` is one function used by both templates; the only template-specific branch inside it is the `if (template === 'divider-tray')` block that adds divider slots/dividers (`index.html:1999-2005`). Normalization already has an established, working precedent for this exact situation: `normalizeDesignDraft()`'s shared `dice-tray`/`divider-tray` branch (`:2177-2186`) contains fields common to both templates (`jointStyle`, `trayWidth`, `trayDepth`, `trayHeight`) plus one template-scoped field (`dividerCount`, gated by its own `if (template === 'divider-tray')` sub-check).

**Recommendation: add the new field the same way, scoped the other direction** — inside the shared branch, add `if (template === 'dice-tray') { values.bottomCover = ...; }`, leaving Divider Tray's `values.bottomCover` permanently `undefined` (falsy) for this phase. Inside `buildTrayModel()` itself, the cover-adding logic only needs `if (values.bottomCover)` — no `template === 'dice-tray'` check is required there at all, because normalization alone already guarantees the flag can never be true for a Divider Tray draft.

This directly answers the scope question: **limiting to Dice Tray is a genuine, safe, small-first-phase choice, not a workaround for an "inseparable" shared model.** The shared model does not force Dice and Divider together — it already cleanly supports exactly this kind of one-template-scoped optional field, using the same pattern `dividerCount` already established. A future Divider Tray extension (if ever wanted) would only need normalization's gate widened to also accept `template === 'divider-tray'`; `buildTrayModel()`'s cover logic itself would need no change, since a cover plate underneath a Divider Tray's base would conceal the same kind of through-slots (wall-tab slots plus divider slots) using the identical plain-rectangle geometry — there is no additional geometric complexity for Divider Tray, only additional review/fixture surface, which is exactly why deferring it is the correct "smallest safe phase" choice rather than a technical necessity.

## 5. Current production SVG contract (baseline, unchanged parts)

Traced directly from `serializeTrayCompatibilitySvg()` (`:2023-2026`) and `designSvgDocument()` (`:1912-1914`):

- The tray family's SVG is **fully anonymous**: `designSvgDocument(width, height, shapes)` emits one `<svg>` wrapper and exactly one `<g fill="none" stroke="#ff0000" stroke-width="0.1">` group containing each shape's raw `<rect .../>` or `<path .../>` string, in array order — **no per-shape `<g id>`, no `<title>`, no score group, no metadata, ever**, for either template, regardless of any option. This is structurally different from (and simpler than) the panel-based `serializeDesignSvg()` contract used by Finger Box/Sliding-Lid/Drawer Cabinet/Joint Fit Coupon, which always wraps every panel and score/label entry in an id-and-title-bearing `<g>`.
- Component order for the current default Dice Tray (finger profile): `tray-base` (1), then 16 `base-wall-slot` rectangles emitted inside `buildTrayModel()`'s `addWallSlots()` calls in the order front → back → left → right (4 slots each for the finger profile, 2 each for tab-slot), then the 4 `trayWallComponent` walls in the order front, back, left, right (`:1991-1997`). `layout.items` (and therefore the emitted shape order) is exactly `components.map(trayLayoutItem)`, i.e. array order.
- Confirmed present, unchanged default goldens: Dice Tray **1726 characters / `51a55721`**; Divider Tray **1965 characters / `a55dda6e`** (`:3502`).
- `metrics.pieceCount = materialComponentIds.length`, currently `[base.id, ...walls.map(w=>w.id), ...dividers.map(d=>d.id)]` (`:2007`) — this deliberately excludes the slot rectangles (they are cut-outs in the base, not separate physical pieces), a distinction the new cover component must respect (it **is** a separate physical piece and belongs in `materialComponentIds`).
- `buildTrayDesignResult()`'s returned `metrics` object is currently just `{template:normalized.template}` (`:3026`) — essentially empty; the `designResultsHtml()` "dice"/"divider" metrics blocks (`:3314-3326`) instead read from `result.finishedView` (`trayFinishedViewProjection()`'s output), a semantic projection object, not a rendered view. This is a convenient existing pattern for a metrics table (plain data, no SVG bytes involved) but is explicitly scoped to Finished-View semantics; see §10 for why this feature's own metric should instead extend `result.metrics` directly rather than reuse or grow `trayFinishedViewProjection()`.
- The tray path (`buildTrayDesignResult()`) has **no existing ">400 mm, verify sheet" warning**, unlike Finger Box/Sliding-Lid/Drawer Cabinet (`:3033`, `:3073`, `:3147` all have one; `:3020-3027` does not). This is a pre-existing gap, not introduced by this feature — noted as Informational in §14, not something this phase should fix as an "unrelated tray correction."

## 6. Geometry contract

### 6.1 New component, reusing existing helpers only

One new call to the existing `trayRectComponent()` (`:1964-1966`) — no new geometry helper is needed at all. Recommended shape:

```text
trayRectComponent('tray-bottom-cover', 'Tray bottom cover', 'bottom-cover', coverXMm, coverYMm, coverWidthMm, coverDepthMm, { role:'bottom-cover' })
```

- **Square corners**, matching every other tray component (`base`, walls, slots, dividers all emit plain `<rect>`/orthogonal `<path>` — there is no rounded-corner concept anywhere in the tray system, and introducing one here would be a new, unjustified visual precedent).
- No slots, no score marks, no engrave marks, no tabs, no ids/titles beyond what `trayRectComponent()` already produces for every other tray component (i.e., none — see §5).

### 6.2 Full-size versus inset — derived answer, not a guess

The task asks for the largest safe inset that still covers every wall slot at every valid dimension and both tab profiles. This is answered by directly deriving the worst-case slot inset from `trayModelValidationErrors()`'s own boundary, not by inspection of a few examples.

For the width axis (front/back wall slots, positioned via `designTabPositions(baseOutsideWidthMm, count, tabWidth)`, `:1981-1986`), the first/last slot's inset from the base's own edge is exactly `(baseOutsideWidthMm − tabWidth) / (count + 1)` by the tab-position formula's even spacing (`:1915-1916`) — confirmed symmetric at both ends by direct algebra. `trayTabProfile()` clamps `tabWidth = clamp(8, 16, lengthMm/(2·count))` (`:1943-1946`), and `trayModelValidationErrors()` requires `baseOutsideWidthMm ≥ tabWidth·(count+2)` (`:1960`) — exactly the boundary the model itself enforces, not an assumption.

Working the clamp regions (finger profile, count = 4):
- Below the clamp (`tabWidth = 8` fixed), the constraint requires `baseOutsideWidthMm ≥ 48`; at that exact boundary the inset is `(48 − 8)/5 = 8 mm` — the tightest point in this region.
- In the unclamped middle region (`tabWidth = baseOutsideWidthMm/8`), the constraint `baseOutsideWidthMm ≥ 6·tabWidth` is always satisfied, and the inset `= 7·baseOutsideWidthMm/40` grows monotonically from `11.2 mm` upward — never tighter than the clamped region.
- Above the clamp (`tabWidth = 16` fixed), inset `= (baseOutsideWidthMm − 16)/5`, growing without bound as the tray gets larger.

The two-tab profile (count = 2) works out to the same boundary case: `baseOutsideWidthMm ≥ 32` at `tabWidth = 8`, giving the identical **8 mm** worst-case inset. The depth axis (left/right wall slots, positioned within `insideDepthMm` plus a fixed `+thicknessMm` offset from the base's own edge, `:1988-1989`) is structurally identical but gets an *additional* thickness-sized bonus margin, so it is never tighter than the width axis.

**Global proven worst-case minimum safe inset across every valid dimension and both tab profiles: 8 mm**, occurring only at the exact smallest valid tray size for a given profile (itself only reachable because `trayModelValidationErrors` permits equality, confirmed by re-reading the `<` — not `<=` — comparison at `:1960`).

**Recommendation: a small, fixed inset — 3 mm per side** (`coverWidthMm = baseOutsideWidthMm − 6`, `coverDepthMm = baseOutsideDepthMm − 6`), chosen at roughly a third of the proven 8 mm floor, giving a 5 mm safety margin at the tightest reachable tray and a much larger margin everywhere else. This is preferred over a full-size (zero-inset) cover for two source-grounded reasons: (a) it costs no additional formula complexity beyond a fixed constant applied to values already computed (`model.dimensions.baseOutsideWidthMm`/`baseOutsideDepthMm`), so it does not duplicate the base's own outside-dimension formula, it only subtracts a constant from it; and (b) a small, deliberate reveal is *more* forgiving of minor real-world glue-up misalignment than a full-size plate, where any 1 mm of drift reads as an overhang or a one-sided gap (directly answering Q4). A full-size (no-inset) plate remains a safe fallback if a reveal is not wanted, since it requires literally zero new formula (reuse `baseOutsideWidthMm`/`baseOutsideDepthMm` verbatim) — both options were evaluated; the inset is the recommended default.

**Minimum-size validation**: mathematically unreachable to fail (`baseOutsideWidthMm`'s own minimum valid value is 48 mm at the tightest configuration, and `48 − 6 = 42 > 0` always), so no new error path is strictly required — but a defensive `coverWidthMm > 0 && coverDepthMm > 0` check costs nothing and should be added anyway, consistent with this codebase's general style of validating computed geometry rather than assuming it, with a fixture proving the check never actually trips (§12 item 7).

### 6.3 Placement in the layout — new trailing row, no change to existing rows

The cover is placed as one additional row **below** the existing wall row(s), at `x = marginMm` (same column as `tray-base`), `y` = the existing bottom-of-content expression already used to compute `layoutHeightMm` today, plus the standard `gapMm`. This requires two small, additive edits to `buildTrayModel()`, and nothing else:

1. Append the cover component to `components`/`materialComponentIds` only `if (values.bottomCover)`, after every existing push (base, wall-slots, walls) — literally the last element, so it is guaranteed to serialize last in `layout.items`/the emitted shape order without any explicit sort.
2. Extend `layoutHeightMm`'s existing formula by one more `+ gapMm + coverDepthMm` term when the cover is present; `layoutWidthMm` needs **no change**, because the cover's width (`coverWidthMm ≤ baseOutsideWidthMm`) at `x = marginMm` always fits inside the existing `Math.max(..., baseOutsideWidthMm + marginMm·2)` term that `layoutWidthMm` already computes for the base row.

**One correction to the task's own premise**: the task refers to "the same 10 mm component gap." The tray model's actual established constants are `marginMm = 10, gapMm = 15` (`:1978`) — the *gap* is 15 mm, matching `layoutDesignPanelRows()`'s own defaults (`margin=10, gap=15`, `:2855`) used elsewhere in this app. The cover should reuse the tray's own existing `gapMm` (15 mm) constant as-is, not a new or different 10 mm value.

Because the cover is a strictly new trailing row placed below all existing rows with the standard gap, it **cannot overlap** any wall, divider, base, or slot at any valid dimension — this follows from the same non-overlap guarantee every other stacked tray/wall row already has, not a new check.

### 6.4 Tab-flush question (Q6/Q7/Q8)

In the mathematical model, every wall's tab depth equals `thicknessMm` exactly (`trayWallComponent()` passes `thicknessMm` directly as `designWallPath()`'s `tabDepth`, `:1968`), and the base's own thickness is likewise `thicknessMm` by construction (`baseOutsideWidthMm`/`baseOutsideDepthMm` already assume a flush base) — so **the model has zero tab protrusion by construction; tabs are flush with the base's underside in the idealized geometry.** Real material (kerf, char, focus variance, incomplete seating) can leave tabs proud regardless of what the model says, exactly as the task anticipates — this is why the instructions must explicitly require confirming flush tabs (workflow step 2) as a physical, human verification step before gluing the cover, not a software-checkable one.

## 7. User-facing terminology (settled)

| Concept | Decision |
|---|---|
| Feature name | **Underside cover plate** (working name "Layered slot-hiding base" retained only as an internal/planning label, not user-facing copy — it reads as more mechanical/structural than the feature is) |
| Checkbox text | **"Add underside cover plate (hides wall slots)"** — matches the candidate given in the task exactly; accurate, and doesn't claim strength |
| Internal draft field | **`trayBottomCover`** — matches the candidate; consistent with the existing template-prefixed-field convention for other optional tray/cabinet checkboxes |
| Normalized field | **`bottomCover`** — matches the candidate; consistent with `shelfGuides`/`guideMarks`/`assemblyLabels` naming style already used elsewhere |
| Component id / role | **`tray-bottom-cover`** / role **`bottom-cover`**, kind **`bottom-cover`** — follows the existing `tray-base`/`base` naming pattern exactly |
| Result metric label | **"Bottom cover"** (see §10 for exact wording) |

Confirmed avoided everywhere in the above: "dado," "groove," "slot" (in the load-bearing sense), "captured," "locking," "reinforcement," "structural," or any phrase implying the plate participates in the joint. It is consistently described as a **cosmetic cover glued underneath an already-complete, already-glued structural tray**.

## 8. Production SVG contract (this feature)

- **Disabled**: `buildTrayModel()` takes exactly the same path as today when `values.bottomCover` is falsy — no new component is constructed, `components`/`materialComponentIds`/`layout.widthMm`/`layout.heightMm` are byte-for-byte what they are today. This makes the disabled-state byte-identity claim provable by construction, not merely by testing: the new code path is behind a single `if` that, when false, executes nothing.
- **Enabled**: exactly one additional anonymous `<rect>` appended as the **last** shape in the existing single `<g stroke="#ff0000">` group — no new group, no new LightBurn layer, no id, no title, no metadata, matching the task's strong preference exactly. This requires **no serializer change** at all: `designSvgDocument()` already accepts an arbitrary shape array and `serializeTrayCompatibilitySvg()` already passes `model.layout.items.map(item => item.svg)` unconditionally (`:2025`) — the new component flows through both functions completely unchanged, because they were already generic over "however many shapes are in `components`."
- **Preview/download identity**: unaffected — the tray path already reuses one `result.svg` for both, exactly like every other template; no new code path is introduced here.
- **Filename/MIME**: unchanged — the template key stays `dice-tray`; no new filename branch is needed.
- **New deterministic golden required**: yes, for the enabled state (a new fixed SVG length/hash for a specific default-plus-cover configuration), exactly analogous to how the Wall-to-base Tab Clearance Coupon's enabled output got its own golden (`1551 / d9ffc278`) alongside the unchanged disabled/legacy goldens. The existing disabled goldens (Dice `1726/51a55721`, Divider `1965/a55dda6e`) must remain the ones asserted for the disabled state.

No part of the actual source contradicts the task's stated strong preference; the anonymous, single-group, no-serializer-change design is not just preferred here, it is the only design consistent with how `designSvgDocument()`/`serializeTrayCompatibilitySvg()` already work.

## 9. Layout and material-use disclosure

- Extra sheet area is bounded and proportional: height grows by `coverDepthMm + gapMm ≈ baseOutsideDepthMm − 6 + 15`, i.e. roughly one more tray-depth-sized band plus the standard gap. This is bounded (not exponential, not unbounded) and scales the same way every other stacked tray row already does.
- No automatic nesting or sheet-size optimization is introduced or needed — the cover is one more row in the same fixed, non-nested row layout the tray already uses.
- **Disclosure recommendation**: a single always-visible result metric (§10) stating the cover's dimensions and that it adds one piece, rather than a new conditional ">400 mm" warning. Since the pre-existing tray path has no general sheet-size warning at all today (§5), adding a scoped version of it (checking `layout.widthMm`/`layout.heightMm` and warning only when the cover pushes it over 400 mm) is reasonable and directly tied to this feature — but a general "add the missing 400 mm warning to the whole tray path regardless of the cover" would be an unrelated tray correction and is explicitly out of scope here.

## 10. Normalization, UI, and results wiring

- **Normalization**: `values.bottomCover = draft.trayBottomCover === true || draft.trayBottomCover === 'true' || draft.trayBottomCover === 'on';`, added inside the existing shared `dice-tray`/`divider-tray` branch of `normalizeDesignDraft()`, gated `if (template === 'dice-tray')` (§4). Absent/blank/malformed values all normalize to `false`, identical to every other checkbox field in this codebase (`assemblyLabels`, `guideMarks`, `shelfGuides`). Boolean is sufficient; no enum needed.
- **Session-only, off by default, no storage change**: `designDraft`-only, exactly like every other Designs option; no reference to `state`/`persist()`/`backupObject()` is needed anywhere in this feature, confirmed by the fact that `buildTrayModel()`/`buildTrayDesignResult()` never touch those functions today.
- **UI**: add to `renderDesigns()`'s shared `legacyFields` tray block (`:1861-1865`), gated `d.template === 'dice-tray'` (mirroring how `dividerCount` is gated `d.template === 'divider-tray'` in the same block):
  ```html
  <label class="row"><input name="trayBottomCover" type="checkbox" style="width:auto" ...> Add underside cover plate (hides wall slots)</label>
  <div class="small muted" style="grid-column:1/-1">Adds a separate plain rectangle glued underneath the finished, already-assembled tray to conceal the wall-slot cutouts and tab ends. This is cosmetic only, not a structural joint or a captured bottom; confirm every wall tab is seated flush and clean before gluing. Adds one piece, the material thickness to visible base thickness, and modest extra material and cut time.</div>
  ```
- **Triggers a full render**, same as `assemblyLabels`/`guideMarks`/`shelfGuides`, since it changes `result.metrics`/the results panel, not just numeric preview values.
- **Template/row-count changes**: no stale-state risk — switching away from Dice Tray and back, or changing any tray dimension, simply leaves `trayBottomCover` in `designDraft` and the cover is recomputed fresh from current `model.dimensions` on every rebuild; there is no cached per-cover state to invalidate.
- **Result metrics** (extend `buildTrayDesignResult()`'s currently near-empty `metrics:{template:...}` directly, not `trayFinishedViewProjection()`, keeping Cut-Layout metrics sourced independently from the Finished-View-specific projection object):
  - `designMetric('Bottom cover', result.metrics.bottomCover ? \`Enabled - ${W} × ${D} mm, +1 piece\` : 'Disabled')`
  - Recommended additional line: `designMetric('Base + cover thickness', result.metrics.bottomCover ? \`${designNumberText(2·t)} mm total (visible)\` : \`${designNumberText(t)} mm\`)` — answers Q10 directly in the UI.
- **Warnings** (pushed once, only when `bottomCover` is true): "The cover plate is cosmetic only; it does not strengthen the joint or replace glue in the structural tray." / "Confirm every wall tab is seated flush with the base underside before gluing the cover." / "Clean soot or char from both glue surfaces before gluing." / "Verify alignment and flatness on the first physical tray built with this option." No glue brand, clamp duration, or finishing procedure is specified, per the task's instruction.

## 11. Finished View decision

**Defer to a bounded follow-up; do not add cover geometry to the Finished View in this phase.** `buildDiceTrayFinishedViewSvg()` currently renders the tray from `result.finishedView` (`trayFinishedViewProjection()`'s output), which contains no cover-plate field today and — per §4/§10 — should not be extended with production geometry to serve this feature, to keep the projection object's existing, already-audited contract stable. Rendering an accurate "thin second layer" would require the Finished View to know the cover's inset and thickness, which is easy to compute but adds rendering surface for a purely cosmetic, screen-only view in the same phase as the production geometry change — unnecessary coupling for a first commit. A single optional text note (mirroring the existing precedent at `buildSlidingLidFinishedViewSvg()`'s `guideNote`/`buildDrawerCabinetFrontElevationSvg`'s planned shelf-guide note) reading e.g. *"Underside cover plate enabled (not shown; production Cut Layout only)"* is a safe, zero-geometry addition if wanted, sourced only from the same boolean already in `result.metrics`. The Finished View must not, and under this plan does not, ever consume or reflect production SVG bytes either way.

## 12. Fixture plan (representative, not combinatorial)

1. Missing `trayBottomCover` defaults to `bottomCover:false`.
2. Explicit `false`/malformed values normalize to `false`.
3. Default Dice Tray SVG unchanged, both fields absent and explicitly `false` — assert the existing `1726/51a55721` golden.
4. Default Divider Tray SVG unchanged — assert the existing `1965/a55dda6e` golden (proves the Dice-only gate in §4 does not leak into Divider Tray).
5. Enabled Dice Tray adds exactly one component: `model.components.length` and `metrics.pieceCount` both increase by exactly 1.
6. Every pre-existing component's `id`/`svg`/`geometry` is byte-identical between enabled and disabled builds at the same dimensions (proves no existing base/wall/slot geometry changed).
7. Cover dimensions match `baseOutsideWidthMm − 6` / `baseOutsideDepthMm − 6` exactly, hand-computed independently for at least the default case and one near-minimum-valid case (not merely re-asserting the implementation's own formula).
8. Cover's emitted shape is a plain anonymous `<rect>` with no id/title/metadata beyond the existing contract (regex/structural check against the actual generated SVG, not just the model object).
9. Cover is placed after every existing component, using the tray's own existing `gapMm` (15 mm), with no overlap against any wall/base/slot/divider (reuse the generic overlap-check utilities already used elsewhere in this codebase).
10. All cover geometry values are finite for representative dimensions at both tab profiles (finger and tab-slot) and at least one near-minimum-valid tray size.
11. `layout.widthMm` unchanged, `layout.heightMm` increases by exactly `coverDepthMm + gapMm` when enabled.
12. Preview and downloaded SVG bytes match for an enabled cover.
13. Filename and MIME unchanged with the cover enabled.
14. Storage/backup bytes unchanged before/after generating, previewing, and downloading a cover-enabled tray.
15. Finished View output is unaffected (or differs only by the optional text note from §11) between cover-enabled and cover-disabled builds at the same dimensions.
16. Existing Tray-model (248) and Designs-geometry (887) / complete-suite (1679) totals reconcile exactly once the new assertions above are added, following this project's established static-plus-loop-aware counting method.
17. A new deterministic golden (length + hash) is pinned for one specific enabled default configuration, cross-checked against at least two independent structural assertions (piece count and cover dimensions) rather than accepted on the hash alone.

Estimated size: roughly 12–16 new assertions, comparable in scale to the Wall-to-base Tab Clearance Coupon's 11-assertion addition — not a combinatorial matrix; both tab profiles are exercised only where the inset formula's clamp-region boundary actually depends on them (§6.2), not across every dimension combination.

## 13. Physical verification

**Can wait until the next actual Dice Tray build; no dedicated new test cut is required to continue development**, consistent with Joe's low plywood supply. Nothing in this design requires physical proof before the software can be committed, because the one real hazard (tabs proud in practice, not in the model) is a known, disclosed, physical-only tolerance question — not something fixtures can resolve, and not something blocking a correct, honest software implementation.

When the next Dice Tray is actually built with this option enabled, check specifically:
- tab ends are flush (or sanded flush) before the cover is glued on;
- the cover sits flat with no rocking;
- every wall slot and tab end is fully concealed from the underside;
- edge alignment/reveal looks intentional at the chosen 3 mm inset;
- the glue surface (both the tray's true underside and the cover's top face) is clean and not still sooty;
- no visible warping in the cured cover;
- the added visible base thickness looks and feels as expected;
- if the optional Finished View note (§11) was implemented, that it still accurately reflects what was actually built.

## 14. Risk register

| # | Severity | Risk | Mitigation designed in above |
|---|---|---|---|
| 1 | Major | Cover fails to hide slots at some untested combination of tray size/profile | §6.2's inset is a proven worst-case-derived bound (8 mm floor, 3 mm chosen inset), not a guess; fixture 7/10 covers both profiles and a near-minimum case |
| 2 | Major | Plate rocks on proud tabs or dried glue squeeze-out | Physical-only risk, explicitly disclosed in workflow step 2/§3/§13, not resolvable in software; wording must require confirming flush tabs before gluing |
| 3 | Minor | Duplicated tray dimension formula | Avoided: cover dimensions are `baseOutsideWidthMm/DepthMm` (already computed) minus a fixed constant, not a re-derivation |
| 4 | Minor | Legacy tray bytes change while the option is disabled | Avoided by construction: the new component is behind one `if`, executing nothing when false (§8) |
| 5 | Minor | Divider Tray accidentally affected | Avoided: field is normalized only inside the `dice-tray` branch (§4/§10); fixture 4 proves it directly |
| 6 | Minor | Serializer-contract change | None needed; `designSvgDocument()`/`serializeTrayCompatibilitySvg()` are already generic over shape count (§8) |
| 7 | Minor | Misleading strength claims | Terminology explicitly avoids "structural," "captured," "reinforcement," etc. (§7); warnings state cosmetic-only explicitly (§10) |
| 8 | Minor | Cover placement overlap with an existing piece | Avoided structurally: new trailing row below all existing rows with the standard gap (§6.3) |
| 9 | Minor | Inaccurate Finished View thickness | Avoided by deferring any Finished View geometry change to a bounded follow-up (§11); only an optional boolean-driven text note is in scope now |
| 10 | Informational | Unbounded sheet-size growth | Growth is bounded/proportional (§9); tray path's pre-existing lack of a general 400 mm warning is a known, unrelated gap, not newly introduced |
| 11 | Informational | Accidental storage change | None designed; feature is `designDraft`-only throughout (§10) |

No Blocker-severity risk was found.

## 15. Protected boundaries (confirmed unaffected by this design)

`STORAGE_KEY`, `SCHEMA_VERSION`, `persist()`, `backupObject()`, import/export, Production Settings, evidence/promotion, Library, Inventory, Project, Pricing, Test Grid, Material Test, `trayModelValidationErrors()`'s existing checks, `trayTabProfile()`, `traySlotWidthMm()`, `designTabPositions()`, `designWallPath()`, `trayWallComponent()`, every existing tray component's geometry, Divider Tray's own template branch, `trayFinishedViewProjection()`'s existing fields, and every non-tray Designs template — none require or receive any change under this design.

## Final verdict

**READY FOR IMPLEMENTATION**

The feature has a small, fully additive implementation path: one new `trayRectComponent` call, one new normalized boolean scoped to Dice Tray only inside the existing shared normalization branch, a derived (not guessed) inset formula proven safe against the model's own validation boundary, zero serializer changes because the existing anonymous tray contract already generalizes over shape count, and a Finished View boundary respected by deferring any rendered representation to an optional, non-geometric text note. The one real residual risk (tabs proud in physical reality versus flush in the model) is a disclosed physical tolerance, not a software defect, and does not require a dedicated test cut before this phase can be implemented — it is exactly the kind of check that belongs at the next real build, per Joe's current plywood constraint.
