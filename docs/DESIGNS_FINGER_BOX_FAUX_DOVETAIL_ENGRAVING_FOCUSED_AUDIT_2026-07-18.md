# Finger Box Faux Dovetail Engraving — Focused Audit

**Date:** 2026-07-18  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit type:** Independent, adversarial, read-only implementation audit  
**Expected baseline:** `0fa1d37` — Add custom row drawer cabinet layouts  
**Feature under audit:** Finger Box faux dovetail engraving  

This audit does not trust the implementation report. Findings come from the committed baseline, the working-tree source, complete diff enumeration, independent finger-interval derivation, and direct `file://` headless Microsoft Edge runtime reproduction.

---

## 1. Repository and synchronization state

| Check | Result |
| --- | --- |
| `git status -sb` | `## main...origin/main`; tracked `M README.md`, `M index.html` |
| `git log -1 --oneline` | `0fa1d37 Add custom row drawer cabinet layouts` |
| `git rev-parse HEAD` | `0fa1d37ae092fff19bebec1b9ad3b8aec550b025` |
| Matches expected `0fa1d37` | **Yes** |
| `git branch --show-current` | `main` |
| `git rev-list --left-right --count HEAD...origin/main` | `0 0` (fully synchronized) |
| `git diff --check` | Clean (CRLF warnings only; no conflict markers / whitespace errors) |
| `git diff --stat` | `README.md 3 ±`; `index.html 98 ±`; **2 files, 96 insertions, 5 deletions** |
| `git diff --numstat` | `2 1 README.md`; `94 4 index.html` (reconciles with `--stat`) |
| `git diff --cached --name-only` | **Empty** (nothing staged) |
| Tracked modified only | **`index.html`, `README.md` only** — matches expectation |
| Staged state | Empty |
| Read-only audit actions | No application or documentation files edited except **this** audit report; no stage/commit/push/reset/clean/stash |

### Untracked inventory (untouched by this audit)

Expected new implementation materials present:

- `docs/DESIGNS_ADDITIONAL_JOINT_TYPES_ARCHITECTURE_REVIEW_2026-07-18.md`
- `docs/DESIGNS_FINGER_BOX_FAUX_DOVETAIL_ENGRAVING_IMPLEMENTATION_2026-07-18.md`

Also present and **left untouched** (pre-existing / out of scope):

- `LightBurn Projects/*`, `debug.log`, `.claude/` (if present), `parametric_qr_stand_generator.py`
- Historical `docs/*` audit/implementation reports unrelated to this feature
- This audit’s temporary runner scripts lived only under the auditor worktree `agent-tools/` (not the product tree)

---

## 2. Complete working-tree diff-hunk enumeration

Source: `git diff` against `HEAD` / `0fa1d37`. Insertion/deletion totals reconcile with `git diff --numstat` (**94/4** `index.html`, **2/1** `README.md`).

### `index.html` hunks

| Hunk | Approx. region | Classification | Assessment |
| --- | --- | --- | --- |
| `@@ -412,7 +412,7 @@` | `designDefaults()` | **default field** | Adds session-only `boxCornerDecoration: 'none'`. Expected. |
| `@@ -1813,6 +1813,8 @@` | Finger Box fields in `renderDesigns()` | **Finger Box UI** | Select + help text only on Finger Box path. Expected. |
| `@@ -2194,9 +2196,11 @@` | `normalizeDesignDraft()` Finger Box branch | **normalization** | Raw `boxCornerDecoration` → normalized `cornerDecoration`; allow-list error. Expected. |
| `@@ -2904,6 +2908,45 @@` | After `layoutDesignPanels` | **decoration geometry helper** | New `fingerBoxCornerDecorationSegments()`. Expected. |
| `@@ -3038,13 +3081,21 @@` | `buildBoxDesignResult()` | **result/SVG assembly**, **warnings**, **metrics** (return object) | Wires decoration into score paths; zero-mark error; decorative warnings; metrics fields. Expected. |
| `@@ -3351,6 +3402,7 @@` | `designResultsHtml()` Finger Box metrics | **metrics** | Corner decoration metric line. Expected. |
| `@@ -3369,7 +3421,7 @@` | Finished View assembly note | **Finished View note** | Cut-Layout pointer only; screen-only. Expected. |
| `@@ -3757,6 +3809,44 @@` | `runDesignGeometryFixtures()` | **fixtures** | 34 new assertions. Expected. |

