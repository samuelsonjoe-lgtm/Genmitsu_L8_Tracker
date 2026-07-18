# Concealed Cleat Full-Box Prototype (J2.5) — Adversarial Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit date:** 2026-07-18  
**Audit type:** focused, adversarial, **read-only**  
**Baseline commit (expected and actual HEAD):** `30ab1f9` — *Fix concealed cleat placement guides*  
**Full HEAD:** `30ab1f96021dae8170056f61f7ad703b5de4e6c4`  

**Authoritative design contract:** `docs/DESIGNS_CONCEALED_CLEAT_FULL_BOX_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md` (untracked; not modified)  
**Implementation report (claims under test):** `docs/DESIGNS_CONCEALED_CLEAT_FULL_BOX_PROTOTYPE_IMPLEMENTATION_2026-07-18.md` (untracked; not modified)  
**Prior J2 materials consulted:** corrected J2 design review, J2 implementation report, J2 adversarial audit  

This audit did **not** edit, stage, commit, push, reset, clean, stash, move, rename, or delete any application file, design review, implementation report, LightBurn project, utility, or other unrelated working-tree content. The only file created is this audit report.

---

## 1. Repository state (verified before review)

| Check | Result |
| --- | --- |
| `git rev-parse HEAD` | `30ab1f96021dae8170056f61f7ad703b5de4e6c4` |
| `git log -1 --oneline` | `30ab1f9 Fix concealed cleat placement guides` |
| Branch | `main` |
| `git rev-list --left-right --count origin/main...HEAD` | `0 0` — **fully synchronized** with `origin/main` |
| Staging | empty |
| Tracked modifications | `index.html`, `README.md` only |
| Diff vs `30ab1f9` | `README.md` ~5 lines; `index.html` **+129 / −16** (≈118 insertions / 16 deletions net in stat) |
| `git diff --check 30ab1f9` | **clean** |

### Complete working-tree status

**Modified (tracked):**

- `index.html`
- `README.md`

**Untracked (left untouched):**

- `docs/DESIGNS_CONCEALED_CLEAT_FULL_BOX_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md`
- `docs/DESIGNS_CONCEALED_CLEAT_FULL_BOX_PROTOTYPE_IMPLEMENTATION_2026-07-18.md`
- Many historical `docs/*` review/audit files
- `LightBurn Projects/`
- `debug.log`
- `parametric_qr_stand_generator.py`

Nothing was staged, committed, or pushed during this audit.

---

## 2. Reviewed files and symbols

| Area | Symbols / locations |
| --- | --- |
| Defaults / UI | `designDefaults()` (`boxOutsideWidth/Depth`, `boxWallHeight`); `renderDesigns()` template + `fullCleatFields` |
| Normalization | `normalizeDesignDraft()` — `outsideWidth` / `outsideDepth` / `wallHeight` |
| Shared J2 helpers | `concealedCleatRectIntersection`, `concealedCleatGuideSegments` (J2 only), `serializeConcealedCleatPathGroup`, `serializeConcealedCleatSvg` |
| J2.5 core | `concealedCleatFullBoxGuideSegments`, `buildConcealedCleatFullBoxDesignResult`, `serializeConcealedCleatFullBoxSvg` (~3310–3351) |
| Dispatch | `buildDesignResult` early return for `concealed-cleat-full-box-prototype` |
| Preview | `buildConcealedCleatFullBoxFinishedViewSvg`; `designPreviewModeForTemplate` / `designResultsHtml` |
| J2 protection path | `buildConcealedCleatCornerDesignResult` + J2 goldens in fixtures |
| Fixtures | 29× `add('Full-box …')` plus retained J2 Concealed Cleat suite |

---

## 3. Commands and tests actually run

| Action | Result |
| --- | --- |
| `git status`, `rev-parse`, `log -1`, `rev-list`, `diff --stat`, `diff --check` | HEAD `30ab1f9`; origin sync `0 0`; check clean |
| Full `git diff 30ab1f9` inspection | J2.5 template + fixtures + README; reuses J2 local serializer |
| `python -m html.parser` on `index.html` | **OK** |
| Independent geometry (audit-local rectangle + z math) | All six base pairs area 0; seam/base z-overlap 0 |
| Headless Edge (`playwright` / `msedge`) on temp **copy** with hooks only | `?selftest=all` + probe |
| Before/after J2 rebuild around J2.5 generation/preview | J2 bytes stable |
| Before/after localStorage + `backupObject()` around J2.5 generate/preview | Unchanged; no field leaks |

