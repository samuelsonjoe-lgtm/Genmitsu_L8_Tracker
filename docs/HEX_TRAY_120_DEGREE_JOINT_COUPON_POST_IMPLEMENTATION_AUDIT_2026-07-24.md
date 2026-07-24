# Hex Tray 120° Joint Coupon — Post-Implementation Audit

Date: 2026-07-24  
Auditor role: independent adversarial geometry and regression reviewer (read-only)  
Repository: `C:\Genmitsu L8 Tracker`  
Authorized write: this report only  

---

## Executive summary

The uncommitted phase adds one Designs template, **Hex Tray 120° Joint Coupon** (`hex-tray-120-joint-coupon` / `hex-tray-120-joint-coupon-j1`), as a **prototype / physical-test coupon**. It generates three tab-depth candidates (C1/C2/C3) as handed LEFT/RIGHT wall pairs with a coupon-local 120° reference gauge, reusing unmodified `buildFingerPattern` / `designPatternEdge` / `buildFingerPanel` and **not** spoofing material thickness into `buildBoxModel`.

Independent Edge CDP verification: hex fixtures **32/0**, Designs geometry **1095/0**, T1 **76/0**, T2 **93/0**, T3A **23/0**, T3B **37/0**, Token tray **35/0**, storage **15/0**, S1/S2/S3 **21/17/37**, full suite **failed: 0**. Protected production goldens (T1, T2, Token tray, Dice tray) remain at their reported signatures.

**Gauge 120°:** independently recomputed from gauge vertex vectors → **exactly 120°** (no 60° inversion). Measurement faces are the long bottom edge and the relieved arm along the 120° direction; the outer connecting edge is not the angle reference.

**LEFT/RIGHT:** complementary by construction (right edge phase `true` vs left edge phase `false` on a shared vertical finger pattern with distinct tab depths per candidate). Paths are distinct. This establishes **credible mating candidates for scrap cutting**, not assembled angle or retention.

**Physical testing remains fully pending.** No source path can prove angle retention, best candidate, gap quality, or hex closure.

**Final recommendation: APPROVE FOR PHYSICAL COUPON TEST AND COMMIT**

Safe to commit `index.html` and the implementation report; safe enough to cut from scrap as a test coupon only.

---

## Final recommendation

| Item | Value |
| --- | --- |
| Recommendation | **APPROVE FOR PHYSICAL COUPON TEST AND COMMIT** |
| Safe to commit `index.html` + implementation report | **Yes** |
| Safe enough to cut from scrap | **Yes** (test coupon only) |
| Physically validated joint | **No** — still required |
| Existing production bytes changed | **No** |
| Gauge mathematically 120° | **Yes** (independent calculation) |
| LEFT/RIGHT genuinely complementary (2D profiles) | **Yes** (phase + edge placement; not physical proof) |
| Direct-file multi-width UI | **Not independently re-run** |
| Critical / High / Medium / Low | **0 / 0 / 0 / 2** |

---

## Actual HEAD

| Item | Observed |
| --- | --- |
| `git rev-parse HEAD` | `8f468db861ae5d1fe2f12ef5fd98ab0735f438e3` |
| `git log -1 --oneline` | `8f468db Add tabletop token and dice tray generator` |
| Matches stated baseline | **Yes** |

---

## Working-tree state

| Item | Observed |
| --- | --- |
| Tracked modification | `index.html` only |
| Diff stat | **107 insertions, 18 deletions** |
| `git diff --check` | **Pass** (LF/CRLF warning only) |
| Staged | **None** |
| Phase files | `index.html` + untracked implementation report |
| Architecture review | untracked (controlling design) |
| Other untracked | Protected |

---

## Files inspected

- Complete `git diff` for `index.html`
- `buildHexTray120JointCouponDesignResult`, gauge helpers, Finished/Assembly Guide
- Finger helpers: `buildFingerPattern`, `designPatternEdge`, `designSafePatternEdgePoints`, `buildFingerPanel` / `buildSlidingLidBodyPanel` (context; bodies not changed)
- `runHexTray120JointCouponFixtures` and Designs umbrella registration
- Architecture review + implementation report

---

## Diff summary

**Additive only:** draft defaults, template option under Coupons and prototypes, form fields, normalize/readback, local gauge panel/label, main coupon builder, screen-only guide, dispatch/preview/metrics, fixtures + selftest route, template-count 16→17.

**Shared helper function bodies:** not modified (diff scan of `buildFingerPattern`, `designPointsPath`, `buildBoxModel`, `layoutDesignPanelRows`, `designSvgValidation`, `serializeTabletopAccessorySvg`, existing `build*DesignResult` / Finished View definitions).

