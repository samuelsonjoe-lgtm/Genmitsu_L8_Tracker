# Security Architecture Review — Password-Encrypted Backups and Optional Local Vault

**Date:** 2026-07-22  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Review mode:** Independent adversarial security and data-safety architecture (read-only)  
**Implementation status:** None. No application files modified. This report is the only write.

---

## Exact final verdict

**READY FOR PHASED IMPLEMENTATION** — Safe **S1** boundary: native Web Crypto capability gate (refuse weak fallbacks) plus disposable probes, encrypted-backup **envelope design freeze**, and non-shipping harness tests. Do **not** yet change Export/Import UI or live `localStorage` until S2.

---

## 1. Actual repository state

| Check | Result |
|-------|--------|
| HEAD | `19d4834` — Graduate tabletop storage tray to Workshop-ready |
| Full hash | `19d48340a293184c748c2767ec8fd0654f7ce376` |
| Branch | `main` |
| `origin/main...main` | `0 0` (synchronized) |
| Staged | Empty |
| Tracked working tree | Clean (no modified tracked files) |
| `git diff --check` | Clean |
| Complete tracked diff | Empty |
| Untracked | Long-standing docs, LightBurn, `debug.log`, utility script — preserved, not modified |

**Identity constants (measured at runtime):**

| Constant | Value |
|----------|--------|
| `APP_ID` | `genmitsu-l8-tracker` |
| `APP_NAME` | Genmitsu L8 Tracker |
| `APP_VERSION` | `0.9.0` (pre-1.0) |
| `BUILD_DATE` | `2026-07-19` |
| `SCHEMA_VERSION` | `4` |
| `STORAGE_KEY` | `genmitsu-l8-tracker-v1` |
| `BACKUP_FORMAT` | `genmitsu-l8-tracker-backup-v1` |

**Fresh baseline suite (unique registered groups, disposable Edge profile):**  
**3003 passed / 0 failed across 46 groups.**  
Matches the expected post-T4 verified suite. `python -m html.parser index.html` exit 0.

---

## 2. Current storage and backup architecture

### 2.1 Live browser storage

- **Single key:** `localStorage['genmitsu-l8-tracker-v1']`.
- **Payload:** one JSON object written by `persist()`: schema version, Log `entries`, Library `profiles` (including nested tests, production settings, evidence, photos-in-profiles where applicable), Test Grids `grids`, Projects `projects` (including embedded photos), Inventory, tabletop raw-result collections, unit and UI preference fields, machines / `activeMachineId` / `machineIdentity`, Pricing draft and preferences.
- **Not persisted:** Designs session draft (`designDraft`), preview modes, most modal UI state, first-run onboarding dismissal (session-only), in-memory promotion drafts.
- **Encoding:** **plaintext JSON** — any process with access to the browser profile can read the full object via DevTools Application storage or disk profile inspection.
- **Copying only `index.html`:** does **not** copy records (correct). Security problem is profile-local and backup-file-local, not the HTML file alone.

### 2.2 Load / recovery

| Path | Behavior |
|------|----------|
| Missing key | `freshState()` empty workspace |
| Invalid JSON / non-object | `storageRecoveryIssue` set; temporary empty UI; **persist blocked** until recovery overwrite or import |
| Valid object | Normalize collections; machine migration via `machineRootFromData` |

Recovery UI: download damaged raw text, import backup, or explicit **Start fresh** with confirm. This pattern is the right foundation for vault authentication failures.

### 2.3 Export

- Header **Export** → `JSON.stringify(backupObject(), null, 2)` → download `l8-tracker-backup-YYYY-MM-DD.json`.
- Photo-size confirm only when embedded Project photos exist.
- **No** encryption, **no** passphrase, **no** “contains readable financial/workshop data” default warning for text-only backups.

### 2.4 Import

1. FileReader → `decodeStoredState` (must be JSON object).
2. `validateBackupImport`: refuse future `schemaVersion`, foreign `app` if present, wrong `backupFormat` if present; **legacy backups without app/format still accepted**.
3. Mode dialog: **Merge** / **Replace** / Cancel; Replace requires second confirm.
4. `applyBackupImport` → merge or replace → `persist` (may clear recovery flag).

