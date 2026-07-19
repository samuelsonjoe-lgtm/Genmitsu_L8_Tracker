# Beginner Friction and First-run Usability Audit — Genmitsu L8 Tracker

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-19
**Reviewer:** Claude Opus 4.8
**Type:** read-only usability audit. No file was edited, staged, committed, pushed, or otherwise changed. A disposable headless Edge profile was used for all interaction; no real user profile or data was touched.

---

## 1. Repository state and baseline

- `git status -sb`: `## main...origin/main`; tracked tree **clean**; only the long-standing untracked set present (`LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, historical `docs/*.md`).
- HEAD: **`20325e779689d7c37fabfbec1fdb23ad5c14459e`** — "Improve empty states and unavailable references" (matches baseline `20325e7`).
- Branch `main`; ahead/behind origin/main: `0 0`. Nothing staged (`git diff --cached` empty). `git diff --check` clean.
- Constants: `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `SCHEMA_VERSION=2`, `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.
- **Fixture baseline verified at runtime: 2182 / 0.** Dynamic enumeration found 25 `run*Fixtures` groups totaling 2462/0; the `selftest=all` complete suite runs 23 of them (excluding the standalone `runTrayModelFixtures` = 264 and `runPromotionTargetSwitchFixtures` = 16): `2462 − 264 − 16 = 2182`, matching the stated baseline. All groups pass with zero failures. No product edit, commit, or push occurred; unrelated files preserved.

## 2. Audit environment and methodology

Disposable clean Edge profile (`--headless=new`, fresh `--user-data-dir`), opened directly via **`file:///C:/Genmitsu%20L8%20Tracker/index.html`** (true `file://`, verified by the CDP tab URL). A blind first-run pass was performed before reading source/README/Help; the app was then driven through real workflows via CDP (`Runtime.evaluate`, DOM clicks, form submits, emulated viewport widths). Source, README, Help, and fixtures were inspected afterward only to verify observed behavior. `state` is correctly IIFE-encapsulated (not global), so all verification used the DOM and `localStorage` as a real user's tooling would.

## 3. Blind first impression

Landing screen (clean profile): title **"Genmitsu L8 Tracker"**, subtitle "Standalone offline materials, power, and settings log"; header controls **in / mm / Import / Export / Help**; eight tabs **Log · Library · Test Grids · Designs · Reference · Projects · Inventory · Pricing**; default tab **Log**. A prominent first-run panel **"Welcome to Genmitsu L8 Tracker"** states the app works offline, data is stored **locally in this browser—not in the cloud**, urges regular **Export** backups (restorable with Import), and offers three starter actions plus **Hide this introduction**. Below it the Log empty state reads "No entries yet … Log a job." A footer shows **v0.9.0 / About** with build/schema/backup-format and "Works offline and stores records locally in this browser."

Blind verdict on the primary questions: within one minute a beginner learns (1) what the app is (offline settings/materials log), (2) where data lives (this browser, not cloud), and (5) the safe first action (Log a job / the three starter buttons). What the landing does **not** state, and which only Help contains, is (3) that the Tracker **does not control the laser** — see Finding F-12 (POLISH). Everything clickable is a real, labeled button or tab; nothing important is icon-only.

## 4. Persona findings

**Persona A (new laser owner).** Onboarding, empty states, and tooltips serve this persona well: every form field carries a plain-language `?` tooltip ("Cut goes through material; engrave marks the surface."). The main friction is conceptual, not mechanical: the difference between a **Log entry** and a **proven Library setting**, and between a **Test Grid** and a **Material Test**, is never explained in-app (F-3). The Log entry form's ~19 fields (only Material required) can feel intimidating (F-4), and "Focus height" vs "Material height / Z-offset" is opaque (F-5).

**Persona B (LightBurn-aware hobbyist).** Moves quickly; the Library tab leading with a "Find closest saved setting" match tool above an empty library is mildly odd (F-6). This persona is most likely to use Import to bring existing data and is therefore most exposed to the Merge/Replace confirm wording (F-1). They benefit from the strong Designs/LightBurn Help, but the inline Designs result omits the red=cut / blue=score color→operation mapping (F-7).

## 5. Task-by-task results

Format per task: outcome + friction + smallest correction.

**Task 1 — Data & backup.** Onboarding + footer + Help all state offline/local/no-cloud and urge Export; Import offers Merge/Replace; Help explains photos ride inside backups and external LightBurn/PDF files do not. A beginner can learn this **without** the README. **Friction:** Merge-vs-Replace mental model is set by a single ambiguous confirm (F-1). Otherwise strong.

**Task 2 — First material / Library profile.** "Add Material" opens a profile form; empty state and toolbar both offer "Add a Library profile." **Friction:** the tab's first heading is the match helper, not "Library" (F-6); "material," "profile," and "Library setting/profile" are used somewhat interchangeably (F-3/terminology). Completable.

**Task 3 — Record an engraving (Log).** "New Entry" / "Log a job" opens "New log entry"; only Material is required; empty submit yields "Add a material name first." and creates no record (verified). Found again in the Log list. **Friction:** field density (F-4); Log-vs-Library distinction unexplained (F-3).

**Task 4 — Test Grid.** "New test grid" exposes name/material/ranges/steps and shows "Machine: Genmitsu L8 20W" auto-snapshot; primary "Create grid." **Friction:** a beginner isn't told how a Test Grid differs from a Material Test (F-3), and "Custom machine label" introduces machine identity without context (F-11). Physical-inspection guidance exists in Reference/Help but not adjacent to the grid.

**Task 5 — Material Test workflow.** Reached from Library (per-profile or the toolbar "Material Test Wizard"). On an empty Library the toolbar button shows a clear blocking alert: **"Add or save a Library material before starting a material test."** — a correct prerequisite message, though delivered as an `alert()` rather than an inline/disabled hint (F-9). The distinction from Test Grids is the central unexplained concept (F-3).

**Task 6 — Promote a proven production setting.** Requires existing evidence (a completed Material Test / grid winner / etc.). With an empty workspace there is nothing to promote; the promotion entry points live inside Library profile detail and the grid cell modal ("Save to library," "Promote selected result"). Reference tables are clearly labeled manufacturer starting points, so the "starting point vs proven" line is reasonably drawn. **Friction:** the vocabulary chain evidence → winner → promotion → proven/production setting is dense and unexplained for beginners (F-3/terminology), but the workflow is gated (you cannot promote nothing), so there is no silent dead-end.

**Task 7 — Create a Project.** "New project" uses collapsible sections **Details / Photos / Settings snapshot / Accounting / Notes** — good progressive disclosure; only name+material required. **Friction:** whether a Project is a quote, a production record, or a job tracker is left implicit ("Finished pieces with their exact repeatable settings snapshot" leans "record"), which is fine but could be stated (POLISH, folded into F-3).

**Task 8 — Inventory.** Clear: "Manual raw materials and finished batches. Nothing auto-deducts from Projects." Add raw material / finished batch both reachable; empty state guides. Double-counting risk is explicitly pre-empted by the "nothing auto-deducts" subtitle. Strong.

**Task 9 — Pricing.** "Quick project estimate for revenue, costs, profit, margin, and break-even price"; Apply/Save defaults, Print, Export CSV, **Create project from estimate**. Honest and self-contained. No misleading authority observed. Good.

**Task 10 — Generate a Design.** Opens with a live QR-stand preview and an enabled "Download LightBurn SVG"; result shows Production SVG size, Operations (Cut/Score/Labels counts), finished size, and a clear **exact-scale 100%** notice. "Start Project from Design" present. Help's "Using Designs with LightBurn" covers 100% scale, operation assignment, framing, and "does not send jobs to the laser." **Friction:** the result never states the color→operation mapping (red=cut, blue=score/engrave) inline (F-7); a beginner with a multi-layer box could misassign layers.

**Task 11 — Configure workshop machine.** Reference hosts "Current workshop machine" (name + advisory mm work area) clearly separated from the relabeled **"Genmitsu L8 Reference profile"** selector, with advisory-only wording. Work area does not affect Designs/fit (stated). Low risk of confusing the 20W/40W selector for one's actual machine, given the new attribution copy. Well-handled.

**Task 12 — Recover from mistakes.** Validation blocks empty required fields with actionable text and preserves the open form; Cancel discards cleanly; deletes confirm and offer an undo window; delete-of-source messages state "Its source records will not be changed"; empty filters show "No … match" copy; unavailable/dangling references are handled (the `runEmptyStateDanglingReferenceFixtures` group, 60/0, covers this). Strong recovery posture.

**Task 13 — Import a backup.** Future-schema, foreign-app, and foreign-format backups are refused with clear messages ("This backup was created by a newer data format…", "…does not appear to be a supported Genmitsu L8 Tracker backup."); malformed files show "Could not read that file…". Merge keeps local machine setup; Replace restores it (documented). **Friction:** the mode chooser is a single `confirm('Import mode: OK = merge with current data, Cancel = replace everything.')` where **Cancel performs Replace** rather than aborting (F-1) — the sharpest beginner-usability issue in the app.

## 6. Information architecture findings

| Tab | Beginner guess | Actual | Label clear? | Order sensible? | Prerequisite | Empty state explains it? |
|---|---|---|---|---|---|---|
| Log | Record jobs | Journal of every job | Yes | Yes (first) | None | Yes ("Log a job") |
| Library | Saved materials | Curated best-known settings + Material Tests + proven production settings | Adequate | Yes | Benefits from Log/Reference input | Yes, but match helper shown first (F-6) |
| Test Grids | Test patterns | Power×speed matrix planner | Adequate | Yes | None | Yes ("Plan a Test Grid") |
| Designs | Make designs | Session-only parametric SVG generators | Yes | Reasonable | None | N/A (live default preview) |
| Reference | Help charts | Genmitsu starting-point tables + focus helper + machine setup | Yes | Reasonable | None | N/A |
| Projects | Saved projects | Finished-piece gallery + accounting | Yes | Yes | Benefits from materials | Yes ("Add a Project") |
| Inventory | Stock | Manual raw/finished tracker | Yes | Yes | None | Yes ("Add raw material") |
| Pricing | Price calc | Cost/profit estimator | Yes | Yes | None | N/A |

Tab labels are clear and should **not** be renamed. The only IA friction is cross-concept, not cross-tab: Test Grids (tab) vs Material Test (inside Library) are two testing systems whose relationship is never stated (F-3), and Library leads with an empty-dependent tool (F-6). No workflow forces undocumented tab-hopping to *complete* a task; the empty states now generally point to the next action.

## 7. Terminology inventory

Well-defined in-context (tooltip/label/Help): Log entry, Thickness, Job type, Passes, Air assist, Kerf offset, Exact-scale/Production SVG, Finished View, Merge, Replace, Backup format, Current workshop machine, Genmitsu L8 Reference profile, Raw inventory, Finished batch.

Unclear or unexplained for beginners:
- **Test Grid vs Material Test** — two testing concepts, no in-app "which/when" (F-3, IMPORTANT).
- **Evidence / promotion / proven / production setting / winner / baseline / discovery** — a dense chain surfaced in Library/promotion UI without a one-line primer (F-3).
- **Focus height vs Material height / Z-offset** — both in the Log/entry form; difference opaque; this is the flagged future terminology cleanup. Tooltip-only clarification recommended; **do not rename the stored keys** (`focusValue`/`focusUnit`, `materialHeightValue`/`materialHeightUnit`) — that risks backward-compatibility/backup confusion (F-5, POLISH / product decision).
- **"Save" (Reference rows)** — bare "Save" that actually copies a manufacturer row into the Library; inconsistent with the grid cell modal's "Save to library" (F-2, IMPORTANT).
- **Library "profile" vs "material" vs "Library setting"** — used interchangeably; minor (folded into F-3).

No term change should proceed without checking it is label/tooltip-only; none of the recommended wording fixes touch the data model.

## 8. Discoverability findings

Discoverable and well-placed: New Entry / Log a job, Add Library profile (toolbar + empty state), Plan a Test Grid, Add Project, Add raw material / finished batch, Export, Import, **Help** (header), Design generate/download, Start Project from Design, machine setup (Reference), recovery tools (appear only when storage is damaged — correct).
Lower discoverability (acceptable): Start Material Test (inside Library, per-profile or toolbar); Promote setting (inside Library profile detail / grid cell modal) — both are correctly gated behind having a record to act on. Self-tests are URL/console only and **not** user-facing (correct; see §18).
**Reference "Save" per-row copy-to-Library** is discoverable but its *meaning* is not (F-2). Beginners are not expected to hunt for anything essential without guidance.

## 9. Defaults and assumptions

Inspected defaults are safe and mostly honest: unit `in`; `machineProfile='20W'` (now clearly a *Reference profile*, not a claim about the user's machine); machine identity falls back to the Genmitsu label only for display and is never written into records; new Test Grid auto-snapshots "Genmitsu L8 20W"; Designs ship a valid QR-stand default (thickness 3 mm, fit clearance 0.15 mm) with the exact-scale notice; Project status defaults `kept`; Pricing starts empty/zero and is presented as an estimate. **Risk to watch:** Reference table values and Designs default clearances are *starting points* a beginner could mistake for proven — mitigated by the "manufacturer starting points, not universal settings" and "Official values are starting points" copy, and by Designs' testing caveats. No default silently creates a persisted record (verified: empty Log submit creates nothing).

## 10. Safety and LightBurn knowledge findings

Safety communication is a strength. Help covers ventilation/fume extraction, fire watch, never-unattended, PVC/vinyl/chlorinated/unknown-coating prohibitions, focus/flatness/framing, and explicitly "**does not send jobs to the laser**"; Reference repeats a material-safety reminder and "starting points" caveat; material-name fields warn on risky terms. The app does not invite an unsafe job through unclear UI. **Two residual LightBurn-knowledge gaps for beginners:** (a) the color→operation mapping (red cut / blue score-engrave) is not stated on the Designs result, only generically in Help (F-7); (b) "does not control the laser" lives only in Help, not on the landing/Designs surface where a beginner forms the assumption (F-12). Neither creates a *destructive* path given the 100%-scale notice and validation, so both are POLISH.

## 11. Error/warning/confirmation findings

Generally exemplary: validation errors state the fix ("Add a material name first."); import refusals explain the cause and risk; delete confirmations name the record and add an undo window; the source-delete message states "Its source records will not be changed"; storage-recovery copy explains that saving is blocked to protect damaged data. No message exposes internal IDs or developer jargon to the user in the paths exercised. **The one weak message is the import mode confirm (F-1):** it overloads "Cancel" to mean "Replace everything," so it fails the "what to do next / what will change" test for a beginner who wants to abort.

## 12. Dead-end findings

No true silent dead-ends were found. Potential dead-ends are handled: the Material Test Wizard on an empty Library alerts the prerequisite rather than doing nothing (F-9, but delivered via `alert()`); promotion is gated behind having evidence; empty lists show actionable empty states; dangling/unavailable references are covered by dedicated fixtures. The smallest residual improvement is converting the MTW `alert()` to an inline disabled-with-reason control (F-9, POLISH).

## 13. First-run onboarding findings

Appears at the right time (empty workspace, Log tab), sequences to a beginner's likely goals (log a job / plan a test / open Reference), uses understandable actions, introduces a bounded number of concepts, and offers a clear "Hide this introduction" (session-only; returns on reload while empty, and naturally disappears once any record exists — verified by the first-run fixtures). Backup guidance appears early (in the panel and footer). It does **not** re-show after dismissal within a session; Help is the persistent fallback, which is acceptable (do not add persisted onboarding-completion state without a concrete problem). Sample data or a guided first-project walkthrough would help Persona A but is a **product decision**, not a defect. Machine setup correctly lives in Reference, not first-run (it is optional and advanced).

## 14. Help and README findings

Help is discoverable (header + Reference callout), uses UI-consistent terminology, and answers the top beginner questions (offline/local storage, Export timing, Merge/Replace, 100% scale, LightBurn import, safety, starting-points-vs-proven, "does not send jobs to the laser"). README is thorough and consistent with the UI. **Content that exists in README/Help but not where a beginner first needs it:** the "does not control the laser" statement (only in Help; F-12) and the color→operation mapping (F-7). No Help section is too technical for first-run; it is appropriately layered (About → data/backups → Designs/LightBurn → safety → starting points).

## 15. Responsive beginner findings

At emulated **320 / 390 / 768 px** and desktop: **no horizontal page overflow** (body scrollWidth ≤ viewport at every width) and the entry modal fits within the viewport at all widths (dialog right edge inside the frame). First-run action buttons wrap cleanly (4 buttons). Header actions, tabs, and the machine-setup panel remain reachable and legible. The recent "Improve responsive workshop layouts" work is effective; local table scrolling (Reference) is intentional and not a defect.

## 16. Accessibility friction findings

Native controls throughout: real `<button>`s, `<label>`-wrapped inputs, `<details>`/`<summary>` disclosures, `role="dialog"`/`aria-modal` on modals with focus containment and restoration (the `runModalAccessibilityFixtures` 26/0 and `runAccessibilityPolishFixtures` 36/0 groups exercise these). Keyboard tab navigation, focus visibility, error text, and action names are present and pass their fixtures. For a beginner this *reduces* friction (predictable focus, clear labels). No accessibility behavior was observed to confuse a beginner. (No formal certification is claimed.)

## 17. Data-safety findings

Fear of data loss is reasonably addressed: Export is in the header and urged by onboarding/footer/Help; backup wording is honest ("not automatic"); Replace is behind two confirmations; recovery blocks silent overwrite of damaged data; storage limits and photo weight are explained; imports of unsupported files are refused clearly; dangling references degrade gracefully. **Where reassurance is thin:** persistence success/failure is mostly invisible — saves are silent (a failure raises a storage warning, but routine success gives no "saved" indicator), so a cautious beginner cannot always tell their latest work is safely stored. This is honest (it does persist) but a small, honest "saved" affirmation after meaningful actions would reduce anxiety without misleading (POLISH, F-13). Reassurance must not overpromise (no "backed up to cloud" language — currently correctly absent).

## 18. Self-test and diagnostic findings

Self-tests are developer-only: they run via `?selftest=…` query values or console `run*Fixtures()` calls and are not surfaced in the UI. This is correct — do not expose them to beginners. An in-app "Run diagnostics" action could have future support value but belongs in a later phase, not this one, and only if paired with plain-language output.

## 19. Fixture / runtime results

- `git diff --check`: clean. **Direct `file://` startup**: loads to `readyState complete`, title correct, first-run panel renders on a clean profile.
- **`?selftest=all` equivalent** (all 23 registered groups, run via automation): **2182 / 0**. Full dynamic enumeration of all 25 groups (incl. standalone Tray 264 and promotion-switch 16): **2462 / 0**, zero failures. Relevant beginner-focused groups all green: first-run 19/0, empty-state/dangling-reference 60/0, responsive tuning 45/0, accessibility polish 36/0, modal accessibility 26/0, Help/safety 37/0, machine setup 50/0, storage recovery 15/0, Designs geometry 1093/0.
- **Console noise:** two SVGs use `height="auto"` (an invalid SVG length), producing browser console warnings. Per instruction this is **not** treated as a fixture failure and was **not** edited; it is logged as F-8 because red console output can make a curious beginner or support helper think the app is broken.
- Console-error capture during fixture execution returned no error-level messages from app logic.

## 20. Findings classified by severity

**BLOCKER (0):** none. No realistic data-loss path (Replace is double-confirmed; recovery guards damaged data), no unusable core workflow, no unsafe-laser invitation, no un-warned invalid production output, and the required first-run path completes.

**IMPORTANT (3):**

- **F-1 — Import mode confirm overloads "Cancel" to mean "Replace everything."** Persona A/B · Import. Repro: Import → select any JSON → the first dialog reads "Import mode: OK = merge with current data, Cancel = replace everything." Expectation: Cancel aborts the import. Actual: Cancel starts the destructive Replace path (a second confirm then gates it). Confusing because "Cancel" universally means "back out." Data risk: mitigated to *near-miss* by the second confirmation, but the mental model is dangerous. Evidence: source (`index.html:12064-12066`) + blind interaction. Smallest fix: reword to a non-destructive first step, e.g. a clear two-option prompt where neither option is "Cancel = destroy," and keep an explicit abort — wording-only (no workflow/storage change). Codex-implementable; a quick re-audit of the new wording is worthwhile.
- **F-2 — Reference rows use a bare "Save" button (×42) that actually copies a manufacturer starting point into the Library.** Persona A · Reference. Repro: Reference → any table row → "Save." Expectation: unclear where/what is saved. Actual: copies that row into Library; inconsistent with the grid cell modal's "Save to library." Confusing/ambiguous; low data risk (additive). Evidence: source (`index.html:6713`) + interaction. Smallest fix: relabel "Copy to Library" (wording-only). Codex-implementable.
- **F-3 — Core testing/promotion vocabulary is unexplained: Test Grid vs Material Test, and evidence/promotion/proven/winner/baseline.** Persona A · Library/Test Grids/promotion. Repro: open Test Grids then Library's Material Test Wizard. Expectation: understand which to use when. Actual: no in-app primer relating the two testing systems or the promotion chain. Frustrating; causes wrong-tool starts. No data/safety risk. Evidence: blind + source + Help comparison. Smallest fix: one-sentence contextual notes (Test Grids empty state: "A Test Grid explores many power×speed settings at once; a Material Test records and proves a single dialed-in setting in Library.") plus a short Help subsection; wording-only. Codex-implementable.

**POLISH (10):**

- **F-4** — Log/entry form density (~19 fields, only Material required); advanced fields not visually grouped. Fix: optional grouping/"Advanced" disclosure (larger; treat as product decision). *Layout/workflow.*
- **F-5** — "Focus height" vs "Material height / Z-offset" opaque; the flagged future terminology cleanup. Fix: clarifying tooltips only; **do not rename stored keys**. *Wording; product decision.*
- **F-6** — Library leads with the "Find closest saved setting" match helper above the empty state. Fix: render the match helper after the empty state, or suppress it when Library is empty. *UI ordering.*
- **F-7** — Designs result omits red=cut / blue=score-engrave color→operation mapping inline. Fix: one line in the Designs result or Help. *Wording.*
- **F-8** — Two `height="auto"` SVGs emit console warnings (looks broken to a curious beginner/support). Explicitly **not** fixed here. Fix later: valid height. *Designs output markup (defer).* 
- **F-9** — Material Test Wizard prerequisite uses a blocking `alert()` instead of an inline disabled-with-reason control. Fix: inline hint / disabled button. *UI.*
- **F-10** — No "show the introduction again" affordance after dismissal (Help is the fallback). Acceptable; leave unless testing shows demand. *Product decision.*
- **F-11** — Three machine-identity concepts (Test Grid "Custom machine label", "Current workshop machine", "Genmitsu L8 Reference profile") can blur. Mostly handled by new wording; consider a one-line cross-note. *Wording.*
- **F-12** — "Does not control / send jobs to the laser" appears only in Help, not on the landing/Designs surface where the assumption forms. Fix: one short line in onboarding or the Designs result. *Wording.*
- **F-13** — Routine save success is invisible (only failures warn). Fix: a subtle "Saved" affirmation after meaningful actions (must not imply cloud). *UI/wording.*

**NOT A DEFECT:** session-only onboarding dismissal; gated promotion (cannot promote without evidence); Reference tables presented honestly as starting points; local table horizontal scroll; self-tests being developer-only; the intentionally rich Log entry form for advanced users.

## 21. Prioritized bounded correction plan

Group the wording changes into **one bounded "Beginner clarity" phase** (do not scatter into many commits):

1. **Must fix before paid release:** *(none are hard blockers)*.
2. **Strongly recommended before paid release:** **F-1** (import mode wording — the highest-value single fix), **F-2** (Reference "Save" → "Copy to Library"), **F-3** (Test Grid vs Material Test contextual notes + short Help subsection). These are wording/one-sentence changes with no storage/schema/Designs-output impact.
3. **Safe post-release polish:** F-6 (Library ordering), F-7 (Designs color→operation line), F-9 (MTW inline hint), F-11/F-12 (machine + "no laser control" one-liners), F-13 ("Saved" affirmation).
4. **Product decisions for Joe:** F-4 (Log entry Advanced grouping), F-5 (Focus/Z-offset terminology — tooltip-only, no key rename), F-10 (re-show onboarding), and whether to add optional sample data / a first-project walkthrough.
5. **Requires real human beginner testing:** F-3 comprehension (does the note actually resolve confusion?), F-1 wording (does the new prompt read unambiguously?), overall "confident to return later?" (Question 12).
6. **Already solved adequately by existing Help/onboarding:** data location & no-cloud, Export/backup urgency, 100%-scale requirement, laser safety (ventilation/fire watch/PVC), recovery/undo, dangling references, Reference "starting points" attribution, responsive layout, modal accessibility.

## 22. Product decisions needed from Joe

- F-4: introduce an "Advanced" disclosure in the Log/entry form, or leave the full field set visible?
- F-5: preferred beginner-facing phrasing for Focus height vs Material height / Z-offset (tooltip-only; keys stay `focusValue`/`materialHeightValue`).
- Onboarding: add optional sample data and/or a guided first-project walkthrough? Add a "show intro again"?
- F-8: schedule the `height="auto"` cleanup as its own tiny fix (kept out of this phase deliberately).

## 23. Human beginner-test script

**Setup (Joe, before the session):** open `index.html` in a fresh browser profile (or clear this app's site data). Do not pre-load records. Have the tester use their own words aloud. Record time-to-first-success and every hesitation.

**Tasks (give one at a time, no coaching):**
1. "Spend one minute. Tell me what this app is for and where your data is kept."
2. "Record a cut or engrave you did recently."
3. "Save a material setting you'd want to reuse."
4. "You have a brand-new material and don't know settings — what would you do here?"
5. "You have an existing backup file — load it." (provide a disposable JSON) "Now, back out of importing without changing anything."
6. "Make a small laser-cut design and get the file ready for LightBurn. What would you cut vs engrave?"
7. "Set up your actual machine and its work area."
8. "You made a mistake and want to undo a deletion."

**After each task ask:** "What did you expect? What surprised you? Was anything unclear? How confident are you that your work is saved?" For task 5 specifically: "In the first pop-up, what did you think 'Cancel' would do?" For task 6: "Which lines will cut and which will engrave — how do you know?"

**Do NOT explain during the test:** the Log-vs-Library or Test-Grid-vs-Material-Test distinction; what Reference "Save" does; what Import "Cancel" does; the color/layer mapping; where backups go.

**Record per task:** Pass (completed unaided) / Friction (completed after visible hesitation or a wrong click) / Fail (could not complete or acted unsafely/destructively). Flag any moment the tester believed the app talks to the laser, mistook a starting point for a proven setting, or feared losing data.

## 24. Final verdict

**BEGINNER-READY WITH BOUNDED FIXES RECOMMENDED**

The Tracker gives a genuinely good beginner first run: honest offline/local-storage messaging, actionable empty states everywhere, plain-language field tooltips, clear validation, strong delete/undo/recovery safety, discoverable Help covering the critical safety and "does not control the laser" points, honest Reference attribution, and clean responsive/accessible behavior — all backed by a green 2182-assertion suite including dedicated first-run, empty-state, responsive, and accessibility groups. No blockers exist. Three bounded, wording-level improvements (import mode prompt, Reference "Save" label, Test-Grid-vs-Material-Test explanation) would meaningfully reduce the remaining confusion before release-candidate testing with real beginners.

## 25. Remaining unverified areas

- Real human comprehension of the terminology (F-3) and the reworded import prompt (F-1) — requires live beginner testing (§23).
- Live end-to-end Import **Replace** was not driven to completion against seeded data (the exact confirm wording was verified from source and the refusal/merge/replace logic is covered by passing fixtures); driving it live was avoided to keep the audit purely observational and to not fabricate destructive flows.
- Tablet/desktop *visual grouping* judgments beyond overflow were spot-checked, not exhaustively reviewed at every width for every modal (Project Wizard / Material Test Wizard interiors were opened but not width-swept).
- Subjective "confident to return later" (Question 12) is inferable but ultimately a human-testing outcome.

## 26. Physical laser testing required?

**No.** Every finding concerns UI wording, discoverability, or communication. None depends on physical laser behavior, and none of the recommended fixes changes geometry, production output, or settings.

## 27. May Codex begin implementing bounded fixes?

**Yes** — for the wording-only items F-1, F-2, F-3, and the post-release POLISH one-liners (F-6, F-7, F-9, F-11, F-12, F-13). These touch wording/UI only, not storage, schema, import/export logic, or Designs output. F-4/F-5/F-8/F-10 should await the product decisions in §22 (F-5 in particular must remain tooltip-only with **no stored-key rename**).

## 28. Is another audit needed afterward?

A **focused re-audit of the import-mode prompt (F-1)** after rewording is warranted (it is the one change with any data-safety adjacency). The remaining wording fixes need **real human beginner testing** (§23) rather than another automated audit. No full re-audit is required.

## 29. Confirmation

No source or documentation file was edited; nothing was staged, committed, or pushed. HEAD remains `20325e7`, the tracked tree is clean, and `git diff --check` is clean. All interaction occurred in a disposable headless Edge profile (terminated at the end); the user's real profile and data were never used or modified.
