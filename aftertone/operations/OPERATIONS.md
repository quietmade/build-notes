# AFTERTONE — Operations Runbook

> Status: this runbook was written from the repository code, `README.md`,
> `DEPLOYMENT.md`, `SECURITY.md`, and `PROJECT-CHECKLIST.md` at
> `quietmade/aftertone` main, latest commit at the time of writing
> `ce0ea5c54a7eff487426f95c572a254dd611e085` ("Merge remote-tracking
> branch 'origin/main'", 2026-10-25).
>
> **How to read this document.** Steps marked **Verified from code** were
> checked against the source in this repository. Steps marked **Owner
> verification required** need Render dashboard access, a live service, or
> an owner decision, and were *not* performed here. Nothing in this
> runbook deploys, restarts, rolls back, or changes the live service.
>
> **Production preparation is on hold.** `PROJECT-CHECKLIST.md`
> Section 9 is unchecked. Do not treat this runbook as a production
> launch approval. It is the procedure to use *when* the owner authorises
> a release, and the record of what is still untested.

## 0. At a glance — what this service actually is

- A single Node.js HTTP/WebSocket process: `npm start` runs
  `node server/server.js` (see `package.json`). Default port `3000`;
  production uses the host-assigned `PORT`.
- **No database, no message history, no persistent storage.**
  Pairing, invites, reconnect grace, verification reports, key exchanges,
  and the bounded replay window live in process memory (`server/server.js`,
  `server/connections.js`). A process restart ends active sessions.
  A browser-session Terms acknowledgement can survive a restart *only*
  if `TERMS_SECRET` / `TERMS_SECRET_FILE` is stable (see §5).
- **Single process, no shared state.** Multiple Node instances do not
  share pairing state. Do not scale horizontally without a shared-state
  decision (see `DEPLOYMENT.md`).
- The only runtime dependency is `ws`. Browser tests use
  `playwright` / `puppeteer-core` as dev dependencies only.
- There is **no `render.yaml` and no deploy configuration in this
  repository**. CI (`.github/workflows/ci.yml`) runs `npm test` and a
  Gitleaks secret scan only; it does not deploy. Any Render service
  therefore exists only as dashboard configuration, which is
  **Owner verification required** — its service name, branch, build
  command, start command, auto-deploy setting, and rollback settings
  cannot be confirmed from this repository.
- The public hostname recorded in `README.md` and in the example
  `ALLOWED_ORIGINS` value is `https://aftertone-u6rr.onrender.com`.
  That is evidence a Render hostname was previously configured; it is
  **not** proof that this hostname is the current production service,
  that it is running the latest commit, or that its environment matches
  §5. `DEPLOYMENT.md` also warns that the repository still contains
  that Render hostname in public metadata and that changing the
  hostname is an owner-controlled release decision.
- Cloudflare, if used, is a DNS/TLS/reverse-proxy layer in front of a
  Node host — not Workers/Pages. There is no Workers entry point or
  `wrangler` configuration in this repository (see `DEPLOYMENT.md`).
- Health endpoint, verified from code (`server/server.js`): `GET /health`
  returns HTTP 200 with body `{"ok":true}` and `Cache-Control: no-store`.
  It carries **no version, commit, or build identifier**, so it cannot
  tell you which commit is running in production. See §1, step 5.
- Logging in the application is minimal: on startup the server prints
  one line, `AFTERTONE server listening on port <PORT> (env: <NODE_ENV>; ...)`.
  There is no structured log, metrics endpoint, or alerting in the code
  (see §7).

---

## 1. Production deployment procedure

### Prerequisites — Owner verification required where noted

1. Repository access to `https://github.com/quietmade/aftertone.git`,
   branch `main` (canonical, per §4). **Verified from code:** the
   repository is private and readable by the authorised GitHub App;
   the owner must confirm who else holds access.
2. A release commit chosen by the owner (step 2 below). Do not deploy
   an unreviewed working tree.
3. A Render (or other Node host) service already created by the owner,
   with the environment in §5 already set. Creating or changing that
   service is out of scope for this runbook and requires owner action.
4. The public hostname and its `ALLOWED_ORIGINS` value agreed in
   advance, consistently, across the page URL, WebSocket hostname,
   canonical URLs, sitemap, and robots file (see `DEPLOYMENT.md`).

### Step 1 — Identify the intended release commit (Verified from code)

```bash
git log --oneline -5
git status
```

- The release commit must be a commit on `main`, not a local-only
  change. Record its full SHA in the release note.
- `git status` must be understood before release: any uncommitted
  change in the deployment source is *not* part of the release unless
  it is committed first. Never deploy from a dirty tree and call it a
  named release.

### Step 2 — Verify the working tree and the release changes

1. Run the test suite locally (see §6 and `npm test`). A release with
   a failing suite is not healthy, even if the build succeeds.
2. Review the diff since the last known-good release:
   `git log <last-good>.. <release> --oneline` and
   `git diff <last-good> <release> --stat`.
3. Confirm `DEPLOYMENT.md` and `SECURITY.md` still match the release
   code, especially the fingerprint-verification and key-confirmation
   behaviour. If docs and code disagree, fix the docs before release,
   not after.

### Step 3 — How deployment is triggered (Owner verification required)

Because there is no deploy configuration in the repository:

- If the Render service is set to auto-deploy on push to `main`,
  pushing the release commit triggers a deploy. Confirm this setting
  in the Render dashboard before relying on it.
- If auto-deploy is off, deployment is a manual dashboard action
  (deploy a specific commit) by the owner.
- **Do not assume either setting.** Record the actual setting in the
  release note the first time it is confirmed, and update this
  section then. CI pushing to `main` does not, by itself, deploy.

### Step 4 — Observe deployment progress and diagnose a failed build

**Owner verification required** — Render dashboard, service logs.

1. Watch the Render deploy log for the service. A healthy deploy ends
   with the application process starting and printing its listening
   line (see §0). A build failure, a missing dependency, or a startup
   crash appears here first.
2. The most common startup failures are configuration failures, and
   they are fail-fast by design (verified from code, §5):
   - invalid `NODE_ENV` → startup throws;
   - invalid `PORT` → startup throws;
   - production `ALLOWED_ORIGINS` missing or with no valid origin →
     startup throws;
   - production `TERMS_SECRET` shorter than 32 characters, or
     `TERMS_SECRET_FILE` unreadable / shorter than 32 bytes, or
     neither set → startup throws.
   Fix the environment in the dashboard; do not "fix" it by editing
   the code to accept a weak secret or an empty origin list.
3. If the process starts but the site does not answer, distinguish
   layers (see §3): host process, proxy/Cloudflare, DNS/TLS, and the
   application itself. A 502/504 from the proxy is not, by itself,
   proof of an application bug.

### Step 5 — Identify the commit actually running in production

**Limitation, verified from code:** `/health` returns only
`{"ok":true}`. There is no version endpoint, and the startup log line
contains no commit SHA.

Therefore the deployed commit can only be identified from:

1. The Render dashboard's deploy history (deployed commit SHA and
   time) — **Owner verification required**, and the authoritative
   source until a version endpoint exists.
