# Finished Views and Practical Product Generators — Architecture, Geometry, and Product-Workflow Review

Date: 2026-07-23
Repository: `C:\Genmitsu L8 Tracker`
Reviewer role: read-only architecture, geometry, and product-workflow review (Claude Sonnet 5)
Product files modified by this review: **none**
Authorized output only: this report

---

## Executive summary

This review's single most important finding is that **the app already implements far more of the requested "Finished Views and product generators" work than the phase name suggests.** Direct source inspection (not prior reports) confirms: 12 of the 15 existing Designs templates already have a working, fixture-tested, screen-only Finished View (Finger Box, Gift Box, Sliding-Lid Box, Dice Tray, Divider Tray, both Concealed Cleat prototypes, and all four Tabletop Accessories templates — T1 corner/floor coupon, T2 rectangular shell, T3A storage-fit coupon, and **T3B Tabletop Storage Tray, which already has a full assembled-with-removable-false-bottom Finished View**). Drawer Cabinet has its own dedicated Finished Front View. Only QR Stand, Hanging Sign, and Joint Fit Coupon lack one — and inspection shows none of the three would gain real assembly comprehension from one (a stand/sign's assembly is legible directly from its flat cut layout, and a coupon is a test tool, not a product).

Equally significant: the existing **`dice-tray`** template already implements almost every feature Candidate A ("Parametric Dice Tray") asks for — a full-depth storage bay with a fixed divider, an optional rolling-surface insert with cork/felt/leather/generic-liner options explicitly treated as **measurement-only** (never cut, for material-safety reasons), an optional wood insert panel, an optional bottom cover, two wall-to-base joint styles, and a Finished View that visually distinguishes all of the above. It uses a simpler **tab-and-slot** wall-to-base joint, explicitly documented in its own UI as "not corner finger joinery."

Separately, **T3B (Tabletop Storage Tray)** already implements the physically-validated, finger-jointed, closed-three-way-corner shell plus a removable false-bottom panel resting on four glued rails — the exact construction underlying Joe's real physical evidence (80×60×25 mm interior, 2.88 mm measured 3 mm basswood, −0.075 mm clearance, 9 mm finger width, Genmitsu L8 20W). T3B is composed cleanly from two already-existing, independently-tested building blocks (`buildTabletopRectangularShellDesignResult` for the shell, and the T3A rail/false-bottom pattern), which is the strongest possible evidence that this composition pattern is safe to reuse again.

**Recommended phase (Option B): exactly one new generator, no existing-generator Finished View additions.** The new generator — working name **"Stackable Token & Dice Tray"** — reuses the T2 shell-building function and the T3A rail/false-bottom pattern verbatim (calling existing functions with new parameters, not modifying them), at a smaller footprint suitable for dice, tokens, and small game accessories, adding exactly one new, safety-conscious feature: an optional cork/felt/leather liner **allowance** for the play surface, modeled directly on `dice-tray`'s already-proven "measurement-only, never cut" liner pattern. A stacking lip is explicitly evaluated and **deferred** — it is the one place a truly new geometric feature would be required, and this review recommends against introducing new geometry in the same phase as a new generator. This keeps the phase's actual diff to "new template registration, two new composed builder functions calling existing helpers, one new Finished View function copying an existing rendering pattern, and new fixtures" — nothing shared is modified, so every existing generator's production bytes are provably unaffected by construction, not merely by testing.

---

## 1. Actual HEAD and working-tree state

