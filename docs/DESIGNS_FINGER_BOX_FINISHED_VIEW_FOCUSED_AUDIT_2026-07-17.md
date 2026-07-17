# Finger-jointed Box Finished View — Focused Independent Audit

**Date:** 2026-07-17  
**Target:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `c46fb40` — Record machine identity on test grids  
**Primary implementation report:** `docs/DESIGNS_FINGER_BOX_FINISHED_VIEW_IMPLEMENTATION_2026-07-17.md`  
**Architecture review:** `docs/DESIGNS_FINISHED_VIEWS_ARCHITECTURE_REVIEW_2026-07-17.md`  
**Mode:** Read-only audit (no application edits; no git write operations)

---

## Pre-audit git snapshot

| Check | Result |
|---|---|
| `git status -sb` | `main...origin/main` at `c46fb40`; modified `README.md`, `index.html`; many pre-existing untracked docs |
| `git log -1 --oneline` | `c46fb40 Record machine identity on test grids` |
| `git diff --check` | Clean (CRLF warnings only) |
| `git diff --stat` vs `c46fb40` | `README.md` 4 lines; `index.html` +54 / −10 net (Finished View + fixtures + metrics fields) |

The Finger Box Finished View lives in the **working tree** relative to `c46fb40`. Claims in the implementation report were verified against source and isolated Edge `file://` runtime, not accepted at face value.

---

## Runtime baseline (fresh isolated Edge / Chromium `file://`)

Self-tests report via function return values (console groups). Authoritative totals are **one call per top-level runner**, not nested console sums.

| Run | Observed |
|---|---|
| `index.html` (bare) | Loads; Designs available |
| Designs geometry (`runDesignGeometryFixtures`) | **605 passed / 0 failed** |
| Designs production application | **118 / 0** |
| Evidence promotion | **58 / 0** |
| Production settings | **66 / 0** |
| Test Grid machine identity | **18 / 0** |
| Complete suite (15 top-level runners once) | **1397 passed / 0 failed** |

### Complete suite breakdown (observed)

| Group | Passed | Failed |
|---|---:|---:|
| grid-machine | 18 | 0 |
| promotion | 58 | 0 |
| production | 66 | 0 |
| design-production | 118 | 0 |
| design (geometry) | **605** | 0 |
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| grid | 23 | 0 |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| metadata | 12 | 0 |
| storage | 8 | 0 |
| project-wizard | 216 | 0 |
| **Total** | **1397** | **0** |

Matches expected **1397 / 0**, geometry **605 / 0** (587 baseline geometry + 18 new Finger Box Finished View assertions).

Independent probes of renderer, production isolation, mode gates, shapes, and backup: **53 / 53** Finger Box–relevant checks passed (two Drawer Cabinet draft-construction probe failures were auditor harness mistakes using `formDefaults('design')` instead of `designDefaults()`; Drawer Cabinet geometry fixtures in the 605 suite all passed).

---

## Diff scope vs `c46fb40` (source)

Additive / localized only:

1. `buildBoxDesignResult` metrics gain `boxLid` and `materialThickness` from normalized draft values (no geometry change).
2. New `buildFingerBoxFinishedViewSvg(result)` — pure screen SVG from `result.metrics`.
3. `designPreviewModeForTemplate` / selector / results / bind handlers accept `finished-view` for `finger-box` alongside existing `finished-front` for Drawer Cabinet.
4. Finger-specific assembly message when `designDraft.template === 'finger-box'`.
5. Eighteen new assertions inside `runDesignGeometryFixtures()`.
6. README Finished View wording + suite total **1397** / geometry **605**.

---

## Findings