---

## Implementation-boundary compliance

| Allowed | Present |
| --- | --- |
| One template + j1 version | **Yes** |
| Eight additive draft fields | **Yes** |
| Three fixed tab-depth factors | **Yes** |
| Six wall panels + optional gauge | **Yes** |
| Screen-only assembly guide | **Yes** |
| Additive fixtures / goldens | **Yes** |

| Forbidden | Introduced? |
| --- | --- |
| Full hex / floor / rails / false bottom / liner / stacking | **No** |
| Generic diagonal mode / `designPointsPath` change | **No** |
| Evidence promotion / physical-proof status | **No** |
| Schema migration / Project handoff | **No** |
| Existing generator output changes | **No** (protected goldens) |

**Boundary compliance confirmed.**

---

## Defaults and fields

| Field | Default |
| --- | --- |
| `hexJointCouponWallLength` | 55 mm |
| `hexJointCouponWallHeight` | 30 mm |
| `hexJointCouponMaterialThickness` | 3 mm |
| `hexJointCouponFingerWidth` | 7 mm |
| `hexJointCouponClearance` | 0.00 mm |
| `hexJointCouponPanelSpacing` | 5 mm |
| `hexJointCouponIncludeGauge` | true |
| `hexJointCouponAssemblyLabels` | true |

Legacy missing-field normalization fixture-backed. Prior physical values (−0.075, +0.10, 2.88, 0.45) not used as defaults (fixture).

Wall dimensions describe the **nominal rectangular panel envelope** (width × height = 55 × 30); fingers extend within that envelope on one edge via existing pattern construction. UI/metrics report wall length/height as those envelope values.

---

## Candidate mathematics

| ID | Factor | Depth @ 3 mm (designRound) |
| --- | ---: | ---: |
| C1 | `1` | 3.000000 mm |
| C2 | `1/√3` | 1.732051 mm |
| C3 | `0.7` | 2.100000 mm |

**Independently verified** scaling: depth = thickness × factor at 2/3/4 mm (within float tolerance).  
All candidates share one clearance. Factors are not rounded before multiply; `designRound` applies to displayed/stored tabDepth after multiply. Clearance and factor are not conflated in copy.

C1 presented as comparison; C2 as √3-derived candidate (not “proven”); C3 intermediate — **confirmed** in form help and warnings.

---

## Physical-thickness separation

**Confirmed:**

- Manufacturing limits and finger minimums use `v.materialThickness`.
- `buildFingerPattern(wallHeight, preferredFingerWidth, materialThickness, …)` — thickness for min segment only.
- Tab depth passed separately: `designPatternEdge(pattern, phase, candidate.tabDepth, clearance)`.
- **`buildBoxModel` is not called** with fake thickness.
- Metrics report true `materialThickness` independent of candidate depths.

**No spoofed-thickness implementation.**

---

## LEFT/RIGHT handedness

**Confirmed from construction (source fact + fixture):**

```text
LEFT  → edge[1] (right side of panel), phase true,  depth = candidate.tabDepth
RIGHT → edge[3] (left side of panel),  phase false, depth = candidate.tabDepth
```

Edge indices match the existing four-edge panel convention (top, right, bottom, left). Opposite `phase` on opposite vertical edges is the same complementary convention used throughout Finger Box / coupon mating fixtures. Paths differ; each candidate carries two edges with matching `depth`.

**What this establishes:** 2D profiles are **credible interlocking mating pairs** for dry-fit experiments.

**What this does not establish:** assembled interior angle, retention under load, optimal depth, or residual gap after seating (architecture residual ~0.42·t for square-ended tabs remains a **physical** question).

Fixture strength is moderate (phase + path inequality, not interval-transform mating). Still sufficient for “safe to cut for testing.”

---

## Angle-retention claim review

User-facing hex coupon strings emphasize:

- prototype / physical test;
- none of C1/C2/C3 physically proven;
- dry-fit against gauge;
- angle retention and closure require physical testing;
- coupon is wall-to-wall starting point only, not full tray.

**No blocking “locks/fixes/proven 120° joint” claims** found in the new template UI/warnings/guide.

Metric “Joint target: 120° interior wall-to-wall joint” correctly describes **intent**, not proof.

---

## Gauge-only diagonal review

| Check | Result |
| --- | --- |
| `designPointsPath` unchanged | **Yes** |
| Wall paths cardinal-only (no `L`) | **Yes** (fixture + path regex) |
| Only `ANGLE-GAUGE-120` has diagonal `L` | **Yes** |
| Gauge path local to coupon | **Yes** (`hexJointCouponGaugePanel`) |
| Disable gauge removes diagonals; walls byte-identical | **Yes** (fixture) |