Merge/Replace semantics for machines and records are already carefully specified; encryption must decrypt **before** these steps and must not invent new merge meanings.

### 2.5 Threat-relevant data classes in storage/backups

Pricing and cost fields, project accounting, inventory stock/costs, workshop machine lists, production settings and fit evidence, customer-ish project notes, material recipes — all **plaintext** when stored or exported.

---

## 3. Threat model and explicit non-goals

### 3.1 In scope (app can improve)

| Threat | Intended protection |
|--------|---------------------|
| Encrypted backup file left on USB / email / cloud sync | Confidentiality and integrity of records at rest in the **file** without the passphrase |
| Casual inspection of a stolen **encrypted** backup | Same |
| Optional local vault: profile storage read without passphrase | Confidentiality of live records at rest in localStorage ciphertext |
| Wrong passphrase / bit-flipped ciphertext | Authentic decryption **failure** (no silent empty workspace) |
| Accidental plain export mistaken for “secure” | UX labeling and warnings |
| Silent encryption of existing data | **Forbidden** — explicit opt-in only |

### 3.2 Out of scope / residual (must not claim)

| Threat | Reality |
|--------|---------|
| Compromised Windows account / malware / keylogger | After unlock, or while typing passphrase, attacker sees plaintext |
| Malicious browser extension with host access | Same |
| Modified `index.html` that steals passphrase or dumps memory | App cannot trust its own code if the adversary controls the file |
| Superficial “password gate” over **plaintext** storage | **Anti-goal** — security theater |
| Unattended unlocked session | Optional idle lock is polish only; OS lock is primary |
| Physical theft of powered-on unlocked machine | Outside app boundary |
| BitLocker / full-disk encryption | Complementary; not replaced |
| Forgotten passphrase | **No recovery** if user has no plaintext backup or prior encrypted backup they can open |
| Copy of only `index.html` | Already non-sensitive for records |

**Plain statement:** Application encryption protects **files and optional at-rest browser blobs**. It does **not** replace Windows sign-in, disk encryption, physical control, or safe handling of unlocked sessions.

---

## 4. Direct-file Web Crypto capability results

**Method:** Disposable Edge profile, headless, **`file://`** origin, remote-debugging evaluate of a standalone probe page (no app source mutation, no real localStorage).

**User-Agent sample:** HeadlessChrome/150… Edg/150…  
**Protocol:** `file:`

| Capability | Result |
|------------|--------|
| `crypto` | Available |
| `crypto.subtle` | Available |
| `getRandomValues` | Available |
| PBKDF2-HMAC-SHA-256 → AES-GCM-256 key | Success |
| AES-GCM encrypt/decrypt round-trip | Success |
| Wrong passphrase | `OperationError` (auth failure) |
| Tampered ciphertext | `OperationError` |
| Unique IV → different ciphertext | Success |
| AES-GCM AAD wrong → fail | Success |
| PBKDF2 210 000 iterations (this host) | ~**45 ms** |
| PBKDF2 600 000 iterations (this host) | ~**93 ms** |

**Conclusion:** Defensible **native** cryptography is available under direct offline `file://` in current Edge (Chromium). **No** network, CDN, or secure `https` origin required for SubtleCrypto in this environment.

**Fallback policy:** If `crypto.subtle` is missing or required operations throw at capability probe, **refuse encryption features entirely**. Do **not** fall back to XOR, reversible encoding, password-only “gates,” or pure JS crypto libraries pulled from the network.

Chrome was not separately re-probed in this session; Edge Chromium results are representative for modern Chromium. Implementation S1 should re-run the same probe on the user’s normal Edge **and** Chrome profiles (still disposable data).

---

## 5. Recommended cryptographic primitives and parameters policy

