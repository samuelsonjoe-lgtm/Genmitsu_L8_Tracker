# Designs-to-Projects Handoff Phase 1 — Focused Adversarial Audit

**Date:** 2026-07-19  
**Auditor:** Grok (read-only adversarial audit)  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `b35ed48` — *Organize Designs form controls*  
**Full baseline hash:** `b35ed48356b3ba190951e2e58dde2647ae99f04e`  
**Authoritative documents:**

- `docs/DESIGNS_TO_PROJECTS_HANDOFF_REVIEW_2026-07-19.md`
- `docs/DESIGNS_TO_PROJECTS_HANDOFF_IMPLEMENTATION_2026-07-19.md`

**Primary audit question:**  
Is the Designs-to-Projects Handoff implementation safe to commit, with correct point-in-time Project prefilling, explicit save confirmation, cancellation safety, Project-data integrity, Inventory/Pricing neutrality, production neutrality, and no storage/schema/import-export regression?

**Physical cutting:** Not required (production outputs verified byte-identical to baseline).

---

## Verdict

# SAFE TO COMMIT WITH NON-BLOCKING NOTES

- Another Claude Opus review: **unnecessary** (no persistent Project/schema/import-export or production risk found).
- Grok: **sufficient** for this bounded schema-neutral phase.
- Fable 5: **unnecessary**.
- Physical cutting: **not required**.

---

## 1. Working-tree scope

### Git state (audit start)

| Item | Value |
|------|--------|
| Branch | `main` |
| HEAD | `b35ed48356b3ba190951e2e58dde2647ae99f04e` |
| Message | Organize Designs form controls |
| Ahead/behind `origin/main` | **0 / 0** |
| Staging area | **Empty** |
| Unrelated untracked content | **Preserved** (LightBurn projects, historical docs, `debug.log`, generator script, etc.) |

### Tracked modifications (intentional product surface)

| File | Diffstat (`git diff b35ed48 --numstat`) |
|------|------------------------------------------|
| `README.md` | 6 insertions / 4 deletions (net prose + suite totals) |
| `index.html` | 134 insertions / 2 deletions |
| **Combined `--stat`** | **2 files, 140 insertions, 6 deletions** |

```
 README.md  |  10 +++--
 index.html | 136 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++-
 2 files changed, 140 insertions(+), 6 deletions(-)
```

### Exact changed hunks (`index.html`)

| Region | Change |
|--------|--------|
| `renderDesigns` actions row (~1935) | Add `#startProjectFromDesign` button |
| `refreshDesignPreview` (~3651–3654) | Wire disabled state for start button with download |
| Designs event binding (~3668–3669) | `startProject.onclick → startProjectFromDesign` |
| ~3688–3745 | New helpers + mapper + handler |
| ~10277–10346 | `runDesignToProjectHandoffFixtures` + export |
| ~11247 | `selftest=design-project` / `all` registration |

### Documentation deliverable (untracked, expected)

- `docs/DESIGNS_TO_PROJECTS_HANDOFF_IMPLEMENTATION_2026-07-19.md`
- Review doc also present untracked (not product code)

### Functions added

| Function | Role |
|----------|------|
| `projectDesignTemplateName(template)` | Display-name map for Project title |
| `projectDesignDimensions(...values)` | Compact finished/outside size text via existing `designDimensionsText` |
| `projectDesignWarningNote(warnings)` | Bounded, deterministic warning → one note line |
| `projectDraftFromDesign(result, draft)` | Pure structured-data → partial Project draft |
| `startProjectFromDesign()` | Guard + open existing Project modal |
| `runDesignToProjectHandoffFixtures()` | 17 dedicated assertions |

### Functions modified (presentation / wiring only)

| Function | Change |
|----------|--------|
| Designs results action HTML in `renderDesigns` | Button markup |
| `refreshDesignPreview` | Enable/disable start button with validity |
| Designs init binding | onclick |
| Selftest bootstrap | Register handoff group |

### Protected functions inspected — **unchanged** (no hunks)

