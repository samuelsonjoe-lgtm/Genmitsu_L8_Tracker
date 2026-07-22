# T3B-E1 Complete-Tray Evidence — Post-Audit Correction Verification

**Date:** 2026-07-22
**Verifier:** Claude Sonnet 5 (independent, read-only — nothing implemented, nothing corrected)
**Scope:** narrow verification of the three bounded post-audit corrections only. The full architecture is not re-audited here.

## 1. Actual repository state

`git rev-parse HEAD` = `c16324b6e2827e42692889c99431ac2956b6f1bd` — **"Add T3B tabletop storage tray"** (confirmed). Branch `main`; `origin/main...main` = `0 0` (synchronized). Nothing staged. Tracked diff: `CHANGELOG.md` (+3/−1), `README.md` (+8/…), `index.html` (+137/−35 net across 161 changed lines — larger than the pre-correction diff, reflecting the three corrections). The implementation report's "Post-audit corrections — 2026-07-22" section documents the same three changes verified below. Untracked set (`LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, all prior reports) preserved — confirmed unchanged before/after this verification (`git diff --stat` identical).

## 2. Exact correction diff inspected

- **Correction 1:** `tabletopTrayLiveComponentMatches` (index.html:787) is now exactly `{shell:tabletopStorageTrayShellEvidenceMatches(design,profiles,raw.machine),storage:tabletopStorageEvidenceMatches(design,profiles,raw.machine)}` — the prior raw-record-existence filter (`state.tabletopShellResults.find(...)` / `state.tabletopStorageResults.find(...)`) is gone. It relies solely on the existing, unmodified exact Library-evidence matchers.
- **Correction 2:** new `tabletopTrayObservationLabels` map (index.html:442–446, 29 entries) and `tabletopTrayObservationLabel(key)` lookup (index.html:447). Used in `tabletopTrayResultEligibility`'s blocker messages and in the recorder's `bool(key, tabletopTrayObservationLabel(key))` calls (index.html:788, 5522).
- **Correction 3:** `openTabletopTrayResultRecorder` (index.html:5516) now computes `readOnly=!!existing?.promotedEvidence` and threads a `disabled` attribute onto every form control, omits the Save button and the `onsubmit` handler entirely when read-only, renders a "Promoted evidence and frozen component provenance" panel, and changes the heading/Cancel-vs-Close text. `openTabletopTrayResultsList`'s `[data-tray-edit]` handler (index.html:5527) no longer has the `!record.promotedEvidence` guard that previously made "View" a silent no-op.

## 3. Component-gate results (Correction 1)

**Source-confirmed and live-confirmed.** Built real promoted T2 Shell-proven and T3A Storage Fit Coupon-proven evidence via the actual UI (material profile, machine, T2 generation+recording+promotion, T3A generation+recording+promotion — all real clicks/form submits), then **deleted both raw source results** via the real Delete buttons (confirmation text: *"Delete raw shell result? Library Evidence and the Production Setting remain independent."* / *"Delete this raw result? Its Library evidence remains independent."*). Navigated to T3B: the evidence card still read **"Outer shell: T2 Shell-proven — Exact"** and **"Storage mechanism: Storage Fit Coupon-proven — Exact"** (expected, since these matchers never depended on raw records). Recorded a full qualifying T3B pass with blank optional measurements → **eligible for promotion (confirmed true) despite both raw sources being deleted** — this is the core fix, live-verified, not merely inferred.

Fixture-level verification (56/0 → now part of 66/0) independently confirms the remaining component-gate requirements with genuinely non-tautological test data (inspected the actual construction, not just the assertion names):
- `withoutShellEvidence`/`withoutStorageEvidence`: clone profiles and empty the relevant `evidence` array → **"missing or revoked T2/T3A promoted evidence blocks"** (both pass).
- `staleShellEvidence` (corrupts `clearanceMm` to `-0.05`) / `staleStorageEvidence` (corrupts `railHeightMm` to `11`) → **"stale T2/T3A promoted evidence cannot substitute for an exact match"** (both pass) — these are real configuration-mismatch constructions, not copies of the passing profile.
- **"exact T2/T3A evidence remains eligible after its raw source is deleted"** and **"both deleted component raw sources remain eligible with exact promoted evidence"** — all pass with genuinely reconstructed state (raw shell/storage results explicitly emptied from `state.tabletopShellResults`/`state.tabletopStorageResults` inside the fixture, matching the live test).
- **"promotion after component raw deletion preserves complete frozen provenance"** — passes; live-verified separately: after promoting with both raw sources deleted, the results list showed *"Supporting provenance: frozen T2 + T3A snapshots retained"*, and the recorder's read-only provenance panel displayed populated T2/T3A evidence/setting/source-result/frozen-stage fields (not "Not recorded").
- No deleted raw result is recreated or mutated — confirmed by reading `finalizeTabletopTrayPromotion` (unchanged from the prior audit: it only writes to the tray record and the new Library evidence, never to `tabletopShellResults`/`tabletopStorageResults`) and by the live test showing both lists still read empty after promotion.
- T2/T3A matchers and evidence meanings unweakened — confirmed absent from the diff (`tabletopRectangularShellEvidenceMatches`, `tabletopStorageExactMatch`, `tabletopStorageEvidenceMatches` unchanged).

