# Pilot — Security & Privacy Model

## Core Principle

**Nothing leaves the device. Ever.**

This is not a policy decision — it's an architectural one. There is no server to send data to. There is no cloud endpoint. The app has no network capability by default.

## Data Classification

### Tier 1 — Ephemeral (never stored)
- Raw screenshots (captured, processed, discarded)
- AI app responses (read, parsed, discarded)
- Clipboard contents during copy/paste operations

### Tier 2 — Local Storage (encrypted at rest)
- User preferences and settings
- Task history (what Pilot did, when)
- Memory/learning data (user patterns, preferences)
- Workflow templates

### Tier 3 — Never Captured
- Passwords (Pilot never reads password fields)
- Financial credentials
- Private keys / seed phrases
- Content in apps the user hasn't authorized

## Permission Model

### macOS Permissions Required
1. **Screen Recording** — to capture screenshots (required, core functionality)
2. **Accessibility** — to send keyboard/mouse events (required, core functionality)
3. **Automation** — to interact with specific apps (per-app, requested on first use)

### Pilot's Internal Permission Model
Users authorize Pilot per-app:
- "Pilot can see and interact with: Gmail (Safari), Calendar, ChatGPT"
- "Pilot cannot touch: Banking apps, 1Password, Terminal"
- Default: nothing authorized. User adds apps explicitly.

## Action Approval Tiers

### 🟢 Green — Automatic (no confirmation)
- Read/scan content (emails, calendar, documents)
- Organize and categorize
- Draft content (saved locally, not sent)
- Search and lookup
- Navigate between apps/pages

### 🟡 Yellow — Confirm Before Execute
- Send email/message
- Schedule/modify calendar events
- Fill and submit forms
- Download files
- Change settings in any app

### 🔴 Red — Multi-Step Approval
- Any financial transaction (payments, transfers, purchases)
- Signing or agreeing to terms
- Deleting data
- Sharing content with others
- Actions in apps flagged as sensitive by user

## Encryption

- **Storage:** SQLite with SQLCipher (AES-256)
- **Encryption key:** Derived from macOS Keychain (tied to user's login)
- **No master key.** If the user's Mac login is compromised, Pilot's data is accessible — same threat model as the rest of their Mac.

## App Sandboxing

Pilot runs in a macOS sandbox with:
- No network access (hardcoded — no outbound connections)
- File system access limited to its own container + user-authorized directories
- No access to Contacts, Photos, Location, Microphone, Camera unless explicitly added later

## Audit Log

Every action Pilot takes is logged locally:
```json
{
  "timestamp": "2026-02-17T17:30:00Z",
  "action": "click",
  "target": "Gmail — Reply button",
  "app": "Safari",
  "approval_tier": "green",
  "screenshot_hash": "sha256:abc123...",
  "result": "success"
}
```

User can review full history anytime. "What did Pilot do today?" → shows complete log with screenshots.

## Threat Model

| Threat | Mitigation |
|--------|-----------|
| Malicious app impersonating ChatGPT | Verify app bundle ID before sending prompts |
| Screenshot contains sensitive data in background | Capture only the target app's window, not full screen |
| User's AI app logs/stores Pilot's prompts | User's responsibility — same as if they typed it. Document in onboarding. |
| Pilot executes wrong action (misclick) | Undo capability + approval tiers + verification screenshots |
| Someone gains access to user's Mac | Same threat model as all Mac apps — Pilot's data is encrypted at rest |
| App update introduces vulnerability | Code signing + notarization + update verification |

## Privacy Marketing Claims (verified by architecture)

✅ "No data leaves your device" — No network capability
✅ "No account required" — No server to authenticate against
✅ "No cloud processing" — AI runs in user's own app
✅ "Fully encrypted" — SQLCipher AES-256
✅ "Complete audit trail" — Every action logged
✅ "You control what Pilot can see" — Per-app authorization
