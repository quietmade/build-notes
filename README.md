<p align="center">
  <img src="https://github.com/user-attachments/assets/9ba2e5d7-2e77-463d-b782-5e55d65078a6" alt="AFTERTONE banner" width="100%">
</p>

<p align="center">
  <em>A retro two-person messaging terminal with browser-side encrypted message relay.</em>
</p>

> *AFTERTONE's code repo will never be public, so its docs, status, bugs, and history live here instead — in a separate repo — keeping the code itself protected and the code repo clean.*

## Table of Contents

| Document | What it covers | Updated |
| --- | --- | --- |
| [Status](aftertone/STATUS.md) | Current state, open items, known limitations, and the evidence still needed before release | 29 Oct 2026 |
| [Bugs](aftertone/BUGS.md) | Open bugs, closed only with named evidence such as a failing test, a trace, or a written report | 29 Oct 2026 |
| [Changelog](aftertone/CHANGELOG.md) | What changed, by date, with the source-project history kept separate from these documentation notes | 29 Oct 2026 |
| [Architecture](aftertone/architecture/README.md) | System shape, message flow, trust boundaries, and the four interactive Archify diagrams | 29 Oct 2026 |
| [Bug report template](.github/ISSUE_TEMPLATE/bug-report.md) | How to report a bug without leaking invite codes, encryption keys, or message content | 29 Oct 2026 |

---

<table>
  <tr>
    <td width="50%">https://github.com/user-attachments/assets/d4314c99-09d7-4212-b640-4af4471479d3</td>
  </tr>
</table>

## AFTERTONE Runtime Architecture: Two Clients and Server

Two browser clients and one server. Each client holds its own keys and encrypts messages in the browser; the server only relays encrypted messages and never sees plaintext.

Flow: Terms → Invite and pairing → Key exchange → Bilateral fingerprint verification → Encrypted relay over WebSocket.

Server: Node.js WebSocket relay, with server-snapshotted expiry and origin checks on WebSocket upgrade and Terms acceptance.
