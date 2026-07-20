# Designs Dice Tray System — Architecture & Product-Design Review

**Date:** 2026-07-20  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Review type:** Read-only architecture and product design (no implementation)  
**Baseline (committed):** `294cc06` — *Add parametric Gift Box generator*  
**Project priority:** Workshop-useful cuttable Designs first; local gifts / game-night products; software monetization secondary  

---

## 0. Executive recommendation

| Question | Answer |
|----------|--------|
| Is **DT1** architecturally ready? | **Yes** — as a **bounded extension of the existing `dice-tray` / tray-model pipeline**, with frozen goldens for the simple open tray. |
| Sliding lid in DT1 or DT2? | **DT2** — separate commit after open + storage + insert contract is green. |
| Best template strategy | **Extend `dice-tray` with additive modes** (storage / insert / later lid). Do **not** rewrite or replace the simple tray. Do **not** invent a parallel geometry stack. |
| First joint strategy | Keep **proven wall-to-base tab system** (finger or two-tab profiles) as structural default. **Exposed finger-joint outer walls** are a later option (DT3), not a DT1 requirement that forces golden breakage. |
| Smallest useful first boundary | **DT1:** open rolling tray ± left/right storage bay + fixed structural divider + insert measurement (optional cut panel) + Finished View + SVG + fixtures. |

---

## 1. Repository state

| Check | Result |
|-------|--------|
| `git status -sb` | `## main...origin/main`; **no tracked modifications** |
| `git log -1 --oneline` | `294cc06 Add parametric Gift Box generator` |
| `git rev-parse HEAD` | `294cc064f82020ce02ca95e2ced997f003715ba9` |
| `origin/main...main` | `0 0` |
| `git diff --check` | Clean |
| `git diff` / staged | Empty |
| Untracked | Historical docs, LightBurn projects, `debug.log`, utility script — **untouched** |

### Application constants (current)

| Constant | Value |
|----------|--------|
| `APP_ID` | `genmitsu-l8-tracker` |
| `APP_NAME` | `Genmitsu L8 Tracker` |
| `APP_VERSION` | `0.9.0` |
| `BUILD_DATE` | `2026-07-19` |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | `2` |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` |

### Fixture baseline (live `file://` Edge/Playwright, 2026-07-20)

| Suite | Expected | Live |
|-------|----------|------|
| Complete | 2362 / 0 | **2362 / 0** |
| Complete-suite groups | 28 | **28** |
| Designs geometry | 1093 / 0 | **1093 / 0** |
| Gift Box | 69 / 0 | **69 / 0** |
| M1 / M2 / M3 | 29 / 31 / 27 | **Match** |
| Tray standalone | 264 / 0 | **264 / 0** |
| Promotion-switch standalone | 16 / 0 | **16 / 0** |
| Page exceptions | none | **none** |

---

## 2. Existing Dice Tray architecture

### 2.1 Templates

| Template | Role today |
|----------|------------|
| **`dice-tray`** | Open-top rolling tray: inside W×D, wall height, material thickness, slot clearance, wall-to-base joint style (`finger` = four-tab, `tab-slot` = two-tab), optional cosmetic underside cover plate |
| **`divider-tray`** | Same shell + 1–6 removable left-to-right dividers with base slots |

Both share **`buildTrayModel` / `generatorVersion: 'tray-model-phase1'`**, `buildTrayDesignResult`, tray Finished Views, and the large **`runTrayModelFixtures`** golden suite (264 assertions standalone; also embedded in Designs geometry).

### 2.2 Structural model (important)

Current trays are **not** finger-jointed outer wall boxes like Finger Box / Gift Box.

They are:

1. **Base plate** with through-cut wall slots  
2. **Four walls** with tabs into the base (`designWallPath` + `trayTabProfile`)  
3. Optional **divider slots** in base + plain rectangular divider panels (divider-tray)  
4. Optional **bottom cover** (dice-tray only) — cosmetic, not structural  

Dimensions are **inside cavity** driven:  
`baseOutside = inside + 2×thickness`.

