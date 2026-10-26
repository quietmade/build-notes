# AFTERTONE Physical Device Testing Checklist

This checklist is for manual testing on physical iOS and Android devices. Execute only if real devices are accessible. Do not use emulators or browser emulation as substitutes for physical-device testing.

## Status as of 2026-10-25 — BLOCKED, no item marked complete

Recorded from the Linux VM used for the pre-production review (not from a
device): no macOS/Safari environment, no physical iPhone/iPad, and no
physical Android device was available to this environment, and no device
was attached (no adb/idevice tooling, no Android/iOS device node). All
156 checklist items below therefore remain unchecked. Nothing in this
note is a pass, a fail, or a substitute result.

- **Real Safari/macOS — BLOCKED.** Required: a real Mac running Safari
  (record Safari and macOS versions). Linux WebKit (Playwright) must NOT
  be reported as Safari on macOS. Evidence to collect: pairing,
  bilateral verification, bidirectional messaging, refresh/reconnect,
  session expiry, browser console errors, network failures, each with
  version numbers.
- **Physical iOS — BLOCKED.** Required: two physical iPhones/iPads
  (or one, paired against a second device). Evidence to collect: the
  scenarios in sections 1–16 below, plus device model, iOS version,
  browser and browser version, and screenshots/console output for any
  failure. Objective completion: every section's items checked with
  recorded device/OS/browser details, or a failure record with steps
  to reproduce.
- **Physical Android — BLOCKED.** Required: physical Android device(s)
  under the same conditions as iOS. Evidence and completion criteria
  are the same as for iOS, with Android version and browser version
  recorded instead.
- **Desktop emulation is not a substitute** for any of the above, and
  automated browser tests (Firefox/WebKit in CI or locally) do not
  close these items.

## Prerequisites

- Two physical devices (iOS and/or Android)
- Both devices on the same network OR on different networks to test cross-network scenarios
- Access to AFTERTONE deployment (local or production)

## Test Execution

### 1. Initial Application Load

**Device A:**
- [ ] Open browser and navigate to AFTERTONE URL
- [ ] Verify page loads completely
- [ ] Verify Terms dialog appears (first-time users)
- [ ] Accept Terms
- [ ] Verify UI renders correctly (CRT terminal, keyboard, status LEDs)
- [ ] Verify no console errors

**Device B:**
- [ ] Repeat above steps

### 2. Create and Join Invitation

**Device A (Creator):**
- [ ] Press INVITE button
- [ ] Select duration (e.g., 60 MIN)
- [ ] Verify invitation code is displayed (format: MB88-XXXX-XXXX)
- [ ] Verify invitation URL is displayed
- [ ] Note the invitation code

**Device B (Joiner):**
- [ ] Enter `/connect [CODE]` in composer
- [ ] Press TRANSMIT
- [ ] Verify "LINK ● WAITING" or "LINK ● INCOMING" status

**Device A:**
- [ ] Verify connection request appears
- [ ] Press ACCEPT button
- [ ] Verify "LINK ● CONNECTED" status
- [ ] Verify session key fingerprint is displayed

**Device B:**
- [ ] Verify "LINK ● CONNECTED" status
- [ ] Verify session key fingerprint is displayed
- [ ] Compare fingerprints with Device A (must match)
- [ ] Verify no key-confirmation or crypto error notice is shown
- [ ] Verify the key state reads "UNVERIFIED — COMPARE WITH PEER" on both devices

### 3. Mandatory Fingerprint Verification

**Before either device presses VERIFY:**
- [ ] Attempt to send a message from Device A
- [ ] Verify messaging is blocked and no plaintext appears on Device B

**Device A:**
- [ ] Press VERIFY
- [ ] Verify key state reads "VERIFIED BY YOU — WAITING FOR PEER"
- [ ] Attempt to send a message; verify it is still blocked because B has not verified

**Device B:**
- [ ] Press VERIFY
- [ ] Verify key state reads "✓ VERIFIED — MESSAGING ENABLED" on both devices

**Both Devices:**
- [ ] Send a message in each direction and verify delivery now succeeds

