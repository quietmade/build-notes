# AFTERTONE — Changelog

> What actually shipped, in reverse chronological order. Each entry names the PR, commit, or live check that proves it. Status changes that are not shipments (suspensions, pauses) are recorded here too, because they change what "current" means.

## 2026-10-29

- **This repository rebuilt.** Previous 25 Oct documentation set removed from the working tree; new tracking system (STATUS, BUGS, ROADMAP, DECISIONS, CHANGELOG, professional README, re-added Archify diagrams) created on branch `rebuild/build-notes-2026-10-29`. Old content remains in Git history at `41bf4e8`.

## 2026-10-28

- **Site suspended.** Render service `beeperline` suspended by the owner ~18:54 IST. Verified "Suspended by you" with a Resume option; `/health` shows "Service Suspended". Not deleted; no code, env, or deploy changes.
- **UptimeRobot paused.** Monitor `beeperline` (`/health` every 5 min) paused ~20:30 IST after reporting down from 18:54:58 IST. Paused, not deleted; Resume button ready.
- **PR #31 merged** (`29121198`): Google Search Console verification meta tag in `client/index.html` only. Search Console: verified owner via HTML tag. Sitemap submitted (detail: 5 pages processed; list view: "Couldn't fetch" — conflicting, see B-04). Homepage: "Discovered, currently not indexed"; manual Request Indexing rejected (Google quota). Owner chose sitemap crawl only.
- **CI #69** (main `2912119`): SUCCESS, 11m34s. CI #68 (PR #31 branch `145b29a0`): FAILURE, cause unchecked (B-03).
- **PR #30 merged** (`51708747`): docs update for beeperline/GA/AdSense-hold status (README, DEPLOYMENT, OPERATIONS, HOSTING-HOSTNAME-DECISION, PROJECT-CHECKLIST header). Branch CI #66 green 297/297. Post-merge main CI #67: attempt #1 failed 296/297 (graceful-shutdown drain flake, B-01); attempt #2 rerun passed 297/297.
- **PR #29 merged** (`45d04314`): Google Analytics fix — inline gtag config externalised to `client/ga.js`. Branch CI #64 and main CI #65 both 297/297. GA Realtime working (26 users / 63 views, 2026-10-28 ~16:12 IST).
- **PR #28 merged** (`56a022d7`): Google Analytics install (gtag snippet + CSP hosts). Merged before its branch CI result was seen; branch CI #62 and main CI #63 both failed 292/297 — the inline config script was blocked by CSP `script-src 'self'`. Root cause of the GA "No data received" state before PR #29.
- **PR #27 merged** (`f0d43508`): SEO for beeperline — sitemap with 5 URLs, robots advertising it, canonical/`og:url` on 6 pages moved from dead `aftertone-u6rr` to beeperline. Branch CI #60 passed; main CI #61 failed, cause unchecked (B-02).
- **AdSense decision:** WAIT AND BUILD CONTENT FIRST. No application, no ads.txt, no CSP change.
- **UptimeRobot set up:** `beeperline` `/health` every 5 minutes, free plan, status Up at setup.
- **Fresh deployment:** Render service `beeperline` created (Free, Oregon, main `15063b9`); first deploy failed on missing `ALLOWED_ORIGINS`, fixed, redeploy succeeded in 35.8s; `/health` ok, homepage 200.
- **Full reset:** PNTR `aftertone.pntr.dev`, Cloudflare Worker `aftertone-gateway`, and old Render service `aftertone` all deleted at owner request. Old `TERMS_SECRET` lost (never recorded). Code repo untouched.
- **Earlier live proof:** Full two-client check on `aftertone.pntr.dev` passed (WebSocket, Terms, pairing, key exchange, bilateral fingerprint verification, encrypted relay; peer-side decrypt skipped).

## 2026-10-27

- **PR #25 merged** (`882038b7`): XSS step 2 — 1,603-line inline script moved byte-identically to `client/app.js`; CSP `script-src 'self'`. CI #53 green 276/276.
- **PR #26 merged** (`342802c7`): D2 sign-off document (Illyon-only review, not a ChatGPT countersign, not an audit).
- **PR #20 merged** (`5cbb3a6`): Terms/Privacy placeholders applied — operator `quietmade`, contact `invasiveminds.tech@gmail.com`, location India, law Laws of India, no placeholders left.
- **Main `15063b9`** (2026-10-27 20:30 IST): baseline for the 2026-10-28 deployment work. CI #59 on it: 297/297.

## 2026-10-26

- **This repository created** (`quietmade/build-notes`, private, manual creation by owner). Draft pushed as `41bf4e8` (20 text files): architecture, security, policies, testing, reports, operations snapshots dated ~25 Oct 2026. Demo video `assets/aftertone-demo.mp4` was never pushed (binary) and remained a pending manual upload.

## 2026-10-25

- **Abuse tracking** committed to the code repo (`feat/abuse-tracking`, `15ec113`), disabled by default. Deferred for later planning; not to be implemented by this assistant in the meantime (owner decision at the time).
- **D2 scope:** server-snapshotted expiry (Option B), finite/live validation, extension non-mutation, race windows, pending-key replacement — later signed off 2026-10-27 (Illyon-only).