**This directly and fully resolves the Medium finding from the prior independent audit.**

## 4. Label coverage results (Correction 2)

`tabletopTrayObservationLabels` contains exactly the 9 dry-fit + 10 post-cure required-positive keys and the 4+6 required-negative keys — **29 entries total**, verified both by direct reading and by the fixture's own assertion (`tabletopTrayObservationValues.length===29 && every key has a non-empty label !== the key itself`, passing). Live UI: opened the recorder and read the rendered label text for all 29 controls — e.g. *"Pre-glue test completed," "All five shell panels fit," "All four corners seated," "Shell could be squared," "All four rails seated against floor and walls"* — genuinely human-readable, no raw camelCase key exposed anywhere in the recorder. Blocker text was independently confirmed live and via fixture (`"All five shell panels fit must be recorded Yes."`) — the promotion-eligibility errors use the same label lookup, so no camelCase key leaks into blocker messages either.

**Serialized keys unchanged:** `tabletopTrayRequiredPositiveObservations`/`…NegativeObservations`/`…ObservationValues` (the actual storage-key arrays) are untouched by this correction — only a parallel, purely additive label map and a lookup function were added. Required-positive/disqualifying meanings are unchanged (same `tabletopTrayResultEligibility` gate logic, now using the label function only for message text). Unknown still blocks (`raw.observations[key]!==true`/`!==false` logic unchanged); no observation defaults to a passing value (recorder still renders `""`/Unknown-selected for every fresh control, confirmed live). The label map covers exactly the complete 29-key set with no missing or extra entries (confirmed by the exact-count fixture assertion and by direct counting of the object literal).

## 5. Promoted View and unpromoted-edit results (Correction 3)

**Live-verified, not only source-read:**
- Opened **View** on the just-promoted T3B result: modal opened successfully, heading read **"View promoted Tabletop Storage Tray result"**, with the note *"Read-only: this raw result has already been promoted. Record a new result for a later test."*
- **All 43 non-hidden form controls were `disabled` or `readonly`** (checked programmatically: `nonHiddenControls.every(c=>c.disabled||c.readOnly)` → `true`).
- **No Save button present** (`hasSaveBtn: false`), Close (not Cancel) shown.
- **No `onsubmit` handler attached at all** (`form.onsubmit` was `null`/falsy) — a second, independent layer of protection beyond the `disabled` attributes.
- **Frozen provenance panel present**, showing "T2 shell evidence"/"T3A storage evidence" sections (both populated, not "Not recorded", per §3).
- **Escape closed the modal** (`closedByEscape: true`); `localStorage` was byte-identical before and after (`dataUnchanged: true`); **reopening via View again** showed the same read-only heading and state.
- **A second, separate, unpromoted T3B raw result** was created and opened via its own "View / edit" button: heading read **"Edit Tabletop Storage Tray result"**, the notes field was **not** disabled, and a Save button **was** present — confirming unpromoted records remain fully editable while promoted ones do not.
- **Existing deletion warnings and revocation behavior remain intact**: the T2/T3A raw-delete confirmations used their original, unmodified wording (§3); the T3B raw-result delete confirmation logic (`record.promotedEvidence?'Delete this raw result? Its independent Complete Tray-proven Library evidence remains.':'Delete this raw Tabletop Storage Tray result?'`) is unchanged from the prior audit and was not re-broken by this correction (read directly in source, unchanged region).

