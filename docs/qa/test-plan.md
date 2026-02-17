# OpenPaw MVP Test Plan

**Version:** 1.0  
**Date:** February 17, 2026  
**Author:** QA Engineering Team  

## Test Case Format
- **ID:** Unique test identifier
- **Description:** What the test validates
- **Preconditions:** Setup requirements
- **Steps:** Execution steps
- **Expected Result:** Success criteria
- **Priority:** P0 (Blocker), P1 (Critical), P2 (Important), P3 (Nice-to-have)

---

## 1. Onboarding Flow

### TC-ONB-001
**Description:** First launch onboarding - happy path  
**Preconditions:** Fresh install, no previous config  
**Steps:**
1. Launch OpenPaw for first time
2. Complete welcome screen
3. Grant accessibility permissions when prompted
4. Grant screen recording permissions when prompted
5. Set AI provider preference
6. Complete setup wizard
**Expected Result:** User lands on main chat interface, all permissions granted, basic config saved  
**Priority:** P0

### TC-ONB-002
**Description:** Onboarding with accessibility permission denied  
**Preconditions:** Fresh install  
**Steps:**
1. Launch OpenPaw
2. Click "Deny" on accessibility permission dialog
3. Attempt to continue onboarding
**Expected Result:** Clear error message explaining accessibility is required, option to retry or open System Preferences  
**Priority:** P0

### TC-ONB-003
**Description:** Onboarding with screen recording permission denied  
**Preconditions:** Fresh install  
**Steps:**
1. Launch OpenPaw
2. Grant accessibility permission
3. Click "Deny" on screen recording permission
4. Attempt to continue onboarding
**Expected Result:** Warning message about limited functionality, option to continue with reduced features or grant permission  
**Priority:** P1

### TC-ONB-004
**Description:** Onboarding interrupted and resumed  
**Preconditions:** Fresh install  
**Steps:**
1. Launch OpenPaw
2. Start onboarding process
3. Force quit app mid-onboarding
4. Relaunch app
**Expected Result:** Onboarding resumes from last completed step, no corruption  
**Priority:** P2

### TC-ONB-005
**Description:** Onboarding with no AI provider keys  
**Preconditions:** Fresh install, no API keys available  
**Steps:**
1. Complete onboarding flow
2. Skip AI provider setup
3. Attempt to send first message
**Expected Result:** Helpful message explaining AI provider setup is required, link to settings  
**Priority:** P1

---

## 2. Gmail OAuth Connection and Email Triage

### TC-GML-001
**Description:** Gmail OAuth connection - happy path  
**Preconditions:** Valid Google account  
**Steps:**
1. Open Settings > Integrations
2. Click "Connect Gmail"
3. Complete OAuth flow in browser
4. Return to app
**Expected Result:** Gmail connected, account info displayed, basic email access confirmed  
**Priority:** P0

### TC-GML-002
**Description:** Gmail OAuth connection denied  
**Preconditions:** Valid Google account  
**Steps:**
1. Open Settings > Integrations
2. Click "Connect Gmail"
3. Click "Cancel" or "Deny" in OAuth flow
**Expected Result:** User returned to settings with clear message about connection failure  
**Priority:** P1

### TC-GML-003
**Description:** Gmail token expiration handling  
**Preconditions:** Gmail connected with expired token  
**Steps:**
1. Attempt email operation with expired token
2. Observe app behavior
**Expected Result:** Automatic token refresh attempt, or prompt to re-authenticate if refresh fails  
**Priority:** P0

### TC-GML-004
**Description:** Email triage - basic categorization  
**Preconditions:** Gmail connected, test emails available  
**Steps:**
1. Trigger email triage process
2. Review categorization results
3. Verify high-priority items are flagged
**Expected Result:** Emails categorized correctly (urgent, important, promotional, etc.)  
**Priority:** P1

### TC-GML-005
**Description:** Email triage with network interruption  
**Preconditions:** Gmail connected  
**Steps:**
1. Start email triage process
2. Disconnect network mid-process
3. Reconnect network
**Expected Result:** Process pauses gracefully, resumes when network available, no data corruption  
**Priority:** P2

---

## 3. Calendar Integration

