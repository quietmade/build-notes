<!-- Adapted from the AFTERTONE public page in the private source repository (security.html), last updated 23 October 2026. No source code is included. Operator, contact, and governing-law details in TERMS.md remain placeholders to be completed before publication. -->

AFTERTONE · SECURITY

 

# SECURITY

 

Last updated: 23 October 2026

 

**Verification status: UNVERIFIED.** This page describes the current source implementation and local
 tests. The protocol has not been independently reviewed or verified against a live deployment. This is not an
 authenticated end-to-end encryption design, and these are not production security guarantees.

 

## MESSAGE ENCRYPTION

 

AFTERTONE uses browser-based cryptography to encrypt message content before it is relayed through the server.

 

The implementation uses the browser Web Crypto API with the following mechanisms:

 

 
- **Key agreement:** an ephemeral P-256 ECDH key pair per paired session.

 
- **Key derivation:** HKDF-SHA256 derives a 256-bit AES key from the shared ECDH secret.

 
- **Key confirmation:** each side sends an HMAC-SHA256 tag over the exchange transcript using a separate HKDF-derived key, proving it derived the same session key.

 
- **Message encryption:** AES-256-GCM with a fresh random 96-bit IV for each message. The encrypted payload binds the exchange, sender, recipient, and a per-sender sequence number.

 

 

These mechanisms are intended to keep plaintext message content away from the server during normal relay. Key
 confirmation proves possession of the derived key, not who the peer is. The mechanisms do not establish identity.

 

The protection provided by encryption depends on the correctness of the implementation, the security of the key
 exchange, and the integrity of the devices and software involved.

 

## KEY EXCHANGE AND FINGERPRINT COMPARISON

 

Before encrypted communication can take place, the participating browsers generate ephemeral P-256 key pairs and
 exchange their public keys through the server.

 

**The exchange is not authenticated.** There is no long-term identity key, signature, or
 authenticated identity binding. An active attacker who controls the relay can substitute its own public keys and
 complete a separate exchange with each participant.

 

Before a session becomes ready, both sides send an HMAC-SHA256 key confirmation over the server-issued exchange
 identifier, both sorted client IDs, both sorted public keys, the connection expiry, and the sender. A client will
 not display a verifiable session or accept a local VERIFY action until the peer's confirmation tag validates. The
 server requires that both confirmation frames exist before it will coordinate verification, but it cannot validate
 the tags. This proves key possession, not identity: a relay that substituted both keys would know both derived keys.

 

To make substitution detectable by a human, AFTERTONE derives a **96-bit fingerprint** from both
 exchanged public keys plus the pair context and displays the same value to both clients. Compare it through a
 separate trusted channel, such as in person or by voice.

 

Both participants must press **VERIFY** on the same fingerprint before the server relays application
 messages. One-sided verification is not enough, and a mismatched or stale completion does not enable messaging.
 Even a completed comparison does not authenticate the peer or cryptographically confirm that the relay was honest;
 it can detect an active exchange only when both people compare the full value accurately over a channel the
 attacker does not control.

 

Each refresh, reconnect, or new pairing generates a fresh ephemeral key pair and resets verification. The previous
 session key and authorization state are discarded when the session ends.

 

## FORWARD SECRECY AND KEY ROTATION

 

Ephemeral per-session keys mean that a later session does not reuse the previous session's ECDH key pair.
 However, there is no in-session rekey, ratchet, or additional forward-secrecy mechanism.

 

Compromise of a live browser process or session key can expose the plaintext of that session. There is no
 long-term static identity key to compromise, because the implementation does not use one.

 

## MESSAGE HANDLING

 

AFTERTONE is designed for real-time message relay rather than permanent message-history storage.

 

The server facilitates connections and relays encrypted message data between participants. Connection-management
 information may be held temporarily in server memory to support service operation.

 

This design reduces the need for permanent application-level message storage, but it does not establish that all
 information disappears immediately when a connection closes.

 

Technical logs, hosting-provider records, browser data, and information retained by recipients may exist
 separately.

 

Users should not assume that messages can be recalled or erased from a recipient's device after delivery.

 

## CONNECTION SECURITY

 

AFTERTONE uses invitation-based connections and WebSocket communication to facilitate messaging.

 

Connection security depends on the correct validation of invitations, session state, incoming messages, and
 connection transitions.

 

The service may implement protocol checks and other safeguards to reject invalid or unexpected messages. However,
 the existence and effectiveness of any specific safeguard must be evaluated against the current implementation.

 

AES-GCM rejects modified ciphertext and IV. Each application payload also contains an authenticated structure
 binding the exchange identifier, sender, recipient, sequence number, and text. The receiving client accepts a
 message only when that binding exactly matches the verified session and the sequence number is strictly newer than
 the last one accepted from that sender in that exchange. Re-encrypted duplicates, reordered messages, stale-session
 payloads, and cross-exchange payloads are rejected. This is receive-side message replay protection; a relay can
 still drop or delay traffic. The server's bounded in-memory duplicate-envelope window is a secondary check, not
 the replay defence.

 

Invitation links should be shared only with intended recipients. Anyone who obtains a usable invitation may be
 able to attempt to access the associated connection, subject to the service's validation rules.

 