### `README.md` hunks

| Hunk | Classification | Assessment |
| --- | --- | --- |
| Feature bullet under “Other features” | **README** | Decorative-only contract; no physical claim. Accurate. |
| Built-in checks totals | **README** | `1786` / Designs `994` / Tray `264`; `1752 + 34 = 1786`. Matches runtime. |

### Functions **not** modified (confirmed by hunk list + content review)

| Function | Status if changed |
| --- | --- |
| `buildFingerPattern()` | Unchanged — **Major gate clear** |
| `designPatternEdge()` | Unchanged |
| `designSafePatternEdgePoints()` | Unchanged |
| `designEdgeTerminalOffset()` | Unchanged |
| `designPointsPath()` | Unchanged (still H/V-only for cuts) |
| `designScoreSegmentsPath()` | Unchanged (already supports diagonal `L`) |
| `designSegmentsContainedInPolygon()` / `designMinimumSegmentClearance()` | Unchanged (reused) |
| `serializeDesignSvg()` | Unchanged (uses existing `scorePaths` + `scoreGroupId`) |
| `layoutDesignPanelRows()` / `layoutDesignPanels()` | Unchanged |
| Storage (`STORAGE_KEY`, `SCHEMA_VERSION`, `freshState`, `loadState`, `persist`, `backupObject`, import/export) | Unchanged |
| Suite registration / `selftest` dispatcher | Unchanged (only Designs fixture body grew) |
| Unrelated templates (Sliding-Lid, Tray, Drawer Cabinet, Coupons) | Production builders unchanged |

No unexplained or unrelated template/serializer/storage hunks.

---

## 3. Files and functions inspected

**Source:** working-tree `index.html`, committed `0fa1d37` baseline via `git show`, `README.md`, architecture review, implementation report, prior Finger Box / assembly-label / score-layer context.

**Functions / regions (minimum set, actual line numbers from working tree — not report prose):**

- `designDefaults()` (~414)
- Finger Box UI field construction in `renderDesigns()` (~1809–1818)
- `normalizeDesignDraft()` Finger Box branch (~2196–2208)
- `buildFingerPattern()`, `designPatternEdge()`, `designPointsPath()` (~2243–2274)
- `buildBoxModel()` wall edge phases (~2278–2313)
- `designEdgeTerminalOffset()`, `designSafePatternEdgePoints()` (~2513–2528)
- `designSegmentsContainedInPolygon()`, `designSegmentContainedInPolygon()`, `designSegmentsIntersect()` (~2593–2608)
- `designScoreSegmentsPath()` (~2669)
- `designMinimumSegmentClearance()`, assembly-label path builders (~2762–2800)
- `layoutDesignPanelRows()` / `layoutDesignPanels()` (~2889–2910)
- **`fingerBoxCornerDecorationSegments()`** (~2911–2948)
- `serializeDesignSvg()` (~2950–2955)
- `buildBoxDesignResult()` (~3078–3098)
- `designResultsHtml()` metrics / Finished View note / preview (~3400–3433)
- Designs fixtures decoration block (~3812–3848)
- Storage constants; `applyDesignProductionValues()`; complete-suite dispatcher (~10744–10760)
- Existing Finger Box open/loose goldens; Sliding / Tray / Cabinet fixture regions (no golden byte changes)

**Independent runtime:** headless Edge `channel=msedge`, direct `file://` to product `index.html` and to a temp file of `git show 0fa1d37:index.html`.

---

## 4. Baseline runtime reproduction (`0fa1d37`)

Method: write committed `index.html` to a temp path; open `file://…?selftest=all` in isolated headless Edge; aggregate console group `N passed` / `M failed` pairs (Material browser uses final cumulative pair only, matching prior audits).

| Group | Passed | Failed |
| --- | --- | --- |
| Baseline resolution fixtures | 20 | 0 |
| Material test normalization fixtures | 12 | 0 |
| Production settings fixtures | 66 | 0 |
| Evidence promotion fixtures | 58 | 0 |
| Design production settings fixtures | 118 | 0 |
| Grid promotion fixtures | 23 | 0 |
| Test Grid machine identity fixtures | 18 | 0 |
| Grid browser fixtures | 67 | 0 |
| Material browser fixtures (final) | 57 | 0 |
| Library browser fixtures | 56 | 0 |
| Project browser fixtures | 61 | 0 |
| Wizard metadata fixtures | 12 | 0 |
| Storage recovery fixtures | 8 | 0 |
| Project Wizard fixtures | 216 | 0 |
| Design geometry fixtures | **960** | 0 |
| **Complete suite** | **1752** | **0** |

