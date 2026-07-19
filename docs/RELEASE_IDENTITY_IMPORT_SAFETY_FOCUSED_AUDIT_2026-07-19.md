# Release Identity and Import Safety — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer model:** Claude Opus 4.8
**Type:** read-only implementation audit. No file was edited, staged, committed, pushed, reset, cleaned, stashed, checked out, moved, renamed, or deleted.

---

## 1. Repository state and actual baseline

- `git status -sb`: `## main...origin/main`. Modified (unstaged): **`index.html`**, **`README.md`**. Untracked: **`CHANGELOG.md`**, `docs/RELEASE_IDENTITY_IMPORT_SAFETY_IMPLEMENTATION_2026-07-19.md`, `docs/PAID_RELEASE_READINESS_ROADMAP_REVIEW_2026-07-19.md`, plus the long-standing unrelated set (`LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, older `docs/*.md`).
- `git log -1 --oneline`: **`56bff51 Add Designs to Projects handoff`**.
- `git rev-parse HEAD`: **`56bff511215dbfea941cfe832ba51e808d6fa79d`** — equals the expected baseline `56bff51`.
- `git rev-list --left-right --count origin/main...main`: **`0 0`** (synchronized; no local commits, nothing pushed).
- `git diff --check`: clean (only benign LF→CRLF warnings; no whitespace/conflict errors).
- `git diff --stat`: `README.md` +4/-4, `index.html` +48/-6 (54 changed lines).
- `git diff --cached`: **empty — nothing staged.**
- **Baseline cleanliness:** the tracked baseline was clean before this phase; the implementation exists solely as uncommitted working-tree changes to two tracked files plus the new untracked `CHANGELOG.md`. No commit or push occurred. Unrelated untracked content is untouched.

## 2. Files and functions reviewed

`index.html`: new constants `APP_ID`/`APP_NAME`/`APP_VERSION`/`BUILD_DATE`/`BACKUP_FORMAT` (`:265-269`); unchanged `STORAGE_KEY` (`:266`) and `SCHEMA_VERSION` (`:267`); `validateBackupImport()` (`:423-430`); `decodeStoredState()` (unchanged); `backupObject()` (additive fields, `:544-547`); `loadState()`/`freshState()`/`persist()` (unchanged); `mergeData()`/`replaceData()` (unchanged); new `applyBackupImport()` (`:11063-11068`); the file-import `onchange` handler (`:11248-11274`); `renderReleaseIdentity()` (`:920-924`) + footer element (`:248`) + `.release-identity` CSS (`:61-63`); `runStorageRecoveryFixtures()` (7 new assertions); startup wiring (`:11293`). `README.md` and `CHANGELOG.md` reviewed in full. Protected Designs geometry/serializer/download/finished-view code confirmed absent from the diff.

## 3. Summary of implementation behavior

The phase adds five identity constants; renders a compact, native-`<details>` **release identity footer** from those constants; writes **`app`/`appVersion`/`backupFormat`** additively at the top of `backupObject()` (keeping `schemaVersion` and every existing field); introduces **`validateBackupImport(data)`** which refuses (a) a non-object/array/null payload, (b) a **numeric** `schemaVersion` greater than `SCHEMA_VERSION`, (c) a present `app` that differs from `APP_ID`, and (d) a present `backupFormat` that differs from `BACKUP_FORMAT`; runs that validation in the import handler **before** the merge/replace prompt and before any state change; and routes accepted imports through a new **`applyBackupImport(data, mode, validation)`** helper that preserves the existing merge (id-based) and replace (with confirmation) semantics. Seven storage-recovery fixtures were added.

## 4. Release identity findings

- Constants match the contract exactly: `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`; `SCHEMA_VERSION` remains `2`; `STORAGE_KEY` remains `'genmitsu-l8-tracker-v1'`. Verified in source.
- The footer renders **from the constants** (no duplicated literals): headline `Genmitsu L8 Tracker v0.9.0` plus an About disclosure showing name, version, `Build 2026-07-19`, `Storage schema 2`, `Backup format genmitsu-l8-tracker-backup-v1`, and "Works offline and stores records locally in this browser." All interpolations pass through `esc()`.
- Uses a **native `<details>/<summary>`** disclosure — keyboard accessible, no custom modal, no new focus-trap burden.
- Wording is accurate and does **not** imply cloud backup, accounts, synchronization, automatic updates, or completed 1.0/paid-release readiness (version is explicitly 0.9.0).
- The footer is a normal-flow element inside `<main class="shell">`, after `#app`; `.release-identity` uses `flex-wrap:wrap` and a top border, no `fixed`/`sticky` positioning, so it does not obstruct content at desktop/tablet/narrow widths.
- **No duplicate HTML IDs:** `id="releaseIdentity"` occurs exactly once; the About block introduces no `id`. `renderReleaseIdentity()` is called exactly once at startup, and `render()` does not touch the footer (it rewrites only `#app`/`#tabs`), so identity persists across tab switches without re-render.

## 5. Import-validation findings

Edge-case matrix against `validateBackupImport()` (source-confirmed):

| Input `schemaVersion` / metadata | Result | Correct? |
|---|---|---|
| number `> 2` | **Refused** ("newer data format") | ✅ |
| number `=== 2` | Accepted | ✅ |
| number `=== 1` | Accepted | ✅ |
| missing | Accepted (legacy) | ✅ |
| numeric **string** `"3"` | **Accepted** (`typeof !== 'number'`) | ⚠️ see below |
| NaN-like / malformed text | Accepted | ⚠️ same class |
| `null` | Accepted (typeof object, not number) | ✅ acceptable |
| object/array | Accepted | ⚠️ same class |
| missing `app` | Accepted | ✅ |
| correct `app` | Accepted | ✅ |
| incorrect `app` | Refused | ✅ |
| `app: null` | Refused (`'app' in data` true) | ✅ (only foreign/edited files carry a null key; legacy omits the key) |
| `app` non-string | Refused | ✅ |
| missing `backupFormat` | Accepted | ✅ |
| correct `backupFormat` | Accepted | ✅ |
| incorrect `backupFormat` | Refused | ✅ |
| `backupFormat: null` / non-string | Refused | ✅ (same reasoning as `app`) |
| object JSON, no metadata at all | Accepted | ✅ deliberate legacy-compat tradeoff |
| legacy Tracker backup | Accepted | ✅ |

**Order of operations is correct:** in the handler, `validateBackupImport(data)` runs on the parsed object immediately after `decodeStoredState`, and on refusal does `alert(message); return;` — **before** the merge/replace `confirm`, before `applyBackupImport`, and before `persist`. `applyBackupImport` independently re-guards (returns the validation object on refusal before calling `mergeData`/`replaceData`).

**String/malformed `schemaVersion` tradeoff (⚠️):** because the guard uses `typeof data.schemaVersion === 'number'`, a `schemaVersion` of `"3"` (string), NaN-like text, or a non-number type slips past the future-version refusal. This is a **low-severity robustness gap, not a data-loss blocker**: every backup this application lineage produces writes `schemaVersion` as a genuine number (see `backupObject()` → `SCHEMA_VERSION`), so a real future version of *this* app is correctly refused; a string `schemaVersion` can only arise from a hand-edited or foreign file, which must additionally lack `app`/`backupFormat` and structurally resemble a Tracker backup to cause any harm, after which `persist()` re-stamps a numeric `SCHEMA_VERSION` anyway. An optional one-line hardening (`Number(data.schemaVersion) > SCHEMA_VERSION`, which stays legacy-safe because `Number(undefined)` is `NaN` and `NaN > 2` is `false`) would close it, but it is not required for the bounded contract. Classified **POLISH**.

**False-acceptance of metadata-free object JSON is *not* a defect:** the contract explicitly warns against rejecting all metadata-free object JSON because it is not safely distinguishable from a legitimate legacy backup (legacy backups have no `app`/`backupFormat` and may lack `schemaVersion`). Accepting it and relying on downstream defensive normalization is the correct bounded choice.

**Retry after refusal works:** `e.target.value = ''` executes synchronously in the `onchange` body (`:11272`, after `reader.readAsText`), so the input is cleared on every selection regardless of accept/refuse — the same filename can be re-selected after a refusal.

## 6. Legacy compatibility findings

Legacy backups remain fully importable: those lacking `app`, `appVersion`, `backupFormat`, and/or `schemaVersion` are all accepted (no key ⇒ `'app' in data`/`'backupFormat' in data` are false; missing/`1`/`2` numeric `schemaVersion` passes). `loadState()` continues to read only its known key set, so the additive `app`/`appVersion`/`backupFormat` fields are simply ignored on restore (they never enter `state` and are freshly re-emitted by `backupObject()` on the next export). Fixture "Legacy backups without release metadata or schema version remain accepted" exercises both the `schemaVersion:1` and missing-schema cases. **No legacy-backup rejection introduced.**

## 7. State / localStorage mutation analysis

- **Refused path:** the handler alerts and returns before the merge/replace prompt; `applyBackupImport` returns the validation object before any mutation. Fixture "Refused imports do not reach merge or replace or mutate storage" drives `applyBackupImport({...backupObject(), app:'foreign-app'}, 'merge')` and asserts `state` and `localStorage` are byte-identical before/after. **No mutation on refusal.**
- **No pre-decision stamping:** accepted data is passed unchanged into `mergeData`/`replaceData`; nothing rewrites `schemaVersion` or normalizes records before the user chooses merge/replace. `persist()` (which stamps `SCHEMA_VERSION`) is only reached after an accepted import.
- **No hidden mutation via shared references in validation:** `validateBackupImport` only reads properties (`typeof`, `in`, `!==`) and never mutates or deep-touches `data`; it constructs no shared references. Fixtures that mutate `state.entries` restore it from a JSON snapshot in a `finally` block via the new `restore()` helper, and also restore `localStorage`, so the fixture run itself is side-effect-neutral.
- **Corrupt-recovery overwrite protection preserved:** a refused import returns before `persist({ allowRecoveryOverwrite: recovering })`, so damaged stored data is never overwritten by a rejected file; the accepted path retains the existing `allowRecoveryOverwrite` semantics.

## 8. Export and round-trip findings

- Export metadata is **additive only**: `backupObject()` prepends `app`/`appVersion`/`backupFormat` and leaves `schemaVersion` and every prior field present and semantically unchanged.
- Round-trip preserves data, including **unknown additive record fields**: fixture "Current-format backups retain replace behavior and additive record fields" round-trips an entry carrying `futureEntryField:'preserve'` through `replaceData` and `backupObject()` and confirms it survives. Merge fixture confirms id-based merge preserves both existing and incoming records and their extra fields.
- Photos/large backups are unaffected (no change to photo handling or the export size-warning path).
- Replace still requires the existing confirmation; merge still uses existing id semantics — the `onchange` refactor preserves both prompts and only relocates the `mergeData`/`replaceData` calls into `applyBackupImport`.
- Export filename, MIME type, and download path are unchanged (not in the diff).
- `BACKUP_FORMAT` is used only as an **identity gate** and for display; it is never conflated with `SCHEMA_VERSION`. `appVersion` is informational — the storage-schema gate uses the numeric `schemaVersion` vs `SCHEMA_VERSION`, never `appVersion`. No automatic schema downgrade path for a numeric future backup exists (it is refused).

## 9. Corruption-recovery findings

`decodeStoredState()` is unchanged; corrupt storage still yields `status:'corrupt'`, sets `storageRecoveryIssue`, opens with empty data, and blocks silent overwrite. The import guard runs after `decodeStoredState` confirms `status==='ok'` on the *imported file* (independent of stored-state recovery). A refused import cannot overwrite recovery data because it returns before `persist`. The recovery panel's own import entry point (`recoveryImportBackup` → the same `importFile` input) now also benefits from the guard. **Recovery architecture intact.**

## 10. Fixture-quality and total reconciliation

Seven new assertions were added to `runStorageRecoveryFixtures()` (group grew 8 → 15), each exercising the **real** seams (`backupObject`, `validateBackupImport`, and `applyBackupImport` → `mergeData`/`replaceData`), not stub helper calls:
1. New backups carry `app`/`appVersion`/`backupFormat`/`schemaVersion`.
2. Legacy (schema 1 and missing-schema) backups accepted.
3. Future numeric schema refused with the "newer data format" message.
4. Foreign `app` and unsupported `backupFormat` refused with the supported-backup message.
5. Refused import does not reach merge/replace and does not mutate `state`/`localStorage` (via `applyBackupImport`).
6. Current-format **merge** behavior retained (adds incoming, keeps existing, preserves additive field).
7. Current-format **replace** behavior + additive record field round-trip retained.

**Coverage gaps (acceptable):** the DOM `<input type="file">` `onchange` seam itself (FileReader, `alert`, `confirm`, `e.target.value` reset) is not fixture-driven — hard to automate without browser I/O — but its decision logic is fully covered by `validateBackupImport`/`applyBackupImport` fixtures; the thin `alert/return` wrapper is straightforward source review. No dedicated fixture asserts the footer render or duplicate-ID absence (verified here by source/grep instead).

**Total reconciliation:** README now claims `1909 / 0` with `Storage recovery 15/0`, `Designs 1093/0`, arithmetic `816 non-Design + 1093 Designs = 1909`. The only runner touched is `runStorageRecoveryFixtures` (**+7**); every other group is unchanged in the diff. Prior baseline was `1902` (`809 + 1093`); `809 + 7 = 816`, `816 + 1093 = 1909`. **The 1909 total is analytically supported by the registered runners.** (Runtime execution was not performed — see §11.)

## 11. Browser / file:// validation results

- `git diff --check`: clean.
- **Runtime execution was NOT performed.** `node` is unavailable in this environment and no browser automation was used, so the `?selftest=all` suite, `file://` startup, and live footer/About render were **not** executed. All fixture-count and pass/fail statements here are **analytical reconciliations of source**, not observed runs.
- Static/source checks that *were* performed: full unified diff review; confirmation of a single `id="releaseIdentity"`; confirmation that **no** network/dependency constructs were introduced (`grep` for `fetch`/`XMLHttpRequest`/`WebSocket`/`<script src>`/`<link>`/CDN hosts returned nothing new — offline/`file://` posture preserved); constants and `STORAGE_KEY`/`SCHEMA_VERSION` values; `loadState` additive-field handling; `e.target.value` reset location; CHANGELOG contents.
- **Recommendation:** before commit, run `index.html?selftest=all` in Edge and confirm `1909 / 0` and a rendered footer, since this auditor could not execute it.

## 12. Documentation findings

- `README.md`: title now `Genmitsu L8 Tracker v0.9.0`; storage section accurately states backups include app/version/backup-format metadata, that **legacy backups without the new metadata remain supported**, and that **backups declaring a newer storage schema are refused rather than silently imported**; fixture totals updated to `1909`, `Storage recovery 15/0`, and the Designs-run line to `1909`. No claim of a formal migration framework, no 1.0/paid-release-complete claim, no broadened app/format-validation claim beyond what is implemented, no contradictory backup instructions.
- `CHANGELOG.md`: new, versioned `0.9.0 — 2026-07-19`, four accurate bullets (visible version/build identity; machine-readable export metadata; future-schema/incompatible-import safeguards; legacy compatibility preserved). No overclaiming.
- **Version duplication:** `0.9.0` appears as the `APP_VERSION` constant, in the README title, in the README body totals, and in CHANGELOG. For a no-build-system single-file app this manual duplication is acceptable; the in-app footer is constant-driven (the one that matters for users). Minor drift risk noted as POLISH.

## 13. Protected-boundary comparison

Confirmed by full diff that the implementation does **not** alter: Designs geometry; SVG serializers; SVG downloads; Finished Views; kerf/clearance calculations; dimensions/panel layout; production filenames; LightBurn output assumptions; project-data semantics; machine-profile semantics; `STORAGE_KEY`; `SCHEMA_VERSION`; merge id rules; replace semantics; corrupt-recovery architecture; or external-dependency/network behavior. The Designs fixture group (`1093`) is unchanged, consistent with zero production-byte impact. The entire diff is confined to identity constants, the footer, backup metadata, the import-guard/apply helpers, the import handler refactor, and storage-recovery fixtures/docs.

## 14. Findings classified by severity

**BLOCKER:** none.

**IMPORTANT:** none.

**POLISH (optional, safe to defer):**
- **P-1 — String/malformed `schemaVersion` bypasses the future-version guard.** `typeof === 'number'` lets `"3"`/NaN-text/non-number types through. Realistic data-loss risk is very low (this app lineage always emits numeric `schemaVersion`; a foreign file would also need to lack `app`/`backupFormat`). Optional legacy-safe hardening: `Number(data.schemaVersion) > SCHEMA_VERSION`. Boundary: `validateBackupImport` (`:425`).
- **P-2 — DOM `onchange` import seam not fixture-covered.** Decision logic is covered; the FileReader/alert wrapper is not. Acceptable given automation limits.
- **P-3 — Footer not hidden in `@media print`.** Cosmetic; identity text on a print is harmless.
- **P-4 — `0.9.0` duplicated across constant/README/CHANGELOG.** Manual bump drift risk; acceptable for a no-build app.

**NOT A DEFECT (deliberate, contract-endorsed):**
- Accepting metadata-free object JSON (indistinguishable from legacy legit backups).
- Refusing `app:null`/`backupFormat:null` (only foreign/edited files carry a null-valued key; legacy omits the key entirely).
- `loadState` dropping `app`/`appVersion`/`backupFormat` from live state (correct; re-emitted on export).

## 15. Final verdict

**APPROVED WITH POLISH DEFERRED**

The implementation delivers the bounded Release Identity and Import Safety phase completely and safely: exact constants, additive backup metadata, a correctly-ordered pre-decision import guard that refuses numeric future-schema/foreign-app/foreign-format backups while preserving every legacy and current-format import, no state/localStorage mutation on refusal, intact merge/replace/recovery semantics, no production-byte or network/offline impact, an accurate constant-driven footer, and honest README/CHANGELOG. The only findings are optional POLISH items (chiefly the string-`schemaVersion` robustness nuance), none of which blocks commit.

## 16. Remaining unverified areas

- **Runtime execution** of `?selftest=all`, `file://` startup, and the rendered footer/About (node unavailable, no browser automation). Counts are reconciled analytically only. Run once in Edge before commit to confirm `1909 / 0` and footer render.
- The live file-input `onchange` UX (alert copy, confirm flow, same-file retry) is source-verified but not runtime-observed.

## 17. Physical laser testing

**Not required.** This phase changes no geometry, dimensions, serializers, downloads, filenames, or production output; the protected-boundary comparison confirms zero production-byte impact.

## 18. Whether Codex may proceed to commit

**Yes** — Codex may proceed to commit after (a) a one-time Edge `?selftest=all` run confirming `1909 / 0` and a rendered footer (the runtime check this auditor could not perform), and (b) optionally applying POLISH P-1's one-line `Number()` hardening if desired. The POLISH items may otherwise be deferred without risk.

## 19. Audit integrity confirmation

This audit made **no edits** to any source or documentation file other than creating this report, and did **not** stage, commit, push, or otherwise alter git state. The working tree remains exactly as the implementation left it (HEAD `56bff51`, nothing staged, nothing pushed).
