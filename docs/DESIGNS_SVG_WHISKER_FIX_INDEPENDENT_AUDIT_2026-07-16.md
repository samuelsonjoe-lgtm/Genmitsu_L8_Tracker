# Designs SVG Whisker Fix — Independent Second-Pass Audit

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `c7c6845` — `Add drawer cabinet design generator`

Mode: read-only audit; no application files were modified, staged, committed, pushed, reset, cleaned, stashed, moved, or deleted.

Primary reports reviewed (not accepted as proof): `docs/DESIGNS_SVG_WHISKER_INVESTIGATION_2026-07-16.md`, `docs/DESIGNS_SVG_WHISKER_FIX_2026-07-16.md`

## Executive conclusion

**SAFE TO COMMIT**

The uncommitted whisker fix correctly replaces independent per-edge corner concatenation with terminal-offset-aware stitching. Live regeneration confirms zero positive-length collinear overlaps on all Finger Box body panels, all Drawer Cabinet drawer panels (rows 1–3), and unchanged Sliding Lid and cabinet shell/shelf geometry. Designs fixtures report **445/0**; the complete direct-file suite reports **977/0**.

---

## Repository state verified

| Check | Result |
|---|---|
| Branch | `main...origin/main` |
| HEAD | `c7c6845 Add drawer cabinet design generator` |
| Tracked changes | `README.md`, `index.html` (unstaged) |
| `git diff --check` | Passed (LF/CRLF warnings only) |
| `git diff --stat` | `README.md` 2 lines; `index.html` +60 −12 (50 net) |

---

## Diff summary (from `c7c6845`)

1. **`buildFingerPanel()`** — delegates to `buildSlidingLidBodyPanel(definition)` instead of concatenating four `designEdgePoints()` slices at raw rectangle corners.
2. **`buildBoxModel()`** — runs `designPanelGeometryErrors()` on every generated panel.
3. **`buildDrawerCabinetModel()`** — runs `designPanelGeometryErrors()` on all panels (shell, shelves, and namespaced drawers), not only `cabinetPieces`.
4. **New helpers** — `designCollinearSegmentOverlapLength()`, `designPositiveCollinearOverlaps()`.
5. **`designPanelGeometryErrors()`** — rejects positive-length collinear overlaps.
6. **Fixtures** — 17 new Designs assertions; updated Finger Box / Drawer Cabinet SVG signatures; README suite total 960 → 977.

---

## 1. Root-cause correction

### Verified

| Item | Detail |
|---|---|
| Semantic validity | `buildFingerPanel()` and `buildSlidingLidBodyPanel()` accept the same `{ name, id, width, height, edges }` definition object. Finger Box and Drawer definitions use the same edge ordering (top → right → bottom → left), the same `designPatternEdge()` objects, the same four-corner winding (`corners` + `inward` vectors), and the same phase/clearance/depth semantics. |
| Composite isolation | `buildSlidingLidBodyPanel()` composite handling (`edge.composite`, `compositeSpan`, `designEdgeTerminalOffset` composite terminal rule) is used only by `buildSlidingLidSidePanel()`. Finger Box and Drawer panels never pass composite edges, so delegation cannot trigger sliding-lid-specific front-field truncation. |
| Panel output shape | Delegation routes through `designPanelFromPoints()`, which computes bounds from actual points rather than forcing `{ minX:0, minY:0, maxX:width, maxY:height }`. Envelope `width`/`height` on the returned panel are unchanged; only bounds min/max may reflect inset finger geometry. `designPanelGeometryErrors()` checks overflow only (`maxX > width`), so inset bounds do not false-fail. |

**Affected file/function:** `index.html` — `buildFingerPanel()`, `buildSlidingLidBodyPanel()`

**Physical consequence:** Corrected paths eliminate 3 mm collinear retraces at recessed-pattern corners without altering finger counts, mating boundaries, or panel envelopes.

**Fixture detection:** Corner-stitch fixtures (5) and overlap-free template fixtures (4) detect regressions.

---

## 2. Corner topology — independent traces

Fixture corner model: 30×30 mm panel, `depth = 3`, `clearance = 0`, 3-segment pattern (`actualWidth = 10`). Join formula independently derived:

```
join(cornerIndex) = rawCorner
  + inward[previous] × terminalOffset(previous, end)
  + inward[current]  × terminalOffset(current, start)
```

| Corner case | Expected join (independent) | Production `designEdgeTerminalOffset` join | Join present in path | Collinear overlaps |
|---|---|---|---|---|
| Patterned → patterned (corner 1) | `(27, 0)` | `(27, 0)` | Yes | 0 |
| Plain → recessed patterned (corner 1) | `(27, 0)` | `(27, 0)` | Yes | 0 |
| Recessed patterned → plain (corner 1) | `(30, 3)` | `(30, 3)` | Yes | 0 |
| Two recessed patterned edges (corner 1) | `(27, 3)` | `(27, 3)` | Yes | 0 |
| Closure to first edge (corner 0) | `(3, 3)` | `(3, 3)` | Yes; `points[0]` equals join | 0 |

**Baseline `c7c6845` (same join expectations):** patterned cases visited expected joins but still carried **2** collinear overlap pairs per mini-panel; recessed/closure cases often **missed** the expected join (`has: false`) while retaining overlaps. Default Finger Box `front` path on baseline contained visible whiskers (`V 53 V 50`, `V 53 V 42.45`); corrected path removes those excursions while preserving finger positions and envelope (`126 × 53`).

**Affected file/function:** `index.html` — `designEdgeTerminalOffset()`, `designSafePatternEdgePoints()`, `buildSlidingLidBodyPanel()`, `buildFingerPanel()`

**Physical consequence:** Shared join coordinates eliminate material-thickness backtracking; mating boundaries and finger lengths outside the corner trim remain unchanged.

**Fixture detection:** Five corner-stitch fixtures independently calculate joins; first fixture dumps expected vs actual points on failure.

---

## 3. Overlap diagnostic

### Verified detections (live, current code)

| Case | Expected | Observed length(s) |
|---|---|---|
| Adjacent immediate reversal | Detect | 3 mm |
| Non-adjacent closure overlap | Detect | 3 mm |
| Partial overlap | Detect | 5 mm |
| Exact duplicate traversal | Detect | 10 mm |
| Reversed horizontal traversal | Detect | 3 mm |
| Reversed vertical traversal | Detect | 7 mm |

### Verified non-flags

| Case | Result |
|---|---|
| Endpoint-only contact | 0 overlaps |
| Separated parallel segments (offset 5 mm) | 0 overlaps |
| Legitimate short finger loop (0.25 mm) | 0 overlaps |
| Perpendicular corner contact | 0 overlaps |

Epsilon handling uses `1e-9` for collinearity, degenerate length, and positive overlap threshold; axis choice follows dominant segment direction, covering both horizontal and vertical traversals in either direction.

**Affected file/function:** `index.html` — `designCollinearSegmentOverlapLength()`, `designPositiveCollinearOverlaps()`, `designPanelGeometryErrors()`

**Physical consequence:** Diagnostic catches whisker-class defects without deleting legitimate short fingers; false positives on perpendicular or endpoint-only contact were not observed in independent probing.

**Fixture detection:** Six literal-path fixtures cover the table above; reversed-direction cases are not fixture-listed (see Minor).

---

## 4. Validation scope

### Verified coverage

| Panel set | `designPanelGeometryErrors()` applied | Overlap count (live) |
|---|---|---|
| Finger Box body (5) | Yes — via `buildBoxModel()` | 0 / panel |
| Drawer Cabinet drawers (5 × rows) | Yes — via `buildDrawerCabinetModel()` on all `panels` | 0 / panel (rows 1–3) |
| Drawer Cabinet shell (5) | Yes | 0 / panel |
| Support shelves (0–2) | Yes | 0 / panel |
| Sliding Lid physical pieces (8) | Yes — pre-existing in `buildSlidingLidBoxModel()` | 0 / panel |

Invalid geometry fails safely: undersized box draft (`1×1×1` mm) returns `valid: false` with errors; no mutation of draft or `localStorage` (existing fixtures pass).

**Affected file/function:** `index.html` — `buildBoxModel()`, `buildDrawerCabinetModel()`, `buildSlidingLidBoxModel()`, `designPanelGeometryErrors()`

**Software consequence:** Preview/download blocked when validation errors exist; storage state unchanged.