### Verified — Authoritative-model boundary

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildFingerBoxFinishedViewSvg` ← `result.metrics.dimensions` / `boxLid` / `materialThickness` from `buildBoxDesignResult` ← `buildBoxModel` |
| **Scenario** | Inspect renderer source and result path for open-top and loose-lid boxes |
| **Expected** | No production SVG parsing; no path-string geometry rebuild; no duplicated finger joinery; no altered geometry-builder calls; no model/draft mutation; only minimal additive result metadata |
| **Observed** | Renderer reads only `result.metrics` numbers and `boxLid`; does not call `buildBoxModel`/`buildFingerPanel`; does not touch `result.svg`; does not parse paths. `buildBoxDesignResult` only adds `boxLid` + `materialThickness` to metrics. Production panel paths and SVG generation path unchanged. Draft JSON unchanged after Finished View generation |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Nonmutation + production absence + dimension match fixtures |

### Verified — Preview-mode behavior

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `designPreviewMode`, `designPreviewModeForTemplate`, `designPreviewSelectorHtml`, `bindDesignPreviewActions`, `resetDesignWorkspaceSession` |
| **Scenario** | Finger Box Cut Layout default; switch Finished View; switch templates; invalid dimensions |
| **Expected** | Session-only mode; not in backup; template gate; invalid reset; no draft/result/SVG mutation; compact accessible selector |
| **Observed** | Default `cut-layout`; `finished-view` only when template is `finger-box`; Drawer Cabinet still uses `finished-front`; foreign modes normalize to cut-layout; invalid result forces mode to cut-layout and disables Finished button; `backupObject()` has no preview mode; selector uses `role="group"` + `aria-pressed` buttons |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Selector, invalidation reset, template-switch isolation covered |

### Verified — Open-top rendering

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildFingerBoxFinishedViewSvg` for `boxLid !== 'loose'` |
| **Scenario** | Valid open-top Finger Box |
| **Expected** | Four walls, cavity/open top, outside W/D/H, Front/Back/Left/Right, screen-only notice; no lid/hinge/groove/rail/retention/sliding/decorative joints/fabricated kerf |
| **Observed** | `#finger-box-cavity`, four `.finger-box-wall` rects, labels Front/Back/Left/Right, `Open top`, outside dimension texts; no loose-lid group; no hinge/rail/retention/slide id/class tokens; restrained top-down schematic (not 3D). Shallow/tall/square/long-narrow/oversized cases remain finite SVG without NaN/Infinity |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Orientation, open-top/no-lid, dimensions, extreme finite SVG |

### Verified — Loose-lid rendering

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | Finished View when `metrics.boxLid === 'loose'`; production lid panel from model |
| **Scenario** | Finger Box with loose flat lid |
| **Expected** | Exactly one separate flat lid labeled `Loose lid`; dimensions from model; not hinged/sliding/retained/merged; production cut unchanged when toggling preview |
| **Observed** | Single `#finger-box-loose-lid` with rect sized to outside width × depth (matches model lid panel W×D); label `Loose lid`; caption that lid is separate; production SVG has no Finished View / “Loose lid” screen text; mode switch does not alter `result.svg` |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Separate lid + no mechanical geometry |

### Verified — Dimension and orientation honesty

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | Wall geometry attributes vs `metrics.dimensions` / `materialThickness` |
| **Scenario** | Standard inside 120×90×50 / t=3 → outside 126×96×53 |
| **Expected** | Labels and wall extents from model; stable Front/Back; no viewport-derived dimensions; no NaN/Infinity |
| **Observed** | Front wall `width === outsideWidth` (126); wall band thickness equals material thickness (3) when not clamped; height shown as text from `outsideHeight`; Front y &gt; Back y consistently; repeated renders identical. Visual wall uses `Math.min(thickness, width/3, depth/3)` only as a **display** band clamp for extreme aspect ratios—not as a recalculated box dimension |
| **Consequence** | None for honesty of labeled outside sizes |
| **Recommended correction** | None required |
| **Fixture coverage** | Width match, wall thickness for fixture case, viewBox finite |

