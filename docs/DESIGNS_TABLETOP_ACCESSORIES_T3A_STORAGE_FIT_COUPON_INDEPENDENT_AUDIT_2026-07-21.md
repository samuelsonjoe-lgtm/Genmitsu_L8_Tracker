# Tabletop Accessories T3A — Storage Fit Coupon — Independent Audit

**Date:** 2026-07-21
**Auditor:** Claude Opus 4.8 (independent, read-only)
**Committed baseline:** `ec8f35f` — "Fix Tabletop shell startup initialization" (confirmed)
**Implementation under audit:** uncommitted working-tree changes to `index.html`, `README.md`, `CHANGELOG.md` plus the untracked implementation report.

This was an independent verification pass, not an implementation or correction pass. No tracked file was edited; nothing was staged, committed, pushed, reset, cleaned, or stashed. Only this report was created. All runtime checks used a disposable headless Edge profile that was deleted afterward; Joe's real browser profile and `localStorage` were never touched.

## 1. Actual repository state

- `git rev-parse HEAD` = `ec8f35f4869e806493c3cf510453a68add4058df`; branch `main`; `origin/main...main` = `0 0` (synchronized).
- Nothing staged. Tracked working-tree changes: `CHANGELOG.md` (+2), `README.md` (+1/−2 net wording), `index.html` (+149/−19, 3 hunks of additions). `git diff --check` clean (only benign CRLF-on-next-touch notices).
- Untracked set unchanged apart from the two new T3A docs (implementation report + this audit). All unrelated modified/untracked files preserved.
- `index.html` parses under Python's HTML parser; the page loads and executes with no runtime exception under a direct `file://` startup.

## 2. Files changed by the implementation

All changes are **purely additive**; no existing generator, serializer, matcher, or golden was modified.

| Area | Change |
| --- | --- |
| `designDefaults()` | New `tabletopStorage*` default keys (interior 80/60/25, nominal 3, measured 2.88, clearance −0.075, finger 9, rail 10, false-bottom 2.88, panel clearance 0.40, spacing 15, material id/label). No existing default altered. |
| `tabletopAccessoryProductRegistry` | Appends `tabletop-storage-fit-coupon`. |
| `renderDesigns()` | New `tabletopStorage` branch, form section, and note; existing branches untouched. |
| `normalizeDesignDraft` | New `else if (template === 'tabletop-storage-fit-coupon')` numeric-parse block. |
| New builders | `tabletopStorageFitCouponPanel`, `buildTabletopStorageFitCouponDesignResult`, `buildTabletopStorageFitCouponFinishedViewSvg`. |
| `buildDesignResult` | New routing branch before the shell branch. |
| `buildFingerBoxFinishedViewSvg` | Early-returns the storage finished view when `metrics.tabletopStorageFitCoupon`. |
| Preview/summary/assembly | Additive `tabletopStorage` handling in preview-mode gating, selector, summary metrics, and assembly-order text. |
| Project handoff | New `else if (template==='tabletop-storage-fit-coupon')` note block; no Project schema change. |
| Fixtures | New `runTabletopStorageFitCouponFixtures`, registered under `?selftest=tabletop-storage-fit-coupon`/`all`; template-selector count fixtures bumped 13→14. |
| Docs | README + CHANGELOG entries; both accurate to the implementation. |

## 3. Findings by severity

**Blocking:** none.

**Major:** none.

**Minor / non-blocking observations:**

1. **Rail-inset provenance is proven by the real generator but only self-referentially asserted in the fixture.** The fixture checks `RAIL-FRONT.width === interiorWidth − 2×cornerInsets.width`, where `cornerInsets.width` is the same value the builder used, so the assertion alone would still pass if the finger source were wrong. The **code is correct** — I independently confirmed (§4) that `cornerInsets` equals the real T2A shell generator's floor-pattern `actualWidth`. A future hardening could assert `cornerInsets.width` equals the T2A shell's `widthAxisFloor.actualWidth` directly. Test robustness only.
2. **The T3A production SVG (897 bytes / `b5c549ee`) is deterministic but not pinned as a regression golden.** The implementation report states this deliberately ("a new output, not a replacement golden"). Acceptable; a future byte/hash pin would guard against silent drift once the mechanism is proven.
3. **Documentation nit (auditor-side, not code):** the previously circulated "Joint Fit Coupon 7764 / `db7ea7e9`" pin does not correspond to any current literal in `index.html`. The current pinned joint-coupon golden is the wall-to-base coupon `1551` bytes / `d9ffc278`. T3A changes neither.

