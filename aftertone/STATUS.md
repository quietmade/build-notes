# AFTERTONE — Current Status

> **Last updated: 2026-10-29. This page is the current truth.**
> Older status lives in [CHANGELOG.md](CHANGELOG.md). If this page and any other file disagree, this page wins and the other file is stale.

## What AFTERTONE is

A retro-pager-style, CRT-interface messaging app for exactly two people. Two devices pair with a private invite code, exchange keys bound to a single exchange ID, both sides compare a fingerprint over an independent channel and press VERIFY, and only then is messaging enabled. Messages are encrypted in the browser (P-256 ECDH, HKDF-SHA256, AES-256-GCM); the relay sees only ciphertext (`{ iv, ciphertext }`). No accounts, no database, no message history. All server state is in memory.

The source code is in a private repository (`quietmade/aftertone`), separate from this documentation repository, by design. No code is stored here.

## Live state — SUSPENDED, not deleted

| Item | State (2026-10-29) |
|---|---|
| Live site | https://beeperline.onrender.com — **SUSPENDED by the owner on 2026-10-28 ~18:54 IST** |
| Render service | `beeperline` (`srv-dbgrkt8u01pc73au0vcg`), Free, Oregon, branch `main`, auto-deploy on commit |
| Render env | `NODE_ENV=production`, `NODE_VERSION=22`, `ALLOWED_ORIGINS=https://beeperline.onrender.com` exactly, Render-generated `TERMS_SECRET` (value never recorded, never written here) |
| `/health` while suspended | Render "Service Suspended" page (expected — not an outage, not a deletion) |
| Custom domain | None. No PNTR subdomain, no Cloudflare Worker for this deployment |
| Gateway-secret code | Local only, never deployed, on hold (direct-to-Render route adds no gateway header) |
| Monitoring | UptimeRobot `beeperline` monitor (`/health` every 5 min) — **PAUSED 2026-10-28 ~20:30 IST, not deleted**. Unpause on resume |
| Old hosts | Deleted 2026-10-28: old Render service `aftertone`, Cloudflare Worker `aftertone-gateway`, PNTR `aftertone.pntr.dev`. Their `TERMS_SECRET` was lost with the old service |

## Code state

| Item | State |
|---|---|
| Main HEAD | `29121198` (PR #31 merge, 2026-10-28) |
| Last change on main | Search Console verification meta tag in `client/index.html` only — no app code, no CSP change |
| CI on main | #69 (main `2912119`) **SUCCESS**, 11m34s |
| Test suite | 297 tests, 54 suites (Chromium + WebKit in CI) |
| Open PRs | 0 |

## What works (proven, with evidence)

- Pairing, key exchange bound to one exchange ID, bilateral fingerprint verification, and encrypted relay — proven live on the earlier `aftertone.pntr.dev` deployment (full two-client check). A fresh full two-client proof on `beeperline.onrender.com` itself has **not** yet been performed; peer-side decrypt was skipped in the earlier live script.
- Google Analytics 4 — working via external `/ga.js` after the PR #29 CSP fix. Realtime showed 26 active users / 63 page views on 2026-10-28 ~16:12 IST. GA Home / data-stream status badges lagged behind Realtime at that time.
- Google Search Console — URL-prefix property `https://beeperline.onrender.com` verified via HTML tag (PR #31). Sitemap submitted 2026-10-28; detail view said processed successfully with 5 discovered pages, while the list view said "Couldn't fetch" / 0 pages — **status conflicting, recheck needed**.
- SEO basics — sitemap with 5 beeperline URLs, robots advertising it, canonical/`og:url` on 6 pages pointing at beeperline (PR #27).

## What is NOT done — do not overclaim

- **The site is NOT indexed and NOT ranked.** URL Inspection (2026-10-28): "not on Google — Discovered, currently not indexed". The manual Request Indexing attempt was rejected by Google ("Quota Exceeded — try again tomorrow"). The owner chose on 2026-10-28 to leave indexing to Google's own sitemap crawl; **no manual indexing retry is scheduled**.
- **NOT READY FOR RELEASE as a production approval.** Being deployed on Render Free, monitored, GA-tracked, and CI-green does not make it production-approved.
- **No independent security audit.** D2 sign-off is Illyon-only (2026-10-27), not a ChatGPT countersign and not an audit.
- **Physical-device evidence is incomplete.** The device checklist still has 156 items open; owner-confirmed testing is claimed in older checklists without detailed per-item evidence recorded here. Real Safari, physical iOS, and physical Android proof are not independently established.
- **Open security decisions:** F-02, F-03, F-04; the MITM / unauthenticated-exchange model; forward secrecy; live decryption of a relayed message (not independently proven).

## Tracking — the open threads

See [BUGS.md](BUGS.md) for the full list. In short: the graceful-shutdown drain-test flake, the unchecked CI #61 and CI #68 failure causes, the Search Console sitemap status conflict, and the device-checklist gap are all still open. Secret-scan status is unconfirmed for several recent CI runs — never assume it passed when it was not shown.

## Parked (explicitly, not forgotten)

- AdSense: **WAIT AND BUILD CONTENT FIRST** (2026-10-28). No application, no ads.txt, no CSP change for ads. Future path is a separate content site on a custom domain with original articles, Search Console indexing, and real traffic first.
- PNTR custom domain for beeperline (`beeperline.pntr.dev`): not authorised. Order if ever done: Render custom domain first, then PNTR CNAME, verify domain/certificate, keep both origins in `ALLOWED_ORIGINS`, redeploy, verify live, then update sitemap/robots/canonicals.
- Content building for AdSense / ranking: not started, not to be started unprompted.
- Oracle Always Free hosting: ruled out (requires a credit card the owner does not have).
- Gateway-secret code: local only, on hold.

## Resume checklist

1. Render: Resume the `beeperline` service
2. Check `/health` returns `{"ok":true}` and the homepage loads
3. UptimeRobot: unpause the `beeperline` monitor
4. Recheck Google Analytics Realtime and Search Console status (including the sitemap conflict and indexing state)
5. Update this page with the resume date and the new checks
