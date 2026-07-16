# Drawer Cabinet Phase 5.1 Cleanup

Date: 2026-07-16

Repository: `C:\Genmitsu L8 Tracker`

Baseline: `67860a6` — `Refine sliding-lid guides and add assembly labels`

## Initial repository state

Before editing, the tracked Drawer Cabinet implementation was uncommitted in:

- `index.html`
- `README.md`

The working tree also contained unrelated untracked files and prior reports. They were preserved without reset, clean, stash, staging, deletion, or modification. The initial Drawer Cabinet implementation passed:

- Designs fixtures: 424 passed / 0 failed
- Complete suite: 956 passed / 0 failed

The initial `git diff --check` passed, with only Git line-ending and inaccessible user-level ignore-file warnings.

## Cleanup implemented

### Encoding-safe Designs output

The corrupted Drawer Cabinet result separators were replaced with ASCII-safe output:

- dimension and layout separators now use `x`;
- the enabled-label count now uses `-`;
- the large-layout warning uses `x`.

No SVG path, geometry, coordinate, formula, panel, layout, ID, or serialization code was changed.

### Template-specific kerf guidance

A small pure `designKerfGuidance(template)` helper now supplies the existing Designs form message:

- Finger Box distinguishes joint clearance from laser kerf.
- Sliding Lid Box distinguishes joint and lid clearances from laser kerf.
- Drawer Cabinet distinguishes cabinet joints, drawer joints, drawer running clearances, and laser kerf.
- Other Designs templates retain concise general fit-clearance guidance.

Every variant states that kerf must be established through material testing and is not automatically included in generated dimensions.

### Regression fixtures

Four focused Designs assertions were added:

1. Drawer Cabinet result markup contains neither known mojibake sequence.
2. Drawer Cabinet kerf guidance contains cabinet-specific wording and no lid wording.
3. Sliding Lid Box retains lid-specific guidance.
4. Finger Box retains joint-specific guidance.

The existing geometry-signature assertion was clarified to explicitly cover Finger Box, Sliding Lid Box, and deterministic Drawer Cabinet SVG output. Its established hashes, lengths, and geometry conditions remain unchanged.

`README.md` was changed only to update the verified complete fixture total from 956 to 960.

## Validation

### Static checks

- `git diff --check`: passed
- Python `html.parser` validation of `index.html`: passed
- Source scan for the two known mojibake sequences: 0 occurrences each

### Direct local Edge fixture run

The complete fixture suite was executed in Microsoft Edge from:

`file:///C:/Genmitsu%20L8%20Tracker/index.html`

| Fixture group | Passed | Failed |
|---|---:|---:|
| Baseline resolution | 20 | 0 |
| Material test normalization | 12 | 0 |
| Test Grid promotion | 23 | 0 |
| Grid Browser | 67 | 0 |
| Material Browser | 57 | 0 |
| Library Browser | 56 | 0 |
| Project Browser | 61 | 0 |
| Wizard metadata | 12 | 0 |
| Storage recovery | 8 | 0 |
| Project Wizard | 216 | 0 |
| Designs | 428 | 0 |
| **Complete suite** | **960** | **0** |

### Direct-file UI checks

A fresh direct-file Edge session confirmed:

- the page reached `readyState: complete`;
- tabs were visible;
- Designs rendered normally;
- Finger Box displayed joint-specific kerf guidance;
- Sliding Lid Box displayed lid-specific kerf guidance;
- Drawer Cabinet displayed cabinet/drawer-specific kerf guidance with no lid language;
- rendered Drawer Cabinet results contained neither known mojibake sequence;
- switching among the three templates did not change the application storage value.

Normal `index.html` was used without a self-test query parameter, preserving ordinary direct-file startup behavior.

## Scope confirmation

No changes were made to:

- cabinet or drawer geometry;
- dimensional formulas;
- finger-joint phases;
- piece IDs or panel ordering;
- layout behavior;
- SVG serialization;
- clearances;
- storage keys or schema;
- persistence or localStorage behavior;
- backup/import/export behavior;
- dependencies.

No physical plywood cut, LightBurn import, or visible interactive desktop-browser session was performed. Automated direct-file Microsoft Edge rendering and interaction were completed.

Nothing was staged, committed, or pushed.

READY FOR REVIEW
