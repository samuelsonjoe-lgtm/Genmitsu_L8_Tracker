# Additional Joint Types — Deep Architecture and Design Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-18
**Review type:** read-only architecture and design review. No application or documentation file was edited except creating this report. Nothing was staged, committed, or pushed.

## 0. Repository and verification state

- `git status -sb`: `## main...origin/main`; tracked working tree **clean**; staging **empty**; `git branch --show-current` → `main`; `git rev-list --left-right --count HEAD...origin/main` → `0 0` (**fully synchronized**).
- `git log -1 --oneline`: **`0fa1d37 Add custom row drawer cabinet layouts`** — matches the expected baseline exactly. The commit contains `index.html`, `README.md`, and the three custom-row-layout reports (design review, implementation, focused audit), confirmed via `git show --stat`.
- `git diff --check` / `--stat` / full diff / `--cached --name-only`: all empty. Working tree is byte-identical to `0fa1d37`, so every source reading below **is** the committed baseline.
- Untracked files: the same long-standing 67-entry set (`LightBurn Projects/*`, `debug.log`, `parametric_qr_stand_generator.py`, prior `docs/*.md` reports). All untouched.
- Hunk enumeration of `0fa1d37`'s `index.html` change (all 30+ hunks listed via `git show -U0`): every hunk falls in Drawer Cabinet regions (defaults `:415`, cabinet fields, normalization, dimensions/model `:2373-2476`, result/metrics `:3124-3297`, fixtures) — the Finger Box, Sliding-Lid, Tray, Coupon, serializer, and score machinery regions are untouched by the latest commit, so all prior verified facts about those regions remain current.

### Verified baseline totals

From the committed `README.md` at `0fa1d37`: complete suite **1752 / 0**, Designs geometry **960 / 0**, Tray-model **264 / 0** — matching the task's stated baseline exactly. Superseded totals (922/934/945, 1714/1726/1679/1737) are not used anywhere below. Live runtime was not re-executed this turn; the committed README totals derive from the custom-row audit chain, and this review's recommendations do not depend on re-running them.

## 1. Files and functions inspected

- `index.html` at `0fa1d37` (== working tree), read directly this session: `designDefaults()`; Finger Box / Sliding-Lid / Tray / Drawer Cabinet / Coupon UI field construction in `renderDesigns()`; `normalizeDesignDraft()` (all template branches); `buildBoxModel()` (`:2250` region — full edge assignments); `buildFingerPattern()` (`:2215` — odd-count, `minimumWidth = max(2t, 4)`, `boundaries[]`); `designPatternEdge()`; `designPointsPath()` (**strictly rectilinear — returns `''` for any non-H/V segment**); `designScoreSegmentsPath()` (**supports diagonal `L` segments** — re-confirmed verbatim this turn); `buildSlidingLidBodyPanel()` / `buildFingerPanel()` / `designSafePatternEdgePoints()` / `designEdgeTerminalOffset()` (edge ownership and polarity); `designPanelFromPoints()`; `namespaceDesignPanel()`; `mirrorDesignPanel()`; tray tab family (`trayTabProfile()`, `traySlotWidthMm()`, `designTabPositions()`, `designWallPath()`, `trayWallComponent()`); `serializeDesignSvg()` (`:2910` — **generalized `scoreGroupId` parameter confirmed present**, default `'rail-guides'`, used with `'shelf-guides'` by the cabinet); `layoutDesignPanelRows()`; `buildBoxDesignResult()` (**passes no `scorePaths` today** — its `partial` carries only `labelPaths`); `buildJointCouponModel()` and `buildWallBaseTabCouponModel()` (both coupon modes); assembly-label machinery (`designAssemblyLabelSpecs/Geometry`, `buildAssemblyLabelPaths`, `designSegmentsContainedInPolygon`, `designMinimumSegmentClearance`, `designSerializedAssemblyLabelValidation`); Finished View builders; Designs fixtures including the Finger Box byte-stability signatures; Production Settings applicability; storage-isolation fixtures; the `selftest` dispatcher.
- `README.md` at `0fa1d37`.
- Reports read this session: the joint-system architecture review and challenge review (2026-07-17), Drawer Cabinet architecture/implementation/audit chain, hidden-assembly-labels chain, sliding-lid guide reports, Dice/Divider Finished View chain, both Joint Fit Coupon mode reports (finger-edge, wall-to-base), Dice Tray underside-cover chain, Finger Box corner-blocks design review (DO NOT IMPLEMENT, 2026-07-17 — directly load-bearing for candidates C/E below), multi-grid dual-thickness chain, custom-row-layouts chain. No required historical report was found missing.

