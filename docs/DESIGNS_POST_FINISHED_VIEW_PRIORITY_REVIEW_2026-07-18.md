# Designs Tab Priority Review — After Finger Box Finished View

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-18
**Review type:** read-only architecture and UX priority review. No file was edited. Nothing staged, committed, or pushed.

## 0. Repository state

- `git status -sb`: `## main...origin/main`; tracked tree **clean**; staging **empty**; `git branch --show-current` → `main`; `git rev-list --left-right --count HEAD...origin/main` → `0 0` (**fully synchronized**).
- `git log -1`: **`ad4532cfa7d5c112a25da4cbd17ac5cefd164e1c` — "Add Finger Box finished view"** — matches the stated baseline `ad4532c` exactly.
- `git diff --check` / `--stat` / full diff / `--cached --name-only`: all empty. Working tree is byte-identical to `ad4532c`.
- Untracked files (69 entries — the same long-standing set: `LightBurn Projects/*`, `debug.log`, `parametric_qr_stand_generator.py`, prior `docs/*.md` reports). All preserved untouched.
- Baseline totals (264 / 1,072 / 1,864) and the three Finger Box goldens are taken as given per the task and were not independently re-executed this turn; nothing in this review's conclusions depends on re-verifying them, since this is a forward-looking priority review, not a correction audit.

## 1. Files and functions reviewed

`index.html`: `buildDesignResult()` dispatch (`:3353-3362`, ten registered result builders); every `build*FinishedViewSvg()` function (Finger Box, Dice Tray, Divider Tray, Sliding-Lid, Drawer Cabinet front elevation, Concealed Cleat corner, Concealed Cleat full-box — read in full, `:3365-3428`); `designPreviewModeForTemplate()` / `designPreviewSelectorHtml()` (`:3429-3448`); `designResultsHtml()` metrics/warnings block; `normalizeDesignDraft()`'s per-template branches; all ten `build*DesignResult()` builders, specifically searched for the layout-size warning (`buildLegacyDesignResult`, `buildTrayDesignResult`, `buildBoxDesignResult`, `buildSlidingLidDesignResult`, `buildWallBaseTabCouponDesignResult`, `buildJointCouponDesignResult`, `buildDrawerCabinetDesignResult`, `buildConcealedCleatCornerDesignResult`, `buildConcealedCleatFullBoxDesignResult`); `machineProfile` and every reference to it (`designProductionMachineKey()`, Reference-tab rendering, Production Settings promotion, storage read/write, import/export) — confirmed it is a two-value string (`'20W'`/`'40W'`) used only to select reference charts and tag records, **never a stored physical bed dimension**; `downloadCurrentDesignSvg()`/`downloadTextFile()`; the shared `layoutDesignPanelRows()`/`layoutDesignPanels()` finite/overlap checks. `README.md`'s Designs section. Docs read: `docs/DESIGNS_FINISHED_VIEWS_ARCHITECTURE_REVIEW_2026-07-17.md` (the original Finished View roadmap — its own completion criteria are used below to confirm the roadmap is now finished); `docs/DESIGNS_FINGER_BOX_FINISHED_VIEW_DESIGN_REVIEW_2026-07-18.md` / `_IMPLEMENTATION_2026-07-18.md` / `_IMPLEMENTATION_AUDIT_2026-07-18.md` (today's landed work, verdict SAFE TO COMMIT WITH NON-BLOCKING NOTES, no shared-helper or production risk found); the concealed-cleat corner and full-box design reviews (this reviewer's own prior turns, read for the physical scaling-incident context and the current cleat geometry). No `docs/*NEST*` or `docs/*RELEASE*` file exists in the repository — searched and confirmed absent; no multi-sheet-arrangement or release-readiness document was found, so none is cited.

## 2. Verified inventory of Designs templates and Finished Views

