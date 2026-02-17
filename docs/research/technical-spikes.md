# OpenPaw — Technical Spikes (Pre-Development Prototypes)

## Purpose
Before building the full app, we need to prove that key technical risks actually work. Each spike is a minimal prototype that validates one assumption.

## Spike 1: Bundle Node.js in a Mac .app
**Question:** Can we ship a working Node.js + OpenClaw inside a macOS .app bundle?
**Approach:**
1. Download Node.js arm64 binary
2. Create minimal .app with embedded Node.js in `Resources/`
3. Swift app launches Node.js process on startup
4. Verify OpenClaw gateway starts and responds to WebSocket
**Success criteria:** .app launches, OpenClaw gateway responds, total bundle < 200MB
**Risk level:** Low — this is well-trodden (Electron does it, VS Code does it)
**Estimated time:** 2-3 hours

## Spike 2: CGVirtualDisplay
**Question:** Can we create a virtual display that the OS renders to, capture its framebuffer, and inject input events?
**Approach:**
1. Swift app creates CGVirtualDisplay
2. Open Safari on the virtual display
3. Capture screenshot of virtual display
4. Send click event to coordinates on virtual display
**Success criteria:** Screenshot shows Safari rendering, click navigates to URL
**Risk level:** Medium — CGVirtualDisplay is newer API, less documented
**Estimated time:** 4-6 hours

## Spike 3: AI App Bridge (ChatGPT Desktop)
**Question:** Can we programmatically type into ChatGPT.app and read its response?
**Approach:**
1. Use Accessibility API to find ChatGPT's text input field
2. Type a prompt programmatically
3. Wait for response to appear
4. Read the response text via Accessibility API
**Success criteria:** Send "What is 2+2?" → read back "4" (or equivalent)
**Risk level:** High — ChatGPT may not expose accessibility tree properly
**Alternative:** If Accessibility fails, fall back to screen capture + OCR
**Estimated time:** 4-8 hours

## Spike 4: Swift ↔ OpenClaw WebSocket
**Question:** Can Swift UI communicate reliably with OpenClaw gateway via WebSocket?
**Approach:**
1. Start OpenClaw gateway on localhost
2. Swift app connects via URLSessionWebSocketTask
3. Send a message, receive agent response
4. Handle connection drops and reconnection
**Success criteria:** Bidirectional messaging works, auto-reconnects on drop
**Risk level:** Low — standard WebSocket, well-documented
**Estimated time:** 2-3 hours

## Spike 5: OAuth Flow from Swift App
**Question:** Can we initiate OAuth (e.g., Google) from a native Swift app and capture the callback?
**Approach:**
1. Open system browser with Google OAuth URL
2. Register custom URL scheme (openpaw://) for callback
3. Capture the auth code from the redirect
4. Exchange for tokens
5. Store in macOS Keychain
**Success criteria:** Full OAuth flow completes, token stored in Keychain
**Risk level:** Low — standard pattern, ASWebAuthenticationSession handles most of it
**Estimated time:** 3-4 hours

## Spike 6: Prompt Engineering for Screen Actions
**Question:** Can we reliably get an AI to output structured actions from a screenshot?
**Approach:**
1. Take screenshots of common UIs (Gmail, Calendar)
2. Send to Claude/GPT-4o with action prompt
3. Parse response into structured action (click X, type Y)
4. Measure accuracy over 50 screenshots
**Success criteria:** >90% accuracy on action parsing from screenshots
**Risk level:** Medium — this is the core product bet
**Estimated time:** 4-6 hours

## Spike Priority Order
1. Spike 4 (WebSocket) — foundational, blocks everything
2. Spike 1 (Bundle Node.js) — foundational, blocks everything
3. Spike 5 (OAuth) — needed for any service connection
4. Spike 2 (Virtual Display) — core feature but can defer
5. Spike 3 (AI App Bridge) — core feature but has fallbacks
6. Spike 6 (Prompt Engineering) — can iterate on this throughout

## Spike Results
Results will be documented here as each spike completes.

### Spike 1 Results: TBD
### Spike 2 Results: TBD
### Spike 3 Results: TBD
### Spike 4 Results: TBD
### Spike 5 Results: TBD
### Spike 6 Results: TBD
