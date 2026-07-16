# Phase 7.1 Proven Production Settings — Second-Pass Review (Independent Check of Grok's Audit)

**Reviews:** `docs/PHASE7_1_PRODUCTION_SETTINGS_INDEPENDENT_AUDIT_2026-07-16.md` (Grok's audit — already present in the working tree before this pass; not overwritten)
**Also read for context:** `docs/PHASE7_MATERIAL_PROFILES_ARCHITECTURE_REVIEW_2026-07-16.md`, `docs/PHASE7_1_PRODUCTION_SETTINGS_IMPLEMENTATION_2026-07-16.md`
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `c93ebd1` — *Add production joint-fit tuning*
**Reviewed:** Uncommitted changes in `index.html` (514/16), `README.md` (7/7)
**Date:** 2026-07-16
**Note on filenames:** the task asked me to save this audit to the same path as Grok's existing file (`...INDEPENDENT_AUDIT_2026-07-16.md`). Consistent with every prior review in this chain (and with the Joint Fit Coupon turn immediately before this one, where the same "save to the reviewed file's own name" instruction appeared and was treated as a template artifact), I did not overwrite Grok's audit. This report is saved separately.
**Mode:** Read-only, independent re-derivation. I built the entire independent audit below **before** reading Grok's file, by reading the actual function bodies in `index.html` directly and hand-tracing scenarios equivalent to every one the task requests, then cross-checked the result against Grok's report afterward. I also attempted to get live execution evidence via the Browser tool available in this session (`file://` navigation to the local `index.html`), which none of my prior reviews in this chain had access to — this was blocked by the browser pane's sandbox (file: navigation denied), so this pass remains static-analysis-based like the others, but the attempt is disclosed rather than silently skipped.

---

## Verdict on Grok's audit

**Materially accurate, with one disputed finding and one omission worth adding.** Grok's core safety conclusions (no Blocker, no Major, SAFE TO COMMIT AFTER MINOR CLEANUP) match my independently-derived conclusion exactly, and the great majority of Grok's per-area "Verified" rows match findings I reached independently by reading the same functions myself before reading Grok's report. Three of Grok's four substantive Minor findings (unclamped power/thickness bounds, narrow preferred-conflict key, unrestricted supersede-target picker) are confirmed. The fourth (evidence `sourceProfileId` not remapped on duplicate) I disagree with — see below. I also found one concrete mechanism Grok's audit didn't specifically name: silent Inventory-link clearing on resave after the linked item is deleted. Fixture totals (47 production-settings assertions, 1138 grand total, 559 Designs) all recompute exactly, independently, matching Grok's numbers.

---

## Repository inspection (re-run independently)

| Check | Result |
|---|---|
| HEAD | `c93ebd1`, matches |
| Staged files | None |
| Tracked changes | `README.md` (7/7), `index.html` (514/16) |
| `git diff --check` | Passed (CRLF warnings only) |
| Committed `c93ebd1:index.html` contains `productionSettings`? | Confirmed **no** — this feature is entirely in the uncommitted working tree, matching Grok's "critical scope note" |
| Shared/legacy functions touched by this diff? | `git diff \| grep` for every pre-existing `run*Fixtures` function name plus `decodeStoredState`, `buildBoxModel`, `buildSlidingLidDesignResult`, `buildDrawerCabinetModel`, `buildJointCouponModel`, `normalizeDesignDraft`, `buildDesignResult`, `testIsCompatibleWithOperation`, `libraryProfileTestSummary`, `normalizeMaterialTests`, `del(` → **zero hits on all of them** |
| `productionSettings` referenced anywhere in Designs code? | **Zero occurrences** outside Library-editor and fixture code — independently confirms Grok's §18 (Phase 7.2 not implemented) rather than trusting the report |
| Live browser execution attempted? | Yes — `mcp__Claude_Browser__navigate` to `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all` was **denied by the browser pane's sandbox** ("navigation to https://file was denied or failed"). Grok's claimed Edge-headless/Playwright execution could not be reproduced or independently confirmed by any means available in this environment, same limitation as every prior review in this chain. |

---

## Independently confirmed, by area (derived before reading Grok's file)

### 1–2. Profile-edit preservation and normalization purity

`buildLibraryProfileRecord(previous, recognized, tests, productionSettings)` (`index.html:7007`) is `{ ...(previous||{}), ...(recognized||{}), tests: normalizeMaterialTests(tests), productionSettings: normalizeProductionSettings(productionSettings) }`, exactly the pattern the architecture review prescribed. The caller (`openProfile`'s submit handler, `index.html:7040`) builds `previous` as `{ ...(profile||{}), ...(current||{}) }`, re-fetching the live persisted object (`current = state.profiles.find(...)`) at submit time so a stale modal-open-time snapshot can never win over live data. `tests`/`productionSettings` come from session-local draft arrays (`profileDraftTests`/`profileDraftProductionSettings`) untouched by ordinary material-field edits. `saveProfileRecord` (`index.html:7015`) is a standard upsert-with-rollback: on `persistFn()` failure it restores the exact previous list entry or removes the just-added one — hand-traced against a `persistFn = () => false` input and confirmed `JSON.stringify` equality before/after. This matches Grok's 1.1/1.2/1.4 exactly.

`normalizeProductionSetting` (`index.html:4615`) spreads the source object, and each nested subobject, before overwriting only recognized keys — this is what makes unknown top-level and nested fields survive. Confirmed non-mutation by hand-tracing `JSON.stringify(source)` equality before/after normalization. Matches Grok's 2.1/2.3.

### 3. Strict optional numeric handling

`strictOptionalNumber` (`index.html:4583`) correctly keeps blank distinct from zero (`''`/`null`/`undefined` → `''`; `0` stays `0`), preserves negative values (`-.02` → `-0.02`), and rejects malformed/`NaN`/`Infinity`/hex-like input as `''` rather than coercing to zero. Hand-traced `lightBurn.kerfOffsetMm: 0` staying `0` (not `''`) — the specific case the architecture review called out as the parser's whole reason to exist, since a generic `num()`-style fallback elsewhere in this codebase would make blank and zero indistinguishable.

**Confirms Grok's 3.2 (exponent notation) as a fact, disputes the framing.** `strictOptionalNumber`'s regex (`/^[+-]?(?:\d+(?:\.\d*)?|\.\d+)(?:e[+-]?\d+)?$/i`) does accept `"1e-2"` → `0.01`, exactly as Grok found. I read this differently than "should tighten": `strictOptionalNumber` is a general free-typed numeric-field parser (used for speed, thickness, kerf, clearance — fields where a user might reasonably type or paste any valid JS-style number), deliberately different from `parseJointCouponClearances`'s fixed-decimal-only grammar (which serves a constrained clearance-*list* text field where exponent notation would be genuinely out of place). Accepting standard numeric notation in an open numeric input isn't itself a defect; I'd downgrade this from "should reject" to "worth a fixture documenting the accepted grammar," but the underlying fact Grok reports is accurate.

**Confirms Grok's 3.3 exactly.** `powerMinPercent`/`powerMaxPercent` and both thickness fields receive no range clamp in `normalizeProductionSetting` — only `strictOptionalNumber`'s finite-number check — unlike `confidence` (`[1,5]`) and `passes` (`≥1`), which are explicitly clamped. This mirrors an existing app-wide pattern (`normalizeMaterialTests`'s `winningPower` is likewise unclamped), so it isn't a new regression, but it's a real gap in a feature positioned as "proven, trusted" data. Not caught by any fixture, confirmed by both of us independently.

### 4. Nested merge-import correctness — the critical area

I hand-traced `mergeProductionSettingRecord` (`index.html:8166`) with paper inputs before reading Grok's report: it merges `machine`/`materialCondition`/`cutSettings`/`lightBurn`/`fitSettings` **individually** (`{...(existing.X||{}), ...(incoming.X||{})}` per subobject), not as one whole-record spread — this is exactly the shallow-spread trap the task calls "critical," and the actual code avoids it by construction. `evidence` merges by ID via `mergeNestedRecordsById`. `mergeLibraryProfileRecord`/`mergeLibraryProfiles`/`mergeData()` route Library merges through this nested-ID-aware path specifically (not the generic `mergeList` used for entries/grids/projects), confirmed by direct reading of `mergeData()` at `index.html:8257-8268`.

Grok's 4.1 claims to have independently probed this exact scenario — divergent unknown fields injected into `machine`/`cutSettings` on both local and incoming sides for a matching setting ID — via an adversarial script executed outside the shipped fixture suite, and reports both sides' unique fields surviving. **I could not reproduce or verify this specific claim** (no browser/JS execution available to me in this environment, confirmed by the failed `file://` navigation attempt above). What I can independently confirm is: (a) the shipped `runProductionSettingsFixtures()` fixture does **not** exercise this scenario — its `local`/`incoming` `shared-setting` records share the identical `machine`/`materialCondition`/`cutSettings`/`lightBurn`/`fitSettings` object structurally (both spread from the same base `setting` fixture; only top-level scalar fields and the `evidence` array actually diverge), and (b) my own hand-trace of the merge function against constructed inputs reaches the same "correct, no defect" conclusion Grok's probe claims. Two independent methods (Grok's claimed live probe, my static hand-trace) agree on the answer; only the *automated regression fixture suite* has a real, reproducible gap here — worth closing regardless of whether Grok's manual probe genuinely ran, since a probe that isn't part of `runProductionSettingsFixtures()` won't catch a future regression.

### 5–6. Replace-import and backup round-trip

`replaceData()` (`index.html:8247`) does whole-collection replacement with normalization; `normalizeProductionSetting` returns `null` only for a non-object entry (a malformed-but-object entry still normalizes to a safe-defaulted record rather than being dropped) — hand-traced `productionSettings: [setting, null]` → `null` dropped, `setting` kept, parent profile intact. `backupObject()` (`index.html:493`) includes `profiles: state.profiles` unchanged, no new top-level key. `STORAGE_KEY`/`SCHEMA_VERSION` (`index.html:256-257`) unchanged. `loadState()` normalizes via `normalizeLibraryProfiles`, so old backups load `productionSettings: []`. Matches Grok's 5.1/5.2/6.1/6.2/6.3 exactly.

### 7–8. Preferred and supersede behavior

`productionSettingConflict` (`index.html:5067`) keys on `operation === setting.operation && machine.key === setting.machine.key` only — no `measurementScope`/batch dimension. **Confirms Grok's 7.2 exactly**: two distinct sheet/batch records under the same machine+operation will contend for one preferred slot. This matches the architecture review's own explicitly-documented Phase 7.1 boundary, so it's implemented as scoped, not a coding defect — but nothing in the UI warns the user about this narrowness before the demotion prompt fires.

`upsertProductionSettingDraft` (`index.html:5070`): decline leaves `next` completely unmutated and the modal open (no `closeModal` call on a `null` return), confirmed by tracing the caller. Accept touches only the conflicting record's `preferred`/`updatedDate`; other machines' records are left as the same object reference. `supersedeProductionSettingDraft` (`index.html:5084`) rejects self-supersede and nonexistent targets before any mutation; on success it spreads the *existing* record first (preserving its evidence) and never writes to the replacement. Matches Grok's 7.1/8.1/8.2.

**Confirms Grok's 8.3 exactly and independently.** `openSupersedeProductionSetting` (`index.html:5195`) builds its choice list from `profileDraftProductionSettings` filtered only by `id !== oldId && verificationStatus !== 'superseded'` — no machine/operation filter, so a 20W record can be offered (and accepted) as the replacement for a 40W record with no warning. Not destructive (the old record survives, re-editable), but worth a targeted fix.

### 9. Delete isolation

Evidence/setting/test delete handlers each filter their own draft array by `.id !== deletedId` and refresh only their own section — confirmed no cross-array mutation by direct reading. Evidence displays `sourceLabel`/`sourceId` as plain stored strings (`productionSettingSummaryHtml`, `index.html:5044`), never a live lookup, so deleting a Material Test or Project a piece of evidence references leaves the snapshot fully readable with no dangling-reference crash. Generic profile delete (`del()`, `index.html:8328`, pre-existing and unmodified) is a plain single-collection splice-by-id; since nested records live inside the profile object, deleting the profile removes them as an intrinsic consequence, not via a separate cascade path that could reach another profile. Matches Grok's 9.1/9.2.

### 10. Duplicate-profile behavior

`duplicateProductionSettings` (`index.html:4725`) generates new setting and evidence IDs, individually re-spreads every nested subobject (so the copy shares no object references with the source — hand-verified via a constructed mutation-independence trace), clears `preferred`/`supersededById`, downgrades `verificationStatus` to `'estimated'` **regardless of source status** (even `verified` → `estimated`), and appends a reverify note without discarding existing notes. Matches Grok's 10.1 exactly.

**I disagree with Grok's 10.2.** Grok flags evidence `sourceProfileId` surviving duplication unchanged as "misleading provenance." I read the field differently: `sourceProfileId` is a manually-typed, optional free-text field (`field('sourceProfileId', 'Source profile ID', ..., 'Optional Library profile ID retained for provenance.')`, `index.html:5126`) — it is **not** auto-populated to self-reference the setting's own containing profile anywhere in the code I read (`openProductionEvidence` initializes fresh evidence with no `sourceProfileId` at all). Its documented purpose is to record *which profile the evidence itself originated from* (e.g., a joint-coupon test logged under a related-but-different profile), which is independent of which profile currently *contains* the production-setting record. The architecture review's explicit duplicate-profile rule is "preserve values and source snapshots" (§12) — I read `sourceProfileId` as exactly this kind of snapshot, on the same footing as `sourceId`/`sourceLabel` (which Grok's own 10.1 credits as correctly preserved). Grok's 10.1 and 10.2 are, read together, inconsistent: the same "snapshot preserved unchanged" behavior is praised for `sourceId`/`sourceLabel` and flagged as a defect for `sourceProfileId`. I'd downgrade 10.2 from "Minor defect, needs remap/clear" to "not confirmed as a defect under the current (manual, free-text) implementation" — worth revisiting only if a future phase starts auto-populating `sourceProfileId` to self-reference the containing profile, at which point stale self-references on duplication would become a real concern.

### 11–14. Unknown-field preservation, unit stability, UI binding, Library summary semantics

All independently traced and matching Grok's "Verified" rows: unknown fields survive at every level through load/normalize/profile-edit-overlay/merge/backup/duplicate, dropping only on a genuine same-key merge conflict (incoming wins, as documented) or explicit user deletion. `convertStoredUnits` (`index.html:548-557`) never references `productionSettings` in its body at all — confirmed by full read, not by trusting the "byte-stable" claim. Evidence editing is a structurally separate `<form>`/modal from the production-setting form (not a nested `<form>`, which HTML doesn't permit), so there's no possibility of the evidence form's submit reaching the parent form's handler. `libraryProfileStatusLabel`/`libraryProfileTestSummary` (pre-existing, unmodified — confirmed via diff-touch check) reference only `tags`/`notes`/`tests`, never `productionSettings` — Best Known semantics are untouched.

