# AFTERTONE

**Two devices. One private line.**

A retro-inspired, real-time messaging terminal for two people. Pair two devices using a private invite code and exchange encrypted messages through a minimal CRT-style interface.

**[Production deployment](https://aftertone-u6rr.onrender.com/)** · **[Report a Vulnerability](mailto:invasiveminds.tech@gmail.com)**

![License: MIT](https://img.shields.io/badge/license-MIT-8bf03c?style=flat-square)
![Node.js](https://img.shields.io/badge/Node.js-Runtime-339933?style=flat-square&logo=nodedotjs&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![CSS3](https://img.shields.io/badge/CSS3-Styling-1572B6?style=flat-square&logo=css3&logoColor=white)
![Web Crypto API](https://img.shields.io/badge/Web_Crypto_API-Encryption-8A2BE2?style=flat-square)
![Tests](https://img.shields.io/badge/Tests-150_Passing-2EA44F?style=flat-square)

---

## Security

> **Verification status: UNVERIFIED.** This section describes the current source implementation and local tests. It has not been independently reviewed or verified against a live deployment. Do not treat these properties as production security guarantees.

AFTERTONE encrypts message content in the browser before relaying it through
the server. Each paired session uses an ephemeral **P-256 ECDH** key pair,
**HKDF-SHA256**, and **AES-256-GCM** through the browser Web Crypto API. The
server relays public keys, key-confirmation tags, and `{iv, ciphertext}`
envelopes; it does not receive the private keys, the derived AES key, or
plaintext during normal operation. Before a session is ready, each side proves
it derived the same session key by sending an HMAC-SHA256 key confirmation over
the exchange transcript (exchange ID, both sorted client IDs, both sorted public
keys, expiry, and sender). This is key confirmation, not peer authentication.

This is **not authenticated end-to-end encryption**. There is no long-term
identity key, signature, or authenticated identity binding. A server or other
active relay can substitute public keys and derive separate keys with each
participant. Both participants must explicitly verify the same 96-bit
fingerprint before application messaging is enabled. The fingerprint is derived
locally from both exchanged public keys plus the pair context, including the
server-issued exchange identifier. Verification gates relayed messages, but it
is still a human out-of-band comparison, not cryptographic peer
authentication.

Each application message is sealed as an authenticated
`{v, exchangeId, sender, recipient, seq, text}` payload. The receiver enforces
the exact session binding and a strictly increasing per-sender sequence, so
duplicate, reordered, stale, and cross-session application messages are
rejected after decryption. This is receive-side message replay protection, not
protection against traffic suppression or delay.

### What the fingerprint does and does not prove

- Both honest clients derive the same fingerprint from the same pair of public
  keys. A substituted key changes it.
- Comparing the fingerprint over a separate trusted channel can detect a
  substitution, but only if both people compare the full value accurately and
  the comparison channel is not controlled by the attacker.
- Displaying a fingerprint, or one participant confirming it, does not verify
  the pair. Both sides must explicitly VERIFY, and the server only relays
  messages after both reports match the current exchange and both sides have
  sent a key confirmation.
- Even a completed comparison does not prove the peer's identity or prevent an
  active man-in-the-middle when the comparison channel is attacker-controlled.

### Limitations

- The key exchange is unauthenticated and remains vulnerable to an active
  man-in-the-middle. Key confirmation proves possession of the derived key, not
  peer identity. Mandatory fingerprint verification can detect a substitution
  only when both people compare the full value over an independent trusted
  channel. It is not authenticated key exchange or complete MITM resistance.
- There is no forward secrecy beyond discarding each session's ephemeral key
  material when the session ends. There is no ratchet or in-session rekey.
- Compromise of a browser, the page origin, or the running process can expose
  the session key and plaintext.
- `sessionStorage` is used for the reconnect token and Terms state, not for
  cryptographic identity keys.
- AES-GCM rejects modified ciphertext and IV. Sequenced payload binding rejects
  replay, reordering, stale-exchange, and cross-session application messages at
  the receiver. The server's bounded in-memory duplicate-envelope window is a
  secondary layer, not the replay protection. A relay can still drop or delay
  traffic.
- The server still sees connection and pairing metadata, message sizes, and
  timing. It does not see message plaintext or key material under normal
  operation.
- The cryptographic implementation has not been independently audited. Do not
  treat AFTERTONE as the sole safeguard for highly sensitive communications.

### Reporting a Vulnerability

Report suspected vulnerabilities privately to [invasiveminds.tech@gmail.com](mailto:invasiveminds.tech@gmail.com). Do not publish vulnerability details in public issues. See [SECURITY.md](SECURITY.md) for what to include and response expectations.

The address is the project owner's designated security contact. Mailbox monitoring and response timelines have not been independently verified.

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 18 or newer
- npm (included with Node.js)

<<<<<<< HEAD
### Installation

Clone the repository and install dependencies:

```bash
git clone https://github.com/quietmade/aftertone.git
cd aftertone
npm ci
```

### Run Locally

Start the server:

```bash
npm start
```

=======
>>>>>>> origin/main
To test messaging, open the application in two browser windows or use two devices that can reach your server.

1. Create an invite on the first device.
2. Join using the invite code on the second device.
3. Accept the connection to start messaging.

### Available Commands

| Command | Description |
| --- | --- |
| `npm start` | Start the application server |
| `npm test` | Run the automated test suite |

The server listens on port `3000` by default. To use a different port:
```bash
PORT=4000 npm start
```

### Configuration

Production must set `NODE_ENV=production` and configure the exact browser
origin(s) that may open a WebSocket connection:

```text
ALLOWED_ORIGINS=https://aftertone-u6rr.onrender.com
```

Multiple origins may be separated by commas. Local development permits
`localhost`, `127.0.0.1`, and `::1` HTTP origins. In production, a missing or
invalid `ALLOWED_ORIGINS` setting fails startup, and a missing or unexpected
browser `Origin` is rejected before a WebSocket connection is allocated.

Optional resource controls use safe defaults when unset:

| Variable | Default | Description |
| --- | ---: | --- |
| `MB88_MAX_CONN` | `2000` | Maximum open WebSocket connections |
| `MB88_RATE_BURST` | `30` | Per-connection message burst |
| `MB88_RATE_PER_SEC` | `5` | Per-connection refill rate |
| `MB88_JOIN_LIMIT` | `20` | Join attempts per connection per minute |
| `MB88_MAX_LIVE_INVITES` | `5` | Live invites per connection |
| `MB88_SESSION_GRACE_MS` | `30000` | Reconnect grace before a browser session, invite, and pair are closed |
| `MB88_MAX_B64` | `512` | Maximum relayed public-key field size (base64 characters) |
| `MB88_MAX_IV_B64` | `64` | Maximum IV field size |
| `MB88_MAX_CIPHERTEXT_B64` | `1024` | Maximum ciphertext field size |
| `MB88_REPLAY_MAX` | `2048` | Duplicate-envelope entries per session |
| `MB88_REPLAY_WINDOW_MS` | `600000` | Duplicate-envelope retention window |
| `MB88_REPLAY_TOTAL_MAX` | `10000` | Aggregate duplicate-envelope entries |

Terms acknowledgements are signed with a server-held key. Production must set
`TERMS_SECRET` to a stable value of at least 32 characters, or
`TERMS_SECRET_FILE` to a persisted server-only file, so a browser's
acknowledgement survives a server process restart. Development and test runs
may use the ignored `.cache/terms-secret` fallback; production never generates
a replacement secret automatically. The acknowledgement is still a
browser-session cookie and a new browser session must accept the current Terms.

The application keeps pairing and replay state in memory. It has no database
or message history; a process restart ends active sessions.

## Deployment preparation

The current application is a Node.js HTTP/WebSocket server. It is not a
Cloudflare Worker, Pages Function, or other edge-native application. See
[DEPLOYMENT.md](DEPLOYMENT.md) for reverse-proxy requirements and the
deployment decisions that remain owner-controlled.

For day-to-day operations — deployment, rollback, source and secret
recovery, smoke tests, monitoring, and known limitations — see the
operational runbook: [docs/OPERATIONS.md](docs/OPERATIONS.md).
Its status against the master checklist is recorded in
[PROJECT-CHECKLIST.md](PROJECT-CHECKLIST.md) Section 10. Production
preparation remains on hold (Section 9); the runbook does not by
itself authorise a deployment.
