# Security S2 — Encrypted Backup Export and Import Implementation

Date: 2026-07-22  
Repository: `C:\Genmitsu L8 Tracker`  
Committed baseline: `8c06800 Add Security S1 crypto foundation`

## Scope and repository state

S2 connects the committed S1 native-Web-Crypto envelope contract to downloaded backup files only. It does not implement a live encrypted vault, encrypt `localStorage`, change `STORAGE_KEY`, alter schemas, migrate data, retain passphrases, or modify Designs production generation.

Before editing, `main` was synchronized with `origin/main` at `8c06800`; nothing was staged and there was no tracked diff. Pre-existing untracked reports, `.claude/`, LightBurn Projects, `debug.log`, and the utility script were preserved. This implementation changes only `index.html` and this report. Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## S2 behavior

### Export

The header Export action now opens an accessible choice dialog:

- **Password-encrypted backup** is the primary, recommended action.
- **Plain JSON backup (readable)** remains available as a secondary compatibility/recovery action, but requires a custom warning confirmation that it contains readable workshop, project, pricing, inventory, and evidence records. The existing photo-size information is retained in that warning.

Opening Export itself performs no PBKDF2 work. Choosing the encrypted path first runs S1's real native capability probe. If Web Crypto is unavailable or cannot complete the required operations, the UI refuses encrypted export with a clear message and no weak fallback.

After the probe succeeds, the encrypted form uses passphrase and confirmation fields. Empty means exactly `''`; values are neither trimmed nor normalized. The passphrase is passed directly to `createEncryptedBackupExport()`, which calls `backupObject()` and S1's `encryptSyntheticEncryptedBackup()`. The downloaded JSON is the frozen S1 v1 encrypted envelope, named:

`l8-tracker-backup-encrypted-YYYY-MM-DD.json`

The readable path retains its existing exact pretty serializer, MIME type, and filename:

`JSON.stringify(backupObject(), null, 2)` → `application/json` → `l8-tracker-backup-YYYY-MM-DD.json`

Neither export path calls `persist()` or changes `localStorage`. The encrypted form fields and transient operation references are cleared on completion, failure, cancellation, Escape, or dialog close. Submit controls and operation tokens prevent duplicate work; closing an in-flight encrypted action prevents a later download.

### Import

The file reader still decodes JSON, but now routes the outer object first:

1. `isEncryptedBackupEnvelope()` requires S1's exact own encrypted `backupFormat` marker.
2. An exact-marker object is structurally checked by `validateEncryptedBackupEnvelopeV1()` **before** any PBKDF2/decryption work.
3. A malformed encrypted marker is rejected and cannot fall through to legacy plaintext import.
4. A structurally valid encrypted envelope receives the same native capability check and an accessible passphrase dialog.
5. `decryptSyntheticEncryptedBackup()` authenticates and decrypts it; the resulting plaintext then passes the existing `validateBackupImport()` before reaching the unchanged `openImportModeDialog()` Merge/Replace flow.

Wrong passwords, damaged ciphertext, unsupported envelopes, foreign/future metadata, and invalid decrypted backups leave state and browser storage unchanged. Plain historical JSON backups retain their legacy `validateBackupImport()` and Merge/Replace behavior.

## Root cause and correction rationale

There was no crypto defect in S1. The product had no UI route from the committed S1 helpers to real backup files, so all exports/imports were plain JSON. S2 adds only that missing application boundary. The strict S1 envelope is deliberately detected before generic plaintext validation because legacy plaintext compatibility accepts some marker-less objects; an exact encrypted marker must never use that permissive path.

## Fixture coverage

Added the uniquely registered asynchronous route:

`?selftest=security-s2-encrypted-backup`

`Security S2 Encrypted Backup` contains **17 passed / 0 failed** assertions. It uses the real S1 primitives and a fixture-only download capture, never an uncontrolled browser download. It verifies:

- native capability availability and explicit unavailable-platform refusal with no fallback;
- the encrypted-primary/readable-secondary choice and readable-data confirmation;
- exact legacy plain serializer, filename, and MIME type;
- password/confirmation UI, exact-empty rejection, and no passphrase-persistence controls;
- S1 envelope validation and encrypted filename;
- no passphrase or serialized plaintext in the encrypted file;
- decrypt → existing plaintext validation → unchanged import-mode handoff;
- exact whitespace passphrase behavior and wrong-password rejection;
- malformed own encrypted marker rejection before plaintext fallback;
- encrypted passphrase UI before Merge/Replace, then unchanged Merge/Replace handoff without mutation;
- duplicate and cancelled encrypted-export suppression; and
- state, `localStorage`, and `backupObject()` serialization isolation before explicit import application.

The dispatcher awaits S1 and S2 independently. A normal `?selftest=all` run invoked both exactly once (`securityS1CryptoFixtureRunCount: 1`, `securityS2EncryptedBackupFixtureRunCount: 1`) before the completion promise resolved.

## Validation results

