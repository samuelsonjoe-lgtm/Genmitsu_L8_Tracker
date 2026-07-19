# Finger Box Finished View — Architecture & UX Design Review

**Date:** 2026-07-18  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Scope:** Read-only design for a safer, bounded upgrade of the Finger Box screen-only Finished View  
**Baseline claimed:** `0ce2f5b` — *Add concealed cleat full-box prototype*  
**Verified suite (user-stated, not re-run in this review):** Tray-model 264/0 · Designs geometry 1,069/0 · Complete suite 1,861/0  

---

## 0. Repository state (before analysis)

| Check | Result |
| --- | --- |
| **Actual HEAD** | `0ce2f5bb8f6dad66ee0fd013514a5f993598a26c` (`0ce2f5b`) |
| **Subject** | Add concealed cleat full-box prototype |
| **Branch** | `main` |
| **Sync with `origin/main`** | `0 0` (in sync; no ahead/behind) |
| **Tracked working tree** | Clean — no modified or staged tracked files |
| **Staging area** | Empty |
| **Untracked (preserve; do not touch)** | `LightBurn Projects/`, `debug.log`, many historical `docs/*` review/audit files, `parametric_qr_stand_generator.py` |

**Constraint honored:** This review did not edit, stage, commit, push, reset, clean, stash, move, rename, or delete any product file. Only this report file is the intended deliverable under `docs/`.

---

## 1. Reviewed files and functions

### Core product
- `index.html` — Designs Finger Box pipeline, Finished View helpers, preview dispatch, fixtures
- `README.md` — Designs / Finished View documentation (lines ~21–33, ~84–86)

### Finger Box contract
| Symbol | Role |
| --- | --- |
| Form fields / `normalizeDesignDraft` (`template === 'finger-box'`) | Inputs and validation |
| `designBoxDimensions` | Inside ↔ outside conversion |
| `buildFingerPattern` | Odd segment count, boundaries, actual width |
| `designPatternEdge` / `buildFingerPanel` | Semantic joint edges |
| `buildBoxModel` | Panels, patterns, dimensions, lid, closed height |
| `layoutDesignPanels` | Production panel ordering (`bottom|lid`, `front|back`, `left|right`) |
| `fingerBoxCornerDecorationSegments` | Faux-dovetail blue score marks (production only) |
| `buildBoxDesignResult` | Layout + decoration + labels + `serializeDesignSvg` |
| `serializeDesignSvg` | Production red cut (+ optional blue score) |

### Finished View / preview architecture
| Symbol | Role |
| --- | --- |
| `buildFingerBoxFinishedViewSvg` | **Existing** Finger Box Finished View (orthographic open-top) |
| `buildSlidingLidFinishedViewSvg` | Isometric-style precedent (closed/open lid states) |
| `buildDrawerCabinetFrontElevationSvg` | Elevation Finished Front View |
| `buildDiceTrayFinishedViewSvg` / `buildDividerTrayFinishedViewSvg` | Orthographic semantic tray previews |
| `buildConcealedCleatFinishedViewSvg` / `buildConcealedCleatFullBoxFinishedViewSvg` | Semantic schematic previews |
| `designPreviewMode` (session `let`, default `'cut-layout'`) | Session-only preview mode |
| `designPreviewModeForTemplate` | Template-scoped mode allow-list |
| `designPreviewSelectorHtml` | Cut Layout / Finished View buttons |
| `designResultsHtml` | Dispatch + metrics + assembly copy |
| `bindDesignPreviewActions` / `refreshDesignPreview` | Click handlers; rebuild UI only |
| `downloadCurrentDesignSvg` | Always production Cut Layout bytes |

### Fixtures (existing Finger Box Finished View coverage)
- Determinism, orientation labels, open-top vs loose lid, outside dimensions, wall thickness display, invalid reset, production absence, decoration independence, selector / screen-only markup (~4525–4553, ~4072)

---

## 2. Current Finger Box semantic and production contract

### 2.1 User inputs (session-only draft fields)

| UI field | Normalized key | Contract |
| --- | --- | --- |
| Dimension basis | `dimensionMode` | `inside` \| `outside` |
| Width / depth / height | `width`, `depth`, `height` | Positive mm; mode-dependent meaning |
| Material thickness | `materialThickness` | Measured stock thickness |
| Joint clearance | `clearance` | Signed; −0.10 … positive; not kerf |
| Preferred finger width | `preferredFingerWidth` | Drives odd segment count selection |
| Top | `lid` | `open` \| `loose` only |
| Corner decoration | `cornerDecoration` | `none` \| `faux-dovetail-engrave` |
| Assembly labels | `assemblyLabels` | Optional blue vector labels on production SVG |

