# AFTERTONE deployment preparation

## Current runtime boundary

AFTERTONE currently runs as the Node.js process started by:

```bash
NODE_ENV=production npm start
```

It uses Node's HTTP server, the `ws` WebSocket server, local static-file
access, and in-memory pairing/session state. This repository intentionally
does **not** contain a Cloudflare Workers entry point, Pages Function, or
`wrangler` configuration. Cloudflare Pages/Workers cannot directly execute the
existing application without an architectural port that has not been
authorized.

Cloudflare may be used as a DNS/TLS/reverse-proxy layer in front of an
owner-selected Node.js host. Selecting that host, changing DNS, and enabling
the production service are deployment-owner decisions and are not performed
by this repository preparation.

## Required production environment

Set these values in the Node.js process environment or its secret manager:

```text
NODE_ENV=production
PORT=<port assigned by the Node host>
ALLOWED_ORIGINS=https://<public-aftertone-hostname>
TERMS_SECRET=<stable random value of at least 32 characters>
```

`ALLOWED_ORIGINS` accepts a comma-separated list of exact `http(s)` origins.
Do not include a path, query, fragment, credentials, or a trailing slash.
Production startup fails if the variable is missing, empty, or contains no
valid origins.

Use `TERMS_SECRET_FILE` instead of `TERMS_SECRET` when the host provides a
stable, server-only mounted secret file containing at least 32 bytes. Do not
rely on the development `.cache/terms-secret` fallback in production. Losing
the configured Terms secret invalidates existing signed acknowledgements after
restart.

The `MB88_*` resource and lifetime controls have documented safe defaults in
the README. Review and set them for the selected host's capacity and abuse
model; they do not provide distributed coordination or persistence.

## Reverse-proxy requirements

For an HTTPS public hostname with TLS terminated by Cloudflare or another
proxy:

1. Forward ordinary HTTP requests to the Node process and preserve the request
   host used by the public site.
2. Forward WebSocket `Upgrade` and `Connection` handling for the same hostname;
   the browser connects to `wss://<public-hostname>` automatically when the
   page is HTTPS.
3. Set `X-Forwarded-Proto: https` on requests that arrived at the proxy over
   HTTPS. In production the server uses this exact value to add HSTS and the
   `Secure` attribute to the Terms cookie. Do not expose the Node origin
   directly if an untrusted client can forge this header.
4. Do not cache WebSocket upgrades, `/health`, Terms acceptance responses, or
   invite/session routes. The application emits its own cache policy, but the
   proxy must not override it with public caching.
5. Keep proxy idle/read timeouts compatible with long-lived WebSocket sessions.
   A proxy timeout is a transport interruption and can cause the application
   session to enter its reconnect-grace path.

The browser `Origin` header must exactly match one of `ALLOWED_ORIGINS`.
Cloudflare's public hostname, the page URL, the WebSocket hostname, canonical
URLs, sitemap, and robots file must therefore be chosen consistently. The
current repository still contains the previously configured Render hostname
in public metadata; changing that hostname is an owner-controlled release
decision, not inferred here.

## State and scaling constraints

Pairing, reconnect grace, invite lifecycle, and bounded replay detection are
in memory. A process restart ends active sessions, and multiple Node instances
do not share state. Before enabling horizontal scaling, the owner must choose
whether to keep a single process, add suitable shared coordination, or change
the architecture. This preparation does not make that decision.

## CI and verification boundary

CI runs the Node.js test suite and secret scan. It does not deploy, configure
Cloudflare, verify DNS, or perform live production checks. Before release, the
owner must verify the selected Node host, environment secrets, Cloudflare TLS
mode, WebSocket support, forwarded-protocol behavior, public origin allowlist,
and process restart/scaling policy.

## Remaining owner decisions

- Which Node.js host will run the application process behind Cloudflare, if
  Cloudflare is used only as the edge proxy.
- The final public hostname(s) and corresponding `ALLOWED_ORIGINS` values.
- Where the stable Terms secret will be stored and how it will be rotated.
- Whether the service will remain single-process or gain shared state before
  any horizontal scaling.
- Cloudflare TLS, WebSocket, timeout, caching, origin-firewall, and DNS
  settings, followed by live verification.
