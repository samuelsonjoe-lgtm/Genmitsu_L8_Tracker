# Machine-neutral Setup and Advisory Work Area — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit date:** 2026-07-19  
**Audit mode:** Read-only (no product source edits; no stage/commit/push/reset/clean/stash/checkout)  
**Implementation report reviewed:** `docs/MACHINE_NEUTRAL_SETUP_IMPLEMENTATION_2026-07-19.md`  
**Expected committed baseline:** `ea5465a` — *Add in-app help and laser safety guidance*  
**Expected pre-phase fixture baseline:** 1991 passed / 0 failed  

---

## 1. Repository state and actual baseline

### Commands / observations

| Check | Result |
|--------|--------|
| `git status -sb` | `## main...origin/main`; tracked mods: `CHANGELOG.md`, `README.md`, `index.html`; many pre-existing untracked files |
| `git log -1 --oneline` | `ea5465a Add in-app help and laser safety guidance` |
| `git rev-parse HEAD` | `ea5465ac95e7351b90d48b3fee2241a7c22b106c` |
| `git rev-list --left-right --count origin/main...main` | `0	0` (not ahead, not behind) |
| `git diff --check` | Clean (only CRLF/LF working-copy notice on `CHANGELOG.md`) |
| `git diff --stat` | `CHANGELOG.md` +1; `README.md` ~12 lines; `index.html` +169/−17; **3 files, +165/−17** |
| `git diff --cached` | Empty — **nothing staged** |
| `git ls-files --others --exclude-standard` | Unrelated untracked set intact (`LightBurn Projects/`, `debug.log`, historical `docs/*`, `parametric_qr_stand_generator.py`, implementation report, etc.) |

### Confirmations

- **Actual HEAD** matches expected baseline subject and short hash `ea5465a`.
- **Branch:** `main`, synchronized with `origin/main` (`0/0`).
- **Committed baseline before this phase:** clean tree at `ea5465a` (phase work is uncommitted working-tree only).
- **Modified tracked files only:** `index.html`, `README.md`, `CHANGELOG.md`.
- **Relevant new untracked file for this phase:** `docs/MACHINE_NEUTRAL_SETUP_IMPLEMENTATION_2026-07-19.md` (plus this audit report written after review).
- **Nothing staged.** No commit or push by this audit (or indicated by git state for the phase).
- **Unrelated untracked assets** (LightBurn projects, debug logs, utilities, historical reports) remain present and were not modified by this audit.

### Constants (source + runtime)

| Constant | Value | Unchanged? |
|----------|--------|------------|
| `APP_ID` | `genmitsu-l8-tracker` | Yes |
| `APP_NAME` | `Genmitsu L8 Tracker` | Yes |
| `APP_VERSION` | `0.9.0` | Yes |
| `BUILD_DATE` | `2026-07-19` | Yes |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` | Yes |
| `SCHEMA_VERSION` | `2` | Yes |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` | Yes |

Runtime probe after save: only one `localStorage` key — `genmitsu-l8-tracker-v1`. No second machine key.

### Pre-phase / post-phase fixtures

- Pre-phase complete suite (per implementation report + phase contract): **1991 / 0**.
- This audit re-ran focused groups under headless Edge `file://` (see §26): machine **50/0**, storage **15/0**, first-run **19/0**, help **37/0**, modal **26/0**, production **66/0**, design-production **118/0**, design **1093/0**, tray **264/0**. Zero page errors. Reconciliation **1991 + 50 = 2041** matches README and implementation report complete-suite claim; complete `?selftest=all` was not re-run end-to-end in this session, but the new group and all high-risk sibling groups were executed green, and Designs geometry (1093) was re-verified.

---

## 2. Files and functions reviewed

### Files

- `index.html` (full phase surface: state, normalize, display, persist, backup, merge/replace, Reference UI, save handler, first-run, Help, fixtures, selftest registry)
- `README.md` (machine setup, merge/replace, fixture totals)
- `CHANGELOG.md` (0.9.0 bullet)
- `docs/MACHINE_NEUTRAL_SETUP_IMPLEMENTATION_2026-07-19.md`
- Git diff of the three tracked files vs `HEAD`

