# Tabletop Accessories T1.5 — Physical Coupon Results and Evidence Reuse — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-20
**Reviewer:** Claude Sonnet 5
**Type:** read-only focused implementation audit. No product file, report, README, CHANGELOG, fixture, storage, or application record was edited. Nothing was staged, committed, or pushed. Only this report was written.

## 1. Repository state and actual HEAD

- `git log -1 --oneline` / `git rev-parse HEAD`: `0b7f89f275cc19e4be763c10e1e9753d5666ff28` — "Add Tabletop Accessories engine and joint coupon." Matches the stated committed baseline.
- Branch `main`; `git rev-list --left-right --count origin/main...main` = `0 0` (synchronized).
- `git diff --check`: clean (CRLF-will-be-replaced warnings only, no conflict markers or whitespace errors).
- `git diff --stat`: `CHANGELOG.md | 2 +`, `README.md | 10 +-`, `index.html | 284 ++...+---` (278 insertions, 18 deletions). Nothing staged (`git diff --cached --stat` empty).
- Tracked changes are limited to exactly the three expected files. T1.5 remains fully uncommitted.
- `git ls-files --others --exclude-standard`: identical to the pre-existing untracked set (LightBurn Projects, `debug.log`, historical `docs/*.md`, `parametric_qr_stand_generator.py`) plus the two new T1.5 doc files (architecture review, implementation report) that were already present before this audit began. Nothing else was created or removed by this audit except this report.
- Both the architecture review and implementation report are present on disk and were read in full.
- Constants confirmed unchanged: `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.
- Re-ran `git status -sb` and `git diff --stat` after all browser-driven testing below; output was byte-identical to the pre-audit snapshot, confirming the audit made no product-file changes.

## 2. Files and functions reviewed

Full `git diff index.html` (all 284 inserted/18 deleted lines), plus direct source reads of `buildTabletopCornerFloorCouponDesignResult`, `buildBoxModel`, `buildFingerPattern` (to establish ground truth for `patterns.width`/`patterns.vertical`/`actualWidth`/`count`), `normalizeTabletopCouponResult` and its helpers, `tabletopCouponResultEligibility`, `tabletopCouponConstructionIdentityFromResult`, `buildTabletopCouponResultRecord`, `openTabletopCouponResultRecorder`, `openTabletopCouponResultsList`, `openTabletopCouponPromotion`, `buildTabletopCouponPromotionCandidate`, `finalizeTabletopCouponPromotion`, `tabletopCouponConstructionMatches`, `tabletopCouponMaterialMatches`, `tabletopCouponThicknessTier`, `tabletopCouponRankMatches`, the modified `normalizeProductionEvidence`, `promotionStatusRules`, `promotionEvidenceSnapshot`, `renderPromotionReview`, `startPromotion`, `mergeData`/`replaceData` additions, `freshState`/`loadState`/`persist`, and the three new fixture functions. Cross-checked against `docs/DESIGNS_TABLETOP_ACCESSORIES_T1_ENGINE_COUPON_IMPLEMENTATION_2026-07-20.md` for the T1 baseline contract.

## 3. Locked-contract comparison

| Locked decision (architecture report) | Classification | Evidence |
| --- | --- | --- |
| New additive `tabletopCouponResults` array, no new key/schema bump | **Implemented exactly** | `freshState`, `loadState`, `persist` diffs; no `SCHEMA_VERSION`/`STORAGE_KEY`/`BACKUP_FORMAT` change |
| Raw record = source of truth for observation; promoted setting/evidence = source of truth for reusable evidence | **Implemented exactly** | `normalizeTabletopCouponResult` vs. `buildTabletopCouponPromotionCandidate`/`finalizeTabletopCouponPromotion` |
| Promotion never automatic, explicit action | **Implemented exactly** | Confirmed live: raw save with no profile created zero Production Settings; promotion requires a separate modal + explicit Library profile pick |
| Coupon-proven / Shell-proven / Production-proven terminology, current result locked to Coupon-proven | **Implemented exactly** | `fitContext.evidenceStage:'coupon-proven'`; `verified` blocked for `tabletop-coupon` sourceType in both the status-choice `<select>` and `promotionStatusRules` |
| Machine identity reuses M3 exact-match rule | **Implemented exactly** | `productionMachineIdentityMatches` reused unmodified in `tabletopCouponRankMatches`; fixture confirms one-sided ID rejection |
| Material identity: Library-linked or free text, never invented | **Implemented exactly** | `tabletopCouponNormalizeMaterialSnapshot`, `tabletopCouponMaterialSnapshot`; live test saved a record with no material selected as "Unlabeled material" rather than inventing one |
| Measured-thickness matching within clearance-step tolerance, wider band as "close" | **Implemented exactly** | `tabletopCouponThicknessTier` matches the locked strongest/close/none tiers and epsilon-bounded comparisons |
| Construction identity separates horizontal/floor and vertical/corner finger count and actual width | **Implemented exactly** | See §6; sourced from real `patterns.width`/`patterns.vertical` fields, not collapsed |
| Candidate model: signed clearance + small controlled vocabulary + freeform notes | **Implemented exactly** | `tabletopCouponFitClassifications`/`tabletopCouponSelfSupportValues` five/four-value enums plus `assemblyEffort`/`notes` free text |
| Exactly one required preferred, optional tighter alternate, no third slot | **Implemented exactly in code; inaccurately described in the implementation report** | See §8. Report text says alternate is "also required"; source and live UI test prove otherwise |
| No silent apply, no automatic recommendation selection | **Implemented exactly** | No `data-tabletop-apply-recommendation` control exists anywhere; fixture and manual grep confirm |
| fitContext frozen snapshot, optional on legacy evidence | **Implemented exactly** | `normalizeProductionEvidence` spreads `fitContext` conditionally; legacy evidence without it is unaffected |
| Editing raw does not rewrite promoted evidence; deleting raw does not delete evidence | **Implemented exactly** | Confirmed by fixture and by source (`finalizeTabletopCouponPromotion` only writes a back-reference; delete only filters `state.tabletopCouponResults`) |
| No Project schema change in T1.5 | **Implemented exactly** | No Project-related diff lines present |
| Recommendation card UI deferred to T2 | **Deferred as intended** | Only the pure `tabletopCouponRankMatches` helper exists; no rendering consumer |
| Fixture organization: three small standalone groups | **Implemented exactly** | `runTabletopCouponResultsFixtures`, `runTabletopEvidencePromotionFixtures`, `runTabletopEvidenceMatchingFixtures`, each registered once under its own `?selftest=` route |

## 4. Root storage and normalization

`tabletopCouponResults` is the only new root array; it is included unconditionally in `freshState()`, `loadState()`, `persist()`, `backupObject()` (session-recovery serialization), `mergeData()`, and `replaceData()`. No second storage key, no `SCHEMA_VERSION` bump, no `BACKUP_FORMAT` change. `normalizeTabletopCouponResults(undefined)` returns `[]` (fixture-confirmed and independently re-tested). `normalizeTabletopCouponResult` rejects non-object input, requires a positive `measuredThicknessMm` and at least one valid candidate, and requires a valid `sourceMode` (`'manual'`/`'generated'`) — anything else normalizes to `null` and is filtered out by `normalizeTabletopCouponResults`. Duplicate IDs collapse to one entry (`Set`-based dedupe, fixture-confirmed). Merge uses the existing generic `mergeList`/`mergeListStats` (local wins on ID conflict, matching every other collection). Replace fully restores the collection. `freshState()` seeds `tabletopCouponResults: []` — no synthetic/sample records. Live testing (see §9) left no residual record in the repository's own state, since browser-side localStorage lives only in the disposable Edge profile directory, never in the tracked repository.

## 5. Raw record shape

Verified fields present exactly as required: `id`, `createdAt`, `updatedAt`, `testedAt`, `sourceMode`, `templateId` (hardcoded to `'tabletop-corner-floor-coupon'`), `generatorVersion`, `machine` snapshot, `material` snapshot (with `materialId` only when a profile is genuinely linked), `measuredThicknessMm`, `nominalThicknessMm`, `clearanceStepMm`, `testGeometry` (`preferredFingerWidthMm`/`requestedInteriorLegMm`/`wallHeightMm`), `constructionIdentity`, `candidates`, `preferredCandidateId`, `alternateCandidateId`, `notes`, `promotedEvidence`. Measured thickness is required and positive (`tabletopCouponFinitePositive`); nominal thickness is a separate, independently normalized field and is never substituted for measured thickness anywhere in ranking or promotion (`tabletopCouponThicknessTier` and `buildTabletopCouponPromotionCandidate` both consume `measuredThicknessMm` only). Generated records derive `generatorVersion`/`constructionIdentity`/`measuredThicknessMm`/`clearanceStepMm`/candidate clearances directly from the live `buildDesignResult` output. Manual records default `constructionIdentity` to `{}` (incomplete) rather than inventing plausible-looking values; incomplete manual records remain readable (they still normalize successfully) but `tabletopCouponResultEligibility` and `tabletopCouponRankMatches` correctly exclude them from promotion/ranking (fixture: `'manual incomplete identity cannot rank'`, live-confirmed logically consistent with the identity-completeness gate in `tabletopCouponConstructionIdentityComplete`). Unknown extra fields on the input object are preserved via `{...value, ...}` spread in `normalizeTabletopCouponResult` but do not affect any typed field read elsewhere — they do not corrupt storage since `JSON.stringify` round-trips arbitrary extra keys harmlessly.

## 6. Construction identity (high priority)

**Both a horizontal/floor and a vertical/corner finger count and actual width are stored separately, and both are sourced correctly.** Ground truth traced directly to `buildTabletopCornerFloorCouponDesignResult` (index.html:4691-4717): each candidate model's `patterns` field is `box.patterns = {width, depth, vertical}` from `buildBoxModel`, where `width`/`depth` (numerically identical for this square coupon) govern the wall-to-floor connection and `vertical` governs the 90° wall-to-wall corner connection. Each of `width`/`vertical` is a `buildFingerPattern()` result with real `count` and `actualWidth` fields (index.html:3450-3460). `tabletopCouponConstructionIdentityFromResult` (index.html:653) reads `result.finishedView.candidates[0].patterns.width` and `.vertical` — both are genuine, populated fields on the real result object (`result.finishedView = {template, candidates:candidateModels, orientation}`, confirmed at index.html:4714), not placeholders. The stored identity is:

```
{ family:'tabletop-accessories', templateId:'tabletop-corner-floor-coupon', generatorVersion,
  wallCornerJoint:'finger-90', floorJoint:'finger-jointed-floor',
  pattern: { horizontalFloorFingerCount, horizontalFloorActualWidthMm, verticalCornerFingerCount, verticalCornerActualWidthMm } }
