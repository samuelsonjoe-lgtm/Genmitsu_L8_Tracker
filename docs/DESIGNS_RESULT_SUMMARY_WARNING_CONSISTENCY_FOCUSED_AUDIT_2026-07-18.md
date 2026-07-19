# Designs Result Summary and Warning Consistency — Focused Adversarial Audit

**Date:** 2026-07-18 (audit executed 2026-07-19 local)  
**Auditor:** Grok (read-only)  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `fea4b90` — *Add SVG scale safety guidance*  
**Full HEAD hash:** `fea4b90105a85dd60ede79e57853bd266a648c3f`  
**Authoritative documents read:**  
- `docs/DESIGNS_RESULT_SUMMARY_WARNING_CONSISTENCY_REVIEW_2026-07-18.md`  
- `docs/DESIGNS_RESULT_SUMMARY_WARNING_CONSISTENCY_IMPLEMENTATION_2026-07-18.md`  

**Primary question:** Is the Designs Result Summary and Warning Consistency implementation correct, honest, accessible, complete across registered templates, and safe to commit without changing production geometry, SVG bytes, downloads, Finished Views, storage, schema, or unrelated workflows?

---

## Verdict

**SAFE TO COMMIT WITH NON-BLOCKING NOTES**

- Another Claude review: **unnecessary** for commit (no production serialization, storage/schema, Finished View helper, or multi-output download risk found).  
- Claude Opus 4.8: **unnecessary**.  
- Fable 5: **unnecessary**.  
- Physical cutting: **not required** for this presentation-only commit (production bytes match baseline `fea4b90`).

---

## 1. Working-tree scope

### Synchronization

| Item | Value |
|------|--------|
| HEAD | `fea4b90105a85dd60ede79e57853bd266a648c3f` |
| Branch | `main` |
| `origin/main...HEAD` | `0 ahead / 0 behind` |
| Staging area | Empty (nothing staged) |
| Commit / push | None performed by this audit |

### Initial working-tree state (preserved)

**Tracked modifications only:**

- `index.html`
- `README.md`

**Untracked (preserved, not modified by this audit):**

- `LightBurn Projects/`
- `debug.log`
- `parametric_qr_stand_generator.py`
- Historical and phase docs under `docs/` (including the implementation and review docs for this phase)
- This focused audit report (deliverable only)

### Diffstat

```
 README.md  |  6 +++---
 index.html | 78 ++++++++++++++++++++++++++++++++++++++++--------------------
 2 files changed, 59 insertions(+), 25 deletions(-)
```

`git diff --numstat HEAD`: `33 3 README.md`, `56 22 index.html`.

### Scope assessment

| Check | Result | Severity |
|-------|--------|----------|
| Only intentional product tracked files changed | Yes (`index.html`, `README.md`) | — |
| Implementation report documentation-only | Yes (untracked) | — |
| Unrelated refactor / formatting sweep | No | — |
| Production serializers / storage / FV helpers / download functions unexpectedly changed | No — presentation helpers + `designResultsHtml` + legacy metrics fields + fixtures only | — |
| Untracked content preserved | Yes | — |

**Scope creep:** none.  
**NOTE:** `buildLegacyDesignResult` metrics now carry `materialThickness`, `finishedWidth`, `finishedHeight`, `fitClearance` for presentation only; SVG path unchanged.

---

## 2. Files and functions reviewed

### Touched / inspected

- `designDimensionsText`, `designOperationsText`, `designAssemblyLabelsText`, `designWarningPriority`, `designOrderedWarnings`
- `buildLegacyDesignResult` (metrics enrichment only)
- `designResultsHtml` (grouping, Full-Box assembly branch, Operations, labels, shell/interior size labels, physical-piece wording, status region)
- New Designs fixtures (six assertions)
- README Built-in checks / Designs summary paragraphs
- Protected boundaries: `STORAGE_KEY`, `SCHEMA_VERSION`, `backupObject`, production builders, Finished View SVG builders, download paths (by diff + runtime isolation)

### Diff summary of product behavior

1. Presentation helpers for dimensions, operations, assembly-label wording, warning sort.  
2. Dead Full-Box assembly branch removed from the metrics ternary; real assembly ternary gains Full-Box instructions.  
3. Result panel: `Result summary` / conditional `Messages` / conditional `Assembly`; `role="status"` + `aria-live="polite"`.  
4. Shell vs generic Production SVG size labeling for multi-output cabinets.  
5. QR / Hanging flat summaries; unified “Physical pieces”; Operations metric.  
6. Six behavioral fixtures; README counts 1086 / 1878.