**Not Finger Box:** sliding rails, hinges, grooves, magnets, Drawer Cabinet shelves, tray base slots.

### 2.2 Outside vs inside dimensions

```text
inside  → outsideWidth  = width + 2t
          outsideDepth  = depth + 2t
          outsideHeight = height + t   (bottom thickness in height stack)

outside → insideWidth   = width − 2t
          insideDepth   = depth − 2t
          insideHeight  = height − t
```

Assembled **outside body** proportions for preview **must** use `metrics.dimensions.outsideWidth|outsideDepth|outsideHeight`. Do not substitute inside values for the assembled envelope.

### 2.3 Joint parameters and fit

- Three independent patterns on the semantic model: `patterns.width`, `patterns.depth`, `patterns.vertical`
- Each pattern: `{ length, count, actualWidth, minimumWidth, boundaries[] }` with **odd** `count ≥ 3`
- Clearance adjusts **production cut transitions only**; it does not change outside envelopes
- Kerf is **never** baked into Designs SVG

### 2.4 Lid modes (production)

| Mode | Panels | Geometry notes |
| --- | --- | --- |
| `open` | 5: bottom, front, back, left, right | No lid piece |
| `loose` | 6: + plain rectangular `lid` (no finger edges) | Warning: no retention, hinge, rail, magnet, or locating feature |
| Metrics | `closedHeight = outsideHeight + (loose ? t : 0)` | Stack height if lid merely rests on top |

### 2.5 Bottom relationship

- Bottom is a full finger-jointed panel sized to **outside** W × D
- Walls sit on bottom thickness; outside height includes bottom in the `inside` mode formula

### 2.6 Faux-dovetail option (production)

- Blue **score** subgroup `corner-decoration` only
- Marks derived from **vertical finger intervals** on front/back/left/right left+right edges
- Explicitly decorative; red cut group byte-identical to undecorated
- Engraved faces intended **outward**
- Must **not** be read back from production SVG for Finished View

### 2.7 Panel ownership, ordering, labels, layers

| Concern | Contract |
| --- | --- |
| IDs | `bottom`, `front`, `back`, `left`, `right`, optional `lid` |
| Layout order | `bottom|lid` row, then `front|back`, then `left|right` |
| Assembly labels | Optional; score-before-cut; path glyphs; may share face with decoration |
| Production layers | Blue score (optional decoration/labels) then red cut |
| Download | Single production SVG; Finished View never included |

### 2.8 Values already on the semantic result vs SVG-only reconstruction

| Available on `result` / `metrics` without parsing SVG | Reconstructed only during production serialization |
| --- | --- |
| `valid`, `errors`, `warnings` | Exact panel `path` strings / point chains for cut |
| `metrics.dimensions` (inside + outside) | Exact clearance-shifted finger transitions |
| `metrics.patterns` (count, boundaries, actualWidth) | Exact score mark segment coordinates |
| `metrics.materialThickness`, `boxLid`, `closedHeight` | Layout x/y of panels on sheet |
| `metrics.cornerDecoration`, mark counts, constants | Serialized `result.svg` bytes |
| `panels[]` with edges/phases (if needed) | DOM path order |
| `scorePaths` / `labelPaths` (production artifacts — **do not require** for preview) | |

**Preview rule:** consume dimensions, patterns, lid, thickness, cornerDecoration flag only. Optionally read pattern `count` / phase conventions from `panels` edges **without** using path strings or SVG text.

---

## 3. Existing Finished View (baseline to upgrade)

`buildFingerBoxFinishedViewSvg(result)` already exists and is wired:

- Mode: `finished-view` via `designPreviewModeForTemplate('finger-box', …)`
- UI: Cut Layout | Finished View
- Geometry: **top-down orthographic** open box — four wall bands + cavity
- Lid: if `boxLid === 'loose'`, a **separate** flat rectangle beside the box labeled “Loose lid”
- Inputs: outside W/D/H + thickness + lid only
- Accessibility: `role="img"`, `aria-label`, `<title>`, `<desc>` screen-only language
- Gaps vs preferred product:
  - No isometric front + side + top
  - **No visible finger-joint pattern**
  - **No faux-dovetail surface cue** when enabled
  - Wall thickness clamped with `min(t, width/3, depth/3)` for schematic readability (acceptable for orthographic; isometric should still use true `t` for proportion, with stroke legibility rules)
  - Height shown only as text callout, not as extruded walls

