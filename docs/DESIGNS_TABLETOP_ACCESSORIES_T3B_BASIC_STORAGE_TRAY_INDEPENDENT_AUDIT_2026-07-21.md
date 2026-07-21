# Tabletop Accessories T3B — Basic Storage Tray — Independent Audit

**Date:** 2026-07-21
**Auditor:** Claude Opus 4.8 (independent, read-only — nothing implemented, nothing corrected)

## 1. Actual repository state

`git rev-parse HEAD` = `12c430404288fb11fdf6500fb4db0e0cdd417443` — **"Add T3A storage fit evidence workflow"** (confirmed; this is the committed baseline, and it already includes the T3A evidence workflow plus the fixture correction that resolved the two stale assertions I flagged in the prior T3A-evidence audit). Branch `main`; `origin/main...main` = `0 0` (synchronized). Nothing staged.

Uncommitted T3B implementation, tracked diff: `CHANGELOG.md` (+2), `README.md` (+9/−… wording), `index.html` (+167/−23 net, ~179 changed lines). Untracked set: the long-standing unrelated docs/LightBurn/utility/debug files plus the T3B implementation report and this audit — all preserved. No unrelated changes present. `git diff --check` clean; `python html.parser` parses `index.html`.

All ten requested reports were read in full, including the T3B implementation report and the T3A fixture-correction report (which confirms the prior audit's High finding was resolved and committed).

## 2. Files and functions changed by T3B

Confirmed directly from `git diff` (not from the report): purely additive.

- `designDefaults()`: new `tabletopTray*` default keys (interior 80/60/25, nominal 3, measured 2.88, clearance −0.075, finger 9, rail 10, false-bottom 2.88, panel clearance 0.40, spacing 15, material id/label). No existing default changed.
- `tabletopAccessoryProductRegistry`: appends `tabletop-storage-tray`.
- `renderDesigns()`: new `tabletopStorageTray` branch, form section, note; existing branches untouched.
- `normalizeDesignDraft`: new `else if (template === 'tabletop-storage-tray')` numeric-parse block (mirrors the T3A block).
- New builders: `buildTabletopStorageTrayDesignResult`, `buildTabletopStorageTrayFinishedViewSvg`.
- New evidence helpers: `tabletopStorageTrayShellEvidenceMatches` (a **tighter filter wrapping** the existing `tabletopRectangularShellEvidenceMatches`), `tabletopStorageTrayRecommendationHtml`.
- `buildDesignResult` routing, `designPreviewModeForTemplate`, `designPreviewSelectorHtml`, `designOperationGuidance`, summary metrics, assembly text, preview resolver, `projectDesignTemplateName`, `projectDraftFromDesign` note block: additive `tabletop-storage-tray` handling.
- New fixture `runTabletopStorageTrayFixtures` (36 assertions) registered under `?selftest=tabletop-storage-tray`; the pre-existing template-selector count assertion bumped 14→15.

**No protected function was modified.** `buildBoxModel`, `buildFingerPattern`, `buildTabletopRectangularShellDesignResult`, `buildTabletopStorageFitCouponDesignResult`, `tabletopRectangularShellEvidenceMatches`, `tabletopStorageEvidenceMatches`/`tabletopStorageExactMatch`, `serializeTabletopAccessorySvg`, `layoutDesignPanelRows`, `designSvgValidation`, `productionMachineIdentityMatches` — all **called**, none changed. Every change is necessary (a new template needs its own builder/form/routing) and safe (no shared behavior altered).

## 3. Architecture-conformance assessment

**T3B genuinely composes the existing verified builders; it does not duplicate their formulas.** `buildTabletopStorageTrayDesignResult` constructs an adapted `shellDraft` and `storageDraft`, calls `buildTabletopRectangularShellDesignResult(normalizeDesignDraft(shellDraft))` and `buildTabletopStorageFitCouponDesignResult(normalizeDesignDraft(storageDraft))`, and **reuses their generated panel paths unchanged**. I independently confirmed this at runtime (see §4): the ten T3B panel paths are **byte-identical** to the five standalone T2 shell panels and the five standalone T3A coupon panels. No rail-length, false-bottom, or finger-width formula is re-implemented in the tray builder — the values flow out of the composed component metrics.

Identities are deliberate and internally consistent: template `tabletop-storage-tray`, public name **Tabletop Storage Tray**, generator `tabletop-storage-tray-t1`, construction `t2-floor-owned-shell-plus-bottom-supported-four-rail-removable-panel-t1`, with `outerShellConstruction:'closed-three-way-corner'` and `storageMechanismConstruction:'bottom-supported-four-rail-removable-panel-t1'` recorded separately. Bounds are respected: exactly 5 shell + 4 rails + 1 false bottom, no cork, no complete-tray recorder, no new persistence collection, no schema change (all verified — §9).

## 4. Independent geometry calculations

Recomputed from source composition and cross-checked against the standalone component builders at runtime (not merely against the report). The composed T2/T3A finger geometry is the same actual floor-finger geometry independently verified in prior audits: width-axis 9.52888888888889 mm (9 fingers/85.76 mm), depth-axis 9.394285714285715 mm (7 fingers/65.76 mm).

| Quantity | Independently confirmed | 
| --- | --- |
| Finished exterior | 85.76 × 65.76 × 27.88 mm ✓ |
| False bottom | 79.60 × 59.60 mm ✓ |
| Per-side clearance | 0.20 mm (= 0.40/2) ✓ |
| Front/back rail length | 60.942222 mm (= 80 − 2×9.528889) ✓ |
| Left/right rail length | 41.211429 mm (= 60 − 2×9.394286) ✓ |
| Rail overlap | 2.68 mm (= 2.88 − 0.20) ✓ |
| Remaining visible wall | 12.12 mm (= 25 − 10 − 2.88) ✓ |
| Composition | FLOOR, FRONT, BACK, LEFT, RIGHT + RAIL-FRONT, RAIL-BACK, RAIL-LEFT, RAIL-RIGHT + FALSE-BOTTOM = 10 ✓ (5 shell + 5 storage) |
| Floor-owned closure | `floorCornerOwnership===true`, sourced from the composed shell metrics ✓ |
| Clearance applied once per axis | 79.6 = 80−0.4 and 59.6 = 60−0.4 (once each) ✓ |
| Kerf | not added to false-bottom clearance (kerf input pinned `'0'`; storage builder adds no kerf) ✓ |
| Component byte identity | shell 5 panels === standalone T2; storage 5 panels === standalone T3A coupon (`M 0 0 H 60.942222 V 10 …`, `M 0 0 H 41.211429 V 10 …`, `M 0 0 H 79.6 V 59.6 …`) ✓ |
| Invalid component blocks combined output | undersized shell, sub-minimum rail, excessive false bottom, non-finite, and zero/excessive clearance each block (§7) ✓ |
| Panel/rail/shell assembled overlap | rails inset ≥9.39 mm from corners (far beyond the ~2.88 mm corner-joint zone); false bottom 79.6×59.6 clears the clean 80×60 interior; rests on 2.68 mm rail ledges — no improper assembled overlap (geometry unchanged from the physically-audited T3A mechanism) ✓ |

## 5. Exact SVG assessment

Verified against the **actual intercepted production download** in a disposable headless browser (not only the fixture): **2,860 bytes** (UTF-8), FNV-1a **`3e256fad`**, **deterministic** (byte-identical on re-render). Matches the expected new golden exactly.

- **Path count: 10** closed red (`stroke="#ff0000"`) paths; no blue guide layer, no green layer unless labels enabled.
- **Piece containment + pairwise non-overlap:** independently computed all ten pieces' layout bounding boxes → **0 overlapping pairs**; global bbox (10,10)–(196.52,286.12) sits inside the 206.52 × 296.12 mm sheet with a 10 mm margin. (Note: the fixture's "…contained, and non-overlapping" assertion actually only checks containment; true pairwise non-overlap is guaranteed by the shared, long-proven `layoutDesignPanelRows` packer and confirmed here independently — see §11 finding.)
- **Exact-scale:** `width="206.52mm" height="296.12mm"`, viewBox `0 0 206.52 296.12`; `sheetCount:1`; no rotation/scaling/multi-sheet.
- **Metadata/filename:** `l8-tabletop-storage-tray-<date>.svg`, `image/svg+xml;charset=utf-8`.
- **Preview/production separation:** the downloaded SVG contains **no** `tabletop-storage-tray-shell` / `storage-tray-false-bottom` / `tray-rail` / `finished-view` identifiers — confirmed on the intercepted download. Finished-Assembled markup, helpers, and decorative geometry cannot enter the production SVG.

