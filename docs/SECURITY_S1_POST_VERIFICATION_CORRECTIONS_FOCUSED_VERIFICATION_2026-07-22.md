# Security S1 Post-Verification Corrections — Focused Verification

**Date:** 2026-07-22  
**Repository:** `C:\Genmitsu L8 Tracker`  
**Scope:** Only the two post-audit corrections (awaited aggregate self-test; own-property encrypted detection).  
**Mode:** Read-only. Only this report was created.

---

## Exact final verdict

**VERIFIED — Security S1 is safe to commit.**

Both corrections behave as required under direct-file Edge execution. Remaining non-blockers (Chrome not installed; rejection path proven by source inspection rather than force-injected product run) do not prevent commit.

---

## 1. Actual repository state

| Check | Result |
|-------|--------|
| HEAD | `19d4834` — Graduate tabletop storage tray to Workshop-ready |
| Full hash | `19d48340a293184c748c2767ec8fd0654f7ce376` |
| Branch | `main` |
| Upstream | `0 0` vs `origin/main` |
| Staged | Empty |
| Tracked working tree | `M index.html` only (full uncommitted S1 + corrections) |
| `git diff --stat` (vs HEAD) | `index.html` \| 338 lines (+292 / −46) |
| Untracked (security-related) | Architecture, S1 implementation, S1 post-impl verification, S1 corrections report |
| Other untracked | Historical docs, LightBurn, `debug.log`, utility — preserved |
| `git diff --check` | Clean |
| `python -m html.parser index.html` | exit 0 |

S1 remains **uncommitted**. Nothing staged.

---

## 2. Exact diff inspected

Relative to committed baseline `19d4834`, the tracked change is still the S1 crypto infrastructure **plus** the two focused corrections:

1. **`runRequestedSelftests` / `selftestCompletionPromise`** — async dispatcher that **awaits** S1 before Designs and before completion resolution.
2. **`isEncryptedBackupEnvelope`** — requires `Object.prototype.hasOwnProperty.call(value,'backupFormat')` and exact encrypted format marker.
3. Fixture assertion renamed/strengthened for own-property detection; `securityS1CryptoFixtureRunCount` increment.

No Export/Import/vault/UI/S2 changes.

---

## 3. Async dispatcher verification (source)

```text
async function runRequestedSelftests(selftest) {
  … synchronous collect(...) for all non-S1 groups …
  if (selftest === 'security-s1-crypto' || selftest === 'all') {
    try {
      window.securityS1CryptoFixturePromise = runSecurityS1CryptoFixtures();
      const securityResult = await window.securityS1CryptoFixturePromise;
      window.securityS1CryptoFixturePromise = Promise.resolve(securityResult);
      collect(securityResult);  // failed += securityResult.failed
      …
    } catch (error) {
      const securityResult = { passed:0, failed:1, results:[…] };
      … collect(securityResult);
    }
  }
  if (selftest === 'design' || … || selftest === 'all')
    collect(runDesignGeometryFixtures());
  return { selftest, completed:true, failed };
}
if (selftest)
  window.selftestCompletionPromise = runRequestedSelftests(selftest).catch(…);
```

| Requirement | Source |
|-------------|--------|
| S1 starts exactly once per dispatcher run | Single `if` branch; one call to `runSecurityS1CryptoFixtures()` |
| Awaits actual S1 promise | `await window.securityS1CryptoFixturePromise` |
| Completion after S1 | Designs runs **after** await; `return` only then |
| S1 failures in aggregate | `collect(securityResult)` adds `securityResult.failed` |
| Rejection → reported failure | `catch` builds `{passed:0,failed:1}` and `collect`s it; outer `.catch` on completion promise |
| No timer/polling workaround | Pure `async`/`await` |
| No selftest query → no S1 | `if (selftest)` guard only |

---

## 4. Proof that final completion awaits S1 (executed)

**Browser:** Microsoft Edge **150.0.4078.83**, disposable profile, headless CDP, `file://` temp copy of current `index.html`.

### Early snapshot (`?selftest=all`, ~50 ms after load)

| Field | Value |
|-------|--------|
| `hasPromise` | `true` |
| `settled` (`__S1_CORR_SETTLED`) | **`false`** |
| `securityS1CryptoFixtureRunCount` | **1** (S1 already started) |

### After `await selftestCompletionPromise`

| Field | Value |
|-------|--------|
| `settledAtFirstMicrotask` | **`false`** (not resolved on first `await Promise.resolve()`) |
| `completion.completed` | `true` |
| `completion.failed` | **0** |
| `completion.selftest` | `"all"` |
| S1 result | **21 passed / 0 failed** |
| `securityS1CryptoFixtureRunCount` | **1** |
| Unhandled rejection marker | none |

**Conclusion:** Aggregate completion is **not** reported at the first microtask while S1 is pending; it resolves only after S1 finishes. S1 contributes its failure count (here 0) to `completion.failed`.

---

## 5. Proof that S1 executes once

| Route | `securityS1CryptoFixtureRunCount` |
|-------|-------------------------------------|
| `?selftest=security-s1-crypto` | **1** |
| `?selftest=all` | **1** |
| Ordinary startup (no query) | **0** |

---

## 6. Rejection-path result

| Method | Result |
|--------|--------|
| **Source inspection** | Explicit `try/catch` around S1 await; rejection → `failed:1` synthetic result + `collect` + `console.error`; no rethrow that would leave `selftestCompletionPromise` hanging as an unhandled rejection without the outer `.catch` |
| **Force-injected rejected fixture in product browser** | **Not executed** — would require mutating the live fixture or non-exported internals; not done (read-only) |

