# Concealed Cleat Corner Prototype (J2) — Design and Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-18
**Review type:** read-only design and architecture review. No application file was edited. Nothing staged, committed, or pushed. The only file created/revised is this report.

> **Correction pass (2026-07-18).** This revision fixes two load-bearing geometry defects in the prior draft and every consequence that followed from them:
> - **Defect 1 — base-cleat collision.** The prior draft claimed "stop short by one material thickness" prevented the two perpendicular base cleats from overlapping. That was false whenever `cleatWidth > materialThickness` (i.e. at both worked examples). The corrected §5/§6 use an explicit 2-D coordinate model with a single ownership rule and a formal zero-intersection proof; the corrected base-cleat-B length is `O − t − W`, not the prior `L − 2t`.
> - **Defect 2 — leg-length semantics.** The prior draft called `L` an "inside leg length" but used it as `Base = L × L` and `Wall A = L`, which is an **outside** dimension in a base-beneath-walls model. The corrected contract renames it to `O` (outside leg length), raw field `cleatLegLength`, label *"Prototype outside leg length."*
> Worked examples, intrusion figures, guide records, label strategy, assembly sequence, validation, fixtures, and the risk table are all updated to match. The false risk-table claim about stop-short-by-t has been removed.

## 0. Repository and verification state

- `git status -sb`: `## main...origin/main`; tracked working tree **clean**; staging **empty**; `git branch --show-current` → `main`; `git rev-list --left-right --count HEAD...origin/main` → `0 0` (**fully synchronized**).
- `git log -1 --oneline`: **`d838041 Add faux dovetail engraving to Finger Box`** — matches the expected baseline. `git diff --check` / `--stat` / full diff / `--cached --name-only` all empty; the review file under revision is **untracked** and safe to revise; no application file, no unrelated report, and no LightBurn/`.claude`/utility file was touched.
- **Verified baseline totals** (`git show d838041:README.md`): Tray-model **264 / 0**, Designs geometry **994 / 0**, complete suite **1786 / 0** — matching the task's stated baseline exactly.

## 1. Files and functions inspected (re-confirmed this pass)

`docs/DESIGNS_ADDITIONAL_JOINT_TYPES_ARCHITECTURE_REVIEW_2026-07-18.md` (authoritative prior roadmap — J2 defined there as this prototype, gating any Finger Box structural change); `docs/DESIGNS_FINGER_BOX_CORNER_BLOCKS_DESIGN_REVIEW_2026-07-17.md` (the rejected corner-block feature, the §3 comparison baseline); the Finger Box faux-dovetail implementation/audit (the `scoreGroupId` additive-score precedent). `index.html` at `d838041`: `designDefaults()`; the Designs template selector and per-template field blocks in `renderDesigns()`; `normalizeDesignDraft()`; `buildBoxModel()` (Front/Back run full outside width, Left/Right are shortened — the asymmetric-winner convention reused in §5); `designPanelFromPoints()` / `designPointsPath()` (rectilinear-only); `namespaceDesignPanel()`; `mirrorDesignPanel()` (confirmed unnecessary, §9); `layoutDesignPanelRows()` / `layoutDesignPanels()`; `serializeDesignSvg()` (`:2910` — generalized `scoreGroupId`, default `'rail-guides'`); `designScoreSegmentsPath()`; the assembly-label helpers (`designAssemblyLabelSpecs()`, `designAssemblyLabelGeometry()`, `buildAssemblyLabelPaths()`, `designSegmentsContainedInPolygon()`, `designMinimumSegmentClearance()`); the Drawer Cabinet shelf-guide implementation (`drawerCabinetShelfGuideSegments()` — the tick-mark + containment precedent reused in §8); `buildJointCouponModel()` / `buildWallBaseTabCouponModel()` (dedicated-small-generator precedent); Finished View builders; the `selftest` dispatcher; storage-isolation and Production Settings fixtures. No required historical report was missing.

## 2. Terminology and classification (settled)

**Selected name: "Concealed Cleat Corner Prototype."** "Concealed" describes only the cleat and fastening geometry — **the exterior butt seam line remains plainly visible from outside**, stated everywhere the name appears. Banned terms confirmed absent: no "hidden joint," "self-squaring," "mortise," "tenon," or "dovetail" in this construction's naming.

