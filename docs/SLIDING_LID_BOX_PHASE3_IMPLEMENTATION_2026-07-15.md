# Sliding-Lid Box Phase 3 Implementation Report

Date: 2026-07-15

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `0a304d8` — `Add parametric finger-box generator`

## 1. Baseline verification

The requested baseline was inspected before finishing this pass:

- `HEAD`, `main`, `origin/main`, and `origin/HEAD` are at `0a304d8`.
- The tracked tree was not clean at the start of this pass: `index.html` already contained an uncommitted sliding-lid implementation.
- `README.md` was initially unchanged.
- Nothing was staged.
- Existing unrelated untracked files were preserved.
- No reset, clean, stash, move, delete, commit, or push operation was performed.

The existing uncommitted `index.html` implementation was preserved and verified against the attached Phase 3 brief. This pass added the README description and this report.

## 2. Initial repository state

Initial status included:

```text
 M index.html
?? .claude/
?? LightBurn Projects/
?? docs/* existing audit and review reports
?? parametric_qr_stand_generator.py
```

The untracked files were outside the requested implementation scope and were not edited. The existing sliding-lid source changes were not reset or replaced.

## 3. Changed files

Final scoped changes are:

- `index.html` — existing uncommitted Phase 3 implementation retained and verified.
- `README.md` — added the sliding-lid template description and updated the verified fixture total.
- `docs/SLIDING_LID_BOX_PHASE3_IMPLEMENTATION_2026-07-15.md` — this detailed report.

No storage schema, backup code, unrelated tab, existing report, LightBurn project, or generator Python file was changed.

## 4. Legacy SVG signatures captured before editing

The representative legacy outputs were generated in an isolated direct-file browser probe. The signatures below are length plus the existing FNV-style `designFixtureHash()` value.

| Template | Valid | SVG length | Hash | Layout |
|---|---:|---:|---|---|
| `qr-stand` | yes | 359 | `fe737a09` | 300 × 180 mm |
| `hanging-sign` | yes | 341 | `656e633d` | 150 × 180 mm |
| `dice-tray` | yes | 1726 | `51a55721` | 421 × 307 mm |
| `divider-tray` | yes | 1965 | `a55dda6e` | 656 × 307 mm |
| `finger-box` open-top | yes | 2559 | `4e2a6f4b` | 287 × 252 mm |
| `finger-box` loose-lid | yes | 2691 | `c202cef2` | 287 × 252 mm |

The two legacy finger-box outputs retain their original panel order. The four older non-box generators also remain valid and stable under the same signatures.

## 5. Architecture implemented

The new template is `sliding-lid-box`, dispatched as a separate branch from `buildDesignResult()`. It is not a lid option inside the existing `finger-box`, and `buildBoxModel()` remains the standard-box implementation.

The model is composed from pure helpers for:

- `slidingLidBoxDimensions()`;
- `buildSlidingLidRail()`;
- `buildSlidingLidPanel()`;
- `buildSlidingLidSidePanel()`;
- composite shortened-front edge construction;
- `buildSlidingLidBoxModel()`;
- row-driven `layoutDesignPanelRows()`;
- actual translated path-bound and segment checks.

The existing finger pattern, edge, point compaction, orthogonal path, serializer, and preview/download pipeline remain shared. `layoutDesignPanels()` is retained as the legacy wrapper with its exact existing rows.

The new construction contains eight through-cut pieces:

1. Bottom.
2. Sliding lid.
3. Front (open end).
4. Back (closed stop end).
5. Left.
6. Right.
7. Left rail.
8. Right rail.

Both rails are present as separate physical pieces and have identical local paths. They are glue-on laminated inner channels, not engraved grooves, removable rails, or wall-slot assemblies.

## 6. Exact dimension formulas

The implementation uses measured thickness `t`, upper web `U`, lower web `L`, and stop length `S`, each defined as:

```text
U = max(t, 4 mm)
L = max(t, 4 mm)
S = max(t, 4 mm)
```

For usable-cavity mode:

```text
Wi = Uw + 2t
Di = Ud
Hi = Uh + t + Cv + U

Wo = Wi + 2t
Do = Di + 2t
Ho = Hi + t
```

For outside-body mode:

```text
Wi = Wo - 2t
Di = Do - 2t
Hi = Ho - t

Uw = Wi - 2t
Ud = Di
Uh = Hi - U - t - Cv
```

Rail and lid dimensions are:

```text
Ch = t + Cv
Rh = U + Ch + L
Rl = Di
Rc = Di - S

G = Wi - 2t
Lw = Wi - Cs
E = (Lw - G) / 2
Tw = G - Cs
```

The lid length is `Di + t`. Its shoulders stop at `Rc`; the narrower center tongue continues to `Di` and closes the center between the two rail stop webs. The shortened Front wall height is:

```text
Hf = t + Uh - Ci
```

Usable cavity is shown separately from full wall-to-wall interior and outside body dimensions.

## 7. Exact clearance semantics

