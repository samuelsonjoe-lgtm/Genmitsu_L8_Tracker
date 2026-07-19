# Designs Form Organization and Progressive Disclosure — Focused Adversarial Audit

**Date:** 2026-07-19  
**Auditor:** Grok (read-only adversarial audit)  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `e4bb285` — *Improve Designs result summaries*  
**Full baseline hash:** `e4bb285314f5ada3d7e0e5ee2c557bf397359915`  
**Authoritative documents:**

- `docs/DESIGNS_FORM_ORGANIZATION_PROGRESSIVE_DISCLOSURE_REVIEW_2026-07-19.md`
- `docs/DESIGNS_FORM_ORGANIZATION_PROGRESSIVE_DISCLOSURE_IMPLEMENTATION_2026-07-19.md`

**Primary audit question:**  
Is the Designs Form Organization and Progressive Disclosure implementation complete, accessible, behavior-preserving, and safe to commit without changing draft values, normalization, production geometry, SVG bytes, downloads, Finished Views, storage, schema, or unrelated templates?

**Physical cutting:** Not required for this presentation-only phase (production outputs verified byte-identical).

---

## Verdict

# SAFE TO COMMIT WITH NON-BLOCKING NOTES

- Another Claude review: **unnecessary** (no normalization, storage/schema, shared form architecture, production-output, or conditional-value regressions found).
- Claude Opus 4.8: **unnecessary** (no high-impact data or production risk).
- Fable 5: **unnecessary**.
- Physical cutting: **not required** (production and Finished View digests match baseline).

---

## 1. Working-tree scope

### Git state (recorded at audit start)

| Item | Value |
|------|--------|
| Branch | `main` |
| HEAD | `e4bb285314f5ada3d7e0e5ee2c557bf397359915` |
| Message | Improve Designs result summaries |
| Ahead/behind `origin/main` | **0 / 0** |
| Staging area | **Empty** (`git diff --cached` empty) |
| Unrelated working-tree content | **Preserved** (not modified by this audit) |

### Tracked modifications (intentional product surface)

| File | Diffstat |
|------|----------|
| `README.md` | 4 lines changed (`2 ++ / 2 --` in combined stat) |
| `index.html` | 60 lines changed (`46 ++ / 14 --`) |
| **Total** | **2 files, 48 insertions, 16 deletions** |

```
 README.md  |  4 ++--
 index.html | 60 ++++++++++++++++++++++++++++++++++++++++++++++--------------
 2 files changed, 48 insertions(+), 16 deletions(-)
```

### Documentation deliverable (untracked, expected)

- `docs/DESIGNS_FORM_ORGANIZATION_PROGRESSIVE_DISCLOSURE_IMPLEMENTATION_2026-07-19.md`
- Review doc also present untracked (not product code)

### Other untracked (preserved; not product of this phase)

Historical audit/review docs, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, and similar — **not altered** by this implementation scope.

### Functions added (presentation only)

| Function | Role |
|----------|------|
| `designTemplateSelect(value)` | Native `<select>` with four `<optgroup>`s; escapes labels/options; selected value deterministic |
| `designFormSectionLabel(text)` | Compact visible section label for Drawer Cabinet basic blocks |
| `designAdvancedSection(summaryText, bodyHtml)` | Native `<details class="project-section span2">` + `<summary>` + body `.project-section-body.formgrid`; no JS, no draft mutation |

### Functions modified (presentation only)

| Function | Change |
|----------|--------|
| `renderDesigns` | Template select → `designTemplateSelect`; Sliding-Lid / Drawer Cabinet field grouping into basic + one Advanced each |
| `runDesignGeometryFixtures` | +7 assertions (1086 → 1093 expected) covering optgroups, Advanced placement, compact templates |

### Protected functions inspected (unchanged)

| Boundary | Evidence |
|----------|----------|
| `normalizeDesignDraft` | No hunks in diff |
| `buildSlidingLidDesignResult` / drawer / finger / coupon / tray / QR / hanging / cleat builders | No hunks |
| Production SVG serializers / multi-output download | No hunks |
| Finished View builders | No hunks |
| Storage / backup / `machineProfile` | No hunks |
| Shared `designSelect` / `designNumber` / `designCheckbox` / draft update path | `designSelect` body unchanged; only new helpers added adjacent |

### Scope findings

| Severity | Finding |
|----------|---------|
| **NOTE** | Diff is tightly scoped to form HTML assembly + fixtures + README prose. No builder/serializer/storage rewrite. |
| **NOTE** | No broad formatting sweep; line-ending warnings are Git `core.autocrlf` noise, not content rewrite. |

