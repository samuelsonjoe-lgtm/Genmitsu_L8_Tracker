# Finger Box Finished View — Implementation Audit (Adversarial, Read-Only)

**Date:** 2026-07-18  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Committed baseline:** `0ce2f5bb8f6dad66ee0fd013514a5f993598a26c` (`0ce2f5b` — Add concealed cleat full-box prototype)  
**Authoritative design docs:**  
- `docs/DESIGNS_FINGER_BOX_FINISHED_VIEW_DESIGN_REVIEW_2026-07-18.md`  
- `docs/DESIGNS_FINGER_BOX_FINISHED_VIEW_IMPLEMENTATION_2026-07-18.md`  

**Audit constraint:** Read-only on product tree. No edit, stage, commit, push, reset, clean, stash, move, rename, or delete of product files. This report is the only intended new file under `docs/`.

---

## 1. Repository state at audit start

| Check | Result |
| --- | --- |
| **HEAD (full)** | `0ce2f5bb8f6dad66ee0fd013514a5f993598a26c` |
| **HEAD (short)** | `0ce2f5b` |
| **Branch** | `main` |
| **Sync `origin/main...HEAD`** | `0 0` (up to date) |
| **Staging area** | Empty (nothing staged) |
| **Tracked modifications** | `index.html`, `README.md` only |
| **Untracked (preserved; not modified by this audit)** | `LightBurn Projects/`, `debug.log`, many historical `docs/*` (including design review + implementation report), `parametric_qr_stand_generator.py` |

### Diff summary

| File | Diffstat |
| --- | --- |
| `README.md` | 4 lines changed (fixture totals + Finished View prose) |
| `index.html` | 2 hunks only: rewrite of `buildFingerBoxFinishedViewSvg` + Finished View fixture block (~23 insertions / 19 deletions net; large single-line function bodies) |

**No** changes to production builders, serializers, storage, or other Finished View helpers (verified by function-body identity vs `HEAD` for `buildBoxModel`, `buildBoxDesignResult`, `serializeDesignSvg`, `downloadCurrentDesignSvg`, `designPreviewModeForTemplate`, `fingerBoxCornerDecorationSegments`, `buildFingerPattern`, `designBoxDimensions`).

### Scope creep

| Claim | Finding |
| --- | --- |
| Only intentional product changes are `index.html` + `README.md` | **Confirmed** for tracked files |
| Implementation report present | Untracked `docs/DESIGNS_FINGER_BOX_FINISHED_VIEW_IMPLEMENTATION_2026-07-18.md` (documentation only; not product runtime) |
| LightBurn / debug / historical docs | Untouched |
| Scope creep into production geometry / other templates | **None** |

---

## 2. Files and functions reviewed

### Product
- `index.html` — `buildFingerBoxFinishedViewSvg` (full rewrite), Finished View fixture block (~4525–4555), dispatch path `designResultsHtml` / `designPreviewModeForTemplate` / `designPreviewSelectorHtml` / `bindDesignPreviewActions` / `downloadCurrentDesignSvg` (unchanged bodies)
- `README.md` — Built-in checks totals; Finger Box Finished View paragraph

### Authoritative documents (read fully)
- Design review 2026-07-18  
- Implementation report 2026-07-18  

### Boundaries confirmed identical to `HEAD`
- `buildBoxModel`, `buildBoxDesignResult`, `serializeDesignSvg`, finger pattern / decoration production path, download path, preview mode dispatch allow-list

---

## 3. Semantic architecture findings

### Implementation shape

`buildFingerBoxFinishedViewSvg(result)`:

1. Reads **only** semantic metrics: `dimensions.outsideWidth|Depth|Height`, `materialThickness`, `patterns.*.count`, `boxLid`, `cornerDecoration`, `closedHeight`, and `result.valid`.
2. Returns `''` if invalid / non-finite / non-integer counts / lid not `open|loose` / envelope too thin (`width|depth ≤ 2t`).
3. Builds a fixed `viewBox="0 0 520 340"` SVG with `width="100%"` `height="auto"`, isometric projection (`depthX=0.34`, `depthY=0.38`, `heightY=0.52`), **one** uniform `scale`.
4. Draws bottom, front, right, top rims, cavity; count-based zigzag finger markers on `front-base`, `right-base`, `front-right-vertical`; optional faux group; optional separate loose-lid polygon.
5. Uses `esc()` for text; does not read `result.svg`, `panels[].path`, or score paths.

### Architecture contract (design review)

