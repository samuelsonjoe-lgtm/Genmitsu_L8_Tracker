# Designs Result Summary and Warning Consistency — Architecture and UX Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-18
**Review type:** read-only architecture and UX review. No file was edited. Nothing staged, committed, or pushed.

## 0. Repository state

- `git status -sb`: `## main...origin/main`; tracked tree **clean**; staging **empty**; `git branch --show-current` → `main`; `git rev-list --left-right --count HEAD...origin/main` → `0 0` (**fully synchronized**).
- `git log -1`: **`fea4b90105a85dd60ede79e57853bd266a648c3f` — "Add SVG scale safety guidance"** — matches the stated baseline `fea4b90` exactly.
- `git diff --check` / `--stat` / full diff / `--cached --name-only`: all empty. Working tree is byte-identical to `fea4b90`; every finding below is read from that exact source.
- Untracked files: the same long-standing 69-entry set (`LightBurn Projects/*`, `debug.log`, `parametric_qr_stand_generator.py`, prior `docs/*.md` reports). All preserved untouched.
- Baseline totals (264 / 1,080 / 1,872) are taken as given; this is a forward-looking priority/architecture review, not a correction audit, so they were not re-executed this turn.

## 1. Files and functions reviewed

`designResultsHtml()` in full (`:3457-3608` — every template's `metrics` branch, the shared `messages`/`scaleSafetyNotice` construction, every `assembly` branch, the `productionSvgSize` shared metric, the Drawer Cabinet multi-output `outputCards`); `designMetric()` (`:3364`, a trivial `<div class="metric">` wrapper — confirmed it has no formatting logic of its own, so every inconsistency found below is a call-site choice, not a helper defect); `designLayoutSizeWarning()` (`:3350`, confirmed the shared >400 mm helper now used by all ten builders per the prior phase); `buildDesignResult()` dispatch (`:3353-3362`); `normalizeDesignDraft()`'s per-template branches; every registered builder (`buildLegacyDesignResult`, `buildTrayDesignResult`, `buildBoxDesignResult`, `buildSlidingLidDesignResult`, `buildWallBaseTabCouponDesignResult`, `buildJointCouponDesignResult`, `buildDrawerCabinetDesignResult`, `buildConcealedCleatCornerDesignResult`, `buildConcealedCleatFullBoxDesignResult`) for their `errors`/`warnings` push sites; every `build*FinishedViewSvg()` helper and `designPreviewModeForTemplate()`/`designPreviewSelectorHtml()`; `downloadCurrentDesignSvg()`; the `.design-message`/`.error`/`.warning` CSS rules (`:186-188` — confirmed border-color-only styling, no ARIA role); `refreshDesignPreview()`/`bindDesignPreviewActions()`. README's Designs section. Docs read: the Finger Box Finished View chain, the Output-Size/Scale-Safety priority review (this reviewer's own prior turn), the Drawer Cabinet multi-grid/custom-row chain, Sliding-Lid/Dice/Divider Finished View chains, both Joint Fit Coupon mode reports, both Concealed Cleat design reviews.

## 2. Full template metric inventory (as currently displayed, in order)