---

## Gauge geometry verification

**Independent calculation** (unrounded construction vertices):

| Segment | Role |
| --- | --- |
| p1→p2 | Horizontal reference face (30 mm) |
| p4→p3 | Second reference face at **120°** to horizontal (30 mm) |
| p2→p3 | Outer connecting edge (~62.35 mm) — **not** the 120° pair |
| p4→p1 | Blunt apex relief (~10.39 mm) |

Angle between vectors (p2−p1) and (p3−p4): **120.000000°**.  
Not 60°. Interior measurement uses the horizontal and the 120° arm; the polygon encloses the wedge side consistent with an interior-angle gauge.

Relief = 6 mm; no near-zero outline segments (fixture). Label “120” engraving-only when labels enabled; cut path finite and closed.

**Classification:** mathematically correct **120° interior reference gauge** suitable for physical comparison. It does not force the walls to that angle.

---

## Wall-panel geometry

Default with gauge: seven panels in fixed order  
`C1-WALL-LEFT|C1-WALL-RIGHT|C2-…|C3-…|ANGLE-GAUGE-120`.

Without gauge: six walls only. Unique IDs, unique paths, finite closed outlines, non-overlapping layout, exact-scale SVG, deterministic goldens. Labels optional without changing cut paths. Gauge optional without changing wall paths.

---

## Finger pattern

Uses real `buildFingerPattern` on wall height with physical thickness for minimum segment. Preferred width 7 mm. Odd-count convention preserved. All candidates share one vertical pattern object; only edge depth/phase differ. Manufacturing min uses physical thickness. Residual-web rules inherit from panel geometry validation via existing helpers.

---

## Screen-only guide

- Separate from `result.svg`
- Shows all three candidates with tab depths
- Idealized rectangles at 120° (explanatory, **not** derived from finger mesh)
- Explicit physical-test / not-proven wording
- `role="img"`, aria-label, empty on invalid/mismatch
- Deterministic; preview mode does not alter production SVG

**Classification:** explanatory reference diagram, correctly worded.

---

## Validation

Hard blocks cover non-positive dimensions/thickness/finger width, clearance out of −0.10…0.30, negative spacing, invalid/excessive tab depth, manufacturing min finger width, short edge for three fingers (via pattern helper), unique paths/IDs, panel geometry errors, gauge angle/relief, layout errors, SVG validation.

Mix of **input-range** and **generated-geometry** checks. Invalid fixtures cover zero dims, extreme thickness, bad clearance, zero finger width, negative spacing.

---

## Copy and warning findings

Adequate: C1 comparison language, shared clearance vs tab-depth factors, gauge dry-fit, physical-test requirements, wall-only scope.

**Low:** architecture residual interior mismatch for square-ended tabs (~0.42·t) is not called out as an expected physical observation; user still warned retention/closure are unproven.

---

## Production SVG and goldens

Independently fixture-verified:

| Scenario | Bytes | Hash |
| --- | ---: | --- |
| Default gauge + labels | 5985 | `3543f233` |
| Gauge disabled | 5534 | `481feafb` |
| Labels disabled | 1432 | `88e2ec43` |
| 4 mm / 8 mm finger | 5859 | `330aa05c` |
| +0.05 mm clearance | 6081 | `f1f8d5d5` |

Red cut layer; seven cut paths with gauge. No old golden rewritten.

---

## Existing-generator byte safety

Protected checks in hex fixtures (independently re-run as part of hex **32/0** and Designs **1095/0**):

| Generator | Signature |
| --- | --- |
| T1 corner coupon | 2992 / `4f543f95` |
| T2 shell | 2337 / `ed5d6f6e` |
| Token tray | 3036 / `a86aea09` |
| Dice tray | 1726 / `51a55721` |

Plus green T1/T2/T3A/T3B/token focused groups this audit.

**Existing production bytes: unchanged.**

---

## Fixture-chain quality

- Route `hex-tray-120-joint-coupon` once in dispatcher
- Designs umbrella adds **one** wrapper assert (does not re-spread 32 asserts into Designs total)
- 32 asserts cover registration, defaults, form, factors, handedness, cardinal walls, gauge isolation/angle, goldens, protected generators, persistence, constants
- Isolation via try/finally on draft/preview mode

**Low:** handedness could be strengthened with interval-transform mating, but current checks are real (phase + path), not pure string tautologies.

---