**BLOCKER / HIGH / MEDIUM for scope:** none.

---

## 2. Template selector optgroups

### Rendered groups and membership (from working `designTemplateSelect`)

**Boxes and cabinets**

- `finger-box` — Finger-jointed box  
- `sliding-lid-box` — Sliding-lid finger box  
- `drawer-cabinet` — Drawer Cabinet  

**Trays**

- `dice-tray` — Dice tray  
- `divider-tray` — Divider tray  

**Signs and stands**

- `qr-stand` — QR / sign stand  
- `hanging-sign` — Hanging sign  

**Coupons and prototypes**

- `joint-fit-coupon` — Joint Fit Coupon  
- `concealed-cleat-corner-prototype` — Concealed Cleat Corner Prototype  
- `concealed-cleat-full-box-prototype` — Concealed Cleat Full-Box Prototype  

### Verification

| Check | Result |
|-------|--------|
| Every registered template value appears exactly once | **Pass** (10 options; set equality vs baseline) |
| No missing / duplicate options | **Pass** |
| Option **values** byte-for-byte identical to baseline set | **Pass** |
| Labels accurate | **Pass** |
| Selected template retained after rerender | **Pass** (fixtures + DOM `selected`) |
| Changing selection updates `designDraft.template` | **Pass** (existing change path; not rewritten) |
| Dispatch still reaches same builders | **Pass** (no dispatch map changes) |
| Label/select association | **Pass** (`<label>Template<select name="template">…`) |
| No search / cards / favorites / recent / persistence | **Pass** |
| Orphan options vs registered builders | **Pass** (option set ≡ baseline set) |

### Optgroup label note

| Severity | Finding |
|----------|---------|
| **NOTE** | DOM option **order** changed (grouped order vs prior flat order). Values and labels unchanged. Intentional; not a contract break. |

---

## 3. Local selector helper

| Check | Result |
|-------|--------|
| Local to Designs template selector | **Pass** (`designTemplateSelect` only used for Designs template) |
| Shared `designSelect` unchanged | **Pass** (diff shows original function body intact) |
| Option and optgroup text escaped | **Pass** (`esc(label)`, `esc(key)`, `esc(text)`) |
| Selected-value handling deterministic | **Pass** (`key === value ? 'selected' : ''`) |
| No draft mutation during render | **Pass** (pure HTML string) |
| No storage / machineProfile dependency | **Pass** |
| Unknown template values | **Pass** (no option selected if value missing; falls through existing draft behavior without crash) |

| Severity | Finding |
|----------|---------|
| **NOTE** | Group membership is a second list alongside any builder registry — low maintenance-drift risk; currently matches baseline option set. |

---

## 4. Native Advanced-section helper

| Check | Result |
|-------|--------|
| Emits native `<details>` / `<summary>` | **Pass** |
| Reuses `project-section` styling | **Pass** (`class="project-section span2"`) |
| No custom click/toggle handlers | **Pass** |
| No `role="button"` on summary | **Pass** |
| No custom `aria-expanded` | **Pass** |
| No tabindex overrides | **Pass** |
| Closed by default | **Pass** (no `open` attribute) |
| Contains formgrid body | **Pass** (`.project-section-body.formgrid`) |
| Spans form width | **Pass** (`span2`) |
| No persisted open state | **Pass** |
| Does not read/mutate `designDraft` | **Pass** |
| Does not call `refreshDesignPreview` / `updateDesignDraft` | **Pass** |
| Deterministic re-render | **Pass** |
| No duplicate IDs introduced | **Pass** (no new id attributes on Advanced shell) |
| Used only on Sliding-Lid + Drawer Cabinet | **Pass** (two call sites; other templates compact) |

---

## 5. Sliding-Lid visible-field placement

**Outside Advanced (basic):**

- `slidingDimensionMode`, `slidingWidth`, `slidingDepth`, `slidingHeight`
- `lidPull`, `lidPullWidth` (when applicable)

**Inside single Advanced** (`summary` text exactly: **Advanced fit and production options**):

- `slidingJointClearance`, `slidingPreferredFingerWidth`
- `lidSideClearance`, `lidVerticalClearance`, `frontInsertionClearance`
- `slidingGuideMarks`, `assemblyLabels`, `slidingFitCoupon`

