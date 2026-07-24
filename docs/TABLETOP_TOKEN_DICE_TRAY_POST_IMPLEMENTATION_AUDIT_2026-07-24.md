# Tabletop Token & Dice Tray — Post-Implementation Audit

Date: 2026-07-24  
Auditor role: independent skeptical reviewer (read-only)  
Repository: `C:\Genmitsu L8 Tracker`  
Authorized write: this report only  

---

## Executive summary

The uncommitted phase adds one Designs template, **Tabletop Token & Dice Tray** (`tabletop-token-dice-tray`), as a **prototype / starting point**. The builder composes unmodified T2 (`buildTabletopRectangularShellDesignResult`) and T3A (`buildTabletopStorageFitCouponDesignResult`) results into ten panels, one exact-scale production SVG, a screen-only Finished View, measurement-only liner metadata, and a non-mutating Project handoff. Tracked product change is limited to `index.html` (+163 / −22) against baseline `d1a06e7`.

Independent disposable-profile Edge CDP verification reproduced: token-tray **35/0**, Designs geometry/goldens **1094/0**, Gift Box **69/0**, T2 shell **93/0**, T3A **23/0**, T3B **37/0**, Project handoff **17/0**, storage **15/0**, S1/S2/S3 **21/17/37**, and full suite **failed: 0**. Existing T2/T3A/T3B production bytes are fixture-proven stable after token-tray generation; existing golden pin strings in the diff were not rewritten except additive template-count (15→16) and new token-tray pins.

No Critical or High defects. Stock-fit for nominal 305 mm (12×12) material is **correctly not claimed as proven**; exact layout **277 × 324.2 mm** is disclosed and exceeds 305 mm height. One Low Finished View CSS class mismatch (liner fill style) does not put liner geometry into production SVG.

**Final recommendation: APPROVE FOR COMMIT**

It is safe to commit `index.html` and `docs/TABLETOP_TOKEN_DICE_TRAY_IMPLEMENTATION_2026-07-23.md`. Physical validation remains entirely pending.

---

## Final recommendation

| Item | Value |
| --- | --- |
| Recommendation | **APPROVE FOR COMMIT** |
| Safe to commit `index.html` + implementation report | **Yes** |
| Existing production bytes changed | **No** (confirmed by fixtures + Designs suite) |
| Stock-fit disclosure acceptable | **Yes** (no false “fits 12×12” claim; exact bounds shown; 305 mm unverified stated) |
| Direct-file multi-width UI walkthrough | **Not independently re-run** this audit |
| Physical validation | **Pending** (not claimed) |
| Critical / High / Medium / Low | **0 / 0 / 0 / 2** |

---

## Actual HEAD

| Item | Observed |
| --- | --- |
| `git rev-parse HEAD` | `d1a06e78e7f8ef56d0ba7152f997e9c879ea1034` |
| `git log -1 --oneline` | `d1a06e7 Clarify focus and Z offset terminology` |
| Matches stated baseline | **Yes** |
| Branch | `main`…`origin/main` synchronized |

---

## Working-tree state

| Item | Observed |
| --- | --- |
| Tracked modification | `index.html` only |
| Diff stat | **163 insertions, 22 deletions** |
| `git diff --check` | **Pass** (LF/CRLF warning only) |
| Staged | **None** |
| Phase files | `index.html` + untracked `docs/TABLETOP_TOKEN_DICE_TRAY_IMPLEMENTATION_2026-07-23.md` |
| Architecture review | untracked `docs/FINISHED_VIEWS_PRACTICAL_GENERATOR_ARCHITECTURE_REVIEW_2026-07-23.md` |
| Other untracked | Protected historical docs / LightBurn / debug / generator script |

---

## Files inspected

- `index.html` (full diff + builder/Finished View/fixtures/handoff/metrics in context)
- Architecture review (recommended Option B / composition contract)
- Implementation report (claims and goldens)

---

## Diff summary

**Additive:**

- `designDefaults` / `normalizeDesignDraft` / form fields: fourteen `tabletopTokenTray*` preferences
- Template option under **Trays**
- `buildTabletopTokenDiceTrayDesignResult` (composition)
- `buildTabletopTokenDiceTrayFinishedViewSvg`
- Dispatch in `buildDesignResult`, preview modes, metrics, results HTML
- `projectDraftFromDesign` branch
- `runTabletopTokenDiceTrayFixtures` + selftest route + Designs umbrella wrapper assert
- Template-count fixtures 15→16

**Not modified in body (confirmed by diff scan):**

- `buildTabletopRectangularShellDesignResult`
- `buildTabletopStorageFitCouponDesignResult`
- `buildTabletopStorageTrayDesignResult`
- `buildBoxModel`, `designPanelGeometryErrors`, `layoutDesignPanelRows`, existing Finished View bodies (only new sibling functions inserted)

