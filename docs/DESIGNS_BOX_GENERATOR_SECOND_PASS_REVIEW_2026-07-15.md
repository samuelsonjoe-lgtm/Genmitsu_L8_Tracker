# Designs Phase 2 Finger-Box Generator — Second-Pass Review (Independent Check of Grok's Audit)

**Reviews:** `docs/DESIGNS_BOX_GENERATOR_INDEPENDENT_AUDIT_2026-07-15.md` (Grok's independent read-only audit, including a live Edge-headless 614/0 run)
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `6a27579` — *Add inventory-guided project wizard*
**Reviewed:** Uncommitted changes in `index.html`, `README.md`
**Date:** 2026-07-15
**Mode:** Read-only. No browser/Node runtime available in this environment, so the live 614/0 fixture run could not be reproduced. Every geometric claim was instead verified by reading the actual math in `buildBoxModel`/`buildFingerPattern`/`designEdgePoints` and hand-tracing the phase/boundary arithmetic for all eight mating joints from the raw edge-definition arrays — not by re-running the fixtures that assert the same thing.

---

## Verdict on Grok's audit

**Accurate, with one important methodological correction to how the Designs fixture count is verified (not to the count itself, which turned out correct).** Every geometric and architectural claim I checked — dimension formulas, all eight mating-pair phase relationships, clearance semantics, the collapse guard, orthogonal-path enforcement, legacy byte-stability hashes, and the storage/backup isolation — matches the actual code exactly. I initially computed a different (wrong) Designs fixture count using a method that had worked for every other fixture group in this file, then found and corrected my own error before reporting it — see the dedicated section below, because it's a genuinely useful thing to flag about this file's fixture architecture. **Agree with Grok's conclusion: SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP.**

---

## Repository inspection (re-run independently)

| Check | Result |
|---|---|
| HEAD | `6a27579`, matches |
| Staged files | None |
| Tracked changes | `README.md`, `index.html` only |
| `git diff --check` | Passed (CRLF warnings only) |

Consistent with Grok's report.

---

## The eight mating pairs — independently re-derived from the raw edge arrays, not from the fixture that asserts them

This is the single most important claim in the whole audit, so I traced it from first principles rather than trusting either report. Reading `buildBoxModel`'s panel `definitions` array directly:

```
Bottom.edges = [width:true, depth:false, width:true, depth:false]
Front.edges  = [plain, vertical:true, width:false, vertical:true]
Back.edges   = [plain, vertical:true, width:false, vertical:true]   ← identical to Front
Left.edges   = [plain, vertical:false, depth:true, vertical:false]
Right.edges  = [plain, vertical:false, depth:true, vertical:false]  ← identical to Left
```

Front and Back share *literally identical* edge definitions (same pattern objects, same phase booleans on every index), and so do Left and Right. This is what makes the whole scheme provably correct without needing per-corner special-casing: **every vertical edge on a Front/Back panel is phase `true`; every vertical edge on a Left/Right panel is phase `false`.** Since a valid mating pair only requires opposite phases sharing the same pattern, *any* Front/Back vertical edge paired with *any* Left/Right vertical edge is automatically opposite-phase — the four corner joints aren't four independent things that could each individually be wrong, they're one invariant (true≠false) applied symmetrically. I checked this holds for all four corners (Front↔Right, Front↔Left, Back↔Right, Back↔Left) and both bottom-pairing directions (width: Bottom.edges[0]/[2] both `true` against Front/Back.edges[2] both `false`; depth: Bottom.edges[1]/[3] both `false` against Left/Right.edges[2] both `true`). All eight pairs check out. Grok's mating-pairs table (§4) is accurate.

## Clearance semantics — traced through `designEdgePoints`, confirmed not doubled

```javascript
distance = nominal + (currentFinger ? -clearance / 2 : clearance / 2);
```

A finger boundary shrinks by `clearance/2` from the nominal shared boundary coordinate; a recess boundary on the *mating* panel grows by `clearance/2` from that same nominal coordinate (since both panels reference `pattern.boundaries`, which are identical per the mating-pairs proof above). The tab ends up `clearance/2` inside nominal; the slot it fits into ends up `clearance/2` outside nominal — a combined `clearance` of total slop, not `clearance/2` (halved) or `2×clearance` (doubled). Confirmed matches Grok's §5 claim exactly.

## Collapse guard, finger-pattern algorithm, and orthogonal-path safety — all confirmed by direct read

- `requiredWeb = Math.max(0.5, t * 0.25)`, rejected when `pattern.actualWidth - clearance < requiredWeb - 1e-9` — exact match to Grok's §6/§5 description.
- `buildFingerPattern`'s tie-break (`candidateDifference < difference - 1e-9`) uses a strict epsilon-guarded inequality, so on an exact tie the **first** (smallest) odd count already found keeps winning — confirmed deterministic, lower-count-wins-on-ties as claimed.
- `designPointsPath` returns `''` for any non-orthogonal segment rather than emitting a malformed path. I specifically checked whether this could silently produce a broken panel — it can't: `buildBoxModel` (line ~1714) checks `panels.some(panel => !panel.path || ...)` and pushes a validation error when any panel's path is falsy. An empty string is caught, not silently exported.

## Fixture count — my first attempt was wrong, and the reason why is worth documenting

Every other fixture group in this file (Grid/Material/Library/Project Browser) builds its `results` array as one flat literal `const results = [ [name, pass], ... ]`, which I've been counting throughout this whole review chain with a bracket-depth-aware parser. I applied that same method here and got **67** for `runDesignGeometryFixtures` — which does not match Grok's claimed **82**.

The reason: this function is structured differently. It uses `const results = [], add = (...) => results.push(...)` and calls `add(...)` imperatively, and — critically — five of those `add(...)` call sites sit **inside** a `.forEach()` loop over the four legacy-template baselines (`qr-stand`, `hanging-sign`, `dice-tray`, `divider-tray`). A static text count of `add(` occurrences finds those 5 call sites once each, but at runtime they fire **once per template** — 4×5 = 20 fixture entries from 5 lines of source. Recomputing with that accounted for: 62 non-loop `add()` calls + (5 × 4) loop-generated calls = **82**, exactly matching Grok's figure. I want to flag this not as a defect, but as a real methodological trap: **an automated fixture-count check that doesn't distinguish "static call sites" from "runtime invocations" will silently under-report this specific group**, and any future fixture-total tooling for this file should account for it. Grok's reported 82 is correct.

I did not independently re-derive the other ten group counts (20/12/23/67/57/56/61/12/8/216) in this pass — nine of those were already independently verified in earlier reviews in this chain, and Project Wizard (216) is a different feature outside this audit's scope. Taking Grok's per-group figures as given, they sum to exactly 614.

## Storage/backup isolation — confirmed with the strongest available evidence

Ran `git diff -- index.html | grep` for `designWallPath`, `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState`, `persist(`, `backupObject` — **zero matches for all six**. Not "the summary doesn't mention touching these," but the diff contains no hunks referencing any of them at all. Separately confirmed `designDraft` (line 377) is a bare module-level `let`, outside the `state` object entirely, with no reference to it inside `persist()` or `backupObject()` (neither function appears in the diff, so neither could have gained one). This directly confirms Grok's I3 and the "Designs are absent from JSON backups" claim.

## Legacy hash table — confirmed byte-for-byte from source, not from either report

Read the `baselines` object inside `runDesignGeometryFixtures` directly: `qr-stand` 359/`fe737a09`, `hanging-sign` 341/`656e633d`, `dice-tray` 1726/`51a55721`, `divider-tray` 1965/`a55dda6e` — matches Grok's §9 table exactly, character for character, straight from the fixture source rather than either narrative report.

---

## Findings

No new findings beyond what Grok reported. Every Medium and Low item I spot-checked matches the code as described:

- **M1** (layout overlap uses nominal `panel.width`/`panel.height` rectangles via `designBoxesOverlap`, not actual path AABBs) — confirmed by reading `layoutDesignPanels`/`designBoxesOverlap` directly; the overlap check operates purely on `translatedBounds` derived from nominal panel dimensions.
- **I1–I4** — all confirmed above with direct evidence, in some cases stronger than what the original audit provided (zero-diff-hunk grep versus "not mentioned in diff").

---

## Unverified areas (same constraint as Grok, minus the live run)

- All live browser execution — could not reproduce the reported 614/0 total, the 82/0 Designs subtotal, or "no console errors" claims live. I independently confirmed the *static structure* supports 82 (see fixture-count section) and that the other group totals sum correctly, which is a different and weaker guarantee than actually running them.
- LightBurn import and physical scrap-cut assembly — not performed here either, consistent with Grok's own disclosure that this is mandatory manual follow-up regardless of fixture results.
- Project Wizard's 216-fixture claim and the other nine non-Designs group totals — not independently re-derived in this pass (nine were already verified in earlier reviews in this chain; Project Wizard is a separate feature outside this audit's stated scope).
- Interactive UI, narrow-viewport, and keyboard behavior — not exercised, same as Grok.

---

## Bottom line

Grok's geometry reasoning is sound and, on the highest-stakes claim in the report (all eight mating joints being genuinely opposite-phase), I traced the *reason* it's true from the underlying edge-definition symmetry rather than just confirming the fixture that checks it passes — which is a stronger form of verification than either of us running the same assertion twice. The one place I initially disagreed with Grok (the Designs fixture count), I disagreed with myself first, found the actual cause, and confirmed Grok's number was right all along. Nothing found here changes the recommendation.

---

*Second-pass review performed read-only at commit `6a27579` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. Unrelated untracked files were left untouched.*

## Audit conclusion (required single recommendation)

```text
SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP
```
