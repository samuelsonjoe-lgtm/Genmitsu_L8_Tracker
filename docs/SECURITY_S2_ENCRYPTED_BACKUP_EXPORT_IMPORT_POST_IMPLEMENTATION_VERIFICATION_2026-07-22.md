# Security S2 — Encrypted Backup Export/Import Post-Implementation Verification

Date: 2026-07-22  
Repository: `C:\Genmitsu L8 Tracker`  
Verifier role: independent, read-only post-implementation verification  
Authority: current `index.html` source and executed Edge `file://` probes  
Product files modified by this pass: **none**  
Authorized output only: this report

---

## Verdict

**2. VERIFIED WITH CAUTION — Safe to commit Security S2, with the precise non-blocking cautions below.**

Cautions (do not block S2 commit for scope isolation and security posture):

1. **Google Chrome is not installed** on this machine (standard executable paths and command discovery). Chrome results were not measured and are not inferred from Edge.
2. **Independent full-suite outer-return total is 3068 passed / 0 failed across 48 unique groups**, not the implementation report’s 3041. All groups reported **0 failed**; S1 and S2 each ran once under `?selftest=all`. The 3041 figure is consistent with arithmetic on the prior 3024 baseline (+17) but understates this pass’s outer-return sum by **27** (same +27 gap appears when removing S2 from this sum versus the claimed pre-S2 3024). This is a **reconciliation/reporting discrepancy**, not a failing test.
3. **S2 fixture pack is thin (17 asserts)** relative to the threat model: it does **not** assert Merge/Replace byte-parity versus plaintext, full import-path tamper matrix, instrumented zero-PBKDF2 for every invalid envelope class, or a 1 MiB payload on the S2 UI/download boundary (1 MiB remains covered on the committed S1 helpers). Adversarial Edge probes covered zero-PBKDF2 invalid import routing and UI paths; remaining gaps are **missing fixture evidence**, not confirmed product defects.
4. **T3A production pin `897 / b5c549ee` is not present in source** as a registered length/hash assertion (implementation-report wording “non-registered measured pin” matches source). Design geometry still **1093 / 0**; listed registered goldens that appear in source remained green under that group.

---

## 1. Actual repository state

| Item | Observed |
| --- | --- |
| HEAD | `8c06800ff629281d3e6418004b33552a3d3f9dc3` |
| Subject | `Add Security S1 crypto foundation` |
| Branch | `main` |
| Upstream | `main...origin/main` synchronized (`0 0` left/right) |
| Staged | **Nothing staged** (`git diff --cached` empty) |
| Tracked modification | **`index.html` only** — `237 insertions(+), 18 deletions(-)` (~255 line churn) |
| Untracked (preserved, not part of S2 product) | Pre-existing docs history, `docs/SECURITY_S2_ENCRYPTED_BACKUP_EXPORT_IMPORT_IMPLEMENTATION_2026-07-22.md`, `LightBurn Projects/`, `debug.log`, `parametric_qr_stand_generator.py`, and other historical review docs |
| This verification report | Created only as authorized; no product edits |

### Complete git status (summary)

- Modified tracked: `index.html`
- Untracked: many historical `docs/*` files, S2 implementation report, LightBurn tree, `debug.log`, utility script
- No staged paths

### Exact S2 tracked diff inspected

- Diff isolated to `index.html`
- `git diff --check HEAD -- index.html` → **pass** (exit 0; CRLF warning only)
- Added surface (new functions):  
  `backupOperationIsCurrent`, `backupModalStatus`, `openBackupMessage`, `encryptedBackupUnavailableMessage`, `createEncryptedBackupExport`, `decryptValidatedEncryptedBackup`, `downloadBackupJson`, `openBackupChoiceDialog`, `openEncryptedBackupExportDialog`, `startEncryptedBackupExport`, `openPlainBackupWarning`, `openEncryptedBackupImportDialog`, `startEncryptedBackupImport`, `routeBackupImportData`, `handleBackupImportText`, `runSecurityS2EncryptedBackupFixtures`
