# Designs Tray Model Phase 1 — Focused Independent Audit

**Date:** 2026-07-17  
**Target:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `16128e0` — Add sliding lid box finished views  
**Primary implementation report:** `docs/DESIGNS_TRAY_MODEL_PHASE1_IMPLEMENTATION_2026-07-17.md`  
**Architecture review:** `docs/DESIGNS_TRAY_MODEL_EXTRACTION_ARCHITECTURE_REVIEW_2026-07-17.md`  
**Mode:** Read-only audit (no application edits; no git write operations)

---

## Pre-audit git snapshot

| Check | Result |
|---|---|
| `git status -sb` | `main...origin/main` at `16128e0`; modified `README.md`, `index.html`; pre-existing untracked docs |
| `git log -1 --oneline` | `16128e0 Add sliding lid box finished views` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --stat` vs `16128e0` | `README.md` ~4 lines; `index.html` **+173** (Tray model, serializer, fixtures, normalized `jointStyle`) |

Phase 1 is **working-tree only** relative to `16128e0`. Implementation-report claims were verified against source and isolated Edge `file://` runtime.

---

## Runtime baseline (fresh isolated Edge / Chromium `file://`)

Authoritative totals are **top-level fixture return values**. Designs geometry includes the Tray model suite once; Tray was also run standalone for the dedicated total and **not** double-counted into the complete suite sum.

| Group | Expected | Observed |
|---|---:|---:|
| Tray model fixtures (`runTrayModelFixtures`) | 222 / 0 | **222 / 0** |
| Designs geometry (`runDesignGeometryFixtures`, includes Tray once) | 850 / 0 | **850 / 0** |
| Designs production application | 118 / 0 | **118 / 0** |
| Test Grid machine identity | 18 / 0 | **18 / 0** |
| Evidence Promotion | 58 / 0 | **58 / 0** |
| Production Settings | 66 / 0 | **66 / 0** |
| Storage recovery | 8 / 0 | **8 / 0** |
| Complete suite (all top-level runners once; tray not double-added) | 1642 / 0 | **1642 / 0** |

### Complete suite breakdown (observed)

| Group | Passed | Failed |
|---|---:|---:|
| design (geometry, includes tray) | 850 | 0 |
| design-production | 118 | 0 |
| grid-machine | 18 | 0 |
| promotion | 58 | 0 |
| production | 66 | 0 |
| storage | 8 | 0 |
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| grid | 23 | 0 |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| metadata | 12 | 0 |
| project-wizard | 216 | 0 |
| **Total** | **1642** | **0** |

Independent probes outside fixtures: **45 / 45** passed (matrix byte equality, purity, orientation, divider linkage, jointStyle, production isolation, Finished View mode gates).

---

## Findings

### Verified — Phase boundary (shadow model only)

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildDesignResult` → `buildLegacyDesignResult` → `legacyDesignSvg` → `designTraySvg`; model only via `runTrayModelFixtures` |
| **Scenario** | Confirm production vs fixture routing |
| **Expected** | Live Dice/Divider still use `designTraySvg`; model/serializer not on preview/download; no Finished View, storage, or migration |
| **Observed** | `designTraySvg`, `legacyDesignSvg`, `buildLegacyDesignResult`, `buildDesignResult`, `downloadCurrentDesignSvg` bodies **SAME** vs `16128e0`. Live `buildDesignResult` SVG equals `designTraySvg`/`legacyDesignSvg`. `buildTrayModel` / `serializeTrayCompatibilitySvg` appear only in definitions + fixtures (no live call sites). No tray preview modes; dice/divider `designPreviewModeForTemplate` forces cut-layout for finished modes. No `tray-model-phase1` in `backupObject()` |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Implicit via production goldens + isolation asserts |

### Verified — Normalized `jointStyle`

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `normalizeDesignDraft` tray branch |
| **Scenario** | finger / tab-slot / invalid / missing; non-tray templates |
| **Expected** | Carry validated draft style into `values`; reject invalid; no silent default; no non-tray impact |
| **Observed** | Only change to `normalizeDesignDraft` vs baseline is `values.jointStyle = String(draft.jointStyle \|\| '')` for dice/divider. Validation still uses existing `['finger','tab-slot']` check and wording. Invalid `dovetail` and empty style produce errors and are **not** coerced. Finger Box and other templates do not receive forced tray jointStyle. Live SVG still reads `draft.jointStyle` in `designTraySvg` (unchanged) |
| **Consequence** | None for live output |
| **Recommended correction** | None |
| **Fixture coverage** | Invalid joint style parity case |

### Verified — Model purity

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildTrayModel` and helpers |
| **Scenario** | Source inspection + mutation/determinism probes |
| **Expected** | Normalized values only; no form/draft/storage/legacy SVG; pure deterministic; safe invalid return |
| **Observed** | No `designDraft`, `localStorage`, `persist`, `designTraySvg`, or SVG parsing in model path. Input JSON unchanged after build. Double build deep-equal. Invalid returns empty components/layout without throw. Helpers only use pure math + `designWallPath` / `designTabPositions` (shared geometry primitives also used by legacy) |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Determinism + localStorage non-write |