```

All four pattern fields are present and independently populated (fixture: `'construction identity comes from generated geometry'`, `'horizontal and vertical pattern fields are separate'`). Measured material thickness is **not** embedded as a construction-identity key (it is compared separately via `tabletopCouponThicknessTier`, exactly as the architecture locked). The requested preferred finger width is never substituted for the actual generated width — `horizontalFloorActualWidthMm`/`verticalCornerActualWidthMm` come from `actualWidth`, never from `v.preferredFingerWidth`. Because C1/C2/C3 differ only in `clearance` (which is not a parameter to `buildFingerPattern`), all three candidates genuinely share one identical finger pattern, so using `candidates[0]` as the representative source does not silently flatten a real difference — there is no real difference to flatten. `tabletopCouponConstructionIdentityComplete` requires positive integer finger counts and finite actual widths on both axes plus non-empty `generatorVersion`/exact `family`/`templateId`/`wallCornerJoint`/`floorJoint`; incomplete identity blocks both promotion (`tabletopCouponResultEligibility`) and ranking (`tabletopCouponConstructionMatches`). `tabletopCouponConstructionMatches` compares `generatorVersion` for exact string equality, so a future engine revision cannot silently match. Numeric comparisons use a fixed `epsilon = 1e-6`, not loose approximate matching.

**One dead/unreachable fallback found (POLISH, not a functional defect):** `tabletopCouponConstructionIdentityFromResult`'s fallback path (`metrics.actualFingerWidths?.[0]?.horizontalCount` / `.verticalCount`) references field names (`horizontalCount`, `verticalCount`) that do not exist anywhere on `metrics.actualFingerWidths` (that array's real per-item shape is `{id, horizontal, vertical, floor}` with no `*Count` fields — confirmed at index.html:4714 and the pre-existing fixture at index.html:5572-5573). This fallback can never actually execute in the current call graph, because `buildTabletopCouponResultRecord` only calls the identity builder after confirming `result.valid`, which guarantees `finishedView.candidates[0].patterns.width/vertical` are populated. It is dead code today, not a live defect, but it is misleading and should be cleaned up or removed so a future refactor does not rely on it.

## 7. Candidate model

Each candidate normalizes to `{id, label, clearanceMm, fitClassification, selfSupport, assemblyEffort, notes}`. `fitClassification` is restricted to the fixed five-value enum `good-friction / glue-fit / too-tight / loose / failed` (plus `not-tested` as the unset default) with matching human-readable labels (`tabletopCouponFitLabels`) that do not rely on color. `selfSupport` is restricted to `not-tested / none / two-walls / single-wall` with matching labels. Signed clearances were verified end-to-end: the fixture asserts generated values round-trip as `[0.025, 0.05, 0.075]` (positive test data) and manual entry as `[-0.075, -0.05, -0.025]` (the real physical sign convention); live UI testing additionally confirmed a record entered with measured thickness `2.88` and default candidate clearances persisted, survived modal close/reopen (via View Saved Results), and displayed correctly with no truncation or sign flip. Export/import/promotion round-tripping of signed values is covered by the fixture group and passed (see §12/§14).

## 8. Preferred/alternate behavior (high priority)

**Finding: the optional tighter alternate is correctly implemented as optional in code. The implementation report's statement that "a preferred candidate and tighter alternate are also required" is inaccurate documentation, not an implementation defect.**

Source evidence: `tabletopCouponResultEligibility` only unconditionally requires a preferred candidate (`if (!preferred) errors.push('Choose exactly one preferred candidate.')`); the alternate check is gated behind `if (result.alternateCandidateId && (...))` — it only runs, and only produces an error, when an alternate was actually supplied. The save-form submit handler in `openTabletopCouponResultRecorder` mirrors this exactly: it blocks only on a missing preferred candidate, and only inspects alternate-related errors `if (record.alternateCandidateId && ...)`.

Live-tested directly against the running app (headless Edge, `file://` load, real DOM form fill and `requestSubmit()`, not a closured internal call): a record was created with `preferredCandidateId:'C2'` and `alternateCandidateId:''` (left blank). The save succeeded with zero `alert()` calls, the modal closed, and "View Saved Results" showed exactly one record labeled "Eligible for ranking and explicit promotion." A second live test with `alternateCandidateId:'missing'` and a third with a looser alternate would be rejected by the same `errors.push(...)` branch (fixture-confirmed: `'alternate must exist'`, `'alternate must be tighter'`); rejection when preferred is missing was also fixture-confirmed (`'preferred candidate is required'`).