---

## 3. Full-Box dead-code repair

| Check | Finding |
|-------|---------|
| Unreachable duplicate `fullCleat` branch removed from metrics ternary | **Yes** (diff removes the metrics-chain HTML assembly string) |
| No second logically unreachable `fullCleat` metrics branch remains | **Confirmed** |
| Real rendered assembly path has explicit `fullCleat` branch | **Yes** (assembly ternary) |
| Full-Box no longer falls into generic finger-box assembly-label fallback | **Yes** |
| Rendered Full-Box assembly does not mention “finger-box” | **Yes** (`fingerBoxWord: false`) |
| Single-corner assembly note unchanged in substance | **Yes** (corner sequence still laminate → base A/B → seam on Wall A → square → Wall B) |
| Ternary precedence / right-associativity defect | **No remaining defect** on the assembly path |

### Independent rendered Full-Box assembly content

Headless Edge render of `designResultsHtml` for default `concealed-cleat-full-box-prototype`:

| Required theme | Present |
|----------------|---------|
| Lamination of multi-layer cleats | Yes |
| Base stacks via blue guides; BA/BB/BC/BD | Yes |
| Seam-cleat placement (S-AD/S-AB/S-CD/S-CB) | Yes |
| Wall installation sequence (A/C then D/B) | Yes |
| Square check | Yes |
| Clamping | Yes |
| Curing | Yes |
| Assembly/registration prototype boundary | Yes |
| No strength / load-capacity guarantee | Yes (“not a strength or load-capacity test”) |

**Does not invent:** proven durability, proven load capacity, permanent production readiness, hidden hardware, or glue performance guarantees.

---

## 4. Full-Box presentation regression protection

Default Full-Box result summary still exposes outside box, internal box, cleat construction, base/seam stack counts, clear central floor, interior intrusion/occupancy, approximate clear volume, physical pieces, assembly labels, Production SVG size, Operations, scale-safety notice (Messages), and Finished View selector when applicable. No silent loss of those metrics observed. Operations and Production SVG size appear once each.

---

## 5. Summary grouping architecture

| Check | Finding |
|-------|---------|
| Result summary for valid results | Yes (`h4` Result summary) |
| Messages when warnings and/or scale guidance | Yes (valid results always get exact-scale notice → Messages present) |
| Assembly only when assembly text exists | Yes (omitted for QR / Hanging / invalid without assembly) |
| Empty groups omitted | Assembly omitted when empty; Messages omitted only when neither errors/warnings nor scale notice |
| Heading levels | `h4` under Designs context — acceptable |
| Duplicate IDs on rerender | Fixed IDs `design-messages-heading` / `design-assembly-heading` — single result region only |
| No new fixed-width container | Uses existing `.panel` / `.metrics` |
| Presentation-only | Yes |
| Cut Layout vs Finished View summary parity | Finger Box cut vs finished: same Operations string and group presence |

**Compactness:** Lightweight grouping, not a result-card redesign.

**NOTE:** Valid results always include the exact-scale notice, so the old standalone “Ready:” line is effectively superseded for valid outputs (warnings/scale live under Messages). Non-blocking UX change.

---

## 6. Dimension formatter

| Check | Finding |
|-------|---------|
| Presentation-only | Yes |
| Finite display values | Uses `designNumberText`; does **not** reject `NaN` (`designDimensionsText(100, NaN)` → `100 × NaN mm`) — **LOW** |
| Two- and three-axis | `100 × 50 mm`, `100 × 50 × 25 mm` |
| Joins with `×` | Yes |
| Does not mutate inputs / feed production | Yes |
| Does not force box terms on flat designs | QR/Hanging use finished sign/panel size |

Cross-template metric dimensions use `×` for multi-axis values. Residual ASCII `x` remains in non-dimension strings (e.g. drawer grid `columns x rows`, some finger “segments x width” lines, coupon clearance label hyphens) — **LOW** consistency debt, not a dimension-format false positive.

---

## 7. Drawer Cabinet multi-output labeling

| Mode | Finding |
|------|---------|
| Linked | `Production SVG size` ×1; Shell/Interior size labels absent |
| Separate (`drawerUseSeparateInteriorThickness` + interior thickness 4) | Shell ×1, Interior ×1, generic Production SVG size ×0 |
| Shell / interior dimensions | Match `productionOutputs[0]` / `[1]` width/height |
| `productionOutputs` identity | shell + interior; filenames `l8-drawer-cabinet-shell-…` / `…-interior-…` |
| Physical pieces | 10 total across both outputs (Cut: 10) |
| Cabinet dimension metrics retained | Yes |