| Role | Recommendation |
|------|----------------|
| AEAD | **AES-256-GCM** via Web Crypto |
| KDF | **PBKDF2-HMAC-SHA-256** (native, widely available) |
| Salt | 16+ bytes CSPRNG, **unique per envelope / vault record** |
| IV / nonce | 12 bytes CSPRNG, **unique per encryption** (never reuse with same key) |
| Key length | 256-bit AES key |
| Passphrase storage | **Never** stored (localStorage, sessionStorage, backups, code, Projects, Library) |
| Weak algorithms | Forbidden |

### 5.1 Iteration cost policy (do not freeze a magic number from memory alone)

Implementation **S1** must:

1. Ship a **capability + cost probe** on first encrypted-export / vault-enable (disposable timer).
2. Choose iterations so unlock/export KDF costs roughly **100–500 ms** on the workshop machine, with:
   - **Floor:** ≥ **210 000** iterations (never below industry-common floors without explicit review).
   - **Target:** prefer **≥ 600 000** when the machine is under ~300 ms (this probe: 600k ≈ 93 ms → **acceptable**; may increase further if still &lt;100 ms).
   - **Ceiling:** cap so a single unlock stays under ~2 s on a slow netbook; if the floor already exceeds usability, warn and still use the floor.
3. **Persist the chosen iteration count inside the ciphertext envelope** so other machines can decrypt without re-probing.
4. Document measured probe results in the implementation report.

### 5.2 Determinism

- Ciphertext **must differ** every encryption (random salt and/or IV).
- Authenticated metadata (app id, format version, schema version of **inner** plaintext after decrypt) should bind via AES-GCM **additionalData** or an AEAD-associated structured header that is covered by authentication.

### 5.3 Algorithm agility

- Envelope field `crypto.kdf` / `crypto.cipher` string constants (e.g. `PBKDF2-SHA256`, `AES-256-GCM`).
- Unknown algorithm → hard refuse with user message (no partial open).
- Future Argon2 only if/when widely available in Web Crypto — not required for S1–S3.

### 5.4 Homemade crypto

Forbidden: XOR, Base64-as-encryption, password hash equality as “encryption,” single global IV, encrypting without authentication, client-only “password screen” over plaintext `localStorage`.

---

## 6. Exact encrypted-backup envelope contract

### 6.1 Detection and format constants

| Item | Recommendation |
|------|----------------|
| New constant | `ENCRYPTED_BACKUP_FORMAT = 'genmitsu-l8-tracker-encrypted-backup-v1'` |
| Existing `BACKUP_FORMAT` | **Unchanged** — continues to mean **plaintext** JSON backup |
| Outer file | JSON object (offline-safe, readable metadata, no binary blob requirement) |
| Filename | `l8-tracker-backup-encrypted-YYYY-MM-DD.json` (or `.enc.json`) |
| Magic discrimination | `backupFormat === ENCRYPTED_BACKUP_FORMAT` **or** `encryption.envelopeVersion === 1` |

Do **not** reuse `BACKUP_FORMAT` for ciphertext. Import path must branch **before** treating the root as a workspace document.

### 6.2 Outer envelope fields (non-secret)

```text
{
  "app": "genmitsu-l8-tracker",
  "appVersion": "<originating APP_VERSION>",
  "backupFormat": "genmitsu-l8-tracker-encrypted-backup-v1",
  "envelopeVersion": 1,
  "createdAt": "<ISO-8601>",
  "crypto": {
    "kdf": "PBKDF2-SHA256",
    "kdfIterations": <int>,
    "cipher": "AES-256-GCM",
    "saltB64": "<base64>",
    "ivB64": "<base64>"
  },
  "ciphertextB64": "<base64 AES-GCM output including tag>",
  "innerSchemaVersion": <int declared, also authenticated>,
  "plaintextBackupFormat": "genmitsu-l8-tracker-backup-v1"
}
```

**AAD (recommended):** UTF-8 string of fixed fields, e.g.  
`app|encrypted-backup-v1|innerSchemaVersion|plaintextBackupFormat`  
so renaming outer fields without recomputing AAD fails authentication.