| Boundary | Status |
|----------|--------|
| `buildDesignResult` / all `build*DesignResult` builders | Unchanged |
| Production serializers / `productionOutputs` / downloads | Unchanged |
| Finished View builders | Unchanged |
| `normalizeDesignDraft` | Unchanged |
| `openProject`, Project `onsubmit` / `upsert` | Unchanged |
| `normalizeProject` / `normalizeProjectAccounting` / material lines | Unchanged |
| `projectDraftFromPricing` / `createProjectFromPricing` | Unchanged (pattern reused, not duplicated) |
| Inventory matching / Pricing | Unchanged |
| `persist` / `backupObject` / `mergeData` / `replaceData` | Unchanged |
| Project Wizard (`openProjectWizard`) | Unchanged |

### Scope findings

| Severity | Finding |
|----------|---------|
| **NOTE** | Diff is tightly scoped: UI action + pure mapper + fixtures + README. No golden production constants rewritten. |
| **NOTE** | No formatting sweep; Git CRLF warnings are autocrlf noise. |

**BLOCKER / HIGH / MEDIUM for scope:** none.

---

## 2. Added and modified functions (inventory)

### Reuse vs duplication

| Capability | Implementation choice | Finding |
|------------|----------------------|---------|
| Open modal | Calls existing `openProject(draft, { draftNote })` | **Pass** — no second editor |
| Save path | Existing form submit → `normalizeProject` → `upsert` | **Pass** |
| ID creation | Only on Save via existing path | **Pass** |
| Inventory matching | Not called | **Pass** |
| Pricing handoff | Unchanged parallel precedent | **Pass** |
| Modal infrastructure | Existing `openModal` / conflict guard on `.open` | **Pass** |
| Design generation / download | Not invoked by handoff | **Pass** |

---

## 3. `projectDraftFromDesign` purity

Independent exercise (16 template/mode cases + fixture purity test):

| Check | Result |
|-------|--------|
| Does not mutate `result` | **Pass** |
| Does not mutate `draft` | **Pass** |
| Does not mutate `state` | **Pass** |
| Does not call `persist` / `backupObject` | **Pass** |
| Does not create ID / call `uid` | **Pass** (no `id` in returned object) |
| Does not open modal | **Pass** (mapper only returns object) |
| Does not download / Blob / parse SVG | **Pass** |
| Deterministic for same inputs | **Pass** |
| Invalid result → `null` | **Pass** |
| `null` / empty result → `null` without throw | **Pass** |
| Machine profile 20W vs 40W | **Pass** (fixture: identical draft + SVG) |

`startProjectFromDesign` may call `updateDesignDraft(form)` to sync the live form before building — that is handler behavior, not mapper impurity.

---

## 4. Existing Project field contract

Returned draft keys observed:

`name`, `material`, optional `thicknessValue`/`thicknessUnit`, `jobType`, `quantity`, `status`, `notes`, `sourceEntryId`, `sourceProfileId`.

| Forbidden / schema-creep field | Present? |
|--------------------------------|----------|
| `designId`, `designTemplate`, `sourceDesign`, `designSnapshot` | **No** |
| `svg`, `svgData`, `productionFiles`, `productionOutputs` | **No** |
| `designWarnings`, `savedDesignId`, `handoffDraft` | **No** |
| `id` before Save | **No** |
| `materialLines` / `settings` / accounting / photos | **No** |
| `sourceEntryId` / `sourceProfileId` | **null** (explicit) |

Saved Project after normal Save: fixture asserts no keys matching `/design|svg|template/i` and sources remain null.

---

## 5. Core mapping correctness

All registered templates and required modes exercised (mapper inventory + fixtures):

| Template / mode | name non-empty | material `''` | thickness mm when present | jobType `cut` | qty `1` | status `kept` | notes OK | no SVG |
|-----------------|----------------|---------------|---------------------------|---------------|---------|---------------|----------|--------|
| qr-stand | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| hanging-sign | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| dice-tray | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| divider-tray | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| finger-box open | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| finger-box loose lid | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| finger-box faux dovetail | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| sliding-lid-box | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| drawer-cabinet linked | Pass | Pass | shell | Pass | Pass | Pass | Pass | Pass |
| drawer-cabinet separate | Pass | Pass | shell | Pass | Pass | Pass | Pass | Pass |
| drawer-cabinet custom-row | Pass | Pass | shell | Pass | Pass | Pass | Pass | Pass |
| joint-fit-coupon finger-edge | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| joint-fit-coupon wall-to-base | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| concealed-cleat-corner | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |
| concealed-cleat-full-box | Pass | Pass | Pass | Pass | Pass | Pass | Pass | Pass |

