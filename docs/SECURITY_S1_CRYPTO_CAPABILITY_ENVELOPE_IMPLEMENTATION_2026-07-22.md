# Security S1 — Crypto Capability and Envelope Implementation

Date: 2026-07-22
Repository: `C:\Genmitsu L8 Tracker`
Baseline: `19d4834 Graduate tabletop storage tray to Workshop-ready` on `main`, synchronized with `origin/main` before this work.

## Scope and repository state

Only `index.html` and this new report were changed. The tracked worktree was clean before S1; no files were staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted. Pre-existing untracked files and reports were left untouched.

S1 is a disconnected capability/envelope foundation. It does not call or alter `backupObject`, `validateBackupImport`, `applyBackupImport`, `persist`, `loadState`, Export, Import, local storage, schemas, UI, downloads, filenames, or live data. The sole new route is `?selftest=security-s1-crypto`; it uses a disposable synthetic plaintext object and confirms state, localStorage, and Export/Import handler identities remain unchanged.

## Implementation

Added immutable policy constants:

| Constant | Value |
| --- | --- |
| `ENCRYPTED_BACKUP_FORMAT` | `genmitsu-l8-tracker-encrypted-backup-v1` |
| envelope version | `1` |
| KDF | `PBKDF2-SHA256` (PBKDF2-HMAC-SHA-256) |
| KDF default/minimum | `600000` |
| KDF maximum | `2000000` |
| cipher | `AES-256-GCM`, 256-bit key, 128-bit tag |
| salt | exactly 16 random bytes |
| IV | exactly 12 random bytes |

Added isolated helpers for capability probing, strict UTF-8, chunked Base64 conversion, exact envelope validation, nonextractable PBKDF2-derived AES keys, authenticated synthetic encryption/decryption, and the S1 fixture. `deriveEncryptedBackupKey` creates a nonextractable key (`extractable: false`) and only accepts the exact supplied Unicode passphrase; it does not trim, normalize, log, retain, or persist it.

The capability probe does real `getRandomValues`, `importKey`, PBKDF2/SHA-256 `deriveKey`, AES-GCM encrypt, and AES-GCM decrypt operations. Its injected `cryptoApi` seam lets an unavailable platform return an explicit unavailable result without adding a fallback.

The Base64 encoder converts bytes in 32 KiB chunks, avoiding spread-argument conversion of large typed arrays. The focused fixture successfully round-trips a synthetic 1 MiB payload.

## Envelope contract

Encrypted detection is deliberately exact: an object is an encrypted envelope only when `backupFormat === ENCRYPTED_BACKUP_FORMAT`. `envelopeVersion` by itself is never a discriminator.

Version 1 requires exactly these outer fields: `app`, `appVersion`, `backupFormat`, `envelopeVersion`, `createdAt`, `crypto`, `ciphertextB64`, `innerSchemaVersion`, and `plaintextBackupFormat`. `crypto` requires exactly `kdf`, `kdfIterations`, `cipher`, `saltB64`, and `ivB64`. Unknown or missing fields, a foreign app, unsupported version/KDF/cipher, KDF values outside 600,000–2,000,000, invalid Base64, wrong salt/IV lengths, missing/truncated ciphertext, and invalid/future inner schemas all fail closed.

AES-GCM additional authenticated data is the UTF-8 encoding of a deterministic JSON list in this fixed order:

`app`, `appVersion`, `backupFormat`, `envelopeVersion`, `createdAt`, `crypto.kdf`, `crypto.kdfIterations`, `crypto.cipher`, `crypto.saltB64`, `crypto.ivB64`, `innerSchemaVersion`, `plaintextBackupFormat`.

That binds every non-ciphertext envelope field. The fixture verifies that changing an otherwise valid `appVersion` causes authenticated decryption to fail, and separately verifies ciphertext and wrong-passphrase failures.

## Architecture reconciliations

1. The architecture review discussed a historical 210,000-iteration floor. Executable S1 policy follows the approved current requirement instead: 600,000 is both default and minimum, with a 2,000,000 maximum.
2. The review's `backupFormat OR envelopeVersion` recognition suggestion would create an ambiguous encrypted classification. S1 uses only an exact encrypted `backupFormat`, as required.

## Fixtures and validation

`Security S1 Crypto` adds **21 assertions**. It covers native availability and injected unavailability, policy constants and byte lengths, randomized envelope fields, encrypt/decrypt round trip, nonextractable keys, wrong passphrase, ciphertext/header tampering, exact detection, foreign/future/unknown algorithm rejection, KDF bounds, malformed Base64, salt/IV length rejection, envelope ambiguity, invalid/future schema rejection, exact Unicode/whitespace passphrases, passphrase non-disclosure, 1 MiB round trip, and application-boundary isolation.

Fresh checks passed:

- `git diff --check`
- `python -m html.parser index.html`
- Edge JavaScript/runtime parsing under direct `file://`
- DOM duplicate-ID check: none
- DOMParser malformed-markup check: zero `parsererror` elements
- `?selftest=security-s1-crypto`: **21 passed / 0 failed**
- Normal direct `?selftest=all`: every one of the **47** registered routes executed once with reported **0 failed** results; no page errors
- Storage recovery, backup validation/merge/replace, startup/normalization/migration, modal, persistence, and all Designs production fixture groups ran through that normal complete route.

The unique-suite reconciliation is **3003 + 21 = 3024 passed / 0 failed across 47 unique registered groups**. This uses the prior verified 46-group/3003 baseline and the independently observed new 21-assertion group; aliases, aggregate calls, and repeated internal fixture invocations were not counted.

## Direct-file browser results

### Microsoft Edge

- Browser: Microsoft Edge `150.0.4078.83`, automated headless with a disposable browser context.
- URL: `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=security-s1-crypto`
- `crypto`: available; `crypto.subtle`: available.
- Capability probe: available; all five required operations succeeded.
- PBKDF2 600,000 plus AES-GCM probe round trip: **94.9 ms** in the final run.
- AES-GCM round trip, wrong-passphrase failure, ciphertext tamper failure, authenticated-header tamper failure, and 1 MiB payload round trip: all passed through the 21/0 fixture.
- Reload: second direct-file run completed **21/0**.
- Console errors: none. Page errors: none.

### Google Chrome

Chrome was not installed at `C:\Program Files\Google\Chrome\Application\chrome.exe`, `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`, or `%LOCALAPPDATA%\Google\Chrome\Application\chrome.exe`; it was also absent from `Get-Command`. No Chrome run was substituted or inferred from Edge.

## Production SVG golden verification

The full Designs Geometry and production-golden group passed **1093/0** in the normal direct-file complete run. S1 changes only isolated crypto helpers and a fixture route; no production generator, serializer, SVG, layout, or download code changed. The registered production-byte pins exercised and retained include:

| Registered golden family | Length / FNV-1a pins |
| --- | --- |
| Dice tray matrix (10 cases) | default Finger `1726/51a55721`; tab-slot `1054/41697123`; zero-clearance `1670/c69c671f`; thicker `1816/15ffb4c5`; minimum `1556/91820277`; large `1740/054691bf`; shallow `1726/c5ffc4bb`; tall `1726/d52e71b7`; square `1724/6a71e1be`; long-narrow `1757/ed95d237` |
| Divider tray matrix (13 cases) | default Finger `1965/a55dda6e`; tab-slot `1293/481b681c`; zero-clearance `1903/ca24ec91`; thicker `2059/07c6a461`; minimum `1661/be3f5369`; large `2395/ae7e3501`; shallow `1834/30475bbd`; tall `2452/500f2de9`; square `1963/0bac8f9b`; long-narrow `2462/8938ba4c`; shallow-depth `1816/5d12b86c`; deep-narrow `2518/6c1e7413`; wide `1845/12aeb387` |
| Stand, sign, and default tray pins | QR Stand `359/fe737a09`; Hanging Sign `341/656e633d`; Dice `1726/51a55721`; Divider `1965/a55dda6e` |
| Finger and sliding boxes | Finger open-top `2483/a892f91c`; loose-lid `2615/6181bc75`; decorated Finger `7269/d9512d1c`; sliding-lid `2800/4a7ab718` |
| Tabletop accessories | T1 coupon `2992/4f543f95`; legacy pocketed shell `2181/2ef9606b`; T2 closed shell `2337/ed5d6f6e`; T3B storage tray `2860/3e256fad` |
| Concealed-cleat prototypes | J2 `4457/88721533`; J2 red-cut layer `1606/c16a9ef5`; J2.5 full box `9701/2316430b` |
| Drawer Cabinet and coupon pins | one-row `4456/a6dd23dc`; two-row `6802/36a41b07`; three-row `9153/8c286797`; guides `7534/dd3ff0bd`; linked grid `16429/0814ca2e`; separate shell `2779/956ad870`; separate interior `8609/bad8b97f`; custom linked `13606/500396df`; custom separate `14777/69b0ce8b`; 0.40 one-row `4436/7494c326`; 0.40 three-row `9124/b158a794`; wall-to-base coupon `1551/d9ffc278` |

All of the above remained byte-identical under their registered fixture assertions; no S1 production SVG exists.

## Protected boundaries and remaining work

No live backup was created, read, decrypted, imported, exported, stored, or modified. `STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, state normalizers, migration, evidence/promotion, Projects/Library/Inventory/Machines, existing plaintext backup behavior, and all Designs production geometry remain unchanged.

Unverified by design: real encrypted export/import UI, passphrase UI and recovery language, live backup encryption/decryption, vault persistence/migration, photo-heavy performance on slower machines, and independent Chrome behavior. S2's exact boundary is to wire this frozen envelope contract into encrypted export/import UI before plaintext import decoding, with passphrase flow and no change to the S1 cryptographic contract or plaintext backup compatibility.

S1 COMPLETE WITH CAUTION — Chrome was not installed for the required independent direct-file probe.