### 6.3 Inner plaintext (after decrypt)

Exactly one UTF-8 JSON object that is a **valid existing plaintext backup** as produced by today’s `backupObject()` (same `BACKUP_FORMAT`, `schemaVersion`, collections). That object then passes:

1. `JSON.parse`
2. Existing `validateBackupImport`
3. Existing Merge/Replace pipeline unchanged

### 6.4 Failure behavior

| Condition | Behavior |
|-----------|----------|
| Wrong passphrase | Clear error; **no** state mutation |
| Corrupt / truncated ciphertext | Same |
| Future `envelopeVersion` | Refuse |
| Foreign `app` | Refuse |
| Unknown `crypto.*` | Refuse |
| Cancel password UI | No import |
| Rate-limit retries | Optional UX delay; not a security substitute |

### 6.5 Export UX choices

| Option | Role |
|--------|------|
| **Password-encrypted backup** | Recommended / primary control |
| **Plain JSON backup (readable)** | Explicit compatibility/recovery; secondary control with **warning**: contains readable Tracker records (pricing, projects, inventory, evidence, etc.) |

Do **not** remember export passphrases across sessions. Optional in-session memory only if both: (a) never written to storage, (b) cleared on lock/unload — still **prefer none** for S2.

---

## 7. Exact local-vault storage contract

### 7.1 Model

Optional **whole-workspace AEAD blob** for the same JSON document `persist()` writes today (or an equivalent canonical serialization of that document).

| Key | Contents |
|-----|----------|
| `STORAGE_KEY` (`genmitsu-l8-tracker-v1`) | **Either** legacy plaintext workspace **or** retired after migration (see §8) |
| `genmitsu-l8-tracker-vault-v1` (new) | Encrypted envelope for live data |
| `genmitsu-l8-tracker-vault-meta-v1` (new) | Non-secret bootstrap: vault enabled, envelope version, crypto params, createdAt, **no** record names/counts/secrets |

**Prefer:** after successful migration, plaintext `STORAGE_KEY` is removed **only** after verified unlock-reload and explicit user confirmation; until then keep recoverable plaintext copy under an explicit name such as `genmitsu-l8-tracker-v1-pre-vault-recovery` with UI that says it still exposes data.

### 7.2 Locked startup

| Visible while locked | Hidden |
|----------------------|--------|
| App name/version, unlock form, Help/About offline privacy blurb, Export disabled or limited to “unlock first”, Import of **encrypted** backup with password | All Log/Library/Projects/Inventory/Pricing/grids/evidence lists, machine names, record counts, recent project titles |

**Do not** show “12 projects, $X profit” while locked.

### 7.3 Unlocked session

- Passphrase-derived key held **only in memory** (JS variable / CryptoKey non-extractable preferred).
- Decrypted workspace in memory as today’s `state`.
- `persist()` encrypts full payload → writes vault key → does not write plaintext `STORAGE_KEY` when vault mode is on.
- Refresh / new tab: **always locked** (no sessionStorage passphrase).
- Manual **Lock** button clears key + clears in-memory state to empty/locked shell.
- Optional idle lock: **defer to S4**; if added, default off.

### 7.4 Disable vault

1. Unlock.
2. Export recommended (encrypted + optional plain with warning).
3. Write plaintext `STORAGE_KEY` from current state.
4. Verify reload as plaintext.
5. Remove vault keys.
6. Never leave orphan ciphertext as the only copy without user confirmation.

---

## 8. Safe opt-in migration and rollback

**Never** encrypt existing data on first run or silently.

### 8.1 Sequence (S3)