- Wiring: `exportBtn` → `openBackupChoiceDialog`; import file reader → `handleBackupImportText` → `routeBackupImportData`
- Modal host: `#backupModal`
- Fixture registration: `?selftest=security-s2-encrypted-backup` and inclusion under `all`, awaited like S1
- Unrelated untracked docs/assets **not** mixed into the tracked S2 diff

---

## 2. Scope-isolation result

**Pass — S2 is limited to password-protected encrypted backup file Export/Import integration.**

Verified **unchanged** vs committed HEAD (byte-identical bodies / constant values where compared):

| Boundary | Result |
| --- | --- |
| Full S1 crypto block (`ENCRYPTED_BACKUP_*` through decrypt helpers) | **Identical** to HEAD (~12 954 bytes) |
| S1 fixture `runSecurityS1CryptoFixtures` | **Identical** |
| `isEncryptedBackupEnvelope`, `validateEncryptedBackupEnvelopeV1`, `encryptSyntheticEncryptedBackup`, `decryptSyntheticEncryptedBackup`, `probeEncryptedBackupCryptoCapability`, `deriveEncryptedBackupKey`, `encryptedBackupCanonicalHeader` | **Identical** |
| `STORAGE_KEY`, `SCHEMA_VERSION`, `APP_ID`, `APP_NAME`, `APP_VERSION`, `BUILD_DATE`, `BACKUP_FORMAT` | **Identical values** |
| `backupObject`, `validateBackupImport`, `downloadTextFile`, `mergeData`, `replaceData`, `applyPendingImport`, `showImportReplaceConfirmation` | **Identical** (import path now **calls** existing `validateBackupImport` / `openImportModeDialog` after decrypt) |
| Forbidden vault/storage crypto tokens in added lines | **Absent** (`VAULT_`, local vault, live localStorage encryption, idle lock, recovery codes, constant reassignments) |

Verified **not implemented** by S2 (by absence in diff and architecture):

- Encrypted `localStorage` / live vault enrollment / migration  
- Vault keys, verifiers, markers, lock shell, idle lock, session passphrase retention  
- Recovery codes / passphrase reset  
- Schema / evidence / Designs geometry / production SVG serializers / network dependencies  

---

## 3. Frozen S1 cryptographic contract

**Pass — S2 reuses committed S1 without duplication or weakening.**

| Contract item | Status |
| --- | --- |
| `ENCRYPTED_BACKUP_FORMAT` | Unchanged |
| Envelope version `1` | Unchanged |
| PBKDF2-SHA-256, default **600000**, accepted **600000–2000000** | Unchanged |
| AES-256-GCM, 128-bit tag, 16-byte salt, 12-byte IV | Unchanged |
| Canonical AAD field list/order | Unchanged |
| Strict Base64 + exact-key envelope validation | Unchanged |
| Own-property encrypted-format detection | Unchanged |
| No fallback encryption / reversible encoding / third-party crypto / network crypto | Confirmed in S1 helpers + S2 UI refusal path |

S2 integration:

- Export: `createEncryptedBackupExport` → `backupObject()` + `encryptSyntheticEncryptedBackup`
- Import: `decryptValidatedEncryptedBackup` → envelope validate → `decryptSyntheticEncryptedBackup` → **existing** `validateBackupImport` → **existing** `openImportModeDialog`

---

## 4. Export-choice result

**Pass (with minor copy gaps vs the strict wording checklist).**

Executed on Edge headless disposable profile (`file://`):

