# Security S3-2 — Encrypted Local Vault Enrollment and Enablement  
# Architecture Review (Read-Only) — Corrected

Date: 2026-07-23 (fourth / final narrow correction same day)  
Repository: `C:\Genmitsu L8 Tracker`  
Role: architecture and implementation planning only — **no S3-2 implementation**  
Authorized output: this report only

---

## Correction log

### First correction

1. Foundational S1–S3 re-applied (deferred plaintext retirement; locked restore required).  
2. Immediate plaintext deletion after vault commit rejected.  
3. Caller-only recovery confirm boolean rejected.  
4. Minimal locked-shell encrypted-backup restore in S3-2 scope.

### Second correction

5. Restore must write+verify plaintext **before** removing vault keys.  
6. Commit requires passphrase / staged key material.  
7. Recovery-export approval bound to workspace snapshot; invalidate on change.

### Third correction (this revision) — required before Codex

An external review of the second correction correctly identified one remaining **security race** and one export-binding tightening:

8. **Restore race against a newer vault** — between backup validation and vault deletion, another **unlocked** tab can persist a **newer** vault generation. Deleting “the vault” without re-binding would destroy that newer authoritative ciphertext.  
   **Mandatory:**  
   - Capture **exact starting vault binding** at restore start: full `VAULT_STORAGE_KEY` raw string **and** structural `writeGeneration` (and preferably envelope identity sufficient to detect any change).  
   - After plaintext is written and verified, **immediately re-read** `VAULT_STORAGE_KEY` and require **byte-identical raw** (or at minimum same generation **and** same ciphertext bytes) as the starting binding.  
   - If the vault changed: **abort; do not delete any vault material;** leave the **current** vault authoritative; show stale/reload error.  
   - Deletion order when binding still matches: remove **metadata and pending first**, then **`VAULT_STORAGE_KEY` last**. **Primary-vault key deletion is the commit point** of restore (plaintext already verified).  
   - After successful restore (vault key gone), an older unlocked tab’s vault `persist()` must **not** recreate vault authority over the restored plaintext: coordinator must refuse encrypt-write when this tab’s session generation is stale **or** when re-read shows vault missing / mode no longer unlocked with matching generation; prefer **fail closed** → blocked/reload (“workspace restored or changed in another tab”). Existing generation stale-writer checks are necessary but not sufficient alone if a tab still holds an old session key—implement **re-check of on-disk vault raw/generation immediately before every vault setItem**, and treat **missing vault key while tab thought it was unlocked** as hard failure (clear session, blocked/reload, **do not** create a new vault from stale memory).

9. **Recovery export must freeze one immutable snapshot before any async work** — capture `approvedLiveDocument` / `approvedStorageRaw` (and the exact object used to build `backupObject()` content) **synchronously before the first `await`**. Both the **encrypted recovery backup bytes** and the **approval binding** must be generated from that **same** snapshot. No re-reading live state or `STORAGE_KEY` between snapshot capture and backup encrypt completion for approval purposes.

### Fourth correction (this revision) — required before Codex

An external review of the third correction correctly identified one remaining **multi-tab data-loss** case:

10. **Dormant ordinary plaintext tab** — Tab A loads plaintext and never writes. Tab B enrolls, later restores a **different** backup (vault removed; `STORAGE_KEY` is the restored workspace). Tab A still has no vault key, so the existing vault-presence guard does **not** fire; Tab A’s ordinary `persist()` can overwrite the restored `STORAGE_KEY` with its **stale** in-memory workspace.  
    **Mandatory:** ordinary plaintext `persist()` must track the **`STORAGE_KEY` raw value this tab last loaded or successfully wrote** (`plaintextStorageBaselineRaw`, set at `loadState()` / successful plaintext `setItem` / post-restore install). **Immediately before** each plaintext `setItem(STORAGE_KEY, …)`, re-read current `STORAGE_KEY` and require **byte-identical** equality to that baseline (or both null if this tab legitimately started empty and never saw a key). If current storage differs unexpectedly: **fail closed** — do **not** write; surface reload-required messaging; leave restored (or other-tab) data intact. After a successful write by this tab, update the baseline to the bytes just written.  
    This is independent of vault-session generation checks (those protect **unlocked vault** tabs only).

**Do not implement earlier drafts that omit vault revalidation before deletion, delete `VAULT_STORAGE_KEY` before meta/pending, approve recovery export from a moving workspace, or allow ordinary plaintext `persist()` without a loaded-storage baseline compare.**

---

## Verdict

**1. S3-2 ARCHITECTURE READY — Proceed to bounded Codex implementation** (using **this fourth-corrected** design only).

**Joe decision required:** none for safety. The product decision is settled here:

