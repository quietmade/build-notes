# AFTERTONE — Bugs

A bug closes only when its closing evidence is named (commit / CI run on main).
A bug without evidence stays Unverified.

## Open

- **B-01 — Graceful-shutdown flake:** `test/graceful-shutdown.test.js` "/ready turns 503 during drain" failed once (CI #67 attempt #1, expected 503 got 200), passed on rerun. Cause not proven.
- **B-02 — CI #61 cause unknown:** PR #27 SEO merge to main failed while branch #60 passed. Failing test not checked.
- **B-03 — CI #68 cause unknown:** PR #31 branch run failed, main #69 passed. Failing test not checked.
- **B-05 — Not indexed:** homepage "Discovered, currently not indexed". Owner decision: leave to Google's sitemap crawl, no manual retry.
- **B-06 — Device evidence gap:** 156 physical-device items open; owner-confirmed testing claimed without per-item evidence.
- **B-07 — Stale README line:** code-repo README "Current main" is behind main. Docs only.
- **B-08 — Secret-scan unconfirmed:** status not shown for several recent CI runs. Never assume passed.

## Status words

Open / In progress / Flaky / Fixed / Verified on main / Unverified
