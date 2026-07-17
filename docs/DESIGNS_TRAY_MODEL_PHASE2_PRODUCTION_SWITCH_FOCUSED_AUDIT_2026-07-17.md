# Designs Tray Model Phase 2 Production Switch — Focused Independent Audit

**Date:** 2026-07-17  
**Scope:** Read-only audit of Tray Model Phase 2 (live production switch) in `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `6ed566b` — Add structured tray model verification  
**Working tree under audit:** uncommitted changes to `index.html` and `README.md` (plus unrelated untracked docs/assets)  
**Primary implementation report:** `docs/DESIGNS_TRAY_MODEL_PHASE2_PRODUCTION_SWITCH_IMPLEMENTATION_2026-07-17.md`  
**Supporting docs:** architecture review, Phase 1 implementation, Phase 1 focused audit  

**Method:** Independent source inspection against `6ed566b`, full `git diff` review, Microsoft Edge (`channel=msedge`) `file://` Playwright runs of fixture runners, and separate probes (including a frozen baseline `designTraySvg` bundle re-executed only inside the auditor harness — not present in the app). Application files were not modified. No git write operations.

---

## Pre-audit git snapshot

| Check | Result |
| --- | --- |
| `git status -sb` | `main...origin/main`; modified `README.md`, `index.html`; many unrelated untracked docs/assets |
| `git log -1 --oneline` | `6ed566b Add structured tray model verification` |
| `HEAD` | `6ed566b790bb1535248abcb879571ab600ce7f77` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --stat` / `git diff --stat 6ed566b` | `README.md` 4 lines changed; `index.html` 107 lines changed (+45 / −66) |

Phase 2 is **uncommitted work on top of** the Phase 1 commit. The audit evaluates that working tree.

---

## Observed runtime totals (independent Edge `file://`)

Each top-level runner was executed on a fresh page. Tray results are **not** added on top of Designs geometry (Tray is nested once inside geometry).

| Runner | Observed | Expected |
| --- | --- | --- |
| `runTrayModelFixtures()` | **222 / 0** (total 222) | 222 / 0 |
| `runDesignGeometryFixtures()` | **850 / 0** | 850 / 0 |
| `runDesignProductionSettingsFixtures()` | **118 / 0** | 118 / 0 |
| `runTestGridMachineIdentityFixtures()` | **18 / 0** | 18 / 0 |
| `runEvidencePromotionFixtures()` | **58 / 0** | 58 / 0 |
| `runProductionSettingsFixtures()` | **66 / 0** | 66 / 0 |
| `runStorageRecoveryFixtures()` | **8 / 0** | 8 / 0 |
| Complete suite matching `?selftest=all` composition | **1642 / 0** | 1642 / 0 |

Complete-suite composition verified by summing the same groups `index.html` runs for `selftest=all` (baseline, normalization, production, promotion, design-production, grid, grid-machine, grid-browser, materials, library-browser, project-browser, metadata, storage, project-wizard, design). **No nested double-count of Tray.**

---

## Diff summary (intentional Phase 2 surface)

| Change | Assessment |
| --- | --- |
| **Removed** `designTraySvg()` | Confirmed: zero occurrences of `designTraySvg` in current `index.html` |
| **`legacyDesignSvg()`** | Tray branch removed; hanging-sign → `designHangingSignSvg`, else `designStandSvg` |
| **Added** `buildTrayDesignResult(normalized)` | Live tray result builder |
| **`buildDesignResult()`** | Routes only `dice-tray` / `divider-tray` → `buildTrayDesignResult` |
| **Fixtures** | Shadow old-vs-new comparison → pinned production goldens + live preview/download checks |
| **Added** `trayLivePreviewDownloadFixture(draft)` | Fixture-only DOM intercept of preview/download |
| **`buildTrayModel` / `serializeTrayCompatibilitySvg`** | Bodies **SAME** as baseline (sole geometry + serializer) |
| **README** | Phase 2 wording: model drives live production; no Finished View; no schema migration |