README correctly documents the alternate as optional ("preferred, and optional tighter-alternate provenance"); only the implementation report's prose is wrong. **Classified IMPORTANT** (misleading documentation, not a code defect) because a reader of the implementation report alone would incorrectly believe every saved result needs two candidates observed, which could cause Joe to over-test in the workshop or distrust the feature. **A second, related IMPORTANT finding:** the fixture group has no explicit "save succeeds with preferred set and alternate blank" assertion — every alternate-adjacent fixture case supplies `alternateCandidateId:'C1'`. This is exactly the scenario the report's wording confusion could have hidden a real regression in; it should get its own regression assertion rather than relying on this audit's one-off live test.

## 9. Generated recording workflow

Driven live against the actual running app (headless Edge 150, `file:///C:/Genmitsu L8 Tracker/index.html`, real DOM events — clicking `#tab-designs`, setting `<select name="template">`, dispatching `change`, clicking `[data-record-tabletop-results]`, filling the real form, calling `form.requestSubmit()`):

1. Opening Designs and selecting `tabletop-corner-floor-coupon` with default values produced a valid result; **Record Physical Results** appeared immediately (default params are valid).
2. Setting `materialThickness` to `0` made the result invalid and **Record Physical Results` disappeared** — confirmed the button is conditioned on `result.valid` (index.html: `tabletop && result.valid` in the `promotionEntry` ternary).
3. Restoring thickness to `3` brought the button back.
4. Opening the recorder pre-filled machine label `Genmitsu L8 20W` and measured thickness `3` (from the live draft), with C1/C2/C3 radio rows present.
5. Overriding measured thickness to `2.88` (the real physical value), setting fit/self-support per candidate, selecting preferred `C2`, leaving alternate blank, and submitting: saved with **zero alerts**, modal closed.
6. **View Saved Results** showed `1 raw result`, `Genmitsu L8 20W · 2.88 mm measured · 3 candidates`, `Eligible for ranking and explicit promotion` — confirming machine snapshot, measured thickness, and candidate count are all accurate and that save did not require a material profile.
7. Clicking **Promote to Library evidence** with zero Library profiles present opened the promotion-material picker showing only `Free-text material label` (no profile options) — attempting to submit with an empty/invalid selection produced the alert `Choose a valid Library material profile and a complete result.` and the modal stayed open, confirming promotion is correctly gated on a real Library profile while raw save is not.
8. Deleting the unpromoted record produced the confirmation prompt `Delete this raw physical result?`; confirming removed it and the list returned to its documented empty state (`No saved physical results` / `Record a generated or previous coupon result after cutting.`).
9. Throughout, no JavaScript exceptions were thrown by the app itself (the only `ReferenceError`s encountered were from this auditor's own attempts to call IIFE-closured internals directly from outside, which is expected and unrelated to app correctness).
10. `git status`/`git diff --stat` after the full session were byte-identical to before — confirming no production record, SVG, draft, or default was altered by this exercise, and nothing in the repository was touched (browser state lives only in a disposable scratch profile).

## 10. Manual recording workflow

`openTabletopCouponResultRecorder(null, null)` is wired to `[data-record-tabletop-previous]` and opens the same recorder dialog with a fully manual, blank-defaulted source (`constructionIdentity:{}`, empty geometry) — no live Design draft is required, satisfying the "record a previous result later" requirement. The dialog's own footer text explicitly discloses incompleteness: `"This manual record has incomplete construction identity. It remains readable but cannot rank or promote until identity is complete."` No species, vendor, coating, machine ID, or material ID is invented for a manual entry — `tabletopCouponGeneratedMachineSnapshot()` is used as a convenience default for the machine field (same as generated records), but material defaults to an empty free-text label, and construction identity defaults to an explicitly incomplete `{}` rather than any invented plausible values. A manual record with a complete, user-supplied `constructionIdentity` (matching the exact required shape) becomes eligible through nothing more than the user directly re-entering the matching values into the same form fields — there is no separate hidden completion path.

## 11. Machine identity

Reuses `productionMachineIdentityMatches` completely unmodified for ranking (`tabletopCouponRankMatches` calls it directly). Fixture-confirmed: exact `machineId` match ranks; a one-sided `machineId` (target has an ID, candidate machine does not) is rejected, not silently treated as exact; legacy key-fallback remains available and unmodified. `tabletopCouponGeneratedMachineSnapshot()` defers to the existing `activeOwnedMachineSnapshot()` M3 helper with a legacy fallback, introducing no parallel machine-identity implementation. Archived-machine handling is not separately re-implemented; it flows through the existing `setting.machine?.archived` check already present in `tabletopCouponRankMatches`'s `machineCaveat` computation, annotating rather than excluding an archived-but-matching machine. No historical backfill occurs anywhere in the new code.

## 12. Material identity

Both paths are implemented: `materialId`-linked (via the Library profile `<select>` populated by `tabletopCouponMaterialOptions`) and free-text (`materialLabel` input). No species/vendor/coating/product identity is invented anywhere — confirmed by direct code read and by the live test, where an unlinked record correctly displayed as "Unlabeled material" rather than fabricating a description. `tabletopCouponNormalizeMaterialSnapshot` freezes `label`/`type` regardless of whether the source profile still exists later, so a dangling `materialId` remains readable. Raw save does not require a Library profile (live-confirmed). Promotion does require one (live-confirmed: empty selection blocked with an explicit alert). A free-text raw record can be linked to a Library profile at promotion time via the same picker (fixture: `'free-text raw can be linked during explicit promotion'`). Promotion never silently creates a new Library material profile — `buildTabletopCouponPromotionCandidate` returns `null` if `profile` is falsy, and the picker only offers profiles that already exist in `state.profiles`.

## 13. Raw-save versus promotion

Directly confirmed (fixture `'raw save does not auto-promote'` plus live test): saving a raw result with `state.profiles = []` produced zero Production Settings, zero Evidence, and left `record.promotedEvidence === null`. No default, Design clearance, or global state beyond the one new array entry is altered by raw save. Promotion is a fully separate, explicit action reached only via a distinct "Promote to Library evidence" button and a distinct review modal. `buildTabletopCouponPromotionCandidate` fabricates no `speedMmPerMin`/`powerMinPercent`/`passes`/other unrelated cut-setting fields (fixture: `'no fabricated cut settings'`); it populates only `fingerJointClearanceMm` and `measuredThicknessMm`. `sourceType` is `'tabletop-coupon'` (fixture-confirmed and confirmed in `productionEvidenceKinds`). `verificationStatus` defaults to `'tested'` (fixture-confirmed) and `'verified'` is blocked for `tabletop-coupon` both at the UI level (`statusChoices` restricts the `<select>` to `['tested']` only) and at the validation level (`promotionStatusRules` explicitly rejects `verified` for `couponOnly` candidates) — a genuine belt-and-suspenders implementation of the locked contract. The raw back-reference (`raw.promotedEvidence`) is written only inside `finalizeTabletopCouponPromotion`, which is called only after `savePromotionTransaction` reports `ok:true`; a failed transaction leaves the raw result unpromoted (fixture: `'failed promotion does not claim success'`). Re-promoting an already-promoted record is not hard-blocked, but this is **not a defect**: it is handled by the same pre-existing, established `promotionSourceDuplicateWarnings` mechanism used by every other evidence source type in this app (matches on `evidence.sourceId === candidate.sourceId && evidence.kind === candidate.sourceType`, surfaces a non-blocking warning banner). Tabletop coupon inherits this automatically because it reuses the shared promotion pipeline unmodified — consistent, not a regression.

## 14. Production Setting mapping

`buildTabletopCouponPromotionCandidate` maps the preferred candidate's `clearanceMm` to `fitSettings.fingerJointClearanceMm` and the raw record's `measuredThicknessMm` to `materialCondition.measuredThicknessMm`, using the existing `promotionAddField`/`promotionCandidateBase` machinery unchanged. `operation:'cut'` is used (structurally correct, since this is genuinely cut geometry) rather than inventing a new pseudo-operation. No other cut-setting fields are populated. This directly reuses the pre-existing `fitSettings.fingerJointClearanceMm` field rather than distorting Material Test's speed/power-shaped semantics.

## 15. Evidence fitContext

Confirmed present with every required field: `evidenceStage:'coupon-proven'`, complete `constructionIdentity` (cloned via `promotionClone`), `measuredThicknessMm`, `nominalThicknessMm`, `clearanceStepMm`, `materialSnapshot`, `preferredCandidate:{id,clearanceMm,fitClassification,selfSupport}`, `tighterAlternate` (object or `null`), `sourceResultId`, `testedAt`. Fixture-confirmed: `'fitContext is present and Coupon-proven'`, `'fitContext retains construction and thickness'`, `'fitContext retains preferred and tighter alternate'`. `fitContext` is optional and additive on `normalizeProductionEvidence` — legacy evidence entries without it are spread through unchanged (`...(item.fitContext && typeof item.fitContext === 'object' ? {fitContext:...} : {})`), so no existing Evidence record is altered in shape. Construction identity is not duplicated inconsistently elsewhere — it lives once, inside `fitContext`, and is also mirrored read-only inside `candidate.sourceSnapshot.constructionIdentity` for the promotion-review display, both derived from the same `raw.constructionIdentity` object via `promotionClone` (a copy, not a second independent source of truth). Editing raw notes after promotion does not mutate the frozen evidence snapshot (fixture: `'editing raw leaves evidence snapshot frozen'`, verified by comparing `JSON.stringify(evidence.fitContext)` before and after a raw-field edit). Deleting the raw record does not delete the Production Setting or its Evidence (fixture: `'deleting raw does not delete promoted Library evidence'`).

## 16. Evidence-stage enforcement

Only `'coupon-proven'` is ever written to `fitContext.evidenceStage` by this implementation; there is no code path anywhere in the diff that writes `'shell-proven'` or `'production-proven'`. `verified` is blocked for `tabletop-coupon` sourceType both in the status dropdown and in `promotionStatusRules`. The promotion review modal displays an explicit caveat for tabletop-coupon candidates: `"Coupon-proven only: this promotion does not prove a complete shell or production readiness."` Help text (see §25) reinforces this. No hidden path promotes coupon evidence to verified — the only two places `verificationStatus` can be set for this source type are the restricted `<select>` and `promotionStatusRules`, both of which agree.

## 17. Matching and ranking helper

`tabletopCouponRankMatches` is a pure function: it reads `settings`/`evidence` arguments and module-level normalizer functions, and returns a new array; it does not write to `state`, `designDraft`, or any global default anywhere in its body (confirmed by direct read — no assignment statements exist in the function). Fixture-confirmed: `'no global default mutation'` (compares `designDraft` to `designDefaults()`), `'no design draft mutation'` (draft template unchanged after running matching), `'no automatic apply action exists'` (no `[data-tabletop-apply-recommendation]` element exists in the DOM — also independently reconfirmed: no such attribute string appears anywhere in the 284-line diff). All tied results are returned rather than collapsed or averaged (the function returns an array, sorted, never reduced to a single value); ties break on `evidence.date`/`testedAt` descending (newest first) — confirmed by direct read of the `.sort()` comparator.

## 18. Thickness ranking

`tabletopCouponThicknessTier(target, tested, step)` implements exactly the three-tier contract: difference ≤ `step` → `strongest`; difference ≤ `3×step` → `close`; beyond → `none`. Fixture-confirmed at the exact boundary values from the real physical evidence (`step=0.05`): `2.83` vs `2.88` (difference `0.05`, at the boundary) → `strongest`; `2.82` vs `2.88` (difference `0.06`, just over one step) → `close`; `2.70` vs `2.88` (difference `0.18`, beyond three steps) → `none`. When `step` is missing/non-finite, the function falls back to two-decimal rounded equality rather than an arbitrary tolerance (fixture-confirmed). Nominal-only thickness is never used for ranking — `tabletopCouponThicknessTier` is always called with `measuredThicknessMm`, never `nominalThicknessMm`; nominal thickness is not part of `constructionIdentity` (confirmed in §6).

## 19. Tie/conflict behavior

`tabletopCouponRankMatches` never averages or auto-selects among equally-ranked or conflicting evidence; it returns every match that passes the machine/material/thickness/construction gates, ranked and sorted, with no reduction step. Because the function returns an array rather than a single "best" object, a caller (deferred to T2) would need to explicitly choose which entry to present or apply — consistent with the locked "never hide conflicting evidence, never auto-select" contract. No averaging code exists anywhere in the function (confirmed by direct read: the only arithmetic is `a.rank-b.rank` and a string `localeCompare`, both comparator logic, not aggregation).

## 20. Saved-result UI

Count (`state.tabletopCouponResults.length`) is read directly from the live array, so it is always accurate; live-tested transitions from `0` → `1` → `0` all displayed correctly. List order is deterministic (array order, newest unshifted to the front by `tabletopCouponRecordSave`, matching every other "most recent first" list in this app). View exposes machine, measured thickness, candidate count, and an eligibility/promotion-status line. Edit reuses `openTabletopCouponResultRecorder(null, null, record)`, which preserves the existing `id` (passed through as `existing.id`) — `createdAt` is preserved because `normalizeTabletopCouponResult` only sets `createdAt` when the incoming value doesn't already have one (`typeof value.createdAt === 'string' && value.createdAt ? value.createdAt : new Date().toISOString()`), and `updatedAt` is explicitly reset to "now" on every `tabletopCouponRecordSave` call. A promoted-record edit warning is not a separate modal-level banner but is documented behavior (§8/§15/Help) and does not silently rewrite the evidence snapshot (already-verified). Delete shows a distinct message for promoted vs. unpromoted records (`"This raw result was promoted. Delete the raw result only? Library evidence and the Production Setting remain independently."` vs. `"Delete this raw physical result?"` — live-confirmed for the unpromoted case). Deleting a promoted raw record leaves the setting/evidence intact (fixture-confirmed). Dangling material/machine references degrade gracefully via frozen snapshot fields, consistent with the app's established empty-state/dangling-reference pattern from the earlier beginner-clarity phase. The empty state (`"No saved physical results" / "Record a generated or previous coupon result after cutting."`) is clear and was directly observed live.

## 21. Edit/delete behavior

See §20 and §8/§15 — edit preserves stable identity and does not retroactively alter promoted evidence; delete correctly distinguishes promoted vs. unpromoted with an explicit, honest confirmation message; deleting a promoted record was fixture-confirmed (not separately re-driven live, to avoid needing a Library profile setup mid-session, but the fixture directly exercises the same code path this UI action calls).

## 22. Import/export/merge/replace

`backupObject()` includes `tabletopCouponResults` (fixture: `'export includes raw results'`). An old backup missing the field is accepted and normalizes to `[]` (fixture: `'old backup without collection is accepted'`, using the real `validateBackupImport`). Merge unions by ID via the standard `mergeList` (fixture: `'merge uses id-based raw-result behavior'` — an incoming record with a new ID was added alongside the existing one). Replace fully restores the collection (fixture-confirmed). Duplicate IDs normalize to one entry (fixture-confirmed). Signed clearances, candidate observations, machine/material snapshots, and `promotedEvidence` all survive the round trip because they are plain, non-function-valued JSON fields passed through `JSON.stringify`/`JSON.parse` identically to every other collection — no special-cased serialization exists that could drop or coerce them.

## 23. Recovery

The collection participates in the same `persist()`/`loadState()` path as every other root array; there is no separate recovery code path specific to Tabletop results, so it inherits the app's existing storage-recovery behavior unmodified (confirmed by the diff touching only the destructuring/serialization lists in `persist`/`loadState`, not the recovery logic itself, and by `runStorageRecoveryFixtures` continuing to pass at 15/0 with no changes needed in that group).

## 24. Import preview text and counts

Not separately modified by this diff — the collection flows through the same generic Merge/Replace preview and count machinery already used by every other array collection (`mergeListStats`), so users see it accounted for exactly like Test Grids, Projects, or Inventory entries, without a bespoke, separately-maintained preview path that could drift from the real merge behavior.

## 25. Help and accessibility

Help text was verified present via the `runHelpSafetyFixtures` regex assertions, all of which passed (43/0), covering: recording after cutting, C1/C2/C3 observations, preferred vs. optional tighter alternate, measured thickness requirement, Coupon-proven meaning, explicit promotion requirement, no-automatic-defaults, frozen evidence, edits not rewriting evidence, deletion not deleting evidence, machine/material/thickness/construction specificity, the Shell-proven gate, and laser safety. The actual inserted Help paragraph (index.html, `tabletopHelpSection`) reads in full and matches these fixture assertions verbatim. Accessibility: the recorder/list/promotion-material modals reuse the existing `openModal`/`bindModal` centralized accessible-modal machinery (established in an earlier phase of this project) rather than a bespoke implementation, so focus containment, Escape handling, and focus restoration are inherited, not reinvented. Candidate fit/self-support are `<select>` dropdowns with visible text labels (not color-only). Cancel buttons (`data-tabletop-result-cancel`, `data-close`) are present on every new modal.

## 26. Fixture inventory and gaps

`runTabletopCouponResultsFixtures` (28 assertions), `runTabletopEvidencePromotionFixtures` (17), `runTabletopEvidenceMatchingFixtures` (21) — all independently re-run and confirmed 0 failed. Coverage is genuinely direct rather than happy-path-only: it includes preferred-required, alternate-must-exist, alternate-must-be-tighter, nonfinite-clearance rejection, manual-incomplete-identity readability and non-rankability, generator-version retention, horizontal/vertical pattern separation, save/edit/import/export/merge/replace/duplicate-ID/cleanup, no-auto-promotion, free-text-linked-at-promotion, no-fabricated-cut-settings, tested-not-verified, frozen-fitContext, failed-promotion-rollback, deletion-independence, exact/one-sided/legacy machine matching, exact-vs-different family/template/generatorVersion/wallJoint/floorJoint/fingerCount/actualWidth rejection, incomplete-identity rejection, material-ID tiering, and the three thickness-tier boundaries. **Confirmed gap (IMPORTANT, §8):** no fixture directly asserts a successful save with a preferred candidate and **no** alternate — every alternate-related assertion in `runTabletopCouponResultsFixtures` supplies an alternate. This exact scenario was only verified by this audit's live browser test, not by the automated suite.

## 27. Exact focused totals (independently measured)

Each function was invoked directly in a running instance of the actual `index.html` (headless Edge, `file://`, no `?selftest` param interference for this pass, then re-confirmed under `?selftest=all` on a separate fresh profile):

