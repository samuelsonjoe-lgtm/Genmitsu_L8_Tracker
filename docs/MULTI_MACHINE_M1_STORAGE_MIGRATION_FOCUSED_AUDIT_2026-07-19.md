# M1 — Multi-machine Storage Model and Legacy Migration — Focused Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit date:** 2026-07-19  
**Type:** Read-only storage, migration, backup, and compatibility audit  
**Architecture review:** `docs/MULTI_MACHINE_PROFILES_ARCHITECTURE_REVIEW_2026-07-19.md`  
**Implementation report (not sole evidence):** `docs/MULTI_MACHINE_M1_STORAGE_MIGRATION_IMPLEMENTATION_2026-07-19.md`  
**Audit method:** Independent `git` inspection, source/diff review of `index.html`, README/CHANGELOG, disposable Edge headless runtime (`file://` temp copies only), and high-risk scenario reproduction inside the app IIFE. Product sources were not modified, staged, committed, or pushed by this audit.

---

## Final verdict

**APPROVED WITH POLISH DEFERRED**

| Severity | Count |
|----------|------:|
| BLOCKER | 0 |
| IMPORTANT | 0 |
| POLISH | 4 |
| NOT A DEFECT | several (documented below) |

**May M1 be committed?** Yes.  
**May M2 begin after commit?** Yes — M1 is invisible groundwork only; M2 UI/manager work remains out of scope.  
**Physical laser testing required for M1?** No.

---

## 1. Repository state and actual baseline

| Check | Result |
|-------|--------|
| Branch | `main` (`## main...origin/main`) |
| HEAD | `288d474cacf9556638ded27600925aa3d7848160` |
| Subject | `288d474 Improve beginner clarity and import safety` |
| Matches expected baseline `288d474` | **Yes** |
| Ahead/behind `origin/main` | `0 0` |
| Staged changes | **None** (`git diff --cached` empty) |
| M1 commit/push | **None** — uncommitted working tree only |
| `git diff --check` | Clean (CRLF warnings only) |

**Modified tracked files (M1 worktree):**

| File | Diff stat |
|------|-----------|
| `index.html` | ~+151 / −18 (storage + fixtures + editor) |
| `README.md` | fixture totals + multi-machine storage prose |
| `CHANGELOG.md` | one M1 bullet |