1. User chooses **Enable encrypted local vault**.
2. Strong prompt: export a **plain recovery backup** and/or encrypted backup first; checkbox “I have a recovery file.”
3. Passphrase + confirm; strength meter (length/entropy heuristics only — not a crack guarantee).
4. Explain: no reset, no backdoor, forgotten password loses vault access without a backup.
5. Read current plaintext document (from memory/`STORAGE_KEY`).
6. Encrypt → write **new** vault key only.
7. **Read back** vault → authenticate → decrypt → parse → normalize → structural compare to source (field-level or canonical JSON hash of normalized backupObject-equivalent).
8. Prompt user to **reload** and unlock once (proves path).
9. Only after unlock success: offer **Retire plaintext copy** with second confirm; until then keep `…-pre-vault-recovery` readable and labeled unsafe.
10. On any failure: leave original `STORAGE_KEY` intact; delete incomplete vault artifacts; show error.

### 8.2 Interaction with schema migration

- Decrypt **first**, then run existing `loadState` / normalizers / machine migration.
- Vault meta version independent of `SCHEMA_VERSION`.
- Inner document still carries `schemaVersion` for app data.

### 8.3 Power loss

- Incomplete vault write must not delete plaintext.
- Prefer write vault temp key → verify → rename/swap (localStorage has no atomic multi-key transaction — mitigate with dual keys and “activePointer” meta field).

---

## 9. Password and recovery UX

| Topic | Guidance |
|-------|----------|
| Create / confirm | Two fields; show/hide toggle |
| Not saved | Explicit sentence near unlock |
| No backdoor | Explicit; no security questions |
| Wrong password | “Could not open. Check the passphrase.” |
| Corrupt vault/backup | “Data unreadable or damaged.” + recovery download if raw blob retained |
| Forgotten passphrase | Only path: prior backup; no reset |
| Plain export | Red warning: readable data |
| Retire plaintext | “After this, only the vault + your backups hold data.” |

**Printable recovery key:** **Defer past S3.** Adding one without a careful dual-secret design encourages false confidence and storage mistakes. Prefer second encrypted backup on USB.

---

## 10. Import, Merge, and Replace behavior

| Scenario | Behavior |
|----------|----------|
| Encrypted backup, vault off | Prompt passphrase → decrypt → validate plaintext backup → Merge/Replace UI unchanged |
| Encrypted backup, vault locked | Unlock first **or** allow import-only unlock path that does not leave workspace unlocked without user intent; simplest S2: require unlock if vault enabled |
| Encrypted backup, vault unlocked | Decrypt → validate → Merge into memory → encrypt on persist |
| Plaintext backup → vault mode | Allowed after unlock; becomes vault content on save |
| Encrypted backup → plaintext mode | Decrypt to temp → validate → apply → store plaintext |
| Cancel password | No-op |
| Wrong password N times | Stay closed; optional 1s delay |
| Future envelope / foreign app | Refuse before Merge/Replace |
| Existing Replace confirms | Preserve double-confirm |

Encryption is only the **boundary**; business validation remains authoritative.

---

## 11. Storage and backward-compatibility assessment

| Item | Recommendation |
|------|----------------|
| `SCHEMA_VERSION` | **No change** for S1–S2; S3 only if meta requires it (prefer **not**) |
| `APP_ID` / `APP_NAME` | Unchanged |
| `STORAGE_KEY` | Unchanged string; semantics may become “legacy/plaintext or retired” |
| `BACKUP_FORMAT` | Unchanged (plaintext) |
| New encrypted backup format constant | Yes |
| Project/Library schemas | Unchanged |
| Machine/evidence identities | Unchanged |
| Older app opens encrypted backup | Cannot; message to use newer Tracker |
| Newer envelope on older app | Refuse |
| Designs session data | Remains out of backups/vault unless product later expands scope (out of this phase) |

---

## 12. Recommended phased implementation

### S1 — Crypto capability + envelope freeze (no shipping vault)

| | |
|--|--|
| **Scope** | Capability probe helper; document envelope; optional hidden selftest for crypto round-trip on disposable data; **no** user-facing export change required to complete S1, or a behind-flag prototype only |
| **Files likely** | `index.html` (small helpers + fixture), README note deferred |
| **Keys** | None required |
| **Fixtures** | Crypto availability; encrypt/decrypt disposable object; wrong password; tamper |
| **Browser** | Edge + Chrome file:// probe on workshop machines |
| **Audit** | Light review of probe only |
| **Stop** | `subtle` unavailable; cannot meet iteration floor |

