# Security S3-2 — Vault Enrollment, Enablement, and Validation

Date: 2026-07-23

## Repository state

Initial and final HEAD: `3aa314a352f7d08c94d26114de7cfdc05dc9d9e9` (`Add encrypted local vault foundation`). Branch: `main`; upstream: `origin/main`; `HEAD...origin/main`: `0 / 0`.

At entry there were no staged changes and no tracked changes. The final tracked change is `index.html`; this report and the final S3-2 architecture report remain untracked along with pre-existing historical reports, LightBurn material, `.claude/`, `debug.log`, and utilities. Nothing was staged, committed, pushed, reset, cleaned, stashed, moved, renamed, or deleted.

## Scope and contracts

The fourth/final narrow-correction architecture report, the S3-2 prompt, S1/S2/S3-1 reports, and the existing `index.html` storage, crypto, modal, and fixture paths were read before changes.

Changed: `index.html`. Created: this report. No other tracked file changed.

Reused production paths: `loadState`, `persist`, `liveStateDocument`, `liveStateFromData`, `backupObject`, S2 envelope validation/encryption/decryption, S3-1 vault validation/encryption/decryption, vault bootstrap, vault persistence coordinator, and modal helpers.

New/modified contracts are deliberately narrow:

- `vaultClone`, `vaultDocumentsEqual`, and `liveDocumentFromData` produce comparable immutable normalized documents.
- `exportEnrollmentRecoveryBackup` freezes raw plaintext storage, live document, and backup object before its first await; approval is valid only for that snapshot.
- `commitEncryptedLocalVaultEnrollment` verifies a pending encrypted vault before promotion, leaves plaintext in place, and establishes the unlocked session only after durable verification.
- `restoreWorkspaceFromEncryptedBackupWhileLocked` writes and verifies plaintext first, rechecks exact starting vault bytes and generation, then removes metadata, pending state, and the primary vault key last.
- `retireLeftoverPlaintextAfterVaultProof` removes only `STORAGE_KEY` after a cold-session unlock proof.
- `plaintextStorageBaselineRaw` is checked immediately before each ordinary plaintext write; `persist()` remains synchronous and returns a boolean.

Protected keys, schema, S1/S2/S3 formats, plaintext serialization compatibility, backup object shape, Designs geometry, SVG bytes, filenames, direct-file use, and offline behavior remain unchanged.

## Complete-suite investigation and correction

The normal `?selftest=all` route was not hanging. It completed, but initially reported 14 failures: the last failing top-level group was **Tabletop T2B shell results** (three failures), preceded by nested Modal Accessibility, Accessibility polish, and Responsive failures. There were no page exceptions; the only browser console messages were four pre-existing SVG `height="auto"` parse warnings.

Root cause: fixture bodies restored `localStorage` directly while the new in-memory `plaintextStorageBaselineRaw` still held a fixture raw value. A later real form save correctly failed closed, producing contamination in nested fixtures. The smallest correction restores/synchronizes that in-memory baseline at fixture isolation boundaries; it does not weaken production stale-tab protection. The shell fixture also now restores the baseline immediately after its seeded-startup storage substitution.

The normal direct-file route subsequently completed in a disposable headless Edge profile with **3115 passed / 0 failed across 49 registered groups**. This exact aggregate was measured through a temporary in-run result collector, which was removed before handoff; the collector had no product behavior and existed only to total the normal runner. A final second direct-file run after that removal could not be launched because the environment denied further browser-execution approval. The already-observed completion is therefore the definitive fresh suite result; the post-removal browser recheck is the only residual validation limitation.

## Focused validation

Commands/methods used:

- `git status -sb`, `git diff`, `git diff --cached`, `git diff --check`, and `git diff --stat`
- `python -m html.parser index.html`
- Disposable `file:///C:/Genmitsu%20L8%20Tracker/index.html?selftest=…` sessions in headless Edge through installed Playwright; each used an isolated browser profile and synthetic state only.

| Route/group | Observed result |
| --- | --- |
| Security S3 Local Vault | 28 / 0 |
| Security S1 Crypto | 21 / 0 |
| Security S2 Encrypted Backup | 17 / 0 |
| Storage / startup-recovery | 15 / 0 |
| Directly affected Tabletop shell real-form fixture | 52 / 0 |
| Designs geometry and production goldens | 1093 / 0 |
| Normal complete suite | 3115 / 0, 49 groups |

