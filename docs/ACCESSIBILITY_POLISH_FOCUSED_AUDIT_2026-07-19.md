# Accessibility Polish — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`  
**Audit date:** 2026-07-19  
**Audit mode:** Read-only (no product source edits; no stage/commit/push/reset/clean/stash/checkout)  
**Implementation report reviewed:** `docs/ACCESSIBILITY_POLISH_IMPLEMENTATION_2026-07-19.md`  
**Expected committed baseline:** `23b4706` — *Add optional machine setup and work area*  
**Expected pre-phase fixture baseline:** 2041 passed / 0 failed  

---

## 1. Repository state and actual baseline

### Commands / observations

| Check | Result |
|--------|--------|
| `git status -sb` | `## main...origin/main`; tracked mods: `CHANGELOG.md`, `README.md`, `index.html` |
| `git log -1 --oneline` | `23b4706 Add optional machine setup and work area` |
| `git rev-parse HEAD` | `23b4706486baa98f87c3b42a0aa4deac9b2ec1dc` |
| `git rev-list --left-right --count origin/main...main` | `0	0` |
| `git diff --check` | Clean (CRLF/LF working-copy notices only) |
| `git diff --stat` | `CHANGELOG.md` +1; `README.md` ~7 lines; `index.html` +137/−8; **3 files, +137/−8** |
| `git diff --cached` | Empty — **nothing staged** |
| `git ls-files --others --exclude-standard` | Pre-existing untracked set intact (`LightBurn Projects/`, `debug.log`, historical docs, `docs/ACCESSIBILITY_POLISH_IMPLEMENTATION_2026-07-19.md`, etc.) |

### Confirmations

- **Actual HEAD** matches expected baseline `23b4706` / full hash above.
- **Branch:** `main`, synchronized with `origin/main` (`0/0`).
- **Committed baseline before this phase:** clean tree at `23b4706`; phase work is uncommitted working-tree only.
- **Modified tracked files only:** `index.html`, `README.md`, `CHANGELOG.md`.
- **Relevant new untracked file:** `docs/ACCESSIBILITY_POLISH_IMPLEMENTATION_2026-07-19.md` (plus this audit report written after review).
- **Nothing staged.** No commit or push indicated for this phase.
- **Unrelated untracked assets** remain present and were not modified by this audit.

### Constants (source)

| Constant | Value | Unchanged by phase? |
|----------|--------|---------------------|
| `APP_ID` | `genmitsu-l8-tracker` | Yes |
| `APP_NAME` | `Genmitsu L8 Tracker` | Yes |
| `APP_VERSION` | `0.9.0` | Yes |
| `BUILD_DATE` | `2026-07-19` | Yes |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` | Yes |
| `SCHEMA_VERSION` | `2` | Yes |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` | Yes |

### Fixture baselines

- Pre-phase complete suite (contract + prior phase): **2041 / 0**.
- This phase adds accessibility polish group **+36** → expected complete **2077 / 0** (README arithmetic: 984 non-Design + 1093 Designs).
- This audit runtime (headless Edge, `file://`): accessibility **36/0**, modal **26/0**, help **37/0**, first-run **19/0**, machine **50/0**, storage **15/0**, design **1093/0**. Zero page errors.

---

## 2. Files and functions reviewed

### Files

- `index.html` — CSS focus/target rules; static tablist/tabpanel markup; `renderTabs` / `render` / `setTab`; `isPrimaryTab` / `handlePrimaryTabKeys` / `handleShortcuts`; `runAccessibilityPolishFixtures`; selftest registry; modal helpers (non-regression); Help / first-run / Reference / machine setup surfaces as touched by fixtures.
- `README.md`, `CHANGELOG.md`
- `docs/ACCESSIBILITY_POLISH_IMPLEMENTATION_2026-07-19.md`
- Full `git diff` of the three tracked files vs `HEAD`

### Functions / seams

