# Changelog

## 0.9.0 — 2026-07-19

- Added visible application version and build identity.
- Added machine-readable app, version, and backup-format metadata to exported backups.
- Added safeguards against future-schema and incompatible backup imports.
- Preserved compatibility with legacy backups that lack the new metadata.
- Added a session-only first-run onboarding panel on an empty Log tab that explains local browser storage, regular Export backups, and starter actions for Log, Test Grids, and Reference.
- Added centralized accessible modal-dialog semantics, focus containment, Escape handling, and practical focus restoration without changing modal workflows or persisted data.
- Added one offline Help and Safety modal with local-backup, LightBurn exact-scale, safety, and physical-validation guidance, reachable from the header and Reference tab.
- Added an optional Current workshop machine setup on the Reference tab (machine name and advisory usable work-area width/height in millimeters) and clarified that the 20W/40W tables are a Genmitsu L8 Reference profile of manufacturer starting points; the machine setup is additive, advisory only, kept locally during Merge import, restored by Replace import, and does not change designs, production output, or Reference values.
- Added accessible primary-tab semantics and roving keyboard navigation, a clear keyboard focus indicator, bounded touch-target sizing, and focused accessibility fixtures without changing stored data or application workflows.
- Improved narrow-screen wrapping, form/modal stacking, and local horizontal scrolling for wide tables, Test Grid matrices, and Designs previews without changing desktop workflows or production output.
- Added neutral empty-state actions for first records and filter misses, plus safe unavailable-reference presentation that preserves historical data without automatic repair, reassignment, or schema changes.
- Added beginner-clarity guidance: explicit Merge/Replace/Cancel import choices with a second Replace confirmation, clearer Test Grid and Material Test workflow language, current-machine/Reference/Grid-snapshot distinctions, operation-aware Designs guidance, and empty-Library ordering. This does not change storage schema, SVG output, downloads, or laser control behavior.
- Added M1 multi-machine storage groundwork: root `machines` and `activeMachineId` fields, one-time legacy migration to a fixed compatibility record, an active-machine `machineIdentity` mirror, and ID-keyed Merge/Replace handling. The current Reference form edits the active machine or creates one; machine management, selection, and historical record linkage remain deferred.