**Mismatched fingerprint (requires a controlled mismatch build or test harness):**
- [ ] Force the two devices to display different fingerprints
- [ ] Press VERIFY on both devices
- [ ] Verify neither side enables messaging and no plaintext is delivered

### 4. Messaging in Both Directions

**Device A to B:**
- [ ] Type a message: "Testing from A"
- [ ] Press TRANSMIT
- [ ] Verify message appears on right side of Device A
- [ ] Verify TX LED flashes on Device A

**Device B:**
- [ ] Verify message appears on left side
- [ ] Verify delivery animation plays

**Device B to A:**
- [ ] Type a message: "Reply from B"
- [ ] Press TRANSMIT
- [ ] Verify message appears on right side of Device B
- [ ] Verify TX LED flashes on Device B

**Device A:**
- [ ] Verify message appears on left side
- [ ] Verify delivery animation plays

### 5. Refresh and Session Restoration

**Device A:**
- [ ] Refresh the browser page
- [ ] Verify Terms dialog does NOT appear (already accepted)
- [ ] Verify "LINK ● CONNECTED" status is restored
- [ ] Verify new session key fingerprint is displayed
- [ ] Verify key state resets to "UNVERIFIED — COMPARE WITH PEER"
- [ ] Compare the new fingerprint with Device B (must match and differ from the original)

**Both Devices:**
- [ ] Attempt to send before either side re-verifies; verify it is blocked
- [ ] Press VERIFY on both devices
- [ ] Verify "✓ VERIFIED — MESSAGING ENABLED" on both devices
- [ ] Send message: "After A refresh"

**Device B:**
- [ ] Verify new fingerprint appears (must be different from original)
- [ ] Verify message from A is received
- [ ] Send reply: "B received refresh message"

**Device A:**
- [ ] Verify reply is received

### 6. Background and Foreground (Mobile Lifecycle)

**Device A:**
- [ ] Press home button or switch to another app (background)
- [ ] Wait 10 seconds
- [ ] Return to AFTERTONE (foreground)
- [ ] Verify connection state
- [ ] If a new exchange occurred, verify the fingerprint changed and messaging stays blocked until both devices VERIFY again
- [ ] Send message: "After backgrounding"

**Device B:**
- [ ] Verify message is received

**Device B:**
- [ ] Repeat background/foreground test
- [ ] Send message to Device A
- [ ] Verify Device A receives it

### 7. Lock and Unlock

**Device A:**
- [ ] Lock the device (screen off)
- [ ] Wait 30 seconds
- [ ] Unlock the device
- [ ] Return to AFTERTONE
- [ ] Verify connection state (may show RECONNECTING briefly)
- [ ] Send message: "After lock/unlock"

**Device B:**
- [ ] Verify message is received

### 8. Network Switch (if practical)

**Device A:**
- [ ] If possible, switch from Wi-Fi to cellular data (or vice versa)
- [ ] Observe connection status (should show RECONNECTING)
- [ ] Wait for reconnection
- [ ] Verify "LINK ● CONNECTED" is restored
- [ ] If the exchange rotated, compare the new fingerprint and VERIFY on both devices before messaging
- [ ] Send message: "After network switch"

**Device B:**
- [ ] Verify message is received

### 9. Disconnect and Reconnect

**Device A:**
- [ ] Press DISCONNECT button
- [ ] Verify "LINK ● OFFLINE" status

**Device B:**
- [ ] Verify "LINK ● OFFLINE" status
- [ ] Verify peer-disconnected notification

**Reconnect:**
- [ ] Device A: Create new invitation
- [ ] Device B: Join new invitation
- [ ] Device A: Accept connection
- [ ] Verify both devices show CONNECTED
- [ ] Verify new fingerprints (different from previous session)
- [ ] Verify both devices reset to UNVERIFIED and messaging is blocked
- [ ] Compare fingerprints, press VERIFY on both devices, and confirm messaging enables
- [ ] Exchange messages to confirm new session works

### 10. Keyboard Behavior and Input