| Check | Result |
|-------|--------|
| Exactly one `details.project-section` on form | **Pass** |
| Defaults closed | **Pass** |
| No field duplicated in/out | **Pass** |
| No required field missing | **Pass** |
| Basic dimensions usable without opening Advanced | **Pass** |
| Conditional Pull width outside Advanced | **Pass** |

---

## 6. Sliding-Lid field-contract preservation

Independent baseline-vs-working field-name inventory: **identical name sets**.

Exercises (automated against working app):

| Exercise | Result |
|----------|--------|
| Pull type None → other → None → prior | **Pass** (`lidPullWidth` survives dormant) |
| Advanced values survive W/D/H edits | **Pass** (`slidingJointClearance` retained) |
| Advanced values survive rerender | **Pass** |
| Reach `normalizeDesignDraft` / builder | **Pass** (no normalize/builder diff; digests match) |
| Closed `<details>` controls in FormData | **Pass** |

| Severity | Finding |
|----------|---------|
| **NOTE** | Automated attribute scan reported `select` vs `select-one` type-string noise only — not a real control-type change. |

**Min/max/step/defaults/labels:** no intentional contract changes in diff; production digests confirm geometry path unchanged.

---

## 7. Sliding-Lid disclosure interaction

| Item | Record |
|------|--------|
| Browser | Microsoft Edge (`msedge.exe` headless Chromium) |
| Mode | **Headless** (automated); interactive checks also reported by implementer |
| Enter opens | **Pass** |
| Space closes | **Pass** |
| Focus remains on summary | **Pass** |
| No navigation | **Pass** (no URL change in harness) |
| No draft change from toggle alone | **Pass** (no custom toggle JS) |
| No localStorage / backup mutation from toggle | **Pass** (storage isolation suite) |
| No production mutation from toggle | **Pass** |
| No preview refresh from toggle alone | **Pass** (no listeners) |
| Open: fields in tab order | **Pass** (native) |
| Closed: browser-native behavior | **Pass** (no custom interference) |

---

## 8. Drawer Cabinet visible-field placement

**Outside Advanced:**

- Drawer size/layout: `drawerInsideWidth/Depth/Height`, `drawerRows`, `drawerLayoutMode`, `drawerColumns` (uniform), custom-row selectors (custom mode)
- Material: shell thickness, `drawerUseSeparateInteriorThickness`, `drawerInteriorThickness` when enabled, linked-thickness note when disabled

**Inside single Advanced** (`summary` exactly: **Advanced fit, joints, and markings**):

- Cabinet/drawer joint clearances, preferred finger width  
- Lateral / vertical / rear clearances  
- Shelf guides, assembly labels  

| Check | Result |
|-------|--------|
| Exactly one Advanced `details.project-section` | **Pass** |
| Defaults closed | **Pass** |
| Section labels concise (“Drawer size and layout”, “Material”) | **Pass** |
| No control duplicated | **Pass** |
| Structural layout not hidden in Advanced | **Pass** |
| Custom-row geometry basic/discoverable | **Pass** |

---

## 9. Drawer Cabinet field-contract preservation

| Exercise | Result |
|----------|--------|
| Uniform ↔ custom-row | **Pass** (field sets match baseline for each mode) |
| 1 / 2 / 3 rows | **Pass** (name-set identity) |
| Separate thickness off → on → off → on | **Pass** (`drawerInteriorThickness` restored) |
| Basic edits with Advanced closed | **Pass** (FormData still includes advanced keys) |
| Advanced clearances stable | **Pass** |
| FormData includes advanced when closed | **Pass** |
| Linked vs separate production digests | **Pass** (match baseline) |
| Dormant custom-row / thickness conventions | **Pass** (no normalize change) |

---

## 10. Drawer Cabinet disclosure interaction

Same Edge headless results as Sliding-Lid: native Enter/Space, focus retention, no custom JS, no draft/storage/preview side effects from toggle alone. Uniform/custom-row controls remain outside Advanced. Responsive: no horizontal overflow at 320 / 700 / 1280.

---

## 11. Simple-template compactness

Rendered without template-specific Advanced (`details.project-section` count **0**):

- QR Stand, Hanging Sign, Dice Tray, Divider Tray, Finger Box  
- Joint Fit Coupon finger-edge and wall-to-base  
- Concealed Cleat Corner and Full-Box prototypes  

