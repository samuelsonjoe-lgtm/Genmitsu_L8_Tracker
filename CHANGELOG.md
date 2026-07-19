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
