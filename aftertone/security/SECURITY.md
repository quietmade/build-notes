# Security Policy

> **Verification status: UNVERIFIED.** This document describes the current source implementation and local test coverage. It is not evidence of independent security review or verified production behavior. The properties below are not production security guarantees.

## Protocol and Security Model

AFTERTONE encrypts message content in the browser before relaying it through
the server. Each paired session uses an ephemeral **P-256 ECDH** key pair,
**HKDF-SHA256**, and **AES-256-GCM** through the browser Web Crypto API. The
public keys are exchanged through the server; the private keys and derived AES
keys remain in the browser process.

Before a session can become ready, each side performs an **HMAC-SHA256 key
confirmation** using a separate HKDF-SHA256-derived confirmation key. The HMAC
covers a canonical transcript containing the server-issued exchange identifier,
both sorted client IDs, both sorted public keys, the connection expiry, and the
sender. This proves that the peer derived the same session key for the same
exchange. It does **not** authenticate the peer's identity: an active relay that
substituted both public keys also knows both derived keys and can produce valid
confirmations.

**The key exchange is not authenticated.** There is no long-term identity key,
signature, or authenticated identity binding. A server or other active relay can
substitute public keys and complete a separate exchange with each participant.
The client displays a 96-bit fingerprint derived locally from both exchanged
public keys plus the pair context, including the server-issued exchange
identifier. Both participants must explicitly VERIFY the same value before the
server relays application messages, and the server accepts a verification report
only after both participants have sent a key confirmation for that exchange.
Verification is still a human out-of-band comparison: it does not authenticate
the peer, establish a trusted identity, or make an active man-in-the-middle
impossible.

Each refresh, reconnect, or new pairing generates a fresh ephemeral key pair.
The session key, confirmation key, and per-direction sequence state are discarded
when the session ends. There is no in-session rekey, ratchet, or additional
forward-secrecy mechanism.

### What the server sees

The server relays base64-encoded P-256 public keys, HMAC key-confirmation tags,
and `{iv, ciphertext}` envelopes. It never receives a private key, the derived
AES key, or plaintext during normal operation. It cannot validate a
key-confirmation tag; it only requires that both participants sent one before it
will coordinate verification. It does see connection and pairing metadata,
message timing and sizes, the browser's network metadata, exchange identifiers,
and the fingerprint/context reports used to coordinate verification. Because it
relays the public keys and coordinates verification, it is in a position to
substitute them in an active attack, deny service, or misreport completion.

### Explicit limitations

- The key exchange is unauthenticated and vulnerable to an active
  man-in-the-middle. Key confirmation proves knowledge of the derived session
  key, not peer identity. Mandatory verification can detect a substituted
  exchange only through an independent fingerprint comparison; this
  implementation does not provide authenticated key exchange or complete MITM
  resistance.
- The fingerprint comparison is a human, out-of-band check. It can detect a
  substitution only when both people compare the full value accurately over a
  channel the attacker does not control. It does not prove identity.
- There is no forward secrecy beyond discarding ephemeral session key material
  when the session ends. Compromise of a session key or browser process can
  expose that session's plaintext.
- AES-GCM rejects modified ciphertext and IV. Each application payload also
  carries an authenticated `{v, exchangeId, sender, recipient, seq, text}`
  structure; the receiver requires an exact exchange/sender/recipient match and
  a strictly increasing sequence per sender per exchange. This rejects replayed,
  reordered, stale, and cross-session application messages at the receive side.
  The server's bounded in-memory duplicate-envelope filter remains a secondary
  layer, not the replay protection. Sequence checks do not prevent a relay from
  delaying or suppressing traffic, and a currently authorized peer can send new
  messages.
- `sessionStorage` is accessible to JavaScript in the same origin and is used
  for reconnect and Terms state. XSS, malicious extensions, or a compromised
  browser can expose application state and plaintext.
- The cryptographic code has not been independently audited.
- A server-side deployment can be old, misconfigured, or unreachable even when
  the source tests pass. Source tests are not evidence about a running
  deployment.

