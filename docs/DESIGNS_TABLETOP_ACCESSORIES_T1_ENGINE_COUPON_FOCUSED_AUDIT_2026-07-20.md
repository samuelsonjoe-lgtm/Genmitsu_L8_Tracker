# Tabletop Accessories T1 Engine + Corner/Floor Coupon — Focused Audit

**Date:** 2026-07-20  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit mode:** Strictly read-only (no edit, stage, commit, push, reset, clean, stash, checkout, move, rename, or delete of product files; this report is the sole deliverable write).  
**Committed baseline:** `7876551` — Add Dice Tray storage and insert options  
**Authority order:** Construction-contract review → architecture review → implementation prompt (unless safe justified deferral) → protected production bytes.

**Normative references:**
- `docs/DESIGNS_TABLETOP_ACCESSORIES_CONSTRUCTION_CONTRACT_REVIEW_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_ARCHITECTURE_REVIEW_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_IMPLEMENTATION_2026-07-20.md`

---

## Exact final verdict

**CHANGES REQUIRED BEFORE PHYSICAL CUTTING**

| Severity | Count |
|----------|------:|
| BLOCKER | 1 |
| IMPORTANT | 6 |
| POLISH | 2 |

| Question | Answer |
|----------|--------|
| Coupon geometry coherent for dry-fit? | **Yes** (engine/path level; scrap test still required) |
| Safe to cut via in-app operator workflow for a controlled 2.88 mm case? | **No** — Designs form does not render coupon controls |
| May T1 be committed as-is? | **No** |
| Correction pass required? | **Yes** |
| Claude full audit needed? | **No** (no new shared-helper multi-product defect found) |
| Files edited/staged/committed/pushed by this audit? | **No** (report file only) |

---

## 1. Repository state

| Check | Result |
|-------|--------|
| `git status -sb` | `## main...origin/main`; tracked `M CHANGELOG.md`, `M README.md`, `M index.html` |
| `git log -1 --oneline` | `7876551 Add Dice Tray storage and insert options` |
| `git rev-parse HEAD` | `78765512f58d5a16eea89073958252bd5171b6a7` |
| `git rev-list --left-right --count origin/main...main` | `0 0` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --cached` | Empty (nothing staged) |
| HEAD remains `7876551` | **Yes** |
| Branch `main` | **Yes** |
| T1 uncommitted | **Yes** |
| Architecture / construction / implementation reports present | **Yes** (untracked under `docs/`) |
| LightBurn projects, historical docs, `debug.log`, utility script | Present untracked; not part of T1 tracked delta |

**Conclusion:** Synchronized `main` at baseline commit; T1 lives only in the working tree.

---

## 2. Actual files changed

Tracked modifications (from `git diff --stat`):

| File | Diff |
|------|------|
| `index.html` | +183 / −15 (≈198 lines touched in stat) |
| `README.md` | +1 line (T1 bullet) |
| `CHANGELOG.md` | +1 line (T1 bullet) |

Untracked T1-related docs (not in commit baseline):

- `docs/DESIGNS_TABLETOP_ACCESSORIES_ARCHITECTURE_REVIEW_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_CONSTRUCTION_CONTRACT_REVIEW_2026-07-20.md`
- `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_IMPLEMENTATION_2026-07-20.md`

Long-standing untracked assets remain outside the T1 delta (LightBurn tree, historical audits, `debug.log`, `parametric_qr_stand_generator.py`).

---

## 3. Helpers and product paths reviewed

| Area | Location / symbols |
|------|--------------------|
| Registry | `tabletopAccessoryProductRegistry` → `tabletop-corner-floor-coupon` |
| Normalization | `normalizeDesignDraft` tabletop branch (`tabletopCoupon*`) |
| Form rendering | `renderDesigns` / `tabletopFields` / `formFields` ternary |
| Product dispatch | `buildDesignResult` → `buildTabletopCornerFloorCouponDesignResult` |
| Filename / handoff | Project handoff name `Tabletop Corner + Floor Coupon`; notes with clearances |
| Result summary | tabletop `designMetric` block (candidates, cavity, envelope, pieces, labels) |
| Engine | `tabletopAccessoryRectangleOutline`, `HexagonOutline`, `WallLoop`, `CornerDescriptors`, `Dimensions`, `FinishedCavity`, `DividerFromCavity`, `InsertFromCavity`, `ManufacturingLimits`, `ProjectPoint` |
| Coupon | `buildTabletopCornerFloorCouponDesignResult` (selects box panels `front`→WALL-A, `left`→WALL-B, `bottom`→FLOOR) |
| SVG | `serializeTabletopAccessorySvg` |
| Labels | `buildAssemblyLabelPaths` + green `#00ff00` group |
| Finished View | `buildTabletopCornerFloorFinishedViewSvg` |
| Fixtures | `runTabletopAccessoryEngineFixtures`, `runTabletopCornerFloorCouponFixtures` |
| Selftest | `?selftest=tabletop-engine`, `tabletop-corner-floor-coupon`, `all` |
| README / CHANGELOG / Help | as cited below |