| Requirement | Status |
| --- | --- |
| Semantic-only | **Pass** |
| No `result.svg` parse | **Pass** (fixture + independent probe: empty `svg` yields identical Finished View) |
| No production mutation | **Pass** (hashes stable across repeated preview builds) |
| Finger Box-local (no shared isometric engine) | **Pass** |
| Session-only preview mode | **Pass** (`designPreviewMode` unchanged architecture) |
| Download uses production SVG | **Pass** (independent Edge probe with mode `finished-view`) |

**Hidden coupling:** None material. Projection constants echo Sliding Lid style but are local literals, not a shared helper.

**Severity:** no architecture blockers.

---

## 4. Projection and dimensional findings

| Check | Evidence |
| --- | --- |
| Outside W/D/H on root | `data-outside-width/depth/height` match semantic outside dims (fixture + multi-shape probe) |
| Thickness | `data-material-thickness`; cavity inset uses `t` on all three axes |
| Uniform scale | Single `scale` attribute; body and loose lid share the same scale factor |
| Inside-as-outside | Not observed; data attributes track outside envelopes for wide-shallow, narrow-deep, tall-narrow, low-wide, thick-relative, min-ish |
| Loose lid thickens body | Body height remains `outsideHeight`; `closedHeight` appears only as **text** on loose lid (“closed stack … mm”) |
| Extreme finite | Existing fixture + probe: no NaN/Infinity; no parsererror |

**Note (LOW):** Fixed viewBox with dense footer labels at y≈276–338 leaves little vertical margin; extreme aspect ratios rely on scale only. Probe shapes remained finite and parser-clean; no inverted faces observed in data attributes.

**Browser note (LOW):** Chromium/Edge logs `Error: <svg> attribute height: Expected length, "auto".` when parsing/inserting Finished View SVG. Same pattern exists on Drawer Cabinet Finished Front View (`height="auto"`). Does not fail fixtures or production; responsive intent is HTML embedding via `width="100%"` + viewBox.

---

## 5. Finger-pattern findings

| Check | Status |
| --- | --- |
| Counts from `patterns.width/depth/vertical.count` | **Pass** — fixtures assert marker counts equal semantic counts |
| Count changes with preferred finger width | **Pass** (probe: 7 vs 21 width markers track pattern counts) |
| Visible structural edges only | **Pass** — only `front-base`, `right-base`, `front-right-vertical` |
| Loose lid has no fingers | **Pass** |
| No clearance/kerf claim | **Pass** — copy says “Representative finger pattern” |
| Cap / substitute | **No cap on finger markers** (full count emitted; high counts e.g. 21+15+7 remain complete) |

**Note (LOW):** Zigzag markers are schematic, not production polylines — correctly framed as representative. High counts can be visually busy; acceptable for v1.

---

## 6. Lid honesty findings

| Mode | Requirement | Status |
| --- | --- | --- |
| Open | Cavity; no loose-lid group; no closure hardware | **Pass** |
| Loose | Separate plain panel; body remains open-top; outside W×D on lid data attrs; no fingers; no hinge/rail/groove/magnet/slide **geometry** | **Pass** |
| Closed stack | Text-only `closedHeight` | **Pass** |

User-facing copy explicitly states open-top / separate / no retention. Search for misleading closure **geometry** (ids/classes) is clean. The word “retention” appears only in **negation** (“no retention mechanism”) — honest.

---

## 7. Faux-dovetail honesty findings

| Check | Status |
| --- | --- |
| Driven by `metrics.cornerDecoration === 'faux-dovetail-engrave'` | **Pass** |
| No production score path reuse | **Pass** — no `corner-decoration` in Finished View; production still has it |
| Structural finger markers retained | **Pass** (25 markers open === 25 with decoration) |
| Decorative caption | **Pass** — “decorative only; structural joint remains finger-jointed” |
| Disabled mode: no group / no caption | **Pass** (`#finger-box-faux-dovetail` absent; no “Faux dovetail engraving” caption) |
| Color-only distinction | **Pass** — dashed paths + explicit text; fingers use solid structural stroke class |
| Cap | Decorative marks use `Math.min(6, counts.vertical)` — **capped for legibility** |

**Note (LOW):** Cap is not documented in README; design review allowed capping if documented/tested. Cap is non-blocking; prefer a one-line README note later.

**Note (LOW):** Open-mode SVG **style** block always defines `.finger-box-faux-dovetail-mark` class even when decoration is off. No decorative group or caption; not user-facing as enabled decoration. Not a production leak.

---

## 8. Production-byte neutrality (independent evidence)

Verified on Edge (Playwright, temp **copy** of working-tree `index.html` with audit hooks only in the copy; product tree not modified):