2. The release note recorded at deploy time (step 1).

Do not claim a production commit is known from `/health` alone.
A future, low-risk improvement (not done here, to avoid changing
production behaviour without owner approval) would be a version
endpoint or a build-time commit stamp. It is not required for this
runbook to be usable.

### Step 6 — Verify service health after deployment

Run the smoke tests in §6 against the production URL, in production
mode (read-only / minimally invasive only — see §6 for what is and
is not safe to run against production). All of these must pass:

- `/` returns 200 HTML.
- `/health` returns 200 `{"ok":true}`.
- Static assets (`/style.css`, `/net.js`, `/crypto.js`,
  `/assets/favicon.svg`) return 200.
- A WebSocket connection with an allowlisted origin receives a
  `welcome` frame.
- A WebSocket connection with a non-allowlisted origin is rejected
  (403 on upgrade).
- An unknown invite URL (`/connect/<unknown>`) returns 404.

### Step 7 — Conditions that prevent a release being called healthy

Any one of these blocks a healthy declaration, even if `/health`
returns 200:

- The deploy log shows a crash loop or a startup error.
- `/health` is not 200, or the homepage / static assets are not 200.
- The WebSocket welcome frame is not received with an allowlisted
  origin.
- The deployed commit cannot be identified from the dashboard.
- The smoke tests in §6 fail, or were not run.
- A known regression from the release diff is present and unfixed.

### Step 8 — When to stop troubleshooting and roll back

Stop troubleshooting a bad release and start §3 when:

- The service is down or unusable for new pairings and the cause is
  not identified within a short, owner-set window (suggested: 30
  minutes for a pre-launch service with no users; the owner should
  set the real window before launch), **or**
- Two attempted fixes in a row have not restored health, **or**
- The failure is a startup crash caused by the new release and the
  previous release is known to start cleanly.

Rolling back is preferable to debugging live when the previous
release is known-good. Preserve logs first (§3).

---

## 2. Pre-deployment and post-deployment checklists

### Pre-deployment

- [ ] Release commit SHA recorded; branch is `main`.
- [ ] Working tree / diff since last known-good release reviewed.
- [ ] `npm test` passes locally (see §6 for the environment caveat
      on browser/WebKit tests).
- [ ] `DEPLOYMENT.md`, `SECURITY.md`, and this runbook match the
      release code.
- [ ] Production environment variables (§5) confirmed set in the
      hosting dashboard — values never copied into Git, chat, or
      this runbook.
- [ ] Public hostname and `ALLOWED_ORIGINS` agree.
- [ ] Rollback target (last known-good commit) identified and
      recorded before deploying (see §3).

### Post-deployment

- [ ] Deploy log shows a clean start, no crash loop.
- [ ] Deployed commit identified from the dashboard and recorded.
- [ ] Smoke tests (§6) pass against the production URL.
- [ ] `/health` is 200.
- [ ] No new error pattern in the host logs.
- [ ] Release note updated with the deployed commit, time, smoke
      result, and rollback target.

---

## 3. Rollback and failed-deployment recovery

**Do not perform an actual production rollback to "test" this
procedure.** The procedure below is documented, not tested against
the live service, because production preparation is on hold and no
live rollback was authorised.

### 3.1 Identify the last known-good release

The last known-good release is the most recent commit that:

1. Passed the test suite, **and**
2. Was deployed and passed the §6 smoke tests against production,
   **and**
3. Is recorded in a release note with its SHA.

If no release has ever passed step 2, there is no proven
known-good production release — say so explicitly. Rolling back to
an untested commit is a redeploy of an unknown, not a rollback.

Record the rollback target *before* every deployment (§2).

### 3.2 Inspect the currently deployed commit and status

**Owner verification required** — Render dashboard:

1. Open the service's deploy history. Note the deployed commit SHA,
   deploy time, and status (live, failed, or building).
