# Security S1 — Crypto Capability and Envelope Post-Implementation Verification

**Date:** 2026-07-22  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Role:** Independent adversarial verifier (do not trust the implementation report)  
**Mode:** Read-only. Only this report was created.

---

## Exact final verdict

**VERIFIED WITH CAUTION** — Safe to commit Security S1.

Non-blocking cautions: (1) `?selftest=all` starts the S1 async fixture but does not `await` it; (2) low-level helpers accept empty passphrases (S2 UI must block); (3) Chrome was not installed for a second browser probe; (4) T3A `897 / b5c549ee` is **measured** and stable but **not** a registered length/hash pin in source (implementation report listed it; source only checks T3A determinism).

---

## 1. Actual repository state

| Check | Result |
|-------|--------|
| HEAD | `19d4834` — Graduate tabletop storage tray to Workshop-ready |
| Full hash | `19d48340a293184c748c2767ec8fd0654f7ce376` |
| Branch | `main` |
| Upstream | `0 0` vs `origin/main` |
| Staged | Empty |
| Tracked changes since `19d4834` | **`index.html` only** (+221 lines) |
| Untracked S1/architecture docs | `docs/SECURITY_S1_CRYPTO_CAPABILITY_ENVELOPE_IMPLEMENTATION_2026-07-22.md`, architecture review, other historical untracked files |
| `git diff --check` | Clean |
| `python -m html.parser index.html` | exit 0 |

**Exact diff inspected:** entire `index.html` delta from `19d4834` (S1 constants, helpers, `runSecurityS1CryptoFixtures`, selftest registration). README and CHANGELOG **unchanged**.

---

## 2. Exact diff inspected (S1 surface)

| Addition | Role |
|----------|------|
| Policy constants `ENCRYPTED_BACKUP_*` | Frozen crypto policy |
| Base64 / UTF-8 helpers | Chunked encode (32 KiB), strict decode + non-canonical reject |
| `isEncryptedBackupEnvelope` | Detection |
| `validateEncryptedBackupEnvelopeV1` | Strict validation |
| `encryptedBackupCanonicalHeader` | Deterministic AAD |
| `deriveEncryptedBackupKey` | PBKDF2 nonextractable AES-GCM key |
| `probeEncryptedBackupCryptoCapability` | Real multi-op probe + inject seam |
| `encryptSyntheticEncryptedBackup` / `decryptSyntheticEncryptedBackup` | Synthetic-only envelope ops |
| `runSecurityS1CryptoFixtures` + `?selftest=security-s1-crypto` | Focused tests |

No lines changed inside `exportBtn` / `importFile` handlers, `backupObject`, `validateBackupImport`, `applyBackupImport`, `persist`, `loadState`, or production SVG paths (function-body identity vs `19d4834` confirmed for those symbols and export/import handler text).

---

## 3. S1 isolation and call-boundary result

| Requirement | Independent result |
|-------------|-------------------|
| Export UI/behavior unchanged | **Pass** — handler source identical; no encrypt symbols |
| Import UI/behavior unchanged | **Pass** — same |
| No encrypted download via app | **Pass** — no Export path calls S1 encrypt |
| No real `backupObject` encrypted | **Pass** — encrypt rejects non-synthetic shape; fixtures use disposable object |
| No localStorage encrypt/migrate/clear | **Pass** — storage keys empty before/after adversarial crypto; no vault keys |
| No passphrase UI | **Pass** |
| No vault keys/markers | **Pass** |
| Plaintext backup meaning unchanged | **Pass** — `BACKUP_FORMAT` still `genmitsu-l8-tracker-backup-v1` |
| Merge/Replace unchanged | **Pass** — bodies identical |
| Passphrase/key not persisted | **Pass** — not in envelope JSON; no storage writes |
| Ordinary startup no PBKDF2 | **Pass** — S1 crypto only via helpers/fixtures; not in `render`/`loadState` |
| Helpers only for fixtures (shipping) | **Pass for product paths** — helpers are IIFE-local except `window.runSecurityS1CryptoFixtures` for selftest; no UI caller |