README already documents this orthographic behavior (~line 84). Implementation is an **upgrade of the existing helper**, not a new template or new storage key.

---

## 4. Selected preview scope (first version)

### Verdict on scope

**Bounded v1: one assembled isometric (cabinet) view only.**

Show:

1. Front face (near)
2. One side face (right recommended; matches sliding-lid depth cue)
3. Top face: open cavity (open) or lid surface (loose)
4. Simplified finger-joint accents on **visible structural edges only**
5. Overall outside proportions from `metrics.dimensions`
6. Lid type callout
7. When `cornerDecoration === 'faux-dovetail-engrave'`, a **decorative** surface treatment on exterior vertical corners (not structural)

### Explicitly deferred (not in v1)

| Mode | Decision | Why |
| --- | --- | --- |
| Assembled + Exploded toggle | **Defer** | Second layout system; risk of false assembly order claims |
| Transparent / ghosted lid | **Defer** | Extra state; open-top already clear for `open` |
| Removable lid “hover” above box | **Defer** | Easy to imply fit/retention; current separate panel is safer |
| Open-top alternate camera | **Defer** | Redundant with assembled isometric for open |
| Rotate view | **Defer** | New session state; interaction cost |
| Show dimensions toggle | **Defer** | Always-on outside callouts are enough |

**Inexpensive clean additions allowed in v1 only if they add zero persistent state:** static outside dimension callouts, “Screen-only” footer, “Decorative only” note when faux dovetails on.

---

## 5. Lid-mode mapping

Production creates only **open** and **loose flat lid**. Do not invent hinges, rabbets, grooves, magnets, or sliding channels.

| Mode | Installed? | Raised? | Top open? | Overlap / flush | Dimensions | Avoid implying |
| --- | --- | --- | --- | --- | --- | --- |
| **open** | No lid | N/A | Yes — cavity visible | N/A | Body outside W×D×H | Any closure hardware |
| **loose** | **Not** installed on box in v1 | No | Body still open-top in assembled massing | Lid is **separate flat panel** at outside W×D, thickness `t` only as note | Lid = outside W × outside D; `closedHeight` as text only | Hinge, rail, friction fit, nested lip, locating pins |

**Rationale:** Production lid is a plain rectangle with **no** joint or retention geometry. Showing it “closed flush” would invent a seating mechanism. The **existing separate-panel convention is correct** and should be retained in isometric v1 (place lid to the side or behind the isometric massing with the same honesty as today).

Optional later (not v1): a second **session-only** “lid resting” schematic with explicit caption “resting only — no retention geometry,” still without hardware.

---

## 6. Finger-joint visualization

### Goal
Legible structural joinery cue that reflects **real segment counts**, not production-path fidelity.

### Recommendation: **simplified alternating blocks from semantic patterns**

| Source | Use |
| --- | --- |
| `patterns.width.count` / `actualWidth` | Front horizontal bottom edge + top of front (if needed) |
| `patterns.depth.count` | Right-side horizontal bottom edge |
| `patterns.vertical.count` | Visible vertical corners between front/side and side/back |

**Draw:**

- Alternating inset/outset **schematic teeth** along visible mating edges only
- Phase-agnostic or fixed schematic phase is acceptable if labeled “approximate pattern”
- Prefer using `count` and equal spacing `length/count` rather than clearance-shifted boundaries (clearance is sub-pixel at screen scale and production-specific)

**Do not draw:**

- True production polyline from `panel.path`
- Joints on hidden rear edges that cannot be seen in the chosen camera
- False fingers on the loose lid (plain edges only)
- Millimeter-perfect kerf or clearance gaps

**Avoid:** edge-accent-only (too weak) and full path replay (too coupled to production).

---

## 7. Faux-dovetail visualization