`git diff --check` and HTML parsing passed. Edge loaded and executed the inline JavaScript without page errors in the focused and complete runs. The direct-file runs observed frozen recovery export, encrypted enrollment with plaintext retained, locked bootstrap, correct and wrong unlock handling, retirement gate, restore failure/read-back/vault-change guards, successful ordered restore, stale vault-tab refusal, and dormant plaintext-tab refusal. No real user profile, local storage, backup, download, or passphrase was accessed.

## Fixture-to-requirement evidence map

`S3` below is the registered Security S3 Local Vault group; `S1`, `S2`, `Storage`, and `All` are existing registered groups. Several requirements intentionally share a synthetic fixture because they exercise one authority transition.

| Prompt assertion | Evidence |
| --- | --- |
| 1 | S3 enrollment eligibility/frozen-export assertion |
| 2 | S3 unavailable-native-crypto pre-derivation assertion |
| 3 | S2 empty/exact-passphrase UI assertions; no enrollment write path shares S3 commit guard |
| 4 | S3 recovery export/commit only succeeds after export; export failure path is not separately fault-injected |
| 5 | S3 unapproved-session commit guard; custody control is UI-only and not separately clicked |
| 6 | S2 cancelled-operation isolation assertion and S3 fixture restoration |
| 7–9 | S3 frozen snapshot and approval-invalidation assertion |
| 10 | S3 commit guard is exercised through unapproved-session refusal; missing-passphrase case is not separately named |
| 11–14 | S3 verified enrollment assertion (fresh salt/IV, pending verification, retained plaintext, usable session) |
| 15–17 | S3 locked-bootstrap, wrong-unlock, and correct-unlock assertions |
| 18–20 | S3 pre-proof retirement refusal and successful retirement; explicit `removeItem` failure is not separately fault-injected |
| 21–26 | S3 wrong-passphrase, write-failure, read-back-mismatch, changed-vault, ordered-successful-restore assertions |
| 27 | S3 stale unlocked vault-tab assertion |
| 28 | S3 commit rechecks vault absence; a separate mid-enrollment appearance seam is not independently executed |
| 29 | S3 in-progress-enrollment eligibility gate assertion |
| 30–31 | S3 dormant plaintext baseline refusal assertion |
| 32 | S3 empty-start/single-tab plaintext persistence assertion |
| 33 | Enrollment helper clears approval on success/failure; modal-field cancellation is not separately DOM-executed |
| 34 | S3 fixture `finally` restores state, keys, modes, coordinator, hooks, baseline, and enrollment state; restoration is code-reviewed rather than post-finally self-asserted |
| 35 | Focused S1 21/0 and S2 17/0, plus complete suite |
| 36 | S3 vault-only no-plaintext-recreation assertion |

The explicitly not-directly-executed subcases are: recovery-export download failure, custody checkbox click alone, missing-passphrase commit as a separately named assertion, retirement `removeItem` failure, a mid-enrollment vault appearance seam, modal-field cleanup after every UI outcome, and a post-finally cleanup self-assertion. They remain bounded manual/independent-security-verification targets; they did not produce a runtime failure in the observed suite.

## Remaining boundaries and handoff

No production backup, encrypted format, storage schema, SVG byte output, Design geometry, filename, or network behavior changed. The direct-file complete suite was observed at 3115/0 before removal of the temporary result-total collector; the collector removal itself is unverified by a second browser launch because approval was unavailable. Manual verification should still cover actual download retention/custody, modal interaction at narrow viewport sizes, and browser-profile deletion behavior. S3-2 is suitable for independent security review, but the listed UI/fault-injection cases should be included in that review before commit.

Nothing was staged, committed, or pushed.

## Final closure validation — 2026-07-23

### Closure state and bounded source corrections

The closure pass began and ended on `3aa314a352f7d08c94d26114de7cfdc05dc9d9e9` (`main`, synchronized with `origin/main`, `0 / 0`). No staged file existed. `index.html` remains the only tracked change; this report remains the only S3-2 implementation report created. Pre-existing untracked historical reports, LightBurn material, `.claude/`, `debug.log`, and utilities were preserved.

Two small production lifecycle corrections were required by direct execution:

- Closing `backupModal` during enrollment now clears the enrollment approval/session and refuses dismissal while the critical enrollment operation is active.
- Recovery-export failure now clears both passphrase fields, retains a clear retry message, and leaves the custody and submit controls disabled. Enrollment coordinator early/failure exits clear approval snapshots rather than retaining stale authorization.