---

## Implementation-boundary compliance

| Approved | Present |
| --- | --- |
| Exactly one new template | **Yes** |
| Compose real T2 + T3A builders | **Yes** (unmodified calls) |
| Ten physical panels | **Yes** |
| One production SVG + screen Finished View | **Yes** |
| Measurement-only liner | **Yes** |
| Existing Designs-to-Projects handoff extension | **Yes** |
| Prototype / starting point (not Workshop-ready) | **Yes** |
| No stacking lip / lid / divider / hardware / liner cut | **Yes** (form copy + construction) |
| No evidence/maturity registry enrollment | **Yes** (fixture asserts absence from registry) |

**Boundary compliance confirmed.**

---

## Product-identity findings

| Check | Result |
| --- | --- |
| Name “Tabletop Token & Dice Tray” | Confirmed |
| ID `tabletop-token-dice-tray` / version `tabletop-token-dice-tray-t1` | Confirmed |
| “Stackable” as product name | **Not used** |
| Stacking/nesting feature implied | **No** (explicit exclusion copy in form) |
| Workshop-ready / proven claims for this template | **No** (status `prototype-starting-point`) |
| In `tabletopAccessoryProductRegistry` | **No** |
| Existing T1–T3B identities | Unchanged |

Architecture’s working name “Stackable…” was correctly **not** shipped as the user-facing product name.

---

## Defaults findings

Confirmed via fixtures and source defaults:

| Field | Default |
| --- | --- |
| Interior | 120 × 90 × 32 mm |
| Material thickness | 3 mm |
| Joint clearance | 0.00 mm |
| Preferred finger width | 10 mm |
| Rail height | 12 mm |
| False-bottom thickness | 3 mm |
| Total false-bottom clearance | 0.80 mm |
| Panel spacing | 5 mm |
| Liner | cork, 0.50 mm/side, 2 mm thick |
| Assembly labels | off |

All fourteen fields use `tabletopTokenTray` prefix. Legacy drafts missing fields normalize to defaults. Defaults do **not** copy Joe’s −0.075 / +0.10 / 2.88 / 0.45 evidence values. Default result is valid (fixture).

---

## Composition findings

`buildTabletopTokenDiceTrayDesignResult`:

1. Builds shell sub-draft → `buildTabletopRectangularShellDesignResult`
2. Builds storage sub-draft → `buildTabletopStorageFitCouponDesignResult`
3. Copies panels (`map({...panel})`) before `layoutDesignPanelRows`
4. Propagates child errors/warnings
5. Validates panel ID sets (5+5)
6. Runs `designPanelGeometryErrors` on laid-out panels
7. Serializes once via `serializeTabletopAccessorySvg`
8. Derives liner from **storage** false-bottom metrics

Child SVG strings are not concatenated. Metrics come from child metric objects plus layout bounds. Panel IDs remain unique. **Confirmed safe composition pattern** (mirrors T3B approach).

---

## Panel/geometry findings

Default panel IDs and sizes (fixture-backed):

| ID | Size (mm) |
| --- | ---: |
| FLOOR | 126 × 96 |
| FRONT/BACK | 126 × 35 |
| LEFT/RIGHT | 96 × 35 |
| RAIL-FRONT/BACK | 100.615385 × 12 |
| RAIL-LEFT/RIGHT | 68.666667 × 12 |
| FALSE-BOTTOM | 119.2 × 89.2 |

Closed finite outlines, non-overlapping layout, deterministic SVG **3036 / a86aea09** independently fixture-verified. Rail decimals are intentional finger-pattern outputs from reused T3A math and pin stably. No liner panel in production SVG. `sheetCount: 1` means one output document/layout, not proven physical 12×12 stock fit.

---

## False-bottom/storage findings

| Metric | Value | Source |
| --- | ---: | --- |
| Total / per-side clearance | 0.80 / 0.40 mm | T3A contract |
| False bottom | 119.2 × 89.2 mm | T3A |
| Rail height / hidden storage height | 12 mm | Explicitly `hiddenStorageHeight: storageMetrics.railHeight` |
| Visible wall above | 17 mm | 32 − 12 − 3 |
| Exterior envelope | 126 × 96 × 35 mm | T2 outer |

UI metric label: “Visible wall / hidden storage” with both numbers — accurate that hidden-storage height equals rail height (cavity under false bottom before accounting for false-bottom thickness sitting on rails). Consistent with Finished View / Project notes.

---

## Liner findings

Supported types: `none`, `cork`, `felt`, `leather`, `generic-liner` (“Other verified liner”).  
Default cork allowance: **118.2 × 88.2 × 2 mm** (false bottom − 2×0.50 clearance).  
Type `none` zeros clearance/thickness effects and leaves SVG identical to cork default (same production bytes).  
All liner type/thickness/clearance changes leave production SVG byte-identical (fixture).  
Advisory requires composition/backing/adhesive/ventilation/fire-safety verification.  
No automatic laser-safe claim.

