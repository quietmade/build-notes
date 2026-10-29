<p align="center">
  <img src="https://github.com/user-attachments/assets/9ba2e5d7-2e77-463d-b782-5e55d65078a6" alt="AFTERTONE banner" width="100%">
</p>

<p align="center">
  <em>A retro two-person messaging terminal with browser-side encrypted message relay.</em>
</p>

> *AFTERTONE's code repo will never be public, so its docs, status, bugs, and history live here instead — in a separate repo — keeping the code itself protected and the code repo clean.*

## Table of Contents

<table width="100">
  <thead>
    <tr>
      <th align="left" width="30%">Document</th>
      <th align="left" width="50%">What it covers</th>
      <th align="left" width="20%">Updated</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="aftertone/STATUS.md">Status</a></td>
      <td>Current state and open items</td>
      <td>29 Oct 2026</td>
    </tr>
    <tr>
      <td><a href="aftertone/BUGS.md">Bugs</a></td>
      <td>Open bugs, closed only with named evidence</td>
      <td>29 Oct 2026</td>
    </tr>
    <tr>
      <td><a href="aftertone/CHANGELOG.md">Changelog</a></td>
      <td>What changed, by date</td>
      <td>29 Oct 2026</td>
    </tr>
    <tr>
      <td><a href="aftertone/architecture/README.md">Architecture</a></td>
      <td>Architecture notes</td>
      <td>29 Oct 2026</td>
    </tr>
    <tr>
      <td><a href=".github/ISSUE_TEMPLATE/bug-report.md">Bug report template</a></td>
      <td>How to report a bug</td>
      <td>29 Oct 2026</td>
    </tr>
  </tbody>
</table>

---

https://github.com/user-attachments/assets/d4314c99-09d7-4212-b640-4af4471479d3

## AFTERTONE Runtime Architecture: Two Clients and Server

Two browser clients and one server. Each client holds its own keys and encrypts messages in the browser; the server only relays encrypted messages and never sees plaintext.

Flow: Terms → Invite and pairing → Key exchange → Bilateral fingerprint verification → Encrypted relay over WebSocket.

Server: Node.js WebSocket relay, with server-snapshotted expiry and origin checks on WebSocket upgrade and Terms acceptance.
