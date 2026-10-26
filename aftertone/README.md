# AFTERTONE — documentation index

AFTERTONE is a real-time, two-participant messaging application with a retro CRT-style interface. Two devices pair with a private invite code; message content is encrypted in the browser and relayed by a Node.js server that keeps all state in memory. The source code is in a separate repository. This folder holds its documentation.

## Contents

| Area | File | What it is |
|---|---|---|
| Architecture | [architecture/README.md](architecture/README.md) | Guide to the four Archify diagrams and the mapping brief |
| Security | [security/SECURITY.md](security/SECURITY.md) | Protocol, trust model, explicit limitations, vulnerability reporting |
| Policies | [policies/README.md](policies/README.md) | Public Terms, Privacy Notice, Security Notice, and FAQ (adapted from the live public pages; operator/governing-law placeholders still to complete) |
| Testing | [testing/DEVICE-TESTING-CHECKLIST.md](testing/DEVICE-TESTING-CHECKLIST.md) | Manual iOS/Android/Safari checklist; all items unexecuted |
| Reports | [reports/PROJECT-CHECKLIST.md](reports/PROJECT-CHECKLIST.md) | Master checklist with recorded automated-test results |
| Operations | [operations/DEPLOYMENT.md](operations/DEPLOYMENT.md) | Deployment preparation and reverse-proxy requirements |
| Operations | [operations/OPERATIONS.md](operations/OPERATIONS.md) | Runbook: deploy, rollback, recovery, smoke tests, monitoring |
| Reference | [project-readme-snapshot.md](project-readme-snapshot.md) | The source repository's own README, as received |
| Media | [../media-index.md](../media-index.md) | Demo video (not stored in Git) |

## State at the time of these snapshots (about 2026-10-25)

Taken from the documents in this folder, not independently re-verified:

- **Automated tests:** the checklist records the full suite as 236/236, run twice, plus browser 14/14, Linux WebKit 14/14, and regression 31/31. These are the author's recorded results.
- **Physical devices:** not tested. The device checklist records real Safari, physical iOS, and physical Android as blocked, with no item marked complete. Linux WebKit is not Safari.
- **Production:** preparation is on hold. The runbook says it is not a launch approval, and several deployment settings are marked as needing owner verification.
- **Security:** the key exchange is unauthenticated, so a malicious relay could substitute keys. Mandatory fingerprint comparison over an independent channel is the mitigation. The cryptography has not been independently audited. See [SECURITY.md](security/SECURITY.md).

## Inconsistencies found between the source documents

The copies are left untouched. These are recorded here instead.

1. **Test-count badge.** The README snapshot shows a "150 Passing" badge. The master checklist records 236/236. The badge appears to be out of date.
2. **Merge-conflict markers.** `project-readme-snapshot.md` contains unresolved `<<<<<<<`, `=======`, and `>>>>>>>` markers around the Installation and Run Locally section (lines 102–122), so the page renders with stray conflict text.
3. **Broken relative links inside copies.** Links such as `docs/OPERATIONS.md` and `SECURITY.md` inside the copied files use the source repository's layout and do not resolve here. Use the table above.
4. **Diagram age.** The Runtime Architecture, Message Flow, and Pair Lifecycle diagrams contain no key-confirmation or fingerprint-verification steps, while the security documents describe both. Only the remap diagram mentions them. See [architecture/README.md](architecture/README.md).
5. **Operations runbook scope.** The runbook says the working tree it names was not accessible when it was written, and that production deployment settings were not verified.

## Before making this repository public

Some copied files contain operational details: a private source-repository URL, a hosting hostname, a local filesystem path, and the owner's name in `OPERATIONS.md`. Review these before changing visibility.
