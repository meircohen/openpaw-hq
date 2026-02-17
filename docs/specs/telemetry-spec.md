# OpenPaw — Privacy-Respecting Telemetry

## Principle
We need to know if OpenPaw works in the wild. We will NEVER compromise user privacy to get this data.

## Opt-In Only

Telemetry is OFF by default. Period.

On first launch (after onboarding), one prompt:
"Help us improve OpenPaw by sharing anonymous usage data? You can change this anytime in Settings."
[Yes, help improve OpenPaw] [No thanks]

If no, we collect nothing. Ever. No nagging. No dark patterns. No "remind me later."

## What We Collect (if opted in)

### Allowed ✅
- App version
- macOS version
- Apple Silicon vs Intel
- AI provider type (not the key, just "openai" / "anthropic" / "ollama")
- Feature usage counts (how many emails triaged, not content)
- Error types (crash reports, not context)
- Performance metrics (launch time, response latency)
- Session duration
- Number of connected services (not which ones)

### Never Collected ❌
- API keys
- Email content, subjects, or addresses
- Calendar events
- Chat messages
- File names or contents
- Screenshots
- OAuth tokens
- IP address (use privacy-respecting analytics)
- Device identifiers
- User name or email
- Anything that could identify the user

## Implementation

### Transport
- HTTPS POST to `telemetry.openpaw.dev` (if we set this up)
- OR: just GitHub issue templates for manual bug reports (simpler, no infra)
- Batched: send once per day, not real-time
- Failed sends silently dropped (never retry aggressively)

### Local First
- All telemetry data written to local file first
- User can inspect exactly what would be sent: Settings → Privacy → View Telemetry Data
- User can delete telemetry data anytime

### For MVP
**Skip telemetry entirely.** Use GitHub Issues for feedback. Add opt-in telemetry in v1.1+ if needed. Less to build, less privacy risk, less trust to earn upfront.

## Crash Reporting

Use Apple's built-in crash reporting (users opt in via macOS preferences). No third-party crash reporters (no Sentry, no Crashlytics, no BugSnag). Less dependency, more trust.

For beta: ask testers to share crash logs manually via GitHub Issues.