**Relevant untracked (docs/history; not product code):** includes `docs/MULTI_MACHINE_M1_STORAGE_MIGRATION_IMPLEMENTATION_2026-07-19.md`, `docs/MULTI_MACHINE_PROFILES_ARCHITECTURE_REVIEW_2026-07-19.md`, other historical `docs/*`, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`. Unrelated to M1 product logic.

**Constants (worktree = HEAD for identity constants):**

| Constant | Value |
|----------|-------|
| `APP_ID` | `'genmitsu-l8-tracker'` |
| `APP_NAME` | `'Genmitsu L8 Tracker'` |
| `APP_VERSION` | `'0.9.0'` |
| `BUILD_DATE` | `'2026-07-19'` |
| `STORAGE_KEY` | `'genmitsu-l8-tracker-v1'` |
| `SCHEMA_VERSION` | `2` |
| `BACKUP_FORMAT` | `'genmitsu-l8-tracker-backup-v1'` |

All of the above are **unchanged** vs HEAD for identity/storage/schema/backup format.

**Fixture arithmetic (independent runtime):**

| Suite | Result |
|-------|--------|
| Expected pre-phase complete | 2205 / 0 |
| M1 group | **29 / 0** |
| Expected post-phase complete | 2234 / 0 |
| Actual complete `selftest=all` | **2234 passed / 0 failed** |
| Groups | **25** (including `machines-m1` once) |
| Non-Design | **1141** |
| Designs | **1093** |
| Tray standalone | **264 / 0** |
| Machine setup | **50 / 0** |

`2205 + 29 = 2234`. No extra modal/accessibility count inflation from M1 (no new permanent dialog chrome).

---

## 2. Files and functions reviewed

**Primary:** `index.html` (working tree vs `HEAD:index.html`), `README.md`, `CHANGELOG.md`, architecture review, implementation report.

**Functions / paths (minimum set):**

- `normalizeMachineIdentity`, `positiveMachineNumber`, `normalizeOwnedMachine`, `normalizeMachines`
- `machineById`, `resolveActiveMachineId`, `machineIdentityForActiveMachine`, `machineRootFromData`, `synchronizeMachineIdentity`
- `currentMachineDisplayName`, `freshState`, `loadState`, `persist`, `backupObject`
- `decodeStoredState`, `validateBackupImport`, `mergeData`, `replaceData`, `applyBackupImport`, `applyBackupPreferences`, `resetBackupPreferences`
- `saveMachineSetup`, Reference `machineSetupForm` renderer, submit binding
- `isFirstRunWorkspaceEmpty` / first-run eligibility
- `activeMachineSnapshot`, `gridMachineSnapshot`, `productionMachineKeys`
- `runMultiMachineStorageFixtures`, selftest registry (`machines-m1` / `all`)
- Design builders compared to HEAD (no body changes)

---

## 3. Root storage-shape findings

Verified `freshState()` / persist payload shape:

```text
machines: []
activeMachineId: ''
machineIdentity: { name: '', workAreaWidthMm: null, workAreaHeightMm: null }  // plus any additive fields if present
machineProfile: '20W'   // independent
```

| Requirement | Status |
|-------------|--------|
| `machineIdentity` retained (not removed/renamed) | **Pass** |
| `machineProfile` independent | **Pass** |
| `machines` + `activeMachineId` in `persist()` and `backupObject()` | **Pass** |
| No new `STORAGE_KEY` | **Pass** |
| No second hidden machine collection | **Pass** |
| No machine data only in a transient lookup map | **Pass** — `machineById` is pure lookup |

---

## 4. Machine-normalization findings

**Contract (`normalizeOwnedMachine` / `normalizeMachines`):**

| Field | Behavior |
|-------|----------|
| `id` | Required non-blank string after trim; missing/blank → entry dropped (**not** auto-assigned) |
| `name`, `manufacturer`, `model`, `notes` | Coerced + trimmed strings |
| `type` | Lowercased; only `diode`/`co2`/`fiber`/`other`; else `other` |
| `nominalPowerW`, work-area mm | Finite and `> 0` or `null` (0, negative, NaN, Infinity, non-numeric → `null`) |
| `archived` | `!!value` boolean |
| Unknown additive fields | Survive via object spread |
| Duplicate IDs | First valid occurrence kept; source order stable |
| Input mutation | Array/items not mutated (fixture + inspection) |
| `persist()` from normalizers | **No** |

### “Paired positive-or-null work area”

**Exact behavior:** width and height normalize **independently** via `positiveMachineNumber` / identity `positiveMm`. One invalid dimension does **not** clear the other.

**Editor path:** `saveMachineSetup` still requires **both blank or both filled** (paired form validation) — separate from storage normalization.

**Report phrase** “paired positive-or-null work area” is slightly imprecise relative to the normalizer; form pairing is real. Classified as **POLISH** (documentation wording), not a storage defect.

---

## 5. Legacy-detection findings

Detection:

```js
const legacy = !Object.prototype.hasOwnProperty.call(source, 'machines');
```

| Case | Behavior |
|------|----------|
| No own `machines` | Legacy migration |
| `machines: []` | New-format empty; no fabricate |
| `machines: null` / object / string | `normalizeMachines` → `[]`; not legacy |
| Array of only invalid entries | Empty collection; not legacy |
| Inherited via prototype only | Treated as **absent own property** → legacy (verified runtime) |
| Exceptions | None observed |

**Pass** against required contract.

---

## 6. Legacy-migration findings

Absent own `machines` → exactly one machine:

| Field | Value |
|-------|--------|
| `id` | `legacy-workshop-machine` |
| `name` | normalized legacy identity name (blank allowed) |
| `manufacturer` / `model` / `notes` | `''` |
| `type` | `diode` |
| `nominalPowerW` | `null` |
| work area | from legacy identity |
| `archived` | `false` |
| `activeMachineId` | `legacy-workshop-machine` |

| Requirement | Status |
|-------------|--------|
| Genmitsu fallback **not** persisted as name | **Pass** |
| Labels/tests/settings do not create machines | **Pass** (no such path in M1) |
| Idempotent re-normalize after `machines` present | **Pass** |
| In-memory on load; no auto-write | **Pass** (`loadState` does not call `persist`; runtime confirmed payload unchanged after load) |
| Ordinary save later persists migration | **Pass** (expected) |

---

## 7. Fresh-workspace findings

| Requirement | Status |
|-------------|--------|
| `machines: []`, `activeMachineId: ''` | **Pass** |
| Blank `machineIdentity` | **Pass** |
| No `legacy-workshop-machine` from `freshState()` | **Pass** |
| No Genmitsu as owned machine | **Pass** |
| Display fallback still works | **Pass** (`currentMachineDisplayName` → Genmitsu L8 20W/40W) |
| First-run still eligible | **Pass** (`isFirstRunWorkspaceEmpty` ignores machines/activeMachineId) |
| Machines alone not “meaningful” for onboarding | **Pass** (fixture) |

---

## 8. Active-resolution findings

`resolveActiveMachineId` order:

1. Requested existing **non-archived** machine  
2. First non-archived  
3. First machine (archived) if all archived  
4. `''` if empty  

Final active ID always exists in the normalized list or is blank. Malformed IDs fall through safely. **Pass.**

---

## 9. `machineIdentity` mirror findings

When active exists, mirror is rebuilt as **only**:

- `name`
- `workAreaWidthMm`
- `workAreaHeightMm`

Not copied: manufacturer, model, type, power, notes, archived, unknown additive machine fields, Genmitsu display fallback. Blank name stays blank. Sync is idempotent and does not replace the machine object identity wholesale (map update preserves advanced fields on save). When no active machine, fallback identity is normalized and retained (e.g. explicit empty Replace staging identity). **Pass.**

---

## 10. Existing-editor findings

`saveMachineSetup` via real form submit:

| Scenario | Behavior |
|----------|----------|
| Active machine present | Updates name + work area only; stable ID and advanced fields preserved; one `persist()`; clearing fields keeps the machine |
| No machines | Creates one `uid()` machine, sets active, default structured fields, mirrors identity |
| `machineProfile` | Unchanged by this form |

**Pass.** Runtime also confirmed create → persist → `loadState` keeps stable ID.

---

## 11. Backup findings

`backupObject()` includes `machines`, `activeMachineId`, `machineIdentity` plus prior collections/preferences. Constants and accepted validation paths unchanged. New backups carry active-machine name/work area in `machineIdentity` for older consumers. Documentation does **not** claim inactive owned machines survive import into a pre-M1 app (older apps ignore unknown `machines` and keep only mirrored identity). **Pass.**

---

## 12. Merge findings

| Rule | Status |
|------|--------|
| Local machines + order first | **Pass** |
| Unique imported IDs appended | **Pass** |
| Same ID → local wins | **Pass** |
| Same name, different IDs → both kept | **Pass** (fixture + independent logic) |
| Imported `activeMachineId` ignored | **Pass** |
| Local valid active retained | **Pass** |
| Imported `machineIdentity` does not overwrite local when active remains | **Pass** |
| Legacy import without `machines` → no singleton fabrication | **Pass** |
| Empty imported machines → no-op on collection | **Pass** |
| Archived + additive fields preserved | **Pass** |
| No name/model/power/work-area dedup | **Pass** |
| `machineProfile` Merge baseline via `applyBackupPreferences` | Unchanged baseline behavior |
| Empty local + imported machines → append + active fallback | **Pass** (independent runtime; not in the 29 named asserts) |
| `legacy-workshop-machine` ID collision → local wins | **Pass** (independent runtime) |

---

## 13. Replace findings

| Case | Status |
|------|--------|
| New-format machines list replaces local | **Pass** |
| Archived active falls back to live | **Pass** |
| Sync mirror to resolved active | **Pass** |
| Legacy absent `machines` → one `legacy-workshop-machine` | **Pass** |
| Explicit `machines: []` → empty + blank active + retain staging identity | **Pass** |
| Malformed `machines` → empty, not legacy, clears local | **Pass** (fixture + independent) |
| Other collections/preferences | Still via existing replace + preference helpers |

---

## 14. Recovery findings

Corrupt whole-storage path unchanged; recovery still blocks save unless allowed. Malformed `machines` alone does not force total-storage recovery (treated as empty new-format root). Startup does not throw. Migration does not auto-persist. Import refusal and Cancel remain non-mutating (covered by storage/import fixtures still green). **Pass.**

---

## 15. Historical-record protection

| Check | Status |
|-------|--------|
| No `machineId` added to Logs/Grids/Tests/Projects/etc. | **Pass** (diff + fixture deep equality on synthetic records) |
| `activeMachineSnapshot` / `gridMachineSnapshot` / `productionMachineKeys` | **Unchanged** vs HEAD |
| `machineProfile` still seeds legacy snapshots | **Pass** |
| Evidence / promotion / Project Wizard | No M1 wiring into those paths |
| synchronize does not rewrite historical snapshots | **Pass** |

---

## 16. Designs-output protection

No design builder / geometry / kerf / SVG serialization / download / Finished View function bodies changed vs HEAD. Independent runtime: changing machines/active identity does not alter representative `buildDesignResult` SVG bytes. Design fixture group **1093 / 0**. **Pass.**

---

## 17. First-run and user-visible scope

| Check | Status |
|-------|--------|
| No Machines manager / header selector / archive-delete-duplicate UI | **Pass** |
| Only existing Current workshop machine editor | **Pass** |
| Reference profile separately labeled | **Pass** |
| README/CHANGELOG: invisible groundwork, not seamless switching | **Pass** |
| No claim of owned IDs on historical records or fit checking | **Pass** |

---

## 18. Fixture-quality findings

`runMultiMachineStorageFixtures()` — **29** `add()` assertions (exact match to implementation claim):

1. Fresh empty machines/active  
2. Fresh blank identity + display fallback  
3. Machines alone do not suppress first-run  
4–6. Legacy custom / blank / fields  
7. Idempotence after machines present  
8–9. Present-empty / present-malformed  
10–12. Normalization, bounds, lookup  
13. Active fallback  
14. Helpers do not persist  
15–16. Mirror + idempotent sync  
17. Historical snapshot equality / no `machineId`  
18–19. Backup inclusion + constants; machineProfile untouched  
20–22. Editor update / clear / create  
23–25. Merge union, active preservation, legacy merge  
26–29. Replace new / legacy / empty / malformed  

**Combined assertions** (one `add` covers multiple rules): normalization, merge ordering+collision+name-dup, active resolution matrix.

**Absent from the 29 (behavior verified outside the group):**

| Gap | Risk | Classification |
|-----|------|----------------|
| Empty-local Merge active fallback | High-risk path | **POLISH** (runtime verified correct) |
| Own-property / prototype detection | High-risk | **POLISH** (source + runtime verified) |
| Full `loadState` no auto-persist (only helper path in group) | High-risk | **POLISH** (runtime verified) |
| Explicit `legacy-workshop-machine` Merge collision | Medium | **POLISH** (runtime verified) |
| Independent work-area dimensions | Medium | **POLISH** (runtime verified) |

No missing coverage rose to IMPORTANT after independent reproduction.

---

## 19. Fixture-realism findings

| Style | Present? | Notes |
|-------|----------|-------|
| Source-text search only | Minimal | Not the core of M1 group |
| Normalize without `loadState` | Yes | Root migration unit-tested via `machineRootFromData` |
| Merge helpers without full Import modal framing | Yes | Same functions Import calls after validation |
| Manual state assign + real form submit | Yes | Editor paths use real form events |
| Selected-property vs deep equality | Mixed | Historical records use full JSON equality |
| Invented schemas | No | Uses app helpers and backup shapes |

Independent disposable-profile scenarios (13/13 pass): legacy custom/blank load, no auto-persist, repeated load, empty-local Merge, same-name/different-ID, legacy-ID collision, independent dims, prototype legacy, invalid-only array, malformed Replace, editor create stable reload, Designs bytes.

---

## 20. Fixture-cleanup findings

M1 restore captures/restores: `state`, `localStorage`, `storageRecoveryIssue`, first-run dismiss flag, design draft/preview, `pendingImport`, `importApplying`, then `render()`.

**Not** as exhaustive as some modal-heavy groups (modal HTML, focus origins, open order). After `render()`, no synthetic machines remain in restored state. **Acceptable; POLISH** if tighter parity with other fixture restores is desired later.

---

## 21. Fixture-total reconciliation

| Item | Value |
|------|------:|
| Pre-phase complete | 2205 |
| M1 asserts | 29 |
| Post-phase complete (claimed + measured) | **2234** |
| Non-Design | 1141 |
| Designs | 1093 |
| Complete-suite groups | 25 |
| Tray standalone | 264 / 0 |
| `machines-m1` registered once under `all` | Yes |
| README totals | Match runtime |
| Dynamic DOM inflating a11y counts | **No** for M1 |

---

## 22. Runtime / `file://` results

