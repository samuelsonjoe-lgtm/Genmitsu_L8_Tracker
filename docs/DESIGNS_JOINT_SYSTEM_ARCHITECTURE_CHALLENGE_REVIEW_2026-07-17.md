# Designs Joint-System Architecture — Independent Adversarial Challenge Review

**Date:** 2026-07-17  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Primary document challenged:** `docs/DESIGNS_JOINT_SYSTEM_ARCHITECTURE_REVIEW_2026-07-17.md`  
**Review type:** read-only adversarial challenge. No application files were modified.  
**Method:** git baseline recording; direct inspection of `index.html`, `README.md`, the Claude architecture report, and prior Designs/tray/coupon/production focused audits; source claims re-verified rather than accepted from report prose.

---

## Git baseline (actual, this review)

| Item | Value |
|---|---|
| **HEAD** | `52336097631c5ee3981a55557a1bfafade03eea0` |
| **Short HEAD** | `5233609` — *Add dice tray finished view* |
| **Branch** | `main` (tracking `origin/main`) |
| **`git status -sb`** | `## main...origin/main`; modified `README.md`, `index.html`; many untracked docs / LightBurn / debug artifacts |
| **`git diff --check`** | Clean aside from CRLF line-ending warnings on `README.md` / `index.html` |
| **`git diff --stat`** | `README.md` 4 lines changed; `index.html` +68/−18 (82 lines touched) — Divider Tray Finished View + pluralization cleanup in working tree |
| **Working-tree state** | Uncommitted Divider FV work present; Claude report’s suite totals (1664 / 872 / 244) describe that working tree, not bare HEAD alone |

Recent history (abridged): tray Finished Views and structured tray production switch land immediately before this review (`5233609`, `edd07d5`, `6ed566b`, …).

---

## Executive judgment

Claude’s **diagnosis is largely correct**: two genuinely different joint families (edge fingers vs wall-to-base tabs), a rectilinear panel pipeline that blocks real angled dovetails, session-only Designs drafts, misleading tray labels, a dead pre-whisker edge helper, and a coupon→promotion stack that already exists for finger clearances.

Claude’s **proposed delivery architecture is over-sequenced and partially over-abstracted**. The capability registry, edge-descriptor union, and multi-phase “teach the architecture” coupon plan are not required to fix the honesty problem or to give trays a cuttable fit test. Shipping those abstractions first would expand surface area without proportional user value.

**Verdict at end of document:** `ARCHITECTURE APPROVED WITH REQUIRED REVISIONS`

---

## 1. Validation of Claude’s source claims

| Claim | Verdict | Evidence |
|---|---|---|
| Designs drafts are session-only and do not currently persist `jointStyle` | **Confirmed** | `let designDraft = designDefaults()` (~392); toolbar text “Session-only… excluded from JSON backups” (~1878); `backupObject()` (~516–540) has no Designs field; README states Designs are session-only and backups exclude them |
| Finger Box, Sliding Lid body, Drawer Cabinet, Joint Fit Coupon share the same core finger-edge descriptor and panel-building pipeline | **Confirmed** | `designPatternEdge` (~2198); `buildFingerPanel` → `buildSlidingLidBodyPanel` (~2241–2242, ~2480); coupon uses `designPatternEdge` + `buildFingerPanel` (~2304–2310); cabinet/box call the same body panel builder |
| Dice and Divider Trays use a separate wall-to-base tab system that should remain distinct | **Confirmed (keep separate)** | `trayTabProfile` / `designWallPath` / `buildTrayModel` (~1933–2012); anonymous `serializeTrayCompatibilitySvg`; no `designPatternEdge` on trays |
| `designEdgePoints()` is dead code | **Confirmed** | Defined ~2199–2220; only occurrence of the name is the definition (live generator is `designSafePatternEdgePoints` ~2469, used from `buildSlidingLidBodyPanel`) |
| `designPointsPath()` supports only H/V segments and rejects diagonal geometry | **Confirmed for the writer** | ~2229–2239: non-axis-aligned step returns `''` |
| Real angled dovetails therefore require broader path, validation, and geometry changes | **Confirmed, with one nuance** | Writer is H/V-only; panel errors require closed orthogonal paths (~2586); finger edge generators assume axis-aligned `start`/`end`. **Nuance:** `designPathPoints` already tokenizes `L` (~2875), so the *parser* is not the hard gate — the *production point pipeline + orthogonality diagnostics + edge builders* are |
| Tray `jointStyle` values `finger` and `tab-slot` mean four-tab and two-tab wall-to-base profiles, not corner fingers | **Confirmed** | `trayTabProfile` count 4 vs 2 (~1933–1934); UI still labels select `finger` / `tab-slot` (~1883); help still says “Finger-tab” / “Tab-and-slot” (~1874); Finished Views already say “Four-tab / Two-tab wall profile” |
| Existing coupon and promotion systems can be safely extended without a schema migration | **Mostly confirmed, with qualifications** | Unknown-field preservation via spreads in `normalizeProductionSetting` (~5475–5520); no `SCHEMA_VERSION` bump needed for additive optional numbers. **Qualifications:** (1) coupon identity hash does not yet include a type (~5636); (2) verification rules hard-require `fingerJointClearanceMm` for coupon verify (~5731); (3) form/UI and fingerprint enumerate known fit keys (~1519, ~7045–7057); (4) tray apply-mappings never expose `fitClearance` today (~1677–1678) |
| Finished Views are isolated from production SVG and should remain semantic-only | **Confirmed as policy and current practice** | Dice/Divider FV builders consume projection/model fields; download uses `result.svg`; prior focused audits in this chain established isolation. Keep that rule |