## 6. Control and validation assessment

The T3B form exposes exactly the bounded controls (interior W/D/H, nominal thickness, measured thickness, signed shell clearance, preferred finger width, rail height, false-bottom thickness, total false-bottom clearance, panel spacing, material label, Library profile select, optional labels), and **no** complete-tray recorder button. Decimal entry, `step="any"`, min/max attributes, and the signed-clearance path (`designJointClearanceNumber`) follow existing Designs conventions; machine identity flows through the shared `tabletopCouponGeneratedMachineSnapshot`. The normalizer mirrors the T3A block, so unit/focus/decimal behavior is consistent with the rest of Designs.

## 7. Control-validation (real UI)

Exercised in the browser: undersized interior (20 mm < 40 min), rail below minimum (9.99 mm), false bottom leaving no visible wall (15 mm), and non-finite (`abc`) each produced a visible **Error**, a **disabled download button**, and "Preview unavailable"; restoring valid values re-enabled a valid result. The app **blocks unsafe production** rather than clamping, reinterpreting, or exporting invalid geometry.

## 8. Evidence-boundary assessment

T3B **reuses the existing exact matchers without weakening or broadening them.** `tabletopStorageTrayShellEvidenceMatches` calls the unmodified `tabletopRectangularShellEvidenceMatches` and then **further requires** `evidenceStage==='Shell-proven'`, `exactConstruction===true`, `floorCornerOwnership===true`, exact material-profile id, exact interior W/D/H, exact measured thickness, exact signed clearance, and exact width/depth/vertical actual finger widths (`1e-6`) — i.e. it tightens, never loosens. The storage side reuses the T3A `tabletopStorageEvidenceMatches`/`tabletopStorageExactMatch` unchanged. Cork and complete-tray are **static "Unproven"** strings.

