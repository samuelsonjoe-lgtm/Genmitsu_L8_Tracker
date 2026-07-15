# Sliding-Lid Phase 3.1 Guide Offset Verification

Date: 2026-07-15  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline commit: `df54da0` - `Add sliding-lid box generator`

## 1. Executive conclusion

The Phase 3.1 guide-offset correction is present and consistent with the updated Back-flush rail convention. The live design harness reports `Designs 322 passed / 0 failed` and the full fixture suite reports `854 passed / 0 failed`. The disabled sliding-lid signature remains `2800 / 4a7ab718`.

No further code correction is indicated by this verification pass. The remaining risk is physical follow-up: the corrected guide marks still need a real scrap cut and fit check before anyone treats them as production-safe cutting aids.

## 2. Repository state

Verification commands run before the report was written:

```powershell
git status -sb
git rev-parse --short HEAD
git log -1 --oneline
git diff --check
git diff --stat
git diff -- README.md index.html
git diff --cached --name-only
```

Observed state:

- `HEAD` remained `df54da0`.
- `git log -1 --oneline` reported `df54da0 Add sliding-lid box generator`.
- Nothing was staged.
- Tracked working files remained the expected `README.md` and `index.html`.
- Existing unrelated untracked files were untouched.
- `git diff --check` passed, with only the usual LF-to-CRLF warnings in the working copy.

## 3. Confirmed correction formula

The verified correction is:

```text
railOffsetX = panelWidth - railLength
```

For Golden A:

```text
panelWidth = 86
railLength = 80
railOffsetX = 6
guideInset = 1.5
guideLeg = 3
railHeight = 11.2
```

That places the physical Back-flush rail footprint at:

```text
x = 6 through 86
y = 0 through 11.2
```

The old Front-flush interpretation `x = 0 through 80` is no longer the one represented by the guide overlay.

## 4. Left Golden A coordinate verification

The Left guide endpoints verified against the corrected footprint are:

```text
(7.5,1.5)-(10.5,1.5)
(7.5,1.5)-(7.5,4.5)

(84.5,1.5)-(81.5,1.5)
(84.5,1.5)-(84.5,4.5)

(7.5,9.7)-(10.5,9.7)
(7.5,9.7)-(7.5,6.7)

(84.5,9.7)-(81.5,9.7)
(84.5,9.7)-(84.5,6.7)
```

Confirmed facts:

- Front reference is `railOffsetX + inset`.
- Back reference is `panelWidth - inset`.
- The visible Front gap is exactly `panelWidth - railLength`.
- No segment remains based on the old `railLength - inset` Back coordinate.

## 5. Mirrored Right coordinate verification

The mirrored Right guide endpoints verified for the complete Right panel are:

```text
(78.5,1.5)-(75.5,1.5)
(78.5,1.5)-(78.5,4.5)

(1.5,1.5)-(4.5,1.5)
(1.5,1.5)-(1.5,4.5)

(78.5,9.7)-(75.5,9.7)
(78.5,9.7)-(78.5,6.7)

(1.5,9.7)-(4.5,9.7)
(1.5,9.7)-(1.5,6.7)
```

Confirmed facts:

- The mirror is horizontal only.
- There is no vertical flip.
- No negative coordinates appear.
- Right layout position remains unchanged.
- Right nominal dimensions remain unchanged.
- The separate Right rail piece remains identical and unmirrored.
- The source model and draft were not mutated by the verification run.

## 6. Guided physical mating assessment

The verified guided output still preserves the intended mating relationships:

- Bottom to Right remains a valid complementary match.
- Back to Right remains a valid complementary match.
- The shortened Front lower field remains shortened after the marked-face-in assembly convention.
- The full-height edge remains at Back.
- The rail-stop end remains at Back.
- The guide offset changes placement of the optional marks only; it does not alter any cut geometry.

This pass did not physically cut the board, so the real-world fit still needs a scrap test.

## 7. SVG score and cut verification

With guides enabled, the verified output contains:

- exactly one blue `g#score`;
- score before red `g#cut`;
- exactly two guide subgroups;
- four L marks per side;
- eight segments per side;
- sixteen score segments total;
- guide segments under the corrected rail footprint;
- guide segments inside the side-panel bounds;
- guide segments off the cut edges;
- unchanged cut-piece count;
- unchanged `viewBox`;
- no embedded speed, power, interval, or device settings.