| Check | Result |
| --- | --- |
| Export opens custom choice dialog | **Yes** (`#backupModal` open) |
| Password-encrypted is primary (`.primary`) | **Yes** |
| Plain JSON secondary | **Yes** (`data-plain-backup-export`) |
| Plain described as readable / contains workshop·project·pricing·inventory·evidence | **Yes** |
| Exact phrase “not password-protected” | **Not present** (intent covered by “readable”) |
| Encrypted: passphrase never saved | **Yes** |
| Encrypted form: no recovery if forgotten | **Yes** |
| Exact “lost passphrase → file unusable” | **Not present** (closest: no recovery if forgotten) |
| Only downloaded file protected; browser data stays plaintext | **Yes** on encrypted form status text |
| Does not claim localStorage/browser data encrypted | **Yes** (adversarial scan false) |
| Opening chooser performs **0** PBKDF2 (`crypto.subtle.deriveKey` count) | **Yes** (`derive:0, enc:0, dec:0`) |
| Shared modal infrastructure (`openModal` / `bindModal` / `closeModal`) | **Yes** |
| Cancel / close without download | **Yes** on plain-warning cancel path |

---

## 5. Capability-gate result

**Pass.**

| Check | Result |
| --- | --- |
| Uses real `probeEncryptedBackupCryptoCapability` | **Yes** (default probe on encrypted export/import dialogs) |
| Probe itself is one PBKDF2+AES round-trip (not repeated on render) | **Yes** — opening chooser does not probe; choosing encrypted runs probe once (`derive:1, enc:1, dec:1`) |
| Injected unavailable probe blocks encrypted export, no weak fallback | **Yes** (S2 fixture + message “No weaker fallback”) |
| Plain JSON still available after refusal path | **Yes** (chooser re-openable) |
| Capability failure does not mutate storage | **Yes** (fixture isolation + adversarial storage snaps) |

---

## 6. Encrypted-export result

**Pass (production integration boundary).**

Source + fixtures + UI:

| Check | Result |
| --- | --- |
| Passphrase + confirm fields, `type=password`, labels | **Yes** |
| Empty refused (`''` only); no trim/normalize in JS | **Yes** (fixture calls `startEncryptedBackupExport` directly) |
| HTML5 `required` may intercept empty `requestSubmit` before custom status | **Informational** — native validation; custom empty message still proven via direct API in fixtures |
| Mismatch refused before derivation | **Yes** (status “does not match”; `derive:0`) |
| No complexity/strength policy | **Yes** |
| Fresh salt/IV/ciphertext for same passphrase | **Yes** (S1 fixture; same helpers) |
| Outer envelope strict S1 validation | **Yes** |
| Filename `l8-tracker-backup-encrypted-YYYY-MM-DD.json` | **Yes** |
| Passphrase absent from envelope JSON | **Yes** (fixture + post-success local/session checks) |
| Download via `backupDownloadText` → `downloadTextFile` (blob URL revoked) | **Yes** |
| Failure does not download plaintext | **Yes** (`encrypted.ok` gate before download) |
| Success toast: encrypted backup created only | **Yes** (“Password-encrypted backup created.”) |
| Measured successful UI encrypt (this machine/Edge) | **~101 ms** wall for one export after probe (do not generalize) |

---

## 7. Async export safety

**Pass (fixture + partial UI exercise).**

| Check | Evidence |
| --- | --- |
| Duplicate submit starts at most one in-flight export | `backupModalOperation?.kind === 'encrypted-export'` guard; fixture “duplicate or cancelled…” **pass** |
| Busy/disabled submit while working | Submit disabled during operation |
| Promise rejection caught | `.catch` restores usable status when operation still current |
| Close invalidates operation | `closeModal('backupModal')` sets `cancelled` and clears token; `backupOperationIsCurrent` requires open modal |
| Stale completion cannot download | Fixture cancel path **pass** |
| Passphrase locals cleared on success/failure paths | Assignment to `''` + input cleared |
| State/storage isolation for fixture export path | Final S2 isolation assert **pass** |

---

## 8. Plain JSON export

**Pass.**