| Template/mode | Current metrics, in displayed order (verbatim labels) |
|---|---|
| `qr-stand`, `hanging-sign` | `Shapes` (panel count only) — falls to the final generic fallback (`:3582`); no dimensions, thickness, or fit shown at all |
| `dice-tray` | Inside usable area · Wall height · Outside footprint · Wall profile · Bottom cover · Base thickness |
| `divider-tray` | Inside usable area · Wall height · Dividers / compartments · Divider direction / order · Wall profile |
| `finger-box` | Inside dimensions · Outside body · Overall closed height · Panels · Corner decoration · Assembly labels · Requested finger width · Width/Depth/Vertical-edge fingers |
| `sliding-lid-box` | Usable cavity · Wall-to-wall interior · Outside body · Lid shoulder/tongue/overall length · Side clearance · Vertical clearance · Front insertion clearance · Rail channel height · Lid engagement · Rails · Open channel/Back stop · Shortened Front height · Open end/direction · Rail guide marks · Assembly labels · Sliding-fit coupon · Required pieces · Body joint clearance · Requested finger width · per-edge finger patterns |
| `drawer-cabinet` (uniform) | Drawer grid/count · Thickness mode · Production files · Shell/drawer-interior thickness · Drawer inside · Drawer outside · Cabinet interior · Cabinet outside · Rows/drawers · Cabinet shell/shelves/partitions · Shelf bottom elevations (if any) · Required pieces · Interior production SVG size (if separate mode) · Lateral clearance · Vertical/rear clearance · Cabinet/drawer joints · Shelf-placement guides · Assembly labels · finger-pattern lines |
| `drawer-cabinet` (custom-row) | Drawer layout · Row configuration · Maximum drawers in a row · per-row drawer-count/width lines, *then* the same Thickness-mode-onward list as uniform, **omitting** Drawer inside/outside (replaced by the per-row lines) |
| `joint-fit-coupon` (finger-edge) | Clearance values · Pairs/pieces · Material thickness · Mating edge/grip depth · Finger pattern · Clearance labels |
| `joint-fit-coupon` (wall-to-base) | Coupon type · Wall profile · Test wall/fixed height · Candidate rows · Walls/base plate · Slots per row · Row order |
| `concealed-cleat-corner-prototype` | Prototype type · Material thickness · Prototype outside leg length · Wall height · Wall A/Wall B · Cleat construction · Base cleat A/B · Seam cleat · Physical pieces · Base-cleat footprint/occupancy · Interior intrusion · Assembly labels |
| `concealed-cleat-full-box-prototype` | Prototype type · Outside box · Internal box · Cleat construction · Base/seam cleat stacks · Clear central floor · Interior intrusion/occupancy · Approximate clear volume · Physical pieces · Assembly labels |

Every valid result additionally gets a shared **`Production SVG size`** line prepended before the template-specific block (`:3469`, `productionSvgSize`), and every valid result gets the shared scale-safety notice (`:3584`) after messages.

## 3. Warning/error inventory (representative, not exhaustive)

Every builder's `errors` array blocks the download (`refreshDesignPreview()` disables the button on `!result.valid`); every `warnings` array entry is advisory only and never blocks. Confirmed patterns:

- **Invalid geometry / impossible dimensions** (hard errors, correctly never demoted): non-finite/non-positive thickness, width, depth, height across every template; `Wall B length must be greater than zero` and sibling messages in both Concealed Cleat builders; finger-pattern collapse messages in Finger Box/Sliding-Lid/Drawer Cabinet/coupons.
- **Unsafe/unsupported construction assumptions** (warnings, correctly never promoted to errors): "Zero drawer lateral clearance is unlikely to slide reliably…", "Cabinet joint clearance is negative…", thin-material cautions in both Concealed Cleat builders.
- **Fit/retention limitations**: Sliding-Lid's guide/label/coupon disabled-state lines; Finger Box's loose-lid "no hinge, rail, retention" language (in the `assembly` note, not `warnings`).
- **Production-layout advisories**: the shared `designLayoutSizeWarning()` (>400 mm) now present in all ten builders per the prior phase.
- **Workflow guidance**: the shared scale-safety notice; per-template "Assembly:" notes.

No case was found where a genuine blocker is worded as a warning, or an advisory limitation is wrongly hard-blocked — the error/warning split itself is sound. The problems found are entirely in **wording consistency, placement, and one dead-code defect**, detailed below.

## 4. Dimensional-label findings