`renderTabs`, `render` (`aria-labelledby` update), `setTab`, `bindPage` tab click rebinding, `handlePrimaryTabKeys`, `isPrimaryTab`, `isTypingTarget`, `handleShortcuts`, `activeModal` / modal Tab trap / Escape, `activeSearchInput` / `/` shortcut, `runAccessibilityPolishFixtures`, sibling fixture groups, `runModalAccessibilityFixtures` inventory (still 26), Designs geometry group (protected).

---

## 3. Implementation summary

Bounded accessibility polish:

1. **Primary nav semantics:** `#tabs` is `role="tablist"` with `aria-label="Tracker sections"`; each tab is a native `button` with stable `tab-{id}`, `role="tab"`, `aria-selected`, `aria-controls="app"`, roving `tabindex`; `#app` is the single `role="tabpanel"` with `aria-labelledby` following the active tab.
2. **Keyboard:** Left/Right wrap-activate; Home/End; Enter/Space re-activate focused tab with `focusTab` restore after render; only when focus is a primary tab and no modal is open.
3. **CSS:** global `:focus-visible` 3px `#075985` + 2px offset; `@supports not selector(:focus-visible)` fallback; disabled opacity cursor; primary controls `min-height: 40px`; dense table buttons `min-height: 34px`.
4. **Fixtures:** `runAccessibilityPolishFixtures` (36 assertions), `?selftest=accessibility` / `all`.
5. **Docs:** README keyboard/tablist note; CHANGELOG 0.9.0 bullet; fixture total 2077.

No storage/schema/version/Designs geometry changes.

---

## 4. Primary-tab inventory

| # | Key (`data-tab`) | DOM ID | Visible label (default) |
|---|------------------|--------|-------------------------|
| 1 | `log` | `tab-log` | Log |
| 2 | `library` | `tab-library` | Library |
| 3 | `grids` | `tab-grids` | Test Grids |
| 4 | `designs` | `tab-designs` | Designs |
| 5 | `reference` | `tab-reference` | Reference |
| 6 | `projects` | `tab-projects` | Projects |
| 7 | `inventory` | `tab-inventory` | Inventory (may append low-stock count) |
| 8 | `pricing` | `tab-pricing` | Pricing |

- **Count:** 8 (complete; none conditionally omitted).
- **Active-tab source:** `state.activeTab`.
- **Buttons rebuilt** on every `render()` via `renderTabs()`; IDs remain stable `tab-${id}`.
- **Programmatic navigation:** `setTab(tab, options)` used by keyboard path with `{ focusTab: true }`; click path uses `setTab(b.dataset.tab)` without focus steal; first-run Open Reference uses plain `setTab('reference')` (no forced tab focus).
- **Duplicate IDs after repeated renders:** fixture + probe **0** duplicates.
- **Workflow expectation:** keyboard activation intentionally restores focus to the newly rendered active tab; ordinary `setTab` does not.

**Verdict:** Inventory complete and stable. **NOT A DEFECT.**

---

## 5. Tablist and tabpanel semantic findings

| Requirement | Result |
|-------------|--------|
| `role="tablist"` + accessible label | Pass — `aria-label="Tracker sections"` |
| Native `button`, `role="tab"` | Pass |
| Stable unique IDs | Pass — `tab-log` … `tab-pricing` |
| `aria-selected` true only when active | Pass (string `"true"` / `"false"`) |
| `aria-controls` → real panel `#app` | Pass |
| Roving `tabindex` 0 / −1 | Pass |
| `#app` `role="tabpanel"` | Pass |
| `aria-labelledby` tracks active tab | Pass — updated in `render()` each time |
| No fabricated hidden panels | Pass — single shared panel content swap |
| No unnecessary panel focus stop | Pass — panel not made a tab stop |

Stale attributes after keyboard / click: probe confirmed `aria-labelledby` and selected id stay synchronized (`afterRight`, `afterClick`, `afterSpace`).

**Verdict:** Semantics correct. **NOT A DEFECT.**

---

## 6. Keyboard activation findings

