# Designs Joint Fit Coupon — Independent Audit

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `b1ba69e` — `Fix finger-panel SVG corner retracing`

Mode: read-only audit; no application files were modified, staged, committed, pushed, reset, cleaned, stashed, moved, or deleted.

Primary reports reviewed (not accepted as proof): `docs/DESIGNS_JOINT_FIT_COUPON_ARCHITECTURE_REVIEW_2026-07-16.md`, `docs/DESIGNS_JOINT_FIT_COUPON_IMPLEMENTATION_2026-07-16.md`

## Executive conclusion

**SAFE TO COMMIT**

The Joint Fit Coupon template is correctly scoped, session-only, and reuses the corrected finger-panel geometry path without altering shared box, sliding-lid, or drawer generators. Live Edge headless verification confirms independent mating-coordinate math, signed-clearance semantics, safe validation at signed endpoints, stable layouts, and **515/0** Designs fixtures plus **1047/0** complete-suite fixtures. All pre-existing template SVG signatures remain byte-identical to `b1ba69e`.

---

## Repository state verified

| Check | Result |
|---|---|
| Branch | `main...origin/main` |
| HEAD | `b1ba69e Fix finger-panel SVG corner retracing` |
| Tracked changes | `README.md`, `index.html` (unstaged) |
| `git diff --check` | Passed (LF/CRLF warnings only) |
| `git diff --stat` | `README.md` 4 lines; `index.html` +292 −13 (283 net) |

---

## Diff summary (from `b1ba69e`)

Additive only for the new template:

- Session defaults in `designDefaults()` (`jointCouponEdgeLength`, `jointCouponBodyDepth`, `jointCouponPreferredFingerWidth`, `jointCouponClearances`, `jointCouponLabels`).
- UI: template selector entry, coupon fields, 90° assembly note, test-order guidance, textarea refresh in the Designs form handler.
- `parseJointCouponClearances()`, `jointCouponLabelText()`, `buildJointCouponModel()`, `buildJointCouponDesignResult()`.
- Numeric vector glyphs `0–9`, `.`, `-` in `designAssemblyGlyphs`.
- Designs dispatch, results metrics, assembly guidance, kerf note.
- **70** new Designs fixtures; README suite total updated to **1047**.

**Not modified:** `buildFingerPattern()`, `designSafePatternEdgePoints()`, `buildFingerPanel()`, `buildSlidingLidBodyPanel()`, `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`, `loadState()`, `persist()`, `backupObject()`, import merge/replace paths.

---

## 1. Fixed-decimal parser

### Verified behavior (live `parseJointCouponClearances()`)

| Rule | Result |
|---|---|
| Comma and newline separators | Accept (`0.06\n0.04,0.02`) |
| Optional `+` / `-` signs | Accept (`-0.10,+0.30,0.00`) |
| Exactly one or two fractional digits required | Reject integer token `0` (needs decimal point) |
| Reject exponent notation | First token in `1e-2,0.02,0.00` rejected; list fails overall |
| Reject blank / whitespace-only input | `''` and `'   '` reject |
| Reject blank interior token | `0.06, ,0.04,0.02` errors on blank token |
| Reject trailing separator blank token | `0.06,0.04,0.02,` errors once; generation blocked by accumulated errors |
| Reject more than two decimal places | `0.061` rejected |
| Normalize to integer hundredths | `0.06` → `hundredths: 6` |
| `-0.00` → `0.00` | Duplicate-with-zero fixture passes |
| Reject duplicates after normalization | `0.04,0.04,0.02` and `0.00,-0.00,0.02` reject |
| Preserve input order | Default order `0.06|0.04|0.02|0.00|-0.02|-0.04` preserved |
| Require 3–8 values | 2-value and 9-value lists reject |
| Accept `-0.10` and `+0.30` boundaries | `couponBoundaries` fixture and live probe pass |
| Reject outside `[-0.10, +0.30]` | `-0.11` and `0.31` reject |

### Semantic note (not a defect)

Tokens with **one** decimal place are syntactically valid but denote the literal millimeter value: `0.6` → `hundredths: 60` (0.60 mm) and is rejected as out of range. Users must enter `0.06` for six hundredths of a millimeter. This matches fixed-decimal semantics and the UI’s two-decimal examples.