Temp runners lived only under the audit worktree (`agent-tools/`); product tree was not patched on disk beyond reading.

---

## 4. Fixture totals (actual live)

### Complete suite (`?selftest=all`)

| Group | Passed | Failed |
| --- | --- | --- |
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Production settings | 66 | 0 |
| Evidence promotion | 58 | 0 |
| Design production settings | 118 | 0 |
| Grid promotion | 23 | 0 |
| Test Grid machine identity | 18 | 0 |
| Grid browser | 67 | 0 |
| Material browser (final) | 57 | 0 |
| Library browser | 56 | 0 |
| Project browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Design geometry | **1069** | **0** |
| **Total** | **1861** | **0** |

| Suite | Claimed | Actual |
| --- | --- | --- |
| Tray-model | 264 / 0 | **264 / 0** |
| Designs geometry | 1069 / 0 | **1069 / 0** |
| Complete | 1861 / 0 | **1861 / 0** |

**New focused Full-box assertions:** **29** (`add('Full-box …')`).  
**Arithmetic vs baseline at `30ab1f9`:** design review recorded Designs **1040** / complete **1832**; live now **1069** / **1861** → **+29 / +29**.  
README states `1841 + 20 = 1861` — **totals are correct; delta narrative is wrong** (should be `1832 + 29` or `792 + 1069`).

---

## 5. Independent dimensional calculations (defaults t=3, Ow=180, Od=150, H=90)

```text
cleatLayers = clamp(round(6/3),1,3) = 2
B = 6; W = max(9,10) = 10; guideMinRun = 18
internal = 174 × 144 × 87
Wall A/C = 180 × 90; Wall B/D = 144 × 90; Base = 180 × 150
BA/BC length = 174; BB/BD length = 124; seam length = 81
pieces = 5 + 8×2 = 21
```

Production metrics match for walls, internals, layers, piece count, and all four base footprints.

### 5.1 Six horizontal base-cleat intersections (audit-local formula)

Formula:  
`xOverlap = max(0, min(maxX)−max(minX))`, `yOverlap` analogously, `area = xOverlap × yOverlap`.

| Pair | x overlap | y overlap | area | Classification |
| --- | ---: | ---: | ---: | --- |
| A ∩ B | 10 | 0 | **0** | boundary touch at `y = t+W` |
| A ∩ C | 174 | 0 | **0** | fully separated in y (`Od > 2(t+W)`) |
| A ∩ D | 10 | 0 | **0** | boundary touch at `y = t+W` |
| B ∩ C | 10 | 0 | **0** | boundary touch at `y = Od−t−W` |
| B ∩ D | 0 | 124 | **0** | fully separated in x (`Ow > 2(t+W)`) |
| C ∩ D | 10 | 0 | **0** | boundary touch at `y = Od−t−W` |

Production footprints match independent rectangles for BA/BB/BC/BD. Same zero-area result at **t=6** (`pairsAllZero`).

Minimums: `Ow > 2(t+W)=26`, `Od > 2(t+W)=26` at t=3; hard errors use `Ow <= 2(t+W)` / `Od <= 2(t+W)` (equality rejected).

### 5.2 Seam / base z-range (independent)

| Stack | z-range |
| --- | --- |
| All base cleats | `[0, B] = [0, 6]` |
| All four seams (wall-local y = height) | `[B, H−t] = [6, 87]` |
| **z-overlap length** | **0** (contact only at plane `z = B`) |

Seam length `H − B − t = 81` (3 mm) / `78` (6 mm). Same-wall seams AD vs AB: u-ranges disjoint (`sameWallSeamOverlap = 0`).

### 5.3 Final-wall descent (geometry only)

| Body | Footprint | Nearest seam u-range | Clear? |
| --- | --- | --- | --- |
| Wall B material | `x ∈ [Ow−t, Ow]` | Seam A-B / C-B end at `x = Ow−t` | **Yes** (body does not enter seam u-interval) |
| Wall D material | `x ∈ [0, t]` | Seam A-D / C-D start at `x = t` | **Yes** |
| Wall D vs Wall B path | Opposite sides of box | — | Wall D cannot block Wall B’s vertical drop |

**Physical clamp/glue access remains unverified** (correctly warned in software).

