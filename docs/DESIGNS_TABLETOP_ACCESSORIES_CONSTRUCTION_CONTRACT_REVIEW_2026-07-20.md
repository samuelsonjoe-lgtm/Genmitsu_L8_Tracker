# Tabletop Accessories — Construction Feasibility & Geometry Contract Review

**Repository:** `C:\Genmitsu L8 Tracker`  
**Date:** 2026-07-20  
**Type:** Read-only construction-feasibility and geometry-contract review. No product code was written. No repository file was edited, staged, committed, pushed, reset, cleaned, stashed, checked out, moved, renamed, or deleted except this report file (explicit deliverable).

**Primary architecture context:** `docs/DESIGNS_TABLETOP_ACCESSORIES_ARCHITECTURE_REVIEW_2026-07-20.md` (Option B, wall-loop engine, finished-cavity-first sizing). That direction is **accepted**, with one load-bearing correction: the architecture’s “captured-floor rebate/lip” cannot be treated as a single-wall through-cut feature. This review replaces that hypothesis with laser-manufacturable constructions.

**Final verdict:** **READY FOR CODEX T1 IMPLEMENTATION PROMPT**

| Severity | Count |
|----------|------:|
| BLOCKER TO IMPLEMENTATION | 0 |
| IMPORTANT | 0 |
| POLISH | 0 |

---

## 1. Repository state

Commands run (read-only):

```text
git status -sb
git log -1 --oneline
git rev-parse HEAD
git rev-list --left-right --count origin/main...main
git diff --check
git diff
git diff --cached
git ls-files --others --exclude-standard
```

| Check | Result |
|-------|--------|
| Branch | `main` … `origin/main` |
| HEAD | `78765512f58d5a16eea89073958252bd5171b6a7` |
| `git log -1` | `7876551 Add Dice Tray storage and insert options` |
| Ahead/behind | `0 0` |
| Tracked working tree | **Clean** (no `M` tracked files) |
| Staged | Nothing |
| `git diff --check` | Clean |
| Architecture review present | `docs/DESIGNS_TABLETOP_ACCESSORIES_ARCHITECTURE_REVIEW_2026-07-20.md` |
| Untracked | Historical docs, LightBurn projects, `debug.log`, utility script — **untouched** by this review |
| Legacy generators | Unchanged (clean tree at DT1 baseline) |

Stated suite baseline **2454 / 0 across 29 groups** is accepted as the current validated suite at this commit; this review did not re-run the suite (construction-contract only).

---

## 2. Relevant existing geometry inspected

| Symbol / area | Role for this contract |
|---------------|------------------------|
| `buildFingerPattern` | Odd finger count, min width ≥ `max(2t, 4)`, preferred-width fit — **reuse for 90° wall and floor edges only** |
| `designPatternEdge` / `designSafePatternEdgePoints` / `buildSlidingLidBodyPanel` / `buildFingerPanel` | Proven 90° fingered rectangular panel outlines |
| `buildBoxModel` | Open finger box: bottom + four walls, alternating phase on adjacent edges, inside/outside dimension modes |
| `designPanelFromPoints` / `designPointsPath` | Generic closed orthogonal polygon panels |
| `layoutDesignPanelRows` | Generic sheet packing + gap/overlap checks |
| Gift Box / Sliding-lid Box | Same finger body math; rails/guides are product-specific, not the tray floor strategy |
| `buildTrayModel` (legacy Dice Tray) | Wall-to-base tabs; **centerline slot placement** (`thickness/2` offsets); walls **not** corner-joined; divider span from **nominal** inside — the physical failure mode |
| Divider Tray | Removable dividers sized from nominal inside span |
| Wall/base tab coupon | Clearance candidates for wall-to-base only — not corner squareness |
| Joint Fit Coupon | Finger clearance candidates for 90° joints |
| Finished View builders | Per-template projection; architecture’s shared helper remains correct **after** constructions are chosen |

**Do not modify any of the above.** New family is additive and parallel.

---

## 3. Physical prototype evidence (authoritative)

