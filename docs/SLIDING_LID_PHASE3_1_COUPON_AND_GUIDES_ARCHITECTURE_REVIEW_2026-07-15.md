# Sliding-Lid Phase 3.1 Coupon and Rail-Guide Architecture Review

Date: 2026-07-15

Repository: `C:\Genmitsu L8 Tracker`
Scope: read-only architecture and geometry review; this report is the only authorized file addition

## 1. Executive recommendation

The sliding-fit coupon is bounded and suitable for a Phase 3.1 implementation. The rail-placement marks are also geometrically simple, but the current side-panel output does not yet prove the required inside-face convention for both asymmetric side panels. The shortened Front side edge is different from the full Back side edge, so a face-flip that makes a mark inward can also reverse Front and Back.

Recommend a short physical guide-mark prototype before implementing the two options together. The coupon can be implemented as a six-piece, two-rail captured-channel test using a full-width base/spacer; it should inherit the selected `t`, `Cs`, and `Cv`, and should not add persistence or automatic fit learning. The guide layer should use four short, concealed corner registration marks per side-panel rail footprint, not a full engraved rectangle.

The safe Phase 3.1 boundary is:

- optional blue score/guide layer, separate from red cut geometry;
- optional six-piece sliding coupon in a fifth layout row;
- no changes to the eight existing cut-piece positions when either option is enabled;
- no body-joint sample in the first checkbox;
- no text, arcs, hardware, alternate rail styles, kerf inheritance, or saved fit data;
- a physical face/orientation check before the guide option is called ready.

## 2. Baseline verification

The requested baseline was inspected.

- `HEAD`: `df54da0`
- `main`: synchronized with `HEAD`
- `origin/main`: synchronized with `HEAD`
- `origin/HEAD`: synchronized with `origin/main`
- Latest commit: `df54da0 Add sliding-lid box generator`
- Tracked working tree: clean
- Staging area: empty
- Existing unrelated untracked files: present and untouched
- `git diff --check`: no tracked changes or whitespace errors

The existing reports were read and reconciled with the live source:

- `docs/SLIDING_LID_BOX_ARCHITECTURE_REVIEW_2026-07-15.md`
- `docs/SLIDING_LID_BOX_PHASE3_IMPLEMENTATION_2026-07-15.md`
- `docs/SLIDING_LID_BOX_INDEPENDENT_AUDIT_2026-07-15.md`
- `docs/SLIDING_LID_BOX_SECOND_PASS_REVIEW_2026-07-15.md`

The current baseline reported by the implementation and second-pass reviews is:

```text
Designs: 265 passed / 0 failed
Complete suite: 797 passed / 0 failed
```

The live source is the `df54da0` sliding-lid implementation, not a hypothetical clean-room design. This review does not modify it.

## 3. Current Phase 3 architecture assessment

The current Designs flow remains a module-level session-only draft through:

```text
designDraft
  -> normalizeDesignDraft()
  -> buildDesignResult()
  -> buildSlidingLidBoxModel()
  -> layoutDesignPanelRows()
  -> serializeDesignSvg()
  -> designSvgValidation()
  -> designResultsHtml()/refreshDesignPreview()
```

The sliding model currently provides `slidingLidBoxDimensions()`, `buildSlidingLidRail()`, `buildSlidingLidPanel()`, `buildSlidingLidSidePanel()`, the composite shortened-Front edge, an eight-piece deterministic panel list, a shared four-row layout wrapper, actual path-bound and segment checks, a single red `g#cut` SVG layer, exact preview/download SVG identity, and no `designDraft` persistence or backup inclusion.

The current stable panel order is:

```text
bottom, sliding-lid, front-open, back, left, right, rail-left, rail-right
```

The current `left` and `right` panel paths are identical in the flat output. Local `x=0` is Front and local `x=Di` is Back. The side-panel Front edge is a shortened composite finger field; the Back edge uses the full vertical pattern. This asymmetry is the central constraint for guide-mark face handling.

The serializer emits one red cut group and serializes each placed panel as a translated `<g id="panel-...">` with a title and one path. There is no score/guide layer yet. `layoutDesignPanelRows()` stores panel geometry and nominal/path bounds separately, so score marks can remain outside panel cut objects and cut-layout calculations.

## 4. Recommended guide-mark style

### Compared options

**Full rail-footprint rectangle:** mathematically clear, but it scores a long outline that should be concealed by the rail. Misalignment can leave a visible perimeter, and the extra score length increases char and glue contamination.

**Dashed footprint rectangle:** reduces score time, but still risks visible marks and provides several visual fragments to interpret.

