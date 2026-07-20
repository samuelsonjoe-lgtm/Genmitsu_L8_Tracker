# Designs Gift Box — Architecture & Product-Design Review

**Date:** 2026-07-19  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Review type:** Read-only architecture and product-design (no implementation)  
**Reviewer role:** Independent design architecture review  
**Baseline (committed):** `7a9ec6d` — *Add multi-machine manager and switching*  
**Working tree note:** Uncommitted Phase **M3** changes exist on `index.html` / docs. **M3 is not evaluated as Gift Box scope** and is recorded only for repository honesty.

---

## 0. Executive recommendation

| Question | Answer |
|----------|--------|
| Is G1 architecturally ready? | **Yes** — as a **new Designs template** on the existing A-pipeline, with a **hard G1 product boundary**. |
| Recommended first joint/lid boundary | **Exposed finger joints only** + **lift-off or surface-mounted hinged** lid (flush or overhanging) + **score/engrave hardware guides** (no mortises) + **optional locating frame** for lift-off. |
| Dual wall/lid thickness in G1? | **Defer to G2** unless Joe has a hard requirement; G1 uses **one material thickness** for all structural panels (same as finger-box / open-top). |
| Joint-strategy interface in G1? | **No** — local finger reuse only; document a future joint contract without implementing it. |
| Storage / schema / multi-machine? | **No changes.** Designs remain session-only drafts. |

**Verdict for implementers:** Proceed to G1 after Joe answers open product decisions in §25. Do **not** bolt Gift Box onto `finger-box` or `sliding-lid-box` as a mode switch that mutates their contracts.

---

## 1. Repository state

### Commands recorded (review session)

| Check | Result |
|-------|--------|
| `git rev-parse HEAD` | `7a9ec6d5c7dce0ab02a00b0e81f8e0f4fcd1c9b5` |
| `git log -1 --oneline` | `7a9ec6d Add multi-machine manager and switching` |
| `git rev-list --left-right --count origin/main...main` | `0 0` |
| `git status -sb` | `## main...origin/main` + modified: `index.html`, `README.md`, `CHANGELOG.md` |
| `git diff --check` | Clean |
| `git diff --stat` | `index.html` ~+119/−32; `README.md` +2; `CHANGELOG.md` +4 |
| `git diff --cached` | Empty |
| `git ls-files --others --exclude-standard` | (empty for product sources; review may add this doc only) |

### Committed baseline vs M3 working tree

| Item | Status |
|------|--------|
| **Actual committed baseline for Gift Box design** | **`7a9ec6d`** (M2 multi-machine manager landed) |
| **M3 working-tree changes** | Present, **uncommitted**, **out of Gift Box scope** — do not treat as Designs infrastructure for this feature |
| **Modified files (M3)** | `index.html`, `README.md`, `CHANGELOG.md` |
| **Untracked product sources** | None observed for Designs |
| **This review changed product files?** | **No** (report write only under `docs/`) |
| **M3 evaluated as Gift Box?** | **No** |

### Existing Designs fixture baselines (relevant)

| Area | Location / notes |
|------|------------------|
| Designs suite | `tests/designs-geometry-fixtures.html` (~2071 assertions historically; count grows with phases) |
| SVG / LightBurn contract | `assertDesignSvgLightburnContract`, cut/score path rules, red `#ff0000` cut / blue `#0000ff` score |
| Finger-box / open-top / tray | Geometry, labels, Finished View, multi-sheet, nesting |
| Sliding-lid | Separate template; hinge-like **guides** are **not** present — lid is **sliding rail geometry** |
| Drawer cabinet | Nested drawers, multi-sheet, labels |
| Joint Fit Coupon / Wall-Base Tab Coupon | Fit / tab systems |
| Cleat prototypes | French cleat, corner block (structural **squaring aids**, not gift-box joints) |
| Hidden labels | `hiddenLabels` / `designs-hidden-labels` fixtures |
| Design → Project handoff | `createProjectFromActiveDesign` path + fixtures |
| Direct `file://` | Full suite requires browser/DOM |
| Unit parity | mm / in conversion helpers + fixtures |

**Do not treat current uncommitted M3 as part of Gift Box.**

---

## 2. Existing Designs architecture

### 2.1 Pipeline shape (A-pipeline)

Observed structure (names from live `index.html` / fixtures):

