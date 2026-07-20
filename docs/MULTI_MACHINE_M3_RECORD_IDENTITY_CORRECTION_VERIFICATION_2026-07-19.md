# M3 Record Identity — Correction Re-verification

**Date:** 2026-07-19  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Review type:** Narrow read-only re-verification of bounded M3 corrections  
**Scope:** Focused-audit findings **M3-I1** (Test Grid wording) and **M3-I2** (expanded high-risk fixtures only)  
**Committed baseline:** `7a9ec6d` — *Add multi-machine manager and switching*  
**Uncommitted phase:** M3 — Owned-machine Identity for New Production-critical Records (plus correction pass)

---

## Final verdict

# **APPROVED TO COMMIT**

| Severity | Count |
|----------|------:|
| BLOCKER | **0** |
| IMPORTANT | **0** |
| POLISH | **0** |
| NOT A DEFECT | noted below |

Both focused-audit findings are corrected. M3 identity semantics are preserved. M1/M2 and the complete suite are green at the expected totals. A second full Claude architecture audit is **not** required for these corrections.

**M3 may be committed.**  
**Gift Box G1 may begin after M3 is committed** (architecture already reviewed; do not mix Gift Box work into the M3 commit).

**Physical laser testing:** **Not required** for this correction pass (wording + offline fixture coverage only).

---

## 1. Repository state

| Check | Result |
|-------|--------|
| `git status -sb` | `## main...origin/main`; modified: `CHANGELOG.md`, `README.md`, `index.html` |
| `git log -1 --oneline` | `7a9ec6d Add multi-machine manager and switching` |
| `git rev-parse HEAD` | `7a9ec6d65755ab4a17d24b530c94ec3a9800d15e` |
| `origin/main...main` left-right | `0 0` |
| `git diff --check` | Clean (CRLF warnings only on touch; no conflict markers) |
| `git diff --stat` | `CHANGELOG.md` +1; `README.md` ~10 lines; `index.html` ~+141/−34 |
| `git diff --cached` | Empty — **nothing staged** |
| Branch | `main` |
| M3 commit/push | **None** — still uncommitted working tree |
| HEAD remains `7a9ec6d` | **Yes** |

### Tracked modifications (intended M3 + correction)

| File | Role |
|------|------|
| `index.html` | M3 identity implementation + correction (wording + expanded fixtures/cleanup) |
| `README.md` | Workshop-machine wording + suite totals |
| `CHANGELOG.md` | M3 user-facing identity note (no separate internal-correction entry) |

### Relevant docs present (untracked reports)

- `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_IMPLEMENTATION_2026-07-19.md`
- `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_FOCUSED_AUDIT_2026-07-19.md`
- `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_CORRECTION_2026-07-19.md`
- `docs/DESIGNS_GIFT_BOX_ARCHITECTURE_REVIEW_2026-07-19.md`

### Unrelated untracked material

Historical docs, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, etc. — **not part of M3** and **not modified by this verification**.

---

## 2. Files reviewed

| Artifact | Purpose |
|----------|---------|
| `index.html` | `gridMachineControls`, M3 helpers, `runMachineRecordIdentityFixtures`, selftest registry, identity placement call sites |
| `README.md` | Wording + totals |
| `CHANGELOG.md` | M3 entry accuracy |
| M3 implementation / focused audit / correction reports | Expected claims vs runtime |
| Gift Box architecture report | Present; out of M3 scope |
| Live `file://` Edge/Playwright | `?selftest=machines-m3`, `machines-m2`, `machines-m1`, full 27-group sum, standalones |

---

## 3. Wording verification (M3-I1)

### Source (`gridMachineControls`)

```11903:11905:C:\Genmitsu L8 Tracker\index.html
      const options = isNew
        ? `<option value="active" selected>Use active workshop machine</option>`
        : `<option value="" ${!recorded ? 'selected' : ''}>Leave not recorded</option><option value="keep" ${recorded ? 'selected' : ''}>Keep recorded machine</option><option value="active">Assign active workshop machine</option>`;
```

| Requirement | Status |
|-------------|--------|
| New Grid option text exactly **Use active workshop machine** | **Pass** |
| Existing Grid option text exactly **Assign active workshop machine** | **Pass** |
| Old ambiguous strings absent from this control | **Pass** (only workshop forms in `gridMachineControls`) |
| Option **values** unchanged (`active`, `keep`, `""`, `20W`, `40W`, `custom`) | **Pass** |
| `gridMachineFromForm` selection behavior unchanged | **Pass** (no correction-era change to action branches) |
| Active owned-machine preference via `activeMachineSnapshot()` | **Pass** (still prefers owned snapshot when present) |
| Legacy Reference-profile fallback | **Pass** (`activeOwnedMachineSnapshot() \|\| activeMachineSnapshot(state.machineProfile)`) |
| `machineProfile` separate from `activeMachineId` | **Pass** |