---

## 4. Coupon geometry findings

Each candidate is built with `buildBoxModel({ dimensionMode:'inside', width/depth: interiorLeg, height: wallHeight, …, lid:'open' })`, then **only** three panels are exported:

- `front` → `C{n}-WALL-A`
- `left` → `C{n}-WALL-B`
- `bottom` → `C{n}-FLOOR`

This is a real open-box L-corner (two walls + floor), not three unrelated rectangles. A fourth wall is not required for an open L dry-fit. Metadata strings alone were **not** treated as proof; edge phases, counts, widths, and depths were inspected on the underlying box model (and on C1 pieces after selection).

**Physical coherence (path/model level):** Pass for intended three-piece miniature corner.  
**Physical fit / kerf / glue / square:** Unverified — scrap cut still required.

---

## 5. Representative 2.88 mm coordinates and dimensions

Inputs: t=**2.88** mm, center clearance **+0.10**, step **0.05**, preferred finger **9**, interior leg **40**, wall height **22**, panel spacing **15**.

| Quantity | Measured value |
|----------|----------------|
| Valid | `true` |
| Candidates | **0.05, 0.10, 0.15** mm (distinct after `designRound`) |
| Piece count | **9** |
| Semantic IDs | `C1-FLOOR`, `C1-WALL-A`, `C1-WALL-B`, … `C3-*` (unique) |
| Requested interior | 40 × 40 mm |
| Finished cavity clear | 40 × 40 mm; usable height **22** |
| Outer envelope | **45.76 × 45.76 × 24.88** (= 40+2t, height wall+t) |
| Wall blank | 45.76 × 24.88 |
| Floor blank | 45.76 × 45.76 |
| Cut bounds / sheet | **187.28 × 187.28** mm |
| Min segment max(2t,4) | **5.76** mm |
| Horizontal finger pattern | count **5**, actual width **9.152** mm (≥ 5.76) |
| Vertical finger pattern | count **3**, actual width **≈8.293** mm (≥ 5.76) |
| Layout min gap | **15** mm; AABB overlaps: **none** |
| SVG | width/height/viewBox mm; 9 red cut panels; deterministic; FNV `4f543f95`; 2992 bytes; no NaN/Infinity; no Finished View IDs |
| Labels off | no assembly-labels group; no green |
| Labels on | group present; green stroke; `labelCount` **9** |

### C1 layout positions (mm)

| Piece | Layout x,y | Blank W×H | Closed path |
|-------|------------|-----------|-------------|
| C1-FLOOR | 10, 10 | 45.76 × 45.76 | yes |
| C1-WALL-A | 70.76, 10 | 45.76 × 24.88 | yes |
| C1-WALL-B | 131.52, 10 | 45.76 × 24.88 | yes |

---

## 6. Wall-to-wall mating verification

On `buildBoxModel` for the same inputs (clearance 0.10 representative of center candidate; C1 uses 0.05 with the same phase structure):

| Edge pair | Phases | Count | Actual width | Depth | Clearance |
|-----------|--------|-------|--------------|-------|-----------|
| Front left edge vs left right edge | **true vs false** (opposite) | 3 = 3 | 8.293… = 8.293… | 2.88 = 2.88 | equal |