**Note:** Helpers are callable if exported in a test harness; production UI does not invoke them.

---

## 4. Cryptographic-policy result

| Policy item | Required | Measured / source |
|-------------|----------|-------------------|
| `ENCRYPTED_BACKUP_FORMAT` | `genmitsu-l8-tracker-encrypted-backup-v1` | Match |
| Envelope version | 1 | Match |
| KDF id | `PBKDF2-SHA256` | Match |
| PBKDF2 hash | SHA-256 | Match (`hash:'SHA-256'`) |
| Default iterations | 600000 | Match |
| Accepted iterations | 600000–2000000 inclusive | Match (safe integer bounds) |
| Cipher id | `AES-256-GCM` | Match |
| AES key length | 256 | Match |
| Tag | 128 bits | Match `tagLength:128` |
| Salt | 16 random bytes | Match |
| IV | 12 random bytes | Match |
| CryptoKey nonextractable | yes | Fixture + exportKey throw |
| No trim/normalize passphrase | yes | Exact Unicode/whitespace tests |
| Fresh salt/IV each encrypt | yes | Measured |
| No weak fallback | yes | Unavailable returns structured fail |

**Bypass analysis:** `options.kdfIterations` may override default **only** if still in [600000, 2000000]; outside range fails validation before encrypt completes (`overIter`/`underIter` false). No path below 600000. Injected `cryptoApi` is a **test seam**, not a production weak fallback.

---

## 5. Capability-gate result

| Check | Result |
|-------|--------|
| Requires getRandomValues + importKey + deriveKey + encrypt + decrypt | **Pass** (all must be true) |
| `window.crypto` alone insufficient | **Pass** (partial inject fails) |
| Empty inject → unavailable | **Pass** `web-crypto-unavailable` |
| Real Edge file:// probe | **available: true**, ~**87 ms** for 600k PBKDF2 + AES round trip |
| Fail closed, no state change | **Pass** |

---

## 6. Detection and strict-validation result

### Detection (`backupFormat === ENCRYPTED_BACKUP_FORMAT` only)

| Input | Detected |
|-------|----------|
| Valid envelope | true |
| `envelopeVersion` alone | false |
| `crypto` alone | false |
| `ciphertextB64` alone | false |
| Prefix format string | false |
| Wrong capitalization | false |
| Plain `BACKUP_FORMAT` | false |
| null / array / string | false |
| **Object.create inherited `backupFormat`** | **true** (prototype inheritance) |

**Inherited-property quirk:** detection uses property lookup, so an object with only an inherited `backupFormat` is “detected.” Strict validation then fails without required own keys. **Low** risk for S1; S2 should prefer `Object.hasOwn` / `hasOwnProperty` for detection if desired.

Detection ≠ validation: foreign `app` still detected true but validation `foreign-app`.

### Validation rejections (measured codes)

foreign-app, unsupported-envelope-version, invalid-kdf/cipher, invalid-kdf-iterations (low/high/float/string), invalid-inner-schema (0 / future), ambiguous-envelope/crypto, invalid-base64 (junk + whitespace), invalid-salt/iv length, invalid-ciphertext (too short), invalid-created-at, invalid-plaintext-format, invalid-app-version. Input object not mutated.

---

## 7. Canonical AAD result

- Serializer maps fixed `ENCRYPTED_BACKUP_AAD_FIELDS` order → `JSON.stringify([[path,value],…])` — **not** object key insertion order.
- Independent rebuild matched production header (`aadMatch: true`).
- Field-level tamper (appVersion, createdAt, kdfIterations, salt, iv, …) → decrypt fail (auth or validation). Count of failed tampers: **12/12** attempted.
- Ciphertext not included in AAD (by construction).

---

## 8. Passphrase-handling result

Disposable only. Verified: leading/trailing spaces, empty string, tab, newline, emoji, NFC/NFD café variants encrypt; cross-variant decrypt fails; NFC vs NFD fails; passphrase not in envelope JSON; no trim.