For the exact reference configuration I confirmed (fixture-captured, real matchers): **Outer shell: T2 Shell-proven — Exact**, **Storage mechanism: Storage Fit Coupon-proven — Exact**, **Cork fit: Unproven**, **Complete Tabletop Storage Tray: Unproven**. All independent mismatch tests pass and remove **only** the proof they invalidate:

- Requested width/depth/usable-height, material profile, measured thickness, signed shell clearance, preferred/actual finger geometry → remove **both** component statements (these feed both matchers).
- Rail height, false-bottom thickness, total clearance, nominal thickness, T3A generator identity, T3A construction identity, snapshot Floor-ownership → remove **only** the T3A mechanism statement, T2 shell statement retained.
- T2 shell generator/construction identity → removes **only** the T2 statement, T3A retained.
- T2 evidence cannot prove the storage mechanism; T3A evidence cannot prove the shell (both cross-direction checks pass); matching T2+T3A never yields cork or complete-tray proof; `productionEvidenceKinds` does **not** contain `tabletop-storage-tray`.
- **Generation, preview, Finished View, export, and Project handoff do not create, modify, promote, reinterpret, or delete evidence:** the fixture compares the full evidence-bearing state, `localStorage`, and `backupObject()` before/after and they are byte-identical; existing T2 and T3A records remain unchanged and accessible.

## 9. Project-handoff assessment

`Start Project from Design` produces an **unsaved, reviewable** draft named "Tabletop Storage Tray" with `material` blank, `thicknessValue` 2.88, `quantity` 1, and notes carrying: T3B identity (`tabletop-storage-tray / tabletop-storage-tray-t1`), both component construction identities, requested + finished dimensions, measured/nominal thickness, shell clearance, rail height, both rail lengths, false-bottom thickness, total/per-side clearance, remaining visible wall, production-layout dimensions, piece count (10 = 5+5), material/machine snapshots, and separately-scoped evidence statements (**T2 Shell-proven — Exact / Storage Fit Coupon-proven — Exact / cork Unproven / complete tray Unproven**), plus concise assembly guidance and the explicit "Project creation is not evidence." No Project schema field was added (the fixture asserts no `schema|evidence|storage|svg` key on the draft), and a Project is never represented as physical evidence.

## 10. Persistence and compatibility assessment

`APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `BACKUP_FORMAT`, and **`SCHEMA_VERSION` (still 3)** are unchanged (not in the diff). `productionEvidenceKinds` is unchanged (no tray kind). No new persisted collection, no normalization/migration, no backup-format change. Version-1/version-2/version-3 loading is unaffected (T3B touches no load/persist/backup path). I seeded a **version-3 state** (T2 promoted shell evidence + a T3A raw record) into disposable storage and reloaded via direct `file://`: the app reached `document.readyState==='complete'` with no temporal-dead-zone/runtime error, existing evidence remained accessible, and T3B was navigable — the new template introduces no startup-order dependency (the T3B fixture also asserts this). Replace/Merge/backup export/import behavior is inherited unchanged.