WALL-A (front) and WALL-B (left) form a **90° vertical corner** with opposite phase ownership, matching count/width/depth. Terminal vertical fingers use the existing Finger Box odd-count pattern (not a free-form custom corner solver).

---

## 7. Wall-to-floor mating verification

| Pair | Phases opposite | Count match | Width match |
|------|-----------------|-------------|-------------|
| Front bottom ↔ bottom top (floor front) | **yes** | yes (5) | yes (9.152) |
| Left bottom ↔ bottom left | **yes** | yes (5) | yes (9.152) |

Floor is the same box bottom sized for the selected walls (inside 40 mm + 2t outer), not an unrelated full tray base. Unused floor edges still carry full-box finger patterns (back/right) — acceptable for a coupon cut from a full open-box model; those edges simply have no mate in the three-piece assembly.

Clearance is applied through the shared finger builder on both sides of each joint with **opposite phase**, not by enlarging both male profiles in the same direction.

---

## 8. Terminal-web verification

- Manufacturing helper: `minSegment = max(2t, 4)`, `minWeb = max(t, 1)`.
- Coupon blocks preferred width below `minSegment` and interior/height below panel/segment minima.
- Actual fingers on 2.88 mm case exceed 5.76 mm.
- No zero/negative blank dimensions observed.
- **No direct coordinate fixture** asserts terminal web thickness at the vertical-corner × floor triple junction (fixture gap — IMPORTANT, not a measured geometry failure).

---

## 9. Dimension-contract verification

Distinct fields present on the coupon result metrics:

| Field | 2.88 case |
|-------|-----------|
| `requestedInterior` | legLength/width/depth 40 |
| `finishedCavity.clearBounds` | 40 × 40 |
| `outerEnvelope` | 45.76 × 45.76 × 24.88 |
| `panelDimensions` | wall blank / floor blank / wallHeight 22 / thickness 2.88 |
| `cutBounds` | 187.28 × 187.28 |

Interior-primary coupon path: finished cavity leg **equals** requested interior (within float). Outer = cavity + 2t. Usable height is wall height **above floor top** (floor thickness included in blank height 22+t, not as “clear cavity”). Half-thickness 1.44 is used only as material offset inside box geometry, not reported as clear cavity.

**Caveat:** Coupon production geometry is **delegated to legacy `buildBoxModel` (inside mode)**. Engine descriptors restate cavity math but do not independently generate wall finger paths. Contract is preserved for the coupon’s interior-primary use; future products must not confuse box “inside” naming with unfinished engine-only fields.

---

## 10. Physical-failure regression verification

| Check | Result |
|-------|--------|
| Divider from cavity 96.73, end clearance 1 → span | **94.73** (`source: finishedCavity`) |
| Span does not use nominal 100 | **Pass** (`span < 98.73`) |
| Insert region from finished cavity | width/depth reduced by 2×clearance |
| Material tag independence | wood vs felt region JSON equal when other inputs equal |
| `physical` flag on insert | stored; region math not branched on tag |
| Silent interior shrink rejection in engine helper | **Not implemented** — `tabletopAccessoryDimensions` copies cavity; no throw if finished &lt; requested |
| Coupon interior-primary silent shrink | N/A (cavity forced from requested leg) |
| Half-thickness as cavity report | Not observed on coupon metrics |

---

## 11. Rectangle / hex shared-engine verification

| Check | Result |
|-------|--------|
| Rectangle vertices | 4 ordered: (0,0)→(w,0)→(w,d)→(0,d) |
| Wall loop 4 | ordered previous/next wrap A↔D |
| Hex vertices | 6; equal edge lengths |
| side = F/√3 | **34.641016…** for F=60 |
| point-to-point = 2×side | matches |
| Hex wall loop | 6 edges; wrap; all corners **120°** metadata |
| 120° finger production geometry | **Not emitted** (metadata/join type only) |
| Engine assumes only 4 edges | **No** — loop maps `vertices.length` |

---

## 12. Projection and Finished View verification

