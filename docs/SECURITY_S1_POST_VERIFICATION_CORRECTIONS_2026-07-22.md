# Security S1 Post-Verification Corrections

Date: 2026-07-22
Repository: `C:\Genmitsu L8 Tracker`

## Repository state and inspected diff

Actual HEAD was `19d4834 Graduate tabletop storage tray to Workshop-ready`. `main` was synchronized with `origin/main`; nothing was staged. Before correction, the complete tracked diff was the uncommitted S1-only `index.html` addition (+221 lines), and the relevant untracked files were the S1 architecture, implementation, and independent-verification reports alongside pre-existing historical docs, `.claude/`, LightBurn projects, `debug.log`, and the utility script. All were preserved.

Read completely: the current `index.html`, the S1 architecture review, implementation report, independent post-implementation verification, full dispatcher, S1 fixture/promise references, encrypted-envelope detection/validation, and normal `?selftest=all` control flow. The historical reports were not edited.

## Corrections

### 1. Awaited S1 completion

The synchronous dispatcher is now contained in `runRequestedSelftests(selftest)`. Existing synchronous fixture groups retain their ordering. At the existing S1 registration point, the dispatcher:

1. starts `runSecurityS1CryptoFixtures()` once;
2. assigns its actual promise to `window.securityS1CryptoFixturePromise`;
3. awaits that exact promise before continuing to Designs or resolving `window.selftestCompletionPromise`;
4. records returned fixture failures in the completion result; and
5. catches a rejected S1 promise, logs it as a fixture rejection, and replaces it with a resolved `{ passed: 0, failed: 1 }` result.

`window.selftestCompletionPromise` now resolves only after every registered route has run and includes its aggregated failure count. No query means no self-test runner is invoked, so ordinary application startup does not run PBKDF2.

### 2. Own-property encrypted detection

`isEncryptedBackupEnvelope` now requires a non-array object with an own `backupFormat` property whose value exactly equals `genmitsu-l8-tracker-encrypted-backup-v1`.

The existing S1 detection assertion was strengthened in place, so the group remains 21 assertions. It verifies an inherited marker is false; an own marker is true before strict validation; a malformed own-marker object is detected but rejected by strict validation; and plaintext format, prefixes, capitalization changes, arrays, strings, `null`, and `envelopeVersion` alone are false. No validation rule, crypto policy, AAD field, or envelope serialization changed.

The fixture also increments the self-test-only `window.securityS1CryptoFixtureRunCount`, allowing direct confirmation that the route invokes S1 exactly once.

## Validation

- `git diff --check`: pass.
- `python -m html.parser index.html`: pass.
- Inline JavaScript parsed and executed under direct `file://` Edge.
- Focused `?selftest=security-s1-crypto`: **21 passed / 0 failed**; reload repeated **21/0**.
- Duplicate IDs: none. DOMParser malformed-markup errors: zero.
- Normal `?selftest=all`: the application-owned completion promise was **not** resolved at the first microtask check (`completedBeforeAwait: false`); after it resolved, it reported `{ completed: true, failed: 0 }`. S1 reported **21/0** and `securityS1CryptoFixtureRunCount: 1`.
- All 47 registered dispatcher routes executed once through that normal path. Storage recovery, backup validation/Merge/Replace, startup/normalization/migration, modal, Project/Pricing/Inventory/Library/machine/evidence persistence, and Designs geometry/production fixtures all emitted zero failed results.
- Designs geometry and registered production goldens: **1093 passed / 0 failed**. No generator, serializer, SVG, or production-output code changed.
- The established independently collected unique-suite result remains **3024 passed / 0 failed across 47 groups**: this correction adds/removes no assertion and the direct normal route now waits for the existing 21/0 S1 group before completion. The historical unique-suite accounting intentionally does not double-count nested Material Browser diagnostics.

## Direct-file Edge result

Microsoft Edge `150.0.4078.83` was run headlessly with disposable data at `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=security-s1-crypto` and `?selftest=all`.

The focused probe found `crypto` and `crypto.subtle` available, completed native PBKDF2/AES-GCM in 95.5 ms, and produced no console or page errors. The S1 fixture confirmed localStorage/application-state isolation. Ordinary direct-file startup, instrumented with a disposable `SubtleCrypto.prototype.deriveKey` counter, made **0** derivation calls and exposed neither self-test promise.

The normal all route had no page errors and no S1-related console errors. It emitted four pre-existing Designs preview diagnostics, `Error: <svg> attribute height: Expected length, "auto"`; these occur outside S1 and were not changed. Chrome remains unavailable and was not installed or inferred.

## Protected boundaries and remaining limitations

Only `index.html` and this report changed. Export/Import, passphrase UI, vaults, storage/schema, plaintext backups, promotion/evidence, geometry, production SVGs, app constants, and normal non-self-test behavior remain untouched. No real profiles, records, backups, or passphrases were accessed. Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

The rejected-promise conversion is covered by its explicit dispatcher branch but was not force-injected in a product browser run. Chrome direct-file Web Crypto remains unverified because Chrome is not installed. S2 encryption/export/import and user-facing passphrase policy remain intentionally out of scope.

S1 CORRECTIONS COMPLETE WITH CAUTION — Chrome is not installed for independent direct-file verification.