1. **Template registry** — design type id → defaults, form schema, generator, Finished View, export naming.
2. **Draft state** — session-only; **not** in multi-machine workshop JSON / backups.
3. **Form schema + validation** — numeric fields, units, material thickness, kerf, fit, spacing.
4. **Unit conversion** — mm ↔ in for storage/display consistency.
5. **Geometry generation** — pure-ish builders → panel list + paths.
6. **Panel layout / nesting** — sheet packing, multi-sheet when needed.
7. **SVG serialization** — `serializeDesignSvg` (and related): exact scale, layer colors, deterministic bytes where fixtures pin them.
8. **Finished View** — semantic 3D-ish / orthographic **preview**, not production geometry.
9. **Production preview** — 2D sheet layout of cut/score paths.
10. **Design → Project handoff** — optional; creates project from active design metadata + export association.
11. **Fixtures** — browser-run HTML harness; golden SVG snippets / counts / labels.

### 2.2 Inventory checklist

| Concern | Present? | Gift Box relevance |
|---------|----------|-------------------|
| Design type registry | Yes | **New type** `gift-box` (recommended id) |
| Design defaults | Yes | New defaults block |
| Form schemas | Yes | New schema; do not overload finger-box |
| Validation | Yes | G1-specific validators |
| Unit conversion | Yes | Reuse as-is |
| Material thickness | Yes | G1: single thickness (see §6) |
| Kerf | Yes | **Never bake into geometry**; same rule as all Designs |
| Joint-fit adjustment | Yes | Finger flank clearance / joint fit coupon values |
| Finger-joint geometry | Yes | **Primary G1 structural joint** |
| Finger-count selection | Yes | Reuse selection rules + min-size validation |
| Panel-edge ownership | Yes | Critical for closed box + lid interfaces |
| Panel layout / nesting | Yes | Reuse packer |
| Multi-sheet | Yes | Allowed if panels exceed sheet; **not a G1 feature goal** to design large multi-sheet products |
| SVG layer / color contract | Yes | Cut red, score blue; engrave path if already supported |
| Assembly labels | Yes | Reuse label system |
| Hidden-label support | Yes | Optional; reuse if labels clutter |
| Finished View | Yes | Closed + open-lid views for Gift Box |
| Production preview | Yes | Reuse |
| Design → Project | Yes | Optional G1; low risk if metadata-only |
| Export filenames | Yes | Pattern per template |
| Fixture infrastructure | Yes | New fixture group |

### 2.3 Existing generators (comparative)

| Template | What it is | Gift Box reuse |
|----------|------------|----------------|
| **Open-top box** | 5-panel finger box, no lid | Wall finger patterns, open top as reference for “base shell” |
| **Finger box** | Closed box; lid modes **open** / **loose** only | Closest structural cousin; **loose lid ≠ hinged gift lid** |
| **Drawer cabinet** | Nested carcass + drawers | Multi-panel assembly labels, nesting complexity — **not** lid/hinge |
| **Sliding-lid box** | Rails, lid travel, clearance | **Do not** reuse rail geometry as hinges; **do** study clearance / guide-layer discipline |
| **Tray** | Shallow open form | Simpler panels; less relevant |
| **Joint Fit Coupon** | Finger fit test | **Required workflow** for G1 fingers |
| **Wall / base tab systems** | Tab-and-slot style coupons / panels | Future **hidden tab-slot** (G3), not G1 |
| **Corner block / cleat** | Squaring / hang | **Not** gift-box corner construction in G1 |

---

## 3. Reusable helpers (safe)

Reuse **directly** (or thin wrappers that do not change helper contracts):

| Helper / system | Safe use for Gift Box |
|-----------------|----------------------|
| Unit conversion | All dimension fields |
| Material thickness + kerf fields | Same semantics as finger-box |
| Joint-fit / finger flank clearance | Same as finger-box |
| `designBoxDimensions` / internal↔external conversion | Core dimension model |
| Finger pattern builders (`buildFingerPattern`, edge finger profiles) | Structural walls |
| Panel edge ownership conventions for finger boxes | Wall-to-wall joints |
| Nesting / multi-sheet packer | Sheet layout |
| `serializeDesignSvg` + LightBurn color contract | Export |
| Assembly label + hidden-label machinery | Labels |
| Finished View framing (semantic, not production) | Closed / open previews |
| Design → Project handoff (if included) | Metadata only |
| Validation patterns for “too small for fingers”, sheet overflow | Adapt messages |
| Sliding-lid **guide-layer discipline** (score vs cut separation) | Hinge/latch **guides** |

---

## 4. Unsafe reuse points