No piece/panel/drawer/coupon/file count used as quantity.

---

## 6. Project naming

Sample independent names:

| Case | Example name |
|------|----------------|
| QR Stand | `Freestanding QR / sign stand 130 × 160 mm` |
| Hanging Sign | `Hanging sign 130 × 160 mm` |
| Dice Tray | `Dice Tray 226 × 166 mm` (outside base) |
| Divider Tray | `Divider Tray 220 × 160 mm` (inside) |
| Finger Box | `Finger-jointed box 126 × 96 × 53 mm` (outside) |
| Sliding-Lid | `Sliding-lid finger box 112 × 86 × 50.2 mm` |
| Cabinet grid | `Drawer Cabinet 3 × 2 … mm` (columns × rows + outside) |
| Cabinet custom | `Drawer Cabinet — custom rows … mm` |
| Coupon FE / WTB | `Joint Fit Coupon — Finger Edge` / `— Wall-to-Base` |
| Cleat corner / full | Prototype titles; full-box includes outside dims |

| Check | Result |
|-------|--------|
| Finished/outside vs sheet size | **Pass** (structured metrics / finishedView) |
| No timestamps / customer names / path counts | **Pass** |
| Compact, editable | **Pass** |
| Template / mode distinction | **Pass** |

| Severity | Finding |
|----------|---------|
| **NOTE** | Names can grow with three-axis mm dimensions; still editable before Save and acceptable for Phase 1. |

---

## 7. Material neutrality

| Check | Result |
|-------|--------|
| `material` always `''` | **Pass** |
| No Inventory ID / line / reservation / deduction | **Pass** |
| No `materialLines` prefilled | **Pass** |
| Thickness alone does not assert stock availability | **Pass** |
| Blank-material Save blocked by existing validation | **Pass** (fixture: alert + no record) |
| Material supplied → Save creates one Project | **Pass** |

---

## 8. Thickness mapping

| Template family | Source | Result |
|-----------------|--------|--------|
| Most templates | `metrics.materialThickness` (else finishedView mm) | Copied as mm |
| Drawer Cabinet | `metrics.shellThickness` | Copied as mm |
| Separate cabinet | Shell in thickness; interior in notes only | **Pass** |

Separate-thickness notes (verified):

- `Interior/drawer stock: 2.7 mm`
- `Two production SVG files are required`
- No second material line / identity / availability claim

Does not parse presentation HTML for thickness when structured metrics exist.

---

## 9. Quantity semantics

`quantity: '1'` for every case including multi-piece trays, six-panel boxes, multi-drawer cabinets, two-file cabinets, multi-sample coupons, multi-piece cleat prototypes. After Save, fixture asserts `savedProject.quantity === 1` (normalized number). No fixture treats piece/drawer/file counts as quantity.

---

## 10. Notes contract

| Template class | Notes content (verified) |
|----------------|--------------------------|
| QR / hanging | Panel/stand context; no quantity inflation |
| Dice | Dry-fit wall-to-base guidance |
| Divider | Divider count in notes; qty remains 1 |
| Finger loose lid | No retention |
| Finger faux dovetail | Decorative only; structural joint remains finger |
| Sliding + coupon option | Sliding-fit test coupon; verify on material |
| Cabinet separate | Interior stock + two production files |
| Cabinet custom-row | Row drawer counts |
| Coupons | Test artifact; no winning value |
| Cleats | Assembly/registration prototype; **not** strength/load-capacity test |

Notes are concise, editable, free of SVG text, filenames, cost/sheet/time estimates, and Inventory claims. No universal scale-safety boilerplate copied.

---

## 11. Warning selection