All runs used disposable Edge user-data dirs and temp HTML copies (product tree not executed against the user’s real profile).

| Check | Result |
|-------|--------|
| `git diff --check` | Clean |
| HTML parse | OK |
| `?selftest=machines-m1` | **29 / 0**, no page exceptions |
| `?selftest=all` (25 groups) | **2234 / 0** |
| machine setup | 50 / 0 |
| storage | 15 / 0 |
| first-run | 19 / 0 |
| beginner | 22 / 0 |
| empty-state | 60 / 0 |
| help / modal / a11y | 37 / 27 / 36 |
| responsive | 45 / 0 |
| design | 1093 / 0 |
| tray | 264 / 0 |
| Independent high-risk logic suite | **13 / 0** |

---

## 23. Documentation findings

README + CHANGELOG accurately describe:

- `machines` / `activeMachineId` groundwork  
- `legacy-workshop-machine` + blank identity  
- `machineIdentity` compatibility mirror  
- Merge ID-keyed union + local active  
- Replace restoration + legacy absence rule  
- `machineProfile` separation  
- Unchanged historical snapshots  
- No manager/switcher UI  
- Route `machines-m1` and totals **2234 / 29**  
- Unchanged schema/storage/backup identity  

Implementation report’s “paired work area” wording is the only imprecision noted (see POLISH). No over-claim of seamless multi-machine product completion.