### 2.3 What is missing for the product vision

| Desired capability | Present? |
|--------------------|----------|
| Side storage bay (left/right) | No |
| Rolling vs storage clear dimensions | No (single cavity only) |
| Rolling-surface insert geometry | No |
| Sliding lid over tray | No (separate `sliding-lid-box` exists) |
| Full-tray lid + storage coexistence | No |
| Engraving zones | No (user LightBurn only) |
| Multi-material export grouping | No |

### 2.4 Related generators

| Generator | Relevance |
|-----------|-----------|
| **Open-top / Finger Box** | Finger walls + base; wrong joint model for current tray goldens |
| **Gift Box** | Finger shell, lids, labels, dual Finished Views, hardware guides — good **product UX** patterns, not tray joints |
| **Sliding-lid box** | Laminated rails, lid clearances, pull notch, fit coupon, Closed/Open FV — **reuse helpers carefully**, never mutate its SVG bytes |
| **Divider tray** | Removable L–R dividers + base slots — closest storage-partition pattern |
| **Wall/base tab coupon** | Fit testing for tray-style joints |
| **Joint Fit Coupon** | Finger fit (boxes), not tray slots |

---

## 3. Reusable helpers (safe)

| Helper / system | Safe reuse |
|-----------------|------------|
| `buildTrayModel` / tray validation / tab profiles | Extend with storage + divider fields |
| `trayRectComponent`, `trayWallComponent`, slot placement | Storage divider walls/slots |
| `serializeTrayCompatibilitySvg` / `designSvgDocument` | Production SVG |
| `layoutDesignPanelRows` / nesting | Extra panels (divider, insert, lid later) |
| Unit conversion + mm draft fields | All new fields |
| Assembly labels / hidden-label machinery | Optional labels for BASE, walls, DIVIDER, LID |
| Gift Box / sliding Finished View discipline | Screen-only preview, never in production SVG |
| Sliding-lid **clearance fields**, rail panel builders, pull validation | DT2 only, via thin wrappers |
| `buildSlidingCouponModel` patterns | DT2 lid/rail coupon inspiration |
| Design-to-Project handoff | Generic notes + dimensions |
| Wall/base tab coupon | Physical fit for tray joints |
| Fixture harness + goldens isolation | Pin existing tray hashes first |

---

## 4. Unsafe reuse points

| Temptation | Why unsafe |
|------------|------------|
| Mutate default `dice-tray` SVG / goldens | Breaks 264 tray fixtures + Designs golden paths |
| Replace wall-to-base with finger walls in-place | Different edge ownership, slot model, assembly story |
| Call `buildSlidingLidBoxModel` as the tray body | Full box with shortened front, laminated rails sized for box cavity — wrong product topology |
| Pocket “insert recess” as fake 3D depth in one ply | Through-cut only unless layered base is explicit |
| Nest cork/leather on same sheet as wood with same ops | Wrong LightBurn settings; material safety |
| Auto-claim leather/felt/cork laser-safe | Composition unknown |
| Bolt storage bay onto divider-tray by overloading dividerCount | Different product semantics; confuses organizer vs dice storage |
| Change `sliding-lid-box` for tray needs | Protected output boundary |
| Premature joint-strategy framework | YAGNI until second structural system ships |
| Storage/schema / localStorage for Designs | Non-goal; session-only remains |

---

## 5. Recommended template strategy

| Option | Verdict |
|--------|---------|
| Rewrite tray model from scratch | **No** |
| Separate “Advanced Dice Tray” template only | Acceptable if form complexity explodes, but **not required** for DT1 |
| **Extend `dice-tray` with backward-compatible modes** | **Recommended** |

### Compatibility rules

1. Defaults with **no storage, no insert panel, no lid** produce the **same production SVG bytes** as today’s simple Dice Tray (within explicit golden pins).  
2. New fields are additive; absent/false → legacy path.  
3. `divider-tray` remains the multi-compartment organizer; do not merge products.  
4. Optional later: rename UI label to “Dice tray (open)” vs “Dice tray + storage” without changing template id.