**Four short corner registration marks:** recommended. Four small L-shaped marks communicate the rail's top, bottom, Front, and Back footprint without tracing the glued area. They remain under the rail when placement is correct and make a small offset obvious during dry alignment.

**Bottom and Back reference lines only:** useful references but insufficient to establish the lower rail edge and ambiguous at the open Front channel.

**One central cross or pair of ticks:** too minimal; it does not communicate rail width and can be misread after rotation.

Use four short L marks per side-panel rail footprint. Do not use a complete rectangle, dashed outline, text, or hatch fill.

## 5. Exact guide-mark coordinates and orientation

For both current side panels:

- local origin is the top-Front corner;
- local `x=0` is Front;
- local `x=Di` is Back;
- local `y=0` is the top wall edge;
- local `y=Ho` is the bottom wall edge;
- rail footprint length is `Rl=Di`;
- rail footprint height is `Rh`;
- footprint bounds are `0 <= x <= Rl`, `0 <= y <= Rh`.

This follows from the existing placement: rail Back is flush with the inside Back plane, rail top is flush with the side-wall top, and rail bottom is `railBottom = Uh - L` above the inside floor. In panel-local SVG coordinates, the rail occupies the top `Rh` millimeters from Front to Back.

Use a small derived concealment inset rather than a second rail-placement constant:

```text
guideInset = max(1 mm, t/2)
guideLeg = min(6 mm, max(3 mm, t))
```

Block the guide option if the marks cannot fit:

```text
2 * guideInset + guideLeg < Rl
2 * guideInset + guideLeg < Rh
```

For the current 3 mm Golden A:

```text
Rl = 80 mm
Rh = 11.2 mm
guideInset = 1.5 mm
guideLeg = 3 mm
```

Let `g=guideInset` and `m=guideLeg`. Use these panel-local endpoints on each side panel:

```text
Front/Top:    (g,g)-(g+m,g), (g,g)-(g,g+m)
Back/Top:     (Rl-g,g)-(Rl-g-m,g), (Rl-g,g)-(Rl-g,g+m)
Front/Bottom: (g,Rh-g)-(g+m,Rh-g), (g,Rh-g)-(g,Rh-g-m)
Back/Bottom:  (Rl-g,Rh-g)-(Rl-g-m,Rh-g), (Rl-g,Rh-g)-(Rl-g,Rh-g-m)
```

For Golden A, these are:

```text
Front/Top:    (1.5,1.5)-(4.5,1.5), (1.5,1.5)-(1.5,4.5)
Back/Top:     (78.5,1.5)-(75.5,1.5), (78.5,1.5)-(78.5,4.5)
Front/Bottom: (1.5,9.7)-(4.5,9.7), (1.5,9.7)-(1.5,6.7)
Back/Bottom:  (78.5,9.7)-(75.5,9.7), (78.5,9.7)-(78.5,6.7)
```

Guide paths must be generated from the returned `dimensions`, attached to the `left`/`right` panel transforms, and excluded from `panel.points`, `panel.bounds`, cut segment lists, and cut piece counts. They remain strictly inside the footprint and side-panel bounds.

## 6. Inside-face and mirroring analysis

This is the blocking review issue. The current `left` and `right` cut paths are identical in the flat output, and the current model has no per-panel mirror or face metadata. Both panels have Front at `x=0`, Back at `x=Di`, and the shortened composite Front edge at the same local end.

For a symmetric panel, “mark the displayed face and assemble the marked faces inward” might be sufficient. These side panels are not fully symmetric: the shortened Front edge and full Back edge make orientation meaningful. Turning one identical panel over to expose its opposite physical face can also reverse Front and Back. A guide-only mirror could therefore make the mark mathematically plausible while putting the shortened edge or rail stop at the wrong physical end.

The following wording is not proven safe solely by source inspection:

```text
Engrave the displayed face.
Assemble both marked faces inward.
Keep the marked rail stop end toward Back.
```

The initial eight-piece physical test proves useful assembly evidence, but does not document which sheet face was cut up, which side was flipped, or how Front/Back was maintained while making both inner faces marked.

Before implementation, cut two temporary side-panel guide prototypes with temporary Front/Back orientation ticks outside production geometry. Verify that both marks can face inward, both shortened Front edges remain at Front, and both stop ends remain at Back. If that succeeds, production wording can be:

```text
Guide marks are on the displayed sheet face.
Keep the shortened Front edge toward Front and the rail stop toward Back.
Install the marked faces inward.
```

If it fails, the smallest safe correction is a documented mirrored physical Right-panel workflow or a per-face production design, not a silent guide-only mirror. The disabled-option cut SVG must remain byte-stable.

## 7. Recommended SVG layer structure

