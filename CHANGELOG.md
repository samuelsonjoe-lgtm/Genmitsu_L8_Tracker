# Changelog

## 0.9.0 — 2026-07-19

- Added visible application version and build identity.
- Added machine-readable app, version, and backup-format metadata to exported backups.
- Added safeguards against future-schema and incompatible backup imports.
- Preserved compatibility with legacy backups that lack the new metadata.
- Added a session-only first-run onboarding panel on an empty Log tab that explains local browser storage, regular Export backups, and starter actions for Log, Test Grids, and Reference.
- Added centralized accessible modal-dialog semantics, focus containment, Escape handling, and practical focus restoration without changing modal workflows or persisted data.