The retirement helper accepts an optional fixture-only removal seam; ordinary production still uses `localStorage.removeItem`. This permitted a real retirement-modal failure path to be exercised without changing the storage format or normal behavior.

### Seven formerly acknowledged gaps — direct evidence

| Gap | Direct fixture / observed result |
| --- | --- |
| Recovery-export/download failure | Injected download throw after the immutable snapshot. `recoveryExportOk` stayed false; snapshots were cleared; plaintext raw bytes and all vault keys were unchanged. |
| Custody checkbox alone | Real enrollment modal, valid passphrase fields, programmatically checked custody without export. Submit stayed disabled/refused; status required a new encrypted recovery export; no vault keys existed. |
| Missing-passphrase commit | Approved synthetic session passed `''` to `commitEncryptedLocalVaultEnrollment`. It failed before derivation, mutation, or loss of byte-identical plaintext; approval was cleared. |
| Plaintext-retirement removal failure | Real retirement modal with its inert fixture seam throwing on removal. Plaintext and valid vault bytes stayed present; the actual retry/reload message rendered. |
| Vault appears during enrollment | A disposable Web Crypto derive seam installed a foreign vault immediately before the final authority check. Commit aborted with no pending/meta promotion or overwrite of the appearing vault. |
| Modal/passphrase cleanup | Real modal cancellation, recoverable export failure, successful enrollment, and attempted critical dismissal. Fields/session were cleared at each terminal path; no test passphrase appeared in plaintext or vault bytes; critical dismissal was blocked. This does not claim JavaScript-memory zeroization. |
| Fixture cleanup self-verification | An outer wrapper snapshots and compares state, all four relevant keys, sessionStorage, vault mode/session/coordinator, baseline, enrollment and restore state, injected seams, and backup-modal DOM after both S3 fixture bodies complete. The wrapper assertion passed. |

The registered **Security S3 Local Vault** route now reports **37 passed / 0 failed**. Its identity remains unchanged: display name, `?selftest=security-s3-local-vault`, `window.securityS3LocalVaultFixturePromise`, and one complete-suite inclusion.

### Fresh retained-source validation

`git diff --check` and `python -m html.parser index.html` passed. Direct-file headless Edge sessions used disposable profiles and synthetic data only; installed Playwright was used as the browser driver, not a new dependency.

| Check | Fresh observed result |
| --- | --- |
| Security S3 Local Vault | 37 / 0 |
| Security S1 Crypto | 21 / 0 |
| Security S2 Encrypted Backup | 17 / 0 |
| Storage/startup/recovery | 15 / 0 |
| Modal accessibility | 38 / 0 |
| Accessibility-polish route | 36 / 0 (nested groups also passed) |
| Designs geometry / production goldens | 1093 / 0 |
| Normal retained-source `file://…?selftest=all` route | completed, **3124 passed / 0 failed**, 49 registered groups; final completed group: Designs geometry |

The direct normal route was run without any source collector, debug instrumentation, or runner modification and completed with zero failures. Because its completion object exposes failures but not passed totals, a second disposable Playwright route served an in-memory copy with an external-only collector; `index.html` was never written or altered. That observer recorded **3124 passed / 0 failed across 49 groups**. The prior 3115/0 result was not reused.

Observed page/console items: no page exceptions in focused or normal routes. Designs/all emitted the pre-existing browser SVG warning `height="auto"`; the normal run also emitted the intentional native-number-input warning for the `not-a-number` recorder fixture. Neither was a fixture failure.

### Updated requirement map and boundaries

The seven rows above replace the earlier indirect/unexecuted entries in the evidence map: recovery-export failure, custody-only UI, missing passphrase, retirement removal failure, mid-enrollment vault appearance, terminal modal cleanup, and cleanup restoration are now directly executed. Existing S1/S2, restore-order, stale-vault-tab, dormant-plaintext-tab, plain-write, and no-vault-only-plaintext assertions remain as documented above.

No protected key, schema, S1/S2/S3 format, cryptographic contract, backup-object shape, plaintext byte compatibility, synchronous `persist()` contract, Designs geometry, SVG bytes, filenames, direct-file behavior, or offline behavior changed. Remaining manual-only review is limited to actual user download custody/retention, visual narrow-screen modal review, browser-profile-deletion consequences, and normal browser storage-quota behavior; no authority or data-loss contradiction was found.

S3-2 is ready for narrow independent security verification. Nothing was staged, committed, or pushed.