| Behavior | Source / runtime |
|----------|------------------|
| Right/Left activate + focus next/prev | Fixture 36/0 + probe |
| Wrap last↔first | Fixture |
| Home / End | Fixture |
| Enter | Fixture |
| Space | Live probe: stays on focused tab, selected/labelledby stable; `preventDefault` avoids page scroll |
| Mouse click | Probe: Reference activates, form present |
| Selected + focused synchronized | Fixture + probe |
| `state.activeTab` single source of truth | Yes |

**Enter/Space double-activation:** `handlePrimaryTabKeys` calls `preventDefault()` then `setTab(..., { focusTab:true })`. This suppresses native button activation that would otherwise double-render. No double-activation defect observed.

**Verdict:** Activation model correct. **NOT A DEFECT.** Space coverage is fixture-gap polish only (see §20).

---

## 7. Focus-after-render findings

- Keyboard path: `setTab(id, { focusTab: true })` → `render()` rebuilds tabs → focuses `document.getElementById(\`tab-${tab}\`)` with `preventScroll` when possible.
- Old focused button is detached safely; focus moves to new corresponding node (fixture asserts `document.activeElement ===` new tab).
- Programmatic `setTab` without `focusTab` does not move focus (fixture sentinel).
- Mouse click: `setTab` without `focusTab` — no unexpected focus restore to tab.
- First-run Open Reference: plain `setTab('reference')` — does not steal focus to tab.
- Modal close restoration: Escape after Help still returns focus to `#helpBtn` (fixture); tab handler skipped when modal open.
- Form/filter `render()` paths do not pass `focusTab` — no unexpected tab focus.
- Rapid 20× Right Arrow: focus remains connected to a real tab; selection advances correctly (probe).

**Verdict:** **NOT A DEFECT.**

---

## 8. Key-handler isolation findings

`isPrimaryTab` requires:

- `document.activeElement === target`
- target matches `#tabs [role="tab"][data-tab]`
- no alt/ctrl/meta
- `handleShortcuts` only calls `handlePrimaryTabKeys` when `!activeModal()`

Verified / inspected:

| Target | Isolation |
|--------|-----------|
| Text input (`#search`) | ArrowRight does not switch tabs (fixture) |
| Select (`#machineProfile`) | ArrowLeft does not switch (fixture) |
| Number input (work area) | ArrowRight keeps Reference selected (probe) |
| Home/End in search | Stay on Log; focus remains search (probe) |
| Modal controls | ArrowRight does not switch tabs (fixture) |
| `/` from tab | Focuses search (fixture + probe) |
| Content buttons / links | Not primary tabs → no handler |
| `preventDefault` | Only on handled primary-tab keys |

`isTypingTarget` still protects `/` and Ctrl+N for INPUT/TEXTAREA/SELECT. Contenteditable is not used as a primary typing surface in this app (no regression path).

**Verdict:** Isolation solid. **NOT A DEFECT.**

---

## 9. Existing-shortcut findings

| Shortcut | Status |
|----------|--------|
| `/` focuses active search | Pass; still gated by `!isTypingTarget` |
| Ctrl+N | Unchanged path; not modified beyond tab handler insert |
| Ctrl+S | Unchanged modal form submit path |
| Escape | Modal close first; else grid drill-out — unchanged |
| Modal Tab / Shift+Tab | Still first branch in `handleShortcuts`; modal fixtures **26/0** |
| Single document `keydown` listener | Only `handleShortcuts` (+ unrelated material-suggest keydown) |
| Tab nav vs modal trap | Modal open short-circuits primary tab keys |

**Verdict:** **NOT A DEFECT.**

---

## 10. Focus-visible CSS findings

```css
:where(button, input, select, textarea, summary, [tabindex]):focus-visible {
  outline: 3px solid #075985;
  outline-offset: 2px;
}
@supports not selector(:focus-visible) { ... :focus { same } }
```