| Template ID | User-facing name | Production purpose | Preview modes | Finished View? | Form | Semantic-only? | Reads `result.svg`? | Dedicated fixtures? | Workshop-useful? |
|---|---|---|---|---|---|---|---|---|---|
| `qr-stand` | Freestanding QR / sign stand | Two flat slot-together parts | Cut Layout only | No | — | — | — | N/A | Cut Layout sufficient; no assembly ambiguity |
| `hanging-sign` | Hanging sign | One flat panel with holes | Cut Layout only | No | — | — | — | N/A | Flat part; no view needed |
| `dice-tray` | Dice tray | Open-top slotted tray | Cut Layout / Finished View | **Yes** | Top-down schematic | Yes (structured tray model) | No | Yes | Yes — orientation and slot alignment are the real question |
| `divider-tray` | Divider tray | Tray + slotted dividers | Cut Layout / Finished View | **Yes** | Top-down schematic | Yes | No | Yes | Yes |
| `finger-box` | Finger-jointed box | 5-panel open-top box, optional loose lid | Cut Layout / Finished View | **Yes (landed this commit)** | **Isometric** | Yes | No | Yes | Yes — new isometric replaces the prior top-down-only view and adds finger-count cues and faux-dovetail decorative labeling |
| `sliding-lid-box` | Sliding-lid finger box | 5 body panels + rails + sliding lid | Cut Layout / Finished Closed / Finished Open | **Yes** | Isometric-projection | Yes | No | Yes | Yes — lid direction is the key question, answered |
| `drawer-cabinet` | Drawer Cabinet | Multi-drawer cabinet shell | Cut Layout / Finished Front View | **Yes** | Front elevation (orthographic) | Yes | No | Yes | Yes |
| `joint-fit-coupon` (finger-edge mode) | Joint Fit Coupon | Flat A/B fit-test pairs | Cut Layout only | No (deliberate) | — | — | — | N/A | Correct — it is a test artifact, not a finished object |
| `joint-fit-coupon` (wall-to-base mode) | Joint Fit Coupon (wall/base) | Flat wall+base fit-test pieces | Cut Layout only | No | — | — | — | N/A | Same reasoning |
| `concealed-cleat-corner-prototype` | Concealed Cleat Corner Prototype | Single-corner registration prototype | Cut Layout / Finished View | **Yes** | Schematic isometric-ish | Yes | No | Yes | Yes, for the prototype's own scope |
| `concealed-cleat-full-box-prototype` | Concealed Cleat Full-Box Prototype | Four-wall registration prototype | Cut Layout / Finished View | **Yes** | Schematic top-down | Yes | No | Yes | Yes, for the prototype's own scope |

