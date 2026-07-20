# Designs Gift Box G1 — Correction Re-verification

**Date:** 2026-07-19  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Review type:** Narrow read-only re-verification of focused-audit corrections  
**Committed baseline:** `769069d` — *Add machine identity to production records*  
**Uncommitted phase:** Designs Gift Box G1 + focused-audit correction  

---

## Final verdict

# **APPROVED TO COMMIT**

| Severity | Count |
|----------|------:|
| BLOCKER | **0** |
| IMPORTANT | **0** |
| POLISH | **0** |
| NOT A DEFECT | noted below |

All six focused-audit correction themes are complete. Production geometry isolation holds. Existing Designs and M1–M3 remain green. A second full Gift Box architecture audit is **not** required.

**Gift Box G1 may be committed.**  
**Dice Tray architecture work may begin after commit** (separate change set).

---

## 1. Repository state

| Check | Result |
|-------|--------|
| `git status -sb` | `## main...origin/main`; modified: `CHANGELOG.md`, `README.md`, `index.html` |
| `git log -1 --oneline` | `769069d Add machine identity to production records` |
| `git rev-parse HEAD` | `769069d5a3c8bc2e74d2da912cd2257c10288f96` |
| `origin/main...main` | `0 0` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --stat` | 3 files, **+309 / −28** |
| `git diff --cached` | Empty — **nothing staged** |
| Gift Box commit/push | **None** — still uncommitted |
| Branch | `main` |

### Tracked modifications (intended)

| File | Role |
|------|------|
| `index.html` | Gift Box G1 + Finished View / label / fixture corrections |
| `README.md` | Gift Box docs + totals **69 / 0**, complete **2362 / 0**, **28** groups |
| `CHANGELOG.md` | G1 feature + correction note |

### Relevant reports present (untracked docs)

- `docs/DESIGNS_GIFT_BOX_ARCHITECTURE_REVIEW_2026-07-19.md`
- `docs/DESIGNS_GIFT_BOX_G1_IMPLEMENTATION_2026-07-19.md`
- `docs/DESIGNS_GIFT_BOX_G1_FOCUSED_AUDIT_2026-07-19.md`
- `docs/DESIGNS_GIFT_BOX_G1_CORRECTION_2026-07-19.md`

### Untouched unrelated material

`LightBurn Projects/`, `debug.log`, historical docs, `parametric_qr_stand_generator.py` — present, **not modified by this verification**.

---

## 2. Files / functions reviewed

| Item | Review |
|------|--------|
| `buildGiftBoxFinishedViewSvg()` | Lid projection, `point` / `flatPoint` / `poly`, hinged rotation, lift-off aside |
| Locating-frame FV geometry | Frame rects via `lidProject` (moves with lid) |
| Schematic hinge markers | `gift-box-hinge` line + circle on lid space |
| Captions / labels | Closed three-quarter; lift-off removed aside; hinged ~90° |
| `designAssemblyLabelSpecs()` Gift branch | Conditional FRAME-FRONT/BACK; HINGE-COUPON only when hinged + coupon |
| Result summary (`designResultsHtml` gift branch) | Closed overall height; interchangeable side-rail copy |
| `runGiftBoxFixtures()` | 69 assertions; coordinate-level FV checks |
| Selftest registration | `gift-box` once under `all` |
| Production path | `buildGiftBoxDesignResult` → `serializeDesignSvg` |
| Existing Design fixture group | `runDesignGeometryFixtures` **1093 / 0** |
| README / CHANGELOG / implementation follow-up | Totals and correction note |

---

## 3. Lid coordinate verification

Direct DOMParser checks on `#gift-box-lid` for default and **user scenario** (internal 120×90×50, thickness **2.88**, lift-off flush, frame on):

| Mode | Finite | No NaN/Inf | ≥4 pts | Area > 0 | Non-degenerate bounds | Deterministic FV |
|------|:------:|:----------:|:------:|:--------:|:---------------------:|:----------------:|
| Lift-off Closed | Yes | Yes | 4 | Yes | Yes | Yes |
| Lift-off Open | Yes | Yes | 4 | Yes | Yes | Yes |
| Hinged Closed | Yes | Yes | 4 | Yes | Yes | Yes |
| Hinged Open | Yes | Yes | 4 | Yes | Yes | Yes |

- Points are `{x,y}` objects fed to `poly()` via `designRound(p.x),designRound(p.y)` — **no array/object point-shape mismatch**.
- Fixture assertion **“Finished View lid coordinates are finite and non-degenerate”** parses real `points` attributes (not captions only).