### Disputed / qualified claims

1. **“No storage change is needed” is too absolute.** Free-form evidence snapshots can carry new fields without a schema bump, but **identity, verification applicability, form round-trip, fingerprint, and Designs application** each have enumerated known fields. Additive data can be *stored* and still *silently ignored* or *mis-classified* without small, targeted code updates. That is not a migration of old rows; it is still real integration work.

2. **“Coupon reuses designWallPath without a public shared contract” is incomplete.** Reuse is correct and desirable, but the coupon will freeze a *call-site contract* (argument order, tab depth = thickness, slot width = t + clearance). That is acceptable if the coupon stays a thin consumer of tray helpers; it becomes a second engine if the coupon reimplements spacing, tab width clamps, or slot placement.

3. **Suite totals in Claude’s header** describe the **working tree** (Divider FV + cleanup), not necessarily a clean checkout of `5233609` alone. Architecture conclusions do not depend on the exact count; the baseline note should stay explicit.

---

## 2. Capability registry — firm recommendation

### Challenges

| Question | Answer |
|---|---|
| Duplicate `normalizeDesignDraft`? | **Yes, if both validate the same enums.** Today draft validation already owns template joint values (~2157–2161 for trays). A registry that also lists allowed joints creates two lists unless one is generated from the other. |
| Two sources of truth? | **Yes, unless the registry is the only list and validation only consults it.** Claude proposes both “populate UI from registry” and “normalize cross-check” — safe only if validation never re-lists allowed IDs independently. |
| Enough joint choices to justify it? | **No.** Per template today: trays have a 2-way select; boxes have one fixed structural joint; coupon has one finger type. That is not a capability matrix. |
| First wall-to-base coupon without registry? | **Yes.** `couponType` can be a local select on `joint-fit-coupon` with default `finger-edge` and one new option, validated in `normalizeDesignDraft` only. |
| UI generation from registry improve safety? | **Not yet.** It adds indirection for two options that are already hardcoded. |
| Template-owned config safer? | **Yes for Phase 0–2.** Template branches already own parameters. |
| What Phase 1 problem cannot wait? | **None that users feel.** Terminology honesty does not need a registry. Coupon type selection does not need a registry. |
| Disagreement prevention? | If a registry ever exists: **single const array of IDs; validation iterates that array; no parallel allow-lists; no functions in the registry.** |

### Firm recommendation

**Defer the registry until another structural joint style exists (or a third coupon type forces shared discovery UI).**

Do **not** add the registry before the coupon.  
Do **not** add a “minimal subset” that still powers three consumers — that is still dual validation with no second joint.

When a second real structural choice appears on a product template (e.g. keyed tabs *and* four/two-tab profiles as independently named capabilities), introduce a **data-only** map at that time, owned by one validation path.

---

## 3. Edge-descriptor contract — firm recommendation

Claude’s tagged union (`plain` / `finger` / `tab-row` / `rabbet-step`) is a **useful description of existing architecture**, not a useful near-term refactor.

| Challenge | Verdict |
|---|---|
| Replace `null` with `{kind:'plain'}`? | **Not worth fixture churn.** Behavior-neutral renames still touch every panel edge assignment and many fixtures. |
| Add `kind:'finger'` before a second edge treatment? | **No.** Tagging one live kind is pure ceremony. |
| Would `tab-row` fit the same panel-edge abstraction? | **Only for A-pipeline boxes** (tabs on a rectangular panel edge). Tray wall-to-base tabs are wall-bottom protrusions + base slots, not four edge descriptors on a finger panel. Forcing trays into the union is a disguised universal joint engine. |
| Does `rabbet-step` belong in a diode-laser sheet model? | **Not as a first-class cut joint.** Partial-depth rabbets are not reliable on a 20W through-cut workflow; laminated shoulders (already used by sliding lid) are the honest analogue. |
| Terminal-offset / corner rules joint-specific? | **Already are** (`designEdgeTerminalOffset`, composite Front span). Expanding kinds multiplies special cases inside the shared builder — the opposite of “small architecture.” |
| Second template-owned builder safer? | **For a genuinely different joint family, yes.** Prefer template-owned geometry + shared H/V diagnostics over expanding `buildSlidingLidBodyPanel` into a dispatcher. |