- Covers tabs, header buttons, ordinary buttons, links (via no — **links are not in the selector list**). Links (`a`) are **not** included in `:where(...)`. Most interactive chrome is buttons; PDF/manual links in Reference are anchors. **POLISH:** extend focus-visible to `a[href]` if keyboard users need the same 3px ring on those links (browser default outline may still apply).
- Inputs, selects, textareas, summary, `[tabindex]` (help-tips) covered.
- Modal controls are buttons/inputs — covered.
- Table-row action buttons covered as `button`.
- Close controls covered.
- ~3 px + offset as reported.
- No `outline: none` without replacement in stylesheet (`outlineNoneCount: 0`).
- Disabled: native disabled + opacity; not forced focusable.
- `.help-tip:focus` still uses 2px `var(--blue)` (pre-existing) — can compete with global rule; still visible. **POLISH** only.
- `@supports not selector(:focus-visible)` is valid in Chromium/Edge.

**Verdict:** Core focus indicator good. Link omission and help-tip dual rule = **POLISH**, not blockers.

---

## 11. Focus-indicator contrast findings

Independent WCAG relative-luminance calculations for `#075985` against adjacent surfaces:

| Adjacent background | Contrast |
|---------------------|----------|
| `#ffffff` (panel) | **7.56:1** |
| `#f5f6f8` (body) | **6.99:1** |
| `#eef6fb` / `#fff4db` / `#eef8f1` / `#fbeeee` | ~6.7–6.9:1 |
| `#236d9c` (primary button face, inner edge) | **1.35:1** |

With `outline-offset: 2px`, the ring sits primarily on page/panel backgrounds, where contrast exceeds ~3:1 non-text guidance. The inner edge against solid primary blue is weaker — acceptable tradeoff for this phase; not a confirmed unusable indicator on common layouts.

**No formal WCAG conformance claimed.**

**Verdict:** Adequate for common surfaces. Primary-button inner-edge = **POLISH** / theoretical, not **IMPORTANT**.

---

## 12. Touch-target findings

CSS:

- `.tab { min-height: 40px }`
- `.topbar .row > button, .actions > button, .modal-actions > button, .project-section > summary { min-height: 40px }`
- `.spread > button[data-close] { min-width/height: 40px }`
- `.tablewrap button, .grid-table button { min-height: 34px }` (dense)

Runtime heights (~px):

| Control | Height |
|---------|--------|
| Primary tab | ~45 |
| Help / Import / Export / unit / New Entry / Save machine setup | ~40 |
| Dense table rule present | yes (34px scoped) |

Notes:

- Short tab labels (e.g. “Log”) can be **narrow (~32px wide)** despite 40px height — phase targeted height, not width. **POLISH** for optional `min-width` on tabs if desired.
- Help-tips remain 16×16 (pre-existing; have `aria-label`).
- No evidence of overlapping targets, clipped labels, or global dense-row inflation.
- 320px: horizontal page overflow **false**; tabs scroll horizontally as designed (`tabsScroll: true`).

**Verdict:** Bounded target sizing meets phase intent. **NOT A DEFECT** for height; narrow tab width = **POLISH**.

---

## 13. Contrast-calculation findings

Reported pairs recomputed (WCAG relative luminance):

| Pair | Reported | Audited |
|------|----------|---------|
| `#222831` on `#f5f6f8` | 13.72:1 | **13.72:1** |
| `#667085` on `#ffffff` | 4.97:1 | **4.97:1** |
| `#236d9c` on `#ffffff` | 5.62:1 | **5.62:1** |
| `#2f7d55` on `#eef8f1` | 4.62:1 | **4.62:1** |
| `#9a641d` on `#fff4db` | 4.56:1 | **4.56:1** |
| `#b14242` on `#fbeeee` | 4.98:1 | **4.98:1** |

These map to real CSS variables/uses (`--ink`/`--bg`, `--muted`, `--blue`, pill good/warn/softbad).

Additional independent checks:

| Pair | Ratio | Notes |
|------|-------|-------|
| Muted `#667085` on body `#f5f6f8` | **4.60:1** | Normal text ≥4.5:1 |
| Primary white on `#236d9c` | **5.62:1** | Pass |
| Danger `#b14242` on white | **5.63:1** | Pass |
| Labels `#424b5a` on white | **8.80:1** | Pass |
| Table header `#475467` on `#f3f5f8` | **7.04:1** | Pass |
| Toast white on `#111827` | **17.74:1** | Pass |
| Pill text `#425166` on `#eef1f5` | **7.13:1** | Pass |

**Confirmed weak pairs:** None at normal-text failure level among primary UI pairs inspected. Placeholder color is browser-default (not custom CSS) — not a confirmed app defect. Disabled at `opacity: .65` is intentionally de-emphasized (allowed pattern).

Phase did **not** change the core color tokens for body/muted/pills; it mainly added focus outline color and target sizing. Contrast table in the implementation report is accurate for existing tokens.

**Verdict:** No confirmed contrast **IMPORTANT** failures. **NOT A DEFECT** for reported pairs.

---

## 14. Placeholder and disabled-state findings

- Important inputs retain real `<label>` wrappers (machine setup fixture; existing form patterns).
- Placeholders are supplementary (e.g. search, machine name examples), not sole labels.
- Disabled: `cursor: not-allowed; opacity: .65`; native `disabled` keeps them out of tab order.
- Disabled styling does not grant focusability (fixture asserts CSS presence; native behavior retained).

**Verdict:** **NOT A DEFECT.**

---

## 15. Active-tab-state findings

Active communication is multi-channel:

- `aria-selected="true"`
- class `active`
- color `var(--blue)` + **3px bottom border** in blue (not color-only)

Inactive: muted color + transparent bottom border. Focus outline is distinct from selection underline. Works at 320px and desktop probes.

**Verdict:** **NOT A DEFECT.**

---

## 16. Table / dense-control findings

- Dense actions scoped to `.tablewrap button, .grid-table button` at 34px — primary 40px rule does not blanket-inflate tables.
- Focus outline uses offset; overflow containers may clip outlines in some edge scroll positions (common CSS limitation) — **POLISH** if observed in a specific table, not confirmed as a phase regression.
- No columns/controls removed in diff.
- Row actions remain text buttons with understandable names (Edit/Delete/etc. patterns unchanged).
- Design geometry suite **1093/0** — no production UI regression from CSS.

**Verdict:** **NOT A DEFECT.**

---

## 17. Disclosure-control findings

- `.project-section > summary` gets `min-height: 40px`.
- Global `:focus-visible` includes `summary`.
- Release footer `details`/`summary` remain native open/closed; no custom key hijack for disclosures.
- Keyboard focusable via native summary behavior.

**Verdict:** **NOT A DEFECT.**

---

## 18. Modal non-regression findings

- Modal fixture inventory **still 26 / 0**.
- Primary tab keys do not run while a modal is open.
- Tab trap, Escape topmost, focus restore: fixture-green (including Help open/close path inside accessibility suite).
- Focus CSS applies to modal buttons/inputs.
- Close targets get 40×40 min sizing for `button[data-close]`.
- Modal helper logic not rewritten — only keyboard routing order in `handleShortcuts`.

**Verdict:** **NOT A DEFECT.**

---

## 19. First-run, Help, Reference, and machine-setup findings

| Area | Result |
|------|--------|
| First-run buttons remain `<button>` | Fixture |
| Open Reference uses `setTab('reference')` without focus steal | Source |
| Header Help open/close + focus return | Fixture |
| Help Tab trapping | Sibling help **37/0** + modal **26/0** |
| Machine setup labels nested | Fixture |
| Reference profile select usable; arrows isolated | Fixture |
| No persisted a11y state | Storage/backup fixture asserts unchanged bytes |
| Machine suite | **50/0** |

**Verdict:** **NOT A DEFECT.**

---

## 20. Accessibility-fixture quality findings

**Claimed:** 36 assertions. **Observed:** 36 passed / 0 failed.