Baseline-vs-working production output arrays matched for separate mode.

---

## 8. QR Stand summary

Valid default QR result:

- Finished sign size (from semantic inputs), material thickness, physical pieces (2), slot fit clearance, Production SVG size, Operations (`Cut: 2 · Score: 0 · Labels: 0`)
- No Finished View selector
- No assembled-depth invention
- No finished-box terminology

---

## 9. Hanging Sign summary

Valid default hanging-sign result:

- Finished panel size, material thickness, physical pieces (3), Production SVG size, Operations (`Cut: 3 · Score: 0 · Labels: 0`)
- No Finished View selector
- No invented depth / box wording

---

## 10. Physical-piece wording

| Template / mode | Physical pieces source | Aligns with cut parts intent |
|-----------------|------------------------|------------------------------|
| qr-stand | `panels.length` = 2 | Yes |
| hanging-sign | `panels.length` = 3 | Yes |
| dice-tray | `pieceCount` = 5 (panels array length 21 includes slots/etc.) | **Physical pieces honest**; see Operations note |
| divider-tray | `pieceCount` = 7 (panels 25) | Same |
| finger-box open / lid | panels 5 / 6 | Yes |
| sliding-lid | panels 8 | Yes |
| drawer-cabinet linked | pieceCount 10 | Yes |
| drawer-cabinet separate | pieceCount 10; multi-output panel sum 10 | Yes |
| joint-fit-coupon finger | 12 | Yes |
| joint-fit-coupon wall-base (`couponType`) | 3 pieceCount path in builder; exercised via fixtures | Yes (suite) |
| concealed-cleat corner | 9 | Yes |
| concealed-cleat full-box | 21 | Yes |

Labels on/off does not change physical-piece counts for finger-box (Cut stays 5).

---

## 11. Operations counting contract

**Contract implemented:**  
`Cut` = length of `panels` (summed across `productionOutputs` when present)  
`Score` = `scorePaths`  
`Labels` = `labelPaths`  

Does **not** parse SVG, count DOM nodes, or imply laser passes / duration / material use.

| Check | Result |
|-------|--------|
| Exactly one Operations metric on valid results | Yes |
| Absent on invalid (no production artifact metrics) | Yes (`Operations` / Production SVG size suppressed) |
| Labels on/off changes Labels only | `Labels: 5` vs `Labels: 0`; Cut stays 5 |
| Faux dovetail (`boxCornerDecoration`) | Score 20, Cut 6 (loose lid); Cut not conflated with decoration |
| Separate cabinet | Cut 10 = shell 5 + interior 5 |
| Deterministic | Same ops string on cut vs finished rerender |

**Dice/Divider honesty note (MEDIUM → non-blocking):**  
Operations **Cut** counts structured `panels` entries (dice default Cut: 21) while **Physical pieces** correctly shows 5. README states Operations is not a physical-piece synonym and comes from structured production arrays. Still easy to misread as “21 cut parts.” Acceptable under stated contract; optional future label clarity (e.g. “Cut panels”) is an improvement, not a commit blocker.

**Label vs score double-count:** Separate arrays; Labels and Score are independent categories. No evidence labels are double-counted as both without being separate structured paths. Cleat Full-Box: Cut 21 · Score 8 · Labels 13 (guides vs labels distinct).

---

## 12. Assembly-label wording

Standardized via `designAssemblyLabelsText`:

- `Enabled — N emitted`
- `Disabled`
- `Enabled — N emitted, M omitted (detail)`

Template-specific detail retained (`safe labels`, `intended-face labels`).  
Coupon clearance labels still use older `Enabled - N emitted / M omitted` hyphen style — **LOW** residual inconsistency.

Enabling assembly labels changes production SVG length (expected user toggle), not this phase’s default goldens.

---

## 13. Warning ordering

| Check | Finding |
|-------|---------|
| `result.warnings` not mutated | `orderOk: true` (JSON before/after sort identical) |
| Presentation-only sort | `designOrderedWarnings` returns new array |
| Construction/material before large-layout | Cleat thin-material + prototype warnings before `Large single-SVG layout:` |
| Fit/movement before large-layout | Sliding-lid fit/rail warnings ordered in fit band |
| Exact-scale notice informational after warnings | Lives in Messages after warning list |
| Assembly separate after Messages | Yes |
| Unknown warnings preserved | Default priority band 3 |
| Deterministic | Stable index tie-break |