### Functions / seams (minimum set + related)

`freshState`, `loadState`, `persist`, `backupObject`, `normalizeMachineIdentity`, `currentMachineDisplayName`, `mergeData`, `replaceData`, `applyBackupPreferences`, `resetBackupPreferences`, `applyBackupImport`, `validateBackupImport`, `decodeStoredState` / storage-recovery fixtures, `renderReference`, `saveMachineSetup`, Reference profile selector binding, `isFirstRunWorkspaceEmpty` / `firstRunOnboardingHtml`, Help content, `activeMachineSnapshot`, `designProductionMachineKey`, `wizardMachineLabel`, Test Grid / Material Test / production evidence paths (via fixtures + provenance probe), Designs production byte neutrality fixture, `runMachineSetupFixtures`, `runStorageRecoveryFixtures`, `runFirstRunOnboardingFixtures`, `runHelpSafetyFixtures`, `runModalAccessibilityFixtures`, selftest registry (`?selftest=machine` / `all`).

Protected Designs builders/serializers/downloads/Finished Views: verified unchanged by byte fixture + design group 1093/0; no production geometry edits in the phase scope beyond incidental fixture mentions.

---

## 3. Implementation summary

Additive root field:

```js
machineIdentity: {
  name: string,                    // default ''
  workAreaWidthMm: number | null,  // positive finite or null
  workAreaHeightMm: number | null
}
```

- Default empty identity; no multi-machine model; no manufacturer/serial/firmware/etc. first-class fields.
- `normalizeMachineIdentity` + load/persist/backup wiring.
- **Merge:** preserves local `machineIdentity` (not applied from backup).
- **Replace:** restores normalized backup identity (legacy → empty default).
- Reference form “Current workshop machine” with advisory copy; Genmitsu 20W/40W selector relabeled as **Genmitsu L8 Reference profile**.
- `currentMachineDisplayName` for display fallback only.
- Provenance workflows remain Genmitsu-canonical via `machineProfile`.
- New fixture group **50** assertions; README/CHANGELOG updated; version/build/schema/storage key unchanged.

---

## 4. Data-model findings

| Expectation | Result |
|-------------|--------|
| Only new persisted field is `machineIdentity` | **Pass** — added to `freshState` / `loadState` / `persist` / `backupObject` only |
| Additive; does not replace/rename `machineProfile` | **Pass** |
| No second machine array / multi-machine architecture | **Pass** |
| No manufacturer/model/serial/firmware/wattage/lens/origin/camera/accessory/rotary first-class fields | **Pass** |
| `SCHEMA_VERSION` remains 2 | **Pass** |
| `STORAGE_KEY` remains `genmitsu-l8-tracker-v1` | **Pass** |
| `APP_VERSION` 0.9.0 / `BUILD_DATE` 2026-07-19 | **Pass** |
| No migration framework / no second localStorage key | **Pass** |

**Verdict:** Contract met. **NOT A DEFECT** for bounded additive model.

---

## 5. machineProfile separation findings

- Field name, allowed values (`20W`/`40W`), default (`20W`), merge via `applyBackupPreferences`, and control of Genmitsu Reference tables remain unchanged.
- Saving machine setup does not alter `machineProfile` (fixture + live save probe).
- Changing the Reference profile select does not alter `machineIdentity` (fixture).
- Work-area dimensions never write `machineProfile`.
- UI explicitly states custom name does not imply Genmitsu; selector labeled “Genmitsu L8 Reference profile”; tables described as manufacturer starting points, not universal settings.
- No evidence that Reference table numeric rows were edited (phase limited to labels/attribution/setup panel).

**Verdict:** Separation correct. **NOT A DEFECT.**

---

## 6. Normalization findings

`normalizeMachineIdentity` (source):