2. Open the service logs for the current deploy. Note the last
   startup line, any startup error (see §1, step 4), and whether
   the process is crash-looping.
3. Run `GET /health` against the public URL. Note: a 200 here does
   not prove the right commit is running (§1, step 5).

### 3.3 Restore the previous known-good version

Render's rollback mechanism is **not verified from this
repository** (no `render.yaml`, no dashboard access). Do not invent
a dashboard control. The safest practical recovery supported by
the current setup is, in order:

1. **Redeploy the known-good commit.** In the Render dashboard,
   deploy that specific commit to the service (manual deploy /
   rollback-to-commit, if the dashboard offers it for this service).
   Confirm the exact control in the dashboard at the time; if the
   dashboard offers a one-click "Rollback" to the previous deploy,
   that is the preferred path and should be recorded here after
   first use.
2. **If a specific-commit redeploy is unavailable:** push a revert
   commit to `main` that undoes the bad release, and let the
   normal deploy path (auto-deploy or manual deploy) pick it up.
   A revert is a normal, reviewable commit — never a force-push,
   history rewrite, or reset of `main`.
3. **If the service will not start at all** (including on the
   known-good commit): the failure is almost certainly configuration
   or infrastructure, not the release (see §3.5). Fix the
   environment (see §5) and redeploy; do not keep redeploying code
   against a broken environment.

After any rollback, re-run the §6 smoke tests and record the new
deployed commit. A rollback is not complete until smoke tests pass.

### 3.4 If rollback itself fails

1. Preserve logs and the deploy history (step 3.6) before trying
   anything else.
2. Check configuration first: `ALLOWED_ORIGINS`, `TERMS_SECRET`,
   `PORT`, `NODE_ENV`. A rollback to good code with a broken
   environment still fails at startup, by design.
3. Check the host/proxy layer: DNS, TLS, Cloudflare proxy settings,
   and whether the Node origin is reachable directly by the host
   (without exposing it publicly).
4. If the service remains down after a known-good redeploy *and* a
   configuration check, escalate to the owner (§3.7). Do not
   experiment with production secrets, DNS, or access permissions
   to "try something".

### 3.5 Application failure vs configuration / infrastructure failure

| Symptom | More likely layer |
|---|---|
| Startup throws about `NODE_ENV`, `PORT`, `ALLOWED_ORIGINS`, or `TERMS_SECRET` | Configuration (§5) |
| Process starts, `/health` fails, or proxy returns 502/504 | Host / proxy / network |
| Process starts, `/health` passes, homepage or WebSocket fails | Application or proxy WebSocket handling |
| Everything passes except one browser / origin | Origin allowlist or proxy `Origin` handling |
| Failure appears only after a code change, with unchanged environment | Application release — roll back (§3.3) |

### 3.6 Preserve logs and diagnostic evidence

- Save the Render deploy log and service log for the failed deploy
  and for the rollback deploy (export / copy to the incident note).
- Record: deployed commit SHAs, deploy times, `/health` results,
  smoke-test output, and the exact error messages.
- Do not include secret values, invite codes, fingerprints, or
  message content in incident notes. The server never logs message
  content by design; keep it that way.

### 3.7 Escalation

The project owner is the only currently known operator (see §8).
Escalate to the owner when:

- The service is down and rollback has not restored it.
- A secret may be compromised or its recovery method is unknown.
- A rollback would change the public hostname, origin allowlist,
  or access permissions.
- The failure involves suspected abuse or a security incident.

Do not assign incident responsibility to any other person without
their agreement.

---

## 4. Source-code recovery and repository access

### Canonical source — Verified from code / GitHub

- Repository: `https://github.com/quietmade/aftertone.git`
- Primary branch: `main`
- Latest commit verified during this work:
  `ce0ea5c54a7eff487426f95c572a254dd611e085` (2026-10-25 merge).
- The repository is private. Access is via GitHub authorisation;
  the Muse GitHub App installation must include this repository
  (verified during this work by reading the repository through
  the authorised app).

### Verify repository access

An authorised user with a GitHub account that has access to the
repository can verify access without cloning anything sensitive:

```bash
git ls-remote https://github.com/quietmade/aftertone.git HEAD
```

A successful response prints the `main` HEAD SHA. A failure here
is an access problem, not a code problem — resolve GitHub access
before attempting recovery.

### Clone into a clean directory

```bash
git clone https://github.com/quietmade/aftertone.git aftertone-recovery
cd aftertone-recovery
git status
git log --oneline -5
```

- Clone into a *new* directory. Do not clone over an existing
  working tree, and do not use `--force`, reset, or clean on the
  original tree.
- `git status` on a fresh clone must be clean. Any difference is
  a local change, not part of the recovered source.

### Identify the expected release commit

The expected release commit is the SHA recorded in the release
note (§1, step 1) or, if no release note exists, the latest commit
on `main` at recovery time — recorded explicitly as such, so a
later reader can tell the two cases apart.

```bash
git rev-parse HEAD
git log -1 --format='%H %ad %s'
```

### Install prerequisites and verify the recovered source

Prerequisites are the project's own documented ones (verified
from `README.md` / `package.json`): Node.js 18 or newer, npm
included with Node. No new packages are required.

```bash
npm ci
npm test
```

- `npm ci` uses `package-lock.json`, which is in the repository.
- The full suite includes browser tests (Firefox) and WebKit
  tests (Playwright). Those two suites require browsers/system
  libraries that may not exist on a given machine; on such a
  machine they are environment-blocked, not code failures, and
  must be reported as such (see §6). The non-browser suite is
  the portable verification baseline.