### Firm recommendation

**Remove the descriptor-union refactor from the active roadmap.** Keep it as optional future documentation language only. Revisit only when a second A-pipeline edge treatment is approved and cannot be expressed with today’s `null | designPatternEdge` without ugly boolean flags.

---

## 4. Wall-to-base coupon — best first *user-facing* feature

### Comparison

| Candidate | User value now | Risk | Needs registry? | Needs promotion? | Notes |
|---|---|---|---|---|---|
| Terminology / help cleanup | High (trust) | Near zero | No | No | Honesty floor; no cut path |
| Capability declarations | Near zero | Low–med (dual truth) | Self | No | Architecture theater |
| Wall-to-base tab fit coupon | High (trays have no clearance evidence path) | Medium | **No** | **No for first ship** | Best *cuttable* next step after honesty |
| Layered slot-hiding base | Medium cosmetic | Medium (B-pipeline bytes) | No | Optional | Should follow some physical tab-fit data |
| Cabinet shelf score guides | Medium (known registration gap) | Low–med | No | No | Good, independent; does not fix tray fit |
| Decorative faux dovetail engrave | Low–med cosmetic | Low on A-pipeline | No | No | Learning feature, not fit |
| Keyed tray tabs | Medium usability | Medium (bytes + assembly) | No | Prefer coupon first | Structural change without fit evidence is premature |
| Standalone tray fit-test template | High, cleaner UI | Medium (template sprawl) | No | No | Cleaner than overloading Joint Fit Coupon long-term |

### Coupon challenges

| Challenge | Answer |
|---|---|
| Reuse `designWallPath` without public tray contract? | **Yes, if coupon only calls existing helpers** (`trayTabProfile`, `designTabPositions`, `designWallPath`) and does not invent a parallel profile formula. Document “consumer of tray helpers,” not “shared joint API.” |
| Real production fit or approximation? | **Partial but useful.** A wall segment + base strip tests local tab/slot clearance and heat at that feature scale; it does **not** fully reproduce full-tray grain orientation, closed-corner glue-up, multi-wall cumulative error, or full base warpage. Label it honestly as a **feature clearance coupon**, not a finished-tray proof. |
| Heat, slot length, tab count, grain, assembly? | **Inadequately for full assembly; adequately for clearance ranking** if coupon uses production tab width/count policy and real thickness + clearance. Prefer same sheet as the product run. |
| Same builder prevents second engine? | **Necessary, not sufficient.** Fixtures must assert helper-output equality / shared profile object, not only visual similarity. |
| Extending Joint Fit Coupon UI too complex? | **Risk yes** if promotion, multi-type matrices, and registry land together. A single `couponType` select with two options is fine. |
| Multiple focused coupon templates cleaner? | **Slightly cleaner long-term**; one template with two types is fine for the first addition. Do not invent a coupon registry. |
| Promotion in first coupon phase? | **No.** Geometry + cut + manual notes can ship first. Promotion needs identity, field path, verify rules, and apply mapping work (§5). |

### Firm selection

**Best first user-facing implementation after (or bundled only with) terminology: wall-to-base tab fit coupon.**

**True prerequisites (only these):**

1. Honest tray labels/help (so “finger” does not mean three things while teaching clearance).  
2. Reuse of existing tray tab geometry helpers.  
3. Default Joint Fit Coupon bytes remain pinned when type = current finger behavior.

**Not prerequisites (architectural preference only):**

- Capability registry  
- Edge-descriptor union  
- Internal enum rename  
- Production-setting `tabFitClearanceMm`  
- Phase 7.2 apply mapping for trays  
- Finished View updates for the coupon (coupon has no FV by design today)

---

## 5. Evidence, promotion, and storage

### What exists today (verified)

- Coupon identity: `couponIdentity` object + `promotionCanonicalize` + `designFixtureHash` → `sourceId` (~5636). Fields include clearances, edge length, body depth, machine/material, cut params — **not** a coupon type.  
- Winner field: always `fitSettings.fingerJointClearanceMm` (~5639).  
- Verify rule: joint-coupon verified requires `piecesFit` **and** `fields.fingerJointClearanceMm` (~5731).  
- Evidence list normalization: fixed top-level evidence fields (~1497–1500, ~5446+); snapshots clone freely.  
- Production settings: `fitSettings` spread preserves unknown keys (~5516–5519), but **UI form only edits two known fit fields** (~7045–7057).  
- Fingerprint: enumerates only `fingerJointClearanceMm` and `drawerTotalLateralClearanceMm` (~1519) — unknown fit keys **do not** stale the Designs review fingerprint.  
- Designs application: trays map **thickness only**; no fit clearance apply (~1677–1678). Cabinet collapses two joint clearances onto one stored field (~1668–1671).  
- Merge: nested `fitSettings` object spread (~10121 region per search).

### Field classification

