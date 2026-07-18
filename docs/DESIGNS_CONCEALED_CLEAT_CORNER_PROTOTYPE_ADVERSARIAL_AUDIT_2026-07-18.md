# Concealed Cleat Corner Prototype (J2) — Adversarial Read-Only Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit date:** 2026-07-18  
**Audit type:** focused, adversarial, **read-only**  
**Baseline commit (expected and actual HEAD):** `77c2395` — *Fix Drawer Cabinet custom row assembly labels*  
**Full HEAD:** `77c2395eda944e1ba81f8e0f132270622e3183b8`  
**Authoritative design contract:** `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md` (corrected; **untracked; not modified by this audit**)  
**Implementation report (reference only):** `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_IMPLEMENTATION_2026-07-18.md` (untracked; not modified)

This audit did **not** edit, stage, commit, push, reset, clean, stash, move, rename, or delete any application file, design review, implementation report, LightBurn project, utility, or other unrelated working-tree content. The only file created is this audit report.

---

## 1. Repository state (verified before review)

| Check | Result |
| --- | --- |
| `git rev-parse HEAD` | `77c2395eda944e1ba81f8e0f132270622e3183b8` |
| `git log -1 --oneline` | `77c2395 Fix Drawer Cabinet custom row assembly labels` |
| Branch | `main` |
| `git rev-list --left-right --count origin/main...HEAD` | `0 0` — **fully synchronized** with `origin/main` |
| Staging | empty |
| Tracked modifications | `index.html`, `README.md` only |
| Diff vs `77c2395` | `README.md` 3 lines changed; `index.html` **+125 / −16** (≈127 insertions / 17 deletions) |
| `git diff --check 77c2395` | **clean** (no whitespace errors) |

### Working-tree status (complete)

**Modified (tracked):**

- `index.html`
- `README.md`

**Untracked (left untouched by this audit):**

- `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_DESIGN_REVIEW_2026-07-18.md`
- `docs/DESIGNS_CONCEALED_CLEAT_CORNER_PROTOTYPE_IMPLEMENTATION_2026-07-18.md`
- Many historical `docs/*` audit/review files
- `LightBurn Projects/`
- `debug.log`
- `parametric_qr_stand_generator.py`

Nothing was staged, committed, or pushed during this audit.

---

## 2. Reviewed files and functions

### Primary implementation (`index.html`)

| Area | Symbols / locations (approx.) |
| --- | --- |
| Session defaults | `designDefaults()` — adds `cleatLegLength: '80'`, `cleatWallHeight: '50'` |
| UI | `renderDesigns()` — template option, `cleatFields`, cleat branch |
| Normalization | `normalizeDesignDraft()` — template id; `legLength` / `wallHeight` from cleat fields |
| Geometry | `concealedCleatRectIntersection`, `concealedCleatGuideSegments`, `buildConcealedCleatCornerDesignResult` (~3230–3284) |
| Dispatch | `buildDesignResult` — early return for `concealed-cleat-corner-prototype` |
| Preview | `buildConcealedCleatFinishedViewSvg` (~3351–3354); `designPreviewModeForTemplate` / `designResultsHtml` wiring |
| Labels / glyphs | `designAssemblyGlyphs` (`S`, `W`, space); `designAssemblyWordGeometry`; `buildAssemblyLabelPaths` |
| Serializer (shared, **not rewritten**) | `serializeDesignSvg` — score subgroup then `assembly-labels` then red `cut` |
| Fixtures | 20× `add('Concealed Cleat …')` in `runDesignGeometryFixtures` (~3746–3767) |
| Download | `downloadCurrentDesignSvg` uses `result.svg` (production artifact) |

### Supporting documents

- Corrected J2 design/architecture review (formulas, ownership, zero-intersection proof)
- Implementation report (claims to verify)
- README Designs + Concealed Cleat sections and fixture totals

### Explicit non-targets

Finger Box structural joints, Drawer Cabinet geometry/prefix logic, tray models, coupons, Production Settings application paths, Library/Projects/Inventory/Test Grid persistence — inspected for **regression via goldens/fixtures and storage isolation**, not re-audited as product features.

