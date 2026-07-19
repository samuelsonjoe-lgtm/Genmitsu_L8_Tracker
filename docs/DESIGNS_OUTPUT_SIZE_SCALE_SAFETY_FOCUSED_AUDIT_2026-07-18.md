# Designs Output-Size and Scale-Safety Disclosure — Focused Adversarial Audit

**Date:** 2026-07-18  
**Auditor:** Grok (read-only)  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Claimed baseline:** `ad4532c` — *Add Finger Box finished view*  
**Authoritative docs:**  
- `docs/DESIGNS_POST_FINISHED_VIEW_PRIORITY_REVIEW_2026-07-18.md`  
- `docs/DESIGNS_OUTPUT_SIZE_SCALE_SAFETY_IMPLEMENTATION_2026-07-18.md`  

**Primary question:** Is the Output-Size and Scale-Safety Disclosure implementation safe, honest, complete across all Designs templates, behaviorally tested, and ready to commit without changing production SVG bytes, downloads, storage, schema, Finished Views, or unrelated workflows?

---

## Verdict

**SAFE TO COMMIT WITH NON-BLOCKING NOTES**

- Another Claude review would **not** add meaningful value for this phase (no production serialization, storage/schema, or shared-dispatch risk found).  
- Claude Opus 4.8 is **unnecessary**.  
- Fable 5 is **unnecessary**.  
- Physical LightBurn import is **not** a commit requirement (baseline production bytes remain identical).

---

## 1. Working-tree scope

### Git state (independently recorded)

| Item | Value |
|------|--------|
| HEAD | `ad4532cfa7d5c112a25da4cbd17ac5cefd164e1c` |
| Branch | `main` |
| `origin/main...HEAD` | `0 0` (synchronized) |
| Staging area | Empty |
| Tracked modifications | `index.html`, `README.md` only |
| Diffstat | `README.md` 4/2; `index.html` 51/24 → **2 files, +55 / −26** |
| `git diff --check` | Exit 0 |
| `python -m html.parser index.html` | Exit 0 |

### Untracked (preserved, not part of intentional product change)

- `docs/DESIGNS_OUTPUT_SIZE_SCALE_SAFETY_IMPLEMENTATION_2026-07-18.md` (implementation report)  
- `docs/DESIGNS_POST_FINISHED_VIEW_PRIORITY_REVIEW_2026-07-18.md` and many historical review docs  
- `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`  

### Intentional product surface

| File | Role |
|------|------|
| `index.html` | Shared helper, `buildDesignResult` integration, results UI metrics/notice, fixtures |
| `README.md` | Disclosure contract + fixture totals 1080 / 1872 |
| Implementation doc | Documentation only (untracked deliverable) |

### Scope findings

- **No** serializer, fixture file outside `index.html`, storage module, generated asset, LightBurn project, or utility was modified as tracked product code.  
- Diff is tightly focused: remove three builder-local `>400` warnings; add `designLayoutSizeWarning` / `addDesignLayoutSizeWarning`; wrap `buildDesignResult`; unify `Production SVG size`; add scale-safety notice; expand Designs fixtures.  
- No large unrelated reformat or line-ending rewrite of unrelated regions beyond the disclosure hunks.  
- **Scope creep:** none at BLOCKER/HIGH.  

**NOTE (LOW):** Drawer Cabinet’s separate “Shell production sheet” metric label was removed; primary sheet size is now the shared `Production SVG size` row. Independently, `shellProductionSheet` dimensions match `result.widthMm` × `result.heightMm` for separate-thickness mode (`318.45 × 152.3`), so no shell-size information was lost. Interior retains `Interior production SVG size`.

---

## 2. Shared helper behavior

### Functions

```text
designLayoutSizeWarning(widthMm, heightMm)
addDesignLayoutSizeWarning(result)
buildDesignResult(...)  // returns addDesignLayoutSizeWarning(result)
```

### Independent helper matrix (headless Edge)

| Input | Warns? | Result |
|-------|--------|--------|
| 399.99 × 100 | No | Pass |
| **400 × 100** | **No** | Pass |
| 400.01 × 100 | Yes | Pass |
| 100 × 399.99 | No | Pass |
| **100 × 400** | **No** | Pass |
| 100 × 400.01 | Yes | Pass |
| 401 × 401 | Yes, **one** string | Pass |
| 0 / negative / NaN / Infinity / undefined / null | No | Pass |