| Proposed field / concept | Classification | Notes |
|---|---|---|
| `designJointCapabilities` | **transient** (code only) | Defer entirely for now |
| Tray UI labels | **transient** (HTML strings) | Not stored |
| Internal `jointStyle` `finger`/`tab-slot` | **session draft only** | Not in backups; fixtures use them |
| `couponType` on draft | **session draft** | Include in coupon identity when promotion exists |
| `couponType` in evidence snapshot / identity | **persisted in evidence only** (when promotion ships) | Free-form snapshot + identity hash; not a schema bump |
| Winner clearance for finger coupon | **persisted in production settings** as `fitSettings.fingerJointClearanceMm` | Already |
| Winner clearance for tab coupon | Prefer **reuse `fitClearance` semantics**, not a new production key initially | If promoted later: either map to a new optional `fitSettings.traySlotClearanceMm` **or** keep evidence-only until tray apply mapping exists |
| `fitSettings.tabFitClearanceMm` | **unnecessary for first coupon**; **optional later in production settings** | Needs `strictOptionalNumber`, form field, fingerprint entry, apply mapping — not free |
| Edge descriptors `kind` | **unnecessary** | Code shape only |
| Decorative option flags | **session draft** | Never evidence unless user promotes cut settings |
| Real dovetail parameters | **unnecessary / out of roadmap** | — |

### Challenge answers

| Question | Answer |
|---|---|
| Does adding `couponType` change identity for existing finger coupons? | **Yes, if inserted into `couponIdentity` without versioning.** New hashes differ from old `sourceId`s for the same physical finger session parameters. Old evidence rows keep old IDs; duplicate warnings may miss “same session, new code.” Prefer `version: 2` identity **or** omit type when type is default finger-edge so legacy identity stays stable. |
| Old/new evidence collision? | **Risk is under-dedup, not overwrite.** Finger vs tab coupons with identical clearance lists would collide if type is omitted. Type (or separate sourceType) is required before two coupon kinds share promotion. |
| Identity vs snapshot metadata? | **Both when promotion exists:** identity for dedup; full draft/result snapshot for audit. |
| Free-form snapshot enough? | **For storage yes; for product behavior no** — verify/apply/UI need explicit fields. |
| `tabFitClearanceMm` need normalization/fixtures? | **If persisted in production settings, yes** — same path as existing fit keys. Prefer not for phase-1 coupon. |
| Ship coupon without promotion? | **Yes — recommended.** |
| Schema version bump unnecessary? | **Yes for additive optional fields**, matching Phase 7 practice. Still need code path updates where fields are enumerated. |
| Hidden enumerations that drop new values? | **Fingerprint** (stale detection blind to new fit keys); **production form** (cannot edit unknown keys but preserves them via `...data.fitSettings` if not wiped); **verify rules** (tab coupon cannot verify via finger field); **apply mappings** (trays never receive clearance). Spreads alone do not make a feature complete. |

### Persistence recommendation (when promotion is eventually added)

Affected paths (must be updated together):

1. `buildJointCouponPromotionCandidate` identity + winner field selection  
2. `promotionStatusRules` verify applicability  
3. `normalizeProductionSetting` / `productionSettingFromForm` / `productionSettingFields` if a new fit key is written  
4. `designProductionFingerprint` if that fit key should invalidate Designs review  
5. `productionSettingDesignApplicability` if Designs can apply tray clearance  
6. Promotion fixtures around coupon winner and evidence-only flows  

Until then: **geometry-only coupon; user records winner in notes or Library manually.**

---

## 6. Tray terminology plan

### Decision: **rename UI only** (labels + help + README). Keep internal IDs `finger` / `tab-slot`.

| Question | Answer |
|---|---|
| Permanent legacy internal IDs? | **Yes for now.** Grep-ability loss is real but small; fixture churn for rename is pure cost. |
| Aliases useful if drafts are session-only? | **Low value.** Session drafts die on refresh; aliases mainly protect mid-session renames during development. |
| Internal rename maintainability? | **Marginal.** Honest labels already fix user confusion; Finished Views already say four/two-tab. |
| Fixture churn if renamed? | **Many tray fixtures** hardcode `jointStyle:'tab-slot'` / default finger (~3382+). Avoid. |
| URL/form/fixture/evidence/export/browser state? | Form draft session state and fixtures only; **not** backups. No evidence field stores tray jointStyle. |
| Preferred wording | Prefer **walls + tabs + wall-to-base**, not “finger,” not vague “tab-and-slot” alone |

### Exact recommended labels and help

**Select options (display labels; values unchanged):**

- `finger` → **Four-tab walls (wall-to-base)**  
- `tab-slot` → **Two-tab walls (wall-to-base)**

**Select field label:** `Wall-to-base tab profile` (not “Tray joint style” if that implies corner joinery)

**Help note (replace ~1874-style text):**  
“Tabs on each wall bottom mate with through-slots in the base. Adjacent walls meet as butt joints and need glue after a dry fit. This is not corner finger joinery.”