---

## 1. Live routing

### Verified

Live Dice/Divider path:

```text
normalizeDesignDraft(rawDraft)
  → buildTrayDesignResult(normalized)
      → validateLegacyDesign(normalized)
      → (if errors) empty non-downloadable result
      → buildTrayModel(normalized.values)
      → (if !model.valid) empty result
      → serializeTrayCompatibilitySvg(model)
      → designSvgValidation + anonymous Shape N panels
```

| Check | Result |
| --- | --- |
| Only `dice-tray` / `divider-tray` use `buildTrayDesignResult` | **Yes** |
| QR stand / Hanging sign still `buildLegacyDesignResult` | **Yes** |
| Finger Box / Sliding Lid / Drawer Cabinet / Joint Fit Coupon routes unchanged | **Yes** (`buildBoxDesignResult`, `buildSlidingLidDesignResult`, `buildDrawerCabinetDesignResult`, `buildJointCouponDesignResult`) |
| Model on live preview/download | **Yes** — `renderDesigns` / `downloadCurrentDesignSvg` → `buildDesignResult` → tray branch |
| Validation before serialization | **Yes** — `validateLegacyDesign` then model validity gate |
| Invalid trays non-downloadable | **Yes** — empty `svg`, download path returns without writing file |

### Finding — latent non-live footgun (Minor)

| Field | Detail |
| --- | --- |
| **Severity** | Minor |
| **Path** | `buildLegacyDesignResult` → `legacyDesignSvg` |
| **Scenario** | Direct call with a valid `dice-tray` / `divider-tray` normalized draft (not used by `buildDesignResult`) |
| **Expected** | Either reject trays or never accept tray templates on this path |
| **Observed** | Falls through to `designStandSvg`; probe produced QR-stand golden **359 / `fe737a09`**, not a tray |
| **Consequence** | No live UI/download impact; only a footgun if internal code later calls `buildLegacyDesignResult` with tray templates |
| **Recommended correction** | Optional guard: reject tray templates inside `buildLegacyDesignResult` / `legacyDesignSvg` |
| **Fixture coverage** | Not covered (live path correctly uses `buildTrayDesignResult`) |

---

## 2. Legacy generator removal

| Check | Result |
| --- | --- |
| `designTraySvg` in app source | **Absent** |
| Renamed second calculator | **Not found** — sole tray geometry is `buildTrayModel` + shared primitives `designTabPositions` / `designWallPath` / `designSvgDocument` |
| `legacyDesignSvg` tray dispatch | **Removed** |
| Live fallback to old generator | **None** |
| Frozen golden strings in fixtures | **Present and appropriate** (oracles, not a second engine) |

**Independent oracle:** Auditor re-executed the **exact** `designTraySvg` + helpers from commit `6ed566b` in an isolated eval bundle (not shipped in the app) and compared live `buildDesignResult` SVG for all **23** matrix cases: **23 / 23 byte-identical**.

---

## 3. Production SVG contract

### Default goldens (live `buildDesignResult`)

| Template | Length | `designFixtureHash` | Dimensions |
| --- | --- | --- | --- |
| Dice Tray | **1726** | **`51a55721`** | 421 × 307 mm, viewBox `0 0 421 307` |
| Divider Tray | **1965** | **`a55dda6e`** | 656 × 307 mm, viewBox `0 0 656 307` |

### Hash function

`designFixtureHash` is the historical **FNV-1a–style 32-bit** fixture hash (offset `2166136261`, prime `16777619`, hex padded to 8). **Not cryptographic.** Same mechanism as Phase 1 / Designs geometry goldens.

### Structure (defaults, independent parse)

| Property | Dice | Divider |
| --- | --- | --- |
| One anonymous red group | `fill=none`, `stroke=#ff0000`, `stroke-width=0.1` | same |
| Element count | 21 | 25 |
| IDs / title / desc / score | **Absent** | **Absent** |
| Named LightBurn layers | **None** | **None** |
| Live SVG ≡ serializer | **Yes** | **Yes** |