Leading zeros (`00.04`) are accepted and normalize to `0.04`.

**Affected file/function:** `index.html` — `parseJointCouponClearances()`

**Consequence:** Malformed lists fail closed via `normalizeDesignDraft()` errors; partial value arrays never reach geometry when any parser error exists.

**Fixture detection:** 18 parser-focused assertions (mostly direct parser calls).

---

## 2. Signed-clearance semantics

### Independent verification (45 mm edge, five 9 mm segments, 3 mm thickness, clearances `0.04 / 0.00 / -0.04`)

| Case | Coupon A transitions | Coupon B transitions |
|---|---|---|
| Nominal boundaries | `0, 9, 18, 27, 36, 45` | same pattern object |
| `+0.04` | `8.98, 18.02, 26.98, 36.02` | `9.02, 17.98, 27.02, 35.98` |
| `0.00` | `9, 18, 27, 36` | `9, 18, 27, 36` |
| `-0.04` | `9.02, 17.98, 27.02, 35.98` | `8.98, 18.02, 26.98, 36.02` |

Additional checks:

- Phases complementary (A `true`, B `false`).
- Same nominal `buildFingerPattern()` object shared per pair.
- Positive pair separation magnitude `0.04` mm; negative reverses direction.
- Recess depth `3` mm on all coupons.
- Finger Box, Sliding Lid, Drawer Cabinet still reject negative joint clearances (live probe + fixture).

**Affected file/function:** `index.html` — `designPatternEdge()`, `designSafePatternEdgePoints()`, `buildFingerPanel()`, `buildJointCouponModel()`

**Physical consequence:** Positive clearance loosens mating; negative clearance tightens into interference; zero is nominal. Existing templates remain non-negative only.

**Fixture detection:** Eight independent transition-coordinate assertions; four template negative-clearance rejection assertions.

---

## 3. Physical coupon topology

### Verified

| Property | Result |
|---|---|
| Patterned edges per coupon | 1 (top edge `edges[0]`) |
| Plain edges | 3 (`edges[1..3] === null`) |
| Local 90° finger joint | Same `buildFingerPanel()` path as box/drawer walls |
| Not a flat coplanar puzzle | UI and assembly text explicitly require 90° assembly |
| Grip depth default 22 mm | ≥ `max(4×3, 16) = 16` mm |
| Pieces per clearance | 2 (A then B) |
| Emission order | A before B; normalized clearance order preserved |
| Default output | 6 pairs → 12 pieces |

Instructions in UI note, form help, results assembly block, and kerf guidance all state 90° assembly and that the coupon selects a prototype setting.

**Affected file/function:** `index.html` — `buildJointCouponModel()`, `renderDesigns()`, `designResultsHtml()`

**Physical consequence:** Coupons test orthogonal panel mating, not in-plane interlock.

**Fixture detection:** Topology, ID order, piece-count, and patterned-edge assertions.

---

## 4. Safety validation

### Verified rules

```text
requiredWeb = max(0.5 mm, materialThickness × 0.25)
actualSegmentWidth - abs(clearance) >= requiredWeb

minimumGripDepth = max(4 × materialThickness, 16 mm)
```

| Probe | Result |
|---|---|
| Default 9 mm segments at `±0.30` | Valid; zero overlaps/geometry errors on all 6 endpoint panels |
| Grip `15.99` mm at 3 mm thickness | Rejected |
| Grip `19.99` mm at 5 mm thickness | Rejected |
| Edge length `17` mm | Rejected (too short for three fingers) |
| Symmetric web collapse (`-6.00` clearance, 18 mm edge) | `buildJointCouponModel()` invalid |
| All 12 default panels | Closed orthogonal paths; no zero-length segments, self-intersections, collinear overlaps, duplicate local cuts, or bounds violations |

**Affected file/function:** `index.html` — `buildJointCouponModel()`, `designPanelGeometryErrors()`, `designPositiveCollinearOverlaps()`

**Consequence:** Collapsed fingers/recesses and malformed paths block export; signed extremes within declared range remain geometrically clean on default 45×9 pattern.