**Kerf / clearance voice:** keep “slot clearance,” “test on scrap.”

**Do not** claim interlocking corners or “finger-jointed tray.”

---

## 7. Dovetail conclusion

### Rectilinear limitation — Claude is correct on the hard path

- Production panel paths are built as point lists and serialized with `designPointsPath` H/V only (~2229–2239).  
- `designPanelGeometryErrors` requires closed orthogonal paths (~2586).  
- Live finger edges come from `designSafePatternEdgePoints` assuming axis-aligned edges.  
- Shortcut “emit raw path strings with diagonals for one template” would **bypass** panel diagnostics unless those are extended — that is a second unvalidated pipeline. **Reject.**  
- “Polygon points + L commands” still needs validation, layout bounds, and mating rules. `designPathPoints` already reads `L` (~2875); that does **not** make diagonal production safe.

### Option ratings (Genmitsu L8 20W, ~3 mm plywood)

| Concept | Rating | Candor |
|---|---|---|
| Faux engraved dovetails (score lines) | **practical now** | Cosmetic; must label decorative; char line softens fine pins visually |
| Applied decorative dovetail pieces | **practical only after physical testing** | Alignment labor, kerf-rounded interiors, glue mess; commercial only for premium pieces |
| Layered dovetail appearance (laminations) | **uneconomical** (often **poor fit** for production) | Multiplies cut time, glue, tolerance stack |
| Actual through dovetails (angled in-plane) | **poor fit** for engine + **misleading** if sold as superior strength in 3 mm ply | Glue line dominates; short-grain pins chip; kerf taper hurts angled fit |
| Half-blind appearance | **misleading** as real; decorative engrave only | Diode cannot reliable pocket |
| Sliding dovetail laminations | **poor fit** as “dovetail”; laminated channel is a different product story | Rename if ever built |

**Real dovetails stay outside the active roadmap.**

---

## 8. Cleaner-joint options (Claude’s list)

| Option | Customer benefit | Cut time | Material | Labor | Dimensional risk | Structural vs cosmetic | Special geometry? | Coupon? |
|---|---|---|---|---|---|---|---|---|
| Decorative wall skins | High “hidden joint” look | High (2× walls) | High | High glue/align | Kerf match outer size | Cosmetic | Plain panels only | Optional overhang scrap |
| Layered slot-hiding base | Medium underside finish | Low+ | +1 base | Medium | Doubled base thickness semantics | Cosmetic | Extra rect | Optional |
| Internal corner blocks | Medium rigidity / glue area | Low | Low | Medium | Block size vs inside volume | Structural aid | Rectangles + assembly text | Optional |
| Keyed / interrupted tray tabs | Medium assembly correctness | Same order | Same | Same | Tab rhythm / minimum web | Structural usability | Extend tab profile | **Yes, after basic tab coupon** |
| Captured bottoms | Medium look/feel | Medium | Same order | Medium | New mating | Structural | New edge treatment | Yes |
| Face frames | High furniture look | High | High | High | Reveal math | Cosmetic structure | New parts | Later |
| Shelf score guides | Medium registration | Negligible | None | Low | Score placement | Assembly aid | Score paths | No |

### Top three for Joe’s L8 / likely products

1. **Terminology honesty + wall-to-base tab coupon** (fit learning before more tray geometry)  
2. **Cabinet shelf-position score guides** (cheap fix for a known real gap; A-pipeline score pattern already proven)  
3. **Layered slot-hiding base *or* internal corner blocks** — pick by product: trays → layered base; boxes/cabinet → corner blocks  

**Reject as near-term product bets:** layered faux through-dovetails, real dovetails, fiddly slot caps, face frames before skins prove decorative-part workflow.

---

## 9. Production SVG recommendations

| Challenge | Verdict |
|---|---|
| Named score/cut vs anonymous tray vs future engrave | **Correct split today.** Keep both contracts. |
| Trays anonymous indefinitely? | **Default yes** until a user-facing option needs score/engrave on tray sheets. Do not rename groups “for architecture.” |
| Option-specific anonymous vs named switching | **Confusing if silent.** If ever needed: explicit versioned output mode, default bytes frozen, README callout. |
| Score/engrave on trays → new versioned template? | **Prefer option-gated serializer branch with pinned default**, not a whole new template key, unless UX demands it. |
| LightBurn color/layer assumptions | **Unsafe to over-assume.** Color mapping is user-controlled; group order is a convention, not a guarantee. Document; do not encode LightBurn layer *names*. |
| Faux dovetails: score vs engrave fill vs out of scope | **Score lines in existing blue group first**; filled engrave group later only if needed. Out of scope for first joint work. |
| Filenames distinguish experimental contracts? | **Helpful but insufficient.** Bytes + README + fixtures are the contract. |
| Operation summary before third group? | **Nice UX, not a blocker.** Do not block Phase 0–coupon on it. |

**Do not change existing tray production bytes without a clear user-facing structural or cosmetic option.**