- If the recovered source's test results differ from the release
  note's recorded results, stop: the recovered source does not
  match the intended release until the difference is explained.

### Recover deployment configuration without exposing secrets

Deployment configuration is **not** in Git (see §5 and
`.gitignore`: `.env`, `.env.*`, `*.pem`, `*.key`, `*.crt` are
ignored). Recover it only from the hosting dashboard / secret
manager, by the owner, following §5. Never reconstruct a secret
from source code, old logs, or a chat transcript — if a secret
cannot be recovered from the authorised store, rotate it (§5)
instead of copying a guessed value.

### Mirror / backup — recommendation

No additional mirror is created by this work, and none is
required at this stage, for these reasons:

- GitHub itself is the durable copy: the full history is on the
  remote, and a fresh clone recovers it.
- The application has no database or user data to back up (see
  §0); there is nothing else whose loss would require a backup.
- A second mirror would add a second place where access,
  visibility, and secret handling must be kept correct, with no
  corresponding benefit while the project is pre-production.

If the owner later wants a mirror, the recommended, simple form
is: a second private GitHub repository (or an owner-controlled
organisation repository) updated by a `git push --mirror` from an
authorised machine, or GitHub's own repository backup. Creating
it, changing visibility, or altering permissions requires explicit
owner authorisation and is **pending owner action**, not done here.

### Source-recovery concern — record, not resolved by assumption

Previous project records noted a discrepancy between expected and
observed repository history. In the remote history verified during
this work, that discrepancy is no longer visible: the remote
`main` now includes the 2026-10-25 merge
(`ce0ea5c54a7eff487426f95c572a254dd611e085`) and the
"chore: harden security and reliability" commit it merges, which
were absent from an earlier remote check (latest then: 2026-10-22).

**What remains unverified:** the working tree named in the
original task, `/home/e14station/Documents/aftertone`, is not
accessible from this environment, so its branch, `git status`,
uncommitted changes, and relationship to remote `main` could not
be checked here, and no history was rewritten, reset, or
force-pushed to resolve anything. The owner should confirm on
that machine:

```bash
cd /home/e14station/Documents/aftertone
git status
git log --oneline -5
git fetch origin
git status -sb
```

If that tree contains uncommitted work, preserve it (commit it
on a branch, or copy the tree) before any clone-over or cleanup.
Do not assume the remote and that tree agree until this check is
done. This is **pending owner action**.

---

## 5. Production environment variables and secrets recovery

Inventory verified from code: `server/config.js`,
`server/terms.js`, `server/server.js`, `server/connections.js`,
and `README.md` / `DEPLOYMENT.md`. No secret values are recorded
here, and none should ever be added.

### Required in production

| Variable | Controls | Validated at startup? | Security / availability impact |
|---|---|---|---|
| `NODE_ENV` | Must be `development`, `test`, or `production`. Production enables strict origin handling, HSTS/`Secure` cookie handling via `X-Forwarded-Proto: https`, and strict secret requirements. | Yes — any other value throws. | High: wrong value either blocks production hardening or blocks startup. |
| `PORT` | Port the Node process listens on. In production this is the host-assigned port. | Yes — must be an integer 1–65535, else throws. Default `3000` if unset. | Availability: wrong port = host cannot reach the process. |
| `ALLOWED_ORIGINS` | Comma-separated exact `http(s)` origins allowed to open a WebSocket and to call `/terms/accept`. No path, query, fragment, credentials, or trailing slash; each entry is parsed and normalised to its origin. | Yes — production startup throws if missing, empty, or with no valid origin. | Security + availability: too narrow breaks the site; too broad admits untrusted origins. This is the origin allowlist, not authentication. |
| `TERMS_SECRET` **or** `TERMS_SECRET_FILE` | Signing secret for the Terms acknowledgement cookie and socket proof. Must be stable across restarts or existing acknowledgements are invalidated after restart. | Yes — production requires one of them: `TERMS_SECRET` ≥ 32 characters, or `TERMS_SECRET_FILE` pointing to a readable file ≥ 32 bytes, else startup throws. | Security: a weak, shared, or leaked secret lets an attacker forge Terms acceptance. Availability: losing it forces all users to re-accept Terms. |

Use **one** of `TERMS_SECRET` / `TERMS_SECRET_FILE`, not both
confusedly: `TERMS_SECRET` wins in code if set, because
`loadSigningSecret()` checks it first. Prefer the file form when
the host provides a mounted secret file, so the value does not
live in an environment-variable dump.

### Optional — safe defaults, verified from code

All of these fall back to a documented default when unset, and
invalid values fall back to the default (never to unlimited),
except `MB88_SESSION_GRACE_MS`, where `0` is a valid test value
for immediate-close cleanup:

| Variable | Default | Controls |
|---|---|---|
| `MB88_MAX_CONN` | `2000` | Maximum open WebSocket connections; excess sockets are closed with `1008 server busy`. |
| `MB88_RATE_BURST` | `30` | Per-connection message burst. |
| `MB88_RATE_PER_SEC` | `5` | Per-connection refill rate. |
| `MB88_JOIN_LIMIT` | `20` | Join attempts per connection per minute. |
| `MB88_MAX_LIVE_INVITES` | `5` | Live invites per connection. |
| `MB88_SESSION_GRACE_MS` | `30000` | Reconnect grace (ms) before a browser session, invite, and pair are closed. |
| `MB88_MAX_B64` | `512` | Maximum relayed public-key field size (base64 chars). |
| `MB88_MAX_IV_B64` | `64` | Maximum IV field size. |
| `MB88_MAX_CIPHERTEXT_B64` | `1024` | Maximum ciphertext field size. |
| `MB88_REPLAY_MAX` | `2048` | Duplicate-envelope entries per session. |
| `MB88_REPLAY_WINDOW_MS` | `600000` | Duplicate-envelope retention window (ms). |
| `MB88_REPLAY_TOTAL_MAX` | `10000` | Aggregate duplicate-envelope entries. |

These affect availability and abuse resistance, not message
confidentiality. Changing them is an owner tuning decision, not a
secret rotation.

### Development / test only — never set in production

| Variable | Purpose |
|---|---|
| `MB88_TEST_DURATIONS` | Shortens invite/session durations in automated tests (e.g. `0.05` minutes). Setting it in production would shorten real sessions. Do not set it in production. |

### Never in Git

`.gitignore` already excludes: `.env`, `.env.*` (except
`.env.example`, which does not exist in this repository),
`*.pem`, `*.key`, `*.crt`, `*.p12`, `*.pfx`, `.cache/`
(including the development Terms-secret fallback file),
`*.log`, and `node_modules/`. CI also runs a Gitleaks secret
scan. If a secret is ever committed by accident: rotate it
immediately (below), then remove it from history only with owner
authorisation — never by rewriting shared history casually.

### How to verify configuration without printing a value

1. Start the process (locally, or observe the host deploy log).
   A clean startup with no config error proves the *shape* of
   the configuration is valid. It does not prove a secret's value.
2. `GET /health` → 200 `{"ok":true}` proves the process is up.
3. Open the site from an allowlisted origin and create an invite:
   success proves `ALLOWED_ORIGINS` includes that origin.
   A 403 on WebSocket upgrade proves the origin is *not*
   allowlisted (or the value is wrong) — check the dashboard
   value, character by character, for a trailing slash, path,
   or wrong hostname.
4. Accept Terms, then restart the process (in a test environment,
   never production to "test" this): if the acknowledgement
   survives, the Terms secret is stable. If it does not, the
   secret changed or was process-local.
5. Never print, log, screenshot, or paste a secret value to
   "verify" it. Verification is by behaviour, not by reading
   the value.

### Recovery and rotation procedure (owner action)

**Recovery** (service moved, host replaced, or config lost):

1. In the hosting dashboard / secret manager, read the stored
   values — or, if they are lost, treat them as lost and rotate
   (below). Do not copy values from code, logs, or chat.
2. Set `NODE_ENV=production`, `PORT` as assigned by the host,
   `ALLOWED_ORIGINS` to the exact public origin(s), and
   `TERMS_SECRET` or `TERMS_SECRET_FILE` to the recovered value.
3. Redeploy / restart and run §6 smoke tests. A clean start plus
   a successful invite creation from the public origin confirms
   recovery.

**Rotation** (secret leaked, suspected compromise, or lost):

1. Generate a new random value of at least 32 characters (for
   `TERMS_SECRET`) or 32 bytes (for a secret file), using the
   host's secret generator or `node -e` / `openssl rand` on an
   authorised machine. Never reuse an old value.
2. Set the new value in the hosting dashboard / secret manager
   only. Do not commit it.
3. Redeploy / restart. Consequence, by design: existing signed
   Terms acknowledgements become invalid and users must accept
   Terms again in a new browser session. Active pairs are ended
   by the restart in any case (in-memory state, §0).
4. Run §6 smoke tests, including a fresh Terms acceptance.
5. If the old secret may have been exposed outside the owner's
   control, note the incident (date, scope, rotation time) in the
   incident note — without recording either value.

**Live production values are unverified by design.** This
runbook does not, and must not, state what the live Render
environment currently contains. Confirming it is a dashboard
check for the owner, listed in §9.

---

## 6. Production smoke-test procedure

### Script — Verified locally

`scripts/smoke-test.mjs` (in this repository) runs the
read-only / minimally invasive checks below against a base URL.
It uses only Node built-ins plus the existing `ws` runtime
dependency — no new package.

```bash
# Against a locally started server (default):
npm start &
node scripts/smoke-test.mjs
# or, explicitly:
SMOKE_BASE_URL=http://localhost:3000 node scripts/smoke-test.mjs

# Against production (owner only, see "Production" below):
SMOKE_BASE_URL=https://aftertone-u6rr.onrender.com \
SMOKE_ORIGIN=https://aftertone-u6rr.onrender.com \
  node scripts/smoke-test.mjs
```

Exit code `0` means every check passed; non-zero means at least
one check failed, with the failing check named in the output.
The script is covered by `test/smoke-script.test.js`, which runs
it against a test server started by the existing test helpers.

### Checks, expected results, and failure criteria

| # | Check | Expected | Failure means |
|---|---|---|---|
| 1 | `GET /` | 200, HTML content | Process down, proxy misrouted, or static serving broken. |
| 2 | `GET /health` | 200, body exactly `{"ok":true}` | Process up but health route broken, or a proxy is answering instead of the app. |
| 3 | `GET /style.css`, `/net.js`, `/crypto.js`, `/assets/favicon.svg` | 200 each | Static serving / deploy packaging broken; the UI will not load. |
| 4 | `GET /connect/<unknown-code>` | 404 | Invite-route handling broken (a 200 here would render the UI for a dead invite). |
| 5 | WebSocket with an allowlisted origin | Receives a `welcome` frame with a `clientId` | Origin allowlist, WebSocket upgrade, or proxy WebSocket handling broken. |
| 6 | WebSocket with a non-allowlisted origin (`https://example.invalid`) | Upgrade rejected (403 / connection closed without `welcome`) | Origin policy not enforced — security failure, stop and investigate. |

