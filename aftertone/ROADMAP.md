# AFTERTONE — Roadmap

> **Last updated: 2026-10-29.**
> "Parked" means an explicit owner decision to wait, not something forgotten. "Never" means ruled out unless the owner reverses it with new facts.

## Now (when the site resumes)

1. Resume the site — Render Resume, `/health` check, unpause UptimeRobot, recheck GA and Search Console (see [STATUS.md](STATUS.md))
2. Fresh full two-client protocol proof on `beeperline.onrender.com` itself — homepage, `/health`, `/ready`, WebSocket, Terms, invite, pair, key exchange, bilateral fingerprint verification, encrypted relay **including peer-side decrypt** (skipped in the earlier live proof)
3. Recheck the Search Console sitemap conflict (B-04) and indexing state (B-05)

## Next (needs owner approval each time)

- Investigate the CI failure causes: B-02 (#61) and B-03 (#68) logs, and the B-01 graceful-shutdown flake — on branches, with evidence, not blind reruns
- Resolve the secret-scan contradiction (B-08): what CI actually runs vs what the docs claim
- Independent security audit; close or formally accept F-02, F-03, F-04 and the MITM / forward-secrecy questions
- Physical-device evidence for the 156-item checklist (B-06)

## Parked

- **Custom domain** (`beeperline.pntr.dev` or any other): order is Render custom domain first, then PNTR CNAME, verify domain/certificate, keep both origins in `ALLOWED_ORIGINS`, redeploy, verify live, then update sitemap/robots/canonicals
- **Content for AdSense and ranking:** separate content site on a custom domain with original articles, About and Contact pages, Search Console indexing, and real traffic first — then apply. The app itself stays ad-free until then
- **Gateway-secret code:** local only, on hold for the direct-to-Render route
- **Search Console manual indexing retry:** owner chose sitemap crawl only (2026-10-28); no retry scheduled

## Never (unless the owner reverses with new facts)

- Making the code repository public
- Oracle Always Free hosting (requires a credit card the owner does not have)
- InfinityFree hosting (PHP/MySQL shared hosting — no Node/WebSockets)
- Counting cancelled, skipped, or environment-blocked tests as passes
- Treating a commit as merge approval, or a merge as production readiness
