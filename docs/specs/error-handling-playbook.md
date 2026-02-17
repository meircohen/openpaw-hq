# OpenPaw Error Handling Playbook

**Version:** 1.0  
**Date:** February 17, 2026  
**Author:** Engineering Team  

## Overview

This document defines how OpenPaw detects, handles, and recovers from all error conditions. Every error scenario must have a clear detection method, user-friendly message, recovery action, and appropriate logging.

## Error Handling Principles

1. **User-First Communication:** All error messages in plain English, no technical jargon
2. **Graceful Degradation:** Continue functioning with reduced capabilities when possible
3. **Automatic Recovery:** Attempt self-healing before prompting user intervention
4. **Comprehensive Logging:** Log everything for debugging, but sanitize sensitive data
5. **Progressive Disclosure:** Show simple message first, detailed info on request

---

## 1. OpenClaw Process Crashes

### Detection Method
- **Heartbeat Monitoring:** OpenPaw pings OpenClaw engine every 10 seconds
- **Process Monitoring:** Check if OpenClaw process exists in system process list
- **Response Timeout:** No response to API calls within 30 seconds
- **Connection Loss:** WebSocket/IPC connection drops unexpectedly

### User-Facing Messages

**Initial Detection:**
```
🔧 OpenClaw engine has stopped responding. Attempting to restart...
```

**Restart Successful:**
```
✅ OpenClaw engine restarted successfully. You can continue chatting.
```

**Restart Failed:**
```
❌ Unable to restart OpenClaw engine automatically.

What you can try:
• Restart OpenPaw completely
• Check if another OpenClaw instance is running
• Restart your Mac if issues persist

Need help? Click here for troubleshooting guide.
```

**Multiple Crashes (>3 in 10 minutes):**
```
⚠️ OpenClaw engine keeps crashing. This might indicate a system issue.

OpenPaw will now run in Safe Mode with basic functionality only.

• Chat will work with slower responses
• Some advanced features are disabled
• Email and calendar sync paused

Click "Full Diagnostics" to create a support report.
```

### Recovery Actions
1. **Immediate:** Wait 5 seconds, attempt automatic restart
2. **First Retry:** Kill any orphaned processes, restart with clean state
3. **Second Retry:** Clear cache/temp files, restart with reset config
4. **Third Retry:** Switch to Safe Mode (direct AI API calls, no engine)
5. **Final:** Show manual recovery options

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "ERROR",
  "event": "openclaw_process_crash",
  "details": {
    "pid": 12345,
    "uptime_seconds": 3600,
    "last_response": "2026-02-17T22:47:45Z",
    "exit_code": -9,
    "memory_usage_mb": 150,
    "restart_attempt": 1,
    "crash_count_10min": 1
  },
  "user_id": "hashed_user_id",
  "session_id": "session_uuid"
}
```

---

## 2. AI API Key Invalid/Expired

### Detection Method
- **API Response Codes:** 401 Unauthorized, 403 Forbidden from AI providers
- **Response Message Analysis:** Keywords like "invalid key", "expired", "unauthorized"
- **Startup Validation:** Test API key validity on app launch
- **Background Validation:** Periodic key validation every 24 hours

### User-Facing Messages

**Invalid Key Detected:**
```
🔑 Your AI provider API key appears to be invalid.

This usually happens when:
• The key was typed incorrectly
• The key has been regenerated
• Your account has been suspended

Would you like to update your API key now?
[Update Key] [Use Different Provider] [Get Help]
```

**Key Expired:**
```
🕐 Your AI provider API key has expired.

• Go to your provider's dashboard to generate a new key
• Copy and paste it in Settings > AI Providers
• Your chat history will be preserved

[Open Settings] [Provider Dashboard] [Help]
```

**No Valid Keys:**
```
⚠️ No valid AI provider keys found.

OpenPaw needs at least one AI provider to function:
• OpenAI (GPT-4, GPT-3.5)
• Anthropic (Claude)
• Local models (if configured)

[Add API Key] [Setup Guide] [Contact Support]
```

### Recovery Actions
1. **Automatic:** Try backup providers if configured
2. **Guided Setup:** Open settings to API key configuration
3. **Provider Switcher:** Quick toggle to working provider
4. **Offline Mode:** Basic functionality without AI (if possible)

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "WARN",
  "event": "api_key_invalid",
  "details": {
    "provider": "openai",
    "error_code": 401,
    "error_message": "Invalid API key",
    "key_prefix": "sk-1234...",
    "last_successful_call": "2026-02-17T20:15:30Z",
    "fallback_attempted": true,
    "fallback_provider": "anthropic"
  }
}
```