**Low:** Finished View CSS defines `.token-liner` but polygon class is `token-tray-liner`, so liner fill styling may not apply (geometry/id still present; not production).

---

## Production-layout and stock-fit findings

| Fact | Status |
| --- | --- |
| Default bounds 277 × 324.2 mm | Confirmed (fixture + result metrics) |
| No auto-scale/shrink | Confirmed (exact-scale path) |
| Generic >400 mm warning | Not triggered (324.2 < 400) — correct |
| Explicit “fits 12×12 / 305 mm” claim | **Absent** (good) |
| Explicit “does not fit 305 mm” UI warning | **Absent** (optional polish) |
| Metrics show exact production SVG size | **Yes** (`Panels / sheet` → dimensions) |
| Project notes claim 12×12 fit | **No** |
| Hard-coded 305 mm schema limit | **No** |

**Verdict:** stock-fit disclosure is **acceptable for commit**. The layout exceeds nominal 305 mm height; this is honestly documented in implementation report and fixtures as unverified. Not a High false claim. Optional future: surface a soft note when either dimension exceeds 305 mm without inventing a hard block.

---

## Finished View findings

- Invalid/mismatch/non-finite → `''`
- Uses `tabletopAccessoryProjectPoint`
- Reads `result.finishedView` (outer envelope, insets, rails, liner)
- Shell, rails, false bottom, hidden-storage messaging, optional liner layer
- `role="img"`, useful `aria-label`, screen-only / not-in-download text
- Deterministic; production SVG lacks Finished View classes
- No stacking lip / magnets / divider / lid in diagram

**Low:** CSS class name mismatch for liner fill (above).

---

## Project handoff findings

`projectDraftFromDesign` for this template:

- Single dedicated branch
- Name with actual interior dims (`Tabletop Token & Dice Tray — 120 × 90 × 32 mm`)
- Prototype / starting-point construction notes
- False bottom / rails / liner measurement-only when active
- Thickness, clearance, finger width
- Coupon + physical validation language
- No proven / workshop-ready language
- Non-mutating for state/localStorage (fixture)

---

## Storage/schema/backup findings

Unchanged constants (fixture + source): `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`. Diff does not alter encrypted/vault formats, merge/replace, Project/Library/Inventory/Test Grid/Pricing schemas, evidence identity fields, maturity registry entries, or existing Designs template IDs beyond additive registration.

New draft fields are additive prefixes only. Generation does not persist. No migration marker. Template not in staged-evidence promotion.

---

## Existing-generator byte-safety findings

| Evidence | Result |
| --- | --- |
| Token fixture: T2/T3A/T3B SVG before/after token generation | **Identical** |
| Designs suite | **1094/0** (includes all prior goldens + wrapper) |
| Gift Box | **69/0** |
| T2 rectangular shell fixtures | **93/0** |
| T3A storage-fit fixtures | **23/0** |
| T3B storage tray fixtures | **37/0** |
| Diff edits to prior golden length/hash strings | **None** found (only new token pins + template count 16) |

**Existing production bytes: not regressed.**

---

## Fixture-chain findings

- New route `tabletop-token-dice-tray` registered once in dispatcher
- Also summarized (not re-spread) inside `runDesignGeometryFixtures` as one pass/fail wrapper — **no double-count of 35 asserts into Designs total**
- 35 meaningful asserts: registration, defaults, form readback, legacy, panels/dims, layout, goldens, liner isolation, FV, invalid boundaries, T2/T3A/T3B isolation, handoff, storage constants
- State/localStorage restored on handoff checks

---

## Production-golden findings

Independently verified by fixtures (and suite green):

| Scenario | Bytes | Hash |
| --- | ---: | --- |
| Default cork | 3036 | `a86aea09` |
| Liner none | 3036 | `a86aea09` |
| 4 mm material | 2980 | `94e12bcf` |
| 0.10 mm clearance | 3060 | `b2c3b6ab` |
| Assembly labels | 8617 | `7ac9570d` |

Cork ≡ none production equality confirmed. No old golden rewritten.

---

## Validation performed

| Check | Result |
| --- | --- |
| `git status` / HEAD / `diff --check` / `diff --stat` / full diff | Done |
| `python -m html.parser` | **Pass** |
| JS runtime via Edge fixtures | **Pass** |
| Token tray | **35 / 0** |
| Gift Box | **69 / 0** |
| Designs geometry/goldens | **1094 / 0** |
| T2 shell | **93 / 0** |
| T3A fit coupon | **23 / 0** |
| T3B storage tray | **37 / 0** |
| Designs→Project handoff | **17 / 0** |
| Storage recovery | **15 / 0** |
| Security S1 / S2 / S3 | **21 / 0**, **17 / 0**, **37 / 0** |
| Full suite `?selftest=all` | **completed: true, failed: 0** |
| Multi-width headed UI | **Not re-run** (implementation report only) |