### Properties verified

| Property | Evidence |
|----------|----------|
| Pure | No storage/machine reads; numeric only |
| Finite positive mm only | `Number.isFinite` and `> 0` gates |
| Threshold | `width > 400 \|\| height > 400` (strict greater) |
| Single advisory when both overflow | One `Large single-SVG layout:` string |
| Invalid results | `addDesignLayoutSizeWarning` skips when `!result.valid` |
| No shared-array mutation | Spreads new `warnings` via `Set` |
| Idempotent re-add | Set dedupes identical advisory |
| Return type | String or `''`; call sites treat as optional warning string |
| Does not block download / add errors | Advisory only in `warnings` |

Wording (honest): includes actual size, asks user to verify sheet and machine travel, states Tracker does **not** automatically split or nest. Does **not** call 400 mm an L8 bed size.

---

## 3. Complete template coverage

Independently exercised via `defaultsFor` + `buildDesignResult` + `designResultsHtml` for every registered path (modest + oversized where practical):

| Path | Modest | Width >400 | Height >400 | Both | Size metric | Notice | Dim match |
|------|--------|------------|-------------|------|-------------|--------|-----------|
| qr-stand | 300×180, adv 0 | 842×180, adv 1 | (layout height driven by stand geometry) | — | 1 | 1 | Yes |
| hanging-sign | 150×180 | 421×180 | 150×421 | 421×421 | 1 | 1 | Yes |
| dice-tray | 361×307 | 701×307 | 131×725 | 961×567 | 1 | 1 | Yes |
| divider-tray | 188×195 | 1216×307 | (tall layouts via tray height / stack) | — | 1 | 1 | Yes |
| finger-box | 287×252 | 837×240 | — | — | 1 | 1 | Yes |
| sliding-lid-box | 259×262.6 | 837×236.2 | — | — | 1 | 1 | Yes |
| drawer-cabinet | 368×292.8 | 1271×292.8 | — | — | 1 | 1 | Yes |
| joint-fit-coupon finger-edge | 230×106 | 1654×106 | — | — | 1 | 1 | Yes |
| joint-fit-coupon wall-to-base | 190×122.5 | 832×122.5 | — | — | 1 | 1 | Yes |
| concealed-cleat-corner | 192×240 | 834×561 | (both axes on large leg) | yes via large leg | 1 | 1 | Yes |
| concealed-cleat-full-box | 195×390 | 837×580 | — | yes via large width | 1 | 1 | Yes |

Also: Finger Box **Finished View**, Sliding-Lid **Finished Closed**, Drawer Cabinet **Finished Front**, Full-Box **Finished View** — each still shows exactly one Production SVG size and one scale-safety notice; production size remains cut-layout document bounds.

**No template skipped.** Both coupon modes covered separately.

---

## 4. Boundary and invalid-input behavior

| Case | Finding |
|------|---------|
| 399.99 / 400 / 400.01 | Matches contract (warn only strictly above 400) |
| Width-only / height-only / both | One advisory each |
| Zero / negative / NaN / Infinity / undefined | No advisory |
| Invalid builder (e.g. QR `signWidth:0`) | No Production SVG size metric, no scale-safety notice, no large-layout advisory |
| Floating display near 400 | `designNumberText` uses `designRound`; 400.01 displays as `400.01` with warning — no “shows 400 but warns” confusion observed |

---

## 5. Exact Production SVG size metric

- Source: `result.widthMm` / `result.heightMm` only (not Finished View bounds, CSS, or path parsing).  
- Displayed via shared `productionSvgSize` prepended to metrics.  
- Root SVG `width` / `height` / `viewBox` independently matched `designNumberText` of result dimensions for all valid probe cases.  
- Redundant per-template **Layout** / **Production sheet** rows removed; template-specific metrics (outside size, usable cavity, piece counts, cleat leg length, etc.) **retained**.  
- Cabinet: primary production size + **Interior production SVG size** when separate interior thickness applies.  

