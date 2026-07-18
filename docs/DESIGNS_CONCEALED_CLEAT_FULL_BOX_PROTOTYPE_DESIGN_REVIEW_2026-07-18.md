# Concealed Cleat Full-Box Prototype (J2.5) — Architecture, Geometry, and Assembly Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-18
**Review type:** read-only architecture, geometry, and assembly review. No file was edited. Nothing staged, committed, or pushed. The only file created is this report.

## 0. Repository state

- `git status -sb`: `## main...origin/main`; tracked tree **clean**; staging **empty**; `git branch --show-current` → `main`; `git rev-list --left-right --count HEAD...origin/main` → `0 0` (**fully synchronized**).
- `git log --oneline -5`: **`30ab1f9 Fix concealed cleat placement guides`** ← `7d89889 Fix concealed cleat LightBurn layer compatibility` ← `f6aed7f Add concealed cleat corner prototype` ← `77c2395` ← `d838041`. **HEAD is `30ab1f9` — the commit immediately following `7d89889`**, exactly as the task specifies, and it is the commit that landed the physically-validated J2.3 four-L-corner guide correction referenced in the implementation report's own final section.
- `git diff --check` / `--stat` / full diff / `--cached --name-only`: all empty. Working tree is byte-identical to `30ab1f9`.
- **Independently re-verified baseline totals via live headless-Edge execution (Chrome DevTools Protocol, `Runtime.evaluate` against every one of the 15 dispatched fixture-group functions plus `runTrayModelFixtures()` standalone — not console-log scraping):** baseline `20`, normalization `12`, production `66`, promotion `58`, design-production `118`, grid `23`, grid-machine `18`, grid-browser `67`, materials `57`, library-browser `56`, project-browser `61`, metadata `12`, storage `8`, project-wizard `216`, **Designs geometry `1040`**, Tray-model (standalone) `264`. Sum of the 15 dispatched groups = **1832**. This **exactly matches the task's stated 264 / 1,040 / 1,832** — confirmed live, not merely read from the README.
- **Documentation discrepancy found and flagged:** the *committed* `README.md` at this exact HEAD still states `Designs geometry reports \`1034 / 0\`` and complete suite `\`1826 / 0\`` — six short of the live, correct totals in both cases. This is the same category of stale-arithmetic-narrative gap this project has hit before (the earlier 1679-vs-1737 case); it is a **pre-existing documentation lag in the current baseline, not something this review created or is asked to fix**, and it does not affect any conclusion below, since every geometry claim here is checked against actual source and hand computation, not against the README's summary sentence. Noted per this project's established practice of never trusting a summary total without independent reproduction.

## 1. Files and functions inspected

- `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md` (the corrected J2 design review — authoritative formulas, ownership rule, zero-intersection proof, all reused and extended below).
- `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_IMPLEMENTATION_2026-07-18.md` (full implementation history including the J2.3 four-L-corner guide correction and the LightBurn blue/green layer fix).
- `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_ADVERSARIAL_AUDIT_2026-07-18.md` (independent audit; verdict SAFE TO COMMIT; the "auditor independent math closes this" pattern for the shared-intersection-helper risk is reused as a fixture-quality bar in §18 below).
- `index.html` at `30ab1f9` (== working tree), read directly: `designDefaults()`; the Designs template selector and `normalizeDesignDraft()`'s `concealed-cleat-corner-prototype` branch (`:2126-2133`); `concealedCleatRectIntersection()` (`:3238-3240`, the exact axis-aligned zero-area-on-boundary-contact formula reused below); `concealedCleatGuideSegments()`/its `cornerTicks()` helper (`:3241-3257` — **the physically-validated four-L-corner-tick generator, confirmed to operate on an arbitrary footprint rectangle, not J2-specific**, directly reusable for J2.5's 8 cleats); `buildConcealedCleatCornerDesignResult()` in full (`:3258-3303` — ownership rule, all six formulas, panel/layout/guide/label construction, error/warning set); `serializeConcealedCleatPathGroup()` and `serializeConcealedCleatSvg()` (`:2965-2971` — the **J2-local, non-shared** dedicated serializer built for LightBurn compatibility: separate blue `#0000ff` no-fill guide group and green `#00ff00` no-fill label group, each before the unchanged red `#ff0000` cut group; confirmed this does **not** touch `serializeDesignSvg()`); `buildConcealedCleatFinishedViewSvg()` (`:3369-3373`, semantic-only, no `result.svg` read); `designPreviewModeForTemplate()`/`designResultsHtml()` wiring for the template; `buildBoxModel()` (re-confirmed the Front/Back-full-width-vs-Left/Right-shortened convention this review extends to a fourth wall); `designPanelFromPoints()`, `layoutDesignPanelRows()`, `designAssemblyGlyphs`/`buildAssemblyLabelPaths()` (confirmed `S`, `W`, `-`, and space glyphs already exist, needed for `WALL A`-style and hyphenated corner labels); the Designs fixture block's 20 Concealed Cleat assertions (`~3746-3767`); storage-isolation and Production Settings fixtures generally.