**Brittleness (NOTE, not current incorrectness):** Classification uses broad regex/substrings (`fit`, `rail`, `prototype`, etc.). Future warning wording could mis-band; not a failure on current catalog.

Large multi-warning cabinet/sliding cases exercised; large-layout placement verified on oversized thin cleat prototype.

---

## 14. Errors versus warnings

| Check | Finding |
|-------|---------|
| Hard errors → `valid: false` | Yes (`boxWidth: 0`) |
| Invalid disables production metrics / download path in UI fixtures | No Production SVG size / Operations; Preview unavailable |
| Warnings alone do not invent download block | Warning-bearing valid results still `valid: true` |
| Large-layout advisory non-blocking | Yes |
| Exact-scale informational | Yes |
| Visible `Error:` / `Warning:` prefixes | Yes |
| Screen-reader-visible text | Prefixes in DOM text, not color-only |

---

## 15. Accessibility semantics

| Check | Finding |
|-------|---------|
| `role="status"` on replaced result panel | Yes |
| `aria-live="polite"` | Yes |
| `aria-atomic="false"` | Yes (reduces full-region re-read severity) |
| Nested duplicate status regions | Single status on summary panel; preview outside |
| Error/warning not color-only | Prefixed text |
| Heading hierarchy | `h4` groups |
| Preview buttons keyboard-operable | Unchanged selector buttons |
| Valid HTML | `python -m html.parser index.html` OK |

**NOTE:** Status region includes metrics, messages, and long assembly text. Polite live updates after draft changes may be verbose for assistive tech. Technically valid; optional future refinement is to live-announce a short status line only.

---

## 16. Production-byte neutrality (baseline `fea4b90` vs working tree)

**True isolated baseline:** `git show fea4b90105a85dd60ede79e57853bd266a648c3f:index.html` executed in headless Edge alongside working tree.

Compared digests (length + content hash) for production SVGs / multi-outputs:

| Case | Match |
|------|-------|
| QR stand | Yes |
| Hanging sign | Yes |
| Dice Tray | Yes |
| Divider Tray | Yes |
| Finger Box open | Yes (`2483` / FNV `a892f91c` in suite goldens) |
| Finger Box loose lid | Yes |
| Finger Box faux dovetail (when correctly parameterized) | Suite goldens unchanged |
| Sliding-Lid Box | Yes vs baseline executable |
| Drawer Cabinet linked | Yes |
| Drawer Cabinet separate shell + interior | Yes (outputs array identity) |
| Joint Fit Coupon modes | Yes vs baseline digs |
| Concealed Cleat Corner | Yes (`4457` / `88721533` suite) |
| Concealed Cleat Full-Box | Yes (`9701` / `2316430b` suite) |

**`prod_mismatches`:** empty.  
**No production golden constants updated** in this phase.

---

## 17. Finished View neutrality

Finished View **preview stage** hashes compared baseline vs working for:

- Finger Box Finished View  
- Sliding-Lid Finished Closed / Open  
- Drawer Cabinet Finished Front  
- Dice / Divider Finished View  
- Concealed Cleat Corner / Full-Box Finished View  

**`fv_mismatches`:** empty.  
No Finished View helper function bodies changed in the product diff. Summary text is outside preview SVGs.

---

## 18. Download neutrality

- Design geometry fixtures continue to assert download bytes equal production SVG (including large advisory + Finished View cases).  
- Probe download helper failed only because the live form was not mounted in the isolated hook page (`Cannot read properties of null (reading 'elements')`) — environment artifact, not product regression.  
- Production SVG contents do not include “Operations”, “Result summary”, or Full-Box assembly prose.  
- Multi-output cabinet filenames/ids retained.

---

## 19. Storage and schema isolation