### Verified — Geometry equivalence (23 matrix cases)

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `legacyDesignSvg` vs `serializeTrayCompatibilitySvg(buildTrayModel(...))` |
| **Scenario** | Full Dice + Divider matrix listed in the implementation report |
| **Expected** | Byte-for-byte equality for every valid case; goldens 1726/`51a55721` and 1965/`a55dda6e` |
| **Observed** | Independent re-run of all 23 cases: **0 mismatches**. Default live SVGs match goldens. Fixtures assert dimensions, viewBox, element type/attributes/order, single anonymous red group `fill=none|stroke=#ff0000|stroke-width=0.1`, and no NaN/Infinity/undefined |
| **Hash function** | `designFixtureHash` is the historical non-cryptographic FNV-1a-style 32-bit hex fixture hash (same mechanism as other Designs geometry goldens), not SHA/MD5 |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | 23 × multi-assert matrix + golden pins |

### Verified — Compatibility serializer independence

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `serializeTrayCompatibilitySvg` |
| **Scenario** | Source + output contract |
| **Expected** | Project model layout only; no legacy generator call; anonymous red group; no IDs/titles/score/layers |
| **Observed** | Body is only `designSvgDocument(width, height, items.map(svg))` when valid, else `''`. No `designTraySvg`/`legacyDesignSvg`. Output has no `id=`, no `<title`/`<desc`, single red cut group via `designSvgDocument` (unchanged vs baseline) |
| **Consequence** | None — equality is dual-path, not delegation |
| **Recommended correction** | None |
| **Fixture coverage** | Byte identity + invalid empty SVG |