| Check | Result |
| --- | --- |
| Warning required before download | **Yes** |
| “This backup is readable” + records including pricing/inventory/evidence | **Yes** |
| Cancel/close → no download | **Yes** (adversarial) |
| Pretty `JSON.stringify(backupObject(), null, 2)`, `application/json`, `l8-tracker-backup-YYYY-MM-DD.json` | **Yes** (fixture) |
| Zero PBKDF2/AES on plain path | **Yes** |
| Historical plaintext import path retained | **Yes** (`routeBackupImportData` non-encrypted branch) |

---

## 9. Import detection and pre-PBKDF2 validation

**Pass — executed with `crypto.subtle.deriveKey` instrumentation.**

Routing (`routeBackupImportData`):

1. Own-property `isEncryptedBackupEnvelope` → exclusive encrypted handling  
2. `validateEncryptedBackupEnvelopeV1` before passphrase UI / derivation  
3. Malformed exact marker → reject message; **no** plaintext fallthrough  
4. Plain backups → existing `validateBackupImport` → `openImportModeDialog`

### Invalid envelopes (import File injection) — **deriveDelta = 0**, storage unchanged, no Merge/Replace

| Case | deriveDelta | Rejected |
| --- | --- | --- |
| Marker-only exact format | 0 | Yes |
| Foreign app | 0 | Yes |
| Envelope v2 | 0 | Yes |
| Unknown KDF | 0 | Yes |
| Unknown cipher | 0 | Yes |
| Iterations 599999 | 0 | Yes |
| Iterations 2000001 | 0 | Yes |
| Non-integer iterations | 0 | Yes |
| Malformed Base64 | 0 | Yes |
| Extra field | 0 | Yes |
| innerSchemaVersion 0 | 0 | Yes |
| Plaintext historical backup | 0 | Opens import mode (expected) |

Inherited-marker prototype object cannot be expressed as pure JSON File text; S1 fixture still proves non-own inherited marker is not detected.

Incorrect salt/IV length and truncated ciphertext classes are covered by **S1 validation fixtures** (closed) and share the same validator used before import passphrase UI.

---

## 10. Decryption, inner validation, Merge/Replace

| Area | Result |
| --- | --- |
| Passphrase dialog before Merge/Replace | **Pass** (fixture) |
| Empty import passphrase refused before decrypt | **Pass** |
| Wrong passphrase → decrypt stage fail; no import mode | **Pass** (fixture) |
| User-facing errors non-raw | **Pass** (“Could not open this backup…”) |
| Inner path uses `validateBackupImport` + APP_ID / BACKUP_FORMAT / schema rules | **Pass** (source) |
| UTF-8 fatal decode + JSON parse in S1 decrypt | **Pass** (committed S1) |
| Merge/Replace application path | **Existing** `openImportModeDialog` → `applyPendingImport` → `applyBackupImport` (no parallel apply engine) |
| Second Replace confirmation | **Unchanged** `showImportReplaceConfirmation` |
| Encrypted vs plaintext Merge/Replace **byte-parity** | **Not independently executed** in this pass (source path identity + no-mutation until apply). **Missing fixture evidence** |
| Cancel passphrase / import mode / replace confirm | Source: closes without apply; storage fixture isolation on cancel export |

S1 still covers wrong passphrase, ciphertext tamper, AAD/header tamper, 1 MiB helper round-trip.

---

## 11. Passphrase lifecycle

**Pass for inspected surfaces.**

| Check | Result |
| --- | --- |
| Not written to localStorage / sessionStorage | **Pass** (adversarial after export; S2 fixture isolation) |
| Not in normal app state / envelope / filename | **Pass** |
| Not in toast/status success strings | **Pass** |
| Fields cleared after success/failure paths | **Pass** (source) |
| No cross-operation cache of raw/derived key | Non-extractable keys; locals cleared; no global passphrase store |
| Dialog reopen does not prefill | New form HTML each open |
| Password-manager `autocomplete` attributes present | `new-password` / `current-password` — browser manager behavior **not** over-claimed |

---

## 12. Accessibility

**Pass at infrastructure level.**