With guides disabled, the output remains byte-stable at the expected signature and contains no score geometry.

## 8. Four option-combination table

The harness verified all four combinations:

| Combination | Valid | Cut pieces | Score groups | Document size |
| --- | --- | ---: | ---: | --- |
| guides off / coupon off | yes | 8 | 0 | 259 × 262.6 mm |
| guides on / coupon off | yes | 8 | 1 | 259 × 262.6 mm |
| guides off / coupon on | yes | 14 | 0 | 336.8 × 320.6 mm |
| guides on / coupon on | yes | 14 | 1 | 336.8 × 320.6 mm |

For all four combinations:

- the SVG output was deterministic;
- preview and download used identical SVG text;
- the first eight layout positions stayed unchanged.

## 9. Disabled and legacy signatures

Disabled sliding-lid output:

- SVG length: `2800`
- hash: `4a7ab718`
- no score group
- no coupon pieces
- eight existing panel positions unchanged

Legacy signatures remained unchanged:

| Template | SVG length | Hash |
| --- | ---: | --- |
| QR stand | 359 | `fe737a09` |
| Hanging sign | 341 | `656e633d` |
| Dice tray | 1726 | `51a55721` |
| Divider tray | 1965 | `a55dda6e` |
| Finger box open | 2559 | `4e2a6f4b` |
| Finger box loose lid | 2691 | `c202cef2` |

## 10. Coupon regression

The coupon behavior remained intact:

- six stable coupon pieces;
- Golden A numeric values unchanged;
- `Cs/2` lateral clearance per side;
- `Cv` added once to channel height;
- `Cj` does not affect coupon geometry;
- first eight box positions unchanged;
- coupon positions and final `viewBox` unchanged;
- Front insertion, capture, stop, and tongue relationships remain coherent.

## 11. Fixture totals and coverage

The harness reported:

- Designs: `322 passed / 0 failed`
- Total suite: `854 passed / 0 failed`

Group totals:

| Group | Passed | Failed |
| --- | ---: | ---: |
| Baseline resolution | 20 | 0 |
| Material Test normalization | 12 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Project Wizard | 216 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Designs | 322 | 0 |

The verification confirms the fixtures now independently lock:

- rail offset against panel width;
- corrected Left endpoints;
- corrected mirrored Right endpoints;
- physical Back reference;
- four option combinations;
- disabled sliding signature;
- guided mating spot checks;
- coupon regression values.

I did not find any remaining fixture that still verifies the old Front-flush assumption.

## 12. UI and documentation verification

The updated UI/documentation language now clearly states:

- four concealed L marks per side;
- blue score before red cut;
- rails install top-flush and Back-flush;
- marked faces assemble inward;
- shortened edges face Front;
- stop ends face Back;
- the visible Front gap is intentional;
- the coupon requires glue and curing before judging fit.

Checkbox-level help is present and matches the guide/coupon behavior.

## 13. Storage and offline verification

This verification confirmed no changes to:

- `STORAGE_KEY`
- `SCHEMA_VERSION`
- `freshState()`
- `persist()`
- `backupObject()`
- import merge/replace behavior
- Library
- Projects
- Inventory
- Pricing
- Test Grids

The harness also confirmed:

- toggles do not write `localStorage`;
- preview/export do not write `localStorage`;
- backups exclude Designs;
- no network or external dependency was added;
- normal `file://` startup remains functional.

## 14. Remaining physical tests

Still worth doing outside the browser:

- cut one scrap sample with guides on;
- confirm the blue guide marks really sit under the Back-flush rail footprint;
- confirm the rail and lid behave as intended after glue-up and cure;
- confirm the marks are useful in the actual LightBurn workflow.

## 15. Final `git status -sb`

At the end of the verification pass, `git status -sb` still showed the expected tracked edits in `README.md` and `index.html`, plus the newly created verification report in `docs/`. Existing unrelated untracked files remained untouched.

## 16. No edit, stage, commit, or push

This pass did not edit the application code, stage anything, commit anything, or push anything. The only file created during this verification was this report.

SAFE TO COMMIT WITH DOCUMENTED MANUAL FOLLOW-UP