- Side-effect free; new object; stable under re-normalization (fixtures + probes).
- Non-object / null / array / primitive → empty default shape.
- `name`: string kept verbatim (including whitespace and HTML-like text); nullish → `''`; other types → `String(...)`. **Trim is not applied in normalize**; **trim is on the save path** — intentional and verified.
- Dimensions: positive finite numbers or numeric strings via `Number()`; blank, zero, negative, NaN, Infinity, nonnumeric (`400mm`), objects, arrays → `null`.
- Decimals accepted; scientific notation strings (e.g. `4e2` → 400) follow `Number()` — deliberate, finite, safe.
- No default dimensions from `machineProfile` or manufacturer specs.
- Display uses `esc()` for names in form value attribute; HTML-like names not interpreted as markup.

**Browser number inputs:** invalid free text often never reaches JS as a non-empty value; save path still re-validates with `Number` + `isFinite` + `> 0`.

**`min="0"` vs JS `> 0`:** HTML allows typing `0`; submit is rejected by JS with inline error. No partial persist. Minor HTML/JS strictness mismatch only.

**Verdict:** Normalization correct and safe. Polish only for `min` attribute (see §29).

---

## 7. Unknown-field preservation findings

- Spreads `...source` then overrides known keys — matches project forward-compat convention; known fields win over malformed source.
- Unknown fields survive (fixture `futureField`); stable on re-normalize.
- `backupObject()` emits `normalizeMachineIdentity(state.machineIdentity)`, so unknown additive keys round-trip if present on live state.
- Prototype-pollution keys (`__proto__`, `constructor`, `prototype`): same pattern as other spread-preserving normalizers in this app; JSON load + known-key override does not create a credible exploit path for this local tracker. **Theoretical, not inflated to a defect.**

**Verdict:** **NOT A DEFECT.**

---

## 8. Legacy-state and backup-compatibility findings

| Scenario | Result |
|----------|--------|
| Stored state missing `machineIdentity` | Loads empty default via `normalizeMachineIdentity` |
| Legacy backup missing field | Accepted; Replace → empty default; Merge → local preserved |
| Legacy + only `machineProfile` | Profile behavior unchanged; identity defaults |
| Malformed identity | Normalized; does not block rest of valid import |
| Unknown top-level backup fields | Existing ignore/accept behavior retained |
| Future schema / foreign app / unsupported format | Still refused before merge/replace; fixtures green |
| Corrupt-storage recovery | Architecture unchanged; storage fixtures 15/0 |

**Verdict:** Compatible. **NOT A DEFECT.**

---

## 9. Backup-object findings

`backupObject()` adds `machineIdentity` once, normalized, after `machineProfile`. Metadata (`app`, `appVersion`, `backupFormat`, `schemaVersion`) unchanged. Existing arrays/preferences preserved. No UI/transient state. No alternate machine key. Plain JSON data only. Export filename/MIME/photo/size paths untouched by this phase.

**Verdict:** **NOT A DEFECT.**

---

## 10. Merge findings

- `mergeData` does not assign `machineIdentity` (explicit comment).
- `applyBackupPreferences` does **not** list `machineIdentity` (so neither merge nor replace preference path overwrites identity except Replace’s explicit assignment).
- Records still merge; `machineProfile` still merges as before.
- Real-seam fixtures: local identity preserved; incoming entry merged; legacy merge preserves local.
- Rejected import leaves identity untouched.
- Merge summary messages list record stats only — no false claim that machine setup was imported.
- README documents Merge keeps local machine setup.

**Empty local + non-empty incoming on Merge:** local remains empty (documented intentional “local workshop configuration” rule). Surprising only if users expect machine setup to behave like `machineProfile`; the product states the opposite honestly. **Do not change the rule for symmetry alone.**

**Verdict:** Contract correct. Empty-local edge case = **NOT A DEFECT** (deliberate).

---

## 11. Replace findings

- `replaceData` sets `state.machineIdentity = normalizeMachineIdentity(data.machineIdentity)` **before** `resetBackupPreferences()` / `applyBackupPreferences()`.
- Preference reset/apply do not touch identity → no overwrite of restored identity.
- Current-format Replace restores name + both dimensions (fixture + probe).
- Legacy Replace clears prior local identity to empty default.
- Malformed values normalize safely; recovery overwrite safeguards unchanged.
- Ordering is correct; no stale local identity retained after successful Replace.

**Verdict:** **NOT A DEFECT.**

