# Tabletop Accessories T1.5 — Physical Coupon Results and Evidence Reuse — Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-20
**Reviewer:** Claude Sonnet 5
**Type:** read-only architecture review. No product file, README, CHANGELOG, fixture, storage, or application record was edited. Nothing was staged, committed, or pushed. Only this report was written.

---

## 1. Repository state and actual HEAD

- `git status -sb`: clean tracked tree; only the long-standing unrelated untracked set present (LightBurn projects, `debug.log`, historical `docs/*.md`, `parametric_qr_stand_generator.py`) — all preserved, none inspected further.
- **Actual HEAD: `0b7f89f275cc19e4be763c10e1e9753d5666ff28` — "Add Tabletop Accessories engine and joint coupon."** This is the authoritative baseline for this review (the task's stated "latest completed phase" description matches this commit's content).
- Branch `main`; `git rev-list --left-right --count origin/main...main` = `0 0` (synchronized). `git diff --check` clean. Nothing staged.
- All seven referenced T1 reports are present on disk (`ARCHITECTURE_REVIEW`, `CONSTRUCTION_CONTRACT_REVIEW`, `T1_ENGINE_COUPON_IMPLEMENTATION`, `_FOCUSED_AUDIT`, `_CORRECTION`, `_CORRECTION_VERIFICATION`, `_T1_SUMMARY_FIX`) and were read for context; the implementation report's own final count (§39: "2559 passed, 0 failed across 31 groups") matches the task's stated current suite total exactly.
- Constants confirmed: `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.

## 2. Existing systems inspected

Direct source read of `buildTabletopCornerFloorCouponDesignResult` (the actual T1 coupon builder) confirmed its real `metrics`/`finishedView` shape — this review's recommendations are grounded in that exact shape, not an assumed one (see §11-12). Also inspected, drawing on this session's own prior deep audits of the same codebase: `normalizeMaterialTests` (flat `machine`/`machineKey`/`machineId` strings, speed/power/passes-shaped), `normalizeProductionEvidence` (flat `sourceId`/`sourceProfileId`/`sourceLabel`/`machineId`/`date`/`result`/`notes`), `normalizeProductionSetting` (nested `machine{key,label,machineId,lens,focusMethod}`, `materialCondition{nominalThicknessMm,measuredThicknessMm,measurementScope}`, `cutSettings{...}`, **`fitSettings{fingerJointClearanceMm,...}`**, `lightBurn{kerfOffsetMm}`, `verificationStatus`, `evidence[]`), `productionMachineIdentityMatches` (exact-ID-required-if-either-side-has-one, else legacy key match — the committed M3 rule), `productionSettingsCanSupersede`, the promotion pipeline (`promotionCandidateBase`, `promotionEvidenceSnapshot`, `savePromotionTransaction`, explicit-review-before-save), Project `settings.machine{machineId,key,label}` snapshot convention, the generic Design-to-Project handoff (`projectDraftFromDesign`), `activeOwnedMachineSnapshot()`, `machineReferencePaths` (M2 deletion-protection scanner), and the generic list-merge machinery (`mergeList`/`mergeListStats`) used by every existing root collection.

## 3. Physical-result evidence summary (locked interpretation, restated)

Machine: Genmitsu L8 20W (use `machineId` when the active owned machine has one; otherwise the existing legacy key). Material: plywood, nominal 3 mm, **measured 2.88 mm** — no species/vendor/product/coating identity invented. Construction: `tabletop-corner-floor-coupon`, 90° wall-to-wall finger corner + finger-jointed floor, preferred finger width 9 mm, requested interior leg 40 mm, wall height 22 mm. Candidates: C1 −0.075 mm, C2 −0.050 mm, C3 −0.025 mm. **Preferred: −0.050 mm** (good friction fit, assembles most easily, self-supports only after ≥2 connected walls). **Tighter alternate: −0.075 mm** (tighter friction fit, a single wall can self-support). C3 not selected. This is joint-fit-and-assembly evidence only — **not** proof of a complete tray, long-term strength, glue performance, or production readiness, and must never become a universal default.

## 4. Architecture options evaluated

- **Option A (Material Test subtype) — rejected.** `normalizeMaterialTests` is shaped for laser-setting outcomes (`winningSpeed`, `winningPower`, `winningInterval`, `winningPasses`, a fixed `testType` enum like `'engrave_matrix'`/`'cut'`/`'kerf'`). A joint-fit clearance coupon has no speed/power/passes to report. Forcing it in would mean fabricating meaningless laser-setting values or leaving core Material Test fields blank — exactly the semantic distortion the task warns against.
- **Option B (directly create evidence/production setting on save) — rejected as the *save-time* behavior, but its destination is reused.** The existing promotion pipeline's entire design philosophy — established and audited repeatedly in this codebase — is *explicit review before evidence creation, never automatic*. Auto-creating evidence at save time would violate that established, already-trusted pattern and would also assume a Library material profile exists at the moment of recording, which it may not.
- **Option C (a new, independent coupon-result root collection with its own full lifecycle) — rejected as a standalone destination, but its storage mechanism is reused narrowly.** A wholesale parallel record system duplicates normalize/merge/dispatch machinery the app already has proven answers for elsewhere. Justified only in the narrow form described below (§5).
- **Option D (Design-linked raw result, evidence only through explicit promotion) — selected.** This is the only option that (a) avoids distorting Material Test's shape, (b) does not assume a Library profile exists before the user has even decided to keep the result, (c) matches the task's own explicitly favored policy almost exactly, and (d) has a real, already-existing, well-fitting *destination* for the promoted stage: `normalizeProductionSetting`'s `fitSettings.fingerJointClearanceMm` field already exists, is already signed, and is already used for exactly this kind of clearance evidence elsewhere in the app.

## 5. Selected record model

**One new, small, optional, additive root array — `tabletopCouponResults` (name illustrative) — holding raw physical observations, with promotion into the *existing, unmodified* Production Setting + Evidence system as the only path to reusable Library evidence.** This is not "parallel duplicate records": there is exactly one lifecycle, one direction, and one source of truth at each stage (§7). The new collection exists only because no existing record shape can honestly hold "per-candidate joint-fit observations plus a preferred/alternate selection" without either distortion (Material Test) or premature Library commitment (Evidence/Production Setting created before the user has decided to keep the result, or before a Library profile for the material exists).

Each raw record holds: `id`, `template:'tabletop-corner-floor-coupon'`, `generatorVersion` (copied verbatim from the result's own `generatorVersion`), a **construction identity** snapshot (§10), a **machine** snapshot (§8), a **material** snapshot (§9, Library-linked *or* free-text — never invented), `measuredThicknessMm`, the three **candidate observations** (§11), `preferredCandidateId`/`alternateCandidateId` (§12), overall `notes`, `dateTested`, and a nullable `promotedEvidence:{profileId,settingId,evidenceId}` back-reference set only once promotion occurs.

## 6. Storage and schema contract

- **No new `localStorage` key.** The new array lives inside the same `genmitsu-l8-tracker-v1` object, exactly like every existing root collection.
- **No `SCHEMA_VERSION` change.** This is purely additive — the same treatment already given to M1's `machines` array and M3's optional `machineId` fields, neither of which required a bump.
- **No destructive migration, no legacy backfill.** A legacy backup lacking this field normalizes to `[]` via the same `Array.isArray(data.x) ? data.x.map(normalize) : []` idiom used everywhere.
- **Old imports and old exports remain fully valid.** An older app version importing a *new* backup simply does not recognize the new array key and ignores it (consistent with every other additive field added in M1-M3); a newer app importing an *old* backup sees the field as absent and defaults to empty — no failure mode either direction, and no encrypted-backup-wrapper conflict (this data is no more sensitive than existing free-text notes and rides inside whatever wrapper eventually wraps the whole backup object).
- **Merge/Replace:** reuse the existing generic `mergeList`/`mergeListStats` (id-keyed union, local wins on conflict) and full-replacement-on-Replace — no new merge logic required.
- **Deletion protection:** a raw coupon result is *not* itself a target of the M2 machine-reference scanner (it stores a machine *snapshot*, not a live dependency that blocks machine deletion) — but the reverse must hold: if the *machine* referenced by a raw result is later archived or its identity becomes unavailable, the record must remain fully readable via its frozen snapshot fields, exactly like every other "unavailable reference" case already established in this app.
- **Byte-compatibility:** every existing record type, constant, and fixture golden remains untouched; this addition only ever appends a new, empty-by-default array field.

## 7. Source-of-truth behavior

Two independent sources of truth, one direction of flow, never the reverse:
1. **The raw `tabletopCouponResults` record** is the source of truth for "what did I physically observe on this date, on this machine, with this stock." It is never auto-generated from, and never auto-updates, Library evidence.
2. **The Library Production Setting / Evidence entry**, created only by explicit promotion, is the source of truth for "what does the Library consider evidence-backed for this material profile." It is a frozen snapshot at promotion time (matching the app's existing, well-established "evidence is a snapshot, not a live reference" convention) and is never retroactively altered by later edits to the raw record (§15).

## 8. Machine identity

Reuse the committed M3 machine model exactly, with no new rule: default the raw record's machine snapshot to `activeOwnedMachineSnapshot()` (giving a stable `machineId` when the active owned machine has one, `key:'custom'`, and a frozen label); if no owned machine is active, fall back to the existing legacy Genmitsu-profile-derived snapshot exactly as every other record type already does. Matching for recommendations reuses `productionMachineIdentityMatches` **unchanged**: exact `machineId` equality required whenever either side has one (a one-sided ID never counts as a match); legacy key-based fallback only when *neither* side has an ID; no historical backfill; a machine that is later archived does not invalidate a prior match, since matching is evaluated against the frozen snapshot, not the machine's current archived status. Recommendation ranking treats an archived-machine-with-matching-ID as full-confidence-but-annotated ("this machine has since been archived"), never as a hard mismatch.

## 9. Material identity

Support **both** an optional Library `materialId` (when the user selects an existing profile) **and** a free-text material label with `measuredThicknessMm` (when no profile exists or is selected) — mirroring how "material" already works as a free-text field with optional Library linkage everywhere else in this app (Log entries, Test Grids). Never invent species, vendor, product name, coating, or a Library identity that wasn't actually selected — this is a hard rule, matching the task's own locked constraint. If a `materialId` is present, its name/type snapshot is stored alongside it (display-only, so the record stays readable even if that profile is later deleted).

## 10. Thickness matching

**Recommend exact measured-thickness matching within an explicit, principled tolerance equal to the coupon's own tested clearance step (default 0.05 mm), not a casually-invented number and not brittle floating-point equality.** Physical rationale: the coupon system's own smallest meaningful clearance increment is the step value the user already chose when generating candidates — using that same increment as the thickness-match tolerance means "close enough to matter" is defined by the same physical granularity the evidence itself was tested at, not an arbitrary constant. Beyond that tolerance but within a wider band (recommend 3× the step, i.e. 0.15 mm by default) the match is shown as "close — verify stock before relying on this" rather than silently treated as exact. Beyond the wider band, the evidence is not offered as a ranked match at all (it may still be viewable via "View all evidence," never hidden). Nominal-thickness-only matches (3 mm vs 3 mm) with differing *measured* thickness are explicitly a lower-confidence tier than measured-thickness matches, since the entire physical lesson motivating T1.5 is that nominal and measured diverge and measured is what determines fit.

## 11. Construction identity

A structured, versioned identity — built entirely from fields the T1 result already produces, no invention required:

```
{
  family: 'tabletop-accessories',
  constructionTemplate: 'tabletop-corner-floor-coupon',
  generatorVersion: 'tabletop-accessory-t1',   // from result.generatorVersion
  wallCornerJoint: 'finger-90',                 // from candidate.wallJoin.type
  floorJoint: 'finger-jointed-floor',           // from candidate.floorJoin.type
  materialThicknessMm: 2.88,                    // from metrics.materialThickness
  actualFingerWidthMm: <per-candidate actual>   // from metrics.actualFingerWidths, NOT the requested preferredFingerWidth
}
```

**Match only on the exact, complete combination** — not "any 90° finger joint" broadly. Rationale: floor-jointed vs. non-floor-jointed construction materially changes assembly/squareness behavior (the exact physical lesson this phase exists to capture), so blurring the match would let evidence leak across meaningfully different constructions. A **different `generatorVersion`** (a future engine revision) must **never** silently match — construction identity is versioned specifically so a future T1.x/T2 engine change cannot silently inherit stale evidence; it is instead surfaced as "no matching evidence for this construction version" until re-tested. Matching keys on the **actual generated finger width**, not the requested `preferredFingerWidth` input, because the requested value is only ever a target (the app's own established language: "the exact width is calculated from an odd segment count") — two coupons with slightly different requested widths that happen to generate the identical actual pattern are the same construction; two with the identical requested width that generate different actual patterns (different leg length) are not.

## 12. Candidate observation model

A small controlled vocabulary plus freeform notes, not a laboratory form. Per candidate (C1/C2/C3): `{id, clearanceMm, fitClassification, selfSupport:boolean, assemblyEffort, notes}`, where:
- `fitClassification` ∈ a fixed enum: **Good friction fit / Glue fit (loose, needs glue) / Too tight / Loose / Failed (broke or split)** — five options, matching real workshop vocabulary rather than a lab scale.
- `selfSupport` is the specific observation this test surfaced as meaningful ("could one wall stand alone" vs. "needed two connected walls") — a simple boolean plus an optional free-text qualifier (e.g., "single wall," "two walls"), not a numeric scale.
- `assemblyEffort` is a short free-text field ("assembled easily," "needed persuasion") rather than another enum — this is exactly the kind of workshop-voice detail (e.g., "C2 assembled more easily") that a rigid form would lose.
- `notes` is fully freeform for anything else observed.

## 13. Preferred and alternate behavior

Support **exactly one preferred result (required to save), an optional tighter alternate, and no other alternate slots.** An "easier/glue-fit alternate" slot is explicitly **not** added in T1.5 — the current evidence only needs preferred + tighter-alternate, and adding an unused third slot now would be speculative. "No winner / inconclusive" is supported as a valid raw-record state (all three candidates recorded, none marked preferred is *not* allowed for a *save*, but the record can be left as **draft/unsaved** in the dialog without being forced to commit prematurely — however once saved, a preferred selection is mandatory, matching the current locked evidence which does have a clear winner). For the current evidence: `preferredCandidateId:'C2'` (−0.050 mm), `alternateCandidateId:'C1'` (−0.075 mm), C3 simply recorded with `fitClassification` set but neither flag. The two must never be presented as equal-weight recommendations — the UI's provenance line always names one "Recommended/Preferred" and the other explicitly "Tighter alternate," never a bare list of two options.

## 14. Recording workflow

Entry point: a **"Record Physical Results"** action on a currently-valid Tabletop Corner + Floor Coupon result in Designs (the same generic-handoff-style affordance already established for "Start Project from Design"). Opens a small, bounded dialog (reusing the existing modal system) listing C1/C2/C3 with the controlled-vocabulary fields from §12, machine/material confirmation (defaulting from the active owned machine and, optionally, a Library material picker or free text), and an overall notes field. **Also support a standalone, always-available "Record a Tabletop coupon result" entry point independent of a currently-open matching draft** — since Designs drafts are explicitly session-only, a user returning a day later to log results needs a path that doesn't require regenerating the exact original draft; this path takes the same dialog with all fields editable/manual rather than pre-filled from a live result. This satisfies "manual entry of an externally generated coupon" without adding a new tab or a full history system. The saved result is surfaced via a small **"N saved results"** indicator next to the Tabletop coupon template in Designs (not under Material Tests — wrong shape; not automatically under Library evidence — no promotion has occurred yet).

## 15. Editing and deletion

**Editing** an unpromoted raw record is unrestricted. Editing a record **after** promotion is still allowed (it is, after all, the user's own workshop notes) but **never retroactively alters the already-created Evidence/Production-Setting snapshot** — the promoted entry is frozen at promotion time by the existing, established convention. If the raw record is edited after promotion, the UI should say so plainly ("this result was edited after evidence was created from it; the saved evidence reflects the original values") rather than silently reconciling or silently diverging without comment. **Deletion** of a raw record that has a `promotedEvidence` back-reference is allowed, with an explicit confirmation stating that the Library evidence will remain independently, exactly mirroring the already-established "delete the source record, evidence keeps its own frozen snapshot" pattern used throughout Library/Test-Grid/Material-Test promotion today.

## 16. Evidence stage terminology (locked)

Exactly three tiers, and the current result is locked to the first:

1. **Coupon-proven** — a small joint-fit coupon was cut and evaluated; proves joint fit and assembly behavior for *this specific coupon construction* only. **This is the label the current physical result must carry.**
2. **Shell-proven** — (T2, future) a complete multi-joint shell was assembled using this construction and clearance and observed to assemble correctly at full scale.
3. **Production-proven** — (future, later) a specific setting has been used across enough real production runs to be treated as dependable. **The current result must never be labeled this, and no code path in T1.5 may apply this label.**

Per-candidate fit vocabulary (§12) and the preferred/alternate distinction (§13) are the only other locked terms; "Recommended," "Starting point," and "Confirmed" are used only as UI copy fragments (§18), not as stored status values.

## 17. Promotion contract (confirmed, refined)

**Confirmed policy** (matching and slightly sharpening the task's own suggested candidate): saving a raw physical result **never** automatically creates Evidence or a Production Setting. Promotion is a separate, explicit user action reusing the existing, unmodified promotion-review pipeline (`promotionCandidateBase`/`startPromotion`/explicit field-selection/`savePromotionTransaction`), with the raw record's preferred candidate supplying the promotion candidate's machine snapshot, material context, and `fitSettings.fingerJointClearanceMm` (the one substantive field this promotion actually populates — already an existing field, not a new one). One new `sourceType` tag (`'tabletop-coupon'`) is added to the existing evidence-kind vocabulary, consistent with how `'test-grid'`/`'material-test'`/`'joint-fit-coupon-session'`/`'project'` already coexist as source types. The promoted setting's `verificationStatus` **must default to `'tested'`, and promotion from coupon-only evidence must be blocked from defaulting to `'verified'`** (a validation rule using the existing enum, not a schema change) — verified/production-proven confidence requires shell-level (T2) validation first, matching §16's locked tiering.

## 18. Recommendation ranking

Deterministic, five-tier, never auto-averaging across ties:

1. Exact `machineId` + exact construction identity (§11, including `generatorVersion`) + exact `materialId` + measured thickness within the step tolerance → **Recommended**.
2. Exact `machineId` + exact construction identity + material name/type snapshot (no `materialId`) + thickness within tolerance → **Likely match**, annotated "material not Library-linked."
3. Exact `machineId` + exact construction identity + thickness within the wider (3×-step) band → **Close — verify stock**.
4. Legacy key-based machine match (neither side has an ID) + exact construction identity + thickness match → shown with a **"machine identity not confirmed"** caveat.
5. No match on construction identity, or `generatorVersion` differs → **no recommendation offered**; product form uses its ordinary defaults.

**Multiple equal-tier matches are never auto-selected or blended** — all tied candidates are shown, most-recently-tested first, and the user chooses. Conflicting evidence (e.g., two coupon results for the same construction/machine/material disagreeing on preferred clearance) is always shown side by side, never hidden or silently resolved.

## 19. Explicit Apply and override behavior

The recommendation is **only ever offered, never auto-applied.** A recommendation card (built as a small, generic, reusable piece — see §28) states the recommended value, provenance (machine • material • measured thickness • construction • evidence stage • date tested), and the tighter alternate, with exactly three actions: **Apply recommendation**, **Keep current value**, **View evidence** (opens the underlying Library entry), plus **Run a new coupon** when no match exists. Applying fills an ordinary, still-editable form field — the user can change it immediately afterward with no special "override flag" needed in T1.5 (Project-schema-level override tracking is explicitly deferred to T2, §21). No global/universal default is ever changed by any of this — the recommendation only ever affects the one form field it targets, once, on explicit click.

## 20. Full-shell (T2) follow-up relationship

T2's shell-level result should **coexist as a separate, later, higher-confidence record referencing the same construction identity, machine, and material** — it must not update the T1.5 coupon record in place, and must not delete or supersede-and-remove it. Recommend reusing the *existing* Production-Setting `supersede`/`supersededById` mechanism (already established via `productionSettingsCanSupersede`) for **ranking purposes only**: a shell-proven setting may explicitly supersede a coupon-proven one for the same construction+machine+material combination (extending the existing supersede-eligibility check with a construction-identity comparison, not replacing it), so that future recommendation ranking prefers the shell-proven entry — while the original coupon-proven entry, and the raw T1.5 record behind it, remain permanently in history. This preserves the full evidentiary chain rather than overwriting it.

## 21. Designs integration (bounded)

**In T1.5:** the "Record Physical Results" action and dialog (§14), the "N saved results" indicator, a "View saved results" link, and the pure **matching/lookup helper function** (construction-identity + machine + material + thickness ranking, §18) built and fixture-tested now so T2 does not have to invent it later. **Deferred to T2:** the rendered **recommendation card UI** itself — there is no second product yet to consume it, so building the visual component now would be speculative; building only the underlying lookup function keeps T1.5 bounded while fully unblocking T2. No full Designs-history system is added.

## 22. Material Tests / Library integration

The raw result **never** appears under Material Tests (wrong shape, per §4). It **does** appear under Library evidence/production-setting history, but only after explicit promotion (§17), through the existing, unmodified Library UI. No new "Tabletop joint fit" pseudo-operation identity is introduced into the `operation` vocabulary used by `cutSettings`/promotion eligibility — the promoted setting's `operation` remains the structurally correct `'cut'` (this genuinely is cut geometry), with the fit-specific evidence living in the already-existing `fitSettings.fingerJointClearanceMm` field. Forcing this into speed/power fields would be misleading and is explicitly not done; those fields remain blank/optional exactly as their existing shape already allows.

## 23. Project integration

**No Project schema change in T1.5.** There is no new product to hand off to a Project yet beyond the coupon's own existing, already-shipped generic Design-to-Project handoff (which already names the coupon and records candidate clearances with a construction caveat, per the T1 implementation report). Deferred to T2: whether a future rectangular tray's Project handoff should snapshot the selected evidence ID, clearance, evidence stage, and override status — a legitimate question, but one that only makes sense once a real product exists to ask it about.

## 24. Import/export/backup compatibility

Behaves exactly like every other additive root array already in this app: full JSON export includes it wholesale; import normalizes malformed/missing entries to `[]` via the standard `filter(item => item && typeof item === 'object')` idiom; Merge unions by id (local wins on conflict, matching the established convention); Replace fully restores from the backup; storage recovery is unaffected (this is just one more field inside the same recoverable object); an older app version importing a newer backup silently ignores the unrecognized field with no failure; missing machine/material references degrade to their frozen display snapshot, never dropped; duplicate IDs use the same id-keyed merge rule as everywhere else; a deleted source *Design* session has no bearing on an already-saved raw record (it doesn't reference a live Design at all — see §14, manual-entry path); the data contains no more sensitive information than existing free-text Project/Test notes and requires no special handling under a future encrypted-backup wrapper, since it will simply ride inside whatever wraps the whole backup object when that roadmap item is eventually built (not built here).

## 25. Validation contract

| Condition | Behavior |
|---|---|
| No preferred result selected | **Block save** |
| Preferred candidate not in generated set | **Block** (integrity check) |
| Preferred and alternate identical | **Block** |
| Missing/non-finite material thickness or clearance | **Block** |
| Machine missing or archived | **Allow**, annotate; never blocks raw save |
| Material missing/deleted (dangling `materialId`) | **Allow**, display via frozen snapshot |
| Measured-thickness mismatch vs. a linked Library profile's own recorded thickness | **Warning only** |
| Unsupported/unrecognized construction identity or stale `generatorVersion` | **Allow**, recorded and shown, excluded from ranking |
| Duplicate/double-click save | **Guard client-side** (disable Save after first click), same idiom used elsewhere |
| Conflicting evidence | **Never hidden**; always shown, never auto-resolved |
| Editing a promoted result | **Allowed**; does not retroactively alter the frozen promoted snapshot (§15) |
| Deleting a result with downstream promoted evidence | **Allowed with explicit confirmation**; evidence persists independently |

## 26. Fixture organization

Three small, focused, standalone groups — following the exact pattern already established by `runGiftBoxFixtures`/`runTrayModelFixtures`/`runDiceTraySystemFixtures` — not one monolithic group:

- **`runTabletopCouponResultFixtures()`** — save path, candidate structure, preferred/alternate rules and distinctness, machine/material snapshotting (both Library-linked and free-text paths), non-blocking behavior for missing machine/material, edit-preserves-raw-record, delete-with-confirmation, import/export/merge/replace round trip, malformed-entry rejection, no `SCHEMA_VERSION`/`STORAGE_KEY`/`BACKUP_FORMAT` change, cleanup.
- **`runTabletopEvidencePromotionFixtures()`** — explicit-promotion-only (no auto-promotion on save), correct `sourceType`/`fitSettings.fingerJointClearanceMm` population, default status `'tested'` never `'verified'`, construction-identity carried into the promoted entry, supersede-eligibility extension, raw-record-to-evidence back-reference, deletion independence.
- **`runTabletopEvidenceMatchingFixtures()`** — the full ranking-tier truth table (exact match; Library-linked vs. free-text material; legacy-key fallback reused from M3; thickness-tolerance boundary; different-construction non-match; different-`generatorVersion` non-match; multiple-tied-matches never auto-selected).

Each new group re-invokes its nearest existing regression dependency (`runMultiMachineStorageFixtures`, `runEvidencePromotionFixtures`, the existing Tabletop coupon fixture group) as a guard at its end, exactly matching the established "small group + regression call-through" convention. Each is registered once under its own `?selftest=` value.

## 27. Protected boundaries

Confirmed by design (no code was written, but every recommendation above is purely additive): `APP_ID`/`APP_NAME`/`APP_VERSION`/`BUILD_DATE`/`STORAGE_KEY`/`BACKUP_FORMAT`/`SCHEMA_VERSION` all unchanged; localStorage recovery, legacy imports, legacy plaintext backups untouched; M1/M2/M3 machine systems reused verbatim, not modified; material identity, Material Tests, Library, promotion history, Project schema, accounting, Inventory, Pricing untouched; Designs production SVG and the current Tabletop coupon SVG bytes untouched (T1.5 only reads the coupon's already-computed `result.metrics`/`result.finishedView` after generation; it does not touch `buildTabletopCornerFloorCouponDesignResult`); current fixture totals and goldens unchanged except for the addition of new, additional, passing assertions; offline/no-network behavior unaffected.

## 28. First bounded Codex phase

The task's own suggested MVP list is correctly bounded, with one narrowing recommended: **defer the rendered recommendation-card UI to T2** (build only the pure lookup/ranking helper function now, fixture-tested, with no visual consumer yet). Confirmed MVP scope:
- "Record Physical Results" action from a valid Tabletop coupon result, plus a standalone manual-entry entry point.
- Small result dialog: three candidates, controlled-vocabulary fit fields, preferred/tighter-alternate selection, machine/material/thickness confirmation, notes.
- Save to the new `tabletopCouponResults` array; view/edit/delete saved results.
- Explicit "Promote to Library evidence" action reusing the existing promotion pipeline, tagged `sourceType:'tabletop-coupon'`, populating `fitSettings.fingerJointClearanceMm`, defaulting `verificationStatus:'tested'`.
- The matching/ranking lookup helper function (no rendered card).
- Three focused fixture groups (§26).
- README/CHANGELOG/Help wording describing the new recording and promotion workflow honestly, including the coupon-proven-not-production-proven caveat.

## 29. Locked-decision table

| Decision | Locked value |
|---|---|
| Record type (raw stage) | New small optional record, in a new root array (not Material Test, not Evidence, not Production Setting) |
| Storage location | Same `genmitsu-l8-tracker-v1` object; one new additive array field (`tabletopCouponResults`) |
| Schema/version behavior | No `SCHEMA_VERSION` change; purely additive; no migration; no backfill |
| Source-of-truth record | Raw record = "what I observed"; promoted Production Setting/Evidence = "what Library treats as evidence-backed" |
| Promotion behavior | Never automatic; explicit user action via the existing, unmodified promotion pipeline |
| Evidence stage terminology | Coupon-proven → Shell-proven (T2) → Production-proven (future); current result = Coupon-proven only |
| Machine identity rule | Reuse `productionMachineIdentityMatches` unchanged (exact-ID-if-either-has-one; legacy key fallback; no backfill) |
| Material identity rule | Optional Library `materialId` or free-text name; never invented |
| Measured-thickness matching rule | Exact within the coupon's own clearance-step tolerance (default 0.05 mm); wider 3×-step band shown as "verify"; beyond that, not ranked |
| Construction identity | `{family, constructionTemplate, generatorVersion, wallCornerJoint, floorJoint, materialThicknessMm, actualFingerWidthMm}`; exact-combination match only; version-sensitive |
| Candidate observation fields | `{clearanceMm, fitClassification (5-value enum), selfSupport, assemblyEffort (freeform), notes}` |
| Preferred-result rule | Exactly one required to save |
| Alternate-result rule | Optional tighter alternate only; must differ from preferred; no third slot |
| Recommendation ranking | 5-tier deterministic; ties shown, never auto-selected or blended |
| Apply behavior | Explicit "Apply recommendation" button only; never auto-prefilled |
| Override behavior | Applying fills an editable field; no new override flag in T1.5 |
| Edit behavior | Allowed anytime; never retroactively alters an already-promoted snapshot |
| Delete behavior | Allowed with confirmation; promoted evidence persists independently |
| Import/export behavior | Standard additive-array normalize/merge/replace; no new validation gate |
| Recovery behavior | Unaffected; unavailable machine/material references degrade to frozen snapshot |
| Project integration | No schema change in T1.5; deferred to T2 |
| T2 follow-up relationship | Separate, higher-confidence coexisting record; may `supersede` (existing mechanism, construction-aware) for ranking; history preserved |
| Fixture groups | Three small standalone groups (§26), each guarding its nearest existing dependency |
| First Codex implementation scope | Recording + view/edit/delete + explicit promotion + matching helper (no card UI) + fixtures + docs |
| Deferred features | Recommendation card UI, T2 shell recording, Project-schema override tracking, production-proven promotion |
| Non-goals confirmed | No T2 product, no hex UI, no schema migration, no legacy backfill, no new tab, no network/cloud, no encryption, no SVG geometry change |

## 30. Findings by severity

**BLOCKER TO IMPLEMENTATION:** none. Storage location, evidence ownership, and promotion direction are all unambiguous and resolved above.

**IMPORTANT:** none outstanding — every identity/matching/promotion/compatibility question the task raised is resolved with a single locked answer in §29, grounded in fields the T1 engine already produces (`generatorVersion`, `wallJoin.type`, `floorJoin.type`, `actualFingerWidths`) rather than invented ones.

**POLISH (safe to decide later without changing the architecture):** whether the Library material picker is shown before or after the candidate-observation fields in the recording dialog; exact wording of the five-value fit-classification enum; whether the "N saved results" indicator shows a count or a simple link.

**NOT A DEFECT (intentional bounded T1.5 limits):** no recommendation-card UI yet (nothing to recommend to); no Project-schema override tracking (no product form yet to override); no automatic promotion; no production-proven status reachable from this phase.

## 31. Exact final verdict

**READY FOR CODEX T1.5 IMPLEMENTATION**

## 32. Remaining user decision, if any

**None blocking.** One soft, non-blocking default is recommended and may be confirmed or changed by Joe without altering the architecture: whether the recording dialog *requires* a Library material pick before saving, or allows free-text material entry with no Library linkage at all. This review's recommended default is **support both, Library-linked preferred but never required** — consistent with how "material" already behaves everywhere else in the app.

## 33. Whether Claude is needed again

**Yes, once, after implementation** — a focused implementation audit (matching the established pattern for every prior T1/M1-M3 phase in this codebase) should verify the locked decisions in §29 were followed exactly, particularly: no auto-promotion, correct `verificationStatus` default, construction-identity version-sensitivity, and that no existing constant/record/SVG byte was touched.

## 34. Whether a Grok audit should follow implementation

**Yes**, as the standard second-pass adversarial check on the implementation, consistent with this project's established audit cadence for schema-adjacent-but-additive phases — provided the Claude focused audit in §33 finds no unresolved identity/promotion ambiguity first.

## 35. Confirmation

No product source file, README, CHANGELOG, fixture, storage, or application record was edited during this review. Nothing was staged, committed, or pushed. Only this report file was written.