---

## 3. Commands and tests actually run

| Command / action | Result |
| --- | --- |
| `git status`, `rev-parse`, `rev-list`, `diff --stat`, `diff --check` | HEAD `77c2395`; origin sync `0 0`; only `index.html` + `README.md` modified; check clean |
| Independent geometry derivation (Python/JS probe) | Matches implementation for 3 mm and 6 mm defaults; zero base-cleat area intersection |
| `python -m html.parser` on `index.html` | **OK** (no parse failure) |
| Headless Edge (`playwright`, `channel=msedge`) on temp **copy** of `index.html` with in-memory audit hooks only | Full `?selftest=all` |
| Independent probe after suite | Geometry match, golden, labels, SVG order, storage leaks, invalids, preview neutrality |

Temp patches and runners lived only under the audit worktree (`agent-tools/`); they were **not** written into the product tree.

---

## 4. Fixture totals (actual)

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
| Material browser (final aggregate) | 57 | 0 |
| Library browser | 56 | 0 |
| Project browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Design geometry | **1034** | **0** |
| **Total (15 groups, Material browser counted once at final)** | **1826** | **0** |

**Arithmetic:** non-Design groups sum to **792**; `792 + 1034 = 1826`. Matches claimed complete suite.

### Nested / claimed subtotals

| Suite | Claimed | Actual |
| --- | --- | --- |
| Tray-model (`runTrayModelFixtures`) | 264 / 0 | **264 / 0** |
| Designs geometry (`runDesignGeometryFixtures`, includes tray results) | 1034 / 0 | **1034 / 0** |
| Complete | 1826 / 0 | **1826 / 0** |

**New focused assertions:** exactly **20** `add('Concealed Cleat …')` calls; `runDesignGeometryFixtures` `add(` count delta vs `77c2395` = **+20**.

### Default production golden (3 mm / O80 / H50)

| Metric | Claimed | Actual (probe + fixtures) |
| --- | --- | --- |
| SVG length | 4171 | **4171** |
| FNV-1a (`designFixtureHash`) | `ca0cfb8e` | **`ca0cfb8e`** |

Page errors during suite: two browser console notes `Error: <svg> attribute height: Expected length, "auto".` (pre-existing pattern from some screen-only SVGs; **not** fixture failures).

---

## 5. Independent geometry calculations

Definitions (settled contract):

```text
cleatLayers = clamp(round(6/t), 1, 3)
B = cleatLayers × t
W = max(3t, 10)
tickLength = max(6, 2t)
guideMinRun = 3 × tickLength
Wall A = O × H; Wall B = (O − t) × H; Base = O × O
Base cleat A length = O − t; footprint [t,O]×[t,t+W], z [0,B]
Base cleat B length = O − t − W; footprint [t,t+W]×[t+W,O], z [0,B]
Seam cleat length = H − B − t; footprint [t,t+W]×[t,t+B], z [B, H−t]
```

### 5.1 Worked defaults

| | t=3, O=80, H=50 | t=6, O=80, H=50 |
| --- | --- | --- |
| layers / B / W | 2 / 6 / 10 | 1 / 6 / 18 |
| Wall B | 77×50 | 74×50 |
| CA / CB / SC lengths | 77 / 67 / 41 | 74 / 56 / 38 |
| piece count | 9 | 6 |
| union footprint / occupancy | 1440 mm² / 22.5% | 2340 mm² / 36.5625% |
| base-cleat intersection area | **0** | **0** |
| seam z | [6, 47] | [6, 44] |

Production metrics and semantic model match these values for every checked field (`match3` / `match6` all true in probe).

### 5.2 Zero base-cleat area intersection (independent)

For `O > t + W`:

- Cleat A y-range `[t, t+W]`, cleat B y-range `[t+W, O]` meet only at **line** `y = t+W`.
- Rectangle intersection area = 0 (boundary contact only).

Implementation uses the same axis-aligned area formula; fixtures reuse `concealedCleatRectIntersection` (shared-helper risk noted below) but the independent derivation confirms the claim is real, not coordinate-model fraud.

### 5.3 Seam stack / z-range