## Storage/schema/backup safety

Constants unchanged (fixture). No Project handoff branch for this template. No evidence enrollment. Generation/render leave storage/backup identical (fixture). Additive draft fields only.

---

## Direct-file validation

| Check | This audit |
| --- | --- |
| Focused/full selftests via disposable Edge CDP | **Performed** |
| Headed multi-width 1440/1024/768/480 UI walkthrough | **Not re-run** (implementation report only) |

---

## Exact independently observed totals

| Group | Passed | Failed |
| --- | ---: | ---: |
| Hex Tray 120° Joint Coupon | **32** | **0** |
| Designs geometry / goldens | **1095** | **0** |
| T1 corner coupon | **76** | **0** |
| T2 shell | **93** | **0** |
| T3A storage fit | **23** | **0** |
| T3B storage tray | **37** | **0** |
| Tabletop Token & Dice Tray | **35** | **0** |
| Storage recovery | **15** | **0** |
| Security S1 / S2 / S3 | **21 / 17 / 37** | **0** |
| Full suite | completed | **0** failed |

### Reported-only

| Claim | Status |
| --- | --- |
| Full suite **3240 passed / 51 groups** | Failures contradicted (0). Exact outer pass sum not re-instrumented. |
| Multi-width headed UI checklist | Implementation report only |

---

## Findings by severity

### Critical

None.

### High

None.

### Medium

None.

### Low

1. **Optional residual-gap advisory:** square-ended angle-compensated tabs may leave a small interior residual relative to a true miter; copy could mention this expected observation without claiming size.
2. **Handedness fixture depth:** strengthens confidence but does not numerically prove interval-level mating under rigid transform; acceptable for scrap-cut approval.

---

## Required fixes

**None** for commit or scrap cutting of the coupon.

---

## Optional observations

- Strengthen handedness with projected mating intervals.
- Explicit residual-gap note in warnings.
- Headed multi-width smoke before release packaging (optional).

---

## Physical-test status

**None of the following is established by this implementation or this audit:**

- correct assembled angle  
- angle retention  
- best candidate  
- acceptable gap / force / strength / appearance  
- full-hex closure  
- floor-joint compatibility  

**Still required after cut:** measure scrap; cut C1–C3 + gauge; dry-fit without glue; compare to gauge; inspect gap/force/damage/twist/reversibility; only then choose follow-up depths/clearances.

---

## Unverified areas

- Headed multi-width layout  
- Exact 3240 aggregate pass count  
- Physical cut results  

---

## Primary audit questions — answers

| # | Question | Answer |
| --- | --- | --- |
| 1 | One new template? | **Yes** |
| 2 | Thickness vs tab depth independent? | **Yes** |
| 3 | No fake thickness into box model? | **Yes** |
| 4 | Local builder uses helpers safely? | **Yes** |
| 5 | C1/C2/C3 differ only by factor? | **Yes** |
| 6 | LEFT/RIGHT complementary? | **Yes** (2D profiles) |
| 7 | Walls cardinal-only? | **Yes** |
| 8 | Gauge only diagonal production panel? | **Yes** |
| 9 | Gauge is 120° reference? | **Yes** |
| 10 | Screen guide not “proven”? | **Yes** |
| 11 | Deterministic finite closed layout? | **Yes** |
| 12 | Existing bytes/goldens unchanged? | **Yes** |
| 13 | Storage/schema/security intact? | **Yes** |
| 14 | Meaningful fixtures? | **Yes** |
| 15 | Physical testing still required? | **Yes** |

---

## Commit and scrap-cut decision

| Artifact / action | Safe? |
| --- | --- |
| Commit `index.html` | **Yes** |
| Commit implementation report | **Yes** |
| Cut coupon from scrap for testing | **Yes** |
| Treat as proven hex joint | **No** |

---

## Final inspection (audit process)

1. `git status` — only `index.html` modified among tracked files.  
2. `git diff --check` — pass.  
3. `git diff --stat` — 107+/18−.  
4. Diff re-reviewed.  
5. No application file modified by auditor.  
6. Only this audit report written.  
7. Nothing staged.  
8. Nothing committed or pushed.

---

## Evidence classification

- **Confirmed source fact:** construction, fields, boundaries.  
- **Independently calculated geometry:** gauge 120°; candidate depths.  
- **Fixture evidence:** 32 hex asserts + protected goldens + suite.  
- **Browser evidence:** disposable Edge CDP selftests (not multi-width headed).  
- **Inference:** physical mating quality / retention.  
- **Physical result still requiring a cut:** all real-world fit outcomes.

---

*End of audit.*