### TC-CAL-001
**Description:** Calendar OAuth connection  
**Preconditions:** Valid Google account with calendar access  
**Steps:**
1. Open Settings > Integrations
2. Click "Connect Calendar"
3. Complete OAuth flow
4. Verify calendar access
**Expected Result:** Calendar connected, upcoming events visible in app  
**Priority:** P0

### TC-CAL-002
**Description:** Calendar event creation via chat  
**Preconditions:** Calendar connected  
**Steps:**
1. In chat, type "Schedule meeting with John tomorrow at 2pm"
2. Confirm event details
3. Approve event creation
**Expected Result:** Event created in calendar, confirmation shown in chat  
**Priority:** P1

### TC-CAL-003
**Description:** Calendar conflict detection  
**Preconditions:** Calendar connected with existing events  
**Steps:**
1. Attempt to schedule event that conflicts with existing one
2. Review conflict notification
**Expected Result:** Clear conflict warning with options to reschedule or override  
**Priority:** P1

### TC-CAL-004
**Description:** Multiple calendar support  
**Preconditions:** Google account with multiple calendars  
**Steps:**
1. Connect calendar integration
2. Verify all calendars are detected
3. Create event specifying target calendar
**Expected Result:** All calendars available, events created in correct calendar  
**Priority:** P2

---

## 4. Chat Interface

### TC-CHT-001
**Description:** Send basic message and receive response  
**Preconditions:** AI provider configured  
**Steps:**
1. Type message in chat input
2. Press Enter or click Send
3. Wait for response
**Expected Result:** Message sent successfully, AI response received and displayed  
**Priority:** P0

### TC-CHT-002
**Description:** Chat history persistence  
**Preconditions:** Previous chat session exists  
**Steps:**
1. Close and reopen app
2. Check chat history
**Expected Result:** Previous messages visible and accessible  
**Priority:** P1

### TC-CHT-003
**Description:** Long message handling  
**Preconditions:** AI provider configured  
**Steps:**
1. Send message exceeding typical length (>2000 characters)
2. Verify handling
**Expected Result:** Message handled gracefully, appropriate chunking or truncation if needed  
**Priority:** P2

### TC-CHT-004
**Description:** Special characters and emoji in messages  
**Preconditions:** AI provider configured  
**Steps:**
1. Send message with emojis, special characters, and unicode
2. Verify display and AI processing
**Expected Result:** Characters display correctly, AI processes without errors  
**Priority:** P2

### TC-CHT-005
**Description:** Chat history search  
**Preconditions:** Chat history with multiple conversations  
**Steps:**
1. Use search function in chat
2. Search for specific term or phrase
3. Review results
**Expected Result:** Relevant messages found and highlighted  
**Priority:** P3

---

## 5. Approval Flow (Green/Yellow/Red Actions)

### TC-APV-001
**Description:** Green action (safe) - auto-approve  
**Preconditions:** AI configured, approval settings on default  
**Steps:**
1. Request action classified as "green" (e.g., "What's the weather?")
2. Observe execution
**Expected Result:** Action executes automatically without user intervention  
**Priority:** P0

### TC-APV-002
**Description:** Yellow action (moderate risk) - prompt for approval  
**Preconditions:** AI configured  
**Steps:**
1. Request action classified as "yellow" (e.g., "Send email to colleague")
2. Review approval prompt
3. Click "Approve"
**Expected Result:** Clear approval dialog shown, action executes after approval  
**Priority:** P0

### TC-APV-003
**Description:** Red action (high risk) - explicit confirmation  
**Preconditions:** AI configured  
**Steps:**
1. Request action classified as "red" (e.g., "Delete all emails")
2. Review warning dialog
3. Test both approve and deny paths
**Expected Result:** Strong warning shown, action requires explicit confirmation, deny works correctly  
**Priority:** P0

### TC-APV-004
**Description:** Approval timeout handling  
**Preconditions:** Yellow/Red action pending approval  
**Steps:**
1. Request moderate/high risk action
2. Don't respond to approval prompt for 5 minutes
**Expected Result:** Action times out gracefully, user notified, no execution  
**Priority:** P1

### TC-APV-005
**Description:** Custom approval settings  
**Preconditions:** Access to settings  
**Steps:**
1. Modify approval thresholds in settings
2. Test actions with new settings
**Expected Result:** Approval behavior changes according to new settings  
**Priority:** P2