| Check | Result |
|-------|--------|
| Baseline fields present; names/contracts unchanged | **Pass** (name-set identity) |
| No empty Advanced | **Pass** |
| No new conditional behavior | **Pass** |
| Finger Box fully visible (explicit intent) | **Pass** |
| Coupon no range preview / Finished View added | **Pass** (no builder/UI feature change) |
| Cleat warnings / Line-mode guidance intact | **Pass** (no message-path hunks) |

**NOTE:** Do not count Project-tab `<details>` outside Designs form — audit scoped to Designs form root.

---

## 12. Complete field-set preservation

Independent inventory: baseline vs working rendered control **name sets** for every template and exercised internal mode → **zero mismatches**.

| Check | Result |
|-------|--------|
| Exact field-name set unchanged | **Pass** |
| No remove / duplicate / rename | **Pass** |
| No control type / min/max/step / option-value / checkbox-default change in product sense | **Pass** |
| `assemblyLabels` shared behavior | **Pass** (still checkbox in Advanced for SL/cabinet; unchanged for others) |
| No cross-template field leakage | **Pass** |

---

## 13. FormData behavior

Compared for identical drafts:

1. Baseline flat form  
2. Working Advanced **closed**  
3. Working Advanced **open**  

| Check | Result |
|-------|--------|
| Closed details controls successful | **Pass** |
| Values included when closed | **Pass** |
| Closed ≡ open semantic FormData | **Pass** |
| Toggle open/close does not alter FormData | **Pass** (closed → open → reclosed identical) |
| No `disabled` used to hide advanced fields | **Pass** |
| Checkboxes / selects behave as before | **Pass** |

---

## 14. Draft and mode preservation

| Path | Result |
|------|--------|
| Template switching | **Pass** (existing draft merge; no disclosure keys) |
| Sliding-Lid pull-mode | **Pass** |
| Cabinet layout / rows / separate thickness | **Pass** |
| Basic edits with Advanced closed | **Pass** |
| Advanced edits + rerender | **Pass** |
| Invalid-result rendering | **Pass** (suite covers invalid paths; no form-org regression) |
| Finished View switching | **Pass** (digests match) |
| Reload / backup import conventions | **Pass** (storage isolation; no new keys) |
| No disclosure state in draft/storage | **Pass** |
| No dormant-field convention change | **Pass** |
| Normalization output | **Pass** (unchanged code + matching production digests) |

---

## 15. Conditional explanatory text

| Item | Placement |
|------|-----------|
| Pull width | Outside Advanced with pull control |
| Linked-thickness note | Outside Advanced with material controls |
| Separate thickness field | Outside Advanced |
| Uniform/custom-row + row selectors | Outside Advanced |

No essential caveat moved into Advanced while its control stayed outside (or vice versa). No new help text required; existing associations retained.

---

## 16. Accessibility

| Check | Result |
|-------|--------|
| Template label associated | **Pass** |
| Optgroup labels meaningful | **Pass** |
| Native details/summary semantics | **Pass** |
| Focus indication | Inherited from existing `.project-section summary` styles |
| Enter / Space native | **Pass** |
| No custom ARIA duplicating native | **Pass** |
| No new dangling aria-* | **Pass** |
| Labels still wrap relocated inputs | **Pass** (same label helpers) |

---

## 17. Responsive layout

| Viewport | Horizontal overflow (`documentElement` / `body`) |
|----------|--------------------------------------------------|
| 320 px | **false** |
| 700 px | **false** |
| 1280 px | **false** |

Formgrid inside Advanced uses existing CSS (`min-width: 0`, 1-col under 700 px). No page-level overflow introduced.

---

## 18. Production-byte neutrality

SHA-256 digests of production SVG strings (and multi-output panels where applicable) for Sliding-Lid, Drawer Cabinet linked/separate, Finger Box, Dice Tray, QR Stand, Joint Fit Coupon modes, Concealed Cleat Full-Box — **all match baseline** (`dig_mismatches` empty).

| Severity | Finding |
|----------|---------|
| — | **No production geometry or SVG byte change.** |

---

## 19. Finished View neutrality

Finished View string digests for applicable templates — **match baseline**.

---

## 20. Downloads

No download/serializer code changed. Multi-output production digests match baseline → download payload content unchanged for tested templates.

---

## 21. Storage / schema / machine profile

| Check | Result |
|-------|--------|
| localStorage key set after design form use | **Identical** to baseline set |
| Backup object keys | **Identical** |
| No disclosure state persisted | **Pass** |
| No draft-key schema change | **Pass** |
| 20W / 40W profile path | Unaffected (no machineProfile hunks); isolation suite green |

