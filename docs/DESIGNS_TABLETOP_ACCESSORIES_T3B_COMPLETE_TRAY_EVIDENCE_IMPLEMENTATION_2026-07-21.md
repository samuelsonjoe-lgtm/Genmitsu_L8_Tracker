# T3B-E1 Complete-Tray Evidence Implementation

Date: 2026-07-21

## Initial state

- HEAD: `c16324b Add T3B tabletop storage tray` on `main`, synchronized with `origin/main`.
- The tracked tree and index were clean before this work. Existing untracked reports, `LightBurn Projects/`, `debug.log`, utilities, and other user files were preserved.

## Applied architecture

This phase adds the independent `tabletopTrayResults` raw-result collection and increments `SCHEMA_VERSION` from 3 to 4. The new identities are:

- Evidence kind/source type: `tabletop-storage-tray`
- Internal/user-facing stage: `complete-tray-proven` / `Complete Tray-proven`
- Template/generator: `tabletop-storage-tray` / `tabletop-storage-tray-t1`
- Construction: `t2-floor-owned-shell-plus-bottom-supported-four-rail-removable-panel-t1`

The new normalizer is declared before `loadState()`. It preserves unknown record fields, normalizes tri-state observations, leaves blank optional measurements as `null`, filters malformed/duplicate records, and recomputes the derived false-bottom and rail facts with the established `1e-6` integrity check.

## Workflow and provenance

- The Designs T3B result now shows four separate boundaries: T2 outer shell, T3A storage mechanism, T3B complete tray, and cork fit.
- A dedicated recorder saves a contemporaneous or retrospective raw Tabletop Storage Tray result without promoting it. All observation controls begin as Unknown / not recorded.
- Promotion requires a pass, date, exact machine and material identities, exact T3B identity/geometry, all required-positive observations true, all disqualifiers false, and currently existing exact promoted T2 and T3A Library evidence.
- Explicit confirmation is required. Promotion adds an Evidence-only Library evidence record and captures immutable T2/T3A result, setting, evidence, machine, configuration, and stage snapshots. Later component-result deletion does not rewrite the complete-tray evidence.
- The dedicated matcher requires the complete kind/stage plus exact machine, material, dimensions, nominal/measured thickness, signed clearance, preferred finger width, all three actual finger widths, rail height, false-bottom thickness, and panel clearance. It does not change the T2 or T3A matchers.

Complete Tray-proven does not prove cork fit, load capacity, long-term durability, Production-proven status, or a different configuration.

## Persistence and handoff

`tabletopTrayResults` is included in fresh state, startup normalization, persistence, backups, Replace import, Merge import, and merge statistics. Schema-4 builds accept v1-v3 data; older schema-3 builds reject schema-4 backups through the existing future-version guard. No Project or Library schema changed.

Project handoff notes now distinguish T2 outer-shell, T3A storage-mechanism, T3B complete-tray, and cork boundaries. Creating a Project neither saves nor promotes evidence.

## Validation

- `git diff --check`: passed.
- Python `html.parser`: passed.
- Edge headless direct-file route with a disposable profile: `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=tabletop-tray-results` passed `56 / 0`; no JavaScript exception after the final correction.
- Normal supported `?selftest=all` route completed with every emitted group reporting `0 failed`, including T2/T3A/T3B, new T3B-E1, backup/state, Designs-to-Projects, and Designs geometry/golden coverage.
- Current unique-suite accounting is `3032 / 0` across 45 registered groups. The page's normal all-route logs individual group totals rather than a separate aggregate line.
- T3B generator remains `36 / 0`; T3B-E1 is `66 / 0`; Designs geometry is `1093 / 0`.

The all-route retains the existing production-golden coverage. Confirmed protected pins include legacy shell `2181 / 2ef9606b`, T1 coupon `2992 / 4f543f95`, T2 shell `2337 / ed5d6f6e`, T3A storage-fit coupon `897 / b5c549ee`, T3B tray `2860 / 3e256fad`, Dice Tray `1726 / 51a55721`, alternate Dice Tray `1054 / 41697123`, Divider Tray `1965 / a55dda6e`, and wall-to-base coupon `1551 / d9ffc278`.

## Protected boundaries and remaining validation

No protected production geometry, SVG serializer, T2/T3A matcher, generic promotion helper, Project schema, Library schema, backup format, or application identity was changed. No physical evidence was created or promoted for Joe.

The browser validation was automated/headless Edge with disposable local storage. It verified startup, focused T3B-E1 behavior, and the normal all-route; an interactive manual click-through and physical T3B assembly remain external validation. The focused independent audit and its bounded corrections are complete. Joe may safely enter and explicitly promote his real result with the required observations completed truthfully.

## Post-audit corrections — 2026-07-22

- **Promotion gate:** `tabletopTrayLiveComponentMatches` now uses only the existing exact T2 Shell-proven and T3A Storage Fit Coupon-proven Library-evidence matchers. Deleting either underlying raw component result no longer blocks a later T3B promotion when its exact promoted Library evidence remains; missing, removed, stale, wrong-machine, wrong-material, wrong-stage, or configuration-mismatched evidence still blocks. Promotion after raw-component deletion retains the evidence IDs and frozen component snapshots on both the new Library evidence and the promoted raw tray record.
- **Recorder language:** added a dedicated, complete 29-key observation-label map. The recorder and promotion blockers use concise human-readable labels while retaining the serialized observation keys and their meanings unchanged.
- **Promoted View:** the existing View action now opens a clearly labeled read-only recorder/details modal. It shows the saved facts, source mode, date, observations, optional measurements, notes, disposition, promoted evidence reference, and frozen T2/T3A provenance; editable controls and the save action are absent. Unpromoted records remain editable.
- **Fresh validation:** `git diff --check` and Python `html.parser` passed. Direct-file Edge/Playwright focused totals were Modal Accessibility `37 / 0`, T2 promotion `16 / 0`, T2 matching `16 / 0`, T3A evidence `14 / 0`, T3A coupon `23 / 0`, T3B generator `36 / 0`, T3B-E1 `66 / 0`, storage `15 / 0`, and Designs geometry/goldens `1093 / 0`. The normal `?selftest=all` route completed with zero fixture failures and zero page exceptions.
- **Total reconciliation:** the independent audit corrected the prior `3020` report total to `3022 / 0` because the two new T3B modals organically added two Modal Accessibility assertions. This correction pass adds ten focused T3B-E1 assertions (56 → 66), so the current unique all-route accounting is `3032 / 0` across 45 registered groups.
- **Browser notes:** a disposable direct-file UI check opened the T3B recorder, confirmed rendered human-readable labels, and confirmed Escape closes the modal. The normal all-route had no page exceptions; it did emit four pre-existing renderer console messages, `Error: <svg> attribute height: Expected length, "auto".`, while Designs geometry remained `1093 / 0`. Production geometry and the registered golden bytes above remain unchanged.
