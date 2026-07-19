# M1 — Multi-machine storage model and legacy migration

Date: 2026-07-19  
Baseline: `288d474 Improve beginner clarity and import safety`

## Scope and outcome

M1 adds only the persisted multi-machine foundation. It introduces a root `machines` collection and `activeMachineId`, retains `machineIdentity` as a compatibility mirror, and leaves machine management, selection, historical record linkage, production routing, and UI redesign for later phases.

## Root model and normalization

Each machine is an owned record with a mandatory stable `id`, trimmed `name`, `manufacturer`, `model`, and `notes`, allowed laser `type` (`diode`, `co2`, `fiber`, or `other`), finite positive-or-null `nominalPowerW`, paired positive-or-null `workAreaWidthMm` / `workAreaHeightMm`, and boolean `archived`. Unknown additive fields survive normalization. Invalid entries are dropped; duplicate IDs keep the first valid entry in stable source order. The active ID resolves to its requested non-archived record when possible, then the first non-archived record, then the first archived record, or empty when no record exists.

## Compatibility mirror and separate profile

`machineIdentity` remains persisted for existing consumers, but is synchronized only from the active stored machine's name and advisory work area. It does not receive advanced machine fields and does not replace `machineProfile`, which remains a distinct Reference-profile selection. Existing Test Grid snapshots, Project records, Library records, Logs, promotions, and other historical objects are unchanged and receive no new `machineId`.

## Legacy migration and recovery behavior

Root migration is decided from the raw payload, before normalization. If it has no own `machines` property, its legacy `machineIdentity` migrates exactly once into `legacy-workshop-machine`, including a blank legacy identity. A payload that owns `machines` but supplies an empty, malformed, or wholly invalid value stays machine-less rather than fabricating a legacy record. Loading performs this in memory without immediately writing storage, and a migrated backup reloads idempotently as the new model. Existing malformed-localStorage recovery behavior and storage constants are unchanged.

## Persistence, backup, import, and editor behavior

`persist()` writes `machines`, `activeMachineId`, and the compatibility mirror under the existing `genmitsu-l8-tracker-v1` key. `backupObject()` exports the same root fields with the existing app, version, schema, and backup metadata; no backup format, schema, or version constant changed. Merge imports explicit incoming machines by ID, retain local records on collisions, append only incoming IDs not present locally, keep local order and active selection, and never synthesize a record from an imported legacy mirror. Replace applies the imported root model with the same legacy-absence rule. The existing Current workshop machine form now updates the active machine while preserving advanced fields, or creates one UID-backed record when the collection is empty; clearing its ordinary fields keeps that record.

## Boundaries retained

No manager or selector UI, deletion/archive controls, cross-machine duplication, manual history reassignment, or policy-based multi-machine filtering was added. No production SVG generation, preview geometry, downloads, Test Grid/Project/Library/Log schema, promotion behavior, `machineProfile`, `activeMachineSnapshot`, storage key, schema version, import/export format, app/build identity, or external dependency changed. Designs production output remains byte-identical because its builders were not changed.

## Fixture coverage

`runMultiMachineStorageFixtures()` is reachable with `?selftest=machines-m1` and runs once in `?selftest=all`. Its 29 assertions cover: clean-root initialization and onboarding; custom and blank legacy migration; idempotence; explicitly present empty/malformed machine roots; normalization, duplicate handling, additive-field preservation, lookup, and active fallback; mirror synchronization; unchanged historical snapshots; backup root fields and constants; real Reference-form active update/create behavior; Merge conflict, order, archive, active, and legacy-import handling; Replace new-root, archived fallback, legacy, empty, and malformed behavior; and no persistence merely from helper normalization.

## Validation

- `git diff --check` passed.
- `python -m html.parser index.html` passed.
- Direct `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=machines-m1` reported `29 passed / 0 failed` with no uncaught exceptions.
- Direct `?selftest=all` reported `2234 passed / 0 failed` across 25 groups: `1141` non-Design plus `1093` Designs assertions.
- Existing machine setup fixtures reported `50 / 0`; separately callable Tray-model fixtures remained `264 / 0`.

## Remaining validation

M1 does not claim physical machine verification, production safety, or a finished multi-machine workflow. Later phases still need a visible manager/selector, intentional cross-record identity routing, and user validation of any future switching or archival UI.