**Fixture detection:** Rejection fixtures plus circular geometry-diagnostic sweep on default output.

---

## 5. Labels and vector glyphs

### Verified

- Glyphs `0–9`, `.`, `-` exist in `designAssemblyGlyphs`; deterministic paths; no SVG `<text>`.
- Default labels: `A0.06|B0.06|…|A-0.02|B-0.02` (exact order fixture).
- Negative labels include vector `-` glyph runs.
- Labels inside polygons; `cutClearance >= 1` mm.
- Score group precedes cut group.
- Labels disabled → no `id="score"`, no `assembly-labels`; cut paths unchanged.
- Oversized label case omits some labels with warnings but retains valid cut SVG.

**Affected file/function:** `index.html` — `designAssemblyGlyphs`, `buildAssemblyLabelPaths()`, `buildJointCouponDesignResult()`

**Fixture detection:** Glyph, label-order, containment, clearance, disabled-label, and non-blocking omission assertions.

---

## 6. Layout and identity

### Default (6 clearances)

| Property | Live value |
|---|---|
| Pairs / pieces | 6 / 12 |
| Layout | **230 × 106** mm |
| Margin / gap | 10 mm |
| IDs | `joint-coupon-p01-a` … `joint-coupon-p06-b` (no `.`, `+`, spaces) |
| Cut groups | 12 separate `g#cut > g[id^="panel-joint-coupon-"]` |
| Translated duplicate cuts | 0 |
| Panel overlap | 0 |
| 10 mm path separation | Preserved |

### Signed endpoints (`-0.10, 0.00, 0.30`)

| Property | Live value |
|---|---|
| Pairs / pieces | 3 / 6 |
| Layout | **230 × 74** mm |
| Labels | 6 emitted |
| Geometry diagnostics | 0 errors per panel |

**Affected file/function:** `index.html` — `buildJointCouponDesignResult()`, `layoutDesignPanelRows()`

**Fixture detection:** Layout dimension, gap, overlap, ID, and serialization assertions.

---

## 7. Storage and compatibility

### Verified unchanged

- `STORAGE_KEY = 'genmitsu-l8-tracker-v1'`
- `SCHEMA_VERSION = 2`
- `persist()`, `backupObject()`, `freshState()`, `loadState()` bodies — no diff hunks
- `backupObject()` contains no `jointCoupon*` keys
- Coupon generation does not write `localStorage`
- `designDraft` JSON unchanged after `buildDesignResult()` (fixture + live check)

**Affected file/function:** `index.html` — session draft boundary (module-local `designDraft`)

**Fixture detection:** `localStorage` and backup isolation assertions.

---

## 8. Regression safety

### Byte-identical to `b1ba69e` (live compare)

| Template output | Match |
|---|---|
| Open Finger Box SVG | Yes (`2483` / `a892f91c`) |
| Loose-lid Finger Box SVG | Yes (`2615` / `6181bc75`) |
| Sliding Lid SVG | Yes (`2800` / `4a7ab718`) |
| One-row Drawer Cabinet SVG | Yes (`4436` / `7494c326`) |
| Three-row Drawer Cabinet SVG | Yes (`9124` / `b158a794`) |
| QR stand default SVG | Yes (`359` / `fe737a09`) |

Shared geometry functions are only **called** from new coupon code; their implementations are untouched in the diff.

**Fixture detection:** Consolidated regression signature assertion at end of Designs fixtures.

---

## 9. Fixture quality

### Counts (live Edge headless)

| Group | Passed | Failed | Δ from `b1ba69e` |
|---|---:|---:|---:|
| Designs | **515** | **0** | **+70** |
| Complete suite | **1047** | **0** | **+70** |

### Classification of ~70 new assertions

| Class | Count (approx.) | Examples |
|---|---:|---|
| **Independent** | ~22 | Literal transition coordinates; nominal boundary list; ID/layout/piece counts; draft nonmutation; storage/backup isolation; regression SVG signatures; malformed parser inputs |
| **Partially circular** | ~18 | Label text via `jointCouponLabelText()`; grip/web rejection through `buildDesignResult()` / `buildJointCouponModel()`; signed-endpoint layout via full pipeline |
| **Circular** | ~30 | Overlap/self-intersect/duplicate-cut sweeps calling production diagnostics; parser success cases; glyph checks via `designAssemblyWordGeometry()` |

