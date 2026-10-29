# AFTERTONE — Project Notes

> **The wall in front of the code.**
> This repository is the single place for AFTERTONE's documentation, status, bugs, and tracking. The source code lives in a separate, private repository and is never stored here.

AFTERTONE is a retro-pager-style, CRT-interface messaging app for exactly two people. Two devices pair with a private invite code, messages are encrypted in the browser, and the relay sees only ciphertext. No accounts. No database. No message history.

---

## Current status — at a glance

| | |
|---|---|
| **Project** | AFTERTONE — two-person secure messaging |
| **Code repository** | Private, separate repository — not here, by design |
| **Live site** | https://beeperline.onrender.com — **currently SUSPENDED, not deleted** |
| **Code on main** | `29121198` (PR #31, Search Console verification tag) |
| **Tests on main** | CI #69 green |
| **Production** | **NOT READY FOR RELEASE** — deployment is not a production approval |
| **Security audit** | Not independently audited — see [STATUS.md](aftertone/STATUS.md) |

Full, dated status: **[aftertone/STATUS.md](aftertone/STATUS.md)** — read that page first. This README's table is a summary of it, not a replacement.

---

## What we track here

This repository tracks only three things:

| Track | File | What it holds |
|---|---|---|
| Project info & current status | [aftertone/STATUS.md](aftertone/STATUS.md) | What AFTERTONE is, where it stands today, what works, what is open, what is blocked |
| Bugs & open issues | [aftertone/BUGS.md](aftertone/BUGS.md) | Every known bug and loose thread, with evidence and status |
| Changelog | [aftertone/CHANGELOG.md](aftertone/CHANGELOG.md) | What actually shipped, by PR and commit |

Architecture notes live in [aftertone/architecture/README.md](aftertone/architecture/README.md). Bug reports use the template in [.github/ISSUE_TEMPLATE/bug-report.md](.github/ISSUE_TEMPLATE/bug-report.md) so every report carries the same evidence: what happened, where, on which commit, and how to reproduce it.

---

## How it works, in one paragraph

Two devices pair with a private invite code, exchange keys bound to a single exchange ID, both sides compare a fingerprint over an independent channel and press VERIFY, and only then is messaging enabled. Messages are encrypted in the browser (P-256 ECDH, HKDF-SHA256, AES-256-GCM) and the server relays ciphertext only, keeping all state in memory.

---

## What this repository is — and is not

| This repo is | This repo is not |
|---|---|
| Documentation, status, bugs, and tracking for AFTERTONE | A source-code repository |
| A paper trail: what was built, how it works, what was tested, what is still unknown | A launch approval, a security audit, or a production guarantee |
| The wall in front of the private code repository | A place for secrets, credentials, tokens, or invite codes |

**Never commit here:** passwords, API keys, session tokens, `TERMS_SECRET` or any other secret, private keys, invite codes, or message content. If a secret ever lands in this repo, rotate it — do not just delete the file.

---

## How status is kept honest

1. **One status page.** [STATUS.md](aftertone/STATUS.md) is the current truth. Older status is history in [CHANGELOG.md](aftertone/CHANGELOG.md), not edited in place.
2. **Evidence apart from conclusions.** A claim cites a commit, a CI run, or a live check — "green" names the run, "verified" names the check.
3. **Commit, merge, and release are three separate decisions.** A commit is not merge approval. A merged PR is not production readiness. Nothing here is a release approval.
4. **A blocked test is never a pass.** Cancelled, skipped, or environment-blocked tests are reported as such, never counted as green.

---

## Resume checklist (when the site comes back)

The site is suspended, not deleted. To resume, in order:

1. Render: Resume the `beeperline` service
2. Check `/health` returns `{"ok":true}` and the homepage loads
3. UptimeRobot: unpause the `beeperline` monitor
4. Recheck Google Analytics Realtime and Search Console status
5. Update [STATUS.md](aftertone/STATUS.md) with the resume date and the new checks

---

## Repository history

This repository was emptied at owner request on 2026-10-29 and this revised file system was then added to the empty repository. Earlier content (the 25 Oct documentation set, and the first rebuilt tracking system) remains only in Git history (`41bf4e8`, `ea7b2e07`) — it is not current. Do not quote it as the project's status. The first release, `v1.0.0`, is a status snapshot from the empty-tree period.
