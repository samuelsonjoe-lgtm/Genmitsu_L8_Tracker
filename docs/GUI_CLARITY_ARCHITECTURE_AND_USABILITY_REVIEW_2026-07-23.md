# GUI Clarity — Architecture and Usability Review

Date: 2026-07-23
Repository: `C:\Genmitsu L8 Tracker`
Reviewer role: read-only architecture and usability review (Claude Sonnet 5)
Product files modified by this review: **none**
Authorized output only: this report

---

## Executive summary

The application is considerably more mature on "GUI clarity" than a first look at the request would suggest. Prior phases already built and fixture-tested a real first-run welcome panel, an extensive safety-and-workflow Help modal, keyboard-accessible tab navigation with roving tabindex and wraparound, focus-visible styling, modal focus trapping/return, empty states with contextual actions, and a Browse/Manage duality in both Library and Inventory with selection-state handling for filtered/hidden/deleted records. Most of the areas the prompt asks about (production-preview vs. cut-geometry distinction, Test vs. proven-setting distinction, import Merge/Replace safety language, dangling-reference tolerance) are **already correctly handled and already fixture-covered** — confirmed by reading the fixtures, not assumed.

The one genuine, confirmed, unaddressed terminology defect this review found is exactly the one the prompt suspected: the single stored field `focusValue`/`focusUnit` is labeled three different ways in three different places ("Focus height" in the Log/Library/Project forms, "Focus/Z-offset" in Library/promotion/summary displays, "Focus value" in the Production Settings editor), and a second, genuinely distinct stored field `materialHeightValue`/`materialHeightUnit` (labeled "Material height / Z-offset") exists **only** in the Log/Library shared form and has no equivalent in the Project settings form — with help text on both fields ambiguous enough that a user cannot reliably tell which one to fill in or why both exist. A third, unrelated concept literally named `zOffset` also exists, inside the Material Test Wizard's LightBurn-parameter helper, and shares no schema relationship with either of the above despite the name overlap.

**Recommended phase: Option A, a terminology-and-help-text-only pass**, scoped to the Focus Height / Focus-Z-offset / Material Height family plus making the Wizard's unrelated `zOffset` unambiguous by context. No stored field is renamed, no schema changes, no production output changes, and the diff is small enough to review line by line.

---

## 1. Actual HEAD and working-tree state

