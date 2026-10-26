<!-- Adapted from the AFTERTONE public page in the private source repository (faq.html), last updated 23 October 2026. No source code is included. Operator, contact, and governing-law details in TERMS.md remain placeholders to be completed before publication. -->

AFTERTONE · FAQ

 

# FREQUENTLY ASKED QUESTIONS

 

*A few things worth knowing before you connect.*
## WHAT IS AFTERTONE?

 
 
 
 

AFTERTONE is a real-time messaging service designed for two people to connect through an invitation and exchange messages directly. No account creation is required for the intended experience.
## DO I NEED TO CREATE AN ACCOUNT?

 
 
 
 

AFTERTONE is designed to work without traditional user accounts. Connections are established through invitations rather than a conventional username-and-password system.
## IS AFTERTONE FREE TO USE?

 
 
 
 

AFTERTONE is designed to be accessible through its website. Availability may depend on the service remaining operational and on your device, browser, and internet connection.
## HOW DO INVITATION LINKS WORK?

 
 
 
 

An invitation link allows another person to access a connection initiated through AFTERTONE. An unused invitation has no fixed time limit in the current implementation. It remains usable while its creator's session is active and the creator is connected, unless it is consumed or removed by session or connection cleanup. The connection duration you select begins only when the invitation is accepted. Share the link only with the person you intend to communicate with. Anyone who obtains a usable link may be able to attempt to join, subject to the service's validation rules.
## WHAT HAPPENS IF I REFRESH THE PAGE?

 
 
 
 

AFTERTONE can restore the session after a refresh in the same tab, but only within a short reconnect window. The tab keeps a resume token in sessionStorage, and the server holds the session for 30 seconds by default while the page reloads. Within that window, an eligible invitation or paired connection may be restored. A restored connection keeps its existing deadline; an unused invitation has no separate countdown. Messages already displayed are not restored. The resumed session performs a fresh P-256 key exchange, a fresh key confirmation, and derives a new AES key; old session key material is not restored. Verification also resets: both sides must compare the newly displayed fingerprint and press VERIFY again before messaging is re-enabled.

 

If the window has passed, the resume token is unavailable, or the tab was closed, the previous session ends. Do not rely on a refresh to preserve or recover a conversation.
## WHAT HAPPENS IF MY INTERNET CONNECTION DROPS?

 
 
 
 

A network interruption may temporarily disconnect you from the other participant. The browser attempts to reconnect, and the server holds the session during a short grace period (30 seconds by default). If the same browser session reconnects in time, the pair can resume and the browsers perform a fresh key exchange.

 

Messages are not queued for later delivery, so anything sent while the link is down is not relayed. If the grace period expires, the session and paired connection end.
## WHY DOES AN INVITATION LINK STOP WORKING?

 
 
 
 

There is no invitation-specific timeout to wait out. An invitation stops being usable when it is consumed, when its creator's session is cleaned up, when a connection involving the creator expires, or when the service restarts. If the creator explicitly leaves, the creator's session ends and its invitations are removed. A rejected or cancelled join request does not remove the invitation; it returns to AVAILABLE and may be joined again. The 30-second reconnect grace period is a temporary session-recovery window, not an invitation lifetime. If a link fails unexpectedly, check that it is complete and that the invitation has not been consumed or removed.
## ARE MY MESSAGES ENCRYPTED?

 
 
 
 

AFTERTONE uses browser-based cryptography to encrypt message content before it is relayed through the server. Its implementation uses an ephemeral P-256 ECDH key exchange, HKDF-SHA256, and AES-256-GCM. Both sides also send an HMAC-SHA256 key confirmation over the exchange transcript, so neither client treats the session as ready until the peer proves it derived the same key. The exchange is still not authenticated, so an active man-in-the-middle can substitute public keys. Both participants must compare the displayed fingerprint over a trusted channel and press VERIFY; the server accepts verification only after both key confirmations exist and relays messages only after both reports match the current exchange. Verification and key confirmation still do not authenticate the peer's identity. The protection provided depends on the correct operation of the implementation and the security of the devices involved.

 

**Verification status: UNVERIFIED.** This is an implementation description and local test result, not an independent security review or verified production guarantee.
## CAN AN OLD OR DUPLICATE MESSAGE BE REPLAYED?

 
 
 
 

Each encrypted message carries an authenticated version, exchange identifier, sender, recipient, and sequence number. The receiving browser accepts it only when that binding matches the verified session and the sequence number is strictly newer than the last message accepted from that sender. Duplicate, reordered, stale-session, and cross-exchange application messages are rejected after decryption. This protects message delivery within the current session; a relay can still suppress or delay traffic, and a peer authorized in the session can send new messages.
## CAN THE SERVER READ MY MESSAGES?

 
 
 
 