---

## 12. Rejected-import findings

Future-schema, foreign-app, and foreign-format refusals occur in `validateBackupImport` before `mergeData`/`replaceData`. Fixtures assert no state mutation, no storage mutation, identity preserved. User can pick another file (existing import UX).

**Verdict:** **NOT A DEFECT.**

---

## 13. Reference setup-form findings

| Requirement | Result |
|-------------|--------|
| Optional name, maxlength 120, trim on save | Pass |
| No invented default name | Pass |
| Width/height labeled mm; usable work area (not footprint) | Pass |
| Both dimensions required or both blank | Pass |
| Partial entry rejected; prior dims unchanged | Pass (real form fixture) |
| Zero / negative / nonfinite rejected | Pass |
| Decimals accepted | Pass |
| Blank pair clears dims, keeps name | Pass |
| Name alone valid | Pass |
| Saves only `machineIdentity`; not `machineProfile`; no new records | Pass |
| Uses `persist()`; failed persist returns false without success toast path | Pass (structure) |
| Inline `role="alert"` errors; success toast | Pass |
| Re-render after save | Pass |
| Long/HTML-like names escaped in value attribute | Pass |
| JS enforces `> 0` despite `min="0"` | Pass (behavior) |

**Verdict:** Form correct. **POLISH:** align `min` with `> 0` if desired (deferrable).

---

## 14. Current-machine display-helper findings

`currentMachineDisplayName`:

- Trimmed custom name when non-empty.
- Blank name → `Genmitsu L8 20W` / `40W` from `machineProfile`.
- Does not mutate state or write fallback into `machineIdentity`.
- Safe with missing/malformed sources (defaults to Genmitsu 20W-style fallback when profile not `40W`).
- Does not rewrite records or reinterpret work area.

**Verdict:** **NOT A DEFECT.**

---

## 15. Machine-provenance findings

Independently checked via source + fixtures + live probe:

| Workflow | Behavior |
|----------|----------|
| Test Grids / `activeMachineSnapshot` | Still Genmitsu keys/labels from `machineProfile` |
| Material tests / production evidence / promotion | Unchanged canonical keys (`genmitsu-l8-20w` / `40w`) |
| Designs production machine key / wizard label | Unchanged; still profile-based |
| Designs-to-Projects handoff | Production fixtures green; no new machine fields injected |
| Custom `machineIdentity.name` | Does **not** leak into Test Grid snapshot (fixture) |
| Changing identity | Does not rewrite existing grids/records (fixture) |

**Honesty assessment:** Leaving provenance defaults on Genmitsu `machineProfile` while allowing a non-Genmitsu display name is **intentional and in-scope**. UI + Help state that custom setup is advisory and does not imply Genmitsu, and that Reference tables are Genmitsu-specific. Residual risk: a user with a custom name may still create new evidence labeled “Genmitsu L8 20W” unless they manually pick another machine control where available. That is a **documented product boundary**, not an accidental rewrite of historical records. Classified as **NOT A DEFECT** for this phase (would require a separate provenance redesign out of scope). Optional future clarity is polish only.

**Verdict:** No provenance regression; no false rewrite of records.

---

## 16. Reference-attribution findings

- Toolbar: tables are bundled Genmitsu L8 manufacturer starting points, not universal settings.
- Selector: “Genmitsu L8 Reference profile” with “Genmitsu L8 20W/40W” options.
- Table title: `Genmitsu L8 ${profile} official starting points`.
- “Official values are starting points” panel retained.
- Setup copy: custom name does not imply Genmitsu; does not change Reference profile.

**Verdict:** Attribution improved, not broken. **NOT A DEFECT.**

---

## 17. First-run findings

- `isFirstRunWorkspaceEmpty` still counts only records/inventory — **not** `machineIdentity`.
- Populated identity does not suppress onboarding (fixture).
- One list-item wording updated to mention optional current-machine setup; session dismissal still unpersisted.
- First-run fixtures **19/0**.

**Verdict:** **NOT A DEFECT.**

---

## 18. Help findings

