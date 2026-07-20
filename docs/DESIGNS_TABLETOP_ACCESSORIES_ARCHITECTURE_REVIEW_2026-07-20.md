# Tabletop Accessories — Long-Term Designs Architecture Review

**Repository:** `C:\Genmitsu L8 Tracker`
**Date:** 2026-07-20
**Reviewer:** Claude Sonnet 5
**Type:** read-only architecture review. No file was edited, staged, committed, pushed, or otherwise changed. No implementation is proposed — this is an architectural recommendation only.

---

## 0. Repository state

`git status -sb`: clean tracked tree (only the long-standing unrelated untracked set present); HEAD `78765512f58d5a16eea89073958252bd5171b6a7` ("Add Dice Tray storage and insert options"), matching the stated baseline `7876551`; branch `main`, ahead/behind origin/main `0 0`; `git diff --check` clean. Nothing was modified during this review.

## 1. Method

Before designing anything new, this review inventoried what geometry/construction/rendering/validation/fixture infrastructure **already exists and is already shared** in `index.html`, so the proposal builds on real, proven primitives rather than invented ones. The key finding driving this entire proposal: **the codebase already has several genuinely generic, shape-agnostic primitives** (a parameterized finger-pattern generator, a generic polygon-panel descriptor, a generic sheet-layout function, a generic geometry validator, a generic label-placement engine) — but **the one thing that is never shared today is the Finished View projection**, and that exact gap is what produced the Gift Box G1 lid-rendering defect audited immediately prior to this review (`NaN` lid coordinates, no rotation between Closed/Open — see the Gift Box focused audit). This lesson is treated as load-bearing evidence throughout this proposal, not an aside.

## 2. Recommended architecture: Option B, with a lightweight product registry

**Recommend Option B — a reusable geometry engine, with several separate, independently-registered product templates that share its helpers — not Option A (one generic generator with interchangeable "modes").**

Reasoning:
- **Option A repeats a failure pattern already observed in this codebase.** Gift Box G1 already crammed two lid "modes" (lift-off / hinged) into one template, and the resulting cross-mode conditional logic produced a live, confirmed bug (a lift-off configuration emitting a warning about a hinge-coupon panel that mode can never have). A single "Tabletop Accessories" generator handling rectangle, hexagon, deck box, dice vault, and six more products as internal modes would multiply that exact risk across far more products, not fewer.
- **Option B is what already works here.** `buildFingerPattern`, `designPanelFromPoints`, `layoutDesignPanelRows`, `designPanelGeometryErrors`, `designSvgValidation`, and `buildAssemblyLabelPaths` are *already* shared, generic functions called identically by Finger Box, Sliding-Lid Box, Drawer Cabinet, Joint Fit Coupon, and Gift Box today. Each of those remains its own template with its own small fixture group. Formalizing this pattern for a new "Tabletop Accessories" family — one shared engine module, many small registered products — is the lowest-risk, most proven path, not a new idea.
- **Per-product templates keep fixture groups small.** The codebase already has precedent for this exact shape: `runGiftBoxFixtures`, `runTrayModelFixtures`, and `runDiceTraySystemFixtures` are standalone, independently-`?selftest=`-registered groups (the last explicitly re-invokes the one below it as a regression guard). This is the pattern to extend, not replace — and it directly satisfies "avoid monolithic generators with thousands of assertions."
- **Option C (a fully declarative, plugin-driven product-spec engine) is a considered rejection, not an oversight.** For roughly ten products over "many years," a data-driven plugin system is premature abstraction: it would add a schema-versioning and extensibility-contract burden with no current evidence it's needed, and it would make the Gift-Box-style "one function quietly does the wrong thing for one mode" bug *harder* to spot, not easier, by hiding per-product logic behind generic dispatch. The one piece of Option C worth adopting *within* Option B is small and concrete: a **lightweight product registry** — because today, adding one template already requires hand-editing four separate places (the template whitelist in `normalizeDesignDraft`, the dispatch chain in `buildDesignResult`, the display groups in `designTemplateSelect`, and the field blocks in `renderDesigns`), and this scatter will only get worse across ten new products. A registry is a single array of `{id, displayName, group, build, renderFields, fixtureRunner}` entries that each of those four call sites reads from, reducing "add a product" from four hand-edits to one registry entry plus the product's own files. This is a refactor of *how new products register themselves*, not a redesign of how any product's geometry works — and it is scoped to the **new** family only; legacy templates are not touched.