---

## 4. Default user-scenario verification

**Inputs (direct `file://` / in-memory bridge equivalent):** Gift Box · Internal · 120×90×50 mm · material **2.88** mm · Lift-off · Flush · Locating frame on · Assembly labels on.

| Check | Result |
|-------|--------|
| Result valid | **Yes** (`errors: []`, `warnings: []`) |
| Production SVG valid / deterministic | **Yes** |
| Closed FV caption | “Closed three-quarter view” |
| LID label present near lid polygon | **Yes** |
| Open caption | “Lift-off lid removed aside” (no 90° hinge language) |
| Open lid full-size | Area ratio Open/Closed ≈ **1.0** |
| Open moved aside | First lid point X +80+ from Closed |
| Frame rails with lid on Open | **4** polygons; frame points move with lid |
| HINGE-COUPON omission warning | **Absent** |
| FRAME-LEFT/RIGHT omission warning | **Absent** |
| “Identical and interchangeable” guidance | **Present** in assembly message |
| Label count | **8** paths = BASE/FRONT/BACK/LEFT/RIGHT/LID/FRAME-FRONT/FRAME-BACK |
| Physical pieces | **10** panels (shell+lid+4 rails) |
| Closed overall height | `metrics.closedHeight === outsideHeight + t` (**55.76** mm for 2.88 stock) |

---

## 5. Lift-off Closed

- Lid polygon seats over shell top projection (`pz = top`).
- Flush footprint matches shell outside (fixture + metrics).
- Overhanging appearance produces **larger** projected lid bounds (verified).
- Frame rails rendered **with** lid (closed: frame before lid paint order; coordinates from lid project space).

---

## 6. Lift-off Open

- Lid uses `flatPoint(liftOriginX, liftOriginY, …)` — **removed aside**, not hinge kinematics.
- Title/caption: **lift-off lid removed aside**.
- No “approximately 90 degrees” language.
- Area preserved (~full size); not a miniature.
- Frame rails: **4** IDs present; first point of `frame-front` **differs** Closed→Open (moves with lid).

---

## 7. Hinged Closed

- Finite seated lid; all Y &lt; 230 in fixture check.
- Surface hinge markers present (`gift-box-hinge`).
- Caption: flush/overhanging · surface hinges · closed three-quarter.

---

## 8. Hinged Open

| Check | Live result |
|-------|-------------|
| Rear hinge axis fixed | Hinge circle `cx/cy` identical Closed→Open; rear lid corner **Δ = 0** |
| Opposite (front) edge moves | First point Y delta **&gt; 10** |
| Caption ~90° | **Yes** (“Hinged lid approximately 90 degrees open”) |
| Not mere Open caption of lift-off | Distinct hinged path |
| 1 / 2 / 3 hinges | Marker counts **1 / 2 / 3** in Open FV |
| Flush + overhanging | Supported; overhang FV attribute present |
| Coupon on/off | Panel + label only when hinged + coupon enabled |

**NOT A DEFECT:** Screen-space edge orientation cosine can remain ~1 under the isometric pitch model even though the lid is kinematically rotated about the rear axis (fixed rear, moved front). Fixtures correctly prove axis fixity + front motion + caption semantics rather than requiring a 2D edge-angle flip.

---

## 9. Locating-frame visualization

| Check | Result |
|-------|--------|
| Four rail polygons when frame enabled | **Yes** |
| Closed rails with lid | **Yes** |
| Open lift-off rails move with lid | **Yes** |
| Not shell-only floating coords | Rails use same `lidProject` as lid |
| Frame + hinge blocked | **Yes** (generation invalid) |
| Production cut layout still emits four rails | **Yes** (panel IDs present) |

---

## 10. Label-spec and warning behavior

`designAssemblyLabelSpecs('gift-box', …)`:

| Mode | Specs |
|------|--------|
| Lift-off + frame | BASE, FRONT, BACK, LEFT, RIGHT, LID, **FRAME-FRONT**, **FRAME-BACK** only |
| FRAME-LEFT / FRAME-RIGHT | **Not attempted** (intentional interchangeability) |
| Lift-off | **No** HINGE-COUPON |
| Hinged + coupon | … + **HINGE-COUPON** |
| Hinged + coupon off | No HINGE-COUPON; no coupon panel |

- Default lift-off result: **no** irrelevant HINGE-COUPON / FRAME-LEFT/RIGHT omission warnings.
- Emitted label count matches `labelPaths.length` (fixture).
- Other templates’ label logic unchanged (Designs geometry **1093 / 0**).