| Temptation | Why unsafe |
|------------|------------|
| Extend `finger-box` with “hinged” lid mode | Changes form schema, SVG bytes, Finished View, fixtures for an established template; high regression risk |
| Reuse **sliding-lid rails** as hinge geometry | Wrong kinematics; rails are cut features for lid travel, not hardware guides |
| Treat **corner-block** as gift-box structural joint | Different purpose (squaring / cleat ecosystem) |
| Bake **kerf** into hinge gaps or lid setback | Violates Designs kerf contract; kerf is LightBurn/operator concern |
| Auto **mortise / pocket** depth from “hinge thickness” | Untested 3D removal; G1 explicitly score/engrave only |
| Claim **decorative dovetail overlays** are structural dovetails | Product integrity / safety of expectations |
| Shared **joint-strategy interface** before second joint ships | Premature abstraction; YAGNI |
| Dual thickness + multi-material grouping without layout rules | Confuses nesting and LightBurn material assignment |
| Photoreal hinges in Finished View | Implies false precision; prefer schematic |
| Network / hardware library of hinge SKUs | Non-goal; user-entered geometry only |

---

## 5. Recommended G1 product contract

### Name

**Gift Box** (template id recommendation: `gift-box`)

### User jobs (first release)

Help a workshop user generate **laser-ready decorative keepsake / jewelry / dice / card / memory / wedding / presentation boxes** with **useful physical output**, not speculative breadth.

### G1 construction choices

| Control | G1 options |
|---------|------------|
| Plan form | **Rectangular** only |
| Dimension mode | **Internal** or **external** finished dimensions |
| Structural joints | **Exposed finger joints only** |
| Lid type | **Lift-off** \| **Surface-mounted hinged** |
| Lid appearance | **Flush** \| **Overhanging** |
| Locating frame | **Optional** (primarily lift-off; see §10) |
| Hinge geometry | **User-entered** (see §8–9) |
| Hinge guides | Score/engrave leaf rectangles + screw centers |
| Latch / magnet | Optional **center guides** (score/engrave), not cut pockets |
| Assembly labels | Optional, default on |
| Finished Views | **Closed** + **Open-lid** |
| Export | Exact-scale SVG; cut / score / (engrave if contract allows) layers |

### Explicit G1 non-goals

True dovetails, living hinges, hidden barrel hinges, wooden pin hinges, automatic deep mortises, hardware SKU libraries, foam inserts, complex compartments, curved/domed lids, puzzle locks, multi-sheet product design focus, product naming, M4 machine filtering, inventory CSV, desktop packaging, network deps, storage-key or schema changes.

---

## 6. Dimension model

### 6.1 Finished dimensions

- User enters **L × W × H** as either:
  - **Internal finished** cavity (usable space), or  
  - **External finished** outer envelope of the **closed box shell** (walls + base; lid rules below).
- Conversion via existing box dimension helpers (same mental model as finger-box / open-top).

### 6.2 What “finished” means for Gift Box

| Mode | Interpretation (G1 recommendation) |
|------|-------------------------------------|
| Internal | Clear inside length/width/height **with lid closed**, before decorative skins (skins are G2). Locating frame **reduces** usable cavity — disclose in UI. |
| External | Outer footprint and height of **assembled closed box** including lid stack as defined by flush/overhang rules. |

### 6.3 Material thickness

| Field | G1 |
|-------|-----|
| Wall / base / structural lid thickness | **One** `materialThickness` for all structural panels |
| Separate lid thickness | **Defer to G2** unless Joe requires it immediately |
| Kerf | User-entered; **not** applied in geometry |
| Finger-joint fit | User-entered; same as finger-box |
| Lid clearance | Explicit field (see §7) |
| Panel spacing | Nesting gap on sheet |
| Units | mm / in |

**Why defer dual thickness:** Separate lid stock forces material grouping, dual nest groups, dual fit coupons, and Finished View thickness honesty. Architecturally safe later; not required for a useful first keepsake box if lid and walls are same ply.

### 6.4 Safe defaults (starting values, not hardware law)

Label all defaults as **starting values**:

| Parameter | Suggested start (mm) | Note |
|-----------|----------------------|------|
| Lid clearance (radial / perimeter for lift-off) | 0.3–0.5 | Depends on kerf + sanding; user tunes |
| Locating frame fit clearance | 0.2–0.4 | Per side vs lid inner |
| Hinge corner inset | ≥ 12–15 | Edge strength on thin ply |
| Screw edge margin | ≥ 6–8 | Warn below |
| Overhang (per side) | 2–4 | Aesthetic; affects hinge alignment |
| Finger count | Existing finger-box heuristics | Clamp by edge length |

---

## 7. Lid geometry

### 7.1 Lift-off + locating frame