`buildTabletopCornerFloorFinishedViewSvg` for valid 2.88 result:

- Finite coordinates (no NaN/Infinity in SVG text)
- Nine polygons, all positive area (min area ≈ **723** screen units)
- Floor polygon drawn before walls; projection uses z-height for walls → not coplanar with floor in the schematic
- Three candidates side-by-side with clearance labels
- No groove/rebate/bevel/channel/lid/divider/insert language in FV
- Screen-only class/IDs do not appear in production SVG
- `tabletopAccessoryProjectPoint` rejects non-finite inputs (`null`)

Finger cues in FV are **schematic faces**, not true finger profiles — truthful as “screen-only schematic,” not as joint proof.

---

## 13. Assembly-label verification

| Mode | Group | Green | labelCount | Red cuts preserved |
|------|-------|-------|------------|--------------------|
| Off | absent | no | 0 | yes |
| On | `assembly-labels` | `#00ff00` | 9 | yes (same red path count pattern) |

Serializer emits one green path group with per-label nested groups (same helper as other products). Labels are vector paths, not structural cuts. IDs/text distinguish candidate and role (`C1 WALL A` style via `panel.id.replace('-',' ')`).

---

## 14. Production SVG verification

| Requirement | Status |
|-------------|--------|
| Exact mm width/height/viewBox | **187.28** mm square for 2.88 case |
| Deterministic ordering | Repeated build identical |
| Semantic panel IDs | `panel-C1-WALL-A` etc. |
| Red structural cuts | `#ff0000` in `g#cut` |
| Optional green labels | yes |
| Blue guides | none (no genuine guide layer) |
| Closed contours | paths closed with Z |
| No FV IDs in production | pass |
| No NaN/Infinity | pass |
| No auto-scaling transforms on size | translate only for layout |

---

## 15. Panel-layout verification

Nine panels: no AABB overlaps; minimum separation **15** mm (= panel spacing); sheet size matches layout bounds; no measurement-only strip; deterministic repeat.

---

## 16. Validation-boundary results

Runtime `buildDesignResult` matrix (tabletop template):

| Case | Outcome |
|------|---------|
| thickness 0 / negative | **invalid** (blocked) |
| preferred width 5 (&lt; 5.76 at t=2.88) | **invalid** |
| preferred width 5.76 (exact min) | **valid** (with other defaults) |
| short interior leg 10 | **invalid** |
| short wall height 4 | **invalid** |
| clearance step 0 | **valid** (three equal candidates possible — operator caution) |
| negative clearance step | blocked or produces invalid candidate band |
| center clearance outside −0.10…0.30 for any candidate | **invalid** |
| panel spacing 0 | **valid** (touching allowed) |
| inch-mode equivalent of 2.88 case | **valid** when converted |

Errors block production SVG; warnings do not silently rewrite geometry.

---

## 17. Focused assertion inventory

### Engine group (12 / 0) — exact names

1. rectangle outline is closed-size deterministic  
2. regular hex side and point-to-point formulas  
3. rectangle wall loop has ordered links  
4. hex wall loop has six edges  
5. corner descriptors preserve join types  
6. dimension descriptor separates requested, cavity, and envelope  
7. divider derives span from finished cavity  
8. physical cavity regression never uses nominal 100  
9. insert material tag is opaque  
10. manufacturing minimums include 2t and 4  
11. projection returns finite deterministic point  
12. projection rejects malformed points  

### Coupon group (12 / 0) — exact names

1. registry and default route  
2. default coupon is valid  
3. three candidates are ordered  
4. exactly nine physical pieces  
5. semantic IDs cover each candidate  
6. wall and floor construction metadata *(type strings only)*  
7. production SVG uses red cuts and green labels  
8. production SVG has no finished-view IDs  
9. production SVG validates finite and closed  
10. unsafe preferred width is blocked  
11. impossible finger count is blocked  
12. handoff name stays generic  

---

## 18. Missing fixture coverage (IMPORTANT)

High-risk behaviors **not** covered by direct, failure-localizing assertions:

**Engine:** silent finished-cavity shrink rejection; insert `physical` true/false geometry branch; outward normals / winding explicit asserts; terminal-web geometry at triple junction.

**Coupon:** opposite wall–wall and wall–floor phases; coordinate mating counts/widths; 2.88 representative case; layout non-overlap / spacing; labels off path; label-count vs emitted path parity; Finished View floor-beneath / finite polygons; requested=finished cavity on coupon metrics; short wall height; clearance band edges; fixture cleanup of draft/DOM/storage; form field presence (would have caught the BLOCKER).

Bundled metadata assertion (“finger-90” / “finger-jointed-floor”) is **not** equivalent to mating geometry coverage.

---

## 19. Fixture-cleanup results

Coupon/engine fixtures call pure builders and registry reads; they do **not** open modals or rewrite `localStorage` in the fixture body. No explicit save/restore of `designDraft`, unit mode, or DOM was required for these two groups. **No dedicated cleanup assertions** exist (gap). Adjacent full-suite groups that mutate state passed (31 groups / 0 failed), which is indirect evidence only.

---

## 20. Selftest registration

| Route | Registrations in source |
|-------|-------------------------|
| `tabletop-engine` | **exactly once** under `selftest === 'tabletop-engine' \|\| selftest === 'all'` |
| `tabletop-corner-floor-coupon` | **exactly once** under matching condition |

---

## 21. Exact measured fixture totals

Independently measured in disposable Edge profile (working tree), not “expected”:

| Group / standalone | Passed | Failed |
|--------------------|-------:|-------:|
| tabletop-engine | 12 | 0 |
| tabletop-corner-floor-coupon | 12 | 0 |
| design (geometry) | 1093 | 0 |
| dice-tray-system | 92 | 0 |
| gift-box | 69 | 0 |
| runTrayModelFixtures | 264 | 0 |
| promotion-switch standalone | 16 | 0 |
| machines-m1 | 29 | 0 |
| machines-m2 | 31 | 0 |
| machines-m3 | 27 | 0 |
| **Complete registered suite (31 groups)** | **2478** | **0** |

Claimed post-T1 totals **match measurement** for engine, coupon, design, and complete suite.

---

## 22. Complete-suite arithmetic

Measured aggregate: **2478** = pre-T1 **2454** + engine **12** + coupon **12**.  
Group count: **31** = prior **29** + **2**.  
Arithmetic consistent with measured group table in the probe (`allGroups.details`).

---

## 23. Legacy-output comparison

Direct Edge dump of committed HEAD timed out in this environment. Stronger structural proof:

**Byte-identical function bodies HEAD vs working tree** for:

- `buildBoxModel`
- `buildTrayDesignResult`
- `buildGiftBoxDesignResult`
- `buildSlidingLidDesignResult`
- `buildDrawerCabinetDesignResult`
- `buildJointCouponDesignResult`
- `buildBoxDesignResult`
- `buildLegacyDesignResult`
- `layoutDesignPanelRows`
- `buildFingerPattern`
- `designFixtureHash`

T1 only **adds** dispatch for `tabletop-corner-floor-coupon` and does not rewrite legacy serializers’ function bodies.

Working-tree FNV pins (default drafts, for regression notebooks):

| Product | Bytes | FNV |
|---------|------:|-----|
| Dice Tray | 1726 | `51a55721` |
| Dice Tray tab-slot | 1054 | `41697123` |
| Divider Tray | 1965 | `a55dda6e` |
| Finger Box (labels on) | 4470 | `3aa538d3` |
| Gift Box (labels on) | 6859 | `647c8367` |
| Sliding-lid (labels on) | 5045 | `85247b6f` |
| Drawer Cabinet (labels on) | 8194 | `1c009e7f` |
| Joint Fit Coupon | 7764 | `db7ea7e9` |

No unexplained legacy drift indicated. Focused product suites (design 1093, dice 92, gift 69) remain green.

---

## 24. In-app Help assessment

Implementation report §33: *“The existing Help modal was intentionally left unchanged.”*