### Verified — Component semantics, orientation, dividers

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildTrayModel` components / orientation / metrics |
| **Scenario** | Dice default; Divider counts 1/2/6 |
| **Expected** | Named base/walls; Dice no dividers; Divider 1:1 slot linkage; formula `insideDepth*(i+1)/(n+1)`; orientation contract explicit; no invented retention geometry |
| **Observed** | Dice: one `tray-base`, walls `wall-front|back|left|right`, wall slots, dividerCount 0, compartmentCount 1, empty score/labels. Divider: slot/panel counts match, each panel `baseSlotId` matches corresponding slot, positions monotonic Front→Back with divider-01 nearest Front, span L→R, `retention:'unspecified'`, `intendedAssembly:'removable'`. Orientation object hard-codes IDs (`interiorFace:'top'`, wall IDs)—not derived from sheet x/y. Layout emission order matches legacy shape order so SVG bytes match without changing cut geometry meaning |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Semantic + formula + linkage asserts |

### Verified — Joint-style behavior (4 vs 2 tabs)

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `trayTabProfile` / model / legacy |
| **Scenario** | finger vs tab-slot |
| **Expected** | finger→4 tabs, tab-slot→2; 8–16 mm clamp; no true finger-joint algorithm |
| **Observed** | Profiles match legacy tab counts and clamping. Validation “too short for non-overlapping wall tabs” retained in both legacy `validateLegacyDesign` and `trayModelValidationErrors`. README correctly avoids claiming true finger-joinery for trays |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Matrix cases for both styles |

### Verified — Invalid-input parity

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `normalizeDesignDraft` + `buildLegacyDesignResult` vs `buildTrayModel` |
| **Scenario** | thickness, clearance, zero/neg dimensions, too-short, bad style, divider 0/7/1.5 |
| **Expected** | Both reject; model empty; UI wording unchanged; no new valid/invalid flips |
| **Observed** | Independent parity probe: all listed cases leave legacy and model invalid with empty model components. Fixture wording check: `Inside tray width must be at least 20 mm.` unchanged. Invalid trays remain non-downloadable via existing `valid` flag |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | 14 invalid parity cases + wording |

### Verified — Layout / material-box checks

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `trayModelMaterialBoxes` |
| **Scenario** | Dice and Divider layout overlap |
| **Expected** | Physical pieces only; slots not stock pieces; within bounds; match layout dims |
| **Observed** | Material IDs = base + four walls + divider panels only (slots excluded). Dice → 5 boxes; Divider n=6 → 11 boxes. Overlap uses axis-aligned bounding boxes via `designBoxesOverlap`. **Limitation:** AABB checks cannot detect interlocking non-rect path overlaps; walls are path-based but boxes use rectangular extents of path geometry metadata—adequate for the current non-overlapping sheet layout, not a full polygon collision proof |
| **Consequence** | Acceptable for Phase 1 layout confidence |
| **Recommended correction** | Optional future polygon checks if layout densifies |
| **Fixture coverage** | Non-overlap + bounds fixtures |

### Verified — Production isolation and protected boundaries

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | Storage / download / Finished Views |
| **Scenario** | Before/after Phase 1 comparison |
| **Expected** | No production SVG/record change; protected APIs stable |
| **Observed** | Live tray SVG goldens unchanged; panels remain anonymous `Shape N`; download path/body use `result.svg` from legacy; `STORAGE_KEY`/`SCHEMA_VERSION` unchanged; `persist`/`backupObject`/`replace`/`merge`/production normalizers SAME; `serializeDesignSvg` SAME; Finger Box / Sliding Lid / Drawer Cabinet preview mode gates still work; no model in backup |
| **Consequence** | None — production change would be Blocker |
| **Recommended correction** | None |
| **Fixture coverage** | Goldens + storage non-write |

### Verified — Documentation accuracy

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | README + implementation report |
| **Expected** | Internal-only model; legacy authoritative; byte fixtures; no switch/Finished View/migration; physical safety unverified |
| **Observed** | README states internal model, production remains `designTraySvg`, no Tray Finished View, suite **1642** / geometry **850** / tray **222**. Report matches runtime |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | N/A |

### Minor — `runTrayModelFixtures` not listed in README console API list

| Field | Detail |
|---|---|
| **Severity** | Minor |
| **Path** | README “Built-in checks” console function list |
| **Scenario** | Operator wants to run Tray suite alone |
| **Expected** | Optional discoverability of `runTrayModelFixtures()` |
| **Observed** | Function is on `window` and embedded in geometry suite; README list omits it (no dedicated `?selftest=` value either, by design) |
| **Consequence** | Low; `?selftest=design` still runs all 222 |
| **Recommended correction** | Optional README bullet when convenient |
| **Fixture coverage** | N/A |

### Minor — Large fixture matrix has repetitive structure

| Field | Detail |
|---|---|
| **Severity** | Minor |
| **Path** | `runTrayModelFixtures` matrix (23 × 8 asserts ≈ 184 of 222) |
| **Scenario** | Fixture quality review |
| **Expected** | High-value independent checks without pure inflation |
| **Observed** | Byte equality is **independent** (legacy path vs model serializer). Attribute/order/group asserts largely restate byte equality (partially redundant but diagnostic). Semantic/orientation/invalid/golden groups are high value and independent |
| **Consequence** | Count is large but important boundaries are covered |
| **Recommended correction** | None required for commit |
| **Fixture coverage** | Self |

---

## Fixture quality classification (summary)

| Group | Classification | Notes |
|---|---|---|
| Matrix byte-identical | **Independent** | Two calculation paths |
| Matrix element attribute JSON | Partially redundant with bytes | Still useful diagnostics |
| Dice/Divider semantics & orientation | **Independent** | Fixed expected IDs/formulas |
| Divider linkage & spacing | **Independent** | Explicit formula check |
| Invalid parity | **Independent** | Real `buildLegacyDesignResult` |
| Determinism / nonmutation | **Independent** | Deep JSON + snapshot |
| localStorage isolation | **Independent** | Observes storage key |
| Default goldens | **Independent** | Historical length/hash pins |
| Material non-overlap | **Independent** with AABB limits | Slots correctly excluded |

No circular “assert helper equals itself without external oracle” for production equality.

---

## Severity summary

| Severity | Count | Summary |
|---|---:|---|
| Blocker | 0 | No production/download/storage leakage |
| Major | 0 | No byte mismatch, validation flip, impure model, or serializer→legacy delegation |
| Minor | 2 | README console discoverability; matrix assert redundancy |
| Verified | 12 | Phase boundary through docs |

---

## Conclusion

**SAFE TO COMMIT**

Tray Model Phase 1 is a correctly bounded shadow verification: pure `buildTrayModel`, fixture-only compatibility serializer, full 23-case byte identity with the untouched `designTraySvg` production path, stable goldens, invalid-input parity, explicit orientation/divider semantics without inventing retention, and a green suite of **1642 / 0** (Tray **222 / 0**, Designs geometry **850 / 0**). No production switch or Finished View was introduced.