---

## 3. AI API Rate Limited

### Detection Method
- **HTTP Status Code:** 429 Too Many Requests
- **Rate Limit Headers:** X-RateLimit-Remaining, Retry-After
- **Provider-Specific:** OpenAI, Anthropic rate limit response formats
- **Pattern Recognition:** Consecutive failed requests with timing correlation

### User-Facing Messages

**Temporary Rate Limit:**
```
⏳ AI provider is temporarily busy. Your message will be sent in 30 seconds...

This is normal during peak usage times.
```

**Extended Rate Limit:**
```
🚦 Your AI provider account has hit its rate limit.

This usually means:
• You've used your monthly quota
• Many requests in a short time
• Your plan has specific limits

Estimated wait time: 15 minutes
[Switch Provider] [Upgrade Plan] [Wait]
```

**Account Suspended:**
```
🚫 Your AI provider account appears to be suspended or over quota.

Please check your account status:
• Log into your provider dashboard
• Review billing and usage
• Contact provider support if needed

[Provider Dashboard] [Try Different Provider] [Help]
```

### Recovery Actions
1. **Queue Messages:** Hold messages during rate limit period
2. **Smart Retry:** Exponential backoff with jitter
3. **Provider Switching:** Automatic fallback to secondary provider
4. **Batch Processing:** Combine multiple requests when possible
5. **Usage Optimization:** Compress prompts, cache responses

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "WARN",
  "event": "api_rate_limited",
  "details": {
    "provider": "openai",
    "error_code": 429,
    "retry_after_seconds": 1800,
    "requests_this_hour": 100,
    "rate_limit": 60,
    "queue_length": 3,
    "fallback_available": true
  }
}
```

---

## 4. OAuth Token Expired

### Detection Method
- **API Response:** 401 with OAuth-specific error messages
- **Token Inspection:** Check token expiration timestamps
- **Refresh Token Flow:** Failed refresh token attempts
- **Service Responses:** Gmail/Calendar specific auth errors

### User-Facing Messages

**Token Expired - Auto Refresh Successful:**
```
🔄 Refreshing your Google account connection...
✅ Connection refreshed successfully.
```

**Token Expired - Manual Reauth Required:**
```
🔐 Your Google account connection has expired.

This happens for security after 90 days of inactivity.

Your data is safe, but you'll need to reconnect:
• Click "Reconnect Google Account"
• Sign in again (should remember your account)
• Your previous settings will be restored

[Reconnect Account] [Learn More] [Skip for Now]
```

**Multiple Account Issues:**
```
⚠️ Multiple account connections need attention:
• Gmail: Expired (reconnect needed)
• Calendar: Working fine
• Drive: Authentication failed

[Fix All Accounts] [Fix One by One] [Skip]
```

### Recovery Actions
1. **Automatic Refresh:** Use refresh token silently
2. **Guided Reauth:** Open OAuth flow with pre-selected account
3. **Scope Verification:** Request only necessary permissions
4. **Graceful Degradation:** Disable affected features, keep others working
5. **Batch Reconnection:** Handle multiple expired tokens together

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "INFO",
  "event": "oauth_token_expired",
  "details": {
    "service": "gmail",
    "token_age_days": 91,
    "refresh_attempted": true,
    "refresh_successful": false,
    "scopes": ["gmail.readonly", "calendar.events"],
    "user_reauth_required": true
  }
}
```

---

## 5. Network Offline (for API-based AI)

### Detection Method
- **Network Reachability:** macOS Network Framework monitoring
- **HTTP Timeouts:** Failed connections to AI providers
- **DNS Resolution:** Check if provider domains resolve
- **Ping Tests:** Periodic connectivity checks to major services

### User-Facing Messages

**Network Disconnected:**
```
📶 No internet connection detected.

OpenPaw can still:
• Show your chat history
• Work with local features
• Queue messages for when you're back online

[Retry Connection] [View Offline Features] [Queue Message]
```

**AI Services Unreachable:**
```
🌐 Can't reach AI providers right now.

Your internet works, but AI services might be:
• Temporarily down
• Blocked by your network
• Having connectivity issues

Your message will be sent when connection is restored.
[Retry Now] [Check Status Page] [Switch Provider]
```