| Question | Settlement |
| --- | --- |
| Ship enablement before full recovery phase? | **Only if S3-2 includes the minimal locked-shell restore path below.** Full recovery polish remains later; bare “unlock only” is **not** enough. |
| Combine/reorder with a separate recovery-only phase first? | **Rejected** as the sole answer — pre-enrollment users have nothing locked to restore. Recovery-before-enable does not fix post-enable lockout. |
| Include minimal locked restore in S3-2? | **Yes — required.** |

---

## 1. Repository and source baseline

| Check | Result |
| --- | --- |
| HEAD | `3aa314a352f7d08c94d26114de7cfdc05dc9d9e9` — **Add encrypted local vault foundation** |
| Branch | `main` |
| Upstream | `main...origin/main` **0 / 0** |
| Tracked worktree | **Clean** (nothing staged; no tracked diffs) |
| Untracked | Historical docs / LightBurn / `debug.log` / utility — preserved |
| Full suite re-run | **Not executed** in this architecture pass |

---

## 2. Documents fully applied (re-read)

| Document | Role in this correction |
| --- | --- |
| `SECURITY_ENCRYPTED_BACKUPS_LOCAL_VAULT_ARCHITECTURE_REVIEW_2026-07-22.md` | §7–§9 migration: export first; **reload+unlock before retire plaintext**; locked shell hides records; forgotten passphrase → **prior backup only**; import when vault locked must not leave opaque dead-end |
| `SECURITY_S3_ENCRYPTED_LOCAL_VAULT_ARCHITECTURE_REVIEW_2026-07-23.md` | §5.4 authority; §6 locked shell allows **import encrypted backup → recovery**; §7 enrollment steps 9–10 **deferred plaintext removal** after cold unlock; §10 restore = erase vault + import backup |
| S3-1 implementation + verification + corrections + focused verification | Live bootstrap, persist guard, unlock shell, coordinator, fixtures, pending key reserved but unused |
| Current `index.html` | Confirms: locked shell is unlock-only today; **no** restore control; import gated by `vaultOrdinaryMode()`; S2 decrypt/import exists only when ordinary mode |

**Current production gap (must close in S3-2 if enrollment ships):**

- After a vault exists, reload shows unlock form only.  
- `importBtn` / file import **no-op while locked**.  
- Blocked shell text defers “Recovery and restore” to a later phase.  
- Therefore a recovery file created at enroll is **not usable from the locked UI** until restore exists.

---

## 3. Corrected S3-2 scope

### In scope (required)

**A. Enrollment / enablement**

1. Enable action (eligible plaintext only).  
2. Wizard: explain → capability → passphrase+confirm → **mandatory encrypted recovery backup** → final confirm → vault transition.  
3. Encrypt **`liveStateDocument()`** with S3-1 vault crypto.  
4. Pending → verify → vault → meta `mode:vault` using existing keys.  
5. **Retain `STORAGE_KEY` as a labeled leftover plaintext hazard until retire step.**  
6. In-session continue **unlocked** after vault commit (optional but allowed).  
7. **Cold reload + successful unlock proof** before any “retire plaintext” offer.  
8. **Second explicit confirm** to remove leftover plaintext.  
9. Multi-tab preflight / vault-appear abort.  
10. Help copy: vault vs backup; no-reset; leftover plaintext warning.  
11. Fixtures expanded on existing S3 group.

**B. Minimal locked-shell encrypted-backup restore (required for enablement safety)**

12. On **`vault-locked`** (and preferably **`vault-blocked`** when ciphertext is present but unreadable), show:  
    **“Can’t unlock? Restore from an encrypted backup…”**  