**NOTE (LOW):** Renamed interior metric from “Interior production sheet” → “Interior production SVG size” for consistency; shell-specific second label removed because it duplicated primary production size.

---

## 6. Always-present scale-safety notice

Exact UI text (valid results only):

> **Exact-scale production SVG:** It uses real-world millimeters. Import at 100%; in LightBurn, move or rotate complete parts with their cut outlines, guides, score lines, engravings, and labels together, but do not resize or scale them. Zoom or pan instead; large layouts may require manual arrangement or more than one material sheet.

| Check | Result |
|-------|--------|
| Once per valid modest result | Pass |
| Once per valid large result | Pass |
| Visible in Cut Layout and Finished View modes | Pass (notice lives in results panel, not in preview SVG) |
| Absent for invalid results | Pass |
| Not injected into `result.warnings` | Pass (HTML-only `design-message`) |
| Not in production SVG / download / filename | Pass |
| Misleading claims (fits bed/machine/sheet, auto nest, locked LightBurn, safe to resize) | **None** (honest “does not automatically split or nest” is a negation, not a capability claim) |
| Semantics | Informational `design-message` (not `.warning`) — reduces warning fatigue vs treating 100% scale as an alarm |

---

## 7. Large-layout advisory honesty

Exact advisory pattern:

> Large single-SVG layout: {w} × {h} mm. Verify it against your actual material sheet and machine travel; the Tracker does not automatically split or nest designs across sheets.

| Check | Result |
|-------|--------|
| Includes actual width/height | Pass |
| Generic advisory, not L8 bed claim | Pass |
| No machineProfile / 20W–40W bed inference | Pass |
| No sheet count / auto multi-sheet | Pass |
| Advisory not hard error; download stays enabled | Pass |
| One advisory even if both axes overflow | Pass |
| Useful for flat single-part designs (QR, Hanging Sign) | Pass |

---

## 8. `buildDesignResult` integration

- Applied **after** builder returns; dimensions are final production document bounds.  
- Invalid builders: no fabricated size warning.  
- Warnings array always copied before append (`[...new Set(...)]`).  
- Prior builder-local `Generated layout is …` warnings removed from Finger Box, Sliding-Lid, Drawer Cabinet only (the three that had them).  
- All UI/download paths use `buildDesignResult` / `refreshDesignPreview` / `downloadCurrentDesignSvg`.  

**Direct-builder bypass (NOTE, non-blocking):**  
`buildBoxDesignResult(normalizeDesignDraft(...))` does **not** attach the large-layout advisory; `buildDesignResult` does. SVG bytes identical. Acceptable for geometry fixtures; user-facing paths go through dispatch.

---

## 9. Production-byte neutrality

Independent baseline-versus-working-tree comparison for pinned defaults:

| Template / variant | Hash/length match vs `ad4532c` |
|--------------------|--------------------------------|
| QR stand | Match |
| Hanging sign | Match |
| Dice Tray | Match |
| Divider Tray | Match |
| Finger Box open | Match |
| Finger Box loose lid | Match |
| Finger Box faux dovetail | Match |
| Sliding-Lid Box | Match |
| Drawer Cabinet | Match |
| Joint Fit Coupon finger-edge | Match |
| Joint Fit Coupon wall-to-base | Match |
| Concealed Cleat Corner | Match |
| Concealed Cleat Full-Box | Match |

**`all_goldens_match: true`**

Additional checks:

- Render HTML + switch preview modes → `result.svg` unchanged.  
- No disclosure IDs/classes/text in production SVG.  
- No golden updates required or approved for this phase.

---

## 10. Production SVG dimensions

For all valid probe templates:

`result.widthMm` / `result.heightMm` ≡ root `width`/`height` attributes (mm) ≡ `viewBox` width/height (via `designNumberText`).

Fractional examples (e.g. Sliding-Lid `262.6`, cabinet `292.8`) display without collapsing to integers that would hide meaningful millimeters under the established rounder.

---

## 11. Download-path protection

| Scenario | Same as `result.svg` | No notice/advisory leak | Notes |
|----------|----------------------|-------------------------|-------|
| Modest QR Cut Layout | Pass | Pass | |
| Oversized Full-Box (advisory present) | Pass | Pass | Warning does not block |
| Finger Box Finished View selected | Pass | Pass | Download is production cut SVG, not FV |
| Coupon finger-edge | Pass | Pass | |