| Observation | Value | Construction implication |
|-------------|-------|---------------------------|
| Nominal material | 3 mm plywood | User must enter **measured** thickness |
| Measured thickness | **2.88 mm** | All slots/fingers must use measured `t`, not label thickness |
| Requested inside depth | **100 mm** | Nominal “requested interior” |
| LightBurn wall-to-wall (relevant) | ~**97 mm** | Finished clear face distance ≠ nominal |
| Physical inside face-to-face (divider span) | **96.73 mm** | Authoritative finished cavity for that assembly |
| Wall panels + divider generated length | **100 mm** | Divider = wall-panel length = raw nominal — **wrong source** |
| Divider oversize | ~**3.27 mm** | ≈ order of `t` / centerline error stack |
| Full long-wall length | **146 mm** | Matches full inside length ≈ 146 mm (different axis behavior) |
| Outside edge → wall-slot edge | **1.45 mm** | ≈ **half thickness** (1.44 mm) → **centerline slot math** |
| Wall-to-wall join | **None** (butt + glue only) | Corners lean/spread; squareness depends on glue-up |
| Divider slot vs outer-wall slots | **Intersecting bands** | Layout/topology hazard on base plate |
| Overall | Assembles, **not** premium/sellable | New family must not repeat |

**Regression requirements derived from this prototype** are locked in §19–§20 and the fixture section.

---

## 4. Rectangular floor candidates

Laser constraints: flat through-cut panels only; vertical cut edges; no CNC pocket; no routed groove; no automatic bevel; glue allowed; one structural thickness in phase 1.

### A. Finger-jointed floor-to-wall (open finger box)

| Criterion | Assessment |
|-----------|------------|
| How cut | One floor panel with fingers on four edges; four walls with bottom fingers + vertical corner fingers — same family as `buildBoxModel` |
| How assembled | Dry-fit corners and floor, square, glue; self-registers wall loop **and** floor |
| Through-cut only | **Yes** |
| Mechanically registers wall loop | **Yes** (corner fingers) |
| Preserves requested interior | **Yes** if dimension mode is **requested interior = finished clear cavity** between wall **inner faces**, with outer envelope derived as `inside + 2t` (same spirit as Finger Box `inside` mode). Floor thickness sits **under** the usable volume: usable height = wall height (inner) if walls rise from the top face of the floor; exact stack is defined in §8 |
| Cavity intrusion | Floor **defines** the bottom plane; no extra ledge loss if floor is the open-box bottom (not an internal shelf) |
| Works at 2.88 mm | **Yes** — existing finger math already thickness-parameterized |
| Part count | **5** (4 walls + floor); optional loose lid later |
| Glue-up | Moderate; proven workshop pattern |
| Clamps | Helpful on corners; fingers hold alignment better than butt walls |
| Visible exterior | Exposed finger corners and bottom edge fingers — intentional laser aesthetic |
| Strength | Strong for premium tray |
| Fit fields | `cornerJointClearance` (wall–wall vertical + wall–floor shared or split — see §6); kerf separate |
| Geometry complexity | Low — reuse proven helpers |
| Fixture burden | Medium — pin phases, cavity math, no panel overlap |
| Coupon | Corner+floor miniature required |
| Lids/dividers later | Excellent base for lids; dividers sit on floor / into floor slots cut from finished cavity |

### B. Floor panel with tabs into through-slots in walls

| Criterion | Assessment |
|-----------|------------|
| How cut | Floor inside loop; tabs out; slots near bottom of each wall |
| Through-cut only | Yes |
| Registers wall loop | **Weak** — still needs strong wall–wall joints; slots alone do not square a rectangle |
| Cavity | Floor thickness raises bottom; tabs may show if slots are through |
| Weakness | Bottom wall web reduced; corner fingers + floor slots **collide** near corners unless heavily constrained |
| Part count | 5 |
| Verdict | Secondary / special case only — **not** preferred for first premium rectangle |

### C. Floor on interior rail strips

| Criterion | Assessment |
|-----------|------------|
| How cut | 4+ rail strips + floor + 4 walls |
| Through-cut only | Yes |
| Registers wall loop | Only if rails are keyed; usually walls still need corner joints |
| Cavity loss | Rail thickness + clearance permanently |
| Part count | 9+ |
| Glue-up | High complexity |
| Verdict | Reject for first implementation (cost/complexity) |

### D. Layered-wall groove (two wall layers)

| Criterion | Assessment |
|-----------|------------|
| How cut | Inner + outer wall layers form a true channel for floor edge |
| Through-cut only | Yes, but **doubled wall stock** |
| “Captured” literally | Yes — only this or multi-layer rings create a real channel |
| Cost / appearance | Heavy; thick walls; slow cut |
| Verdict | **Deferred product construction**, not the shared phase-1 default |

### E. Floor under shell with wall tabs into floor (legacy tray style)

| Criterion | Assessment |
|-----------|------------|
| How cut | Base with slots; walls with bottom tabs |
| Registers wall loop | **No** — walls not joined to each other |
| Squareness | Glue-up dependent (physical prototype failure) |
| “Captured” | **No** — floor is a registration plate, not a captured channel |
| Verdict | **Rejected** as premium rectangle construction (legacy remains for Dice Tray only) |