- Additive paragraph under starting points: advisory machine name/work area; no resize/nest/bed-fit/control; Genmitsu-specific tables.
- Help fixtures **37/0**; modal a11y still intact.
- No form duplication; no Help persistence changes.

**Verdict:** **NOT A DEFECT.**

---

## 19. Work-area honesty findings

- Explicitly advisory; no resize, nesting, bed-fit guarantee, machine control.
- Designs work-area banner **intentionally deferred** — fixture proves SVG bytes identical with empty vs populated identity; design suite **1093/0**.
- No misleading automated bed-fit claims found in new UI.

**Verdict:** Deferred Designs banner = **NOT A DEFECT** (bounded phase). Optional future UI = **POLISH**.

---

## 20. Accessibility and responsive findings

- Native labels, submit button, `role="alert"`, `aria-labelledby` on form.
- No new modal/focus trap for setup.
- Narrow (320 px) probe: form present, no horizontal overflow, footer visible.
- Duplicate-ID fixture on Reference after setup panel: pass.
- Modal fixtures **26/0**.

**Verdict:** **NOT A DEFECT.**

---

## 21. Persistence/security findings

- Single storage key; schema 2; no migration.
- Persist path includes `machineIdentity` with recovery overwrite guard unchanged.
- Names escaped at HTML render boundary; no `innerHTML` of raw name without `esc`.
- No new network/dependency.
- Console: pre-existing SVG `height="auto"` noise only; no page errors during audit run.

**Verdict:** **NOT A DEFECT.**

---

## 22. Machine-fixture quality findings

`runMachineSetupFixtures` (50 assertions):

- Uses real Reference form submit for save/partial/clear/profile independence.
- Uses real `applyBackupImport` for merge/replace/reject.
- Includes sibling-group health checks (storage, first-run, help, modal) without weakening them.
- Self-restoring: state, localStorage, first-run dismiss flag, tab, help modal DOM/open/aria, final `render()`.
- Registered as `?selftest=machine` and in `selftest=all` **once**.

**Verdict:** Strong fixtures; not bypassing import/form seams. **NOT A DEFECT.**

---

## 23. Fixture-cleanup findings

Outer `restore()` plus nested fixture restores. Machine group **50/0** with clean restore. No durable pollution observed in probes after runs.

**Verdict:** **NOT A DEFECT.**

---

## 24. Storage-fixture depth findings

Storage recovery group still **15/0** including refuse-without-mutation, legacy accept, future/foreign refuse, merge/replace record behavior. Machine group re-invokes storage fixtures as a sibling health check.

**Verdict:** No storage-recovery regression.

---

## 25. Fixture-total reconciliation

| Item | Count |
|------|--------|
| Pre-phase complete baseline | 1991 |
| New machine setup assertions | +50 |
| Expected complete suite | **2041** |
| This audit: machine | 50/0 |
| This audit: design geometry | 1093/0 |
| This audit: storage / first-run / help / modal | 15+19+37+26 all /0 |
| This audit: production / design-production | 66+118 /0 |
| Tray (callable subset; not double-counted in complete suite per report) | 264/0 |

Arithmetic **1991 + 50 = 2041** holds. README claims 2041 and lists machine 50 — consistent with implementation report and this audit’s machine count.

---

## 26. Runtime / file:// validation

- Page loads `file:///C:/Genmitsu%20L8%20Tracker/index.html` to `readyState === 'complete'`, correct title, release footer, Help present.
- Live probes: normalize matrix, display helper, merge empty-local, replace restore, provenance non-leak, form labels/advisory/mm/alert, save, constants, single storage key, Help wording, Designs SVG length equality, Reference attribution, 320 px layout.
- No `_page_errors`. Console only known SVG height noise.

---

## 27. README / CHANGELOG findings

- README: Reference machine setup, advisory limits, Merge preserves / Replace restores, fixture total 2041, `runMachineSetupFixtures` registered.
- CHANGELOG 0.9.0: accurate single bullet; no multi-machine or “paid-release-complete” claim; version remains 0.9.0.

**Verdict:** Documentation honest. **NOT A DEFECT.**

---

## 28. Protected-boundary comparison