| Item | Observed |
| --- | --- |
| HEAD | `994510bbc19d326f96afe0a6e29d02bbf7e1da17` |
| Subject | `Improve inventory organization and material cleanup` |
| Branch | `main` |
| Upstream | `main...origin/main`, **0 / 0** (synchronized) |
| Tracked working-tree diff | **None** — `git diff --stat` and `git diff --check` both empty |
| Staged changes | None |
| Untracked files | All pre-existing (historical `docs/*.md` reports, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`) — none related to GUI, none touched by this review |

The stated baseline matches the actual repository exactly — confirmed directly, not assumed. Recent commit history (`994510b` → `a4d4c06` → `3aa314a` → `386e1e0` → `8c06800`) confirms Inventory Organization and the full Security S1–S3 sequence are already committed and out of scope, consistent with the stated project sequence.

---

## 2. Files, functions, and surfaces inspected

Single-file application, `index.html` (15,000+ lines) — no other product file exists. Inspected via `Read`/`Grep` only, no edits.

- **Navigation/shell**: `tabs` array (`index.html:528-537`), tab rendering and keyboard handling (`runAccessibilityPolishFixtures`, `index.html:1550+`), `setTab`
- **First-run/onboarding**: `firstRunOnboardingHtml`, `bindFirstRunOnboardingActions` (`index.html:2370-2399`), `isFirstRunWorkspaceEmpty`, `runFirstRunOnboardingFixtures` (`index.html:1123+`)
- **Help system**: `openHelp`, the Help modal content, `runHelpSafetyFixtures` (`index.html:1486-1549`) — read the fixture in full as the most reliable source of the Help modal's actual current text (fixtures assert exact substrings, which is stronger evidence than reading the template literal alone)
- **Shared form helpers**: `labelText`, `field`, `select`, `unitField`, `materialFieldHtml`, `modalFields` (`index.html:12931-12967`) — the shared Log-entry/Library-profile form
- **Terminology-relevant fields**: every `focusValue`/`focusUnit`, `materialHeightValue`/`materialHeightUnit`, `zOffset`, `nominalThicknessMm`/`measuredThicknessMm` occurrence (grepped across the full file, ~35 matches read in context)
- **Production Settings editor**: `productionSettingsEditorHtml` region (`index.html:11024`) — the material-condition/cut-settings form with explicit nominal/measured thickness fields
- **Material Test Wizard**: LightBurn Interval/Material Test Generator panels (`index.html:12031-12058`), where the unrelated `zOffset` field lives
- **Accessibility infrastructure**: `runAccessibilityPolishFixtures` (`index.html:1550-1626`), `runModalAccessibilityFixtures` (referenced), `runResponsiveTuningFixtures` (`index.html:1629+`)
- **Empty states / dangling references**: `runEmptyStateDanglingReferenceFixtures` (`index.html:9072+`), `runBeginnerClarityFixtures` (`index.html:9165-9224`) — read in full; this fixture is effectively a live regression suite for exactly the kind of clarity concerns this review is asked to evaluate, and it already passes for import-safety language, empty-state ordering, Reference-tab terminology, Test-Grid-vs-Material-Test explanation, and Designs preview/production-byte separation
- **Inventory Browse/Manage** (from the immediately preceding Inventory Organization review, re-confirmed relevant here): `renderInventory`, `inventoryManageHtml`, `inventoryBrowseHtml`, `materialBrowserResultsHtml`, `resolveMaterialBrowserSelection`
- **Action-hierarchy vocabulary**: grepped every `class="danger"`/`class="primary"` button (68 occurrences) and representative action-button labels across Log, Library, Projects, Designs, Inventory

---

## 3. Review method

Static source reading plus fixture reading, cross-checked against each other: where a fixture asserts an exact substring of rendered text (e.g., `runHelpSafetyFixtures` asserting `/Import can merge/i.test(text)`), that assertion is treated as **confirmed current behavior**, not merely "the code appears to say this," because the fixture would fail if the text drifted. No browser was launched for this review; no runtime/browser result is claimed anywhere in this report (see §16, Unverified areas). This review is source-and-fixture-only, consistent with its "read-only architecture and usability review" framing and the requirement not to claim unexecuted results.

---

## 4. Current navigation model

- Eight top-level tabs (`tabs`, `index.html:528-537`): Log, Library, Test Grids, Designs, Reference, Projects, Inventory, Pricing. Labels are short, workshop-language nouns; none are internal/implementation jargon.
- Implemented as a proper ARIA `tablist` (`role="tablist"`, `aria-label="Tracker sections"`), each tab `role="tab"`, roving `tabindex` (selected = `0`, others = `-1`), Arrow-Left/Right with wraparound, Home/End, Enter — all confirmed by `runAccessibilityPolishFixtures` assertions that exercise real `KeyboardEvent`s and check `document.activeElement`, not just markup.
- The shared content region (`#app`) is one `tabpanel` labeled by the active tab — confirmed.
- Two secondary "modes" exist within tabs, not as separate navigation: Library's Browse/Manage toggle and Inventory's Browse/Manage toggle (added in the Inventory Organization phase), both persisted view preferences, not separate tabs.
- No separate "landing page" exists; the closest equivalent is the first-run onboarding panel, which only renders inside the Log tab when the workspace is completely empty and onboarding hasn't been dismissed (`firstRunOnboardingHtml`, gated by `isFirstRunWorkspaceEmpty() && !state.entries.length`).

---

## 5. Current form patterns

- `modalFields(kind, data)` is a **shared** template used for both Log entries and Library profiles (`kind === 'entry' | 'profile'`), which is why fixing a label or help-text string once fixes it in both places — an existing, favorable pattern for this phase's approach.
- Field order in the shared form: Date (entry only) → Job type → Material (with safety warning) → Thickness → **Focus height** → Power min/max → Speed → Passes → Line interval → Overscan → Kerf offset → Dither mode → **Material height / Z-offset** → Air assist → Air pressure → Software → (profile: Tags) → (entry: Settings file, star rating) → Notes → (profile: Material Tests editor, Production Settings editor).
- This order is broadly basic-to-advanced (material/thickness before power/speed before overscan/kerf/dither), with one exception directly relevant to this review: **Focus height and Material height / Z-offset are separated by six other fields** (power, speed, passes, line interval, overscan, kerf), even though they are the two fields most likely to be confused with each other. Placing them adjacently would itself reduce confusion, independent of any label change.
- Every field uses the shared `field`/`select`/`unitField` helpers, which already attach a consistent inline help affordance (`labelText(label, tip)` → `help(tip)`), so **help text already exists on every relevant field** — the defect is that the help text itself doesn't disambiguate Focus height from Material height / Z-offset (see §7).
- The Production Settings editor (a different, separate form for Library profiles' machine-specific proven settings, `productionSettingsEditorHtml`, `index.html:11024`) already demonstrates the **correct pattern** this review recommends extending: it explicitly labels and separates `nominalThicknessMm` ("Nominal thickness (mm)") from `measuredThicknessMm` ("Measured thickness (mm)"), with an explicit help note ("Stored canonically in millimeters and not rewritten when the app display unit changes.") and a `measurementScope` selector. This is exactly the level of clarity the Focus/Material-height fields lack, and it lives in the same file, making it a natural model to imitate in wording (not in schema).

---

## 6. Current action hierarchy

- `class="danger"` is applied consistently to destructive actions (Delete buttons across Log, Library, Projects, Inventory, Grids) — 68 total `primary`/`danger` class occurrences reviewed by grep; spot-checked a representative sample and found no destructive action styled as `primary` or vice versa.
- Action vocabulary is already mostly consistent and workshop-oriented: "Duplicate" (never "Copy" for record duplication), "Copy to Library" (Reference rows, confirmed by `runBeginnerClarityFixtures`'s explicit assertion that Reference never exposes a bare "Save"), "Save as project" (Library → Project), "Material Test Wizard", "Promote" (Log entry → Library profile), "Export"/"Import" (backup), "Generate" (Designs). No case was found where two different buttons use the same word for different effects, or two different words for the same effect, **except** the Focus/Material-height terminology in §7, which is a labeling problem, not a button-action problem.
- Copy-vs-mutate distinction is already handled carefully in at least one place inspected in depth: the Material Test Wizard's "Copy Interval Setup"/"Copy Material Test Setup" buttons copy a formatted text block to the clipboard for pasting into LightBurn — they do not save anything to the Tracker's own records, and their labels ("Copy … Setup") already make this clear without a schema change.
- Designs already distinguishes "cut layout" preview from "finished view" preview via an explicit, separately named `designPreviewMode` state (`'cut-layout'` vs a finished-view mode), and `runBeginnerClarityFixtures` confirms Designs guidance is "screen-only" and "leaves production SVG bytes unchanged" — this is exactly the "Finished View vs. cut geometry" and "screen preview vs. production artwork" distinction the prompt worries about, and it is **already correctly implemented and tested**.

---

## 7. Terminology findings (Focus Height / Z-offset family)

This is the central, confirmed finding of this review.

### What each field actually stores (confirmed by direct source reading)

| Field | Stored where | Current label(s) seen in the UI | Current help text |
| --- | --- | --- | --- |
| `focusValue` / `focusUnit` | Log entries, Library profiles (`modalFields`, shared), Project settings (`settingsFocusValue`/`settingsFocusUnit`), Production Settings editor (`productionFocusValue`/`productionFocusUnit`) | **"Focus height"** (Log/Library/Project forms, `index.html:12946`, `13068`); **"Focus/Z-offset"** (Library profile display line, promotion field labels, compact card summaries — `index.html:2594`, `8020`, `9492`, `9635`); **"Focus value"** (Production Settings editor, `index.html:11024`, no accompanying help string at all in that form) | "Lens/material focus distance used for this job." (entry form) / "Lens/material focus distance used for this Project." (project form) / none (production settings) |
| `materialHeightValue` / `materialHeightUnit` | Log entries and Library profiles only (`modalFields`, unconditional inside the shared template — **confirmed absent from the Project settings form and the Production Settings editor**) | **"Material height / Z-offset"** (`index.html:12955`) — this exact label string appears nowhere else in the file | "Height adjustment from the material surface." |
| `zOffset` (no `Value`/`Unit` split) | Material Test Wizard plan object only (`index.html:11824` default, `12031` summary text, `12058` form field) — **not part of the Log/Library/Project/Production-Settings schema at all**; purely a scratch value used to build a copy-pasteable "LightBurn Material Test Generator" parameter block | "Z offset (mm)" | none beyond the section heading "LightBurn Material Test Generator" |

### Confirmed defects

1. **The same stored concept (`focusValue`/`focusUnit`) has three different display labels depending on which form or summary the user is looking at** ("Focus height" / "Focus/Z-offset" / "Focus value"). A returning user who learns the term in one place (e.g., the Log form's "Focus height") will see a differently-worded but identical concept later (a Library profile card's "Focus/Z-offset" line) with no visual or textual cue that they are the same value.
2. **`materialHeightValue`/`materialHeightUnit` exists in Log/Library records but has no equivalent field anywhere in Projects or Production Settings.** A user who fills in "Material height / Z-offset" on a Library profile, then later opens a Project derived from that material, will find no such field to carry the value forward — not a data-loss bug (Projects were never designed to copy this field; see §5 of the Inventory review for the general "downstream consumers are name/value snapshots, not live links" pattern, which applies here too), but a **conceptual surprise**: the field looks important enough to fill in, then silently doesn't travel anywhere.
3. **The help text for `focusValue` ("Lens/material focus distance") and `materialHeightValue` ("Height adjustment from the material surface") do not clearly distinguish the two concepts from each other.** Both sentences reference "material" and a vertical distance; neither explains *why* there are two separate fields, *when* to use one vs. the other, or what "Z-offset" means in either case. This is the specific ambiguity the prompt's suspected renames ("Machine Focus Distance" / "Intentional Z Offset") are trying to resolve.
4. **`zOffset` inside the Material Test Wizard shares no schema relationship with either `focusValue` or `materialHeightValue`**, despite the name overlap with "Z-offset" appearing in `materialHeightValue`'s label. A user reading "Z offset (mm)" in the Wizard and "Material height / Z-offset" in a Library profile has no way to know from the labels alone that these are two unrelated concepts (one is a LightBurn-generator scratch parameter for building a test-grid job; the other is a persisted per-record field).

### Determinations required by the prompt

1. **What each field stores** — see table above; confirmed by direct source reading, not assumed.
2. **Which workflows use each field** — `focusValue`/`focusUnit`: Log, Library, Project, Production Settings (four workflows). `materialHeightValue`/`materialHeightUnit`: Log and Library only (two workflows). `zOffset`: Material Test Wizard only (one workflow, and not a persisted record field).
3. **Whether any two labels appear interchangeable while storing different concepts** — **Yes, confirmed**: "Focus/Z-offset" (a display label for `focusValue`) and "Material height / Z-offset" (a display label for the entirely different `materialHeightValue`) both contain the word "Z-offset," inviting a reader to conflate them, even though they are different stored fields with different help text and different form placement.
4. **Whether "Focus Height" should be renamed to "Machine Focus Distance"** — **Recommendation, not a confirmed requirement**: the current label is not wrong, but it is inconsistent (three labels for one field, per finding 1) and slightly ambiguous (does "height" mean above the material, above the bed, or the lens's own focal distance?). A clearer, single, consistently-applied label — either "Focus height" everywhere (simplest, smallest diff) or "Machine focus distance" everywhere (marginally clearer, slightly larger diff because it also changes the already-correct instances) — would resolve finding 1. This review takes no firm position on which exact wording wins; it recommends **picking one and applying it everywhere `focusValue` is labeled**, which is the actual defect.
5. **Whether "Material Height / Z-offset" should be renamed to "Intentional Z Offset"** — **Recommendation**: yes, some renaming or clarifying help text is warranted, because the current label's own internal "/ Z-offset" suffix is what creates the naming collision with `focusValue`'s "Focus/Z-offset" label (finding 3). Removing "Z-offset" from `materialHeightValue`'s label (calling it, for example, "Material height offset" or "Intentional Z offset," as the prompt suggests) and reserving the word "Z-offset" for at most one of the two concepts would directly fix the collision. This review recommends resolving the *collision*; the exact chosen wording is a product/copy decision the implementer should make once, consistently.
6. **Whether labels can change without changing stored keys or data** — **Yes, confirmed safe.** `focusValue`, `focusUnit`, `materialHeightValue`, `materialHeightUnit` are all plain object keys read/written via `f.focusValue`, `data.focusValue`, etc. (`index.html:12985-12986`, `13202`, and the `field`/`unitField` helper's `name="focusValue"` form attribute). The **label string** passed to `unitField(...)`/`field(...)` is a separate literal argument from the stored `name`; changing the label argument does not touch the `name` attribute, the form-read key, or the persisted field. Confirmed by reading `unitField`'s implementation (`index.html:12965-12967`): `name="${valueName}"` (stored key) is entirely independent of `${labelText(label, tip)}` (display text).
7. **Whether old exports, backups, or imported records contain visible labels that must remain backward compatible** — **No.** Backups store the raw field values (`focusValue: 7, focusUnit: 'mm'`, etc.) as JSON, never the display label string. `validateBackupImport`/`applyBackupImport` read data by key, not by any label text. A label change has zero effect on backup/import compatibility, confirmed by the same `field`/`unitField` structure.
8. **Whether help text/tooltips alone are enough, or labels must change too** — **Judgment call, not purely factual**: help text alone (finding 3's fix) would resolve the *ambiguity about what to enter*; the *label-collision* problem (finding 3's "Z-offset" appearing in two different fields' labels) is better fixed by adjusting at least one label, not help text alone, because the collision is visible even to a user who never opens the help tooltip. This review recommends doing both: consistent labels *and* clarifying help text, since the terminology-only pass is exactly the appropriate size to include both.
9. **Whether a rename would affect fixtures, summaries, printed output, CSV, SVG metadata, Project handoffs, Library records, or Test Grid promotion** — **Confirmed, in detail**:
   - **Fixtures**: several fixtures assert exact label/summary text containing "Focus/Z-offset" (e.g., `index.html:2594`, `8020` inline template strings) — these are rendered strings, not fixture assertions themselves in the excerpts read, but any fixture elsewhere that does assert this exact substring would need updating in lockstep with the label change (this review did not find such an assertion in the fixture functions read, but a full-file assertion grep should be the first implementation step before touching any label string, to enumerate every fixture that pins exact label text).
   - **Summaries/compact card text**: yes, several inline summary strings (`index.html:2594`, `8020`, `12295`, `12306`, `12415`) use "Focus:" or embed the field inline — these should be updated to match whatever single label is chosen, for consistency, though they are lower-priority than the actual form labels.
   - **Printed output / CSV**: no CSV export includes `focusValue`/`materialHeightValue` at all (Inventory's CSV export, the only CSV export in the app, covers raw materials/finished batches, not Log/Library/Project settings) — **not affected**.
   - **SVG metadata**: Designs' SVG generation does not reference `focusValue`/`materialHeightValue` (these are Log/Library/Project record fields, unrelated to the Designs generator's own geometry/parameters) — **not affected**; confirmed by the field's usage sites all being inside Log/Library/Project/Production-Settings code, never inside any `buildDesignResult`/generator function.
   - **Project handoffs**: `projectDraftFromWizard` and Designs-to-Project handoff code do not copy `focusValue`/`materialHeightValue` at all today (they copy `material`, `thicknessValue`, `thicknessUnit` only) — **not affected**.
   - **Library records / Test Grid promotion**: `promotionAddField(candidate, 'focusValue', 'Focus / Z-offset', ...)` (`index.html:9492`, `9635`) is a promotion-summary **display label**, exactly the kind of string this phase would update — it is data-safe to change (it's a label argument, not the promoted field's stored key, which remains `cutSettings.focusValue`) but must be updated for consistency with whatever label is chosen elsewhere, or the promotion-review screen will show yet a fourth wording.

### Recommendation on the exact rename (a UX judgment, not a fact)

Standardize on **one** label for `focusValue`/`focusUnit` everywhere it appears (Log, Library, Project, Production Settings, promotion labels, compact summaries) — this review does not mandate "Machine Focus Distance" specifically, only that **the same field must say the same thing everywhere**. Separately, remove or rework the word "Z-offset" from `materialHeightValue`'s label so it no longer collides with whatever word is chosen for `focusValue` (e.g., "Intentional Z offset" or "Material height offset," per the prompt's own suggestion), and adjacent-place the two fields in the shared form (§5) so their relationship is visible without reading either help string.

---

## 8. Selection and expand/collapse findings

- Library's and Inventory's Browse modes both already implement the pattern the prompt is worried about: a `resolveMaterialBrowserSelection`/`resolveBrowserSelection` helper that (a) keeps a selection alive across re-renders, (b) explicitly marks a selection "hidden by the current filters" rather than silently losing it, and (c) falls back to the first visible item only when the previous selection no longer exists at all. This was independently confirmed during the immediately preceding Inventory Organization review (`docs/INVENTORY_ORGANIZATION_MATERIAL_CLEANUP_ARCHITECTURE_REVIEW_2026-07-23.md` §6) and re-confirmed here by re-reading the same functions — **this is a solved problem, not a defect**, and it is a genuine shared pattern (the same helper name/shape is reused between Library and Inventory Browse modes).
- The Browse-mode tree uses native `<details>` disclosure elements (`data-browser-node`), which already gives keyboard/screen-reader expand/collapse semantics without custom ARIA work — confirmed, not assumed.
- No evidence was found of a collapsed parent node hiding an active selection without indication — the "hidden by current filters" notice already covers the filter case; a collapsed-tree-node case was not separately tested by fixture, so this remains a minor unverified edge (see §16).

---

## 9. Browse/Manage findings

- Both Library and Inventory persist their Browse/Manage choice as a `state.*ViewMode` preference (`libraryViewMode`, `inventoryViewMode`), normalized defensively (`normalizeLibraryViewMode`/`normalizeInventoryViewMode`), so an invalid/legacy value always falls back to `'manage'` rather than erroring.
- **Confirmed gap, low severity**: neither mode's toggle button pair currently carries an inline explanation of *why* there are two modes (e.g., "Manage: edit and organize records. Browse: explore by category, material, and size, with related records shown"). A first-time user sees two buttons ("Browse"/"Manage") with no description of the difference until they click one and compare. This is a small, safe, copy-only addition if included in a future phase, but is **not** part of the Focus-Height/Z-offset defect this review is prioritizing, and is listed here as UX judgment/optional polish rather than a confirmed defect blocking anything.

---

## 10. Tab-by-tab findings

For each tab: purpose clarity on entry, main action clarity, competing secondary actions, terminology, missing context, density, implementation-language leakage, cross-tab memory burden, navigation/selection-loss risk, accessibility, and whether any issue found is local or shared.

### Log
- Purpose clear on entry (empty state: "No entries yet… Log your first cut or engrave…"). Main action ("Log a job") is a primary-styled button in the empty state and a header action otherwise. No competing secondary action. Terminology consistent with workshop language throughout. The Focus-Height/Material-height fields (§7) live here; this is the tab where the confusion is most consequential, since Log entries are the app's primary "what actually happened" record.

### Library
- Purpose clear (profiles = reusable material settings). Duplicate/Save-as-project/Material-Test-Wizard/Delete actions are clearly worded and consistently styled (danger only on Delete). Browse mode adds related-Project and matching-Library-setting context that Manage mode lacks (§9). Shares the same Focus-Height/Material-height fields and thus the same §7 defect.

### Test Grids
- Purpose is explicitly explained in-page ("Compare many speed and power combinations at once… Material Test later to confirm, refine, or document one setting" — confirmed verbatim by `runBeginnerClarityFixtures`). This directly answers the prompt's "Tests from proven production settings" production-safety concern — **already solved and tested**, not a gap.

### Designs
- Already distinguishes "cut layout" preview from "finished view" preview via a named state, and guidance text explicitly calls out red-cut-path meaning and that guidance is screen-only (§6) — **already solved and tested**.
- Not deeply re-audited beyond this for the current review given the strong existing fixture evidence and the explicit non-goal of touching Designs/production output in this phase.

### Reference
- Already explicitly distinguishes "workshop machines" (equipment you own), the "Reference profile" (bundled manufacturer starting-point tables), and "historical snapshots" (past saved records) — confirmed verbatim by `runBeginnerClarityFixtures`'s assertion. Reference rows say "Copy to Library," never a bare "Save," which correctly signals "this creates a new Library record" rather than "this overwrites something." **Already solved and tested.**

### Projects
- Not deeply re-audited line-by-line in this pass (no confirmed defect surfaced from the terminology/action-hierarchy greps touching Projects specifically beyond the general Focus-Height field, which Projects also has via `settingsFocusValue`/`settingsFocusUnit`, §7).

### Inventory
- Covered in full by the immediately preceding Inventory Organization review; no new GUI-clarity-specific defect surfaced here beyond what that review already recommended (numeric thickness sort, reference-count delete warning, duplicate-suggestion panel) — those remain that review's scope, not this one's, per the instruction not to let this review's scope creep into the just-completed phase.

### Pricing
- Not deeply re-audited in this pass; no terminology or action-hierarchy defect was surfaced by the greps performed.

### Help/Settings/Backup/Vault surfaces
- The Help modal is extensive and already fixture-verified to cover: offline/no-cloud storage explanation, Export/Import Merge-vs-Replace behavior stated as "not automatic," newer-schema refusal, browser-data-loss risk and recovery tools, the 100%-scale LightBurn import requirement, unit/dimension/layer/framing verification requirements, "Tracker does not send jobs to the laser," physical-result variation caveats, Tabletop Accessories evidence/promotion boundaries, and an extensive laser-safety section (ventilation, fire watch, unattended-use warning, PVC/vinyl/chlorinated-plastic prohibition, "laserable" marketing skepticism). This is **already a mature, tested surface** — no defect found. The Local Vault's own passphrase-vs-backup-passphrase distinction was confirmed as an explicit requirement and verified live during the S3-1 corrections verification earlier in this session (separate report), and is out of scope for this GUI-only review.

### First-run/onboarding surface
- Exists, is real (`<button>` elements, not styled divs), is keyboard-reachable, and is session-only/unpersisted (confirmed by fixture). **Confirmed gap**: it offers exactly three starting points — "Log your first job," "Plan a Test Grid," "Open Reference" — and does not mention Designs, Projects, or Inventory at all, even though Designs in particular is about to become more central with the upcoming Finished Views/product-generator phase. This is a copy-only, low-risk gap, but it is **not part of the Focus-Height/Z-offset defect** this review prioritizes as the bounded phase; it is listed here as a legitimate but separate candidate for a future small addition (see §14, optional polish).

---

## 11. Production-safety clarity findings

Per §6 and the tab-by-tab findings, the specific distinctions the prompt asks about are already handled and fixture-tested:

| Distinction | Status |
| --- | --- |
| Screen-only preview vs. production artwork | **Already clear** — Designs guidance text and fixture-confirmed screen-only/byte-unchanged behavior |
| Finished View vs. cut geometry | **Already clear** — named `designPreviewMode` states |
| Export vs. Save | **Already clear** — Export always produces a downloadable file; no UI element conflates the two words |
| Copying values vs. changing source records | **Already clear** — "Copy to Library," "Save as project," "Duplicate" all imply a new/separate record; the Wizard's "Copy … Setup" buttons copy to clipboard only |
| Tests vs. proven production settings | **Already clear** — Test Grid page explicitly explains its relationship to Material Tests; Production Settings has its own `verificationStatus`/`preferred`/`confidence` fields distinguishing a proven setting from a draft |
| Nominal vs. measured thickness | **Already clear, in the one place it applies** — Production Settings editor explicitly labels and separates `nominalThicknessMm` and `measuredThicknessMm`; the general Log/Library/Project `thicknessValue` field is a single, undifferentiated "Material thickness" concept (not nominal-vs-measured at all) which is appropriate for those simpler record types and is not a defect |
| **Machine focus distance vs. intentional Z offset** | **Confirmed gap — the central finding of this review (§7)** |
| A warning vs. a hard block | Not separately audited in depth this pass; the import Replace confirmation (a two-step confirm before a destructive Replace) and the storage-recovery flow both already model "warn, then require explicit second confirmation for anything destructive," which is the correct pattern, confirmed by `runBeginnerClarityFixtures` |
| Currently selected material vs. a frozen historical snapshot | **Already clear** — Reference tab's explicit "does not rewrite historical records" text (fixture-confirmed), and Production Settings' `inventoryItemName` frozen-snapshot-on-dangling-reference pattern (confirmed in the Inventory Organization review) |

No production-safety clarity defect other than §7 was confirmed in this pass.

---

## 12. First-run and empty-state clarity findings

- Empty states across Log, Library, Inventory, Grids, Projects consistently use a shared `empty(title, description, actions)` helper pattern with a primary creation action — confirmed pattern reuse, not a defect.
- First-run onboarding's narrow starting-point list (§10) is the one confirmed, if minor, gap.
- The previously-discussed "new landing page with large buttons for Designs, Inventory, and Library" is evaluated per the prompt's instruction: the existing architecture does **not** already support this as a trivial change — it would require a new top-level surface (or a new always-visible section) distinct from both the tab bar and the existing empty-workspace-only first-run panel, which is a larger information-architecture decision than "GUI clarity." **This review treats it as a deferred product decision**, not part of the bounded phase, consistent with the prompt's own framing.

---

## 13. Accessibility findings

Distinguishing existing systems that already work from anything newly found:

- **Already working, confirmed by fixture, not touched by this phase**: tablist semantics and roving tabindex with wraparound; one labelled tabpanel; global `:focus-visible` styling with a fallback for engines without native support; bounded touch-target sizing on primary tabs/common actions with a scoped exception for dense table controls; disabled controls remaining genuinely disabled (not merely visually dimmed); modal focus trap/return (confirmed via Help modal's Tab-containment and Escape-restores-trigger-focus assertions); first-run action buttons are real `<button>` elements; no duplicate DOM IDs after tab renders; no external scripts/stylesheets.
- **Local GUI clarity issue relevant to this phase**: the Focus Height/Material Height help text (§7) is present (every field has a `labelText(label, tip)` help affordance) but not disambiguating — this is a content clarity issue, not a technical accessibility regression; the underlying `aria-describedby`/tooltip mechanism itself is not being newly evaluated as broken.
- **No accessibility regression was found or is proposed** — the recommended phase changes label/help-text strings only, touching no ARIA attribute, no focus-management code, and no keyboard handler.
- **Optional polish, not required**: a Browse/Manage inline explanation (§9) would be a small additional accessible-text improvement but is not required for this phase's scope.

---

## 14. Responsive findings

Not independently re-verified via browser in this pass (no browser was launched; see §16). Based on source reading only: `runResponsiveTuningFixtures` (`index.html:1629+`) exists and, per the accessibility-polish fixture's own assertions on bounded touch-target CSS (`min-height: 40px` on tabs/actions, `min-height: 34px` scoped to dense table controls), the app already has deliberate, tested sizing rules for narrower layouts. No responsive defect is confirmed by this review; a live check at the four suggested widths is listed as a required manual check for whoever implements this phase (§18), not claimed here.

---

## 15. Confirmed defects (summary)

1. `focusValue`/`focusUnit` is labeled three inconsistent ways across Log/Library/Project forms, Library/promotion/summary displays, and the Production Settings editor.
2. `materialHeightValue`/`materialHeightUnit` exists only in Log/Library records, with a label ("Material height / Z-offset") that collides textually with `focusValue`'s "Focus/Z-offset" label.
3. Help text for both fields does not explain the distinction between them or when to use each.
4. The unrelated Wizard-only `zOffset` scratch field shares no schema relationship with either of the above, despite label-word overlap.
5. Focus height and Material height / Z-offset are separated by six unrelated fields in the shared form, despite being the two fields most likely to be confused.
6. First-run onboarding mentions only three of eight tabs as starting points (UX concern, not bundled into the recommended phase).

## UX concerns (judgment, not confirmed defects)

- Browse/Manage mode toggles lack an inline explanation of their difference (§9).
- First-run onboarding's narrow tab coverage (§10, §12).
- Whether "Focus height" or "Machine focus distance" is the better final wording is a copy decision, not a technical fact.

## Optional polish (explicitly not recommended for this phase)

- Adding Designs/Projects/Inventory starting points to first-run onboarding.
- Adding inline Browse/Manage mode explanations.
- Standardizing every compact summary string's exact wording for every field in the app (only the Focus/Material-height family is in scope here).

---

## 16. Unverified areas

- No browser was launched for this review; all findings are from source and fixture text reading. Any recommendation involving actual rendered layout at the suggested widths (1440/1024/768/480 px) is unverified and should be manually checked at implementation time.
- Whether a collapsed Browse-mode tree node can hide an active selection without any indication (distinct from the already-confirmed "hidden by filters" case) was not separately fixture-tested or manually verified.
- The exact final wording for the Focus-Height/Material-height relabeling is a product/copy decision this review does not make.
- Whether any additional fixture elsewhere in the file pins an exact "Focus/Z-offset" or "Material height / Z-offset" substring beyond the ones read in this pass was not exhaustively re-verified by a full-file assertion grep in this review (recommended as the implementer's first step, per §7 finding 9).

---

## 17. Options considered

| Option | Verdict |
| --- | --- |
| **A — Terminology-only pass** (Focus Height/Z-offset family + related help text) | **Recommended.** Addresses the one confirmed, well-evidenced defect; smallest possible diff; zero schema/production/storage impact. |
| **B — Navigation and selection clarity** | **Rejected for this phase — not needed.** Selection/expand-collapse/Browse-Manage mechanics are already a solved, shared, tested pattern (§8, §9); the only related item is the optional Browse/Manage inline-explanation polish, which is not a confirmed defect. |
| **C — Form and action hierarchy** | **Rejected for this phase — mostly already solved.** Section headings, basic/advanced grouping, Save/Generate/Export wording, and destructive-action separation are already consistent (§5, §6); the one confirmed exception (Focus/Material-height field adjacency) is folded into Option A rather than justifying a separate, larger form-reorganization phase. |
| **D — Small cross-application clarity bundle** | **Considered, narrowed to Option A.** The only items that share one clear, confirmed user problem (the Focus/Material-height terminology confusion) are already exactly Option A's scope; bundling in the first-run-onboarding or Browse/Manage-explanation UX concerns would mix in un-confirmed judgment calls and enlarge the diff without a shared root cause, so this review declines to bundle them. |
| **E — Broad GUI cleanup** | **Rejected**, per the prompt's own steer and this review's findings: the app's GUI is already substantially polished from prior phases; a broad cleanup would carry high regression risk against extensive existing fixture coverage for comparatively little remaining confirmed benefit, and would risk delaying Finished Views/product generators. |

---

## 18. Recommended bounded implementation phase

### 1. Exact user problem being solved

A user filling in Log entries, Library profiles, Project settings, or Production Settings encounters the same underlying focus-distance concept under three different names, and a second, related-sounding field ("Material height / Z-offset") whose relationship to the first is not explained — making it unclear which field to fill in, why both exist, and what "Z-offset" means in either context.

### 2. Exact included behaviors

- Standardize the display label for `focusValue`/`focusUnit` to one consistent wording everywhere it currently appears as "Focus height," "Focus/Z-offset," or "Focus value" (Log form, Library form, Project form, Production Settings editor, promotion labels, compact summary strings).
- Adjust the display label for `materialHeightValue`/`materialHeightUnit` so it no longer contains the word "Z-offset" in a way that collides with the `focusValue` label (e.g., "Intentional Z offset" or "Material height offset" — final wording is an implementation/copy decision).
- Rewrite both fields' help text to explicitly state what each is for and how they relate to each other (e.g., clarifying that one is the machine/lens-to-material focus setting and the other is a deliberate additional offset applied on top of normal focus).
- Reorder the shared `modalFields` template so the two fields sit adjacently rather than six fields apart.
- Update the Test Grid promotion field labels (`index.html:9492`, `9635`, currently `'Focus / Z-offset'`) to match the new standardized `focusValue` label.
- Update the compact summary/card strings that currently render "Focus/Z-offset" or bare "Focus:" (e.g., `index.html:2594`, `8020`, `12295`, `12306`, `12415`) to the same standardized wording.
- Leave the Material Test Wizard's unrelated `zOffset` field's own label ("Z offset (mm)") as-is, but confirm (and if needed add a one-line clarification) that its section heading context ("LightBurn Material Test Generator") already makes clear it is a LightBurn-parameter helper value, not the same concept as the record-level fields — this is expected to require zero or near-zero change once the other two labels no longer say "Z-offset."

### 3. Exact tabs and surfaces included

Log, Library, Projects, Production Settings editor (a sub-section of Library profiles), Test Grid promotion-review screens, and any compact summary card that renders `focusValue`/`materialHeightValue`. Designs, Reference, Inventory, Pricing, Help, and the Material Test Wizard's own generator-parameter labels are **not** touched (the Wizard's `zOffset` label itself is left alone per the previous paragraph).

### 4. Exact deferred behaviors

- Any Browse/Manage inline explanation addition.
- Any first-run onboarding content change.
- Any change to `thicknessValue`/`nominalThicknessMm`/`measuredThicknessMm` (already correctly labeled).
- Any responsive-layout change.
- Any accessibility infrastructure change (ARIA roles, focus management, keyboard handlers) — none is needed.
- A landing-page redesign (explicitly deferred, per prompt instruction).
- Any change to Designs, production SVG, or geometry.

### 5. Likely files and functions touched

All within `index.html`: `modalFields` (label strings + field order, `index.html:12931-12963`), the Project settings form's Focus-height field (`index.html:13068` region), the Production Settings editor's focus field (`index.html:11024` region), `promotionAddField` call sites for `focusValue`/`focusUnit` (`index.html:9492-9493`, `9635-9636`), and the compact-summary template literals at `index.html:2594`, `8020`, `12295`, `12306`, `12415`.

### 6. Shared helpers or CSS touched

None required. `labelText`/`field`/`unitField`/`select` (the shared form helpers) are called with different label-string *arguments* at each site; no helper's own implementation needs to change. No CSS class or rule needs to change — this is a text/argument change only.

### 7. Protected functions

`unitField`'s/`field`'s own signatures and behavior; every `name="..."` form-field attribute (i.e., every stored key: `focusValue`, `focusUnit`, `materialHeightValue`, `materialHeightUnit`, `zOffset`); `readForm`; `normalizeRawMaterial`/any other normalizer; all persistence, merge, backup, and vault code (untouched by a label-only change).

### 8. Whether stored bytes change

**No.** Confirmed in §7 finding 6: the label argument passed to `field`/`unitField` is independent of the `name`/stored-key argument. No record's `focusValue`/`materialHeightValue`/`focusUnit`/`materialHeightUnit` field is renamed, and no value stored today changes shape or meaning.

### 9. Whether backup/export bytes change

**No.** Confirmed in §7 finding 7: backups serialize field values by key, never by display label. A `backupObject()` produced before and after this phase, given identical record data, is byte-identical.

### 10. Whether production SVG or geometry bytes change

**No.** Confirmed in §7 finding 9: `focusValue`/`materialHeightValue` are not referenced anywhere inside the Designs generator/serialization code.

### 11. Whether schema or migration is required

**No.** `SCHEMA_VERSION` is unaffected; no field is added, removed, or restructured.

### 12. Legacy compatibility behavior

Unaffected — legacy records with any existing `focusValue`/`materialHeightValue` (including empty/missing values, already tolerated by the existing `?? ''`/`|| ''` patterns throughout) render with the new label text exactly as they would have with the old label text; no new validation or default-value logic is introduced.

### 13. Accessibility requirements

No new ARIA attribute or keyboard behavior is required. The existing `labelText(label, tip)` mechanism already associates the label and help text with each input; changing the label/tip string arguments preserves that association automatically. Verify (manual check, §19) that the new label text does not introduce awkwardly long `<label>` text that could wrap poorly in the two-column form grid at narrower widths.

### 14. Responsive requirements

No layout/CSS change is included. The field-reordering (moving Focus height and Material height / Z-offset adjacent to each other) should be verified at the four suggested widths (§14 above / §19 below) purely to confirm the reorder doesn't produce an awkward visual break in the existing `span2`/grid layout — this is a manual verification step, not a new responsive feature.

### 15. Required fixtures

- A new or extended fixture (added to an appropriate existing group, e.g. a "terminology consistency" case inside `runBeginnerClarityFixtures` or a small new focused group) asserting: the same standardized label string appears for `focusValue` in the Log form, Library form, Project form, and Production Settings editor; the `materialHeightValue` label no longer contains the exact substring that collides with the `focusValue` label; the promotion-review screen's `focusValue` label matches the standardized wording; the Wizard's `zOffset` field is unaffected (still renders its own existing label, confirming no cross-contamination between the three concepts).
- A regression assertion that `backupObject()` and the stored `STORAGE_KEY` payload are byte-identical for a fixture record before and after rendering with the new labels (proving the label change touches presentation only).
- A regression assertion that `readForm` still reads `focusValue`/`materialHeightValue` correctly from the reordered form fields (proving the reorder doesn't break form-field name wiring).
- Full re-run of `runHelpSafetyFixtures`, `runBeginnerClarityFixtures`, `runAccessibilityPolishFixtures`, `runResponsiveTuningFixtures`, and the Library/Project/Production-Settings-relevant existing groups, expecting zero regressions, since none of their existing assertions should reference the exact old label wording being changed (a full-file assertion grep for the old label strings, as recommended in §7 finding 9, should be the implementer's first step to confirm this before making any change).
- Full Designs geometry/production-golden group, expecting zero change (proving the label change has no production-output effect, as determined in §7/§18.10).

### 16. Required manual browser checks

Direct `file://` load in a disposable browser profile: open the Log form, Library form, Project form, and a Production Settings editor side by side (or in sequence) and visually confirm the same wording appears for the focus field in all four; confirm the reordered Focus-height/Material-height fields display correctly at approximately 1440, 1024, 768, and 480 px without label/input detachment or overflow; confirm the Test Grid promotion-review screen shows the updated label; confirm the Material Test Wizard's own "Z offset (mm)" field is visually unchanged. None of this was executed as part of this review — this is a review, not an implementation or verification pass.

### 17. Clear acceptance criteria

1. Exactly one consistent label string is used for `focusValue`/`focusUnit` across every form and summary listed in §18.3.
2. `materialHeightValue`'s label no longer shares the ambiguous "Z-offset" wording with `focusValue`'s label.
3. Help text for both fields explains their distinct purpose and relationship.
4. `git diff` for this phase touches only label/help-text string literals and the shared form's field ordering — no stored key, no schema constant, no persistence/merge/backup/vault function body.
5. `backupObject()` output and `STORAGE_KEY` payload are byte-identical before/after for equivalent record data.
6. Full `?selftest=all` shows zero regressions in any group, especially Designs geometry, Help/Safety, Beginner Clarity, Accessibility Polish, and Responsive Tuning.
7. A legacy record with any prior focus/material-height values (including empty) still loads, edits, and saves correctly with the new labels.

### 18. Risks

- **Risk**: choosing final wording that is itself unclear or too similar to another existing label — mitigated by picking one term and applying it consistently (the actual defect being fixed), and by the explicit "no color-only distinction, must survive grayscale/screen readers" principle already implicit in this being a text-only change.
- **Risk**: missing a rendering site that also displays the old label text, leaving one inconsistent holdout — mitigated by the recommended first implementation step (a full-file grep for every current label string before changing any of them, per §7 finding 9).
- **Risk**: reordering fields in the shared `modalFields` template subtly affecting tab order for keyboard users — mitigated by the required manual keyboard-tab-order check alongside the visual check in §19, and by the fact that DOM order and tab order are the same mechanism here (no explicit `tabindex` overrides exist in this form), so a reorder is inherently keyboard-order-consistent, not a separate risk to manage.

### 19. Rollback strategy

Because this phase changes only label/help-text string literals and field order (no stored key, no schema version, no storage key, no production output), rollback is a plain code revert with no data migration to reverse and no persisted state to unwind — materially simpler than any of the Security or Inventory phases' rollback stories, for the same reason those phases' own reports use as their own best-case comparison point.

### 20. Whether one independent post-implementation audit is warranted

**Yes, recommended** — specifically to (a) confirm the full-file label-consistency grep found and updated every occurrence, (b) confirm no fixture assertion was left pinned to old wording, (c) confirm `backupObject()`/`STORAGE_KEY` byte-identity, and (d) perform the manual browser/keyboard/responsive checks this review could not perform. This mirrors the same "focused post-implementation verification" pattern already used for the Tabletop Accessories and Security phases in this repository's `docs/` history.

---

## 19. Suggested independent audit boundary

A follow-up, read-only verification pass should: re-grep the entire file for every previous exact label string ("Focus height," "Focus/Z-offset," "Focus value," "Material height / Z-offset") to confirm none survive outside the Wizard's intentionally-unchanged `zOffset` field; re-run the full fixture suite and independently reconcile the total against the pre-phase baseline; perform the direct-`file://` manual checks (label consistency across four forms, four-width responsive check, keyboard tab order through the reordered fields); and confirm `backupObject()`/`STORAGE_KEY` byte-identity for a representative record set.

---

## 20. Concise Codex implementation handoff

> Implement the GUI Clarity Option-A terminology pass in `C:\Genmitsu L8 Tracker\index.html`, per `docs/GUI_CLARITY_ARCHITECTURE_AND_USABILITY_REVIEW_2026-07-23.md` §18. First, grep the entire file for every current occurrence of "Focus height," "Focus/Z-offset," "Focus value," and "Material height / Z-offset" to build a complete site list (known sites include `index.html:12946`, `13068`, `11024`, `2594`, `8020`, `9492-9493`, `9635-9636`, `12295`, `12306`, `12415`, and `12955`). Standardize `focusValue`/`focusUnit`'s label to one consistent wording at every site, rework `materialHeightValue`/`materialHeightUnit`'s label so it no longer contains the colliding "Z-offset" wording, rewrite both fields' help text to explain their distinct purposes and relationship, and move the two fields adjacent to each other inside the shared `modalFields` template (`index.html:12931-12963`). Do not touch the Material Test Wizard's own unrelated `zOffset` field's label. Do not rename any stored field, change `SCHEMA_VERSION`, alter `backupObject()`'s structure, or touch any Designs/production-generator code. Add fixture coverage asserting label consistency across the Log/Library/Project/Production-Settings forms and the Test Grid promotion screen, plus a `backupObject()`/`STORAGE_KEY` byte-identity regression check, then re-run the complete `?selftest=all` suite and perform the manual direct-`file://` checks in §18.16 before calling the phase complete.