Disposable Edge profiles only.

---

## Exact independently observed totals

| Group | Passed | Failed |
| --- | ---: | ---: |
| Tabletop Token & Dice Tray | **35** | **0** |
| Gift Box | **69** | **0** |
| Designs geometry / production goldens | **1094** | **0** |
| Tabletop T2A shell | **93** | **0** |
| Tabletop T3A storage-fit | **23** | **0** |
| Tabletop T3B storage tray | **37** | **0** |
| Design-to-Project handoff | **17** | **0** |
| Storage recovery | **15** | **0** |
| Security S1 | **21** | **0** |
| Security S2 | **17** | **0** |
| Security S3 | **37** | **0** |
| Full suite | completed | **0** failed |

### Reported-but-not-independently-reproduced

| Claim | Status |
| --- | --- |
| Full suite **3207 passed** | Failures contradicted (0). Exact outer pass sum **not** re-instrumented. |
| Direct-file UI at 1440/1024/768/480 | Implementation report only |
| No network / no page errors on UI pass | Report only |

---

## Findings by severity

### Critical

None.

### High

None.

### Medium

None.

### Low

1. **Finished View liner CSS class mismatch:** style rule `.token-liner` vs polygon class `token-tray-liner` — liner may render without intended fill; production unaffected.
2. **Optional stock-fit copy polish:** UI shows exact 277 × 324.2 mm and prototype status but does not explicitly state “exceeds nominal 305 mm / 12×12 stock height”; not a false fit claim, optional clarity only.

---

## Required fixes

**None** for commit.

---

## Optional deferred observations

- Align Finished View CSS selector with `token-tray-liner` class.
- Soft advisory when layout exceeds 305 mm in either dimension (still not a hard block; architecture correctly defers multi-sheet).
- Physical coupon → shell → rails → false bottom → full tray → separate liner test sequence (below).
- Stacking lip / evidence pipeline / Workshop-ready graduation remain correctly deferred.

---

## Physical-validation status

**No physical cut or proof was claimed or performed.**

Still required before trusting the tray:

1. Measure actual plywood  
2. Run fit coupon  
3. Test shell/corners  
4. Test rail seating/glue  
5. Test false-bottom flatness/removal  
6. Cut full prototype  
7. Separately test liner materials  
8. Record results via ordinary Log/Library  

Fixtures and audits cannot substitute for this.

---

## Unverified areas

- Headed multi-width visual/responsive inspection of the new form and preview modes  
- Exact aggregate pass count 3207  
- Cross-browser beyond Edge CDP  

---

## Primary audit questions — answer matrix

| # | Question | Answer |
| --- | --- | --- |
| 1 | Exactly one new template? | **Yes** |
| 2 | Composes real T2/T3A? | **Yes** |
| 3 | Existing builders/FV unchanged? | **Yes** |
| 4 | Exactly ten panels? | **Yes** |
| 5 | Valid deterministic geometry? | **Yes** |
| 6 | Liner out of production? | **Yes** |
| 7 | Finished View screen-only? | **Yes** |
| 8 | Existing goldens preserved? | **Yes** |
| 9 | Invalid/borderline params handled? | **Yes** |
| 10 | Stock-fit limitations honest? | **Yes** (acceptable disclosure) |
| 11 | Storage/schemas preserved? | **Yes** |
| 12 | Non-mutating Project draft? | **Yes** |
| 13 | Fixture chain correct? | **Yes** |
| 14 | No a11y/network regression found? | **No regression in suite** |
| 15 | No physical-proof claim? | **Yes** |

---

## Whether it is safe to commit

| Artifact | Safe? |
| --- | --- |
| `index.html` | **Yes** |
| `docs/TABLETOP_TOKEN_DICE_TRAY_IMPLEMENTATION_2026-07-23.md` | **Yes** |
| This audit | Optional companion |

Do not include unrelated untracked files.

---

## Final inspection (audit process)

1. `git status` — tracked change remains `index.html` only.  
2. `git diff --check` — pass.  
3. `git diff --stat` — 163+/22−.  
4. Diff re-reviewed — additive generator + fixtures.  
5. No application file modified by auditor.  
6. Only this audit report written.  
7. Nothing staged.  
8. Nothing committed or pushed.

---

## Evidence classification

- **Confirmed:** source + independent CDP fixture returns.  
- **Reported-only:** 3207 pass total; multi-width headed UI.  
- **Inference:** rail decimal stability from T3A math (also golden-pinned).  
- **Deferred physical work:** listed above.

---

*End of audit.*