| Check | Result |
| --- | --- |
| `git diff --check` | Pass |
| `python -m html.parser index.html` | Pass |
| Inline runtime/JavaScript check | Direct `file://` Edge loaded and completed the S2 and normal all routes without a dispatcher rejection or runtime exception |
| S1 crypto | 21 passed / 0 failed |
| S2 encrypted backup | 17 passed / 0 failed |
| Existing storage recovery/import fixture | 15 passed / 0 failed |
| Full Designs Geometry / registered production goldens | 1093 passed / 0 failed |
| Normal complete dispatcher | 0 failed; S1 and S2 each ran once |

The independently reconciled unique registered total is **3041 passed / 0 failed across 48 groups**: the prior committed S1 baseline was 3024/0 across 47 groups, and S2 adds the verified 17/0 group. The count of `selftest === … || selftest === 'all'` registered branches is 48. This deliberately does not add non-unique nested diagnostics to the aggregate.

### Direct-file browser method

Microsoft Edge `150.0.4078.83` was automated headlessly with fresh disposable `--user-data-dir` profiles and direct URLs:

- `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=security-s2-encrypted-backup`
- `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=all`

The application-owned completion promises returned `{ completed: true, failed: 0 }`. The focused route returned S2 `17/0`; the normal route returned S1 `21/0`, S2 `17/0`, and both run counters equal to one. The test exercised the rendered export choice, readable confirmation, password forms, decryption handoff, and cancellation path through the real direct-file application. Chrome remains unavailable on this machine and was not installed or inferred.

## Protected production outputs

S2 changes no generator, geometry, serializer, SVG layout, production colors/layers, filenames, or download code for Designs. The full Designs group passed 1093/0. The known production SVG pins retained by that group are:

| Family | Length / FNV-1a |
| --- | --- |
| Dice tray matrix | default Finger `1726/51a55721`; tab-slot `1054/41697123`; zero-clearance `1670/c69c671f`; thicker `1816/15ffb4c5`; minimum `1556/91820277`; large `1740/054691bf`; shallow `1726/c5ffc4bb`; tall `1726/d52e71b7`; square `1724/6a71e1be`; long-narrow `1757/ed95d237` |
| Divider tray matrix | default Finger `1965/a55dda6e`; tab-slot `1293/481b681c`; zero-clearance `1903/ca24ec91`; thicker `2059/07c6a461`; minimum `1661/be3f5369`; large `2395/ae7e3501`; shallow `1834/30475bbd`; tall `2452/500f2de9`; square `1963/0bac8f9b`; long-narrow `2462/8938ba4c`; shallow-depth `1816/5d12b86c`; deep-narrow `2518/6c1e7413`; wide `1845/12aeb387` |
| Stand, sign, and default trays | QR Stand `359/fe737a09`; Hanging Sign `341/656e633d`; Dice `1726/51a55721`; Divider `1965/a55dda6e` |
| Finger and sliding boxes | Finger open-top `2483/a892f91c`; loose-lid `2615/6181bc75`; decorated Finger `7269/d9512d`; sliding-lid `2800/4a7ab718` |
| Tabletop accessories | T1 coupon `2992/4f543f95`; legacy pocketed shell `2181/2ef9606b`; T2 closed shell `2337/ed5d6f6e`; T3A current measured output `897/b5c549ee` (non-registered measured pin); T3B storage tray `2860/3e256fad` |
| Concealed-cleat prototypes | J2 `4457/88721533`; J2 red-cut layer `1606/c16a9ef5`; J2.5 full box `9701/2316430b` |
| Drawer Cabinet and wall-to-base coupon | one-row `4456/a6dd23dc`; two-row `6802/36a41b07`; three-row `9153/8c286797`; guides `7534/dd3ff0bd`; linked grid `16429/0814ca2e`; separate shell `2779/956ad870`; separate interior `8609/bad8b97f`; custom linked `13606/500396df`; custom separate `14777/69b0ce8b`; 0.40 one-row `4436/7494c326`; 0.40 three-row `9124/b158a794`; wall-to-base coupon `1551/d9ffc278` |

## Storage/schema and remaining boundaries

`STORAGE_KEY`, `SCHEMA_VERSION`, `BACKUP_FORMAT`, backup schema, import/export format for readable backups, merge/replace semantics, normalizers, machine/evidence identity, and `localStorage` remain unchanged. Export does not turn the live workspace into a vault; the only ciphertext is the explicit downloaded encrypted backup file. No passphrase is written to browser storage, backup JSON, app state, logs, or user records.

Remaining physical/product validation is intentionally outside S2: Chrome/other-browser direct-file behavior, very large photo-heavy backup timing, real-world passphrase/recovery usability, and all live encrypted-vault/lock/migration work belong to later phases. A user may now create and import password-encrypted backup files in Edge; this does not claim protection for the existing plaintext browser profile.

S2 COMPLETE WITH CAUTION — Chrome direct-file behavior and photo-heavy backup performance were not independently measured; no live local-storage vault is implemented by design.