| Rule | Detail |
| --- | --- |
| Trigger | `metrics.cornerDecoration === 'faux-dovetail-engrave'` only |
| Semantic only | Boolean + optional mark count in metrics; **never** parse `scorePaths` or production SVG |
| Appearance | Light hatch, chevron band, or dashed trapezoid **icons** on exterior vertical corner bands of **front** and **right** faces |
| Copy | Caption: “Faux dovetail engraving — decorative only; structural joint is finger” |
| Not structural | Do not replace finger teeth with dovetail silhouettes |
| Detail level | Coarse (3–8 marks max per edge), not microscopic engraver paths |
| Disabled | No decoration marks when `none` |

Existing fixture already asserts Finished View does not embed production decoration path data (`Finished View stays production-independent with decoration enabled`). Keep that contract.

---

## 8. Dimensional integrity

| Quantity | Source | Preview use |
| --- | --- | --- |
| Assembled outside width | `dimensions.outsideWidth` | Front face width; W callout |
| Assembled outside depth | `dimensions.outsideDepth` | Depth projection; D callout |
| Assembled body height | `dimensions.outsideHeight` | Vertical extrusion |
| Lid contribution | `metrics.closedHeight` / `boxLid` | Text: closed stack = body + t when loose; do not silently grow isometric body by lid thickness unless lid is drawn on top (v1: do not) |
| Base placement | Bottom implied under walls | Show base as bottom face or ground edge; walls rise from z=0 |
| Wall thickness | `materialThickness` | True `t` for cavity inset; minimum stroke if `t` is tiny on screen |

**Projection:** Reuse sliding-lid style oblique factors (e.g. depthX ≈ 0.34, depthY ≈ 0.38, heightY ≈ 0.52) for visual consistency, applied to **outside** mm.

**Forbidden:** using inside dimensions as outside faces; using sheet layout width/height; clamping that changes aspect ratio of W:D:H beyond uniform scale.

---

## 9. Preview architecture

### Preferred pattern

```text
buildDesignResult(draft)
        │
        ├─► result.svg  ──────────────────► Cut Layout / download (unchanged)
        │
        └─► metrics + valid ──► buildFingerBoxFinishedViewSvg(result)
                                      │
                                      └─► screen-only SVG string (no write-back)
```

| Requirement | Decision |
| --- | --- |
| Dedicated helper | **Yes** — keep / rewrite body of `buildFingerBoxFinishedViewSvg` only |
| Semantic input only | **Yes** |
| Parse `result.svg` | **No** |
| Mutate `result.svg` | **No** |
| Path order dependence | **No** |
| Deterministic markup | **Yes** — fixed face order, fixed IDs, `designRound` / `designNumberText` |
| Dispatch | Existing `finished-view` path in `designResultsHtml` |

### Optional small semantic extension

**Not required** if patterns + dimensions + lid + decoration flag suffice.

If implementers want cleaner tests, optional **internal-only** field is acceptable:

```js
// Optional; not storage; not export; not production
metrics.fingerFinishedView = {
  outsideWidth, outsideDepth, outsideHeight, thickness,
  lid: 'open'|'loose',
  fingerCounts: { width, depth, vertical },
  cornerDecoration: 'none'|'faux-dovetail-engrave',
  closedHeight
};
```

Populate inside `buildBoxDesignResult` from already-computed model values. Do **not** change public backup/export structure, `STORAGE_KEY`, or schema version.

### Shared-helper risk

Do **not** generalize into a shared “isometric box” used by Sliding Lid / trays / cleats in this phase. Finger-local helper only. Sliding Lid remains the **style reference**, not a shared dependency.

---

## 10. Visual style

Align with Sliding Lid / tray Finished Views:

| Element | Recommendation |
| --- | --- |
| Faces | Distinct fills: front slightly darker, side mid, top/cavity light |
| Projection | Modest isometric/oblique; no WebGL |
| Edges | Darker strokes; higher contrast than fills |
| Shading | Flat + optional one-step darker side face |
| Dependencies | None beyond existing SVG string + `esc` |
| Labels | Front / optional Right; “Open top” or “Loose lid (separate)” |
| Dimensions | Outside W × D × H mm callouts |
| Screen-only | Title + desc + muted footer (existing pattern) |
| Ownership labels | Optional FRONT on front face only if space; avoid clutter |
| Front arrow | Optional small “Front” text is enough; no assembly animation |

Decorative vs structural: **shape + caption**, never color alone (fingers = block teeth; decoration = hatch + text).

---

## 11. Controls: now vs deferred