### Live DOM

`runMachineRecordIdentityFixtures` assertion **“Test Grid choices explicitly identify the workshop machine”** opens real new and edit Grid forms via `openGridForm()` / `openGridForm({…})`, reads `#gridForm select[name=machineAction]` textContent, requires both workshop strings, and rejects the old non-workshop phrasing. **Pass at runtime (27/0 group).**

Note: An older fixture **name** in Test Grid machine-identity tests still says “Assign active machine updates only the old Grid” — that is assertion **metadata**, not the user-facing control string. **NOT A DEFECT.**

---

## 4. Fixture assertion inventory (M3-I2)

**Independent recount:** **27** `add(...)` calls in `runMachineRecordIdentityFixtures()` — matches runtime **27 passed / 0 failed**.

| # | Assertion name | Behavioral? | Covers requested risk |
|--:|----------------|:-----------:|------------------------|
| 1 | Active owned snapshot is stable, custom, and ID-bearing | Yes | Baseline snapshot |
| 2 | Blank owned name uses the unnamed workshop fallback | Yes | Label fallback |
| 3 | Missing active owned machine falls back to legacy snapshot without an ID | Yes | Reference fallback |
| 4 | Test Grid choices explicitly identify the workshop machine | Yes (DOM) | M3-I1 wording |
| 5 | Grid snapshots retain owned identity and grid-derived tests inherit it | Yes | Grid + derived MT |
| 6 | Legacy grid snapshots remain unchanged and unowned | Yes | No backfill |
| 7 | Standalone Material Test snapshots the active owned machine | Yes | MT placement |
| 8 | Production settings preserve optional machineId | Yes | Setting placement |
| 9 | Promoted evidence inherits source identity without resnapshotting | Yes | Evidence placement |
| 10 | **Candidate ID with target without ID is not an exact match** | Yes | One-sided ID #1 |
| 11 | **Candidate without ID with target ID is not an exact match** | Yes | One-sided ID #2 |
| 12 | **Same label and key with different IDs is not an exact match** | Yes | Same-name / different ID |
| 13 | **Same ID with renamed labels remains an exact match** | Yes | Renamed label match |
| 14 | **Promotion-time active-machine switching preserves frozen source identity…** | Yes | Switch A→B freeze |
| 15 | Project settings snapshot copies source identity | Yes | Project normalize |
| 16 | **Project hidden machine fields round-trip through the real edit/save path** | Yes (DOM submit) | Hidden-field RT |
| 17 | **Reference Copy remains an attributed, unowned Library draft** | Yes (`saveReference`) | Reference Copy |
| 18–22 | Exact Grid / MT / Setting / Evidence / Project references detected | Yes | Deletion scanner roots |
| 23 | Referenced non-active machine can be archived without rewriting records | Yes | Archive allowed |
| 24 | Label-only snapshots without machineId are not references | Yes | Label-only non-block |
| 25 | Permanent deletion remains blocked for a referenced machine | Yes | Delete blocked |
| 26 | Unknown machine IDs survive normalization without machine fabrication | Yes | Unknown ID retain |
| 27 | Historical label/key records receive no backfilled machineId | Yes | No historical backfill |

**Conclusion:** New assertions are **not** mere source-string greps. They call real matchers, promotion save, Project form submit, Reference Copy, archive/delete confirmations, and DOM Grid wording.

---

## 5. Promotion-time switching verification

Fixture flow (live pass):

1. Build candidate from Material Test with **Machine A** (`m3-a`).
2. `setActiveWorkshopMachine('m3-b')` so active becomes **Machine B**.
3. `savePromotionTransaction(switchSession, …)` creates setting + evidence.

**Observed (assertion pass):** candidate, created setting, and evidence remain **`m3-a` / Machine A**; Machine B is not substituted.

---

## 6. Project hidden-field round-trip verification

Fixture flow (live pass):

1. `normalizeProject` with `settingsMachineId/Key/Label` → project.settings.machine.
2. `openProject` → real form; hidden fields read as `m3-a|custom|Machine A`.
3. Unrelated `notes` edit; real form `submit`.
4. Saved project keeps machine identity; `state.activeMachineId` remains **`m3-b`** (active does not overwrite).

---

## 7. Reference Copy verification

Fixture flow (live pass):

1. `saveReference('20W:…')` real handler.
2. Notes include **Official SainSmart 20W starting point**.
3. `profileDraftTests` have **no** `machineId`.
4. No production settings draft / evidence draft created.
5. Not represented as physically proven production.

