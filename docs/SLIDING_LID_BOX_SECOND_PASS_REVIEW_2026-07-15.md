# Sliding-Lid Box Phase 3 — Second-Pass Review (Independent Check of Grok's Audit)

**Reviews:** `docs/SLIDING_LID_BOX_INDEPENDENT_AUDIT_2026-07-15.md` (Grok's audit, including a live Edge-headless 797/0 run)
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `0a304d8` — *Add parametric finger-box generator*
**Reviewed:** Uncommitted changes in `index.html`, `README.md`
**Date:** 2026-07-15
**Mode:** Read-only. No browser/Node runtime available here, so the live 797/0 run could not be reproduced. Every geometric claim was verified by reading the actual generator code (`slidingLidBoxDimensions`, `buildSlidingLidBoxModel`, `buildSlidingLidRail`, `buildSlidingLidPanel`, `buildSlidingLidSidePanel`, `layoutDesignPanelRows`) and re-deriving the arithmetic by hand — not by re-running the fixtures that assert the same formulas.

---

## Verdict on Grok's audit

**Accurate.** Golden A hand-recalculation matches the production code on all 15 quantities I re-derived, the fixture count (265) recomputed exactly using loop-aware counting, storage isolation is confirmed at the strongest evidence level (zero diff hunks), and the highest-risk geometry (shortened Front, composite side edges, rail path, lid tongue coordinates) all check out against source. I add three refinements Grok's report doesn't state: the rail's upper/lower web labeling is moot *by construction* (U=L always), several insertion-path "checks" are algebraic identities that can never fail, and the finger-box layout wrapper claim is right for a more specific reason than stated. **Agree: SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP.**

---

## Repository inspection (re-run independently)

| Check | Result |
|---|---|
| HEAD | `0a304d8` — *Add parametric finger-box generator*, matches |
| Staged files | None |
| Tracked changes | `README.md`, `index.html` only |
| `git diff --check` | Passed (CRLF warnings only) |
| `git diff --numstat` | `README.md` 2/2, `index.html` **456/20** |

**Recurring diff-stat discrepancy, seventh occurrence in this review chain:** Grok reports `index.html` +476/−22; my `--numstat` on the identical tree says 456/20. As before, this affects no functional finding, but the pattern is now well-established — Grok-reported diff line counts should not be quoted verbatim.

---

## Dimensional model — independently re-derived from `slidingLidBoxDimensions` (lines 1749–1772)

Hand-calculated Golden A (usable 100×80×40, t=3, U=L=S=max(3,4)=4, Cv=Cs=Ci=0.2) directly from the code's formulas, not from either report's tables:

| Quantity | Code formula | Hand calc | Grok's table |
|---|---|---:|:---:|
| Wi | `usableWidth + 2t` | 106 | ✓ |
| Hi | `usableHeight + t + Cv + upperWeb` | 47.2 | ✓ |
| Ho | `insideHeight + t` | 50.2 | ✓ |
| Ch | `t + Cv` | 3.2 | ✓ |
| Rh | `U + Ch + L` | 11.2 | ✓ |
| Rc | `insideDepth − S` | 76 | ✓ |
| G (railGap) | `Wi − 2t` | 100 | ✓ |
| Lw | `Wi − Cs` | 105.8 | ✓ |
| E | `(Lw − G)/2` | 2.9 | ✓ |
| Tw | `G − Cs` | 99.8 | ✓ |
| Hf | `t + Uh − Ci` | 42.8 | ✓ |
| lidLength | `Di + t` | 83 | ✓ |
| railBottom | `Uh − L` | 36 | ✓ |
| shoulderEnd (lid local) | `t + shoulderStop` = 3+76 | 79 | ✓ |
| tongueEnd (lid local) | `t + insideDepth` = 83 = lidLength | 83 | ✓ |

All match. The lid coordinate-origin question the task flagged is resolved in code: `shoulderStop` is stored as `channelLength` (76) and `tongueEnd` as `insideDepth` (80) in the dimensions object, then `buildSlidingLidPanel` adds `t` to both — so the extra `t` is applied exactly once, at panel-construction time, and the tongue terminates precisely at the end of the `Di + t` piece. Consistent, not double-counted.

## Shortened Front and composite sides — confirmed at source

- `frontPattern = buildFingerPattern(dimensions.frontHeight, ...)` (line 1898) — genuinely built from `Hf`, **not** cropped from `verticalPattern`. A cropped pattern would share boundary spacing with the full-height pattern; this one gets its own odd-count/width solution for the shorter length. Confirmed as claimed.
- Phase symmetry mirrors the Phase 2 proof: Front's two vertical edges are both `frontPattern, true`; both side panels' `frontEdge` is `frontPattern, false`; Back is `vertical, true` against sides' `backEdge` `vertical, false`. Same single-invariant construction that made the Phase 2 corner scheme provably correct — the lower-Front joints can't be individually wrong.
- The composite mechanism (`buildSlidingLidSidePanel` → `composite: true, compositeSpan: frontHeight`) trims the pattern to the lower `Hf` span via `designSafePatternEdgePoints`, then emits the plain run to the full corner. `designEdgeTerminalOffset` returns 0 for a composite edge's far end (line 1782), so the pattern-to-plain transition lands on the outer face — no gap/overlap at the transition by construction.

## Rail — confirmed, with one observation Grok's report misses

The 9-point path (line 1858–1860) traces a closed orthogonal C-channel: perimeter, then channel notch from `x=0` (open Front) to `stopStart=Rc` (integral stop of exactly `S = 80−76 = 4`). The two collinear left-edge segments (y 0→`channelTop` and `channelBottom`→height) don't overlap — the gap between them *is* the channel mouth — so no self-intersection or duplicate segment. All as Grok describes.

**Observation:** Grok's table labels y 0→4 as the *lower* web and y 7.2→11.2 as the *upper* web; in SVG y-down local coordinates the reverse reading is equally defensible. This is unfalsifiable and harmless **because `upperWeb` and `lowerWeb` are both `Math.max(t, 4)` — identical by construction, always** (line 1750). The rail is vertically symmetric, so glue-up orientation can't be gotten wrong vertically. Worth documenting though: if a future phase ever lets U≠L, the rail's local-coordinate orientation becomes load-bearing and is currently implicit.

## Clearance semantics — confirmed

- `Cs` total across both sides: `Lw = Wi − Cs`, `Tw = G − Cs`, so tongue-to-rail-gap slack = `G − Tw = Cs` (0.1/side), and `E = t − Cs/2` = 2.9 ✓. Not doubled or halved.
- `Cv` added once: `Ch = t + Cv` ✓. `Ci` affects only `frontHeight` ✓ (single use, line 1763). `Cj` (`parameters.clearance`) reaches only `designPatternEdge` on body fingers ✓.
- Blocking rules match spec: `minimumEngagement = max(1, t/3)`, `channelLength > max(3t, 12)`, outside width `> 4t`, `railBottom ≥ max(t,4)`, insertion-collapse guard (`usableHeight ≤ Ci` → error) — all at lines 1881–1897, rejecting rather than clamping.

## Fixture count — 265 confirmed via loop-aware counting

Applying the lesson from the Phase 2 audit (where naive static counting under-reports loop-generated fixtures): 203 static `add(` sites, of which 8 sit inside loops — 5 in the legacy-baselines loop (×4 templates), 1 in the Golden A key loop (×21 keys), 1 in Golden B (×21), 1 in the mating-pairs loop (×8 pairs). Runtime total: 195 + 20 + 21 + 21 + 8 = **265**, exactly matching Grok's claim. The mating-pair labels enumerate all 8 body joints including both shortened-Front lower fields. Grand total: 20+12+23+67+57+56+61+216+12+8+265 = **797** ✓ (per-group figures for non-Designs groups taken from prior verified passes).

## Legacy stability and storage isolation — strongest-evidence confirmations

- Finger-box signatures `2559/4e2a6f4b` (open) and `2691/c202cef2` (loose) confirmed present in fixture source (line 2211), alongside the four unchanged Phase 2 legacy hashes.
- `git diff | grep` for `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState`, `persist(`, `backupObject`, `buildBoxModel(` — **zero hunks touch any of them**.
- Panel ID order fixture asserts `bottom|sliding-lid|front-open|back|left|right|rail-left|rail-right` and the four-row layout array at line 2034 matches the required rows exactly.

## Refinement to Grok's L2 (finger-box nominal envelopes)

Grok's L2 says the finger-box layout still uses nominal envelopes while sliding uses path AABBs. True, but the *mechanism* is worth precise statement: `layoutDesignPanels` was **replaced** by a one-line wrapper over the new shared `layoutDesignPanelRows`, which applies the path-AABB gap check (line 1949) to *every* caller — finger-box included. The finger-box still effectively gets nominal-envelope checking only because `buildFingerPanel` hardcodes `bounds = {0,0,width,height}` (nominal), whereas sliding panels use `designPanelFromPoints`, which computes true bounds from actual points. So the gap isn't in the layout engine anymore — it's in the Phase 2 panel builder's hardcoded bounds. A future fix is one change in `buildFingerPanel`, not layout work. The finger-box byte-stability fixtures passing confirms the wrapper rewrite changed no output.

## Fixture-quality note beyond Grok's M2

Some insertion-path table rows in Grok's §7 (and the corresponding fixtures) are **algebraic identities** that cannot fail for any input: `Hf − t = Uh − Ci` is true by definition of `Hf`, and `Rc + S = Di` by definition of `Rc`. These validate formula transcription, not geometry — consistent with M2's spirit, but worth naming: the genuinely falsifiable insertion checks are `Tw < G` (depends on Cs > 0) and the rail-channel-open path shape, both of which are also covered.

---

## Findings

No new findings beyond Grok's. M1 (3D fold unproven), M2 (helper-mirroring fixtures), L1 (no forced-download DOM test), L2 (finger-box nominal bounds — mechanism refined above), I1–I4 all confirmed consistent with source.

## Unverified areas

Live 797/0 and 265/0 runs, LightBurn import, physical scrap assembly, interactive UI/keyboard/viewport behavior — same constraints Grok disclosed; static structure and arithmetic fully support the claimed totals.

---

## Required conclusion

```text
SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP
```

*Second-pass review performed read-only at commit `0a304d8` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. Unrelated untracked files were left untouched.*