13. Flow (bounded) — **authority-safe order is mandatory**:  
    - **At restore start (sync, before any async decrypt if possible, and before any deletion):** capture  
      `startingVaultRaw = localStorage.getItem(VAULT_STORAGE_KEY)` (must be non-null for the normal locked path),  
      parse + `validateVaultEnvelopeV1` → `startingVaultGeneration = envelope.writeGeneration`  
      (binding = **exact raw string**; generation is a secondary check and fixture aid).  
    - Pick file → parse → `isEncryptedBackupEnvelope` / `validateEncryptedBackupEnvelopeV1` **before** PBKDF2.  
    - Passphrase for **the backup file** (may equal vault passphrase; treat as independent input).  
    - Decrypt + `validateBackupImport` **in memory only** — no storage mutation yet.  
    - Destructive confirm: **“Restore this backup into ordinary browser storage, then remove the local vault if it is still the same vault that was present when restore started.”** (Confirm only authorizes the transition; it does **not** reorder steps.)  
    - **Write restored plaintext to `STORAGE_KEY` while `VAULT_STORAGE_KEY` remains in place** (vault still authoritative for startup if interrupted). Prefer writing the normalized live document bytes that `persist()` would write (from `liveStateFromData(backup)` then the same field set as plaintext `persist()`), not a partial object.  
    - **Read back** `STORAGE_KEY`, decode, normalize, and verify **normalized equality** to the intended restored document.  
    - **Immediate pre-delete revalidation (mandatory):** re-read `localStorage.getItem(VAULT_STORAGE_KEY)`.  
      - If `null`, or **not byte-identical** to `startingVaultRaw`, or parsed generation ≠ `startingVaultGeneration` when raw was valid: **abort restore cleanup; do not delete vault, pending, or meta;** report “The local vault changed in another tab. Reload this page.”  
      - Vault remains authoritative; restored plaintext on `STORAGE_KEY` may exist as a **non-authoritative leftover** (same class as interrupted dual-copy) — do **not** auto-delete it; user reloads and either unlocks the newer vault or retries restore.  
    - **Only when vault binding still matches — deletion order:**  
      1. Remove `VAULT_META_KEY` (and any migrating fields).  
      2. Remove `VAULT_PENDING_KEY` if present.  
      3. Remove **`VAULT_STORAGE_KEY` last** — **this is the restore commit point.**  
    - Then set `vaultMode = 'plaintext'`, clear `vaultSession`, install in-memory state from the restored document, render full app.  
    - **If any step fails** (quota, setItem throw, read-back mismatch, decode failure, vault revalidation fail): **do not remove `VAULT_STORAGE_KEY`**; leave vault authoritative; show error; locked/blocked UI remains.  
    - Never treat restore as “password reset.” Never open the old vault with the backup passphrase unless decrypt succeeds on the **file**.  
14. While locked: do **not** enable ordinary Merge UI over live vault without this path; restore is **verified plaintext write first, vault revalidation, then ordered vault retirement**, landing **plaintext**.  
15. **Post-restore stale unlocked vault tab:** any tab still holding `vault-unlocked` + old session must, on next `queueVaultPersist` / `drainVaultPersist`, re-read on-disk vault **before** `setItem`. If vault key is **missing**, or raw/generation does not match `vaultSession.knownGeneration` expectations for a safe update (generation advanced by another tab, or key gone after restore): **do not write vault**; clear session; enter blocked/reload messaging. This prevents recreating a vault over restored plaintext.  
16. **Post-restore / post-foreign-write stale ordinary plaintext tab (mandatory):** tabs in `vaultMode === 'plaintext'` that never saw a vault key must still not clobber storage another tab rewrote. Maintain `plaintextStorageBaselineRaw` (exact `STORAGE_KEY` string last known good for this tab). On every ordinary plaintext `persist()` path, **immediately before** `localStorage.setItem(STORAGE_KEY, …)`: if `localStorage.getItem(STORAGE_KEY) !== plaintextStorageBaselineRaw`, **return false without writing**, set status/toast or blocked notice to reload, do not clear the foreign `STORAGE_KEY`. Initialize baseline at `loadState()` success (the raw string just read), on successful plaintext write (update to the new raw), and when restore installs plaintext in this tab (set baseline to the restored raw). Empty-start tabs: baseline `null` until first successful write; if storage becomes non-null from another tab before first write, fail closed rather than overwrite.  
17. Optional narrow: offer download of raw vault blob as diagnostic **only** on blocked/unreadable path — **defer** if it expands scope; not required for S3-2 acceptance if erase+restore works.

### Explicit non-goals (still deferred)

- Full recovery center polish, Start Fresh vault-specific UX beyond existing storage recovery  
- Plaintext-backup restore-from-locked as the **only** recovery path (encrypted is mandatory for the enroll-required file; plain restore may be added later)  
- Opaque “download vault file” as a general feature  
- Disable Vault, manual Lock UI, idle lock, passphrase rotation  
- S1/S2/S3-1 format changes  

### Why not “recovery phase first, then enable”?

A recovery-only phase **before** any vault exists cannot exercise locked restore. Enablement that deletes plaintext without locked restore creates a **dead-end**. Therefore minimal restore is **part of S3-2**, not a predecessor phase.

---

## 4. Corrected enrollment state machine and authority

### Authority after vault commit (foundational contract)

| Stage | Authoritative | `STORAGE_KEY` | User path |
| --- | --- | --- | --- |
| Plaintext / wizard / recovery-export only | Plaintext | Present | Cancel-safe |
| Pending written / verify fail | Plaintext | Present | Drop pending |
| **Vault + meta committed** | **Vault** | **Still present (leftover hazard)** | Reload → locked; unlock; warn about leftover plaintext |
| After cold unlock success | Vault | Leftover until retire | Offer **Remove readable browser copy** |
| After retire confirm | Vault only | Removed | Normal locked/unlocked |
| Locked + forgotten passphrase | Vault (inaccessible) | None if retired | **Restore from encrypted backup** (S3-2 min path) |
| Restore: backup validated in memory only | **Vault** (still) | Unchanged | No mutation yet; starting vault raw/generation captured |
| Restore: plaintext written + verified, vault not yet removed | **Vault still wins on cold start** if process dies | New plaintext present **and** vault present | Pre-delete revalidation required |
| Restore: vault raw changed before delete | **Newer vault** (other tab) | Leftover non-authoritative plaintext may exist | **Abort delete**; keep vault |
| Restore success (`VAULT_STORAGE_KEY` removed last) | **Plaintext** | Restored from backup | App usable; re-enroll optional later |
| Stale unlocked vault tab after restore | Plaintext on disk | Present | Must **not** recreate vault; fail closed to reload |
| Stale ordinary plaintext tab after restore | Restored plaintext on disk | Present (restored) | Must **not** overwrite with stale memory; baseline compare fails closed |

