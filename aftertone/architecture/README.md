# AFTERTONE — architecture

This folder separates three kinds of material:

| Kind | Location | Notes |
|---|---|---|
| Original Archify exports | [archify/](archify/) | Unmodified HTML, Archify 3.2.0 |
| Input to the mapping run | [archify-mapping-brief.md](archify-mapping-brief.md) | Instructions given for the repository map |
| Explanation | this file | Written commentary |

## Diagrams

Each export is a single self-contained HTML page with pan, zoom, search, and export controls. GitHub shows these as source text. Download the file and open it in a browser.

| File | Type | What it shows |
|---|---|---|
| [aftertone-architecture.html](archify/aftertone-architecture.html) | Architecture | Runtime components: browser clients, static UI, crypto layer, and Node server pieces (HTTP server, origin guard, terms service, WebSocket session manager) |
| [message-flow.html](archify/message-flow.html) | Sequence | Successful path: create invite, join, accept, key exchange, encrypted message relay between Client A, Crypto A, Node Server, Crypto B, Client B |
| [aftertone-pair-lifecycle.html](archify/aftertone-pair-lifecycle.html) | Lifecycle | Invite and pair states: Available, Pending, Connected, Ended, plus the 30 s reconnect grace |
| [aftertone-remap.html](archify/aftertone-remap.html) | Architecture | Repository architecture and remaining-work map; names client files (`client/index.html`, `client/net.js`) and server files (`server/server.js`, `config.js`) and includes verification and key-confirmation components |

Descriptions come from the text visible in each file.

## Limits of these diagrams

- The files do not record the source commit or generation date. How closely they match the current code is unknown.
- The Runtime Architecture, Message Flow, and Pair Lifecycle diagrams show no key-confirmation or fingerprint-verification step, while [SECURITY.md](../security/SECURITY.md) describes both. They may predate those changes. The remap diagram does include them.
- The Runtime Architecture diagram includes a Google Fonts CDN node, so the live UI depends on an external font host.
- None of these files has been checked against the source code as part of building this repository.

## Mapping brief

[archify-mapping-brief.md](archify-mapping-brief.md) is the brief for a repository mapping run. It asks for a map that separates verified source facts, reported results, inferences, and external blockers, and lists remaining work (real Safari, physical iOS and Android, manual end-to-end checks, independent audit, production readiness). The remap diagram appears to be the output of that run, though the files do not say so. The test counts in the brief are reported results, not results the mapping tool ran.

## Source code references

Component paths named in the remap diagram and in the operations runbook include `client/index.html`, `client/net.js`, `client/crypto.js`, `server/server.js`, `server/config.js`, `server/connections.js`, and `server/terms.js`. They refer to the separate source repository.