---

## 10. Fixture strategy

Claude’s full matrix (11 categories per joint) is **aspirational and excessive for every phase**.

| Challenge | Verdict |
|---|---|
| Full hash matrix every joint? | **No.** Reserve parametric matrices for production geometry that ships on product templates. Coupons need fewer size cells. |
| Circular checks? | Shared-builder equality is **semi-circular** but valuable against copy-paste drift; pair with **literal goldens** and hand-derived min-web cases. |
| Shared-builder equality enough for coupon fidelity? | **Necessary, not sufficient** — also pin default SVG for new type; pin default finger type unchanged. |
| Physical coupon results as fixture expectations? | **Never** as numeric goldens. Physical results belong in Library/evidence notes. |
| Min feature size without dual formula? | Assert against **literal thresholds** already in production code constants, or error string matches — do not re-derive `t/4` in the fixture. |
| Golden files vs inline constants? | **Inline hashes/length** match current suite culture; extract only if a single case becomes huge. |
| Browser interception every phase? | **No.** Terminology: none. Coupon: optional later. FV: when FV changes. |
| Codex vs independent audit fixtures? | Codex: determinism, default bytes, validation errors, builder reuse. Audit: physical notes, adversarial wording, isolation greps. |
| Suite growth vs single-file maintainability? | **Real long-term risk.** Prefer compact tables over new fixture frameworks. |

### Minimum fixture sets

| Change | Minimum fixtures |
|---|---|
| Terminology-only | Label/help string presence; tray default goldens **unchanged** (length/hash); no new matrix |
| Capability declaration | **Defer** — if ever: single-source allow-list vs normalize errors only |
| Wall-to-base coupon | Default finger coupon bytes unchanged; new type deterministic SVG golden; invalid type rejected; tab profile matches `trayTabProfile`/`designWallPath` output equality for same inputs; min-web / clearance collapse errors with literals |
| One new structural joint | Off-by-default legacy bytes; on-option golden; parametric extremes subset; FV isolation if FV updated same commit or next |
| Decorative engraving option | Cut-group byte identity on/off; score presence when on; wording warning fixture |

---

## 11. Roadmap challenge and revision

### Claude phases — keep / revise / remove

| Phase | Claude intent | Challenge | Disposition |
|---|---|---|---|
| 0 Terminology + dead helper | Honesty | Correct; independently valuable; dead helper deletion is safe and small | **Keep (first Codex task)** |
| 1 Capabilities registry | Single source of truth | No second joint yet; dual validation risk | **Remove from active roadmap** |
| 2 Wall-to-base coupon + promotion | Fit path | Valuable, but **too large** if registry + promotion + type identity all included | **Keep geometry coupon; strip registry; defer promotion** |
| 3 Structural tray enhancement | Keyed tabs / layered base | Too early before physical tab clearance learning | **Defer until after coupon use** |
| 4 Finished View for new options | FV notes | Only when a product option changes semantics | **Merge into the feature commit that needs it** — not a standalone phase |
| 5 Decorative faux dovetails | Score decoration | Low risk but low priority vs fit honesty | **Optional side quest; not sequenced before coupon** |
| 6 Descriptor refactor + captured bottom | Formalize union | Speculative abstraction + large feature | **Remove refactor; defer captured bottom** |
| 7 Broader adoption | Cabinet fields, skins, guides | Omnibus | **Split into independent optional items; no Phase 7 bucket** |

### Revised active roadmap (only justified phases)

#### Phase A — Terminology honesty + dead helper removal  
- **Purpose:** Stop lying about tray “finger” joints; remove foot-gun dead code.  
- **Exact scope:** Tray select labels, help/README wording, delete `designEdgePoints`, wording fixture(s), confirm tray goldens unchanged.  
- **Expected files:** `index.html`, `README.md`.  
- **Production-byte impact:** **None** (verify Dice/Divider defaults).  
- **Storage impact:** None.  
- **Fixture depth:** Light.  
- **Physical testing:** None.  
- **Audit depth:** Focused light.  
- **Suggested commit message:** `Clarify tray wall-to-base tab labels and remove dead edge helper`

#### Phase B — Wall-to-base tab fit coupon (geometry only)  
- **Purpose:** Give trays a cuttable clearance ranking tool reusing tray helpers.  
- **Exact scope:** `couponType` (or equivalent) on Joint Fit Coupon; default finger behavior byte-pinned; new wall-base type; normalize validation; builder-reuse fixtures; honest UI copy that coupon ≠ full tray proof. **No** registry, **no** production-setting field, **no** promotion identity change required for first ship (promotion UI can remain finger-only or disabled for new type).  
- **Expected files:** `index.html`, `README.md` (coupon section).  
- **Production-byte impact:** Default coupon unchanged; new type new golden only.  
- **Storage impact:** None required.  
- **Fixture depth:** Medium (not full product matrix).  
- **Physical testing:** **Required** before calling any clearance “production.”  
- **Audit depth:** Full focused on geometry isolation + reuse.  
- **Suggested commit message:** `Add wall-to-base tab clearance coupon`