### F. Other

No better first-phase option than **A** for a premium rectangular laser tray under the stated constraints.

---

## 5. Selected rectangular floor construction

### LOCKED: **Finger-jointed open-box floor (Candidate A)**

**Physical definition:**

1. Four vertical walls form a closed loop with **90° finger-jointed wall-to-wall corners**.
2. One floor panel is the **bottom** of the open box, finger-jointed to the **bottom edge of every wall**.
3. All parts are single-thickness, flat, through-cut panels.
4. Glue after dry-fit; no hardware in phase 1.
5. No rebate, groove, dado, or pocket is claimed or drawn.

**Why this wins:**

- Only candidate that **both** mechanically locks the wall loop **and** registers the floor using already-proven code paths (`buildFingerPattern` + finger panel builders).
- Matches preferred aesthetic (exposed fingers).
- Avoids the legacy “walls only register to the base” failure.
- Does not pretend a single-layer wall contains a routed channel.

**Usable cavity height:**  
`finishedUsableHeight = requestedWallHeight` when wall **inner clear height** is the user field and the floor top face is the cavity floor (walls’ bottom fingers engage the floor; the cavity volume sits above the floor’s top face). If the product UI labels “wall height” as **panel flat height**, the result summary must show both **panel height** and **usable height above floor** explicitly (they differ by how finger depth is counted — contract in §8).

---

## 6. Rectangular corner-joint contract

| Item | Contract |
|------|----------|
| Strategy | **Classic 90° interlocking finger joints** on vertical wall edges |
| Reuse | `buildFingerPattern`, edge phase ownership, `buildSlidingLidBodyPanel` / `buildFingerPanel` style outlining — **composed for tabletop wall loop**, not by calling legacy `buildBoxModel` as a black box if that freezes front/back naming; math reuse is mandatory, topology must be wall-loop based |
| Phase / ownership | Adjacent vertical edges use **opposite phases** so fingers interleave (same rule as Finger Box: e.g. front/back one phase, left/right opposite on shared corners) |
| Floor vs corner | Bottom edge of each wall is fingered to the floor; vertical corner fingers and bottom floor fingers meet at corner terminals using existing **terminal offset / join** rules — must not leave zero-web corners |
| Min finger width @ 2.88 mm | Existing rule: segment ≥ `max(2t, 4)` → **≥ 5.76 mm** preferred minimum web discipline; prefer target finger width ≥ ~8–12 mm when edge allows |
| Prior +0.10 mm finger-fit result | **Starting guess only** — not transferable without re-test on this exact wall+floor stack, measured thickness, and kerf workflow |
| Fit fields | **`cornerJointClearance`** (mm, signed allowed with warning) for finger interleave; optionally **same value** for floor fingers in phase 1 to reduce UI load, with code structured so floor clearance can split later |
| Kerf | **Separate** operator/LightBurn concern; not folded into clearance by default (match existing Designs convention) |
| First coupon | **Dedicated corner-plus-floor miniature** (not Joint Fit alone, not wall/base tab alone) |

---

## 7. Official floor terminology

| Term | Use? |
|------|------|
| Captured floor | **Do not use** for the selected construction — implies enclosure/channel not present in single-layer open-box fingers |
| Recessed / grooved / rebated floor | **Forbidden** unless a future layered product actually creates that geometry |
| Layered-channel floor | Reserved for deferred Candidate D only |
| Tab-registered floor | Describes hex / legacy-style slots, not the rectangle default |
| Finger-jointed floor | **Accurate** |
| Registered floor | Accurate but vague alone |
| Open-box floor | Accurate |

### LOCKED official term: **Finger-jointed floor**

**Definition (normative):**  
A single flat floor panel whose perimeter is cut with interlocking finger (or equivalent tab) geometry that mates the bottom edges of all walls of the shell, so the floor and walls co-register during dry-fit. The floor is the structural bottom of an open box. It is **not** a floating internal shelf, **not** a base-only slot plate with unjoined walls, and **not** a multi-layer channel.

Architecture review “captured floor” language is **superseded** for implementation prompts by this term.

---

## 8. Rectangular dimension contract

### Order of operations (mandatory)

1. **Inputs:** requested usable interior width `W_req`, depth `D_req`, usable wall height `H_req`, measured material thickness `t`, preferred finger width, joint clearance, kerf (reported, not silently baked unless product policy says otherwise).
2. **Corner joint geometry:** compute finger patterns on outer edge lengths (or on panel lengths derived below — must be consistent; prefer patterns on **outer envelope edges** as Finger Box does when using outside patterns, or document inside-mode mapping).
3. **Shell outer envelope (default product mode = requested interior is clear cavity):**  
   - `outerW = W_req + 2t`  
   - `outerD = D_req + 2t`  
   - Wall panel **flat width** along each side = corresponding outer edge length (front/back = `outerW`, left/right = `outerD`) for classic open-box finger topology matching `buildBoxModel`.