| Group | Passed | Failed |
| --- | ---: | ---: |
| Tabletop engine | 27 | 0 |
| Tabletop coupon | 76 | 0 |
| Raw coupon results | 28 | 0 |
| Evidence promotion | 17 | 0 |
| Evidence matching | 21 | 0 |
| Help | 43 | 0 |
| Designs geometry | 1093 | 0 |
| Dice Tray | 92 | 0 |
| Gift Box | 69 | 0 |

All nine exactly match the reported totals.

## 28. Exact complete-suite total and group count

All 34 groups registered under `?selftest=all` were invoked directly (once on a fresh Edge profile, once again on a second independent fresh profile for reproducibility) and summed: **`2632 passed / 0 failed` across 34 groups**, reproducibly. **This differs from the reported `2629 / 0`** in the implementation report, README, and CHANGELOG by **+3**. All 34 individual groups reported zero failures both times; there is no functional regression, only a stale/incorrect manual arithmetic total in the documentation. Classified IMPORTANT under documentation accuracy (not a BLOCKER, since no test actually fails and every individually-reported focused total in §27 is exact).

## 29. Production SVG comparisons

The Tabletop coupon fixture group directly asserts and passed: `result.svg.length===2992 && designFixtureHash(result.svg)==='4f543f95'` — the exact reported pin, reconfirmed passing in both independent full-suite runs. Dice Tray (92/0), Gift Box (69/0), and Designs geometry (1093/0) — which respectively contain the Divider Tray, Joint Fit Coupon, alternate Dice Tray, Finger Box, Sliding-lid Box, and Drawer Cabinet byte/hash goldens referenced in the task — all passed with zero failures in both independent runs, confirming no unexplained golden drift anywhere in the legacy production-SVG surface.