## Reporting a Vulnerability

The project owner has designated the following address for private security reports:

**invasiveminds.tech@gmail.com**

Please report suspected vulnerabilities privately by email. Do not publish sensitive technical details in public issues.

Where possible, include:

- The affected component, route, or file.
- Relevant AFTERTONE version, commit, or deployment information.
- The potential security impact.
- Steps to reproduce the issue.
- Sanitized logs, screenshots, or a proof of concept that does not expose other people's information.

Do not send passwords, private keys, personal messages, credentials, or unnecessary personal data. If sensitive evidence is required, ask for a safer way to provide it before sending it.

## Response Expectations

This address is the project owner's designated security contact. Acknowledgement and remediation timelines are not guaranteed. The project does not currently promise a bug bounty, a specific response time, a coordinated-disclosure deadline, or a guaranteed fix. Any future commitments will be published only when the project owner can support them.

Mailbox monitoring and delivery have not been independently tested or verified as part of this policy.

## Mandatory Fingerprint Verification

AFTERTONE requires both participants to explicitly verify the same session
fingerprint before application messaging is enabled. The fingerprint is
computed locally from both ephemeral public keys plus session context: sorted
client IDs, connection expiry, and the server-issued exchange identifier. The
server only coordinates verification reports; it never sees private keys or
plaintext.

Verification requires an explicit user action (VERIFY). Displaying, copying,
or opening the fingerprint is not verification. One participant's verification
does not verify the other. A verification acknowledgement is bound to the
exact fingerprint, context, and exchange identifier; stale, mismatched,
premature, unsolicited, and duplicate completions are ignored. Reconnection,
key replacement, or session restoration resets verification synchronously, and
an in-flight decryption cannot deliver plaintext after its session is
invalidated.

A client also requires a **valid peer key confirmation** before it reports
`crypto-ready`, displays a verifiable session, or permits a local VERIFY action.
The server rejects a verification report until both participants have sent a
key-confirmation frame for the current exchange. The server can check only that
both frames exist; each client verifies the other's HMAC tag against the
canonical transcript. A forged or tampered tag produces an error and never
unlocks verification.

This remains first-contact, out-of-band verification, not persistent identity
authentication. If users do not compare fingerprints over an independent
trusted channel, an active MITM may still substitute keys. Mandatory
verification does not make MITM attacks impossible.

## Application Message Binding and Replay Protection

Application text is not sent as a bare AES-GCM ciphertext. `client/crypto.js`
seals a versioned structure:

```
{ v: 1, exchangeId, sender, recipient, seq, text }
```

inside the encrypted payload. `client/net.js` opens it only when the decrypted
`exchangeId`, `sender`, and `recipient` exactly match the verified session and
the local client's own ID, and only when `seq` is strictly greater than the last
accepted sequence from that sender in that exchange. A re-encrypted duplicate,
an older/reordered frame, a stale-session payload, or a payload bound to another
exchange is rejected and produces no `message` event. Sequence state resets only
when the session key changes, at which point old ciphertext no longer
authenticates anyway.

This is cryptographic receive-side replay protection for application messages,
not a complete anti-replay network protocol: a relay can still drop, delay, or
observe traffic, and the server's in-memory duplicate filter is only a secondary
defence.

Verification reports are bound to a server-issued exchange identifier for the
current key exchange, and the server will not relay application messages until
both participants have reported the same fingerprint and context for that
exchange. Completion is emitted once per exchange; duplicate reports do not
produce duplicate completions. A client only treats itself as verified after
its own explicit VERIFY action and a matching completion for the current
exchange. A new key exchange, reconnect, or session teardown invalidates the
prior verification and clears the server's reports and exchange state.
The server still coordinates and observes the verification metadata; it never
receives private keys or plaintext, and it remains able to deny service or
misreport completion. Comparing the fingerprint over an independent trusted
channel remains the only way to detect a malicious relay.