| Artifact | Length | FNV-1a | Match claimed golden |
| --- | --- | --- | --- |
| Default open Finger Box production | **2483** | **`a892f91c`** | Yes |
| Loose-lid production | **2615** | **`6181bc75`** | Yes |
| Faux-dovetail production | **7269** | **`d9512d1c`** | Yes |

| Neutrality test | Result |
| --- | --- |
| Hash before Finished View | `a892f91c` |
| Hash after 1 preview | `a892f91c` |
| Hash after 3 previews | `a892f91c` |
| `result.svg` identity preserved | true |
| Rebuild production after preview | equal |
| Finished View strings absent from production | true |
| Goldens **not** updated by this feature | Confirmed (match baseline pins) |

---

## 9. Download-path findings

`downloadCurrentDesignSvg` still:

1. Builds `buildDesignResult(designDraft)` via `refreshDesignPreview()`  
2. Downloads **`result.svg`** (production), never Finished View markup  

Independent probe with `designPreviewMode = 'finished-view'`:

| Field | Value |
| --- | --- |
| filename | `l8-finger-box-2026-07-19.svg` (date from runtime) |
| downloaded === open production | **true** |
| hash | `a892f91c` |
| contains Finished View ids | **false** |

Invalid preview returns `''`; download gate remains `result.valid` + production SVG validation — Finished View failure does not substitute preview bytes.

---

## 10. Storage and schema findings

| Item | Status |
| --- | --- |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` (unchanged; not in diff) |
| `SCHEMA_VERSION` | `2` (unchanged) |
| Diff touches `freshState` / `loadState` / `persist` / `backupObject` / import-export | **No** |
| Preview generation storage isolation | Fixture + probe: localStorage + `backupObject()` byte-identical |
| `designPreviewMode` persisted | **No** (session `let` only) |

---

## 11. Fixture quality findings

### Suite arithmetic (independently executed)

| Suite | Passed | Failed |
| --- | --- | --- |
| Tray-model | **264** | **0** |
| Designs geometry | **1072** | **0** |
| Complete `?selftest=all` (Edge) | **1864** | **0** |

Net Designs growth 1069 → 1072 is **three net assertions**, but the Finished View block was **rewritten** into denser multi-condition `add(...)` calls covering envelope attributes, accessibility, lid modes, count-based markers, faux on/off, semantic-empty SVG, production hash stability, and storage/backup isolation. This is **adequate for commit**, not merely “three weak checks.”

### Contract coverage map

| Contract | Fixture / independent probe |
| --- | --- |
| Semantic-only / empty production SVG | Fixture **+** probe |
| Production byte neutrality | Fixture (hash on same object) **+** probe (before/after/rebuild) **+** golden pins |
| Determinism | Fixture + probe |
| Open / loose honesty | Fixture + probe |
| Count-based markers | Fixture + probe (width preference change) |
| Faux on/off + decorative language | Fixture + probe |
| Storage / backup isolation | Fixture + probe |
| Accessibility (role, title, desc, aria, 100% width) | Fixture |
| Invalid result | Existing invalid fixtures retained |
| Download protection | **Independent probe only** (suite relies on existing decoration live-download + download architecture; no new dedicated finished-mode download fixture) |
| Non-Finger protection | Production helpers unchanged; full suite green |

### Weaknesses (non-blocking)

| Issue | Severity |
| --- | --- |
| “Repeated … production bytes unchanged” partially asserts object identity before rebuild; mitigated by `buildDesignResult` equality clause and independent probe | LOW |
| No fixture asserting preferred-finger-width count **change** (probe only) | LOW / optional |
| No fixture documenting faux mark cap of 6 | LOW / optional |
| No dedicated download-with-finished-view fixture in suite | LOW (probe covers) |
| Responsive “height=auto” asserted despite SVG attribute warning | NOTE |

**No missing fixture is a commit blocker** given full-suite green + independent production/download probes.

---

## 12. Other-template regression risk

| Surface | Finding |
| --- | --- |
| Sliding Lid / Drawer / Dice / Divider / Cleat helpers | Unchanged bodies |
| `serializeDesignSvg` / layout helpers | Unchanged |
| Preview selector allow-list | Unchanged |
| Full suite 1864/0 | Covers cross-template fixtures |

---

## 13. Accessibility and responsive findings

| Item | Status |
| --- | --- |
| `role="img"` | Present |
| `title` / `desc` / `aria-label` | Present; screen-only + representative language |
| Visible screen-only note | Present in SVG + results footer |
| Text escaped | `esc()` used |
| Keyboard controls | No new controls; existing preview buttons unchanged |
| Mode-specific aria/desc | **Static** — does not mention loose vs open or faux in `aria-label`/`desc` (visible body text does) |

**MEDIUM/LOW improvement (non-blocking):** Vary `aria-label`/`desc` by lid and decoration for screen-reader accuracy.

**LOW:** `height="auto"` console warning (see §4).

---

## 14. Documentation findings

README updates:

- States isometric, screen-only, semantic outside dims + pattern counts  
- Representative finger cues  
- Loose lid separate, no retention  
- Faux decorative; structural finger remains  
- Does not alter production SVG; downloads remain Cut Layout  
- Does not claim fit/strength/kerf/safety  
- Fixture totals updated to 1072 / 1864 with corrected arithmetic  

**Matches** design review intent. Optional later: document decorative mark cap of 6.

Implementation report accurately describes scope; residual “physical dry-fit” sentence is about production geometry in general, not a claim that this screen-only feature needs physical validation before commit.

---

## 15. Browser / visual validation

| Check | Observed |
| --- | --- |
| Headless Edge `file://` temp copy with working-tree HTML | Design 1072/0, tray 264/0, full 1864/0 |
| Results HTML with Finished View mode | Contains `finger-box-finished-view`, selector, screen-only note |
| Multi-shape semantic envelopes | Correct outside data attributes; markers match counts |
| Pixel-level visual inspection of isometric aesthetics | **Not** performed (no interactive screenshot review of legibility/overlap polish) |
| Direct interactive Edge UI (non-headless) | **Not** re-run beyond automated Edge channel |