## 3. Geometry representation

The engine should represent a tabletop accessory shell as a small stack of descriptors, each building on the last, all reusing the codebase's existing low-level polygon primitive (`designPanelFromPoints`'s `{points, path, bounds}` shape) rather than inventing a new one:

- **Shell outline** — an ordered ring of vertices in the horizontal plane, at the wall's outer face. A rectangle outline has 4 vertices; a regular hexagon has 6. This is the one shape-specific input; everything downstream is generic.
- **Wall loop** — derived from the shell outline: an ordered list of **edge descriptors**, each `{id, ordinal, startVertex, endVertex, length, outwardNormal, wallHeight, prevEdgeId, nextEdgeId}`. This directly answers the question posed: **yes, the shell must be represented as an ordered list of wall segments, not hard-coded front/back/left/right.** `buildBoxModel` today hard-codes a 4-edge, 4-corner rectangular topology; that assumption is exactly what blocks hexagon support, and generalizing it to a wall loop is the single most consequential design decision in this proposal — it is what lets rectangle, hexagon, and any future N-gon share the rest of the engine.
- **Corner descriptor** — `{id, edgeA, edgeB, interiorAngle, joinType}`. The interior angle matters structurally: a rectangle's corners are always 90°, a regular hexagon's are 120°. Finger-joint interleaving (the existing `buildFingerPattern` math) is inherently a 90°-perpendicular concept and is not valid at other angles — so `joinType` must be selectable per corner-angle family (finger-jointed at 90°; captured-floor-registered butt/miter at other angles — see §6).
- **Panel descriptor** — a semantic wrapper *around* the existing low-level polygon shape: `{edgeId | role, kind:'wall'|'floor'|'lid'|'divider'|'insert', cornerJoins, floorJoin, fingerPatternRef, materialThicknessMm, points/path/bounds}`. Today, "what a panel means" is expressed only informally via string IDs like `'front'`/`'FRAME-LEFT'`, which is exactly why Gift Box's hardware-guide and label logic had to pattern-match on raw ID strings throughout. A real semantic layer removes that fragility for the new family.
- **Finger descriptor** — a thin, per-edge wrapper around the *already-generic* `buildFingerPattern(length, preferredWidth, thickness, errors, label)`: `{edgeId, count, actualFingerWidth, phase}`. No new finger math is needed — this is composition, not reinvention.
- **Captured-floor descriptor** — `{floorOutline (the shell outline inset by wall thickness), captureDepth, captureLipWidth}`. This is the one genuinely new geometry concept in this proposal (§6).
- **Divider descriptor** — `{id, orientation, position, length, thicknessMm, joinType}`, where `length` is **always** derived from the engine's own finished-cavity output, never from raw user input (§7 — this directly closes the physical lesson from the Dice Tray storage divider).
- **Insert descriptor** — `{id, physical:boolean, materialTag (opaque string), boundingRegion, thicknessMm?, clearancePerSide}` (§8).
- **Lid descriptor** — `{strategy:'lift-off'|'hinged'|..., footprint, attachmentEdgeId?}`, reusing the existing lift-off/overhanging/hinged vocabulary already proven in Sliding-Lid Box and Gift Box, generalized to attach to any wall-loop's top edge rather than a hard-coded rectangle.

## 4. Shapes

**Rectangle and regular hexagon** as the first two shell-outline generators, exactly as directed. **Additional polygon support does naturally follow — conditionally.** It follows *only if* the wall-loop abstraction in §3 is genuinely edge-ordered rather than secretly still assuming four named sides. The concrete acceptance test for whether the abstraction was built correctly: **adding a third shape (e.g., a pentagon or an L-shaped tray) should require writing only a new shell-outline generator function and choosing a corner-join strategy — it should require zero changes to the divider, insert, label, layout, validation, or Finished-View code.** If a third shape ever requires touching any of those, the abstraction has leaked and should be revisited before a fourth shape is attempted.

## 5. Dimension contract and terminology