- **Shell:** four walls + base, finger-jointed (open top).
- **Lid:** flat panel, flush or with **overhang** beyond outer walls.
- **Locating frame (optional):**
  - Prefer **four rails** or a **picture-frame** of four pieces over a single fragile closed ring if thin stock — implementation may generate four rails for glue access.
  - Frame sits on **inner** top of walls (or slightly proud — product choice; recommend **flush with wall top** or **recessed by one thickness step** only if documented).
  - **Clearance:** frame outer = internal cavity − 2×frameClearance; frame width (rail section) user or default; **block** if rail width &lt; min (e.g. 6 mm) or if frame leaves unusable cavity.
- **Hinges:** none for pure lift-off.

### 7.2 Surface-mounted hinged — flush lid

- Lid outer footprint ≈ **outer box footprint** (flush).
- **Back wall / lid gap:** required so barrel can sit without binding.
  - Model as user **closed hinge gap / barrel offset** + optional small **rear setback** of lid or back-wall top relief **as score guide only** in G1 (no auto pocket cut).
- **Barrel clearance:** validate that closed gap ≥ f(barrel diameter, leaf thickness assumptions); if insufficient → **block** or strong **warn** (recommend **block** when computed interference is geometric certainty).
- **Opening rotation:** Finished View open-lid assumes ~90° schematic; do not claim full 180° clearance analysis.
- **Hinge guides:** on **lid underside** and **back wall outer** (or top edge region) — both, for dry-fit alignment.

### 7.3 Surface-mounted hinged — overhanging lid

- Lid larger than outer shell by overhang on front/sides (and optionally rear).
- **Hinge alignment:** barrel axis must remain consistent; rear overhang can **misalign** surface leaves — recommend:
  - default **no rear overhang** for hinged mode, or  
  - explicit **rear overhang** field with validation that hinge leaf still lands on back wall.
- Flush vs overhang must be **visible** in Finished View.

### 7.4 Locating frame vs hinges

- **Default recommendation:** locating frame **enabled by default only for lift-off**; **off by default for hinged**.
- If both enabled: validate **frame does not intersect hinge leaf zones**; **warn** or **block** on overlap.
- Frame does not replace latch/magnet guides.

### 7.5 Where hinge guides live

| Surface | G1 recommendation |
|---------|-------------------|
| Lid (inner face) | Leaf outline + screw centers (score/engrave) |
| Back wall (outer face near top) | Matching leaf outline + screw centers |
| Side walls | No hinge leaves |
| Front wall | Optional latch/magnet **center only** |

---

## 8. Hinge input model (user-entered hardware)

Do **not** assume a brass-hinge SKU.

### 8.1 Full candidate field list

| Field | Purpose |
|-------|---------|
| Hinge count | 1 / 2 / 3 (typical) |
| Overall length | Leaf span along hinge axis |
| Leaf width | Projection from barrel onto panel |
| Leaf height | If distinct from length (often length = along axis) |
| Barrel diameter | Clearance / gap |
| Closed hinge gap / barrel offset | Closed-box lid–wall relationship |
| Distance from box corners (inset) | Placement |
| Screw-hole count per leaf | Guide pattern |
| Screw-hole spacing | Centers along leaf |
| Screw-hole / center-mark diameter | Engrave mark size |
| Surface-mounted vs recessed | G1: **surface-mounted only** |

### 8.2 G1 required fields (minimal useful set)

| Field | G1 |
|-------|-----|
| Hinge count | **Required** (hinged mode) |
| Overall length (along axis) | **Required** |
| Leaf width | **Required** |
| Barrel diameter | **Required** |
| Closed hinge gap | **Required** (or derived with user override) |
| Corner inset | **Required** |
| Screw count per leaf | **Required** (default 3) |
| Screw spacing | **Required** if count &gt; 1 (or auto-even with override) |
| Center-mark diameter | **Required** (default small, e.g. 1–1.5 mm mark) |
| Leaf height distinct | **Optional** — only if product needs non-rectangular leaves; else leaf = L × W rectangle |
| Recessed | **Out of G1** |

### 8.3 G1 hinge policy

- Prefer **score/engrave guides**, not cut mortises.
- No deep recess geometry without explicit tested depth controls (future).
- No assumption of “standard” small brass sizes.
- Validate count/spacing vs **back-wall width**.
- Warn screw marks too close to plywood edges / finger joints.
- Distinguish **construction guides** from **cut** geometry in UI + SVG layers.

---

## 9. Hinge-guide geometry

### 9.1 Generated artifacts (G1)