- **`x` vs `×` inconsistency, verified at the source level.** Drawer Cabinet's metrics (`:3516-3520`) use the literal ASCII letter `x` ("`... x ... x ...`") for Drawer inside/outside, Cabinet interior/outside, and Rows/drawers, while every other template with a multi-axis dimension (Finger Box `:3572-3573`, Sliding-Lid `:3533-3535`, both Concealed Cleat templates `:3487-3488`/`:3503`) uses the Unicode multiplication sign `×`. This is a genuine, easily-verified inconsistency, not a stylistic assumption.
- **"Production SVG size" is ambiguous for Drawer Cabinet's separate-thickness mode.** The shared top-level metric (`:3469`) reads `result.widthMm`/`result.heightMm`, which for a separate-thickness cabinet is **silently the shell output's size only** (confirmed: `buildDrawerCabinetDesignResult()`'s separate-mode branch sets `baseResult.widthMm = shellOutput.widthMm`), while a second, explicitly-labeled `Interior production SVG size` line appears further down (`:3524`) only when `interiorProductionSheet` exists. A user who reads only the first, always-present "Production SVG size" line has no indication it describes the shell alone — the label itself does not say "Shell."
- **No other template mislabels finished-vs-production dimensions.** Coupons correctly use coupon-specific language ("Test wall / fixed height," "Mating edge / grip depth") rather than forcing box-style width×depth×height onto flat test artifacts; both Concealed Cleat prototypes correctly label "Prototype outside leg length" / "Outside box" rather than implying a finished consumer product. QR stand/Hanging sign correctly avoid box terminology entirely (they show only `Shapes`) — though this is arguably *too* sparse (§13).

## 5. Material/fit findings

Material thickness is shown with a clear, consistent label everywhere it appears (coupons: "Material thickness"; Concealed Cleat: "Material thickness"; Drawer Cabinet: "Shell/drawer-interior thickness"), and no template exposes Focus/Z-offset terminology — confirmed by direct search: those terms exist only in Log/Production-Settings/Material-Test rendering, never in `designResultsHtml()` or any Designs builder. **This confirms the Designs phase should not touch that terminology at all** (it is out of scope by construction, not merely by policy). Clearances (lateral/vertical/rear for the cabinet, side/vertical/insertion for sliding-lid, joint clearance for coupons) are each labeled with what they affect, and finger-pattern lines already distinguish "Requested finger width" (input) from the per-edge "N segments × actual mm" (generated) — this input-vs-generated distinction is already handled well and needs no change.

## 6. Piece/operation findings

Piece counts already exist per template (`Panels`, `Required pieces`, `Physical pieces`, `Pairs/pieces`) but under **three different label words** for the same concept across templates — a real, minor inconsistency. **A reliable, non-SVG-parsing operation summary is derivable today**: every result already carries `scorePaths`/`labelPaths`/`panels` arrays (or their per-output equivalents) before serialization — a `Cut: N · Score: N · Labels: N` line can be built directly from `result.panels.length`/`result.scorePaths?.length`/`result.labelPaths?.length` without touching `result.svg`. Assembly-label state is already reported per template but with inconsistent enabled/disabled phrasing ("Enabled - N safe labels" vs "Enabled — N intended-face labels" vs "Enabled - N emitted / N omitted") — same concept, three different sentences.

## 7. Warning-ordering and actionability findings

Messages currently render in **declaration order within each builder**, not by any cross-template priority rule — a builder that happens to push a thin-material warning before its clearance warnings will show them in that arbitrary order. Several existing warnings are already genuinely actionable ("increase X," "reduce Y" is implicit in e.g. "Cabinet ... fingers collapse below the required web at this clearance" naming the exact offending value), but none are ever reordered by severity within a single result — e.g. a fit/retention limitation can currently appear *after* a production-layout advisory in source order, which the task's own priority scheme (invalid → unsafe-assumption → fit/retention → layout advisory → workflow guidance) would put last.

## 8. Warning-fatigue findings

The scale-safety notice (`:3584`) is a single `.design-message` block with no distinguishing class from a plain informational note — it sits visually identical in weight to a real error (aside from border color) since it shares the same `.design-message` box styling and always appears once per valid result. It is not spammed per-warning, so it does not itself cause repetition, but it is not visually separated from the template's own `warnings` list either — both render as a flat stack of same-styled boxes with no grouping heading.

## 9. Finished View interaction findings

No conflict was found: the metrics/warnings block renders identically regardless of which preview mode is selected (`designResultsHtml()` builds `metrics`/`messages`/`scaleSafetyNotice` before branching on preview mode for the `preview` variable only) — so the production-SVG-size metric and scale-safety notice remain accurate and visible under every Finished View. This is correct and needs no change.

## 10. Drawer Cabinet multi-output findings

The recent consolidation is **not fully sufficient**, precisely because of the ambiguity in §4: "Production SVG size" (unlabeled) plus "Interior production SVG size" (labeled) together describe two artifacts, but only one is named as such. **Recommendation: rename the shared top-level metric to read "Shell production SVG size" specifically when `result.requiresMultipleProductionOutputs` is true**, paired with the existing "Interior production SVG size" — both explicitly scoped, neither implying it is the whole cabinet. In linked (single-file) mode, the current unscoped "Production SVG size" remains correct and unchanged, since there is only one file.

## 11. Concealed-cleat findings — a real, verified defect, not a style nit

**The Concealed Cleat Full-Box Prototype's assembly instructions are dead code and never reach the user.** Tracing `designResultsHtml()`'s `metrics` ternary precisely (`:3485-3498`): `jointCoupon ? … : fullCleat ? [metrics list, :3486-3495] : fullCleat ? [assembly-instructions <div>, :3497] : cleat ? …`. Because JS ternaries are right-associative, the **second** `fullCleat` check at `:3496` can only ever be reached when the **first** `fullCleat` check at `:3485` was false — meaning `fullCleat` is already known false by the time the dead branch is evaluated, so **that assembly-instructions block can never render, for any input.** Worse, the actual `assembly` variable (`:3589-3605`, the block genuinely rendered below the metrics panel) **never checks `fullCleat` at all** — its ternary chain goes `jointCoupon → cleat → cabinet → finger → dice → divider → sliding → assemblyLabels-fallback → ''`. Since the Full-Box prototype does have assembly labels enabled by default, it falls into the generic fallback at `:3605`: *"Assembly labels: Marks are placed on the displayed sheet face... the **finger-box** cut geometry is unchanged."* **This literally names the wrong template** and omits the Full-Box prototype's real, carefully-written 7-step lamination/base-cleat/seam-cleat/square-and-clamp sequence, which exists in source but is unreachable. This is a genuine, source-proven defect that this phase should fix as its most concrete, highest-value single correction — not a hypothetical UX nicety. The single-corner `concealed-cleat-corner-prototype`'s assembly note, by contrast, **is** correctly wired (`:3593-3594`) and renders as intended.

Both templates already correctly avoid strength/durability claims and correctly state "assembly and registration prototype, not a strength test" language, consistent with the physically-validated-for-fit-only status.

## 12. Coupon findings

Both coupon modes clearly show the tested variable (`Clearance values` / `Candidate rows`), a sample count implied by array length, material thickness, and an explicit "Use:" note distinguishing test-artifact status from a finished product ("does not replace final box testing," "Promotion is not available… record any winning clearance manually"). No Finished View exists for either mode, correctly. One clarity gap: neither mode's metrics explicitly states step size or whether values are read as offsets vs. clearances vs. kerf as a labeled concept — the "Use:" note explains the testing procedure but the metrics block itself doesn't define the unit semantics of "Clearance values"/"Candidate rows" beyond the numbers themselves. A one-line label clarification ("signed clearance offset, mm — not kerf compensation") would close this without adding a new field.

## 13. QR stand and Hanging Sign findings

These two templates currently show **only `Shapes: N`** — no finished size, no material thickness, no operations. This is the sparsest case in the inventory and is arguably under-informative relative to every other template, even though the original Finished-View architecture review correctly decided these need no *Finished View*. A result summary is a different, smaller ask than a Finished View: **recommend adding, at minimum, finished size and material thickness** (both already computed in `buildLegacyDesignResult()`/`designStandSvg()`/`designHangingSignSvg()`'s inputs) to bring these two up to the same minimal-summary floor as every other template, without adding assembled-view complexity.

## 14. Accessibility findings

- `.design-message.error`/`.warning` differ **only by left-border color** (`:186-188`) — but the message text itself already carries a bold `Error:`/`Warning:` text prefix (`:3583`), so the distinction is **not actually color-only** even though the CSS alone would be; this should be preserved, not treated as already broken.
- **No `role="alert"` or `aria-live` region exists** on `#designResults` or the individual message `div`s — a screen-reader user who edits a field and introduces an invalid dimension will not be automatically notified that new errors appeared; they must manually re-navigate to the results panel. This is a real, fixable gap appropriate for this phase (adding `role="status"` to the results container, or `role="alert"` specifically to newly-rendered error blocks, is a small, presentation-only change).
- Repeated labels/long prose: several assembly notes (`:3596`, `:3604`) are dense single paragraphs; no heading structure currently separates "Design results" from "Assembly:" from the scale-safety notice — they are visually sequential `.design-message` boxes with no `<h4>`/`<h3>` grouping, relying entirely on a leading bold word. A small, `<h4>`-level heading per group (Metrics / Messages / Assembly) would improve screen-reader landmark navigation without new interactive controls.
- Narrow-viewport wrapping was not found to be broken — `.metrics`/`.design-message` use flex/block layout already exercised across long labels (e.g., "Base-cleat footprint / occupancy") without a discovered overflow issue in source.

## 15. Candidate comparison

| Candidate | User value | Size | Production-byte risk | Storage/schema risk | Shared-renderer risk | Fixture burden | Scope-creep risk |
|---|---|---|---|---|---|---|---|
| A — ordering/label consistency only | Medium | Small | None | None | Low | Small | Low |
| B — grouped sections with headings | Medium-High | Medium | None | None | Low-Medium (new grouping logic in one shared function) | Medium | Medium |
| C — warning wording/ordering only | Medium | Small-Medium | None | None | Low | Small-Medium | Low |
| **D — combined compact summary + warning pass** | **High** | **Medium** | **None** | **None** | **Low (one function, `designResultsHtml`, plus per-builder metric-array tweaks)** | **Medium, fully behavioral** | **Low, if scope is held to presentation** |
| E — full result-card redesign | High long-term | Large, ill-defined | Unknown | Unknown | High | Large | High |

**D is selected.** A alone would leave the dead-code Full-Box defect and the QR/Hanging-Sign sparseness unaddressed; C alone would leave the `x`/`×` and label-word inconsistencies unaddressed; B alone (headings without fixing wording) would look more organized while still showing wrong or ambiguous text. D combines the smallest set of concrete, already-identified fixes without inventing new UI chrome. E is not justified — every problem found in §§2-14 is fixable inside the existing `designResultsHtml()`/per-builder metric-array structure; no source evidence supports a redesign.

## 16. Exact recommended implementation scope

**Files changed:** `index.html` only (presentation-layer, one function plus small per-builder metric-string edits), `README.md` (documentation, §20).

**Functions changed:**
- `designResultsHtml()` — the primary target. Fix the dead-code `fullCleat` duplicate branch (move the assembly-instructions block from the unreachable `metrics` ternary slot into the `assembly` ternary chain, adding an actual `fullCleat` branch there); standardize the `x`→`×` separator in the Drawer Cabinet metric lines; rename "Production SVG size" to "Shell production SVG size" conditionally on `result.requiresMultipleProductionOutputs`; standardize piece-count and assembly-label enabled/disabled wording to one shared phrasing pattern reused across every template's metric string; add a compact `Operations` metric line (`Cut: N · Score: N · Labels: N`) derived from existing `panels`/`scorePaths`/`labelPaths` lengths; add light `<h4>` grouping headings around the existing metrics/messages/assembly blocks; add `role="status"` (or `aria-live="polite"`) to the results container.
- `buildLegacyDesignResult()` — add finished-size and material-thickness metrics for `qr-stand`/`hanging-sign` (reading already-computed `designDraft` values; no new geometry).
- No geometry/model builder's **panels, scorePaths, labelPaths, panel IDs, or SVG-affecting fields** are touched — only which already-computed values are *displayed* and in what order.

**Should `designResultsHtml` own ordering/grouping?** Yes — it already owns 100% of this rendering; a small `designMessageGroup()`-style local helper (heading + message list) is a reasonable, minimal addition, not a new architecture layer.

**Is a normalization helper appropriate?** Yes, narrowly: a tiny shared formatter for "N × M × H mm" (replacing every ad hoc `x`/`×` string-build call) removes the class of separator inconsistency at its source rather than patching each call site independently.

**Exact metrics preserved:** every metric currently shown, unchanged in meaning and value. **Exact metrics renamed (not removed):** "Production SVG size" → "Shell production SVG size" (separate-mode cabinet only); piece-count labels unified; assembly-label enabled/disabled phrasing unified. **Exact metrics added:** finished size + material thickness for QR stand/Hanging Sign; one `Operations` summary line per template. **Nothing is removed.**

## 17. Production-byte contract

**No production SVG bytes may change anywhere.** Protected, unmodified functions: every `build*DesignResult()` builder's geometry/panel/score/label construction; `serializeDesignSvg()`; the Concealed Cleat local serializer (`serializeConcealedCleatPathGroup`/`serializeConcealedCleatSvg`); `layoutDesignPanelRows()`/`layoutDesignPanels()`; `downloadCurrentDesignSvg()`; every `build*FinishedViewSvg()` (unless a display-label conflict forces a tiny local text correction — none was found necessary; §9 confirms no conflict exists). **Fixtures proving neutrality:** for every one of the ten templates, `buildDesignResult(draft).svg` byte-identical before/after this phase for a representative draft; every currently-pinned golden hash unchanged; `result.widthMm`/`heightMm` unchanged; `downloadCurrentDesignSvg()` output unchanged; Finished View markup byte-identical for every `build*FinishedViewSvg()` output on the same result.

## 18. Storage/schema contract

**No new storage field, no schema change, no persisted UI preference, no backup/import/export change, no draft migration.** Every proposed change is derived entirely from values already present in `result.metrics`/`result.panels`/`result.scorePaths`/`result.labelPaths`/`result.widthMm`/`result.heightMm` — nothing new needs to be computed, stored, or remembered across a reload. `STORAGE_KEY`/`SCHEMA_VERSION`/`freshState()`/`loadState()`/`persist()`/`backupObject()` are untouched.

## 19. Fixture strategy (behavioral, table-driven)

1. For every template/mode (11 rows matching §2's table), assert the exact expected metric label set is present in `designResultsHtml(result)`'s output (by parsing the returned markup's metric labels, not by grepping `index.html` source).
2. **Full-Box assembly-note fix, verified directly:** assert the rendered `assembly` block for a valid `concealed-cleat-full-box-prototype` result contains the lamination/base-cleat/seam-cleat sequence text and does **not** contain the string "finger-box."
3. No duplicate "Production SVG size" ambiguity: for a separate-thickness Drawer Cabinet result, assert the top metric reads "Shell production SVG size" and its value equals `result.productionOutputs[0].widthMm`/`heightMm` exactly (independently read from the output object, not restated).
4. Separator consistency: assert every multi-axis dimension metric across all templates uses `×`, never a bare `x`, via a single regex pass over the rendered metrics markup for a representative result per template.
5. Piece-count and assembly-label wording: assert the same phrasing pattern appears across at least three templates that report each concept.
6. Operations line: assert `Cut: N` equals `result.panels.length`, `Score: N` equals `(result.scorePaths||[]).length` (or per-output sum for Drawer Cabinet separate mode), `Labels: N` equals `(result.labelPaths||[]).length`, each computed independently in the fixture.
7. QR stand/Hanging Sign: assert finished size and material thickness now appear and match `designDraft` inputs.
8. Warning-priority ordering: for a representative draft engineered to trigger both a fit-limitation warning and the large-layout advisory simultaneously, assert the fit-limitation message renders first.
9. Advisory-vs-error distinction: assert `result.errors.length > 0` disables the download button and `result.warnings.length > 0` alone does not, for at least one template.
10. One scale-safety notice, one large-layout advisory: assert exactly one instance of each appears regardless of how many other warnings are present.
11. Invalid-result handling: assert an invalid draft still renders a coherent (non-crashing) results panel with the "Preview unavailable" message and no stray metric referencing undefined values.
12. Finished View consistency: assert the metrics/warnings/scale-notice markup is byte-identical between Cut Layout and Finished View modes for the same valid result.
13. Accessibility markup: assert the results container carries `role="status"` (or equivalent) and that error/warning divs retain their bold text prefixes.
14. Narrow-viewport behavior: no new fixture needed beyond confirming no fixed-width style was introduced (existing responsive CSS is reused, not changed).
15. Production-byte neutrality and download neutrality: per §17.
16. Storage/backup isolation: `localStorage`/`backupObject()` byte-identical before/after rendering results for every template.
17. Machine-profile independence: assert identical rendered output regardless of `state.machineProfile` (there is no dependency to break, confirmed in the prior phase's review; this fixture guards against one being accidentally introduced).

## 20. Documentation plan

README's Designs paragraph gains one short addition: result summaries are now grouped and consistently worded across every template; a shared "Operations" line reports cut/score/label counts derived from the same production model, not a re-parsed SVG; Drawer Cabinet's separate-thickness mode explicitly labels its shell output size distinctly from its interior output size; the Concealed Cleat Full-Box Prototype's assembly note now correctly displays its own instructions. No claim of new capability (no nesting, no multi-sheet export, no bed configuration) is added to documentation, matching the non-goals below.

## 21. Risks

- **Primary risk is scope creep from "consistency" into a visual redesign** — mitigated by holding strictly to relabeling/reordering existing values and fixing the one verified dead-code path, with no new layout containers beyond light heading text.
- **The dead-code fix must not be treated as a "small" change without a specific fixture** (item 2, §19) verifying the corrected text actually renders — this is the one place in this phase where a careless refactor could re-introduce the same right-associative-ternary mistake.
- Renaming "Production SVG size" conditionally could be missed for one of the two separate-mode call sites if not centralized through the single shared `productionSvgSize` construction already at `:3469` — the implementation should extend that one expression, not add a second, parallel one.

## 22. Non-goals

Full Designs form redesign; navigation redesign; automatic nesting; multi-sheet export; machine-bed settings; material-sheet profiles; cut-time estimates; cost estimates; warning-dismissal persistence; custom per-user result-card layouts; new Finished Views; new generators; new joints; physical fit or strength prediction; kerf simulation; storage-key renaming; Focus Height/Z-offset terminology changes (confirmed out of scope by source inspection, §5 — Designs never shows this terminology today).

## 23. Post-implementation audit recommendation

**Grok performs a focused adversarial audit; a second Claude review is unnecessary** for this phase specifically because it changes no shared semantic model, no production serializer, no storage/schema, and no multi-output download behavior — it is a presentation-layer pass over one already-understood function plus two small builder additions. **Claude Opus 4.8 is unnecessary** (no production-serialization/shared-geometry/storage-schema/multi-sheet-export risk). **Fable 5 is unnecessary** (no unusually deep cross-system risk).

## Final verdict

**READY FOR BOUNDED IMPLEMENTATION**

- **Recommended implementation report filename:** `docs/DESIGNS_RESULT_SUMMARY_WARNING_CONSISTENCY_IMPLEMENTATION_2026-07-18.md`
- **Recommended implementation audit filename:** `docs/DESIGNS_RESULT_SUMMARY_WARNING_CONSISTENCY_FOCUSED_AUDIT_2026-07-18.md`
- **Should Codex implement directly?** Yes — the scope is fully enumerated above, including the one concrete defect (§11) that should be fixed as part of this same phase since it is squarely a result/assembly-summary correctness issue, not a separate feature.
- **Is Grok sufficient afterward?** Yes.
- **Is Claude Opus 4.8 unnecessary?** Yes.
- **Is Fable 5 unnecessary?** Yes.