| Required topic | In Help modal? | On Designs form when coupon works? |
|----------------|----------------|-------------------------------------|
| Why separate from Dice Tray | **No** dedicated section | Partial (T1 copy exists in dead `tabletopFields` / note strings) |
| Rectangle finger walls + floor | No | Intended form muted copy |
| Future hex different corner strategy | No | Not in Help |
| Coupon tests wall corner + floor | No | Intended form copy |
| Clearance starting point; measured thickness; kerf separate; dry-fit | Generic Designs/LightBurn + safety sections only | Partial |
| No groove/rebate/bevel/CNC pocket | No explicit Tabletop section | Intended form copy |
| Fire watch / ventilation / focus / flat material / framing / unattended | **Yes** (global Laser safety) | — |

**Assessment:** Omitting a Tabletop Accessories Help section **violates the accepted implementation prompt** for a paid-release-quality offline app. Form copy is **not** equally discoverable (and currently **not even rendered** — see BLOCKER). README-only is insufficient.

**Classification:** IMPORTANT (would be sufficient alone to block commit even if form were fixed).

---

## 25. Documentation assessment

| Item | Status |
|------|--------|
| README describes T1 as engine + coupon only | **Yes** |
| README measured totals 2478 / 31 | **No** — still states **2454 / 0 / 29 groups** and omits tabletop selftest query tokens in the list |
| README claims physical validation done | **No** |
| CHANGELOG overclaims full tray / hex product / captured floor / schema | **No** — appropriately bounded |
| Implementation report lists Help intentional omit | **Yes** (accurate about intent; still a contract gap) |
| Implementation report “expected 2478” | Matches measured; wording is “expected,” not “measured” |
| Implementation report §7 “form exposes …” | **Inaccurate relative to runtime UI** due to form ternary bug |
| Terminology finger-jointed floor | Consistent; not “captured/rebated” |
| Physical testing required | Stated |

---

## 26. Direct file:// and disposable-browser results

| Check | Result |
|-------|--------|
| `python -m html.parser index.html` | exit **0** |
| `git diff --check` | clean |
| Disposable Edge probe of instrumented index | startup OK; builders/fixtures OK |
| Programmatic 2.88 coupon | valid, 9 pieces, layout/SVG OK |
| Labels off/on | as §13 |
| Invalid short leg / pref width | blocked |
| Deterministic repeat | yes |
| Focused routes + full suite | §21 |
| Form UI selection of Tabletop coupon | **Broken** — see BLOCKER |
| Console exceptions in probe path | none for generation/fixtures |

---

## 27. Protected-boundary comparison

| Constant / area | Status |
|-----------------|--------|
| APP_ID / APP_NAME / APP_VERSION 0.9.0 | Unchanged |
| BUILD_DATE 2026-07-19 | Unchanged |
| STORAGE_KEY `genmitsu-l8-tracker-v1` | Unchanged |
| SCHEMA_VERSION 2 | Unchanged |
| BACKUP_FORMAT | Unchanged |
| Machines M1/M2/M3, promotion, storage recovery | Suites pass; not rewritten by T1 |
| Project schema / accounting / inventory / pricing | Untouched |
| Legacy product builders (function identity) | Identical to HEAD |
| Kerf conventions / offline behavior | Unchanged |
| LightBurn color contract for legacy | Unchanged |

---

## 28. Findings by severity

### BLOCKER (1)

1. **Designs form wiring erases the Tabletop coupon UI.**  
   In `renderDesigns`, the `formFields` ternary resolves `tabletop` to a **plain note string**, not `${templateField}${materialThicknessField}${tabletopFields}`. The correct branch is **dead code** after an unreachable second `tabletop` check.  
   **Effect:** Selecting Tabletop Corner + Floor Coupon removes the template selector and all coupon inputs (thickness, clearances, leg, height, spacing, labels). Operators cannot enter measured **2.88 mm** stock or recover the form without leaving Designs state by other means. Implementation report §7 claims the form exposes these controls — it does not at runtime.  
   **This alone blocks controlled physical coupon cutting and commit.**

### IMPORTANT (6)