Limitation: rejection conversion is **source-proven**, not runtime force-injected. Behavior under happy path and outer dispatcher `.catch` are runtime-proven.

---

## 7. Own-property detection results

Source:

```js
return encryptedBackupIsPlainObject(value)
  && Object.prototype.hasOwnProperty.call(value, 'backupFormat')
  && value.backupFormat === ENCRYPTED_BACKUP_FORMAT;
```

Uses **`Object.prototype.hasOwnProperty.call`**, not `value.hasOwnProperty`, so an overridden instance method cannot spoof ownership.

### Independent Edge matrix

| Case | Result |
|------|--------|
| Own exact marker | **true** |
| Inherited exact marker (`Object.create`) | **false** |
| `envelopeVersion` alone | false |
| `crypto` / `ciphertextB64` alone | false |
| Prefix / suffix / uppercase | false |
| Plaintext `BACKUP_FORMAT` | false |
| null / [] / string / number | false |
| Own marker only (malformed) | **detected true**, **validation fails** (`ownButInvalid`) |
| `Object.create(null)` + own `backupFormat` | true |

Detection remains distinct from `validateEncryptedBackupEnvelopeV1` (strict exact-key rules unchanged).

S1 fixture assertion **“encrypted detection requires an exact own backupFormat marker”** passed (includes inherited-false and own-marker-but-invalid-validation).

---

## 8. Focused fixture result

`?selftest=security-s1-crypto` via app dispatcher:

- **Security S1 Crypto: 21 passed / 0 failed**
- `selftestCompletionPromise`: `{ completed: true, failed: 0 }`
- localStorage keys after focused route: **[]** (no residue from S1)
- PBKDF2+AES capability ~**89 ms** in fixture actual field

---

## 9. Full unique-suite result

Application-owned `?selftest=all`:

- Completion: **`completed: true`, `failed: 0`**
- S1 within that run: **21 / 0**, run count **1**
- Designs geometry (separate corroborating run): **1093 / 0**
- Registered production pins re-measured: T1 `2992/4f543f95`, T2 `2337/ed5d6f6e`, T3A `897/b5c549ee`, T3B `2860/3e256fad`, Dice `1726/51a55721`, wall-base `1551/d9ffc278`

Assertion total for unique groups remains **3024** (= prior 3003 + 21 S1) with **47** groups; the application aggregate reports **zero group failures** after awaiting S1. No re-enumeration of every group’s pass count was repeated in this narrow pass beyond the app’s own failure aggregate + S1/design corroboration (consistent with full prior 3024/0 measurement and zero `completion.failed`).

**Note:** After `?selftest=all`, disposable profile had `localStorage['genmitsu-l8-tracker-v1']` written by **other** fixture groups (not S1 vault). Expected for full suite; not an S1 storage artifact.

---

## 10. Edge direct-file result

| Item | Value |
|------|--------|
| Browser | Edge 150.0.4078.83 |
| Executable | `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` |
| Profile | Disposable headless user-data-dir only |
| Focused S1 | 21/0 |
| Aggregate awaits S1 | Yes (`settledAtFirstMicrotask: false`) |
| Aggregate completion | failed: 0 |
| S1 run count under `all` | 1 |
| Unhandled rejection | None observed |
| S1 console/page errors | None observed |
| Ordinary startup | `selftestCompletionPromise` absent; S1 run count 0; **deriveKey count 0** |
| Chrome | Not installed (not run) |

Preexisting Designs preview SVG `height="auto"` diagnostics were not re-litigated; none appeared on S1-only routes. Full suite may still emit them from Designs Finished View paths; they remain outside this correction.

---

## 11. Console, page-error, and storage results

| Area | Result |
|------|--------|
| S1-related console/page errors | None in focused/aggregate probes |
| S1 localStorage residue | None on focused route |
| Ordinary startup deriveKey | **0** |

---

## 12. Protected-boundary confirmation

Corrections did **not** alter (source review of correction-focused logic + constants still present):

- Crypto algorithms/params (600000–2000000, AES-256-GCM, PBKDF2-SHA256, 16/12 byte salt/IV)
- Envelope fields / AAD field list
- Passphrase non-trim / non-normalize handling
- Base64 helpers
- Export / Import / `backupObject` / plaintext `BACKUP_FORMAT`
- `STORAGE_KEY`, `SCHEMA_VERSION`, `APP_*`
- No vault / passphrase UI / S2 wiring

---

## 13. Findings and limitations

| Item | Severity | Notes |
|------|----------|--------|
| Corrections correct prior Medium (await) and Low (own-property) issues | Resolved | Executed proof |
| S1 rejection `catch` not force-injected in browser | Limitation | Source-proven only |
| Chrome file:// not re-tested | Limitation | Chrome absent |
| Aggregate pass total inferred from `failed:0` + prior 3024 accounting | Limitation | Acceptable for narrow correction scope |

No blockers for commit.

---

## 14. Whether the complete S1 change is safe to commit

**Yes.** Full uncommitted S1 (crypto foundation + these two corrections) is safe to commit: Export/Import remain plaintext, no vault, fixtures green, aggregate self-test waits for S1, detection requires own exact marker.

---

## Hygiene

- Created only this verification report.  
- Did not modify `index.html` or existing reports.  
- Did not stage/commit/push/reset/clean/stash.  
- Did not access real browser profiles or records.  
- Disposable Edge profiles only.

---

*End of focused verification.*