---

## 8. Deletion-protection verification

| Check | Result |
|-------|--------|
| Grid `machineId` reference detected | Pass |
| Material Test reference detected | Pass |
| Production setting reference detected | Pass |
| Evidence reference detected | Pass |
| Project settings reference detected | Pass |
| Matching label without `machineId` not a reference | Pass |
| Referenced non-active machine may be archived | Pass (records byte-stable) |
| Permanent deletion blocked | Pass |
| No rewrite/removal of referenced records on archive attempt | Pass |

---

## 9. Fixture cleanup

`runMachineRecordIdentityFixtures` restore captures and restores:

- Full `state` (machines, activeMachineId, profiles/tests/settings/evidence, grids, projects, entries, inventory, …)
- `localStorage` (`STORAGE_KEY`, including absent→remove)
- `activeTab`
- Profile/production/evidence drafts; project photo/line drafts
- Pending import / importApplying / first-run dismissal
- Designs draft + preview mode
- Machine-manager search/archived filters + machine modal mode fields
- Modal HTML / open / aria-hidden for machine, grid, profile, production-setting, project modals
- Modal focus origins + open order
- Focus + `render()`

Runtime residual probe after M3 run: group returns cleanly with **0 failed**; synthetic `m3-*` IDs are confined to the try block and cleared by restore (no post-group failures in subsequent suite groups).

---

## 10. M3 semantics comparison (correction did not regress)

Identity core remains:

```7530:7534:C:\Genmitsu L8 Tracker\index.html
    function productionMachineIdentityMatches(candidateMachine, targetMachine) {
      const candidate = candidateMachine || {}, target = targetMachine || {};
      const candidateId = normalizeOptionalMachineId(candidate.machineId), targetId = normalizeOptionalMachineId(target.machineId);
      if (candidateId || targetId) return !!candidateId && !!targetId && candidateId === targetId;
      return !!candidate.key && !!target.key && candidate.key === target.key;
    }
```

| Semantic | Status |
|----------|--------|
| `normalizeOptionalMachineId()` | Present; trim-only optional ID |
| `activeOwnedMachineSnapshot()` | Present; owned ID + custom + display label |
| `activeMachineSnapshot()` | Owned first, else Reference profile path |
| `productionMachineIdentityMatches()` | Exact ID when either side has ID; else key |
| Grid / MT / setting / evidence / Project `machineId` placement | Exercised by fixtures |
| Promotion eligibility / one-sided-ID rejection / exact-ID / legacy key-only | Pass |
| Reference Copy attribution / unowned | Pass |
| Deletion scanner | Pass |
| Import/Merge/Replace / legacy no-ID preservation | Covered by M1 + M3 unknown/legacy asserts |
| Correction-only surface | Wording strings + fixture body/cleanup — **no identity truth-table change intended or observed** |

---

## 11. M1 / M2 regression

| Route / group | Expected | Live |
|---------------|----------|------|
| `?selftest=machines-m1` / `runMultiMachineStorageFixtures` | 29 / 0 | **29 / 0** |
| `?selftest=machines-m2` / `runMachineManagerFixtures` | 31 / 0 | **31 / 0** |
| Machine compatibility (`runMachineSetupFixtures`) | 50 / 0 | **50 / 0** |
| Modal accessibility | 28 / 0 | **28 / 0** |

M2 group continues to cover header switching, dirty-form guard, Add/Edit/Duplicate, Archive/Restore/Delete, Merge/Replace, and reference scanning (prior M2 verification; re-run green here).

---

## 12. Protected-boundary comparison

| Boundary | Unchanged vs baseline constants / role |
|----------|----------------------------------------|
| `APP_ID` | `genmitsu-l8-tracker` |
| `APP_NAME` | `Genmitsu L8 Tracker` |
| `APP_VERSION` | `0.9.0` |
| `BUILD_DATE` | `2026-07-19` |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` |
| `SCHEMA_VERSION` | `2` |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` |
| Project accounting / Inventory / Pricing | Not in M3 correction surface; suite groups green |
| Designs builders / geometry / kerf / SVG / Finished Views | Designs **1093 / 0**; no M3 Designs edits |
| Offline / no-network `file://` | Confirmed |

Pre-existing browser SVG `height="auto"` console warnings remain on broad runs — **NOT A DEFECT** for M3; no page exceptions.

---

## 13. Fixture arithmetic (independently reconciled)