## 4. Exact geometry calculations (independently reproduced at runtime)

Inputs: interior 80 × 60 × 25 mm, measured thickness 2.88 mm, shell joint clearance −0.075 mm, preferred finger 9 mm, rail height 10 mm, false-bottom thickness 2.88 mm, total panel clearance 0.40 mm.

**Actual floor-finger widths (cross-checked against the real `tabletop-rectangular-shell-prototype` generator, not just T3A's internal call):**

- Width-axis floor edge: panel length **85.76 mm** (= 80 + 2×2.88), **9 fingers**, `actualWidth = 9.52888888888889 mm`.
- Depth-axis floor edge: panel length **65.76 mm** (= 60 + 2×2.88), **7 fingers**, `actualWidth = 9.394285714285715 mm`.

The T2A shell fixture reports the identical `widthAxisFloor.actualWidth = 9.52888888888889` (count 9). This confirms the rail insets derive from genuine target-shell finger geometry — **not** the nominal 9 mm preferred width and **not** a guessed value (a guess of 9.0 would have produced 62.0 / 42.0 mm rails; the real output is 60.94 / 41.21 mm).

| Piece | Generated size (mm) | Derivation | Verified |
| --- | --- | --- | --- |
| RAIL-FRONT | 60.942222 × 10 (×2.88 thick) | 80 − 2×9.528889 | ✓ |
| RAIL-BACK | 60.942222 × 10 | same | ✓ |
| RAIL-LEFT | 41.211429 × 10 | 60 − 2×9.394286 | ✓ |
| RAIL-RIGHT | 41.211429 × 10 | same | ✓ |
| FALSE-BOTTOM | 79.6 × 59.6 (×2.88 thick) | (80 − 0.40) × (60 − 0.40) | ✓ |

- **Corner setback at each rail end:** 9.528889 mm (front/back), 9.394286 mm (left/right) — one full actual floor-finger width per end.
- **Total clearance:** 0.40 mm applied **once** per dimension; **per-side equivalent 0.20 mm** (0.40 / 2). Not double-applied.
- **Rail overlap of the false bottom on each rail:** 2.68 mm (= 2.88 material − 0.20 per-side) ≥ minimum 1.0 mm (= max(1, 2.88/3)). Meaningful overlap on all four rails.
- **Remaining visible wall height:** 12.12 mm (= 25 − 10 − 2.88) > 0.
- **Rail height:** 10 mm = minimum max(3×2.88 = 8.64, 10) = 10 mm.
- **Outer envelope (reported):** 85.76 × 65.76 × 27.88 mm.
- **Production SVG:** 156.884444 mm × 129.6 mm; **897 bytes; FNV-1a `b5c549ee`; deterministic** (byte-identical on re-render).
- **No kerf compensation** is added anywhere (kerf is not a T3A input; the sizes above contain no kerf term).

**Collision / seating analysis (the flagged risk):** The T2 wall-to-wall vertical finger joints occupy only the wall-thickness corner posts; both walls' inside faces are flush at the 80 × 60 interior boundary, so **no finger intrudes into the nominal 80 × 60 cavity at any elevation.** The 79.6 × 59.6 panel therefore descends cleanly from the open top with 0.20 mm/side, and it is not trapped by any inward-projecting tab. The floor-owned three-way corner joints live at the floor plane (z ≈ 0), well below the 10 mm rail-top elevation, so they do not obstruct the panel either. The panel is wider/deeper than the clear gap between opposing rail inner faces (74.24 mm × 54.24 mm) and rests on the 2.68 mm ledges; it does not fall through. Rail ends are inset 9.39–9.53 mm — far beyond the ~2.88 mm corner-joint contamination zone — so rails glue against clean wall faces and never interfere with the wall-to-wall or floor-to-wall fingers. **No collision exists.** The design does not require cutting another shell; the five pieces are manufacturable from 2.88 mm plywood.

## 5. Production-output inspection

Verified on the actual serialized SVG:

- Exactly **five** structural red-cut pieces, in order `RAIL-FRONT | RAIL-BACK | RAIL-LEFT | RAIL-RIGHT | FALSE-BOTTOM`; four independent rail identities + one false bottom.
- **No** T2 shell panels (`panel-(FLOOR|FRONT|BACK|LEFT|RIGHT)`), **no** `tabletop-storage-context-shell` / `storage-shell` / `storage-false-bottom` context geometry, **no** cork, notch, hole, groove, pocket, or engraving.
- All panel paths are closed (`Z`), stroke `#ff0000`, single `<g id="cut">`; no blue guide layer (`#0000ff`), no duplicate cut paths.
- Exact-scale: `width`/`height` in `mm`, viewBox in mm; no scaling, rotation, or multi-sheet split (`sheetCount: 1`). All pieces within declared bounds (validated by the existing `designSvgValidation`).
- Filename `l8-tabletop-storage-fit-coupon-<date>.svg` follows the established sanitized convention.
- With labels enabled: exactly 5 green (`#00ff00`) `#assembly-labels` paths; the red panel path strings are **byte-identical** with and without labels (labels do not alter structural geometry).
- **Deterministic default output = 897 bytes / `b5c549ee`**, reproduced exactly. No discrepancy.

## 6. UI and Finished View findings

The summary and notes communicate: retrofit-pieces-only; T2 shell not included; existing-shell requested interior (80×60×25) and derived outer envelope; all five pieces with dimensions; rail/storage height and its minimum; false-bottom thickness; total and per-side clearance; rail overlap and remaining visible wall height; production SVG size; the 100%-import exact-scale notice; the work-area advisory; the assembly order; **outer-shell evidence scope**; and an explicit **"Unproven — result recording and promotion deferred"** mechanism-evidence line.

The screen-only Finished View renders the contextual existing shell (`tabletop-storage-context-shell`), all four rails, the storage cavity, and the false bottom seated on the rails, with pull-tab/"not in downloaded cut SVG" captions. Confirmed: **none of that contextual geometry leaks into the production SVG.**

## 7. Evidence verdict

The deferral conclusion is **accurate and honestly presented.**

- `productionEvidenceKinds` does **not** include `tabletop-storage-fit-coupon` (asserted and confirmed) — no new source type is partially accepted, normalized, imported, or promoted.
- `tabletopJointFitCompatible` and the T1/T2 matchers are untouched (not in the diff); existing evidence records and meanings are unchanged; T1↔T2 identity separation still holds (all matching/promotion fixtures pass).
- The builder attaches a `compatibilityIdentity` describing the **outer shell** as `tabletop-accessory-t2`, and separate `outerShellEvidenceScope` / `mechanismEvidenceStage` strings that state the shell evidence "does not verify this storage mechanism" and that the mechanism is "Unproven." T3A never inherits Shell-proven status.
- The form exposes **no** physical-result recorder (`data-record-tabletop-shell-result` absent; "Record Physical Shell Result" absent) — no nonfunctional or misleading recording workflow.
- No schema, `STORAGE_KEY`, `SCHEMA_VERSION`, or `BACKUP_FORMAT` change. Deferring dedicated T3A recording genuinely would require a new persisted record shape; the interface does not pretend otherwise. **Acceptable for a physical prototype.**

## 8. Project-handoff verdict

`Start Project from Design` produces a Project named "Tabletop Storage Fit Coupon" with `material` blank (unconfirmed), `thicknessValue` 2.88 mm, `quantity` 1, `jobType` cut, and notes that: enumerate all five retrofit pieces "for an existing T2 shell with requested interior 80 × 60 × 25 mm"; give rail height/minimum and the actual finger insets; give the false-bottom size/thickness and total/per-side clearance; and state outer-shell-evidence scope plus the deferred/unproven mechanism status. No Project-schema change.

This **accurately** distinguishes the existing shell from the five generated pieces, counts only the retrofit pieces as material (the shell is described as existing, never as new stock), does not imply the complete tray was generated, preserves the outer-shell-vs-mechanism evidence distinction, and produces no misleading cost/material accounting. Sufficient under the existing editable-notes handoff contract.

## 9. Fixture-quality assessment

23/23 focused assertions, and they prove the intended behavior, not just totals. Against the enumerated plausible defects:

| Plausible defect | Caught by | 
| --- | --- |
| Rails use a guessed finger width | Insets flow from a real `buildBoxModel(...,floorCornerOwnership:true)` call; independently cross-checked equal to the T2A shell (§4). Fixture pins RAIL widths to `interior − 2×inset`. (Hardening noted in §3.1.) |
| Clearance subtracted twice | False bottom pinned to 79.6 × 59.6 and `perSideClearance === 0.2`. |
| Only two rails support the panel | Exact five-piece id-order check + positive rail-overlap block. |
| False bottom overlaps no usable rail | `railOverlap === 2.68` and `< minimumRailOverlap` blocked; excessive-clearance case asserted blocked. |
| Contextual shell leaks into production | "production excludes outer shell panels and screen-only context" regex assertion. |
| T2 evidence presented as T3A proof | Evidence-scope/`Unproven` assertions + `productionEvidenceKinds` exclusion. |
| Handoff treats shell as new material | Notes-content assertion (retrofit-pieces/deferred). |
| Saved T2 record → startup exception | Storage/backup byte-identical assertion; clean seeded startup path unaffected (T3A does not touch `loadState`/normalization/init order). |

Negative, zero, non-finite, impossible-cavity, inadequate-height, and lost-overlap inputs are all asserted to block. Contextual-only Finished View, green-label isolation, and work-area/exact-scale behavior are asserted.

## 10. Fresh test totals

- `git diff --check`: clean. HTML parser: OK. Inline JS: parses and runs under `file://` with no exception.
- **Focused T3A (`runTabletopStorageFitCouponFixtures`): 23 / 0.**
- Every exposed `run*Fixtures()` invoked once (44 functions): **3192 passed / 0 failed.** Individually confirmed: Raw Shell Results 52/0, T1 Coupon 76/0, T2C Rectangular Shell 93/0, T2C Closed Corners 13/0, Shell Evidence Promotion 16/0, Shell Evidence Matching 16/0, Designs→Projects handoff 17/0, Tray model 264/0, Dice Tray system 92/0, Designs geometry 1093/0, Gift Box 69/0.
- **`?selftest=all` registered groups (42): 2912 / 0**, reconciled exactly: the 42 dispatched groups = the 41 in my batch (2855 clean) + `runMaterialBrowserFixtures` (57) = **2912**. (`runTrayModelFixtures` and `runPromotionTargetSwitchFixtures` are exposed but invoked via other routes, which is why 44 exposed ≠ 42 registered.) The report's "2912 / 0 across 42 groups" is **accurate.**
- **Methodology caveat (not a product defect):** when many groups are invoked back-to-back through external CDP `Runtime.evaluate`, the keyboard-tab-navigation assertions in `runAccessibilityPolishFixtures`/`runResponsiveTuningFixtures` intermittently fail (the documented headless-focus timing artifact). I confirmed the *only* ever-failing assertions are arrow/Home/End/Enter/Slash tab-focus checks, and the same groups return 36/0 and 45/0 in clean order. T3A touches no accessibility/responsive/tab code.

## 11. Protected production hashes (actual current values)

| Golden | Expected | Present as asserted literal + fixture passing |
| --- | --- | --- |
| Legacy pocketed shell | 2181 / `2ef9606b` | ✓ |
| T1 coupon | 2992 / `4f543f95` | ✓ |
| T2 shell | 2337 / `ed5d6f6e` | ✓ |
| Dice Tray | 1726 / `51a55721` | ✓ |
| Alternate Dice Tray | 1054 / `41697123` | ✓ |
| Divider Tray | 1965 / `a55dda6e` | ✓ |
| Wall-to-base joint coupon | 1551 / `d9ffc278` | ✓ (this is the current joint-coupon golden; see §3.3) |

Additional goldens checked and passing (literal present, fixture green): Dice bottom-cover `8e2ea3f4`; Concealed Cleat corner `88721533` / cut `c16a9ef5`, full box `2316430b`; decorated Finger Box `d9512d1c`; Sliding-lid `4a7ab718`, inside `a892f91c`, lid `6181bc75`; Drawer Cabinet `a6dd23dc` / `36a41b07` / `8c286797`, guides `dd3ff0bd`, grid `0814ca2e`, separate outputs `956ad870` / `bad8b97f`, custom `500396df` / `69b0ce8b`, legacy `7494c326` / `b158a794`. **None changed by T3A.** T3A's own 897 / `b5c549ee` is deterministic and intentionally not pinned (§3.2).

## 12. Protected boundaries

Confirmed unchanged (not present in the diff): `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, `localStorage`/backup/import-export behavior, Project & Library schemas, machine identity, existing evidence records, T1/T2 identity separation, `buildBoxModel` (called, not modified), `buildFingerPattern`, `tabletopJointFitCompatible`, kerf conventions, LightBurn colors, existing generators/filenames, direct offline `file://` startup, and the T2C startup-initialization hotfix.

## 13. Browser scenarios actually tested

Automated headless CDP against a disposable `file://` load only (no manual clicking): clean startup with no runtime exception; navigation exercised programmatically via `buildDesignResult`/`renderDesigns`/`projectDraftFromDesign` with the exact physical-test inputs; Finished View SVG generation; production SVG generation + determinism; impossible-input validation (zero clearance, sub-minimum rail, no-visible-wall, excessive-clearance); Project handoff; and storage/backup byte-identity across generation, preview, and handoff. Startup/data-safety: T3A does not touch `loadState`, observation normalization, or initialization order, so no temporal-dead-zone regression is reachable from this change; the storage-isolation assertions and a clean `file://` startup both passed. I did **not** perform manual seeded-reload clicking; the seeded-record safety is covered by the unchanged T2C hotfix plus the storage-isolation fixtures.

## 14. Unverified physical areas

Everything about real-world fit, by design: whether 0.20 mm/side actually clears Joe's built shell; whether the 2.68 mm ledge reliably supports a rolling load without the panel tipping into the cavity; rail-to-wall glue/clamp behaviour and cure; false-bottom insertion/removal feel; racking; and usable capacity. These are exactly what the dry-fit test exists to establish and are correctly presented as unproven.

## 15. Exact cut instructions (dry-fit only)

Five parts, all from the same **2.88 mm measured Basswood plywood** used for the proven shell:

- **RAIL-FRONT** — 60.942222 × 10 mm
- **RAIL-BACK** — 60.942222 × 10 mm
- **RAIL-LEFT** — 41.211429 × 10 mm
- **RAIL-RIGHT** — 41.211429 × 10 mm
- **FALSE-BOTTOM** — 79.6 × 59.6 mm

Procedure:

1. Generate with the T3A defaults (interior 80/60/25, measured 2.88, clearance −0.075, finger 9, rail 10, false-bottom 2.88, total clearance 0.40). Confirm the summary shows rail lengths 60.94 / 41.21 mm, false bottom 79.6 × 59.6, overlap 2.68 mm, visible wall 12.12 mm.
2. Download the SVG and **import into LightBurn at 100% scale** (do not resize). Output only the **red `#ff0000` cut layer**. Leave assembly labels off (or, if on, assign the green `#00ff00` layer to a no-output/tool layer — do not cut or engrave labels). There are no score/engrave/pocket operations to enable.
3. Cut all five pieces from 2.88 mm stock. Deburr lightly.
4. **Dry-fit, no glue:** stand the four rails on the existing shell floor, each flat against its wall, inset from both corners by the generated finger width. Rest the plain **FALSE-BOTTOM** on top of the four rails. Add a small **temporary masking-tape pull tab** to the false bottom for removal.
5. Record: do the rails sit flat on the floor and flush to the walls; are the corner setbacks equal front/back and left/right; does the false bottom sit level on all four rails without rocking; is there visible even clearance (~0.2 mm/side) to the walls; does it lift out cleanly by the tape tab; and is the remaining wall height (~12 mm) as expected.
6. **Pass:** false bottom rests level on all four rails, lifts out without binding, rails seat flat with even setbacks. **Marginal:** panel is snug but removable with light care, or one rail rocks slightly — adjust clearance/rail and recut before proceeding. **Fail:** panel binds, cannot be removed without force, sits on fewer than four rails, or drops toward the floor — stop and revise dimensions.
7. **Do not apply any glue until the complete dry fit passes.** A pass here authorizes only gluing the rails in a later step, not that the storage mechanism is proven.

## 16. Cut-readiness verdict

**1. SAFE TO CUT** — the geometry and production output are internally verified (five correct pieces, insets derived from the real T2 floor-finger geometry, clean cavity clearance, meaningful rail overlap, deterministic 897-byte SVG, all protected goldens intact, 23/0 focused and 2912/0 registered fixtures). Physical fit remains intentionally unproven. Joe may cut the four rails and false bottom for a **dry-fit test only**; this does not authorize gluing until the dry fit succeeds.

**Corrections required before commit:** none. The two hardening ideas in §3 (assert insets against the T2A shell width; optionally pin the 897/`b5c549ee` output) are non-blocking suggestions, not defects.