| Boundary | Diff / runtime |
|----------|----------------|
| `STORAGE_KEY` / `SCHEMA_VERSION` values | Unchanged |
| `APP_VERSION` / `BUILD_DATE` / `APP_ID` / `BACKUP_FORMAT` | Unchanged |
| `machineProfile` meaning | Unchanged (labels only) |
| Reference numeric values | Not edited |
| Library promotion/evidence semantics | Untouched (production fixtures green) |
| Test Grid history / project accounting / inventory / pricing | Not in phase diff scope |
| Designs geometry / SVG serialization / downloads / Finished Views | Byte-neutral fixture + 1093/0 |
| Import future/app/format guards | Unchanged, fixture-proven |
| Corruption recovery architecture | Unchanged |
| First-run dismissal persistence | Still session-only |
| Help/modal a11y infrastructure | Unchanged behavior; wording additive |
| Network/dependency | None added |

`activeMachineSnapshot` / Designs builders appear in fixture **call sites** only; provenance implementation bodies remain Genmitsu-profile-based.

---

## 29. Findings classified by severity

### BLOCKER

*None.*

### IMPORTANT

*None confirmed.*

### POLISH (safe to defer)

1. **HTML `min="0"` vs JS `> 0`** on work-area inputs — browser may accept zero until submit; JS correctly rejects. Optional: set `min` to a positive epsilon or drop min and rely on JS.
2. **Deferred Designs “configured work area” advisory line** — intentional; add later without touching production bytes if a safe layout metric exists.
3. **Optional future provenance UX** — when a custom workshop name is set, additional reminders on Test Grid/Material Test default Genmitsu labels (out of this phase’s contract).
4. **`Number()` acceptance of scientific notation / hex-like numeric strings** — deliberate JS conversion; invalid units like `400mm` already null. Document or tighten only if product wants stricter parsing.

### NOT A DEFECT (deliberate / acceptable)

- Merge never applies incoming `machineIdentity`, including when local is empty and backup has values.
- Custom name does not rewrite Genmitsu-canonical evidence keys/labels.
- Name whitespace preserved by normalize; trimmed only on save.
- Unknown additive fields inside `machineIdentity` preserved.
- No Designs production output change; no schema bump; no second storage key.
- Theoretical prototype-key concerns without credible exploit path.

---

## 30. Exact final verdict

# APPROVED WITH POLISH DEFERRED

---

## 31. Remaining unverified areas

- Full `?selftest=all` single-shot 20-group run was not re-executed in this audit session (high-risk groups + design 1093 + machine 50 were). Implementation report’s 2041/0 is consistent with arithmetic and partial re-run.
- OS-level Export file-picker / download dialog not exercised (Blob path unchanged).
- Every non-machine production sub-path not line-audited beyond fixtures and provenance probe.
- Physical laser / bed-fit behavior not applicable to software-only advisory storage.

---

## 32. Whether physical laser testing is required

**No** for this phase. Changes are advisory setup, backup identity, and UI attribution only — no motion control, no cut/engrave parameter generation changes, no geometry changes.

---

## 33. Whether Codex may proceed to commit

**Yes.** Codex may commit the three tracked files (`index.html`, `README.md`, `CHANGELOG.md`) plus any intentional documentation the user chooses (implementation report / this audit are untracked docs). No correction commit is required before commit. Polish items may ship later.

No second independent audit is **required** solely for the deferred polish list. A re-audit is only needed if the commit set expands beyond the reviewed phase or if merge/replace/normalize behavior is later changed.

---

## 34. Audit hygiene confirmation

This audit:

- Made **no** edits to product source (`index.html`, `README.md`, `CHANGELOG.md` left as found).
- Did **not** stage, commit, push, reset, clean, stash, checkout, move, rename, or delete any file.
- Wrote **only** this audit report at  
  `C:\Genmitsu L8 Tracker\docs\MACHINE_NEUTRAL_SETUP_FOCUSED_AUDIT_2026-07-19.md`.
- Used temporary out-of-repo / worktree agent tooling for Playwright probes only; product tree git state for the three phase files remains uncommitted working-tree modifications as audited.

---

*End of focused audit report.*
