# Designs Dice Tray Underside Cover Plate — Focused Audit (2026-07-17)

**Mode:** Read-only adversarial audit (no application edits, no git writes, no staging, no LightBurn / debug / utility changes).  
**Primary app path:** `C:\Genmitsu L8 Tracker`  
**Feature under audit:** Optional Dice Tray **Underside cover plate** (session-only, off by default).  
**Primary reports (not trusted alone):**  
- `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_DESIGN_REVIEW_2026-07-17.md`  
- `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_IMPLEMENTATION_2026-07-17.md`  
**This document:** Independent verification of working-tree implementation vs baseline `c064896`.

---

## 1. Actual HEAD, baseline, working-tree state

| Check | Result |
| --- | --- |
| `git log -1 --oneline` | **`c064896` Add drawer cabinet shelf-placement guides** |
| Matches expected baseline `c064896`? | **Yes** |
| `git status -sb` | `## main...origin/main` · `M index.html` · `M README.md` · pre-existing untracked docs / LightBurn / `debug.log` / utilities |
| Staged files | **None** (`git diff --cached --name-only` empty) |
| Tracked modified (feature scope) | **`index.html`**, **`README.md` only** — no unexpected tracked file changed |
| Unrelated untracked reports / LightBurn / debug / utilities | **Untouched** by this audit (left as-found) |
| `git diff --check` | **Clean** (CRLF warnings only; no whitespace errors) |
| `git diff --stat` | `README.md` 5 ± · `index.html` 47 ± · **2 files changed, 38 insertions(+), 14 deletions(-)** |
| `git diff --numstat` | README `3 2` · index `35 12` · **totals 38 / 14** — **reconciles** with `--stat` |
| Read-only status | **Confirmed** for application sources; this audit **only created** this report under `docs/` |

**Expected new untracked implementation report:** present.  
**This audit report:** `docs/DESIGNS_DICE_TRAY_BOTTOM_COVER_FOCUSED_AUDIT_2026-07-17.md` (new).

---

## 2. Complete diff-hunk enumeration

### 2.1 `README.md` (numstat 3 / 2)

| Hunk | Classification | Content |
| --- | --- | --- |
| README feature list | **README documentation** | Underside cover plate blurb: 3 mm reveal, cosmetic only, +1 piece/thickness, first-tray verify; Finished View deferred |
| README fixture totals | **README documentation** | 1710 complete / 260 Tray-model / 918 Designs (was 1698 / 248 / 906) |
| README validation sentence | **README documentation** | Designs geometry **918**; complete suite **1710** |

### 2.2 `index.html` (numstat 35 / 12)

| # | Region | Classification | Summary |
| --- | --- | --- | --- |
| 1 | `designDefaults` | **Dice Tray defaults/UI** | `trayBottomCover: false` |
| 2 | `legacyFields` / Dice form | **Dice Tray defaults/UI** | Checkbox + help (cosmetic, flush tabs, clean surfaces, first-tray verify) |
| 3 | `normalizeDesignDraft` | **normalization** | Dice-only `bottomCover` for `true` / `'true'` / `'on'` |
| 4 | `buildTrayModel` | **tray model/component generation** + **layout calculation** + **result metrics/warnings** | Cover from `baseOutside* − 6`; append last; extend height; metrics/warnings |
| 5 | `buildTrayDesignResult` | **result metrics/warnings** | Merge `model.metrics` + `model.dimensions` into result metrics; model warnings |
| 6 | `designResultsHtml` (dice) | **result metrics/warnings** | Bottom cover + base thickness lines |
| 7 | Tray self-tests | **fixtures** | 12 new assertions; goldens retained |
| 8 | Form change handler | **form event handling** | `trayBottomCover` live re-render |

**Unexpected / unrelated production hunks:** **None.**  
**Serializers (`designSvgDocument`, `serializeTrayCompatibilitySvg`):** **unchanged** vs `c064896` (bodies identical; only call-site context differs via extra shape).  
**Finished View production path:** **unchanged** (no cover geometry injected).

---

## 3. Files and functions inspected