4. **Floor panel dimensions:** `outerW × outerD` before finger profiling; cut path bounds from fingered outline.
5. **Finished cavity (authoritative):**  
   - `cavityW = W_req`  
   - `cavityD = D_req`  
   - `cavityH = H_req` (usable above floor top face)  
   When implementation uses outside-first mode instead, finished cavity is `outer − 2t` and **must** be shown beside requested values.
6. **Wall panel flat height:** height of the wall blank such that after floor engagement, usable inner height = `H_req`. Document the exact relationship in code comments + result summary (panel height vs usable height).
7. **Actual cut-path bounds:** from fingered polygons (may differ slightly from nominal panel width/height due to finger depth).
8. **Divider span:** derived from **finished cavity** along the divider’s axis, minus explicit divider end clearance if any — **never** from raw `W_req`/`D_req` without going through the finished-cavity object.
9. **Insert span:** finished cavity footprint minus `2 × insertClearancePerSide` (and any storage region rules later).

### Can requested interior be preserved exactly?

**Yes**, as the definition of finished clear cavity in interior-primary mode, subject to:

- measured `t` accuracy,
- joint clearance not stealing clear span (clearance is on finger width, not on cavity width, in the Finger Box model),
- no internal rails.

If any future option adds rails, liners, or layered walls, finished cavity **shrinks** and both numbers must display.

### Disclosure rule

Whenever finished cavity ≠ requested interior (mode switch, intrusion, or bug), the result summary **blocks silent substitution**: show both, and block export if finished cavity is below requested without an explicit “allow smaller cavity” product rule (default: **error** if computed cavity &lt; requested for interior-primary mode — that indicates an engine bug).

---

## 9. Hex corner candidates

Assume six equal walls, 120° interior corners, vertical unbeveled laser edges, regular hexagonal floor, no lid/storage in first hex prototype.

### A. Ordinary box fingers “at 120°”

| Criterion | Assessment |
|-----------|------------|
| Physically valid with vertical square-cut edges? | **No** as a drop-in of `buildFingerPattern` |
| Why | Classic finger interleave is designed for **90° wall meeting** with coplanar joint seating along the corner. At 120° interior, square-cut wall ends do not form the same mating stack; “change the angle parameter” does not create valid 3D engagement without **bevels** or non-planar deformation |
| Manual bevel required? | Yes, if forced |
| Verdict | **Reject** for automated laser-only first hex |

### B. Through-tab corner joints (tab into adjacent wall end)

| Criterion | Assessment |
|-----------|------------|
| Possible? | Dubious: tab leaves one wall’s plane and must enter a slot in a wall 120° away — tab does not approach the slot face-normal without twist |
| Bevel? | Often needed for clean seat |
| Verdict | Reject as primary (possible decorative research later) |

### C. Alternating keyed corner joints (separate keys/splines)

| Criterion | Assessment |
|-----------|------------|
| Possible with vertical cuts? | **Yes** |
| Locks? | Mechanically yes if key geometry is tight |
| Part count | +6 keys |
| Appearance | Can be premium decorative |
| Complexity | Higher fixture + fit burden |
| Verdict | **Strong phase-2 upgrade**, not required for first hex prototype |

### D. Internal triangular corner blocks

| Criterion | Assessment |
|-----------|------------|
| Possible? | **Yes** |
| Angle registration | Excellent |
| Cavity loss | Non-trivial (block footprints) |
| Part count | +6 |
| Verdict | Viable premium path; **heavier** than needed for first hex |

### E. Floor-registered butt joints

| Criterion | Assessment |
|-----------|------------|
| Possible with vertical edges? | **Yes** |
| Manual bevel? | **No** for assembly; exterior may show a small geometric seam/gap at vertices because edges are square — disclose as expected laser aesthetic, not a defect |
| Mechanical wall–wall lock? | **Glue only** at corners |
| Floor registers angle? | **Yes** if each wall’s bottom is tabbed/fingered into the correct hex floor edge |
| Strength | Adequate for open trays with full glue; weaker than fingered rectangle corners |
| Visible seams | Corner butt lines + possible outer V-character at vertices |
| Cavity loss | Only wall thickness inset from outer hex — no block loss |
| Part count | **7** (6 walls + floor) |
| Glue-up | Moderate; floor is the jig |
| Coupon | 120° three-wall section + small full hex |
| Later lid/insert | Floor-registered shell supports inserts; lids need careful outer hex definition |