1. **Leaf rectangle** (score) — outer envelope of each leaf on lid and back wall.  
2. **Screw center marks** (engrave or score cross/circle) — not cut holes by default (operator drills).  
3. Optional **barrel axis line** (score, reference) if SVG reference layer is safe.  
4. Optional **latch / magnet center** on front wall + lid front (score/engrave).

### 9.2 Placement algorithm (conceptual)

```
backWallInnerWidth = ...
usableHingeSpan = backWallInnerWidth - 2 * cornerInset
if hingeCount * overallLength + (hingeCount-1) * minGap > usableHingeSpan → BLOCK
place hinges evenly or user-fixed first/last with equal gaps
for each hinge:
  reject if leaf intersects finger-joint zone on back wall top edge region
  reject/warn if screw centers within edgeMargin of panel edge or finger flanks
  reject if barrel closed gap implies lid/back intersection (geometric)
```

### 9.3 Collision classes

| Collision | Severity |
|-----------|----------|
| Hinge leaf ∩ finger joint band | **Block** |
| Screw mark edge distance &lt; margin | **Warn** (block if &lt; absolute min) |
| Hinge count overflow | **Block** |
| Barrel vs rear wall closed interference | **Block** |
| Guide on cut layer | **Block** (export invariant) |

---

## 10. Locating-frame model

| Decision | Recommendation |
|----------|----------------|
| Single closed frame vs four rails | **Four rails** (or frame with corner breaks) for glue access and less trapped scrap |
| Clearance | Per-side `frameClearance`; outer frame = cavity − 2×clearance |
| Height / section | Rail width user/default; thickness = material thickness |
| Hinged mode | Default **off**; validate vs hinge leaves if on |
| Lift-off | Primary use case; improves lid registration |
| Labels | “FRAME-FRONT” etc. if assembly labels on |

---

## 11. Joint architecture (G1 + future contract)

### 11.1 G1 decision

| Option | Recommendation |
|--------|----------------|
| Reuse finger-joint builder **directly** | **Yes** |
| Wrap edge-profile helpers | **Thin wrap OK** if needed for lid/top edge without fingers |
| Shared joint-strategy interface | **No for G1** |
| Local joint selection until second joint | **Yes** — `jointType: 'exposed-finger'` constant in G1 |

### 11.2 Future-compatible joint contract (document only)

```text
JointStrategy {
  id: 'exposed-finger' | 'hidden-tab-slot' | 'corner-block' | 'miter-core-skins' | 'decorative-keys' | 'faux-dovetail-overlay'
  role: 'structural' | 'decorative' | 'hybrid'
  generateStructuralPanels(ctx) -> Panel[]
  generateDecorativePanels(ctx) -> Panel[]  // may be empty
  generateGuides(ctx) -> GuidePath[]        // score/engrave only
  validate(ctx) -> { errors, warnings }
  requiresFitCoupon: boolean
}
```

Implement **only** `exposed-finger` in G1 as **concrete code**, not a full strategy framework. When G3 adds a second structural joint, introduce the interface by extracting the finger path.

---

## 12. Future joint comparison table

| Option | Role | Extra panels | Strength | Thin-ply risks | Grain/veneer | Glue / assembly | Fit coupon? | Phase |
|--------|------|--------------|----------|----------------|--------------|-----------------|-------------|-------|
| **1. Exposed finger joints** | Structural | None beyond box+lid(+frame) | High for ply boxes | Fingers too small / few | End-grain fingers visible | Excellent access | **Yes** (existing joint fit) | **G1** |
| **2. Hidden tab-and-slot** | Structural | Tabs on walls; slots in mates | Good if tab length adequate | Tear-out at slots | Cleaner exterior | Moderate; dry-fit critical | **Yes** (tab coupon) | **G3** |
| **3. Internal corner blocks** | Structural aid / hybrid | 4+ blocks | Helps squaring; not sole load path if walls only butted | Small blocks fragile | Visible inside | Blocks can trap glue | Optional | **G3** (or limited G2 experiment) |
| **4. Miter-look core + exterior skins** | Hybrid: core structural, skins decorative | Full set of face skins | Core must carry load | Skins peel if under-glued | Veneer direction matters | Skins after core cure | Core fit + skin gap | **G2–G3** |
| **5. Decorative corner keys / splines** | Decorative (+ mild alignment) | Keys at corners | Not primary structure | Micro keys break | Accent grain | Keys after assembly | Optional key coupon | **G2** |
| **6. Faux-dovetail overlays** | **Decorative only** | Overlay pieces | **Not structural dovetails** | Tiny tails fail | Cosmetic | Overlays after structure | No structural coupon | **G2** |

**Do not** classify (6) as true woodworking dovetails in UI copy.