---

## 6. Recommended DT1 boundary (first shippable)

**Name:** Dice Tray (modes on `dice-tray`)

### In scope

| Feature | DT1 |
|---------|-----|
| Rectangular open tray | Yes |
| Wall-to-base structural joints (existing) | Yes |
| One material thickness for wood structure | Yes |
| Inside dimension entry (existing) | Yes |
| Optional **storage bay left or right** | Yes |
| User-entered storage width | Yes |
| Storage full tray depth | Yes |
| **Fixed structural divider** (base slot + wall engagement) | Yes |
| Optional **insert size report** + optional **wood insert cut panel** | Yes |
| Insert material type as **label only** (cork/leather/felt/generic) | Yes — geometry + safety copy, not material claims |
| Finished View (open three-quarter) with storage + insert surface | Yes |
| Cut layout, labels, Design-to-Project | Yes |
| Fixtures protecting legacy goldens | Yes |

### Out of scope for DT1

Sliding lid, split lids, removable multi-dividers beyond one storage divider, decorative skins, finger outer walls, engraving generators, foam, magnets, towers, deck boxes, M4, inventory CSV, schema changes.

---

## 7. Recommended DT2 boundary

| Feature | DT2 |
|---------|-----|
| Shared **sliding lid** over full outer footprint | Yes |
| Laminated inner rails adapted to tray wall height | Yes |
| User lid clearances (side / vertical / insertion) | Yes |
| Pull notch (none / rectangular) | Yes |
| Lid-fit coupon | Yes |
| Finished Closed + Finished Open (lid translates along channel) | Yes |
| Storage + rolling remain visible under open lid | Yes |
| Existing `sliding-lid-box` bytes frozen | Mandatory |

**Do not implement DT1 and DT2 in one commit** unless both are complete with separate fixture groups and independent audits. Default: **two commits**.

---

## 8. Dimension model

### 8.1 User-facing entry (DT1 recommendation)

**Primary mode: inside rolling + storage widths** (consistent with today’s inside tray width/depth).

Optional later: external envelope mode for “fits this shelf.”

### 8.2 Definitions

| Quantity | Definition |
|----------|------------|
| Inside total width | Clear width wall-to-wall (existing `trayWidth`) |
| Inside depth | Clear depth front-to-back (existing `trayDepth`) |
| Wall height | Clear height above base top (existing `trayHeight`) |
| Storage width | Clear storage bay width along total inside width |
| Rolling width | `insideTotalWidth − storageWidth − dividerThickness` (when storage on) |
| Rolling depth | Full inside depth (DT1) |
| Storage depth | Full inside depth (DT1) |
| Base outside W/D | inside + 2×thickness |
| Divider thickness | = material thickness (DT1 single stock) |
| Insert thickness | User field (affects usable wall height reporting only if insert sits on base) |
| Insert clearance | Per-side gap: insert footprint = cavity − 2×clearance |
| Lid clearance fields | DT2 only (side/vertical/insertion) |
| Closed overall height | DT1: wall height + base thickness (+ cover if on); DT2: + lid stack / rail stack as modeled |
| Thumb / pull clearance | DT2 pull notch + front insertion |

### 8.3 Result summary must report

- Outer base footprint  
- Clear rolling W×D  
- Clear storage W×D (or “none”)  
- Insert cut size (if any)  
- Lid footprint (DT2)  
- Closed height  
- Physical piece count  
- Sheet count  

---

## 9. Storage-bay model

### DT1 choices

| Option | DT1 |
|--------|-----|
| No storage | Default |
| Left storage | Yes |
| Right storage | Yes |
| User storage width | Yes (mm) |
| Min storage width | **Block** below ~18–22 mm clear (retrieval) |
| Full depth bay | Yes |
| Shorter compartment | Defer DT3 |
| One open storage compartment | Yes |
| Two equal storage compartments | Defer DT3 |
| Fixed structural divider | **Yes (default when storage on)** |
| Removable divider | Optional later; prefer fixed for transport safety |