No default production-byte drift vs Phase 1 pinned constants.

---

## 4. Full 23-case production matrix

Independent comparison of live production SVG vs frozen baseline `designTraySvg`: **23 passed / 0 failed**.

In-app fixtures also pin length, hash, width/height, element count, element-order/attribute signature hash, red group, no NaN/Infinity/undefined, and live preview/download/filename/MIME for each case — all green inside **222 / 0**.

Sample independent UI intercept (Edge):

| Case | Preview = result | Download = result | Filename | MIME | Hash |
| --- | --- | --- | --- | --- | --- |
| Dice default Finger | Yes | Yes | `l8-dice-tray-2026-07-17.svg` | `image/svg+xml;charset=utf-8` | `51a55721` |
| Divider default count 2 | Yes | Yes | `l8-divider-tray-2026-07-17.svg` | same | `a55dda6e` |
| Dice Tab-and-slot | Yes | Yes | (same naming pattern) | same | `41697123` / 1054 chars |
| Dice invalid width 0 | Download blocked | empty | `filename` empty | — | — |

---

## 5. Result contract (`buildTrayDesignResult`)

| Property | Observed |
| --- | --- |
| `valid` / `errors` / `warnings` | Same shape as former legacy result path |
| `widthMm` / `heightMm` | From SVG validation against model layout |
| `metrics.template` | `dice-tray` / `divider-tray` only |
| Panels | Anonymous `Shape N` only (no wall/divider names in UI panels) |
| Semantic model | Ephemeral; not on result object beyond SVG/panels/metrics.template |
| Invalid | Empty SVG, `valid:false` |

---

## 6. Validation parity

`normalizeDesignDraft` body **SAME** as baseline. `validateLegacyDesign` body **SAME**. Model validation remains a secondary gate with parallel rules.

Independent invalid matrix (live + model): all rejected, empty SVG, no components for thickness 0, negative clearance, zero/negative width/depth/height, too-short edges, invalid joint style, divider 0/7/fractional.

| Check | Result |
| --- | --- |
| UI wording for zero width | Unchanged: `Inside tray width must be at least 20 mm.` |
| Download blocked when invalid | **Yes** |
| No valid→invalid or invalid→valid flip observed | **Yes** |

Note: divider count `0` still surfaces the pre-existing awkward message path via numeric minimum helper (`… at least 1 mm`); **not introduced by Phase 2** (`normalizeDesignDraft` unchanged).

---

## 7. Preview and download workflow

Exercised via `trayLivePreviewDownloadFixture` / `downloadCurrentDesignSvg` intercept (same code path as Designs download):

- Dice default Finger: preview bytes = download bytes = `buildDesignResult.svg`
- Divider default: same identity
- Joint-style and dimension changes update result (matrix + tab-slot probe)
- Invalid dimensions: no download payload
- Preview mode for trays: `designPreviewModeForTemplate('dice-tray'|'divider-tray', finished-*)` → **`cut-layout` only** (no Finished View UI)

Template switching / non-tray Finished Views remain covered by existing Designs geometry fixtures (**850 / 0**).

---

## 8. Filename and MIME preservation

| Item | Observed |
| --- | --- |
| Pattern | `l8-{fileSlug(template)}-{today()}.svg` |
| Dice | `l8-dice-tray-YYYY-MM-DD.svg` |
| Divider | `l8-divider-tray-YYYY-MM-DD.svg` |
| MIME | `image/svg+xml;charset=utf-8` |
| `downloadCurrentDesignSvg` vs `6ed566b` | **SAME** body |

---

## 9. Model and serializer integrity