---

## 13. Decorative layering architecture (G2+)

### Goals

- Baltic birch (or similar) **structural** box.
- Contrasting ply **face panels**, engraved borders/monograms, corner overlays, lid inset/applied panel.

### Non-negotiable constraints

| Risk | Mitigation |
|------|------------|
| Unexpected structural dimension change | Decorative thickness is **additive outside** or **pocketed into** lid with explicit mode; default **applied outside** with documented outer envelope growth **or** “within outer” requiring thinner structure — pick one and validate |
| Covering hinge guides | Decorative layers **exclude** hinge guide zones or guides regenerate on outer decorative face with offset rules |
| Blocking lid movement | Clearance envelope for open lid + overhang |
| Trapped glue joints | Prefer applied panels with breathable glue-up sequence notes in UI help |
| Microscopic pieces | Min feature size validation |
| Confusing decorative vs cut | Separate SVG groups / colors / filenames; never put decorative engrave on cut layer |

### Layering model (future)

```text
StructuralShell (G1)
  + OptionalLocatingFrame (G1)
  + HardwareGuides (G1)
  + DecorativeFacePanels (G2)     // non-structural
  + CornerOverlays / Keys (G2)  // decorative
  + LidAppliedOrInsetPanel (G2)
```

---

## 14. Validation and warnings

| Case | Severity |
|------|----------|
| Box too small for selected finger pattern | **Block** |
| Lid too small for hinge count / length | **Block** |
| Hinge guides ∩ finger joints | **Block** |
| Screw marks too close to edge | **Warn**; **Block** if &lt; hard minimum |
| Barrel colliding with rear wall (closed) | **Block** |
| Locating frame ∩ hinge leaves | **Block** (or auto-disable frame with warn — prefer block if both forced on) |
| Overhang incompatible with hinge placement | **Block** |
| Negative interior dimensions | **Block** |
| Material too thick for requested dims | **Block** |
| Frame rails too narrow | **Block** |
| Excessively small fingers | **Block** |
| Insufficient lid clearance | **Warn** or **Block** if ≤ 0 |
| Panels exceed active sheet size | **Warn** + multi-sheet **or** **Block** if multi-sheet disabled for template |
| Duplicate / overlapping cut geometry | **Block** (export invariant) |
| Guides on cut layer | **Block** |
| Hinge mode + zero hinges | **Block** |
| Lift-off + hinge fields shown | UI hide; if present ignore |

---

## 15. Finished Views

| View | G1 content |
|------|------------|
| **Closed three-quarter** | Outer shell, lid flush/overhang distinction, visible **finger joints**, schematic hinge blocks if hinged |
| **Open-lid** | Lid rotated ~90° schematic; interior; locating frame if present |
| Finger joints | Visible on structure |
| Locating frame | Visible when enabled |
| Hinge placement | **Schematic** rectangles/barrel — not photoreal brass |
| Decorative layers | G2+ optional toggle |
| Hardware | Schematic only |
| Role | **Preview only** — never production geometry |

---

## 16. SVG / LightBurn contract

### Layers (G1)

| Layer / intent | Color / rule (align existing) | Content |
|----------------|-------------------------------|---------|
| Structural cuts | Cut red `#ff0000`, rectilinear cut rules | Outer profiles, finger notches, frame parts |
| Hardware score guides | Score blue `#0000ff` | Leaf outlines, optional axis |
| Assembly labels | Existing label convention | Part names |
| Optional engraving | Existing engrave path if supported | Monogram later; screw centers may be engrave |
| Reference-only | Only if existing contract supports safely | Non-cut dashed refs; **do not invent** unsupported transforms |

### Production requirements

- Exact scale (1 user unit = 1 mm or documented).
- Deterministic bytes where fixtures pin them.
- No hidden duplicate cuts.
- No guides on cut layers.
- No unsupported transformations.
- Safe filenames (`gift-box-…` pattern).
- Material grouping: **single thickness G1** → one material group.
- **No change** to existing generator SVG bytes (finger-box, sliding-lid, etc.).

### LightBurn workflow compatibility

Preserve: import SVG → assign operations by color/layer → frame → cut. Same as current Designs.

---

## 17. Design-to-Project recommendation

| Choice | Recommendation |
|--------|----------------|
| Include in G1? | **Optional nice-to-have**, not required for architecture readiness |
| If included | Reuse existing handoff; project title from design name; attach export note |
| Risk | Low if no storage schema change |
| If deferred | User exports SVG and creates project manually |

---

## 18. Fixture plan

### Focused group: `gift-box` (recommended ~55–75 assertions)

