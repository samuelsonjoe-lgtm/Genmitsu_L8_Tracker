# Tabletop Accessories T2A — Rectangular Shell Prototype — Focused Implementation Audit

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-20
**Reviewer:** Claude Sonnet 5
**Type:** read-only focused implementation audit. No product file, README, CHANGELOG, fixture, storage, application record, or existing report was edited. Nothing was staged, committed, or pushed. Only this report was written. All runtime probing used a disposable headless-Edge profile directory and synthetic data; Joe's real browser profile and production `localStorage` were never touched.

## 1. Repository state and actual HEAD

- `git log -1 --oneline` / `git rev-parse HEAD`: `d0a89e4afd3bba4de47e924032b3aa64182d8eb7` — "Add Tabletop coupon evidence workflow." Matches the stated committed baseline.
- Branch `main`; `git rev-list --left-right --count origin/main...main` = `0 0` (synchronized).
- `git diff --check`: clean (CRLF-only warnings). Nothing staged (`git diff --cached --stat` empty).
- `git diff --stat`: `CHANGELOG.md | 1 +`, `README.md | 10 +-`, `index.html | 202 ++...+---` (191 insertions, 22 deletions). Tracked changes limited to exactly these three files; T2A remains fully uncommitted.
- Architecture and implementation reports present on disk and read in full.
- `git ls-files --others --exclude-standard`: identical to the long-standing untracked set — untouched.
- Constants confirmed unchanged: `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `SCHEMA_VERSION=2`, `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`.
- Re-ran `git status -sb`/`git diff --stat` after all browser-driven testing; output was byte-identical to the pre-audit snapshot.

## 2. Files/functions reviewed

Full `git diff index.html` (202/22 lines). Direct reads of: `tabletopAccessoryProductRegistry`, `designDefaults`/`normalizeDesignDraft`'s new `tabletop-rectangular-shell-prototype` branch, `buildTabletopRectangularShellDesignResult`, `buildTabletopRectangularShellFinishedViewSvg`, `tabletopJointFitCompatible`, `tabletopRectangularShellEvidenceMatches`, `tabletopShellRecommendationHtml`, `applyTabletopShellEvidenceMatch`, the new `designForm.oninput` override-detection branch, `projectDraftFromDesign`'s new shell branch, and the three new fixture functions (`runTabletopRectangularShellFixtures`, `runTabletopRectangularShellEvidenceFixtures`, `runTabletopLayoutFixtures`). Cross-checked against `buildBoxModel`/`buildFingerPattern`'s existing panel/edge/phase definitions (re-derived from the prior T2A architecture review) to independently verify geometric correctness rather than trust either report.

## 3. Locked-contract comparison

| Locked decision | Classification | Evidence |
| --- | --- | --- |
| Template name/ID | Implemented exactly | Registry entry confirmed; live-tested |
| 80×60×25 mm interior default | Implemented exactly | Live-confirmed |
| Interior-primary, no silent shrink | Implemented exactly | Live-confirmed finished cavity = requested interior |
| Outer envelope = interior + 2t | Implemented exactly | Live-confirmed 85.76×65.76×27.88 mm |
| Five panels FLOOR/FRONT/BACK/LEFT/RIGHT, fixed order | Implemented exactly | Order-check in source; live-confirmed |
| Reuse `buildBoxModel` unmodified | Implemented exactly | No diff lines touch `buildBoxModel`/`buildFingerPattern` |
| One shared signed clearance for corners + floor | Implemented exactly | Single `v.clearance` value passed once into `buildBoxModel` |
| Kerf separate, informational | Implemented exactly | `kerfReference` field only, never subtracted |
| Evidence-compatibility model (Option B, separate helper) | Implemented exactly | `tabletopJointFitCompatible` is new and separate; `tabletopCouponConstructionMatches` untouched (§11) |
| Exact-vs-compatible terminology | Implemented exactly | Verbatim card heading confirmed live |
| Machine exact requirement | Implemented exactly | Reuses `productionMachineIdentityMatches` unmodified |
| Material ranking tier | Implemented exactly | Computed outside the hard gate, in `tabletopRectangularShellEvidenceMatches` |
| Thickness tolerance tier | Implemented exactly | Reuses `tabletopCouponThicknessTier` unmodified |
| Finger-width tolerance (15%/minWeb-2 rule) | Implemented exactly | §14 |
| Generator-version exact | Implemented exactly | Hard-gated in `tabletopJointFitCompatible` |
| Explicit Apply, no auto-apply | Implemented exactly | Live-confirmed |
| Override tracked, editable | Implemented exactly | Live-confirmed via real form input event |
| No-evidence / conflicting-evidence never hidden | Implemented exactly | Live-confirmed no-evidence state; ties never averaged (source read) |
| Role-grouped layout `[['FLOOR'],['FRONT','BACK'],['LEFT','RIGHT']]` | Implemented exactly | Exact row array confirmed in source and fixture |
| No rotation | Implemented exactly | No `rotate(` anywhere in the diff; `rotation:'No rotation applied'` field |
| Project handoff, no schema change | **Partially implemented** | No schema change confirmed, but machine/material snapshot **text is never actually included** in the handoff notes (§26 — IMPORTANT) |
| No new storage/schema | Implemented exactly | No new root collection, no `SCHEMA_VERSION` change |
| Fixture groups: geometry/evidence/layout | Implemented exactly | Three groups registered, each once |
| Shell-result recording deferred (T2B) | Deferred as intended | No T2B code present |

## 4. Template registration and form

Registry entry confirmed: `{ id:'tabletop-rectangular-shell-prototype', displayName:'Tabletop Rectangular Shell Prototype', group:'Tabletop Accessories' }`, appended after the existing coupon entry — existing `tabletop-corner-floor-coupon` entry and every other registry entry untouched. Live-tested: selecting the template renders a dedicated form section with units, measured material thickness (shared `materialThicknessField`), interior width/depth (`min="40" max="200"`), usable wall height (`min="15" max="100"`), joint clearance (`min="-0.10" max="0.30"`), preferred finger width, panel spacing, kerf reference, an optional Library material profile select plus free-text label, and the assembly-labels checkbox. Cut Layout / Finished Assembled preview buttons, SVG download, and Project handoff (Start Project from Design) all confirmed present and functioning live. Defaults confirmed live: `80 × 60 × 25` mm, clearance `0` (manual, not Joe's `-0.075`), preferred finger width `9` mm — **Joe's 2.88 mm thickness and −0.075 mm clearance are correctly absent from any universal default**; they only ever appear after explicit user entry or explicit Apply. Width/depth/height ranges confirmed exactly `40–200` / `40–200` / `15–100` mm in the rendered `<input min/max>` attributes. No rotation control of any kind exists in the form. No duplicate control `name` attributes were found in the rendered form (`assemblyLabels` is shared across templates by design, matching the existing app-wide convention, not a T2A-specific duplication).

## 5. Rectangular dimension contract (representative case)

Live-generated with thickness 2.88 mm, interior 80×60×25 mm, preferred finger width 9 mm, clearance −0.075 mm (via applied evidence), labels off, spacing 15 mm:

- Requested interior: **80 × 60 × 25 mm** ✓
- Finished cavity: **80 × 60 × 25 mm** — identical to requested interior, no silent shrink ✓
- Outer envelope: **85.76 × 65.76 × 27.88 mm** ✓ (exactly `80+2×2.88`, `60+2×2.88`, `25+2.88`)
- Floor relation and wall-height meaning read truthfully in the summary: "Requested interior," "Finished cavity," and "Outer envelope" are three distinct, separately labeled lines, never conflated.
- Exactly five pieces; "Physical pieces" metric literally reads `5`.
- No lid, groove, rebate, channel, pocket, dado, divider, insert, bevel, magnet, or hardware appears anywhere in the summary, Help text, or SVG — confirmed by direct text/DOM inspection and by source read (`buildTabletopRectangularShellDesignResult` calls `buildBoxModel` with `lid:'open', cornerDecoration:'none'` only).

## 6. Five-panel topology

Confirmed live and in source: exactly one `FLOOR`, one each of `FRONT`/`BACK`/`LEFT`/`RIGHT`. Panel order check in source (`panels.map(id).join('|')!=='FLOOR|FRONT|BACK|LEFT|RIGHT'` → error) is a genuine generation-time guard, not just documentation, and is fixture-asserted and live-confirmed. All panels are literally the `buildBoxModel` output with only `id`/`name` remapped via a lookup table (`roleMap`) — no independently approximated geometry exists anywhere; panel `points`/`path`/`bounds` are the exact objects `buildBoxModel` produced. `designPanelGeometryErrors` (the same existing geometry-integrity check used everywhere else) is run against every panel, so closed-path and non-zero-area guarantees are inherited, not re-implemented. `buildBoxModel` and `buildFingerPattern` have **zero diff lines** — confirmed unchanged. Legacy Finger Box continues to call its own existing builder path (no diff lines touch it).

## 7. Finger patterns (independently computed and live-confirmed)

Analytically recomputed from `buildFingerPattern`'s formula ahead of testing, then confirmed by direct live generation (not merely trusting the implementation report):

| Axis | Predicted (architecture) | **Live-measured (this audit)** |
| --- | --- | --- |
| Width-axis floor | count 9, ≈9.53 mm | **9 segments × 9.528889 mm** |
| Depth-axis floor | count 7, ≈9.39 mm | **7 segments × 9.394286 mm** |
| Vertical/corner | count 3, ≈9.29 mm | **3 segments × 9.293333 mm** |
| Manufacturing minimum | 5.76 mm / 2.88 mm web | **5.76 mm segment / 2.88 mm web** |

All three pattern families are independently disclosed as three separate result-summary lines ("Width-axis floor fingers," "Depth-axis floor fingers," "Vertical/corner fingers") — **not** flattened into one metric, directly satisfying the audit's explicit high-priority concern. Clearance applied uniformly: confirmed a single `v.clearance` value flows into the one `buildBoxModel` call that produces all five panels, so every corner and every floor joint receives the identical signed clearance.

## 8. Wall-corner mating checks

Re-derived `buildBoxModel`'s exact edge/phase arrays to verify the four `wallCorners` comparisons are not just broad metadata: `Front`/`Back` both have edges `[plain, vertical(phase:true), width(phase:false), vertical(phase:true)]`, and `Left`/`Right` both have edges `[plain, vertical(phase:false), depth(phase:true), vertical(phase:false)]`. Because Front's own two vertical edges (`edges[1]` and `edges[3]`) are **structurally identical** (same phase, same shared `verticalPattern` object reference), and likewise for Back and for Left/Right, the code's choice to compare `front.edges[1]`/`left.edges[1]`, `front.edges[1]`/`right.edges[1]`, `back.edges[1]`/`left.edges[1]`, and `back.edges[1]`/`right.edges[1]` is **not** a shortcut that skips real edges — every vertical edge on every wall panel shares one of exactly two phase/pattern states, so these four comparisons genuinely exercise all four physical corners. `opposite` (phase inequality) and `patternCompatible` (identical count/actualWidth) both evaluate `true` for all four pairs, live-confirmed via the passing fixture assertion and independently re-derived by hand from the edge definitions. No duplicate/missing terminal segment, detached island, or open contour is possible here beyond what `buildFingerPanel`/`designPanelGeometryErrors` already guard for every panel generically. **Coverage gap (not a geometry defect):** the fixture assertion (`wallCorners.every(item=>item.opposite&&item.patternCompatible)`) is one bundled boolean check across all four corners, not four separate, failure-localizing assertions — a genuinely broken single corner (e.g., from a future edit) would show as one generic "false" rather than naming which corner failed. Classified IMPORTANT under fixture coverage (§29).

## 9. Wall-to-floor mating checks

Independently re-derived `Bottom`'s edges: `[width(phase:true), depth(phase:false), width(phase:true), depth(phase:false)]`. `FLOOR-FRONT` compares `floor.edges[0]` (width, true) against `front.edges[2]` (width, false); `FLOOR-BACK` compares `floor.edges[2]` (width, true) against `back.edges[2]` (width, false); `FLOOR-LEFT` compares `floor.edges[1]` (depth, true) against `left.edges[2]` (depth, false); `FLOOR-RIGHT` compares `floor.edges[3]` (depth, true) against `right.edges[2]` (depth, false). **This correctly selects the width-axis pattern for the front/back floor edges and the depth-axis pattern for the left/right floor edges — no cross-axis mix-up exists.** This was the audit's own stated high-priority concern (a rectangle has distinct width/depth floor patterns) and it is implemented correctly. Same bundled-assertion coverage gap as §8 applies here too (one `floorPairs.every(...)` check, not four separate ones) — IMPORTANT, tracked in §29.

## 10. Clearance and kerf

One signed clearance value (`v.clearance`) is read once and passed into the single `buildBoxModel` call producing all five panels — confirmed by source read, there is no second, independent clearance parameter anywhere in the shell builder. Live end-to-end trace of **−0.075 mm**: entered as a promoted evidence value → survived normalization (`designJointClearanceNumber`) → survived geometry generation (visible in the "Current joint clearance" metric) → survived into the result summary → survived into the Project handoff notes text (`"Clearance -0.075 mm"` before the manual-override step, then `"-0.05 mm"` after, both accurately reflected) — confirmed. Negative means interference under the existing, unmodified sign convention (unchanged from every other template). Kerf remains a separate `kerfReference` field, live-confirmed not subtracted from clearance or geometry anywhere (the outer envelope and finger widths are unaffected by the kerf field's value). Evidence Apply modifies only `tabletopShellJointClearance` in the draft (confirmed by direct source read of `applyTabletopShellEvidenceMatch`, which only spreads and overrides that one key) — no universal default changes (`designDefaults()` itself has zero diff to its shell defaults from this action).

## 11. Exact identity protection

`tabletopCouponConstructionMatches` has **zero diff lines** in this change — confirmed directly from the full diff, it is not present at all, meaning it is byte-for-byte identical to the committed T1.5 baseline. It continues to require exact `family`/`templateId`/`generatorVersion`/`wallCornerJoint`/`floorJoint`/finger count and actual width on both axes (unchanged source). T2A does not weaken, replace, bypass globally, or mutate this contract — `tabletopJointFitCompatible` is a wholly separate, newly added function (§12) that calls `tabletopCouponConstructionMatches` only to populate an informational `exactConstruction` field in its own return value, never to gate compatibility itself.

## 12. Compatibility helper

**Name:** `tabletopJointFitCompatible(evidenceIdentity, targetIdentity, evidenceMachine, targetMachine, evidenceThicknessMm, targetThicknessMm, clearanceStepMm)`. **Return shape:** `{compatible, exactConstruction, machineMatch, materialTier:'', thicknessTier, widthAxisCompatible, depthAxisCompatible, verticalCompatible, reasons[], caveats[], tolerance, widthDifference, depthDifference, verticalDifference}`. Confirmed pure by direct source read: no assignment to `state.*`, `designDraft`, any Production Setting/Evidence/raw-result object, any default, or the DOM anywhere in its body — it only reads its arguments and two other already-pure functions (`productionMachineIdentityMatches`, `tabletopCouponConstructionMatches`) and returns a new object. Exact requirements confirmed hard-gated with individual `reasons.push(...)` lines: `family==='tabletop-accessories'` (both sides), `generatorVersion` equal and non-empty, `wallCornerJoint==='finger-90'` (both sides), `floorJoint==='finger-jointed-floor'` (both sides), clearance sign convention equal, and `machineMatch` via the unmodified `productionMachineIdentityMatches`. A wrong machine, generator version, family, wall joint, or floor joint each independently pushes its own reason and forces `compatible:false` — confirmed both by source read and by the live/fixture test showing a mismatched-machine evidence entry excluded from `tabletopRectangularShellEvidenceMatches`'s results entirely.

## 13. Machine/material/thickness

**Machine:** exact `machineId` match required via the unmodified M3 `productionMachineIdentityMatches`; live-confirmed a different `machineId` produces zero matches (fixture: `'mismatched machine is rejected'`). One-sided-ID rejection and legacy-key-only fallback are inherited unchanged from the existing, already-audited M3 function — not re-implemented, so no new risk. **Material:** tier computed in `tabletopRectangularShellEvidenceMatches` (outside the hard gate): exact `materialId` → `'material-id'`; label-snapshot match → `'material-snapshot'`; otherwise `'unconfirmed'` with an explicit caveat string pushed (`'Material identity is unconfirmed; verify stock.'`) — never a hard exclusion, never an invented identity, and no automatic Library profile is ever created (confirmed: the function only reads `state.profiles`, never writes to it). **Thickness:** reuses `tabletopCouponThicknessTier` completely unmodified — strongest within one clearance step, close within three, excluded beyond, nominal thickness never substituted (the function is always called with `context?.measuredThicknessMm`/`targetThickness`, never a nominal field), and a `tier==='none'` result (including from invalid/missing thickness, since `Number.isFinite` guards fail closed) pushes an explicit reason and excludes the evidence.

## 14. Finger-width tolerance — highest-risk audit area

Directly inspected and hand-verified the formula:

```
tolerance = max(0.15 × |evidence.horizontalFloorActualWidthMm|, (target.minimumWeb ?? evidence.minimumWeb ?? 0) / 2)
widthAxisCompatible    = |evidence.horizontal − target.widthAxisFloor|  ≤ tolerance + 1e-9
depthAxisCompatible    = |evidence.horizontal − target.depthAxisFloor|  ≤ tolerance + 1e-9
verticalCompatible     = |evidence.vertical   − target.verticalCorner| ≤ verticalTolerance + 1e-9   (own 15%/minWeb-2 term, same formula, vertical basis)
```

This matches the architecture's locked rule exactly: 15% is computed from the **evidence (coupon) side's own actual width**, not the target shell's, exactly as locked ("grounded in... the coupon's own actual width"); the manufacturing-minimum floor term is included; a small `1e-9` epsilon is used only at the tolerance boundary, not as a general fuzz factor. **Both required cross-axis comparisons are present independently** — coupon floor width vs. shell width-axis floor width, and coupon floor width vs. shell depth-axis floor width — both compared against the single coupon `horizontalFloorActualWidthMm` value (correct, since the square coupon has only one horizontal/floor value to offer). Finger counts are never compared anywhere in this function — confirmed by absence of any `.count` read in `tabletopJointFitCompatible`. Requested/preferred finger width (the target-only input) is never substituted — the function only ever reads `actualWidthMm` fields, never `preferredFingerWidth`. Failing any one of the three axes independently excludes compatibility (each has its own `if (!xCompatible) reasons.push(...)` line), and `reasons` names the specific failed axis in plain English ("shell width-axis tolerance," "shell depth-axis tolerance," "shell corner tolerance" respectively). `exactConstruction` and `compatible` are returned as two clearly separate booleans, so Exact-vs-Compatible is structurally never conflated.

**Boundary testing (fixture-verified, cross-checked by hand):** a pattern-width deviation of ×1.1 (10%, inside the 15% band) passes (`'pattern width tolerance is bounded'`); a ×2 deviation (100%, far outside) is rejected (`'large pattern mismatch is rejected'`). This is not an exhaustive exactly-at-boundary/just-inside/just-outside sweep across all three axes independently — the existing fixtures test only the horizontal/floor axis at two magnitudes, not the depth or vertical axes at the boundary, and not a finger-count-mismatch-with-width-match case explicitly. **This is a real fixture-coverage gap** (IMPORTANT, tracked in §29) even though the formula itself, independently re-derived by hand against the real geometry in §7, is implemented correctly.

## 15. Synthetic Joe-equivalent evidence

Built and torn down entirely within disposable browser state, matching the real physical scenario precisely: measured thickness 2.88 mm, preferred clearance −0.075 mm, `alternateCandidateId` empty (no tighter alternate, matching the real result exactly since −0.075 is already the tightest tested value), Coupon-proven, Tested, exact synthetic `machineId`, a disposable Library material profile, matching `generatorVersion`. This was exercised **twice**: once via the existing fixture's in-memory construction (`t2a-fixture-*` IDs, all torn down inside the fixture's own try/finally state-restore block — confirmed no residue via the fixture's own `stateBefore`/`storageBefore` restore-and-compare pattern), and once via a **full, real, click-driven UI workflow** in this audit (Add Material → generate T1 coupon with C1/C2/C3 = −0.075/−0.050/−0.025 → Record Physical Results with preferred C1, no alternate, measured 2.88 mm → View Saved Results → Promote to Library evidence → complete the real promotion-review form). Both paths independently confirm: the 80×60×25 mm shell is **Compatible**, not **Exact** (the coupon's 5×9.152/3×8.293 mm pattern never equals the shell's 9×9.529/7×9.394/3×9.293 mm pattern); recommended clearance is exactly **−0.075 mm**; the card heading reads verbatim **"Compatible Coupon-proven evidence — Not an exact full-shell validation."** After the live UI test's browser profile was discarded (a fresh disposable Edge user-data directory used only for this session), no synthetic evidence persists anywhere; Joe's real profile was never opened.

## 16. Recommendation card

Live-confirmed via the full real-UI chain in §15: the card appears only once qualifying evidence exists (confirmed the no-evidence state beforehand showed no card, only "No compatible Coupon-proven evidence"); heading is a semantic `<h5>`/`<h4>` (accessible, not a styled `<div>`); recommended clearance, machine (`Genmitsu L8 20W`), material (`Plywood`), measured thickness, and tested date are all shown as labeled metric rows; the cumulative-tightness warning line ("Coupon-proven evidence covers the coupon joints only; a complete shell may become tighter") is present in every card. **View evidence**, **Apply recommendation**, **Keep current value**, and **Run a new coupon** are all present and independently clickable (`data-tabletop-shell-view-evidence`/`-apply`/`-keep`/`-run-coupon`). No-evidence state is non-blocking (shell still generates and validates normally, confirmed live). `tabletopRectangularShellEvidenceMatches` returns every compatible match with no reduction step — confirmed no averaging/auto-selection anywhere in the function body — sorted by rank then by date descending (newest first) for ties, matching the locked contract; this was not separately exercised with two live conflicting entries in this audit (relying on the existing fixture-level ranking assertions, since constructing a second conflicting Library evidence entry live would have required a second full promotion cycle for marginal additional confidence). **One gap:** the card itself does not show a labeled "Evidence stage" or "Compatibility" metric row separately from its heading text — the information is present (conveyed via the "Compatible Coupon-proven evidence" heading) but not as a discrete metric line the way machine/material/thickness are. Classified POLISH.

## 17. Apply/override behavior

Directly, live-tested end to end (not merely via the fixture's internal simulation):

1. Opened T2A with the manual default clearance `0`.
2. Confirmed the compatible-evidence card appeared without altering the field (still `0`) before any click.
3. Clicked **Apply recommendation**.
4. Field changed to exactly **`-0.075`**.
5. Confirmed only the current T2A draft's `tabletopShellJointClearance` changed (source read confirms `applyTabletopShellEvidenceMatch` touches no other key).
6. The raw `tabletopCouponResults` record, the promoted Evidence/Production Setting, `designDefaults()`, and the separately-generated T1 coupon draft were all unaffected — confirmed by source (the function has no code path touching any of them) and by the fact the T1 coupon's own regenerated result and golden remained at their exact prior values throughout this session.
7. Manually edited the field to `-0.050` via a real `input` DOM event (not a closured function call).
8. Confirmed "Clearance source" updated to **"Evidence-applied then manually overridden"** — live-observed, not merely fixture-asserted.
9. The Project handoff, generated at this exact moment, correctly showed `Clearance -0.05 mm; source: evidence-applied then manually overridden` — truthful current-state provenance, not stale "still applied" text.
10. Template switching was exercised (T2A → T1 coupon → back, and T2A → Library tab → back) with no console exceptions.
11. Ephemeral state behaved consistently with the existing Design-draft contract — clearing on template change (`tabletopShellEvidenceSession={}` on the existing `designForm.onchange` template-switch path), matching every other template's existing reset behavior.
12. **Nothing was ever marked Shell-proven** — confirmed by absence of that string anywhere in the diff outside comparison/prohibition text, and by direct inspection of every object the workflow produces (`result.metrics`, `tabletopShellEvidenceSession`, the Project handoff notes) — none contain a Shell-proven or Production-proven assignment.

No silent Apply and no false provenance were found — this was the audit's own flagged highest-risk item for BLOCKER/IMPORTANT classification, and it passed cleanly on live, real-DOM testing.

## 18. No-evidence and conflicting-evidence behavior

No-evidence state live-confirmed non-blocking: the shell still generated and validated with `clearance: 0` and a plain "No compatible Coupon-proven evidence… Run a new coupon" card. Conflicting-evidence handling was verified by source read only in this audit (not re-driven live with two entries): `tabletopRectangularShellEvidenceMatches` collects every passing match into one array with no reduction, and `tabletopShellRecommendationHtml` renders `matches.map(card).join('')` — every match becomes its own card, never averaged or collapsed into one. This matches the locked "never hide conflicting evidence, never auto-select" contract.

## 19. Run-new-coupon behavior

Source-confirmed and partially live-exercised (the T1 coupon template switch itself was performed live in §15's workflow, though via the template `<select>` rather than this specific button): `data-tabletop-shell-run-coupon`'s handler sets `designDraft.template='tabletop-corner-floor-coupon'`, resets `designPreviewMode='cut-layout'`, and clears `tabletopShellEvidenceSession={}` — an explicit switch, no automatic result generation or save, no T1 geometry alteration (zero diff lines touch the coupon builder), and no silent carry-over of shell values into the coupon draft (the coupon builder only reads its own `tabletopCoupon*`-prefixed fields, ignoring any stale `tabletopShell*` fields left in the shared flat draft object — the same pre-existing, already-safe pattern every other template pairing already relies on). It is a plain `<button type="button">`, standard-focusable and keyboard-activatable with no non-standard tabindex or event-suppression.

## 20. Role-grouped layout

Confirmed in source: `const rows=[['FLOOR'],['FRONT','BACK'],['LEFT','RIGHT']]` — **exactly** the locked row definition, no deviation. `layoutDesignPanelRows(panels, rows, 10, v.panelSpacing)` is called with **zero modification** to that shared function (no diff lines touch it) — confirmed no general nesting engine was added, no scaling occurs (plain millimeter pass-through), and no `rotate(` string appears anywhere in the new code (`grep`-level confirmed absent from the diff). Deterministic placement and stable panel order are inherited from the unmodified packer. Live-confirmed spacing/no-overlap indirectly via the passing `svg validates and has finite output`/geometry fixtures and the exact, reproducible golden (§23). **Reported bounds of 206.52 × 171.52 mm were independently reproduced** via live generation with the exact representative inputs (§23) — matches exactly, no undocumented difference. Panel geometry is translation-equivalent before/after placement (`layoutDesignPanelRows` only ever adds an `x`/`y` translation to each panel's existing points, confirmed by its own unmodified source). Labels move with panels — fixture-confirmed (`'labels do not alter panel placement'` compares panel `x`/`y` with and without labels enabled, unchanged) and live-confirmed (toggling labels on left panel placement/bounds unaffected while `labelCount` became `5`). Label readability was not independently re-measured pixel-by-pixel in this audit, but label paths are generated from `buildAssemblyLabelPaths` (the same unmodified, already-audited helper) against the same placed panels. **Materially compact:** grouping FLOOR alone, FRONT+BACK together (identical dimensions, pack directly side by side with no wasted rotation), and LEFT+RIGHT together achieves a tighter layout than the generic default `layoutDesignPanels` grouping would for this shape, since it avoids ever placing a wide floor panel in the same row as a narrow tall wall panel (the exact waste Joe reported) — confirmed structurally sound without requiring a general nesting optimizer, matching the architecture's own reasoning.

## 21. Rotation enforcement

**Confirmed: no rotation anywhere.** `grep`-level check of the full diff for `rotate(` returns zero matches. `result.metrics.rotation` is the fixed literal string `'No rotation applied'`, never computed or conditional. No rotation control exists in the form (§4). This satisfies the locked "no rotation in T2A" policy exactly.

## 22. Labels

Labels off (the reported golden's state): confirmed live `labelCount: 0`, no green group present in the generated SVG (structural absence, not merely an empty array — `labelResult.paths=[]` when `assemblyLabels` is false, so the served SVG never emits an `assembly-labels` group at all). Labels on: live-confirmed `labelCount: 5` (one per panel: FLOOR/FRONT/BACK/LEFT/RIGHT), `labels.valid===true`, panel placement identical to the labels-off case (§20). Labels correctly identify each of the five panels by their new role names (`labelSpecs=panels.map(panel=>({panelId:panel.id,text:panel.id,...}))`), so labels read `FLOOR`/`FRONT`/`BACK`/`LEFT`/`RIGHT` rather than any legacy `bottom`/`front`/`back`/`left`/`right` internal name. Red cut paths are unaffected by the labels toggle (confirmed by the fixture's own `'labels remain optional and do not change piece count'` assertion and by the shared, unmodified `serializeTabletopAccessorySvg` wrapper). Finished View IDs (`tabletop-shell-floor`/`-front`/`-back`/`-left`/`-right`) do not enter the production SVG — confirmed by the fixture's own explicit assertion (`!/tabletop-shell-/.test(result.svg)`), which passed.

## 23. Production SVG and golden

Independently regenerated the exact representative case (2.88 mm thickness, 80×60×25 mm interior, 9 mm preferred finger width, labels off, default spacing) via direct fixture execution in a fresh, disposable headless-Edge profile: **length 2181 bytes, FNV `2ef9606b`, `widthMm`/`heightMm` = `206.52` / `171.52` mm** — matches every reported figure exactly, with zero discrepancy. Confirmed exact millimeter `width`/`height`/`viewBox` via the shared, unmodified `designSvgValidation` pipeline (the fixture's own `'svg validates and has finite output'` assertion passed and is not template-specific — it is the same validator every other template uses). Five structural panels (`(svg.match(/<path /g)||[]).length===5`, fixture-confirmed), all closed (`Z` count ≥5, fixture-confirmed). No `NaN`/`Infinity` (fixture-confirmed via regex). Red cuts confirmed present via the shared `serializeTabletopAccessorySvg` wrapper (unmodified, already used and validated by T1). No blue guide layer is emitted (no guide-generation code exists in this builder). No green labels in the reported golden (labels off for this representative case, confirmed structurally absent, not merely empty). No Finished View geometry or IDs leak into the production SVG (§22). Deterministic repeated output is structurally guaranteed since the entire pipeline (`buildBoxModel`→`layoutDesignPanelRows`→`serializeTabletopAccessorySvg`) contains no randomness, timestamps, or non-deterministic ordering anywhere — confirmed by source read, and indirectly by the golden matching exactly on a completely fresh, independently-launched browser profile. No unexplained difference from the implementation report was found for this golden.

## 24. Finished Assembled View

Source-reviewed `buildTabletopRectangularShellFinishedViewSvg`: builds five positive-area schematic polygons (floor + front + right + back + left) via the shared, unmodified `tabletopAccessoryProjectPoint` helper, with an explicit finite-coordinate guard (`if([floor,front,right,back,left].flat().some(point=>!point||!Number.isFinite...)) return '';`) that fails closed to an empty string rather than emitting malformed SVG. Open top confirmed (no top face polygon is drawn). Relative width/depth/height are read directly from `result.finishedView.dimensions` (the shell's own real outer envelope), so the schematic's proportions are not independently guessed. Deterministic face ordering (floor, back, left, right, front — a fixed literal sequence in the template string, not computed/sorted at runtime). No groove, rebate, channel, lid, divider, insert, bevel, magnet, hinge, or hardware is drawn — confirmed, the function draws exactly five flat polygons and two text labels, nothing else. Finished View IDs (`tabletop-shell-*`) are confirmed absent from the production SVG (§22) — the two SVGs are structurally and textually disjoint. The view carries an accessible `role="img"` and `aria-label="Screen-only Finished Assembled view of Tabletop Rectangular Shell Prototype"` plus a `<title>`/`<desc>` pair, matching the coupon's own established accessible-schematic pattern. Live-confirmed rendering by switching the preview selector to "Finished Assembled" for the representative case with no console exceptions.

## 25. Result summary

All required metrics confirmed present and correctly labeled in the live-rendered summary (§5/§7): requested interior, finished cavity, outer envelope, measured material thickness, current clearance, clearance source (including the override-aware label), evidence stage, compatibility level, preferred finger width target, width-axis floor count/width, depth-axis floor count/width, vertical count/width, manufacturing minimum, physical pieces (`5`), sheet dimensions, layout strategy (`Role grouped`), rotation (`No rotation applied`), and assembly-label count. The existing exact-scale notice and a physical-prototype-specific assembly-order message are both present. No `NaN`, `undefined`, hidden shrink, or "Shell-proven"/"Production-proven"/sellable wording was found anywhere in the live-rendered summary text across all tested states (default, evidence-applied, manually-overridden).

## 26. Project handoff

**No Project schema change confirmed** — the diff only adds one more `else if` branch inside the existing `projectDraftFromDesign` function; no new field is added to the Project object shape itself. Live-generated Project draft for the representative, evidence-applied-then-overridden case correctly included: template name (`Tabletop Rectangular Shell Prototype`), requested interior/finished cavity/outer envelope dimensions, selected clearance (`-0.05 mm`, correctly reflecting the live override), clearance source (`evidence-applied then manually overridden`), compatibility level (`Compatible`), evidence stage (`Coupon-proven`), source setting/evidence IDs as plain text (`mrtl8teqt8ui1q` / `mrtl8terg1uifd`), applied preferred candidate (`C1`), measured thickness (`2.88 mm`), layout strategy (`role grouped`), rotation (`no rotation applied`), and the prototype-only caveat with an explicit "not Shell-proven, Production-proven, production-ready, or a sellable product" sentence. No live foreign-key dependency exists — the IDs are plain descriptive text, confirmed structurally (they are string-interpolated into a notes field, never stored as a reference the Project object would need to resolve later); deleting the source evidence would not make the Project unreadable. Project accounting/schema fields are otherwise completely unaffected (confirmed: the new branch only sets `name` and calls `addNote(...)`, both pre-existing mechanisms).

**IMPORTANT finding, confirmed live:** the handoff notes text literally reads `"...machine/material snapshots are descriptive only."` — this is a **generic disclaimer sentence, not the actual machine or material snapshot data**. At no point in the live-generated Project draft did the notes mention `Genmitsu L8 20W` (the machine actually used) or `Plywood` (the material actually used), even though this exact information was available on `result.metrics.materialSnapshot` and was known throughout the session. The architecture explicitly required "machine snapshot; material snapshot" as part of the preserved provenance; the implementation preserves a placeholder statement that such snapshots exist "descriptively" but never actually writes the machine or material identity into the visible text a user would read later. This is a genuine Project-provenance omission, not merely an inaccurate description — confirmed by direct live reproduction, not just source inspection.

## 27. Validation

Boundary behavior confirmed by direct source read (each condition maps to an existing, generic, already-shared validator — `dimension()`'s own inline range check, `buildFingerPattern`'s own null-return path, `tabletopAccessoryDimensions`'s own interior-primary guard, and `layoutDesignPanelRows`'s own overlap/bounds checks): width/height below or above their `min`/`max` attributes are blocked by the shared `dimension()` helper's own `value < min || value > max` check (confirmed present for `tabletopShellInteriorWidth`/`Depth`/`WallHeight`); width/height exactly at `40`/`200`/`15`/`100` are accepted (inclusive bounds, `<`/`>` not `<=`/`>=`); invalid thickness is blocked via the shared `tabletopAccessoryManufacturingLimits(v.materialThickness).valid` check; invalid panel spacing is blocked via an explicit `v.panelSpacing <= 0` check; preferred finger width below the manufacturing minimum is blocked via an explicit comparison against `limits.minSegment`; impossible width/depth/vertical finger patterns are blocked by `buildFingerPattern`'s own existing `maxOdd < 3` early return (unchanged, inherited); invalid clearance is blocked via an explicit `-0.1`/`0.3` range check; non-finite values anywhere in these paths are caught because every `dimension()` call itself requires `Number.isFinite`; silent cavity shrink remains blocked by the unmodified, reused `tabletopAccessoryDimensions` interior-primary guard; malformed paths, AABB overlap, cut-path overlap, and layout-bounds mismatch are all caught by the unmodified, reused `layoutDesignPanelRows`/`designPanelGeometryErrors` machinery. Evidence exclusions (wrong machine/generator/joint-system/missing thickness) are correctly implemented as **exclusions from the recommendation set**, not generation blocks — the shell still generates and validates normally regardless of evidence state, confirmed live in §18. Warnings vs. blocks vs. informational caveats are correctly differentiated in the returned data shape (`errors` vs. `warnings` vs. `reasons`/`caveats`), matching the architecture's three-tier validation model.

## 28. Help and accessibility

Help text (`tabletopHelpSection`, live-read) covers: five-piece topology (`FLOOR, FRONT, BACK, LEFT, RIGHT`), interior-primary dimensions, Compatible-vs-Exact framing ("Compatible is not Exact and is not full-shell validation"), explicit Apply, manual override ("If you edit the clearance afterward, it is marked manually overridden"), Coupon-proven-only limitation, cumulative tightness ("multiple joints engage simultaneously" concept implicit in the dry-fit guidance), role-grouped layout, no automatic rotation, exact-scale import, the explicit assembly order **FLOOR → FRONT → LEFT → RIGHT → BACK**, the −0.050 mm fallback trigger, an explicit "not Shell-proven, Production-proven" statement, focus/framing/ventilation/fire-watch/suppression/never-unattended language (reused from the existing safety paragraph style). Grain/veneer/readability rationale for the no-rotation decision specifically is **not** spelled out in the Help text itself (the Help text states rotation doesn't happen but does not explain why) — a minor documentation completeness gap, POLISH. Accessibility: the recommendation card uses a semantic heading element (§16); Apply/Keep/View/Run are four visually and textually distinct button labels, not icon-only or ambiguous; compatibility status is conveyed through text ("Compatible Coupon-proven evidence"), not color alone; the Finished View carries `role="img"`/`aria-label`/`<title>`; the shell form reuses the existing, already-audited centralized modal/form accessibility conventions (no new bespoke modal was introduced by T2A). No duplicate control IDs were found in the rendered form. Responsive behavior was not independently re-tested at a narrow viewport in this audit (relying on the fact that T2A reuses the exact same `.design-preview-layout`/`.metrics`/`.panel` CSS classes and form-grid structure every other template already uses, which has been previously audited for responsive behavior).

## 29. Fixture inventory and gaps

Reported counts independently reproduced exactly: shell geometry **22/0**, shell evidence **15/0**, layout **6/0**. These do cover real, direct assertions rather than only broad happy paths (e.g., exact panel order, exact outer-envelope numbers, exact golden bytes, generator-mismatch rejection, machine-mismatch rejection, pattern-tolerance boundary at two magnitudes, no-auto-apply, applied-clearance-visible-in-draft, prototype-only handoff wording) — but the counts are smaller than the requested assertion inventory specifically because several required checks are **bundled** rather than **separated**, which the audit explicitly warned against treating as equivalent. Confirmed missing high-risk **direct, failure-localizing** assertions:

- All four wall corners are checked as one bundled `.every(...)` boolean, not four independent named assertions (§8).
- All four wall-floor pairs are checked as one bundled `.every(...)` boolean, not four independent named assertions (§9).
- Width-axis vs. depth-axis pattern distinction is implicitly correct (§9) but has no assertion that would fail if a future edit swapped which floor edge maps to which axis.
- Tolerance boundary is tested at only two magnitudes (×1.1 pass, ×2 fail) on one axis (horizontal/floor); the depth axis, the vertical axis, and true exactly-at-boundary / just-inside / just-outside values are not independently asserted (§14).
- A finger-count-mismatch-with-width-match case and a width-match-with-count-mismatch case are not directly asserted (the architecture's own "count may differ safely" rule is exercised only implicitly, via the shell's own naturally-different counts, not via a deliberate targeted case).
- Explicit-Apply immutability of the raw result/Evidence/Production Setting is not directly asserted in the T2A evidence fixture itself (it relies on the T1.5 promotion fixture's own separate coverage of that same underlying invariant).
- Manual-override provenance transition (evidence-applied → then further manually edited → label changes) is not fixture-asserted at all — only live-confirmed in this audit (§17); this is a real, currently-uncovered regression risk.
- Tie/conflict handling (two compatible entries shown together, ranked, unaveraged) is not exercised by any fixture — only reasoned about from source (§18).
- Project handoff provenance is checked only for the presence of the template name and the prototype caveat regex, not for the machine/material snapshot omission this audit found live (§26) — the existing fixture assertion would not have caught that gap.
- Finished View parsed geometry (actual polygon point values, not just "contains the string 'Screen-only Finished Assembled'") is not independently asserted.

Every one of these is classified IMPORTANT (missing high-risk fixture coverage) rather than BLOCKER, because in each case this audit independently re-derived or live-reproduced the correct underlying behavior by other means — the gap is in regression protection, not in current correctness.

## 30. Exact focused totals

Independently reproduced on a fresh, disposable headless-Edge profile, calling each exposed group function directly:

| Group | Passed | Failed |
| --- | ---: | ---: |
| Rectangular shell | 22 | 0 |
| Shell evidence | 15 | 0 |
| Role layout | 6 | 0 |
| Designs geometry | 1093 | 0 |

All four exactly match the reported figures.

## 31. Unique flat-suite total and group count

Counted the actual `if (selftest === '<name>' || selftest === 'all') run...()` registration block directly in source: **37 unique top-level groups** are registered exactly once each under `?selftest=all` (the prior 34 plus the three new T2A groups: `tabletop-rectangular-shell`, `tabletop-rectangular-shell-evidence`, `tabletop-layout`). Invoking each of the 37 exposed functions exactly once, summing passed/failed directly (not relying on any console total), on a completely fresh disposable browser profile: **`2685 passed / 0 failed` across 37 groups** — reproduced twice for consistency (same profile session, re-invoked), identical both times. This reconciles exactly with the prior audited unique baseline: `2642` (prior 34-group unique total) `+ 22 + 15 + 6 = 2685`. **This is the correct number directly comparable to the prior `2642` total** — the task's own instruction to independently determine this rather than trust the nested `4487` figure was well-founded, since `4487` is not derivable by simple addition from the reported per-group figures and is not comparable to `2642` at all.

## 32. Nested invocation total, separately

The reported `4487 passed / 0 failed across 60 console fixture invocations` was not independently re-derived assertion-by-assertion in this audit (doing so would require tracing every internal regression-guard call across all 37 groups, an exercise with low incremental value once the unique flat total is independently confirmed correct). It plausibly arises from fixture groups that internally re-invoke other groups as regression guards (a pattern already established in T1.5, e.g., new Tabletop groups calling `runMultiMachineStorageFixtures`/`runEvidencePromotionFixtures`/`runTabletopCornerFloorCouponFixtures` internally), which would inflate both the invocation count (60 vs. 37 unique) and the summed assertion count (4487 vs. 2685 unique) by counting shared assertions more than once. **This nested figure is not, and should not be presented as, a substitute for the unique flat-suite total** — README and CHANGELOG both already carry an explicit caveat to this effect ("nested regression fixture calls make that count non-comparable to older flat-suite summaries"), which is honest, but neither document states the actual comparable number. That is a genuine documentation gap this audit resolves: **the comparable unique total is 2685, not 4487, and not left unstated.**

## 33. Existing output comparisons

Re-confirmed via the independently-run 37-group total (§31), since every one of these groups' internal byte/hash assertions is embedded in the group's own pass/fail count: T1 coupon **2992 bytes / FNV `4f543f95`** (embedded in the 76/0 `runTabletopCornerFloorCouponFixtures` result); Dice Tray **1726 / `51a55721`**, alternate Dice Tray **1054 / `41697123`**, Divider Tray **1965 / `a55dda6e`** (embedded in the 92/0 `runDiceTraySystemFixtures` result); Joint Fit Coupon **7764 / `db7ea7e9`** (embedded in existing coupon fixture groups, all passing). Finger Box, Gift Box (69/0), Sliding-lid Box, Drawer Cabinet, and the wall/base tab coupon show no unexplained drift — all their containing groups report `0` failed. `tabletopCouponConstructionMatches` is unchanged (§11). T1.5 storage and evidence fixtures remain green (`runTabletopCouponResultsFixtures` 33/0, `runTabletopEvidencePromotionFixtures` 22/0, `runTabletopEvidenceMatchingFixtures` 21/0 — all matching the committed baseline exactly). No existing golden was updated to accept drift — confirmed by the diff itself containing zero modifications to any pre-existing fixture assertion's expected byte/hash value.

## 34. Direct file:// results

`git diff --check`: clean (CRLF-only). `python -m html.parser index.html`: exit code 0, no output. Live `file://` testing in headless Edge (fresh disposable profile) confirmed: clean startup with no console exceptions across the entire session; Designs opens normally; the T2A form renders with correct defaults; a valid representative shell generates correctly; Cut Layout and Finished Assembled previews both render; Role-grouped layout, labels off/on, the evidence recommendation card (both no-evidence and compatible-evidence states), explicit Apply, manual override, View evidence (opens the Library tab), Keep current value, and Run a new coupon were all exercised via real DOM events, not simulated function calls; the no-evidence state was directly observed; Project handoff was generated and inspected in full; switching among T2A, T1 coupon, and the Library tab produced no exceptions; no network dependency exists (`file://` load throughout); no storage/schema migration occurred (confirmed no `SCHEMA_VERSION`-related code executed differently). Keyboard-accessibility and narrow-viewport responsive behavior were assessed by source/structure inspection rather than live pixel-level testing (§28).

## 35. Protected boundaries

Confirmed unchanged by direct diff inspection: `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`; recovery; legacy imports/backups (no diff lines touch any of this machinery); machines M1/M2/M3 and machine snapshots (only called, never modified); Material Tests; Test Grids; Library profile schema (only read); existing Production Settings and existing Evidence (both only additively read via `tabletopRectangularShellEvidenceMatches`, and the one live promotion performed in this audit used the pre-existing, unmodified promotion pipeline exactly as T1.5 already established); T1.5 raw coupon results (only read, never written by T2A code); promotion history; Project schema (§26, no field added); accounting; Inventory; Pricing; legacy Designs; T1 coupon geometry and SVG (zero diff lines, golden reconfirmed unchanged); kerf conventions; LightBurn colors; filenames; offline/no-network behavior (§34).

## 36. Findings by severity

**BLOCKER:** none. Five-piece geometry, all four corner and floor mating pairs, the finger-width compatibility formula, explicit Apply/override behavior, Role-grouped placement, no-rotation enforcement, and the production golden were all independently verified correct — no malformed geometry, non-mating joint, silent evidence application, wrong-machine recommendation, incompatible-evidence-treated-as-compatible, malformed SVG, broken scale, overlap, or protected-output drift was found anywhere.

**IMPORTANT:**
1. Project handoff omits the actual machine and material snapshot identifying text, substituting a generic "descriptive only" disclaimer instead of the real values already available on `result.metrics` — confirmed by live reproduction (§26).
2. The four wall-corner and four wall-floor mating checks are each collapsed into one bundled `.every(...)` fixture assertion rather than four independent, failure-localizing assertions (§8, §9, §29) — the underlying geometry is correct, but a future regression in one specific corner would not be individually pinpointed.
3. The finger-width tolerance boundary is fixture-tested at only two magnitudes on one axis; the depth axis, vertical axis, and true exact-boundary values are untested (§14, §29).
4. The manual-override provenance transition (Apply → further manual edit → label change) has no fixture coverage at all, only this audit's live confirmation (§17, §29).
5. Tie/conflicting-evidence display has no fixture coverage (§18, §29).
6. README/CHANGELOG document the non-comparable nested `4487` total with an honest caveat but never state the actual comparable unique total; this audit supplies it (§31, §32).

**POLISH:**
1. The recommendation card does not show "Evidence stage"/"Compatibility" as discrete labeled metric rows separate from its heading text (§16).
2. "Keep current value" records no session state at all, so it has no visible effect on the displayed clearance-source text (§17, noted in passing).
3. Help text does not explain the grain/veneer/readability rationale behind the no-rotation decision, only that rotation does not happen (§28).

**NOT A DEFECT (intentionally deferred or excluded):** T2B shell-result recording; Hex product; premium/sellable product features; rotation toggle; general-purpose nesting; any Shell-proven/Production-proven confidence prior to physical testing; exploded Finished View mode.

## 37. Exact final verdict

**APPROVED FOR CONTROLLED PHYSICAL SHELL CUT**

Justification: zero BLOCKER findings across every high-priority area this audit was asked to scrutinize — five-piece geometry and complete wall/floor mating (independently re-derived by hand from the real edge/phase definitions, not accepted from metadata strings alone), the exact-vs-compatible evidence boundary (source-verified and live-reproduced with Joe's real physical values end to end), the 15%/minimum-web finger-width rule (independently re-derived and matched against the real computed geometry), explicit Apply/override behavior (live-tested via real DOM events, not simulated), Role-grouped placement and no-rotation enforcement (both confirmed structurally sound and reproducibly deterministic), and existing production-output protection (all legacy goldens reconfirmed byte-for-byte unchanged). The six IMPORTANT findings are real and should be corrected, but none of them affect whether the geometry Joe would actually cut is trustworthy — they concern documentation completeness, Project-note content, and fixture regression coverage, not the physical correctness of the shell itself.

## 38. Whether T2A may be committed

Yes, though the Project-handoff machine/material omission (§26/§36-1) is cheap to fix (it is a one-line text-generation change, not a geometry or evidence-model change) and would ideally be corrected in the same commit or as an immediate fast-follow, alongside splitting the two bundled mating-check fixture assertions into eight named ones and adding the missing tolerance-boundary/override-transition/conflict-display fixture coverage.

## 39. Whether Joe may perform the controlled shell cut

**Yes.** The exact physical scenario (2.88 mm measured plywood, −0.075 mm evidence-backed clearance, 80×60×25 mm interior) was independently reproduced end to end in this audit via the real UI, and the recommendation, Apply, and result-summary/Project-handoff provenance were all confirmed truthful for that scenario. This verdict concerns exactly one controlled exploratory shell cut, not Shell-proven or sellable-product status.

## 40. Whether T2B remains deferred

Yes, confirmed by the complete absence of any shell-result-recording code, new record type, or new `sourceType`/`evidenceStage` value beyond the already-existing `'coupon-proven'` in this diff.

## 41. Whether Grok is needed

Yes, as the standard second-pass adversarial check before Joe commits to the physical cut, focused specifically on: (a) independently re-deriving the wall-corner and wall-floor edge/phase mating this audit hand-verified (§8/§9), since that reasoning is subtle and benefits from a second independent derivation; and (b) the Project-handoff provenance gap (§26), to confirm whether the missing machine/material text is best fixed before or after this initial controlled cut.

## 42. Confirmation

No product source file, README, CHANGELOG, fixture, storage, application record, or existing report was edited during this audit. Nothing was staged, committed, or pushed. All browser-driven verification — including the full live Add-Material → generate-coupon → record-result → promote-to-evidence → apply-to-shell → override → Project-handoff workflow — ran against a disposable, isolated headless-Edge profile directory outside the repository and was discarded at the end of the session; Joe's real browser profile and production `localStorage` were never opened or altered. Only this report file was written.
