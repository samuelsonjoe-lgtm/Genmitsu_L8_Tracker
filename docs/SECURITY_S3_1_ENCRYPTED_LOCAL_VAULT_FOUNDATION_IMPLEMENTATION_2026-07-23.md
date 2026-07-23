# Security S3-1 — Encrypted Local Vault Foundation

Date: 2026-07-23  
Baseline: `386e1e0 Add encrypted backup export and import` on synchronized `main`.

## Implemented boundary

Only `index.html` and this report changed. S3-1 adds versioned vault constants (`VAULT_STORAGE_KEY`, `VAULT_META_KEY`, `VAULT_PENDING_KEY`, `VAULT_FORMAT`, version 1), a separate strict live-vault envelope, bootstrap metadata parsing, synchronous authoritative-mode detection before plaintext `loadState()`, a record-free locked shell, accessible passphrase unlock, an internal-only lock cleanup, and a serialized encrypted persistence path.

The vault envelope requires exactly `app`, `appVersion`, `vaultFormat`, `vaultEnvelopeVersion`, `createdAt`, `writeGeneration`, `crypto`, `ciphertextB64`, and `innerSchemaVersion`; `crypto` requires exactly `kdf`, `kdfIterations`, `cipher`, `saltB64`, and `ivB64`. AAD is deterministic JSON in this order: app, appVersion, vaultFormat, vaultEnvelopeVersion, createdAt, writeGeneration, crypto.kdf, crypto.kdfIterations, crypto.cipher, crypto.saltB64, crypto.ivB64, innerSchemaVersion. `appVersion` is therefore authenticated while its seeded origin value is retained across writes.

The S1 primitives are reused unchanged: strict Base64/UTF-8, native Web Crypto, PBKDF2-SHA-256 at 600000–2000000 iterations, AES-256-GCM, 16-byte salt, fresh 12-byte IV per write, and non-extractable keys. `writeGeneration` is an authenticated stale-writer ordering guard only; it does not prevent replacement of a complete old valid envelope/metadata pair.

Valid vault ciphertext is authoritative and starts locked without calling plaintext `loadState()`. Invalid/foreign/future vault data is blocked without mutation. Plaintext startup remains the existing path when no vault exists. Metadata is non-secret and not trusted for cryptographic decisions. Unlock validates the envelope before PBKDF2, rejects only exact empty passphrases, does not trim/normalize, installs state only after authenticated decrypt plus normalization, and retains only the derived CryptoKey/session context in memory.

`persist()` remains synchronous and boolean: plaintext behavior remains its existing write path; locked/blocked states refuse; an unlocked vault queues an encrypted latest-wins write and returns true only when accepted by the coordinator. It never falls back to plaintext. A newer on-disk generation causes stale-writer refusal. Internal lock is deliberately not user-reachable and clears state/session/DOM only after queued writes drain.

## Explicitly deferred

No user-reachable Enable, Lock, Recovery, Restore, Disable, migration, pending-key creation, vault download, reset, passphrase rotation, timeout, or cloud/account feature exists. S3-2 recovery-first and S3-3 enrollment/manual lock remain required before users can enable a vault.

## Validation

- `git diff --check`: pass.
- `python -m html.parser index.html`: pass.
- Disposable direct-file Edge (headless, fresh `--user-data-dir`): focused `?selftest=security-s3-local-vault` **9 passed / 0 failed**; validated strict envelope, AAD-bound appVersion tamper failure, locked no-record shell, wrong-passphrase non-mutation, correct unlock, encrypted persist/generation, and internal lock cleanup.
- Normal direct-file `?selftest=all`: `{ completed: true, failed: 0 }`.

The new route is `?selftest=security-s3-local-vault`, promise `window.securityS3LocalVaultFixturePromise`, with one run counter. It is awaited by the normal dispatcher. Existing S1/S2 formats, plaintext backups, SVG generation, and schemas were not changed. The normal suite had zero failures; the focused S3 group contributes 9 assertions, so the reconciled unique total is **3077 passed / 0 failed across 49 registered groups** (3068 baseline + 9).

Chrome was not installed or inferred. Large-photo performance, full multi-tab adversarial testing, quota fault injection, recovery flows, and independent browser verification remain unverified. Production Designs output was not modified; the normal complete suite returned zero failures. Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

S3-1 FOUNDATION COMPLETE WITH CAUTION — bounded direct-file verification is green; recovery-first/enrollment phases and broader adversarial validation remain required before any user-reachable vault control.