- Uses existing `openModal` / `bindModal` / `closeModal` / focus-origin restoration  
- Dialogs include headings, close controls with aria-labels, `aria-live` status regions  
- Modal accessibility fixture group remains green under outer-return suite (38/0)  
- Focused keyboard trap matrix for backup dialogs specifically was not re-audited beyond shared infrastructure + successful open/close in automation

---

## 13. 1 MiB payload

| Boundary | Result |
| --- | --- |
| S1 helper 1 MiB encrypt/decrypt | **21st S1 assert pass** |
| S2 fixture / UI download 1 MiB integration | **Not covered by S2 fixtures** |
| Synthetic large-payload S2 UI round-trip | **Not completed** in this pass (tooling limited by IIFE-private helpers for custom state injection) |

---

## 14. S2 fixture-quality mapping (17 asserts)

| # | Assertion (short) | Exercises | Notes |
| --- | --- | --- | --- |
| 1 | Capability available | Production probe | OK |
| 2 | Choice encrypted primary / plain secondary | Production UI | OK; no PBKDF2 claim explicit |
| 3 | Unavailable crypto blocks export | Injected probe | OK; no download |
| 4 | Plain warning | UI | OK |
| 5 | Plain filename/MIME/pretty JSON | Production download stub | Strong |
| 6 | Encrypted form fields / no persist copy | UI | OK |
| 7 | Empty export passphrase | Direct `startEncryptedBackupExport` | Bypasses HTML5 `required` |
| 8 | S1 envelope + encrypted filename | Production encrypt + stub download | Strong |
| 9 | No passphrase/plaintext in file | Download text | Strong |
| 10 | Decrypt → validateBackupImport | `decryptValidatedEncryptedBackup` | Strong integration |
| 11 | Whitespace exact / wrong pass | Decrypt helper | Strong |
| 12 | Malformed marker reject | `routeBackupImportData` | Strong pre-PBKDF2 |
| 13 | Passphrase dialog before import mode | UI | OK |
| 14 | Empty import passphrase | Direct start import | OK |
| 15 | Import mode open without mutation | UI + state/storage | Strong pre-apply |
| 16 | Duplicate/cancel no extra download | Async token | Strong |
| 17 | State/storage/backup isolation | Snapshot equality | Compares state + STORAGE_KEY + `backupObject()`; not full sessionStorage matrix |

**Collapsed / missing evidence (not defects by count alone):**

- No per-class PBKDF2-zero matrix in fixtures (executed adversarially instead)  
- No Merge/Replace parity vs plaintext  
- No ciphertext/AAD tamper via full import UI  
- No 1 MiB S2 download boundary  
- No sessionStorage assert in the 17  
- Cancellation assert is real (close mid-flight + download count)

Focused isolation: S2 restores modal HTML, focus, download hook, operation token, state, storage.

---

## 15. Full-suite reconciliation

### Registered unique groups

**48** unique `selftest === '…' || selftest === 'all'` branches (alias `designs` not double-counted). S2 is group 47th key order; design remains last.

### Execution discipline

| Check | Result |
| --- | --- |
| Ordinary startup runs neither S1 nor S2 | **Pass** (`s1Run:0`, `s2Run:0`, no completion promise) |
| Focused S1 | **21 / 0**, run count **1**, S2 **0** |
| Focused S2 | **17 / 0**, run count **1**, S1 **0** |
| `?selftest=all` | S1 **21 / 0**, S2 **17 / 0**, run counts **1 / 1** |
| Dispatcher awaits S1 then S2 before design | **Yes** (source order + await) |
| Rejected async fixture → reported failure | **Yes** (try/catch → `failed:1` collect) |
| Completion cannot finish while S1/S2 pending | **Yes** (await before return) |

### Independent sum method

For each of 48 keys, focused `file://?selftest=<key>` on disposable Edge; take **outer runner return total** = last `console.group` with finite passed/failed (nested fixture calls log first; outer logs last). Materials final runner returns cumulative **57** (last group), not sum of intermediate peer logs. S1/S2 use fixture promises.