**Both Devices:**
- [ ] Test on-screen keyboard typing
- [ ] Test physical keyboard (if device has one)
- [ ] Type 160-character message (verify counter shows 160/160)
- [ ] Try typing beyond 160 characters (verify cannot exceed limit)
- [ ] Test SPACE bar
- [ ] Test DEL (backspace)
- [ ] Test TRANSMIT (Enter)
- [ ] Verify LCD updates correctly as you type

### 11. Viewport and Layout

**Both Devices (Portrait):**
- [ ] Verify CRT terminal fits viewport
- [ ] Verify keyboard is fully visible
- [ ] Verify status bar is visible
- [ ] Verify link panel is accessible
- [ ] Scroll if needed and verify no layout breakage

**Both Devices (Landscape):**
- [ ] Rotate device to landscape
- [ ] Verify layout adapts appropriately
- [ ] Verify all UI elements remain accessible
- [ ] Test typing and messaging in landscape
- [ ] Rotate back to portrait
- [ ] Verify layout returns to normal

### 12. Button Placement and Touch Targets

**Both Devices:**
- [ ] Test all buttons in link panel (INVITE, COPY, DISCONNECT, ACCEPT, REJECT, CLOSE, INFO, ADD TIME)
- [ ] Verify touch targets are adequate (not too small)
- [ ] Verify no accidental double-taps
- [ ] Test keyboard buttons (all letters, SPACE, DEL, TRANSMIT)
- [ ] Verify buttons provide visual feedback when pressed

### 13. Scrolling

**Both Devices:**
- [ ] Exchange at least 10 messages to create scrollable content
- [ ] Scroll through message cards on desk area
- [ ] Verify smooth scrolling
- [ ] Verify no content cut-off
- [ ] Verify oldest messages remain accessible

### 14. Terms Acceptance and Session Restoration

**Fresh Device C (if available):**
- [ ] Open AFTERTONE for first time
- [ ] Verify Terms dialog appears
- [ ] Cancel Terms dialog
- [ ] Verify cannot create or join invitation
- [ ] Accept Terms
- [ ] Create invitation
- [ ] Verify invitation is created successfully

### 15. Connection Expiry

**Devices A and B:**
- [ ] Create a 30-minute connection (or shortest available)
- [ ] Note the TIME LEFT counter
- [ ] Verify counter decrements
- [ ] Wait near expiry (or use short test duration if available)
- [ ] Observe expiry behavior
- [ ] Verify both devices return to OFFLINE state
- [ ] Verify connection cannot be used after expiry

### 16. Replay, Reordering, and Stale Messages (controlled harness required)

These checks need a test harness or development build that can inject relay frames; a normal UI session does not expose them.

- [ ] Capture an encrypted application envelope and resend it unchanged
- [ ] Verify no duplicate message is displayed
- [ ] Re-encrypt the same sequence number with different text and send it
- [ ] Verify no message is displayed and the client reports a replay rejection
- [ ] Send a newer sequence, then an older one
- [ ] Verify the older message is not displayed
- [ ] Send an envelope bound to a different exchange or recipient
- [ ] Verify it is rejected and no plaintext is displayed
- [ ] Refresh one device, then send an envelope from the previous session
- [ ] Verify the stale-session envelope cannot be decrypted or displayed
- [ ] Send a forged key-confirmation tag before verification
- [ ] Verify the session never becomes ready and VERIFY remains unavailable

## Results Recording

For each test section, record:
- ✅ Pass
- ❌ Fail (with description)
- ⚠️ Partial (with notes)
- ⏭️ Skipped (with reason)

### Device Information

**Device A:**
- Model: _________________
- OS: ____________________
- Browser: _______________
- Browser Version: ________

**Device B:**
- Model: _________________
- OS: ____________________
- Browser: _______________
- Browser Version: ________

### Network Configuration

- [ ] Same Wi-Fi network
- [ ] Different Wi-Fi networks
- [ ] Wi-Fi and cellular data
- [ ] Both on cellular data

### Summary

Total tests executed: _____ / 16
Tests passed: _____
Tests failed: _____
Critical issues: _____
Non-critical issues: _____

### Notes

Use this space to record any observations, anomalies, or issues not covered by the checklist:

---

**Testing Date:** ______________
**Tester Name:** ______________
**AFTERTONE Version/Deployment:** ______________
