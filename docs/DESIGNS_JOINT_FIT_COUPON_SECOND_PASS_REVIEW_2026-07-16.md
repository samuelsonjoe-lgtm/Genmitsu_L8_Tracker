# Joint Fit Coupon — Second-Pass Review (Independent Check of Grok's Audit)

**Reviews:** `docs/DESIGNS_JOINT_FIT_COUPON_INDEPENDENT_AUDIT_2026-07-16.md` (Grok's audit)
**Repository:** `C:\Genmitsu L8 Tracker`
**Baseline commit:** `b1ba69e` — *Fix finger-panel SVG corner retracing*
**Reviewed:** Uncommitted changes in `index.html` (281/11), `README.md` (2/2)
**Date:** 2026-07-16
**Mode:** Read-only. No browser runtime here, so the live fixture run couldn't be reproduced. Every claim was checked by reading `parseJointCouponClearances`, `jointCouponLabelText`, `buildJointCouponModel`, and `buildJointCouponDesignResult` directly, hand-deriving the parser rules and the signed-clearance worked example from the raw formulas, and recomputing the fixture count with a bracket/loop-aware script — not by trusting the report's tables or re-running the fixtures that assert the same thing.

---

## Verdict on Grok's audit

**Accurate.** The signed-clearance worked example (45 mm edge, five 9 mm segments, ±0.04 mm) checks out exactly against the raw `designSafePatternEdgePoints` distance formula, not just against the fixture that asserts it. The default and signed-endpoint layout dimensions (230×106 mm, 230×74 mm) both fall out of `layoutDesignPanelRows`' margin/gap arithmetic by hand. The fixed-decimal parser's range/duplicate/normalization rules match the spec exactly, including the `-0.00`→`0.00` collapse. The coupon topology is genuinely one patterned edge + three plain edges reusing `buildFingerPanel`→`buildSlidingLidBodyPanel`, not a parallel reimplementation. `git diff` confirms none of the shared geometry primitives' signatures were touched. Fixture count (515) and grand total (1047) both recompute exactly. All four Minor findings (M1–M4) independently confirmed. No blockers, no new findings. **Agree: SAFE TO COMMIT.**

---

## Repository inspection (re-run independently)

| Check | Result |
|---|---|
| HEAD | `b1ba69e`, matches |
| Staged files | None |
| Tracked changes | `README.md` (2/2), `index.html` (281/11) |
| `git diff --check` | Passed (CRLF warnings only) |
| Shared primitives touched? | `git diff \| grep` for `function buildFingerPattern\|designSafePatternEdgePoints\|buildFingerPanel\|buildSlidingLidBodyPanel\|designPatternEdge\|designEdgeTerminalOffset` → **zero hits** |
| Persistence functions touched? | `git diff \| grep` for `STORAGE_KEY\|function freshState\|function persist(\|function backupObject` → **one hit**, and it's a new fixture line reading `STORAGE_KEY`, not a change to any persistence function |

---

## Independently confirmed

### Signed-clearance worked example — re-derived from the raw formula, not the fixture table

`designSafePatternEdgePoints` computes each internal boundary as `distance = pattern.actualWidth*(index+1) + (currentFinger ? -clearance/2 : clearance/2)`, where `currentFinger = (index%2===0)===edge.phase`. For the 45 mm/five-9mm-segment case (`actualWidth=9`, boundaries `0,9,18,27,36,45`), hand-computing all four internal transitions at `clearance=+0.04`:

- Coupon A (`phase=true`, so index 0/2 are fingers, 1/3 are recesses): `9−0.02=8.98`, `18+0.02=18.02`, `27−0.02=26.98`, `36+0.02=36.02`.
- Coupon B (`phase=false`, parity flipped): `9+0.02=9.02`, `18−0.02=17.98`, `27+0.02=27.02`, `36−0.02=35.98`.

Both match the task's worked example exactly. At `clearance=−0.04` the sign on every term flips, which by inspection swaps Coupon A's and Coupon B's transition lists — confirming the "these reverse" claim without needing to separately re-run the negative case. At `clearance=0` every term vanishes, giving the literal nominal boundaries `9,18,27,36`. This was derived directly from the formula in `index.html:1999-2007`, independent of the `literalPositiveA`/`literalPositiveB` arrays the fixture (`index.html:3264-3269`) compares against.

### Layout dimensions — re-derived from `layoutDesignPanelRows` arithmetic, not trusted from the report

Default draft: `jointCouponEdgeLength=45`, `jointCouponBodyDepth=22`, 6 clearances → 12 panels → 3 rows of 4 (`orderedIds.slice(index,index+4)`), `margin=10`, `gap=10` (explicit args at `index.html:2573`). Hand-walking the placement loop: each row of four 45 mm-wide panels advances `x` by `45+10` four times from `margin=10`, giving `documentWidth = 230`. Three rows of 22 mm-tall panels stack as `10 → 42 → 74 → 106` (two interior `+gap`, one final `+margin`). Result: **230×106 mm**, matching Grok's table and the fixture at `index.html:3289` exactly.

For the signed-endpoint case (3 clearances → 6 panels → rows of `[4,2]`), the second row's 2 panels don't extend `documentWidth` past the first row's 230, and the height stack is `10 → 42 → 74` (two rows, one interior gap, one final margin): **230×74 mm**, matching Grok's second figure.

### Parser — every stated rule traced to source, not summarized from the report

`parseJointCouponClearances` (`index.html:1625-1662`): token regex `/^[+-]?\d+\.\d{1,2}$/` requires an integer part, a literal decimal point, and 1-2 fractional digits — no bare integers, no exponents, no 3+ decimal places. `hundredths = Math.round(numeric*100)`, with `Object.is(hundredths,-0)` explicitly collapsed to `0` (confirmed this makes `"0.00"` and `"-0.00"` collide as duplicates, matching fixture `index.html:3243`). Range gate is `hundredths < -10 || hundredths > 30`, i.e. exactly `[-0.10, +0.30]` inclusive. Duplicate detection uses a `Set` keyed on the normalized `hundredths`, so it catches cross-representation duplicates (`"0.00"` vs `"-0.00"`), not just literal string duplicates. Count gate (`3-8`) is applied to the **surviving** values after per-token errors, matching the spec's stated order of checks. Values are pushed in encounter order — no re-sorting — so input order is preserved.

### Topology and safety formulas — traced directly, confirmed shared machinery

`buildJointCouponModel` (`index.html:1851-1904`) builds each panel via `buildFingerPanel({..., edges:[designPatternEdge(pattern,phase,t,clearance.value), plain, plain, plain]})` — one patterned edge, three literal `null` (plain) edges, confirmed by the fixture at `index.html:3279` and by direct reading. `buildFingerPanel` is the one-line delegation to `buildSlidingLidBodyPanel` confirmed correct in the immediately preceding Whisker-Fix review — the coupon feature adds no new panel-construction path. `requiredWeb = Math.max(.5, t*.25)` and the collapse guard `pattern.actualWidth - Math.abs(item.value) < requiredWeb - 1e-9` (line 1858) match the spec's "collapse guard" formula exactly, with the standard `1e-9` epsilon slack already used elsewhere in this codebase (e.g. `designPositiveCollinearOverlaps`). `minimumBodyDepth = Math.max(4*t, 16)` matches the stated grip-depth rule exactly.

### Fixture count — 515 confirmed via a from-scratch static/loop split, independent of Grok's number

Counted every `add(` call-site occurrence (not just line-leading ones, to avoid the undercounting risk a naive line-anchored regex carries) inside `runDesignGeometryFixtures` (`index.html:2733-3323`): **423** static occurrences. Of those, exactly 16 sit inside the six loop bodies established in prior reviews in this chain (legacy-baselines ×5 lines/4 iters, mating-pairs ×1 line/8 iters, Golden A ×1 line/21 iters, Golden B ×1 line/21 iters, glyphExpectations ×3 lines/6 iters, optionCombinations ×5 lines/4 iters — each independently re-inspected at its source location this pass, not assumed from the prior report). Non-loop static: `423 − 16 = 407`. Loop-generated at runtime: `20+8+21+21+18+20 = 108`. Runtime total: `407 + 108 = 515` — **exact match** to Grok's claim, arrived at independently. Since the diff is scoped entirely to the Designs generator/fixture region (confirmed by the numstat and by every touched line falling between `index.html:1625` and `:3323`), the grand total follows from the prior chain's established `977` total (at the pre-coupon `445`-fixture baseline) by substitution: `977 − 445 + 515 = 1047`, matching Grok's claimed grand total exactly.

### Minor findings — all four independently confirmed

| # | Grok's claim | Verification |
|---|---|---|
| M1 | Leading-zero tokens accepted (e.g. `"007.04"`) | **Confirmed** — `\d+` in the token regex places no limit on leading zeros or integer-part digit count; only the final range check (`-10..30` hundredths) would reject an out-of-range value, not the zero-padding itself |
| M2 | No fixed byte/hash signature fixture for the default coupon SVG (unlike the legacy-template baselines and the Finger Box/Drawer/Sliding regression-hash fixture) | **Confirmed** — the coupon block has `'Joint coupon SVG is deterministic'` (compares two live calls to each other) but no `svg.length===N && designFixtureHash(svg)==='...'` literal pin anywhere in the block (`index.html:3231-3318` inspected line-by-line) |
| M3 | Geometry-safety fixtures call the production helpers directly (`designPathSelfIntersects`, `designPositiveCollinearOverlaps`) rather than an independent reimplementation | **Confirmed** at `index.html:3282-3283` — same "fixture mirrors implementation" pattern already flagged and accepted as a known, harmless limitation in the Drawer Cabinet Phase 5.1 review's M2 |
| M4 | A single trailing decimal digit denotes tenths of a millimeter, not a different magnitude (e.g. `"0.5"` = 0.50 mm) | **Confirmed** — `\d{1,2}` accepts one or two fractional digits and `Number("0.5")=0.5` is scaled the same way as two-digit input; no separate code path, just a documentation nuance worth stating explicitly |

---

## Findings

No new findings beyond Grok's four Minor items, all confirmed above. No Blockers, no Majors — consistent with Grok's report.

---

## Unverified areas

Live 515/1047 execution and the browser-based fixture run could not be reproduced without a JS runtime in this environment (same limitation disclosed in every prior pass in this chain). Physical cutting/dry-fit of the coupon (the feature's actual purpose — dialing in clearance before committing to a full box) was not performed here, consistent with every prior Designs-tab audit; this feature's entire value proposition depends on that physical step, which remains outside any static-analysis review's reach.

---

## Required conclusion

```text
SAFE TO COMMIT
```

Every quantitative claim in Grok's audit — the signed-clearance worked example, both layout-dimension figures, the parser's fixed-decimal rules, the safety formulas, the topology, and both fixture totals — was independently re-derived from source rather than taken on the report's word, and all matched exactly. `git diff` confirms the shared geometry primitives this feature depends on were not modified, so the regression-safety claim rests on direct evidence, not narrative. Nothing found here changes Grok's recommendation.

---

*Second-pass review performed read-only at commit `b1ba69e` with uncommitted `index.html`/`README.md` changes. No application files were modified, staged, committed, or pushed. Unrelated untracked files were left untouched.*