This is the most important product-honesty fix this proposal makes, directly answering the Dice Tray divider lesson. Recommend one consistent vocabulary for the new family (distinct from, and not required to retrofit onto, legacy templates' looser "inside/outside" language):

| Term | Meaning |
|---|---|
| **Requested interior** | The number the user types in — what they want to fit inside. Nominal only. |
| **Finished cavity** | The actual usable interior space *after* wall thickness, floor/captured-floor inset, and any divider/insert intrusion are accounted for. This is what dividers and inserts must be sized against — never the requested interior. |
| **Outer envelope** | The assembled, closed exterior footprint — what has to physically fit on a table or in a bag. |
| **Panel dimensions** | The flat, uncut bounding size of an individual piece, before finger/joint profiling. |
| **Cut dimensions** | The actual exported production geometry size for a panel, which may differ from panel dimensions if kerf or fit-clearance is applied as a geometric offset rather than left as a separate reference value. |

**Mandate:** whenever *finished cavity* differs from *requested interior* — which it always will, by at least wall thickness, and further whenever a captured floor, divider, or insert intrudes — **both numbers must be shown side by side in the result summary**, never silently substituted for one another. This is the direct architectural fix for the physical lesson: today, nothing in the Tray model distinguishes them at all (nominal user input is used verbatim as the divider/insert span and as the reported "finished" dimension).

## 6. Joints — construction philosophy

The physical lesson from the legacy wall-to-base tray is specific: **walls are not mechanically connected to each other**, so squareness depends entirely on the builder's care during glue-up. The new construction philosophy addresses this directly:

- **Corner joints, 90° shells (rectangle):** finger-jointed wall-to-wall corners, reusing the existing, already-proven `buildFingerPattern`/`buildSlidingLidBodyPanel` interleaving exactly as Finger Box and Gift Box already do. This remains the premium default where the geometry supports it.
- **Floor joints (all shells):** a **captured floor** — the floor sits in a rebate/lip formed by each wall's inner-face profile, rather than butting flush against wall bottoms or relying on base-plate slots alone. This is the single most important new construction primitive: at assembly, every wall is mechanically registered against its two neighbors' correct positions *by the floor's fixed perimeter shape*, not by the builder holding four independent walls square by hand. This is offered as the architectural hypothesis that should replace wall-to-base tabs as the premium default — but it is a hypothesis, not yet a proven fact, and must clear a physical-prototype checkpoint (§16) before being treated as doctrine.
- **Corner joints, non-90° shells (hexagon and beyond):** finger-joint interleaving is not valid at non-perpendicular angles. Rather than inventing a bespoke hexagon-specific joint now (a product decision, out of scope for this review), the architecture should treat corner-join strategy as **selectable per corner-angle family** (`finger` at 90°; `captured-floor-registered butt/miter` otherwise), with the captured floor carrying the primary squaring responsibility for non-rectangular shells. The exact non-90° corner treatment is a physical-prototype question, not an architecture question, and is flagged as a geometry risk (§15) and a required checkpoint (§16) rather than decided here.
- **Divider joints:** generalize the existing Dice Tray precedent (a divider is a full-height wall-like panel registered into a floor slot) into the shared engine, with the slot always cut from the *finished-cavity* geometry, and length always derived from finished cavity (§7).
- **Future lid joints:** treated as a pluggable "lid attachment strategy" hung off the wall loop's top edge — reusing the existing lift-off/hinged vocabulary from Sliding-Lid Box/Gift Box as prior art, not a new joint system.
- **Future removable dividers:** a parametric *variant* of the same divider joint (a through-slot sized for slide-in/out rather than a glued slot), not a separate system.
- **Future magnets and hardware:** generalize the pattern Gift Box already proved works — score/drill-reference guide marks attached to a panel by id, never structural cut geometry. This is already a reusable pattern; the new engine should adopt it directly rather than reinvent it per product.

## 7. Dividers

Recommend a single shared divider primitive: `{position, thicknessMm, joinType}` in, `{length, cutSlotGeometry}` out — where **length is computed exclusively from the engine's own finished-cavity output**, never from the raw requested-interior number. This is a direct, mandatory architectural fix: the current Dice Tray divider computes its span from nominal `trayWidth`/`trayDepth` input with no intermediate "measure what was actually built" step, which is precisely the lesson physical testing surfaced. The new engine must compute finished cavity *first*, as an authoritative internal value, and every divider/insert/lid-footprint calculation downstream must consume that value — never the original user-entered numbers directly.

## 8. Inserts

Recommend a generic insert descriptor with exactly one geometry-relevant branch: **is this insert physical (needs a cut panel) or measurement-only (just reports a bounding region)?** Everything else — wood, foam, leather, cork, felt, acrylic — is carried as an **opaque material tag**, used only for display/notes text, never branched on in geometry code. This directly corrects a code smell already present in the just-shipped Dice Tray insert feature, which literally checks `insertType === 'wood'` as a *geometry-generating* condition. The new engine's insert primitive should instead ask only `physical: true/false`; a future foam insert is not a new code path, it is the same `physical:true` path with a different tag string.

## 9. Finished View engine

This is the highest-priority shared component to build, and the one place today with **zero** sharing: every existing Finished View builder (`buildFingerBoxFinishedViewSvg`, `buildSlidingLidFinishedViewSvg`, `buildGiftBoxFinishedViewSvg`, the Concealed Cleat builders) hand-rolls its own local `point(x,y,z)` isometric-projection closure with its own inline constants, and the dice/divider tray builders use a third, unrelated flat top-down scheme. This is not a style preference — it is the exact root cause of the Gift Box G1 defect (a lid built as `[x,y]` array pairs rather than `{x,y}` objects, producing literal `NaN` coordinates, invisible in both Closed and Open states, with no rotation ever applied between them). Recommend:

- **One shared projection helper**, parameterized once (origin, scale, depth/height projection factors), used by every new product's Finished View — not reinvented per product.
- **One shared shell renderer** that walks the wall-loop descriptor (§3) and draws however many walls the shape has, generically — not a hard-coded front/back/left/right polygon set.
- **One shared lid-state renderer** that takes a lid descriptor plus a `state` (`closed`/`open`/`removed`) and, for hinged strategies, applies an **actual rotation transform** about the hinge axis — never a mere reposition-and-rescale standing in for rotation, and for lift-off strategies, a clearly distinct "set aside" representation rather than a hinge pretense. This bakes the Gift Box audit's most important lesson directly into the shared engine's contract, so it cannot be silently skipped by a future product author under deadline pressure.
- **A coordinate-sanity check built into the shared point-conversion step itself**, so a panel-points shape mismatch (array vs. object) is caught structurally rather than only by a fixture author remembering to check for `NaN` after the fact.

## 10. SVG and production-panel layout

The existing `layoutDesignPanelRows(panels, rowIds, margin, gap)` and `serializeDesignSvg(result)` are already generic (any panel with `{id,width,height}`; `scoreGroupId` lets each product name its own guide-mark layer without touching the serializer) and should be reused directly, unmodified, by the new family — no new layout engine is needed. The one improvement worth making for the new family specifically: **consolidate the 2-3 currently-duplicated serializer variants** (the plain `serializeDesignSvg`, the Concealed-Cleat-specific serializer, the Tray-compatibility serializer) into the single generic one for every new product, since nothing about a rectangle-vs-hexagon tray requires a bespoke serializer — this is a reuse decision, not new engine work.

## 11. Validation layers

Recommend explicit layering, formalizing a separation that is already informally present:

1. **Geometry validation** (shape-agnostic, reuse as-is): closed/orthogonal contours, no self-intersection, no duplicate cut segments, bounds containment — the existing `designPanelGeometryErrors` already generalizes to hexagon/polygon panels, since it only inspects point lists.
2. **Manufacturing validation** (shared, currently *not* centralized — worth creating once for the new family): minimum web thickness, minimum finger width vs. material thickness, captured-floor rebate depth vs. material thickness. Today every template inlines its own magic-number thresholds; the new engine should have one shared `tabletopManufacturingLimits(thicknessMm)`-style helper instead of repeating them per product.
3. **Product validation** (legitimately per-product, should stay per-product): e.g., a hex tray's own hinge-count rule, or a deck-box-specific overhang limit. Centralizing these would be over-engineering; they are genuine product decisions.
4. **Layout validation** (shape-agnostic, reuse as-is): the existing `layoutDesignPanelRows` overlap/gap checks and `designSvgValidation`.
5. **Material validation**: a shared thickness-consistency *checker* (declared vs. actual) can be centralized, but *which* panels are required to share one thickness remains a per-product decision.

## 12. Fixtures

Extend the already-established pattern rather than inventing a new one:

- **One shared engine fixture group** (e.g. a `run…EngineFixtures()`-style standalone group) testing the shared primitives in isolation and independent of any product's UI: wall-loop construction for both rectangle and hexagon outlines, finished-cavity math, captured-floor rebate math, divider-length-derives-from-finished-cavity (not nominal), insert bounding-region derivation, and — critically, learning directly from the Gift Box audit's fixture-quality gap — **coordinate-level assertions on the shared Finished-View projection helper** (no `NaN`, non-degenerate bounding boxes, an actual angle-based transform present for hinged lid states), not merely string/attribute presence checks.
- **One small, standalone fixture group per new product**, each asserting only that product's own field defaults, product-specific validation, and result-summary contents, and each explicitly re-invoking the shared engine group as a regression guard at the end — exactly as `runDiceTraySystemFixtures` already does for `runTrayModelFixtures` today. This keeps each product's fixture count in the tens, not the hundreds, directly satisfying "avoid monolithic generators with thousands of assertions."
- Register each new group under its own `?selftest=` value, following the existing dispatcher pattern, rather than folding anything into the legacy `runDesignGeometryFixtures`.

## 13. Future-product fit evaluation

Evaluating fit, not designing the products:

- **Hex tray, dice vault, token trays, component organizers** — fit cleanly: a shell-outline generator (hex or rectangle) plus the existing divider/insert primitives cover all of these with no new engine concept required. These are the strongest validation candidates for the architecture.
- **Deck box** — fits cleanly: a rectangle or hex shell with a lid (reusing the existing lift-off/hinged vocabulary) and a divider system for card compartments; no new engine concept required.
- **Miniature storage** — fits cleanly via the divider/insert system; foam is simply an insert with `physical` true or false and a `materialTag` of `"foam"`, not new code.
- **Campaign storage** — fit is **less obvious**: this plausibly implies a large, compound, possibly multi-sheet product, which touches concerns (large-format sheet-fit, multi-sheet arrangement) explicitly out of scope for this phase and for the app's current Designs system generally. Flagged as an open question for a later phase, not solved here.
- **Initiative tracker** — fit is **the least obvious of the list**: this is plausibly a flat reference/tracking item, not a container at all, and may belong with the existing flat-panel template family (alongside QR stands/hanging signs) rather than the Tabletop Accessories container family. Flagged as a scope question rather than force-fit into this architecture.

## 14. Non-goals confirmed

Consistent with the request: this review recommends **no implementation**, **no changes to any existing generator** (Dice Tray, Finger Box, Sliding-Lid, Drawer Cabinet, Gift Box, coupons, and cleat prototypes are all untouched by this proposal), **no schema changes**, **no storage changes**, and **no network/cloud features**. The legacy Dice Tray's wall-to-base-tab construction remains exactly as committed and audited; the new family is additive and parallel, never a replacement.

## 15. Technical risks

- **Extraction risk to legacy Finished Views.** The new shared projection helper must be built *only* for the new family. Do not retrofit it onto Finger Box/Sliding-Lid/Gift Box's existing (bespoke, already-shipped) Finished Views in the same phase — that would reopen already-audited, already-fixed surfaces for no immediate product benefit.
- **Incomplete generalization risk.** The wall-loop abstraction is a real cognitive shift from "front/back/left/right" thinking. The concrete failure mode to watch for: a "generalized" engine that still secretly assumes exactly four edges somewhere (e.g., in the Finished View shell renderer, or in how corner descriptors are consumed). The pentagon/L-shape acceptance test in §4 is the guard against this.
- **Registry scatter, unresolved.** Without the lightweight product registry proposed in §2, every new product continues touching four hand-maintained lists, and that risk compounds with each of the ten planned products.
- **Fixture drift.** Without discipline, a "shared" engine fixture can quietly become product-specific again (a product author tweaks a shared helper to fix their product, silently breaking an assumption a different product relied on). The per-product "re-invoke the shared group" pattern (§12) is the existing, proven mitigation.

## 16. Geometry risks

- **Non-90° corner joints are genuinely harder than they look.** Finger-joint interleaving is fundamentally a perpendicular-angle concept; it does not generalize to 120° corners by simply changing a parameter. The captured-floor-primary strategy (§6) sidesteps this for squareness, but the exact wall-to-wall corner treatment at non-90° angles (glued butt, beveled miter, or a small alignment tab) is an open physical question, not an architecture question, and must not be decided by assumption.
- **Kerf/clearance compounding.** As captured-floor rebate, divider slot, and insert clearance stack, the gap between requested interior and finished cavity can grow non-obviously large. The architecture must *surface* this compounding in the result summary every time (§5), not just compute it correctly internally — computing it correctly but hiding it would repeat the exact lesson that prompted this review.
- **The captured-floor squareness hypothesis is unproven.** It is the architectural centerpiece of the new construction philosophy, but it is a hypothesis about physical assembly behavior, not a demonstrated fact. It must clear a physical-prototype checkpoint before being treated as the premium default (§17, checkpoint 1).

## 17. Physical-prototype checkpoints — where testing must occur before more products are built

1. **Before any product is built on the new engine:** cut a scrap captured-floor + finger-jointed-wall rectangle shell (no lid, no dividers) and physically compare its squareness against the known weakness of the legacy wall-to-base tray. This is the single most important checkpoint — if captured floor does not measurably improve squareness in hand, the construction-philosophy premise needs revisiting before anything ships on it.
2. **Before hexagon ships:** physically test the chosen non-90° corner-join strategy on scrap material with an actual laser-cut hexagon shell, confirming it is practical to assemble by hand.
3. **Before divider/insert ship on the new engine:** confirm a divider cut to *finished cavity* length (not nominal) actually fits with the intended clearance in a real assembled shell — this directly closes the loop on the lesson that motivated this review.
4. **Before any hinged-lid Finished View ships on the new engine:** re-verify, against an actual opened physical prototype, that the rendered "open" rotation looks plausible — not merely that the SVG is non-`NaN` and differs from the closed state (the exact bar the Gift Box fixture suite mistakenly treated as sufficient).
5. **Only after checkpoints 1-4 pass** should a second or third product (deck box, dice vault) be built on the shared engine — discovering an engine-level defect after several products already depend on it is far more expensive than discovering it once, early, on the first product.

## 18. Recommended first implementation phases (for future planning; not to be built now)

- **Phase T1 — shared engine only, no user-facing template.** Rectangle shell-outline generator, wall-loop/corner descriptors, captured-floor descriptor and rebate math, the authoritative finished-cavity computation, per-edge composition of the existing `buildFingerPattern`. A standalone engine fixture group, tested via a synthetic harness, with no product UI yet.
- **Phase T2 — first real product on the engine.** A new, distinctly-named rectangular tray product (explicitly not replacing or renaming legacy `dice-tray`), exercising divider/insert usage and a new Finished View built on the shared projection helper. This is the architecture's proof-of-concept and the target for physical checkpoints 1 and 3.
- **Phase T3 — hexagon.** Add the hexagon shell-outline generator and a non-90°-corner-join strategy to the *same* engine; validate via a hex product. This is the concrete test of the §4 acceptance criterion and the target for physical checkpoint 2.
- **Phase T4+ — subsequent products.** Each additional product (deck box, dice vault, etc.) should require only its own normalize/dispatch/form/fixture entry plus a registry line — zero engine changes. If any of them requires an engine change, that is itself useful signal about where the engine's generalization was incomplete.

**Shared helpers to build first (Phase T1), in priority order:** (1) shell-outline generator for rectangle, (2) wall-loop/corner-descriptor derivation, (3) per-edge finger-pattern composition (wraps existing `buildFingerPattern`, does not reimplement it), (4) captured-floor descriptor and rebate math, (5) the authoritative finished-cavity computation, (6) divider length-from-finished-cavity primitive, (7) insert bounding-region/physical-flag primitive, (8) the one shared Finished-View projection helper, (9) the shared shell + lid-state renderer with real rotation, (10) a shared manufacturing-limits helper.

## 19. Final answer

**If this project eventually grows into the best offline tabletop-accessory generator for hobby laser users, the architecture that gives it the best chance of remaining maintainable five years from now is: one small, shared geometry engine built around an ordered wall-loop (not named sides), with an explicit and always-surfaced distinction between requested interior and finished cavity, a captured-floor-first construction philosophy validated by physical prototyping rather than assumed, and — learned directly and specifically from this codebase's own Gift Box experience — exactly one shared, coordinate-sanity-checked Finished View projection helper that every product reuses instead of reinventing.** Each individual product then remains its own small, independently registered, independently fixtured template — cheap to add, cheap to test, and safe to get wrong in isolation without threatening the others. The two failure modes to actively avoid, both demonstrated by this exact codebase's own history, are: (1) collapsing many products into one generator's internal "modes" (the Gift Box lift-off/hinged pattern, which already produced a real cross-mode bug), and (2) letting the one truly shared, highest-leverage piece — the Finished View — remain unshared and hand-rolled per product (the exact root cause of that same bug's most visible symptom).