### F. Layered hexagonal ring stack

| Criterion | Assessment |
|-----------|------------|
| Possible? | Yes |
| Engine fit | **Different construction paradigm** — stacked rings, not wall-loop panels |
| Verdict | Separate product construction later; **not** the shared wall-panel engine default |

### G. Other

None better than **E** for first hex under “no bevel, laser-only, shared wall-loop engine.”

---

## 10. Selected hex corner construction

### LOCKED: **Floor-registered butt corners (Candidate E)**

**Physical definition:**

1. Six flat wall panels of equal length.
2. Adjacent walls meet as **glued butt joints** at 120° interior corners (no 120° finger interleave).
3. Each wall’s **bottom edge** is mechanically registered to the corresponding edge of a regular hexagonal floor via **tabs into through-slots** or **finger/tab pattern on each hex edge** (implementation may use a simplified tab-slot profile first for easier coupons).
4. The **floor outline establishes the hex angle and side positions**; walls do not free-float on glue alone.
5. Optional future upgrade: Candidate C keys or D blocks — **not** phase-1.

**Explicit split:**

| Shape | Corner strategy |
|-------|-----------------|
| Rectangle | 90° **finger joints** |
| Hexagon | **Floor-registered butt** (different joint strategy) |

Shared engine holds wall loops, floors, cavity math, dividers, inserts, layout, validation, FV — **not** one generic joint solver for both.

---

## 11. Hex floor construction

### LOCKED: **Hexagonal finger/tab-registered floor (not the rectangle’s four-edge finger box copy, not layered channel)**

| Item | Contract |
|------|----------|
| Floor outline | Regular hexagon derived from finished-cavity or requested-interior primary field (§12), expanded by wall thickness rules consistently |
| Wall-to-floor registration | Each of 6 sides: wall bottom tabs/fingers ↔ floor edge slots/fingers |
| Interference with corners | Floor joint geometry must **stop short of vertices** enough to leave glue faces / avoid triple-thickness pileup at corners; validate minimum web |
| Floor location | **Under** the wall bottoms as structural base (same role as open-box floor, hex topology) — not an internal shelf |
| Finished cavity | Inner regular hex after wall thickness; height above floor top face |
| Fit fields | `floorJointClearance` (may alias to shared joint clearance in UI phase 1); measured `t`; kerf separate |
| Coupon | Small hex floor + two or three walls |

**Why rectangle’s exact four-edge finger-floor recipe is not copy-pasted:**  
`buildFingerPattern` + rectangular panel builders assume **orthogonal** edge walks. Hex needs a **6-edge outline generator** and per-edge joint profiles along non-orthogonal plan directions. Concept (floor registers walls) is shared; **panel builders are joint-family-specific**.

---

## 12. Hex dimension terminology and formulas

### LOCKED primary user field: **Inside flat-to-flat**

Do **not** label a hex control simply “width.”

| Term | Meaning |
|------|---------|
| **Inside flat-to-flat** `F_in` | Clear distance between parallel inner wall faces (primary entry) |
| **Inside point-to-point** `P_in` | Clear distance between opposite inner vertices |
| **Inside side length** `a_in` | Clear length of one inner flat (cavity side) |
| **Outer flat-to-flat** `F_out` | Outer parallel-face distance of assembled shell |
| **Outer point-to-point** `P_out` | Outer vertex-to-vertex |
| **Outer side length** `a_out` | Outer wall length along one flat (panel length basis) |

### Regular hexagon relations (plan geometry)

For side length `a`:

- flat-to-flat \( F = a\sqrt{3} \)
- point-to-point \( P = 2a \)

Equivalently from flat-to-flat `F`:

- \( a = F / \sqrt{3} \)
- \( P = 2F / \sqrt{3} \)

### Mapping to product dimensions (interior-primary)

Given requested inside flat-to-flat `F_req` and thickness `t`:

1. Finished cavity flat-to-flat `F_cav = F_req` (interior-primary).
2. Finished cavity side \( a_{cav} = F_{cav}/\sqrt{3} \).
3. Finished cavity point-to-point \( P_{cav} = 2F_{cav}/\sqrt{3} \).
4. Outer flat-to-flat \( F_{out} = F_{cav} + 2t \) (walls outside cavity; confirm with wall centerline model in implementation — if walls are centered on a mid-hex, document; **preferred model:** outer face outside cavity by `t`, so outer flat-to-flat = cavity flat-to-flat + 2t).
5. Wall panel length along each outer flat derived from outer hex side \( a_{out} = F_{out}/\sqrt{3} \) (or from outer polygon edge length consistently).
6. Floor outer hex matches outer envelope used for wall bottoms.
7. Divider/insert spans use **finished cavity** polygon, not raw labels.

