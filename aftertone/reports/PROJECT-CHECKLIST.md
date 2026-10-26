# AFTERTONE — Master Project Checklist

> Status recorded: 2026-10-25. Automated results below are from the current
> working tree, not from historical runs. Production preparation remains on
> hold. No commit, push, pull, merge, or deployment was performed.

## 1. Mandatory Fingerprint Verification

- [x] Implement bilateral fingerprint verification.
- [x] Bind fingerprints to session context.
- [x] Gate outgoing and incoming application messages.
- [x] Coordinate verification between both participants.
- [x] Migrate existing tests to mandatory verification.
- [x] Complete verification test migration batches 1–4.
- [x] Verify rejection of forged, premature, stale, unsolicited, and duplicate verification-completion messages.
- [x] Verify synchronous trust invalidation when a new key exchange begins or session keys change.
- [x] Verify asynchronous decryption cannot deliver plaintext after session invalidation.
- [x] Verify server-side message-relay enforcement.
- [x] Verify exchange-bound verification reports and stale-report invalidation.
- [x] Verify transcript binding between fingerprints, relayed keys, and pair context.
- [x] Verify exchange rotation and per-member key storage.
- [x] Add HMAC-SHA256 key confirmation derived from the ECDH session.
- [x] Bind confirmation to exchange ID, sorted client IDs, sorted public keys, expiry, and sender.
- [x] Gate `crypto-ready` and local VERIFY on a valid peer key confirmation.
- [x] Require both participants' key confirmations before the server accepts a verification report.
- [x] Reject a replacement public key within one exchange.
- [x] Bind every application message to exchange, sender, recipient, and sequence.
- [x] Enforce strictly increasing per-sender sequences on receipt.
- [x] Reject duplicate, reordered, stale, and cross-session application messages.
- [x] Fix the re-pair race where an old peer's grace expiry could delete a newer pair.

## 2. Automated Testing & Reliability

- [x] Complete earlier test migration batches.
- [x] Pass earlier targeted migration suites.
- [x] Test forged, unsolicited, duplicate, and one-sided verification scenarios.
- [x] Test exchange rotation and transcript binding.
- [x] Test asynchronous decryption invalidation.
- [x] Test key-confirmation validation, forgery, and the server confirmation gate.
- [x] Test duplicate, reordered, stale, and cross-session application messages.
- [x] Test the authenticated message transcript and invalid contexts.
- [x] Test re-pairing while an old peer's grace timer is still pending.
- [x] Test republishing a key that the relay rejected during the peer's grace window.
- [x] Investigate timing-sensitive cross-session ciphertext failures.
- [x] Investigate the intermittent reconnect `crypto-ready` timeout.
- [x] Run targeted security and regression suites.
- [x] Run the full test suite twice after the final changes.
- [x] Rerun browser, WebKit, and regression suites.
- [x] Fix genuine test failures at their root cause.
- [x] Rerun affected suites after each fix.
- [x] Confirm exact final pass/fail counts from actual output.

### Verified results

| Suite | Result |
|---|---|
| Targeted security/lifecycle suites | 194 / 194 pass |
| Full suite, pass 1 | 236 / 236 pass |
| Full suite, pass 2 | 236 / 236 pass |
| `test/browser.test.js` | 14 / 14 pass |
| `test/webkit.test.js` | 14 / 14 pass |
| `test/regression.test.js` | 31 / 31 pass |

The timing-sensitive cross-session failure was traced to an unbound lifecycle
notice: an old pair's grace expiry could reset a newer session. Lifecycle
notices now carry the peer id they describe, and stale notices are ignored.

A second re-pair race surfaced while migrating the raw-client tests: an old
peer's grace expiry could delete the link of a newer pair. Pair detachment now
checks that the reciprocal link still points back, and accepting a new pair
first detaches any stale pair for either participant.

The intermittent reconnect `crypto-ready` timeout was traced to the peer grace
window: the first participant to reclaim its session published a fresh key
while the other was still disconnected, so the relay rejected it as
`not-connected` and the peer never received that key. When the peer returned,
the exchange could derive on one side only. A participant that already
reclaimed now republishes its own (identical, idempotent) public key when the
peer reconnects, so both sides finish the same exchange. A deterministic
regression test covers this ordering.

## 3. Final Security Assessment

- [x] Complete the initial security assessment.
- [x] Identify the initial verification and session-lifecycle weaknesses.
- [x] Validate the latest implementation against the identified attack scenarios.
- [x] Verify bilateral verification and message-gating invariants.
- [x] Assess key substitution and malicious-server scenarios.
- [x] Assess replay, stale reports, reconnect, and session expiry.
- [x] Assess ECDH, HKDF, AES-GCM, transcript binding, and key lifecycle.
- [x] Assess the HMAC-SHA256 key-confirmation design and its limits.
- [x] Assess authenticated message binding and per-sender sequence enforcement.
- [x] Document confirmed vulnerabilities, resolutions, and remaining limitations.
- [x] Complete the final security assessment based on the actual tested code.

