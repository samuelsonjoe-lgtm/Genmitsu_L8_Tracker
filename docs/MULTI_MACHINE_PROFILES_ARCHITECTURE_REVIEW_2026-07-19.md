# Multi-machine Profiles and Seamless Machine Switching — Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer:** Claude Opus 4.8
**Type:** read-only architecture review. No file was edited, staged, committed, pushed, or otherwise changed. Nothing was implemented.

---

## 1. Repository state and synchronized baseline

- `git status -sb`: `## main...origin/main`; tracked tree **clean**; only the long-standing untracked set present (`LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, historical `docs/*.md`) — preserved exactly.
- HEAD: **`288d474cacf9556638ded27600925aa3d7848160`** — "Improve beginner clarity and import safety" (matches baseline `288d474`).
- Branch `main`; ahead/behind origin/main `0 0`. `git diff --cached` empty (nothing staged); `git diff --check` clean.
- Constants: `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`, `SCHEMA_VERSION=2`, `STORAGE_KEY='genmitsu-l8-tracker-v1'`.
- **Fixture baseline:** taken as stated (**2205 / 0**). A live re-run was attempted via headless Edge but the environment blocked the scratch-profile cleanup command; because this is a read-only architecture review (no implementation), the baseline is accepted as given and all findings are grounded in direct source inspection. (At the prior `288d474`-adjacent baseline this reviewer independently confirmed the harness structure summed to 2182 across the 23 `selftest=all` groups; the +23 delta to 2205 is consistent with the two beginner-clarity/import-safety commits.)

## 2. Review method

Direct source inspection of every machine-related field, helper, renderer, and fixture in `index.html`, plus README/Help. No new machine behavior was executed; the review characterizes the *existing* data model and derives the safest additive path. The single most consequential discovery drives the whole recommendation: **there are two independent machine systems today, and record snapshots are keyed to the Genmitsu Reference profile — not to the user's "current workshop machine."**

## 3. Current machine architecture inventory

Two parallel, deliberately-separate systems exist:

**System 1 — Reference profile + record-snapshot layer (`machineProfile`).**
- `machineProfile` (root string `'20W'`/`'40W'`, `freshState` `:510`) selects the bundled Genmitsu 20W/40W Reference tables **and** is the source for the machine stamped onto new records.
- `productionMachineKeys = ['genmitsu-l8-20w','genmitsu-l8-40w','custom']` (`:398`) — a **fixed allowlist**.
- `activeMachineSnapshot(active = state.machineProfile)` (`:7669`) maps `'20W'/'40W'` → `{key:'genmitsu-l8-20w'|...,label}`.
- `gridMachineSnapshot(machine)` (`:7660`) validates a machine object and **returns `null` for any key not in the allowlist** (`:7665`).
- `productionMachineDefaultLabel(key)` (`:6997`); `normalizeProductionSetting` forces `machine.key` to the allowlist or `'custom'` (`:7022,7036-7041`) and keeps `lens`/`focusMethod`.
- Records carrying a machine snapshot: **Test Grids** (`{key,label}` via `normalizeTestGrid`, `:7677-7683`), **Production settings** (`{key,label,lens,focusMethod}`), **Material Tests** (label string + `machineKey`, `:7693`), **Evidence/promotion candidates** (`{key,label}`), and the **Project Wizard**'s label-based match (`projectWizardMachineCompatibility`, alias-aware).
- Grid form choices (`gridMachineFromForm`, `:11460`): keep / active (=`machineProfile`) / 20W / 40W / custom-label. **There is no notion of selecting among several owned machines.**

**System 2 — Current workshop machine (`machineIdentity`), added in the machine-neutral phase.**
- Root singleton `machineIdentity = { name, workAreaWidthMm|null, workAreaHeightMm|null }` (`normalizeMachineIdentity` `:491`; default in `freshState` `:510`; normalized in `loadState` `:542`; persisted `:549-551`; in `backupObject` `:1031`).
- `currentMachineDisplayName(source=state)` (`:504`) returns the trimmed `machineIdentity.name`, else the Genmitsu fallback (`'Genmitsu L8 20W'/'40W'`) derived from `machineProfile`.
- **`machineIdentity` is display/advisory only. It does NOT flow into any record snapshot** — a fixture explicitly asserts "Custom machine name does not leak into Genmitsu Test Grid machine identity" (`:710`). Work-area values are advisory (no fit/nesting/production effect; a fixture asserts Designs bytes are unchanged, `:734`).

## 4. Current persisted relationship table

| Object / collection | Field | Persisted? | Meaning | Identifies | Machine-specific? | In backup? | Merge | Replace | Legacy fallback | Risk if active machine changes |
|---|---|---|---|---|---|---|---|---|---|---|
| root | `machineProfile` | yes | Genmitsu Reference profile **and** new-record snapshot source | Reference table | n/a | yes | **overwritten** by import (`:11889`) | restored | `'20W'` | new records snapshot the new profile; historical records unaffected |
| root | `machineIdentity` | yes | Current workshop machine (name + advisory work area) | actual machine | display only | yes | **preserved local** (`:11923`) | restored (`:11918`) | empty default | display name changes; no record changes |
| Test Grid | `machine{key,label}` | yes | snapshot at creation | historical snapshot | yes | yes | id-keyed list merge | replaced | dropped if key ∉ allowlist | none (frozen at save) |
| Material Test | `machine`(label)+`machineKey` | yes (nested in profile) | snapshot | historical snapshot | yes | yes | via profile merge | replaced | `''`/`''` | none |
| Production setting | `machine{key,label,lens,focusMethod}` | yes (nested) | proven-for-this-machine | historical + machine-specific | yes | yes | via profile merge | replaced | key→`'custom'` | none; but promotion eligibility is machine-aware |
| Evidence / promotion candidate | `machine{key,label}` | yes (nested) | source machine | historical snapshot | yes | yes | via profile merge | replaced | `'Not recorded'` | none |
| Project | `settings.machine` (optional, from promotion) | yes | snapshot if present | historical snapshot | sometimes | yes | id-keyed merge | replaced | absent | none |
| Log entry | — none — | — | no machine field | — | no | — | — | — | — | — |
| Inventory / Pricing | — none — | — | no machine field | — | no | — | — | — | — | — |
| CSV / backup exports | inherit above | — | snapshots as stored | historical | — | — | — | — | — | — |

## 5. Singleton `machineIdentity` contract

- **Always present after normalization** (`normalizeMachineIdentity()` returns the full shape for any input, incl. `undefined`/malformed); blank fields are valid (`name:''`, dims `null`).
- `currentMachineDisplayName` falls back to Genmitsu wording when the name is blank.
- Fixtures assuming the singleton: the entire `runMachineSetupFixtures` group (`:685-734`) plus a beginner-flow fixture (`:6864`). These assume exactly one `machineIdentity` object with three known keys (plus additive-field survival).
- **No record references `machineIdentity`.** Records only store `machineProfile`-derived `{key,label}` snapshots. Updating the singleton never changes historical display (verified `:711`). Clearing it cannot create dangling references (nothing points to it).
- `machineProfile` is **not** used as a real owned-machine identity — it is the Reference profile that *also* seeds record snapshots. This dual role is the crux of the multi-machine design.

## 6. `machineProfile` separation

`machineProfile` must remain a **root-level Reference-profile preference**, distinct from owned machines. It should **not** become per-owned-machine, and selecting a non-Genmitsu owned machine must **not** auto-change it (a CO₂ owner may still consult Genmitsu tables). The only coupling to sever carefully is its second job — seeding record snapshots — which multi-machine will optionally supersede (see §17/§18). Do not conflate the Reference selector with the active-machine selector in the UI.

## 7. Storage architecture options

| Option | Backward compat | Schema impact | Migration complexity | Backup compat | Merge/Replace | Recovery | Code churn | Lookup safety | Large-format ready | Historical safety | Import collision | Maintainability |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **A** Replace singleton with `machines[]`+`activeMachineId` | Breaks every `machineIdentity` reference/fixture | additive keys but removes one | High (rewrite machineIdentity layer + all its fixtures) | Old backups need conversion; new backups drop `machineIdentity` | Must redefine both | OK | **High** | array + `activeMachineId` | Yes | Good | Must design | Cleaner long-term but risky now |
| **B** Add `machines[]`+`activeMachineId`, keep `machineIdentity` as a compatibility mirror of the active machine | **Full** — all existing machineIdentity code/fixtures keep passing | additive only | **Low** (synthesize on load) | Old backups load unchanged; new backups additive | Reuse machineIdentity precedent | **Best** | **Low** | array + `activeMachineId`, mirror keeps singleton readers working | Yes (add capability fields to `machines`) | Best (records unaffected) | id-keyed, explicit | Highest during transition |
| **C** Nested `workshop:{activeMachineId,machines[]}` | Breaks singleton shape and backup shape | additive but relocates | High | Old backups lack `workshop` | Redefine | OK | High | nested | Yes | Design needed | Clean but unnecessary churn |
| **D** (from source) Extend the **record-snapshot allowlist layer** to carry optional `machineId`, plus B's collection | — | additive | Low–med | additive | reuse | best | Med | id + existing key/label | Yes | Best | explicit | Required regardless of A/B/C |

## 8. Recommended storage architecture

**Option B, combined with D's additive record-snapshot extension.** Concretely:
- Add root `machines: [{ id, name, type, workAreaWidthMm|null, workAreaHeightMm|null, notes, archived, createdDate, updatedDate, …additive }]` and `activeMachineId: string`.
- **Retain `machineIdentity` as a compatibility mirror of the active machine** (kept in sync: editing the active machine updates `machineIdentity.{name,workAreaWidthMm,workAreaHeightMm}`, and vice-versa). This keeps `currentMachineDisplayName`, the Reference machine-setup panel, the backup field, and **every existing machine-setup fixture** working unchanged — the lowest-risk transition.
- **Do not touch `productionMachineKeys`, `gridMachineSnapshot`, or the Reference profile.** Instead, when record snapshots begin carrying an owned-machine reference (Phase M3), add an **optional `machineId`** field *alongside* the existing `{key,label}` snapshot. Existing records (no `machineId`) keep displaying from their `{key,label}`; the allowlist gate is untouched, so no existing grid/setting is ever rejected.

Rejecting A/C: the record-snapshot layer and its fixtures are deeply keyed to `machineProfile`→Genmitsu keys; replacing or relocating the singleton forces immediate rework of that layer and dozens of `genmitsu-l8-20w` assertions for no user benefit. Per the mandate, do not rewrite merely for a cleaner greenfield shape.

## 9. Exact migration contract

Single, idempotent, normalization-time migration in `loadState()` (Phase M1):
1. **Existing `machineIdentity` → first machine.** If `data.machines` is absent/not-an-array: create `machines = [{ id:'legacy-workshop-machine', name: mi.name || '', type:'diode', workAreaWidthMm: mi.workAreaWidthMm, workAreaHeightMm: mi.workAreaHeightMm, notes:'', archived:false, createdDate: today(), updatedDate: today() }]` and `activeMachineId='legacy-workshop-machine'`. The fixed ID makes repeated loads idempotent (never duplicates). Blank identity fields → the machine still exists with `name:''` and null dims (display continues to fall back to Genmitsu wording via `currentMachineDisplayName`). **Recommended default:** create the machine even when the singleton is blank so the manager has an editable entry (a product decision, §31).
2. **`activeMachineId`.** After migration it equals the migrated/first machine. If missing at load → default to the first non-archived machine, else the first machine, else `''` (display falls back to Genmitsu wording). If invalid (points to no machine) → same fallback. If it points to an **archived** machine → keep it selectable but surface an "archived" note; do not auto-switch (avoid silent behavior). An active machine **may be absent** (empty `machines`), in which case display uses the Genmitsu fallback and new-record snapshots keep their current `machineProfile` behavior.
3. **Genmitsu fallback stays display-only** — it is never persisted into a machine's `name`.
4. **`machineIdentity` remains persisted** as the active machine's mirror (compatibility), so old readers and backups keep working.

Migration must run only from `loadState`/normalization, **never from rendering**, and must be idempotent under repeated load and repeated import.

## 10. Schema-version recommendation

**No `SCHEMA_VERSION` increment; normalization-only evolution; no `BACKUP_FORMAT` change; no `STORAGE_KEY` change.** `machines`/`activeMachineId` are additive and forward-compatible: an older `0.9.0` app importing a newer backup reads only known keys and silently drops `machines` (it still has the `machineIdentity` mirror, so the active machine's name/work area survive; only the *extra* machines are lost) — consistent with how `machineIdentity` itself was added at v2 without a bump. **Bumping to 3 would be actively harmful**: the existing future-version import guard (`schemaVersion > SCHEMA_VERSION`) would make current apps **refuse** every new backup, breaking cross-version portability for a purely additive change. Retain compatibility fields (`machineIdentity` mirror) rather than bumping. This is an IMPORTANT decision (§32): the additive-no-bump path is what keeps old backups working; a reflexive bump would be a self-inflicted compatibility break.

## 11. Stable machine-ID strategy

- New machines: `uid()` (existing convention).
- Migrated singleton: the **deterministic fixed ID `legacy-workshop-machine`** (idempotent; repeated loads/imports of the same legacy singleton never create duplicates).
- **Identity is the ID, never the display name.** Same name/model → distinct IDs → distinct machines (correct for two identical units). Renaming preserves the ID and therefore all references.
- Merge collision handling: id-keyed (§13). Two backups each contributing a `legacy-workshop-machine` is the one deterministic-ID collision to handle explicitly (treat as the same machine, keep local on merge).

## 12. Legacy-backup behavior

- **Replace with old backup** (no `machines`): migrate its `machineIdentity` (or blank default) into a single `legacy-workshop-machine`; `activeMachineId` = that; Reference profile restored from `machineProfile`.
- **Merge with old backup**: per §13, `machineIdentity`/machines are workshop-local config → **the local machine list and active selection are preserved**; only records merge.
- Old backup with **blank** `machineIdentity` on Replace → one blank-name machine (display falls back to Genmitsu wording).
- Old backup with **custom** `machineIdentity` on Replace → one named machine.
- Old backup containing **machine snapshots but no `machines` collection** → snapshots remain on their records verbatim (they are self-describing `{key,label}`); no `machines` entries are fabricated from snapshots (avoid inventing owned machines from historical labels).

## 13. New-backup Merge behavior

Treat `machines`/`activeMachineId` as **local workshop configuration** (following the `machineIdentity` precedent, `:11923`), not a mergeable record collection:
- **`activeMachineId` stays local** after Merge.
- Machine list Merge: **id-keyed union that preserves existing entries** — same ID in both → keep local (do not overwrite fields); ID only in import → add it; never silently deduplicate by name/model/work-area; never auto-rename on conflict. Same apparent machine with different IDs → **two entries** (explicit user review beats clever guessing). Archived-state conflicts → keep local. Empty imported `machines` → no change. Legacy imported `machineIdentity` under Merge → ignored for the list (local preserved), consistent with today.
- Evidence/records referencing an imported machine ID that isn't present → display "machine not recorded / unavailable" using the record's own `{key,label}` snapshot (records are self-describing; a missing `machines` entry never breaks a record). **Prefer preserving data and explicit review over automatic dedup.**

## 14. New-backup Replace behavior

- Restore `machines` and `activeMachineId` from the backup; if the backup's `activeMachineId` is invalid/archived → fall back (first non-archived, else first, else `''`).
- Legacy backup → convert per §9/§12. Empty-machine backup → empty list + Genmitsu-fallback display.
- Local machines are **removed** (Replace = restore the backup's workspace), matching existing Replace semantics; keep the existing two-step Replace confirmation. Recovery/confirmation wording should gain one clause noting that Replace also replaces the machine list (product-facing copy, not logic).
- Reference profile (`machineProfile`) restored as today.

## 15. Recovery behavior

Unchanged architecture: malformed `machines` → normalization coerces to `[]` (like `normalizeInventory`'s array guards) and `activeMachineId` to fallback; corrupt whole-storage still routes to the existing recovery panel (raw preserved, saving blocked). No machine history is silently lost — records keep their embedded snapshots regardless of the `machines` list. Migration must tolerate a malformed `machines` array without throwing.

## 16. Historical snapshot contract by record type

| Record | Current | Proposed | Migration? | Display after rename | After archive | After delete/unavailable |
|---|---|---|---|---|---|---|
| Log entries | none | **none** (do not add) | no | n/a | n/a | n/a |
| Material Tests | label+`machineKey` snapshot | add optional `machineId` (M3) | no (optional back-ref) | label snapshot stays accurate | evidence still valid | "machine not recorded" from snapshot |
| Test Grids | `{key,label}` | add optional `machineId` (M3) | no | snapshot stays accurate | unchanged | falls back to `{key,label}` |
| Grid cells | inherit grid | inherit | no | — | — | — |
| Production settings | `{key,label,lens,focusMethod}` | add optional `machineId` (M3) | no | accurate | remains machine-specific | snapshot label shown |
| Evidence / promotion | `{key,label}` | add optional `machineId` (M3) | no | accurate | valid | snapshot label |
| Projects / settings snapshot | optional `settings.machine` | leave; add `machineId` only if a snapshot exists | no | accurate | unchanged | snapshot label |
| Inventory / Pricing | none | **none** | no | n/a | n/a | n/a |
| Design sessions / handoff | Designs are session-only; handoff copies thickness/name only | **no machine snapshot** in SVG; optional advisory machine note in the Project draft notes only | no | n/a | n/a | n/a |
| Future fabrication packages | n/a | design to carry `machineId`+snapshot when introduced (M5) | n/a | — | — | — |
| CSV / backup exports | inherit | inherit; may add a machine-name column later | no | — | — | — |

Principle: **`machineId` + compact `{key,label}` snapshot for production-critical records; `{key,label}` snapshot remains the display source of truth** so history is correct even if a machine is renamed, archived, or deleted. Do not add machine IDs to Log/Inventory/Pricing merely for completeness.

## 17. Machine-specific evidence and production-setting contract (smallest safe version)

Recommended minimal contract for the first phases (defer richer rules to a later evidence phase):
- **Library profiles stay global** (per material), each able to hold production settings from multiple machines (already true — settings carry their own `machine`). Do not partition Library by machine.
- **Production settings remain machine-specific via their existing snapshot**; add optional `machineId` for precise back-reference (M3) without changing eligibility yet.
- **Promotion eligibility keeps its current machine-aware behavior** (evidence must match the source machine; the Project Wizard already compares machine labels with alias support). Multi-machine adds `machineId` as a *stronger* match signal but must **not** silently make Machine-A evidence promote for Machine-B.
- **Legacy no-machine evidence stays usable** as "machine not recorded" (current behavior) — do not force confirmation retroactively.
- **Switching the active machine does not re-promote or reinterpret existing proven settings.** Visible "preferred" settings remain per their stored machine; whether the UI *filters* production settings by active machine is an M4 concern and a product decision (§31). Copied Reference starting points should retain explicit "from Genmitsu Reference" source attribution and should **not** silently attach to the active owned machine.
- This area is high-impact: recommend the **smallest** change now (add `machineId` back-references; keep all existing eligibility rules), and defer machine-scoped filtering/preference to M4 with its own review.

## 18. Active-machine switching contract

Switching the active machine should immediately affect **only newly opened workflows**, and must never rewrite records, change snapshots, re-promote, alter SVG bytes, or mutate open drafts.

| Surface | On switch |
|---|---|
| New Test Grid / Material Test / Project / Log defaults | use the new active machine's work area/notes as *context*; the grid's machine snapshot source is a product decision (§31) — today it derives from `machineProfile`; recommended: continue to snapshot the Reference profile **unless** Joe chooses to snapshot the active owned machine |
| Project Wizard recommendations | recompute on next open |
| Designs fit context / work-area advisory | reflect the new active work area (advisory only) |
| Reference table profile | **unchanged** (separate `machineProfile` selector) |
| Pricing / Inventory | unaffected |
| Open modal drafts, unsaved Design inputs, current preview, selected Grid/Project | **untouched** |

Recommended behavior: **apply to newly opened workflows; leave open drafts alone.** Because a grid/entry modal captures its machine at save, offer a **confirmation only if a machine-sensitive modal (grid) is currently open** — otherwise switch immediately for seamlessness. Do not auto-regenerate the current Design preview on switch (it would surprise the user and is advisory anyway).

## 19. UI selector recommendation

- Place the **Machines manager + active selector inside the Reference tab**, extending the existing "Current workshop machine" panel (it already owns machine setup and is separated from the Reference profile selector). This reuses established, responsive, accessible UI and avoids header clutter at 320 px.
- Surface the **active machine name always-visible** in a compact, low-cost location — recommend the Reference panel header and optionally the existing footer/About line; **defer a global header selector** unless beginner/switch-frequency testing shows friction (a product decision, §31). Do **not** add a new primary tab (no evidence it is needed).
- The manager supports: active selector, Add, Edit, Duplicate (optional, §31), Set default/active, Archive, Restore archived, and a one-line explainer that historical records keep their own machine snapshot and do not change. Assess at 320 px (reuse `.formgrid`/`.span2`/`.actions`, which already collapse) and desktop.

## 20. Machines manager scope

**Phase-2 essentials (minimum useful manager):** list machines; add; edit; active selector; set default/active; archive/restore; usable work-area width/height (mm); `type` (diode/CO₂/fiber/other); free-text `notes` (manufacturer/model/power live here initially). **Phase-5-or-later:** structured manufacturer/model/power fields, safe margins, pass-through, preferred sheet sizes, material-thickness limits, rotary, calibration, machine-specific kerf libraries, maintenance logs, usage counters. Keep the first manager a bounded CRUD panel, not a machine-management application.

## 21. Archive vs delete recommendation

**Archive is the default for any referenced machine; delete only an unreferenced, non-active machine.** Rules: cannot archive the active machine without first switching active (or auto-advance active with a clear notice); a machine referenced by evidence/Projects/settings → archive-only; a machine referenced only by historical snapshots → archive-only (records keep their snapshot regardless); a machine with **no** references → deletable; imported archived machines → retained as archived. **Never cascade-delete** Tests, Projects, settings, or evidence.

## 22. Large-format capability contract

To avoid a second migration, the `machines` object should reserve a stable, additive **capability shape** now (populate later): `type`, `workAreaWidthMm`, `workAreaHeightMm` (structured, mm) are the fields future Design fit checks will consume; `safeMarginMm`, `passThrough`, `preferredSheetSizes[]`, `origin`, `maxPracticalThicknessMm` can remain **additive future fields** (normalization tolerates them) and need not exist in M1. Required-for-future-fit: work area + `type`. Everything else may start as notes. **Do not implement panelization, multi-sheet nesting, pass-through production, or auto-scaling.** Define the capability contract as "an active machine may expose `{workAreaWidthMm, workAreaHeightMm, type}`; Designs may read them for advisory fit context only."

## 23. Designs integration boundary

For the initial multi-machine phases: active machine provides **advisory context only**. It must **not** affect the preview geometry, production SVG bytes, filenames, or Finished Views; work-area checks stay **advisory, non-blocking**; generated/downloaded SVG **must not embed machine metadata** (protect the exact-scale byte contract — a fixture already guards Designs byte-neutrality against `machineIdentity`, `:734`, and the same guarantee must extend to `machines`/`activeMachineId`). Switching machines must **not** auto-regenerate the current preview. Fit-status display and any machine-aware advisory in Designs belong to M5 and must remain advisory with no automatic scaling.

## 24. Product-naming implications

The multi-machine architecture does **not** depend on the "Genmitsu L8 Tracker" product name, except that the **Reference profile labels** (`'Genmitsu L8 20W'/'40W'`) and the display fallback are Genmitsu-specific — which is correct and should stay. Recommend product naming be settled **after** multi-machine but **before** release-candidate testing, so machine-manager copy and the Reference-vs-owned-machine wording are finalized once. **Do not rename** `APP_NAME`/`APP_ID`/`STORAGE_KEY`/`BACKUP_FORMAT`/repo/folder/exported files in this work.

## 25. Offline / file:// compatibility

Fully compatible: `machines` is a small array in the same `localStorage` object, no new key, no server, no external dependency, no IndexedDB. Backup download/import, the modal/accessibility system, and direct `index.html` operation are unaffected. Machine records are tiny (a handful of scalar fields each), so `localStorage` pressure is negligible relative to Project photos. **No framework or storage-engine change is warranted or recommended.**

## 26. Performance and scale considerations

Typical 1–3, advanced 5–20, edge 50+. A plain array with an `activeMachineId` lookup is more than adequate; a transient `machineById` Map (rebuilt per render or memoized) is a reasonable convenience but **should stay transient** (never persisted). The only potentially costly pattern is *filtering all records by machine* (M4) — with realistic record counts this is trivial; genuine stress testing (hundreds of machines × thousands of records) belongs in a later P9 stress phase, not now. Do not over-engineer.

## 27. Proposed fixtures

New focused group **`runMachineProfilesFixtures`** (`?selftest=machines-multi`), est. **45–70 assertions**, covering: migration from current singleton; blank-singleton migration; custom-singleton migration; idempotent normalization (repeat load/import no duplicates, incl. two `legacy-workshop-machine` inputs); active-machine fallback (missing/invalid/archived); add/edit/rename preserving ID; seamless switching updates defaults but not records/drafts; archive; active-archive guard; record `machineId` snapshot + `{key,label}` retained; historical display correct after rename; unavailable-machine fallback; legacy-backup Merge (local machines preserved) and Replace (restored/converted); new-backup Merge (id-keyed union, active local) and Replace (restored, invalid active falls back); same-ID conflict; same-name/different-ID distinct; malformed `machines` array → `[]`; storage recovery intact; first-run compatibility (machines don't suppress onboarding); **Reference-profile separation** (switching owned machine ≠ changing `machineProfile`); Test Grid / Material Test / Project machine snapshots unchanged for legacy records; evidence machine matching unchanged; production-setting behavior unchanged; responsive selector; modal accessibility; import safety guards unchanged; **no Designs production-byte change** (extend the `:734`-style guard to `machines`/`activeMachineId`); **no `STORAGE_KEY`/`BACKUP_FORMAT`/`SCHEMA_VERSION` change**. Also **extend `runMachineSetupFixtures`** to assert the `machineIdentity` mirror stays synced with the active machine. Do not weaken existing assertions.

## 28. Human workflow scenarios

- **A (L8 20W, later upgrade):** migration creates one machine; later Add a second and Set active. Safe/clear.
- **B (diode + CO₂):** two machines, distinct IDs, `type` differs; Reference profile stays Genmitsu and can still be viewed; owned-machine switch doesn't change Reference. Safe, provided the Reference-vs-owned separation copy is clear.
- **C (two identical units):** two distinct IDs despite identical name/model — correct; the "identity is ID not name" rule handles it. (Whether Joe *wants* them distinct is a product decision, §31; recommended yes.)
- **D (replace a machine, keep history):** archive the old machine; its records keep their snapshots; add the new machine. Safe — no cascade delete.
- **E (import another workshop's backup, similar names):** Merge keeps local machines and active selection; imported machines with different IDs are added (not deduped); the user reviews. Safe; no silent collision damage.
- **F (switch active while a grid/design is open):** open drafts untouched; a confirmation appears only for an open grid modal; Design preview not auto-regenerated. Safe/understandable.
- **G (archive the machine tied to best settings):** archive-only; settings and evidence remain valid and visible with the archived machine's snapshot. Safe.
- **H (future large-format pass-through):** the reserved additive capability shape (`type`, work area, later `passThrough`/`preferredSheetSizes`) lets a future phase read capabilities without re-migration. Safe as a contract; no implementation now.

All eight behave safely under Option B.

## 29. Implementation phases

- **M1 — Storage model + legacy migration (invisible).** Goal: add `machines`/`activeMachineId`, migrate the singleton idempotently, keep `machineIdentity` as the synced mirror, additive backup inclusion, Merge=local-preserve / Replace=restore. Files: `freshState`, `loadState`/normalization, `persist`, `backupObject`, `replaceData`, `mergeData`/`applyBackupPreferences`. Protected: `SCHEMA_VERSION`, `STORAGE_KEY`, `BACKUP_FORMAT`, `machineProfile`, record snapshots, Designs bytes. Migration risk: **medium** (the one migration). Fixtures: migration/idempotency/merge/replace/recovery. **Independent audit: yes.** Committable alone (invisible groundwork).
- **M2 — Machines manager + active selector (Reference tab).** Goal: CRUD + active/default + archive/restore + mirror sync; always-visible active name. Files: `renderReference`, new manager render/handlers, `bindPage`. Protected: Reference profile selector, storage shape from M1. Risk: low. Fixtures: UI/switch/archive/responsive/a11y. Audit: recommended. Committable. **M1+M2 is the minimum coherent user-visible release unit.**
- **M3 — Record `machineId` snapshots (additive).** Goal: stamp optional `machineId` on new Grids/settings/evidence/Project snapshots alongside `{key,label}`; legacy records unchanged. Protected: allowlist gate, historical display. Risk: low–med (touches snapshot writers). Fixtures: snapshot presence + legacy fallback. Audit: yes (touches evidence/promotion).
- **M4 — Machine-aware filtering & recommendations.** Goal: optional filtering/grouping of production settings/evidence by active machine; Project Wizard machineId matching. Product-decision-gated (§31). Risk: med (eligibility semantics). Audit: yes.
- **M5 — Large-format capability fields + Designs advisory fit.** Goal: structured capability fields + advisory (non-blocking, no-scale, no-byte-change) fit context. Risk: med. Audit: yes.

Minimum coherent release unit: **M1 + M2** (a machines collection with no manager is not user-visible; a manager with no storage cannot exist). M1 may ship first as safe invisible groundwork.

## 30. Risk register

| Risk | Likelihood | Impact | Mitigation | Evidence required |
|---|---|---|---|---|
| Data loss during migration | Low | High | idempotent normalization-only migration; keep `machineIdentity` mirror; recovery preserves raw | migration + idempotency + recovery fixtures |
| Duplicate migrated machines | Low | Med | fixed `legacy-workshop-machine` ID; migrate only when `machines` absent | idempotency fixture (repeat load/import) |
| Broken old backups | Low | High | additive fields, **no SCHEMA_VERSION bump**, no format change | legacy Merge/Replace fixtures |
| Invalid `activeMachineId` | Med | Low | fallback chain (first non-archived→first→'') + Genmitsu display fallback | active-fallback fixtures |
| Silent evidence cross-contamination | Low | High | keep existing machine-aware eligibility; add `machineId` as stronger match only; no auto-universalization | promotion/evidence fixtures |
| Rename changing history | Low | High | `{key,label}` snapshot is display source of truth; ID stable | historical-rename fixture |
| Archived machine unavailable | Med | Low | archive retains record snapshots; archived still selectable with note | archive fixtures |
| Active switch mutating drafts | Med | Med | switch affects new workflows only; confirm on open grid modal | switching fixtures |
| Reference-profile confusion | Med | Med | keep selectors separate; clear copy; no auto-coupling | separation fixture + human test |
| Import collision (similar machines) | Med | Med | id-keyed, no silent dedup, explicit review | merge-collision fixtures |
| Designs output change | Low | High | advisory only; extend byte-neutrality guard to `machines` | Designs byte fixture |
| Fixture cleanup regressions | Med | Low | snapshot/restore state+storage+activeMachineId+modals in every group | fixture self-restore review |
| User confusion (owned vs Reference vs snapshot) | Med | Med | manager explainer copy; §31 decisions; human testing | human beginner test |

## 31. Product decisions required from Joe (with recommended defaults)

1. **Should new-record machine snapshots switch from `machineProfile`-derived to active-**owned**-machine-derived?** *Recommended default: keep snapshotting the Reference profile in M1–M3 (lowest churn, preserves all fixtures), and revisit in M4.* — the single most consequential semantic choice.
2. **Do identical model units stay distinct physical machines?** *Recommended: yes (distinct IDs).*
3. **Archive-only vs delete-unused?** *Recommended: archive referenced, delete only unreferenced+non-active.*
4. **Are Library profiles global (not per-machine)?** *Recommended: yes, global.*
5. **Are preferred production settings per machine or global?** *Recommended: keep per-stored-machine; defer machine-scoped filtering to M4.*
6. **Does switching change open unsaved drafts?** *Recommended: no; confirm only on open grid modal.*
7. **Do copied Reference starting points attach to the active machine?** *Recommended: no; retain "from Genmitsu Reference" attribution.*
8. **Which machine fields in the initial manager?** *Recommended: name, type, work area, notes, archived (structured mfr/model/power later).*
9. **Where does the active selector live?** *Recommended: Reference-tab manager + always-visible name; defer header selector.*
10. **May a machine be duplicated?** *Recommended: yes (convenience), as a new ID.*
11. **Should the blank Genmitsu singleton become a real persisted machine on migration?** *Recommended: yes (one editable entry), with display still falling back to Genmitsu wording when unnamed.*

## 32. Findings by severity

**BLOCKER (0):** none — provided the additive Option B path is followed. (Choosing Option A/C, bumping `SCHEMA_VERSION`, or replacing the allowlist gate destructively **would** create blocker-class compatibility/data risk; they are avoided by the recommendation.)

**IMPORTANT (5):**
- **I-1** Record snapshots are keyed to `machineProfile`→Genmitsu allowlist, not to `machineIdentity`. Multi-machine must add an **optional `machineId` alongside** the existing `{key,label}` snapshot and **must not** extend/replace the `productionMachineKeys` gate destructively, or real-machine grids/settings would be silently dropped by `gridMachineSnapshot`.
- **I-2** Decide whether new-record snapshots come from the Reference profile (today) or the active owned machine (§31.1) before M3 — it changes evidence/promotion semantics.
- **I-3** No `SCHEMA_VERSION` bump; keep additive + `machineIdentity` mirror, or old-app backup import breaks via the future-version guard (§10).
- **I-4** Merge must treat `machines`/`activeMachineId` as **local workshop config** (id-keyed union, no silent dedup, active stays local), following the `machineIdentity` precedent — not the `machineProfile` overwrite-on-merge precedent (§13).
- **I-5** Evidence contract: keep existing machine-aware eligibility; adding `machineId` must never silently make one machine's evidence promote for another (§17).

**POLISH (4):** selector placement/header-vs-Reference (§19); machine duplication (§31.10); structured mfr/model/power vs notes in M1 (§20); always-visible active-name location (§19).

**NOT A DEFECT:** `machineIdentity` being display-only today; records snapshotting the Reference profile; the fixed allowlist (it correctly protects historical keys and is *extended*, not removed); `machineProfile` remaining a root preference.

## 33. Final architecture verdict

**READY AFTER PRODUCT DECISIONS**

The additive **Option B + optional-`machineId` snapshot** architecture is sound, low-risk, backward-compatible, offline-safe, and requires no schema/format/key change or rewrite. It is not yet ready to *implement* only because several genuine product-semantic choices (§31 — especially I-2: Reference-profile-derived vs active-owned-machine-derived record snapshots) materially shape M3–M4 and should be settled first. Once decisions 1–7 are made, the work is ready for bounded implementation starting at M1.

## 34. Remaining unverified areas

- Live re-execution of the 2205 fixture baseline this session (environment blocked the headless-Edge scratch cleanup; baseline accepted as stated and cross-checked against prior harness structure).
- Exact assertion-count impact on each existing fixture group when `machineId` is added (estimated in §27; confirmed only at implementation).
- Human comprehension of the owned-machine vs Reference-profile vs record-snapshot distinction (needs beginner testing after M2 copy exists).

## 35. Physical laser testing required?

**No.** This is a storage/UI architecture review; nothing depends on physical laser behavior, and the recommended path changes no geometry, production output, or settings.

## 36. May Codex begin implementation?

**Not yet** — Codex may begin **M1** immediately **after** Joe answers §31 decisions 1, 3, 6, and 11 (the ones that shape M1's migration and merge behavior); decisions 2, 4, 5, 7, 9, 10 can be settled before M2–M4. Do not start M3+ until decision 1/I-2 is fixed.

## 37. Independent audit after each phase?

**Yes for M1, M3, M4, M5** (each touches storage/migration, evidence semantics, or Designs advisory). **M2** (manager UI) warrants a lighter focused audit. Each phase is separately committable and should be audited before the next begins, matching the project's established phase→audit rhythm.

## 38. Confirmation

This review made **no edits** to any source or documentation file other than creating this report, and did **not** stage, commit, or push. HEAD remains `288d474`, the tracked tree is clean, and `git diff --check` is clean. A disposable headless Edge profile was launched for an attempted baseline re-run and terminated; no repository file or real user data was touched.