Users should avoid sharing invitation links publicly or through channels they do not trust.

 

## TRANSPORT SECURITY

 

The security of network communication depends on the transport configuration used to access AFTERTONE.

 

When deployed over HTTPS and secure WebSockets (WSS), transport encryption helps protect network traffic against
 interception and modification while it travels between the browser and the service.

 

Users should verify that they are accessing the legitimate service through its intended secure origin.

 

Transport encryption and browser-based message encryption address different risks. Neither can guarantee
 protection against compromised endpoints, malicious software, or every form of active attack.

 

The production deployment must be verified before any specific transport-security configuration is represented as
 guaranteed.

 

## BROWSER AND DEVICE SECURITY

 

AFTERTONE's browser-based security measures operate within the user's browser environment.

 

They cannot guarantee the security of a device or protect against every threat outside the application's control.

 

Potential risks include:

 

 
- Malware or other malicious software on a device.

 
- Malicious or compromised browser extensions.

 
- Vulnerabilities in the browser or operating system.

 
- Someone gaining unauthorized access to an unlocked device.

 
- A recipient copying, recording, photographing, or sharing message content.

 
- A compromised or modified version of the application.

 

 

Users should keep their browsers and operating systems updated, protect access to their devices, and exercise
 care when sharing sensitive information.

 

## SECURITY LIMITATIONS

 

AFTERTONE aims to provide privacy-conscious communication, but its security protections have limits.

 

In particular:

 

 
- Encryption does not guarantee complete anonymity.

 
- The key exchange is unauthenticated. An active man-in-the-middle can substitute public keys and establish
 separate sessions with each participant. Key confirmation proves knowledge of the derived key, not peer identity.

 
- The service cannot prove that a participant is who they claim to be. Comparing the fingerprint can detect a
 substitution only when both people do so accurately over a trusted channel.

 
- Both participants must press VERIFY to enable messaging, but the acknowledgement still does not authenticate
 the peer's identity or prove that the relay did not substitute keys.

 
- There is no in-session rekey, ratchet, or forward secrecy beyond discarding each session's ephemeral key
 material.

 
- Message replay and ordering protections apply to the current session. A relay can still suppress or delay
 traffic, and a peer authorized in the session can send new messages.

 
- SessionStorage, browser extensions, malware, and XSS can expose session state or plaintext on a compromised
 device.

 
- Network and connection metadata may still be processed by the service or its infrastructure providers.

 
- A recipient may retain information received through the service.

 
- Security depends on the implementation, deployment configuration, and software dependencies.

 
- The cryptographic implementation has not been independently audited.

 
- No system can promise complete protection against every vulnerability or attack.

 

 

AFTERTONE should not be treated as a guarantee of protection against targeted attacks or as the sole safeguard
 for information requiring independently verified identities and stronger security assurances.

 

## REPORTING A SECURITY VULNERABILITY

 

We welcome responsible reports from security researchers and users who identify potential vulnerabilities in
 AFTERTONE.

 

If you believe you have discovered a security issue, please report it privately rather than publishing technical
 details that could expose users to harm.

 

**Preferred reporting channel:**

 

 
- **Security email:** invasiveminds.tech@gmail.com

 

 

This address is the project owner's designated security contact. Mailbox monitoring and
 response availability have not been independently verified.

 

### WHAT TO INCLUDE IN A REPORT

 

Where possible, please provide:

 

 
- A concise description of the suspected vulnerability.

 
- The affected component, route, or file.

 
- The potential security impact.

 
- Reproduction steps.

 
- Relevant logs, screenshots, or a proof of concept that does not expose real users or their private
 information.

 
- Any relevant environment or configuration details.

 

 

Do not send passwords, private keys, personal messages, credentials, or unnecessary personal data. If sensitive
 evidence is required, ask for a secure way to provide it before sending it.

 

### RESPONSIBLE DISCLOSURE

 

Please give the maintainer a reasonable opportunity to investigate and address a reported issue before publicly
 disclosing technical details.

 

Do not access, modify, or extract another person's data; disrupt the service; or conduct testing beyond what is
 necessary to demonstrate a suspected vulnerability.

 

AFTERTONE does not currently promise a specific response time, resolution deadline, or financial reward. Any
 future commitments will be published only when the maintainer can support them.

 

## SECURITY UPDATES

 

Security issues may be addressed through code changes, configuration changes, dependency updates, or deployment
 changes, depending on their nature.

 

Where appropriate, important security fixes may be communicated through repository updates, release notes, or
 security advisories.

 

Users should keep their browsers updated and use the legitimate, current version of the service.

 

A fix in the source code does not necessarily mean that the production service has been updated. Deployment
 status must be verified separately.

 

## SCOPE AND CONTACT

 

This page describes AFTERTONE's intended security approach and the limitations users should understand when using
 the service.

 

It does not constitute a guarantee that the service is free from vulnerabilities or that every security measure
 is effective in every environment.

 

For security-related questions or vulnerability reports, use the security email listed in the
 vulnerability-reporting section.

 

*AFTERTONE — Security through thoughtful design, transparent limitations, and responsible
 improvement.*
