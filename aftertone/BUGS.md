# AFTERTONE — Bugs & Open Issues

> **Last updated: 2026-10-29.**
> Status words: `Open` = confirmed, not fixed. `Flaky` = failed once, passed on rerun, root cause not proven. `Unverified` = claimed but not independently checked here. A closed issue names the commit or check that closed it — never just "fixed".

New bugs use the templates in [.github/ISSUE_TEMPLATE/](../.github/ISSUE_TEMPLATE/) and get an ID below.

## Open

### B-01 — Graceful-shutdown `/ready` drain test is flaky
- **Status:** Flaky / Open (no code fix investigated or authorised)
- **Evidence:** CI #67 attempt #1 (main `51708747`, 2026-10-28) failed 296/297. Only failure: `test/graceful-shutdown.test.js` — "/ready reflects accepting work, then turns 503 during drain" (expected 503, got 200). Rerun attempt #2 on the same run passed 297/297, including this test. Branch CI #66 and main CI #65 also passed it.
- **Interpretation:** Drain timing race, not a regression — PR #30 was docs-only and cannot have caused it. That interpretation is not a root-cause proof.
- **Next step:** Investigate the test's timing on a branch if the owner authorises it. Do not rerun CI blindly to make it green.

### B-02 — CI #61 failure cause unchecked
- **Status:** Open / Unverified
- **Evidence:** CI #61 (PR #27 SEO merge to main `f0d4350`) FAILED, while its branch CI #60 PASSED. The failing test or log cause was never checked.
- **Next step:** Read the #61 logs and name the failing test before claiming any pattern.

### B-03 — CI #68 failure cause unchecked
- **Status:** Open / Unverified
- **Evidence:** CI #68 (PR #31 branch `145b29a0`) FAILED (Test suite exit 1, no failing test name shown on the run page), while post-merge main CI #69 on `2912119` PASSED. Branch-vs-main difference suggests flaky, not proven.
- **Next step:** Read the #68 logs and name the failing test. Do not assert it is the same flake as B-01 without log evidence.

### B-04 — Search Console sitemap status conflicts with itself
- **Status:** Open
- **Evidence:** Sitemap `https://beeperline.onrender.com/sitemap.xml` submitted 2026-10-28. Submission confirmed, detail view said "processed successfully", 5 discovered pages. The submitted-sitemaps list view said "Couldn't fetch", 0 pages, last read blank.
- **Next step:** Recheck Search Console after the site resumes; report one consistent status. The site is suspended, so this cannot be settled while it is down.

### B-05 — Site not indexed, manual indexing request rejected
- **Status:** Open, by owner decision (not a code bug)
- **Evidence:** URL Inspection 2026-10-28: homepage "not on Google — Discovered, currently not indexed". Request Indexing returned Google's "Quota Exceeded — try again tomorrow". Owner chose 2026-10-28 to leave indexing to Google's own sitemap crawl; no manual retry is scheduled.
- **Next step:** None until the owner asks. Check `site:beeperline.onrender.com` after resume; if still unindexed after a reasonable crawl window, retry Request Indexing then.

### B-06 — Physical-device testing evidence incomplete
- **Status:** Open
- **Evidence:** The device checklist still has 156 items open. Older checklists claim owner-confirmed testing without detailed per-item evidence recorded. Real Safari, physical iOS, and physical Android proof are not independently established. Linux WebKit is not Safari.
- **Next step:** Per-item evidence, or an explicit owner statement of what was actually tested on which device. Do not convert the checklist claim into a pass.

### B-07 — AFTERTONE README "Current main" line is stale
- **Status:** Open (docs-only, low priority)
- **Evidence:** After PR #30 merged, the README said "Current main: `45d04314`" while main had moved to `51708747`. PR #31 has since moved main to `29121198`, so the line is now two commits behind.
- **Next step:** Fix in the next docs update. Not worth a PR on its own.

### B-08 — Secret-scan status unconfirmed for several CI runs
- **Status:** Open / Unverified (applies per run, not as a blanket claim)
- **Evidence:** Run pages for several recent runs (including #57–#59 historically, #64, #67 attempt #2) did not show a secret-scan job or status. Other runs did report it passing. `docs/OPERATIONS.md` in the code repo claims CI does not run a secret scan, contradicting runs that reported one — that contradiction is itself unresolved.
- **Next step:** Name the run before claiming its secret-scan status. Never assume a pass that was not shown.

## Closed

_None yet in this tracking system. When an issue closes, move it here with the closing evidence (commit, CI run, or live check)._