Checks 1–4 are plain HTTP GETs. Checks 5–6 open a WebSocket,
read at most the welcome frame, and close immediately. They do
**not** create an invite, pair, verify, or send a message.

### Full two-client verification (manual, local or staging only)

The script above deliberately does not do pairing or messaging,
because doing so against production would create real pairing
state. For a full verification, run it locally or in staging,
using two independent browser sessions (or the existing
`test/*.test.js` suites, which already cover this):

1. Client A creates an invite; Client B joins with the code;
   Client A accepts. Expected: both reach "connected" state.
2. Both compare the displayed fingerprint over an independent
   channel and press VERIFY. Expected: messaging is enabled only
   after both verify; before that, messages are not relayed.
3. Client A sends a message; Client B receives it, and vice
   versa. Expected: correct direction and delivery state on both
   sides.
4. Send a duplicate / replayed envelope (covered by the automated
   suite). Expected: rejected by the receiver's sequence check.
5. Attempt messaging from an unpaired or unverified state.
   Expected: rejected safely, no plaintext delivered, no crash.

These behaviours are covered by the automated suites
(`test/protocol.test.js`, `test/netclient.test.js`,
`test/key-exchange-lifecycle.test.js`, `test/regression.test.js`,
and others). Do not re-run them against production as a smoke
test — they create pairing state and are not production-safe.

### Recovery behaviour — Verified from code, do not test on production

- **Process restart / redeploy:** all in-memory state is lost.
  Active pairs end, invites end, verification reports end, and
  session keys are discarded. Users must pair again. This is by
  design (no database, no message history) and is a user-visible
  limitation, not a bug to "fix" by adding persistence without a
  separate design decision.
- **Socket disconnect within the grace window (default 30 s):**
  the session is suspended, not closed; the peer sees
  "peer-reconnecting". If the client returns within the window,
  the pair resumes with a *fresh* key exchange, fresh key
  confirmation, and fresh verification — old session keys are
  not restored, and both sides must VERIFY again before messaging
  is re-enabled (see `client/faq.html`).
- **Socket disconnect after the grace window:** the session,
  invite, and pair are closed. Re-pairing from scratch is required.
- **Terms secret lost / changed:** signed acknowledgements are
  invalidated; users must accept Terms again in a new browser
  session. The Terms cookie itself is session-scoped, so a new
  browser session must accept Terms regardless.
- **Do not deliberately restart production to test recovery.**
  The behaviour above is verified from code and from the
  automated `test/server-failure-recovery.test.js` and
  `test/session-resume.test.js` suites.

### Production rules for smoke tests

- Read-only or minimally invasive only: the script's checks
  above. No invites, no pairing, no messages, no real-user
  contact, no load testing, no repeated polling (a single run
  is enough; do not loop it against production).
- If production access is unavailable, run the script locally
  and give the owner the exact production command above to run
  later. Report the local result and the unexecuted production
  result separately — never merge them into one "passed".
- **Production smoke-test execution is pending owner action**
  (see §9). It was not run against the live Render service here.

---

## 7. Monitoring, logging, and alerting — decision

### Recommendation (decision, not a purchase)

**For the current stage — pre-production, Section 9 on hold,
no live users — monitoring and alerting are recommended but
optional, and no new service should be bought or installed now.**

**Before any production launch, the following minimum becomes
required:**

1. **Uptime / health monitoring** on `GET /health` (and on `/`
   as a user-visible check), from outside the host, at least
   every 5 minutes.
2. **Deployment failure notification** from the hosting
   dashboard to the owner (Render deploy notifications, or
   equivalent on the chosen host).
3. **Owner review of host logs** after every deploy and after
   any incident, until a log-retention decision is made.

Rationale: the service is a single in-memory process with no
database. Its failure modes are almost entirely "process down",
"deploy failed", or "origin/config wrong" — all of which a
`/health` check plus deploy notifications catch. Application
error visibility beyond host logs, WebSocket-specific alerting,
and log-retention tooling are disproportionate until there are
real users and a real incident history. Adding them now would
add cost, a second place holding (potentially sensitive) logs,
and maintenance, for a service that is not yet live. This
recommendation is consistent with the project's zero-budget
constraint for operational additions: use free, host-native
capabilities first.

### What already exists — Verified from code / repository

| Capability | Status |
|---|---|
| `/health` endpoint (200 `{"ok":true}`, no-store) | Exists, verified from code and by the test suite. Suitable as an uptime-check target. |
| Host (Render) deploy logs and service logs | Available in principle via the dashboard; **not verified** here (no dashboard access). |
| CI test + secret scan on push / PR to `main` | Exists (`.github/workflows/ci.yml`). Not a production monitor. |
| Application structured logging, metrics, error tracking | **Does not exist in code.** Only the startup line is logged. |
| Alerting (email, webhook, paging) | **Not configured in code or in this repository.** Requires dashboard / external service setup by the owner. |
| Log retention policy | **Not defined in code.** Host log retention is whatever the host provides; not verified here. |

### What is genuinely necessary, and when

