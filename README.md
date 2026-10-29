# AFTERTONE — Project Notes

> **The wall in front of the code.**
> This repository is the single place for AFTERTONE's documentation, status, bugs, and tracking. The source code lives in a separate, private repository and is never stored here.

AFTERTONE is a retro-pager-style, CRT-interface messaging app for exactly two people. Two devices pair with a private invite code, messages are encrypted in the browser, and the relay sees only ciphertext. No accounts. No database. No message history.

---

## Demo

<video src="assets/aftertone-demo.mp4" controls width="100%">
  Your browser does not support the video tag. The demo file is
  <a href="assets/aftertone-demo.mp4">assets/aftertone-demo.mp4</a>.
</video>

> The demo shows the retro terminal interface, pairing with a private invite code, and a message going from typed text to encrypted relay to printed delivery. It does not show the fingerprint VERIFY step — both sides must compare the fingerprint and press VERIFY before messaging is enabled.
>
> **Note:** the video file itself is uploaded separately via the GitHub web interface (binary files are not pushed with the text workflow). Until `assets/aftertone-demo.mp4` is uploaded, this player will not play. See [assets/README.md](assets/README.md).

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
| **Analytics** | Google Analytics 4, working via external `/ga.js` |
| **Search** | Google Search Console verified owner — **not yet indexed** |
| **Monitoring** | UptimeRobot monitor paused while the site is suspended |
| **Security audit** | Not independently audited — see [STATUS.md](aftertone/STATUS.md) |

Full, dated status: **[aftertone/STATUS.md](aftertone/STATUS.md)** — read that page first. This README's table is a summary of it, not a replacement.

---

## What we track here

This repository tracks only what the owner asked it to track:

| Track | File | What it holds |
|---|---|---|
| Project info & current status | [aftertone/STATUS.md](aftertone/STATUS.md) | What AFTERTONE is, where it stands today, what works, what is open, what is blocked |
| Bugs & open issues | [aftertone/BUGS.md](aftertone/BUGS.md) | Every known bug and loose thread, with evidence and status |
| Roadmap | [aftertone/ROADMAP.md](aftertone/ROADMAP.md) | What is next, what is parked, what will never be done |
| Decisions | [aftertone/DECISIONS.md](aftertone/DECISIONS.md) | Decisions that must not be re-asked, with dates |
| Changelog | [aftertone/CHANGELOG.md](aftertone/CHANGELOG.md) | What actually shipped, by PR and commit |
| Architecture | [aftertone/architecture/](aftertone/architecture/) | Diagrams and how the system fits together |

Bug reports use the templates in [.github/ISSUE_TEMPLATE/](.github/ISSUE_TEMPLATE/) so every report carries the same evidence: what happened, where, on which commit, and how to reproduce it.

---

## Architecture

The diagrams below are self-contained HTML exports. GitHub does not render them inline — download the file and open it in a browser.

| Diagram | File |
|---|---|
| System architecture | [aftertone/architecture/archify/aftertone-architecture.html](aftertone/architecture/archify/aftertone-architecture.html) |
| Pair lifecycle | [aftertone/architecture/archify/aftertone-pair-lifecycle.html](aftertone/architecture/archify/aftertone-pair-lifecycle.html) |
| Message flow | [aftertone/architecture/archify/message-flow.html](aftertone/architecture/archify/message-flow.html) |
| Remap / re-pairing | [aftertone/architecture/archify/aftertone-remap.html](aftertone/architecture/archify/aftertone-remap.html) |

How it works, in one paragraph: two devices pair with a private invite code, exchange keys bound to a single exchange ID, both sides compare a fingerprint over an independent channel and press VERIFY, and only then is messaging enabled. Messages are encrypted in the browser (P-256 ECDH, HKDF-SHA256, AES-256-GCM) and the server relays ciphertext only, keeping all state in memory.

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

This repository was rebuilt on 2026-10-29. The previous documentation set (25 Oct 2026 snapshots) was removed from the working tree and replaced with this tracking system. The old content remains in Git history at commit `41bf4e8` — it was not erased from history, and it is not current. Do not quote it as the project's status.