---

## 24. Protected-boundary comparison

Confirmed **unchanged** by diff and/or runtime:

`APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, accepted backup validation, `machineProfile` semantics, Reference tables, `productionMachineKeys`, `gridMachineSnapshot`, `activeMachineSnapshot`, historical record schemas, evidence identity, promotion eligibility, Project Wizard recommendations, first-run eligibility core, deletion/accounting/inventory/pricing calculations, Designs geometry/SVG/colors/filenames/downloads/Finished Views/production bytes, network/dependency behavior.

---

## 25. Findings by severity

### BLOCKER

*None.*

### IMPORTANT

*None.*

### POLISH

1. **Implementation report wording:** “paired positive-or-null work area” — storage normalizes dimensions independently; pairing is editor validation. Prefer clarifying in the implementation report (not a storage bug).  
2. **Fixture gap:** empty-local Merge active fallback not named in the 29 asserts (behavior correct; add when convenient).  
3. **Fixture gap:** full `loadState` non-persistence not asserted in-group (helpers-only check + independent verification).  
4. **Fixture restore parity:** optional expansion to modal/focus snapshots like heavier UI fixture groups.

### NOT A DEFECT

- Mirror rebuild drops non-name/work-area fields on `machineIdentity` when an active machine exists — intentional compatibility mirror.  
- Merge still applies preference fields (`machineProfile`, sorts, etc.) via baseline `applyBackupPreferences` — pre-M1 behavior.  
- First save after upgrading a legacy workspace writes `machines` — ordinary persistence, not silent load rewrite.  
- Older apps ignore `machines[]` and only see mirrored identity for the active machine — expected compatibility tradeoff.

---

## 26. Exact final verdict

**APPROVED WITH POLISH DEFERRED**

No blockers or importants. Polish items may ship deferred; they do not require another full audit before commit if only docs/fixture nits are addressed later.

---

## 27. Remaining unverified areas

- Physical laser / LightBurn production jobs (out of M1 scope).  
- Long-lived multi-version backup exchange on real user machines beyond synthetic payloads.  
- Full Import modal click-path for every Merge/Replace edge (helpers used by Import were exercised; modal framing covered by beginner/import fixtures still green).  
- Promotion-switch standalone route not re-run as a separate named route (promotion group inside `all` passed).

---

## 28. Physical laser testing

**Not required** to accept M1 storage/migration groundwork.

---

## 29. Whether M1 may be committed

**Yes.** Working tree is ready for a single focused commit of `index.html`, `README.md`, and `CHANGELOG.md` (plus any intentional docs the author chooses). This audit did not create that commit.

---

## 30. Whether M2 may begin after commit

**Yes**, after M1 is committed. M2 should not reopen M1 storage contracts without cause; manager/selector UI remains deferred to later phases.

---

## 31. Audit hygiene confirmation

- **No product source files were edited** by this audit.  
- **Nothing was staged, committed, or pushed.**  
- Only this audit report was written under `docs/`.  
- Runtime used temporary HTML copies and disposable Edge profiles under `%TEMP%\m1_audit_runtime`.

---

*End of focused audit report.*