**Trust limitation:** Mandatory fingerprint verification is not persistent
identity authentication. Key confirmation proves possession of the derived
session key, not the peer's identity, and a malicious relay that substitutes
both public keys knows both derived keys. First-contact MITM protection depends
on users comparing the full fingerprint through an independent trusted channel.
The implementation does not claim complete MITM resistance or authenticated key
exchange.

## 4. Browser & Regression Testing

- [x] Previous `test/browser.test.js` batch passed: 14/14 (historical).
- [x] Previous `test/webkit.test.js` batch passed: 14/14 (historical).
- [x] Previous `test/regression.test.js` batch passed: 30/30 (historical).
- [x] Rerun these suites against the latest implementation: 14/14, 14/14, 31/31.
- [x] Fix any failures and rerun affected suites. (No failures.)
- [x] Distinguish Linux WebKit testing from actual Safari testing.

Linux WebKit is not Safari on a physical iOS device. No physical Safari result
is claimed.

## 5. Server Hardening & Reliability

- [x] Existing production configuration validation implemented.
- [x] Existing Terms-secret validation implemented.
- [x] Existing origin-validation and server-hardening measures implemented.
- [x] Verify the latest server-side verification and relay authorization.
- [x] Verify relevant error paths and regression coverage.

## 6. Physical Device Testing

- [x] Prepare the iOS device-testing checklist.
- [x] Prepare the Android device-testing checklist.
- [ ] Test pairing and fingerprint comparison on physical iOS devices.
- [ ] Test pairing and fingerprint comparison on physical Android devices.
- [ ] Test mismatched fingerprints and blocked messaging.
- [ ] Test backgrounding, foregrounding, reconnect, and network recovery.
- [ ] Test expiry, re-pairing, and rejection of stale messages.
- [ ] Record actual results; leave unexecuted tests pending.

The manual checklist is in `test/DEVICE-TESTING-CHECKLIST.md`. None of its
physical-device items were executed as part of this work.

## 7. Documentation & Repository Hygiene

- [x] Verify `SECURITY.md` matches the implementation.
- [x] Verify README protocol notes.
- [x] Verify the device-testing checklist.
- [x] Update the master checklist from verified results.
- [x] Check relevant setup and local testing instructions.
- [x] Resolve relevant broken documentation links.
- [x] Check for unintended whitespace errors or unrelated changes.

`git diff --check` passes. Existing unrelated working-tree changes were
preserved.

## 8. Final Release Readiness

- [x] Confirm all applicable automated tests pass.
- [x] Confirm security fixes have regression coverage.
- [x] Confirm documentation matches the tested implementation.
- [x] Record outstanding physical-device or external-access requirements.
- [x] Complete the final security assessment.
- [x] Produce one final report with exact test results and limitations.

## 9. Production Preparation — ON HOLD

- [ ] Decide deployment architecture.
- [ ] Finalize Cloudflare, hostname, TLS, and WebSocket configuration.
- [ ] Configure production secrets and origin allowlisting.
- [ ] Decide shared-state requirements.
- [ ] Validate production configuration.
- [ ] Authorize and perform deployment.
- [ ] Verify the live production system.

**Strict restriction:** Production preparation and deployment remain on hold
and were not performed.

## Remaining limitations

- The key exchange is unauthenticated. A malicious relay can substitute keys
  and coordinate a separate exchange with each participant. Key confirmation
  proves possession of the derived key, not peer identity.
- Verification is a human out-of-band comparison, not persistent identity
  authentication. A malicious relay can deny service or misreport completion.
- No forward secrecy beyond discarding ephemeral session keys; no ratchet or
  in-session rekey.
- Application messages have authenticated exchange/sender/recipient binding and
  strictly increasing per-sender sequences, so duplicate, reordered, stale, and
  cross-session messages are rejected at the receiver. A relay can still
  suppress or delay traffic. The server's bounded in-memory duplicate-envelope
  window remains a secondary layer.
- Browser compromise, XSS, malicious extensions, and device compromise remain
  out of scope.
- Physical iOS and Android testing, including real Safari, is still pending.

## 10. Backups, Rollback, and Operations