| Metric | Value |
| --- | --- |
| Unique groups | **48** |
| Independently summed passed | **3068** |
| Independently summed failed | **0** |
| Implementation claim | 3041 / 0 / 48 |
| Delta vs claim | **+27 passed** (all still 0 failed) |
| Pre-S2 implied by this method | 3051 (3068−17) vs claimed 3024 (**+27**) |

Naive summing of **all** nested console groups under a single `all` run **over-counts** (observed ~4891) because responsive/accessibility/machine runners nest other groups. Outer-return method matches `collect(result.failed)` semantics for failures and outer `passed` for totals.

### Per-group outer-return table (this pass)

| Key | Passed | Failed |
| --- | ---: | ---: |
| baseline | 20 | 0 |
| normalization | 12 | 0 |
| production | 66 | 0 |
| promotion | 58 | 0 |
| design-production | 118 | 0 |
| grid | 23 | 0 |
| grid-machine | 18 | 0 |
| grid-browser | 67 | 0 |
| materials | 57 | 0 |
| library-browser | 56 | 0 |
| project-browser | 61 | 0 |
| design-project | 17 | 0 |
| metadata | 12 | 0 |
| storage | 15 | 0 |
| first-run | 19 | 0 |
| machine | 50 | 0 |
| machines-m1 | 29 | 0 |
| machines-m2 | 31 | 0 |
| machines-m3 | 27 | 0 |
| help | 43 | 0 |
| modal | 38 | 0 |
| accessibility | 36 | 0 |
| responsive | 45 | 0 |
| empty-state | 60 | 0 |
| beginner | 22 | 0 |
| project-wizard | 216 | 0 |
| gift-box | 69 | 0 |
| dice-tray-system | 92 | 0 |
| tabletop-engine | 27 | 0 |
| tabletop-corner-floor-coupon | 76 | 0 |
| tabletop-storage-fit-coupon | 23 | 0 |
| tabletop-coupon-results | 33 | 0 |
| tabletop-evidence-promotion | 22 | 0 |
| tabletop-evidence-matching | 21 | 0 |
| tabletop-rectangular-shell | 93 | 0 |
| tabletop-rectangular-shell-evidence | 49 | 0 |
| tabletop-closed-corners | 13 | 0 |
| tabletop-layout | 6 | 0 |
| tabletop-shell-results | 33 | 0 |
| tabletop-shell-evidence-promotion | 16 | 0 |
| tabletop-shell-evidence-matching | 16 | 0 |
| tabletop-storage-results | 14 | 0 |
| tabletop-storage-tray | 37 | 0 |
| tabletop-workshop-ready | 15 | 0 |
| tabletop-tray-results | 66 | 0 |
| security-s1-crypto | 21 | 0 |
| security-s2-encrypted-backup | 17 | 0 |
| design | 1093 | 0 |
| **Total** | **3068** | **0** |

Also green when exercised: storage recovery (15), modal accessibility (38), design geometry (1093), project/library/inventory/pricing/machine/evidence groups as listed.

### Static parse checks

| Check | Result |
| --- | --- |
| `git diff --check` | Pass |
| HTML parse | Pass; **0** duplicate IDs |
| Node `--check` inline JS | **Node unavailable** on verifier host; Edge load + full fixture completion used as runtime JS proof |
| Malformed markup | No parser errors observed |

---

## 16. Production goldens