**Fixture detection:** Template overlap-free assertions; existing invalid-dimension rejection fixtures.

---

## 5. Regression safety

### Byte-identical (current vs `c7c6845`, fixture-exact drafts)

| Output | Length | Hash | Status |
|---|---:|---:|---|
| Sliding Lid SVG (`slidingJointClearance: 0.10`) | 2800 | `4a7ab718` | Identical |
| Sliding Lid per-piece path concat | 1710 | `bd403ad1` | Identical |
| Cabinet shell + shelves (3-row) | 2288 | `63d818d0` | Identical |

### Intentionally changed (corrected corner stitching)

| Output | Length | Hash |
|---|---:|---:|
| Open-top Finger Box SVG | 2483 | `a892f91c` |
| Loose-lid Finger Box SVG | 2615 | `6181bc75` |
| One-row Drawer Cabinet SVG | 4436 | `7494c326` |
| Three-row Drawer Cabinet SVG | 9124 | `b158a794` |
| Three-row Drawer path set | 4094 | `56166d5a` |

### Unchanged metadata (current vs baseline, live compare)

| Property | Match |
|---|---|
| Calculated dimensions | Yes |
| Panel envelopes (`width`/`height`) | Yes |
| Finger Box layout positions | Yes |
| One-row Drawer Cabinet layout positions | Yes |
| Panel IDs and piece counts | Yes |
| Cabinet metrics (`rows`, `shelfCount`, clearances) | Yes |

**Baseline overlap debt removed:** Finger Box — 16 overlapping runs totaling 48 mm retrace length across 5 panels; Drawer (3-row) — 48 runs across 15 drawer panels. All zero after fix.

**Affected file/function:** `index.html` — `buildFingerPanel()`, fixture signature blocks ~L2675, ~L3005–L3048

**Physical consequence:** Cut files for Finger Box and Drawer panels lose spurious 3 mm laser retraces; Sliding Lid and cabinet shell cuts are unchanged.

**Fixture detection:** Updated SVG/path signatures; shell shelf signature fixture; layout/ID/mating fixtures.

---

## 6. Fixture quality

### Counts (live direct-file execution, Microsoft Edge headless)

| Group | Passed | Failed | Δ from `c7c6845` |
|---|---:|---:|---:|
| Designs (`runDesignGeometryFixtures`) | 445 | 0 | +17 |
| Complete suite (`?selftest=all` runners) | 977 | 0 | +17 |

Complete suite breakdown: Baseline 20, Normalization 12, Grid 23, Grid Browser 67, Materials 57, Library Browser 56, Project Browser 61, Metadata 12, Storage 8, Project Wizard 216, **Designs 445**.

### New assertion independence

| # | Assertion type | Independence |
|---|---|---|
| 1–5 | Corner stitch | **Independent** — `fixtureTerminalOffset` / `fixtureExpectedJoin` duplicate join math without calling production trim helpers; verifies join coordinates and absence of overlaps. |
| 6–11 | Overlap literals | **Circular** — invoke `designPositiveCollinearOverlaps()` directly on hand-built point lists. |
| 12–15 | Template overlap-free | **Partially circular** — call production overlap helper on generated panels; still valuable as regression guards. |
| 16–17 | Signature stability | **Independent** — hash/length gates on serialized output. |

**Malformed classes still missed:** Self-crossing paths without collinear overlap; whisker patterns below epsilon; composite-edge regressions routed through `buildFingerPanel()` (currently unreachable). None are production defects for Finger Box / Drawer.

---

## 7. Deferred SVG close

**Verified:** Corrected paths still explicitly return to the start vertex and append `Z` via `designPointsPath()` (e.g. Finger Box `front` ends `… H 0 V 0 Z`). Investigation correctly separates this redundant zero-length close from the whisker defect.

**Recommendation:** Do **not** bundle explicit-return/`Z` cleanup into this commit; it does not create a current production defect and lacks a dedicated fixture.

**Fixture detection:** None for deferred item; existing closed-path fixtures still pass.

---

## Findings classification

### Blocker

None.

### Major

None.

### Minor

**M1 — Overlap fixtures mirror production helper**

