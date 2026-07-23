# Security S3-1 — Encrypted Local Vault Foundation Corrections

Date: 2026-07-23  
Repository: `C:\Genmitsu L8 Tracker`  
Baseline: `386e1e0 Add encrypted backup export and import` (`main...origin/main` synchronized)  
Scope: required post-verification corrections only; no staging, commit, or push.

## Corrections applied

### H1 — interrupted transition authority

`vaultBootstrapDetect()` now treats valid plaintext under `STORAGE_KEY` as authoritative when vault metadata says `mode: "vault"` but no vault ciphertext exists. It leaves metadata and the pending marker untouched, loads the plaintext records, and allows the normal plaintext `persist()` path to continue using the existing document format.

The plaintext guard still fails closed when a vault ciphertext appears before a plaintext write: it clears the stale in-memory state and renders the blocked vault shell instead of writing plaintext over the new vault.

### H2 — one production vault implementation

There is now exactly one production definition each of `validateVaultEnvelopeV1`, `vaultReadMetadata`, `vaultBootstrapDetect`, `vaultEncryptWithKey`, and `vaultDecryptWithPassphrase`. The shared implementation now enforces:

- canonical UTC ISO `createdAt` values;
- exact envelope and metadata field policy;
- structural envelope refusal before PBKDF2;
- capability refusal before PBKDF2 when native Web Crypto is unavailable;
- pre-encryption envelope validation; and
- decrypted `schemaVersion` equality with `innerSchemaVersion`.

### Encrypted persistence retry

The encrypted persistence coordinator no longer leaves a failure permanently non-retriable. A later ordinary `persist()` accepts the retry, preserves the previous ciphertext until the replacement write succeeds, keeps the failure status visible, uses a new IV for the replacement encryption, and never falls back to plaintext.

## Fixture coverage

The existing `Security S3 Local Vault` group (`?selftest=security-s3-local-vault`) was expanded without registering another group. Its 14 assertions now include:

- metadata-only interrupted transition persistence and a vault-appears-before-plaintext-write block;
- non-canonical timestamp and malformed metadata refusal with zero PBKDF2 derivations;
- unavailable Web Crypto and mismatched decrypted/inner schema refusal;
- encrypted write-failure retention followed by a successful fresh-IV retry; and
- the existing locked bootstrap, exact-passphrase, wrong-passphrase, persist, and internal-lock coverage.

The write-failure injection is fixture-only through the vault write seam; production behavior, schemas, storage keys, import/export, and non-vault persistence format are unchanged.

## Validation

| Check | Result |
| --- | --- |
| `git diff --check` | Pass |
| `python -m html.parser index.html` | Pass |
| Inline JavaScript | Parsed and executed by direct Edge `file://` validation |
| Security S1 crypto | 21 passed / 0 failed |
| Security S2 encrypted backup | 17 passed / 0 failed |
| Security S3 local vault | 14 passed / 0 failed |
| Designs geometry / protected production outputs | 1093 passed / 0 failed |
| Normal complete runner | 49 registered groups, 0 failed |
| Reconciled unique complete aggregate | 3082 passed / 0 failed (`3068` non-S3 assertions + `14` S3 assertions) |

Direct disposable Edge `file://` execution covered the interrupted transition, locked bootstrap, non-canonical timestamp refusal, unavailable capability refusal, schema mismatch refusal, injected encrypted-write failure and retry, correct unlock/reload path, and the normal complete runner. No page exceptions or console errors were recorded. Production geometry fixtures remained green; no SVG, generator, evidence, storage-schema, import/export, or backup-format changes were made.

## Readiness

The post-verification findings are corrected and the S3-1 working tree is ready for the requested review/commit decision. Remaining scope is intentional S3-2 user-facing vault enrollment, recovery/restore, disable-vault, and lifecycle work; this pass does not introduce those capabilities.