When guides are enabled, add a separate score group before the existing cut group:

```xml
<g id="score" fill="none" stroke="#0000ff" stroke-width="0.1">
  <g id="guide-rail-left" transform="translate(...)" ...>
    <path d="..."/>
  </g>
  <g id="guide-rail-right" transform="translate(...)" ...>
    <path d="..."/>
  </g>
</g>
<g id="cut" fill="none" stroke="#ff0000" stroke-width="0.1">
  ... existing eight panel groups ...
</g>
```

Use stable IDs `score`, `guide-rail-left`, and `guide-rail-right`. Blue is a distinct LightBurn operation cue from the existing red cut layer. The app must not embed score power, speed, mode, or device settings. The UI/README must instruct the user to assign safe low-power settings manually and verify them on the actual material.

Guide paths should use panel-local coordinates with the same transforms as the related cut groups. They must never join cut paths, enter panel bounds used for cutting, alter cut geometry, or expand the viewBox. With guides disabled, no score group is emitted and the existing sliding SVG remains byte-identical. Preview and download must contain the exact same two-layer SVG when enabled.

## 8. Fit-coupon construction comparison

### One-sided rail gauge

A short wall strip, one rail, and one lid strip is compact and easy to cut, but tests only one engagement face and does not test total two-sided lateral clearance. It is insufficient as the primary `Cs` coupon.

### Two-rail captured-channel coupon

Two short walls, two rails, a spacing/base piece, and a lid strip represent the actual construction. It tests both `Cs` and `Cv`, vertical capture, insertion, removal, and the Back stop. It has more pieces but can self-register using one full-width base/spacer.

This is recommended.

### Flat slot-and-tab gauge

It is fast but tests a different mechanism. A flat slot does not represent laminated rail capture and can give false confidence.

### Reduced-depth miniature box section

It is most representative but consumes more sheet and reintroduces body assembly complexity. Defer it as a later diagnostic.

## 9. Exact recommended coupon design

Use six cut pieces in a fifth row:

1. `coupon-base` — one full-width base/spacer plate.
2. `coupon-wall-left` — short left wall strip.
3. `coupon-wall-right` — short right wall strip.
4. `coupon-rail-left` — short C-channel rail.
5. `coupon-rail-right` — short C-channel rail.
6. `coupon-lid` — short shouldered/tongued sliding strip.

The coupon is a test fixture, not a miniature finished box. The base fixes wall spacing. The wall strips sit against its long outer edges and are glued. The rails are glued top-flush and Back-flush to the wall strips. The lid enters from Front and is captured by both channels.

Use a compact clear rail-face gap and preserve the production minimum run:

```text
Gc = 32 mm
Dc = max(40 mm, S + max(3t, 12 mm) + 5 mm)
B  = max(6 mm, 2t)
```

For current 3 mm stock, `Dc=40 mm`, giving a 36 mm open channel after the 4 mm stop.

The coupon dimensions are:

```text
Wi,c = Gc + 2t
Wb   = Wi,c + 2t = Gc + 4t
Hc   = B + Rh
```

The base is `Wb × Dc`. Each wall strip is `Dc × Hc` and physically has thickness `t`. Walls are placed at the base's two long outside edges, leaving `Wi,c` between inner wall faces. Rails consume `t` per side, leaving exactly `Gc` between rail inner faces.

The coupon is registered by the base's long edges and Back edge, not freehand spacing. Do not add tabs/slots in the first coupon: they would introduce a second fit variable and obscure whether a problem came from `Cs/Cv` or registration. Glue is required and must cure before testing.

## 10. Coupon formulas and piece list

Rails use `Rl,c=Dc` and:

```text
Rc,c = Dc - S
Ch,c = t + Cv
Rh,c = U + Ch,c + L
```

The lid uses:

```text
Lw,c = Wi,c - Cs = Gc + 2t - Cs
Tw,c = Gc - Cs
Ec   = (Lw,c - Gc) / 2 = t - Cs/2
lid length = Dc + t
shoulder end = t + Rc,c
center tongue end = t + Dc
```

For the default 3 mm / 0.20 mm / 0.20 mm values:

```text
Gc=32, Dc=40, Wi,c=38, Wb=44
B=6, Hc=17.2
Ch,c=3.2, Rh,c=11.2, Rc,c=36
Lw,c=37.8, Tw,c=31.8, Ec=2.9
lid length=43, shoulder end=39, tongue end=43
piece count=6
```

The coupon should not call the full-box model and crop its panels. It needs a small pure model with hand-derived fixture values. Block nonpositive dimensions, insufficient engagement, short channel, nonpositive web, out-of-bounds registration, and invalid paths.