| Control | Now? | Notes |
| --- | --- | --- |
| Cut Layout / Finished View | **Keep** (existing) | Session `designPreviewMode` only |
| Assembled / Exploded | Defer | |
| Show / hide lid | Defer | Loose already separate |
| Transparent lid | Defer | |
| Rotate | Defer | |
| Show dimensions | Defer | Always show outside summary |

**No new persistent state.** No localStorage key. No schema field. Any future control must remain session-only like `designPreviewMode`.

---

## 12. Production-byte neutrality contract

| Assertion | How to prove |
| --- | --- |
| Production SVG byte-identical before/after preview generation | Capture `result.svg` hash/length; call `buildFingerBoxFinishedViewSvg(result)` N times; assert `result.svg` unchanged |
| Opening/closing preview does not rebuild production | UI path only sets `designPreviewMode` and re-renders HTML; download still uses `buildDesignResult(...).svg` |
| Faux-dovetail production bytes unchanged | Golden pin remains `7269` / `d9512d1c` (or current HEAD golden if re-pinned later for unrelated reasons — **this feature must not change it**) |
| Lid-mode goldens unchanged | Open and loose production goldens unchanged |
| Preview determinism | Same result object → identical Finished View SVG string twice |
| Download uses production artifact | Live preview/download fixture: downloaded === production svg; filename `l8-finger-box-…` |
| No production SVG text from Finished View | Production remains path-only labels/score; Finished View text must not appear in `result.svg` |
| No SVG text regression in production | Existing “no text in cut” / score-path fixtures remain green |

---

## 13. Storage and schema conclusion

| Item | Conclusion |
| --- | --- |
| New storage key | **No** |
| Schema version change | **No** |
| Backup / import / export field | **No** |
| Saved preview preference | **No** (continue session-only `designPreviewMode`) |
| Behavioral storage-isolation fixture | Keep existing pattern: snapshot `localStorage` + `backupObject()` around preview generation and mode toggles; assert identity |

---

## 14. Existing-template protection

| Template / surface | Risk if shared code mutates | Mitigation |
| --- | --- | --- |
| Sliding-Lid Box | Isometric helper collision | Do not refactor sliding helper |
| Drawer Cabinet | Finished Front dispatch | Leave `finished-front` branch alone |
| Divider / Dice Tray | Orthographic finished views | Separate helpers |
| Concealed cleat prototypes | Schematic helpers | Separate helpers |
| Coupons | N/A | No touch |
| `serializeDesignSvg` | Production bytes | Never call from Finished View |
| Generic preview renderer | Over-broad mode | Keep finger branch finger-only |
| Designs result cards | Metrics HTML | Only update Finger assembly copy if needed |

**Prefer Finger Box-local implementation** confined to `buildFingerBoxFinishedViewSvg` (+ optional metrics mirror + fixture/docs).

---

## 15. Accessibility and usability

| Concern | Plan |
| --- | --- |
| Title | `Finished View - Finger-jointed box` (keep/extend) |
| aria-label / desc | Describe open vs loose, decorative-only faux if on, screen-only, not cut file |
| Toggles | Existing preview buttons already keyboard-focusable; no new toggles in v1 |
| Contrast | Dark labels on light fills; stroke ≥ current tray style |
| Color-only | Forbidden for structural vs decorative distinction |
| Narrow viewport | `viewBox` + width 100% / height auto like other previews; wrap under results column |
| Failure | Return `''`; selector disables Finished View when invalid; Cut Layout / download unaffected for valid production |

---

## 16. Fixture plan

Independent assertions (extend existing Finger Box Finished View block; avoid “helper returned a string” alone):