| Theme | Assertions (approx.) | DOM? |
|-------|----------------------|------|
| Internal/external conversion | 4–6 | No pure if helpers pure; often yes via form |
| Lift-off lid | 4–6 | Yes |
| Hinged flush | 4–6 | Yes |
| Hinged overhanging | 3–5 | Yes |
| Locating frame clearances | 4–6 | Mix |
| Hinge count 1/2/3 | 3–6 | Mix |
| Hinge–edge / finger collision | 4–6 | Mix |
| Screw–edge collision | 2–4 | Mix |
| Barrel clearance | 2–3 | Mix |
| Finger-joint parity with helpers | 3–5 | Mix |
| Small impossible boxes | 3–4 | Mix |
| Separate cut vs guide layers | 4–6 | Yes (SVG parse) |
| Closed Finished View | 2–3 | Yes |
| Open Finished View | 2–3 | Yes |
| Deterministic SVG | 2–3 | Yes |
| No duplicate cuts | 2–3 | Yes |
| Existing Design bytes unchanged | 2–4 smoke | Yes |
| Direct `file://` suite inclusion | meta | Yes |
| Unit parity | 2–3 | Mix |
| Design→Project (if included) | 2–3 | Yes |

**Realistic total:** ~**65** assertions in first fixture landing.  
**Requires browser/DOM:** Finished View, SVG contract, form generate path, LightBurn color checks, handoff — majority of integration tests.

---

## 19. Physical-test plan

### Workshop process (G1)

1. Enter finished dimensions (internal or external).  
2. Measure **actual** material thickness.  
3. Enter tested **kerf** and **finger fit**.  
4. Measure physical hinges (L, leaf W, barrel Ø, screw pattern).  
5. Generate Gift Box.  
6. Inspect Closed + Open Finished Views.  
7. Export SVG.  
8. Assign LightBurn ops by layer (cut vs score/engrave).  
9. Frame the job.  
10. Cut **joint fit coupon** (existing).  
11. **Optional:** cut hinge-placement coupon (see below).  
12. Cut panels.  
13. Dry-fit.  
14. Mark/drill hardware using engraved centers.  
15. Assemble; maintain fire watch during laser ops.

### Hinge-placement coupon

| Question | Recommendation |
|----------|----------------|
| Dedicated coupon in G1? | **Yes, lightweight** — single panel with leaf outline + screw centers at actual spacing for the user’s hinge fields; not a full second product |
| Why | Validates drill positions and leaf footprint before committing full box stock |
| If deferred | Acceptable if physical hinge marking on scrap is documented; coupon is lower risk |

---

## 20. G1 / G2 / G3 implementation phases

### G1 — Production-ready structural Gift Box

| Aspect | Detail |
|--------|--------|
| Scope | Lift-off + surface hinged; flush/overhang; exposed fingers; hardware guides; locating frame; Finished Views; SVG; fixtures |
| Files likely | `index.html` (registry, form, generators, FV, export); `tests/designs-geometry-fixtures.html`; `README.md` / `CHANGELOG.md` |
| Risk boundaries | Must not alter existing template SVG bytes; kerf never in geometry; guides ≠ cuts |
| Independent audit | **Required** before commit (geometry + SVG + regression) |
| Physical tests | Fit coupon; at least one lift-off and one hinged prototype |
| Production bytes | **New** gift-box only; existing templates unchanged |
| Storage/schema | **None** |

### G2 — Decorative layering

| Aspect | Detail |
|--------|--------|
| Scope | Skins, lid panels, keys/splines, faux-dovetail **overlays** (decorative copy only) |
| Files | Generators + FV + fixtures + help copy |
| Risk | Outer envelope, hinge occlusion, min feature size |
| Audit | Yes |
| Physical | Glue-up of skins; overlay fragility |
| Bytes | Gift-box SVG grows; other templates still frozen |
| Storage | Still session drafts only |

### G3 — Alternate structural joints

| Aspect | Detail |
|--------|--------|
| Scope | Hidden tab-and-slot; internal corner-block construction options |
| Files | Extract joint interface; new builders; coupons |
| Risk | High — structural correctness |
| Audit | Yes + physical destructive tests |
| Bytes | New joint modes; pin carefully |
| Storage | None required |

---

## 21. Protected existing-output boundaries

| Boundary | Rule |
|----------|------|
| Finger-box SVG | Unchanged |
| Open-top / tray / sliding-lid / cabinet / coupons | Unchanged |
| LightBurn color semantics | Unchanged |
| Kerf-in-geometry ban | Unchanged |
| Session-only Designs storage | Unchanged |
| Multi-machine keys | Unrelated; do not touch |
| M3 WIP | Do not revise for Gift Box |