- **Affected file/function:** `index.html` — `runDesignGeometryFixtures()` overlap literal block (~L2597–L2602) and template overlap-free block (~L3006–L3009).
- **Explanation:** Six diagnostic and four template assertions call `designPositiveCollinearOverlaps()` directly, so they cannot catch a broken implementation that still passes literals crafted for the same algorithm.
- **Software / physical consequence:** A future bad edit to overlap math could pass fixtures while failing real panels unless corner-stitch or signature fixtures also fail.
- **Recommended correction:** Add at least one fixture with expected overlap lengths computed outside the production helper (corner-stitch fixtures already partially cover real panels).
- **Fixture detection:** Does not detect helper-internal regressions in isolation.

**M2 — Corner fixture offset omits composite terminal rule**

- **Affected file/function:** `index.html` — `fixtureTerminalOffset()` in `runDesignGeometryFixtures()`.
- **Explanation:** Fixture offset logic omits `edge.composite && !atStart` zero-terminal rule present in `designEdgeTerminalOffset()`.
- **Software / physical consequence:** None today — Finger Box and Drawer never use composite edges through `buildFingerPanel()`.
- **Recommended correction:** Align fixture offset with production if composite panels are ever routed through `buildFingerPanel()`.
- **Fixture detection:** Sliding Lid side panels use a separate code path; not covered by corner fixtures.

**M3 — Implementation report sliding path-set hash typo**

- **Affected file/function:** `docs/DESIGNS_SVG_WHISKER_FIX_2026-07-16.md` (documentation only).
- **Explanation:** Report cites sliding full path set `1728` / `edf8cec5`; independent live concatenation with fixture-exact draft yields `1710` / `bd403ad1`, identical on **both** `c7c6845` and the corrected tree.
- **Software / physical consequence:** Documentation-only; no code regression.
- **Recommended correction:** Update report hash if desired; not required for commit safety.
- **Fixture detection:** Sliding SVG signature fixture (`2800` / `4a7ab718`) covers the exported SVG.

### Verified

| ID | Summary |
|---|---|
| V1 | `buildFingerPanel()` → `buildSlidingLidBodyPanel()` is semantically valid for all Finger Box and Drawer rectangular panels. |
| V2 | Composite / sliding-lid-specific assumptions are unreachable from delegated finger panels. |
| V3 | All five corner topologies stitch at independently derived join coordinates with zero overlaps. |
| V4 | Default Finger Box `front` path whiskers removed; baseline had 6 mm retrace on front panel alone. |
| V5 | Overlap diagnostic detects adjacent, closure, partial, duplicate, and reversed-direction collinear overlaps. |
| V6 | Diagnostic does not flag endpoint-only, separated parallel, short-finger, or perpendicular contact. |
| V7 | All relevant physical panels are overlap-free under live regeneration. |
| V8 | Invalid geometry rejects export without draft / `localStorage` mutation. |
| V9 | Dimensions, envelopes, mating phases, IDs, piece counts, and layout positions unchanged. |
| V10 | Sliding Lid SVG `2800` / `4a7ab718` byte-identical to `c7c6845`. |
| V11 | Cabinet shell + shelf paths `2288` / `63d818d0` byte-identical to `c7c6845`. |
| V12 | Corrected Finger Box / Drawer SVG signatures match live hashes claimed in working tree. |
| V13 | Designs fixtures **445/0**; complete suite **977/0** (live Edge headless, direct file). |
| V14 | Deferred explicit return + `Z` remains separate; not a production whisker defect. |
| V15 | `README.md` suite count updated to 977. |

---

## Live verification method

- Engine: Microsoft Edge headless (`msedge.exe`), Playwright-driven `file:///` load of `index.html`.
- Baseline HTML: `git show c7c6845:index.html` (read-only cat).
- Overlap counting on baseline: production-equivalent helper attached to `window.__positiveOverlaps` (same algorithm as new helpers) because baseline lacks `designPositiveCollinearOverlaps()`.
- Signatures: fixture-exact drafts (`slidingJointClearance: '0.10'`, default Finger Box / Drawer Cabinet values).

---

## Final verdict

**SAFE TO COMMIT**

The whisker root cause is fixed with the narrowest correct delegation, validation is expanded appropriately, regressions are guarded by updated signatures and live fixture passes, and unaffected Sliding Lid / cabinet shell geometry remains byte-identical to `c7c6845`. Minor findings are fixture-hardening and documentation nits only.