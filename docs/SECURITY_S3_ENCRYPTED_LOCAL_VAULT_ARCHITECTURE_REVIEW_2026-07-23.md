# Security S3 — Encrypted Local Vault — Architecture Review

**Date:** 2026-07-23
**Reviewer:** Claude Opus 4.8 (independent, read-only, architecture-only — nothing implemented)
**Scope:** the highest-risk storage phase to date — optional encryption of the *live* browser-stored workspace. This settles the storage, state-machine, migration, persistence, recovery, multi-tab, and test contracts before any implementation.

This is a static source-and-document review. No product file was modified; this report is the only write. No browser profile, real `localStorage`, backup, download, or passphrase was accessed. I did **not** re-execute the fixture suite (architecture-only); the baseline total below is carried from the committed S2 post-implementation verification and cross-checked structurally against source (registered-group count). No runtime, browser, hash, or security property is claimed as "verified" here beyond what direct source reading establishes.

---

## 1. Actual repository state

| Check | Result |
|---|---|
| HEAD | `386e1e01d29029d53a7b2be1cdd976eb5d0bc2fb` — **"Add encrypted backup export and import"** (S2). Confirmed. |
| Recent chain | `386e1e0` (S2) ← `8c06800` (S1) ← `19d4834` (T4 Workshop-ready) ← `1d2d860` (T3B-E1) … |
| Branch | `main` |
| `origin/main...main` | `0 0` (synchronized) |
| Staged | Nothing (`git diff --cached` empty) |
| Tracked working tree | Clean (`git diff` empty) |
| Untracked | Long-standing `docs/*` history, the S1/S2 security reports, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py` — all preserved, none modified |
| Suitable for read-only review? | **Yes** — clean, synchronized, S1+S2 committed, no in-flight edits |

**Identity constants (read from source):** `APP_ID='genmitsu-l8-tracker'`, `APP_NAME='Genmitsu L8 Tracker'`, `APP_VERSION='0.9.0'`, `BUILD_DATE='2026-07-19'`, `SCHEMA_VERSION=4`, `STORAGE_KEY='genmitsu-l8-tracker-v1'`, `BACKUP_FORMAT='genmitsu-l8-tracker-backup-v1'`, `ENCRYPTED_BACKUP_FORMAT='genmitsu-l8-tracker-encrypted-backup-v1'`.

**Baseline suite:** the committed S2 post-implementation verification established **3068 passed / 0 failed across 48 unique registered groups** (its outer-return methodology corrected the S2 implementation report's 3041 undercount). I structurally confirmed **48** registered `selftest === '<key>' || selftest === 'all'` branches in current source, and separately confirmed **0** `sessionStorage` references and **0** pre-existing `vault`/`Vault` tokens — a clean slate for S3.

---

## 2. Current plaintext storage architecture (as read from source)

- **Single live key.** `localStorage['genmitsu-l8-tracker-v1']` holds one plaintext JSON document written by `persist()` (index.html:1008): `schemaVersion`, `entries`, `profiles` (nested tests/production-settings/evidence/photos), `grids`, `projects` (embedded photos), `inventory`, the four tabletop raw-result collections, unit/UI prefs, `machines`/`activeMachineId`/`machineIdentity`, and `pricing`/`pricingPrefs`. `designDraft`, preview modes, most modal state, and first-run dismissal are session-only and intentionally excluded.
- **Startup is fully synchronous.** `const state = loadState()` (index.html:617) runs at IIFE-parse time; `loadState()` (967) reads `STORAGE_KEY` synchronously via `decodeStoredState()` (667), normalizes, and returns. The tail (≈14607–14627) wires event handlers, `beforeunload → flushPricingPersist`, then calls `render()` synchronously. **There is no async gate anywhere in startup.** This is the single most important structural fact for S3.
- **`persist()` is synchronous and returns a boolean that is load-bearing.** It is called from ~45 sites, and 9 evidence-save/promotion paths (`tabletopCouponRecordSave`, `tabletopShellResultSave`, `finalizeTabletopShellPromotion`, `tabletopStorageResultSave`, `finalizeTabletopStoragePromotion`, `tabletopTrayResultSave`, `finalizeTabletopTrayPromotion`, plus three sites at 7672/7721/7741) perform **synchronous in-memory rollback when `persist()` returns `false`**. Any S3 change to `persist()` must preserve a synchronous boolean return whose `false` still means "the change was not accepted" — encryption's asynchronous success/failure cannot be reported through that boolean.
- **`persist()` already fails soft.** On `localStorage.setItem` throw it calls `showStorageWarning()` (an `alert`, once) and returns `false`; it also returns `false` (without writing) when `storageRecoveryIssue` is set unless `allowRecoveryOverwrite`. This is the correct async-safe failure channel to reuse for encrypted-write failures.
- **Corruption recovery is well designed and is the template for vault-auth failure.** `decodeStoredState` returns `{status:'corrupt'}` for non-JSON / non-object; `loadState` then sets `storageRecoveryIssue`, returns `freshState()` (empty UI, **records never loaded**), and **blocks `persist()`** until an explicit recovery overwrite or import. `renderStorageRecoveryPanel` (2183) offers *download damaged raw text*, *import backup*, or *Start fresh* (confirm). **Never** clears storage on failure; **never** treats unreadable data as empty success.
- **Backup/import boundary (S2).** `backupObject()` (1657) produces the plaintext backup document. `validateBackupImport()` (681) refuses future `schemaVersion`, foreign `app`, or wrong `backupFormat` (but still accepts legacy marker-less objects — plaintext only). The import file reader (14612) routes text through `handleBackupImportText → routeBackupImportData`: an exact-own encrypted `backupFormat` marker branches to the strict encrypted path *before* any plaintext handling; everything else uses legacy `validateBackupImport → openImportModeDialog` (Merge/Replace, second confirm on Replace). Encrypted files never touch live storage — the only ciphertext today is the downloaded backup file.

---

## 3. Frozen S1/S2 contracts (must be reused, not redesigned)

Read directly from source (index.html:327–499). S3 **must not** alter any of these:

| Item | Frozen value |
|---|---|
| Crypto provider | Native Web Crypto only; **no** fallback, no reversible-encoding-as-encryption, no third-party/network crypto |
| KDF | `PBKDF2-SHA256` (`SHA-256`), default **600000**, accepted **600000–2000000** (`Number.isSafeInteger`, bounds-checked) |
| Cipher | `AES-256-GCM`, 256-bit non-extractable key (`extractable:false`), **128-bit** tag |
| Salt / IV | **16** / **12** random bytes (CSPRNG) |
| Base64 | Strict canonical round-trip validator (`encryptedBackupBase64ToBytes` re-encodes and compares; rejects non-canonical, wrong-length-mod-4, non-alphabet) |
| Key derivation | `deriveEncryptedBackupKey` — validates passphrase is a string (no trim/normalize), salt length, iteration bounds; `importKey(...,false,['deriveKey'])` → `deriveKey(...,false,['encrypt','decrypt'])` |
| AAD | `encryptedBackupCanonicalHeader` — deterministic JSON of a **fixed ordered field list**, bound as GCM `additionalData`; tamper on any bound field → authentication failure |
| Envelope validation | `validateEncryptedBackupEnvelopeV1` — exact-key sets (outer + `crypto`), foreign-app refusal, envelope-version refusal, KDF/cipher/iteration/schema/format/base64/salt-length/iv-length/min-ciphertext checks, **all before PBKDF2** |
| Detection | `isEncryptedBackupEnvelope` — own-property `backupFormat === ENCRYPTED_BACKUP_FORMAT` only (never `envelopeVersion` alone) |
| Backup formats | Plaintext `BACKUP_FORMAT` and encrypted `ENCRYPTED_BACKUP_FORMAT` behaviors both unchanged |

**Envelope-reuse decision — use a SEPARATE strict vault envelope format that SHARES S1's lower-level helpers.**

- **Share** (do not fork): `encryptedBackupCryptoSupport`, `encryptedBackupUtf8Encode/Decode` (fatal UTF-8), `encryptedBackupBytesToBase64` / `encryptedBackupBase64ToBytes`, `deriveEncryptedBackupKey`, the AAD-canonicalization *pattern*, the AES-GCM encrypt/decrypt call shape, and the same PBKDF2/AES-GCM parameter policy. These are format-agnostic and already hardened.
- **Do NOT reuse the backup envelope structure verbatim.** The S1 backup envelope carries `backupFormat: ENCRYPTED_BACKUP_FORMAT`, `plaintextBackupFormat: BACKUP_FORMAT`, and `innerSchemaVersion` bound to the *backup* document. A live-vault blob is a different thing (at-rest live storage, not a portable file). Reusing the identical format would let a vault blob be mistaken for an importable backup (and vice-versa), couple two independently-versioned artifacts, and force any future change to touch both. **Reason for the choice:** distinct `VAULT_FORMAT` gives unambiguous discrimination at every read site, lets the vault envelope carry vault-only AAD fields (write-generation, vault schema) without polluting the backup contract, and keeps S1/S2's frozen formats literally untouched.
- **Consequence:** a downloaded backup and the at-rest vault are never confusable; import logic and vault-load logic branch on different exact format constants; S1/S2 fixtures remain valid unchanged.

---

## 4. Threat model and accurate security claims

**What the local vault protects:** confidentiality and integrity of the live workspace **at rest in `localStorage`** when the app is **not unlocked** (fresh start, after reload, after manual lock). An attacker who copies the browser profile, reads the profile off disk, or opens DevTools storage on a locked session sees only an authenticated ciphertext envelope, not records. Wrong passphrase or tampered ciphertext produces an authenticated **failure**, never a silent empty workspace.

**What it explicitly does NOT protect (must be stated in UI, never contradicted):**

| Vector | Reality |
|---|---|
| While **unlocked** | Records are plaintext in JS memory and the DOM; anything with page/process access sees them |
| Browser **memory / DevTools / heap / swap / crash dump / hibernation** | JS cannot guarantee secure erasure; unlocked plaintext and the derived key may persist in memory/swap beyond `lock` |
| Malicious **browser extension** with host access | Can read page memory/DOM while unlocked; app cannot defend |
| **Malware / keylogger / compromised OS / shared OS account** | Captures the passphrase as typed and plaintext after unlock |
| Tampered **`index.html`** (XSS/injected script in the single file) | If the adversary controls the app's own code, the app cannot trust itself — it could exfiltrate the passphrase or key. The vault assumes an intact application file. |
| **Physical access** to a powered-on unlocked machine | Outside app boundary; OS lock is primary |
| **Backups / downloaded files** | Governed by S2 (plain = readable; encrypted = protected). The vault does not change backup confidentiality. |
| **Forgotten passphrase** | **No recovery, no backdoor.** Only a valid prior backup restores data. |
| **Deleted/corrupted browser profile** | Vault ciphertext is lost with the profile; only a backup restores data |
| Full-disk encryption (BitLocker) | Complementary, not replaced |

**Approved user-facing claim language (and forbidden phrasing):**
- ✅ "Encrypts your saved workspace in this browser when the vault is locked."
- ✅ "Only protects data at rest while locked. Anyone using this computer while the vault is unlocked can see your records."
- ✅ "There is no password reset and no backdoor. If you forget the passphrase, only a backup can restore your data."
- ✅ "This does not replace your Windows sign-in, disk encryption, or physical security."
- ❌ "All your data is always encrypted." ❌ "Complete/absolute security." ❌ "Secure app / certified." ❌ any claim implying protection while unlocked, against malware, or against a tampered app file.

---

## 5. Recommended vault and metadata formats + storage-key strategy

### 5.1 Storage keys

| Key | Role | Present when |
|---|---|---|
| `genmitsu-l8-tracker-v1` (`STORAGE_KEY`, unchanged string) | Plaintext workspace | Plaintext mode only. **Removed** (not repurposed, not rewritten) once vault migration is verified. |
| `genmitsu-l8-tracker-vault-v1` (new) | Encrypted live-data envelope (authoritative ciphertext) | Vault mode |
| `genmitsu-l8-tracker-vault-meta-v1` (new) | Non-secret bootstrap metadata (hint only) | Vault-enabled or mid-transition |
| `genmitsu-l8-tracker-vault-v1-pending` (new, transient) | Staging slot for crash-safe migration/rotation | Only during a transition; removed on completion |

**Names accepted as proposed** (`…-vault-v1`, `…-vault-meta-v1`), with the added transient staging key and a versioned `-pending` slot. All new; none collides with existing keys.

### 5.2 Vault envelope format (the ciphertext at `…-vault-v1`)

```
{
  "app": "genmitsu-l8-tracker",
  "appVersion": "<originating APP_VERSION>",
  "vaultFormat": "genmitsu-l8-tracker-vault-v1",      // discriminator (exact own-property)
  "vaultEnvelopeVersion": 1,
  "createdAt": "<ISO-8601 Z>",
  "writeGeneration": <monotonic int, +1 each successful write>,
  "crypto": { "kdf":"PBKDF2-SHA256", "kdfIterations":<600000..2000000>,
              "cipher":"AES-256-GCM", "saltB64":"<16-byte>", "ivB64":"<12-byte, FRESH per write>" },
  "ciphertextB64": "<AES-GCM(inner workspace JSON) incl. 128-bit tag>",
  "innerSchemaVersion": <= SCHEMA_VERSION      // schema of the inner workspace document
}
```

- **Inner plaintext (after decrypt):** exactly the same JSON document `persist()` writes today (the object built inline in `persist()`), so `loadState`'s existing normalizers/migration run unchanged on it. It is **not** a `backupObject()` — it is the live-storage document (which lacks `app`/`backupFormat` wrappers). Validation after decrypt: `JSON.parse` (guarded) → object shape check → existing collection normalizers → `schemaVersion <= SCHEMA_VERSION`.
- **AAD (authenticated, fixed order):** `app`, `vaultFormat`, `vaultEnvelopeVersion`, `createdAt`, `writeGeneration`, `crypto.kdf`, `crypto.kdfIterations`, `crypto.cipher`, `crypto.saltB64`, `crypto.ivB64`, `innerSchemaVersion`. Binding `writeGeneration` and the full crypto header means a rolled-back or param-swapped envelope fails authentication.
- **Strict policy (mirror S1):** exact required-field set; unexpected field → refuse; foreign `app` → refuse; `vaultFormat` mismatch → refuse; `vaultEnvelopeVersion > 1` → refuse (future-version refusal); invalid/non-canonical Base64 → refuse; salt≠16, iv≠12, iterations out of 600000–2000000, `innerSchemaVersion<1` or `>SCHEMA_VERSION`, ciphertext ≤ 16 bytes → refuse. **All structural validation before PBKDF2.** Corruption/tampering → authenticated failure → recovery UI (never empty-success, never storage clear).

### 5.3 Bootstrap metadata (the plaintext at `…-vault-meta-v1`)

Metadata is a **hint for the locked bootstrap only** and is **never trusted for a security-sensitive decision** — the envelope's own AAD re-declares every crypto parameter and is authoritative; any disagreement between meta and envelope is resolved by trusting the **envelope** (and surfacing an inconsistency warning).

| Field | Required before unlock? | Leaks? | Authenticated? | On disagreement with envelope |
|---|---|---|---|---|
| `mode` (`"vault"` / `"migrating"` / `"disabling"`) | **Yes** — the synchronous startup branch depends on it | Only "a vault exists" (already obvious from key presence) | No (plaintext hint) | Envelope presence + validity wins; `mode` only selects which recovery/transition screen to show |
| `vaultFormat`, `vaultEnvelopeVersion` | Yes — to show "unsupported future version" before asking for a passphrase | No | No | Envelope AAD authoritative; mismatch → refuse before PBKDF2 |
| `crypto` params (`kdf`, `kdfIterations`, `saltB64`) | **Convenience only** — the envelope also carries them | Salt is public by design; iteration count is not secret | No — **the envelope AAD is the authoritative copy** | Ignore meta; use envelope's authenticated params. Never derive a key from meta-only params. |
| `createdAt`, `lastWriteAt` | No (display) | Minor timing metadata | No | Cosmetic only |
| `lastWriteGeneration` | Optional | No | No | Used only to *detect* a possible stale/rolled-back envelope (envelope generation < meta) → warn, do not auto-act |
| `pendingTransition` (`{kind, phase}`) | **Yes during a transition** — drives crash recovery | Only "a migration was in progress" | No | Cross-checked against which keys actually exist; see §7 recovery table |
| Failed-attempt count | **No — do NOT store.** | Would let an attacker with storage access reset it; offline rate-limiting via storage is theater | — | Keep attempt counting in memory only (per session) |
| Any recovery marker / verifier / key check | **No — forbidden.** | A stored "password verifier" is an offline-crackable oracle equivalent to the ciphertext itself and adds attack surface | — | Do not add. Authentication *is* the verifier (GCM tag). |

**Hard rule:** never make a security decision (which key to derive, whether a passphrase is "correct", whether to overwrite) based on unauthenticated metadata. Metadata selects *which screen* to render; the encrypted envelope's authenticated header decides *everything cryptographic*.

### 5.4 Authoritative-copy rule (single source of truth at all times)

At every startup, discover the authoritative live document by this deterministic precedence, cross-checking meta against actual key presence:

1. **`…-vault-v1` present AND validates structurally** → vault is authoritative (locked). Ignore any stray `STORAGE_KEY` (see interrupted-migration recovery).
2. **`…-vault-v1` absent, `STORAGE_KEY` present, meta absent/`mode!=vault`** → plaintext is authoritative (today's behavior, unchanged).
3. **`…-pending` present** (crash mid-transition) → run the §7 recovery decision table; never auto-delete either copy without user confirmation.
4. **Meta says `vault` but `…-vault-v1` is missing/invalid AND only `STORAGE_KEY` remains** → *interrupted enrollment*: plaintext is still authoritative and usable; offer to re-run or abandon enrollment. **Never** delete the plaintext.
5. **Meta says `vault`, envelope valid, but `STORAGE_KEY` also still present** → *enrollment removal step didn't complete*: vault is authoritative; the leftover plaintext is a **data-exposure hazard** and must be surfaced ("a readable plaintext copy remains — remove it") and removed only after the user unlocks and confirms.

The invariant: **exactly one copy is authoritative; a second copy is only ever a labeled, user-visible transient, never silently trusted or silently deleted.**

---

## 6. Startup state machine

The synchronous bootstrap reads `…-vault-meta-v1` and the presence of the three keys **before** loading any records, then enters exactly one state. Records are loaded into `state` (via `Object.assign(state, decrypted)`) **only** in the Unlocked state.

| State | Keys that may exist | Records in memory? | UI shown | Allowed | Blocked | Enter from | On reload / termination |
|---|---|---|---|---|---|---|---|
| **Plaintext** | `STORAGE_KEY`; no vault keys | Yes (as today) | Full app | Everything (today) | — | fresh install, plaintext user | Plaintext |
| **Enrollment pending** | `STORAGE_KEY` (+ transient meta `mode:migrating`) | Yes (plaintext still authoritative) | Enrollment wizard over usable app | Cancel (→Plaintext), proceed | Auto-migrate without confirm | Plaintext + user "Enable vault" | Re-detects `migrating`; offers resume/abandon; plaintext intact |
| **Migration in progress** | `STORAGE_KEY` + `…-pending` + meta `mode:migrating,phase` | Yes | "Encrypting…" busy | wait/cancel | edits during write | Enrollment after passphrase confirmed | §7 recovery table by phase |
| **Vault enabled + Locked** | `…-vault-v1` + meta `mode:vault` | **No** | **Locked shell** (unlock form only) | unlock, import *encrypted* backup→recovery, capability/Help/About, download inaccessible ciphertext | all record UI, export of records, self-tests, normal shortcuts | any reload of a vault; after manual lock | Locked |
| **Unlock in progress** | same | No | Locked shell, busy | wait, cancel | duplicate submit, record UI | Locked + passphrase submit | Locked (async result discarded) |
| **Vault enabled + Unlocked** | `…-vault-v1` (+ meta) | Yes | Full app | Everything + manual Lock, disable-vault, encrypted persist | writing plaintext `STORAGE_KEY` | successful unlock | Locked (key never persisted) |
| **Persistence in progress** | `…-vault-v1` (+ transient during swap) | Yes | Full app + "saving…" indicator | edits (coalesced) | manual Lock / disable / destructive nav until flushed | any state change while unlocked | last-known-good ciphertext remains |
| **Persistence failed** | `…-vault-v1` (old, intact) | Yes | Full app + persistent "unsaved — export now" banner | export, retry | silent success claim | encrypt or setItem failure | old ciphertext authoritative; unsaved delta lost on reload unless exported |
| **Manual locking** | `…-vault-v1` | transitioning → No | brief busy → Locked shell | — | new edits | user "Lock" (only when not persisting) | Locked |
| **Vault disabling pending** | `…-vault-v1` + (new `STORAGE_KEY` staging) | Yes | disable wizard | verify + confirm, cancel | ambiguous dual-authoritative | Unlocked + "Disable vault" | §7 recovery table |
| **Recovery required** | any inconsistent set | No | recovery panel (import backup / erase+restore / Start Fresh) | import, erase-and-restore, download diagnostic | normal record UI | auth failure, corrupt vault, inconsistent keys | Recovery required |
| **Corrupt / missing vault** | meta `mode:vault`, envelope invalid/absent | No | recovery panel | as above | pretend-empty success | validation failure at startup | Recovery required |
| **Unsupported cryptography** | any | No | "This browser can't run the required encryption" | plaintext-only fallback *only if plaintext mode*; in vault mode: refuse, offer import-to-new-browser guidance | enabling/using vault | `crypto.subtle` probe fails | same |
| **Unsupported future vault version** | `vaultEnvelopeVersion>1` | No | "Made by a newer version" refusal | download ciphertext, Help | opening/erasing-by-accident | future envelope | same |

**No-flash guarantee.** Because the vault branch is taken in the synchronous bootstrap *before* `loadState()` would read any records, and because in vault mode `STORAGE_KEY` does not exist (nothing to load), there is **no moment** where records are in `state` or the DOM while locked. The locked-shell render is a **distinct render path** that does not call the normal tab/record renderers. To keep initialization, navigation, keyboard shortcuts, dialogs, import/export, self-tests, and ordinary event handlers unavailable until the correct state is established: the tail wiring must be **gated** — in Locked state, wire *only* the unlock form and the minimal locked-shell controls; wire the full handler set (and call the normal `render()`) *only* on successful unlock. `handleShortcuts`, `exportBtn`, `importBtn` record paths, and self-test dispatch must all check a single `appMode` gate and no-op while locked. (Self-tests that seed a synthetic vault run in their own disposable context and set the gate explicitly.)

---

## 7. Enrollment / plaintext→vault migration protocol + crash-recovery table

**Enrollment sequence (refined from the intended sequence):**

1. User explicitly chooses **"Enable encrypted local vault."**
2. App **requires an actual successful recovery-backup download in this session** (recommended over acknowledgment-only — see decision below), accepting **either** a plain **or** an S1/S2 encrypted backup as the recovery file.
3. Explicit acknowledgment checkbox: "I have saved a recovery file."
4. App states plainly: **no password reset, no backdoor; forgetting the passphrase makes the vault unrecoverable without a valid backup.**
5. Passphrase + confirm. A strength meter may **advise** but must **not** trim, normalize, silently reject, or alter any non-empty passphrase. The only enforced policy is a deliberate, disclosed **non-empty minimum** (empty string rejected); any length/complexity minimum beyond that must be justified and disclosed — recommend **no complexity gate**, only reject `''`.
6. Read the complete current plaintext live document (from in-memory `state`, serialized identically to today's `persist()` payload).
7. **Write phase A:** derive key (fresh 16-byte salt, chosen iterations), encrypt with a fresh 12-byte IV, write the envelope to **`…-pending`**; write meta `{mode:'migrating', phase:'pending-written'}`. **`STORAGE_KEY` untouched.**
8. **Verify phase:** read `…-pending` back, structurally validate, decrypt, `JSON.parse`, normalize, and **compare canonically** to the source (see comparison method below). Any failure → delete `…-pending`, meta back to plaintext, error; **plaintext intact and usable.**
9. **Commit phase:** on verify success, atomically-as-possible: copy `…-pending` → `…-vault-v1`, set meta `{mode:'vault', phase:'committed', lastWriteGeneration:N}`, delete `…-pending`.
10. **Plaintext-removal step (separate, explicit):** only **after** the user reloads-and-unlocks once (proving the round-trip on a cold start) **and** gives a second explicit confirmation, remove `STORAGE_KEY`. Until then, `STORAGE_KEY` remains as a labeled, readable **recovery copy** with UI stating it still exposes data. (Optionally rename to `…-v1-pre-vault-recovery` at commit, so the app knows it is a deliberate leftover, not the authoritative live doc.)
11. Any failure/cancel at any step leaves the original plaintext authoritative and usable; transient artifacts are removed.

**Decisions settled:**
- **Require a real backup download, not acknowledgment.** A checkbox is trivially lied to; a completed download materially reduces permanent-loss risk, which is the dominant threat here. Accept **both** plain and encrypted backups as the recovery file (plain is a valid recovery source; encrypted is preferred for confidentiality).
- **Confirmation order:** (a) recovery-file saved → (b) "no reset/backdoor" understood → (c) passphrase set/confirmed → (d) migrate+verify → (e) reload+unlock proof → (f) explicit "remove readable plaintext copy." Vault becomes the *authoritative* copy at step 9 (meta `mode:vault` + valid `…-vault-v1`); plaintext removal is deferred to step (f).
- **Verification requirements:** structural-validate → decrypt → parse → normalize → **canonical compare**. Comparison method: compare the normalized re-serialization of the decrypted document to the normalized serialization of the source (e.g. `JSON.stringify` over a stable key ordering, or a hash thereof). Because `persist()` already re-normalizes collections deterministically, comparing `normalize(source)` to `normalize(decrypt(vault))` is well-defined; do not compare raw strings (key order / whitespace differences would false-fail).
- **Transient key + phase marker are BOTH necessary.** `localStorage` has no multi-key transaction; the `…-pending` staging key plus a `pendingTransition` phase marker in meta constitute a **recoverable journal**: startup reads the phase and the actual key set and resolves deterministically.
- **Transient passphrase/key release:** the passphrase string variable is overwritten and dropped as soon as the key is derived; the derived `CryptoKey` (non-extractable) is held in one module-scoped reference only while unlocked; on cancel/failure both are dropped. (JS cannot guarantee erasure — stated.)

**Crash / interruption recovery table** (next startup discovers state safely from keys + phase):

| Interruption point | Keys present | Authoritative copy | Automatic action | User action offered |
|---|---|---|---|---|
| Before phase A | `STORAGE_KEY` | Plaintext | none | none (nothing happened) |
| After `…-pending` write, before verify | `STORAGE_KEY` + `…-pending` + meta `migrating` | **Plaintext** | delete stale `…-pending` on confirm | "Resume enabling vault?" / "Discard" |
| After verify, before commit | `STORAGE_KEY` + `…-pending` (verified) | **Plaintext** (safe default) | none auto | "Finish enabling vault" (re-verify then commit) / "Discard" |
| After commit, before plaintext removal | `…-vault-v1` + `STORAGE_KEY`(leftover) + meta `vault` | **Vault** | none auto | Locked shell; after unlock: "A readable plaintext copy remains — remove it" |
| After plaintext removal | `…-vault-v1` + meta `vault` | Vault | none | normal Locked → Unlock |
| Steady-state persist crash mid-write | `…-vault-v1`(old) [+ `…-pending`(partial)] | **Old `…-vault-v1`** | delete partial `…-pending` | none (last-known-good intact); "some recent changes may not be saved" if generation gap |
| Disable mid-write | `…-vault-v1` + `STORAGE_KEY`(staging) + meta `disabling` | resolve by phase (see §9) | never delete both | "Finish disabling" / "Keep vault" |

---

## 8. Encrypted persistence / write-coordinator contract

The synchronous-`persist()`-with-rollback contract (§2) forbids making `persist()` `async`. Recommended design: **`persist()` stays synchronous and boolean; in vault-unlocked mode it enqueues an encrypted write and returns `true` (accepted-for-write); asynchronous encryption/quota failure is surfaced through the existing `showStorageWarning()` + a persistent "unsaved changes" indicator, not through the boolean.**

**Write-coordinator contract:**
- **One derived key per unlocked session.** PBKDF2 runs exactly once at unlock; the resulting non-extractable `CryptoKey` is reused for every write. Never re-derive per write (fixture: derive-count == 1 across many writes).
- **Fresh IV per write, same salt+key.** Each encrypted write generates a new 12-byte CSPRNG IV; the enrollment salt and derived key are retained. **AES-GCM IV must never repeat under the same key** — a fresh CSPRNG 12-byte IV per write gives negligible collision probability, and a monotonic `writeGeneration` in AAD additionally guards against replaying an old IV/ciphertext. Do **not** use a counter-derived IV (a crash + counter reset could repeat one); random-per-write is safer here.
- **Serialized, coalesced queue.** A single in-flight encrypted write at a time; while one is running, further `persist()` calls mark the state "dirty (latest)"; when the current write finishes, if dirty, encrypt-and-write the *current* `state` once (latest-wins coalescing). This prevents concurrent PBKDF2 (there is none — key is cached), out-of-order writes, older ciphertext replacing newer, and duplicate persistence.
- **Generation counter.** Each successful write increments `writeGeneration` (in the envelope AAD and mirrored in meta). On the next unlock, if the envelope generation is *older* than meta's last-known generation, warn (possible interrupted write / other tab). This is the out-of-order/stale-write detector.
- **Last-known-good stays intact until replacement succeeds.** For steady-state writes, the in-memory `state` is the true source, so a failed write does not corrupt anything; the coordinator must not delete/overwrite the good `…-vault-v1` until the new ciphertext is computed and `setItem` succeeds. If `setItem` throws (quota), keep the old `…-vault-v1`, keep `state` in memory, raise the unsaved-indicator + `showStorageWarning()`.
- **`beforeunload` cannot be relied on for async encryption.** Encryption is async; `beforeunload` cannot await it. Therefore: (a) encrypt eagerly (don't batch on a long timer — reuse the existing 400 ms pricing-debounce cadence only for the pricing field, persist other changes promptly), (b) show a visible "saving…/unsaved" indicator whenever a write is pending, (c) **block manual Lock and destructive navigation (disable/import Replace) while a write is pending** — flush first, and (d) accept that a hard tab-close during an in-flight write loses only the last uncommitted delta (the previous generation remains authoritative), which is the standard, safe outcome. A best-effort synchronous `beforeunload` warning ("changes may not be saved") is appropriate when a write is pending.
- **No silent plaintext fallback, ever.** If encryption is unavailable while in vault mode, the coordinator must fail loud (unsaved indicator + warning), never write `STORAGE_KEY` plaintext.

**Affected `persist()` callers:** all ~45 sites keep calling `persist()` unchanged. The 9 rollback-on-`false` sites (§2) keep working because the synchronous pre-checks (recovery-blocked, not-unlocked) still return `false` synchronously; only *asynchronous* encryption failure moves to the warning channel — which is acceptable because in that case the in-memory state is still authoritative and the user is told to export. **No caller signature changes**; the change is entirely inside `persist()` and a new coordinator module. `flushPricingPersist` (beforeunload) must additionally request a coordinator flush.

---

## 9. Manual lock and vault-disabling decisions

**Manual lock — INCLUDE in S3 (safe, bounded).** On "Lock": (1) **require pending persistence to complete** (or block with "saving… try again")—never lock with an unflushed encrypted write; (2) drop the derived-key reference and the passphrase variable; (3) invalidate any pending async tokens (unlock/write) so a stale completion cannot re-enter Unlocked; (4) close all dialogs, cancel in-flight operations; (5) reset `state` to `freshState()` and clear DOM containers/previews that hold record data; (6) render the Locked shell; (7) restore focus to the unlock field. **State plainly that JavaScript cannot guarantee secure memory erasure** — a full **page reload remains the strongest practical lock**, and the Lock button should be described as "lock (or reload for a full clear)." Idle/timed auto-lock is **deferred to S4**.

**Vault-disabling — DEFER to the last S3 sub-phase (S3-4), and it is safely deferrable.** If included, the reverse migration mirrors enrollment: require unlock → strongly recommend/require a recovery backup → explain "live browser storage will become readable plaintext" → write `STORAGE_KEY` from current state into a **staging** position and **verify a plaintext round-trip** → set meta `mode:disabling,phase` → only after verification + explicit confirm, remove `…-vault-v1` and finalize `STORAGE_KEY` as authoritative → clear meta. Recover from quota/cancel/crash via the phase marker; never leave both `…-vault-v1` and an authoritative `STORAGE_KEY` ambiguously "live." **If S3 is getting too large, disable can be omitted from the first S3 release** because users retain a complete recovery path without it: export a backup while unlocked, then use Recovery → "erase vault and restore from backup" to land in plaintext, or Start Fresh. Document this interim clearly.

---

## 10. Forgotten-passphrase and recovery design

**No backdoor, no verifier, no "reset password."** The GCM tag is the only correctness oracle; a valid backup is the only recovery source.

- **Entry points to recovery:** the Locked-shell offers, alongside unlock: "Can't unlock? Restore from a backup." The Recovery panel offers: **import a valid plaintext backup**, **import a valid S1/S2 encrypted backup** (asks for that file's passphrase — distinct from the vault passphrase), **download the inaccessible vault ciphertext** as a diagnostic artifact, and **Start Fresh** (empty workspace).
- **Restore requires explicit erasure of the inaccessible vault.** Because the old ciphertext cannot be opened, restoring means replacing it: the UI says **"Replace/erase the inaccessible vault and restore from this backup"** with a **destructive confirmation**, never "reset password." Before erasure, offer the ciphertext download so a user who later remembers the passphrase (or recovers a keystroke) retains a chance.
- **After a valid restore:** the user chooses to land in **plaintext mode** (default, simplest) *or* immediately re-enroll a **new** vault passphrase (full enrollment flow). Re-enrollment must not silently overwrite — same explicit confirm + backup as first enrollment.
- **Inconsistent-state recovery (all resolve to the Recovery panel, never to data loss or silent clear):**
  - Corrupt/invalid `…-vault-v1` → "unreadable or damaged" + import/erase/diagnostic options.
  - Missing bootstrap meta but valid envelope → reconstruct meta from the envelope's authenticated header (envelope is authoritative); proceed to Locked.
  - Meta says `vault` but envelope key missing → interrupted enrollment/removal (§5.4 / §7): if `STORAGE_KEY` still present, plaintext is authoritative and usable; otherwise Recovery.
  - Only old plaintext `STORAGE_KEY` remains after an interrupted migration → treat as plaintext-authoritative and usable; offer to re-enroll.
- **Start Fresh** keeps its existing destructive-confirm pattern and additionally warns that it does **not** decrypt or recover the vault. Recovery snapshots (the existing damaged-data download model) extend naturally: the inaccessible ciphertext is the "damaged raw" analogue.

---

## 11. Backup / import interaction (S2 preserved, extended)

| Scenario | Behavior |
|---|---|
| Export while **locked** | Record export disabled ("unlock first"). Optionally allow downloading the **opaque vault file** as a distinct artifact (see below). |
| Export while **unlocked**, plain | Existing S2 plain path + readable-data warning; content = current unlocked `state` via `backupObject()`; no plaintext written to live storage. |
| Export while **unlocked**, encrypted (S2) | Existing S2 encrypted-backup path, independent passphrase; unchanged. |
| Import while **locked** | Only an **encrypted S1/S2 backup** may be imported as part of **recovery** (erase-and-restore); a plain/normal import that would populate records is disabled until unlock (no record data before unlock). |
| Import while **unlocked** | Full existing routing (`routeBackupImportData`): encrypted-envelope branch or plaintext branch → `validateBackupImport` → Merge/Replace (second confirm on Replace). Applied result then persists **through the vault path** — no plaintext copy is ever written. |
| Plain/encrypted **Merge/Replace** | Unchanged Merge/Replace engine; the only difference is the post-apply `persist()` encrypts. |
| Import during **enrollment** | Blocked/queued until enrollment resolves (avoid racing two storage transitions). |
| Import during **persistence-failed** | Allowed (import is itself a recovery); resolves the unsaved state by replacing it. |
| Backup before **disabling** | Strongly recommended/required, exactly as enrollment. |

**Opaque vault file (decision): DEFER past S3-1/2; optional in a later sub-phase.** A downloadable copy of the raw `…-vault-v1` envelope (distinct from a normal backup — it is at-rest ciphertext, only openable by unlocking a matching vault) is a reasonable *diagnostic/portability* artifact, but it is **not** a substitute for an S2 encrypted backup (which is the portable, re-importable format). Keep the diagnostic download in Recovery (§10); do not add a general "download vault file" feature to S3's core. **Imported data applied while unlocked must follow existing validation + Merge/Replace, then persist via the vault path with no plaintext copy left behind** — this is a hard requirement, satisfied because the import apply path already ends in `persist()`, which will be the vault coordinator in vault mode.

---

## 12. Multi-tab and concurrency strategy

**Simplest release-safe strategy: single-writer, defensive per-write mode re-check, storage-event UX assist, no conflict merging.**

- **Detection:** on load, each tab records a per-tab session id and reads meta. A `storage` event listener (fires in *other* same-origin documents on `setItem`/`removeItem`) updates each tab when meta/vault changes. **However, `storage`-event reliability under `file://` is not guaranteed** across Chromium builds, so it is a UX assist only, not a safety mechanism.
- **Primary guard — defensive re-check inside `persist()`:** every `persist()` re-reads meta `mode` immediately before writing. If a tab believes it is in Plaintext mode but meta now says `vault` (another tab enrolled), it **must not write plaintext** — it refuses, drops to a "vault was enabled in another tab; reload to continue" locked notice. This closes the "older pre-enrollment tab recreates plaintext after enrollment" hole deterministically, independent of storage-event delivery.
- **Write-generation guard:** the coordinator refuses to overwrite `…-vault-v1` if the on-disk envelope's `writeGeneration` is newer than the generation this tab last wrote (another unlocked tab advanced it) → surface "changes were saved in another tab; reload before editing" rather than clobbering.
- **Recommendation: block concurrent *editing*, do not merge.** If two tabs are unlocked simultaneously, the second-to-write is told to reload; the app does not attempt to merge divergent workspaces (there is no field-level merge model, and inventing one is out of scope and dangerous). For a single-user offline workshop app this is the correct, minimal, safe choice.
- **On enable/disable/replace/lock/erase in one tab:** other tabs detect via storage-event (assist) and/or the next `persist()`/generation re-check (guard) and route to a "reload required" locked notice. No other tab may act on stale record data.