### 5.4 Interior metrics (algebraic dual check)

```text
intrusion = interior − clear
          = (Ow−2t)(Od−2t) − (Ow−2t−2W)(Od−2t−2W)
          = 2W(Ow−2t) + 2W(Od−2t) − 4W²
```

At defaults: both forms give **5960 mm²**; clear floor **154×124 = 19096**; occupancy **≈23.79%**; approximate volume `clearArea × internalHeight = 1,661,352 mm³` (labeled approximate in semantic model / UI).

### 5.5 Layer / piece counts at other thicknesses

| t | layers | pieces |
| ---: | ---: | ---: |
| 3 | 2 | **21** |
| 2.5 | 2 | 21 |
| 5.96 | 1 | 13 |
| 6 | 1 | **13** |

Production at 6 mm: layers 1, W 18, Wall B width 138 — matches independent formulas.

---

## 6. Contract verification by priority

### 6.1 Outside-dimension integrity

Ow/Od/H preserved as outside; internals derived only; Wall A/C full width 180; Wall B/D yield by **2t** (144 at 3 mm). No inside→outside substitution found.

### 6.2 Four-wall ownership (all four corners checked)

| Corner | Owning wall | Seam host | Yielding wall | Labels |
| --- | --- | --- | --- | --- |
| A-D | Wall A | `seam-ad` on Wall A | Wall D | S-AD |
| A-B | Wall A | `seam-ab` on Wall A | Wall B | S-AB |
| C-D | Wall C | `seam-cd` on Wall C | Wall D | S-CD |
| C-B | Wall C | `seam-cb` on Wall C | Wall B | S-CB |

Guides: base footprints on `cleat-full-base`; AD/AB on Wall A; CD/CB on Wall C. Semantic ownership strings match. **No reversed corner found.**

### 6.3 Guides

All eight footprints receive four L-corner markers (8 open segments each), extents match installed rectangles, unique IDs (`cleat-full-guide-base-a` … `seam-cb`), blue `#0000ff`, `fill="none"`, separate top-level group before labels/cuts.

### 6.4 Labels and laminate ownership

BASE, WALL A–D, BA/BB/BC/BD, S-AD/S-AB/S-CD/S-CB — 13 path labels; **no SVG `<text>`** in production; only outer layers (`…-layer-02` at 3 mm) receive short IDs; intermediate layers unlabeled.

### 6.5 Production SVG ordering / colors

Top-level groups (not nested under a shared `#score`):

1. `#cleat-placement` stroke `#0000ff` fill none  
2. `#assembly-labels` stroke `#00ff00` fill none  
3. `#cut` stroke `#ff0000`  

Filename `l8-concealed-cleat-full-box-prototype-<date>.svg`; MIME `image/svg+xml;charset=utf-8`; validation passes; deterministic rebuilds; preview markup absent from production.

**LightBurn:** software correctly states colors do not force operation mode. This audit **did not** import into LightBurn; Joe must still verify Line mode for blue and green, red cut, open-path warnings, and physical mark visibility.

### 6.6 J2 single-corner protection

| Metric | Before J2.5 work in session | After full-box build + preview |
| --- | --- | --- |
| J2 SVG length | **4457** | **4457** |
| J2 FNV-1a | **`88721533`** | **`88721533`** |
| J2 red cut subgroup | 1606 / `c16a9ef5` | unchanged (same rebuild) |

J2 uses the same local `serializeConcealedCleatSvg` path; J2.5 does not rewrite generic `serializeDesignSvg`. Fixtures re-pin J2 golden after full-box tests.

### 6.7 Non-J2 production protection

Complete Designs **1069 / 0** retains pre-existing goldens (dice 1726/`51a55721`, divider 1965/`a55dda6e`, finger 2483/`a892f91c`, sliding/cabinet/coupons/faux-dovetail/custom-row covered by suite). No Finger Box joint-option leak.

### 6.8 Validation / warnings

Hard rejects: non-positive / non-finite dimensions; `Ow/Od <= 2(t+W)`; short runs below `guideMinRun`; non-positive wall/cleat lengths. No silent clamp of invalid geometry. Warnings include prototype/not-strength, visible butt seams, **final-wall clamp access**, slower than finger joints, occupancy >30% when applicable.

### 6.9 Storage / session-only