Mapping to core requirements (combined assertions where appropriate):

| # | Requirement | Covered? |
|---|-------------|----------|
| 1–2 | Tablist role + label | Yes (1 assert) |
| 3–6 | Native tab, role, IDs, aria-controls | Yes |
| 7–10 | aria-selected / tabindex active+inactive | Yes (selected count + roving) |
| 11–12 | Tabpanel role + labelledby | Yes (log case; keyboard paths recheck labelledby indirectly via selection) |
| 13 | Visible active treatment | Yes |
| 14–15 | Focus-visible + fallback CSS | Yes (source regex) |
| 16–17 | Touch targets + dense 34px | Yes (CSS contract) |
| 18 | Disabled not focusable (CSS) | Partial — CSS presence, not runtime disabled focus probe |
| 19–24 | Arrow/Home/End wrap | Yes |
| 25 | Enter | Yes |
| 26 | Space | **No fixture** (live probe OK) |
| 27 | Programmatic no focus steal | Yes |
| 28–30 | Input/select/modal isolation | Yes |
| 31 | Escape close Help | Yes |
| 32 | Slash search | Yes |
| 33–35 | Machine / Reference Help / first-run buttons | Yes |
| 36+ | Dup IDs, offline, storage, constants, siblings | Yes |

Relative to a larger “47 behavioral points” contract: **36 assertions meaningfully combine** multiple points (e.g. one assert covers all tabs’ role+controls). Important behavioral guarantees are largely present. Gaps (Space, mouse click, rapid nav, Home/End-in-input, live focus outline paint, contrast numbers) are **fixture completeness polish**, not evidence of missing implementation.

**Does not claim** screen-reader certification or full WCAG audit (implementation report honest).

**Verdict:** Adequate for phase. Gaps = **POLISH**.

---

## 21. Fixture-realism findings

- Keyboard tests use synthetic `KeyboardEvent` on real focused tab nodes (not pure string mocks of attributes only).
- Semantics asserted on live DOM after `render()`.
- Isolation uses real `#search`, `#machineProfile`, Help modal.
- CSS contracts are regex-on-stylesheet (appropriate for style presence; not pixel rasterization).
- Sibling groups re-run real suites.

**Verdict:** Real enough for this phase. **NOT A DEFECT.**

---

## 22. Fixture-cleanup findings

`runAccessibilityPolishFixtures` restores state, localStorage, first-run dismiss flag, modal DOM/open/aria, modal origin maps, focus, removes sentinel, and `render()`s. Sibling groups self-restore. No durable pollution observed (`lsKeys` empty after runs in clean profile).

**Verdict:** **NOT A DEFECT.**

---

## 23. Fixture-total reconciliation

| Item | Count |
|------|--------|
| Pre-phase complete | 2041 |
| New accessibility polish | +36 |
| Expected complete | **2077** |
| README non-Design + Design | 984 + 1093 = 2077 |
| Groups in complete suite | **21** (was 20) |
| This audit a11y / modal / help / first-run / machine / storage / design | 36+26+37+19+50+15+1093 all **0 failed** |

Full `?selftest=all` single-shot was not re-run end-to-end in this session; high-risk groups and Designs geometry were re-verified green. Arithmetic and README match the implementation report.

---

## 24. Runtime / file:// validation

- Page loads to complete; title correct; release footer present.
- Accessibility fixtures execute without exceptions.
- Keyboard, isolation, slash, Space, rapid-arrow, click probes succeed.
- No page errors; console only pre-existing SVG `height="auto"` noise during design fixtures.

---

## 25. Responsive findings

- **320×720:** `scrollWidth === clientWidth` (320); no page horizontal overflow; tabs use horizontal scroll inside tablist; tab height ~45px.
- **1280:** no page overflow; active tab `aria-selected="true"`.
- No new global layout breakpoints; target CSS reuses existing flex-wrap patterns.

**Verdict:** **NOT A DEFECT.**

---

## 26. README / CHANGELOG findings