**Partial Connectivity:**
```
⚠️ Limited internet connectivity detected.

• Email and calendar sync paused
• AI responses may be slower
• Some features temporarily unavailable

OpenPaw will resume full functionality when connection improves.
```

### Recovery Actions
1. **Message Queuing:** Store messages locally, send when online
2. **Offline Mode:** Enable available local features
3. **Smart Retry:** Progressive retry intervals (5s, 30s, 2m, 5m)
4. **Service Status:** Show which services are reachable
5. **Local Processing:** Use cached responses or local models if available

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "WARN",
  "event": "network_offline",
  "details": {
    "connectivity_state": "offline",
    "last_successful_request": "2026-02-17T22:45:12Z",
    "reachable_services": [],
    "unreachable_services": ["openai.com", "api.anthropic.com"],
    "queued_messages": 2,
    "retry_attempt": 3
  }
}
```

---

## 6. Disk Full

### Detection Method
- **File System Monitoring:** Check available space before writes
- **Write Operation Failures:** ENOSPC errors during file operations
- **Log Rotation Issues:** Failed log writes or rotations
- **Cache Management:** Unable to write cache files
- **Threshold Monitoring:** Alert when disk usage >95%

### User-Facing Messages

**Disk Space Warning (>90% full):**
```
💾 Your disk is almost full (2.1 GB remaining).

This might cause OpenPaw to:
• Stop saving chat history
• Unable to download attachments
• Slower performance

Consider freeing up some space for best performance.
[Check Disk Usage] [Clean OpenPaw Cache] [Remind Later]
```

**Disk Full (>98% full):**
```
🚨 Your disk is critically full (500 MB remaining).

OpenPaw has switched to minimal mode:
• Chat history limited to memory only
• File operations disabled
• Cache cleared automatically

Please free up disk space immediately.
[Open Storage Settings] [Clear All Cache] [Emergency Mode]
```

**Write Operations Failed:**
```
💿 Unable to save data due to insufficient disk space.

Your recent:
• Chat messages (temporarily in memory)
• Settings changes (not saved)
• Downloaded files (failed)

Free up space and restart OpenPaw to recover.
[View Unsaved Changes] [Emergency Export] [Get Help]
```

### Recovery Actions
1. **Cache Cleanup:** Automatically clear temporary files and caches
2. **Log Rotation:** Aggressive log cleanup and compression
3. **Emergency Mode:** Minimal functionality, memory-only operations
4. **Data Export:** Offer to export important data before shutdown
5. **Storage Analysis:** Show what's using space (if possible)

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "ERROR",
  "event": "disk_full",
  "details": {
    "available_bytes": 524288000,
    "total_bytes": 1000000000000,
    "usage_percent": 99.9,
    "failed_operation": "chat_history_write",
    "cache_cleared_bytes": 150000000,
    "emergency_mode_enabled": true
  }
}
```

---

## 7. macOS Permission Revoked

### Detection Method
- **Permission Status APIs:** TCC (Transparency, Consent, and Control) database
- **Operation Failures:** Screen recording, accessibility API failures
- **Capability Testing:** Periodic checks of permission-dependent features
- **System Notifications:** macOS permission change notifications

### User-Facing Messages

**Accessibility Permission Revoked:**
```
🛡️ Accessibility permission has been turned off.

OpenPaw needs this permission to:
• Read and interact with other apps
• Automate tasks you approve
• Provide full AI assistance

[Open System Preferences] [Learn More] [Skip Features]
```

**Screen Recording Permission Revoked:**
```
📺 Screen recording permission has been disabled.

This limits OpenPaw's ability to:
• See what's on your screen for context
• Help with visual tasks
• Capture screenshots when requested

You can still use chat and basic features.
[Restore Permission] [Continue Limited] [Learn More]
```

**Multiple Permissions Revoked:**
```
⚠️ Several permissions have been disabled:
• ❌ Accessibility (required for automation)
• ❌ Screen Recording (for visual assistance)
• ✅ Microphone (working fine)

OpenPaw will work with reduced functionality.
[Fix All Permissions] [Review Each] [Continue Anyway]
```