`downloadCurrentDesignSvg` body not rewritten for disclosure; still uses production result bytes / multi-output path as before.

---

## 12. Finished View protection

Diff does **not** rewrite:

- `buildFingerBoxFinishedViewSvg`  
- Sliding-Lid finished helpers  
- Drawer Cabinet front elevation  
- Dice / Divider Finished Views  
- Concealed Cleat Finished Views  
- `designPreviewModeForTemplate` / selector / bind / refresh logic (aside from consuming the same `designResultsHtml` disclosure chrome)

Disclosure remains outside the preview SVG; production-size metric remains about the **downloaded Cut Layout** document while FV is selected.

---

## 13. Storage and schema isolation

| Boundary | Status |
|----------|--------|
| `STORAGE_KEY` | Unchanged (`genmitsu-l8-tracker-v1`) |
| `SCHEMA_VERSION` | Unchanged (`2`) |
| `freshState` / `loadState` / `persist` / backup / import-export | Not in intentional diff |
| New draft/backup/export fields | None |

Independent probe: localStorage and `backupObject()` JSON identical before/after large result generation, results HTML render, and Finished View re-render.

---

## 14. Machine-profile independence

Same Full-Box oversized draft under `machineProfile` **20W** vs **40W**:

- Production SVG bytes identical  
- `widthMm` / `heightMm` identical  
- Advisory wording identical  
- No new dependence on `machineProfile` or Reference-tab bed values  

---

## 15. Fixture quality

Designs geometry: **1072 → 1080** (+8 assertions). Net addition covers:

| Behavior | Covered? |
|----------|----------|
| All templates + both coupon modes | Yes (11 paths) |
| Normal vs oversized advisory | Yes |
| Width-only / height-only / both-axis | Yes (dice height-only + both; hanging axes) |
| Exact-400 boundary | Yes (`designLayoutSizeWarning(400,400)===''`) |
| Invalid result | Yes |
| One advisory only | Yes |
| Exact Production SVG size vs root attrs | Yes |
| One scale-safety notice; no fit claims | Yes |
| Production-byte neutrality after render | Yes |
| Download large + FV | Yes |
| Storage / backup isolation | Yes |
| Machine-profile neutrality | Yes |

Fixtures call real builders + `designResultsHtml` + download interceptors — not mere source-string greps.

**Non-blocking optional improvements:**

- Explicit 320 px / a11y assertions not in the +8 set (viewport validated externally in this audit).  
- Height-only overflow not forced for every multi-panel template (documented where layout naturally overflows width first).  

**No commit-blocking fixture gaps** for the disclosure contract.

---

## 16. Direct builder vs shared dispatch

User-facing generation/download uses `buildDesignResult`. Direct builders remain for geometry unit paths; they omit the advisory by design of the shared dispatch. **No user-facing advisory hole** found.

---

## 17. Accessibility and responsive behavior

| Check | Result |
|-------|--------|
| Understandable without color | Text messages, existing `design-message` patterns |
| Notice is visible text, not tooltip-only | Yes |
| No new inaccessible controls | Yes |
| 320 px viewport (headless Edge) | `scrollWidth === clientWidth === 320`, `overflow: false`; notice 1, advisory 1 |
| Warning vs info semantics | Large layout uses `.warning`; scale-safety uses plain informational message — appropriate |
| Screenshots / interactive Edge | **Not** performed; headless Chromium channel `msedge` only |

---

## 18. Documentation accuracy (README)

README updates correctly state:

- Exact Production SVG width/height in real-world mm  
- 100% scale; move/rotate complete parts; keep cut/guides/score/engrave/labels together  
- Do not resize/scale (thickness-dependent geometry)  
- Zoom/pan do not change dimensions  
- `>400` mm generic advisory, not verified bed  
- Verify real sheet and machine travel  
- No automatic nesting / bed-fit / sheet-count / multi-sheet export  
- Fixture totals **1080** Designs / **1872** complete suite  

No claim of LightBurn control or physical validation automation.

---

## 19. Browser validation