`projectDesignWarningNote` joins `result.warnings` and maps via ordered regexes to **at most one** note line (negative clearance; loose/zero clearance / slide/rack; underside cover cosmetic).

| Check | Result |
|-------|--------|
| Valid warnings do not disable action | **Pass** |
| Universal exact-scale guidance not copied | **Pass** |
| Large-layout advisory not copied | **Pass** (not in selector) |
| Ordering deterministic | **Pass** |
| Does not parse rendered HTML | **Pass** |
| Does not mutate `result.warnings` | **Pass** |

| Severity | Finding |
|----------|---------|
| **LOW** | Warning selection is brittle substring/regex logic. Correct today; future warning rewording could silently drop notes. Prefer stable warning codes later if warnings proliferate. |

---

## 12. Start Project action state

| State | Button present | Label | `type="button"` | Disabled |
|-------|----------------|-------|-----------------|----------|
| Valid normal | Yes | Start Project from Design | Yes | No |
| Valid + warnings | Yes | same | Yes | No |
| Invalid | Yes | same | Yes | **Yes** (native) |
| Valid multi-output cabinet | Yes | same | Yes | No |
| Finished View selected | Yes | same | Yes | No (validity-driven) |

Button is outside SVG markup; render does not create Projects; refresh does not create Projects. Keyboard activation uses native button + existing modal focus path.

---

## 13. Handler invalid-state protection

Direct `startProjectFromDesign()` with invalid Design:

| Check | Result |
|-------|--------|
| Returns safely with invalid result | **Pass** |
| Modal not opened | **Pass** |
| No Project / ID / success toast | **Pass** |
| state / localStorage / backup unchanged | **Pass** |
| Inventory / Pricing unchanged | **Pass** |

Not reliant solely on disabled attribute.

---

## 14. Existing Project-modal conflict guard

When `#projectModal` has class `open`, handler shows info toast and returns without replacing form.

| Scenario | Result |
|----------|--------|
| Unsaved New Project (handoff re-click) | **Pass** — name preserved (fixture) |
| Edit existing Project (`openProject` with id) | **Pass** — name, material, notes, `editingProjectId`, photos draft, material-lines draft unchanged (independent audit) |

No merge, no nested modal, no false “draft opened” success path when blocked.

---

## 15. Open lifecycle

With no modal open, valid Design → Start Project:

| Check | Result |
|-------|--------|
| Existing Project modal opens | **Pass** |
| “New project” heading | **Pass** |
| `editingProjectId === null` | **Pass** |
| draftNote present (nothing saved until Save) | **Pass** |
| Prefill name / blank material / thickness / qty 1 / notes | **Pass** |
| No Project record / no ID | **Pass** |
| `state.projects` deep-equal | **Pass** |
| localStorage / backup / Inventory / Pricing unchanged | **Pass** |
| Design draft not linked live | **Pass** (point-in-time; Design edit after open does not rewrite modal fields) |
| Production SVG unchanged | **Pass** |

---

## 16. Cancellation lifecycle

| Path | Result |
|------|--------|
| `[data-close]` Cancel | Modal closes; no Project; storage/backup/Inventory/Pricing/Design production unchanged | **Pass** |
| Escape | Existing modal infrastructure; no Project created when closed | **NOTE** — full Escape UX relies on shared modal code (unchanged); not a handoff-specific regression |

---

## 17. Save lifecycle

| Step | Result |
|------|--------|
| Save with blank material | Existing alert; blocked; modal stays open; no record | **Pass** |
| Save with material set | Exactly one normal Project via existing submit | **Pass** |
| Saved fields | material set; qty 1; job cut; sources null; no design/svg keys | **Pass** |
| Inventory / Pricing | Unchanged | **Pass** |
| Production / Finished View after Save | Unchanged | **Pass** |

---

## 18. Duplicate-protection / re-activation

Repeated Start while modal open: form preserved; project count unchanged. After Cancel, a new handoff can open a fresh snapshot. No automatic second record without Save.

---

## 19. Point-in-time findings

| Check | Result |
|-------|--------|
| Snapshot at click, not live link | **Pass** |
| Design field edits after open do not rewrite Project form | **Pass** |
| No Design identity stored on Project | **Pass** |
| No SVG stored on Project | **Pass** |