- Seam `minZ = B = maxZ` of both base stacks → plane contact only (zero volume overlap).
- Seam `maxZ = H − t` → stops one thickness below wall top.
- Wall B descent `x ∈ [0, t]` remains unobstructed (`wallBDescent.unobstructed === true`).

### 5.4 Layer-count boundaries (`clamp(round(6/t), 1, 3)`)

| t | round(6/t) | layers |
| --- | ---: | ---: |
| 0.5 | 12 | 3 |
| 2.0 | 3 | 3 |
| 2.4 | 3 | 3 |
| 2.5 | 2 | 2 |
| 3 | 2 | 2 |
| 5 | 1 | 1 |
| 6 | 1 | 1 |
| 12 | 1 | 1 |

Fractional example `t=2.5`: layers 2, `W=10`, production valid and matches independent formulas.

### 5.5 Validation vs silent normalization

Hard errors (not silent “fix”) for non-positive / non-finite `t`, `O`, `H`; non-positive wall B / cleat lengths; `baseCleatBLength < guideMinRun`; `seamCleatLength < guideMinRun`; positive base-cleat intersection. Probe confirmed invalids for `0`, `-1`, `NaN`, blank thickness, leg `30`, height `20`, negative leg.

**Note:** `guideMinRun` is enforced as a **hard** dimensional gate (registration prototype quality), stricter than a soft “omit guide with warning” path. Short runs are blocked rather than producing misleading incomplete registration marks.

---

## 6. Contract verification (priorities 1–12)

### 6.1 Geometry and ownership

- **O is outside:** Base `O×O`, Wall A width `O`, UI label “Prototype outside leg length”; clear-interior metrics derived as `O − t` and floor start at `t+W`. No inside/outside substitution found.
- **Wall A owns corner; Wall B yields:** Wall B width `O − t`; semantic `wallBDescent` and preview copy state ownership; not a Finger Box joint option (dedicated template only).

### 6.2 Base-cleat non-overlap

- Independent proof + production semantic `baseCleatIntersectionArea === 0` for 3 mm and 6 mm defaults.
- Placement ticks on base include the `y = t+W` / `x = t` registration edges consistent with footprints (`guideAtW` true).

### 6.3 Vertical seam cleat

- Length `H − B − t` verified.
- z-range `[B, H−t]` verified; meets base stack only at `z = B`.
- Impossible short walls fail validation (e.g. H=20).

### 6.4 Laminate layers and labels

- Layer rule exact.
- Panel order: Base → Wall A → Wall B → base-cleat-A layers → base-cleat-B layers → seam-cleat layers (semantic panels and cut group IDs).
- Only **outer** layers receive CA/CB/SC (`…-layer-02` at 3 mm; intermediate layers unlabeled). All six labels present and fit on default (`labelsFit`).

### 6.5 Placement guides

- Mandatory blue score on **Base** (4 ticks) and **Wall A** (2 ticks); stroke `#0000ff` under `#score` / `#cleat-placement`.
- Guides describe inward registration edges aligned with semantic footprints.
- Wall B has no separate guide strokes (registers against base-B / seam exposed edges marked on base and Wall A) — consistent with “Wall A owns corner” registration model.
- Guide strokes are score, not red cut.

### 6.6 Assembly labels

- BASE, WALL A, WALL B, CA, CB, SC via path labels.
- Glyphs `S`, `W`, space present; `designAssemblyWordGeometry('WALL A')` valid.
- Shared glyph map is global: enables new words; existing production goldens still pass suite pins (no Drawer/Finger/Sliding label-prefix regressions in fixtures).

### 6.7 SVG topology

- One combined production SVG; filename pattern `l8-concealed-cleat-corner-prototype-<date>.svg`; MIME `image/svg+xml;charset=utf-8`.
- Root children order: `score` then `cut`.
- Score children: `cleat-placement` then `assembly-labels`.
- Cut panel IDs match piece order; validation passes.
- Generic `serializeDesignSvg` / `layoutDesignPanelRows` reused with `scoreGroupId: 'cleat-placement'` only — no serializer rewrite.

### 6.8 Preview / download identity