| Suite | Expected | Live |
|-------|----------|------|
| M3 focused | 27 / 0 | **27 / 0** |
| M2 | 31 / 0 | **31 / 0** |
| M1 | 29 / 0 | **29 / 0** |
| Machine compatibility | 50 / 0 | **50 / 0** |
| Modal accessibility | 28 / 0 | **28 / 0** |
| Non-Design | 1200 | **1200** (`2293 − 1093`) |
| Designs | 1093 / 0 | **1093 / 0** |
| Complete | 2293 / 0 | **2293 / 0** |
| Complete-suite groups | 27 | **27** (all present, none missing) |
| Tray standalone | 264 / 0 | **264 / 0** |
| Promotion-switch standalone | 16 / 0 | **16 / 0** |
| `machines-m3` registration under `all` | once | **Exactly one** `if (selftest === 'machines-m3' \|\| selftest === 'all')` |
| README / implementation report totals | 27 / 2293 | **Match runtime** |

`1200 + 1093 = 2293` holds.

Standalone Tray and promotion-switch are **not** in the 27-group complete sum (correct).

---

## 14. Direct `file://` results

Environment: headless Microsoft Edge via Playwright; disposable contexts; URL `file:///C:/Genmitsu%20L8%20Tracker/index.html`.

| Run | Result |
|-----|--------|
| Startup `?selftest=machines-m3` | Console **27 passed / 0 failed**; no page exceptions |
| Re-invoke `runMachineRecordIdentityFixtures` | **27 / 0**; all 27 named results `pass: true` |
| `?selftest=machines-m2` | M2 green; re-invoke **31 / 0** |
| `?selftest=machines-m1` | **29 / 0** |
| Full 27-group direct sum (complete-suite arithmetic) | **2293 / 0** |
| Tray + promotion-switch | **264 / 0**, **16 / 0** |
| `git diff --check` | Clean |
| `python -m html.parser index.html` | Exit **0** |

Groups exercised in the 2293 sum include storage/recovery, import safety (within storage/machine groups), first-run, browsers, production, evidence/promotion, grid machine identity, project wizard, Designs geometry, accessibility/modal/responsive/empty/beginner, M1/M2/M3, etc. — **0 failed**.

---

## 15. Documentation results

| Doc | Status |
|-----|--------|
| README explicit **workshop machine** wording | **Pass** |
| README **27 / 0** M3 and **2293 / 0** complete | **Pass** |
| M3 implementation report correction note + final totals | **Pass** |
| Correction report matches runtime | **Pass** |
| CHANGELOG M3 identity note; no separate internal-correction entry | **Pass** (appropriate) |

---

## 16. Findings by severity

### BLOCKER

*None.*

### IMPORTANT

*None.* M3-I1 and M3-I2 from the focused audit are closed.

### POLISH

*None required for commit.* Optional later: refresh historical Test Grid identity docs that still use pre-workshop phrasing in narrative examples (outside M3 correction scope).

### NOT A DEFECT

| Item | Reason |
|------|--------|
| SVG `height="auto"` console warnings | Pre-existing; no fixture/page failure |
| Fixture name “Assign active machine updates…” | Metadata only; not UI control text |
| Full M3 still uncommitted with M1/M2 baseline | Expected; this verification approves the working tree for commit |
| Gift Box report untracked | Separate architecture deliverable |

---

## 17. Exact final verdict

**APPROVED TO COMMIT**

---

## 18. Remaining unverified areas

| Area | Note |
|------|------|
| Interactive human click-through of all 13 disposable UI steps outside fixtures | Equivalents covered by behavioral fixtures + Edge `file://`; not a separate manual workshop session |
| Production SVG byte golden vs pre-M3 commit for every design template | Designs suite **1093 / 0** and constants unchanged; no Designs edits in M3 |
| Network / multi-browser matrix | Out of scope; offline Edge sufficient |
| Physical laser cut of any M3 record | Not applicable to identity/wording/fixtures |

---

## 19. Physical laser testing

**Not required** for correction verification.

---

## 20. Whether M3 may be committed

**Yes.** Commit the intended M3 tracked files (`index.html`, `README.md`, `CHANGELOG.md`) plus any intentionally included M3 docs if desired. Do not stage unrelated untracked LightBurn projects or historical audits unless Joe wants them.

---

## 21. Whether Gift Box G1 may begin after commit

**Yes**, after M3 is committed on a clean baseline. Keep Gift Box implementation on a separate change set from M3.

---

## 22. Confirmation — this verification mutated nothing product-side

| Action | Performed? |
|--------|------------|
| Edit product sources | **No** |
| Stage / commit / push | **No** |
| Reset / clean / stash / checkout / move / rename / delete | **No** |
| Write this verification report only | **Yes** → `docs/MULTI_MACHINE_M3_RECORD_IDENTITY_CORRECTION_VERIFICATION_2026-07-19.md` |

---

*End of M3 correction re-verification.*