---

## 20. Accessibility

| Check | Result |
|-------|--------|
| Native `<button type="button">` | **Pass** |
| Disabled via native attribute | **Pass** (non-color-only) |
| Keyboard activation | **Pass** (native + click handler) |
| Focus after open | Existing modal path (Pricing-equivalent) |
| Info toast on success / conflict | **Pass** |
| No custom ARIA modal-in-modal | **Pass** |

---

## 21. Production-byte neutrality

Baseline `b35ed48` vs working, representative digests (`svgLen` + FNV `designFixtureHash` + multi-output hashes):

| Template | Match |
|----------|-------|
| Finger Box (loose lid) | **Yes** (`2615` / `6181bc75`) |
| Sliding-Lid | **Yes** (`2818` / `468e9fd8`) |
| Drawer Cabinet linked | **Yes** |
| Drawer Cabinet separate | **Yes** (outputs match) |
| Dice Tray | **Yes** |
| QR Stand | **Yes** |
| Joint Fit Coupon | **Yes** |
| Concealed Cleat Full-Box | **Yes** |

Designs geometry group remains **1093 / 0** on both baseline and working (no golden constant drift).

---

## 22. Finished View neutrality

Lifecycle fixture compares `finishedView.svg` before/after handoff open, cancel, and save — **identical**. Mapper does not read Finished View markup for naming beyond structured `finishedView` dimension fields where appropriate (trays).

---

## 23. Download findings

No download function modified. Handoff never calls download helpers. Multi-output cabinet remains eligible for Start Project while single Download stays disabled — by design.

---

## 24. Storage / schema findings

| Check | Result |
|-------|--------|
| No new Project fields | **Pass** |
| No new storage keys | **Pass** |
| `SCHEMA_VERSION` untouched | **Pass** |
| localStorage unchanged until explicit Project Save | **Pass** |
| backupObject isolation before Save | **Pass** |
| Import/export / merge / replace untouched | **Pass** |

---

## 25. Inventory findings

Inventory JSON deep-equal across open/cancel/save (except Project array growth on successful Save only). No match, reservation, deduction, or line creation from handoff.

---

## 26. Pricing findings

Pricing state deep-equal across handoff lifecycle. Pricing → Project path unchanged.

---

## 27. Import/export findings

No changes to backup encode/decode, merge, or replace. Saved handoff Projects are ordinary records.

---

## 28. Fixture quality

| Item | Result |
|------|--------|
| Dedicated group | `runDesignToProjectHandoffFixtures` — **17 passed / 0 failed** |
| Coverage | Purity, field contract, thickness/shell, naming, notes, action state, invalid handler, open/cancel/save, conflict, production/FV stability, machine profile |
| Gaps | **LOW:** Edit-Project conflict covered by independent audit, not dedicated fixture assertion; Escape path not asserted; warning-regex brittleness not fixture-locked to codes |

---

## 29. Complete-suite arithmetic

Entrypoint: sequential call of the **16** `selftest=all` groups (same set as page bootstrap), summing `passed`/`failed`.

| Group | Passed | Failed |
|-------|--------|--------|
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| production | 66 | 0 |
| promotion | 58 | 0 |
| design-production | 118 | 0 |
| grid | 23 | 0 |
| grid-machine | 18 | 0 |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| **design-project** | **17** | **0** |
| metadata | 12 | 0 |
| storage | 8 | 0 |
| project-wizard | 216 | 0 |
| design | 1093 | 0 |
| **Complete total** | **1902** | **0** |

| Metric | Value |
|--------|--------|
| Detached baseline complete (15 groups, no handoff) | **1885 / 0** (implied; 1902 − 17 = 1885) |
| Working complete | **1902 / 0** |
| Net increase | **exactly 17** |
| Non-Design + Designs | 809 + 1093 = **1902** |
| Tray-model (separate, not in 16) | **264 / 0** |
| Designs geometry alone | **1093 / 0** (baseline and working) |

---

## 30. Browser and responsive validation