Unverified remaining: visual polish (occlusion, label collision, dense high-count markers), narrow-viewport human eyeballing.

Physical dry-fit is **out of scope** for this screen-only feature (production geometry unchanged).

---

## 16. Commands and tests run

| Command / action | Result |
| --- | --- |
| `git status` / `git rev-parse` / `git rev-list --left-right --count origin/main...HEAD` | Clean staged; HEAD `0ce2f5b`; sync `0 0`; modified `index.html`, `README.md` |
| `git diff --stat` / full unstaged diff inspection | Scope as §1 |
| `git diff --check` | Pass (CRLF warnings only) |
| `python -m html.parser index.html` | Pass |
| Function identity vs `HEAD` (production helpers) | All listed helpers **IDENTICAL** |
| Edge Playwright: `runDesignGeometryFixtures` | **1072 / 0** |
| Edge Playwright: `runTrayModelFixtures` | **264 / 0** |
| Edge Playwright: `?selftest=all` | **1864 / 0** |
| Independent production golden probe | open/loose/faux match §8 |
| Preview before/after hash + determinism | Pass |
| Download under Finished View mode | Production bytes only |
| localStorage + backupObject isolation | Pass |
| Shape matrix (wide/shallow, narrow/deep, tall, low, thick, min) | Finite FV; envelope attributes correct |

Audit runner lived only under worktree `agent-tools/` and a **temp** HTML copy; product tree not modified by hooks.

---

## 17. Blocking findings

**None.**

No BLOCKER or HIGH defects found that force production, storage, honesty, or golden changes.

---

## 18. Non-blocking improvements

| ID | Severity | Item |
| --- | --- | --- |
| N1 | LOW | Document faux decorative mark cap (`min(6, vertical)`) in README |
| N2 | LOW | Vary `aria-label` / `<desc>` for open vs loose vs faux |
| N3 | LOW / NOTE | `height="auto"` triggers SVG attribute console warning; consider omitting height or using CSS only (Drawer Cabinet has same pattern) |
| N4 | LOW | Optional fixture: preferred finger width changes marker counts; download while Finished View selected |
| N5 | NOTE | Style block always includes faux class name when decoration disabled (harmless) |
| N6 | NOTE | High finger counts remain fully drawn — may look busy; acceptable for v1 |
| N7 | NOTE | Optional human visual pass for label packing / lid placement on extreme aspect ratios |

---

## 19. Verdict

### **SAFE TO COMMIT WITH NON-BLOCKING NOTES**

The implementation is a bounded, Finger Box-local upgrade of `buildFingerBoxFinishedViewSvg` plus README and focused fixtures. Production geometry, serialization, goldens, storage/schema, and other templates are unchanged and independently verified. Honesty constraints (open cavity, separate loose lid, decorative-only faux, representative fingers, screen-only) hold. Suite totals **264 / 1072 / 1864** all at **0 failed**.

**Would another Claude review add meaningful value?** **No.** There is no shared-helper risk, no production-serialization change, and no storage/schema risk. Non-blocking notes above are polish/docs/a11y follow-ups suitable for a later pass or a short Grok post-commit spot-check if desired—not a second architecture audit.

---

*End of audit. Product files were not modified for this report.*