## 2. Current joint architecture (from actual source)

Two structurally distinct joint families exist today:

1. **A-pipeline finger joints** — `buildFingerPattern()` (odd segment count, `max(2t,4)` minimum width, exact `boundaries[]`) → `designPatternEdge(pattern, phase, depth, clearance)` descriptors → `buildSlidingLidBodyPanel()`/`buildFingerPanel()` compose four edges into one closed **rectilinear** outline via `designSafePatternEdgePoints()` with corner-safe terminal offsets. Used by Finger Box, Sliding-Lid Box, Drawer Cabinet (shell + drawers), and the Finger-edge coupon. Mating is by shared pattern + opposite `phase`; fit is one signed `clearance` applied at finger flanks.
2. **B-pipeline tray tabs** — `trayTabProfile()` (4 or 2 tabs, clamped width) + `designTabPositions()` (even spacing) + `designWallPath()` (rectilinear tab-wall outline) + `traySlotWidthMm(t, clearance)` through-slots in the base. Used by Dice/Divider Tray and the Wall-to-base coupon. Anonymous single-red-group SVG contract.
3. **Non-joints**: Drawer Cabinet shelves and partitions are plain glued rectangles (`designPanelFromPoints()`, zero edge descriptors) — the committed precedent that "glued, unjointed interior piece" is an accepted construction class.

**Two serializer facts govern everything below** (both re-confirmed verbatim this turn):
- **Cut outlines are rectilinear-only.** `designPointsPath()` returns `''` for any diagonal segment. No trapezoid, taper, or angled profile can be *cut* through the A-pipeline without modifying a shared helper that every existing golden depends on.
- **Score paths support diagonals.** `designScoreSegmentsPath()` emits `L` commands for non-orthogonal segments (already exercised by label glyph diagonals such as `1`, `M`, `N`, `K`). Diagonal *engraved* geometry is representable today with zero serializer change, and `serializeDesignSvg()` already accepts an additive `scorePaths` + `scoreGroupId` per result — Finger Box simply doesn't use that slot yet.

Fit-clearance semantics today: `jointClearance` (A-pipeline, signed, −0.10..+, applied at finger flanks), `fitClearance`/slot clearance (B-pipeline, additive slot width), both distinct from LightBurn kerf compensation, which the app deliberately never applies. Measured thickness flows from Production Settings as a value copy only; joint style is a design choice, never a stored material setting.

## 3. Structural versus decorative — terminology ruling

The tracker should model **two separate axes**, and this review's first phase deliberately implements only one:

- **Structural joint** (`finger`, `tab-slot`, future `concealed-cleat`) — the cut geometry locates/locks/aligns the assembly.
- **Decorative treatment** (`none`, future `faux-dovetail-engrave`, possible later `dovetail-overlay`) — visual only; never changes cut bytes.

The separation was challenged and survives: merging the axes would put "dovetail" in a structural selector (misleading — see §4.F), would create combinatorial names (`finger-with-dovetail-look` × every future joint), and would allow impossible combinations to look selectable. Keeping decoration a separate, explicitly-labeled axis makes "this is not structure" a property of the UI's shape, not just of help text. **The first phase implements only the decorative axis** because every structural candidate requires physical validation first (§8) while the decorative axis requires none, and Joe's plywood supply is currently limited (established constraint this session).