Fixture-level: **"promoted View opens a read-only modal with frozen provenance"** and **"promoted read-only View cannot resave or mutate the raw result"** both pass (66/0 total), and **"unpromoted result recorder remains editable"** passes independently.

## 6. Actual browser interactions completed

Disposable headless Microsoft Edge (`--headless=new`), disposable `--user-data-dir`, direct `file:///C:/Genmitsu%20L8%20Tracker/index.html` navigation, driven via real DOM clicks/`dispatchEvent` form submissions — not only self-test routes. Completed, in this order, all against synthetic disposable data only: (1) material profile + machine creation; (2) real T2 shell generation → recording → promotion to Shell-proven; (3) real T3A generation → recording → promotion to Storage Fit Coupon-proven; (4) **real deletion of both raw T2 and T3A source results** via the actual Delete buttons, confirmed wording captured; (5) navigation to T3B with matching material, confirming the evidence card still showed both component rows Exact; (6) opened the T3B recorder, confirmed all 29 observation controls start Unknown, confirmed human-readable labels, entered retrospective-manual mode, all required observations, left every optional numeric field blank, saved; (7) confirmed the raw result was **eligible for promotion** despite both raw component sources being deleted; (8) **canceled** the promotion confirmation first and confirmed no promotion occurred; (9) **confirmed** the promotion for real; (10) confirmed the results list immediately showed "Complete Tray-proven" and "frozen T2 + T3A snapshots retained"; (11) confirmed the **main Designs evidence card refreshed to "Complete Tray-proven — Exact" immediately upon closing the results modal, with no reload**; (12) opened **View** on the promoted record and confirmed full read-only behavior (43/43 controls disabled, no Save, no onsubmit, provenance populated); (13) closed with **Escape**, confirmed `localStorage` byte-unchanged, reopened via View and confirmed the same read-only state; (14) created a **second, unpromoted** T3B result and confirmed it opened in a genuinely editable "Edit" mode; (15) **reloaded the page** and confirmed all four evidence rows, both raw records, and the promoted status all persisted with `document.readyState==='complete'` and no error banner; (16) confirmed **zero duplicate DOM ids** (56 total, 56 unique) after all of the above.

**Not separately re-performed this session** (relying instead on the prior audit's already-passing, unmodified coverage plus this session's fixture confirmation): a fresh manual UI-driven revocation of *Library* evidence specifically through the Library browser's own evidence-management controls (as opposed to the fixture's direct-state construction of `withoutShellEvidence`/`staleShellEvidence`, which is a faithful and already-passing proxy for the same effect). This is disclosed rather than substituted as equivalent.

## 7. Console results

No dedicated `Console.enable`/log-listener CDP session was attached from the start of this verification; console-error absence was inferred from every scenario completing with its expected DOM/state outcome at each checkpoint (no thrown exceptions surfaced through any `Runtime.evaluate` call, and `document.readyState==='complete'` with no visible error banner after every reload). This is a weaker guarantee than an explicit console listener and is disclosed as such, consistent with how the implementation report itself notes a pre-existing, unrelated renderer message (`Error: <svg> attribute height: Expected length, "auto".`) that was not independently reproduced or investigated in this narrow pass, since it is reported as pre-existing and unrelated to the T3B evidence corrections.

## 8. Focused fixture totals (fresh, this verification)

- **`runTabletopTrayResultFixtures`: 66 / 0** (matches the claim exactly; all 66 assertion names individually reviewed, see §3–§5 for the substantive ones).
- `runTabletopStorageTrayFixtures` (original T3B generator): **36 / 0**.
- `runModalAccessibilityFixtures`: **37 / 0**.
- `runDesignGeometryFixtures`: **1093 / 0**.
- `runTabletopShellEvidencePromotionFixtures`: **16 / 0**. `runTabletopShellEvidenceMatchingFixtures`: **16 / 0**.
- `runTabletopStorageFitCouponFixtures`: **23 / 0**. `runTabletopStorageResultFixtures`: **14 / 0**.

## 9. Unique full-suite accounting

Every exposed `run*Fixtures()` function invoked once (**47** functions): **3312 passed / 0 failed.** Summing only the **45** groups actually dispatched by `?selftest=all` (44 direct dispatch-line matches + `runMaterialBrowserFixtures`, dispatched via a different line format, as established in prior audits): **3032 / 0 — exact match to the claimed total**, with the two exposed-but-differently-routed functions (`runTrayModelFixtures` 264, `runPromotionTargetSwitchFixtures` 16) correctly excluded from the registered-group sum, same as every prior reconciliation in this project. No registered group disappeared; no total was inflated by aliases or repeated execution — this is a direct, independent summation, not a re-quote of the report.