---

## 6. Settings Changes

### TC-SET-001
**Description:** Change AI provider  
**Preconditions:** Multiple AI providers configured  
**Steps:**
1. Open Settings > AI Providers
2. Select different provider
3. Apply changes
4. Test chat functionality
**Expected Result:** Provider switches successfully, chat works with new provider  
**Priority:** P1

### TC-SET-002
**Description:** Update API key  
**Preconditions:** AI provider configured  
**Steps:**
1. Open Settings > AI Providers
2. Update API key for current provider
3. Save changes
4. Test functionality
**Expected Result:** New key accepted, functionality maintained  
**Priority:** P1

### TC-SET-003
**Description:** Invalid API key handling  
**Preconditions:** AI provider configured  
**Steps:**
1. Enter invalid API key in settings
2. Attempt to save
3. Test chat functionality
**Expected Result:** Validation error shown, invalid key rejected or warning displayed  
**Priority:** P1

### TC-SET-004
**Description:** Settings persistence  
**Preconditions:** Settings modified  
**Steps:**
1. Change multiple settings
2. Restart application
3. Verify settings
**Expected Result:** All settings persist correctly across restarts  
**Priority:** P1

### TC-SET-005
**Description:** Reset to defaults  
**Preconditions:** Settings modified from defaults  
**Steps:**
1. Use "Reset to Defaults" option
2. Verify all settings reset
**Expected Result:** All settings return to default values  
**Priority:** P2

---

## 7. App Lifecycle

### TC-LIF-001
**Description:** Clean app launch  
**Preconditions:** App not running  
**Steps:**
1. Launch OpenPaw from Applications
2. Observe startup sequence
**Expected Result:** App launches successfully, main interface appears within 5 seconds  
**Priority:** P0

### TC-LIF-002
**Description:** Background operation  
**Preconditions:** App running  
**Steps:**
1. Minimize or hide app
2. Wait 10 minutes
3. Restore app
**Expected Result:** App remains responsive, background tasks continue, UI restores correctly  
**Priority:** P1

### TC-LIF-003
**Description:** Graceful quit  
**Preconditions:** App running with active tasks  
**Steps:**
1. Use Cmd+Q or Quit from menu
2. Observe shutdown process
**Expected Result:** App saves state, completes critical tasks, quits cleanly  
**Priority:** P1

### TC-LIF-004
**Description:** Force quit recovery  
**Preconditions:** App running  
**Steps:**
1. Force quit app (Activity Monitor or kill command)
2. Relaunch app
3. Check state recovery
**Expected Result:** App detects improper shutdown, recovers state, shows recovery message if needed  
**Priority:** P1

### TC-LIF-005
**Description:** System sleep/wake handling  
**Preconditions:** App running  
**Steps:**
1. Put system to sleep for 1 hour
2. Wake system
3. Test app functionality
**Expected Result:** App handles sleep/wake gracefully, reconnects to services as needed  
**Priority:** P2

---

## 8. OpenClaw Engine Health

### TC-ENG-001
**Description:** OpenClaw engine start  
**Preconditions:** Engine not running  
**Steps:**
1. Start OpenClaw engine from app
2. Monitor startup process
3. Verify connection
**Expected Result:** Engine starts successfully, connection established, status indicator shows green  
**Priority:** P0

### TC-ENG-002
**Description:** OpenClaw engine stop  
**Preconditions:** Engine running  
**Steps:**
1. Stop engine from app interface
2. Verify shutdown
**Expected Result:** Engine stops cleanly, resources released, status updated  
**Priority:** P1

### TC-ENG-003
**Description:** OpenClaw engine restart  
**Preconditions:** Engine running  
**Steps:**
1. Restart engine from app
2. Monitor process
**Expected Result:** Engine stops and restarts successfully, minimal downtime  
**Priority:** P1

### TC-ENG-004
**Description:** Engine crash detection and recovery  
**Preconditions:** Engine running  
**Steps:**
1. Simulate engine crash (kill process)
2. Observe app behavior
**Expected Result:** Crash detected, automatic restart attempted, user notified of temporary disruption  
**Priority:** P0