### Corrected durable transition sequence

Use existing keys only.

**Phase E0 — Wizard (no vault writes)**  
- Capability probe.  
- Passphrase + confirm (exact; no trim).  
- **Recovery export (coordinator-internal) — freeze-before-async is mandatory:**  
  1. **Synchronously, before any `await`:** capture one immutable snapshot pack:  
     - `approvedStorageRaw = localStorage.getItem(STORAGE_KEY)`  
     - `approvedLiveDocument =` normalized clone of `liveStateDocument()`  
     - `approvedBackupObject =` structured clone of `backupObject()` built **immediately from the same in-memory state** that produced `approvedLiveDocument` (do not call `backupObject()` again after an await).  
  2. **Only then** run async encrypt of **`approvedBackupObject`** (not a live re-read) and download.  
  3. On encrypt+download-initiate success: set `recoveryExportOk = true`, retain the snapshot pack + `recoveryExportedAt`.  
  4. On failure: `recoveryExportOk = false`; clear snapshot pack.  
- **Invalidate approval** if, before commit, either:  
  - `localStorage.getItem(STORAGE_KEY) !== approvedStorageRaw`, or  
  - normalized `liveStateDocument()` is not equal to `approvedLiveDocument` (another tab or local edit).  
  When invalidated: set `recoveryExportOk = false`, clear snapshots, force user to re-export recovery backup.  
- User checkbox “I saved the recovery file…” only **enables** Next if `recoveryExportOk` still true; checkbox alone never sets export-ok.  
- Final confirm: create local vault; **readable browser copy remains until after reload and unlock.**

**Phase E1 — Encrypt & commit (still do not delete plaintext)**  

1. Preflight: plaintext mode, no vault key, capability OK; **`recoveryExportOk` still true**; re-check snapshot binding (storage raw + live document) — if stale, abort with “workspace changed; create a new recovery backup.”  
2. Enrollment token.  
3. Cryptographic material for this call (see §7): either  
   - **Option A (preferred):** wizard still holds passphrase only until commit; `commitEncryptedLocalVaultEnrollment(enrollmentSession, passphrase, seams)` derives salt+key inside the call, or  
   - **Option B:** wizard already derived into a non-extractable `CryptoKey` held on `enrollmentSession.pendingKey` + `pendingSaltB64` + `pendingCreatedAt` after passphrase accept; commit consumes and clears those fields.  
   Empty passphrase / missing key material → refuse without writes.  
4. Encrypt **the approved snapshot document** (or a fresh document only if it still equals the approved snapshot — never enroll a different workspace under an old recovery approval).  
5. Write `VAULT_PENDING_KEY`.  
6. Meta `mode:'migrating'`, `pendingTransition:{kind:'enroll', phase:'pending-written'}` (best-effort).  
7. Read-back verify (structural + decrypt + normalize compare to approved document). On fail: delete pending; stay plaintext.  
8. Promote pending → `VAULT_STORAGE_KEY`; delete pending.  
9. Meta `mode:'vault'`, `lastWriteGeneration:1`.  
10. **Do not remove `STORAGE_KEY`.**  
11. Activate `vault-unlocked` in this tab with session; clear enrollment passphrase/key staging fields; toast: vault on; **reload recommended**; leftover plaintext still readable until removed.  
12. Infer leftover retirement from vault + `STORAGE_KEY` coexistence (no trust in meta alone for security).

**Phase E2 — Cold proof (required before retire)**  

13. User reloads (or new tab) → `vault-locked` shell, **no record flash**.  
14. Unlock with passphrase → workspace matches.  
15. Only then show **Remove readable browser copy** (second confirm).  

**Phase E3 — Retire leftover plaintext**  

16. `removeItem(STORAGE_KEY)` after confirm.  
17. If remove fails: vault remains authoritative; retry UI after unlock.

**Crash table (aligned with S3 §7):**  
- Before E1: plaintext only.  
- After pending, before promote: plaintext authoritative; discard pending.  
- After vault commit, before retire: vault authoritative; leftover plaintext is labeled hazard, **not** deleted automatically.  
- After retire: vault only; recovery = unlock or **restore from encrypted backup**.