1. **Dispatch** — `designPreviewModeForTemplate('finger-box','finished-view') === 'finished-view'`; other templates ignore
2. **Default assembled preview exists** for valid open-top defaults
3. **Outside proportions** — front width attribute / projected span equals `outsideWidth` (within projection math)
4. **Lid-mode** — open: no loose-lid group; loose: separate lid group, no hinge/rail/retention IDs
5. **Open-top** — cavity / open-top label present for open
6. **Finger pattern** — visible finger markers only on structural edges; count matches `patterns.*.count` (or documented schematic rule tied to count)
7. **Faux-dovetail when enabled** — decorative markers present + “decorative” text; **absent** when disabled
8. **Decorative not structural** — finger teeth still present under decoration mode
9. **Semantic-only** — building preview does not require `result.svg` non-empty if dimensions present (or explicitly only when valid with metrics)
10. **No result.svg parsing** — unit-style: empty `svg` with metrics still previews if valid geometry model allows; production path independent
11. **Production byte neutrality** — svg length/hash unchanged after N preview builds
12. **Repeated determinism** — identical strings
13. **No production text pollution** — finished class names / “Loose lid” not in production svg
14. **Storage isolation** — localStorage + backup bytes
15. **Finger Box goldens unchanged** (open default production)
16. **Faux-dovetail golden unchanged** (`7269`/`d9512d1c` at this HEAD)
17. **Non-Finger-Box goldens unchanged** (sliding, cabinet, trays smoke)
18. **Invalid geometry** — Finished View empty; button disabled; mode resets to cut-layout
19. **Preview failure ≠ block download** — valid production still downloadable when Finished View path throws/returns empty only for invalid cases; for valid, both work
20. **Regression** — orientation Front/Back/Left/Right still meaningful or Front labeled in isometric + callouts

---

## 17. Documentation recommendations

Update **README.md** Designs / Finished View paragraph (currently orthographic open-top description ~line 84) to state:

1. Finger Box **Finished View** is a **screen-only assembled isometric** preview of outside proportions, visible finger pattern, and lid type.
2. It is **not** in the downloaded cut file and does not change cut geometry.
3. **Faux dovetail** appears only as a decorative surface cue when Corner decoration is enabled; structural joint remains finger.
4. Proportions are **representative** of the generated design’s outside dimensions; they do not prove fit, strength, kerf, glue, or production safety.
5. Loose lid remains a **separate** flat panel with **no** retention mechanism implied.

Optional one-line in Designs form help near Top / Corner decoration pointing to Finished View for visualization only.

**Do not** claim physical strength or mil-perfect joint fidelity.

---

## 18. Non-goals (explicitly deferred)

- Interactive 3-D rotation  
- WebGL / canvas / external 3D libraries  
- Exploded assembly animation  
- Individual panel selection / highlight  
- Dimensional editing inside the preview  
- Material textures / photorealism  
- Saveable camera angles  
- Production SVG overlays in Finished View  
- Physical strength / fit prediction  
- New lid mechanisms  
- Shared isometric engine across templates  
- Storage of preview preferences  

---

## 19. Risks

| Risk | Severity | Mitigation |
| --- | --- | --- |
| Isometric implies closed/fitted loose lid | Medium | Keep lid separate; caption honesty |
| Finger teeth look production-exact | Medium | Label “approximate”; use count-based blocks |
| Faux dovetail looks structural | Medium | Hatch + explicit decorative caption |
| Accidental production golden drift | High | Byte-identity fixtures; no serialize path |
| Shared helper breaks Sliding Lid | High | Finger-local only |
| Wall thickness clamp distorts proportions | Low | Prefer true `t` with min stroke |
| Scope creep (explode/rotate) | Medium | Stick to assembled-only v1 |

---

## 20. Implementation sketch (for implementer; not executed here)

1. Rewrite `buildFingerBoxFinishedViewSvg` body to isometric massing using outside dims + thickness.  
2. Draw simplified finger blocks from `metrics.patterns`.  
3. Preserve open vs separate loose-lid behavior and accessibility nodes.  
4. Add decorative corner treatment when `cornerDecoration === 'faux-dovetail-engrave'`.  
5. Update assembly message + README only.  
6. Extend fixtures per §16; re-run Designs + complete suite.  
7. Confirm production goldens including faux-dovetail pin.

No storage, schema, import/export, or other-template changes.

---

## 21. Final verdict

### **READY FOR BOUNDED IMPLEMENTATION**

**Rationale:** Finger Box already has a production-neutral Finished View hook, semantic dimensions/patterns/lid/decoration flags, strong production fixtures, and a proven isometric precedent in Sliding Lid. The gap is UX fidelity (isometric + fingers + decorative faux), not architecture. Upgrading the local helper is the smallest safe path.

| Question | Answer |
| --- | --- |
| Can Codex implement directly after this review? | **Yes** — within the bounds above; no architecture redesign required |
| Is a focused Grok audit after implementation sufficient? | **Yes** — production-byte neutrality, goldens, lid honesty, decoration-as-decorative, storage isolation |
| Is Claude necessary? | **No** — unnecessary for this bounded upgrade |

---

*End of design review. No product code was modified for this document.*