### Recovery Actions
1. **Permission Check:** Verify current permission status
2. **Guided Restoration:** Open System Preferences to correct section
3. **Feature Graceful Degradation:** Disable affected features cleanly
4. **Alternative Workflows:** Offer manual alternatives where possible
5. **Permission Education:** Explain why each permission is needed

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "WARN",
  "event": "permission_revoked",
  "details": {
    "permission_type": "accessibility",
    "previous_status": "authorized",
    "current_status": "denied",
    "affected_features": ["automation", "ui_interaction"],
    "detection_method": "capability_test_failure",
    "user_guided_to_settings": true
  }
}
```

---

## 8. Corrupted Config/Database

### Detection Method
- **File Integrity:** Checksum verification of config files
- **JSON Parsing:** Invalid JSON structure in config files
- **Database Corruption:** SQLite database integrity checks
- **Migration Failures:** Version upgrade/downgrade issues
- **Startup Validation:** Configuration validation on app launch

### User-Facing Messages

**Minor Config Corruption:**
```
⚙️ Some settings were corrupted and have been reset to defaults.

Affected settings:
• Window position and size
• Theme preferences
• Keyboard shortcuts

Your AI providers and chat history are safe.
[Review Settings] [Import Backup] [Continue]
```

**Major Config Corruption:**
```
🔧 OpenPaw's configuration is corrupted and needs to be reset.

What will be reset:
• All app settings (not chat history)
• AI provider connections (will need to reconnect)
• Custom preferences

What's preserved:
• Your chat conversations
• Downloaded files
• Integration data

[Reset Config] [Try Recovery] [Restore Backup]
```

**Database Corruption:**
```
💾 Chat history database is corrupted.

Options to recover your conversations:
• Automatic repair (may lose recent messages)
• Restore from backup (loses data since last backup)
• Export readable text (for manual recovery)

[Try Repair] [Restore Backup] [Export Text] [Start Fresh]
```

### Recovery Actions
1. **Automatic Repair:** Attempt to fix minor corruption
2. **Backup Restoration:** Restore from most recent good backup
3. **Partial Recovery:** Salvage what data is recoverable
4. **Clean Slate:** Reset with option to import critical data
5. **Data Export:** Export critical data in readable format

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "ERROR",
  "event": "config_corruption",
  "details": {
    "corruption_type": "json_parse_error",
    "affected_files": ["config.json", "user_preferences.json"],
    "backup_available": true,
    "backup_age_hours": 6,
    "repair_attempted": true,
    "repair_successful": false,
    "data_loss_risk": "settings_only"
  }
}
```

---

## 9. User Interrupts Task Mid-Execution

### Detection Method
- **User Interface:** Cancel button, Escape key, window close
- **System Signals:** SIGTERM, SIGINT handling
- **UI State Monitoring:** User navigates away during long operations
- **Timeout Detection:** User doesn't respond to prompts

### User-Facing Messages

**Task Interrupted:**
```
⏹️ Task stopped at your request.

Progress so far:
• Email scan: 45% complete (234 emails processed)
• Calendar sync: Not started
• Action items: Draft saved

[Resume Task] [View Progress] [Start Over] [Cancel]
```

**Partial Completion:**
```
⚡ Task partially completed before interruption:

✅ Completed:
• Connected to Gmail
• Read 15 urgent emails

⏳ Interrupted:
• Calendar event creation (draft saved)
• Email responses (3 pending drafts)

[Finish Remaining] [Review Drafts] [Start Fresh]
```

**Critical Operation Interrupted:**
```
⚠️ You stopped a critical operation in progress.

• File upload: 80% complete (may be corrupted)
• Database update: Partially applied
• Email sending: 2 of 5 messages sent

[Complete Safely] [Rollback Changes] [Check Status]
```

### Recovery Actions
1. **Graceful Shutdown:** Stop at safe checkpoint
2. **State Preservation:** Save progress for later resumption
3. **Rollback Capability:** Undo partial changes if needed
4. **Progress Reporting:** Show what was accomplished
5. **Resume Options:** Allow continuation from interruption point

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "INFO",
  "event": "user_task_interruption",
  "details": {
    "task_type": "email_processing",
    "completion_percent": 45,
    "interruption_method": "cancel_button",
    "safe_checkpoint_reached": true,
    "data_at_risk": false,
    "resume_possible": true,
    "cleanup_required": false
  }
}
```

---

## 10. AI Returns Garbage/Unexpected Response

### Detection Method
- **Response Validation:** Check for expected JSON structure, response format
- **Content Analysis:** Detect nonsensical, repetitive, or malformed responses
- **Length Anomalies:** Responses that are too short or excessively long
- **Encoding Issues:** Character encoding problems, corrupted text
- **Context Mismatch:** Response doesn't relate to the prompt

### User-Facing Messages

**Invalid AI Response:**
```
🤖 The AI gave an unexpected response. Let me try that again...