### S2 — Encrypted backup export/import

| | |
|--|--|
| **Scope** | Encrypted export UI; import branch; plain export warning; fixtures for encrypted import Merge/Replace |
| **Protected** | All geometry/evidence/calculators; plaintext `BACKUP_FORMAT` path remains |
| **Audit** | **Independent review recommended before commit** |
| **Stop** | Any path that writes ciphertext then deletes plaintext without verification; silent format confusion |

### S3 — Optional local vault + migration

| | |
|--|--|
| **Scope** | Vault keys, locked shell, unlock, persist encrypt, migration sequence, disable vault |
| **Audit** | **Mandatory independent adversarial review before commit** |
| **Stop** | Decrypt failure treated as empty; start-fresh without warning; passphrase persistence |

### S4 — Lock polish, idle optional, docs, adversarial recovery drills

| | |
|--|--|
| **Scope** | Idle lock default-off; Help/About; recovery drills on disposable profiles |
| **Stop** | Scope creep into 1.0 certification claims |

**Why this order:** S2 delivers the highest practical win (backup files in email/USB) without touching live storage. S3 is higher risk for data loss and needs the S2 pipeline mature.

---

## 13. Files/functions likely to change (future implementation)

| Area | Symbols / UI |
|------|----------------|
| Constants | New `ENCRYPTED_BACKUP_FORMAT`, vault key names |
| Crypto | New pure helpers: `deriveBackupKey`, `encryptJsonBlob`, `decryptJsonBlob`, `probeWebCrypto` |
| Export | `exportBtn` flow → dual action or modal |
| Import | `importFile.onchange` → envelope detect → password modal → existing validation |
| Persist/load | S3: branch in `persist` / `loadState` |
| Recovery | Extend recovery panel for vault auth failure |
| Help | Encryption / vault / no-recovery wording |
| Fixtures | New security groups under selftest |

**Do not modify:** Designs builders, evidence matchers, promotion transactions, production SVG serializers, pricing/inventory calculators (except incidental if they call `persist` — keep call sites stable).

---

## 14. Protected boundaries (future work)

Unless a blocking defect is found (none blocking S1 found):

- Production SVG geometry/bytes  
- Designs template identities  
- Evidence meaning/matchers/promotion  
- Project accounting / Pricing / Inventory semantics  
- Machine-record identity  
- Project/Library schemas  
- Plaintext `BACKUP_FORMAT` meaning  
- `APP_ID` / `APP_NAME`  
- Offline `file://` operation  
- Existing data without explicit user action  

**Hard safety rules:** never clear localStorage on decrypt failure; never treat unreadable vault as empty success; never “start fresh” without destructive confirmation and recovery alternatives.

---

## 15. Required fixtures (by phase)

### S1

- SubtleCrypto present/absent branches  
- Round-trip disposable JSON  
- Wrong password / tamper → throw  
- Non-deterministic ciphertext  
- Envelope field validation unit tests (pure)

### S2

- Export encrypted file structure  
- Import encrypted → Merge/Replace parity with plaintext fixtures  
- Plain export warning path  
- Future envelope / foreign app refuse  
- Cancel password no mutation  
- Storage recovery still works with plaintext damaged data  

### S3

- Migration success / mid-failure rollback  
- Locked UI hides records  
- Unlock → edit → lock → unlock data preserved  
- Disable vault restores plaintext  
- Passphrase not present in localStorage dumps (fixture scans keys/values)  

---

## 16. Required browser and failure-injection tests

- Disposable profile only  
- file:// open  
- Encrypt 1 MB synthetic backup (photos optional later)  
- Kill browser mid-migration (S3)  
- Truncate ciphertext  
- Wrong AAD / renamed format  
- Import encrypted on build without crypto → refuse  
- Confirm plain JSON still imports on older mental model  

---

## 17. Fresh baseline totals