**Gap:** No default Joint Fit Coupon full-SVG byte-stable signature fixture (unlike legacy templates). Layout and determinism are otherwise covered.

---

## 10. Scope and physical limitations

### Verified

- Drawer Cabinet lateral clearance default remains **`0.40` mm** in `designDefaults()`.
- UI warns that negative values are interference tests; stop on crush/split/lock.
- Copy states coupon selects next prototype setting; does not replace final box testing.
- Kerf remains separate (coupon kerf guidance + material-test note).

---

## Findings classification

### Blocker

None.

### Major

None.

### Minor

**M1 — Leading-zero tokens accepted**

- **Affected:** `parseJointCouponClearances()`
- **Explanation:** `00.04` matches the decimal regex and normalizes to `0.04`.
- **Consequence:** Cosmetic input leniency only; no geometry impact.
- **Correction:** Optional stricter regex if desired.
- **Fixtures:** Do not test leading zeros.

**M2 — No default coupon SVG signature fixture**

- **Affected:** `runDesignGeometryFixtures()`
- **Explanation:** New template lacks a `length/hash` baseline assertion like QR/tray templates.
- **Consequence:** Future serialization drift may escape detection until layout/validity fixtures fail.
- **Correction:** Add byte-stable signature after output stabilizes.
- **Fixtures:** Partially covered by determinism and preview/download identity checks.

**M3 — Geometry safety fixtures mirror production diagnostics**

- **Affected:** Overlap/self-intersect/duplicate-cut assertions (~6)
- **Explanation:** They invoke the same helpers they validate.
- **Consequence:** Broken diagnostics could pass while real panels fail; mitigated by independent transition-coordinate fixtures.
- **Correction:** Optional cross-check with external overlap probe on one literal panel.
- **Fixtures:** Transition fixtures provide partial independent coverage.

**M4 — Single-decimal tokens denote full millimeter magnitude**

- **Affected:** `parseJointCouponClearances()`
- **Explanation:** `0.6` → 60 hundredths (0.60 mm), rejected as out of coupon range.
- **Consequence:** Users typing one decimal place for sub-tenth values may see range errors rather than format errors.
- **Correction:** Optional UI example clarifying two-decimal requirement for sub-0.30 values.
- **Fixtures:** `whole-number token rejects` covers integers; not one-decimal ambiguity.

### Verified

| ID | Summary |
|---|---|
| V1 | Parser enforces fixed-decimal signed syntax, hundredths normalization, duplicates, count, and range. |
| V2 | Independent mating transitions match architecture spec for `±0.04` and zero. |
| V3 | Complementary phases, shared pattern, thickness depth, separation magnitude/direction. |
| V4 | One patterned + three plain edges; 90° joint semantics documented. |
| V5 | Grip-depth and signed-web rules enforced; endpoint clearances geometrically clean. |
| V6 | Vector numeric glyphs deterministic; labels score-before-cut; disabled labels omit score only. |
| V7 | Default `230×106` / endpoint `230×74` layouts; stable IDs; no overlap or duplicate cuts. |
| V8 | Session-only; no storage/backup pollution. |
| V9 | All pre-existing template signatures unchanged. |
| V10 | Designs **515/0**; complete suite **1047/0** (live). |
| V11 | Drawer lateral default **0.40** mm preserved. |
| V12 | Negative clearance limited to coupon; existing templates reject negatives. |

---

## Live verification method

- Engine: Microsoft Edge headless, Playwright `file:///` load of `index.html` with in-IIFE `window.__audit` export (read-only temp HTML, not repo files).
- Baseline comparison: `git show b1ba69e:index.html` for signature regression.
- Transition extraction: vertical top-edge segment x-coordinates from generated panel paths (independent of `designEdgePoints()`).

---

## Final verdict

**SAFE TO COMMIT**

The Joint Fit Coupon implementation is correctly bounded, physically meaningful, safely validated, session-isolated, and fully regression-clean against `b1ba69e`. Minor findings are fixture-hardening and input-clarity items only.