| Pin | In source as registered assert? | Suite evidence |
| --- | --- | --- |
| Legacy pocketed shell `2181 / 2ef9606b` | Yes | Design geometry **1093 / 0** |
| T1 coupon `2992 / 4f543f95` | Yes | Design geometry **1093 / 0** |
| T2 shell `2337 / ed5d6f6e` | Yes | Design geometry **1093 / 0** |
| T3A `897 / b5c549ee` | **No** — hash **absent** from `index.html`; length 897 not asserted | **Non-registered measured pin** (report wording accurate). Not executed as a pin. Not an S2 regression signal |
| T3B `2860 / 3e256fad` | Yes | T3B fixtures + design path green |
| Dice Tray `1726 / 51a55721` | Yes | Green |
| Alternate Dice Tray `1054 / 41697123` | Yes | Green |
| Divider Tray `1965 / a55dda6e` | Yes | Green |
| Wall-to-base coupon `1551 / d9ffc278` | Yes | Green |

S2 diff does not touch generators/SVG serializers; no production-byte change attributed to S2.

---

## 17. Direct-file browser verification

### Microsoft Edge

| Item | Value |
| --- | --- |
| Product | Microsoft Edge **150.0.4078.83** (`msedge.exe` FileVersion) |
| Executable | `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` |
| Mode | Headless CDP, disposable `--user-data-dir` under system temp (`s2v-edge-*`) |
| URL form | `file:///C:/Genmitsu%20L8%20Tracker/index.html` (+ query) |
| `window.crypto` / `crypto.subtle` | **Available** |
| Ordinary startup | Loads; S1/S2 run counts **0**; deriveKey **0** (instrumented) |
| Export choice / encrypted primary / plain warning | **Pass** |
| Empty/mismatch export | Mismatch **pass**; empty via native `required` or fixture direct call |
| Cancel no download | **Pass** |
| Encrypted success toast + storage passphrase absence | **Pass** |
| Invalid encrypted import matrix | **Pass** (0 derive) |
| Focused S1 / S2 | **21/0**, **17/0** |
| Full suite completion | `failed: 0`; S1/S2 once each |
| Console errors / page errors / unhandled rejections (adversarial session) | **None recorded** |
| Known Designs `height="auto"` diagnostics | Not attributed to S2; not re-triaged as S2 regressions |

### Google Chrome

| Item | Value |
| --- | --- |
| Search | `%ProgramFiles%`, `%ProgramFiles(x86)%`, `%LocalAppData%\Google\Chrome\Application\chrome.exe`, `where chrome` |
| Result | **Not installed / not found** |
| Action | Not installed by verifier; **no Chrome results inferred** |

---

## 18. Performance / DoS boundary (this machine + Edge only)

| Observation | Result |
| --- | --- |
| Ordinary startup derivation | **0** |
| Open Export chooser derivation | **0** |
| Plain export/import derivation | **0** |
| Invalid envelope validation derivation | **0** (import matrix) |
| Deliberate encrypted export | **1** derive (+ AES encrypt) after successful probe |
| Capability probe on encrypted path | **1** derive + encrypt + decrypt |
| Iterations > 2000000 | Rejected pre-derivation (validator) |
| Concurrent duplicate export | Guarded |
| UI freeze | No obvious hang; probe/export complete in low hundreds of ms here |
| Timing generalization | **Forbidden** — values are local only |

S1 capability probe self-report sample: `pbkdf2AndAesRoundTripMs ≈ 92` on this host (fixture actual).

---

## 19. Protected-boundary confirmation

S2 does **not** encrypt live Tracker browser storage. Ciphertext exists only in the **downloaded encrypted backup file**. `STORAGE_KEY` contents remain plaintext workshop data. No vault unlock shell.

---

## 20. Findings by severity

### Critical

*None.*

### High

*None.*

### Medium

#### M1 — S2 fixtures omit Merge/Replace parity and import-path tamper matrix  
- **Location:** `runSecurityS2EncryptedBackupFixtures`  
- **Behavior:** Stops at opening import mode without mutation; does not apply Merge/Replace or compare to plaintext apply  
- **Evidence:** Fixture source; no parity assert in 17  
- **Impact:** Regression risk for apply wiring would be caught only by storage/import fixtures + manual use  
- **Smallest correction:** Add disposable parity asserts (encrypt → decrypt → merge/replace vs plaintext twin) and 1–2 import-path tamper cases  
- **Blocks S2 commit?** **No** (path reuses unchanged apply; isolation proven)  
- **Blocks S3?** **No** (vault-unrelated); recommended before trusting long-term backup UX  