### Product rules

1. Divider is a **full-height wall** between rolling and storage so dice cannot migrate during play (open tray).  
2. With DT2 lid, same divider keeps dice out of the rolling cavity in transport.  
3. Storage on **left or right** only (not front/back) — simpler lid rails and dice retrieval.  
4. Glue access: assemble shell, dry-fit divider into base slot + optional wall slots, then glue.  
5. Narrow storage: warn below ~28 mm; block below hard min.

---

## 10. Divider comparison

| Construction | Reliability | Assembly | SVG complexity | Fit coupon? | Thin-ply risk | Phase |
|--------------|-------------|----------|----------------|-------------|---------------|-------|
| **1. Fixed glued divider** (no slots) | Low registration | Easy but drift | Low | Optional | Warping | Avoid alone |
| **2. Divider tabbed into base only** | Good | Easy | Medium | Tray slot clearance | Tear-out at slots | **DT1 preferred** |
| **3. Base + front/back wall tabs** | Best square | Harder | High | Yes | More small features | DT1.1 if needed |
| **4. Removable paired slots** | Good if tight | Easy swap | Medium | Yes | Loose = dice leak | DT2/DT3 |
| **5. Drop-in feet/guides** | Weak for transport | Easy | Medium | No | Fragile feet | DT3 only |

**DT1 recommendation:** Option **2** — through-cut base slot (same family as divider-tray / wall slots), full-height divider panel, glue after dry-fit.  
**No pocketed non-through “slots.”**  
If transport leak appears in physical tests, escalate to option **3** without redesigning the form model.

---

## 11. Sliding-lid architecture (DT2)

### Reuse from `sliding-lid-box`

| Reuse | How |
|-------|-----|
| `slidingLidBoxDimensions` concepts | Clearance stack, rail webs, engagement |
| `buildSlidingLidRail` / `buildSlidingLidPanel` | Geometry primitives |
| Pull validation | Same safety webs |
| Fit coupon pattern | Tray-specific coupon |
| Finished Closed/Open translation semantics | Gift/sliding discipline |

### Do not reuse blindly

| Piece | Reason |
|-------|--------|
| Full `buildSlidingLidBoxModel` body | Shortened front + box finger walls ≠ tray wall-to-base shell |
| Open-end assumptions without revalidation | Tray front is full wall height today |
| Changing `sliding-lid-box` template | Protected bytes |

### DT2 product rules

1. **One lid covers entire outer footprint** (rolling + storage).  
2. Rails laminated **inside** left/right walls (same proven approach).  
3. Insertion from **front** (match existing open-end convention).  
4. Back stop as today.  
5. User-entered side / vertical / insertion clearances — **starting values, not guarantees**.  
6. Divider top must clear lid underside by vertical clearance; wall height must support rail stack — **block** if not.  
7. Insert must not collide with rails (insert height ≤ rolling floor stack; rails sit on sides).  
8. Removable lid; no hardware.  
9. Optional rectangular pull notch.  

### Split-lid / separate storage lid (variant E)

Evaluate-only: **more parts, more fit failures, weaker transport story**. Defer to DT3; shared lid is simpler.

---

## 12. Insert system (rolling surface)

### Material handling (safety)

The generator **models size only** and shows mandatory copy:

- Verify cork / leather / felt / adhesives for laser composition.  
- Do not laser unknown leather, vinyl-backed felt, mystery coatings, or unknown adhesives.  
- Prefer hand-cut or verified materials for liners.  
- Separate process from wood; ventilation + fire watch always.  

### DT1 insert contract (recommended)

| Capability | DT1 |
|------------|-----|
| No insert | Default |
| Material type enum (cork / leather / felt / generic / wood panel) | Label + help only |
| Insert clearance field (start **0.3–0.5 mm/side**) | Yes |
| Report cut size in summary | Yes |
| Optional **wood insert panel** on cut layout (same thickness as structure **or** explicit insert thickness field) | Yes if wood; optional |
| Cork/leather/felt as **measurement-only** (no auto-nest with wood) | **Yes — preferred for non-wood** |
| Score outline on base | Optional guide (blue score), not structural recess |
| Layered true recess base | **DT3** (extra base layer complexity) |
| Separate download for insert | Prefer when non-wood or different thickness |