## 11. Optional body-joint sample recommendation

Use one checkbox named `Include fit coupon` and make it mean the sliding-fit coupon only. Do not add a separate body-joint checkbox in Phase 3.1.

The selected `Cj` should be displayed as body-joint context, not used to change the sliding coupon. The user's slightly loose body joints are important, but adding a finger-pair sample would add another piece category and another interpretation of the same checkbox. Review a separate body-joint sample later after the current material's body fit is measured.

## 12. Layout and stable-ID plan

Preserve the existing rows exactly:

```text
Row 1: bottom, sliding-lid
Row 2: front-open, back
Row 3: left, right
Row 4: rail-left, rail-right
```

Append only when the coupon is enabled:

```text
Row 5: coupon-base, coupon-wall-left, coupon-wall-right,
       coupon-rail-left, coupon-rail-right, coupon-lid
```

Rows 1–4, their positions, local geometry, and cut paths must remain unchanged. Coupon pieces appear only afterward with the existing 10 mm margin and 15 mm gap. A deterministic subrow is acceptable only if one row is too wide; it must remain inside the fifth-row coupon region and never move the first eight pieces.

Guide marks create no panels and do not affect layout or viewBox. Coupon cut paths expand the viewBox only when enabled.

Stable coupon IDs/titles:

| ID | Title |
|---|---|
| `coupon-base` | Coupon base / spacer |
| `coupon-wall-left` | Coupon left wall |
| `coupon-wall-right` | Coupon right wall |
| `coupon-rail-left` | Coupon left rail |
| `coupon-rail-right` | Coupon right rail |
| `coupon-lid` | Coupon sliding lid |

## 13. UI wording

Under `sliding-lid-box`, add two unchecked options:

```text
Include rail placement guide marks
Include fit coupon
```

Guide help:

```text
Adds low-power score marks inside the Left and Right wall rail footprints.
Assign a safe score/engrave operation in LightBurn and run it before cutting.
Verify the marked-face assembly convention on scrap first.
```

Coupon help:

```text
Adds a compact six-piece sliding-fit test using the current material thickness,
lid side clearance, and lid vertical clearance. Glue the rails and test it
before cutting the full box. It does not save or change Library settings.
```

Fixed result text should identify the guide layer as alignment-only and the coupon as a six-piece Front-insertion test requiring glue. Do not expose coupon dimensions, alternate gaps, guide inset, or score power in this phase. Do not expose unsupported directions or rail styles.

README must say that guide marks are low-power alignment references, score/cut operations are assigned manually in LightBurn, the coupon is a preflight physical test, success does not update Library or persistence, and clearances remain material-specific.

## 14. Blocking errors and warnings

Guide blocking errors:

- unknown option value;
- non-finite/nonpositive derived footprint, inset, or leg;
- inset/leg cannot fit in `Rl` and `Rh`;
- coordinate outside its side panel or rail footprint;
- mark touches/crosses a cut edge;
- Left/Right guide ID or transform mismatch;
- score geometry enters panel cut points or `g#cut`;
- unsupported/non-orthogonal/non-finite score path;
- unresolved production face/orientation convention.

Coupon blocking errors:

- unknown coupon mode;
- nonpositive `Gc`, `Dc`, `Wi,c`, `Hc`, `Ch,c`, `Rh,c`, `Rc,c`, `Lw,c`, `Tw,c`, or `Ec`;
- insufficient engagement or short channel;
- base/wall registration outside bounds;
- insufficient web;
- self-intersecting, zero-length, duplicate, non-orthogonal, or out-of-bounds path;
- translated layout overlap or duplicate cut segment;
- malformed SVG or invalid viewBox.

Warnings should cover zero/loose clearances, guide visibility if misaligned, glue/char changing fit, coupon cure requirement, the coupon's inability to prove full-length lid straightness, slightly loose body joints, unresolved face orientation, and the absence of automatic score power/speed.

## 15. Helper/function change plan

No implementation is performed in this review. A bounded future pass should:

- add session-only `slidingGuideMarks:false` and `slidingFitCoupon:false` defaults;
- conditionally render the two fields only for `sliding-lid-box`;
- validate checkbox values without changing existing templates;
- add pure `slidingGuideDimensions()`, `buildSlidingRailGuidePaths()`, and `validateSlidingGuidePaths()` helpers;
- add pure `slidingCouponDimensions()`, `buildSlidingCouponModel()`, and validation helpers;
- extend row IDs only when the coupon is enabled;
- serialize `score` before `cut` only when guides are enabled;
- keep score objects separate from cut panels and AABBs;
- preserve the disabled sliding SVG byte-for-byte;
- leave storage, backup, imports, and unrelated tabs unchanged.

