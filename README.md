<p align="center">  
  <img src="https://github.com/user-attachments/assets/9ba2e5d7-2e77-463d-b782-5e55d65078a6" alt="AFTERTONE banner" width="100%">
</p>

<p align="center">
  <em>A retro two-person messaging terminal with browser-side encrypted message relay.</em>
</p>

> *AFTERTONE's code repo will never be public, so its docs, status, bugs, and history live here instead — in a separate repo — keeping the code itself protected and the code repo clean.*

| Document | What it covers |
| --- | --- |
| [Status](aftertone/STATUS.md) | Current state, and the evidence still needed before release | 29 Oct 2026 |
| [Bugs](aftertone/BUGS.md) | Open bugs, closed only with named evidence such as a failing test, a trace, or a written report | 29 Oct 2026 |
| [Changelog](aftertone/CHANGELOG.md) | What changed, with the source-project history kept separate from these documentation notes | 29 Oct 2026 |
| [Architecture](aftertone/architecture/README.md) | System shape, message flow, and the four interactive Archify diagrams | 29 Oct 2026 |
| [Bug report template](.github/ISSUE_TEMPLATE/bug-report.md) | How to report a bug without leaking invite codes, encryption keys, or message content

---

## AFTERTONE Runtime Architecture: Two Clients and Server

Two browser clients and one server. Each client holds its own keys and encrypts messages in the browser; the server only relays encrypted messages and never sees plaintext.

Flow: Terms → Invite and pairing → Key exchange → Bilateral fingerprint verification → Encrypted relay over WebSocket.

Server: Node.js WebSocket relay, with server-snapshotted expiry and origin checks on WebSocket upgrade and Terms acceptance.

> AFTERTONE runtime architecture — browser clients and crypto layers on
> both sides of a Node.js HTTP/WebSocket relay, with the origin guard,
> Terms service, config limits, session manager, in-memory pairing store,
> encrypted relay, and replay guard.
>

| Project | What it is | Documentation | Source code |
|---|---|---|---|
| AFTERTONE | Real-time, two-participant messaging with a retro CRT interface. Two devices pair with a private invite code, messages are encrypted in the browser, the relay sees only ciphertext. | | Private, separate repository — not here, by design |

### How a session works

| Step | What happens | Where |
|---|---|---|
| 1. Create | Device A creates an invite. The relay stores it in memory and returns a code and link. | Browser A + relay |
| 2. Join | Device B opens the link or enters the code; the relay validates the invite. | Browser B + relay |
| 3. Accept | The invited side accepts the connection. The session timer starts when the invite is accepted. | Both browsers |
| 4. Exchange | Each browser generates an ephemeral P-256 ECDH key pair and sends its public key through the relay. | Browsers + relay |
| 5. Confirm | Both sides derive a session key with HKDF-SHA256 and send an HMAC-SHA256 key confirmation over the exchange transcript. | Browsers |
| 6. Verify | Both people compare the 96-bit fingerprint and press VERIFY. The relay enables messaging only after both reports match and both confirmations exist. | Browsers + relay |
| 7. Transmit | The sender seals a bound payload with AES-256-GCM; the relay forwards `{iv, ciphertext}`; the receiver checks session binding and sequence. | Browsers + relay |

### Security model at a glance

| Property | Mechanism | What it gives you | What it does not give you |
|---|---|---|---|
| Message confidentiality | AES-256-GCM in the browser | Ciphertext to the relay and integrity checks on the message | Protection if a device or browser is compromised |
| Key agreement | Ephemeral P-256 ECDH | A fresh shared secret for each session | Authentication of the peer's identity |
| Key derivation | HKDF-SHA256 | Separate encryption and confirmation keys | Protection from a malicious relay that substitutes keys |
| Key confirmation | HMAC-SHA256 over the exchange transcript | Proof the peer derived the same session key | Proof of who the peer is |
| Fingerprint verification | 96-bit value compared out of band | A way to detect a substituted key if both people compare the full value | Cryptographic peer authentication |
| Replay protection | Authenticated sequence binding checked by the receiver | Rejection of duplicate, reordered, stale, and cross-session messages | Protection from traffic suppression or delay |
| Forward secrecy | Ephemeral keys discarded at session end | No long-term session key to recover | A ratchet or post-compromise security |

## What we deliberately did not build

🚫 **Accounts and profiles.** There is no sign-up, no password, and no
identity to recover. Pairing is the only onboarding step.

🚫 **Contact lists and discovery.** You cannot search for a person, browse
a directory, or see who else is online.

🚫 **Message history and search.** Nothing is stored server-side, and the
client does not keep an archive. Closing the session is the delete button.

🚫 **File and image sharing.** The channel carries short text pages, not
attachments. There is no upload path to secure.

🚫 **Group conversations.** Two devices, one line. A third device cannot
join an existing pair.

🚫 **Push notifications.** A notification service would learn that a
message exists and when it arrived. The design avoids that metadata trail.

🚫 **Backups and export.** A backup would outlive the session and the keys
that protected it. There is nothing to export.

🚫 **An admin or support backdoor.** No operator path can read a message
or recover a session key. Support cannot restore what was never stored.

## What "private line" means here

The word *private* carries more weight than the design can support if it
is left unqualified. Three distinctions matter:

**Private is not anonymous.** The relay does not require an account, and
it does not build a contact graph. But the other participant may know who
you are, and the network still sees that two addresses are connected.
Privacy here means the message contents are not readable by the relay,
not that either person is untraceable.

**Private is not unbreakable.** The message layer is encrypted in the
browser. The endpoints are ordinary browsers on ordinary devices. Malware,
a shared screen, a camera pointed at the display, or a compromised
extension defeats the encryption without touching the protocol.

**Private is not audited.** The protocol and the source have been reviewed
internally, and the automated tests are recorded. No independent security
firm has examined the design. Until that changes, the honest label for the
security claims is *unverified*.

## Failure is part of the design

AFTERTONE does not try to hide failure behind retries and durable state.
If the relay restarts, sessions end. If a tab closes, the line is gone. If
a message arrives while the socket is down, it is not queued for later.
If a session expires, there is no archive to reopen.

This is a deliberate trade. A message queue, a history store, and a
reconnect cache would each make the product more convenient and each would
create a new place where plaintext or metadata could accumulate. The
design refuses those trade-offs and documents the cost instead.

What the protocol does provide is narrower: a receiver checks that an
envelope belongs to the current session and to the expected sequence, so
duplicate, reordered, stale, and cross-session messages are rejected. That
is integrity and replay resistance, not delivery. A relay that drops
traffic is behaving within its abilities, and the interface is expected to
show the gap rather than paper over it.