## 30. Legacy-output comparisons

No unexplained change: `runDesignGeometryFixtures` (1093/0), `runDiceTraySystemFixtures` (92/0), and `runGiftBoxFixtures` (69/0) all passed at their exact previously-reported totals, and the diff itself touches no legacy template builder, serializer, or SVG-producing function outside the new Tabletop-coupon-adjacent additions.

## 31. Documentation accuracy

README accurately describes raw recording, explicit promotion, Coupon-proven limitation, "never change defaults," and lists all three new `?selftest=` routes; its complete-suite total (`2629/0` across 34 groups) shares the same **+3 discrepancy** identified in §28 (IMPORTANT, documentation only). CHANGELOG is bounded to two new bullet points describing the additive scope and validation totals, sharing the same total-count discrepancy. The implementation report is accurate except for the alternate-required wording addressed in §8 (IMPORTANT) and the same total-count discrepancy (§28). No claim of T2, Shell-proven, Production-proven, or automatic recommendations appears anywhere in the new documentation.

## 32. Direct file:// results

`git diff --check` clean (CRLF-only warnings). `python -m html.parser index.html` completed with exit code 0 and no output (no parse errors). Live `file://` testing in headless Edge confirmed: app startup with no console exceptions; Designs opens and the existing coupon generation is visually/functionally unchanged (default draft still produces a valid three-candidate result); the new recording workflow, manual-entry button, saved-results list, edit path (verified via source given the Library-profile setup cost of a full live promotion re-run), delete path, and Help section all function as documented; no network requests were required (`file://` load, no `fetch`/`XMLHttpRequest` observed); switching template selection between `tabletop-corner-floor-coupon` and back exercised the shared `#designForm` selector without error.