Result summary must list at least: `F_cav`, `P_cav`, `a_cav`, `F_out`, wall panel length, floor flat-to-flat, usable height.

---

## 13. Shared-engine boundary

After construction choices, the shared engine **should** own:

| Shared | Notes |
|--------|-------|
| Ordered wall loop | Edge descriptors, normals, prev/next |
| Edge / corner descriptors | Including `joinType` enum |
| Panel descriptors | Semantic roles |
| Shape outline generators | Rectangle; regular hexagon |
| Finished-cavity computation | Authoritative object consumed by all children |
| Floor descriptor | Outline + joint role; not rebate math |
| Divider descriptor | Length from finished cavity only |
| Insert descriptor | `physical` + material tag + clearance |
| Projection helper | One isometric/helper for new family FV |
| Shell renderer | Walk wall loop generically |
| Panel layout | Reuse `layoutDesignPanelRows` |
| Validation layers | Geometry, manufacturing limits, layout |
| SVG serialization conventions | Colors, scale, no FV IDs |

---

## 14. Product- / joint-specific boundary

| Not shared as one solver | Owner |
|--------------------------|--------|
| 90° fingered corner + floor edge builder | Rectangle joint module (wraps `buildFingerPattern`) |
| Hex wall-bottom ↔ floor registration builder | Hex joint module |
| Floor connection strategy selection | Per `joinType` / product |
| Product form fields & warnings | Per template |
| Lid strategies | Later products |
| Layered-channel walls | Deferred product |
| Corner keys / internal blocks | Deferred hex upgrade |
| Legacy Dice Tray wall-to-base | **Untouched** legacy only |

**Do not build a generic “joint solver”** that claims to emit both 90° fingers and 120° fingers from one angle parameter.

---

## 15. Validation contract

### Blocking errors (both shapes unless noted)

- Nonpositive or nonfinite dimensions / thickness / clearances  
- Impossible requested cavity (≤ 0 after any required inset)  
- Material thickness too large for product size / finger minimums  
- Finger count/width impossible (`buildFingerPattern` null)  
- Floor connection web too narrow  
- Floor tabs/slots collide with rectangle corner fingers or hex vertices  
- Hex connector collision (when keys/blocks exist later)  
- Self-intersecting shell or panel outline  
- Wall-loop edge count ≠ shape edge count  
- Open contours / duplicate cut paths in production SVG  
- Finished cavity &lt; requested in interior-primary mode (engine inconsistency)  
- Divider span ≠ finished-cavity span − explicit clearance (tolerance 1e-6 mm class)  
- Insert footprint larger than finished cavity  
- Physical panel AABB overlap in layout  
- Missing required panels / wrong edge count  

### Warnings

- Loose/tight joint clearance vs thickness  
- Large outer envelope vs work area / sheet  
- Hex outer corner seam aesthetic (square-cut butts)  
- Measured thickness differs from common nominal label (if detectable)  
- First-cut scrap reminder for new joint clearance  

### Shape-specific

- Rectangle: odd finger count failure, phase mismatch between adjacent edges  
- Hex: non-regular inputs rejected; primary field must be flat-to-flat (or explicit mode enum)

---

## 16. Production SVG contract

| Rule | Requirement |
|------|-------------|
| Structural cuts | Red (`#FF0000` family per existing LightBurn convention) |
| Score / alignment | Blue guides only when product opts in |
| Assembly labels | Green score/engrave group, gated by control, outside cut group |
| Scale | Exact mm, no viewBox “fit” scaling of production paths |
| Determinism | Stable path order and rounding (`designRound`) |
| Contours | Closed; no duplicates |
| Finished View | **No** FV element IDs, titles, or screen-only geometry in download SVG |
| Transforms | No unsupported/hidden transform stacks that alter cut scale |
| Claims | **No** path claiming pocket/rebate/groove that is not a real multi-part channel |
| IDs | Explicit panel IDs + semantic roles (wall ordinal, floor, divider, …) |
| Material separation | Non-wood inserts remain measurement-only unless physical wood panel |

### Construction coupons

| Mode | Recommendation |
|------|----------------|
| Phase 0 / first delivery | **Separate Designs templates** (or coupon product entries) — same pattern as wall/base tab + joint-fit coupons |
| Engine fixtures | Synthetic coupon geometry also asserted in `?selftest=` without requiring UI |
| Embedded in every tray result | **No** by default (clutters production); optional checkbox later |