| Area | Items |
| --- | --- |
| App | Working-tree + `git show c064896:index.html` / README |
| Reports | Design review; implementation report; wall-to-base coupon precedent context |
| Tray | `trayTabProfile`, `designTabPositions`, `traySlotWidthMm`, `trayModelValidationErrors`, `trayRectComponent`, `buildTrayModel`, `serializeTrayCompatibilitySvg`, `designSvgDocument`, `trayFinishedViewProjection`, normalize, UI, metrics, fixtures, `trayLivePreviewDownloadFixture` |
| Isolation | Divider path; storage/backup symbols in diff; suite totals |

---

## 4. Feature contract (verified)

| Contract item | Verdict |
| --- | --- |
| UI feature | **Underside cover plate** |
| Checkbox | **Add underside cover plate (hides wall slots)** |
| Raw draft | **`trayBottomCover`** |
| Normalized | **`bottomCover`** (Dice only) |
| Component id / name / kind / role | **`tray-bottom-cover` / Tray bottom cover / bottom-cover / bottom-cover** |
| Result metric label | **Bottom cover** |
| Session-only, off by default, Dice only | **Yes** |

### Normalization

| Input | Expected | Observed |
| --- | --- | --- |
| `true`, `'true'`, `'on'` | enable | **Yes** (Dice) |
| absent / false / `'false'` / `''` / `1` / `'cover'` | disable | **Yes** (fixture) |
| whitespace / `null` / other numbers | disable | **Implied** by strict equality gate; not every case fixture-listed |
| Divider + `trayBottomCover:true` | no `bottomCover` | **Yes** |

Field remains only in session draft defaults/UI, Dice normalize, tray model/result, fixtures — **not** storage/schema/import/export/evidence/promotion/Production Settings.

---

## 5. Disabled byte identity

Baseline `c064896` pins (confirmed in committed source):

| Template | Length / hash |
| --- | --- |
| Dice Tray default | **1726 / 51a55721** |
| Divider Tray default | **1965 / a55dda6e** |

Working tree still asserts both. Disabled path adds **no** component when `bottomCover` is falsy; pre-existing component construction is unchanged. Structural identity of existing components is fixture-proven via `JSON.stringify` equality enabled vs disabled.

Also covered: absent default draft; explicit false; malformed values normalize disabled → same empty-cover path.

**Do not treat enabled golden alone as disabled proof** — disabled pins are **pre-existing baseline literals**, re-asserted.

---

## 6. Divider Tray isolation

| Check | Verdict |
| --- | --- |
| Checkbox in Divider UI | **No** (Dice-gated) |
| Normalize `bottomCover` true | **Impossible** (Dice-only assignment) |
| Cover component | **None** even with raw `trayBottomCover:true` |
| Piece count / SVG golden | **Unchanged** (1965 / a55dda6e) |
| Finished View / filename / MIME | **Unchanged** |

---

## 7. Shared tray model — existing components

Compared baseline `buildTrayModel` to working tree:

- Structural base, wall slots, walls, dividers, tab profile, slot width, clearances, outside dimensions, existing layout-row formulas: **same arguments and order** when cover is off.
- When cover is on: every pre-existing component keeps same id, name, kind/role, geometry, svg string, and relative order; cover is **appended last**.
- `materialComponentIds` still **excludes** slot rectangles; cover is included as a physical piece.
- `pieceCount` increases by **exactly one** when enabled.

**No existing-component drift** found.

**Note:** Disabled models now always populate extra metric keys (`bottomCover`, `bottomCoverWidthMm`, …) even when false. SVG production is unaffected; this is metric-surface expansion only.

---

## 8. Cover geometry source of truth

Implementation:

```text
bottomCoverInsetMm = 3
bottomCoverWidthMm = designRound(baseOutsideWidthMm - 6)
bottomCoverDepthMm = designRound(baseOutsideDepthMm - 6)
```

`baseOutsideWidthMm` / `baseOutsideDepthMm` are the **already-computed** model values (`inside + 2×thickness`). Cover code does **not** re-derive outside dimensions from tray width/depth/clearance/tabs.

Cover is a plain `trayRectComponent` rectangle: no slots, tabs, scores, labels, holes, rounded corners, or engrave geometry.

---

## 9. Independent default geometry derivation

Defaults: thickness **3**, inside **220×160**, wall height **35**, margin **10**, gap **15**.