Terminology rules settled: **"True dovetail" is reserved and never used** for anything this laser produces in this configuration. Accepted user-facing terms: *"Faux dovetail engraving (decorative only)"*, later *"Dovetail corner overlay (applied piece)"*, *"Concealed cleat corner (glued, hidden from outside)"*. Banned unqualified labels: "hidden joint" (unless no exterior seam pattern is visible — a butt seam line is acceptable, visible fingers are not), "dovetail", "mortise", "tenon".

## 4. Candidate evaluation

**A. Finger joint (baseline).** Keep exactly as is. Finger-width/count configuration, asymmetric patterns, and reinforced corner fingers are all *possible* but change production bytes and multiply fixture surface for no expressed need — explicitly not expanded in this roadmap's first phases.

**B. Straight tab-and-slot box corner.** Feasible and proven as a *family*: the trays already do wall-to-base tab-slot, and the Wall-to-base coupon already tests its fit dimension. Applied to vertical box corners: wall A carries edge tabs, wall B carries through-slots inset ~t from its edge; assembly is perpendicular insertion — genuinely assemblable. **Honestly not hidden**: tab ends show as small rectangles on wall B's exterior face (fewer visible elements than a full finger rhythm, but visible). Slot rows near an edge weaken the edge web; minimum edge length and end margins needed. Value over fingers is aesthetic (sparser corner) and simpler alignment; strength inference: less glue area than fingers (inference, unverified). **Defer** to a later structural phase, coupon-extended first.

**C. Concealed butt joint with internal cleats.** **This is the honest "hidden joint" answer for 2D sheet cutting** — the exterior shows only a plain butt seam line, no joint pattern. Critically, this is *not* the rejected corner-blocks feature: the 2026-07-17 corner-blocks review rejected blocks as *supplements to already-finger-jointed corners* (redundant glue area, false squaring claim). Here the cleats **are** the joint — the box has no finger pattern at all, and the cleats supply both glue area and, decisively, **registration**: a cleat strip face-glued to the base, inset exactly one material thickness from the base edge, gives each wall a physical fence to register against (the classic liner/ledge construction). One flat cleat face-glues to one panel (full face contact) and presents its long edge to the mating panel — real, honest geometry, all plain rectangles via `designPanelFromPoints()`, fully rectilinear. Cost: butt+cleat in 3 mm ply is glue-dependent and needs manual squaring plus clamps; strength relative to fingers is unproven (inference: less mechanical interlock, comparable or better glue area depending on cleat sizing). Naming: **"Concealed cleat"** — not "hidden butt joint," since the seam line remains visible. **Realistic; requires a physical corner prototype before any generator integration** (§8). Second-ranked candidate; first *structural* phase.

**D. Captured/internal-tab joint.** Tabs engaging slots in internal strips: this is C plus through-slots moved onto hidden internal pieces. Better self-registration than plain cleats, but strictly more parts, an assembly-order constraint (internal strip must be placed before walls close), and it trends toward a double-wall box. **Defer** — reconsider only if the C prototype shows registration is insufficient.

**E. Keyed corner joints.** Analyzed in three sub-forms. (1) **Bowtie/spline across a perpendicular corner seam: rejected** — a flat through-cut key lies in one sheet's plane; the seam of a 90° corner lies along an edge between two non-coplanar faces, and no flat key can cross it. Bowties are coplanar-seam devices only. (2) **Tab-and-wedge knock-down** (tab protrudes through slot; wedge drops through a hole in the protruding tab): genuinely assemblable, genuinely locking, fully rectilinear — but tabs protrude beyond outside dimensions, the pierced tab neck is fragile in 3 mm ply, tolerances are kerf-sensitive, and nobody has asked for knock-down behavior. **Defer, coupon-first if ever.** (3) **Decorative key inserted into a face slot near the corner**: possible later as part of the overlay family; defer.