Original request: "Check my calendar for tomorrow"
AI response: [garbled text/JSON error/empty response]

[Retry with Same Provider] [Try Different Provider] [Report Issue]
```

**Partial Response:**
```
⚠️ The AI response was cut off or incomplete.

I received: "Your calendar for tomorrow has..."
Expected: Full calendar information

This sometimes happens with complex requests.
[Try Again] [Simplify Request] [Use Different Provider]
```

**Nonsensical Response:**
```
🎭 The AI response doesn't make sense for your request.

Your request: "What's the weather like?"
AI response: "The purple elephant dances with mathematical..."

This might be a temporary AI provider issue.
[Retry] [Report Response] [Switch Provider] [Manual Override]
```

### Recovery Actions
1. **Automatic Retry:** Retry same request with modified prompt
2. **Provider Switching:** Try request with different AI provider
3. **Response Filtering:** Detect and discard clearly invalid responses
4. **Fallback Responses:** Provide safe default responses when possible
5. **Error Context:** Preserve original request for debugging

### Logging
```json
{
  "timestamp": "2026-02-17T22:48:00Z",
  "level": "ERROR",
  "event": "ai_response_invalid",
  "details": {
    "provider": "openai",
    "model": "gpt-4",
    "request_token_count": 150,
    "response_token_count": 25,
    "validation_errors": ["json_parse_failed", "context_mismatch"],
    "response_preview": "The purple elephant...",
    "retry_attempted": true,
    "fallback_provider": "anthropic"
  }
}
```

---

## Error Logging Standards

### Log Levels
- **DEBUG:** Detailed debugging information
- **INFO:** General operational information
- **WARN:** Warning conditions that should be noted
- **ERROR:** Error conditions that affect functionality
- **FATAL:** Critical errors that may cause app termination

### Required Fields
All error logs must include:
```json
{
  "timestamp": "ISO 8601 format",
  "level": "ERROR|WARN|INFO|DEBUG|FATAL",
  "event": "snake_case_event_name",
  "details": {
    "specific_error_details": "value"
  },
  "user_id": "hashed_user_identifier",
  "session_id": "unique_session_identifier",
  "app_version": "1.0.0",
  "os_version": "macOS 14.3",
  "build": "2026.02.17.1"
}
```

### Sensitive Data Handling
- **Never log:** API keys, passwords, tokens, personal data
- **Hash:** User identifiers, email addresses
- **Truncate:** Long responses, file contents
- **Redact:** Replace sensitive patterns with [REDACTED]

### Log Rotation
- **Size limit:** 50MB per log file
- **Retention:** 30 days of logs
- **Compression:** Gzip older logs
- **Cleanup:** Automatic cleanup of old logs

### Error Reporting
- **Crash Reports:** Automatic crash report generation
- **User Consent:** Always ask before sending reports
- **Privacy:** Strip all personal data from reports
- **Actionable:** Include enough context for debugging

---

## User Experience Guidelines

### Error Message Tone
- **Friendly:** Avoid technical jargon, speak naturally
- **Helpful:** Always offer next steps or solutions
- **Honest:** Don't hide problems, explain what happened
- **Reassuring:** Indicate what data is safe when relevant

### Visual Design
- **Icons:** Use consistent icons for error types
- **Colors:** Red for critical, yellow for warnings, blue for info
- **Layout:** Clear hierarchy, most important info first
- **Actions:** Primary action should be most helpful choice

### Accessibility
- **Screen Readers:** Proper ARIA labels and roles
- **High Contrast:** Error messages visible in all contrast modes
- **Keyboard Navigation:** All error dialogs fully keyboard accessible
- **Voice Control:** Error buttons and links properly labeled

### Testing Error Conditions
- **Simulate Failures:** Regular testing of all error scenarios
- **User Testing:** Observe real users encountering errors
- **Message Clarity:** Verify error messages are understandable
- **Recovery Success:** Measure how often users successfully recover

This error handling playbook ensures OpenPaw fails gracefully and helps users recover quickly from any problem they encounter.