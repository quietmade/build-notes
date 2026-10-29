# AFTERTONE — Architecture

Two-person, browser-side encrypted messaging. The server relays ciphertext; it does not hold message plaintext.

## Flow

1. Terms acceptance (server-gated, `TERMS_SECRET` on the server)
2. Invite + pairing (two participants, one session)
3. Key exchange, bound to one exchange ID
4. Bilateral fingerprint verification (both sides must verify)
5. Encrypted relay over WebSocket

## Server

- Node.js, WebSocket relay
- D2: server-snapshotted expiry (Option B) — expiry is snapshotted at publication, extension does not mutate it
- Origin checks on WebSocket upgrade and Terms acceptance; `/health` and `/ready` exempt
- CSP: `script-src 'self'` (app script externalised to `client/app.js`)

## Client

- Encryption and key handling in the browser
- Terms, pairing, verification, and messaging UI in `client/`

## Deployment (current)

- Render Free, `beeperline.onrender.com` — suspended, not deleted
- No custom domain, no gateway header, no `GATEWAY_SECRET` in this deployment
- Gateway-secret code exists locally only, on hold for the direct-to-Render route