> Added 2026-10-25. Section 10 did not exist in this checklist before this
> work; it was created here, not updated from a prior Section 10.
> Evidence is in `docs/OPERATIONS.md`, `scripts/smoke-test.mjs`, and
> `test/smoke-script.test.js`. Status vocabulary: **Complete** =
> documented/implemented and supported by evidence; **Partial** = some
> work exists but an important requirement remains; **Pending owner
> action** = requires dashboard access, credentials, a decision, or an
> authorisation unavailable to the agent. A documented procedure is
> not the same as a tested production recovery, so no production
> procedure below is marked Complete on the basis of documentation alone.
>
> Work was done in a repository copy, not in the
> `/home/e14station/Documents/aftertone` tree named in the task, which
> is not accessible from the environment used here. That tree's branch,
> status, and uncommitted changes were therefore not verified; see the
> source-recovery note in `docs/OPERATIONS.md` §4. No commit, push,
> deployment, rollback, or live production change was performed.

| # | Original Section 10 item | Status | Evidence / remaining requirement |
|---|---|---|---|
| 1 | Document the production deployment procedure. | **Partial** | `docs/OPERATIONS.md` §1–§2 document the procedure from code, `DEPLOYMENT.md`, and CI. Remaining: Render dashboard settings (service, branch, build/start, auto-deploy) are unverified — no `render.yaml` exists in the repo and no dashboard access was available. Section 9 production preparation remains on hold. |
| 2 | Establish a rollback procedure. | **Partial** | `docs/OPERATIONS.md` §3 documents last-known-good identification, dashboard redeploy / revert, and failure handling. Remaining: rollback capability itself is unverified and was not tested against live production (deliberately not performed). |
| 3 | Confirm source recovery and repository access. | **Partial** | Repository access confirmed: private `quietmade/aftertone`, branch `main`, HEAD `ce0ea5c54a7eff487426f95c572a254dd611e085` at the time of writing; recovery instructions and clone/test steps are in `docs/OPERATIONS.md` §4, and a clean file copy was fetched and tested locally. Remaining: a real `git clone` verification and the `/home/e14station/Documents/aftertone` tree check are pending owner action. No mirror was created. |
| 4 | Document recovery of production environment variables and secrets. | **Partial** | Full inventory verified from code (`server/config.js`, `server/terms.js`, `server/server.js`) and recovery/rotation procedure in `docs/OPERATIONS.md` §5, with no secret values recorded. Remaining: live Render environment values are unverified by design — confirming them is a dashboard check for the owner. |
| 5 | Confirm how to restore the service if deployment fails. | **Partial** | Restore procedure is in `docs/OPERATIONS.md` §3 (redeploy known-good commit, revert if needed, configuration-first diagnosis). Remaining: it has not been exercised against a live failed deployment. |
| 6 | Establish a practical production smoke-test procedure. | **Partial** | Procedure in `docs/OPERATIONS.md` §6; `scripts/smoke-test.mjs` (existing `ws` dependency only) passes 9/9 locally and is covered by `test/smoke-script.test.js` (2/2). Remaining: the production run against the live URL is pending owner action and was not executed here. |
| 7 | Decide whether production monitoring and alerting are required. | **Complete** | Decision in `docs/OPERATIONS.md` §7: recommended but optional at the current pre-production stage; minimum required before launch is an external `/health` check, deploy-failure notification, and host-log review, using free host-native capabilities first. Implementation (dashboard checker, notifications, log retention) remains pending owner action and is tracked separately from the decision itself. |
| 8 | Document known limitations and operational responsibilities. | **Complete** | Consolidated, code-verified limitations and the proposed owner-only responsibility model are in `docs/OPERATIONS.md` §8, cross-checked against `SECURITY.md` and this checklist's Remaining limitations. Owner confirmation of the responsibility model is still requested, but the documentation itself is complete. |

- [ ] 1. Document the production deployment procedure. — Partial, see table.
- [ ] 2. Establish a rollback procedure. — Partial, see table.
- [ ] 3. Confirm source recovery and repository access. — Partial, see table.
- [ ] 4. Document recovery of production environment variables and secrets. — Partial, see table.
- [ ] 5. Confirm how to restore the service if deployment fails. — Partial, see table.
- [ ] 6. Establish a practical production smoke-test procedure. — Partial, see table.
- [x] 7. Decide whether production monitoring and alerting are required. — Complete as a decision; implementation pending owner action.
- [x] 8. Document known limitations and operational responsibilities. — Complete as documentation; owner confirmation of the responsibility model requested.

**Checklist ticks the owner can safely apply now:** items 7 and 8 only,
as marked above. Items 1–6 must stay unchecked until their remaining
requirements in the table are met and evidenced; in particular, no
item that depends on a live Render deploy, rollback, or production
smoke run can be ticked before that live step is performed and recorded.