The server is designed to relay public keys, opaque key-confirmation tags, and encrypted envelopes rather than plaintext messages during normal operation. It does not receive private keys or the derived AES key, and it cannot validate the confirmation tags. This does not guarantee protection against every attack: because the key exchange is unauthenticated, an active attacker controlling the relay can substitute public keys and establish separate sessions with each participant. Comparing the fingerprint can help detect that only if both people compare the full value over a channel the attacker does not control and both press VERIFY.
## ARE MY MESSAGES STORED?

 
 
 
 

AFTERTONE is designed for real-time messaging rather than permanent message-history storage. Temporary application state, technical logs, hosting-provider records, and copies retained by recipients are separate considerations. Do not assume that every trace of a message disappears immediately when a connection closes.
## CAN I RECOVER MESSAGES AFTER LEAVING?

 
 
 
 

AFTERTONE is not designed as a permanent message archive. Whether any particular message remains available depends on the application's current behavior and whether a copy exists elsewhere. Do not rely on AFTERTONE to recover messages after a connection ends.
## CAN THE OTHER PERSON SAVE OR SHARE MY MESSAGES?

 
 
 
 

Yes. A recipient may copy, photograph, record, or share content they can access. Encryption does not prevent a recipient from retaining information after receiving it.
## DOES AFTERTONE MAKE ME ANONYMOUS?

 
 
 
 

No service should be assumed to provide complete anonymity. Technical information such as IP addresses, connection events, and request details may be processed by the server or its hosting infrastructure, depending on their configuration.
## HOW DOES AFTERTONE PROTECT MY CONNECTION?

 
 
 
 

AFTERTONE uses invitation-based connections and WebSocket communication. The security of a connection also depends on invitation validation, transport configuration, cryptographic key exchange, and the security of the devices involved.
## CAN SOMEONE INTERCEPT MY COMMUNICATION?

 
 
 
 

Encryption is intended to protect message content, but it cannot eliminate every risk. Because the key exchange is unauthenticated, an active attacker controlling the relay may be able to substitute keys. Comparing the fingerprint through a trusted channel and pressing VERIFY on both devices can help detect that, but verification is still a human comparison, not identity authentication. Keep your browser updated, protect your device, and confirm the identity of your intended recipient when necessary.
## WHAT SHOULD I DO IF I FIND A SECURITY VULNERABILITY?

 
 
 
 

Please report suspected vulnerabilities privately by email to invasiveminds.tech@gmail.com. Do not publish vulnerability details publicly before the issue has been investigated.

 

Include the affected component, potential impact, reproduction steps, and relevant version information where possible. Do not send passwords, private keys, personal messages, or unnecessary personal data. Acknowledgement and remediation timelines are not guaranteed.
## WHY CAN'T I CONNECT?

 
 
 
 

Check your internet connection and make sure you are using a valid invitation. Temporary network problems, an unavailable session, browser restrictions, or a service interruption may prevent a connection from being established.
## WHY AM I SEEING AN OFFLINE MESSAGE?

 
 
 
 

An offline message generally indicates that the application cannot currently communicate with the server. Check your connection and try again when the service is reachable. The exact behavior may vary with the current application state.
## WHAT SHOULD I DO IF AFTERTONE STOPS RESPONDING?

 
 
 
 

Wait briefly, check your network connection, and try again if it is safe to do so. Refreshing may interrupt an existing session and will not restore messages already displayed, so avoid refreshing if preserving the current connection is important.
## WHAT AM I ALLOWED TO USE AFTERTONE FOR?

 
 
 
 

AFTERTONE is intended for lawful, responsible communication. It must not be used to promote violence, facilitate criminal activity, threaten others, or engage in other unlawful conduct. Please review the Terms of Service for the applicable rules.
## WHERE CAN I LEARN MORE ABOUT PRIVACY?

 
 
 
 

The Privacy Notice explains the information AFTERTONE may process, how it may be used, and the limitations of its privacy protections.
## WHERE CAN I LEARN MORE ABOUT SECURITY?

 
 
 
 

The Security Overview describes AFTERTONE's security design, encryption mechanisms, known limitations, and vulnerability-reporting process.
## WHERE CAN I READ THE TERMS OF SERVICE?

 
 
 
 

The Terms of Service explain the conditions and responsibilities associated with using AFTERTONE.

 
 
 

 
 

## STILL CURIOUS?

 

Privacy deserves transparency. Security deserves honesty. Communication deserves care.

 

Explore the details before you connect.

 
 

**AFTERTONE — Less noise. More meaning.**
