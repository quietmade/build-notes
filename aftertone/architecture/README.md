# AFTERTONE — Architecture

How the system fits together, in notes — no diagram files are stored in this repository (they were removed from the example file system at owner request).

In one paragraph: two devices pair with a private invite code, exchange keys bound to a single exchange ID, both sides compare a fingerprint over an independent channel and press VERIFY, and only then is messaging enabled. Messages are encrypted in the browser (P-256 ECDH, HKDF-SHA256, AES-256-GCM) and the server relays ciphertext only, keeping all state in memory. No accounts, no database, no message history.

Key points to remember when writing about the architecture:

- **Pairing:** private invite code, one exchange ID per key exchange.
- **Verification:** bilateral fingerprint comparison over an independent channel, both sides press VERIFY before messaging is enabled. This is the mitigation for the unauthenticated key exchange — a malicious relay could otherwise substitute keys.
- **Relay:** sees only ciphertext (`{ iv, ciphertext }`), never plaintext or keys.
- **State:** server state is in memory only; there is no message history to leak.
- **Audit status:** the cryptography has not been independently audited. Do not describe this design as audited or proven secure beyond what [STATUS.md](../STATUS.md) and [BUGS.md](../BUGS.md) actually record.