---

## 5. Recovery-backup contract (corrected)

| Topic | Settlement |
| --- | --- |
| When | Before any vault write |
| Format | S2 encrypted envelope via production helpers |
| Content | `backupObject()` at the approval moment |
| Passphrase | Same string as vault passphrase for this flow; UI states dual use; restore UI still asks for the **file** passphrase |
| App-known success | Download path invoked **and** encrypt returned `ok` → internal `recoveryExportOk` **plus** bound snapshots |
| Snapshot binding | Freeze `approvedStorageRaw`, `approvedLiveDocument`, and `approvedBackupObject` **before first await**; backup encrypt uses only `approvedBackupObject`. Commit refuses if live storage/doc diverges. |
| User checkbox | Confirms custody only; **cannot** set export-ok by itself |
| Coordinator API | Must **not** accept an external boolean as sole proof of export |
| Restore usability | **Mandatory:** locked-shell path in §3.B must open this file class |
| Restore authority order | Plaintext write+verify → revalidate starting vault binding → meta/pending off → **`VAULT_STORAGE_KEY` last** |
| Restore multi-tab | Abort vault deletion if on-disk vault raw ≠ starting binding |
| Filename | Prefer `l8-tracker-recovery-encrypted-YYYY-MM-DD.json` or label UI “recovery backup” on existing encrypted name |

Honest copy: the Tracker can only start a download; the user must confirm they saved the file.

---

## 6. Passphrase contract

Unchanged from prior draft: reject only exact empty; no trim/normalize; exact confirm; soft length guidance; clear fields after exit; no storage of secrets; honest zeroization limits.

---

## 7. Crypto / storage reuse

Reuse S3-1 vault helpers and S2 backup encrypt/decrypt/import validation. No format changes.

### Enrollment coordinator (second-corrected signatures)

```text
// Wizard-owned session object (in-memory only):
enrollmentSession = {
  recoveryExportOk: false,           // set ONLY by successful exportEnrollmentRecoveryBackup
  recoveryExportedAt: null,
  approvedStorageRaw: null,          // exact STORAGE_KEY at freeze (before first await)
  approvedLiveDocument: null,        // normalized live document freeze
  approvedBackupObject: null,        // immutable backupObject freeze; encrypt THIS only
  passphraseAccepted: false,
  pendingKey: null,                  // optional Option B CryptoKey staging
  pendingSaltB64: null,
  pendingCreatedAt: null,
  pendingKdfIterations: null,
  token: 0
}

function enrollmentApprovalStillValid(enrollmentSession) → boolean
  → recoveryExportOk
  && localStorage.getItem(STORAGE_KEY) === approvedStorageRaw
  && normalizedEqual(liveStateDocument(), approvedLiveDocument)
  // On false: clear recoveryExportOk and all approval snapshots

async function exportEnrollmentRecoveryBackup(enrollmentSession, passphrase, seams?)
  → SYNCHRONOUSLY freeze approvedStorageRaw, approvedLiveDocument, approvedBackupObject
     before any await (no live re-read after freeze for the export payload)
  → await encrypt(approvedBackupObject) + download initiate
  → sets recoveryExportOk only on success
  → does not create vault keys

async function commitEncryptedLocalVaultEnrollment(enrollmentSession, passphrase, seams?)
  → passphrase required unless enrollmentSession.pendingKey already set (Option B)
  → requires enrollmentApprovalStillValid(enrollmentSession)
  → encrypts approvedLiveDocument (must still equal live doc via validity check)
  → E1 only (does NOT retire plaintext)
  → clears passphrase locals and pendingKey staging on exit
  → MUST NOT accept untrusted recoveryDownloadConfirmed boolean as sole gate

async function retireLeftoverPlaintextAfterVaultProof(seams?)
  → requires vault unlocked, vault key valid, STORAGE_KEY present, user confirmed
  → removeItem(STORAGE_KEY)
  → if this tab becomes plaintext afterward, set plaintextStorageBaselineRaw appropriately
     (null after remove if no key remains; or not applicable while vault-unlocked)

// Ordinary plaintext path (extend existing persist(); still synchronous boolean):
// plaintextStorageBaselineRaw — session memory, set at loadState / successful plaintext write / this-tab restore install
// immediately before setItem(STORAGE_KEY, nextRaw):
//   if localStorage.getItem(STORAGE_KEY) !== plaintextStorageBaselineRaw → return false (no write; reload required)
//   else setItem; plaintextStorageBaselineRaw = nextRaw

async function restoreWorkspaceFromEncryptedBackupWhileLocked(fileText, backupPassphrase, seams?)
  → locked or blocked only
  → capture startingVaultRaw + startingVaultGeneration (sync) before mutations
  → structural validate backup → decrypt → validateBackupImport (memory only)
  → destructive confirm already obtained by UI
  → write normalized plaintext to STORAGE_KEY WHILE vault keys still exist
  → read back STORAGE_KEY; verify normalized equality
  → re-read VAULT_STORAGE_KEY; require byte-identical to startingVaultRaw
       (else abort with no vault deletion)
  → delete order: VAULT_META_KEY, then VAULT_PENDING_KEY, then VAULT_STORAGE_KEY LAST
  → install plaintext mode + in-memory state
  → set plaintextStorageBaselineRaw to the verified restored STORAGE_KEY raw in this tab
  → on any failure before VAULT_STORAGE_KEY removal: leave primary vault untouched
```

