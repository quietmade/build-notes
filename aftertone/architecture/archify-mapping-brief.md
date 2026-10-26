# AFTERTONE — Archify Mapping Brief

## Purpose

Use Archify to inspect the **actual current AFTERTONE repository** and build a trustworthy site map / architecture map showing how the project is organized, how its components connect, and where the remaining work belongs.

Read this brief before generating the map. This brief is not a substitute for repository evidence.

## Project Context

AFTERTONE is a real-time, two-participant messaging application with a retro CRT-inspired interface. Its reported implementation includes WebSocket-based session coordination, ephemeral key exchange, AES-GCM encrypted application messages, mandatory bilateral fingerprint verification, cryptographic key confirmation, and receive-side application-message replay/order checks.

The latest project report states that automated verification passed:
- Targeted security/lifecycle suites: 194/194
- Full suite run 1: 236/236
- Full suite run 2: 236/236
- Browser tests: 14/14
- Linux WebKit tests: 14/14
- Regression tests: 31/31
- Concurrent-operation repetitions: 15/15
- Reconnect-race regression repetitions: 10/10

These are **reported results**, not results Archify should claim to have executed. Inspect repository artifacts to establish what is actually present. Do not rerun tests as part of mapping unless separately instructed.

## Security Context and Limitations

The latest report describes these behaviors:
- Both participants must complete mandatory fingerprint verification before messaging is authorized.
- HMAC-SHA256 key confirmation uses a separate HKDF-SHA256-derived confirmation key.
- The confirmation transcript binds the server-issued exchange ID, both client IDs, both public keys, frozen connection expiry, and sender.
- The server tracks confirmations and requires both before accepting verification reports.
- Application messages bind version, exchange ID, sender, recipient, sequence number, and text.
- Receivers reject duplicate, reordered, stale-session, and cross-exchange messages.
- Session trust is invalidated on new exchanges, reconnects, and teardown.
- Reconnect and stale lifecycle-notice race fixes are reported.

Known limitations from the latest report:
- Key exchange is **not authenticated** and complete MITM resistance is not established.
- Key confirmation proves possession of the derived key, not peer identity.
- Fingerprint comparison through an independent trusted channel is still required to detect substituted exchanges.
- A relay can delay, suppress, or observe traffic.
- No ratchet or in-session rekey is implemented.
- An independent security audit has not been completed.
- Physical iOS, physical Android, and real Safari/macOS testing were unavailable and remain pending.
- Linux WebKit results must not be represented as real Safari testing.

Treat these as context from the latest report. Check relevant source and documentation before attributing behavior to a particular file or component. Flag mismatches rather than silently resolving them.

## Remaining Work to Map

Connect each remaining task to the actual repository components, interfaces, and dependencies Archify discovers.

### A. Real Safari / macOS compatibility
- Identify browser-facing entry points, client initialization, WebSocket lifecycle, crypto initialization, and pairing/verification UI flows.
- Identify existing browser and WebKit test coverage and what it does not establish about real Safari.
- Mark real Safari/macOS execution as an external requirement if no suitable environment exists.

### B. Physical iOS testing
- Identify entry points and flows relevant to phone-sized viewports, pairing, fingerprint verification, refresh/reconnect, backgrounding/foregrounding, expiry, and stale-message rejection.
- Identify existing coverage and missing evidence.
- Mark physical iOS execution pending unless a real device and test environment are available.

### C. Physical Android testing
- Map the relevant Android device/browser flows.
- Identify platform-sensitive lifecycle or browser APIs from actual code.
- Mark physical Android execution pending unless a real device and test environment are available.

### D. Manual end-to-end verification
Map each scenario to relevant source components and available automated tests:
- Successful bilateral pairing and verification.
- One-sided verification must not enable messaging.
- Mismatched fingerprints must block messaging.
- Refresh and reconnect.
- Backgrounding and foregrounding.
- Exchange expiry.
- Stale messages and old-session notices must not affect a newer exchange.
- Reverification when required by the protocol.

### E. Independent security audit
- Map cryptographic protocol, key derivation and confirmation, fingerprint transcript, server relay authorization, message envelope/replay checks, session lifecycle, and related tests/docs.
- Identify trust boundaries and critical data flows.
- Record the unauthenticated key-exchange limitation explicitly.
- Do not claim the application is secure, audited, or MITM-resistant merely because automated tests pass.

### F. Production readiness (mapping only; not authorized to execute)
- Map areas that would need a future deployment review: transport security, hosting/runtime configuration, environment configuration, logging, monitoring, and incident response.
- Do not change production configuration or infer that a deployment architecture is finalized.

## Archify Mapping Requirements

1. Inspect the actual current repository, including relevant source, tests, documentation, and configuration.
2. Generate a site/architecture map based on observed imports, calls, events, protocol messages, state transitions, and data flows.
3. Clearly distinguish:
   - Verified in source
   - Reported by the latest project report but not independently verified
   - Inferred relationship
   - External/manual requirement
   - Unknown or unavailable evidence
4. For every remaining task, include:
   - Task name
   - Why it remains
   - Relevant files/components, discovered rather than guessed
   - Main dependencies and flow
   - Existing automated coverage
   - Missing evidence
   - Practical next action
   - Objective completion criteria
5. Link security-sensitive flows across client, server, cryptography, tests, and documentation.
6. Highlight entry points, trust boundaries, security-critical state, and lifecycle/race-sensitive paths.
7. Do not invent filenames, symbols, APIs, relationships, or test coverage.
8. Do not mark external device/browser testing complete based on simulated or Linux-only results.
9. Do not treat mapping as permission to implement changes.

## Guardrails

- Mapping and documentation analysis only; do not modify application source code.
- Do not install packages or add dependencies.
- Do not run tests unless separately requested.
- Do not commit, amend, push, pull, merge, deploy, or change DNS, hosting, TLS, Cloudflare, or production configuration.
- Preserve all existing tracked, untracked, and unrelated working-tree changes.
- Do not redesign the UI or perform unrelated refactoring.
- Do not reopen completed implementation work without a concrete repository discrepancy or evidence of a defect.

## Expected Output

Produce:
1. An architecture/site map of the actual repository.
2. A task-to-component map for all remaining work above.
3. A prioritized, dependency-aware roadmap for the next steps.
4. A short evidence ledger separating verified source facts, reported results, inferences, and external blockers.
5. Unknowns that cannot be resolved from the current repository/environment.

The map must explain **what remains, where it belongs, why it matters, what must happen next, and how completion will be demonstrated**.

Do not implement the remaining tasks. Finish with the map and roadmap only.