## 10. Production-golden results

All independently re-confirmed fresh this verification, via each generator's own fixture-captured hash-check value (not re-quoted from any report):

| Golden | Expected | Confirmed |
| --- | --- | --- |
| Legacy pocketed shell | 2181 / `2ef9606b` | ✓ literal present, group green |
| T1 coupon | 2992 / `4f543f95` | ✓ |
| T2 shell | 2337 / `ed5d6f6e` | ✓ |
| T3A storage-fit coupon | 897 / `b5c549ee` | ✓ — freshly regenerated: `{"length":897,"hash":"b5c549ee"}` |
| T3B storage tray | 2860 / `3e256fad` | ✓ — freshly regenerated: `"2860 / 3e256fad"` |
| Dice Tray | 1726 / `51a55721` | ✓ |
| Alternate Dice Tray | 1054 / `41697123` | ✓ |
| Divider Tray | 1965 / `a55dda6e` | ✓ |
| Wall-to-base coupon | 1551 / `d9ffc278` | ✓ |

No additional or unexplained production golden changes found. No production geometry, serialization, panel-layout, or filename code appears anywhere in this correction's diff (confirmed by direct reading — the diff touches only the observation-label constant, the `tabletopTrayLiveComponentMatches` function body, and the recorder/results-list HTML-generation functions).

## 11. Protected-boundary confirmation

Confirmed unmodified (absent from the diff): `buildBoxModel`, `buildFingerPattern`, `serializeTabletopAccessorySvg`, `layoutDesignPanelRows`, `designSvgValidation`, all three T2/T3A/T3B design-result builders, `tabletopRectangularShellEvidenceMatches`, `tabletopStorageExactMatch`, `tabletopStorageEvidenceMatches`, `tabletopStorageTrayShellEvidenceMatches`, `tabletopStorageTrayExactMatch`, `productionMachineIdentityMatches`, `tabletopJointFitCompatible`, `promotionCandidateBase`, `savePromotionTransaction`, `SCHEMA_VERSION` (still 4), `APP_ID`/`APP_NAME`/`APP_VERSION`/`BUILD_DATE`/`STORAGE_KEY`/`BACKUP_FORMAT`, Project schema, Library schema. Existing T1/T2/T3A record meanings and Joe's own (real) promoted evidence are untouched by any correction in this diff.

## 12. Unverified behavior (explicit disclosure)

- A dedicated CDP console-log listener was not attached; console-error absence is inferred from correct outcomes at each step, not from a captured log stream (§7).
- Library-side manual evidence revocation through its own UI (rather than the fixture's direct-state construction) was not separately click-driven this session; the fixture's construction is a faithful proxy and both pass, but this is disclosed as not independently re-driven through that specific UI surface.
- Schema-1/2/3 seeded-reload scenarios were not separately re-run in this narrow verification (unchanged code path from the immediately preceding full audit, which did verify the analogous schema-3→4 transition).

## 13. Safe-to-commit determination

**Yes.** All three targeted corrections are genuinely implemented (not merely claimed), verified both by direct source reading and by live, real UI interaction with synthetic data, with no regression to any protected function, matcher, schema behavior, or production golden. The 66/0 and 3032/0-across-45-groups totals are independently reproduced exactly as claimed.

## 14. Whether Joe may safely enter his real T3B result

**Yes.** The recorder still starts every observation Unknown, never invents a value, permits retrospective-manual entry with blank optional measurements, and now displays genuinely readable labels throughout — nothing in these corrections changes what data is captured or how it is validated.

## 15. Whether Joe may safely promote his real T3B result

**Yes, unconditionally now** — this is the direct improvement over the prior audit's caution. Promotion no longer depends on Joe's raw T2/T3A result records still being present; it correctly and exclusively depends on his exact promoted T2 Shell-proven and T3A Storage Fit Coupon-proven Library evidence still existing and matching. He may freely delete raw component records for tidiness at any time, before or after this promotion, without losing the ability to promote a genuine, currently-eligible T3B result.

## 16. Exact final verdict

**1. VERIFIED — Safe to commit, record, and explicitly promote Joe's real T3B result.**