### Protections (additions)

| Hazard | Mitigation |
| --- | --- |
| Lockout after retire | Minimal locked restore |
| Retire before cold unlock | Phase E2 gate |
| Fake “I saved backup” | Internal `recoveryExportOk` |
| Recovery approval for stale workspace | Freeze-before-async + `enrollmentApprovalStillValid` |
| Backup encrypt of moving state | Encrypt only `approvedBackupObject` |
| Commit without crypto material | Passphrase or `pendingKey` required |
| Restore removes vault before plaintext durable | Write+verify plaintext first |
| **Newer vault deleted by restore** | Starting vault raw binding + pre-delete revalidation |
| Wrong deletion order | Meta/pending first; **`VAULT_STORAGE_KEY` last** (commit) |
| Quota mid-restore | Vault retained |
| Stale unlocked tab recreates vault after restore | Re-read vault before every vault setItem; missing/changed vault → fail closed, no write |
| **Dormant plaintext tab overwrites restored STORAGE_KEY** | **`plaintextStorageBaselineRaw` compare immediately before every plaintext setItem** |
| Dual authority if restore interrupted after plaintext write | Cold start: vault authoritative until key removed |

---

## 8. Multi-tab

| Scenario | Behavior |
| --- | --- |
| Second plaintext tab persists during wizard | Allowed until export approval; any `STORAGE_KEY` or live-doc change **invalidates** `recoveryExportOk` |
| Edit after recovery export, before commit | Same tab or other: approval invalid; must re-export recovery backup |
| Vault appears mid-enroll | Abort; do not delete foreign vault; this tab blocked if vault key present |
| Two tabs enroll | First durable vault wins; second fails preflight |
| Reload mid-enroll | Plaintext if no vault; if pending only, plaintext + discard pending on next enable or auto-clear stale pending at enroll start |
| Storage events | Optional status refresh only (not required for validity checks) |
| Older plaintext tab after vault commit | Next `persist()` hits vault-key guard → blocked shell / reload message (existing) |
| Restore interrupted after plaintext write, vault not removed | Reload: vault authoritative; user may unlock (if they know passphrase) or retry restore |
| Unlocked tab writes newer vault during another tab’s restore | Pre-delete revalidation fails; restore aborts; newer vault kept |
| Unlocked tab persists after restore removed vault | Must not recreate vault; re-read shows missing key → fail closed / reload |
| **Dormant plaintext tab never wrote; other tab enrolled + restored different backup; dormant tab later `persist()`s** | Baseline raw ≠ current `STORAGE_KEY` → **no write**; restored data unchanged; reload required |

Preflight immediately before encryption: `enrollmentApprovalStillValid` must be true; do **not** enroll a newer workspace under an old recovery approval.

After successful restore in one tab, other tabs must reload (vault gone / plaintext present). **Ordinary plaintext tabs are not exempt** — baseline compare enforces this even if they never open a vault.

---

## 9. UI flow and wording (corrected)

### Enable (Reference → Browser security; Help link)

Steps:

1. Explain (at-rest only; unlocked visibility; no reset).  
2. Passphrase + confirm.  
3. Create encrypted recovery backup from a **frozen** workspace snapshot → internal success → checkbox enabled. If you edit data afterward, export again.  
4. Final confirm: create vault; **browser may still hold a readable copy until you reload, unlock, and choose to remove it.**  
5. Busy.  
6. Success: unlocked; prompt to **reload now** to verify lock.  
7. After unlock: if `STORAGE_KEY` still present → panel **Remove readable browser copy**.

### Locked shell additions

- Unlock form (existing).  
- Link/button: **Can’t unlock? Restore from encrypted backup…**  
- Short note: restore writes ordinary browser storage first, verifies it, rechecks that the vault was not changed elsewhere, then removes the local vault; requires the recovery file passphrase.

### Copy additions