#### M2 — 1 MiB not proven on S2 download/import UI boundary  
- **Location:** S2 fixtures / export download  
- **Evidence:** Only S1 helper 1 MiB assert  
- **Impact:** Theoretical large-file edge cases on download path untested at S2 layer  
- **Smallest correction:** One fixture using stub download with large `backupObject` payload  
- **Blocks S2 commit?** **No**  
- **Blocks S3?** **No**

### Low

#### L1 — UI copy omits a few exact checklist phrases  
- **Location:** `openBackupChoiceDialog` / plain warning  
- **Missing exact phrases:** “not password-protected”; “lost passphrase makes the file unusable”; plain warning “someone with file access may read the contents”  
- **Present:** readable/recommended/never saved/no recovery/plaintext browser storage remains  
- **Impact:** Clarity only  
- **Blocks S2 commit?** **No**

#### L2 — Implementation report suite total 3041 understates outer-return independent sum 3068  
- **Location:** implementation report validation table  
- **Evidence:** This pass’s 48-group outer-return sum  
- **Impact:** Reporting accuracy  
- **Blocks S2 commit?** **No**

### Informational

#### I1 — HTML5 `required` may handle empty export submit before custom empty message  
Fixtures prove JS empty path via direct function call.

#### I2 — Capability probe cost is full 600k PBKDF2 per encrypted-export entry  
By design; not on ordinary render.

#### I3 — Absence of live localStorage encryption  
**Out of S2 scope** (intentional). Not classified as defect.

#### I4 — T3A `897/b5c549ee` unregistered  
Report wording correct; not an S2 defect.

#### I5 — Chrome unverified  
Environment limitation.

---

## 21. Unverified behavior

- Chrome / non-Edge Chromium `file://` SubtleCrypto and download UX  
- Full interactive Merge then Replace **byte equality** encrypted vs plaintext on disposable state  
- Ciphertext/AAD tamper through full Import file picker UI after passphrase (S1 covers crypto; S2 import UI error path partially covered)  
- Photo-heavy multi-megabyte real workshop backups timing  
- Password-manager save prompts on real user profiles (intentionally not tested against Joe’s profile)  
- Node static parse of inline script (Node missing)  
- Non-headless focus-trap manual audit of backup dialogs  

---

## 22. Safe to commit?

**Yes — Security S2 is safe to commit** as the encrypted backup **file** Export/Import layer on top of frozen S1, with the cautions in the verdict (Chrome gap, suite total reconciliation note, fixture thinness, T3A non-registered pin).

No code correction is **required** to commit S2 for security isolation and core behavior verified here.

### Smallest correction boundary (optional, non-blocking)

1. Expand S2 fixtures: Merge/Replace parity + one import tamper + optional 1 MiB stub download.  
2. Optional UX copy polish for exact risk phrases.  
3. Correct implementation-report suite arithmetic to outer-return methodology if republishing totals.

### Exact safe S3 boundary (do not implement here)

S3 may introduce **local vault / encrypted-at-rest browser storage** only as a **separate** phase:

- Must not weaken S1 envelope or S2 file backup formats without versioned migration  
- Must not silently re-encrypt existing `STORAGE_KEY` without explicit enrollment UX  
- Must not retain passphrases across reloads without a deliberate session model  
- Must keep offline `file://` and zero-network constraints  
- Must leave plaintext export/import available as recovery until vault is proven  
- S2 encrypted **files** should remain importable after vault work  

---

## 23. Final verdict line

**2. VERIFIED WITH CAUTION — Safe to commit Security S2; Chrome unavailable; independent suite total 3068/0 across 48 groups (not 3041); S2 fixtures thin on Merge/Replace parity and 1 MiB UI; T3A 897/b5c549ee non-registered.**