### TC-ENG-005
**Description:** Engine health monitoring  
**Preconditions:** Engine running normally  
**Steps:**
1. Monitor engine status indicators
2. Check health metrics
**Expected Result:** Status accurately reflects engine state, health metrics are meaningful  
**Priority:** P2

---

## 9. Memory Persistence Across Sessions

### TC-MEM-001
**Description:** Chat history persistence  
**Preconditions:** Active chat session  
**Steps:**
1. Have extended conversation
2. Quit and restart app
3. Check chat history
**Expected Result:** All messages preserved and accessible  
**Priority:** P1

### TC-MEM-002
**Description:** Settings persistence  
**Preconditions:** Custom settings configured  
**Steps:**
1. Modify various settings
2. Restart app multiple times
3. Verify settings
**Expected Result:** Settings remain unchanged across restarts  
**Priority:** P1

### TC-MEM-003
**Description:** Integration state persistence  
**Preconditions:** Gmail and Calendar connected  
**Steps:**
1. Connect integrations
2. Restart app
3. Test integration functionality
**Expected Result:** Integrations remain connected and functional  
**Priority:** P1

### TC-MEM-004
**Description:** Memory corruption handling  
**Preconditions:** Corrupted data files simulated  
**Steps:**
1. Corrupt config or data files
2. Launch app
3. Observe recovery behavior
**Expected Result:** App detects corruption, attempts recovery, or prompts for reset  
**Priority:** P2

### TC-MEM-005
**Description:** Large memory footprint handling  
**Preconditions:** Extended use with large chat history  
**Steps:**
1. Generate large amount of chat data
2. Monitor memory usage
3. Test performance
**Expected Result:** Memory usage remains reasonable, performance acceptable  
**Priority:** P3

---

## 10. Multiple AI Provider Switching

### TC-AIP-001
**Description:** Switch between configured providers  
**Preconditions:** Multiple AI providers configured (OpenAI, Anthropic, etc.)  
**Steps:**
1. Send message with Provider A
2. Switch to Provider B in settings
3. Send message with Provider B
4. Compare responses and functionality
**Expected Result:** Switching works seamlessly, both providers function correctly  
**Priority:** P1

### TC-AIP-002
**Description:** Provider fallback on failure  
**Preconditions:** Multiple providers configured, primary provider down  
**Steps:**
1. Simulate primary provider failure
2. Send message
3. Observe fallback behavior
**Expected Result:** App automatically tries secondary provider, user notified of switch  
**Priority:** P2

### TC-AIP-003
**Description:** Provider-specific features  
**Preconditions:** Providers with different capabilities  
**Steps:**
1. Test features specific to each provider
2. Verify appropriate handling
**Expected Result:** Provider-specific features work correctly, unavailable features handled gracefully  
**Priority:** P2

### TC-AIP-004
**Description:** Concurrent provider usage  
**Preconditions:** Multiple providers configured  
**Steps:**
1. If supported, test using different providers simultaneously
2. Verify isolation and functionality
**Expected Result:** Providers operate independently without interference  
**Priority:** P3

---

## 11. Keyboard Shortcuts

### TC-KEY-001
**Description:** Basic keyboard shortcuts  
**Preconditions:** App active and in focus  
**Steps:**
1. Test Cmd+N (new conversation)
2. Test Cmd+R (restart engine)
3. Test Cmd+, (preferences)
4. Test other documented shortcuts
**Expected Result:** All shortcuts work as documented  
**Priority:** P2

### TC-KEY-002
**Description:** Global hotkey for activation  
**Preconditions:** App running in background  
**Steps:**
1. Minimize app
2. Use global hotkey (if configured)
3. Verify app activation
**Expected Result:** App comes to foreground and is ready for input  
**Priority:** P2

### TC-KEY-003
**Description:** Keyboard shortcuts with modifiers  
**Preconditions:** App active  
**Steps:**
1. Test combinations with Shift, Option, Control
2. Verify no conflicts with system shortcuts
**Expected Result:** Custom shortcuts work, no system conflicts  
**Priority:** P3