| Check | Result |
| --- | --- |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | `2` |
| localStorage before/after generate+preview | **identical** |
| `backupObject()` before/after | **identical** |
| Leak of `boxOutsideWidth/Depth`, `boxWallHeight`, template name | **none** |
| `freshState()` has box fields | **false** |

**Storage/schema conclusion:** unchanged; J2.5 remains session-only form state.

### 6.10 Preview / download isolation

Preview reads metrics/semantic only; does not parse `result.svg`; production bytes unchanged across preview; honest prototype language; shows eight cleat positions and exterior butt marks. Download continues to use production `result.svg`.

### 6.11 Fixture quality

**Strengths:** independent local intersection helper in fixtures; explicit four-corner footprint pins; 3 mm and 6 mm runs; SVG color order; outer-layer labels; J2 golden re-check; invalid gates; interior dual metrics.

**Weaknesses (Low):**

1. Full-box storage fixture checks key absence in `freshState`/`backupObject` but not a true generate before/after localStorage pair (auditor probe closed this).  
2. Final-wall descent fixture is largely ownership-string + seam edge equality, not a full body-volume test (auditor geometry closed this).  
3. README complete-suite delta text wrong (`1841 + 20` vs actual `+29` from 1832).

Fixtures are **not** mere tautologies of production helpers for the critical overlap path.

---

## 7. Reported goldens (independently rebuilt)

| Artifact | Claimed | Actual |
| --- | --- | --- |
| J2.5 default SVG | 9701 / `2316430b` | **9701 / `2316430b`** |
| J2 default SVG | 4457 / `88721533` | **4457 / `88721533`** |
| J2.5 pieces @ 3 mm | 21 | **21** |
| Eight cleat stacks | 8 | **8** (each `cleatLayers` deep) |

---

## 8. Findings by severity

### Critical / High / Medium

*None that block commit.*

### Low

1. **README arithmetic narrative** (`README.md` Built-in checks): `1841 + 20 = 1861` misstates the delta from HEAD baseline (actual **+29** Designs assertions from **1832 → 1861**). Totals themselves match live execution.  
2. **Occupancy warning wording** (`buildConcealedCleatFullBoxDesignResult`): says “clear interior floor area” while the ratio is intrusion/interior floor — slightly misleading language; production math is correct.  
3. **Full-box storage fixture** weaker than J2’s before/after pattern (auditor confirmed real isolation).  
4. **Error key strings** like `baseA cleat length` are slightly opaque for users (functional).

### Informational

- Preview is schematic top-down, not a metrically exact 3-D install drawing.  
- LightBurn Line-mode assignment, physical clamp access, glue-up, squareness, strength, and J3 readiness remain **out of scope**.  
- Not a Finger Box joint option; dedicated template only.

---

## 9. Physical-validation boundaries

This audit does **not** claim:

- LightBurn import or inferred operation modes  
- Physical guide accuracy after cut  
- Glue-up success, final-wall clamp access, squareness  
- Strength, durability, or production suitability  
- J3 readiness for Finger Box adoption  

Software correctly frames the feature as an assembly/registration prototype.

---

## 10. Remaining unverified areas

- Real four-corner physical build and clamp strategy for the last wall  
- Cumulative squareness vs single-corner J2  
- Practical usefulness vs reinforced butt joint  
- Exhaustive UI click-paths beyond fixtures  
- Every fractional (Ow, Od, H, t) near all simultaneous gates  

---

## 11. Final verdict

### **SAFE TO COMMIT**

**Rationale:** J2.5 is a bounded, session-only dedicated template implementing the design review’s outside dimensions, four-wall ownership, eight non-overlapping cleat stacks, four-L guides, green path labels on outer laminate layers only, separate blue/green/red top-level groups, honest prototype warnings, and storage isolation. Independent math confirms all six base intersections are zero-area, seam/base z-contact is plane-only, and yielding-wall descent paths are geometrically clear. Live suite is **1861 / 0** (Designs **1069 / 0**, Tray **264 / 0**); goldens **9701 / `2316430b`** (J2.5) and **4457 / `88721533`** (J2) rebuild correctly. Residual findings are documentation and fixture-strength nits, not contract breaks.

**Human committer scope recommendation:** stage only `index.html` and `README.md`. Leave design review, implementation report, this audit, LightBurn projects, and other untracked files unstaged unless intentionally published.

---

*End of adversarial implementation audit.*