### Geometry

```
insertWidth  = rollingClearWidth  - 2 × insertClearance
insertDepth  = rollingClearDepth  - 2 × insertClearance
usableWallAboveInsert = wallHeight - insertThickness   (report; warn if shallow)
```

Corner radius: optional later; DT1 square is fine.

---

## 13. Engraving strategy

| Option | Phase |
|--------|-------|
| User engraves in LightBurn on base/lid/insert | **DT1** (document clear zones) |
| Score rectangle “engrave safe area” on insert/base | DT1 optional blue score |
| Parametric name/campaign text paths | DT3 |
| Dice borders / original ornaments | DT3 |
| Copyrighted logos / card art | **Never** |

**DT1:** no embedded artwork generator. Summary + help: “engrave before assembly on base top or insert face.”

---

## 14. Joint strategy

### DT1 structural

- **Wall-to-base tabs** (existing finger / tab-slot profiles).  
- Storage divider: base slot (+ optional wall engagement later).  
- Single thickness wood.

### Future (DT3+)

| Option | Role | Notes |
|--------|------|-------|
| Exposed finger outer walls | Structural alternate | New construction mode; new goldens; don’t replace default |
| Hidden tab-and-slot | Structural | High risk |
| Corner blocks | Aid | Optional |
| Decorative skins / keys / faux-dovetail overlays | Decorative only | Never claim true dovetails |

Do **not** implement alternate joint strategies in DT1.

---

## 15. Validation and warnings

### Blocking errors

- Nonpositive / nonfinite dimensions  
- Rolling or storage clear area ≤ 0  
- Storage width below hard minimum  
- Divider outside tray / colliding with wall slots improperly  
- Insert dimensions ≤ 0 after clearance  
- Insert larger than rolling cavity  
- (DT2) Lid/rail engagement failures, channel too short, wall too short for rails, lid travel blocked by divider height, rail–joint collisions  
- Duplicate cut segments, open contours, panel overlap in layout  

### Warnings (non-blocking)

- Insert clearance below practical start  
- Thin liner may curl / need adhesive (composition unknown)  
- Shallow walls (dice escape)  
- Narrow storage retrieval  
- Multi-sheet / large work-area advisory  
- Untested lid/divider fit  
- Starting clearances are not guarantees  
- Cosmetic bottom cover caveats (existing)  

---

## 16. Finished Views

### Open tray (DT1)

- Three-quarter open-top view (extend `buildDiceTrayFinishedViewSvg` / projection data)  
- Rolling cavity + storage bay colors distinct  
- Divider visible  
- Insert as distinct fill when enabled  
- Screen-only; honest “does not prove fit” copy  

### Sliding-lid (DT2)

- **Finished Closed** — lid seated  
- **Finished Open** — lid **translates** along front→back channel (not hinged rotation; not uniform scale)  
- Cavities readable under/behind lid  
- No FV IDs in production SVG  

### Optional later

Top-down assembly or cutaway — polish only if three-quarter is ambiguous.

---

## 17. SVG / LightBurn contract

| Layer | Color / rule |
|-------|----------------|
| Structural cuts | Red `#ff0000` |
| Score guides (insert outline, rail guides DT2) | Blue `#0000ff` |
| Assembly labels | Existing green/label convention |
| Engrave | User ops; optional score guides only in DT1 |

Requirements:

- Exact scale, deterministic bytes where fixtures pin  
- No duplicate cuts; no FV geometry in production  
- No unsupported transforms  
- Existing templates **byte-stable**  
- `file://` LightBurn workflow unchanged  

### Material separation

| Material | DT1 handling |
|----------|----------------|
| Structural wood | Main SVG |
| Wood insert panel | Same SVG only if same thickness + same ops; else separate export or measurement-only |
| Cork / leather / felt | **Measurement-only** + safety copy; optional second export later |