### TC-KEY-004
**Description:** Customizable shortcuts  
**Preconditions:** Settings accessible  
**Steps:**
1. Open keyboard shortcut preferences
2. Modify a shortcut
3. Test new shortcut
**Expected Result:** Custom shortcuts can be set and work correctly  
**Priority:** P3

---

## 12. Accessibility (VoiceOver)

### TC-ACC-001
**Description:** VoiceOver navigation  
**Preconditions:** VoiceOver enabled in System Preferences  
**Steps:**
1. Launch app with VoiceOver active
2. Navigate through main interface
3. Test all interactive elements
**Expected Result:** All UI elements are properly labeled and navigable with VoiceOver  
**Priority:** P1

### TC-ACC-002
**Description:** Screen reader chat interaction  
**Preconditions:** VoiceOver enabled  
**Steps:**
1. Send message using VoiceOver
2. Listen to response being read
3. Navigate chat history
**Expected Result:** Chat interface fully accessible, messages read clearly  
**Priority:** P1

### TC-ACC-003
**Description:** Keyboard-only navigation  
**Preconditions:** App running  
**Steps:**
1. Navigate entire interface using only keyboard
2. Test all functions without mouse
**Expected Result:** Full functionality available via keyboard navigation  
**Priority:** P1

### TC-ACC-004
**Description:** High contrast mode  
**Preconditions:** High contrast mode enabled in System Preferences  
**Steps:**
1. Launch app in high contrast mode
2. Verify readability
3. Test all interface elements
**Expected Result:** App displays correctly and remains usable in high contrast mode  
**Priority:** P2

### TC-ACC-005
**Description:** Font size scaling  
**Preconditions:** System font size increased  
**Steps:**
1. Increase system font size to maximum
2. Launch app
3. Test interface scaling
**Expected Result:** App interface scales appropriately, maintains usability  
**Priority:** P2

---

## 13. Performance Benchmarks

### TC-PRF-001
**Description:** App launch time  
**Preconditions:** App not running, cold start  
**Steps:**
1. Time from app launch to main interface ready
2. Repeat 10 times
3. Calculate average
**Expected Result:** Average launch time < 5 seconds, no launch failures  
**Priority:** P2

### TC-PRF-002
**Description:** Message response time  
**Preconditions:** AI provider configured, app ready  
**Steps:**
1. Send standard test message
2. Measure time to first response
3. Repeat with different message types
**Expected Result:** Response time < 10 seconds for simple queries, reasonable for complex ones  
**Priority:** P1

### TC-PRF-003
**Description:** Memory usage baseline  
**Preconditions:** Fresh app launch  
**Steps:**
1. Monitor memory usage at startup
2. After basic operations
3. After extended use (2+ hours)
**Expected Result:** Reasonable baseline memory usage, no major memory leaks  
**Priority:** P2

### TC-PRF-004
**Description:** CPU usage monitoring  
**Preconditions:** App running normally  
**Steps:**
1. Monitor CPU usage during idle state
2. During active AI processing
3. During background tasks
**Expected Result:** CPU usage reasonable for activities performed, returns to low idle usage  
**Priority:** P3

### TC-PRF-005
**Description:** Large file handling performance  
**Preconditions:** Large test files available  
**Steps:**
1. Process large document or image
2. Monitor performance impact
3. Verify completion
**Expected Result:** Large files handled without crashing, reasonable processing time  
**Priority:** P3

---

## Test Execution Guidelines

### Test Environment Setup
- Clean macOS installation (preferably multiple versions)
- Various network conditions (fast, slow, intermittent)
- Different hardware configurations (M1/M2/Intel Macs)
- Fresh user accounts for permission testing

### Test Data Requirements
- Test Google accounts with various data states
- Sample emails and calendar events
- Test API keys (valid, invalid, expired)
- Various file types and sizes

### Regression Testing
- Run P0 tests before every release
- Run P1 tests weekly during development
- Run full test suite before major releases
- Automated smoke tests for each build

### Bug Reporting
- Include exact steps to reproduce
- System information and logs
- Screenshots/screen recordings when applicable
- Expected vs actual behavior
- Severity assessment

### Success Criteria
- P0 tests: 100% pass rate required for release
- P1 tests: 95% pass rate required for release
- P2 tests: 90% pass rate for minor releases
- P3 tests: Advisory only, tracked for future improvement