1. **Required in-app Help section omitted** (intentional per implementation report; still contract-violating for discoverability/safety parity with Gift Box / Dice Tray Help sections).  
2. **README complete-suite totals still 2454 / 29** and selftest list omits tabletop routes — inaccurate after measured 2478 / 31.  
3. **High-risk fixture gaps** for mating phases, terminal web, 2.88 case, layout non-overlap, FV geometry, form presence, silent-shrink, cleanup (§18).  
4. **Result summary does not report actual finger width** (only preferred input and other metrics) — operator cannot see 9.152 / 8.293 without inspecting geometry.  
5. **Implementation report overstates form exposure** relative to runtime.  
6. **`note` under the form does not special-case tabletop**, so freestanding-sign fallback note can appear beneath a broken tabletop form.

### POLISH (2)

1. Floor retains finger profiles on edges without mates in the three-piece set (harmless scrap; slightly wasteful).  
2. Dead second `tabletop` formFields branch should be removed when fixing the ternary (cleanup).

### NOT A DEFECT

- No complete premium tray, no user-facing hex product, no 120° finger production, no lid/divider/insert/rebate/groove/pocket, no schema migration — intentional T1 limits.  
- Reusing `buildBoxModel` for coupon cut paths is acceptable if cavity contract remains honest (it does for interior-primary).

---

## 29. Exact final verdict (restated)

**CHANGES REQUIRED BEFORE PHYSICAL CUTTING**

Geometry and shared engine groundwork are largely sound and fixture-green, but the **operator-facing form is non-functional** for the coupon, Help/README/fixture/disclosure gaps remain, and commit is not appropriate.

---

## 30. Remaining unverified physical areas

- Actual laser kerf and joint fit on measured stock  
- Squareness after dry-fit and glue  
- Triple-junction finger collision under real kerf  
- Clamp access and glue-up sequence  
- Long-term strength  
- Inch-mode operator workflow after form fix  
- Visual LightBurn import scale confirmation by human

---

## 31. Whether the coupon is safe to cut

| Path | Safe? |
|------|-------|
| Programmatic / default-draft generation of geometry for scrap (after independent SVG inspection) | **Conditionally yes** for exploratory scrap only; not production proof |
| In-app controlled cut of a **measured 2.88 mm** case | **No** until form BLOCKER is fixed |
| Production / sold product use | **No** — construction coupon only |

---

## 32. Whether T1 may be committed

**No.** Fix form wiring, Help, README totals, and strengthen high-risk fixtures (at minimum form presence + mating/phases + 2.88 case) before commit.

---

## 33. Whether a correction pass is required

**Yes.** Minimum correction set:

1. Fix `formFields` ternary so tabletop renders `templateField + materialThickness + tabletopFields` and a proper note.  
2. Add Tabletop Accessories T1 section to Help (or equally discoverable in-app equivalent covering required topics).  
3. Update README measured suite totals and selftest list.  
4. Add direct fixtures for form fields, opposite phases / mating, and 2.88 representative validity.  
5. Prefer reporting actual finger width in the result summary.

---

## 34. Whether Claude is needed

**No.** Defects are local to T1 form wiring, documentation, Help, and fixture depth. Shared box builders remain identical to HEAD; no multi-product shared-helper regression was found.

---

## 35. Confirmation of audit hygiene

This audit did **not** edit, stage, commit, push, reset, clean, stash, checkout, move, rename, or delete any repository product file.  
The only write performed for the deliverable is this report:

`C:\Genmitsu L8 Tracker\docs\DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_FOCUSED_AUDIT_2026-07-20.md`

Temporary Edge profiles and probe scripts under OS temp / worktree `agent-tools/` were used for measurement only and are not product commits.

---

## Appendix A — Probe evidence

- Disposable Edge audit JSON: `%TEMP%\t1audit-soq6_jh5\audit.json` (and prior `t1audit-7ctn2vxt`)  
- Worktree probe scripts (non-product): `agent-tools/t1_audit_probe.py`, `agent-tools/t1_legacy_pin.py`  
- Complete suite arithmetic measured 2026-07-20 in browser harness: **2478 / 0 / 31 groups**

---

*End of focused audit.*