---

## 17. Finished View contract

Shared projection helper from the architecture review **remains appropriate**.

Must communicate:

- Actual wall count (4 or 6)  
- Corner strategy truthfully (fingers vs butt; **no** drawn bevel/groove that is not built)  
- Floor as bottom plate under walls  
- Requested interior vs finished cavity callouts in UI metrics (not fake 3D text that implies CNC rebate)  
- No NaN/Infinity; no FV geometry in production SVG  

| View | Use |
|------|-----|
| Three-quarter / isometric | Shell massing, corner style, floor under walls, height |
| Top-down | Cavity footprint, divider/insert regions, hex flats orientation |

---

## 18. Fixture organization

| Group | Scope |
|-------|--------|
| `tabletop-engine` (name illustrative) | Wall loop rect+hex outlines; finished-cavity math; divider-from-cavity; insert bounds; FV projection coordinates; manufacturing limits — **coordinate-level** |
| `rectangular-premium-tray` | Product fields, 90° fingers + finger-jointed floor, result summary, production SVG pins when stable |
| `hex-premium-tray` | Flat-to-flat primary field, butt corners, hex floor registration, cavity formulas |
| Construction coupons | Own small groups if user-facing templates |

**Do not** dump the new family into monolithic `runDesignGeometryFixtures` unless a single regression pin is required.

### Mandatory numeric regression fixtures (from physical failure)

Using **t = 2.88 mm**, requested depth **100 mm**:

1. Finished cavity depth must be an **explicit computed field**; divider length must equal that field (± clearance), **not** 100 when cavity is ~96.7–97.1.  
2. Engine must **fail or warn loudly** if divider length == wall panel length == nominal while cavity &lt; nominal (legacy failure signature).  
3. Fixture with centerline-style half-thickness **1.44–1.45 mm** offsets must prove clear-face reporting is **not** half-thickness mistaken for cavity.  
4. Rectangle product must assert **wall–wall finger presence** (not wall-to-base-only topology).  
5. Pin: `dividerLength === finishedCavitySpan - explicitClearance` for a synthetic shell with `t=2.88`, `D_req=100`.

The new engine **must not** reproduce the bad legacy 96.73-vs-100 result as its happy path; fixtures prove it cannot ship silently.

---

## 19. Physical coupon plan

| Coupon | Proves |
|--------|--------|
| 90° finger-corner coupon | Vertical finger fit at measured t; clearance starting point |
| Floor-to-wall finger coupon | Bottom joint fit; web integrity |
| **Combined corner+floor miniature (required first)** | Full rectangle registration; squareness vs legacy tray |
| Divider-to-finished-cavity coupon | Divider cut to cavity, not nominal |
| 120° three-wall hex section | Butt corners + floor edge registration on a partial hex |
| Small complete rectangular shell | Product-scale dry-fit before sellable claims |
| Small complete hex shell | Hex assemble-ability and cavity measure |

No power/speed settings in this contract.

---

## 20. Physical prototype checkpoints

1. **Before shipping rectangular premium tray:** pass combined corner+floor coupon + small complete rectangular shell; measure cavity vs summary.  
2. **Before shipping hex:** pass three-wall section + small hex shell; measure flat-to-flat cavity.  
3. **Before dividers on new engine:** divider coupon fits finished cavity with intended clearance.  
4. **Before additional products (deck box, vault):** rectangle construction proven; engine not still hypothetical.

---

## 21. First implementation sequence

### LOCKED: **Option 4 — Construction coupons first, then engine + rectangle, then hex**

Ordered steps:

1. **Coupon templates / synthetic coupon fixtures** for 90° corner+floor and clearance (reuse Joint Fit lessons; new combined coupon).  
2. **Shared engine (T1)** — wall loop, outlines, finished cavity, divider/insert primitives, manufacturing limits, FV projection — **no** multi-product UI yet.  
3. **Rectangular premium tray product (T2)** on the engine; physical shell test.  
4. **Hex product (T3)** with floor-registered butts; physical hex test.  
5. Later products only after 1–4.

**Rejected:** Option 3 (rect+hex together).  
**Rejected as sole path:** Option 1 without coupons (engine-only delays workshop feedback).  
**Option 2** is acceptable only if step 1 coupons are explicitly inside the same phase **before** calling the tray sellable — Option 4 states that gate more clearly.

---

## 22. Locked-decision table