---

## 13. Quota and payload analysis

- **Current size:** the plaintext workspace is text-heavy but can include embedded Project photos (data-URIs), which already trigger a size confirmation on export today. Photo-laden workspaces can reach multiple hundreds of KB to low MB.
- **Ciphertext/Base64 expansion:** AES-GCM adds a 16-byte tag; Base64 inflates by ~**1.37×**. So the vault envelope ≈ 1.37× the plaintext JSON plus small header — comparable to today's plaintext footprint order-of-magnitude.
- **Migration peak duplication:** during enrollment both `STORAGE_KEY` (plaintext) and `…-pending`/`…-vault-v1` (ciphertext) coexist → transient peak ≈ **~2.4×** the plaintext size. Disable has a symmetric peak. `localStorage` budgets are typically ~5–10 MB per origin; a text/photo workshop workspace is normally well within this even at 2.4×, but a very photo-heavy workspace could approach the ceiling.
- **Quota handling:** there is no reliable preflight for `localStorage` quota. The safe pattern is **attempt the staging write and catch `QuotaExceededError`**, leaving the authoritative copy intact and showing an actionable message ("storage is full — remove Project photos or older records, then retry"). This mirrors the existing `showStorageWarning()` behavior. Never delete the old copy to make room for a new write that hasn't succeeded.
- **Synthetic-scale testing:** S1 already round-trips a **1 MiB** payload in a helper fixture. S3 fixtures should exercise a ~1 MiB *workspace* through enrollment → vault persist → unlock → compare, and a quota-injection path.
- **Future SVG Asset Library — clear statement, do NOT design into S3:** a whole-document `localStorage` AEAD blob does **not** scale to a large binary asset store (re-encrypting/re-writing the entire vault on every asset change is O(total) per write, and `localStorage`'s ~5–10 MB ceiling is far too small for an SVG/image library). A future asset store will need **IndexedDB with its own encrypted storage contract** (per-asset or streamed AES-GCM records, its own key-wrapping tied to the same unlocked session key, its own versioned envelope). S3's vault key/session model can *supply the session key* to such a future store, but the storage format and coordinator for assets must be a **separate future contract**, not an extension of the `localStorage` whole-blob vault. This is explicitly **out of S3 scope**.

---

## 14. Accessibility and locked-shell UX

- **Locked shell must expose zero record-derived information** — no counts, no recent project names, no previews, no search index, no "12 projects" summaries. Only: app name/version, unlock form, offline-privacy blurb, Help/About, capability status, and recovery entry.
- **Reuse the existing modal accessibility infrastructure** (`openModal`/`bindModal`/`closeModal`, focus-origin restoration, `aria-live` status, heading ids) that the Modal Accessibility group (38/0) already covers.
- **Requirements:** initial focus on the passphrase field; focus trap within the unlock/enrollment dialogs; explicit labels and instructions; `aria-busy` during PBKDF2/encryption; error text announced via `aria-live` (wrong passphrase / corruption distinguished carefully — see §15); full keyboard operation; Escape/close semantics consistent with existing modals; focus restoration on close; password-manager attributes (`autocomplete="current-password"` on unlock, `new-password` on enrollment) — without over-claiming manager behavior; a reveal-passphrase toggle; strength-meter output that is advisory and screen-reader-labeled; **Caps-Lock guidance** if practical (a `getModifierState('CapsLock')` hint on keydown); an unsupported-browser message when `crypto.subtle` is absent; and a **clear, persistent distinction between the local-vault passphrase and encrypted-backup passphrases** (different labels, different dialogs, copy that says "this is your vault passphrase, not a backup-file password").

---

## 15. Failure and no-mutation matrix

Authoritative-copy and mutation guarantees for every major operation:

| Operation / event | State mutation | Storage mutation | Authoritative copy after | Discovered next startup by |
|---|---|---|---|---|
| Cancel / Escape / dialog close (any wizard) | none | none | unchanged | — |
| Wrong passphrase (unlock) | none | none | vault (still locked) | meta+envelope |
| Unsupported cryptography | none | none | unchanged | capability probe |
| Invalid envelope / future version / foreign app | none | none | recovery | strict validation |
| Tampering (any AAD/ciphertext bit) | none | none | recovery (auth fail) | GCM tag |
| Malformed decrypted JSON / invalid decrypted state | none | none | recovery | post-decrypt guard |
| Encryption failure (write) | none | old ciphertext kept | old `…-vault-v1` | generation/meta |
| Decryption failure | none | none | recovery | GCM tag |
| `setItem` write failure (quota) | none | old copy kept, new discarded | old `…-vault-v1` | unchanged envelope |
| `removeItem` failure (leftover plaintext) | none | both may exist | vault (labeled leftover) | §5.4 rule 5 |
| Stale async completion (after cancel/lock/transition) | **none** — token invalidated | none | current state | token check |
| Repeated activation (double submit) | at most one op | at most one write | consistent | queue/token |
| Page reload | in-memory dropped | intact | last committed copy | key set + meta |
| Browser crash | in-memory dropped | last durable write | last committed copy | key set + phase |
| Interrupted migration | none to authoritative | staging may exist | **plaintext** (safe default) | §7 table |
| Interrupted disabling | none to authoritative | staging may exist | resolve by phase, never both | §7 table |
| Import cancellation / Replace cancellation | none | none | unchanged | existing pending-import guards |

**Error distinction:** wrong-passphrase and corruption are *both* GCM `OperationError` and are cryptographically indistinguishable after PBKDF2. Distinguish them **only** by the *pre-PBKDF2 structural validation* result: if `validateVaultEnvelope` fails → "damaged/unreadable" (corruption); if it passes but decrypt throws → "could not open — check the passphrase" (most likely wrong passphrase, possibly tamper). Never claim certainty. **Repeated failed attempts:** in an offline app, storage-based rate limiting is theater (an attacker with storage access edits the counter); use only a small in-memory throttle/delay for UX (discourage fat-fingering), reset on reload, and do **not** persist an attempt count.

**Stale-async invariant:** every long async op (unlock, write, migrate) carries a monotonic operation token captured at start; on completion it verifies the token still matches the module's current token (unchanged by any intervening cancel/lock/transition) before applying results. A cancelled/superseded op's completion is discarded. This prevents a late unlock from re-entering Unlocked after the user cancelled or after a vault replacement.

---

## 16. Test and verification architecture

**Focused group name:** `Security S3 Local Vault`. **Route:** `?selftest=security-s3-local-vault`. **Async integration:** register exactly like S1/S2 — start once, expose `window.securityS3LocalVaultFixturePromise` and a run-counter, and have `runRequestedSelftests` await it before completion; ordinary startup (no query) must run **no** vault fixture and **no** PBKDF2 (assert derive-count 0 on plain startup). All fixtures use **disposable synthetic records, disposable storage keys, and a disposable/injected crypto seam**; each restores `state`, all three vault keys, `STORAGE_KEY`, and modal/DOM to their pre-fixture values (mirror the existing `stateBefore`/`storageBefore` restore discipline). **Never** touch Joe's real profile/records/backups/passphrases.

**Coverage (minimum):**
- Plaintext startup byte-unchanged; vault-disabled `persist()` writes identical plaintext to today.
- Enrollment: cancel at every stage; verified success; failed migration leaves plaintext usable; interrupted migration recovery at **each** persisted phase (§7 rows) via seeded key sets.
- Locked startup exposes **no** record data (assert no record strings/counts in locked DOM; assert `state` has no records).
- Correct-passphrase unlock loads records; wrong-passphrase no mutation; tamper (each AAD field + ciphertext) no mutation; corrupt/absent envelope → recovery; future `vaultEnvelopeVersion` refused; foreign `app` refused; invalid envelope rejected **before** any `deriveKey` (assert derive-count 0 on each invalid class).
- Passphrase absent from `localStorage` **and** `sessionStorage` after every op (scan all keys/values); derived key absent from storage; reload requires unlock.
- Manual lock: state cleared, key ref dropped, pending tokens invalidated, DOM record containers cleared, returns to locked shell.
- Coordinator: derive-count **== 1** across many writes per unlock; **fresh IV every write** (collect IVs, assert all-unique); **no IV reuse**; serialized/coalesced (N rapid `persist()` → ≤ expected writes, latest state wins); out-of-order completion resistance (inject delayed encrypt); duplicate-action suppression; stale-op cancellation discarded.
- Quota failure keeps old ciphertext authoritative; persistence failure surfaces indicator + warning, no silent success, no plaintext fallback.
- Multi-tab: seed meta `mode:vault` then call a plaintext-era `persist()` → refuses to write plaintext; generation-guard refuses stale overwrite.
- 1 MiB workspace round-trip through the vault path; Unicode/emoji and large Base64-like inner content survive round-trip.
- Merge/Replace through an unlocked vault (parity vs plaintext apply, then persists as ciphertext); plain and S1/S2 encrypted backup Export/Import still function unchanged; recovery via plaintext backup; recovery via encrypted backup; disable (if included) restores plaintext + verifies; Start Fresh destructive confirm.
- No unhandled promise rejections; no console/page errors in an adversarial session; complete state/storage cleanup after disposable fixtures; **existing S1 (21/0) and S2 (17/0) totals unchanged**; **every registered production SVG golden byte-identical** (design group 1093/0); ordinary startup runs no crypto fixtures.

**Production functions fixtures must exercise (not reimplement):** the real vault envelope validator/encrypt/decrypt, the real `persist()` coordinator, the real `loadState`/startup branch, the real enrollment/migration/disable functions, the real recovery routing, and the real Merge/Replace apply path.

**Direct-file browser plan:** disposable Edge `--user-data-dir`, `file://`, `?selftest=security-s3-local-vault` and `?selftest=all`; drive enrollment → reload → unlock → edit → lock → unlock → disable via real DOM interaction on synthetic data; **kill the browser mid-migration** and re-open to prove phase recovery; truncate/tamper the seeded envelope; inject unavailable crypto. **Chrome if installed** (do not install it) — S1/S2 could not, so Chrome remains an open verification item. Measure PBKDF2 and encrypted-persist wall time (local, non-generalizable). Prove complete state/storage equality by full before/after snapshots of `state`, all four storage keys, and `backupObject()`.

---

## 17. Recommended S3 implementation phases (and why partial-vault ordering matters)

**A partially-implemented vault is dangerous if "enable" ships before "unlock" and "recovery" both work** — a user could enroll and then be permanently locked out with no rescue. Therefore the ordering is **recovery-capable-before-enrollable**, and enrollment ships **last** among the core phases:

| Sub-phase | Scope | New storage keys | New transitions | Existing fns changed | Fixtures | Browser checks | Deferred | Committable safely? |
|---|---|---|---|---|---|---|---|---|
| **S3-1 Foundation** | Vault envelope format + helpers (sharing S1 primitives); meta reader; **synchronous startup branch**; locked-shell render + gated handler wiring; unlock flow; **encrypted-persist write coordinator**. **No user-reachable "Enable" — vault reachable only by fixture-seeded keys.** | `…-vault-v1`, `…-vault-meta-v1`, `…-pending` (defined/handled, not user-created) | Plaintext, Locked, Unlock-in-progress, Unlocked, Persist states | `persist()` internals, `loadState`/startup tail (gated), `flushPricingPersist` | full unlock/persist/coordinator/locked-shell/no-flash set | Edge seeded-vault unlock/persist/lock; no-PBKDF2 plain startup | enrollment, recovery UI, disable | **Yes** — real users see zero change (no vault keys exist → plaintext path); dead-to-users but fully tested |
| **S3-2 Recovery-first** | Recovery panel for vault (import plaintext/encrypted backup → erase-and-restore; diagnostic ciphertext download; Start Fresh for vault); inconsistent-key resolution (§5.4/§7). Still **no Enable**. | none new | Recovery-required, Corrupt/missing | recovery routing, import gating in vault mode | recovery-from-each-inconsistency, erase+restore, diagnostic download | Edge seeded-corrupt-vault → recovery | enable, disable | **Yes** — guarantees a rescue exists *before* anyone can enter vault mode |
| **S3-3 Enrollment + manual lock** | Expose **"Enable encrypted local vault"** with required-backup + verified migration protocol (§7); manual Lock. **First commit where a real user can enter vault mode** — unlock (S3-1) and recovery (S3-2) already exist and are tested. | user now creates the keys | Enrollment-pending, Migration, Manual-lock | enrollment/migration fns, Export (require backup hook), Lock | migration cancel/verify/interrupt-recovery, lock, multi-tab, quota | Edge full enable→reload→unlock→lock; kill-mid-migration | disable, idle lock (S4) | **Yes** — enable is safe only because 1+2 shipped first |
| **S3-4 Disable (optional)** | Reverse migration to plaintext (§9), symmetric verify + phase recovery. | reuse staging | Disabling-pending | disable fns | disable verify/interrupt | Edge disable→reload plaintext | idle lock (S4) | **Yes** — deferrable; interim recovery path (§9) covers users without it |

**Recommendation:** implement **S3-1, S3-2, S3-3 as three separate commit-ready phases in that exact order**; **S3-4 optional/deferrable**. This is *not* a single atomic S3 — splitting is *safer* here because it guarantees the "recovery before enrollment" invariant at every commit boundary. The one thing that must never be split apart is **within** a phase: e.g. S3-1's unlock and persist must land together (an unlockable-but-unpersistable vault is useless), and S3-3's migration and its crash-recovery handling must land together.

**Exact first Codex implementation boundary:** **S3-1 Foundation only** — vault envelope format + shared-primitive helpers, `…-vault-meta-v1` reader, the synchronous startup mode-branch, the locked-shell render with gated handler wiring (no-flash), the unlock flow, and the encrypted-persist write coordinator, all exercised **exclusively by fixtures that seed a synthetic vault**; **no** user-facing "Enable" control, **no** enrollment, **no** recovery UI, **no** disable. Exit criteria: plain startup unchanged (derive-count 0, byte-identical plaintext persist), seeded-vault unlock/persist/lock green, no-flash proven, S1/S2 and all goldens unchanged.

**Independent-verification boundary after each commit-ready phase:** a mandatory independent adversarial verification before each of S3-1/S3-2/S3-3 (and S3-4 if built) is committed — matching the S1/S2 precedent — focused on: no plaintext user regression, no-flash locked startup, no-mutation matrix, IV-uniqueness/derive-once, recovery leaves no dead-end, and all protected boundaries + goldens intact.

---

## 18. Protected boundaries

S3 must **not** casually change any of: existing `STORAGE_KEY` string, `SCHEMA_VERSION`, `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, plaintext `BACKUP_FORMAT`, the S1/S2 encrypted-backup format and behavior, `backupObject` structure/meaning, all record schemas/identities (Project/Library/Inventory/Pricing/machine/evidence), evidence matching/promotion/revocation, Designs geometry, SVG serialization, production filenames, **production SVG bytes**, direct `file://` support, offline/zero-network operation, and unrelated generators. Confirmed by design: every S3 change is **additive** (new vault keys/format/coordinator/states/UI) or **internal to `persist()`/startup wiring**; no generator, serializer, matcher, schema, or golden is touched. **Any necessity to change a protected boundary would be a blocking architecture issue — none is identified.** In particular, S3 does **not** require a `SCHEMA_VERSION` bump (the inner document keeps today's schema; vault versioning is independent), and does **not** change `STORAGE_KEY`'s string (only its lifecycle: present in plaintext mode, removed in vault mode).

---

## 19. Findings by severity

**Critical:** none. (No confirmed current defect; plaintext live storage is an accepted product posture, and the vault is the mitigation being designed.)

**High (architecture decisions required before S3 code, all resolved above):**
- **H1 — Synchronous startup + synchronous `persist()`/rollback vs. async crypto.** *Boundary:* `loadState`/init tail and `persist()` (+9 rollback sites). *Risk:* naive async conversion breaks rollback callers or flashes records before lock. *Resolution:* synchronous mode-branch that loads no records in vault mode; `persist()` stays sync-boolean and enqueues async encryption with failures on the warning channel (§6, §8). *Blocks S3?* Must be honored by S3-1; not a defect. *S4?* No.
- **H2 — `localStorage` has no multi-key atomicity.** *Boundary:* enrollment/disable. *Risk:* a crash between "write ciphertext" and "remove plaintext" yields ambiguous dual copies or loss. *Resolution:* dual-key `…-pending` staging + `pendingTransition` phase journal + authoritative-copy precedence + never-delete-both (§5.4, §7). *Blocks S3?* Honored by S3-3. *S4?* No.
- **H3 — AES-GCM IV reuse under one session key.** *Boundary:* write coordinator. *Risk:* catastrophic GCM confidentiality/integrity loss if an IV repeats with the cached key. *Resolution:* fresh 12-byte CSPRNG IV per write + `writeGeneration` in AAD; no counter-derived IV; derive-once/IV-unique fixtures (§8, §16). *Blocks S3?* Honored by S3-1. *S4?* No.
- **H4 — "Enable" shipping before a rescue exists.** *Boundary:* phase ordering. *Risk:* permanent lockout with no recovery. *Resolution:* recovery-first, enable-last phase order (§17). *Blocks S3?* Determines commit ordering. *S4?* No.

**Medium (required S3 behavior / non-blocking gaps):**
- **M1 — Multi-tab plaintext-resurrection hole.** *Boundary:* `persist()`. *Risk:* an old pre-enrollment tab writes plaintext after another tab enrolled. *Resolution:* defensive per-write meta re-check refusing plaintext when `mode:vault` (§12). *Blocks S3?* Required in S3-1/S3-3. *Belongs to:* S3.
- **M2 — S2 fixture thinness carried forward** (Merge/Replace parity, import tamper matrix, 1 MiB UI boundary — from the S2 verification's M1/M2). *Risk:* the vault's Merge/Replace-through-encrypt path inherits under-tested apply wiring. *Resolution:* S3-3 fixtures add encrypted Merge/Replace parity + 1 MiB workspace round-trip. *Blocks S3?* No; recommended within S3-3. *Belongs to:* S3.
- **M3 — Unsaved-write window on tab close.** *Boundary:* coordinator + `beforeunload`. *Risk:* a hard close during an in-flight encrypted write loses the last delta. *Resolution:* eager encrypt, visible unsaved indicator, block lock/destructive-nav while pending, best-effort `beforeunload` warning; accept last-delta loss as safe (previous generation authoritative) (§8). *Blocks S3?* No; required behavior in S3-1. *Belongs to:* S3.

**Low:**
- **L1 — Passphrase-vs-backup-passphrase confusion.** Distinct labels/dialogs/copy (§14). Non-blocking; S3 UX.
- **L2 — Reporting-total hygiene.** The S2 implementation report's 3041 undercounts the verified 3068/48; S3 reports should adopt the outer-return methodology to avoid recurrence. Non-blocking.

**Informational / intentionally deferred:**
- **I1 — Idle/timed auto-lock → S4** (explicitly deferred).
- **I2 — Chrome direct-file crypto still unverified** (S1/S2 environment gap; carry into S3 verification if Chrome becomes available; do not install).
- **I3 — Future SVG Asset Library needs its own IndexedDB encrypted-storage contract** — the `localStorage` whole-blob vault does not scale to a binary asset store; S3's session key may feed a future asset store, but that storage format is a separate future phase, **excluded from S3** (§13).
- **I4 — JS cannot guarantee secure memory erasure** — reflected in all lock/claim language; reload is the strongest practical clear.
- **I5 — Recovery snapshots / diagnostic ciphertext download** reuse the existing damaged-data recovery model (good foundation).

---

## 20. Explicit S4 and future-work exclusions

Out of S3 entirely: idle/timed auto-lock and lock-timeout policy; any hardening beyond manual lock; the SVG Asset Library and its encrypted IndexedDB store; recovery codes / printable recovery keys / cloud recovery / accounts / remote key escrow / password reset (all permanently out of scope, not just deferred); Argon2 or KDF changes; non-Edge browser support commitments; any 1.0 "certification" claim.

---

## 21. Unresolved decisions (all have a clear recommended default; none blocks starting S3-1)

1. **Require-real-backup vs acknowledge-only at enrollment** — recommend **require a real download**; team may accept acknowledgment-only with a stronger warning. (Recommendation favors require.)
2. **Include disable in S3 or defer** — recommend **defer to S3-4/optional**; the §9 interim recovery path covers users without it.
3. **Opaque vault-file download feature** — recommend **defer**; keep the ciphertext download only inside Recovery as a diagnostic.
4. **Any passphrase minimum beyond non-empty** — recommend **non-empty only** (no complexity gate); a deliberate length floor would need explicit justification.

---

## 22. Verdict

The S3 architecture is complete, internally consistent, reuses the frozen S1/S2 contracts without weakening them, and resolves every high-risk decision (synchronous-startup/persist vs async crypto, `localStorage` non-atomicity via a dual-key phase-journaled protocol, IV-uniqueness, authoritative-copy rules, no-flash locked startup, no-backdoor recovery, multi-tab safety, and a recovery-before-enrollment phase order) with one recommended answer each and no required change to any protected boundary, schema, or production golden.

**1. S3 ARCHITECTURE READY — Safe to prepare the first bounded implementation prompt (S3-1 Foundation), on the explicit conditions that the phases ship in the recovery-before-enrollment order of §17, that `persist()` remains synchronous-boolean with async encryption failures on the warning channel, that AES-GCM uses a fresh per-write IV with derive-once, and that a mandatory independent adversarial verification runs before each commit-ready phase.**