### 15. Inventory references — one addition beyond Grok's audit

Grok's 15.2 states the general fact ("no live re-link; snapshots informational by design... documented behavior") but doesn't name a specific mechanism. I traced one concretely: `productionInventoryOptions` (`index.html:5137`) builds its `<select>` from the **live** `rawInventoryMaterials()` list on every render. If a production setting's linked item is **deleted**, reopening that setting's editor produces a `<select>` with no matching `<option>` for the stored `inventoryItemId` — the browser defaults the control to its first option (`"No Inventory link"`, value `""`). If the user then saves without touching that specific dropdown, `f.inventoryItemId` submits as `""`, **silently clearing the link** on an otherwise-unrelated edit. The `inventoryItemName` text snapshot is a separate field and survives (so the record stays informationally readable — not the data-loss scenario the architecture review worried about), but the ID linkage disappears without explicit user action or warning. Worth surfacing as its own line item since "the ID silently clears on next save" is a more specific and more actionable fact than "no live re-link."

### 16–18. Regression safety, fixture quality, scope confirmation

Confirmed via the diff-touch check that none of the ten pre-existing fixture-group functions, Designs generator functions, `decodeStoredState`, `del()`, or `libraryProfileTestSummary`/`normalizeMaterialTests` were modified by this diff — combined with zero `productionSettings` references anywhere in Project Wizard/Material Browser/Test Grid/Designs/Inventory-matching code, this independently corroborates Grok's §16 and §18 conclusions.