## 33. Protected boundaries

Confirmed unchanged by direct diff inspection: `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`. Machines M1/M2/M3 code is reused, never modified (only called). Material Tests, Test Grids, Library profile shape, existing Production Settings, and existing Evidence records are unmodified in shape except the one additive, optional `fitContext` field on evidence (spread conditionally, backward compatible). Promotion history (`promotionEvidenceSnapshot`) gained one additive optional field the same way. Project schema, accounting, Inventory, and Pricing have zero diff lines. The Tabletop coupon's own production geometry, panel construction, and SVG bytes are untouched — T1.5 only reads the coupon's already-computed `result.metrics`/`result.finishedView` after generation; `buildTabletopCornerFloorCouponDesignResult` itself has no diff. All legacy production SVGs remain byte-identical per §29/§30. Offline/no-network behavior is preserved (§32).

## 34. Findings by severity

**BLOCKER:** none.

**IMPORTANT:**
1. Implementation report states "a preferred candidate and tighter alternate are also required," which is inaccurate — the code and live-tested UI correctly treat the tighter alternate as optional. Misleading documentation, not a code defect. (§8)
2. `runTabletopCouponResultsFixtures` has no explicit assertion for a successful save with a preferred candidate and no alternate — a real, missing regression guard for exactly the scenario the report's wording confusion could have masked. (§8, §26)
3. The reported complete-suite total (`2629/0` across 34 groups, repeated in the implementation report, README, and CHANGELOG) undercounts the actual, reproducibly-measured total of `2632/0` across the same 34 groups by 3. All individual focused totals (§27) are exact; only the aggregate arithmetic is off. (§28, §31)