### Verified — Determinism

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildFingerBoxFinishedViewSvg` pure function of result metrics |
| **Scenario** | Same model rendered twice |
| **Expected** | Identical SVG; stable viewBox/IDs/labels/lid placement; no random/session/machine dependence |
| **Observed** | Byte-identical SVG on double call; fixed IDs (`finger-box-wall-front`, etc.); layout math from dimensions only |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Determinism assertion |

### Verified — Production isolation (critical)

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `downloadCurrentDesignSvg` → `result.svg` via `serializeDesignSvg`; preview injects Finished View only in `designResultsHtml` |
| **Scenario** | Capture draft/model/result SVG/panels before and after Finished View mode; download while Finished View selected |
| **Expected** | Production cut SVG byte-stable; download is Cut Layout; no Finished View elements in download; LightBurn red cut / blue score layers unchanged |
| **Observed** | `result.svg` free of `finger-box-finished-view`, `finger-box-assembled`, screen labels; Finished View appears only in results markup; download path still passes `result.svg` only; `serializeDesignSvg` body **unchanged** vs `c46fb40`; panel count/geometry signature stable across mode use |
| **Consequence** | None — Finished View **does not** enter production SVG (would be Blocker if it did) |
| **Recommended correction** | None |
| **Fixture coverage** | Production absence + screen-only markup fixtures; Drawer Cabinet download isolation fixtures still present |

### Verified — Invalid and boundary dimensions

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | `buildDesignResult` validation + Finished View early return |
| **Scenario** | Extreme valid boxes; `boxWidth: 0` invalid |
| **Expected** | Valid renders finite; invalid does not render Finished View; no negative cavity math from unusable data; validation messages authoritative |
| **Observed** | Invalid result `valid:false` and `buildFingerBoxFinishedViewSvg` returns `''`; selector disables Finished View and resets mode; extreme valid outside 200×160×120 remains finite |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Extreme + invalid handling |

### Verified — Drawer Cabinet regression

| Field | Detail |
|---|---|
| **Severity** | Verified |
| **Path** | Existing Finished Front View + geometry fixtures |
| **Scenario** | Mode gates and Drawer Cabinet fixtures after Finger Box mode expansion |
| **Expected** | Cut Layout \| Finished Front View; front elevation; download isolation; existing fixtures green |
| **Observed** | Mode keys remain distinct (`finished-front` vs `finished-view`); `buildDrawerCabinetModel` / `buildDrawerCabinetDesignResult` bodies **SAME** vs `c46fb40`; full geometry suite 605/0 includes all prior Drawer Front fixtures |
| **Consequence** | None |
| **Recommended correction** | None |
| **Fixture coverage** | Full prior Drawer Front suite retained |

### Verified — Protected boundaries vs `c46fb40`

| Symbol | Status |
|---|---|
| `STORAGE_KEY` / `SCHEMA_VERSION` | Unchanged |
| `persist` / `backupObject` / `replaceData` / `mergeData` | SAME |
| `normalizeProductionEvidence` / `Setting` / `Settings` | SAME |
| `buildFingerPanel` | SAME |
| `buildSlidingLidBoxModel` | SAME |
| `buildDrawerCabinetModel` / `buildDrawerCabinetDesignResult` | SAME |
| `serializeDesignSvg` | SAME |
| `downloadTextFile` | SAME |
| `buildDesignResult` routing | SAME (still dispatches to existing builders) |
| `buildBoxDesignResult` | **DIFF** — additive metrics only (`boxLid`, `materialThickness`); layout/SVG path unchanged |
| Material Test / Test Grid / Project schemas | Untouched by this diff |
| Persistence of preview mode | None |

### Minor — Fixture name slightly overstates wall-thickness check

| Field | Detail |
|---|---|
| **Severity** | Minor |
| **Path** | `runDesignGeometryFixtures` — “Finished View wall thickness is finite and positive” |
| **Scenario** | Assertion body |
| **Expected** | Name matches check |
| **Observed** | Asserts front wall height `=== 3` (fixture material thickness), not a general “finite and positive for any thickness” property. Still a valid independent check for the standard fixture box |
| **Consequence** | Noncritical documentation of coverage |
| **Recommended correction** | Optional rename to “Finished View wall band matches fixture material thickness” |
| **Fixture coverage** | Itself |

### Minor — Left/Right label crowding on very thin wall bands

| Field | Detail |
|---|---|
| **Severity** | Minor |
| **Path** | Wall label placement in `buildFingerBoxFinishedViewSvg` |
| **Scenario** | Very small outside dimensions or thick material relative to plan (wall clamp still leaves narrow bands) |
| **Expected** | Labels readable where practical |
| **Observed** | Labels rotate into wall bands; may crowd on extreme aspect ratios. Outside dimension texts and Front/Back remain clear. Not a wrong-dimension issue |
| **Consequence** | Cosmetic / readability only |
| **Recommended correction** | Optional future label offset outside wall band |
| **Fixture coverage** | Not required for commit |

### Visual review (section 12)

Isolated Edge rendering of the Finished View SVG (open-top and loose-lid) was generated from the authoritative result path:

- **Open top:** Top-down cavity with four wall bands; Front at bottom edge, Back opposite; Left/Right side labels; “Open top” and outside W/D/H text; muted “Screen-only assembly preview…” notice in results chrome; restrained 2D schematic—no false 3D, no kerf/gap fabrication.
- **Loose lid:** Same box plus a single separate flat panel to the right labeled “Loose lid,” with explicit “no hinge or retention” caption; not merged into the body.
- **Honesty:** Does not claim fit, strength, kerf, glue, or production success (notice + assembly copy).

Full-app screenshot automation did not reliably scroll the Designs result pane in headless mode; visual conclusions rely on direct SVG isolation plus DOM/text probes rather than full UI chrome captures.

---

## Fixture quality (18 new assertions)

| # | Name | Classification | Notes |
|---:|---|---|---|
| 1 | Finger Box Finished View is deterministic | Independent | Double-call equality |
| 2 | … is a finite SVG document | Independent | Parse + NaN scan + title/desc |
| 3 | … shows all four orientation labels | Independent | Text content |
| 4 | … shows open top and outside dimensions | Independent | Hard-coded expected mm for fixture model |
| 5 | Open-top … cavity and no lid | Independent | DOM structure |
| 6 | Loose-lid … keeps the lid separate | Independent | |
| 7 | Loose-lid … no hinge rail retention or sliding | Independent | id/class token scan (not text heuristics) |
| 8 | … width matches authoritative outside width | Independent | |
| 9 | … wall thickness is finite and positive | Partially overstated name | Hard-coded `=== 3` |
| 10 | … viewBox is finite | Independent | |
| 11 | … does not mutate the draft | Independent | |
| 12 | … absent from production SVG | Independent | Critical isolation |
| 13 | Extreme valid … remains finite | Independent | |
| 14 | Invalid … cannot render Finished View | Independent | |
| 15 | Selector exposes Finished View beside Cut Layout | Independent | Markup + aria-pressed |
| 16 | … screen-only in results markup | Independent | |
| 17 | Invalid dimensions reset … to Cut Layout | Independent | |
| 18 | Template switching cannot carry Finished View | Independent | Mode gate |

No circular “helper equals itself without observing DOM/result” checks. Coverage matches implementation report intent; download byte-for-byte while Finished View is selected is implied by production SVG isolation + unchanged `downloadCurrentDesignSvg` path (stronger explicit download fixture exists for Drawer Cabinet, not duplicated for Finger Box—acceptable given shared download function).

---

## Documentation accuracy (README)

README states Finger-jointed box **Cut Layout | Finished View**; screen-only; open-top and loose-lid; download remains cut SVG; does not prove fit/strength/kerf/safety; no hinge/rail/retention/sliding claims; geometry suite **605**; complete suite **1397**. Matches implementation and runtime.

Architecture review P1-A recommendations are followed: top-down assembled view, model-driven dimensions, separate loose lid, template-scoped mode, screen-only notice.

---

## Severity summary

| Severity | Count | Summary |
|---|---:|---|
| Blocker | 0 | No Finished View in production SVG/download; no draft/geometry mutation; no record persistence |
| Major | 0 | Dimensions/orientation/lid semantics honest; renderer stable |
| Minor | 2 | Fixture wall-thickness name; possible label crowding on extremes |
| Verified | 11 | Boundary, modes, open/loose, dimensions, determinism, production isolation, invalid, drawer regression, protected APIs, docs |

---

## Conclusion

**SAFE TO COMMIT**

The Finger-jointed Box Finished View is a correctly bounded screen-only preview: it consumes additive `boxLid` / `materialThickness` and existing model dimensions, does not parse or alter production geometry, keeps downloads on the Cut Layout SVG, isolates mode state from backup/storage, preserves Drawer Cabinet Finished Front View behavior, and ships with **18** green geometry fixtures inside a full suite of **1397 / 0**.