**Empty passphrase:** encrypt **and** decrypt succeed. Acceptable as low-level infrastructure; **S2 must refuse empty UI passphrases**.

---

## 9. Base64 and payload-safety result

- Full 0–255 byte round-trip: **pass**
- 1 MiB payload encrypt/decrypt: **pass** (~355 ms including KDF)
- Chunked encode (no giant spread): source uses 32 KiB chunks
- Malformed Base64 / whitespace: validation fails

---

## 10. Fixture-quality result

`Security S1 Crypto` exercises: real capability + injected fail, encrypt policy, freshness, round-trip, nonextractable key, wrong pass, CT/header tamper, detection, foreign/future/KDF/cipher/iter/schema/B64/length/ambiguous fields, Unicode passphrase exactness, non-disclosure, 1 MiB, isolation of state/handlers.

| Quality concern | Severity |
|-----------------|----------|
| Combined asserts (e.g. foreign+future in one `add`) | Low — failures still fail the group |
| `selftest=all` does not await `securityS1CryptoFixturePromise` | **Medium** harness (see caution) |
| Empty passphrase not rejected by fixture | Low — S2 |

Fresh focused run: **21 passed / 0 failed**.

---

## 11. Focused fixture total

**Security S1 Crypto: 21 / 0** (independent re-run).

---

## 12. Independently calculated full-suite total

Every unique registered group executed **once** (S1 awaited explicitly):

| Metric | Measured |
|--------|----------|
| Unique groups | **47** |
| Passed | **3024** |
| Failed | **0** |

Reconciliation: prior unique baseline **3003 / 46** + S1 **21** = **3024 / 47**. Arithmetic confirmed by summing group results, not by trust alone.

`regS1 === 1`. `awaitsS1 === false` for the normal `?selftest=all` registration line (async fire-and-forget).

---

## 13. Complete production-golden result

Designs geometry group: **1093 / 0** within suite.

Independently measured pins:

| Golden | Bytes / hash | Status |
|--------|--------------|--------|
| T1 coupon (2.88 case) | 2992 / `4f543f95` | Pass |
| T2 shell (clearance 0 fixture draft) | 2337 / `ed5d6f6e` | Pass |
| **T3A storage-fit (default/measured)** | **897 / `b5c549ee`** | **Pass (measured)** |
| T3B tray | 2860 / `3e256fad` | Pass |
| Dice | 1726 / `51a55721` | Pass |
| Dice alt | 1054 / `41697123` | Pass |
| Divider | 1965 / `a55dda6e` | Pass |
| Wall-to-base | 1551 / `d9ffc278` | Pass |

Registered suite pins (source) still include legacy pocketed shell `2181/2ef9606b`, full dice/divider matrices, finger/sliding, cleat, drawer cabinet families — all exercised under design **1093/0**.

### T3A omission determination

- Source search for string `b5c549ee`: **0 hits**.
- T3A fixture asserts deterministic/closed/red SVG but **does not** pin length/hash.
- Implementation report listed T3A in a golden table without a matching registered assert.
- **Conclusion:** report omission of a **registered** pin, not S1 byte drift. Independent generation still yields **897 / b5c549ee**.

---

## 14. Edge and Chrome results separately

### Microsoft Edge (measured)

| Item | Value |
|------|--------|
| Product version | **150.0.4078.83** |
| Executable | `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` |
| Mode | Headless + remote debugging, disposable profile |
| URL | `file:///…/index.html?s1verify=1` (instrumented temp copy of app) |
| `crypto` / `subtle` | Available |
| Capability probe | available |
| PBKDF2 600k + AES probe | ~**87 ms** |
| Round-trip / wrong pass / tamper / AAD / 1 MiB | Pass |
| localStorage before/after crypto | Empty; no vault keys |
| Console / page errors (probe) | None |

### Google Chrome

Searched:

- `C:\Program Files\Google\Chrome\Application\chrome.exe` — missing  
- `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe` — missing  
- `%LOCALAPPDATA%\Google\Chrome\Application\chrome.exe` — missing  
- `Get-Command chrome` — not found  

**Chrome not installed.** No Chrome run; no inference from Edge.