| Need | Necessary now? | Trigger for investigation |
|---|---|---|
| `/health` down for > 5 minutes (or 2 consecutive checks) | At launch: yes | Process down, host down, or proxy broken — start §3. |
| Deploy marked failed, or deploy succeeds but smoke tests fail | At launch: yes | Do not declare healthy; roll back (§3) if the previous release was good. |
| Repeated WebSocket 403s from allowlisted origins | At launch: investigate | Origin allowlist or proxy `Origin` handling wrong — check §5, not the client. |
| Unexplained process restarts / crash loops in host logs | At launch: investigate | Startup config (§5) or host resource limits. |
| Message / pairing failures reported by users | After launch: investigate | Use the automated suites locally to reproduce; do not debug by reading production traffic (there is none to read — messages are ciphertext, and the server must not log them). |

### Who receives / acts on alerts

The project owner, as the only currently known operator (see
§8). No other person is assigned alert duty in this runbook.
If the owner wants a second recipient, that is an owner decision
to record here — do not add a recipient silently.

### What to do when an alert fires

1. Confirm with `GET /health` and the §6 smoke script (read-only).
2. Check the host deploy log: is this a failed deploy, a crash
   loop, or a running process with a proxy problem? (§3.5)
3. If the previous release was known-good and the current one
   is not, roll back (§3). Otherwise, fix configuration (§5)
   or escalate (§3.7).
4. Record the incident (time, symptom, cause if known, action,
   smoke result) without secrets, invite codes, fingerprints, or
   message content.

### Cost, privacy, and maintenance implications

- A free-tier uptime check (e.g. the host's own health check,
  or a free external checker chosen by the owner) costs nothing
  and sees only `/health` and `/` — no user data.
- A paid monitoring service, a log aggregator, or an agent
  installed on the host would see host logs (which contain no
  message content by design, but do contain connection metadata
  such as timing and pairing events) and would add cost and a
  new trust boundary. Not recommended at this stage.
- Whatever is chosen, its retention and access must be set by
  the owner; this runbook does not grant any monitoring service
  access to production.

### Abuse tracking — `server/abuse.js` (Phase 1, metadata-only)

Abuse tracking is **disabled by default**. Enable it only when the
owner is ready to review its output:

- `MB88_ABUSE_ENABLED=1`
- `MB88_ABUSE_LOG_DIR` — directory for daily JSONL files
  (`abuse-YYYY-MM-DD.jsonl`). File logging is off unless this is set.
- `MB88_ABUSE_LOG_SALT` — stable HMAC salt, at least 32 chars in
  production. Without it, hashes are process-local and not stable
  across restarts.
- `MB88_ABUSE_RETENTION_DAYS` (default 30),
  `MB88_ABUSE_BAN_MINUTES` (default 60),
  `MB88_ABUSE_BAN_THRESHOLD` (default 5 hits),
  `MB88_ABUSE_BAN_WINDOW_MS` (default 600000, 10 minutes).
- `MB88_ABUSE_MANUAL_BAN_FILE` — optional file with one IP hash
  (64 hex chars) per line; those hashes are banned for the life
  of the process. The file must contain hashes, never raw IPs.

What it records: connection, invite create/join/accept, and a
defined set of rejections (invalid JSON, rate-limited, join-limit,
too-many-invites, join failures, plaintext-not-allowed,
verification-required, duplicate-message), plus aggregate relay
counters per client/pair (message count and total iv+ciphertext
bytes). It does **not** log per-message events and never logs
plaintext, iv, ciphertext, fingerprints, key-confirmation tags,
raw invite codes, or raw IPs — IPs and invite codes are stored
only as HMAC-SHA256 hashes.

Enforcement in Phase 1: when hits in the window reach the
threshold, that IP hash is banned for the ban duration — new
WebSocket upgrades from it get 403, and the offending socket is
closed with code 1008. A banned invite hash is rejected on join.
Bans and counters are in-memory and reset on restart; log files
on an ephemeral host do not survive a restart either. There is
no HTTP admin endpoint for bans in Phase 1, by design — manual
bans are via the file above, or by restarting with a new file.

Limitation to be aware of: IP identity is the direct peer
address seen by Node. Forwarded headers are not trusted, so
behind Render/Cloudflare the "IP" may be the proxy address and
IP bans will be coarse. Do not enable forwarded-header trust
without a trusted-proxy check.

### Pending owner actions for monitoring

- Confirm the Render service's health-check / deploy-notification
  settings in the dashboard.
