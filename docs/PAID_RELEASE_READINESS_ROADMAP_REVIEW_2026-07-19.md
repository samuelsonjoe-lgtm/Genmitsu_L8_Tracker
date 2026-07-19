# Paid-Release Readiness and Roadmap Audit — Genmitsu L8 Tracker

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer model:** Claude Opus 4.8
**Type:** read-only audit. No file was edited, staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

---

## 1. Repository state

- `git status`: on `main`, "up to date with origin/main." Only untracked files present (long-standing `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, prior `docs/*.md`). **No tracked file is modified.**
- HEAD full hash: **`56bff511215dbfea941cfe832ba51e808d6fa79d`** — "Add Designs to Projects handoff" — matches stated baseline `56bff51`.
- Branch `main`; ahead/behind origin/main: `0 0` (**fully synchronized**).
- `git diff` (unstaged) and `git diff --cached` (staged): both empty. Tracked tree is **byte-identical to `56bff51`**.
- `index.html`: 11,256 lines. `README.md`: 96 lines. `SCHEMA_VERSION = 2`; `STORAGE_KEY = 'genmitsu-l8-tracker-v1'`.
- Baseline fixture totals (1,093 / 17 / 1,902 / 264) taken as given; not re-executed (forward-looking audit).
- All unrelated untracked content preserved untouched.

## 2. Files and functions reviewed

Navigation/state: `tabs` registry (`:264-273`), `render()` (`:968-982`), `renderTabs()` (`:1011`), `setTab()` (`:550`), header/topbar markup (`:225-244`). Storage/backup/recovery: `freshState` (`:433`), `loadState` (`:436`), `persist` (`:470`), `decodeStoredState`, `backupObject` (`:516`), `replaceData` (`:10996`), `mergeData` (`:11006`), `mergeList`/`mergeListStats`, storage-recovery panel (`:999-1003`), `runStorageRecoveryFixtures` (`:494`). Import/export: header handlers (`:11202-11232`), export photo-size guard (`:11196`). Machine profile: `state.machineProfile` (`:434/465`), Reference machine selector (`:4957-5003`), `designProductionMachineKey` (`:1502`), Test Grid machine identity (`:7443-7507`). Tab renderers: `renderLog`, `renderLibrary`, `renderGrids`, `renderDesigns`, `renderReference`, `renderProjects`, `renderInventory`, `renderPricing`. Modals/wizards: `openModal`/`closeModal`/`bindModal` (`:9430-10511`), `openProject`, `openEntry`, `openProfile`, `openGridForm`, `openProjectWizard`, Material Test Wizard, promotion/evidence modals. Designs: `buildDesignResult`, `designResultsHtml`, `downloadCurrentDesignSvg`, finished-view builders, scale-safety notice (`:3621`). Accessibility/responsive: full CSS head (`:1-221`), single breakpoint (`:190`), print block (`:203`), `esc()` usage. Fixture registration (`:11235-11251`) and all 16 runners. README.md in full. Prior docs referenced: Designs-to-Projects handoff review, result-summary and form-organization reviews, Drawer Cabinet / New Project Wizard / accounting-separation reports.

## 3. Current product inventory

| Workflow | User problem | Maturity | Data written | Destructive actions | Recovery | Fixtures | Browser-validated | Physical-validation dependent | Status |
|---|---|---|---|---|---|---|---|---|---|
| **Log** | Record every job's settings | Mature | `entries[]` | delete (undo) | backup/undo | indirect | yes | no | **Release-ready** |
| **Library** (+ Browse, production settings, evidence) | Curated best-known settings + proven evidence | Mature | `profiles[]` nested | delete (undo/confirm) | backup/undo | Library/production/evidence fixtures | yes | records real results | **Release-ready** |
| **Test Grids** (+ Browse, machine identity) | Dial in new material via P×S matrix | Mature | `grids[]` | delete (undo) | backup/undo | grid/machine/browser fixtures | yes | yes | **Release-ready** |
| **Designs** (10 templates/modes) | Session-only parametric SVG for LightBurn | Mixed | none (session-only draft) | none persistent | n/a | 1,093 geometry + 264 tray | yes | **strongly** (cut/fit/strength) | **Functional but needs polish** (some templates experimental — see below) |
| **Reference** | Genmitsu starting points + safety/manual notes | Mature | none (machineProfile pref) | none | n/a | none | yes | starting points only | **Nearly ready** (Genmitsu-specific) |
| **Projects** (+ Browse, Wizard, accounting, photos) | Finished-piece gallery + costing | Mature | `projects[]` (+photos) | delete (undo) | backup/undo | project-browser/wizard/handoff | yes | no | **Release-ready** |
| **Inventory** (raw + batches, Browse) | Manual stock tracker | Mature | `inventory{}` | delete (undo) | backup/undo | material-browser fixtures | yes | no | **Release-ready** |
| **Pricing** | Cost/profit estimator | Mature | `pricing`,`pricingPrefs` | none destructive | backup | indirect | yes | no | **Release-ready** |
| **Import/Export/Recovery** | Portable backup + corruption safety | Mature-minus | localStorage/file | replace (confirm) | download-damaged/start-fresh | storage-recovery fixtures | yes | no | **Nearly ready** (no future-version guard, no format id) |

Designs template maturity sub-classification: `qr-stand`, `hanging-sign`, `dice-tray`, `divider-tray`, `finger-box`, `sliding-lid-box`, `drawer-cabinet` (linked/separate/custom-row) — **Release-ready generators**. `joint-fit-coupon` (both modes) — **Functional** (fit-learning tool; wall-to-base cannot promote yet). `concealed-cleat-corner-prototype`, `concealed-cleat-full-box-prototype` — **Experimental/Prototype** (fit/registration validated; **strength not validated**; clearly labeled as such). No workflow is marked release-ready on fixtures alone; Designs and Reference carry physical-validation caveats.

## 4. Release-readiness scorecard

| Dimension | State | Blocker? |
|---|---|---|
| Offline / no-network / opens via `index.html` | **Pass** — zero `fetch`/XHR/WebSocket/CDN; only `navigator.clipboard` (local) | — |
| Data persistence & backward compat | **Pass** — defensive `normalize*`, `schemaVersion` stored, additive v1→v2 | — |
| Corruption handling | **Pass** — `decodeStoredState` blocks silent overwrite; recovery panel | — |
| DOM-injection safety | **Pass** — `esc()` used pervasively on user content | — |
| Destructive-action safety | **Pass** — 13 `confirm()` guards + undo toast for deletes | — |
| Import future-version safety | **Gap** — no schema-version gate; imports any object, re-stamps v2 | **Pre-release essential** |
| Release identity (name/version/build/About/changelog) | **Gap** — none surfaced; no backup format id | **Pre-release essential** |
| First-run onboarding | **Gap** — no onboarding; good per-list empty states only | **Pre-release essential** |
| Accessibility — modals | **Gap** — no `role="dialog"`/`aria-modal`, no focus trap/restore | **Pre-release essential** |
| Accessibility — tabs/focus-visible | **Polish** — tabs lack `role="tablist"`; no global focus ring | Polish |
| Machine-neutrality | **Gap** — app profile is 20W/40W Genmitsu only | Pre-release essential *if* sold machine-neutral; optional if sold Genmitsu-specific |
| Responsive | **Nearly** — one 700px breakpoint; tables scroll; shell caps at 1180px | Polish |
| Maintainability | **Acceptable** — 11k-line single file, disciplined helpers, 16 fixture groups | Not a blocker |

**No architecture-level defect and no data-loss architecture flaw were found.** Remaining work is bounded polish/identity/onboarding/accessibility, not revision or rewrite.

## 5. First-run findings

A brand-new user (empty storage) lands on **Log** (`activeTab:'log'` in `freshState`). There is **no onboarding, welcome, sample data, guided setup, or backup-importance nudge** — searches for `sample/onboard/first-run/welcome` return nothing. Per-list empty states exist via the `empty(title, subtitle)` helper with helpful copy (e.g. "No entries yet — Log your first cut or engrave…"), but there is no app-level orientation: the user is not told storage is local-only/at-risk, is not walked to set a machine, add a material, or run a Test Grid, and is not shown where Export lives before they have data worth losing. Advanced surfaces (production settings, evidence promotion, concealed-cleat prototypes, custom-row cabinets) are not gated or de-emphasized for beginners. **Smallest useful onboarding phase:** a dismissible, session-or-flag-driven first-run panel on the Log tab (empty-state variant) that (1) states data is stored locally in this browser and to Export backups regularly, (2) links to the three starter actions (add a Log entry / plan a Test Grid / open Reference for your machine), and (3) points at Import/Export. No account, cloud, or network — purely a conditional render when all record arrays are empty.

## 6. Machine-neutral findings

`state.machineProfile` is a 2-value enum (`'20W'`/`'40W'`), Genmitsu-L8-specific, defaulting to `'20W'` (`:434`). The Reference tab and its starting-point tables are explicitly SainSmart/Genmitsu content (`reference20w`/`reference40w`, `:274+`). **No stored physical work-area/bed size exists anywhere** — the Designs >400 mm advisory explicitly disclaims being a bed size (confirmed at `:3621` and its neutrality fixture `:4915`). Test Grids already support custom machine identity and machine-less legacy grids (per README and `:7443-7507`), so provenance is machine-neutral there. Focus terminology is honest (fixed-focus helper shown only for 20W). Air-assist/power/speed guidance is presented as *starting points*, not universal settings. **Honest for other diode machines today only if the user treats Reference as Genmitsu-specific and enters their own numbers elsewhere.** For a machine-neutral paid release, place: machine identity + optional work-area in **first-run setup** and **machine profiles**; keep manufacturer starting-point charts in **Reference** (clearly attributed); keep per-result machine stamping in **test records** (already done); leave **production settings** machine-tagged (already done). Do **not** invent universal laser settings — only capture the user's own machine identity and (optionally, later) a work-area used for honest layout-vs-sheet comparison.

## 7. Data-safety findings

- **localStorage capacity**: `persist()` wraps `setItem` in try/catch → `showStorageWarning()` (advises export + photo removal) and returns `false`; callers that check the return avoid claiming success. Photos are the dominant pressure (compressed, ≤3 per project, resized ~1200px) with an export size warning (`:11196`) and a save-failure alert in `openProject`. **Acceptable.**
- **JSON parse / corruption**: `decodeStoredState` classifies empty/ok/corrupt; corrupt storage opens empty **and blocks saving** so damaged data is never silently overwritten; recovery panel offers download-damaged / import / explicit start-fresh. **Strong.**
- **Unknown/missing fields**: `normalize*` functions apply defaults and preserve unknown fields (material-browser fixture asserts "Unknown fields survive normalization" and "Repeated normalization stable"). **Good forward/backward tolerance.**
- **Older backups**: import path tolerates missing production settings (README-confirmed) via defensive normalization.
- **Replace vs merge**: `replaceData` overwrites arrays; `mergeData` merges by id via `mergeList`. Both confirmed. Duplicate IDs resolve through `mergeList` (id-keyed).
- **Realistic data-loss paths identified:** (a) **Future-schema import** — a `schemaVersion:3` backup imports into a v2 app with no guard and is re-persisted stamped v2, potentially dropping/mis-handling future fields silently (low probability today, one-directional, real). (b) **Replace-import mistake** — guarded by a confirm, but the confirm wording (see §8) is a double-negative and could be mis-clicked; there is **no pre-import preview of record counts** and **no automatic pre-replace safety snapshot**. Both are bounded to fix; neither is an architecture flaw.

## 8. Import/export and migration findings

Export: `backupObject()` emits all persisted state stamped with `schemaVersion`, with a photo-size confirm before download. **No app name, app version, or backup format identifier** is written — the file is identifiable only by its field shape. Import (`:11203-11232`): reads file → `decodeStoredState` (must parse to an object) → `confirm('OK = merge, Cancel = replace')` → for replace a second `confirm`. Findings: (1) **no format/app identification** — any object-shaped JSON is accepted; (2) **no schema-version validation or future-version handling** — `loadState` reads `schemaVersion: data.schemaVersion || 1` but never branches on it; (3) **no preview** of what will be merged/replaced before it happens (merge shows a summary toast *after*); (4) confirm UX is a **double-negative** ("Cancel = replace everything") that is easy to misread; (5) malformed records inside an otherwise-valid object are normalized rather than rejected (generally safe, but no per-record error reporting). **Migration layer:** a formal migration framework is **not required now** (only v1→v2 exists and normalization handles it) and should remain **deferred until the next real schema change** — but a **minimal future-version guard + backup format/app-version stamp** should land at/near release (see §19), because it is the cheap half of migration readiness and closes the one real import data-loss path.

## 9. Accessibility findings

**Release blockers (pre-release essentials):**
- **Modals are not dialogs.** Zero `role="dialog"`/`aria-modal="true"` in the file. Modal open (`openModal`, `:9442`) injects HTML and adds `.open` but (except `projectWizardModal`, `:8250`) does **not** move focus into the dialog, does **not** trap focus, and does **not** restore focus to the trigger on close. Screen-reader and keyboard users are not properly scoped to the modal. This spans every modal (entry/project/profile/grid/cell/test/inventory/production/evidence/wizard). This is the single most material accessibility item.

**Polish (not blockers):**
- Tabs are plain `<button class="tab">` with no `role="tablist"`/`role="tab"`/`aria-selected`/`aria-controls`; the `.tabs` bar scrolls horizontally (`overflow-x:auto`) so nothing clips.
- No global `:focus-visible` outline for buttons/inputs/tabs (only `.help-tip:focus` has one); relies on browser defaults.
- No `prefers-reduced-motion` handling (toast/modal transitions are minor).
- Color-only state is largely avoided: errors/warnings carry bold text prefixes; Designs results panel already has `role="status" aria-live="polite"`; SVG previews use `role="img"` + `<title>`/`<desc>`. Good baseline.
- Native controls (`<label>`-wrapped inputs, `<details>`/`<summary>`, real `<button>`s) are used throughout — strong foundation; the gap is dialog semantics, not custom-widget soup.
- Contrast: `--muted:#667085` on light backgrounds is ~AA for normal text; `--amber:#9a641d`/`--red:#b14242`/`--blue:#236d9c` as text on white pass AA. Spot-check only; a full contrast pass belongs in the accessibility phase.

## 10. Responsive findings

- Single breakpoint `@media (max-width:700px)` (`:190`) collapses `.formgrid`/`.metrics` to one column, stacks browser/design two-pane layouts, and full-widths modal action buttons. `.shell` caps at 1180px (large-desktop safe). `.tabs` and every `table` (`min-width:720px` inside `.tablewrap`/`.grid-table` with `overflow-x:auto`) scroll horizontally rather than overflow the page.
- **320 px:** functional via the 700px→1fr collapse; tables/matrix scroll inside their wrappers; no fixed-pixel form control blocks reflow. Touch targets: `.cell` 76×54 (fine); `.star` 30×30 and `.tab` (11px vertical padding) are on the small side for touch — polish.
- **~700 px / tablet:** covered by the breakpoint; no dedicated tablet tuning but no observed overflow.
- **Desktop / large desktop:** two-pane browser and Designs layouts use `minmax()` grids; content caps at 1180px — no stretched line-length problem.
- No clipped dialogs (`.modal` is `overflow-y:auto`, `align-items:flex-start` with padding; `.modal-actions` is sticky-bottom). **No release-blocking responsive defect; remaining items are polish (tablet tuning, touch-target sizing).**

## 11. Documentation findings

README.md (96 lines) is unusually thorough and honest — it documents every workflow, the storage model, recovery, safety caveats, fixtures, and the exact-scale/no-nesting limits. In-app help is present as `help()` hover/focus tooltips on laser/Project/Pricing fields and as always-visible `.small muted` caveats in Designs. **Gaps:** no in-app "About/Help" surface (version, backup instructions, LightBurn import workflow, safety summary) reachable without leaving the app; onboarding text (§5) absent; error-recovery guidance lives only in the recovery panel and README. **Placement recommendation:** keep the exhaustive narrative in **README**; add a compact in-app **About/Help panel** (version, "your data is local — export backups," LightBurn 100%-scale reminder, safety one-liner, link to README/manual); keep a **separate manual** (the bundled PDF) for deep laser guidance; **defer** any per-screen long-form manuals (avoid a warning wall). A **CHANGELOG.md** should accompany versioning (§9/§12).

## 12. Versioning findings

The app exposes **no** app name-as-version, version number, build date, release notes, changelog, About, or support/contact anywhere in the UI or data. `SCHEMA_VERSION = 2` exists in code and is written into storage/backups but is **not user-visible** and **not used as an import gate**. Backups carry `schemaVersion` but **no app version or backup-format id**. **Recommendation (offline, no build system required):** define `APP_VERSION` (semver string) and `BUILD_DATE` as literals near `SCHEMA_VERSION`; render them in a small footer/About surface; write `app:'genmitsu-l8-tracker'`, `appVersion`, and `backupFormat` **additively** into `backupObject()` (old backups lacking them still import; `loadState` ignores unknown keys); maintain `CHANGELOG.md`. Manual literal bumps are acceptable — no bundler needed. This is machine-readable (in JSON) and user-visible (footer/About) and works from `file://`.

## 13. Safety and honesty findings

Laser guidance is a notable strength: Reference includes explicit "cannot engrave / use another machine" and prep notes; material-safety warnings flag PVC/vinyl/chlorinated/unknown plastics/fiberglass/carbon fiber without blocking saves; Inventory Browse suppresses laser-action drafts for unsafe/mystery materials; Designs pervasively disclaims that clearances are starting points, that prototypes are "not a strength test," that kerf is separate, and that output must stay at 100% scale. **Gaps to close without building a warning wall:** there is no single consolidated **safety summary** (ventilation/fume extraction, fire watch / never-leave-unattended, material flatness, focus, unknown-coating/adhesive caution) surfaced in-app — it is scattered across Reference and Designs. Recommend one concise in-app safety note (in the About/Help panel and Reference header) covering ventilation, fire watch, flatness/focus, and unknown-material caution, cross-linking the manual — not repeated on every screen. Machine-travel/sheet-size honesty is already correct (the >400 mm advisory disclaims bed size). Prototype strength limits are already explicit. **No dishonest or missing-critical-hazard claim was found; the gap is consolidation, not correction.**

## 14. Workflow-integration findings

- **New material → Test Grid → winner → Library → Project:** works end-to-end (Grid cell promotion → Library draft/Material Test → Project from profile). Machine provenance carried. **Solid.**
- **Inventory material → Project Wizard → Project:** four-step wizard selects verified on-hand Inventory, ranks evidence, copies baselines unchanged, opens the normal Project form; never auto-deducts. **Solid.**
- **Design → Project → production:** new handoff (`Start Project from Design`) opens a reviewable draft (snapshot; material blank; thickness in mm; qty 1); download remains the separate production path. **Solid, newly landed.**
- **Project → Pricing/accounting:** Pricing→Project draft bridge and per-project accounting/CSV exist. Note the bridge is Pricing→Project (one-way); Project→Pricing is via manual copy. **Acceptable.**
- **Backup → restore:** export/import with merge/replace + recovery. **Functional; see §8 UX/guard gaps.**
- **Existing Project → edit → archive/history:** projects have `status` (kept/gifted/for-sale/sold) for lifecycle but **no explicit archive/version history** — edits overwrite in place. Ambiguous "ownership" of the settings snapshot (copied point-in-time, not linked) is intentional and documented. **Minor dead-end: no project history/audit trail** — acceptable for 1.0, notable for a platform release.
- **Multi-output Drawer Cabinet:** separate-thickness yields two named production downloads with explicit "two files required" messaging. **Solid.**
- **Coupon → record → later production:** finger-edge coupon can promote a physical winner; **wall-to-base coupon cannot promote** (must record manually) — a known, documented handoff gap and the clearest "record result then reuse" dead-end. **Bounded future item.**

No broken handoff or silent data-duplication was found; the notable gaps are wall-to-base coupon promotion and the absence of project history.

## 15. Empty-state findings

Per-list empty states are implemented via `empty()` with distinct "no records yet" vs "no matches" copy (Log confirmed at `:1027-1029`; the pattern recurs across tabs). Missing/deleted referenced records: browsers use id lookups that return null safely (material-browser fixture: "ID lookup remains ID based"), and Designs handoff/normalization tolerate absent references. Storage-full, malformed-import, and corrupt-storage all have explicit handling (§7/§8). **Gaps:** no app-level first-run empty state (§5); "missing referenced record" cases (e.g., a Project's `sourceProfileId` pointing at a deleted profile) degrade quietly rather than surfacing a friendly "source no longer exists" note — minor. **Recommend** folding the first-run orientation into the Log empty state and adding light "source record no longer available" notes where a dangling reference is displayed.

## 16. Maintainability findings

Single 11,256-line `index.html`. Strengths: consistent helper vocabulary (`field`/`select`/`designSelect`/`labelText`/`help`/`empty`/`metric`), one dispatch per concern (`render()`, `buildDesignResult()`), protected production boundaries proven by byte-identity fixtures, and **16 registered fixture groups** discoverable via `?selftest=` and `window.run*Fixtures` (README documents the arithmetic `809 + 1093 = 1902`). Risks: (1) the file's size raises the odds of **accidental cross-tab regressions** when shared helpers change — mitigated by the broad fixtures; (2) **report/document drift** — the `docs/` tree holds dozens of point-in-time reports and the README embeds exact counts that must be hand-updated; (3) **test-runner arithmetic** is manual (a miscount is possible — one such discrepancy was caught in a prior session). **Recommendation:** do **not** rewrite or split into modules for its own sake. If any extraction is done, prefer bounded, behavior-preserving moves (e.g., isolating the fixture registry or the CSS) only where it demonstrably lowers regression risk. Keep the single-file, `file://`-openable constraint.

## 17. Performance and capacity findings

- **Proven-safe at expected scale:** localStorage-bound; the debounced pricing persist and search-only partial re-renders (Log/Projects/Library/Grids/Inventory) already avoid full-tab rebuilds on keystroke — evidence of prior attention to interactive cost.
- **Theoretical/at-scale concerns (not observed problems):** whole-`state` JSON serialization on each `persist()` grows with record + photo count; a user with hundreds of photo-bearing projects approaches localStorage limits (bounded by the ≤3-photos-per-project + compression + size warnings). Large Designs SVGs render via `<object>` data URIs — fine for single designs. Backup import parses the entire file synchronously (acceptable for realistic sizes). `file://` startup is instant (no bundler/network).
- **Distinction:** the only *capacity* risk with real teeth is **photo-driven localStorage exhaustion**, and it is already surfaced (export size warning + save-failure alert + recovery). No CPU/render performance defect found.

## 18. Privacy/security findings

Offline threat model is favorable: **no network calls** (no `fetch`/XHR/WebSocket/CDN/font/analytics; only local `navigator.clipboard`), so no exfiltration surface. User content is escaped via `esc()` pervasively before DOM insertion (spot-checked across Designs, Library, Grids, Projects) — DOM-injection risk is low. Imported JSON is parsed with `JSON.parse` (no `eval`), object-shape-validated, and normalized. Designs SVG is generated internally (not imported), and previews render generated SVG in `<object>` data URIs. CSV export quotes fields via `csvRows`. External links use `target="_blank" rel="noopener"`. **Credible residual risks (low):** (1) a **malicious/oversized backup file** could carry hostile strings — mitigated by `esc()` at render, but a hostile `schemaVersion`/field could interact with the missing import guard (§8) — the format/version guard hardens this; (2) imported **notes/material strings** are trusted as data and only made safe by escaping — acceptable given no network sink; (3) project **photos** are user-provided data URIs stored/rendered locally — bounded. **No release-blocking security defect.** The format/version import guard (§19) is the one security-adjacent hardening worth doing.

## 19. Reconciled roadmap

| Item | Classification | Evidence-based note |
|---|---|---|
| Additional hidden joint types | Post-release feature | No current gap; capability, not readiness |
| More decorative joint treatments | Experimental/deferred | Decorative-only precedent exists (faux dovetail) |
| Saved Design presets / Library integration | Post-release feature | Explicitly deferred in handoff review; needs schema thought |
| Drawer Cabinet mixed-width rows | Experimental/deferred | Out of scope of custom-row design; not readiness |
| Drawer Cabinet vertical-spanning layouts | Experimental/deferred | Same |
| Automatic nesting | Should not be pursued (now) | README already disclaims; large risk, low readiness value |
| Multi-sheet export | Post-release feature | Disclaimed; not readiness |
| **Machine-neutral setup** | **Useful before release** (essential *if* sold machine-neutral) | 20W/40W-only profile; honest today only as Genmitsu-specific |
| **Onboarding / first-run** | **Required before paid release** | No onboarding exists (§5) |
| **GUI cleanup / accessibility (modals)** | **Required before paid release** | No dialog semantics/focus management (§9) |
| **Documentation (About/Help + safety consolidation + changelog)** | **Required before paid release** | No in-app About/version/safety summary (§11-13) |
| **Release packaging (version/build/format id + import guard)** | **Required before paid release** | No version identity; no future-version import guard (§8/§12) |

Explicitly acknowledged as already done: **custom-row Drawer Cabinet is implemented**; **Finished Views are complete for all meaningful current templates**; **Designs-to-Projects handoff is implemented**; **concealed-cleat fit is validated but strength is not** (and is labeled accordingly).

## 20. Candidate release plans

**Plan A — Minimum credible paid release.** Phases: (1) Release Identity & Import Safety (version/build/About + backup format id + future-schema import guard); (2) First-run onboarding + backup-importance nudge; (3) Modal accessibility (dialog semantics, focus trap/restore); (4) In-app About/Help + consolidated safety note + CHANGELOG. Deferred: machine-neutral setup (ship as "Genmitsu L8-focused"), all new Designs capabilities. Data/schema impact: **additive only** (no key renames, no migration). Review depth: per-phase Grok, with Opus for the import/backup phase. Physical validation: none beyond existing Designs caveats. Risks: low; smallest set that makes it honestly *sellable* as polished Genmitsu-L8 software.

**Plan B — Strong polished 1.0 (recommended).** Plan A **plus** (5) Machine-neutral setup (custom machine identity + optional work-area, Reference clearly attributed); (6) Accessibility polish (tab roles, global focus-visible, contrast pass, touch targets); (7) Responsive tablet/large-screen tuning; (8) Empty-state/dangling-reference polish. Deferred: new joints, presets, nesting, multi-sheet, project history. Schema impact: additive (machine identity/work-area are new optional fields with defaults). Review depth: Grok per phase; Opus for any phase touching storage/schema/import. Physical validation: none new. Risks: moderate breadth, each phase bounded and fixture-gated. **This is a defensible, genuinely finished 1.0.**

**Plan C — Extended workshop platform.** Plan B **plus** wall-to-base coupon promotion, saved Design presets/Library integration, project history/versioning, and selected new Designs capabilities (mixed-width rows, etc.). Schema impact: **real new schema + migration layer required.** Review depth: Opus for every schema/migration phase; physical validation for any new joint/strength claim. Risks: high; explicitly **post-1.0**.

**Recommendation: Plan B**, executed A-first so the app is releasable after Plan A's four phases and reaches a polished 1.0 through phases 5-8. Do not pursue Plan C before 1.0.

## 21. Exact prioritized roadmap

**BLOCKERS (must land before any paid release):**

**P1 — Release Identity & Import Safety.** *Problem:* paid software needs a visible version/About and must not silently mishandle a future/foreign backup. *Why first:* lowest risk, closes the one real import data-loss path (§8), and other phases reference the version/About. *Scope:* add `APP_VERSION`+`BUILD_DATE` literals near `SCHEMA_VERSION`; render a small footer/About surface; write `app`/`appVersion`/`backupFormat` **additively** into `backupObject()`; in the import handler add a `schemaVersion > SCHEMA_VERSION` guard that refuses with a clear message and an optional format/app sanity check. *Files/systems:* header/footer markup, `backupObject`, import `onchange` (`:11203`), a CHANGELOG.md. *Storage/schema risk:* additive only (no rename/migration; old backups without new fields still import). *Production-output risk:* none. *User-data risk:* net **reduction** (guards a loss path). *Implementer:* Codex. *Reviewer:* **Claude Opus 4.8** (touches import/export + backup bytes). *Fixtures:* backup contains new fields; old backup (no fields) still imports; future-version backup is refused; export byte-shape stable for current data; round-trip merge/replace unchanged. *Physical testing:* none. *Gate:* **release gate.**

**P2 — First-run onboarding & backup nudge.** *Problem:* new users get no orientation and don't learn data is local/at-risk (§5). *Why here:* highest user-facing readiness value; benefits from P1's version/About. *Scope:* conditional first-run panel in the Log empty state (all record arrays empty) with local-storage/export message and three starter links; dismissible via a session flag (not persisted schema). *Files/systems:* `renderLog`/`empty()` path, a session-only flag. *Storage/schema risk:* none (session-only). *Production risk:* none. *User-data risk:* none. *Implementer:* Codex. *Reviewer:* Grok. *Fixtures:* panel shows only when empty; hidden once any record exists; dismiss is session-only and not persisted/backed-up. *Gate:* **release gate.**

**P3 — Modal accessibility.** *Problem:* modals aren't dialogs; no focus trap/restore (§9). *Why here:* the one accessibility **blocker**; moderate cross-tab surface so it follows the smaller phases. *Scope:* add `role="dialog"`/`aria-modal="true"`/`aria-labelledby` to modal containers; on open move focus to the first control; trap Tab within the open modal; restore focus to the trigger on close; reuse the existing Escape-close. *Files/systems:* `openModal`/`closeModal`/`bindModal`, modal markup. *Storage/schema risk:* none. *Production risk:* none. *User-data risk:* none. *Implementer:* Codex. *Reviewer:* Grok (Opus only if the change unexpectedly touches shared render/state). *Fixtures:* focus enters on open, is trapped, restores on close; Escape still closes; no duplicate-id regressions. *Gate:* **release gate.**

**P4 — In-app About/Help + consolidated safety note + CHANGELOG.** *Problem:* no in-app version/backup/safety summary (§11-13). *Scope:* compact About/Help surface (version, "data is local — export backups," LightBurn 100%-scale reminder, one safety paragraph: ventilation/fire watch/flatness/unknown materials, link to README/manual); README/CHANGELOG updates. *Storage/schema risk:* none. *Production risk:* none. *Implementer:* Codex. *Reviewer:* Grok. *Fixtures:* light (presence/version-string render). *Gate:* **release gate.**

**PRE-RELEASE ESSENTIALS (for a machine-neutral 1.0; optional if sold Genmitsu-specific):**

**P5 — Machine-neutral setup.** *Problem:* app profile is Genmitsu 20W/40W only (§6). *Scope:* optional custom machine identity (name) and optional work-area in first-run/profile; Reference clearly attributed as Genmitsu starting points; keep result/production machine stamping. *Storage/schema risk:* **additive** optional fields with defaults; no rename/migration. *Production risk:* none (work-area is advisory only; no auto-nesting). *Implementer:* Codex. *Reviewer:* **Opus** (schema-adjacent). *Fixtures:* defaults for legacy records; backup round-trip; no change to Designs bytes. *Gate:* release gate **iff** sold machine-neutral; else optional.

**POLISH (post-blocker, pre- or post-1.0):**

**P6 — Accessibility polish** (tab `role="tablist"`/`aria-selected`, global `:focus-visible`, contrast pass, touch-target sizing). **P7 — Responsive tuning** (tablet/large-screen, `.star`/`.tab` touch targets). **P8 — Empty-state/dangling-reference polish** (friendly "source record no longer available" notes). All: schema-neutral, Codex + Grok, optional gates.

**POST-RELEASE FEATURES:** wall-to-base coupon promotion; saved Design presets/Library integration; project history/versioning; additional joints/decorative treatments. **EXPERIMENTS (do not pursue before 1.0):** automatic nesting, multi-sheet export/arrangement, mixed-width/vertical-spanning cabinet layouts.

## 22. Immediate next phase

**P1 — Release Identity & Import Safety** (defined above). It is bounded (a few literals, one About/footer surface, an additive `backupObject` change, one import guard, a CHANGELOG), high value (establishes paid-software identity and closes the only real import data-loss path), low risk (additive, schema-neutral, well-fixtured), directly tied to release readiness, and supported by source evidence (§8/§12: no version anywhere; import accepts any object and re-stamps v2).

## 23. Risks

- **Cross-tab regression** from shared-helper edits (mitigated by 16 fixture groups; keep changes bounded).
- **Scope creep** turning P1/P4 into a settings-tab rewrite (hold to additive About/footer + guard).
- **Over-warning** in P4 (one consolidated safety note, not per-screen walls).
- **Machine-neutral honesty** (P5): work-area must stay advisory — never imply bed-fit/nesting the app doesn't do.
- **Document drift**: version/count strings in README must be updated in lockstep (CHANGELOG helps).

## Final verdict

**READY TO EXECUTE RELEASE ROADMAP**

The application is offline-safe, data-safe (corruption-guarded, escaped, undo/confirm-protected), broadly fixtured, and honest about physical limits, with **no architecture defect and no data-loss architecture flaw**. What remains between it and a credible paid release is a bounded set of identity/onboarding/accessibility/documentation phases — not revision or rewrite. Execute Plan B, P1 first.

- **Immediate next phase implementation report filename:** `docs/RELEASE_IDENTITY_IMPORT_SAFETY_IMPLEMENTATION_2026-07-19.md`
- **Immediate next phase audit filename:** `docs/RELEASE_IDENTITY_IMPORT_SAFETY_FOCUSED_AUDIT_2026-07-19.md`
- **Should Codex implement directly?** Yes — scope is small and fully enumerated (§21 P1).
- **Is Grok sufficient afterward?** No — because P1 touches the import decision path and backup bytes, a **Claude Opus 4.8** implementation review is warranted; Grok alone is insufficient for this phase.
- **Should Claude Opus 4.8 review the implementation?** Yes, for P1 (and P5); for the schema-neutral polish phases P2/P3/P4/P6/P7/P8, Grok is sufficient.
- **Is Fable 5 unnecessary?** Yes.