## 11. Fixture-quality assessment and focused totals

Fresh in a disposable headless Edge session: **T3B `runTabletopStorageTrayFixtures`: 36 / 0.** Its 36 assertions genuinely prove the intended behavior — exact identities; the byte-identical composition of both component panel sets; the ten-piece order; exact envelope/false-bottom/rail/overlap/visible-wall values; the pinned 2860/`3e256fad` golden with determinism; the screen-only Finished View with no production leakage; invalid-input blocking; and eighteen independent evidence-boundary mismatch cases in both directions plus cork/tray-unproven and no-evidence-mutation checks.

Other groups, fresh this audit: T2 rectangular shell **93/0**, T2 closed corners **13/0**, T3A coupon **23/0** (restored by the committed fixture correction), T3A storage results **14/0**, T2 shell promotion **16/0**, T2 shell matching **16/0**, Designs→Projects handoff **17/0**, Designs geometry/production goldens **1093/0**, Tray model **264/0**, Dice Tray system **92/0**.

**Fixture-quality nits (Low/Informational):** (a) the "production layout is exact-scale, contained, and non-overlapping" assertion checks only containment, not pairwise non-overlap — the name slightly over-claims; I verified true pairwise non-overlap independently (0 overlapping pairs). (b) The T3B golden pin is present as a literal (`2860`/`3e256fad`), unlike the T3A coupon (897/`b5c549ee`), which remains reported-not-pinned — a pre-existing T3A characteristic, not a T3B regression.

## 12. Normal full-suite totals and aggregate reconciliation

Every exposed `run*Fixtures()` function invoked once (46 functions): **3244 passed / 0 failed** (the documented accessibility/responsive keyboard-nav timing artifact did not trigger in this run). The **44 `?selftest=all`-registered groups sum to exactly 2964 / 0**, matching the report and README.

**Reconciliation (no group disappeared):** the 46 exposed functions minus the two that are dispatched via other routes — `runTrayModelFixtures` (264, via `dice-tray-system`) and `runPromotionTargetSwitchFixtures` (16, via `promotion`) — equal the 44 registered groups (3244 − 280 = 2964). The "higher aggregate counts" in earlier T3A-era reports (3198/3206, and 3244 here) are precisely those all-exposed-function batches that include the extra 280 assertions from those two other-route functions; their occasional "10 failures" were only the keyboard-focus timing artifact in accessibility/responsive, never a product failure. Relative to the T3A-evidence era (45 exposed / 43 registered), T3B added **exactly one** exposed function and **one** registered group (`runTabletopStorageTrayFixtures`, +36). No registered group is missing, and the total is not inflated by aliases or duplicate executions.

## 13. Production goldens

Independently verified this audit — every registered production golden literal present in source and its group green:

| Golden | Expected | Confirmed |
| --- | --- | --- |
| Legacy pocketed shell | 2181 / `2ef9606b` | ✓ |
| T1 coupon | 2992 / `4f543f95` | ✓ |
| T2 shell | 2337 / `ed5d6f6e` | ✓ |
| **T3B storage tray** | **2860 / `3e256fad`** | ✓ (literal pinned; intercepted download matches) |
| Dice Tray | 1726 / `51a55721` | ✓ |
| Alternate Dice Tray | 1054 / `41697123` | ✓ |
| Divider Tray | 1965 / `a55dda6e` | ✓ |
| Wall-to-base joint coupon | 1551 / `d9ffc278` | ✓ |
| T3A storage fit coupon | 897 / `b5c549ee` | deterministic, reported-not-pinned (pre-existing T3A trait; `runTabletopStorageFitCouponFixtures` 23/0) |

Additional goldens exercised green by Designs geometry (1093/0): Dice bottom-cover `8e2ea3f4`; Concealed Cleat corner `88721533`/cut `c16a9ef5`, full box `2316430b`; decorated Finger Box `d9512d1c`; Sliding-lid `4a7ab718`/inside `a892f91c`/lid `6181bc75`; Drawer Cabinet `a6dd23dc`/`36a41b07`/`8c286797`/guides `dd3ff0bd`/grid `0814ca2e`/separate `956ad870`/`bad8b97f`/custom `500396df`/`69b0ce8b`/legacy `7494c326`/`b158a794`. **No existing production SVG changed.**