No required historical report was missing.

## 2. Physical findings carried forward from J2 (binding on this design)

Located precisely (four-L-corner guides), no ambiguity, no base/seam interference, acceptable glue access, surprising no-clamp squareness on the *single* corner, slower than finger joints, **clamp access on a full box unproven**, interior-space cost **application-dependent**, best compared to a **reinforced butt joint**, **not physically validated for strength**. This review treats "clamp access on a full box" and "practical usefulness at full-box scale" as the two open questions J2.5 exists to answer — not questions this document can close from source alone.

## 3. Dimensional convention

**Outside dimensions, consistent with J2 — no exception found that justifies switching.** Raw fields `boxOutsideWidth` (`Ow`), `boxOutsideDepth` (`Od`), `boxWallHeight` (`H`); `materialThickness` (`t`) shared. All are outside/full dimensions, exactly as J2's `cleatLegLength` is the outside leg. Derived, reported, never user-entered: nominal internal width `Ow − 2t`, internal depth `Od − 2t`, internal height `H − t` — the same "base contributes one thickness, walls contribute two" rule every box template in this app already uses (`buildBoxModel()`, Drawer Cabinet). No place in this design accepts or infers an inside dimension where an outside one is expected, or vice versa — every formula in §6 is traced to `Ow`/`Od`/`H` directly.

## 4. Box scope and wall-ownership diagram

**Scope, decided:** one base; four walls (two long, two short); four horizontal base cleats (one per wall — **not reduced**, see below); four vertical seam cleats (**one per corner — not reduced**, see below); mandatory placement guides on every cleat; mandatory assembly labels; one combined production SVG; one screen-only assembled-box preview. No lid (§11), one shared material thickness (§12).

**Ownership — extends J2's own rule, does not invent a new one.** Exactly as `buildBoxModel()`'s Front/Back run the full outside width while Left/Right are shortened, this design uses:

```text
Wall A (front, long, OWNS both its corners)  — full width Ow, at y = 0
Wall C (back,  long, OWNS both its corners)  — full width Ow, at y = Od
Wall B (right, short, YIELDS at both ends)   — shortened Od − 2t, at x = Ow
Wall D (left,  short, YIELDS at both ends)   — shortened Od − 2t, at x = 0
```

```
        Wall A (owns, full Ow)
   D ┌───────────────────────┐ B
   ▲ │ S-AD             S-AB │ ▲
Wall │                       │ Wall
  D  │        (interior)     │  B
 (yields) │                  │ (yields)
   ▼ │ S-CD             S-CB │ ▼
     └───────────────────────┘
        Wall C (owns, full Ow)
```

