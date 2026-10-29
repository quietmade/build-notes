# AFTERTONE — Architecture

Self-contained HTML diagram exports. GitHub does not render these pages inline — download the file and open it in a browser.

| Diagram | File | What it shows |
|---|---|---|
| System architecture | [archify/aftertone-architecture.html](archify/aftertone-architecture.html) | How the two devices, the relay, and the browser encryption fit together |
| Pair lifecycle | [archify/aftertone-pair-lifecycle.html](archify/aftertone-pair-lifecycle.html) | Invite, pair, key exchange, fingerprint verification, messaging |
| Message flow | [archify/message-flow.html](archify/message-flow.html) | Typed text → browser encryption → ciphertext relay → printed delivery |
| Remap / re-pairing | [archify/aftertone-remap.html](archify/aftertone-remap.html) | What happens when a device is replaced or re-paired |

These files were re-downloaded from the previous documentation set on 2026-10-29 and verified byte-identical before being placed here. They are kept unmodified; commentary lives in this file, not inside the exports.

In one paragraph: two devices pair with a private invite code, exchange keys bound to a single exchange ID, both sides compare a fingerprint over an independent channel and press VERIFY, and only then is messaging enabled. Messages are encrypted in the browser (P-256 ECDH, HKDF-SHA256, AES-256-GCM) and the server relays ciphertext only, keeping all state in memory. No accounts, no database, no message history.