The serializer should emit no score group for non-sliding templates or disabled guides. The coupon must not be implemented by cropping the full box model.

## 16. Fixture plan with hand-derived goldens

Retain all 265 Designs assertions, all 797 full-suite assertions, all six legacy signatures, and the disabled eight-piece sliding output.

### Guide Golden A

For `t=3`, `Rl=80`, `Rh=11.2`, `g=1.5`, and `m=3`, fixture the sixteen score segments using the exact endpoint table in section 5. Verify four L corners per side, footprint/boundary containment, no cut intersection, no score geometry in `g#cut`, unchanged eight panel positions, deterministic output, and one score group before one unchanged cut group.

### Coupon Golden A

For `Gc=32`, `Dc=40`, `t=3`, `Cs=.20`, `Cv=.20`, `U=L=S=4`, fixture:

```text
Wi,c=38, Wb=44, B=6, Hc=17.2
Ch,c=3.2, Rh,c=11.2, Rc,c=36
Lw,c=37.8, Tw,c=31.8, Ec=2.9
lid length=43, shoulder end=39, tongue end=43
piece count=6
```

Expected values must remain hand-derived rather than produced by the helper under test.

Required categories:

- disabled options produce byte-identical sliding SVG with no score group or fifth row;
- enabled guides add exactly two stable guide subgroups and do not change cut geometry;
- enabled coupon adds exactly six cut pieces and preserves rows 1–4;
- `Ch,c=t+Cv`, `Ec=t-Cs/2`, and `Rc,c=Dc-S` are independently checked;
- changing `Cs` does not change channel height, changing `Cv` does not change lateral gap, and `Cj` does not change coupon geometry;
- coupon insertion/removal, capture, stop, base registration, and spacing are fixture-tested;
- all paths are closed, finite, orthogonal, non-self-intersecting, separated, and duplicate-free;
- score group contains only guides, cut stays red, no automatic LightBurn settings exist, no text enters cut geometry, and storage remains unchanged.

## 17. Storage and integration boundaries

Do not change `STORAGE_KEY`, `SCHEMA_VERSION`, `freshState()`, `persist()`, `backupObject()`, merge import, or replace import. Do not add saved guide preferences, saved coupon results, Library/Project/Inventory/Pricing integration, automatic Library updates, automatic kerf/cut settings, network access, or external libraries.

Editing/exporting either option must remain session-only and absent from JSON backups.

## 18. Manual LightBurn and physical test plan

1. Generate a box with both options enabled.
2. Import into LightBurn and confirm separate blue score and red cut layers.
3. Assign a very light score setting manually and run score before cut.
4. Confirm the marks land on the intended inside faces with Front/Back correct.
5. Cut the six-piece coupon.
6. Glue walls and rails using the base as the spacing reference; keep glue out of channels.
7. After cure, insert/remove the lid from Front and test lateral/vertical play, capture, stop, and rail parallelism.
8. Adjust `Cs`/`Cv` if necessary without saving automatically.
9. Generate, cut, and assemble the full box; place rails over guide marks and confirm marks are concealed.

Do not claim a particular score power, speed, kerf, or production clearance without a material test.

## 19. Bounded implementation sequence

1. Resolve the Right/Left inside-face convention with a two-side physical guide prototype.
2. Freeze the disabled eight-piece SVG signature and positions.
3. Add session-only option fields and sliding-template UI.
4. Implement/fixture derived four-L guide paths without altering cut panels.
5. Add conditional score serialization while preserving disabled bytes.
6. Implement/fixture the six-piece base-registered coupon.
7. Append the fifth row only when enabled.
8. Add layer, guide, coupon, layout, mutation, and storage fixtures.
9. Update README with manual score/coupon limitations.
10. Run HTML/browser/self-test/storage validation and then perform physical guide/coupon tests.

Do not implement a mirrored guide workaround before the face/orientation test.

## 20. Final `git status -sb`

At review completion, the tracked state remains clean at `df54da0` with only pre-existing unrelated untracked files. The only authorized new file is this report:

```text
docs/SLIDING_LID_PHASE3_1_COUPON_AND_GUIDES_ARCHITECTURE_REVIEW_2026-07-15.md
```

No `index.html`, README, existing report, LightBurn project, or Python file was changed.

## 21. Change-safety confirmation

- Read-only source and geometry review only.
- No implementation was added.
- Nothing was staged, committed, or pushed.
- No stash, reset, clean, move, or delete operation was performed.

REQUIRES PHYSICAL GUIDE-MARK PROTOTYPE FIRST