**Do all four corners need independent cleats, or can ownership reduce the count?** Ownership reduces which **walls** carry seam cleats (only A and C — two walls, not four) but **cannot** reduce the **corner** count: each of the four corners is a physically distinct registration point, so each gets its own seam cleat, hosted two-per-owning-wall. **Four seam cleats, mounted on two walls.** Every base cleat likewise registers one wall's own foot, independent of that wall's seam role, so all four walls get a base cleat — **four base cleats, one per wall**, with the *same* owner/yield asymmetry applied at the base level (§5) so the geometry stays internally consistent between the base plane and the vertical seams.

## 5. Horizontal base-cleat geometry (independent 2-D proof)

Top-down base coordinates, outside corner at `(0,0)`, `x` along width, `y` along depth, `t`/`W` as in J2.

```text
Base-cleat A (under Wall A, owns): x∈[t, Ow−t], y∈[t, t+W]              length = Ow − 2t
Base-cleat C (under Wall C, owns): x∈[t, Ow−t], y∈[Od−t−W, Od−t]        length = Ow − 2t
Base-cleat D (under Wall D, yields, stops short by W at both ends):
                                    x∈[t, t+W], y∈[t+W, Od−t−W]         length = Od − 2t − 2W
Base-cleat B (under Wall B, yields): x∈[Ow−t−W, Ow−t], y∈[t+W, Od−t−W]  length = Od − 2t − 2W
```