- `readyState`: complete; no page exceptions.
- Every group connected and passing.
- Separately callable Tray-model (nested inside Designs at baseline and working tree; re-run alone on working tree): **264 / 0**.
- Matches expected baseline: Tray **264**, Designs **960**, complete **1752**.

---

## 5. Working-tree runtime reproduction

Same browser, `file://` product tree, same aggregation method.

| Group | Passed | Failed | vs baseline |
| --- | --- | --- | --- |
| Baseline resolution | 20 | 0 | unchanged |
| Material test normalization | 12 | 0 | unchanged |
| Production settings | 66 | 0 | unchanged |
| Evidence promotion | 58 | 0 | unchanged |
| Design production settings | 118 | 0 | unchanged |
| Grid promotion | 23 | 0 | unchanged |
| Test Grid machine identity | 18 | 0 | unchanged |
| Grid browser | 67 | 0 | unchanged |
| Material browser (final) | 57 | 0 | unchanged |
| Library browser | 56 | 0 | unchanged |
| Project browser | 61 | 0 | unchanged |
| Wizard metadata | 12 | 0 | unchanged |
| Storage recovery | 8 | 0 | unchanged |
| Project Wizard | 216 | 0 | unchanged |
| Design geometry | **994** | 0 | **+34** |
| **Complete suite** | **1786** | **0** | **+34** |

### Complete-suite arithmetic

```text
20 + 12 + 66 + 58 + 118 + 23 + 18 + 67 + 57 + 56 + 61 + 12 + 8 + 216 + 994 = 1786
```

Verified:

- Designs increased by **exactly 34** (960 → 994).
- No unrelated group count changed.
- No group skipped; no double-count (Material browser final pair only).
- `readyState` complete; no runtime exceptions in suite logs.
- Tray-model alone: **264 / 0** (unchanged).
- Source, README, and implementation-report totals reconcile: **264 / 994 / 1786**.

Additional: `python -m html.parser index.html` passed.

---

## 6. Normalization and UI contract

### Raw field: `boxCornerDecoration`  
### Normalized field: `cornerDecoration`

| Input | Normalized | Valid? |
| --- | --- | --- |
| missing / undefined | `none` | yes |
| blank / whitespace | `none` | yes |
| `'none'` | `none` | yes |
| `'faux-dovetail-engrave'` | `faux-dovetail-engrave` | yes |
| unknown nonblank (e.g. `'dovetail'`) | retained as raw string | **error:** exact text `Choose a recognized corner decoration.` |

Runtime probe confirmed all five cases.

### UI

- Field label: **Corner decoration**
- Options: **None**; **Faux dovetail engraving (decorative only)**
- Help text states: marks over actual finger positions; structural joint remains finger; decorative only; cut/fit/strength unchanged; blue score before red cut.
- Field is built only inside the Finger Box `boxFields` string — not Sliding-Lid, Dice Tray, Divider Tray, Drawer Cabinet, Joint Fit Coupon, or wall-to-base coupon field blocks.
- Probe: Sliding-Lid / Dice Tray / Drawer Cabinet SVGs unchanged when the raw field is forced on.
- Default: `designDefaults().boxCornerDecoration === 'none'`; absent from `freshState()` / backup keys.
- Preview/metrics/warnings refresh via existing Designs draft → `buildDesignResult` → `renderDesigns` path (fixtures exercise form HTML + live preview/download helper).
- No storage or event-history state introduced.

---

## 7. Disabled / legacy byte identity

For missing, blank, and explicit `none`, independent probe:

| Property | Result |
| --- | --- |
| Full SVG bytes vs default open Finger Box | **Identical** |
| Open golden | **2483 / `a892f91c`** (unchanged) |
| Loose lid golden fixtures still in suite | **Pass** (suite green) |
| No `score` / `corner-decoration` when disabled | **Confirmed** |
| Sliding / Tray / Cabinet / coupon goldens | Unchanged (suite green; probe equality on Sliding/Tray/Cabinet) |

**No disabled-state byte drift.** Major gate clear.

---

## 8. Enabled cut identity

Default Finger Box, decoration on vs off, identical structural inputs:

| Property | Equal? |
| --- | --- |
| Ordered panel IDs | Yes |
| Panel titles, width, height | Yes |
| Panel path strings | Yes |
| Layout x/y | Yes |
| Entire red `g#cut` outerHTML | **Byte-identical** |
| Filename / MIME (live fixture) | Unchanged (`l8-finger-box-…svg`, SVG MIME) |

Only additive blue score content differs in the full SVG. Model-path equality alone was **not** accepted; cut-group markup was compared.

Enabled golden pin (after structural checks): **7269 bytes / `d9512d1c`** — matches claim; re-build deterministic.

---

## 9. Finger-pattern ownership

### Wall edge descriptors (`buildBoxModel`)

| Panel | Left edge (index 3) | Right edge (index 1) |
| --- | --- | --- |
| Front | `verticalPattern`, **phase true** | `verticalPattern`, **phase true** |
| Back | same | same |
| Left | `verticalPattern`, **phase false** | `verticalPattern`, **phase false** |
| Right | same | same |

`isFinger(index) = (index % 2 === 0) === edge.phase` matches `designSafePatternEdgePoints` / cut generation.

### Decorated intervals (default 3 mm box)

Independent re-derivation using `buildBoxModel` + `layoutDesignPanels` + the same clearance/terminal-trim formulas as the cut pipeline, **without** reading helper output as ground truth:

- Eligible outer-finger intervals: **20**
- Helper marks: **20**, omitted: **0**
- Interval keys (panel|edge|index|start|end): **exact match** (`intervalsMatch: true`)

| Edge | Phase | Decorated source indices | Physical meaning |
| --- | --- | --- | --- |
| Front left/right | true | 0, 2, 4 | Protruding fingers on Front |
| Back left/right | true | 0, 2, 4 | Protruding fingers on Back |
| Left left/right | false | 1, 3 | Protruding fingers on Left (odd indices) |
| Right left/right | false | 1, 3 | Protruding fingers on Right |

Front/Back phase-true mates with Left/Right phase-false — consistent with assembled corner finger ownership. Marks follow **actual outer finger intervals**, not an ornamental rhythm.

### Left/right orientation

| Edge | Edge start / direction (matches cut edge walk) | Inward |
| --- | --- | --- |
| Left (index 3) | bottom → top (`y: height`, direction −1) | `+x` (positive panel x) |
| Right (index 1) | top → bottom (`y: 0`, direction +1) | `−x` |

Probe: left marks have inner x > outer x and outer ≥ 0.4 mm; right marks have inner x < outer x and outer ≤ width − 0.4 mm. Base is interior vertical segment; flanks open toward the cut edge; no outward pointing.

---

## 10. Decoration geometry

### Constants (source-verified)

| Constant | Value |
| --- | --- |
| Edge inset | 0.5 mm |
| End inset | 0.5 mm |
| Min decoration-to-cut clearance | 0.4 mm |
| Min inner width | 1.5 mm |
| Min source interval | 5 mm |
| Depth | `min(3, max(1.5, 0.75 × t))` |
| Taper | `min(1.5, max(0.25, 0.35 × t))` |

| Thickness | Depth | Taper |
| --- | --- | --- |
| 3 mm | **2.25** | **1.05** |
| 6 mm | **3** | **1.5** |

### Shape

Each mark: **3 score segments** — two diagonals + one interior base; open toward the cut edge; finite; non-self-intersecting under containment rules; deterministic path via `designScoreSegmentsPath` (diagonal `L` supported). No closed cut polygon; no score line on the red cut edge (edge inset 0.5 mm).

---

## 11. Containment and cut clearance

For every emitted mark on the default enabled box:

- Coordinates finite: **yes**
- Full-segment containment via `designSegmentsContainedInPolygon` (endpoints **and** non-touching all polygon edges, not endpoints alone): **yes**
- Min distance to complete owning-panel cut outline (`designMinimumSegmentClearance` over all panel outline segments, including finger notches): **min observed 0.5 mm** ≥ 0.4 mm
- Owning panel association correct: **yes**

### Challenge notes

1. **Full segment containment:** `designSegmentContainedInPolygon` rejects segments that come within 1e-9 of any polygon edge and requires both endpoints strictly inside — not endpoint-only.
2. **Diagonal vs axis-aligned intersect helper:** pre-existing `designSegmentsIntersect` is axis-aligned-oriented. Distance fallback uses endpoint-to-segment distances. For marks confined inside solid finger intervals by construction, observed clearance is ≥ 0.5 mm and containment passes. This is a **latent general limitation** of the shared helper for arbitrary diagonals, not an observed failure for this feature. Severity: **Informational** (not a commit blocker).