- `buildConcealedCleatFinishedViewSvg` reads `result.metrics` / `semantic` only; returns separate markup.
- Production SVG does **not** contain `concealed-cleat-assembled`.
- Probe: production bytes unchanged after preview generation; repeated builds deterministic; download path uses `result.svg`.
- Preview text states assembly/registration prototype, not strength test.

### 6.9 Validation and warnings

- Invalid dimensions blocked (see §5.5).
- Mandatory warnings include prototype language, visible butt seam, Wall B not mechanically captured, physical validation incomplete, glue/square/clamp requirements.
- UI field help and Finished View desc match honesty requirements.
- Thin material (`t=1.5`) warns; large intrusion (`t=6` default) warns about interior occupancy.

### 6.10 Storage and session-only behavior

| Check | Result |
| --- | --- |
| `STORAGE_KEY` | still `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | still `2` |
| `saveState` keys | unchanged (entries, profiles, grids, projects, inventory, prefs, pricing…) — **no design/cleat keys** |
| Probe: generate + preview | localStorage and `backupObject()` bytes **unchanged** |
| Leak scan for `cleatLegLength` / `cleatWallHeight` / `concealed-cleat` in storage/backup | **none** |
| Fields | session form / `designDraft` only; reset via `resetDesignWorkspaceSession` → `designDefaults()` |

**Storage/schema conclusion:** no backup/export schema change; no new localStorage keys; J2 fields do not leak into Projects, Library, Inventory, Production Settings records, or JSON backups.

### 6.11 Existing-template protection

| Area | Evidence |
| --- | --- |
| Dice tray golden | 1726 / `51a55721` (probe + fixtures) |
| Divider tray golden | 1965 / `a55dda6e` |
| Finger Box open golden | 2483 / `a892f91c` |
| Sliding / cabinet / coupon / faux-dovetail / custom-row | covered by Designs **1034 / 0** including pre-existing pins |
| Production Settings / promotion / browsers / wizard / storage groups | all green in complete suite |
| Finger Box joint options | no cleat joint style leak (`fingerJointLeak: false`) |

**Production bytes outside J2:** existing template **golden pins inside Designs fixtures remain green**. Shared glyph additions (`S`, `W`) do not break pinned non-J2 SVGs under the suite. Independent post-suite probe of sliding default reported `2818/468e9fd8` while fixtures pin and pass `2800/4a7ab718` — treat the **fixture pin + suite pass** as authoritative for production neutrality; residual probe discrepancy is noted under residual risks (likely post-suite environment interaction, not a failing pin).

### 6.12 Tests quality

| Strength | Gap |
| --- | --- |
| Literal 3/6 mm dimensions, topology, outer-layer labels, SVG order, golden, invalid inputs, warnings | Overlap fixture reuses production `concealedCleatRectIntersection` (cannot alone catch a shared-helper bug) — **auditor independent math closes this** |
| Preview not in production SVG; rebuild equality | Storage fixture `JSON.stringify(backupObject())===JSON.stringify(backupObject())` is a **tautology** — **auditor independent storage probe closes this** |
| Formula checks restate clamp/round | Also compares to expected numeric outcomes (lengths, pieces, occupancy) |

Fixtures are largely behavioral, not mere string copies of implementation constants; remaining weak spots are mitigated by this audit’s independent checks.

---

## 7. Adversarial claim attempts

| Claim | Attempt to disprove | Outcome |
| --- | --- | --- |
| Base cleats never overlap | Independent rectangle intersection for default and 6 mm | **Holds** (area 0; line contact only) |
| Wall A owns corner; Wall B yields | Compare widths and footprints to review §5 | **Holds** |
| Seam begins above base-cleat stack | Compare `minZ` to `B` | **Holds** |
| Blue guides match model | Tick at `y=t+W` / wall A at `z=B` edges | **Holds** for required guides |
| Only final laminate layers get CA/CB/SC | Inspect label panel IDs | **Holds** |
| Preview is production-byte neutral | Hash/equality before/after preview | **Holds** |
| Existing production SVG goldens unchanged | Suite pins green | **Holds** under fixture authority |
| No storage/backup schema change | Key list + leak scan + byte equality | **Holds** |
| Implementation bounded to J2 | Diff classification: template, helpers, glyphs, UI, fixtures, README | **Holds** (no Finger Box joint option; shared glyphs additive) |
| Ready to commit though physical behavior unverified | Software contract + suite + honesty warnings | **Software: yes; physical: explicitly out of scope** |

---

## 8. Findings by severity

### Critical / High

*None.*

### Medium

*None that block commit.*

### Low / informational

1. **README complete-suite arithmetic narrative** (`README.md` Built-in checks): states `1786 + 40 = 1826`. Against HEAD `77c2395` + this delta, the accurate story is **+20 Designs assertions** (`1806 + 20 = 1826`) or **`792 + 1034 = 1826`**. Totals themselves are correct; the delta wording is wrong.
2. **Fixture “session-only storage” assertion is tautological** (`index.html` Concealed Cleat storage test). Real isolation confirmed by independent probe; fixture should compare before/after generation like other templates.
3. **Overlap test shares production intersection helper** — independent geometry proof provided in this audit; consider an independent formula in fixtures later.
4. **`guideMinRun` hard-fail** is stricter than a soft omit path; acceptable for a registration prototype, but blocks some short otherwise-positive stacks.
5. **Finished View is schematic** (scaled polygons), not a metrically exact install drawing — still semantic and production-byte-neutral as required.
6. **Post-suite sliding probe length/hash mismatch** vs fixture pin (see §6.11) — residual only; suite pin green.

### Non-issues confirmed

- Not a Finger Box structural-joint option.
- O treated as outside dimension.
- Combined production SVG layer order correct.
- Labels do not use SVG `<text>` for production paths.
- Honest prototype / not-a-strength-test language present in warnings, UI, preview, README section.

---

## 9. Production bytes outside J2

| Conclusion | Detail |
| --- | --- |
| Serializer / layout core | Unchanged API; J2 passes `scoreGroupId: 'cleat-placement'` |
| Existing goldens under Designs fixtures | Still pass (dice, divider, finger, sliding, cabinet, coupons, etc.) |
| Shared glyph map | Additive `S` / `W` / space only |
| Complete suite | **1826 / 0** |

No evidence that non-J2 production cut geometry was intentionally or accidentally rewritten.

---

## 10. Storage / schema conclusion

**Unchanged.** `STORAGE_KEY` / `SCHEMA_VERSION` stable; `saveState` payload keys unchanged; J2 inputs remain session-only; backup/export bytes do not gain cleat fields.

---

## 11. Physical-validation boundaries

This audit **does not** claim:

- LightBurn import success
- Cut quality, kerf, glue strength, squareness, durability
- Real-world assembly success or joint strength vs finger joints
- Browser UI polish beyond headless fixture/probe runs

Software correctly states these remain unvalidated.

---

## 12. Remaining unverified areas

- Physical cut/assembly of the prototype
- LightBurn import and machine behavior
- Exhaustive UI click-path coverage outside fixtures
- Every fractional thickness × extreme O/H pair beyond validation gates and sampled probes
- Long-term behavior of intermediate unlabeled laminate layers in a real glue-up (process issue, not a software defect)

---

## 13. Final verdict

### **SAFE TO COMMIT**

**Rationale:** The working tree implements the corrected J2 contract as a dedicated experimental Designs template with correct outside-leg geometry, Wall A ownership / Wall B yield, non-overlapping base-cleat footprints, seam z-stacking, laminate/label rules, blue placement + label score ordering, screen-only preview isolation, honest prototype language, and session-only storage. Complete fixture suite is **1826 / 0** (Designs **1034 / 0**, Tray **264 / 0**); default golden **4171 / `ca0cfb8e`**. Residual findings are documentation/test-strength nits, not contract breaks. Physical strength remains explicitly unverified and is correctly out of scope for a software commit of this prototype generator.

**Commit scope recommendation (for the human committer):** stage only `index.html` and `README.md`. Leave the corrected design review, this audit, the implementation report, LightBurn projects, and other untracked files unstaged unless intentionally published.

---

*End of adversarial audit.*