---

## 22. Storage / schema implications

| Topic | G1 |
|-------|-----|
| Workshop JSON / machines[] | **No change** |
| Designs persistence | Session draft only (existing) |
| New localStorage keys | **Avoid** |
| Schema migrations | **None** |
| Backup inclusion of Designs | Still out of scope unless product later decides otherwise |

---

## 23. Recommended Codex implementation scope (G1)

1. Register template `gift-box` with defaults + form schema.  
2. Dimension mode + single thickness + kerf + fit + lid clearance + spacing + units.  
3. Generate structural panels: base, 4 walls, lid; optional frame rails.  
4. Finger joints via existing builders.  
5. Lid modes: lift-off / hinged × flush / overhang.  
6. Hinge guide generation + validation.  
7. Latch/magnet optional centers.  
8. Assembly labels.  
9. Closed + open Finished Views (schematic hardware).  
10. SVG export with cut/score separation.  
11. Fixture group ~65 assertions.  
12. README/CHANGELOG user-facing G1 notes.  
13. **Do not** implement G2/G3 joints, dual thickness, mortises, hardware libraries, storage changes.

---

## 24. Recommended audit scope (post-implementation)

| Check | Required |
|-------|----------|
| Existing Designs SVG byte / contract regression | Yes |
| Gift-box cut vs guide layer separation | Yes |
| Finger parity with finger-box helpers | Yes |
| Hinge collision validation cases | Yes |
| Finished View closed/open | Yes |
| Unit conversion | Yes |
| file:// full suite green | Yes |
| M3 / multi-machine isolation | Confirm untouched |
| Physical prototype notes | Attach or link |
| No true-dovetail claims in UI | Copy review |

---

## 25. Open product decisions (Joe)

| # | Decision | Why it matters |
|---|----------|----------------|
| 1 | Default box size for jewelry vs card box presets? | First-run UX |
| 2 | Default hinge count (1 vs 2)? | Small boxes |
| 3 | Overhang default mm and whether rear overhang allowed when hinged | Hinge alignment |
| 4 | Locating frame default on for lift-off only? | Recommended yes — confirm |
| 5 | Screw centers: score vs engrave layer | LightBurn ops |
| 6 | Include hinge coupon panel in G1 export? | Stock + time |
| 7 | Dual lid thickness forced into G1? | Architecture complexity |
| 8 | Design→Project in G1? | Scope |
| 9 | Min screw edge margin and finger-band exclusion width | Safety numbers from shop practice |
| 10 | Preferred starting lid clearance after Joe’s kerf tests | Default calibration |
| 11 | Label language: “Gift Box” vs “Keepsake Box” | Product naming (non-goal to overthink; pick one) |
| 12 | Frame as four rails vs one ring | Assembly preference |

---

## 26. Confirmation: no product mutation by this review

| Action | Performed? |
|--------|------------|
| Edit source (`index.html`, tests, etc.) | **No** |
| Stage / commit / push | **No** |
| Reset / clean / stash / checkout | **No** |
| Evaluate or revise M3 | **No** (recorded only) |
| Write this report | **Yes** — `docs/DESIGNS_GIFT_BOX_ARCHITECTURE_REVIEW_2026-07-19.md` |

**Committed baseline for design work remains `7a9ec6d`.** Uncommitted M3 is unrelated working-tree noise for Gift Box.

---

## Appendix A — Suggested G1 field schema (sketch)

```text
giftBox:
  units: mm|in
  dimensionMode: internal|external
  length, width, height
  materialThickness
  kerf
  jointFit / fingerClearance
  panelSpacing
  lidType: lift-off | hinged-surface
  lidAppearance: flush | overhanging
  overhang (if overhanging)
  lidClearance
  locatingFrame: bool
  frameClearance, frameRailWidth
  hingeCount, hingeLength, leafWidth, barrelDiameter
  closedHingeGap, cornerInset
  screwCount, screwSpacing, centerMarkDiameter
  frontClaspGuide: bool
  magnetGuides: bool
  assemblyLabels: bool
```

## Appendix B — Risk register (G1)

| Risk | Mitigation |
|------|------------|
| Users treat score leaves as cut-out mortises | UI copy + layer names |
| Thin ply screw blowout | Edge margin validation + coupon |
| Over-constrained hinged + frame | Defaults + block overlap |
| Scope creep into G2 skins | Phase gate |
| Fixture flaky FV | Prefer structural SVG asserts over pixel FV |

---

*End of architecture review.*
