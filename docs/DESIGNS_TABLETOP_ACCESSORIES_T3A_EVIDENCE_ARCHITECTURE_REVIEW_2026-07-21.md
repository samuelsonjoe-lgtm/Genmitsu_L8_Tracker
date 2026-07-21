# Tabletop Accessories T3A — Evidence Recording — Architecture Review

**Date:** 2026-07-21
**Reviewer:** Claude Opus 4.8 (independent, read-only, architecture-only — nothing implemented)
**Scope:** design the safest additive way to record and promote the completed T3A physical validation without weakening, changing, or overstating any existing T1 or T2 evidence.

This was a static source-and-document review. I did **not** run fixtures or a browser this task; no runtime claims are made here except where explicitly attributed to the prior T3A independent audit. No file was edited except this new report; nothing was staged, committed, or otherwise changed; no browser profile or real `localStorage` was touched.

## 1. Actual repository state

- `git rev-parse HEAD` = `b71cfc1ece0976ed0789a1f4f9521a802840bb52` — **"Add Tabletop storage fit coupon"** (the T3A implementation is committed; the pre-T3A baseline `ec8f35f` is its parent).
- Branch `main`; `origin/main...main` = `0 0` (synchronized). Nothing staged; `git diff`/`git diff --cached` empty (clean tracked tree).
- Untracked set: the long-standing docs/utility files plus the two new T3A reports (implementation + independent audit). All preserved.
- All three named reports were read in full: the T3 architecture review, the T3A implementation report, and the T3A independent audit (verdict: SAFE TO CUT, geometry internally verified, 897 B / `b5c549ee`, focused 23/0).

## 2. Relevant existing evidence architecture (as read from source)

The T2B **shell-results** subsystem is a complete, working precedent that T3A should parallel — not extend.

- **Constants:** `SCHEMA_VERSION = 2` (index.html:323), `BACKUP_FORMAT` (:328). `productionEvidenceKinds` (:418) is the promoted-evidence `kind` allowlist and currently contains `tabletop-coupon` and `tabletop-shell` but **no** storage kind.
- **Raw-result collection:** `tabletopShellResults` is a first-class state array. It is created in `freshState` (:749), read in `loadState` via `normalizeTabletopShellResults(data.tabletopShellResults)` (:769), written in `persist` (:795), exported in `backupObject` (:1452), and handled in Replace (`replaceData` :13840) and Merge (`mergeData`/`mergeListStats`/`mergeList` :13859–13871). A missing collection normalizes to `[]`.
- **Normalization (:733–742):** `normalizeTabletopShellResult` returns `null` for malformed input (wrong `templateId`, missing thickness), spreads `{...value}` first (so **unknown fields are preserved**), and fixes the known fields. `normalizeTabletopShellResults` **dedupes by `id`**.
- **Observation tri-state (:723–727):** `bool(key)` → `true` / `false` / **`null`** (unknown); `choice(key,values)` → an allowed value or **`'not-tested'`**. Unknown is never coerced to a passing value.
- **Eligibility / promotion gate (:744, `tabletopShellResultEligibility`):** requires complete construction identity, generator version, exact machine identity, (for promotion) a Library material profile, finite requested + measured dimensions, in-range clearance, `overallResult === 'pass'`, specific damage/gap/racking observation gates, and at least one "meaningful" (non-null, non-`not-tested`) observation.
- **Promotion (:746–747):** `buildTabletopShellPromotionCandidate` calls the generic `promotionCandidateBase('tabletop-shell', …)` (:8929) with `defaultStatus:'tested'`, `canVerify:false`, then attaches `candidate.fitContext = { evidenceStage:'shell-proven', … immutable snapshot … }`. `finalizeTabletopShellPromotion` writes a `promotedEvidence` back-reference **into its own raw record only**. Promotion is persisted through the shared `savePromotionTransaction` (:9221).
- **Promoted-evidence normalization (:8791 `normalizeProductionEvidence`):** an **unknown `kind` is mapped to `'manual'`** via `productionKnownValue(item.kind, productionEvidenceKinds, 'manual')`, while the whole item (including `fitContext`) is preserved by spread. Unknown kinds are therefore neither rejected nor lost — they degrade to `manual` with context intact.
- **Matching (:9072 `tabletopRectangularShellEvidenceMatches`, :9051 `tabletopJointFitCompatible`, :9022 `tabletopCouponConstructionMatches`):** the shell matcher only considers `item.kind==='tabletop-shell'` with `evidenceStage==='shell-proven'` or `item.kind==='tabletop-coupon'` with `evidenceStage==='coupon-proven'`. **Any other kind/stage is invisible to it.** `tabletopJointFitCompatible` gates on `family`/`generatorVersion`/`wallCornerJoint`/`floorJoint`/sign convention/machine + finger-width tolerances — it describes finger-joint fit only.
- **Future-format guard (:500):** import is refused only when `data.schemaVersion > SCHEMA_VERSION`.
- **T2C startup hotfix:** the `tabletopShellObservationValues` const is declared **before** `loadState()` runs to avoid a temporal-dead-zone `ReferenceError` when stored records normalize at load.