| Item | Record |
|------|--------|
| Browser | Microsoft Edge (Chromium), **headless** automation via Playwright |
| Mode | DOM + fixture lifecycle; implementer also reported interactive `file://` |
| Screenshots | Not required; DOM assertions used |
| Console | No handoff-related page errors in harness |
| Overflow 320 px | **false** (document / body) |
| Overflow 1280 px | **false** |
| Button / modal usability | Existing styles; action row wraps with secondary button |
| Keyboard | Native button click path opens modal |

Physical cutting not performed.

---

## 31. Documentation (README) accuracy

README correctly states:

- Valid Designs open a reviewable New Project draft  
- Point-in-time snapshot  
- Nothing saved until Save project; Cancel creates none  
- Material intentionally blank  
- Measured thickness may prefill in mm  
- Quantity is one finished Design  
- Warnings/prototype context may enter notes  
- No Inventory, Pricing, SVG, schema, or download behavior changes  

Does **not** claim live links, presets, automatic material matching, Inventory deduction, cost/sheet estimates, automatic downloads, guaranteed fit/strength, or new schema fields.

Suite totals and console API list updated for 16 groups / 1902 / handoff runner / `design-project` selftest.

| Severity | Finding |
|----------|---------|
| **NOTE** | Older paragraphs elsewhere in README still mention historical assertion counts in long Design prose; the Built-in checks section is the authoritative suite summary and is updated. |

---

## 32. Validation commands and exact totals

| Command / check | Result |
|-----------------|--------|
| `git status` / HEAD / branch / ahead-behind | Recorded above |
| `git diff --cached` | Empty |
| `git diff --check b35ed48 -- index.html README.md` | **Pass** |
| `python -m html.parser` on `index.html` | **Pass** |
| `runDesignToProjectHandoffFixtures` | **17 / 0** |
| `runDesignGeometryFixtures` | **1093 / 0** |
| `runTrayModelFixtures` | **264 / 0** |
| Complete 16-group sum | **1902 / 0** |
| Baseline Designs geometry | **1093 / 0** |
| Production digests baseline vs working | **Match** (core svg hash/len/fnv) |
| Mapper purity / field contract (16 modes) | **0 failures** |
| Edit-Project conflict guard | **Pass** |
| Overflow 320 / 1280 | **Pass** |

---

## 33. Blocking findings

**None.**

No BLOCKER or HIGH issues. No MEDIUM issues requiring change before commit.

---

## 34. Non-blocking improvements

| Severity | Item |
|----------|------|
| **LOW** | `projectDesignWarningNote` regex/substring mapping is brittle if warning copy changes. |
| **LOW** | `projectDesignTemplateName` duplicates display strings also present in `designTemplateSelect` groups — mild maintenance drift risk. |
| **LOW** | Add fixture asserting Edit-Project conflict (id non-null) explicitly. |
| **NOTE** | Cabinet name uses `columns × rows` order (e.g. `2 × 3` for 2 columns × 3 rows) — consistent with fixtures; document if users expect rows×columns. |
| **NOTE** | Only one warning note is selected even if many warnings exist — acceptable Phase 1 bound. |

---

## 35. Unverified areas

| Area | Status |
|------|--------|
| Interactive visual polish of button wrapping on every template | Rely on overflow + implementer Edge checks; no contradiction |
| Exhaustive import of older backups containing post-handoff Projects | Ordinary Project shape; no schema change |
| Physical cutting | Out of scope; not required |

---

## 36. Final verdict statement

The Designs-to-Projects Handoff Phase 1 implementation is **complete** for the bounded contract: one Start Project from Design action; validity-gated enablement including warnings and multi-output cabinets; pure structured mapping into existing Project fields; blank material; honest thickness; quantity one finished design; concise notes; point-in-time snapshot via real Project modal; cancel and conflict safety; no schema, storage, Inventory, Pricing, production, Finished View, or download regressions.

**SAFE TO COMMIT WITH NON-BLOCKING NOTES**

Reviewer guidance: another Claude Opus pass, Fable 5, and physical cutting are **not** warranted; Grok is sufficient for this schema-neutral phase.