| Check | Result |
| --- | --- |
| `buildTrayModel` pure (no form / draft / storage / persist) | **Yes** — body unchanged from Phase 1 |
| Input nonmutation | **Yes** (fixtures + probe) |
| Deterministic | **Yes** |
| No machine/preview/session dependency | **Yes** |
| Invalid → safe empty model, no throw | **Yes** |
| `serializeTrayCompatibilitySvg` | Only `designSvgDocument(layout)` from layout item SVG fragments; no legacy generator; empty when invalid |
| Sole production tray serializer | **Yes** |

Orientation / divider linkage (probe, count 3): explicit `orientation` contract; positions `insideDepth * (i+1)/(count+1)`; one-to-one `baseSlotId` linkage; material IDs exclude slot cutouts.

---

## 10. Fixture transition quality

### Assertion inventory (~222)

- **23 × 8 matrix assertions** ≈ 184 (live validity, pinned bytes, dims/viewBox, element signature, red group, numeric tokens, preview/download/filename, source nonmutation)
- Semantic Dice/Divider, invalid parity, storage isolation, default goldens ≈ 38

### Classification of important groups

| Group | Class | Notes |
| --- | --- | --- |
| Pinned length + FNV hash vs historical constants | **Independent** | Defaults match Phase 1 goldens; full matrix independently reconfirmed vs frozen `designTraySvg` |
| Element-order/attribute signature hashes | **Partially circular** | New pins of the same serializer output; still useful regression guards; independence restored by full-SVG hash + external legacy oracle |
| `result.svg === serializeTrayCompatibilitySvg(model)` | **Partially circular** | Confirms live path uses serializer; not a second engine |
| Live `buildDesignResult` + `trayLivePreviewDownloadFixture` | **Independent** | Real download intercept + preview object decode |
| Invalid parity via `validateLegacyDesign` + model + live | **Independent** | Real validators |
| Semantic orientation / linkage / material boxes | **Independent** | Fixed expectations / structural checks |
| Material non-overlap | **Bounding-box only** | Does not detect interlocking non-AABB overlap; slots excluded via `materialComponentIds` |

No fixture calls removed `designTraySvg`. No second executable tray calculator remains in-app.

### Finding — partial circularity after engine removal (Minor / expected)

| Field | Detail |
| --- | --- |
| **Severity** | Minor (expected Phase 2 tradeoff; mitigated) |
| **Path** | `runTrayModelFixtures` matrix |
| **Scenario** | After deleting `designTraySvg`, fixtures cannot compare two live engines |
| **Expected** | Fixed historical oracles + live path checks |
| **Observed** | Pinned goldens + live/download checks; auditor-only frozen legacy confirms 23/23 |
| **Consequence** | In-repo tests alone slightly overstate “independent engine” equality; risk is low given pinned full-SVG hashes |
| **Recommended correction** | Optional: store full default SVG fixtures as files; not required for commit |
| **Fixture coverage** | Defaults + matrix hashes |

---

## 11. Persistence and record isolation