The four relevant fit concepts remain separate:

- `Cj` changes only body finger-joint transitions and preserves the existing half-per-side behavior.
- `Cs` is total lid side clearance. Each side receives `Cs / 2`; it changes lid shoulder width, tongue width, and engagement.
- `Cv` is added once to the material thickness to form channel height `Ch`.
- `Ci` is the one-sided gap between the shortened Front wall and the lid underside; it changes `Hf` only.

Kerf is not read from Library or Projects and is absent from generated geometry. The UI and README identify kerf as a separate LightBurn/material-test concern.

The starting defaults are intentionally experimental: usable cavity 100 × 80 × 40 mm, thickness 3 mm, body joint clearance 0.05 mm, preferred finger width 12 mm, total side clearance 0.20 mm, vertical clearance 0.20 mm, Front insertion clearance 0.20 mm, and no pull. These are not production-fit claims.

## 8. Rail geometry

Each rail is one closed orthogonal U-channel path with:

- a lower web of height `L`;
- channel clear height `Ch`;
- an upper web of height `U`;
- an open Front channel of length `Rc`;
- an integral solid Back stop of length `S`.

The rail bottom is placed `Uh - L` above the inside floor. The lower web ends at the lid underside plane `Uh`; the channel reaches `Uh + t + Cv`; and the upper web ends at `Hi`, flush with the side-wall top. Rail Back is flush with the inside Back plane.

The UI identifies the construction as two laminated through-cut inner channels. Assembly text instructs the user to glue the rails top-flush and back-flush, with both channels facing inward, using a thickness-plus-clearance spacer and keeping glue out of the channels.

No registration tabs, wall slots, partial-depth grooves, hinges, magnets, living hinges, removable rails, or external hardware were added.

## 9. Sliding-lid geometry

The lid has stable ID `sliding-lid` and title `Sliding lid`. Its local outline has:

- maximum shoulder width `Lw`;
- center tongue width `Tw`;
- shoulders ending at `t + Rc` in the serialized panel coordinates;
- tongue ending at `t + Di`;
- total length `Di + t`.

The lid is inserted from Front toward Back. The side shoulders contact the integral Back stop webs. The narrower tongue passes between those webs and reaches the inside Back plane, closing the center stop region.

The lid rides inside the laminated rails, below the wall tops. The upper web captures the shoulders vertically; the lower web supports the lid.

## 10. Shortened Front and side composite-edge geometry

The Front panel uses a separate vertical finger pattern based on `Hf`; it is not a cropped copy of the full wall pattern. The bottom joint remains part of the normal body construction.

Each side panel has:

- the proven full Back vertical relationship;
- the proven Bottom relationship;
- a shortened lower Front finger field complementary to the Front panel;
- a plain upper Front run above `Hf`.

The implementation uses a focused composite-edge path helper. It preserves matching lower boundaries and opposite phases, then continues as a plain edge to the full side-wall height. Fixtures cover the eight body mating relationships, phase parity, pattern lengths, no cropped final finger, and the plain upper Front run.

## 11. Pull-notch behavior

Supported modes are:

- `None` (default);
- `Centered rectangular edge notch`.

The notch enters from the lid Front edge, is centered, and uses depth `max(t, 6 mm)`. Side material must remain at least `max(2t, 6 mm)`. The implementation rejects a pull that crosses the shoulder transition or exceeds the available width.

The path remains orthogonal and uses no arcs, semicircles, interior holes, engraved marks, or curved-path support.

## 12. Deterministic layout

The new four-row layout uses the existing 10 mm margin and 15 mm gap:

```text
Row 1: bottom, sliding-lid
Row 2: front-open, back
Row 3: left, right
Row 4: rail-left, rail-right
```

For Golden A, the expected layout is 259 × 262.6 mm with positions:

```text
bottom@10,10
sliding-lid@137,10
front-open@10,111
back@137,111
left@10,176.2
right@111,176.2
rail-left@10,241.4
rail-right@105,241.4
```

Actual translated path bounds are checked in addition to nominal panel envelopes. The first eight rows/positions remain extensible for a future fifth coupon row; no coupon row is generated in this phase.

## 13. Blocking errors and warnings

Blocking validation covers blank, nonnumeric, non-finite, negative, unknown-mode, nonpositive-dimension, insufficient-engagement, short-channel, oversized-rail, collapsed-front-gap, insufficient shortened-front pattern, oversized pull, path, layout, SVG, and viewBox conditions.

In particular:

```text
Omin = max(1 mm, t/3)
E >= Omin
Rc > max(3t, 12 mm)
pull side web >= max(2t, 6 mm)
```

Outside mode rejects `Wo <= 4t`, `Do <= 2t`, or `Ho <= U + 2t + Cv`.

Nonblocking warnings cover zero or unusually large clearances, close-to-minimum engagement, small finger widths, large layouts, rail intrusion into usable width, glue, char cleanup, and unverified plywood sliding fit. Warnings do not claim physical failure.