## 14. Browser scenarios actually tested

Automated headless Microsoft Edge (`--headless=new`), disposable `--user-data-dir`, direct `file:///C:/Genmitsu%20L8%20Tracker/index.html`, driven via the Chrome DevTools Protocol. No human/manual browser interaction was performed. Scenarios completed: clean-data startup and full self-test execution; version-3 seeded-data startup + reload with no exception and existing evidence accessible; navigation to T3B; valid default generation; exact physical-reference inputs; the ten-piece Cut Layout; Finished Assembled view rendering (`tabletop-storage-tray-finished-view`) and its selector resolution; **intercepted production SVG download** (2860 bytes / 10 paths / red / no leakage / 206.52×296.12 mm); the result-summary evidence panel (independent T2/T3A/cork/tray statements); invalid-input blocking (four cases) with disabled download; and no-TDZ reload. Evidence-boundary mismatch matrix and Project-handoff/no-evidence-mutation were exercised through the in-page fixture harness (which executes the real `buildDesignResult`, `renderDesigns`, `designResultsHtml`, `projectDraftFromDesign`, matchers, and Finished-View builders). `document.readyState` reached `complete`; no console/runtime errors were observed in the scenarios run.

## 15. Findings by severity

**Blocking:** none.
**High:** none.
**Medium:** none.

**Low:**
- **[index.html, `runTabletopStorageTrayFixtures`, "production layout … non-overlapping" assertion]** The assertion name claims pairwise non-overlap but the check only verifies each piece's containment within the sheet. **Impact:** a future layout regression that overlapped two pieces while keeping both in-bounds could pass this named assertion. **Not a defect today** — I independently confirmed 0 overlapping pairs, and the shared `layoutDesignPanelRows` packer (used by all multi-piece templates) guarantees separation. Blocks commit: no. Blocks prototype cut: no. Smallest correction: add an explicit pairwise bounding-box non-overlap check (or rename the assertion to "contained").

**Informational:**
- **[index.html, `buildTabletopStorageTrayFinishedViewSvg`, `left` polygon]** The screen-only Finished-View `left` face uses `p(0,0)` where the analogous T3A view uses `p(0,0,0)`. Since `p`'s third arg `z` defaults to `0`, these are numerically identical; the view is screen-only and cannot enter production. Purely cosmetic inconsistency.
- The T3B golden is pinned as a literal; the older T3A coupon golden (897/`b5c549ee`) remains reported-not-pinned. Consistent with the prior T3A audit; not introduced by T3B.

## 16. Unverified areas

- Physical fit/strength of the assembled tray is intentionally unproven (this is a prototype-generation feature; the app correctly states "complete tray Unproven"). No physical validation was performed by this audit.
- Real click-through of the destructive Replace/Merge import path *for a T3B-bearing backup specifically* was not repeated here (T3B adds no persisted collection, so backup content is unchanged from the already-audited T3A schema); backup/import safety is inherited unmodified and was covered in the T3A-evidence audit.
- Browser checks were automated/headless via CDP only; no manual visual interaction.

## 17. Required corrections

None are required before committing or cutting a prototype. The single Low finding (assertion-name over-claim) is an optional test-hardening improvement, not a defect.

## 18. Commit-readiness and first-prototype-cut readiness

**Commit-ready:** yes. The change is purely additive, composes the verified T2 and T3A builders (byte-identical component reuse), changes no protected function, alters no schema/persistence/evidence/existing-golden, passes the full registered suite at 2964/0 with the T3B group at 36/0, and pins a correct new golden. **First-prototype-cut ready:** yes — the exact-scale ten-piece 2860-byte SVG is internally verified (geometry, containment, non-overlap, no preview leakage); physical fit remains an intentional prototype validation step, and the UI never overstates it.

## 19. Exact final verdict

**1. READY TO COMMIT AND CUT A PROTOTYPE** — No correction is required before committing the T3B implementation and cutting the first Tabletop Storage Tray prototype. The generator safely composes the verified T2 Floor-owned shell and the T3A four-rail removable false-bottom mechanism into one accurate, exact-scale, cut-ready ten-piece SVG (2860 bytes / `3e256fad`), with independent exact evidence statements, no schema or evidence changes, no protected-function or existing-golden changes, and a clean full registered suite (2964 / 0 across 44 groups). The only Low finding is an optional fixture-name hardening; physical fit of the assembled tray remains, correctly, an unproven prototype step.