**Fixture counts, recomputed independently with a bracket/loop-aware script (not by trusting either report's table):**
- `runProductionSettingsFixtures()` (`index.html:4778-4905`, flat array, no loops): **47** `fixture(` call sites — exact match to both reports.
- `runDesignGeometryFixtures()` (`index.html:2746-3392`): 467 total static `add(` occurrences; six loop blocks unchanged in shape from my prior Joint Fit Coupon pass (baselines ×5 lines/4 iters, mating-pairs/goldenA/goldenB ×1 line/8,21,21 iters, glyphExpectations ×3 lines/6 iters, optionCombinations ×5 lines/4 iters ⇒ 16 static loop lines, 108 runtime-generated); non-loop static `467−16=451`; runtime total `451+108=559` — exact match to both reports' 559.
- The other ten groups (`20+12+23+67+57+56+61+216+12+8=532`) I did not individually re-derive line-by-line this pass (that would mean re-auditing Material/Library/Project/Grid Browser fixtures already covered in five separate earlier reviews in this chain); I instead confirmed via `git diff` that all ten of those fixture-group functions are byte-untouched by the current diff, so whatever they summed to at the `c93ebd1` baseline (self-consistently, `1091 − 559 = 532`, matching "the prior 1091 assertions remain green") still holds.
- **Grand total: `532+47+559=1138`** — exact match to both reports, arrived at independently.

---

## Findings

| # | Severity | Source | Scenario | Consequence | Recommendation | Fixture detects? |
|---|---|---|---|---|---|---|
| F1 | Minor | Confirms Grok 7.2 | `productionSettingConflict` keys on profile+operation+machine only, not measurement scope/batch | Distinct-sheet/batch records under one machine+operation contend for a single preferred slot; no data loss, just an unexpected demotion prompt | Name the conflicting record's scope/batch in the demotion-confirmation prompt text so the user recognizes a cross-batch conflict before confirming | No |
| F2 | Minor | Confirms Grok 8.3 | `openSupersedeProductionSetting`'s replacement list isn't filtered by machine/operation | A 20W record can supersede a 40W record (or vice versa) with no warning; reversible, not destructive | Filter choices to matching `operation`/`machine.key` by default, or badge mismatched choices | No |
| F3 | Minor | Confirms Grok 3.3 | `powerMinPercent`/`powerMaxPercent`/thickness fields have no range clamp in `normalizeProductionSetting` (unlike `confidence`/`passes`) | A fat-fingered value (e.g. `1500`%, negative thickness) is stored and surfaced as "proven" data with no safety net | Clamp power to `[0,100]` and thickness to `≥0` in the normalizer | No |
| F4 | Minor | New (not in Grok's audit) | `productionInventoryOptions`/`productionSettingFromForm`: after a linked Inventory item is deleted, reopening+resaving the setting (without touching the Inventory dropdown) silently submits `inventoryItemId: ""`, clearing the link | `inventoryItemName` text snapshot survives (no readable-data loss), but the ID vanishes without explicit action or warning | Detect the dangling-ID case and render an explicit "linked item no longer exists (was: <name>)" notice instead of silently defaulting the select | No |
| F5 | Minor (fixture gap, not a code defect) | Sharpens Grok 4.1 | No fixture in `runProductionSettingsFixtures()` constructs a matching setting ID with genuinely divergent *nested subobjects* (as opposed to top-level/evidence-array divergence) between local and incoming | The per-subobject shallow-merge correctness of `mergeProductionSettingRecord` — the single most-critical behavior this task asked me to probe — is proven only by direct code reading (mine and, if genuinely executed, Grok's separate adversarial probe), not by the automated regression suite | Add a fixture with `local.materialCondition={supplier:'Local'}` / `incoming.materialCondition={product:'Incoming'}` on the same setting ID and assert both survive post-merge | Partially (would catch total replacement, not partial) |
| — | Disputed | Grok's 10.2 | Duplicated evidence retains original `sourceProfileId` | Grok calls this "misleading provenance"; I read `sourceProfileId` as a manually-typed provenance snapshot (never auto-set to self-reference the containing profile anywhere in the code), on the same footing as `sourceId`/`sourceLabel`, which Grok's own 10.1 credits as *correctly* preserved | No action recommended under the current manual-field implementation; revisit only if a future phase auto-populates `sourceProfileId` to self-reference the containing profile | — |
| — | Downgraded framing | Grok's 3.2 | Exponent notation accepted by `strictOptionalNumber` | Confirmed as fact; read as a deliberate difference from the constrained Joint-Coupon parser rather than a defect in a general free-typed numeric field | Optional: add a fixture documenting the accepted grammar rather than tightening the regex | — |

No Blocker or Major findings, matching Grok's conclusion.

---

## Unverified areas

Live fixture execution and Grok's claimed Edge-headless/Playwright adversarial probes could not be reproduced — I specifically attempted browser-based verification this pass (`file://` navigation), which was denied by the sandbox, so this remains a genuine, disclosed gap rather than an untried one. Fixture totals were instead independently recomputed via static/loop-aware source analysis, matching both reports exactly. Physical verification is out of scope for this feature (it stores manually-entered production claims; nothing here performs or validates an actual cut).

---

## Required conclusion

```text
SAFE TO COMMIT AFTER MINOR CLEANUP
```

This matches Grok's conclusion. Every "Verified" claim I checked independently before reading Grok's report reached the same result through direct source-reading and hand-traced scenarios; three of Grok's four Minor findings are confirmed as-is; one (evidence `sourceProfileId` on duplicate) is disputed with reasoning above; one additional concrete Minor finding (silent Inventory-link clearing on resave) is added; and the automated-fixture-suite gap around nested-subobject merge conflicts is sharpened from "not covered" to "the single highest-value fixture to add before this feature sees heavier use." None of this changes the bottom line: no credible path to data loss, unstable IDs, or corrupted merge/import behavior was found by either independent pass.

---

*Second-pass review performed read-only at commit `c93ebd1` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. Grok's existing audit file was read but not overwritten. Unrelated untracked files were left untouched.*