| Boundary | Result |
| --- | --- |
| `STORAGE_KEY` | **SAME** `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | **SAME** `2` |
| `persist` / `backupObject` / `replaceData` / `mergeData` | **SAME** vs `6ed566b` |
| Model in localStorage after `buildTrayModel` | **No** |
| Model strings in `backupObject()` | **No** (`tray-model-phase1` / wall IDs not in backup) |
| Migration / backfill | **None** |
| Saved Designs / Projects / Production Settings / Material Tests / Test Grids schema | Unchanged (model not stored) |

Drafts still rebuild model from existing form fields on each `buildDesignResult`.

---

## 12. Existing feature regressions

| Feature | Evidence |
| --- | --- |
| Finger Box cut + Finished View plumbing | `buildBoxDesignResult` **SAME**; geometry suite green; probe valid + dimensions |
| Sliding Lid cut + Finished Closed/Open | `buildSlidingLidDesignResult` **SAME**; probe valid + `slidingFinishedView` |
| Drawer Cabinet + Finished Front | `buildDrawerCabinetDesignResult` **SAME**; probe valid |
| Joint Fit Coupon / QR / Hanging | Legacy/box routes intact; QR/hanging default goldens still **359/`fe737a09`**, **341/`656e633d`** |
| Production / promotion / grid machine / storage | Top-level runners green |
| Tray Finished View | **Not added**; preview selector only for finger/sliding/cabinet |

---

## 13. Protected boundaries vs `6ed566b`

| Symbol | Status |
| --- | --- |
| `STORAGE_KEY`, `SCHEMA_VERSION` | SAME |
| `persist`, `backupObject`, `replaceData`, `mergeData` | SAME |
| `normalizeProductionEvidence`, `normalizeProductionSetting`, `normalizeProductionSettings` | SAME |
| `downloadCurrentDesignSvg`, `serializeDesignSvg` | SAME |
| `validateLegacyDesign`, `normalizeDesignDraft` | SAME |
| `buildLegacyDesignResult` | SAME body (tray no longer reaches it via `buildDesignResult`) |
| `legacyDesignSvg` | **DIFF** — intentional tray removal |
| `buildDesignResult` | **DIFF** — intentional tray branch |
| `buildTrayModel`, `serializeTrayCompatibilitySvg` | SAME |
| `designSvgDocument`, stand/hanging generators | SAME |
| `designPreviewModeForTemplate` | SAME |
| Non-tray geometry builders | SAME (encoding-safe compare) |

Only production-adjacent intentional changes: remove `designTraySvg`, tray routing switch, fixtures, README.

---

## 14. Documentation accuracy

| Claim | Accurate? |
| --- | --- |
| Structured model drives live production SVG | **Yes** |
| Legacy executable tray generator removed | **Yes** |
| Production output remains byte-compatible | **Yes** (23/23 vs frozen baseline; default goldens hold) |
| No Tray Finished View | **Yes** |
| No storage/schema migration | **Yes** |
| Fit / assembly / laser safety unverified | **Yes** (README dry-fit language retained) |
| Suite 1642 / geometry 850 / tray 222 | **Yes** (observed) |
| `runTrayModelFixtures` listed in console API list | **Yes** (fixed vs earlier Phase 1 README gap) |

Implementation report claims match independent runtime results.

---

## Findings register

### Verified (selected)

1. **Live routing** — only dice/divider through `buildTrayDesignResult`; validation then model then serializer.  
2. **Single tray geometry engine** — `designTraySvg` removed; no executable duplicate.  
3. **Production bytes** — defaults **1726/`51a55721`**, **1965/`a55dda6e`**; full matrix matches frozen Phase 1 generator.  
4. **Download/filename/MIME** — unchanged path; identity with preview.  
5. **Result UI contract** — anonymous `Shape N`; no semantic names leaked.  
6. **Validation parity** — invalid cases reject; wording preserved for zero width.  
7. **Purity / storage isolation** — model pure; no backup/localStorage leak.  
8. **No Tray Finished View**; other Finished Views intact.  
9. **Protected storage/schema/download** functions unchanged except intentional tray routing.  
10. **Runtime suite** — 222 / 850 / 118 / 18 / 58 / 66 / 8 / **1642** all **0 failed**.

### Minor

1. **`buildLegacyDesignResult` tray fallthrough** produces stand SVG if mis-invoked (not live).  
2. **Fixture partial circularity** on serializer self-equality / element signature pins (mitigated by full-SVG pins + external oracle).  
3. **`generatorVersion: 'tray-model-phase1'`** string still used after Phase 2 (cosmetic/internal only).

### Major

None.

### Blocker

None.

---

## Conclusion

**SAFE TO COMMIT**

Phase 2 correctly switches Dice Tray and Divider Tray live preview/download to `buildTrayModel` → `serializeTrayCompatibilitySvg` while removing the executable legacy tray generator, preserving the anonymous red-cut SVG contract byte-for-byte (independently reconfirmed against frozen `6ed566b` `designTraySvg` for all 23 cases), leaving storage/schema and non-tray templates untouched, and keeping the full fixture suite green at **1642 / 0**.