**Ten registered templates (counting the two Joint Fit Coupon modes as one template with two internal modes, matching `buildDesignResult()`'s own dispatch), all with a `build*DesignResult()` entry.** Every template the 2026-07-17 Finished View architecture review's own completion criteria called for now has one (Finger Box, Sliding-Lid, Drawer Cabinet, Dice Tray, Divider Tray, both Concealed Cleat prototypes); every template that review said should deliberately have none, still has none (QR stand, Hanging sign, Joint Fit Coupon in both modes). **No deficiency was found that would justify adding or materially reworking a Finished View right now** — the roadmap that started this line of work is complete on its own terms.

## 3. Actual current sheet and bed-fit behavior — verified, not inferred from helper names

- **There is no stored physical bed/work-area dimension anywhere in this application.** `machineProfile` is a two-value string (`'20W'` / `'40W'`) used only by `designProductionMachineKey()` to tag Production Settings/evidence records and by the Reference tab to pick which speed/power chart to display. It is never read as a width/height, and no `bedWidth`/`workArea`-style field exists in `state`, `designDraft`, or anywhere else. A grep of the entire source and every Reference-tab string for an actual working-area dimension (e.g., "400 x 400", "working area", "engraving area") returned **no matches** — the app has never recorded what the L8's real bed size is.
- **The only sheet-size disclosure that exists at all is a generic, unlabeled `> 400 mm` warning**, and it is present in exactly **3 of the 10** registered result builders: `buildBoxDesignResult` (Finger Box), `buildSlidingLidDesignResult` (Sliding-Lid), `buildDrawerCabinetDesignResult` (Drawer Cabinet) — each an identical line, `if (layout.widthMm > 400 || layout.heightMm > 400) warnings.push(...)`. It is **absent** from `buildTrayDesignResult` (Dice/Divider Tray), `buildWallBaseTabCouponDesignResult`, `buildJointCouponDesignResult`, `buildConcealedCleatCornerDesignResult`, and **`buildConcealedCleatFullBoxDesignResult`** — the exact template implicated in the physical incident below.
- **No mention of "100% scale," "do not scale," or any scale-safety language exists anywhere in the source.** Confirmed by direct search; zero matches.
- **A design can currently exceed the (unverified) bed and currently exceed any user-entered sheet size, and the app can only sometimes say so** (only for the 3 templates above), **never split itself across sheets, never export more than one SVG, and never preserve a separated-sheet-group structure** — there is no multi-sheet or nesting code anywhere in Designs. `layoutDesignPanelRows()`/`layoutDesignPanels()` only guarantee panels don't overlap each other and stay within the computed document bounds; they say nothing about any external bed or sheet.
- **The downloaded SVG always has explicit `width="Nmm" height="Nmm"` and a matching `viewBox`** (confirmed in `serializeDesignSvg()` and the Concealed Cleat local serializer) — the file itself is dimensionally exact and portable at 100%; the risk is entirely in what LightBurn (or the user) does with it after import, which the app currently says nothing about.

## 4. Designs workflow findings

Tracing the actual path (select template → dimensions → material/fit → validate → warnings/metrics → Cut Layout/Finished View → download → LightBurn → arrange → cut): the workflow is generally sound and honestly worded up through generation and preview. The concrete friction points, all verified against source rather than assumed:

- **Output size is buried.** `Layout` already appears as one metric line (`designMetric('Layout', ...)`) in every template's metrics block, but it reads as one line among many (dimensions, thickness, clearances, piece counts, warnings) rather than as a decision-relevant "will this fit?" signal.
- **The bed-fit signal that exists (3 of 10 templates) is inconsistent and untraceable** — a user has no way to know from the UI that Dice Tray or either Concealed Cleat prototype simply never checks size at all, versus Finger Box which does.
- **Nothing tells the user the SVG must stay at 100% scale after export.** This is the single largest, most concretely evidenced gap (§ below).
- **Warnings vs. errors are already well-separated per template** (hard `errors` block generation entirely; `warnings` are advisory) — this distinction is sound and does not need rework.
- **Outside vs. inside dimension labeling is already handled per-template with explicit wording** (e.g., "Prototype outside leg length," Drawer Cabinet's "Shell thickness," Finger Box's outside-dimension Finished View) — no confusion was found in the current source between these; this is not a live gap.
- **Focus/Z-offset terminology does not appear anywhere in Designs** (confirmed §7) — it is exclusively a Log/Production-Settings/Material-Test concept, so it is not a Designs workflow friction point at all.

## 5. Recent physical workflow evidence — analysis

The concealed-cleat full-box incident (generated SVG exceeded the visible LightBurn workspace → the whole SVG was selected and scaled down to fit → all thickness-dependent channel widths scaled with it → real 5.96 mm stock no longer fit the now-4.1 mm-wide channels → a smaller, unscaled prototype fit and worked) is treated as real, load-bearing workshop evidence, not a hypothetical. Two independent, verified facts make this exactly the failure mode this review should prioritize fixing:

1. **The template involved (`concealed-cleat-full-box-prototype`) has zero sheet-size warning today** — confirmed in §3. Even the weak, generic `>400 mm` heuristic that 3 other templates have would not have fired for this template regardless of size, because the check simply is not present in its builder.
2. **No part of the app anywhere states that the exported SVG must remain at 100% scale, or explains that moving/rotating pieces in LightBurn is safe while resizing the whole imported group is not.** This is not a template-specific gap; it is a total absence across the entire application.

**Yes — the next priority should directly reduce the chance of this exact mistake recurring**, and it can be done without claiming knowledge this app doesn't have (an unverified exact bed size) and without inventing multi-sheet export the app has never supported.

## 6. Candidate phase comparison

| Candidate | User value | Urgency | Size | Production-byte risk | Storage/schema risk | Shared-helper risk | Fixture burden | Solves the observed incident? | Verdict |
|---|---|---|---|---|---|---|---|---|---|
| **A — Production-size protection and bed-fit guidance** | High | High (real incident) | Small-Medium | None (screen/metrics only) | None | Low (one shared helper, purely additive) | Small-Medium, fully behavioral | **Yes, directly** | **Selected** |
| B — Multi-sheet layout/export | Real but speculative until A ships | Low now | Large | High (new output shape, new goldens per template) | Possible (export manifest?) | High | Large | Only indirectly, and only for boxes actually too big for one sheet | Defer |
| C — Metrics/warning polish | Medium | Medium | Medium | None | None | Low | Medium | Partially, but weaker than A alone | Fold the size-specific piece into A; defer the rest |
| D — Finished View consistency pass | Low right now | Low | Small | None | None | Low | Small | No | Defer — §2 shows the Finished View roadmap is complete |
| E — Broader Designs GUI cleanup | Medium long-term | Low | Large, ill-defined | Unknown until scoped | Unknown | Unknown | Unknown | No | Defer |
| F — Cut-time/material-use estimates | Speculative | Low | Medium | None | None | Low | Medium | No | Defer — no calibrated basis exists yet for honest estimates |
| G — First-run/release-readiness | Unclear priority vs. Designs-specific work | Low-Medium | Large, ill-defined | Unknown | Unknown | Unknown | Unknown | No | Defer; not shown by source to outrank A |

**A is selected.** It is the only candidate that (a) is directly evidenced by a real workshop mistake, (b) is achievable with zero production-byte risk, (c) needs no new storage field, and (d) is small enough to be one bounded, reviewable phase.

## 7. Selected next phase

**Phase name: "Designs Output-Size and Scale-Safety Disclosure."**

**Exact inclusions:**
- Add the existing `layout.widthMm > 400 || layout.heightMm > 400`-style check (generalized, not copy-pasted seven more times) to the **seven** builders that currently lack it: `buildTrayDesignResult` (both Dice and Divider Tray), `buildWallBaseTabCouponDesignResult`, `buildJointCouponDesignResult`, `buildConcealedCleatCornerDesignResult`, and **`buildConcealedCleatFullBoxDesignResult`** (the incident template) — via one small shared helper, e.g. `designLayoutSizeWarning(widthMm, heightMm)`, called from all ten builders (the 3 existing call sites may be left as-is or switched to the helper for consistency; either is acceptable, but the wording must end up identical everywhere).
- Add one new, always-present metric to every template's results panel: **exact output width × height** (already computed as `result.widthMm`/`result.heightMm`; this is a *display* addition, not a new calculation).
- Add a **generic, honestly-worded "scale-safety" notice**, shown once per result (not per-warning-spam), stating: the download is generated at exact real-world millimeters; moving or rotating pieces in LightBurn is safe; **resizing/scaling the imported group is not**, because every wall/joint/channel dimension is tied to the entered material thickness; if the layout looks larger than the visible workspace, **pan/zoom out in LightBurn — do not scale to fit**.
- The existing `> 400 mm` figure is **not** claimed to be the verified L8 bed size (it never has been, and this review found no in-app evidence it corresponds to one) — the wording is corrected everywhere it appears to say *"verify this fits your actual sheet and machine travel"* rather than imply a known, confirmed bed dimension. This is a documentation-honesty fix, not a claim of new bed-size knowledge.

**Exact exclusions (this phase does not do):** multi-sheet splitting or export; any new machine-bed-dimension field or profile; automatic nesting or rotation; cut-time or material-cost estimation; any Finished View change; any change to production panel geometry, IDs, or ordering; any GUI redesign; any terminology migration (Focus/Z-offset is out of scope entirely, §5 workflow findings). This phase does not touch a single existing production formula.

## 8. Production-byte contract

**Production SVG bytes do not change for any template.** The size warning and the "exact output width × height" metric both read `result.widthMm`/`result.heightMm`/`layout.widthMm`/`layout.heightMm`, values already computed before serialization; adding a warning string to the `warnings` array and a metric line to `designResultsHtml()` touches neither `panels`, `scorePaths`, `labelPaths`, nor any call into `serializeDesignSvg()`/the Concealed Cleat local serializer. **Fixtures proving neutrality:** for every one of the ten templates, assert `buildDesignResult(draft).svg` is byte-identical before and after this phase's change (same draft, same output), and assert every currently-pinned golden (Finger Box open/loose/faux-dovetail, Sliding-Lid, Dice/Divider Tray, Drawer Cabinet, both coupon modes, both Concealed Cleat prototypes) is unchanged.

## 9. Storage and schema contract

**No new state of any kind.** The warning/metric values are recomputed on every `buildDesignResult()` call from the existing `designDraft`, exactly like every other metric already is — nothing needs to persist across a reload, nothing needs to be remembered between templates, and no machine-profile data is read (there is none to read, §3). No `STORAGE_KEY`/`SCHEMA_VERSION`/`persist()`/`backupObject()`/import-export change. No new field is justified because none is needed — the phase is pure disclosure of values already in memory.

## 10. Fixture strategy (behavioral, not string search)

1. For each of the ten templates, a representative-oversized draft produces the size warning; a representative-modest draft does not (asserting the **boolean presence/absence** of a warning matching the message pattern, not merely grepping `index.html` source).
2. Exact output bounds: assert the new "output width × height" metric's displayed numeric value equals `result.widthMm`/`result.heightMm` for at least one non-trivial case per template — computed independently in the fixture, not re-read from the same variable the implementation used.
3. One-axis overflow (`width>400, height<400`) and both-axis overflow are both tested and both produce the warning; a case exactly at `400` does not (boundary honesty).
4. Every existing production golden (all templates, all pinned hashes) reconfirmed byte-identical.
5. No fixture anywhere asserts by searching literal warning text inside `index.html`'s source string — every assertion calls `buildDesignResult()`/`designResultsHtml()` and inspects the returned data/markup.
6. `localStorage`/`backupObject()` byte-identity before/after generating a warned and an unwarned result.
7. Download bytes (`downloadCurrentDesignSvg()`) unchanged for a representative case per template.
8. A rotated-fit scenario is **not** claimed as "fits" or "exceeds" without qualification — since there is no real bed dimension to rotate against (§3), this phase does not attempt rotation-aware fit logic at all; the fixture instead confirms the warning wording never claims to know the machine's actual bed size.
9. Machine-neutral default: the warning and metric behave identically regardless of `state.machineProfile`, confirmed directly (no dependency exists to break).

## 11. Accessibility and usability

The notice is **not color-only** — it is text inside the existing `.design-message.warning` pattern already used elsewhere in Designs (which already carries a semantic class, not just a color swatch). It requires no new keyboard interaction (no new interactive control, just a metric line and a warning line). It appears **once per result** to avoid warning fatigue, not once per oversized dimension. It is clearly distinguished from a hard error: it uses the existing advisory `warnings` array (dismissable by fixing dimensions, but never blocking a valid download), never the `errors` array that already disables the download button. Wording stays short (two sentences, not a paragraph) and is identical across templates so a user only has to learn it once. On narrow viewports, the metric and warning wrap using the same `.metrics`/`.design-message` CSS already in place — no new responsive behavior needs to be invented.

## 12. Documentation plan

README's Designs paragraph gains one sentence stating: every template now reports its exact output width and height, warns when either exceeds 400 mm (explicitly *not* claimed as a verified machine bed size — the user must confirm against their own sheet and machine travel), and states that the downloaded SVG must be imported and cut at 100% scale — moving and rotating pieces in LightBurn is safe, resizing the imported group is not, because every material-thickness-dependent dimension (finger widths, tab widths, cleat channels) is computed at that specific scale. Documentation explicitly does **not** claim automatic nesting, multi-sheet export, or any LightBurn automation — none of that is implemented.

## 13. Independent audit requirement

**Claude Sonnet 5 for a broader architecture recheck** is appropriate, not Opus 4.8: this phase touches ten builders with one shared, purely-additive helper and zero production/storage/schema change — real but modest shared-helper surface, not the production-serialization/storage-schema/multi-sheet-export tier that would justify Opus 4.8 per this task's own reservation rule. Grok is a reasonable follow-on adversarial pass after the Sonnet-level review, specifically to independently re-verify golden byte-identity across all ten templates (the one place a subtle mistake could hide). **Fable 5 is not warranted** — no unusually deep cross-system risk was found; this is a bounded, additive disclosure phase.

## 14. Non-goals

Automatic nesting; multiple SVG export; production panel rearrangement; material cost estimation; cut-time prediction; a full Designs redesign; any storage-key change; interactive CAD; LightBurn project-file generation; physical fit prediction; kerf simulation; any new machine-bed-dimension field (deferred pending an honest source for that number — not invented here); rotation-aware bed-fit logic (deferred for the same reason). Nothing essential to making this phase honest or usable is deferred — the "not a verified bed size" caveat is included *in* this phase specifically because omitting it would make the warning dishonest.

## Final verdict

**READY FOR BOUNDED IMPLEMENTATION**

- **Recommended implementation report filename:** `docs/DESIGNS_OUTPUT_SIZE_SCALE_SAFETY_IMPLEMENTATION_2026-07-18.md`
- **Recommended implementation audit filename:** `docs/DESIGNS_OUTPUT_SIZE_SCALE_SAFETY_FOCUSED_AUDIT_2026-07-18.md`
- **Should Codex implement directly?** Yes — the scope is small, additive, and fully specified above (one shared helper, one metric line, one warning sentence, applied to ten already-known builders).
- **Is Grok sufficient afterward?** Yes, as the adversarial byte-identity pass across all ten templates, following a Sonnet-level architecture recheck (§13) — not Grok alone, given the ten-builder surface.
- **Is Claude Opus 4.8 unnecessary?** Yes — no production serialization, shared geometry, storage/schema, or multi-sheet export is involved.
- **Is Fable 5 unnecessary?** Yes — no unusually deep cross-system risk was found.