**POLISH:**
1. `tabletopCouponConstructionIdentityFromResult`'s fallback branch references nonexistent metrics field names (`horizontalCount`/`verticalCount`) and is dead code today; worth removing or fixing so it isn't misleading if the call graph ever changes. (§6)
2. "Promote to Library evidence" remains visible and identically labeled on an already-promoted record; re-promotion is safely handled via the existing non-blocking duplicate-source warning banner (not a defect), but a small UX improvement would relabel or visually distinguish the control once a record is already promoted. (§13)

**NOT A DEFECT (intentionally deferred, per the locked architecture):** recommendation-card UI, Project schema integration, Shell-proven workflow, Production-proven promotion, T2 Premium Rectangular Tray, automatic apply/recommendation selection, re-promotion hard-blocking (uses the app's existing warning convention instead).

## 35. Exact final verdict

**APPROVED WITH POLISH DEFERRED**

Justification: zero BLOCKER findings; all three IMPORTANT findings are documentation-accuracy issues (report wording, a fixture-coverage gap, and a total-count arithmetic slip) rather than functional defects — every locked architectural decision was independently verified correct in the actual source and in live `file://` interaction, including the one item (optional alternate) the task specifically flagged as suspicious. The two POLISH items are cosmetic/dead-code cleanups that do not affect correctness or safety. None of the findings block Joe from safely recording the real physical result or committing this phase; they should be swept up as a small follow-up (correct the report/README/CHANGELOG total to 2632, add the missing no-alternate fixture assertion, fix the alternate-required sentence, and optionally clean up the dead fallback code) either just before or just after commit, at the user's discretion.

## 36. May the current physical result be entered after commit?

**Yes.** The raw-save path (measured thickness 2.88 mm, C1/C2/C3 with signed clearances −0.075/−0.050/−0.025, preferred C2, tighter alternate C1) was live-tested end-to-end with no defect found in that exact shape of data (only the no-alternate variant was additionally live-tested to close the audit's specific concern). Entering it does not require any further code change.

## 37. Is T1.5 ready to commit?

**Yes**, with the three IMPORTANT documentation items ideally corrected first (or immediately after, as a fast-follow) since they are text-only edits with no functional risk: fix the implementation report's alternate-required sentence, correct the three documents' total to `2632 / 0 across 34 groups`, and add the missing no-alternate regression fixture. None of these require touching the storage model, promotion pipeline, or matching helper.

## 38. Does T2 remain blocked?

**Yes.** Nothing in this diff creates a complete rectangular tray, a Shell-proven record, a Production-proven promotion path, a recommendation-card UI, or a Project-schema integration. T2 remains gated on its own physical shell-assembly validation exactly as the architecture review locked.

## 39. Is a Grok audit needed?

**Yes**, as the standard second-pass adversarial check before or shortly after commit, consistent with this project's established cadence for additive-but-schema-adjacent phases — focused specifically on re-verifying the alternate-optional behavior and the construction-identity field sourcing this audit flagged as the highest-risk areas, since an independent second reviewer confirming those same conclusions would materially increase confidence before Joe records the real physical result.

## 40. Confirmation

No product source file, README, CHANGELOG, fixture, storage, or application record was edited during this audit. Nothing was staged, committed, or pushed. All browser-driven verification ran against a disposable, isolated headless-Edge profile directory outside the repository; the repository's tracked and untracked file sets were confirmed byte-identical before and after this audit. Only this report file was written.