**F. Faux-dovetail treatments.** Four meanings, evaluated separately:
1. **Engraved faux dovetail — realistic, top-ranked, selected as the first phase (§6).** Diagonal score segments are already supported (§2); the engraving machinery (containment, clearance, score-before-cut, deterministic paths) already exists for labels and guides. The key design insight making it *visually honest*: derive the trapezoids **from the actual finger pattern** of the decorated edge, so the engraving dresses the real, visible finger end-grain patches as dovetail tails rather than painting an unrelated fake pattern next to a conflicting real one.
2. **Dovetail corner overlay (appliqué)** — requires *cutting* trapezoids, which the rectilinear cut path blocks; would need either a shared-helper change (endangers every golden) or a raw-path panel exception. **Defer**; revisit only with a deliberate, isolated raw-path decision.
3. **Bowtie insert** — coplanar seams only (§4.E); **reject for corners**, defer generally.
4. **Trapezoidal through-tabs as structural corner joints — rejected with a geometric proof.** A laser tab is prismatic (constant silhouette through the sheet). For any insertion direction available at a box corner, a tab whose silhouette flares wider than its root faces a strict dichotomy: if the mating opening admits the widest section, the joint has clearance at the neck and **cannot lock**; if it is sized to grip the neck, the tip **cannot enter**. Real dovetails escape this only because their flare is through the board's thickness — precisely the dimension a perpendicular 2D cut cannot shape. Cut loose enough to assemble, the taper is cosmetic and the joint is *sloppier* than a square finger joint. There is no honest structural version of this shape on this machine.

**G. Half-lap / cross-lap.** Fully rectilinear, through-cut, classic, and genuinely useful — but for **crossing dividers**, not box corners. The Divider Tray currently supports one divider direction with base slots; a crossing grid via half-height interlocking notches is a natural, bounded future Divider Tray phase with its own slot-depth/web validation. **Belongs on the joint roadmap; excluded from box-corner selection; own later phase.**

**H. Knock-down hardware (T-slot bolts, captive nuts, pins).** External hardware in a session-only offline design app, unrequested, with hardware-tolerance complexity. **Defer indefinitely** (not "reject forever" — but nothing in the current product justifies it).

**I. Layered/double-wall (cosmetic skins, liner boxes).** The only way to hide joints *completely* (no visible seam at all), at roughly double material, glue, and alignment cost. The challenge review already deferred skins until a decorative-part workflow is proven. **Defer**; C covers the practical "hidden" need at a fraction of the cost.

## 5. Feasibility matrix

| Candidate | Class | 2D feasible | Through-cut only | Exterior visibility | Extra parts | Glue | Self-aligning | 3 mm | 6 mm | Thickness/clearance sensitivity | Strength vs finger (inference) | Likely templates | Coupon needed | Verdict |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Finger (baseline) | Structural | Yes | Yes | Full rhythm visible | 0 | Yes | Good | Proven | Yes | Proven behavior | Baseline | All boxes | Exists | Keep |
| Faux-dovetail engraving | Decorative | **Yes (score layer)** | Score only | Decorates existing corner | 0 | n/a | n/a | Yes | Yes | None (no fit) | None (no change) | Finger Box first | Scrap look-check only | **Implement first** |
| Concealed cleat | Structural + aid | Yes | Yes | Butt seam line only — no joint pattern | +4–8 strips | Yes, essential | **Yes via inset fence** | Plausible | Better | Inset = t (measured) | Unproven; ≈glue-dependent | Finger Box later; prototype first | **Yes, corner prototype** | Implement after prototype |
| Tab-slot box corner | Structural | Yes | Yes | Tab ends visible on faces | 0 | Usually | Good | Yes (proven family) | Yes | Slot clearance (proven family) | Likely less interlock | Boxes, coupon | Extend existing coupon | Defer (J4) |
| Captured internal tab | Structural | Yes | Yes | Clean exterior | Many | Yes | Good | Marginal | Yes | High | Unproven | Future box variant | Yes | Defer |
| Tab-and-wedge knock-down | Structural | Yes | Yes | Tabs protrude outside | +keys | No | Yes | Fragile necks | Better | Very high | Unproven | Coupon only | Yes | Defer |
| Bowtie key on corner | — | **No** (coplanar-only) | — | — | — | — | — | — | — | — | — | — | — | **Reject** |
| Trapezoidal structural tabs | — | Cut yes, **assembly no** | Yes | — | 0 | — | — | — | — | — | — | — | — | **Reject** (lock/insert dichotomy) |
| Dovetail overlay | Decorative | Blocked by rectilinear cut path | Yes | Visible appliqué | +pieces | Yes | n/a | Yes | Yes | Low | Local reinforcement at most | Later | Look-check | Defer |
| Half-lap/cross-lap | Structural | Yes | Yes | Interior | 0 | Optional | Yes | Yes | Yes | Slot width = t + clearance | n/a (dividers) | Divider Tray | Small | Defer (own phase) |
| Blind/partial-depth anything | — | **Not dependable** on diode | **No** | — | — | — | — | — | — | Extreme | — | — | — | **Reject** (experimental at most) |
| Hardware knock-down | Structural | Yes | Yes | Hardware visible | Hardware | No | Yes | Yes | Yes | Hardware tolerances | n/a | None requested | — | Defer indefinitely |