## 14. Golden fixture results

The Designs fixture group now contains 265 assertions and reports 265 passed / 0 failed.

Golden A, usable cavity mode:

```text
Wi 106, Di 80, Hi 47.2
Wo 112, Do 86, Ho 50.2
G 100, Lw 105.8, E 2.9, Tw 99.8
Ch 3.2, Rh 11.2, Rl 80, Rc 76
lid length 83, shoulder stop 76, tongue end 80, Hf 42.8
8 pieces
```

Golden B, outside-body mode:

```text
Wi 126, Di 96, Hi 60
Uw 120, Ud 96, Uh 52.8
G 120, Lw 125.8, E 2.9, Tw 119.8
Ch 3.2, Rh 11.2, Rl 96, Rc 92
lid length 99, shoulder stop 92, tongue end 96, Hf 55.6
8 pieces
```

Expected values are hand-derived in the fixture code rather than calculated by the production dimension helper. Fixtures cover clearances, rails, lid capture, body mating, actual path bounds, layout, malformed values, warnings, determinism, SVG groups, and mutation safety.

## 15. Designs fixture total

```text
Design geometry fixtures
265 passed
0 failed
```

This replaces the previous 82-assertion Designs total. The existing legacy assertions remain in the same fixture function and continue to pass.

## 16. Full fixture-suite total

The isolated browser run reported:

| Group | Passed | Failed |
|---|---:|---:|
| Baseline | 20 | 0 |
| Material test normalization | 12 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Project Wizard | 216 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Designs geometry | 265 | 0 |
| **Total** | **797** | **0** |

## 17. HTML and JavaScript validation

- `python -m html.parser index.html`: passed.
- Browser execution parsed and executed the inline JavaScript successfully.
- The isolated browser ran all fixture functions without runtime exceptions.
- No external script or stylesheet dependency was added.
- The only URL-like strings found in the source are SVG namespace/output strings and existing documentation/reference links; the app remains self-contained.

No separate Node or standalone JavaScript parser was available in the local environment. Browser execution is the stronger runtime syntax check for this single-file app.

## 18. Direct offline validation

The verification opened:

```text
file:///C:/Genmitsu%20L8%20Tracker/index.html
```

The page rendered eight tabs and the normal app path did not automatically run the injected fixture probe. No server or network was required.

The direct query fixture URL was also reviewed as part of the existing self-test routing, while the successful recorded run invoked the same exposed fixture function directly from the normal offline page to collect all group totals.

## 19. SVG parsing and representative-output checks

The six legacy representative SVGs were valid. The sliding Golden A and Golden B fixture paths parse through `designSvgValidation()` and each contain:

- one red `g#cut` group;
- eight physical panel groups;
- one path per panel;
- millimeter dimensions and matching viewBox;
- orthogonal finite paths;
- no `<text>` in cut geometry;
- no fit coupon or fifth row.

The fixtures also check actual translated path AABB separation, duplicate translated segments, local path closure, self-intersection, zero-length segments, and panel bounds.

## 20. Preview/download identity

Preview uses the result SVG data URL. Download uses the same `buildDesignResult()` SVG string. The fixture suite checks repeated byte stability and compatibility `designSvg()` identity. Invalid values return no export SVG and disable the download button; correcting the values restores the valid preview/download path.

## 21. Storage and backup isolation

The browser probe found:

- `localStorage.getItem('genmitsu-l8-tracker-v1')` was `null` in the isolated profile;
- `backupObject()` does not contain `designDraft`;
- Designs remain module-level session state;
- `STORAGE_KEY` and `SCHEMA_VERSION` were not changed;
- no Design field was added to `freshState()`, `persist()`, or `backupObject()`.

Editing or exporting a sliding-lid design therefore remains outside application persistence and JSON backups.

## 22. Fit coupon status

The fit coupon remains intentionally deferred to Phase 3.1. No checkbox, coupon geometry, coupon labels, coupon row, coupon fixture, or persistent fit-learning behavior was added. The row-driven layout preserves the first four rows so a future fifth row can be added without changing the first eight piece positions.

## 23. Unverified LightBurn and physical-cut checks

The following remain manual follow-up and are not claimed as complete:

- LightBurn import of the new sliding-lid SVG;
- cutting the rails and body;
- physical glue-up;
- physical lid sliding/capture/stop fit;
- production clearance selection;
- material-batch or kerf validation.

The existing standard finger box has prior physical evidence. That evidence does not transfer automatically to this new rail/lid construction.

## 24. Final `git status -sb`

Final tracked status is:

```text
## main...origin/main
 M README.md
 M index.html
```

The repository also retains the pre-existing unrelated untracked files, including LightBurn projects, prior reports, `.claude/`, and `parametric_qr_stand_generator.py`. The new implementation report is intentionally untracked until the user chooses whether to stage it.

## 25. Commit and push confirmation

Nothing was staged, committed, or pushed. No reset, clean, stash, move, or delete operation was performed.