| Decision | Locked value |
|----------|----------------|
| Official family name | **Tabletop Accessories** |
| Architecture option | **Option B** — shared geometry engine + separate product templates (+ lightweight registry for new family) |
| Rectangular corner joint | **90° finger joints** (reuse `buildFingerPattern` composition) |
| Rectangular floor construction | **Finger-jointed floor** (open-box bottom; Candidate A) |
| Official floor terminology | **Finger-jointed floor** (not “captured,” not “recessed,” not “grooved”) |
| Hex corner joint | **Floor-registered butt joints** (Candidate E) |
| Hex floor construction | **Hexagonal tab/finger-registered floor** under walls |
| Hex primary dimension field | **Inside flat-to-flat** |
| Divider sizing source | **Finished cavity only** |
| Insert sizing source | **Finished cavity only** (+ clearances) |
| Fit-clearance fields | Measured `t`; `cornerJointClearance` / shared joint clearance; optional later floor split; **kerf separate** |
| First implementation sequence | **Option 4** — coupons → engine → rectangle product → hex product |
| Required first physical coupon | **Combined 90° corner + floor miniature** |
| Required first complete prototype | **Small rectangular finger-jointed shell** |
| Deferred | Layered-channel walls; hex corner keys/blocks; lids; storage bays; multi-material structural stacks; campaign multi-sheet |
| Non-goals | No CNC pockets; no automatic bevels; no legacy Dice Tray changes; no schema/storage/network changes; no generic 120° finger solver |

---

## 23. Findings by severity

### BLOCKER TO IMPLEMENTATION
None remaining — constructions are defined and laser-manufacturable.

### IMPORTANT
None — terminology and dimension contracts are locked above. (Architecture “captured rebate” ambiguity is **resolved** by supersession, not left open.)

### POLISH
- Optional later hex exterior corner keys as decorative premium upgrade.  
- Optional UI dual-display of hex point-to-point derived from flat-to-flat.

### NOT A DEFECT
- Rectangle and hex using **different** corner strategies.  
- Visible hex butt seams without bevel.  
- Exposed finger aesthetics on rectangle exterior.  
- Legacy Dice Tray remaining wall-to-base.

---

## 24. Exact final verdict

# READY FOR CODEX T1 IMPLEMENTATION PROMPT

(T1 here means: coupons + shared engine groundwork per §21, then rectangular product — not “hex in the first prompt.”)

---

## 25. Remaining user decision

**None required** for construction geometry. Optional product preference only (not blocking):

- Whether phase-1 rectangle joint clearance UI is one field or split floor/corner (default: **one field**).

---

## 26. Whether Claude is needed again

**Not required** for another construction-feasibility pass unless physical coupon tests falsify a locked choice (e.g. finger floor squareness no better than legacy, or hex butt unacceptably weak). Architecture Option B stands; this document supplies the missing physical construction lock.

---

## 27. Whether Codex may begin

**Yes** — Codex may begin an implementation prompt grounded in:

1. This construction contract (normative for joints/floors/dimensions), and  
2. The architecture review (Option B, wall loop, finished-cavity-first, fixture split), with **“captured floor rebate” replaced by “finger-jointed floor” / hex floor-registration** as defined here.

First Codex prompt should target **§21 steps 1–2** (coupons + engine), not full hex product.

---

## 28. Confirmation: no unauthorized repository mutation

This review:

- Did **not** stage, commit, push, reset, clean, stash, or checkout.  
- Did **not** modify `index.html`, README, CHANGELOG, or any generator.  
- Wrote **only** this deliverable report:  
  `docs/DESIGNS_TABLETOP_ACCESSORIES_CONSTRUCTION_CONTRACT_REVIEW_2026-07-20.md`  
- HEAD remains **`7876551`**, branch **`main`**, ahead/behind **`0 0`**, tracked tree clean aside from this new untracked report.

---

## Appendix A — Why architecture “captured floor” was not adopted literally

A true **capture channel** requires either:

- CNC/router depth (out of scope),  
- **layered walls** (Candidate D — deferred), or  
- mislabeling a base plate as “captured.”

Single-thickness finger-jointed open-box floors are the honest premium construction for diode-laser flat stock. Calling them “captured” would recreate the documentation failure mode the project is trying to leave behind.

## Appendix B — Legacy divider length math (illustrative)

If slots are placed on **centerlines** inset ~`t/2` from outer base edges, clear face-to-face ≈ outer_inside_band − contributions of wall thickness seating — empirically **~96.7–97 mm** for 100 mm nominal at t≈2.88. Generating a divider at **100 mm** ignores that. The new engine’s only legal divider length source is the **finished-cavity descriptor** produced after shell+floor assembly math.

---

*End of construction contract review.*