**Conclusion:** every mechanism T3A needs already exists in a proven, generic form. T3A is a *third parallel collection*, exactly as `tabletopShellResults` was added alongside `tabletopCouponResults`.

## 3. Why evidence was previously deferred — precise reasons

The implementation report's "requires a new persisted evidence-record schema" is accurate but under-specified. Precisely:

- **What must change (all additive):** a new root collection `tabletopStorageResults` with its own `normalizeTabletopStorageResult(s)` and observation normalizer; wiring in `freshState`, `loadState`, `persist`, `backupObject`, `replaceData`, `mergeData`; one new string appended to `productionEvidenceKinds`; a new `evidenceStage` value; a dedicated exact-matcher and promotion builder; and a small recorder UI. No existing structure is rewritten.
- **`SCHEMA_VERSION`:** **must NOT change.** T2B added `tabletopShellResults` at version 2 without a bump; `loadState` reads `data.schemaVersion || 1` and normalizes any missing collection to `[]`. Bumping would make the future-format guard (:500) cause *older* app versions to refuse otherwise-importable backups — strictly worse.
- **Migration:** **none required.** Old records remain valid unchanged; a record predating T3A simply lacks the new collection, which loads as `[]`.
- **Old backups importable:** yes, unchanged (missing key → `[]`).
- **Newer backups (with T3A records) in older apps:** fail safe. Same `schemaVersion` (2) means the old app imports normally; it ignores the unknown `tabletopStorageResults` key (raw T3A results silently dropped, not corrupted), and any promoted T3A evidence inside a profile degrades its `kind` to `manual` with `fitContext` preserved. No crash, no data corruption.
- **Unknown source types:** preserved-as-`manual` in `productionEvidence` (:8795); as a top-level collection key, unknown keys are simply not read. Neither path throws.
- **Existing raw envelope reuse:** **do not** reuse the `tabletopShellResults` envelope. Its normalizer hard-rejects any `templateId !== 'tabletop-rectangular-shell-prototype'` and its fields (requested/measured finished interior, corner/floor-gap observations) are shell-specific and wrong for a rail mechanism. T3A needs its **own** record subtype.
- **Generic extension object:** not advisable as the primary carrier — a typed T3A record with an explicit normalizer gives the same forward-compat (via `{...value}` spread) plus real validation and matchable identity fields.
- **Does existing promotion assume joint-fit/shell?** Yes — `tabletopRectangularShellEvidenceMatches` and `tabletopJointFitCompatible` are finger-joint/shell-specific and ignore other kinds/stages. This is *why* T3A is safe to add: a new kind/stage is inert to them, so a dedicated matcher is required (and cannot accidentally cross over).

## 4. Recommended persisted record structure (`tabletopStorageResults[]`)

One raw record per physical test, normalized like the shell record (spread-preserve unknowns, dedupe by id, malformed → `null`). Proposed normalized shape:

```
{
  id, createdAt, updatedAt, testedAt,
  sourceMode: 'generated' | 'manual',
  template: 'tabletop-storage-fit-coupon',
  generatorVersion: 'tabletop-storage-fit-coupon-t1',
  mechanismIdentity: {                       // §5 — the RAIL mechanism, not the shell
    construction: 'bottom-supported-four-rail-removable-panel-t1',
    railCount: 4, falseBottom: 'removable-loose'
  },
  outerShellLink: {                          // §7 — link + frozen snapshot
    sourceResultId, sourceSettingId, sourceEvidenceId,   // may be '' / missing
    snapshot: { compatibilityIdentity(tabletop-accessory-t2),
                requestedInterior:{width,depth,usableHeight},
                measuredThicknessMm, clearanceMm, evidenceStage:'shell-proven' }
  },
  machine:  {machineId,…},                   // reuse tabletopCouponNormalizeMachineSnapshot
  material: {materialId,label,type},         // reuse tabletopCouponNormalizeMaterialSnapshot
  nominalThicknessMm, measuredThicknessMm, clearanceMm,   // shell joint clearance (−0.075)
  requestedInterior:{width,depth,usableHeight},
  preferredFingerWidthMm,
  patterns:{widthAxisFloor,depthAxisFloor,verticalCorner},  // actual finger geometry
  railHeightMm, falseBottomThicknessMm, totalPanelClearanceMm,
  derived:{ falseBottomWidthMm, falseBottomDepthMm, perSideClearanceMm,
            frontRailLengthMm, backRailLengthMm, leftRailLengthMm, rightRailLengthMm,
            railOverlapMm, remainingVisibleWallHeightMm },   // recomputed + cross-checked
  observations:{…},                          // §6 tri-state
  overallResult: 'pass' | 'fail' | 'incomplete',
  notes: '',
  promotedEvidence: null | {profileId,settingId,evidenceId,promotedAt}
}
```

`derived.*` are stored for display and integrity-checked against the identity fields at normalize time (e.g. `frontRailLength === interiorWidth − 2×widthAxisFloor.actualWidth`); they are not independent inputs.

## 5. Source / generator / construction / stage identities

| Item | Recommended value | Verdict |
| --- | --- | --- |
| Promoted-evidence `kind` (append to `productionEvidenceKinds`) | `tabletop-storage-fit-coupon` | **Approve** |
| Promotion `sourceType` | `tabletop-storage-fit-coupon` | **Approve** (equals the kind, as `tabletop-shell` does) |
| Generator version | `tabletop-storage-fit-coupon-t1` | **Approve** (already emitted by the generator) |
| Mechanism construction identity | `bottom-supported-four-rail-removable-panel-t1` | **Approve** — store on `mechanismIdentity.construction`, kept separate from the outer-shell finger-joint identity |
| Raw-result label | `Tabletop Storage Fit Coupon result` | **Approve** |
| Internal `evidenceStage` (on `fitContext`) | `storage-fit-coupon-proven` | **Approve** (kebab, parallel to `coupon-proven`/`shell-proven`) |
| User-facing promoted stage label | `Storage Fit Coupon-proven` | **Approve** |

The stage label is intentionally distinct from `Coupon-proven` (joint-fit), `T1 corner coupon`, `Shell-proven`, `Production-proven`, and any future `Storage Tray-proven`. **Do not** shorten it to "Coupon-proven" (that name already means joint-fit). The mechanism identity string is deliberately different from the finger-joint `compatibilityIdentity`, so the rail mechanism is never confused with the shell's corner joinery.

## 6. Observation model (tri-state; unknown never becomes pass)