| Element | Wording |
| --- | --- |
| Leftover plaintext | A readable copy of your data still exists in this browser until you remove it. Anyone with access to this browser profile can read it. |
| Retire action | Remove readable browser copy |
| Retire confirm | After this, only the encrypted local vault and your backup files hold this workspace. Continue? |
| Restore action | Restore from encrypted backup |
| Restore confirm | Restore this backup into ordinary browser storage, then remove the local vault only if it is still the same vault as when restore started. If anything fails or another tab updated the vault, the vault is kept. This cannot open a forgotten vault passphrase. |
| Restore success | Workspace restored as ordinary browser storage. The local vault was removed only after the restored copy was verified and the vault binding was unchanged. |
| Restore vault-changed abort | The local vault changed in another tab. Nothing was removed. Reload this page. |

---

## 10. Accessibility

Same as prior draft, plus: restore dialog destructive confirm; retire panel accessible name; status live regions for export-ok / enroll / restore phases; focus restore after locked restore completes.

---

## 11. Fixture plan (corrected)

Keep **Security S3 Local Vault** group / route / promise / counter.

**Add assertions covering:**

1–11. Prior enrollment items that still apply, **except** “plaintext removed immediately.”  
12. After E1: vault present **and** `STORAGE_KEY` still present.  
13. Simulated cold bootstrap locked + unlock succeeds.  
14. Retire removes `STORAGE_KEY` only after unlock proof flag/session.  
15. Locked restore with correct backup passphrase → verified plaintext present, then meta/pending cleared, `VAULT_STORAGE_KEY` removed last, data present.  
16. Locked restore wrong passphrase → no mutation (vault + storage unchanged).  
17. Restore write failure (injected `setItem` throw) → vault keys **still present**; no vault removal.  
18. Restore read-back mismatch (injected) → vault retained.  
19. **Vault changes during restore** (after plaintext verify, swap `VAULT_STORAGE_KEY` to a different valid envelope / higher generation) → **abort; original/new vault material not deleted by the failing path; starting binding mismatch → no `VAULT_STORAGE_KEY` removal.**  
20. **After successful restore, old unlocked tab attempts vault persist** (session still “unlocked”, inject drain) → **no vault recreated**; fail closed; plaintext `STORAGE_KEY` remains restored data.  
21. Deletion order observable in seams: meta/pending removed before primary vault key on success.  
22. `commitEncryptedLocalVaultEnrollment` fails when `recoveryExportOk` is false.  
23. After recovery export, mutate state or `STORAGE_KEY` → commit refuses until new export.  
24. Commit without passphrase and without `pendingKey` refuses.  
25. Recovery export encrypts only the pre-await frozen `approvedBackupObject` (mutate live state after freeze, before await resolves → backup still matches freeze).  
26. Recovery export fail leaves zero vault keys.  
27. Cancel before E1 → byte-identical storage.  
28. Multi-tab vault appear aborts E1.  
29. **Dormant plaintext tab:** load workspace A; do not write; other context enrolls then restores workspace B; dormant tab ordinary `persist()` → **blocked**; `STORAGE_KEY` still equals restored B; dormant in-memory A not written.  
30. Cleanup restores all keys/modes.

---

## 12. Protected boundaries

Unchanged: no format/schema/key renames; `persist()` stays sync boolean (baseline compare remains synchronous); Designs/SVG untouched; offline `file://`; S1/S2 envelopes frozen. The plaintext baseline is **session memory only**, never a new permanent storage key.

---

## 13. Implementation blueprint (corrected)

### Files

- **`index.html` only** for product: enroll wizard, commit/retire helpers, locked restore, Help, Reference panel, fixtures.  
- New dated **implementation report** only.  
- Do **not** edit this architecture file during implementation.

### Sequence for Codex

1. Internal enrollment session + recovery export (**freeze-before-async** + `recoveryExportOk`).  
2. `enrollmentApprovalStillValid` + invalidation on change.  
3. `commitEncryptedLocalVaultEnrollment(session, passphrase, …)` (E1, **keeps plaintext**).  
4. Post-unlock retire UI + `retireLeftoverPlaintextAfterVaultProof`.  
5. **`restoreWorkspaceFromEncryptedBackupWhileLocked`**: starting vault bind → plaintext write+verify → revalidate vault → meta/pending → **`VAULT_STORAGE_KEY` last**.  
6. Harden vault persist path: re-read vault before setItem; missing/changed → fail closed (blocks stale vault-tab recreate).  
7. Harden ordinary plaintext `persist()`: `plaintextStorageBaselineRaw` load/write tracking + pre-setItem compare (blocks dormant plaintext-tab clobber).  
8. Help text.  
9. Fixtures: snapshot freeze, approval invalidation, restore vault-changed abort, post-restore stale **vault** persist, post-restore/enroll dormant **plaintext** persist, write-fail retains vault, E1–E3.  
10. Edge disposable: enroll → leftover → reload → unlock → retire → locked restore success; vault-changed-during-restore; stale vault-tab persist; **dormant plaintext tab vs restore**.

### Acceptance criteria (must all pass)