No precise strength percentages are assigned anywhere — all strength statements above are explicitly inferences from glue area and interlock, not measurements.

## 6. Selected first phase — exact specification

**Phase J1: "Faux dovetail engraving" — a decorative-only corner treatment on Finger Box, structural joint unchanged.** (Option 4, chosen over the coupon-first Option 1 for one decisive reason: every structural candidate needs physical material to validate and Joe's plywood is currently limited, while J1 needs no fit validation, risks zero cut bytes, reuses proven machinery end-to-end, and directly delivers the specifically-requested dovetail aesthetic honestly. The structural coupon is J2, first in line once material allows.)

- **Template target:** Finger Box only. **Included:** one decorative option. **Excluded:** every structural change, Sliding-Lid/Tray/Cabinet decoration, overlays, keys, cleats, coupon changes.
- **Raw field:** `boxCornerDecoration` — absent/blank/`'none'` → none; exactly `'faux-dovetail-engrave'` → enabled; any other nonblank value → error *"Choose a recognized corner decoration."* (the established `couponType` pattern). Added to the full-`render()` trigger list. Session-only in `designDraft`.
- **Normalized field:** `values.cornerDecoration` (`'none'`|`'faux-dovetail-engrave'`).
- **UI:** one select in `boxFields`, after the lid/labels block: label **"Corner decoration"**, options **"None"** / **"Faux dovetail engraving (decorative only)"**, help text: *"Engraves dovetail-style taper lines over the actual finger positions at the four vertical corners, on the face that should assemble outward. Decorative only — it does not change the cut geometry, joint strength, or fit. Run the blue score layer before cutting."*
- **Geometry ownership:** one new template-local helper (e.g. `fingerBoxCornerDecorationSegments(patterns, dimensions, thickness, panels)`), Option A architecture (§7). For each of the four wall panels' two vertical corner edges, read the **existing** `verticalPattern.boundaries` (single source of truth — no re-derived spacing) and, for each **recess** segment of that edge (the positions where the mating panel's end grain is visible on the assembled face, phase logic identical to `designEdgePoints`), emit three score segments forming a trapezoid: two diagonal flanks from the edge-adjacent inset toward the interior, one inner base line. Parametrization: depth `d = t` (the mating panel's visible thickness at the corner), flank taper `f = min(actualWidth/5, t/2)`, edge inset `m ≈ 0.5 mm` so no score line touches a cut line; exact aesthetic constants are implementation-tunable but must be fixture-pinned as literals once chosen. Segments that would collapse (`segmentWidth − 2f` below a minimum) are omitted per-mark with a warning, mirroring the label-omission pattern. Containment via `designSegmentsContainedInPolygon()`; a **dedicated** minimum cut-clearance (~0.4–0.6 mm) rather than the label formula `max(1, t/3)`, because decoration legitimately sits near the edge — this distinction must be explicit in code and fixtures.
- **Face/handing:** every Finger Box panel is mirror-symmetric (both vertical edges of each wall use the same pattern and phase — confirmed from `buildBoxModel()`'s edge arrays), so designating the engraved face as the **outward** face imposes no cut change and no `mirrorDesignPanel()` call. Assembly instruction: engraved faces outward.
- **Labels interaction:** assembly labels target inside faces; decoration targets outside faces; both engrave the same sheet face in one job. When both are enabled, emit a **warning** (not an error): *"Assembly labels and corner decoration share the same engraved sheet face; with decoration outward, labels will show on the finished exterior."*
- **Production output:** `buildBoxDesignResult()` additively passes `scorePaths` (placed per-panel like sliding-lid guides) with `scoreGroupId: 'corner-decoration'` — using the existing generalized parameter at `serializeDesignSvg()` `:2910`, zero serializer change, score-before-cut preserved, blue #0000ff, one file, existing filename/MIME, preview/download identity unchanged. When disabled, the `partial` object is byte-identical to today's (no `scorePaths` key), making disabled-state byte identity provable by construction.
- **IDs:** `corner-decoration-<panelId>-<left|right>-<NN>` (e.g. `corner-decoration-front-left-02`), titled "Faux dovetail engraving - Front left edge", ordered panel-order-then-edge-then-index — deterministic and collision-free against `guide-*`, `shelf-guide-*`, and `label-*` namespaces.
- **Metrics:** `Corner decoration` → `Faux dovetail engraving - N marks (decorative only)` / `Disabled`. **Warnings:** decorative-only (no strength/fit change); shared-face label warning when applicable; appearance unverified until first scrap engrave.
- **Assembly contract:** unchanged from the current Finger Box in every respect; the treatment adds zero parts and zero assembly steps.
- **Finished View:** no geometry change; optional one-line note (`'Corner decoration enabled (Cut Layout score marks).'`) following the existing sliding-lid `guideNote` precedent. Deferred rendering of the pattern itself.
- **Production Settings:** no change; no new schema field; thickness continues to apply as today.
- **Physical prerequisite:** none blocking implementation. One **attended scrap engrave** (with ventilation, fire watch, extinguisher nearby, clean optics, correct focus, flat stock, controlled air assist — never unattended) before treating the appearance as production-worthy; alignment across the physical corner should be inspected then.
- **Non-goals:** structural joints, overlays, keys, cleats, other templates, GUI cleanup, nesting, kerf compensation, storage changes.

## 7. Architecture ruling

**Option A — template-local builders — selected.** The decoration helper is Finger-Box-owned; future cleat geometry would be prototype-template-owned. **Option B** partially exists already: `buildFingerPattern()`/`designPatternEdge()` *are* the bounded shared edge helper for pattern joints, and the tray tab family serves the B-pipeline — no new shared abstraction is needed for J1, and cleats (whole pieces, not edges) would make an edge-strategy helper misleading. **Option C (registry/plugin framework) is rejected**, consistent with the standing challenge-review ruling; nothing in the current source justifies it, and the single-file app's byte-identity discipline actively argues against it.

## 8. Coupon and physical-test strategy (for the structural roadmap)

A flat fit-slot coupon **cannot** validate the concealed-cleat corner — the thing being tested is three-dimensional registration and glue-up behavior, not slot tightness. J2 therefore specifies a true **corner prototype**: two short walls, one base section, plus its cleat set (inset by measured thickness), assembled with glue as a real corner. It tests: registration accuracy of the inset fence, seam tightness, squaring behavior under clamps, glue surface adequacy, interior-space loss, and the honest look of the exterior butt seam. It must be labeled an **assembly/registration prototype, not a strength test**. The tab-slot box-corner candidate (J4) extends the existing Joint Fit Coupon family instead, which already has the right shape for fit-dimension learning. Measured thickness from the actual sheet, a prior material test, and attended cutting with full safety practice (ventilation, active fire watch, nearby suppression, clean optics, correct focus, flat material) are required for every physical phase. No speed/power values are invented here.

## 9. Template applicability (realistic candidates only)

| | Finger Box | Sliding-Lid | Dice Tray | Divider Tray | Cabinet shell | Cabinet drawers | Coupons | New prototype template |
|---|---|---|---|---|---|---|---|---|
| Faux-dovetail engraving | **J1** | Later (handed Right panel already solved) | No (no finger corners) | No | Later (many corners; separate-mode face rules) | Later | No | No |
| Concealed cleat | J3 (after J2) | Doubtful (rail/lid interference) | Possible much later | Possible much later | Doubtful (drawer travel + interior loss) | No (interior loss) | No | **J2 prototype** |
| Tab-slot corner | J4+ | Doubtful (shortened front) | Already tab-slot (wall-to-base) | Same | Later study | Later study | **J4 coupon extension** | Alternative host |
| Cross-lap dividers | n/a | n/a | n/a | **J5** | n/a | n/a | Small coupon optional | n/a |

Interior-space, lid-travel, and drawer-travel interference are the recurring disqualifiers — no joint helper should be assumed portable across templates without this table's per-template re-check.

## 10. Fit and clearance ruling

- J1: **no fit parameter at all** (nothing mates). The engraving must not consume or imply `jointClearance`.
- J2/J3 concealed cleat: the load-bearing dimension is the **cleat inset = measured thickness** (reuses the existing measured-thickness semantics exactly; no new clearance field). A deliberate small registration allowance, if the prototype shows it's needed, would be a new, distinctly-named field (`cleatRegistrationAllowance`) — not a reuse of `jointClearance`, whose finger-flank semantics don't apply.
- J4 tab-slot corners: reuses the B-pipeline slot-clearance semantics already proven by the trays and the wall-to-base coupon — honestly reusable because the physical dimension (slot width = t + clearance) is identical in kind.
- Kerf compensation remains permanently separate from all of the above.

## 11. Fixture plan (J1, representative — Designs geometry group ownership)

1. Missing `boxCornerDecoration` → byte-identical to current Finger Box goldens (open-top and loose-lid signatures). 2. Explicit `'none'` → identical. 3. Unknown nonblank value → the exact error string. 4. **Cut-path identity when decoration toggles**: `panels.map(p=>p.id+':'+p.path)` identical between decorated and undecorated builds at the same inputs (the single most important assertion — item 19 of the task list). 5. Score subgroup present with `id="corner-decoration"`, inside `<g id="score">`, before `<g id="cut">`. 6. Diagonal segments present in score content and **absent from every cut path** (regex: no `L` commands in cut `d` attributes beyond existing none). 7. Mark positions cross-checked against an independent `buildFingerPattern()` call's boundaries, not against the helper's own output. 8. Containment and dedicated minimum clearance hold for every emitted segment. 9. Deterministic, collision-free IDs across the full score/label namespace. 10. Small-box omission behavior: undersized recesses omit marks with warnings while cut output stays valid. 11. Labels+decoration co-enabled → the shared-face warning appears; both score subgroups serialize in stable order. 12. Preview/download byte identity with decoration on. 13. `localStorage`/`backupObject()` byte identity around generate/preview/download. 14. Other-template goldens (Sliding-Lid 2800/`4a7ab718`, trays, cabinet, coupons) unchanged. 15. Production Settings application unchanged. 16. Metrics line renders the decorative-only wording (HTML-inclusion check). 17. A new enabled golden pinned **only after** 4–10 pass structurally. Estimated ~14–18 assertions; no Cartesian matrix. Final totals are not estimated here per instruction.

## 12. Phased roadmap

- **J1 (now):** Faux dovetail engraving on Finger Box (this review's spec). Independent audit; scrap engrave check when material allows.
- **J2:** Concealed-cleat **corner prototype** (small dedicated experimental template: two walls + base + cleat set + assembly note; prototype templates may legitimately expose experimental choices that production templates must not). **Hard dependency: physical build and evaluation.** Independent audit.
- **J3:** If J2 validates physically: `boxJointType` (`finger` | `concealed-cleat`) as Finger Box's second structural option — the first use of the structural axis, byte-identity-gated on `finger`. Independent audit.
- **J4:** Tab-slot box-corner study via a Joint Fit Coupon extension; only then decide whether any production template adopts it.
- **J5:** Cross-lap crossing dividers for Divider Tray (own bounded phase, own slot/web validation).
- **J6+:** Decoration rollout to other templates (per §9 face-handling rules); overlay family only after a deliberate raw-path/cut-diagonal decision; skins remain deferred; knock-down remains deferred indefinitely.

Each phase: own commit, own audit, GUI cleanup stays deferred throughout.

## 13. Protected boundaries and byte-identity plan

J1 touches: `designDefaults()` (one key), the Finger Box branch of `normalizeDesignDraft()`, `boxFields`, one new template-local helper, `buildBoxDesignResult()` (additive `scorePaths`), metrics/warnings, fixtures, README. It does **not** touch: `serializeDesignSvg()` (parameter already exists), `layoutDesignPanelRows()`, any storage/schema/import-export function, Production Settings schema, evidence/promotion, any other template's builder, or any existing golden's inputs. Legacy byte identity is structural: the disabled path passes the exact same `partial` object shape as today. All current goldens (Finger Box signatures, Sliding-Lid 2800/`4a7ab718`, tray, cabinet including the six multi-grid pins, coupon 1551/`d9ffc278`) remain asserted and untouched.

## 14. Risk table

| # | Severity | Risk | Disposition |
|---|---|---|---|
| 1 | Blocker (avoided by design) | Describing any 2D-cut trapezoid as a structural dovetail | §4.F.4 rejection proof; terminology rules in §3; "decorative only" in label, help, metrics, and warnings |
| 2 | Major (designed out) | New shared abstraction altering existing finger bytes | Option A template-local helper; no shared-helper edits; fixture 4 |
| 3 | Major (designed out) | Score/cut layer mixing (diagonals leaking into cut) | Diagonals exist only in `scorePaths`; fixture 6 |
| 4 | Major (future phases) | "Hidden" cleat joint whose registration fails physically | J2 prototype is a hard gate before J3; prototype labeled assembly-test, not strength-test |
| 5 | Minor | Decoration visually clashing with real finger end grain | Mitigated by deriving marks *from* the finger pattern; confirmed at scrap engrave |
| 6 | Minor | Labels and decoration on the same engraved face | Explicit warning (§6); user chooses |
| 7 | Minor | Dedicated near-edge clearance mistaken for the label clearance rule | Named separately, fixture-pinned |
| 8 | Informational | Engraving depth/char varies with material and focus | Inherent to diode engraving; warned, verified on scrap |
| 9 | Informational | Scope pressure toward a universal joint framework | Rejected again per standing challenge-review ruling |

## 15. Physical honesty and unavailable checks

Software review cannot verify: joint strength, glue strength, real friction fit, kerf effects, plywood glue-layer behavior, tab/key/cleat durability or alignment, finished squareness, engraving appearance or char, LightBurn import, or machine cutting. No strength comparison against the finger joint is claimed anywhere above beyond explicitly-labeled glue-area/interlock inferences. No browser runtime, LightBurn, machine, or physical check was executed in this review; baseline totals come from the committed README's audited chain.

## Final verdict

**READY FOR BOUNDED IMPLEMENTATION**

Phase J1 — Faux dovetail engraving as a decorative-only, score-layer corner treatment on Finger Box, structural joint unchanged, with the exact field, geometry, output, and fixture contracts in §6/§11 — is safe, honest, byte-identity-preserving by construction, and needs no physical material to build. The structural roadmap (concealed-cleat prototype → optional second structural joint → tab-slot study → cross-lap dividers) is sequenced behind explicit physical validation gates and stays clear of every protected boundary.