| Quantity | Derivation | Value |
| --- | --- | --- |
| `baseOutsideWidthMm` | 220 + 2×3 | **226** |
| `baseOutsideDepthMm` | 160 + 2×3 | **166** |
| Cover width / depth | 226−6 / 166−6 | **220 / 160** |
| `wallY` | 10 + 166 + 15 | **191** |
| `existingLayoutHeightMm` | 191 + (35+3+15)×2 + 10 | **307** |
| Cover `x` | margin | **10** |
| Cover `y` | 307 − 10 + 15 | **312** |
| New layout height | 307 + 15 + 160 | **482** |
| Layout width | unchanged vs disabled | **421** (from fixtures / max formula) |

Report claims **220×160 at (10, 312)**: **confirmed by hand**, not by trusting fixtures alone.

---

## 10. Near-minimum slot coverage proof (independent)

### 10.1 Slot placement (source)

- Front slots: outer edge at **`thickness/2`** from base front (`y = t/2`).
- Back: symmetric from back edge.
- Left/right: outer edge at **`thickness/2`** from side edges (`x = t/2`).
- Slot radial depth = `slotWidthMm = thickness + fitClearance`.
- Lateral first/last tab positions: `designTabPositions` → edge inset `(L − tabWidth)/(count+1)`.

### 10.2 Lateral (along-wall) — design-review bound is correct

At worst valid finger boundary (`baseOutsideWidth = 48`, `tabWidth = 8`, count 4):

- First-slot lateral inset = **8 mm**
- After 3 mm cover inset: **5 mm** remaining lateral margin  
- Near-min 42×48 finger: same **8 mm → 5 mm**  
- Near-min tab-slot: larger lateral margins  

**Lateral** concealment under a 3 mm inset is safe at the model’s own validation boundary.

### 10.3 Radial (toward outer perimeter) — fails for default and near-min thickness

Cover occupies base-local `[3, W−3] × [3, D−3]`.

For **material thickness 3 mm** (default and near-min fixtures):

| Edge | Outer slot edge from perimeter | Required for full cover | Remaining margin after 3 mm inset |
| --- | --- | --- | --- |
| Front / back / left / right | **t/2 = 1.5 mm** | ≥ 3 mm | **−1.5 mm** (1.5 mm of every wall slot lies **outside** the plate) |

Default finger: **all 16** wall slots partially outside the cover.  
Near-min finger 42×48: **all 16** partially outside.  
Near-min tab-slot: **all 8** partially outside.

Full containment requires **`t/2 ≥ 3` ⇒ `t ≥ 6` mm**. Independently verified: at `t = 6` default plan, full containment passes; at `t = 3` it fails.

**Therefore the 3 mm reveal does not conceal all wall-slot openings at valid near-minimum (or default) 3 mm trays.** The design review’s “8 mm floor → 3 mm is safe” proof only analyzed **lateral** tab insets, not the radial slot band at `t/2`. That omission is material.

### 10.4 Fixture overstatement

Assertion name: *“Near-minimum finger and tab-slot trays keep the three millimeter cover reveal inside the model slot margin”*.  
Assertion body: only checks `plate = base − 6` and positive size — **does not** compare any slot rectangle to the cover.  
**Name overstates what the body proves** (partially circular / irrelevant to concealment).

---

## 11. Component and material order

When enabled:

- Exactly **one** new component, **last** in `components`
- **Last** in `materialComponentIds`
- `pieceCount` **+1**
- Slots remain non-physical (excluded from material ids)
- Metadata matches approved names

---

## 12. Layout derivation

Constants: `marginMm = 10`, `gapMm = 15` (confirmed).

| Claim | Verdict |
| --- | --- |
| `x = marginMm` | **Yes** |
| `y = oldLayoutHeight − margin + gap` | **Yes** (312 for default) |
| Height `+= gap + coverDepth` | **Yes** |
| Width unchanged | **Yes** (fixture + formula) |
| Non-overlap with existing components | **Yes** (fixture + trailing-row construction) |
| Finite / inside bounds | **Yes** |
| No nesting optimization | **Yes** |

Semantic note: last wall bottom sits **one gap** above the old content bottom (`oldH − margin`); cover starts **one additional gap** below that content bottom. Net wall-bottom → cover-top = **2×gap**, which matches stacking two “row + trailing gap” steps. Separation uses the existing **15 mm** gap constant; not a layout collision defect.