Booleans return `true` / `false` / `null` (unknown, via the shell's `bool()` idiom); no field is coerced to a pass. Recommend **not** adding a `not-applicable` state — every field below applies to the tested mechanism, and a fourth state risks masking an unknown.

| Field | Type | Good value |
| --- | --- | --- |
| `preGlueTestCompleted` | bool | true |
| `postCureTestCompleted` | bool | true |
| `panelRestsOnAllFourRails` | bool | true |
| `panelLiesFlat` | bool | true |
| `panelRocks` | bool | false |
| `panelBinds` | bool | false |
| `panelLiftsAndReseatsCleanly` | bool | true |
| `clearanceReasonablyEven` | bool | true |
| `railsSeatAgainstFloorAndWall` | bool | true |
| `railBondSurvivesTesting` | bool | true |
| `visibleSag` | bool | false |
| `panelDamage` | bool | false |
| `veneerDamage` | bool | false |
| `shellRemainedSquare` | bool | true |
| `originalDiagonal1Mm` / `originalDiagonal2Mm` | finite +ve or null | 100 / 100 |
| `postCureDiagonal1Mm` / `postCureDiagonal2Mm` | **null** | left unknown |
| `notes` | string | — |

**Joe's result maps cleanly:** all "confirmed" pre-glue and post-cure booleans → `true`; the negatives (rocking, binding, sag, panel/veneer damage) → `false`; `shellRemainedSquare` → `true`; originals `100/100`; **post-cure diagonals stay `null`** (he confirmed square but did not measure a new number — do not invent one).

## 7. Outer-shell evidence-link strategy

**Store both** an id reference and an immutable snapshot.

- **Reference:** `outerShellLink.sourceResultId` (+ `sourceSettingId`/`sourceEvidenceId` if promoted) enables a "View outer-shell evidence" jump when the T2 record still exists.
- **Snapshot:** a frozen copy of the T2 `compatibilityIdentity` (`tabletop-accessory-t2`), requested interior, measured thickness, clearance, and `evidenceStage:'shell-proven'`, captured at T3A test time. This preserves the T3A record's meaning even if the T2 record is later edited/deleted or absent from an imported backup.
- **Missing / deleted / imported-without-source:** allowed and non-blocking — the UI renders the outer shell from the snapshot with a caveat ("linked outer-shell record not found; showing recorded snapshot").
- **No mutation coupling:** the link is one-directional (T3A → T2 by id). T3A **never** writes to or reclassifies the T2 record; editing the T2 record cannot silently change the T3A result because the snapshot is frozen. A linked T2 record is never re-tagged as T3A evidence.
- The UI can therefore state independently: **Outer shell: T2 Shell-proven** and **Storage mechanism: Storage Fit Coupon-proven**.

## 8. Exact-match contract

**Phase 1 = exact matching only; compatible matching deferred.** Approximate/tolerant matching is explicitly rejected here because it could imply an untested rail height, clearance, or thickness is proven.

- **Required identity fields (all must match, mm compared at `epsilon = 1e-6`):** `template`, mechanism `generatorVersion` (`tabletop-storage-fit-coupon-t1`), mechanism `construction` (`bottom-supported-four-rail-removable-panel-t1`), outer-shell `compatibilityIdentity` (family `tabletop-accessories`, `generatorVersion:'tabletop-accessory-t2'`, `wallCornerJoint:'finger-90'`, `floorJoint:'finger-jointed-floor'`, Floor-owned), `requestedInterior.width/depth/usableHeight`, material identity (`materialId`; else material snapshot label), `nominalThicknessMm`, `measuredThicknessMm`, `clearanceMm` (shell joint clearance), `patterns.widthAxisFloor.actualWidthMm`, `patterns.depthAxisFloor.actualWidthMm`, `railHeightMm`, `falseBottomThicknessMm`, `totalPanelClearanceMm`. Machine identity via the existing `productionMachineIdentityMatches` (exact id, legacy-key fallback).
- **Derived verification fields (consistency-checked, not separate gates):** `falseBottomWidth/Depth`, per-side clearance, the four rail lengths, `railOverlap`, `remainingVisibleWallHeight`. Because these are pure functions of the identity fields, an exact identity match implies them; store + assert them, but they are not independent match keys.
- **Descriptive snapshot fields (never affect matching):** material label, machine label, `testedAt`, outer-shell link.
- **Observations & notes:** never identity fields.
- **Numeric normalization:** reuse the existing `tabletopCouponFinitePositive`/`FiniteNumber` helpers; compare mm at `1e-6`; thickness/clearance stored to full precision (no rounding to 2 decimals for matching). A dedicated `tabletopStorageExactMatch(target, evidence)` returns boolean-exact only; it must **not** reuse `tabletopJointFitCompatible` (which is tolerant by design).

Changing rail height, panel clearance, cavity size, material/thickness, or construction identity must each independently break the exact match (each is a required field).

## 9. Promotion gates (raw → Storage Fit Coupon-proven)

Promotion requires **all** of:

- `overallResult === 'pass'`; complete valid dimensions and identity; `generatorVersion` + mechanism construction identity present; exact machine identity; a chosen Library material profile; `testedAt` present; explicit user confirmation.
- Observations required **true**: `preGlueTestCompleted`, `postCureTestCompleted`, `panelRestsOnAllFourRails`, `panelLiesFlat`, `panelLiftsAndReseatsCleanly`, `clearanceReasonablyEven`, `railsSeatAgainstFloorAndWall`, `railBondSurvivesTesting`, `shellRemainedSquare`.
- Observations required **false**: `panelRocks`, `panelBinds`, `visibleSag`, `panelDamage`, `veneerDamage`.
- Any required observation that is **`null` (unknown) blocks promotion** — unknown is never treated as pass.

Rules (matching existing conventions):

- A **marginal** result stays raw (`overallResult` `fail`/`incomplete`, marginal detail in `notes`) and **cannot** be promoted.
- A **failed** result remains visible as a raw record; never promotable.
- Promoted evidence is **immutable**; corrections create a new/superseding raw record and a fresh promotion rather than editing evidence in place.
- **Duplicate promotions blocked:** a record already carrying `promotedEvidence` cannot be re-promoted unless demoted (reuse `savePromotionTransaction`/existing demotion path).
- **Deletion** follows existing raw-result conventions (unpromoted deletes freely; promoted deletion warns/keeps the Library evidence as today).

**Manual-entry workflow (test already done):** open the recorder from the T3A Designs result view with the exact same inputs in hand (`sourceMode:'generated'` regenerates the deterministic design to freeze identity + derived dims), then attach **manually entered** pre-glue and post-cure observations, the real past `testedAt`, and notes. This records the completed result honestly — user-entered observations over a regenerated design, with the true test date — without pretending the app captured it contemporaneously.

## 10. UI workflow (smallest safe addition)

A **dedicated Tabletop Storage Fit Coupon result recorder** — a new `#tabletopStorageResultsModal` — reusing the modal/promotion plumbing, **not** overloading the shell recorder (its fields are shell-specific). Placement mirrors the shell subsystem exactly:

1. A **"Record physical result"** action on the Tabletop Storage Fit Coupon Designs result view.
2. A review step showing all exact inputs + derived dimensions before saving.
3. Pre-glue and post-cure observation entry (tri-state controls; unknown default).
4. A section that visually separates **Outer shell (T2 Shell-proven)** from **Support mechanism (unproven → Storage Fit Coupon-proven once promoted)**.
5. A promote action gated by §9, writing promoted evidence to the chosen **Library** profile's `productionSettings` evidence (exactly like shell) — the raw records list lives in the Designs/Tabletop results surface, promoted evidence lives in the Library, as today.
6. On regenerating a matching T3A configuration, a dedicated exact-match card: **"Storage Fit Coupon-proven — Exact"** for the mechanism, the outer-shell status shown separately, and an explicit line that **cork fit and complete-tray behavior remain unproven**.
7. A results list with edit/promote/delete parallel to `tabletopShellResults`.

Belongs in the existing Designs/Tabletop-results surface plus Library for promoted evidence — **not** a generalized tabletop-results rewrite and **not** a new top-level surface.

## 11. Normalization, migration, backup, import, startup safety

- **`SCHEMA_VERSION` unchanged (2); no migration.** Old data loads unchanged; missing `tabletopStorageResults` → `[]`.
- **Malformed T3A record** → `normalizeTabletopStorageResult` returns `null` → filtered out (fail-safe drop, no throw).
- **Unknown future observation fields** preserved via `{...value}` spread; unrecognized enum/bool values normalize to `null`/`not-tested` (never pass).
- **Unknown future construction version** (e.g. `-t2`): stored as-is, but the exact matcher requires `-t1`, so it will not match a `-t1` target — correct isolation.
- **Duplicate record IDs** deduped by `normalizeTabletopStorageResults` (mirror the shell dedupe).
- **Imported T3A with missing linked T2** → kept; outer shell shown from snapshot with caveat (§7).
- **Backup round trip:** add `tabletopStorageResults` to `backupObject`, `replaceData`, and `mergeData`/`mergeListStats`/`mergeList` (id-based), mirroring the shell lines. Newer backup in older app degrades safely (§3).
- **Startup:** declare any new `tabletopStorageObservationValues` const **before** `loadState()` (the T2C hotfix lesson) to avoid a temporal-dead-zone `ReferenceError` when seeded T3A records normalize at load. Direct `file://` operation is unaffected.
- **Do not change:** `STORAGE_KEY`, `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `BACKUP_FORMAT`, existing record meanings, Project schema, Library schema.

## 12. Fixture plan (focused `runTabletopStorageResultFixtures`)

Prove, with named per-assertion checks (not bundled `.every()`):

- old `localStorage` without T3A records loads unchanged; existing T1/T2 evidence and records unchanged;
- T3A raw-result creation and id-dedupe;
- exact preservation of the tested dimensions (80×60×25, 2.88, −0.075, rails 60.942/41.211, false bottom 79.6×59.6, overlap 2.68, visible wall 12.12);
- false-bottom clearance retains total-across-dimension semantics (0.40 total → 0.20/side);
- unknown observations remain `null` (not pass); a `null` required observation blocks promotion;
- a failed / incomplete-post-cure result cannot promote; a fully-qualifying result promotes;
- promoted `kind` is `tabletop-storage-fit-coupon`, stage label `Storage Fit Coupon-proven`, `evidenceStage:'storage-fit-coupon-proven'`;
- T3A never becomes `Shell-proven`; T2 shell/coupon evidence never proves T3A (shell matcher ignores the new kind; T3A matcher ignores shell/coupon kinds);
- T3A evidence never proves cork fit or a complete storage tray (assert the "unproven" UI strings and that no such stage exists);
- exact match succeeds for the tested config; changed rail height / panel clearance / cavity size / material / thickness / construction identity each break the exact match;
- compatible matching does not exist yet (assert no tolerant match path);
- linked T2 evidence remains independently scoped; missing linked T2 fails gracefully (snapshot render);
- backup export includes the collection; old backup without it is accepted; merge is id-based; replace restores it; round-trip byte-stable;
- malformed record fails safe; startup with seeded T1 + T2 + T3A evidence raises no exception (no TDZ); direct `file://` operation;
- **existing production SVG hashes unchanged** (T3A 897/`b5c549ee`, T1 2992/`4f543f95`, T2 2337/`ed5d6f6e`, legacy 2181/`2ef9606b`, trays, joint coupon) — evidence work touches no geometry.

## 13. Protected boundaries

**Touched (new / additive only):** append one string to `productionEvidenceKinds`; new `tabletopStorageObservationValues` const (declared pre-`loadState`); `freshState`/`loadState`/`persist`/`backupObject`/`replaceData`/`mergeData` gain `tabletopStorageResults` lines; new `normalizeTabletopStorageResult(s)`, `tabletopStorageNormalizeObservations`, `tabletopStorageResultSave`, `tabletopStorageResultEligibility`, `buildTabletopStorageResultRecord`, `buildTabletopStoragePromotionCandidate`, `finalizeTabletopStoragePromotion`, `tabletopStorageMechanismIdentityFromResult`, `tabletopStorageExactMatch`/`…Matches`, `tabletopStorageRecommendationHtml`, recorder open/render/bind + `#tabletopStorageResultsModal`; a "Record physical result" entry on the T3A Designs result view; new fixture group + `?selftest` route; README/CHANGELOG.

**Must remain unchanged:** `buildBoxModel`, `buildFingerPattern`, `tabletopJointFitCompatible`, `tabletopCouponConstructionMatches`, `tabletopRectangularShellEvidenceMatches`, `buildTabletopShellPromotionCandidate`, `normalizeTabletopShellResult(s)`, `tabletopShellResultEligibility`, `productionMachineIdentityMatches`, the generic `promotionCandidateBase`/`promotionAddField`/`savePromotionTransaction`/`normalizeProductionEvidence` (reused, not modified), all existing T1/T2 evidence records and their meanings, T1/T2 identity separation, `SCHEMA_VERSION`/`STORAGE_KEY`/`BACKUP_FORMAT`/`APP_*`, Project and Library schemas, machine identity, T3A production geometry and all production SVG bytes/hashes, kerf conventions, LightBurn colors, offline `file://` startup, the startup-initialization hotfix, and all unrelated generators.

*Note:* appending to `productionEvidenceKinds` edits a shared array, but it is purely additive — existing kinds and their handling are unchanged, and it is required so the current app preserves (rather than degrades-to-`manual`) the new promoted kind.

## 14. Implementation phases

1. **T3A-E1 (this phase — recommended single Codex phase):** additive `tabletopStorageResults` record type + normalization + observation model; recorder UI + manual-entry workflow; exact-match-only matcher; promotion to **Storage Fit Coupon-proven**; backup/import/merge/startup wiring; focused fixtures. No compatible matching, no cork evidence, no full Tabletop Storage Tray, no Project/Library schema change, no shared-geometry change, no `SCHEMA_VERSION` bump.
2. **T3A-E2 (deferred):** compatible/related-size matching for the mechanism, only once a second physical configuration is proven.
3. **T3B+ (deferred):** cork-fit evidence and the complete Tabletop Storage Tray, each with its own identity and its own physical proof — never inheriting T3A status.

## 15. Risks and open decisions

- **Risk — stage-name overreach.** Mitigated by the explicit, distinct `Storage Fit Coupon-proven` label and the "cork/complete-tray unproven" UI line; the matcher proves only the exact tested mechanism.
- **Risk — cross-contamination.** Mitigated bidirectionally: the shell/coupon matchers gate on their own kinds/stages (ignore T3A); the T3A matcher gates on the T3A kind/stage (ignores shell/coupon). Fixtures assert both directions.
- **Risk — silent T2 drift changing T3A meaning.** Mitigated by the frozen outer-shell snapshot (§7).
- **Open decision (non-blocking):** whether promoted T3A evidence should live on the same Library profile as the outer-shell's Shell-proven evidence or be independent. Recommend independent evidence on the chosen material profile, linked to the outer shell by snapshot/id — no schema change either way.
- **Open decision (non-blocking):** exact display wording of the recorder and stage badge (recommended values above are clear and consistent; final wording is a preference).
- **Open decision (non-blocking):** whether to store `overallResult` as `['pass','fail','incomplete']` (recommended, parallel to shell) or add an explicit `marginal` — the gates behave identically since only `pass` promotes.

## 16. Exact verdict

The existing architecture supports an **accurate, purely additive** T3A evidence subsystem with **no `SCHEMA_VERSION` bump, no migration, no shared-geometry change, and no Project/Library schema change**. Exact-match-only recording and promotion to a distinct **Storage Fit Coupon-proven** stage can be added safely, with honest outer-shell/mechanism separation and no path by which T2 proves T3A or T3A claims Shell-, Production-, cork-, or complete-tray proof.

**READY FOR A BOUNDED CODEX IMPLEMENTATION PROMPT**, scoped to **T3A-E1** (§14.1). Every identity, matching, observation, promotion, persistence, and boundary question is resolved with one recommended answer; no further architectural decision is required before implementation.