- Enroll cannot finish without successful recovery encrypt+download initiate + checkbox **and** still-valid snapshot binding.  
- Recovery backup and approval binding come from one pre-await freeze.  
- Workspace change after recovery export blocks commit until re-export.  
- Commit requires passphrase or staged key material.  
- After vault commit, plaintext still present until retire.  
- Reload shows locked shell without record flash; unlock works.  
- Retire requires unlock-in-this-session proof + confirm.  
- Locked restore writes+verifies plaintext, revalidates starting vault binding, then removes meta/pending before **`VAULT_STORAGE_KEY` last**.  
- If vault changes mid-restore, primary vault is **not** deleted.  
- After restore, stale unlocked **vault** tab cannot recreate vault over plaintext.  
- After restore (or foreign plaintext rewrite), dormant ordinary **plaintext** tab cannot overwrite `STORAGE_KEY` without matching baseline.  
- Wrong restore passphrase mutates nothing.  
- S3 fixtures green; S1/S2 still green; suite 0 failed when claimed; no production SVG change.

### Stop conditions

- Omitting locked restore while allowing full plaintext retirement.  
- Restoring by removing vault before verified plaintext write.  
- Restoring without starting-vault bind + pre-delete revalidation.  
- Deleting `VAULT_STORAGE_KEY` before meta/pending on the success path.  
- Accepting caller boolean as sole recovery proof.  
- Encrypting a live re-read backup after approval freeze.  
- Committing enrollment without passphrase/key material or without snapshot-valid recovery approval.  
- Shipping ordinary plaintext `persist()` without loaded-storage baseline compare (dormant-tab clobber).  
- Changing frozen envelopes.  
- Implementing Disable Vault / idle lock / full recovery center instead of minimal restore.

---

## 14. Risks (updated)

| Severity | Risk | Mitigation |
| --- | --- | --- |
| **Critical** | Enable vault then lock out forever | Minimal locked restore **in S3-2** |
| **Critical** | Restore removes vault before plaintext durable | Write+verify `STORAGE_KEY` first |
| **Critical** | Restore deletes a **newer** vault written by another tab | Starting vault raw bind + pre-delete revalidation; abort delete |
| **Critical** | Delete plaintext before cold unlock | Deferred retire (E2/E3) |
| **High** | Recovery backup for wrong/stale workspace | Freeze-before-async + approval validity |
| **High** | Stale unlocked tab recreates vault after restore | Re-read before vault setItem; fail closed |
| **Critical** | Dormant plaintext tab overwrites restored `STORAGE_KEY` | Baseline raw compare before every plaintext setItem |
| **High** | Checkbox-only recovery theater | Internal export-ok + download initiate |
| **High** | Dual authority leftover forgotten | Post-unlock retire prompt |
| **Medium** | Restore lands plaintext user wanted vault | Document re-enroll path |
| **Medium** | Interrupt after plaintext write during restore | Vault still authoritative on reload |
| **Low** | Filename preference | Labeling only |

---

## 15. Draft history — do not implement

| Flawed design | Required contract |
| --- | --- |
| Unlock-only after enroll | Minimal locked restore in S3-2 |
| Immediate `removeItem(STORAGE_KEY)` after vault write | Retire only after reload+unlock+confirm |
| Skip cold reload | Cold unlock is a **gate** for retire |
| `recoveryDownloadConfirmed` argument alone | Internal export-ok + snapshots |
| Restore: remove vault then write plaintext | **Write+verify plaintext, then remove vault** |
| Restore: delete vault without revalidation | **Bind starting raw; revalidate; abort if changed** |
| Restore: delete `VAULT_STORAGE_KEY` first | Meta/pending first; **primary vault last** |
| `commit…(session)` with no passphrase/key | Passphrase or staged `pendingKey` required |
| Recovery approval lasts across workspace edits | Invalidate on `STORAGE_KEY` or live-doc change |
| Recovery encrypt after live re-read | **Single freeze before first await** |
| Stale tab setItem vault after restore | Fail closed; no recreate |
| Dormant plaintext tab setItem after restore | Baseline compare; no overwrite of foreign STORAGE_KEY |

---

## 16. Discipline

- No S3-2 code implemented in this pass.  
- Only this architecture report rewritten.  
- Nothing staged, committed, or pushed.  
- No access to Joe’s real profile or data.

---

## 17. Final verdict line

**1. S3-2 ARCHITECTURE READY — Proceed to bounded Codex implementation using this fourth-corrected design only: freeze-before-async recovery export; commit with passphrase/key material and snapshot-valid approval; deferred plaintext retirement after cold unlock; locked-shell restore with starting-vault bind, pre-delete revalidation, meta/pending then `VAULT_STORAGE_KEY` last; block stale unlocked vault tabs from recreating a vault; and block dormant ordinary plaintext tabs via `plaintextStorageBaselineRaw` compare immediately before every plaintext `setItem`. Do not implement earlier drafts.**