| Item | Detail |
|------|--------|
| Browser | Microsoft Edge via Playwright `channel="msedge"`, **headless** |
| Interactive screenshots | **Not** inspected |
| Console page errors during probe | None material |
| 320 px | No horizontal overflow |
| Representative templates in DOM/HTML probes | Modest + oversized QR/Finger Box/Full-Box/coupons; FV modes as listed |
| Physical LightBurn import | Not required; not performed |

---

## 20. Validation commands and exact totals

| Command / check | Result |
|-----------------|--------|
| `git status` / `rev-parse` / ahead-behind | HEAD `ad4532cfa7d5c112a25da4cbd17ac5cefd164e1c`, `main`, `0 0` |
| `git diff --check` | 0 |
| `python -m html.parser index.html` | 0 |
| Designs geometry fixtures | **1080 passed / 0 failed** |
| Tray-model fixtures | **264 passed / 0 failed** |
| Complete 15-group suite (excl. tray) | **1872 passed / 0 failed** (sum of independently re-run groups) |
| Groups + tray (16) | 2136 / 0 (tray remains separately callable; not part of README’s 1872 arithmetic) |
| Baseline vs working-tree production goldens (13 variants) | **All match** |
| Root SVG dimension agreement | Pass (all valid probe templates) |
| Download modest / large / FV / coupon | Pass |
| localStorage + backupObject isolation | Pass |
| machineProfile 20W vs 40W | Pass |
| 320 px viewport | Pass, no overflow |

Evidence artifacts (worktree only, not product tree):  
`agent-tools/scale_safety_audit.json`, `agent-tools/scale_safety_full_suite.json`, `agent-tools/ss_min.py`.

---

## 21. Files and functions reviewed

**Touched product functions (intentional):**

- `designLayoutSizeWarning`  
- `addDesignLayoutSizeWarning`  
- `buildDesignResult`  
- `designResultsHtml` (metrics + `scaleSafetyNotice`)  
- `runDesignGeometryFixtures` (disclosure cases)  

**Protected boundaries inspected (unchanged behaviorally):**

- Production serializers / `result.svg` bytes  
- `downloadCurrentDesignSvg`  
- Finished View builders  
- `STORAGE_KEY`, `SCHEMA_VERSION`, persist/load/backup  
- `machineProfile` / Reference charts  

---

## Blocking findings

**None.**

---

## Non-blocking improvements

1. **LOW — Metric label consolidation:** Shell production sheet label removed; safe because shell dims ≡ primary production size. Consider a one-line README note that multi-file cabinet interiors still show a separate interior size row.  
2. **LOW — Notice length:** Always-present scale-safety text is long on every valid result; intentional for workshop failure mode, but may feel heavy on small designs. Acceptable for this phase.  
3. **NOTE — Direct builder paths:** Geometry fixtures calling builders without `buildDesignResult` will not see the advisory (by design).  
4. **NOTE — Browser depth:** Headless Edge only; no interactive visual screenshot review.  
5. **NOTE — Height-only overflow:** Not every multi-panel template was forced into height-only overflow; natural layout often expands width first. Height-only proven on hanging-sign and dice-tray.  

---

## Unverified areas

- Interactive (non-headless) visual polish of notice placement relative to download buttons.  
- Physical LightBurn import/cut (explicitly out of scope when bytes are identical).  
- Non-English / extremely long numeric localization (not applicable; app uses fixed mm formatting).  

---

## Severity summary

| Severity | Count | Items |
|----------|-------|-------|
| BLOCKER | 0 | — |
| HIGH | 0 | — |
| MEDIUM | 0 | — |
| LOW | 2 | Shell metric label consolidation; always-on notice length |
| NOTE | 3 | Direct-builder advisory gap; headless-only UI; height-only matrix completeness |

---

## Final verdict

**SAFE TO COMMIT WITH NON-BLOCKING NOTES**

The implementation is safe, honest, complete across registered Designs templates (including both coupon modes), behaviorally fixture-backed (+8 Designs assertions; 1080/0 Designs, 264/0 tray, 1872/0 complete suite), production-byte neutral against `ad4532c`, download-safe, Finished View–neutral, storage/schema-neutral, and machine-profile-independent. Ready to commit when the author chooses; no further Claude/Opus/Fable review or physical LightBurn import is required for commit.