#### Phase C — Coupon promotion integration (optional, separate)  
- **Purpose:** Promote tab clearance winners into Library without abusing `fingerJointClearanceMm`.  
- **Exact scope:** Identity versioning / type field; winner field path; verify rules; optional new fit key + form + fingerprint + tray apply mapping **or** evidence-only promotion.  
- **Files:** `index.html` (+ fixtures).  
- **Bytes:** None.  
- **Storage:** Additive evidence / optional fit key.  
- **Fixtures:** Promotion suite extensions.  
- **Physical:** Uses Phase B results.  
- **Audit:** Focused on storage/identity.  
- **Message:** `Promote wall-to-base tab coupon winners into production settings`

#### Phase D — Independent small product improvements (any order, separate commits)  
- Cabinet shelf score guides  
- Layered tray base (off-by-default)  
- Internal corner blocks (assembly text + plain parts)  
- Keyed tray tabs **after** Phase B physical learning  

Each: own commit, own audit, own byte policy.

#### Explicitly out of active roadmap

- Capability registry  
- Edge-descriptor union / `rabbet-step`  
- Real / half-blind / sliding dovetails  
- Omnibus “Phase 7 broader adoption”  
- Decorative faux dovetails as a gated prerequisite for anything structural  

---

## 12. First Codex task — exact choice

**Choose: terminology plus verified dead-helper removal.**

### Include

- Tray select **display** labels and field/help/README wording per §6  
- Finished View / metrics strings already honest may stay; fix any remaining “Finger-tab” / “Tab-and-slot” user-facing tray copy  
- Delete unused `designEdgePoints` after confirming zero call sites  
- Fixtures: wording assertions; default tray (and preferably default coupon/box) production goldens unchanged  
- Diff limited to honesty + dead code  

### Exclude

- Registry / capabilities  
- Internal enum rename / aliases  
- Coupon types  
- Promotion / fitSettings  
- Descriptor refactor  
- Finished View geometry changes  
- Any production SVG intentional change  

**Rationale:** Smallest reviewable commit with immediate trust value; clears terminology before any new joint feature teaches the wrong words; dead helper removal prevents nearby joint work from extending the wrong function.

---

## 13. Required disagreement table

| Claude proposal or claim | Grok verdict | Evidence | Keep / revise / defer / reject | Required action |
|---|---|---|---|---|
| Capability registry before coupon | Overengineered now | Two tray options already validated in `normalizeDesignDraft`; no multi-joint matrix | **Reject for near term** | Defer until second structural joint |
| Internal tray enum rename in Phase 1 | Unnecessary churn | Session-only drafts; many fixtures; FV already honest | **Defer indefinitely** (UI-only rename) | Keep `finger`/`tab-slot` IDs |
| Delete dead `designEdgePoints` | Correct | Definition only ~2199; live path `designSafePatternEdgePoints` | **Keep** | Include in Phase A |
| Wall-to-base coupon as first meaningful feature | Correct *after* terminology | Trays have no fit coupon; finger coupon uses `buildFingerPanel` only | **Revise sequencing** | Terminology first; coupon second; no registry |
| `couponType` persistence migration-free | Partially true | Free-form snapshots OK; identity/verify/form/fingerprint enumerate known fields | **Revise** | Ship geometry first; design identity carefully when promoting |
| Production setting tab clearance soon | Premature | Trays not in apply fit mappings (~1677–1678); fingerprint omits unknown fit keys | **Defer** | Optional Phase C |
| Edge descriptor union on roadmap | Premature abstraction | `null` edges work; `tab-row` ≠ tray wall-base | **Reject from active roadmap** | Document only |
| Keyed tray tabs as Phase 3 | Too early structurally | B-pipeline byte risk without fit data | **Defer** | After coupon physical use |
| Decorative faux dovetails Phase 5 | Optional cosmetic | Score channel exists; low product urgency | **Defer / optional** | Not a gate |
| Real dovetails deferred | Correct | `designPointsPath` + orthogonal panel errors | **Keep (stronger: out of roadmap)** | Do not schedule |
| Layered base as cleaner joint | Good candidate | Simple rect + assembly warning | **Keep as optional product phase** | After coupon preferred |
| Cabinet shelf guides | Strong independent win | Known plain shelves; score precedent | **Keep as independent item** | Need not wait on joint system |
| Phase 0→7 ordering | Too long / wrong deps | Registry and descriptor not load-bearing | **Revise to A–D** | See §11 |
| “Architecture READY FOR CHALLENGE REVIEW” | Fair self-assessment | Diagnosis solid; plan overbuilt | **Revise plan before multi-phase Codex** | Phase A can start after this review |

---

## 14. Risk findings

### Blocker
*None for Phase A (terminology + dead helper).*

### Major