- README: documents tablist keyboard model, focus indicator, fixture total **2077**, `accessibility` selftest, `runAccessibilityPolishFixtures`.
- CHANGELOG: one accurate 0.9.0 bullet; no false WCAG/full-a11y/mobile/paid-release claims.
- Does not claim screen-reader certification.

**Verdict:** Honest. **NOT A DEFECT.**

---

## 27. Protected-boundary comparison

| Boundary | Diff impact |
|----------|-------------|
| STORAGE_KEY / SCHEMA_VERSION / APP_* / BUILD_DATE / BACKUP_FORMAT | Unchanged values |
| freshState / loadState / normalization / backupObject / merge-replace / import validation / recovery | Not modified (only referenced in fixtures) |
| machineIdentity / machineProfile behavior | Untouched |
| first-run dismissal persistence | Session-only still |
| Help content meaning | Untouched (fixtures only) |
| Modal focus-management logic | Unchanged helpers; routing order only |
| Form validation / Library / grids / projects / inventory / pricing semantics | Untouched |
| Designs geometry / SVG / downloads / Finished Views | Untouched; **1093/0** |
| Network / dependencies | None added |

**Expected change set matches actual:** tab semantics, keyboard nav, focus/target CSS, fixtures, README, CHANGELOG, implementation report.

---

## 28. Findings classified by severity

### BLOCKER

*None.*

### IMPORTANT

*None confirmed.*

### POLISH (safe to defer)

1. **Space key** not asserted in `runAccessibilityPolishFixtures` (behavior verified live).
2. **Optional `min-width`** on primary tabs so short labels (e.g. “Log”) approach square touch targets.
3. **Extend `:focus-visible` to `a[href]`** for Reference PDF/manual links consistency.
4. **Align `.help-tip:focus`** with the global 3px focus-visible style.
5. **Primary-button inner outline contrast** vs `#236d9c` is weak; optional lighter ring or box-shadow for primary only.
6. **Fixture could add** mouse-click assert, rapid-nav assert, Home/End-in-input assert, live computed outline style.
7. Full **`?selftest=all` 2077** single-shot re-run optional confirmation before release tagging.

### NOT A DEFECT

- Single shared tabpanel (not one panel per tab).
- Automatic activation model (not manual activation).
- Dense 34px table buttons vs 40px primary actions.
- Synthetic keyboard events in fixtures (honest limitation).
- No screen-reader certification claim.
- Disabled opacity de-emphasis.
- Horizontal tab scroll at 320px.

---

## 29. Exact final verdict

# APPROVED WITH POLISH DEFERRED

---

## 30. Remaining unverified areas

- Full 21-group `?selftest=all` in one process (partial groups verified).
- Real assistive technology (NVDA/JAWS/VoiceOver) announcement of tablist.
- Every browser engine beyond headless Chromium/Edge.
- Pixel-level outline clipping inside every overflow table.
- Manual high-zoom visual pass.
- Physical keyboard on all OS platforms.

---

## 31. Whether physical laser testing is required

**No.** This phase is UI accessibility only — no laser control, geometry, or production-output changes.

---

## 32. Whether Codex may proceed to commit

**Yes.** Codex may commit `index.html`, `README.md`, and `CHANGELOG.md` as reviewed. No correction commit is required for blockers/important defects. Deferred polish may follow in a later change. Another independent audit is **not required** unless the keyboard/focus model or CSS contracts are substantially revised.

---

## 33. Audit hygiene confirmation

This audit:

- Made **no** edits to product source (`index.html`, `README.md`, `CHANGELOG.md` left as found after inspection).
- Did **not** stage, commit, push, reset, clean, stash, checkout, move, rename, or delete any product file.
- Wrote **only** this audit report at  
  `C:\Genmitsu L8 Tracker\docs\ACCESSIBILITY_POLISH_FOCUSED_AUDIT_2026-07-19.md`.
- Used temporary worktree agent tooling for Playwright probes only.

---

*End of focused accessibility polish audit.*