---

## 11. Result-summary verification

| Check | Result |
|-------|--------|
| Closed overall height metric | Present via `designMetric('Closed overall height', designNumberText(result.metrics.closedHeight) + ' mm')` |
| Uses `metrics.closedHeight` | **Yes** (`outsideHeight + t`, not double base+lid wrongly) |
| Finished internal/external metrics | Unchanged structure |
| Side-rail interchangeability | **Information** in Assembly message, not a warning |
| Physical pieces / labels | Accurate for scenario |

---

## 12. Fixture assertion inventory

**Independent recount:** **69** `add(...)` calls in `runGiftBoxFixtures()` — matches runtime **69 / 0**.

Prior total **60**; correction added **9** coordinate/semantic/label/warning assertions (60 + 9 = 69; complete 2353 + 9 = 2362).

Coordinate-level (not caption-only) examples:

- Finished View lid coordinates finite and non-degenerate  
- Closed lift-off overlap  
- Open lift-off full-size + aside  
- Closed hinged finite seating  
- Open hinged fixed rear axis + moved front  
- Frame rails on Open  
- Conditional labels / no irrelevant warnings  
- Production vs Finished View separation  

All **69** passed on product `file://` and on instrumented verify page.

---

## 13. Fixture cleanup

Gift Box fixtures primarily call pure `buildDesignResult` / FV builders. The result-summary assertion temporarily swaps `designDraft` and restores it in `try/finally`. No durable storage writes required for generation (session-only Designs). Subsequent M1/M2/M3 and Designs groups remain green after Gift Box runs — no suite pollution observed.

---

## 14. Production SVG comparison

### Isolation

| Check | Result |
|-------|--------|
| Finished View IDs in production SVG | **None** (`gift-box-finished-view` / `#gift-box-lid` absent) |
| Cut red / score blue | Preserved |
| Deterministic repeated generation | **Yes** |
| Existing templates byte-stable | **Yes** (see hashes) |

### Correction-report hash reproducibility

Correction report compared production SVGs with **`assemblyLabels: false`** (geometry-only). Under that identical input path, current tree **exactly matches**:

| Output | Bytes | SHA-256 | Match |
|--------|------:|---------|:-----:|
| Gift lift-off (no labels) | 3210 | `5302a64cba33b05e03ea646f9119f6545e4bebf15685c6f7a95df58b6c6894d9` | **Yes** |
| Gift hinged (no labels) | 5490 | `558b9af6e1898dc023b742a4e60a418000d4675496c4de003fa92a58b085dda4` | **Yes** |
| Finger Box | 2483 | `62cc24a3055852797a4f23c9cb224da5f9d3c76fdc5cefc428e90df3ea0d410e` | **Yes** |
| Sliding Lid | 2818 | `5eb28ea7dfcaeb02fa3ecd9c5a4e8e6169453e2b1869fc92e8306e2d284bdb15` | **Yes** |
| Drawer Cabinet | 4456 | `21739161b3f8719f711f95ca9331943ef6b7dec92c577620963661fd0aec6a28` | **Yes** |
| Tray (dice-tray) | 1726 | `2f3a5c091002d79122bcb4051a8ad9fb0ca4fe9381d5266cd8edb2fcf70f01bf` | **Yes** |
| Joint Fit Coupon | 7764 | `e4f03deb042a52e5f08389fcd1e780bb5fc6bee511a1a7600ae2f09a03f7263d` | **Yes** |
| Wall/base coupon | 1551 | `e5758d7104573489428651b0dcfda3f2281ae2dc1aa5292974a30c0aaf8b3ae0` | **Yes** |

**NOT A DEFECT:** Default Gift Box with **assembly labels enabled** produces longer SVGs (e.g. 6859 / 7537). That is expected label path content, not a production regression. Hash mismatch only appears when comparison inputs omit the no-label constraint used in the correction report.

---

## 15. Existing-output regression

| Group / template | Result |
|------------------|--------|
| Designs geometry | **1093 / 0** |
| Finger / open-top / sliding / cabinet / tray / coupons | Valid; golden hashes match where applicable |
| Design-to-Project (handoff fixtures in suite) | Included in complete **2362 / 0** |
| M1 / M2 / M3 | **29 / 0**, **31 / 0**, **27 / 0** |

---

## 16. Fixture arithmetic