---

## 18. Panel layout and material separation

| Panel | DT1 | DT2 |
|-------|-----|-----|
| Base + 4 walls | Yes | Yes |
| Storage divider | If storage | Yes |
| Bottom cover | Existing optional | Existing |
| Wood insert | Optional | Optional |
| Lid + 2 rails | No | Yes |
| Lid coupon | No | Yes |

Do not silently nest non-wood liners with wood as one LightBurn job.

---

## 19. Design-to-Project behavior

Reuse generic handoff:

- Name from outer size + “Dice Tray”  
- Notes: dry-fit wall-to-base joints; storage/insert/lid caveats as enabled  
- No schema change; no inventory mutation  

---

## 20. Fixture plan

### Protect first

- All existing **Dice Tray / Divider Tray** goldens (standalone 264 + Designs embeds)  
- Gift Box, Finger Box, Sliding-lid, Cabinet, coupons  

### New focused group (recommend `runDiceTraySystemFixtures` or extend tray fixtures carefully)

| Theme | Assertions (approx.) | DOM? |
|-------|---------------------:|:----:|
| Legacy default SVG hash unchanged | 2–4 | Mix |
| Open no-storage parity | 3–5 | Mix |
| Storage left/right + widths | 6–8 | Mix |
| Divider geometry + slot | 4–6 | Mix |
| Insert dims / clearance / disabled | 5–7 | Mix |
| Rolling vs storage metrics | 4–6 | Mix |
| Blocking invalid storage/insert | 4–6 | Mix |
| Finished View storage + insert | 4–6 | Yes |
| Labels conditional | 2–3 | Mix |
| Design-to-Project | 2 | Mix |
| No FV in production SVG | 2 | Yes |
| Determinism / closed contours / colors | 4–6 | Yes |
| DT2 (later) lid travel / closed-open | 10–15 | Yes |

**Realistic DT1 assertion add:** ~**45–60** new (or net suite growth).  
**DOM required:** FV parse, SVG contract, live preview/download parity.

---

## 21. Physical-test plan

1. Measure plywood thickness.  
2. Cut wall/base tab coupon; set fit clearance.  
3. Generate open tray (no storage); dry-fit.  
4. Generate storage-left/right; dry-fit divider.  
5. Measure actual rolling cavity; set insert clearance; cut/trim insert **separately** after composition check.  
6. (DT2) Cut lid coupon; test travel before final glue.  
7. Glue only after dry-fit; fire watch + ventilation.  

**DT1 coupon:** existing wall/base coupon sufficient.  
**DT2:** dedicated lid/rail coupon strongly recommended.

---

## 22. DT1 / DT2 / DT3 phases

### DT1 — Open + storage + insert contract

| Item | Detail |
|------|--------|
| Files | `index.html` (tray model, form, FV, summary, fixtures), README/CHANGELOG |
| Risk | Golden breakage; divider/slot collisions |
| Audit | Focused audit after implement |
| Production bytes | Legacy dice-tray default **unchanged** |
| Storage/schema | None |

### DT2 — Sliding lid

| Item | Detail |
|------|--------|
| Files | Tray lid builders wrapping sliding helpers; FV closed/open; coupon; fixtures |
| Risk | Height stack; rail vs divider; must not touch sliding-lid-box |
| Audit | Required |
| Separate commit | **Yes** |

### DT3 — Decorative / advanced

Skins, finger outer walls mode, multi-compartment, split lids, parametric engrave, layered recess bases.

---

## 23. Protected output boundaries

Pin / never change unintentionally:

| Template | Notes |
|----------|--------|
| Dice Tray default | Golden length/hash in tray fixtures |
| Divider Tray defaults | Same suite |
| Finger Box | Designs geometry goldens |
| Open-top / loose lid box | Same |
| Sliding-lid box | **Hard freeze** |
| Gift Box | 69 fixtures + production isolation |
| Drawer Cabinet | Goldens |
| Joint Fit Coupon / Wall-base coupon | Goldens |