1. **Misleading tray terminology still in live UI**  
   - **Source:** ~1874, ~1883  
   - **Concern:** `finger` / “Finger-tab” implies box-style joinery  
   - **Consequence:** Wrong mental model for clearance testing and customer claims  
   - **Resolution:** Phase A labels/help  
   - **Blocks next Codex task?** No — *is* the next task  

2. **Promotion path hard-wired to finger clearance**  
   - **Source:** ~5639, ~5731  
   - **Concern:** Wall-base coupon cannot truthfully promote into `fingerJointClearanceMm`  
   - **Consequence:** Wrong Library semantics if rushed  
   - **Resolution:** Geometry-first coupon; separate promotion design  
   - **Blocks Phase A?** No. **Blocks honest promotion of tab winners?** Yes  

### Moderate

3. **Capability registry dual source of truth**  
   - **Source:** Claude §7 vs `normalizeDesignDraft`  
   - **Concern:** Divergent allow-lists  
   - **Consequence:** UI offers what validation rejects (or reverse)  
   - **Resolution:** Defer registry  
   - **Blocks Phase A?** No  

4. **Coupon identity omits type**  
   - **Source:** ~5636  
   - **Concern:** Colliding `sourceId` across coupon kinds  
   - **Consequence:** Bad dedup / confusing evidence  
   - **Resolution:** Versioned identity when second type is promotable  
   - **Blocks geometry coupon?** No if promotion disabled for new type  

5. **Tray fit never applied from production settings**  
   - **Source:** ~1677–1678  
   - **Concern:** `tabFitClearanceMm` would not reach Designs without mapping  
   - **Consequence:** False sense of closed loop  
   - **Resolution:** Explicit apply work or evidence-only  
   - **Blocks Phase A/B geometry?** No  

6. **Descriptor union / shared-builder expansion**  
   - **Source:** Claude §6 / Phase 6  
   - **Concern:** Hidden universal engine  
   - **Consequence:** Corner-case explosion, golden churn  
   - **Resolution:** Remove from active roadmap  
   - **Blocks Phase A?** No  

### Minor

7. **Working-tree vs HEAD suite totals**  
   - **Source:** uncommitted Divider FV  
   - **Concern:** Reports disagree on 1663 vs 1664  
   - **Consequence:** Audit noise only  
   - **Resolution:** Record baseline carefully  
   - **Blocks?** No  

8. **`designPathPoints` accepts `L` while writers reject diagonals**  
   - **Source:** ~2229 vs ~2875  
   - **Concern:** Future author may think diagonals are “already supported”  
   - **Consequence:** Unvalidated diagonal SVG  
   - **Resolution:** Document; never short-circuit diagnostics  
   - **Blocks?** No  

### Verified strengths

1. **Shared finger pipeline is real and coupon-backed** (`designPatternEdge` + `buildFingerPanel` + Joint Fit Coupon).  
2. **Tray tab system is cleanly separate and should stay so.**  
3. **Designs session-only boundary is real** (`backupObject` omission).  
4. **Score/cut decorative channel and cut-identity fixture culture are proven.**  
5. **Unknown-field spreads on production settings are a sound additive storage pattern** when paired with explicit UI/fingerprint/apply updates.  
6. **Finished View semantic isolation is established practice** and should remain a hard rule.

---

## 15. Final recommendation

1. **Is Claude’s architecture fundamentally sound?**  
   **Yes as a diagnosis of the current system; only partially sound as an implementation plan.** The seams it names are real. The multi-phase abstraction ladder is not required.

2. **What should be removed or deferred?**  
   Capability registry; edge-descriptor union; internal enum rename; promotion/storage work inside the first coupon; real dovetails; omnibus Phase 6–7; registry-gated Phase 1.

3. **What should be implemented first?**  
   **Phase A:** UI/help terminology + delete `designEdgePoints`.

4. **Is another review needed before Codex starts?**  
   **No for Phase A.** **Yes, a short focused design note (not a full architecture re-review) before Phase B coupon geometry**, covering coupon layout, type default pinning, and promotion non-goals. This challenge review is sufficient to start Phase A.

5. **Should the wall-to-base coupon require capability-registry work?**  
   **No.**

6. **Does promotion/storage integration belong in the first coupon implementation?**  
   **No.**

7. **Should real dovetails remain outside the active roadmap?**  
   **Yes.**

---

## Required verdict

```text
ARCHITECTURE APPROVED WITH REQUIRED REVISIONS
```

**Revisions in one line:** keep the diagnosis and the wall-to-base coupon destination; drop the registry and descriptor-union prerequisites; ship terminology (+ dead helper) first; ship coupon geometry without promotion; leave real dovetails and speculative joint engines off the roadmap.

---

*Challenge review performed read-only at HEAD `5233609` with working-tree Divider Tray Finished View changes recorded. No application files were modified, staged, committed, pushed, reset, cleaned, stashed, moved, deleted, renamed, or rewritten.*