**Zero-intersection proof, all six pairs** (reusing `concealedCleatRectIntersection`'s own axis-aligned formula, independently evaluated, not merely re-invoked):

- **A∩C** — y-ranges `[t,t+W]` and `[Od−t−W,Od−t]` are disjoint whenever `Od > 2(t+W)` → **area 0** (a genuinely new minimum-depth requirement, absent from J2's single-corner case).
- **A∩D**, **A∩B**, **C∩D**, **C∩B** — in every one of these four pairs, the y-projections meet only at a single line (`y=t+W` or `y=Od−t−W`); area is the product of the x-overlap and the y-overlap, and the y-overlap is exactly `0` → **area 0 regardless of x-overlap**, the identical "boundary touch, not overlap" pattern J2 already established, now exercised four times.
- **D∩B** — x-ranges `[t,t+W]` and `[Ow−t−W,Ow−t]` are disjoint whenever `Ow > 2(t+W)` → **area 0** (the width analogue of the new depth requirement).

**Consistency check (not assumed, derived twice and compared):** the disjoint-rectangle sum `2W(Ow−2t) + 2W(Od−2t−2W)` is algebraically identical to the "picture-frame" difference `(Ow−2t)(Od−2t) − (Ow−2t−2W)(Od−2t−2W)` — both expand to `2W(Ow−2t) + 2W(Od−2t) − 4W²`. This cross-check confirms the "clear central floor is simply the interior rectangle inset by `W` on every side" metric (§13) is the *exact* consequence of the proven-disjoint cleat footprints, not a separate approximation layered on top.

**No cleat stops "at" another, passes behind it, or alternates ownership mid-run** — every cleat is one continuous rectangle; the owner/yield split is decided once, per wall, and applied at both of that wall's ends identically. **Final-wall registration:** whichever wall installs last still simply drops its foot onto its own already-cured base cleat's exposed edge, exactly like every other wall — the base-cleat design does not treat any wall as structurally special beyond the owner/yield role already fixed in §4.

## 6. Vertical seam-cleat geometry (independent 3-D proof)

Four seam cleats, two per owning wall, using wall-local coordinates (`u` = position along the wall's own length, `z` = height):

```text
Seam A-D: on Wall A, u∈[t, t+W],       z∈[B, H−t]     length = H − B − t
Seam A-B: on Wall A, u∈[Ow−t−W, Ow−t], z∈[B, H−t]     length = H − B − t
Seam C-D: on Wall C, u∈[t, t+W],       z∈[B, H−t]     length = H − B − t
Seam C-B: on Wall C, u∈[Ow−t−W, Ow−t], z∈[B, H−t]     length = H − B − t
```

- **Seam A-D vs Seam A-B (same wall):** both live on Wall A but at opposite ends of its own local-`u` axis; disjoint whenever `Ow > 2(t+W)` — the **same width constraint** already required for base-cleat D/B non-overlap (§5). One inequality governs both; no independent second minimum is needed.
- **Each seam cleat vs its own wall's base cleat:** foot at `z=B` sits exactly on top of that wall's base cleat's top (`z=B` for base-cleat-A or -C) — meets only at the plane `z=B`, **zero volume**, identical to J2's own proof.
- **Each seam cleat vs the perpendicular wall's base cleat** (e.g. Seam A-D vs base-cleat-D): z-ranges `[B,H−t]` and `[0,B]` are disjoint regardless of any xy overlap → **zero volume**, independent of the xy question entirely.
- **Top clearance:** every seam cleat stops `t` below the open top (`z = H−t`), identical to J2's own convention — retained unchanged; nothing in the four-corner case gives a reason to revise it.
- **Does any cleat block the last wall?** No seam cleat's footprint extends into the *material* footprint of the wall it registers (Seam A-B's `u`-range `[Ow−t−W,Ow−t]` lies strictly inside Wall B's own interior-face plane at `x=Ow−t`, never into Wall B's own `x∈[Ow−t,Ow]` body) — the same non-obstruction property J2 already proved for its single yielding wall, now checked at all four corners.

## 7. Assembly sequence and final-wall feasibility

1. Laminate all cleat stacks (4 base + 4 seam, each `cleatLayers` deep) on the bench; clamp flat; cure.
2. **Bench-glue** each seam cleat to its *owning* wall (Wall A gets its A-D and A-B cleats; Wall C gets its C-D and C-B cleats) while the wall is still a flat, unattached panel — the exact lesson J2's own sequence already established (bench accuracy beats reaching into a forming corner). Cure.
3. Glue all four base cleats to the Base at their guide marks. Because §5 proves they only ever *touch* at boundaries, not overlap, there is no forced curing order between them — all four can be placed in one session. Cure.
4. Install Wall A: foot registers on base-cleat-A's exposed edge; both of its seam-cleat feet land on base-cleat-A's top (`z=B`). Check square against the base edge; clamp/tape.
5. Install Wall C the same way, against base-cleat-C. **The base's own edges are the reference for both** — nothing about Wall A constrains Wall C directly, so any drift is between each long wall and the base, not compounding wall-to-wall.
6. Install Wall D: a straight vertical drop into the channel now defined by Wall A's and Wall C's already-fixed inside faces (spacing `Od − 2t`, set entirely by how accurately base-cleat-A and base-cleat-C were placed in step 3) — foot on base-cleat-D, both vertical edges against the A-D and C-D seam cleats. Check square; clamp/tape.
7. Install Wall B — the **final wall** — the same straight vertical drop on the opposite side, foot on base-cleat-B, edges against A-B and C-B. Clean squeeze-out; cure.

**Final-wall feasibility — geometrically sound, physically open.** Wall B's descent path (`x∈[Ow−t,Ow]`) never crosses the A-B/C-B seam cleats' footprints (`u≤Ow−t`, i.e., strictly interior to Wall B's own position) — the drop is unobstructed by construction, and Wall D being already installed on the *opposite* side of the box cannot block it. **What this review cannot resolve, and states plainly:** by the time Wall B goes in, three walls already stand on the base, and any clamp that would normally span the full box width (Wall D's exterior to Wall B's exterior) is now blocked by Wall D. Wall B's own clamp access reduces to whatever can reach its exterior face directly or press down from the still-open top — this is precisely the "clamp access on a full box" question J2's physical build could not yet answer, and this review does not claim it is solved, only that the geometry does not *prevent* installation.

No step requires bending, forcing, or an impossible insertion direction. No wall traps another mechanically (every joint remains glue-dependent, matching J2's own honest classification).

## 8. Placement-guide contract

Reuses `concealedCleatGuideSegments()`'s exact `cornerTicks()` mechanism (already generic over an arbitrary rectangle, already the physically-validated four-L-corner pattern from the J2.3 correction) applied to all eight footprints from §5/§6: `baseCleatA`, `baseCleatB`, `baseCleatC`, `baseCleatD`, `seamAD`, `seamAB`, `seamCD`, `seamCB`. Because §5/§6 prove every footprint is disjoint from every other, the eight guide-tick sets land at eight distinct locations by construction — **no two cleats' guides can be confused**, and each guide's own `name` (`"Base cleat D footprint registration"`, `"Seam cleat A-B footprint registration"`, etc.) states which cleat and which corner it belongs to. Blue guides (`#0000ff`, open strokes) and green labels (`#00ff00`, closed glyph paths) stay in **separate** serialized groups exactly as J2's LightBurn fix already established (§15) — **both remain Line-mode engraving geometry**, and red stays cut geometry. **This review explicitly states, as the task requires: an SVG import does not force LightBurn's per-layer operation mode — color alone is a convention, not an enforced contract — so the UI help text and the results-panel note must instruct the user to assign Line mode to both the blue and the green layers on import**, exactly as J2's own UI/help already does for its three cleats.

## 9. Labels — settled, task's own suggestion partly rejected

**Base and wall labels** (`BASE`, `WALL A`, `WALL B`, `WALL C`, `WALL D`) and **base-cleat labels** (`BA`, `BB`, `BC`, `BD`, one per wall, unambiguous since each wall owns exactly one) are accepted as suggested. **Seam-cleat labels are not accepted as suggested.** The task's candidate `SA`/`SB`/`SC`/`SD` reads as one seam cleat per *wall letter*, but there are only two seam-hosting walls (A, C) carrying two cleats each — that naming would be genuinely ambiguous (which of Wall A's two cleats is "SA"?). **Corrected convention: `S-AD`, `S-AB`, `S-CD`, `S-CB`** — each name states the actual corner (the two walls that meet there), removing the ambiguity at its source rather than relying on installation order to disambiguate. All required glyphs (`S`, `W`, `-`, space) already exist in `designAssemblyGlyphs`; no new glyph is required.

## 10. Recommended default dimensions

**`Ow = 180 mm`, `Od = 150 mm`, `H = 90 mm`** (deliberately rectangular, not square, so the prototype also proves width ≠ depth doesn't break any formula). At `t=3` (`W=10`): both minimums `Ow,Od > 2(t+W)=26` are satisfied with wide margin; at `t=6` (`W=18`): both exceed `2(t+W)=48` with wide margin — the new minimums only meaningfully bind at prototype-corner scale (J2's own 80 mm case), not at this full-box scale, confirming the defaults are comfortably interior to the valid range rather than sitting near an edge case. Large enough to make base-cleat intrusion and last-wall clamp access observable (§7, §13); modest material cost (a handful of ~150-190 mm strips and a ~180×150 mm base, well inside typical hobby-laser bed dimensions); not so small that cleats dominate the interior (§13's occupancy figures stay well under the existing 30%-warning threshold at these defaults).

## 11. Lid and top treatment

**Open top only — no lid, no drop-in test lid.** The task's four review questions (four-corner interaction, assembly sequence, cumulative squareness, last-wall/clamp/glue access, interior-space loss, practical usefulness) are all answerable from an open box; a lid adds an independent risk axis (a fifth structural relationship) that this phase does not need and the task explicitly discourages ("do not add a complex sliding or hinged lid unless clearly necessary"). Smallest sufficient scope.

## 12. Material modes

**One shared material thickness only for this phase — separate shell/cleat thickness deferred.** Drawer Cabinet's dual-thickness model exists and could be reused in shape, but introducing it here would conflate two independent unknowns (does four-corner cleat geometry work at all? does mixed stock change that?) in one physical test. J2.5 should answer the first question cleanly before the second is worth asking.

## 13. Interior-space metrics

```text
Nominal internal width  = Ow − 2t
Nominal internal depth  = Od − 2t
Nominal internal height = H − t
Clear central floor     = (Ow − 2t − 2W) × (Od − 2t − 2W)          (proven exact, §5)
Floor intrusion area    = 2W(Ow − 2t) + 2W(Od − 2t) − 4W²           (proven equal to the strip sum, §5)
Floor-perimeter fraction affected = 100% (all four walls carry a base cleat, unlike J2's single corner)
Approx. clear volume below seam-cleat height ≈ Clear central floor × (H − t)
```

The last figure is stated as an **approximation**: above `z = B`, the four corner regions also lose a further, smaller volume to the seam cleats themselves, not captured by a single floor-area × height product — the report does not claim this is an exact usable-volume guarantee, consistent with the task's explicit caution against overstating irregular-geometry metrics. **Worked defaults (`t=3`):** internal `174×144×87`; clear central floor `154×124=19,096 mm²` against interior `174×144=25,056 mm²` (≈76% of the interior remains clear at these defaults — comfortably below any warning threshold). A warning fires when base-cleat occupancy of the *interior* footprint exceeds 30%, mirroring J2's own threshold, generalized to the four-cleat sum.

## 14. Validation — hard errors vs warnings

**Hard errors:** any of `t, Ow, Od, H` nonfinite or `≤0`; Wall B/D length `Od−2t ≤ 0`; base-cleat-A/C length `Ow−2t ≤ 0`; base-cleat-B/D length `Od−2t−2W ≤ 0`; seam-cleat length `H−B−t ≤ 0`; **`Ow ≤ 2(t+W)`** and **`Od ≤ 2(t+W)`** (the two new minimums from §5/§6, stated as explicit named errors, not silently clamped); any cleat run below `guideMinRun` (reusing J2's own hard-gate policy, not a soft omit — consistent with J2.3's "registration prototype quality" precedent); any nonzero computed cleat-pair intersection (defensive re-assertion of §5's proof); any guide/label geometry escaping its owning panel; any layout overlap; `t` too large relative to `Ow`/`Od` such that no valid wall length remains.

**Warnings:** `t < 2 mm` (thin laminate fragility, reused from J2); any cleat run within `1.2×guideMinRun` of its minimum (marginal-registration caution, reused from J2); base-cleat occupancy `>30%` of the interior footprint (§13); the standing "assembly and registration prototype — not a strength test," glue-dependency, manual-squaring, visible-exterior-seam, and lamination-layer-order warnings from J2, restated for four walls; a **new** warning specific to this phase: *"Clamp access to the final wall has not been physically validated on a full box; plan the assembly order and reachable clamp points before gluing."*

## 15. Production-output contract

**One combined SVG, reusing J2's own local serializer pattern without touching it.** `serializeConcealedCleatPathGroup(paths, groupId, stroke)` is already generic over an arbitrary path list, group id, and stroke color — it makes no J2-specific assumption. J2.5 should call this exact function twice (blue `cleat-placement` guides, green `assembly-labels`) inside its **own** small wrapper (e.g. `serializeConcealedCleatFullBoxSvg`), then the unchanged red cut group — the identical structure `serializeConcealedCleatSvg()` already uses, reused as a pattern, not modified as code, so **J2's own bytes, goldens, and physically-validated behavior are provably untouched.** Deterministic panel order: Base; Wall A; Wall C; Wall B; Wall D; base-cleat-A layers; base-cleat-B layers; base-cleat-C layers; base-cleat-D layers; seam-A-D layers; seam-A-B layers; seam-C-D layers; seam-C-B layers (owning-wall-then-corner order, matching J2's base→wall→cleat convention). No SVG `<text>` element anywhere (label paths only, exactly like J2). Preview is fully independent of production bytes (§16). Filename `l8-concealed-cleat-full-box-prototype-<date>.svg`, MIME `image/svg+xml;charset=utf-8` — a **new**, dedicated template value (`concealed-cleat-full-box-prototype`), not an extension of J2's own template, so J2's committed behavior cannot regress by construction.

## 16. Screen-only preview

One assembled-box diagram, reusing `buildConcealedCleatFinishedViewSvg()`'s exact approach (semantic-values-only, no production-byte parsing): shows all four walls with their owner/yield roles labeled, all eight cleat positions, the open top, the visible exterior seams (at Wall B and Wall D's ends, where they butt the owning walls), and a numeric interior-intrusion note from §13. An exploded or staged-assembly view is **not** included in this phase — the written sequence in §7 already communicates order; a second visual would be scope beyond what the task requires for a first pass. The preview must not read `result.svg`, must not mutate it, and must not be the source of any dimension used elsewhere — every value it draws comes from the same semantic model object §5/§6/§13 describe.

## 17. Storage, roadmap boundaries, and non-goals

Session-only in `designDraft`, exactly like every other Designs field; **no** `STORAGE_KEY`/`SCHEMA_VERSION`/`freshState()`/`loadState()`/`persist()`/`backupObject()`/`replaceData()`/`mergeData()`/import-export change; **no** new storage key, saved preset, Project integration, Library integration, or Production Settings promotion change; **no** Finger Box concealed-cleat selector. **Explicitly deferred to J3 (not this phase):** any decision about Finger Box adopting concealed cleats as a real structural option; any dual-thickness mode for this template; any lid; any decorative treatment; any generic cross-template joint abstraction. This phase remains, like J2, a **dedicated, non-permanent experimental prototype generator.**

## 18. Fixture plan (representative, independent where it matters most)

1. Outside-dimension preservation: `Ow`/`Od`/`H` read back unchanged from a literal draft, distinct from any inside-dimension computation. 2. All four wall dimensions (`Ow×H`, `(Od−2t)×H` twice, `Ow×H` for C) hand-derived, not re-stated from the generator. 3. All eight cleat lengths hand-derived for both 3 mm and 6 mm worked examples. 4. **Horizontal footprint non-overlap: all six pairs independently computed via a *second*, test-local intersection formula** (not merely re-invoking `concealedCleatRectIntersection`, addressing exactly the "shared-helper risk" the J2 adversarial audit itself flagged) — each must equal `0`. 5. Vertical z-range non-overlap for all four seam cleats against both base-cleat stacks, same independent-formula requirement. 6. Unique L-corner guide footprints: assert all eight guide-tick bounding boxes are pairwise non-overlapping and each references the correct owning panel id. 7. Deterministic label ownership: `BASE`/`WALL A-D`/`BA-BD`/`S-AD,AB,CD,CB` each attach to the intended panel id, and no cleat other than the designated outer lamination layer receives a cleat label (reusing J2's outer-layer-only rule). 8. Final-wall installability: assert Wall B's (and Wall D's) computed descent path does not intersect any seam-cleat footprint on the owning walls. 9. SVG ordering/colors: guides blue `#0000ff` open strokes, labels green `#00ff00` closed paths, cut red `#ff0000`, in that order, as three separate top-level groups (not nested under one `#score`), matching J2's LightBurn-driven contract exactly. 10. No SVG `<text>` anywhere in the production output. 11. Production determinism: two builds from the same draft produce byte-identical SVG. 12. Preview byte-neutrality: production SVG bytes identical before/after generating the preview. 13. Invalid dimensions (`Ow ≤ 2(t+W)`, `Od ≤ 2(t+W)`, any cleat length `≤0`, any run below `guideMinRun`) each raise their exact named error. 14. Storage/backup byte-identity around generation, preview, and download; no leaked `boxOutsideWidth`/`boxOutsideDepth`/template-name string in storage or backup JSON. 15. Every existing template's golden (Finger Box incl. faux-dovetail, Sliding-Lid, trays, Drawer Cabinet, both coupons, **and J2's own three cleat goldens byte-for-byte unchanged**) still passes. New assertions belong to the Designs geometry group; no total is estimated here per instruction.

## 19. Physical validation plan

Joe should record: registration accuracy at all eight cleats (not just one corner); the full first-wall-through-final-wall sequence in practice, noting which of Wall B/Wall D actually went in last and why; glue access at each of the eight cleats and four seams; **clamp access specifically at whichever wall goes in last** (the open question from J2); whether the assembly self-squares as well across four corners as it did on the single J2 corner, or whether cumulative error appears; exterior seam consistency at all four corners; actual interior-space loss felt by hand, compared with the computed §13 figures; total assembly time versus an equivalent finger-jointed box; whether the final wall felt awkward, merely slow, or truly blocked; and, stated exactly as the task frames it, **whether the concept is useful enough to justify a J3 design review** — compared honestly against a reinforced butt joint, not against a finger joint, per J2's own established framing.

## 20. Risks and unresolved decisions

| # | Severity | Risk | Disposition |
|---|---|---|---|
| 1 | Major (designed out) | Applying the owner/yield asymmetry inconsistently at one of the four corners | Single rule (§4) applied identically at every corner; §5/§6 prove all six/four pairs independently, not just the ones "expected" to matter |
| 2 | Major (designed out) | A new minimum-dimension requirement going unstated, letting a small full-box draft silently overlap | `Ow>2(t+W)` and `Od>2(t+W)` derived explicitly and required as hard errors (§14), not discovered only in a physical test |
| 3 | Major (designed out) | Seam-cleat label ambiguity from the task's own suggested `SA/SB/SC/SD` naming | Rejected in favor of corner-pair names `S-AD/AB/CD/CB` (§9) |
| 4 | Minor, genuinely unresolved | Clamp access to the final wall on a real, three-walls-already-standing box | Explicitly flagged, not claimed solved; this is exactly what the physical test (§19) must answer |
| 5 | Minor, genuinely unresolved | Whether the concept is *practically* worthwhile at full-box scale versus a reinforced butt joint | Cannot be settled from source; deferred to the physical read-out, same honesty standard J2 itself used |
| 6 | Informational | Interior-volume-above-base-cleat-height figure is an approximation, not exact, given corner-localized seam-cleat intrusion | Stated explicitly in §13, not overstated |
| 7 | Informational | Scope creep toward a Finger Box selector or a cross-template joint framework | Explicitly excluded (§17); new dedicated template only |

## 21. Final verdict

**READY FOR BOUNDED IMPLEMENTATION**

The four-corner, four-base-cleat, four-seam-cleat topology is a direct, provably non-overlapping extension of J2's own physically-validated ownership rule and guide mechanism — no new joint type, no new serializer, no touch to J2's own bytes. Both genuinely open questions (final-wall clamp access, practical usefulness versus a reinforced butt joint) are explicitly physical and are named as the reason this remains a prototype, not a candidate for Finger Box integration. This phase must not become a permanent Finger Box option and must not proceed to J3 without its own physical build and read-out, exactly as J2 required before J2.5 could be specified.

**On the audit-depth question:** Grok alone, working the way it audited J2, is **not** sufficient for the *design* pass — J2.5's combinatorial surface (six pairwise horizontal proofs and four seam/base-cleat vertical proofs, versus J2's single pair and single seam) is large enough that a self-consistency-only audit (checking the implementation against its own stated formulas) could miss an asymmetry applied at only three of four corners. A deeper architecture review (this one) was warranted for the design, exactly as it was requested. For the later **implementation** audit, Grok is sufficient **provided it repeats the discipline the J2 adversarial audit itself already modeled**: independently re-deriving the intersection and z-range proofs with a *second* formula rather than only re-invoking the implementation's own helper, which is the one concrete gap that audit flagged in itself and closed by hand.