---

## 24. Storage / schema implications

| Topic | Decision |
|-------|----------|
| Workshop JSON / machines | No change |
| Designs persistence | Session-only (existing) |
| New localStorage keys | Avoid |
| Schema version | No change |
| Backups | Designs remain excluded |

---

## 25. Product decisions (with recommendations)

| # | Decision | Recommendation |
|---|----------|----------------|
| 1 | Keep template vs new Advanced | **Extend `dice-tray`** with modes; freeze simple defaults |
| 2 | External vs rolling dimensions | **Inside total W×D** primary; report outer derived |
| 3 | Default wall height | Keep **35 mm** (current default) as start |
| 4 | Default storage side | **Right** (arbitrary; left available) |
| 5 | Default storage width | **40–45 mm** clear when enabled |
| 6 | Fixed vs removable divider DT1 | **Fixed** (base slot + glue) |
| 7 | Insert measure vs panel | **Measure always**; optional wood panel; non-wood measure-only |
| 8 | Insert clearance start | **0.4 mm per side** (starting value) |
| 9 | Shared lid vs split | **Shared full lid** in DT2 |
| 10 | Lid insertion direction | **Front** (match sliding-lid) |
| 11 | Pull notch | Default **none**; optional rectangular |
| 12 | DT1+DT2 one implementation? | **No — two commits** |
| 13 | Engraving in first impl? | **Zones/docs only**; no art generator |

---

## 26. Recommended Codex implementation scope (DT1)

1. Additive draft fields: storage mode (none/left/right), storage width, divider enable, insert enable/type/clearance/thickness, optional insert panel flag.  
2. Extend `buildTrayModel` for storage bay + divider slot/panel; metrics for rolling/storage clears.  
3. Validation + warnings per §15.  
4. Result summary metrics.  
5. Finished View projection updates.  
6. Optional wood insert panel in layout.  
7. Labels conditional.  
8. Fixtures: legacy golden lock + new cases (~45–60).  
9. README/CHANGELOG workshop + safety copy for liners.  
10. **Do not** implement sliding lid, schema changes, or sliding-lid-box edits.

---

## 27. Recommended post-implementation audit scope

- Legacy dice/divider SVG hashes  
- New storage geometry correctness  
- Insert never nests as unsafe multi-material  
- FV free of NaN; not in production SVG  
- Suite totals arithmetic  
- M1–M3 / Gift Box isolation  
- Direct `file://` gift-box + design + all  

---

## 28. Whether Claude Sonnet is necessary after implementation

**Focused audit (any capable reviewer) is sufficient** for DT1 if fixtures are coordinate-honest and goldens hold.  
**Full re-architecture audit is not required** unless DT1 breaks tray goldens, mixes materials unsafely, or DT2 touches sliding-lid-box bytes.

---

## 29. Confirmation — this review did not mutate the product tree

| Action | Performed? |
|--------|------------|
| Edit / stage / commit / push | **No** |
| Reset / clean / stash / checkout / move / rename / delete | **No** |
| Runtime fixture verification only | **Yes** (read-only) |
| Write this report | **Yes** → `docs/DESIGNS_DICE_TRAY_SYSTEM_ARCHITECTURE_REVIEW_2026-07-20.md` |

---

## Appendix A — Variant map to phases

| Variant | Phase |
|---------|-------|
| A. Open rolling, no storage | DT1 default |
| B. Open + storage L/R | DT1 |
| C. Sliding lid, no storage | DT2 |
| D. Sliding lid + storage | DT2 |
| E. Split / separate storage lid | DT3 evaluate-only |

## Appendix B — Why finger outer walls wait

Locked preference for exposed finger walls is valid long-term, but **current Dice Tray value is the wall-to-base tray model with 264 locked fixtures**. Forcing finger walls in DT1 would either break goldens or fork a second body generator. Prefer shipping **sellable storage + insert** on the proven joint first; add finger-shell mode only when Joe needs that look and accepts a new golden family.

---

*End of Dice Tray system architecture review.*