| Suite | Expected | Live |
|-------|----------|------|
| Gift Box focused | 69 / 0 | **69 / 0** |
| Prior Gift Box | 60 / 0 | Documented baseline before +9 |
| Previous complete | 2353 | 2353 + 9 = 2362 |
| Complete | 2362 / 0 | **2362 / 0** |
| Complete-suite groups | 28 | **28** (no missing runners) |
| Existing Designs geometry | 1093 / 0 | **1093 / 0** |
| M1 / M2 / M3 | 29 / 31 / 27 | **Match** |
| Tray / promotion-switch standalone | 264 / 16 | **264 / 0**, **16 / 0** |
| `gift-box` under `all` | once | **Exactly one** registration |
| README / implementation report | 69 / 2362 / 28 | **Match runtime** |

---

## 17. Direct `file://` results

| Run | Result |
|-----|--------|
| Product `?selftest=gift-box` | **69 / 0**, no page exceptions |
| Product re-invoke Gift Box | **69 / 0** |
| Product Designs geometry | **1093 / 0** |
| Product M3 | **27 / 0** |
| Full 28-group sum | **2362 / 0** |
| Tray / promotion-switch | **264 / 0**, **16 / 0** |
| `git diff --check` | Clean |
| `python -m html.parser index.html` | Exit **0** |

Browser console may still show pre-existing SVG `height="auto"` warnings on broad runs — **NOT A DEFECT** for this phase; no page exceptions.

---

## 18. Documentation results

| Doc | Status |
|-----|--------|
| README Gift Box **69 / 0** | **Pass** |
| README complete **2362 / 0**, **28** groups | **Pass** |
| Implementation report correction follow-up | **Pass** |
| Correction report vs source/runtime | **Pass** |
| CHANGELOG Finished View + conditional labels | **Pass** |
| No false physical-validation / universal hinge claims | **Pass** |

---

## 19. Protected-boundary comparison

Unchanged:

- `APP_ID`, `APP_NAME`, `APP_VERSION` (`0.9.0`), `BUILD_DATE` (`2026-07-19`)
- `STORAGE_KEY`, `SCHEMA_VERSION` (2), `BACKUP_FORMAT`
- Storage / machines / M1–M3 identity / promotion
- Project schema / accounting / Inventory / Pricing
- Kerf convention (not baked into geometry)
- Existing Design geometry goldens (hashes above)
- LightBurn colors; existing filenames patterns for non-Gift templates
- Import/export/backup; offline `file://` behavior

---

## 20. Findings by severity

### BLOCKER

*None.*

### IMPORTANT

*None.* All six audit correction themes verified.

### POLISH

*None required for commit.*

### NOT A DEFECT

| Item | Reason |
|------|--------|
| Gift labeled SVG longer than 3210/5490 hashes | Correction hashes are **no-label** production captures; both paths deterministic |
| Isometric hinged edge cosine ~1 | Schematic pitch rotation with fixed rear axis; fixtures prove motion correctly |
| Schematic Finished View | Preview only; not production geometry |

---

## 21. Exact final verdict

**APPROVED TO COMMIT**

---

## 22. Remaining unverified areas

| Area | Note |
|------|------|
| Human pixel-perfect UI click tour of every preview toggle | Equivalents covered by FV SVG parse + fixtures |
| Physical cut / hinge drill / glue-up | Out of scope for screen correction |
| Network multi-browser matrix | Offline Edge sufficient |

---

## 23. Physical prototype readiness

Correction does **not** claim physical validation. Ready for the pre-existing G1 physical plan (fit coupon, hinge coupon scrap, measured stock) **after** commit — not proven by this verification.

---

## 24. Physical laser testing required for this correction?

**No** — screen-only FV, labels, warnings, fixtures.  
**Yes** before trusting Gift Box for real workshop production (unchanged G1 safety plan).

---

## 25. Whether Gift Box G1 may be committed

**Yes.** Commit intended tracked files (`index.html`, `README.md`, `CHANGELOG.md`) and optionally the Gift Box docs. Do not mix Dice Tray work into the same commit.

---

## 26. Whether Dice Tray architecture may begin after commit

**Yes**, after Gift Box G1 is committed on a clean baseline.

---

## 27. Confirmation — verification did not mutate the product tree

| Action | Performed? |
|--------|------------|
| Edit product sources | **No** |
| Stage / commit / push | **No** |
| Reset / clean / stash / checkout / move / rename / delete | **No** |
| Temporary instrumented HTML outside repo for deep probes | Yes (system temp only; not product) |
| Write this verification report | **Yes** → `docs/DESIGNS_GIFT_BOX_G1_CORRECTION_VERIFICATION_2026-07-19.md` |

---

*End of Gift Box G1 correction re-verification.*