---

## 12. Omission and zero-mark behavior

### Small valid fixture (`30×30×15`, preferred finger 6)

- Generated marks: **8**
- Omitted: **4**
- Omission reasons: geometric (“available finger region is too small” / containment-clearance path when applicable)
- Cut group byte-identical to undecorated small box
- Warning exact: `4 faux dovetail marks omitted because the available finger regions are too small.`
- Result remains valid with honest metrics

### Zero-mark case

- `valid: false`, empty SVG, production not offered as success
- Exact error: `Faux dovetail engraving could not generate any readable marks for the available finger regions. Choose a larger Finger Box or leave corner decoration disabled.`
- Selecting **None** restores structural generation (legacy bytes)

---

## 13. SVG group hierarchy (actual)

Enabled + labels:

```text
svg
├─ g#score (blue)
│  ├─ g#corner-decoration
│  │  └─ g#corner-decoration-<panel>-<left|right>-NN  (×20)
│  └─ g#assembly-labels
│     └─ g#label-… (×5)
└─ g#cut (red)
   └─ g#panel-… (bottom, front, back, left, right)
```

Verified:

- Exactly one top-level `score`, one `cut`
- Score precedes cut
- No duplicate IDs
- No score content outside score; no cut content inside score
- Disabled: no score group at all

Matches the implementation report hierarchy.

---

## 14. Diagonal safety

| Check | Result |
| --- | --- |
| Score paths contain diagonal segments | Yes |
| Diagonals serialize via score path (`L`) | Yes |
| Diagonals in `panel.points` / `panel.path` | **No** |
| Diagonals in red cut `d` attributes | **No** |
| `designPointsPath()` expanded for diagonals | **No** (unchanged) |
| Cut group before vs after enable | **Identical** |

---

## 15. Label coexistence

Both assembly labels and faux dovetail enabled:

- Decoration present (20 marks); labels present (5); neither dropped
- Score order deterministic: corner-decoration then assembly-labels
- IDs unique
- Warning: same engraved sheet face / may both appear outward
- No automatic face flip or two-sided claim
- Red cut bytes unchanged vs decoration-only structural cut
- Preview/download fixture includes both score sets

Labels are **blue score vector paths** (not SVG `<text>`), consistent with prior assembly-label architecture.

---

## 16. Panel association and IDs

Each mark retains: panel id, edge, pattern index (`sourceFingerIndex`), 1-based `markIndex`, source interval, local segments, path, layout `x`/`y`, `cutClearance`, deterministic id:

```text
corner-decoration-<panelId>-<left|right>-<NN>
```

Ordering: Front → Back → Left → Right; left edge before right; ascending source index. Stable across rebuilds. No collision with panel, label, rail-guide, or shelf-guide IDs.

---

## 17. Production output, preview/download, Finished View

| Contract | Result |
| --- | --- |
| One SVG, one Cut Layout preview, one download | Yes |
| Filename / MIME | Existing Finger Box contract |
| Disabled: no empty score emission | Yes; full SVG = baseline |
| Enabled: additive blue score only; score before cut | Yes |
| Preview object data === result.svg === download bytes | Fixture + live helper **pass** |
| Zero-mark: no successful production SVG | Yes |
| Finished View | Does **not** render decoration; does not read score/SVG for engraving; screen-only; note points to Cut Layout |

---

## 18. Metrics and warnings

| State | Metric |
| --- | --- |
| Disabled | `Corner decoration: None` |
| Enabled (default) | `Faux dovetail engraving — 20 marks, decorative only` (+ omitted count when applicable) |

Warnings cover: decorative only; structural finger joint; no fit/strength change; engraved faces outward; appearance not physically verified; scrap/small first result; unattended-cut safety; labels+decoration same face when co-enabled.

No wording implies true structural dovetail, reinforcement, improved fit, strength, or squareness.

---

## 19. Storage and Production Settings