**Classification: structural, glue-dependent joint** (not a pure assembly aid, not equivalent to the finger joint's mechanical interlock, not self-squaring). Justified in §3.

## 3. Distinction from the rejected corner-block feature — is it meaningful?

**Yes, structural, not semantic.**

| | Rejected corner-block (2026-07-17) | This concealed-cleat prototype |
|---|---|---|
| Existing joint at the corner | **Full-height finger joint already present** | **None** — plain butt edges; no interlock, minimal inherent glue area |
| Role of the new piece(s) | Supplemental to an already-good corner | **Primary** — the cleat is the entire registration/glue mechanism |
| "One flat piece can't face-bond two perpendicular faces" | Caused rejection of a single spanning block | **Resolved** — *independent* cleats, each face-bonded to one piece only, edge-registering the other |
| Squaring claim | Its stated purpose was physically incoherent | **No squaring claim** — location, repeatable inset, physical stop only |
| Honest comparison | Against an already-strong finger joint (marginal) | Against a **bare, unregistered butt joint** (potentially substantial *if physical result is good*) |

The distinction holds: this changes the question asked (does a butt-plus-cleat corner work as a Finger Box's second structural option?) rather than re-litigating an answered one.

## 4. Cleat topology — comparison and selection

| Option | Disposition |
|---|---|
| A — single-layer flat strip | Base case; too thin a fence at 3 mm → upgraded by B |
| **B — laminated strip cleats** | **Selected** — stacking identical strips gives a thicker registration fence; low-risk glue-up; layer count derived from thickness (§6), self-adjusting 3 mm↔6 mm |
| C — laminated internal corner block | Rejected — reintroduces the block's face-bond limitation without cleanly separating base vs seam registration |
| D — two-strip L pre-assembly | Rejected — adds its own unvalidated 90° joint |
| E — base fence cleats only | Rejected as sole topology (leaves the novel wall-to-wall seam untested); its base-cleat role is retained |
| F — no implementation | Rejected — a testable, low-risk registration/glue improvement over a bare butt joint warrants one careful physical test |

**Selected topology: two base cleats + one vertical seam cleat, all laminated per B** — three independently-attached cleats, not a pre-joined L.

## 5. Base/wall construction model and ownership rule

**Model 1 — base beneath walls — selected.** The model used throughout this app (`buildBoxModel()` Bottom, Drawer Cabinet Cabinet Bottom): base outside face down, walls rest on the base's top face, the base defines the outside footprint. It needs no new base-ownership convention and no partial-depth cut, and keeps the prototype's panel arrangement identical *in kind* to today's Finger Box so that only the joint (fingers → butt+cleat) is the isolated variable J3 needs.

**Single ownership rule — "Wall A wins everywhere, Wall B yields everywhere."** Wall A runs the full leg length, flush with the base corner (mirrors Front/Back owning the seam in `buildBoxModel()`). Wall B is shortened by one thickness and butts against Wall A's inside face. Consistently, **base cleat A (under Wall A) owns the inside corner; base cleat B (under Wall B) yields** — it starts past base cleat A's full width. The seam cleat is Wall A's, and Wall B registers against it. This was challenged against the alternative (letting base cleat B own the corner): that would make Wall B's cleat the winner at the base while Wall A owns the wall seam — an inconsistent split — so it is rejected. One winner, expressed identically at the seam and at the base.

### 5.1 Explicit 2-D base coordinate model

Top-down view of the base's top face. Base outside corner at `(0, 0)`; `x` runs along Wall A; `y` runs along Wall B; `O` = outside leg length; `t` = measured thickness; `W` = cleat width; `B` = cleat built thickness.

| Piece | Footprint on base (x-range × y-range) | Height z |
|---|---|---|
| Base panel | `[0, O] × [0, O]` | — |
| Wall A foot | `[0, O] × [0, t]` (inside face at `y = t`) | wall stands in `+z` |
| Wall B foot | `[0, t] × [t, O]` (inside face at `x = t`; end butts Wall A at `y = t`) | wall stands in `+z` |
| **Base cleat A** (owns corner) | `[t, O] × [t, t+W]`; exposed edge `y = t`; length `O − t` | `[0, B]` |
| **Base cleat B** (yields) | `[t, t+W] × [t+W, O]`; exposed edge `x = t`; length `O − t − W` | `[0, B]` |
| **Seam cleat** (on Wall A) | `[t, t+W] × [t, t+B]`; exposed edge `x = t`; vertical run | `[B, H − t]` |

### 5.2 Zero-intersection proof (base plane)

Every relevant pair intersects only on a boundary line (area 0), assuming the §16 minimums hold (`O > t + W`):

- Base cleat A ∩ Base cleat B: y-ranges `[t, t+W]` and `[t+W, O]` meet only at the line `y = t+W`; x-overlap `[t, t+W]` → **intersection area = 0** (the corrected non-overlap; the prior draft overlapped here by `W × W`).
- Wall A foot ∩ Base cleat A: y-ranges `[0, t]` and `[t, t+W]` meet at `y = t` → 0 (this line *is* the intended Wall A registration contact).
- Wall B foot ∩ Base cleat B: x-ranges `[0, t]` and `[t, t+W]` meet at `x = t` → 0 (intended Wall B registration contact).
- Wall B foot ∩ Base cleat A: x-ranges `[0, t]` and `[t, O]` meet at `x = t` → 0 (why cleat A starts at `x = t`).
- Wall A foot ∩ Base cleat B: y-ranges `[0, t]` and `[t+W, O]` are disjoint (gap `t`→`t+W`) → 0.
- Wall A foot ∩ Wall B foot: y-ranges `[0, t]` and `[t, O]` meet at `y = t` → 0 (the corner butt).

All footprints are positive-area rectangles for `O > t + W`, `H > B + t`. Boundary contact is registration, not overlap.

### 5.3 Seam cleat in 3-D (zero-volume-overlap proof)

Seam cleat volume `x∈[t, t+W], y∈[t, t+B], z∈[B, H−t]` (glued flat to Wall A's inside face at `y = t`, laminations stacking in `+y` into the interior; exposed edge at `x = t` where Wall B registers).

- Seam ∩ Base cleat A: base cleat A is `z∈[0, B]`, seam is `z∈[B, H−t]` → meet only at `z = B` (a plane) → **zero volume** ("meets but does not overlap the base-cleat stack," by construction the seam foot rests on base cleat A's top).
- Seam ∩ Base cleat B: same `z = B` boundary → zero volume.
- Seam within Wall A: `x∈[t, t+W] ⊂ [0, O]`, `z∈[B, H−t] ⊂ [0, H]` → contained, does not extend outside Wall A.

**Wall B is not trapped:** Wall B installs by a straight `−z` descent into `x∈[0, t]`. Nothing intrudes into `x < t` (both base cleat B and the seam cleat live at `x ≥ t`), so the descent is unobstructed and Wall B can be lifted straight back out before cure (glue-dependent, not mechanically captured).

## 6. Piece list, dimensions, and formulas (corrected)

**Raw fields:** `materialThickness` (existing shared field, reused), **`cleatLegLength`** (label *"Prototype outside leg length"*), **`cleatWallHeight`** (label *"Prototype wall height"*). **Normalized:** `values.legLength` (= `O`, the outside side length of the square base and the full length of Wall A), `values.wallHeight` (= `H`). **Derived only** (never user-set): `cleatLayers`, `cleatBuiltThickness`, `cleatWidth`.

```text
t   = materialThickness (measured)
O   = legLength         (OUTSIDE leg length: base side and Wall A length)
H   = wallHeight

Wall A = O × H                    (owns the corner; flush with base edge)
Wall B = (O − t) × H              (shortened; butts Wall A's inside face)
Base   = O × O

targetCleatBuiltThickness = 6 mm
cleatLayers = clamp(round(6 / t), 1, 3)
B = cleatBuiltThickness = cleatLayers × t
W = cleatWidth          = max(3·t, 10 mm)

Base cleat A: length = O − t,     width W;  footprint x∈[t,O],   y∈[t,t+W],   z∈[0,B];  exposed edge y=t
Base cleat B: length = O − t − W, width W;  footprint x∈[t,t+W], y∈[t+W,O],   z∈[0,B];  exposed edge x=t
Seam cleat:   length = H − B − t, width W;  footprint x∈[t,t+W], y∈[t,t+B],   z∈[B,H−t]; exposed edge x=t
```

**Derived clear interior (reported, since `O` is an outside dimension):** wall-to-far-edge clear span in each direction = `O − t`; usable floor away from the cleats begins at `x = t+W` and `y = t+W` (i.e., a further `W` is consumed by each base cleat). These are reported as derived geometry — `O` is never labeled an inside dimension.

**Per-cleat 3-D detail:**

| Cleat | Glue face (to owning piece) | Lamination stack direction | Exposed registration edge | Contacting wall |
|---|---|---|---|---|
| Base cleat A | length×W face down onto Base top | `+z` (upward) | vertical `y=t` face, `(O−t) × B` | Wall A inside face |
| Base cleat B | length×W face down onto Base top | `+z` (upward) | vertical `x=t` face, `(O−t−W) × B` | Wall B inside face |
| Seam cleat | length×W face onto Wall A inside face | `+y` (into interior) | `x=t` end face, `(H−B−t) × B` | Wall B inside face (upper span) |

### 6.1 Worked example — measured 3.00 mm, defaults `O = 80`, `H = 50`

`t=3; W=max(9,10)=10; cleatLayers=clamp(round(6/3),1,3)=2; B=6.`
- Wall A `80×50`; Wall B `77×50`; Base `80×80`.
- Base cleat A: length `80−3=77`, width `10`, 2 layers `77×10×3`; footprint `x∈[3,80], y∈[3,13], z∈[0,6]`.
- Base cleat B: length `80−3−10=67`, width `10`, 2 layers `67×10×3`; footprint `x∈[3,13], y∈[13,80], z∈[0,6]`.
- Seam cleat: length `50−6−3=41`, width `10`, 2 layers `41×10×3`; footprint `x∈[3,13], y∈[3,9], z∈[6,47]`.
- **Piece count = 9** (1 base + 1 Wall A + 1 Wall B + 2 + 2 + 2 cleat layers).
- Base-cleat union footprint = `10×77 + 10×67 = 1440 mm²` (cleats proven disjoint, so union = sum); of `80×80 = 6400 mm²` → **22.5 %**.
- Base-cleat intersection area = **0** (touch at `y = 13`).

### 6.2 Worked example — measured 6.00 mm, defaults `O = 80`, `H = 50`

`t=6; W=max(18,10)=18; cleatLayers=clamp(round(6/6),1,3)=1; B=6.`
- Wall A `80×50`; Wall B `74×50`; Base `80×80`.
- Base cleat A: length `80−6=74`, width `18`, single layer `74×18×6`; footprint `x∈[6,80], y∈[6,24], z∈[0,6]`.
- Base cleat B: length `80−6−18=56`, width `18`, single layer `56×18×6`; footprint `x∈[6,24], y∈[24,80], z∈[0,6]`.
- Seam cleat: length `50−6−6=38`, width `18`, single layer `38×18×6`; footprint `x∈[6,24], y∈[6,12], z∈[6,44]`.
- **Piece count = 6** (each cleat single-layer).
- Base-cleat union footprint = `18×74 + 18×56 = 2340 mm²`; of `6400 mm²` → **36.6 %**.
- Base-cleat intersection area = **0** (touch at `y = 24`). Even at `W = 18`, the ownership rule (cleat B starts at `y = t+W = 24`) keeps them disjoint — the exact case the prior "stop short by t" formula failed.

These reproduce the task's expected corrected values (`W = 10/18`, base cleat A `77/74`, base cleat B `67/56`, seam `41/38`) — derived here from the coordinate model, not copied. No physical strength claim is attached to any number.

## 7. Registration and clearance convention

**No exposed clearance field.** `jointClearance` (finger-flank) does not apply (no finger); tray slot clearance does not apply (no slot); no `registrationAllowance` is added (registration is a plain edge-to-face butt at nominal zero clearance; real-world slop stays a kerf/measurement matter this app keeps separate). **One convention everywhere:** every score guide and derived cleat position marks the cleat's **outer/exposed registration edge** — the single line a builder aligns the next piece against (`y=t` for base cleat A, `x=t` for base cleat B and the seam cleat).

## 8. Score guides (exact semantic records) and assembly labels

**Guides are mandatory for this template** (its whole purpose is one correctly-understood physical assembly). Shape reuses the Drawer Cabinet shelf-guide pattern: **two short perpendicular ticks at the ends of each cleat run**, never a continuous line under the glue face. Constants reused from that precedent: `tickLength = max(6, 2t)`, `edgeInset = max(2, t)`. Geometry is derived from the §6 formulas — the model builder, the assembled-corner preview (§14), and the fixtures all read the same values; no offset is re-derived anywhere. Score-group id `'cleat-placement'` via the existing generalized `scoreGroupId` (zero serializer change). Containment via `designSegmentsContainedInPolygon()` / `designMinimumSegmentClearance()`.

| Guide | Owning panel | Marks | Tick 1 | Tick 2 | Orientation |
|---|---|---|---|---|---|
| Base cleat A guide | Base | exposed edge `y = t`, over `x∈[t, O]` | near `(x=t, y=t)` | near `(x=O, y=t)` | short ticks ⊥ to the `y=t` line |
| Base cleat B guide | Base | exposed edge `x = t`, over `y∈[t+W, O]` | near `(x=t, y=t+W)` | near `(x=t, y=O)` | short ticks ⊥ to the `x=t` line |
| Seam cleat guide | Wall A | exposed edge `u=t` (panel-x), over `z∈[B, H−t]` | near `(u=t, z=B)` | near `(u=t, z=H−t)` | short ticks ⊥ to the `u=t` line |

The two base guides visibly communicate the corrected stop-short ownership: base cleat B's guide starts at `y = t+W` (not `y = t`), so the offset from the corner where base cleat B begins is drawn, not implied. Each tick's inset from any cut edge is `≥ edgeInset`; a guide that cannot satisfy containment is a hard error (§16), since guides are mandatory.

**Assembly labels** are built locally (a local spec array passed to the generic `buildAssemblyLabelPaths()`), not by extending the shared `designAssemblyLabelSpecs()`. Compact identifiers to fit narrow strips: `BASE`, `WALL A`, `WALL B`, `CA` (base cleat A), `CB` (base cleat B), `SC` (seam cleat). See §8.1 for which layer carries a cleat label.

### 8.1 Cleat-label layer and face rule

For a laminated stack, **only the final/top layer receives the label**, engraved face outward (exposed after lamination), and all other layers are unlabelled — so no label is ever trapped between glue faces or on a glued face:
- Base cleats stack in `+z`; the top layer's **upward** face is exposed → label the top layer, engraved face up; stack it last with label up. Not on the down (glued-to-base) face, not on the `y=t`/`x=t` registration edge.
- Seam cleat stacks in `+y`; the outermost layer's **interior-facing** (`y=t+B`) face is exposed → label the outermost layer there, engraved face outward. Not on the `y=t` (glued-to-Wall-A) face, not on the `x=t` end (Wall B contact/glue zone).
- **Single-layer 6 mm cleats:** label the exposed face (up for base cleats, interior-facing for the seam cleat); the opposite broad face is the glue face.

Base/wall labels sit in free areas clear of every cleat footprint and guide-tick zone (`BASE` centered in the base's cleat-free quadrant near `(O, O)`; `WALL A`/`WALL B` centered on inside-face area away from the seam-cleat footprint). Because cleat strips are only `W` wide, a compact label is oriented along the strip length; if even a compact label cannot satisfy containment on a very narrow cleat, it is safely omitted with a warning (reusing the existing label-omission behavior) while the **mandatory guide** still identifies placement and the deterministic layout order still identifies the piece.

## 9. Face orientation — one-sided engraving workflow

Single-sided engraving only; **no `mirrorDesignPanel()` and no automatic flipping** (Wall A and Wall B are genuinely different pieces, not mirror duplicates — checked, not assumed). Complete workflow:
- Base: engraved face becomes the **top/interior** face.
- Wall A, Wall B: engraved faces become **inward/interior**.
- Each labeled cleat layer: engraved face remains **exposed after lamination** (top for base cleats, interior-facing for the seam cleat); unlabelled layers stack beneath/behind it.
- The user must **preserve the intended layer order during lamination** (labeled layer outermost) — stated in the assembly note and warnings. The production layout supports this with no mirrored cut geometry.

## 10. Assembly sequence (corrected coordinates)

1. Laminate base cleat A's layers (skip if `cleatLayers === 1`), labeled layer on top; clamp flat; cure.
2. Laminate base cleat B's layers likewise; cure.
3. Laminate the seam cleat's layers likewise, labeled layer outermost; cure.
4. Glue base cleat A to the Base top at its guide ticks — exposed edge on the `y = t` line, body toward `+y`, near end at `x = t`.
5. Glue base cleat B to the Base top at its guide ticks — exposed edge on the `x = t` line, body toward `+x`, near end at `y = t+W` (clearing base cleat A). Cure both base cleats.
6. **Glue the seam cleat to Wall A's inside face on the bench, before Wall A is fixed to the Base** — its exposed edge on Wall A's `u = t` line, bottom at `z = B`, top at `z = H − t`. Bench-gluing a small cleat to a flat panel is far more accurate than reaching into an assembled corner, and (step 8) the seam cleat's foot then lands cleanly on base cleat A. Cure.
7. Dry-check that base cleat A's top (`z = B`) and the seam cleat's foot (`z = B`) will meet.
8. Lower Wall A (with seam cleat attached) straight down onto the Base: outer face flush with the base edge (`y = 0`), inside face at `y = t` against base cleat A's exposed edge; the seam-cleat foot seats on base cleat A's top at `z = B`. Check square; clamp or tape.
9. Lower Wall B straight down (`−z`) into `x∈[0, t]`: foot registers against base cleat B's `x = t` edge, upper inside face against the seam cleat's `x = t` edge, end-grain butting Wall A's inside face at `y = t`. Nothing intrudes into `x < t`, so the descent is unobstructed and Wall B is not trapped.
10. Re-check square; clamp; clean squeeze-out before it hardens; cure.

No step requires bending a panel, forcing a part, or sliding through a blocked path; every insertion is a straight vertical descent. Clamps/tape reach every surface because the corner is open (a single three-way corner, not an enclosed box).

## 11. Squaring honesty

The cleats honestly provide **location, a physical stop, increased glue area vs a bare butt joint, and temporary registration**. They do **not** provide automatic squaring, guaranteed perpendicularity, or any strength guarantee. Squaring still requires a carpenter's square, clamps or tape, and flat reference stock — stated in the assembly sequence, the results-panel note, and warnings, not implied.

## 12. Interior-space intrusion (corrected, non-overlapping union)

Because the base cleats are proven disjoint (§5.2), the union area is the exact sum:
`baseCleatFootprintArea = W·(O − t) + W·(O − t − W)`.

| | 3.00 mm | 6.00 mm |
|---|---|---|
| Base-cleat union area | `10·77 + 10·67 = 1440 mm²` | `18·74 + 18·56 = 2340 mm²` |
| Fraction of `O² = 6400 mm²` | **22.5 %** | **36.6 %** |
| Inward intrusion from Wall A (base cleat A) | `W = 10 mm` | `W = 18 mm` |
| Inward intrusion from Wall B (base cleat B) | `W = 10 mm` | `W = 18 mm` |
| Seam-cleat interior projection from Wall A | `B = 6 mm` over `10×41 mm` at height `z∈[6,47]` | `B = 6 mm` over `18×38 mm` at `z∈[6,44]` |

The prior draft's `1510 mm²` figure (from the overlapping `L−2t` lengths, which double-counted the corner) is **superseded by `1440 mm²`**. All figures are geometric measurements of this prototype's own small footprint, explicitly **not** extrapolated to a full box — the same absolute `W`-wide strip is a much smaller fraction of a realistically larger interior, an assessment deferred to J3.

## 13. Panel IDs and production order

`cleat-proto-base`, `cleat-proto-wall-a`, `cleat-proto-wall-b`, `cleat-proto-base-cleat-a-layer-01`(`-02`…), `cleat-proto-base-cleat-b-layer-01`(`-02`…), `cleat-proto-seam-cleat-layer-01`(`-02`…). "seam-cleat" names the role (registers the wall-to-wall seam) and stays clear of the rejected "corner block." **Deterministic order:** base; Wall A; Wall B; base-cleat-A layers ascending; base-cleat-B layers ascending; seam-cleat layers ascending. Collision-free (new `cleat-proto-*` prefix, shared with nothing). Guide and label references point only to panels present at the current layer count.

## 14. Production output, layout, and preview

**One combined SVG** — all pieces share one thickness (no basis for a second file). Filename `l8-concealed-cleat-corner-prototype-<date>.svg`, MIME `image/svg+xml;charset=utf-8`. One preview, one download, byte-identical. Blue `cleat-placement` + `assembly-labels` score subgroups before the red cut group (score-before-cut preserved via the unmodified serializer). `layoutDesignPanelRows()` places pieces in rows (base; walls; base-cleat layers; seam-cleat layers) — generic over count and size; the existing finite/non-overlap checks apply unchanged; no serializer change; no nesting.

**Preview: one small screen-only assembled-corner diagram** (base, both walls, cleat positions with the corrected stop-short ownership, a note that guide/label faces assemble inward), consuming only the §6 semantic dimensions — **no `result.svg` read** — following the Drawer Cabinet Finished-Front-View precedent. Exploded/lamination diagrams deferred; the written assembly-order text covers sequencing. Preview labels use the corrected outside-leg wording.

## 15. Metrics and warnings

**Metrics:** Prototype type; Material thickness; **Prototype outside leg length** (`O`, explicitly outside); Wall height; Cleat construction (`N-layer laminated strip` / `single-layer`); Cleat built cross-section (`B × W`); Cleat quantity (3 positions); Required physical pieces (semantic total — 9 at 3 mm, 6 at 6 mm); **Interior-space intrusion** (the corrected §12 figures, scoped to this prototype's footprint); Production sheet size. No strength rating, no guaranteed-squareness, no "validated joint" status.

**Warnings:** experimental assembly/registration prototype, **not a strength test**; glue required at every cleat and seam (nothing mechanically locked); manual squaring with a square and clamps required (cleats give location, not squareness); **the exterior corner remains a plain visible butt seam** — only the cleat and fastening are concealed; cleats consume interior space (re-evaluate at real box dimensions before production); preserve the labeled-layer-outermost lamination order; physical validation not yet completed. Plus the standing laser-safety reminders (attended operation, ventilation, fire watch, correct focus, flat stock, clean optics, nearby suppression).

## 16. Impossible-dimension handling (corrected minimums)

The prior `O > 2t` minimum was insufficient — it ignored `W` consuming the perpendicular run and ignored fitting the mandatory guides. Corrected minimums, derived from geometry:

`guideMinRun = 3 · tickLength` (two end ticks + a central gap ≥ one tick, so the ticks read as distinct registration points), `tickLength = max(6, 2t)`.

**Errors (impossible geometry / mandatory guide cannot be placed):**
- `t` nonfinite or `≤ 0`; `O` nonfinite or `≤ 0`; `H` nonfinite or `≤ 0`.
- Wall B length `O − t ≤ 0` (⟺ `O ≤ t`).
- Base cleat A length `O − t ≤ 0` (same bound).
- **Base cleat B length `O − t − W ≤ 0`** (⟺ `O ≤ t + W`) — the binding positivity constraint; this is where "cleat width consumes the usable corner" is caught explicitly, not silently clamped.
- Seam cleat length `H − B − t ≤ 0` (⟺ `H ≤ B + t`).
- **Base cleat B length `< guideMinRun`** (guides can't fit): `O − t − W < 3·tickLength`.
- Seam cleat length `< guideMinRun` (vertical): `H − B − t < 3·tickLength`.
- Any `designPanelGeometryErrors()` failure; any guide/label containment failure; any detected cleat-volume overlap (defensive — should never trigger given §5.2/§5.3, but asserted); any layout overlap.

**Resulting hard minimum leg length** `O ≥ t + W + 3·tickLength`: **≈ 31 mm at 3 mm**, **≈ 60 mm at 6 mm** (thickness-dependent, far above the prior `2t`). **Hard minimum wall height** `H ≥ B + t + 3·tickLength`: **≈ 27 mm at 3 mm**, **≈ 48 mm at 6 mm**. Defaults `O=80, H=50` are valid at both thicknesses (the 6 mm wall-height margin, `50` vs `48`, is modest — see the marginal warning below).

**Warnings (producible but marginal):**
- `t < 2 mm` — thin stock; laminated strips fragile.
- Any cleat length within `1.2 × guideMinRun` of its guide minimum — marginal registration/short run (this flags the tight 6 mm default wall height).
- `W > O / 4` — cleat width is a large fraction of the leg; high interior intrusion.
- Base-cleat union `> 30 %` of `O²` — large prototype-scale interior intrusion.
- Standing manual-squaring / glue-dependency / experimental-status / lamination-order warnings.

No structurally important dimension is silently clamped or reduced.

## 17. Physical acceptance plan

Labeled everywhere **"Assembly and registration prototype — not a strength test."** After software implementation, the physical test observes: pieces cut and self-identify (labels + layout order); lamination layers glue up flat with the labeled layer outermost; guide ticks are legible and communicate the stop-short ownership; base cleats install at their ticks without colliding; the seam cleat pre-attaches to Wall A and its foot seats on base cleat A; Wall A then Wall B install by straight descent without forcing; the exterior seam closes acceptably; the cleats give usable repeatable registration (not merely a jig feel); a square and clamps reach every surface; squeeze-out is manageable; interior intrusion is acceptable at real scale; alignment holds after cure; and, honestly, whether the corner **feels worthwhile vs a finger joint**. **A successful build permits a J3 design review** for Concealed Cleat as a second Finger Box structural option; **an unsuccessful build stops or redesigns J2** — not pre-judged here.

**Required for the physical pass:** measured thickness from the actual sheet, a prior material test if settings are uncertain, correct focus, attended cutting throughout, inspection between passes where practical, ventilation, active fire watch, nearby suppression. No speed/power/pass values invented.

## 18. Fixture plan (representative — Designs geometry group)

Independent literal and coordinate checks (never comparing a helper only against its own returned value):

1. New template value normalizes independently; existing templates' goldens byte-identical (Finger Box incl. faux-dovetail, Sliding-Lid, trays, Drawer Cabinet, both coupons). 2. `t/O/H` finite and positive or explicit errors. 3. Default dimensions produce a valid model. 4. **Leg-length semantic is outside**: `Base = O × O` and `Wall A length = O` for the default draft. 5. `Wall A = O × H`. 6. `Wall B = (O − t) × H`. 7. Base cleat A dimensions `(O−t) × W` and installed footprint `x∈[t,O], y∈[t,t+W]`. 8. Base cleat B dimensions `(O−t−W) × W` and installed footprint `x∈[t,t+W], y∈[t+W,O]`. 9. **Base-cleat footprint intersection area is exactly 0** (independent rectangle-intersection computation). 10. **Base-cleat union area matches an independent hand calculation** (`1440 mm²` at 3 mm, `2340 mm²` at 6 mm). 11. Corrected 3 mm literal example (all §6.1 numbers, hand-derived). 12. Corrected 6 mm literal example (§6.2). 13. Seam cleat vertical range `z = B` to `z = H − t`; length `H − B − t`. 14. Seam cleat zero volume overlap with both base cleats (meet only at `z = B`). 15. Wall B contacts base cleat B's `x = t` exposed edge. 16. Wall B contacts the seam cleat's `x = t` exposed edge. 17. Base cleat A guide coords match the semantic `y = t` edge over `x∈[t,O]`. 18. Base cleat B guide coords match the semantic `x = t` edge over `y∈[t+W,O]` (starting at `t+W`, proving the stop-short). 19. Seam guide coords match the `u = t` edge over `z∈[B,H−t]`. 20. Guide ticks inside their panels and clear of cut paths. 21. Only the designated top/outermost cleat layer carries a label. 22. The labeled cleat face is the exposed (non-glued) face. 23. No label lies within any glue footprint. 24. One-sided engraving orientation explicit (engraved faces inward for panels; labeled cleat faces outward). 25. Piece count exact (9 at 3 mm, 6 at 6 mm); every cleat layer appears exactly once. 26. IDs deterministic/collision-free; panel order matches §13. 27. Corrected impossible dimensions each raise their named error (`O ≤ t+W`; `O−t−W < 3·tickLength`; `H ≤ B+t`; `H−B−t < 3·tickLength`); thin-stock and marginal-length warnings fire accurately. 28. Layout finite and non-overlapping; preview uses semantic positions only (no `result.svg`). 29. Storage/backup byte-identity; Production Settings unchanged; filename/MIME deterministic. 30. One golden pinned only after assertions 4–26 pass structurally. No Cartesian matrix — 3 mm and 6 mm only. New assertions belong to **Designs geometry**; no new total estimated in a design review.

## 19. Storage, Production Settings, protected boundaries

Session-only in `designDraft`; no change to `STORAGE_KEY`/`SCHEMA_VERSION`/`freshState()`/`loadState()`/`persist()`/`backupObject()`/`replaceData()`/`mergeData()`/import-export. Existing templates byte-identical (additive; no shared helper modified, only reused). **Production Settings:** measured `materialThickness` applies as for every template; cleat topology is a design choice, not a material setting; no clearance value exposed; no schema field. Protected boundaries (Finger Box builder, faux-dovetail behavior, Sliding-Lid, trays, Drawer Cabinet, both coupons, generic serializer, generic layout helper, storage/schema, import/export, Production Settings schema, evidence/promotion, Inventory, Projects, Pricing, Library, Test Grid, Material Test) are all read, never modified — the only shared surfaces used are the already-generalized `scoreGroupId` parameter and the generic `buildAssemblyLabelPaths()` (called directly, not via the shared per-template dispatcher).

## 20. Risk table (corrected)

| # | Severity | Risk | Disposition |
|---|---|---|---|
| 1 | Blocker (**fixed this pass**) | Two perpendicular base cleats overlapping by `W×W` near the corner; the prior "stop short by t" claim was false for `W > t` | Corrected ownership + coordinate model (§5) with a formal zero-intersection proof (§5.2); base cleat B length is now `O − t − W`; fixture 9 asserts intersection area = 0 |
| 2 | Blocker (**fixed this pass**) | Leg-length semantics mixed inside/outside (`L` called "inside" but used as `Base = L×L`) | Renamed to outside `O`, raw `cleatLegLength`, label *"Prototype outside leg length"*; clear interior reported as derived (§6); fixture 4 pins the outside semantic |
| 3 | Major (designed out) | Insufficient minimum-dimension check (`O > 2t`) allowing unbuildable or guide-less cleats | Corrected thickness-dependent minimums `O ≥ t + W + 3·tickLength`, `H ≥ B + t + 3·tickLength` (§16); errors, not silent clamps |
| 4 | Major (designed out) | Seam cleat colliding with base cleats, or trapping Wall B | 3-D proof: seam meets base cleats only at `z = B` (zero volume); Wall B descends into `x<t` unobstructed and is not trapped (§5.3, §10) |
| 5 | Major (designed out) | Guides/production geometry diverging via separate formulas | Single §6 formula set feeds model, preview, and fixtures; fixtures 17–19 check guide coords against the semantic edges independently |
| 6 | Major (designed out) | Labels trapped between glue faces / in glue zones | Only the top/outermost layer labeled, engraved face exposed; base/wall labels in cleat-free areas (§8.1); fixtures 21–23 |
| 7 | Major (designed out) | Single-layer strip mislabeled as structurally adequate | Classified "structural, glue-dependent"; layers derived to reach a `6 mm` target cross-section, not left at 1 at thin stock |
| 8 | Minor | Laminated layers hard to align by eye | Identical-strip lamination is low-risk; labeled-layer-outermost order stated; flagged as a physical-test observation, not assumed solved |
| 9 | Minor | Interior intrusion large at 6 mm (36.6 %) or small legs | Corrected §12 figures + `W > O/4` and `>30 %` warnings; scoped to the prototype, not extrapolated |
| 10 | Informational | Scope creep to Finger Box integration | Excluded; J3 hard-gated on a successful physical build |
| 11 | Informational | Nominal (not measured) thickness used | Warnings + acceptance plan require measured thickness from the actual sheet |

The prior draft's false statement that "stop-short-by-t formulas prevent overlap" is removed; no contradictory old formula remains elsewhere in the report.

## 21. Physical honesty

Software review cannot verify: glue strength, cleat durability, real registration quality, actual squareness after cure, exterior seam appearance, plywood glue-layer behavior, warping, real clamp/tape access, final interior usability at production scale, LightBurn import, or machine cutting. **No claim is made that this cleat construction is stronger, weaker, or equivalent to the finger joint** — every comparison is an explicitly-labeled glue-area/registration inference, not a measurement. No browser, LightBurn, machine, or physical check was performed.

## Final verdict

**READY FOR BOUNDED IMPLEMENTATION**

Retained only because the two load-bearing defects are now corrected with proofs: the base cleats are proven non-overlapping under an explicit coordinate model and single ownership rule (§5.2), the seam cleat is proven zero-volume against both base cleats with Wall B non-trapped (§5.3), the assembly sequence is straight-descent throughout (§10), and the leg-length semantic is unambiguously outside with corrected worked examples, intrusion figures, minimums, guides, labels, and fixtures. The prototype remains a small, additive, dedicated experimental generator — one combined SVG, plain rectangular and laminated-strip geometry only, mandatory guides and locally-built labels reusing existing generic helpers, no serializer or shared-function changes beyond the already-generalized `scoreGroupId`, no storage or Production Settings changes. **J3 (Concealed Cleat as a second Finger Box structural option) remains hard-blocked on a successful physical build of this exact corrected prototype** — this review does not, and cannot, supply that evidence.