### Preexisting Edge SVG `height: "auto"` diagnostics

Not observed on the S1 crypto path (no Designs preview). Treated as **preexisting, route-dependent** (Finished View / SVG preview), **not resolved** by S1 and **not introduced** by S1.

---

## 15. Performance result

| Operation | Measured |
|-----------|----------|
| Capability probe (600k + AES) | ~87 ms (Edge headless) |
| 1 MiB encrypt+decrypt | ~355 ms |
| Startup path | No S1 KDF |
| Iterations > 2e6 | Rejected before derive |

---

## 16. Console and page errors

Probe: **none** new for S1. SVG `height: auto` not attributed to S1.

---

## 17. Protected-boundary confirmation

| Boundary | Status |
|----------|--------|
| SCHEMA_VERSION 4, APP_ID, APP_NAME, APP_VERSION 0.9.0, BUILD_DATE, STORAGE_KEY, BACKUP_FORMAT | Unchanged |
| backupObject / Export / Import / decode / validateBackupImport / applyBackupImport / persist / loadState | Function/handler identity vs baseline |
| Merge/Replace | Unchanged |
| Schemas, evidence matchers, geometry, SVG serializers | Unchanged (selected builders identical) |
| Offline file:// | Works |

---

## 18. Findings by severity

### Critical

None.

### High

None for S1 commit.

### Medium

1. **`?selftest=all` does not await `securityS1CryptoFixturePromise`.**  
   - **Location:** selftest registration ~line 14720.  
   - **Behavior:** async S1 starts; design suite continues immediately.  
   - **Impact:** Console “all done” can precede S1 completion; race under slow KDF. Independent enumeration still measured 3024/0 when awaited.  
   - **Correction:** `await` promise before considering `all` complete (or sync barrier).  
   - **Blocks S1 commit?** No. **S2?** Should fix before relying on `all` as sole gate.

### Low

2. **Empty passphrase accepted by encrypt/decrypt helpers.**  
   - **Impact:** Not a UI path in S1; S2 must block.  
   - **Blocks S1?** No.

3. **Detection true for inherited `backupFormat` (prototype chain).**  
   - **Impact:** Must still pass exact-key validation; low for real JSON.parse objects.  
   - **Blocks S1?** No.

4. **T3A production hash not registered as a golden assert** (only measured).  
   - **Impact:** Report vs source mismatch; not S1 regression.  
   - **Blocks S1?** No.

### Informational

5. Chrome unavailable on this machine.  
6. Combined multi-condition fixture asserts (style).  
7. Implementation arithmetic 3003+21=3024 matches independent sum.

---

## 19. Unverified behavior

- Chrome/Safari file:// SubtleCrypto  
- Photo-heavy multi-MB encrypted backup wall time on low-end PCs  
- Headed Edge SVG preview residual `height: auto` count  
- Formal side-channel analysis  

---

## 20. Whether S1 is safe to commit

**Yes.** Cryptography policy holds under adversarial checks; product Export/Import/storage isolated; suite **3024 / 0 / 47** when S1 is awaited; production goldens intact.

---

## 21. Exact safe S2 boundary (do not implement here)

**S2 may:** wire encrypted export/import UI; detect `ENCRYPTED_BACKUP_FORMAT` before plaintext decode; passphrase prompts; default recommend encrypted export; plain JSON as labeled recovery with warning; after decrypt, call existing `validateBackupImport` + Merge/Replace unchanged.

**S2 must not:** change S1 crypto parameters without version bump; encrypt live vault; migrate `STORAGE_KEY`; store passphrases; treat decrypt failure as empty workspace; delete plaintext without recovery; claim vault complete.

**S2 should:** await crypto tests in `selftest=all`; reject empty passphrases in UI; optionally harden detection with own-property checks.

---

## Hygiene

- No product files edited, staged, committed, or pushed.  
- No S2 implementation.  
- No Joe real profile/localStorage.  
- Disposable Edge profile + synthetic data only.  
- Unrelated untracked files preserved.

---

*End of S1 post-implementation verification.*