| Metric | Measured |
|--------|----------|
| Unique registered suite | **3003 passed / 0 failed / 46 groups** |
| HTML parser | exit 0 |
| Storage recovery fixtures | Included in suite (15 asserts in storage group) |
| Merge/Replace storage fixtures | Included |
| APP_VERSION | 0.9.0 pre-1.0 |

---

## 18. Findings by severity

### Critical

None that block architecture or S1. Existing plaintext storage is an accepted **product risk**, not an implementation bug relative to today’s design.

### High

1. **Plaintext Export is the only backup path and is easy to share.**  
   - **Location:** `exportBtn` → `backupObject()` download.  
   - **Concern:** Pricing, projects, inventory, evidence leave the machine readable.  
   - **Impact:** Primary real-world exposure.  
   - **Blocks:** Not S1; **drives S2 priority**.  
   - **Correction:** Encrypted default export + explicit plain option with warning.

2. **Plaintext localStorage under profile access.**  
   - **Location:** `persist` / `STORAGE_KEY`.  
   - **Concern:** Shared PC / profile copy.  
   - **Blocks:** Not S1; **S3**.  
   - **Correction:** Optional vault; never fake lock.

### Medium

3. **Import accepts any JSON object without app markers (legacy).**  
   - **Location:** `validateBackupImport` — missing `app`/`backupFormat` still accepted.  
   - **Concern:** Low for security; needed for old backups; encrypted path must not inherit ambiguity.  
   - **Blocks:** No — encrypted envelopes must require explicit format fields.  
   - **Correction:** Encrypted branch requires full envelope; keep legacy for plaintext only.

4. **No user-facing distinction that plain backup is “readable sensitive data.”**  
   - **Location:** Export button titles/Help.  
   - **Blocks:** S2 UX.  
   - **Correction:** Copy changes in S2.

### Low

5. **Project photos inflate backups** — existing confirm helps; encryption will make files larger/slower; document.  
6. **Designs session data not in backups** — intentional; vault should match (session stays session).

### Informational

7. Corrupt storage recovery model is **well designed** and should be the template for vault auth failure.  
8. Copying `index.html` alone does not copy secrets — already true.  
9. Web Crypto works on `file://` in measured Edge — platform limitation **not** present for S1.

---

## 19. Unverified areas

- Separate Chrome headed probe on Joe’s machine class (Edge Chromium measured).  
- Performance of encrypting multi-megabyte photo-heavy backups on low-end hardware.  
- IndexedDB alternatives (not required; localStorage remains product baseline).  
- Formal side-channel analysis of PBKDF2 timing (out of scope for workshop app).  
- Firefox/Safari if ever supported (currently Edge-centric offline use).

---

## 20. Exact recommended S1 implementation phase

**S1 — Native cryptography capability probe and encrypted-backup envelope freeze**

**Authorized to ship:**

1. Pure helpers (or a dedicated selftest-only module section) that:
   - Detect `crypto.subtle` and required methods.
   - Run disposable encrypt/decrypt selftests.
   - Measure PBKDF2 cost and select iterations under the policy in §5.1.
2. Documented **frozen envelope schema** matching §6 (constants may be added but unused by Export UI until S2).
3. Fixture group `?selftest=crypto` (or under storage) that **must not** touch real user keys or persist secrets.
4. Implementation report stating measured Edge file:// results and iteration choice method.

**Forbidden in S1:**

- Changing default Export behavior without completing S2 review  
- Writing vault keys  
- Migrating or deleting `STORAGE_KEY`  
- Storing passphrases  
- Claiming “secure app” / certification  

**Exit criteria:** Crypto fixtures green; probe refuses when SubtleCrypto missing; envelope fields reviewed; no change to production suite totals beyond additive crypto fixtures.

---

## Hygiene

- No edits to `index.html`, README, CHANGELOG, fixtures, or other product files.  
- No stage/commit/push/reset/clean/stash.  
- No access to Joe’s real browser profile or records.  
- Disposable crypto and suite profiles only.  
- Unrelated untracked files preserved.

---

*End of architecture review.*
