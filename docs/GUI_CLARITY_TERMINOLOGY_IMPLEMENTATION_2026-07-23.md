# GUI Clarity Terminology Implementation - 2026-07-23

## Scope and repository state

Implemented the bounded terminology-only phase from section 18 of `GUI_CLARITY_ARCHITECTURE_AND_USABILITY_REVIEW_2026-07-23.md` against HEAD `994510b Improve inventory organization and material cleanup` on `main` (synchronized with `origin/main` before this work). The change is presentation-only: no stored keys, normalization, storage schema, backup format, production calculations, SVG serialization, or generated output changed.

## Terminology and placement

The existing `focusValue` / `focusUnit` fields now have the exact primary label **Machine Focus Distance**. In the shared Log and Library form renderer, the existing `materialHeightValue` / `materialHeightUnit` fields now have the exact primary label **Intentional Z Offset**.

The shared Log and Library forms now render, in this order:

1. Thickness
2. Machine Focus Distance
3. Intentional Z Offset
4. Power and speed controls

Machine Focus Distance help states that it records the normal machine or lens focus distance to the material surface and is not an additional Z adjustment. Intentional Z Offset help states that it is a deliberate adjustment above or below normal focus and should otherwise be left blank.

Project settings and the Production Settings editor use Machine Focus Distance with that focus-distance help, but do not add an Intentional Z Offset field. The pre-existing Material Test Wizard `zOffset` field remains exactly `Z offset (mm)` and is not connected to these stored setting fields.

## Updated presentation surfaces

- Log and Library shared forms: labels, help, and field order.
- Project settings form: Machine Focus Distance label and help only.
- Production Settings editor: Machine Focus Distance label and help only.
- Log, Library, and Project cards; Library Browser detail; Project Browser settings metric; and Designs production-reference summary.
- Project Wizard copied-settings review.
- Material Test and Project promotion-review field labels.

These changes reuse the existing `focusValue`, `focusUnit`, `materialHeightValue`, and `materialHeightUnit` names and values. Legacy blanks remain blank; import Merge and Replace preserve the existing fields unchanged. Rendering the terminology does not write localStorage, alter backup JSON, add vault fields, or alter production SVG bytes.

## Fixture coverage

Extended the existing cumulative `runBeginnerClarityFixtures` group rather than creating a new dispatcher. The group now verifies:

- exact labels for the shared Log and Library forms;
- DOM order from Thickness through power and speed;
- distinguishing help text;
- unchanged input names and values;
- Project and Production omission of Intentional Z Offset;
- Project-promotion labels;
- card, browser, and Design summary wording;
- the separate Wizard `Z offset (mm)` label;
- legacy blank values;
- Merge and Replace preservation; and
- storage, backup, vault, and SVG isolation.

Observed focused results:

| Group | Result |
| --- | --- |
| Beginner clarity | 33 / 0 |
| Modal accessibility | 38 / 0 |
| Accessibility polish | 36 / 0 |
| Responsive tuning | 45 / 0 |
| Library Browser | 56 / 0 |
| Project Browser | 61 / 0 |
| Evidence Promotion | 58 / 0 |
| Production Settings | 66 / 0 |
| Storage recovery | 15 / 0 |
| Designs geometry and production goldens | 1093 / 0 |
| Complete registered suite | 3171 / 0 |

The complete result was captured through the normal `?selftest=all` sequence in disposable direct-file Edge state. An in-memory observer counted the aggregate only; it did not alter the product file or fixture scheduling.

## Direct-file UI checks

Headless Microsoft Edge opened the real direct-file page with disposable localStorage at 360, 768, 1024, and 1440 px widths. The Log, Library, Project, and Production forms showed the expected labels and help. The changed focus/offset controls remained within their form bounds at every width, Project and Production contained no Z-offset control, and the Log form order was Thickness -> Machine Focus Distance -> Intentional Z Offset -> power/speed. No page errors occurred.

The pre-existing Project action row is 14 px wider than its form at 360 px; the changed Project focus controls themselves fit, so this presentation-only change did not introduce that existing narrow-width behavior.

## Validation and remaining checks

`git diff --check` and `python -m html.parser index.html` passed. The complete run had zero fixture failures. Edge emitted four existing `<svg> attribute height: Expected length, "auto"` console messages during the complete run; no page errors occurred, focused Designs fixtures passed 1093 / 0, and no production output changed.

No manual laser operation is required for this terminology-only update. Direct visual review in a non-headless browser remains optional before commit; no files were staged, committed, or pushed.