- Choose and configure the external `/health` checker (or
  confirm the host's own check is sufficient) before launch.
- Decide host log retention, and record it here.

Until those are done, monitoring is **decided, not implemented**
— see the Section 10 status in `PROJECT-CHECKLIST.md`.

---

## 8. Known limitations and operational responsibilities

### Limitations — verified against the current code, not copied blindly

| Limitation | Impact | Severity / priority | Status |
|---|---|---|---|
| Key exchange is unauthenticated; a malicious relay can substitute keys and coordinate a separate exchange with each participant. Key confirmation proves possession of the derived key, not identity. | First-contact MITM is possible unless users compare the full fingerprint over an independent trusted channel. | High — inherent to the design, not a bug to patch silently. | Verified, unresolved by design. See `SECURITY.md`. |
| Fingerprint verification is a human out-of-band comparison, not persistent identity authentication. A malicious relay can deny service or misreport completion. | Verification detects substitution only if both users actually compare the full value and both press VERIFY. | High — user-behaviour dependent. | Verified. |
| No forward secrecy beyond discarding ephemeral session keys; no ratchet or in-session rekey. | A compromised session key exposes that session's messages; past sessions are not protected by a ratchet. | Medium. | Verified. |
| All pairing, session, verification, and replay state is in memory. Restart / redeploy ends all active sessions; messages are not persisted or recoverable. | Users must re-pair after every deploy/restart. No message recovery is possible, by design. | High operational impact, low surprise if documented. | Verified. |
| Single process, no shared state. Horizontal scaling without shared coordination will break pairing. | Scaling is blocked until a shared-state decision is made. | Medium — blocks scaling, not single-process operation. | Verified, unresolved decision. |
| No version/commit identifier in `/health` or in the startup log. | Deployed commit cannot be identified from the service itself; only from dashboard / release notes. | Medium operational. | Verified. |
| No structured logging, metrics, or alerting in code. | Incidents are visible only via host logs and external health checks (see §7). | Medium, pre-launch. | Verified. |
| No Render deploy configuration in the repository; rollback capability unverified. | Deploy and rollback depend on dashboard settings that are not in Git and not verified here. | High operational, pre-launch. | Verified gap, pending owner action. |
| Physical iOS / Android testing, including real Safari, is still pending (`PROJECT-CHECKLIST.md` §6). | Real-device behaviour (backgrounding, Safari WebSocket handling) is untested. | High for a mobile-facing launch, not for local operation. | Unresolved, pending owner / device access. |
| Production preparation (Section 9) is on hold; public hostname / `ALLOWED_ORIGINS` finalisation, Cloudflare TLS/WebSocket/timeout/caching settings, and live verification are all pending. | The service must not be presented as production-ready on the basis of this runbook alone. | Blocking for launch. | Verified, pending owner action. |
| Browser compromise, XSS, malicious extensions, and device compromise are out of scope. | Encryption does not protect a compromised endpoint. | Medium — threat-model boundary. | Verified, by design. |
| A relay can suppress or delay traffic, even though duplicate/reordered/stale/cross-session messages are rejected by the receiver's sequence check. | Availability, not confidentiality, can be attacked by a relay. | Medium. | Verified. |
| Terms secret loss invalidates acknowledgements; Terms cookie is session-scoped. | Users must re-accept Terms after a secret rotation or in a new browser session. | Low — expected behaviour, documented in §5. | Verified. |

**Do not imply encryption makes the service fully secure.**
The threat model above is the current, verified position. If any
row changes, update `SECURITY.md` and this section together.

### Operational responsibilities — proposed model, for owner confirmation

The owner is the only currently known operator. The proposed
model below is therefore "owner does all of these, until the
owner delegates one explicitly". No other person is assigned
anything here.

- **Deployment approval:** owner approves the release commit
  and the deploy (see §1).
- **Secret management:** owner owns `TERMS_SECRET` /
  `TERMS_SECRET_FILE` and `ALLOWED_ORIGINS` in the hosting
  dashboard, and their rotation (§5).
- **Release verification:** owner (or a person the owner
  delegates) runs §6 smoke tests and records the result.
- **Monitoring:** owner receives alerts and reviews host logs
  (§7).
- **Incident response and rollback decisions:** owner decides
  rollback vs fix (§3, §3.7).
- **Recovery testing:** owner periodically re-runs §4 (clone
  + `npm test`) and §6, and confirms the local working tree
  check in §4 is clean.
- **Documentation:** any change to deployment, hostname,
  secrets storage, or monitoring must update this runbook and
  `PROJECT-CHECKLIST.md` Section 10 in the same change.

Flag for confirmation: if the owner wants a second operator,
a different alert recipient, or a different release approver,
record that person (with their agreement) here. Until then,
"owner" means the project owner, Tejender Thakur
(`quietmade` on GitHub), as the only operator on record.

---

## 9. Outstanding owner actions (consolidated)

These are the only items in this runbook that genuinely require
owner credentials, access, or a decision. Everything else above
is preparatory work done locally.

1. **Confirm the local working tree** at
   `/home/e14station/Documents/aftertone` (branch, status,
   uncommitted changes, relationship to remote `main`) — see §4.
2. **Confirm Render dashboard configuration** for the service:
   service name, branch, build/start commands, auto-deploy on/off,
   health-check settings, deploy notifications, and whether a
   one-click rollback / redeploy-a-commit control exists — see
   §§1, 3, 7.
3. **Confirm the current production hostname and environment:**
   whether `https://aftertone-u6rr.onrender.com` is still the
   production service, what its deployed commit is, and that
   `ALLOWED_ORIGINS` and the Terms secret in the dashboard match
   §5 — without sharing values with anyone.
4. **Run the production smoke tests** (§6) against the live URL
   once, and record the result. Do not run pairing/messaging
   tests against production.
5. **Configure monitoring before launch** (§7): external
   `/health` check, deploy notifications, and log-retention
   decision.
6. **Decide the Section 9 items** (hostname, Cloudflare settings,
   single-process vs shared state, secret storage) and authorise
   any actual deployment. This runbook does not authorise one.
7. **Physical device testing** (`PROJECT-CHECKLIST.md` §6) if a
   mobile-facing launch is intended.
8. **If a mirror is wanted**, authorise and create it (§4) —
   otherwise, explicitly accept GitHub as the sole durable copy.

---

*End of runbook. Companion files: `scripts/smoke-test.mjs`,
`test/smoke-script.test.js`, `PROJECT-CHECKLIST.md` Section 10.*