---

## 22. Fixture quality

| Item | Result |
|------|--------|
| Expected count | **1093** (was 1086; +7 form-organization assertions) |
| Coverage | Optgroups, SL/cabinet Advanced placement & closed default, compact templates without Advanced, summary text strings |
| Gaps | **LOW:** fixtures do not drive real keyboard events (covered separately via Edge headless); do not assert FormData closed/open equality (covered in this audit harness) |

---

## 23. Browser validation

| Item | Record |
|------|--------|
| Channel | Microsoft Edge (Chromium), **headless** automation |
| Screenshots | Not required; DOM + overflow + keyboard scripted |
| Console | No app errors observed in harness load path |
| Keyboard | Enter open / Space close / focus on summary — **Pass** |
| Overflow | 320 / 700 / 1280 — **Pass** |
| Advanced discoverability | Summary text explicit; closed-by-default as designed |
| Project-specific wording bleed | None observed on Designs Advanced summaries |

Physical cutting not performed (not required).

---

## 24. Documentation accuracy

### README (Designs bullet additions)

States / implies correctly:

- Template selector grouped by design type  
- Sliding-Lid and Drawer Cabinet keep common structural controls visible  
- Less-common fit/marking options under native Advanced  
- Closing Advanced never disables, resets, removes, or saves those draft values  
- Form organization does not change production SVGs or downloads  

Does **not** claim: new template capabilities, changed defaults, automatic optimization, nesting, multi-sheet export, bed-fit configuration, improved physical fit, or saved disclosure preferences.

### Implementation report

Aligns with source for helpers, placement, fixture count (1093), and production neutrality. Independent re-run confirmed totals.

---

## 25. Validation commands and exact totals

| Command / check | Result |
|-----------------|--------|
| `git status` / HEAD / branch / ahead-behind | Recorded above |
| `git diff --check` (HEAD vs working, `index.html` `README.md`) | **Pass** (exit 0) |
| `python -m html.parser` on `index.html` | **Pass** |
| Designs geometry fixtures | **1093 passed / 0 failed** |
| Tray-model fixtures | **264 passed / 0 failed** |
| Complete suite (`window.runMaterialBrowserFixtures`, 15 groups) | **1885 passed / 0 failed** |
| Baseline vs working production digests | **All match** |
| Baseline vs working field name sets | **All match** |
| FormData closed/open/reclosed | **Match** |
| Draft preservation (SL + cabinet) | **Pass** |
| Storage / backup isolation | **Identical key sets** |
| Edge headless keyboard + overflow | **Pass** |

**Note on suite total:** The complete selftest entrypoint is `window.runMaterialBrowserFixtures` (not a non-existent `runAll*`). Reported **1885** matches implementer validation when that entrypoint is used.

---

## 26. Blocking findings

**None.**

No BLOCKER or HIGH issues. No MEDIUM issues requiring change before commit.

---

## 27. Non-blocking improvements

| Severity | Item |
|----------|------|
| **LOW** | Template option **display order** changed by optgroups (value set unchanged). Document if any external doc listed “order on the form.” |
| **LOW** | Group membership list is separate from builder registry — optional future single source of truth. |
| **LOW** | Fixtures could assert FormData closed ≡ open for one SL + one cabinet draft. |
| **NOTE** | Review doc optionally discussed Finger Box Advanced; implementation correctly left Finger Box compact per bounded contract. |
| **NOTE** | `select` vs `select-one` is DOM type-string noise in generic attribute diffs only. |

---

## 28. Unverified areas

| Area | Status |
|------|--------|
| Interactive (non-headless) Edge visual polish | Rely on implementer report + headless DOM; no contradiction found |
| Exhaustive every-min/max string compare for all attributes | Spot-checked via digests + name sets + FormData; full attribute matrix not printed |
| Physical cutting | Explicitly out of scope; not required |

---

## 29. Final verdict statement

The Designs Form Organization and Progressive Disclosure implementation is **complete** for the bounded presentation contract: native template optgroups; exactly one native Advanced disclosure each for Sliding-Lid Box and Drawer Cabinet; all other listed templates compact; no custom disclosure JavaScript; no persisted open/closed state; no draft-key, normalization, production, SVG, download, Finished View, storage, or schema changes detected.

**SAFE TO COMMIT WITH NON-BLOCKING NOTES**

Reviewer guidance: another Claude pass, Claude Opus 4.8, Fable 5, and physical cutting are **not** warranted for this commit.