| Item | Observed |
| --- | --- |
| HEAD | `d1a06e78e7f8ef56d0ba7152f997e9c879ea1034` |
| Subject | `Clarify focus and Z offset terminology` |
| Branch | `main` |
| Upstream | `main...origin/main`, **0 / 0** (synchronized) |
| Tracked working-tree diff | **None** — `git diff --stat` and `git diff --check` both empty |
| Staged changes | None |
| Untracked files | All pre-existing (historical `docs/*.md` reports, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`) — none related to Designs, none touched by this review |

The stated baseline matches the actual repository exactly. Recent commit history (`d1a06e7` → `994510b` → `a4d4c06` → `3aa314a` → `386e1e0` → `8c06800` → `19d4834` → `1d2d860` → `c16324b`) confirms GUI Clarity, Inventory Organization, and the full Security S1–S3 sequence are committed, and that T3B's "Workshop-ready" graduation and its complete-tray evidence workflow are the most recent Designs-family work before this phase — consistent with the stated project sequence.

---

## 2. Files, functions, and reports inspected

Single-file application, `index.html` (15,000+ lines) — no other product file. Inspected via `Read`/`Grep` only, no edits.

**Reports read for context (not relied on for current-state claims without re-verifying against source):** the project's own summarized history of `docs/DESIGNS_TABLETOP_ACCESSORIES_*` (T1 coupon → T2 shell → T3A storage-fit → T3B complete tray → Workshop-ready graduation), `docs/DESIGNS_DRAWER_CABINET_ARCHITECTURE_REVIEW_2026-07-16.md`, and the immediately preceding GUI Clarity and Inventory Organization reviews. Every specific factual claim in this report about *current* code behavior was re-confirmed directly against `index.html` in this session, per the task's explicit instruction to inspect the actual implementation rather than rely on old reports.

**Registry/dispatch**: `designTemplateSelect` (`index.html:2883-2893`), `buildDesignResult` (`index.html:5319+`), `designPreviewModeForTemplate` (`index.html:5438-5443`), `designPreviewSelectorHtml` (`index.html:5444-5457`), `tabletopAccessoryProductRegistry` (`index.html:2865`), `tabletopTemplateMaturityFor`/`tabletopProductMaturityFor` (`index.html:2876-2882`).

**Per-generator build functions read**: `buildTabletopCornerFloorCouponDesignResult` (T1, `5175`), `buildTabletopRectangularShellDesignResult` (T2, `5210`), `buildTabletopStorageFitCouponDesignResult` (T3A, `5246`), `buildTabletopStorageTrayDesignResult` (T3B, `5275`), `buildTrayDesignResult` (dice/divider, `4868`).

**Finished View renderers read**: `buildTabletopRectangularShellFinishedViewSvg` (`5236`), `buildTabletopStorageFitCouponFinishedViewSvg` (`5293`), `buildTabletopStorageTrayFinishedViewSvg` (`5299`), `buildConcealedCleatFullBoxFinishedViewSvg` (`5420`), `buildGiftBoxFinishedViewSvg` (`5425`), `buildDiceTrayFinishedViewSvg` (`5362`), `buildDividerTrayFinishedViewSvg` (`5377`) — and the dispatch site that selects among them, `index.html:5700` region.

**Shared geometry/layout helpers read**: `buildBoxModel` (`3968`), `designPanelGeometryErrors` (`4412`), `buildAssemblyLabelPaths` (`4556`), `layoutDesignPanelRows` (`4679`), `designSvgValidation` (`4769`), `tabletopAccessoryRectangleOutline`/`tabletopAccessoryDimensions`/`tabletopAccessoryFinishedCavity`/`tabletopAccessoryManufacturingLimits`/`tabletopAccessoryProjectPoint`/`serializeTabletopAccessorySvg` (`5126-5170`), `tabletopClosedCornerStates` (`5795`), `designLayoutSizeWarning`/`addDesignLayoutSizeWarning` (`5309-5318`), `designFixtureHash` (`5980`).

**Project handoff**: `projectDraftFromDesign` (`5893+`), including its per-template branches for `dice-tray`, `divider-tray`, `finger-box`, `tabletop-corner-floor-coupon`, `tabletop-rectangular-shell-prototype`, and others.

**Dice Tray feature surface** (form fields, validation, fixtures) read in depth: `diceStorageMode`/`diceStorageWidth`/`diceFixedDivider`, `diceInsertType`/`diceInsertClearance`/`diceInsertThickness`/`diceWoodInsertPanel`, `trayBottomCover`, `jointStyle` — form rendering (`index.html:3326-3330`, `3447`), validation (`3899-3905`), and the extensive fixture block (`index.html:6064-6260`, dozens of assertions covering storage bays, dividers, inserts, liners, bottom covers, non-overlap, closed paths, deterministic output, and production-byte isolation from the Finished View).

**Physical-evidence architecture** (for context on what a new template would or would not plug into): `tabletopAccessoryProductRegistry`, `tabletopStorageTrayExactMatch`, `tabletopTrayResultAsDesign`, `buildTabletopTrayPromotionCandidate` — read to confirm exactly how tightly T1–T3B's evidence/promotion system is bound to their specific `templateId`+`generatorVersion`+`construction` identity triples (`index.html:990-1004`).

---

## 3. Review method

Static source reading, cross-checked against the extensive existing fixture assertions (which pin exact production SVG byte lengths and FNV-1a hashes, e.g. `dice-tray` production SVG `1726/51a55721`), was treated as the most reliable evidence of current behavior — a fixture asserting an exact byte length/hash would fail if the described behavior had drifted. No browser was launched for this review; every claim about rendered/runtime behavior in this report is either a direct reading of a template-literal-producing function or an existing fixture assertion, never a browser observation. This is a read-only architecture and geometry review only, consistent with its framing and the requirement not to claim unexecuted results.

---

## 4. Current generator inventory

All 15 registered `template` values, confirmed by direct grep of every `d.template === '...'`/`template === '...'` dispatch site, grouped exactly as `designTemplateSelect` groups them (`index.html:2886-2891`):

| Group | Templates |
| --- | --- |
| Boxes and cabinets | `finger-box`, `sliding-lid-box`, `drawer-cabinet`, `gift-box` |
| Trays | `dice-tray`, `divider-tray` |
| Signs and stands | `qr-stand`, `hanging-sign` |
| Coupons and prototypes | `joint-fit-coupon`, `concealed-cleat-corner-prototype`, `concealed-cleat-full-box-prototype`, plus all non-Workshop-ready Tabletop Accessories entries (`tabletop-corner-floor-coupon` [T1], `tabletop-rectangular-shell-prototype` [T2], `tabletop-storage-fit-coupon` [T3A]) |
| Workshop-ready | `tabletop-storage-tray` [T3B] — the only product currently carrying this maturity label |

Per-generator summary (purpose, key inputs, panels, current preview modes — all confirmed by source, not assumed):

| Template | Purpose | Key inputs | Panels/pieces | Preview modes |
| --- | --- | --- | --- | --- |
| `finger-box` | General finger-jointed open/lidded box | inside/outside dims, thickness, clearance, finger width, lid type, corner decoration | 5–6 (box + optional lid) | Cut Layout, **Finished Assembled** (isometric-style) |
| `sliding-lid-box` | Box with a sliding lid in laminated rails | usable dims, joint clearance, lid-side/vertical/front-insertion clearance, rail mode, open end | multiple + lid | Cut Layout, **Finished Closed/Open** |
| `drawer-cabinet` | Multi-row/column drawer cabinet, linked or separate interior thickness, custom-by-row layout | inside dims, rows/columns, layout mode, per-section column counts, lateral/vertical/rear clearance, shelf guides | many (scales with rows/columns) | Cut Layout, **Finished Front View** (dedicated `finished-front` mode, distinct from the generic `finished-view` mode used elsewhere) |
| `gift-box` | Decorative lidded gift box, hinged or lift-off lid, optional locating frame | dims, lid type/appearance, hinge count/geometry, frame clearance | box + lid (+ frame) | Cut Layout, **Finished Closed/Open** |
| `dice-tray` | Open-top dice tray with optional storage bay, divider, rolling-surface insert/liner, bottom cover | tray W/D/H, material thickness, fit clearance, joint style (finger/tab-slot), `diceStorageMode`, `diceInsertType`, `trayBottomCover` | 5 (tab-slot/finger walls+base) + optional divider + optional insert panel + optional cover | Cut Layout, **Finished Assembled** (shows storage bay, divider, and insert distinctly) |
| `divider-tray` | Open-top tray with 1–6 removable dividers | tray dims, divider count, joint style | 5 + N dividers | Cut Layout, **Finished Assembled** |
| `qr-stand` | Freestanding sign/QR stand with slot-in base | sign W/H, base depth, slot depth, corner radius, joint style | 2 (sign + base) | Cut Layout only |
| `hanging-sign` | Wall-hung sign with a hanging hole | dims, hole size/inset | 1 | Cut Layout only |
| `joint-fit-coupon` | Finger-joint fit-testing coupon (multiple clearance candidates) | edge length, body depth, finger width, clearance list | 2N (per-clearance pairs) | Cut Layout only (a test tool, not a product) |
| `concealed-cleat-corner-prototype` | Single-corner concealed-cleat registration prototype | leg length, wall height | 2 walls + cleats | Cut Layout, **Finished Assembled** |
| `concealed-cleat-full-box-prototype` | Full-box concealed-cleat registration prototype | outside dims, wall height | 4 walls + base + 8 cleats | Cut Layout, **Finished Assembled** |
| `tabletop-corner-floor-coupon` (T1) | Multi-clearance corner+floor joint coupon | interior leg, wall height, clearance step/units | up to 9 (3 candidates × WALL-A/WALL-B/FLOOR) | Cut Layout, **Finished Assembled** |
| `tabletop-rectangular-shell-prototype` (T2) | Open-top rectangular shell, closed three-way floor corners | interior W/D, wall height, thickness, clearance, finger width | 5 (FLOOR/FRONT/BACK/LEFT/RIGHT) | Cut Layout, **Finished Assembled** |
| `tabletop-storage-fit-coupon` (T3A) | Rail + removable false-bottom mechanism, fitted against an *existing* shell context | interior dims (context only), rail height, false-bottom thickness, panel clearance | 5 (4 rails + false bottom) | Cut Layout, **Finished Assembled** (shows shell context + rails + false bottom) |
| `tabletop-storage-tray` (T3B, Workshop-ready) | Complete shell + storage tray: T2 shell **composed with** T3A's rail/false-bottom mechanism | same as T2 + T3A combined, one coordinated set of inputs | 10 (5 shell + 5 storage) | Cut Layout, **Finished Assembled** (shows full shell, rails, and removable false bottom together) |

**Confirmed architecture fact**: `buildTabletopStorageTrayDesignResult` (T3B) is *not* an independent geometry implementation — it builds two internal sub-drafts (one `template:'tabletop-rectangular-shell-prototype'`, one `template:'tabletop-storage-fit-coupon'`), calls `buildTabletopRectangularShellDesignResult` and `buildTabletopStorageFitCouponDesignResult` directly, and concatenates/re-lays-out their panels. This is the exact composition pattern this review recommends reusing for the new generator (§9–§11).

---

## 5. Current Finished View inventory

| Template | Has Finished View? | Type | Reflects production params? | Fixture coverage |
| --- | --- | --- | --- | --- |
| `finger-box` | Yes | Isometric-style assembled | Yes (uses `result.metrics.dimensions`) | Yes |
| `sliding-lid-box` | Yes | Closed/Open assembled | Yes | Yes |
| `drawer-cabinet` | Yes | Dedicated Front View (`finished-front`) | Yes (`buildDrawerCabinetFrontElevationSvg(result.frontElevation)`) | Yes (per prior Drawer Cabinet reports; re-confirmed present in current dispatch) |
| `gift-box` | Yes | Closed/Open assembled, hinge/frame detail | Yes | Yes |
| `dice-tray` | Yes | Assembled, with storage bay/divider/insert regions distinctly rendered | Yes — confirmed by fixture assertions requiring `.dice-tray-storage`, `.dice-tray-storage-divider`, `.dice-tray-insert` classes to appear/not-appear exactly matching input state | Extensive (dozens of assertions, `index.html:6064-6260`) |
| `divider-tray` | Yes | Assembled, canonical divider-position labeling | Yes | Yes |
| `qr-stand` | **No** | — | n/a | n/a |
| `hanging-sign` | **No** | — | n/a | n/a |
| `joint-fit-coupon` | **No** | — | n/a | n/a |
| `concealed-cleat-corner-prototype` | Yes | Assembly/registration diagram | Yes | Yes (referenced by name in `runTabletopCornerFloorCouponFixtures`-adjacent groups) |
| `concealed-cleat-full-box-prototype` | Yes | Assembly/registration diagram, explicit "not a strength test" | Yes | Yes |
| `tabletop-corner-floor-coupon` (T1) | Yes | Assembled candidate diagram | Yes | Yes |
| `tabletop-rectangular-shell-prototype` (T2) | Yes | Assembled isometric shell | Yes | Yes (`index.html:6487` region asserts `finished view is screen-only` and byte-separation from production SVG) |
| `tabletop-storage-fit-coupon` (T3A) | Yes | Assembled context-shell + rails + false bottom | Yes | Yes |
| `tabletop-storage-tray` (T3B) | Yes | Assembled full shell + rails + removable false bottom | Yes | Yes (`index.html:15000` region: dedicated fixture builds a real T2 shell + T3B tray pair and asserts the Finished View DOM) |

**Every single Finished View function inspected begins with an explicit `if (!result?.valid || result.finishedView?.template !== '<own-template>') return '';` guard** — confirmed identically in `buildTabletopRectangularShellFinishedViewSvg`, `buildTabletopStorageFitCouponFinishedViewSvg`, `buildTabletopStorageTrayFinishedViewSvg`, `buildConcealedCleatFullBoxFinishedViewSvg`, `buildGiftBoxFinishedViewSvg`. This is a genuine, consistent shared contract, even though each function is separately implemented (see §7) — every one degrades to an empty string for an invalid/mismatched result rather than rendering something misleading.

**Every Finished View is confirmed screen-only by direct text inspection**: each renders an explicit `<desc>`/note stating it is "screen-only," "not included in the downloaded cut file," or equivalent, and fixtures (e.g., `index.html:6227` "Default production SVG remains separate from Finished View," `index.html:6520` region for T2) explicitly assert the *production* `result.svg` is untouched when Finished View mode is toggled.

---

## 6. Finished View gaps

**Confirmed gap (structural, not value-judged)**: `qr-stand`, `hanging-sign`, and `joint-fit-coupon` have no Finished View function of any kind.

**UX judgment, not a confirmed defect**: none of the three would gain meaningful assembly comprehension from a Finished View. A QR/sign stand is two flat pieces (sign + slotted base) whose assembly is a single slot-in action, self-evident from the cut layout; a hanging sign is one flat piece with a hole; a joint-fit coupon is explicitly a testing tool with no "finished product" state to depict. The prompt's own instruction — "Do not recommend Finished Views merely as decoration... each view must solve a comprehension or assembly problem" — argues directly against spending phase budget here. This review does **not** recommend a Finished View for any of these three in this phase.

**No other Finished View gap was found.** Every remaining template — including the two most complex, storage-bearing, removable-component products (`dice-tray`'s storage bay and `tabletop-storage-tray`'s removable false bottom) — already has a Finished View that specifically depicts the removable/hidden-storage state, which is exactly the category of confusion the prompt asks reviewers to prioritize. This is the central reason this review does not recommend spending the bounded phase's Finished View budget on any existing generator (§17–§18).

---

## 7. Existing geometry/helper architecture

**Shared, generator-agnostic helpers** (confirmed reused across multiple, unrelated templates):

- `buildBoxModel(parameters)` — a generic parametric box-panel generator taking `dimensionMode`, `width`/`depth`/`height`, `materialThickness`, `clearance`, `preferredFingerWidth`, `lid`, `cornerDecoration`, and a `floorCornerOwnership` flag. Confirmed reused verbatim by both `finger-box`-family logic and `buildTabletopRectangularShellDesignResult` (T2) with `floorCornerOwnership:true` — this is the actual shared origin of the "closed three-way corner" construction, not a Tabletop-Accessories-specific reimplementation.
- `designPanelGeometryErrors(panel)` — generic panel-geometry validation (self-intersection/degenerate-panel detection), called identically by every generator inspected.
- `layoutDesignPanelRows(panels, rowIds, margin, gap)` — generic row-based sheet layout, reused by T2, T3A, T3B, and (by extension) any new template following the same pattern.
- `buildAssemblyLabelPaths(panels, specs, guideSegmentsByPanel, options)` — generic optional assembly-label engraving, opt-in via the shared `assemblyLabels` boolean, reused identically across every template that offers it.
- `designSvgValidation(svg, expectedWidth, expectedHeight)` — generic post-serialization SVG sanity check (finite numbers, matching bounds), called by every `build*DesignResult` function before setting `result.valid = true`.
- `tabletopAccessoryRectangleOutline`, `tabletopAccessoryDimensions`, `tabletopAccessoryFinishedCavity`, `tabletopAccessoryManufacturingLimits`, `tabletopAccessoryProjectPoint`, `serializeTabletopAccessorySvg` — a small, cohesive family of Tabletop-Accessories-scoped helpers, already reused across all four T1–T3B templates and directly reusable by a fifth.
- `designFixtureHash` — the shared FNV-1a hashing helper used by every production-SVG golden pin in the fixture suite.
- `projectDraftFromDesign` — one shared dispatcher function with a per-template `else if` branch producing a Project draft's name/notes; confirmed to be the actual, singular Designs-to-Projects handoff mechanism (not a per-generator reimplementation).

**What can be reused unchanged for the new generator**: `buildTabletopRectangularShellDesignResult` (T2 shell, called exactly as T3B already calls it — zero modification needed), the entire T3A rail/false-bottom **pattern** (its validation rules, panel shapes, and false-bottom-clearance math are straightforward to call via a request shaped like T3A's own draft, exactly as T3B already does), `layoutDesignPanelRows`, `designPanelGeometryErrors`, `designSvgValidation`, `buildAssemblyLabelPaths`, `tabletopAccessoryProjectPoint` (for the new Finished View), and `projectDraftFromDesign`'s dispatch pattern (add one more `else if` branch).

**What needs a small, new, generator-specific piece**: a new top-level `buildTabletopTokenTrayDesignResult`-style function (modeled directly on `buildTabletopStorageTrayDesignResult`'s composition pattern, §9–§11), a new liner-allowance validation block (modeled directly on `dice-tray`'s existing `diceInsertType` validation, §12), and a new Finished View rendering function (modeled directly on `buildTabletopStorageTrayFinishedViewSvg`'s existing projection/labeling approach, §14).

**What must remain generator-specific and should not be forced into a shared abstraction**: `dice-tray`'s tab-and-slot geometry and T2/T3A/T3B's finger-jointed geometry are genuinely different constructions with different physical evidence behind them (§10) — this review explicitly does **not** recommend unifying them behind one "tray" abstraction, consistent with the instruction not to force unrelated generators through a new abstraction for theoretical cleanliness.

---

## 8. Existing production-output contracts

Confirmed, consistent across every generator inspected:

- `result.svg` is the sole production artifact; it is generated once by a `serialize*Svg`-family function and never mutated afterward by any preview-mode change.
- `result.finishedView` is a separate, small metadata object (dimensions and a few derived values) — **not** SVG markup — consumed only by the corresponding `build*FinishedViewSvg` function to render a *different*, screen-only SVG string that is never assigned to `result.svg` and never downloaded.
- `designLayoutSizeWarning`/`addDesignLayoutSizeWarning` (`index.html:5309-5318`) already provides the "large single-SVG layout, verify against your material sheet" warning at >400 mm in either dimension — an existing, generic, reusable safety net for "does this fit realistically," not something a new generator needs to reinvent.
- No generator currently auto-splits or nests output across multiple sheets; every generator produces exactly one SVG, and the existing warning pattern is "tell the user, do not silently act" — confirmed consistent with the prompt's explicit "do not silently scale or shrink parts to fit a sheet" requirement.

---

## 9. Existing fixture/golden architecture

- Production-byte goldens are pinned as `[length, FNV-1a hash]` pairs per named scenario (e.g., `['Dice default Finger','dice-tray',{},[1726,'51a55721',...]]`, `index.html:6038`), computed via the shared `designFixtureHash`. A new generator would add its own named scenarios to this same table using the same shared hashing helper — no new golden mechanism is needed.
- Every Finished View has its own dedicated assertion group proving: determinism (`svg === build...(sameResult)` called twice), finiteness (no `NaN`/`Infinity`/`undefined` in the output), correct `viewBox`, and — critically — that the *production* SVG is provably unaffected by Finished View state (e.g., `index.html:6227` "Default production SVG remains separate from Finished View," and the T2-shell fixture's explicit `finished view is screen-only` assertion checking that `result.svg` contains no Finished-View-only class names).
- The `runDesignGeometryFixtures` group (referenced throughout this session's prior reports as the ~1093-assertion protected suite) is the umbrella group that must remain fully green; this review's recommendation does not touch any existing scenario in that suite.

---

## 10. Candidate generator comparison

| Candidate | Real workshop use | Market/product use | Reuse of existing geometry | New schema/storage | Fit with Joe's 12×12 stock | Scope risk |
| --- | --- | --- | --- | --- | --- | --- |
| A — Parametric Dice Tray | High, but **largely already built** (`dice-tray` already has storage bay, divider, liner-as-measurement-only, insert panel, bottom cover) | High | Very high (already exists) | None | Fits | Low, but redundant — see below |
| B — Desktop Organizer | Moderate | Moderate | Low–moderate (no strong existing analog; compartment layout is open-ended) | Possibly a new preset concept | Fits | **High** — explicitly flagged as "too open-ended" by the prompt itself, confirmed by this review's own read: no existing generator provides a generic multi-compartment layout engine to reuse |
| C — Serving-Board Layout Guide | Moderate, mostly as an engraving template | Moderate | Low (primarily an artwork/placement-guide problem, not a fabrication-geometry problem) | None | Fits | **Rejected in kind** — this is an engraving/artwork layout tool, not a parametric fabrication generator, per the prompt's own exclusion of "primarily cosmetic artwork" |
| D — Location Keychain/Ornament | Low fabrication complexity, high personalization value | Moderate–high as a gift-shop item | Low | None for the shape itself; would need a text/shape template concept | Fits easily (small parts) | Requires no map/network dependency **only if** limited to typed text/coordinates and simple outline shapes — but this pushes it toward "primarily cosmetic," which the prompt excludes |
| E — Stackable Token/Dice Tray | **High** — reuses the one proven, physically-validated shell+removable-panel construction (T2/T3A/T3B) at a smaller, market-appropriate size | **High** — directly matches Joe's stated cork-lined-dice-tray exploration and board-game-store relationship | **Very high** — direct reuse of `buildTabletopRectangularShellDesignResult` and the T3A rail/false-bottom pattern, unmodified | None required (see §21) | Fits comfortably (smaller than T3B's own 80×60 mm footprint) | **Low** — see §17–§18 for the explicit scope boundary that keeps this small |
| F — Small removable-bottom keepsake/game box | High, but this is **almost exactly what Finger Box + a T3A-style false bottom would already provide** if composed the same way T3B composes T2+T3A | Moderate–high | High, if built as an extension of Finger Box's existing box-building path | None required if done the same way | Fits | Viable **second** candidate, but Candidate E better matches the explicit "tabletop/gaming accessory" preference and Joe's own stated cork-liner exploration |

### Why the other candidates should wait

- **Candidate A is not rejected because it's a bad idea — it's rejected because it already exists.** Recommending it again would either duplicate `dice-tray`'s already-tested feature set or require justifying why a second, near-identical generator is needed. The genuinely new value left in "dice tray" territory is the *stronger, finger-jointed* construction and a *smaller, more giftable* footprint — which is exactly what Candidate E provides, framed honestly as a sibling to T3B rather than a re-do of `dice-tray`.
- **Candidate B (Desktop Organizer)** is excluded by the prompt's own list of things not to choose ("a generic all-purpose organizer builder") and by this review's confirmation that no existing generator provides a reusable open-ended-compartment-count layout engine — building one would be the single largest, riskiest piece of new geometry among all the candidates.
- **Candidate C (Serving-Board Layout Guide)** is, on inspection, an artwork/engraving-placement problem rather than a parametric-fabrication problem — it does not produce panels, joints, or a physical assembly in the way every other Designs template does, and the prompt explicitly excludes "a generator whose primary purpose is cosmetic artwork rather than repeatable fabrication."
- **Candidate D (Location keychain/ornament)** is small and safe but is fundamentally a personalization/text-layout feature, not a fabrication-geometry feature reusing shell/tray/divider/panel logic — it would not meaningfully exercise or validate any of the proven architecture this phase is meant to build on, and risks drifting toward "cosmetic artwork," which the prompt excludes.
- **Candidate F (keepsake/game box)** is a reasonable second choice and is explicitly acknowledged as such, but Candidate E is preferred because it reuses the *stronger, already-physically-validated* Tabletop Accessories construction (finger-jointed shell, proven rail/false-bottom mechanism) rather than Finger Box's simpler open/lidded-box construction, and it maps more directly onto Joe's stated board-game/cork-liner interest and the "preferred product direction" guidance in the prompt.

---

## 11. Recommended generator

**"Stackable Token & Dice Tray"** (working name; final product name is a deferred product decision, not an architecture fact) — Candidate E, selected over Candidate A because it adds genuinely new market/product value (a smaller, market-ready, stronger-construction sibling to T3B) rather than duplicating `dice-tray`'s already-mature feature set, and selected over Candidate F because it builds on the more rigorously validated Tabletop Accessories construction family and aligns with the prompt's explicit "preferred product direction" toward tabletop/gaming accessories.

**Architecture fact, confirmed**: this generator can be built entirely as new, additive code — a new `buildTabletopTokenTrayDesignResult` function that internally builds a shell sub-draft and a rail/false-bottom sub-draft exactly as `buildTabletopStorageTrayDesignResult` already does, calling `buildTabletopRectangularShellDesignResult` unmodified and reusing T3A's rail/false-bottom validation and panel-shape logic unmodified. No existing function's signature or body needs to change.

---

## 12. Recommended existing Finished View additions

**None.** Per §6, no existing generator has a confirmed, value-justified Finished View gap. This review recommends **Option B** (one new generator, no existing-generator Finished View work), consistent with the prompt's own instruction: "If the new generator itself consumes most of the safe phase budget, recommend only that generator's Finished View and defer existing-generator additions." Given the new generator itself requires a new composed build function, a new validation block, and a new Finished View renderer, this is exactly that situation.

---

## 13. Dimensional contract

**User-entered dimensions** (smallest useful set, modeled directly on T2/T3A/T3B's existing field shapes):
- Interior width, depth, usable height (mm) — reuses the exact `requestedInterior` concept T2/T3B already report.
- Material thickness (mm) — reuses the existing shared field.
- Joint clearance (mm, signed) — reuses the existing shared field and its existing −0.10…0.30 mm validation range (`buildTabletopRectangularShellDesignResult`'s own check).
- Preferred finger width (mm) — reuses the existing shared field and existing minimum-segment validation.
- Rail height and false-bottom thickness (mm) — reuses T3A's exact fields and validation (minimum rail height, minimum rail length, minimum rail overlap).
- Total false-bottom clearance (mm) — reuses T3A's exact field and derivation (`perSideClearance = totalClearance / 2`).
- **New**: liner allowance selection (`none`/`cork`/`felt`/`leather`/`generic-liner`) and, if not `none`, a liner clearance (mm, per-side) and liner thickness (mm, for usable-height reporting only) — modeled directly on `dice-tray`'s existing `diceInsertType`/`diceInsertClearance`/`diceInsertThickness` fields and their existing validation, **not** a new invention.
- Panel spacing (mm) — reuses the existing shared field.

**Derived dimensions** (computed, never separately entered): outer envelope (width/depth/height), finished/clear cavity, false-bottom width/depth, rail lengths, per-side clearance, remaining visible wall height above the false bottom — all reused verbatim from T2/T3A's existing derivation formulas.

**Production dimensions**: the panel list and its exact millimeter dimensions, identical in kind to T3B's own `panels`/`widthMm`/`heightMm`.

**Screen-only Finished View dimensions**: the outer envelope, rail height, false-bottom thickness, corner insets — the same subset T3B's own `result.finishedView` object already carries, reused unchanged.

**Machine/material-specific fit evidence** (explicitly *not* defaults for the new generator, per the prompt's own caution): Joe's T3B evidence (2.88 mm measured 3 mm basswood, −0.075 mm clearance, 9 mm finger width, on a Genmitsu L8 20W) is evidence for *that specific* shell, machine, and material combination — it is not transferable to a different-sized product without a new physical test, and this review does not recommend copying those exact values as the new generator's defaults (see §19).

**Globally safe / starting-point defaults**: the new generator should ship with its own independent, conservative starting-point defaults (e.g., an interior sized comfortably for a handful of dice/tokens/small cards, well within a 12×12 inch sheet with margin to spare), explicitly labeled as a starting point requiring the same kind of test-before-trust treatment every other prototype-stage Designs template already receives.

**Impossible combinations**: reuse T3A's exact existing checks — false-bottom dimensions must be positive and smaller than the interior cavity; rail height must clear a minimum; rail height plus false-bottom thickness must leave visible wall height; rail lengths must clear a minimum; rail overlap must clear a minimum — plus one new check modeled on `dice-tray`'s existing pattern: liner allowance thickness must leave positive remaining wall height, and liner clearance must not exceed the available play-surface footprint.

---

## 14. Construction-method decision

**Recommended: reuse the T2 closed-three-way-corner finger-jointed shell plus the T3A bottom-supported four-rail removable false-bottom panel — the exact construction already proven in T3B — not `dice-tray`'s simpler tab-and-slot construction.**

Rationale, weighed against the prompt's construction-method criteria:
- **Strength**: finger-jointed corners are inherently stronger and more self-registering than butt-jointed tab/slot walls (`dice-tray`'s own UI text already concedes "this is not corner finger joinery... adjacent walls meet as butt joints and need glue after a dry fit").
- **Assembly complexity**: comparable to T3B's existing assembly (already documented and already has a Finished View walking through it).
- **Fit tolerance / material variability**: this is the one construction in the app with *actual physical evidence* behind its clearance/finger-width choices (Joe's 80×60×25 mm shell), even though that specific evidence does not transfer numerically to a new size (§13, §19) — the *method* is proven even where the *exact numbers* are not.
- **Hidden-storage accessibility**: the false-bottom-on-rails mechanism is explicitly the pattern the prompt calls out as most valuable for Finished View comprehension ("removable bottoms," "layered trays," "storage compartments") — reusing it directly serves that stated priority.
- **Beginner success rate**: finger-jointed corners are more forgiving of minor material-thickness variance than a tab/slot fit, because the same clearance/finger-width machinery already handles this (§13).
- **Liner installation**: a liner sits on the play surface or in the storage cavity in either construction; this is independent of wall-corner method.

**Not recommended for this generator**: layered rings-and-base (a materially different, less-tested construction with no existing helper family to reuse); magnetic inserts (explicitly not to be required hardware per the prompt, and no existing generator uses magnets); friction-fit inserts beyond what the existing false-bottom-on-rails mechanism already provides (redundant); a lift notch (a small, plausible future addition, but new geometry, deferred per §17).

**Stacking lip — explicitly evaluated and deferred.** A stacking lip/nesting interface (a stepped rim at the top of the wall panels) is the one feature from Candidate E's list that would require genuinely new geometry not present in any existing generator. This review recommends deferring it: it is not required to make the tray useful or sellable on its own, it would be the single largest new-geometry risk in the phase, and it can be added later, once the base tray's shell+false-bottom construction is physically validated at its new (smaller) size, without disturbing anything built in this phase.

---

## 15. Panel and production-layout contract

Reusing T3B's exact composition pattern:

| Panel | Source | Count |
| --- | --- | --- |
| FLOOR, FRONT, BACK, LEFT, RIGHT | T2 shell (`buildTabletopRectangularShellDesignResult`, unmodified) | 5 |
| RAIL-FRONT, RAIL-BACK, RAIL-LEFT, RAIL-RIGHT, FALSE-BOTTOM | T3A-style rail/false-bottom mechanism (same shapes/validation as `buildTabletopStorageFitCouponDesignResult`) | 5 |
| **Total** | | **10**, one sheet, `sheetCount: 1` |

- Layer count: 1 (cut layer only; no engrave layer is required for the base product — a liner-outline engrave guide is a plausible, small, optional future addition, but this review does not recommend including it in the first phase, to keep the diff minimal; the liner is otherwise handled as a **dimension allowance and Finished View annotation only**, exactly like `dice-tray`'s existing liner treatment).
- Part labels: reuse the existing `assemblyLabels` opt-in mechanism verbatim (same `buildAssemblyLabelPaths` call, same panel-ID-as-label convention).
- Grain/orientation: not tracked by any existing generator in this family; no new behavior needed.
- Sheet footprint: at a token/dice-tray-appropriate size, one 12×12 inch (≈305×305 mm) sheet is realistic — smaller than T3B's own 80×60 mm interior footprint plus wall/rail allowances, which itself already fits comfortably within that stock size. No multi-sheet requirement is anticipated; if a user enters unusually large dimensions, the existing `designLayoutSizeWarning` mechanism (§8) already provides the "verify against your material sheet" warning without any new code.
- Minimum spacing: reuse the existing shared `panelSpacing` field and `layoutDesignPanelRows`'s existing gap handling.
- Automatic nesting: **not needed and not recommended** — the existing simple row-layout helper is sufficient at this part count and size, consistent with every other generator in the app.
- Impossible sheet-fit conditions: presented via the same existing warning mechanism (§8); no new "does not fit" hard-block is needed unless a future phase adds multi-sheet support (explicitly out of scope here).
- Output: **one production SVG**, matching every existing generator's contract — no reason to deviate.

---

## 16. Finished View contract

The new Finished View function must satisfy every point already true of every existing Finished View (§5), reusing the pattern verbatim:

- Explicitly screen-only, stated in its own `<desc>`/note text.
- Never enters `result.svg`; rendered by a separate function called only by the preview UI.
- Never alters panel layout, never adds cut paths, never changes production dimensions — enforced structurally by the fact that the new render function only *reads* `result.finishedView`/`result.metrics`, never writes to `result.panels`/`result.svg`.
- Reuses the generator's actual calculated dimensions (outer envelope, rail height, false-bottom thickness, corner insets) rather than recalculating anything independently — modeled directly on `buildTabletopStorageTrayFinishedViewSvg`'s existing approach of reading `result.finishedView` fields.
- Deterministic: same input produces byte-identical output, provable the same way every existing Finished View fixture already proves it (call twice, compare strings).
- Degrades safely: the same `if (!result?.valid || result.finishedView?.template !== '<own-template>') return '';` guard, plus the same "if any projected point is non-finite, return ''" guard already used in `buildTabletopStorageTrayFinishedViewSvg`.
- Reuses the existing `tabletopAccessoryProjectPoint` isometric-style projection helper rather than inventing new projection math.
- Fixture coverage must include the same "production SVG remains byte-identical across preview-mode changes" assertion every existing Finished View fixture already includes.

**Evaluation of shared architecture sufficiency**: the existing Finished View architecture (one small `build*FinishedViewSvg` function per template, dispatched by a simple `if/else if` chain in the preview-rendering site, §7) is **already sufficiently shared** for this purpose — no new rendering framework or shared component is justified. The one small, genuinely reusable piece worth extracting *if* a future phase adds a second similar tray is the shell+rail+false-bottom projection drawing logic itself (currently near-duplicated between T3A's and T3B's Finished View functions); this review does not recommend that extraction now, since duplicating ~15 lines of projection code one more time is lower-risk than refactoring two already-tested functions in the same phase that adds a new generator.

---

## 17. Validation and impossible-state behavior

Reusing existing patterns exactly:

| Case | Behavior | Source pattern reused |
| --- | --- | --- |
| Zero/negative interior dimensions | Hard block | Existing shared box/shell validation |
| Material thickness ≤ 0 | Hard block | T2's existing `tabletopAccessoryManufacturingLimits` check |
| Finger width below manufacturable minimum for the given thickness | Hard block | T2's existing check |
| Joint clearance outside −0.10…0.30 mm | Hard block | T2's existing check |
| Rail height below minimum (`max(3×thickness, 10mm)`) | Hard block | T3A's existing check |
| False-bottom thickness ≤ 0, or rail height + false-bottom thickness ≥ wall height | Hard block | T3A's existing check |
| Rail lengths below minimum | Hard block | T3A's existing check |
| Rail overlap below minimum | Hard block | T3A's existing check |
| False-bottom width/depth ≤ 0 or ≥ interior | Hard block | T3A's existing check |
| **New**: liner allowance thickness leaves no positive remaining wall height | Hard block, modeled on `dice-tray`'s existing insert-thickness-vs-wall-height check | `dice-tray`'s existing pattern |
| **New**: liner clearance exceeds available footprint | Hard block, modeled on `dice-tray`'s existing insert-clearance validation | `dice-tray`'s existing pattern |
| Output width/height > 400 mm | Warning (not a hard block) | Existing `designLayoutSizeWarning` |
| Non-wood liner selected (cork/felt/leather/generic) | **Advisory, not a cut feature** — reported as a dimension only, with an explicit safety note (reusing `dice-tray`'s exact existing warning text pattern about verifying composition, adhesives, ventilation, and fire safety separately) | `dice-tray`'s existing pattern |
| Geometry self-intersection / degenerate panel | Hard block via existing `designPanelGeometryErrors` | Shared, generic |
| Duplicate cut paths | Prevented structurally by the same panel-composition approach T3B already uses (each panel has a distinct ID and non-overlapping placement via `layoutDesignPanelRows`) | Shared, generic |
| Impossible production layout (panels don't fit within layout bounds) | Surfaced via existing `layout.errors` propagation, same as every generator | Shared, generic |

**No production limitation is hidden behind the Finished View** — every hard-block case above prevents `result.valid` from becoming `true`, and the Finished View function's own guard (`if (!result?.valid...)`) means an invalid configuration never produces a Finished View at all, so there is no risk of a Finished View "looking plausible" over a blocked configuration.

---

## 18. Material/fit/kerf behavior

- Material thickness, clearance, and finger width are handled by the exact same shared fields and validation as T2/T3A/T3B — no new thickness-handling logic is needed.
- Kerf is handled the same way the rest of the Tabletop Accessories family handles it: `clearance` already functions as the effective signed kerf/interference value (T2's `kerfReference` metric is explicitly sourced from the same `clearance` value) — no separate kerf-compensation mechanism needs to be invented.
- The liner allowance is explicitly **not** a cut-geometry feature for non-wood materials — this is a deliberate, already-established safety boundary (`dice-tray`'s own UI text: "Non-wood liners are size reports only; verify composition, adhesives, ventilation, and fire safety separately") that this review recommends preserving exactly, not loosening.

---

## 19. Project/storage integration decision

- **Designs-to-Projects handoff**: include, by adding one new `else if (template==='<new-template-id>')` branch to `projectDraftFromDesign`, following the exact existing pattern (a descriptive name plus a few `addNote(...)` calls summarizing the liner allowance and construction, modeled directly on the existing `dice-tray`/`tabletop-rectangular-shell-prototype` branches).
- **Library material selection / Inventory material selection**: reuse the existing shared `materialId`/`materialLabel` fields exactly as T2/T3A/T3B already do — no new integration code needed.
- **Physical evidence linking**: **explicitly deferred.** T1–T3B's elaborate staged-evidence/promotion pipeline (raw-result recording, Coupon-proven → Shell-proven → Complete-Tray-proven, Workshop-ready graduation) is tightly bound to each product's own `templateId`+`generatorVersion`+`construction` identity triple (§2, `tabletopStorageTrayExactMatch`). Wiring a new template into that pipeline would require a new raw-result schema, new promotion-candidate logic, new matcher functions, and a new maturity-registry entry — a substantial addition explicitly disproportionate to a "small, bounded" phase. This review recommends the new generator ship the same way T1–T3B themselves originally shipped (and the way `dice-tray`/`gift-box` still ship today): as a **prototype/starting-point** template using the app's ordinary Project/Library Material Test mechanism for physical validation, with the elaborate staged-evidence pipeline as an explicit, separate, deferred future phase if Joe wants that level of rigor for this product too.
- **Saved generator records / recent-history entries**: **not recommended.** No existing generator (other than the Tabletop Accessories evidence system, which this review declines to extend) has a "saved preset" concept; inventing one now would be new, non-essential schema.
- **No schema migration is required or recommended.** Every new field is a new, independently-defaulted `designDraft` property (following the exact existing convention of prefixed field names, e.g. `tabletopTrayInteriorWidth` for T3B), and `designDefaults()`/`normalizeDesignDraft`-style handling already tolerates unknown/missing fields the same way it does for every other template — this is additive, not a migration.

---

## 20. Physical validation plan

**What can be verified through fixtures**: exact panel count and dimensions at the new default size; deterministic, finite Finished View output; production-SVG byte identity across preview-mode toggles; every validation rule in §17 (hard blocks and warnings) at their exact boundary values, following the same boundary-testing pattern already used for `dice-tray`'s storage-width validation (`index.html:6229-6235`, testing 17.9/18.0/19.0/19.99/20.0/27.9/28.0 mm explicitly); non-overlap of all physical panels; no duplicate/non-closed cut paths; Designs-to-Projects handoff producing the expected name/notes; direct `file://` operation; no network dependency.

**What requires browser inspection** (not claimed as done in this review): visual correctness of the new Finished View's projection at a genuinely new, smaller footprint than T2/T3A/T3B have ever been rendered at; responsive behavior of the new form fields; keyboard/accessibility behavior of the new liner-type selector, consistent with the existing `dice-tray` selector it's modeled on.

**What requires a physical cut** — this is a **new dimensional/construction combination**, and per the prompt's explicit instruction, none of Joe's existing physical evidence (T3B's −0.075 mm clearance, 9 mm finger width, 2.88 mm measured thickness; the +0.10 mm finger-joint-coupon best fit; the 0.45 mm drawer lateral clearance) should be assumed to transfer automatically to this new size without a fresh test. Recommended minimum physical prototype:
- **Suggested material**: 3 mm basswood plywood or 3 mm Baltic Birch plywood — the two materials Joe has already measured (basswood ≈ per T3B's own 2.88 mm; Baltic Birch not yet measured at 3 mm in the evidence supplied, only at 1/4 inch ≈ 5.96 mm) — this review does **not** assume basswood's 2.88 mm measured value applies to a different sheet without a fresh caliper check, per the prompt's own caution.
- **Dimensions**: the new generator's own starting-point default size (§13) — deliberately smaller than T3B's 80×60×25 mm, chosen to be dice/token-appropriate.
- **Which fit value should be tested rather than assumed**: (a) the finger-joint clearance at this specific new size and this specific sheet's actual measured thickness — a fresh coupon test, not reused from T3B or the +0.10 mm finger-joint-coupon result (that value came from a different joint contract, per the prompt's explicit warning); (b) the rail-height/false-bottom fit at the new, smaller proportions; (c) whether a real piece of cork or felt Joe has on hand actually matches the liner-allowance dimensions reported by the tool.
- **Which parts can be cut as coupons before the full product**: a small finger-joint corner coupon (reusing the existing `joint-fit-coupon`/T1 pattern) at the new material and finger-width combination, before committing to a full shell cut.
- **Pass/fail criteria**: all five shell panels seat; corners close without excessive gap; the false bottom rests on all four rails, lies flat, and lifts/reseats cleanly; no bowing, veneer tear-out, or visible rack; a liner (if used) fits the reported allowance without needing to be trimmed. This mirrors T3B's own already-used pass/fail language.
- **How physical evidence should later be recorded**: through the ordinary Log/Library Material Test mechanism in this first phase (per §19); a dedicated staged-evidence/promotion pipeline (matching T1–T3B's) is an explicit, separate, deferred future decision.
- **What remains "starting point" until tested**: every dimension, clearance, and finger-width default this generator ships with, exactly as every other prototype-stage Designs template in this app is already labeled.

---

## 21. Options considered

| Option | Verdict |
| --- | --- |
| **A — Finished Views only, no new generator** | **Rejected.** §6 found no confirmed, value-justified existing Finished View gap; adding one to `qr-stand`/`hanging-sign`/`joint-fit-coupon` would be decoration, not comprehension, which the prompt explicitly disallows. |
| **B — One practical generator only, no existing-generator Finished View additions** | **Recommended.** Matches the prompt's own fallback instruction for exactly this situation (§12) and keeps the phase reviewable. |
| **C — One generator plus one or two very small Finished View additions** | **Considered and declined**, not because it's unsafe, but because §6 found no existing gap worth spending the budget on; forcing in an unnecessary Finished View addition just to fill the option would work against the "each view must solve a comprehension problem" principle. |
| **D — Broad Finished View completion plus multiple generators** | **Rejected**, consistent with the prompt's own steer: existing Finished View coverage is already broad and mature (§5), and multiple new generators in one phase would violate "bounded, reviewable" and risk delaying nothing else since this is already the terminal planned phase before ongoing generator work, but still carries unnecessary regression surface for no confirmed benefit. |

---

## 22. Recommended bounded implementation

### 1. Exact user problem solved
Joe (and, by extension, a board-game-accessory customer) has no small, market-ready, physically-provable-construction tray sized for dice/tokens/small game pieces with an honest cork/liner allowance — `dice-tray` covers this territory with a weaker construction, and T3B covers the strong construction at a larger, general-storage size.

### 2. Exact practical generator selected
"Stackable Token & Dice Tray" (Candidate E), a new Designs template composing the existing T2 shell and T3A rail/false-bottom builders at a new, smaller default size, plus one new liner-allowance parameter.

### 3. Why that generator is selected
Highest reuse of already-physically-validated construction; directly matches Joe's own stated cork-liner exploration and board-game-store context; smallest genuinely new surface area among the serious candidates (§10).

### 4. Existing Finished View additions included
None (§12, §21).

### 5. Explicit deferred candidates
Desktop Organizer, Serving-Board Layout Guide, Location Keychain/Ornament, a second full keepsake-box generator, a stacking lip/nesting interface, cork/felt/leather as actual cut geometry, an engraved liner-outline guide layer, any staged physical-evidence/promotion pipeline for the new product, CSV/preset saving, multi-sheet output.

### 6. Final input contract
Interior width/depth/usable height, material thickness, joint clearance, preferred finger width, rail height, false-bottom thickness, total false-bottom clearance, panel spacing, liner-allowance type (none/cork/felt/leather/generic-liner), liner clearance, liner thickness — per §13.

### 7. Derived dimensions
Outer envelope, finished/clear cavity, false-bottom width/depth, rail lengths, per-side clearance, remaining visible wall height — reused verbatim from T2/T3A (§13).

### 8. Construction method
T2 closed-three-way-corner finger-jointed shell + T3A bottom-supported four-rail removable false-bottom panel, unmodified (§14).

### 9. Panel/part list
FLOOR, FRONT, BACK, LEFT, RIGHT, RAIL-FRONT, RAIL-BACK, RAIL-LEFT, RAIL-RIGHT, FALSE-BOTTOM — 10 panels, one sheet (§15).

### 10. Production layout behavior
Existing `layoutDesignPanelRows` row-based layout; existing `designLayoutSizeWarning` for oversized output; no new nesting/multi-sheet logic (§15).

### 11. Finished View behavior
New function modeled directly on `buildTabletopStorageTrayFinishedViewSvg`, same screen-only/deterministic/degrade-safely contract (§16).

### 12. Validation and impossible-state behavior
Reuse every existing T2/T3A hard block; add two new liner-specific hard blocks modeled on `dice-tray`'s existing insert validation; reuse the existing >400 mm layout warning (§17).

### 13. Material-thickness behavior
Identical to T2/T3A/T3B — a single shared `materialThickness` field driving finger-width minimums and panel offsets (§18).

### 14. Fit/kerf/clearance behavior
Identical to T2/T3A/T3B's existing signed-clearance convention; no new kerf concept (§18).

### 15. Existing helpers reused
`buildTabletopRectangularShellDesignResult`, the T3A rail/false-bottom pattern, `layoutDesignPanelRows`, `designPanelGeometryErrors`, `designSvgValidation`, `buildAssemblyLabelPaths`, `tabletopAccessoryProjectPoint`, `serializeTabletopAccessorySvg`, `designLayoutSizeWarning`, `projectDraftFromDesign`'s dispatch pattern, `designFixtureHash` (§7).

### 16. New helpers required
One new `buildTabletopTokenTrayDesignResult` composition function; one new Finished View render function; one new liner-allowance validation block (small, modeled on existing code) (§7, §16, §17).

### 17. Likely files/functions touched
All within `index.html`: `designTemplateSelect` (new option), `designDefaults()` (new prefixed fields), a new `buildTabletopTokenTrayDesignResult` function, a new `buildTabletopTokenTrayFinishedViewSvg` function, `buildDesignResult`'s dispatch chain (one new `else if`), `designPreviewModeForTemplate`/`designPreviewSelectorHtml` (add the new template to the existing `finished-view`-mode list, same pattern as T1–T3B), the Finished View dispatch site (`index.html:5700` region, one new branch), `projectDraftFromDesign` (one new branch), and new fixture assertions.

### 18. Protected functions and goldens
Every existing `build*DesignResult`/`build*FinishedViewSvg` function, `buildBoxModel`, `layoutDesignPanelRows`, `designPanelGeometryErrors`, `designSvgValidation`, every existing production-SVG golden pin — none require modification, and none should be modified.

### 19. Stored-byte impact
None for existing records. New `designDraft` fields are additive, defaulted, and follow the exact existing per-template field-naming convention; no schema version change.

### 20. Backup/export impact
None beyond the new draft fields appearing in `backupObject()` the same way every prior template's fields already do — no format change, no migration.

### 21. Production-byte impact for existing generators
**None, by construction** — no existing `build*DesignResult`/`build*FinishedViewSvg` function is modified; the new generator only calls them, it does not change them. This should be proven, not merely argued, via a fixture asserting every existing production-SVG golden hash is unchanged after this phase (§25).

### 22. New generator production-output contract
One production SVG, 10 panels, exact-scale, no engrave layer required, `sheetCount: 1`, filename following the existing `l8-<template>-<date>.svg` convention.

### 23. Schema/migration impact
None (§19, §22.19).

### 24. Projects handoff behavior
New `else if` branch in `projectDraftFromDesign`, following the existing pattern exactly (§19).

### 25. Accessibility requirements
The new liner-type `<select>` and numeric fields must follow the exact existing shared `field`/`select`/`unitField` helper pattern (already accessible, per the GUI Clarity review's confirmation of this shared infrastructure); the new Finished View SVG must include `role="img"` and an `aria-label`, matching every existing Finished View function.

### 26. Responsive requirements
No new layout mechanism; the new form fields render inside the existing Designs form grid, and the new Finished View SVG uses a fixed `viewBox` exactly like every existing one, which already scales via CSS in the existing responsive layout.

### 27. Fixture plan
See §25 below (dedicated section, since the prompt requires a distinct "Required fixtures" report section).

### 28. Manual browser checks
See §26 below.

### 29. Physical validation plan
See §20 above.

### 30. Acceptance criteria
See §27 below.

### 31. Risks
See §28 below.

### 32. Rollback strategy
See §29 below.

### 33. Whether one independent post-implementation audit is warranted
**Yes** — see §31.

---

## 23. Exact non-goals

Stacking lip/nesting interface; cork/felt/leather as actual cut geometry (measurement-only only, per §18); an engraved liner-outline guide layer; a staged physical-evidence/promotion pipeline for the new product; saved-preset/recent-history schema; CSV import/export changes; multi-sheet automatic nesting; any modification to `dice-tray`, `divider-tray`, `finger-box`, `gift-box`, `sliding-lid-box`, `drawer-cabinet`, Concealed Cleat prototypes, or T1/T2/T3A/T3B; a Finished View for `qr-stand`/`hanging-sign`/`joint-fit-coupon`; multi-machine profiles; large-format/multi-sheet fabrication; a Jigsaw Puzzle Generator; any framework/library adoption.

---

## 24. Storage/schema/backup impact

None beyond additive `designDraft` fields (§19, §22.19–22.20). `SCHEMA_VERSION`, `STORAGE_KEY`, `BACKUP_FORMAT`, `ENCRYPTED_BACKUP_FORMAT`, and vault behavior are entirely untouched — this phase does not touch persistence, security, or Inventory/Library/Project/Test Grid record schemas at all.

## Existing-generator production-byte impact

None, by construction (§22.21) — to be proven by an explicit fixture, not merely argued.

## New generator production-output contract

One SVG, 10 panels, exact-scale, single sheet (§22.22, §15).

---

## 25. Required fixtures

At minimum, add:

- **Generator registration**: the new template appears exactly once in `designTemplateSelect`'s options and exactly once in `buildDesignResult`'s dispatch chain.
- **Default values**: `designDefaults()` includes valid, safe starting-point values for every new field.
- **Parameter parsing**: every new field round-trips correctly through the existing `readForm`/normalization path.
- **Validation / impossible dimensions**: boundary tests for every hard block in §17, following the exact existing boundary-testing style already used for `dice-tray`'s storage-width checks (test just-below, at, and just-above each threshold).
- **Exact panel dimensions**: the 10-panel list at a fixed default scenario, with exact expected dimensions asserted (not just "5 panels present").
- **No duplicate cut lines / no self-intersections**: reuse the existing `nonOverlapping`/closed-path fixture pattern already used for `dice-tray` (§ fixture references at `index.html:6236-6247`).
- **Exact production bounds**: `widthMm`/`heightMm` match the expected layout for the default scenario.
- **Layer naming / part labels**: assembly-label opt-in produces the expected label set, same as every existing generator's label fixture.
- **Material-thickness changes**: re-run the default scenario at 2–3 different thicknesses, confirming derived dimensions scale correctly and no hard block fires unexpectedly.
- **Fit/clearance changes**: re-run at a few clearance values within and outside the valid range.
- **Finished View deterministic output**: call the new render function twice on the same result, assert identical strings.
- **Finished View screen-only isolation**: assert the rendered Finished View string never appears inside `result.svg`.
- **Preview-mode changes leave production SVG byte-identical**: toggle preview mode, re-read `result.svg`, assert no change — the single most important fixture for this phase's core safety claim.
- **Projects handoff**: assert the new `projectDraftFromDesign` branch produces the expected name/notes without mutating `state`/`localStorage`.
- **Legacy saved Designs behavior**: a `designDraft`-shaped object missing the new fields entirely (simulating a backup created before this phase) still normalizes to safe defaults and produces a valid result.
- **Direct `file://` operation / no network dependencies**: reuse the existing generic assertions already applied to every other generator (no `fetch`, no external URL).
- **No storage/schema changes unless explicitly approved**: assert `SCHEMA_VERSION` and `STORAGE_KEY` are unchanged, and that `backupObject()`'s shape for every *other* record type is unaffected.
- **No regressions to existing production goldens**: re-run the full existing golden table (§9) and confirm every prior hash is unchanged.
- **Protected full Designs geometry suite**: the full `runDesignGeometryFixtures`/production-golden group must remain fully green.

---

## 26. Required manual browser checks

Direct `file://` load in a disposable browser profile: select the new template from the Designs dropdown and confirm it appears under a sensible group; enter the default parameters and confirm a valid result renders; toggle Cut Layout ↔ Finished Assembled and visually confirm the shell, rails, and removable false bottom render plausibly at the new, smaller scale; enter each liner-allowance type and confirm the Finished View and summary text update accordingly without generating extra cut geometry; deliberately enter each boundary/invalid value from §17 and confirm the expected hard block or warning appears; use "Save as Project" and confirm the resulting Project draft's name/notes are sensible; confirm no console errors and no network requests occur. None of this was executed as part of this review — this is a review, not an implementation or verification pass.

---

## 27. Acceptance criteria

1. Exactly one new template is registered; no existing template's dispatch entry is modified.
2. All 10 panels render at the exact expected dimensions for the default scenario; every T2/T3A validation rule fires correctly at the new size.
3. The new liner-allowance parameter behaves exactly like `dice-tray`'s existing insert-type pattern: measurement-only for non-wood materials, with the same class of safety-note text.
4. The Finished View is provably screen-only and provably leaves production SVG bytes byte-identical across preview-mode toggles.
5. `git diff` for this phase touches only the files/functions listed in §22.17 — no existing `build*DesignResult`/`build*FinishedViewSvg` function body changes.
6. Full `?selftest=design`/`?selftest=all` (or whichever route covers `runDesignGeometryFixtures`) shows zero regressions, and every existing production-SVG golden hash is unchanged.
7. `SCHEMA_VERSION`, `STORAGE_KEY`, `BACKUP_FORMAT`, `ENCRYPTED_BACKUP_FORMAT` are unchanged.
8. A pre-phase backup (missing the new draft fields) still imports and normalizes safely.

---

## 28. Risks

- **Risk**: the new liner-allowance validation subtly diverges from `dice-tray`'s existing pattern, creating two slightly different "measurement-only liner" behaviors in the app — mitigated by explicitly modeling the new validation on the existing `dice-tray` code rather than writing it independently.
- **Risk**: choosing default dimensions that don't actually fit a realistic 12×12 inch sheet with sensible margin — mitigated by the existing `designLayoutSizeWarning` safety net and by the explicit manual-check requirement in §26.
- **Risk**: physical testing reveals the reused finger-width/clearance defaults don't suit the new, smaller size — this is expected and explicitly planned for in §20; it is a "starting point," not a regression, and does not block shipping the generator in a clearly-labeled prototype state.
- **Risk**: scope creep toward the deferred stacking lip or the deferred evidence/promotion pipeline during implementation — mitigated by this report's explicit non-goals list (§23) and by keeping the phase's acceptance criteria (§27) narrow.

---

## 29. Rollback strategy

Because this phase is entirely additive (new template, new functions, new fixtures, no modification to any existing function or schema), rollback is a plain code revert with no data migration to reverse and no persisted state to unwind — the same low-risk rollback profile as the GUI Clarity terminology phase, and simpler than any Security or Inventory phase's rollback story, for the same underlying reason (nothing shared is changed).

---

## 30. Unverified areas

- No browser was launched for this review; all findings are from source and fixture-text reading.
- The exact final starting-point default dimensions for the new generator are a product/copy decision this review does not make (only that they should be smaller than T3B's 80×60×25 mm and should not reuse T3B's specific physical-evidence values as defaults).
- Whether a full-file grep would surface any additional Finished-View-adjacent function this review's targeted greps missed was not exhaustively re-verified beyond the specific function names confirmed present.
- Real-world sheet-nesting efficiency and cut time for the new generator's default size were not measured (no browser/hardware access in this review).

---

## 31. Suggested independent audit boundary

A follow-up, read-only post-implementation verification pass should: confirm every existing production-SVG golden hash is unchanged (proving §22.21's "no existing-generator impact" claim empirically, not just architecturally); confirm the new generator's own fixtures pass and its Finished View is genuinely screen-only; confirm the new `projectDraftFromDesign` branch and `designDefaults()` additions don't regress legacy-draft normalization; and perform the manual browser checks in §26 that this review could not perform. This mirrors the same "focused post-implementation verification" pattern already used for the Tabletop Accessories, Security, and GUI Clarity phases in this repository's `docs/` history.

---

## 32. Concise Codex implementation handoff

> Implement the "Stackable Token & Dice Tray" Designs generator in `C:\Genmitsu L8 Tracker\index.html`, per `docs/FINISHED_VIEWS_PRACTICAL_GENERATOR_ARCHITECTURE_REVIEW_2026-07-23.md` §22. Add a new template (working id e.g. `tabletop-token-tray`) registered once in `designTemplateSelect` and once in `buildDesignResult`'s dispatch chain. Implement `buildTabletopTokenTrayDesignResult` by composing two internal sub-drafts and calling the existing, unmodified `buildTabletopRectangularShellDesignResult` (T2 shell) and the existing T3A rail/false-bottom validation/panel pattern (`buildTabletopStorageFitCouponDesignResult`), exactly the way `buildTabletopStorageTrayDesignResult` (T3B, `index.html:5275`) already composes them — do not modify either reused function. Add one new, small liner-allowance parameter set (type/clearance/thickness) modeled directly on `dice-tray`'s existing `diceInsertType`/`diceInsertClearance`/`diceInsertThickness` fields and validation (`index.html:3328-3330`, `3899-3905`) — non-wood liners remain measurement-only, never cut geometry. Implement a new `buildTabletopTokenTrayFinishedViewSvg` modeled directly on `buildTabletopStorageTrayFinishedViewSvg` (`index.html:5299`), reusing `tabletopAccessoryProjectPoint` for projection, screen-only, with the same guard/degrade-safely contract. Wire the new template into `designPreviewModeForTemplate`'s existing `finished-view`-mode array and into the Finished View dispatch site (`index.html:5700` region) the same way T1–T3B already are. Add one new branch to `projectDraftFromDesign`. Do **not** implement a stacking lip, do not add cut geometry for cork/felt/leather, do not add a staged physical-evidence/promotion pipeline for this product, and do not modify any existing generator, golden, or schema constant. Add fixtures per §25 (registration, defaults, validation boundaries, exact panel dimensions, non-overlap/closed-path checks, Finished View determinism and screen-only isolation, production-SVG byte-identity across preview-mode toggles, Projects handoff, legacy-draft normalization), then re-run the full Designs/production-golden suite and confirm zero regressions before calling the phase complete. Treat every shipped dimension as a starting point requiring a physical coupon/prototype test (§20) before any "proven" claim is made anywhere in the UI.