| Boundary | Result |
|----------|--------|
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` unchanged |
| `SCHEMA_VERSION` | `2` unchanged |
| Diff touches persist/load/import/export/recovery | No |
| localStorage before/after summary renders | Identical |
| `backupObject()` before/after | Identical |
| machineProfile 20W vs 40W finger-box SVG | Identical |

---

## 20. Fixture quality

Designs geometry: **1080 → 1086** (+6), all passing.

| New fixture | Behavioral value |
|-------------|------------------|
| Full-Box Assembly group without finger-box wording | **Strong** — renders real HTML, checks sequence themes |
| One Operations metric + status semantics across templates | **Good** — multi-template; count formula mirrors helper (**mild circularity** on ops equality) |
| QR/Hanging finished size + no FV selector | **Good** |
| Separate cabinet shell/interior labels | **Strong** — compares to `productionOutputs` |
| × formatting + warning before large-layout | **Good** |
| Invalid suppresses fabricated metrics | **Good** |

**Non-blocking fixture gaps / improvements:**

- Operations expected value recomputed with the same reduce as production helper (does not independently hard-code panel counts per template).  
- Warning-order fixture covers one oversized cleat path, not every template.  
- Does not assert empty-group omission matrix for every template.  
- Does not assert production-byte equality vs git baseline (covered by this audit + existing golden constants).

No commit-blocking fixture holes for the stated presentation contract.

---

## 21. Browser and responsive validation

| Item | Result |
|------|--------|
| Browser | Microsoft Edge via Playwright Chromium channel `msedge` |
| Mode | Headless |
| Screenshots | Not captured; DOM/metrics inspected programmatically |
| Console | No product pageerrors from audit hooks |
| 320 px viewport | `scrollWidth === clientWidth` (320); `overflow: false` |
| Desktop 1280 | Used for suite and probes |
| Horizontal overflow | None observed at 320 |

Physical cutting not performed (not required for presentation-only phase).

---

## 22. Documentation accuracy (README)

README correctly states:

- Grouped Result summary / Messages / Assembly  
- Finished dimensions distinct from Production SVG size  
- Separate shell/interior naming  
- Operations from structured data; not a time estimate  
- Errors block; warnings/notices do not  
- Full-Box intended assembly instructions  
- Presentation does not alter downloads  
- Suite totals 1086 / 1878 / tray 264  

Does **not** claim physical strength, guaranteed fit, nesting, multi-sheet export, bed fit, cut-time, material-use estimation, or new production geometry.

---

## 23. Commands and exact totals

| Command / action | Result |
|------------------|--------|
| `git status` / `rev-parse` / ahead-behind | Clean tracked scope; HEAD as above; 0/0 |
| `git diff --check HEAD` | Passed (0) |
| `python -m html.parser index.html` | Passed |
| Designs geometry (`runDesignGeometryFixtures`) | **1086 passed / 0 failed** |
| Tray model (`runTrayModelFixtures`) | **264 passed / 0 failed** |
| Complete 15-group suite (window materials runner + design, matching `selftest=all` arithmetic) | **1878 passed / 0 failed** |
| Baseline vs working production digests | **All matched** |
| Baseline vs working Finished View preview hashes | **All matched** |
| Storage / backup isolation | **Byte-identical** |
| 320 px overflow | **false** |

Independent complete-suite group totals (0 failed each): baseline 20, normalization 12, production 66, promotion 58, design-production 118, grid 23, grid-machine 18, grid-browser 67, materials **57** (must use `window.runMaterialBrowserFixtures`), library-browser 56, project-browser 61, project-wizard 216, design 1086, metadata 12, storage 8.  
Arithmetic: `792 + 1086 = 1878`.

---

## Blocking findings

**None.**

---

## Non-blocking improvements

1. **NOTE:** `role="status"` region may be verbose for screen readers after each input change.  
2. **NOTE:** Warning priority uses substring/regex banding — brittle for future warning text.  
3. **LOW:** Residual ASCII `x` in grid/finger pattern strings; coupon clearance label hyphen style.  
4. **LOW:** `designDimensionsText` does not filter non-finite values (`NaN`).  
5. **MEDIUM (clarity only):** Operations **Cut** on trays equals panel-array length, not physical piece count — documented but easy to misread.  
6. **NOTE:** Valid-path “Ready:” message is overshadowed by always-on scale notice under Messages.  
7. **NOTE:** Operations fixture equality is partly circular with the helper implementation.

---

## Unverified areas

- Interactive (non-headless) Edge visual QA and real AT (NVDA/VoiceOver) announcement quality.  
- Live click download in a fully mounted Designs form (covered by fixtures; hook-page download helper lacked form DOM).  
- Physical cutting / Full-Box strength (explicitly out of scope for this presentation commit).  
- Every custom-row cabinet layout permutation beyond suite + separate-thickness probe.

---

## Final verdict

**SAFE TO COMMIT WITH NON-BLOCKING NOTES**

The implementation is presentation-only, repairs the Full-Box assembly dead path honestly, covers registered templates consistently enough for commit, preserves production SVG bytes and Finished Views against baseline `fea4b90`, leaves storage/schema untouched, and is backed by independent Edge fixture totals **1086 / 264 / 1878** at zero failures.