---

## 13. Anonymous SVG contract

`designSvgDocument` / `serializeTrayCompatibilitySvg` **byte-identical** to baseline.

Enabled default:

- Same SVG wrapper and single anonymous red group
- Existing shapes unchanged and in same order
- **One final plain red `<rect>`**
- No new named group, id, title, metadata, score group, text, or layer

Claimed golden **1778 / ec9b12bc**: supported by structural asserts (anonymous red-only, rect last after paths) **plus** size/hash pin — not hash-only circularity.

Disabled goldens remain baseline pins.

---

## 14. Result metrics, warnings, wording

Metrics present when built: `bottomCover`, inset, width/depth, thickness, total visible base thickness, component id, added piece count.

Result panel: enabled dimensions +1 piece; base thickness normal vs doubled visible.

Warnings communicate: cosmetic only; does not strengthen/replace joint; flush tabs; clean soot/char/fibers/debris; added material/time; first physical tray verify concealment/alignment/flatness.

**No** reinforcement / stronger joint / captured / locking / guaranteed flushness / proven concealment/flatness/glue claims in user-facing cover copy.

**Caveat:** checkbox still says **“hides wall slots”** while radial geometry leaves a 1.5 mm strip of every wall slot in the reveal at 3 mm material — wording overstates capability (tied to Major finding F1).

Physical honesty: software flushness ≠ physical flushness; disclosed. No physical verification claimed by this audit.

---

## 15. Finished View

- `trayFinishedViewProjection` gains **no** production cover geometry
- Dice Finished View SVG does not include `tray-bottom-cover`
- Production SVG independent
- Documentation correctly defers representation

---

## 16. Preview, download, filename, MIME

Fixture drives real `trayLivePreviewDownloadFixture`: preview === download === result SVG; filename `l8-dice-tray-${today()}.svg`; MIME `image/svg+xml;charset=utf-8`; no cover-specific export path.

Storage/backup isolation: real generate + preview/download with before/after `localStorage` / `backupObject` equality.

---

## 17. Protected boundaries

Diff does **not** modify: `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState`, `loadState`, `persist`, `backupObject`, `replaceData`, `mergeData`, import/export, Production Settings, evidence, promotion, Library, Inventory, Projects, Pricing, Test Grid, Material Test, QR Stand, Hanging Sign, Finger Box, Sliding-Lid, Drawer Cabinet, Joint Fit Coupon, unrelated Finished Views (beyond Dice metrics UI).

---

## 18. Fixture count reconciliation

| Item | Value |
| --- | --- |
| Baseline Tray-model | **248** |
| New static `add(` (cover block) | **12** (each one assertion; no multi-assert loops in that block) |
| Current Tray-model | **260** = 248 + 12 |
| Baseline Designs geometry | **906** |
| Current Designs (includes nested Tray) | **918** = 906 + 12 |
| Baseline complete suite | **1698** |
| Current complete suite | **1710** = 1698 + 12 |

**Independent Edge (`channel=msedge`) `file://…?selftest=all`:**

| Group | Result |
| --- | --- |
| Designs geometry | **918 / 0** |
| Tray-model (direct `runTrayModelFixtures`) | **260 / 0** |
| Production settings | 66 / 0 |
| Evidence promotion | 58 / 0 |
| Designs production | 118 / 0 |
| Test Grid machine identity | 18 / 0 |
| Storage recovery | 8 / 0 |
| **Complete suite (reconciled; single final Material-browser total)** | **1710 / 0** |

`git diff --check` clean. HTML loaded successfully in headless Edge.

### Assertion quality

| Class | Examples |
| --- | --- |
| **Independent** | normalize true/on; Divider isolation; baseline disabled goldens; component JSON identity; default 220×160@(10,312); layout width/height delta; anonymous SVG structure; preview/download/MIME; storage isolation; Finished View independence |
| **Partially circular** | Enabled 1778/ec9b12bc (supported by structural checks) |
| **Overstated / weak** | “Near-minimum … reveal inside the model slot margin” — body does not test slots vs cover |
| **Gap** | No independent radial containment assert; incomplete normalize matrix (null/whitespace) |

---

## 19. Runtime / browser checks performed