| Boundary | Status |
| --- | --- |
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged |
| `freshState` / `loadState` / `persist` / `backupObject` / import-export | Unchanged |
| localStorage / backup around decoration generate/preview/download/zero-mark | **Byte-identical** |
| New Production Settings field | **None** |
| `materialThickness` / `jointClearance` behavior | Unchanged |
| Decoration uses jointClearance? | **No** (uses pattern edge clearance only for interval bounds; no new persisted clearance) |
| `applyDesignProductionValues` | Only applies applicability targets — no `cornerDecoration` target |

Inventory, Projects, Pricing, Library, Test Grid, Material Test, and non–Finger-Box production templates: no production-path changes; suite groups all green.

---

## 20. Fixture-quality assessment (34 new assertions)

| Class | Count (approx.) | Notes |
| --- | --- | --- |
| **Independent** | Majority | Normalization wording; disabled full-byte identity; cut-group identity; golden pin; zero-mark error; small omit 8/4 + cut equality; form text; storage isolation; Sliding/Tray non-effect; trapezoid shape; ID uniqueness; preview/download; decorative metrics/warnings |
| **Partially independent** | Few | “Match actual outer vertical finger intervals” checks phase + interval bounds against panel edges but does not fully re-derive boundaries outside the helper. **Independent audit re-derivation closed this gap** (`intervalsMatch: true`). |
| **Circular** | None material | No pure self-compare of helper to itself as sole proof of ownership |
| **Irrelevant** | None | All 34 target this feature |

Bundled multi-condition fixtures use clear names; failures would still identify the assertion name. No overstated name that would pass while core contract fails (e.g. cut identity is a separate assertion).

---

## 21. Enabled golden verification

Before accepting the pin, structural checks above were satisfied (ownership, constants, containment, clearance, hierarchy, cut identity, labels). Then:

| Pin | Actual |
| --- | --- |
| Length | **7269** |
| Hash | **`d9512d1c`** |
| Deterministic rebuild | Yes |

Existing open (**2483 / `a892f91c`**) and loose goldens unchanged.

---

## 22. README / architecture / implementation report accuracy

| Claim area | Verdict |
| --- | --- |
| Decorative only; score only; cuts unchanged | **Accurate** |
| Actual finger positions | **Accurate** (independent interval match) |
| Group hierarchy | **Accurate** |
| Mark constants / 3 mm & 6 mm depth-taper | **Accurate** |
| Small 8/4 omit; zero-mark error | **Accurate** |
| Enabled golden 7269 / d9512d1c | **Accurate** |
| Fixture totals 264 / 994 / 1786 | **Accurate** (runtime reproduced) |
| Storage isolation; no Production Settings field | **Accurate** |
| No physical validation claimed | **Accurate** |
| Architecture review phase-1 decorative axis | Implementation matches selected design |

No unsupported production or runtime claim found that would change the commit decision.

---

## 23. Findings table

| ID | Severity | Location | Consequence | Recommendation | Fixture coverage |
| --- | --- | --- | --- | --- | --- |
| — | — | — | No Blocker or Major defects found | None required for commit | — |
| F1 | Informational | Shared `designSegmentsIntersect` (pre-existing) | Axis-aligned intersection model is imperfect for arbitrary diagonals; not observed to admit unsafe marks for this feature | No change required for this PR; future diagonal score features should keep interval confinement or generalize intersection carefully | Containment + clearance fixtures; independent min clearance 0.5 mm |
| F2 | Informational | Physical / process | Appearance, corner alignment when assembled, fit, strength, LightBurn, and machine cutting are unverified | Scrap/small first cut; already warned in UI | Warning fixtures |
| F3 | Informational | Architecture | Structural joint types (true dovetail, cleats, etc.) remain deferred | Out of scope | N/A |

No Minor wording or report mismatches requiring correction before commit.

---

## 24. Unverified physical areas (explicit non-claims)

This audit does **not** claim and did **not** verify:

- LightBurn import
- Physical engraving appearance
- Assembled-corner visual alignment between mating walls
- Machine cutting
- Fit or strength of the finger joint (unchanged by design)
- Kerf compensation behavior

---

## 25. Final conclusion

The working tree implements the architecture review’s selected decorative-only phase. Structural finger geometry and every red cut byte remain baseline-identical when decoration is disabled; when enabled, only blue score content is additive. Finger intervals, geometry constants, containment, clearance, SVG hierarchy, labels coexistence, omission/zero-mark honesty, storage isolation, Production Settings boundaries, goldens, and complete-suite arithmetic (**1786 = 1752 + 34**) were independently confirmed.

**SAFE TO COMMIT**