| Check | Result |
| --- | --- |
| `git diff --check` | Pass |
| Headless Microsoft Edge `file://` + `?selftest=all` | Pass — **1710 / 0** reconciled |
| Tray-model 260 / Designs 918 | **Reproduced** |
| Direct form click-through | Not separately scripted; suite covers preview/download |

---

## 20. Documentation accuracy

| Claim | Assessment |
| --- | --- |
| Terminology, cosmetic-only, +thickness, FV deferral | **Accurate** |
| Default 220×160 @ (10,312) | **Accurate** (independent derivation) |
| Legacy + enabled goldens; fixture totals | **Accurate** (runtime-confirmed) |
| “3 mm reveal still covers all wall slots” / design-review 8 mm safety story | **Inaccurate for full 2D containment** at t&lt;6 mm; lateral-only proof was over-generalized |
| Implementation “near-minimum coverage” validation | **Overstated** relative to fixture body and radial geometry |
| No physical-cut claim | **Accurate** |
| Changed-file scope index + README | **Accurate** |

---

## 21. Findings table

| ID | Severity | Location | Consequence | Recommendation | Fixture coverage |
| --- | --- | --- | --- | --- | --- |
| **F1** | **Major** | Cover inset 3 mm vs wall slots at `t/2`; default/near-min t=3 | At typical 3 mm material, ~1.5 mm of **every** wall slot (and tab end) lies in the reveal — cover does **not** fully conceal slots; UI “hides wall slots” overstates | Fix geometry and/or inset policy (e.g. inset ≤ max(0, t/2 − ε), or 0 inset, or thickness-aware inset) **and** retarget wording/fixtures to true containment; re-derive enabled golden | **Missing** — named near-min fixture does not test slots |
| **F2** | **Major** (docs/fixtures) | Design review §6.2; implementation report coverage claim; fixture name at ~L3560 | False confidence that 3 mm is “proven safe” for full slot coverage | Correct the lateral-only bound disclosure; add independent containment asserts for both profiles at near-min **and** default thickness | Weak / overstated |
| **F3** | **Minor** | Normalize fixtures | Null / whitespace not explicitly listed | Optional extra cases | Partial |
| **F4** | **Informational** | Metrics always include cover dimension fields when disabled | Harmless metric surface growth; SVG stable | Optional: omit cover metrics when disabled | N/A |
| **F5** | **Informational** | Physical flush / real concealment | Software cannot prove shop result | Keep first-tray verify (already present) | Warning text |
| — | **Blocker** | — | — | **None found** (disabled bytes stable; no storage corruption; serializers intact) | — |

**Boundary scorecard:**

1. Disabled Dice / Divider byte identity — **Pass**  
2. Divider cannot enable cover — **Pass**  
3. Existing tray components unchanged — **Pass**  
4. One new final component + one final anonymous rect — **Pass**  
5. Cover from existing base outside dims — **Pass**  
6. 3 mm reveal covers all slots at near-min — **Fail (Major F1)**  
7. Layout finite, non-overlap, gap, width stable — **Pass**  
8. Piece count +1 — **Pass**  
9. Serializer / FV production contract — **Pass**  
10. Fixture totals — **Pass** (counts); concealment claim fixtures — **Fail quality**  
11. No structural-strength wording — **Pass** (except overstated “hides” given F1)

---

## 22. Explicit unverified physical areas

- Real glue adhesion, char cleanup, clamp flatness, proud tabs  
- LightBurn import of the extra rect  
- Headed manual form walkthrough  
- Shop appearance of the 3 mm reveal with partially exposed slot bands  

---

## 23. Conclusion

Additive implementation is clean on **production isolation** (disabled goldens, Divider isolation, anonymous SVG, no serializer change, storage untouched, suite **1710 / 0**). Layout and default dimensions match independent derivation. However, the core concealment claim fails under independent geometry: with the production default **3 mm** thickness, wall slots sit at **`t/2 = 1.5 mm`** from the base perimeter, so a **3 mm** inset leaves a **1.5 mm** strip of every wall slot outside the plate. The design-review “8 mm floor” only secures **lateral** tab margins; fixtures do not prove radial containment. That is a **Major** correctness defect relative to the feature’s stated purpose and the audit’s highest-risk boundary #6.

### NOT SAFE TO COMMIT
