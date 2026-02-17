# OpenPaw Onboarding Flow Specification

## Overview

The onboarding flow guides new users through initial setup of OpenPaw, establishing their identity, AI provider, service connections, preferences, and first interaction. The experience should feel welcoming, secure, and efficient.

**Design Principles:**
- Conversational and friendly tone
- Progressive disclosure (only show what's needed)
- Clear value proposition at each step
- Graceful error handling with helpful recovery
- Accessible to users with disabilities
- Skip options for power users

---

## Screen 1: Welcome

### Visual Design
- **Window Size:** 480x600px, centered on screen
- **Background:** Soft gradient from #F8F9FF to #FFFFFF
- **Icon:** Large OpenPaw paw print logo (128x128px) with subtle animation (gentle pulse)

### Content & Copy

```
[Large OpenPaw logo - animated paw print]

Hi! I'm OpenPaw 🐾

I'm your personal AI assistant for Mac. I can help you manage emails, 
schedule meetings, research topics, automate tasks, and much more.

Let's get you set up in just a few steps.

[Continue Button - Primary Blue]    [Maybe Later Button - Secondary Gray]
```

### Input Fields
- None on this screen

### Under the Hood
- Initialize application data directory: `~/Library/Application Support/OpenPaw/`
- Create initial config structure:
  ```json
  {
    "version": "1.0.0",
    "onboarding": {
      "started_at": "2026-02-17T17:47:00Z",
      "completed": false,
      "current_step": 1
    },
    "user": {},
    "ai": {},
    "services": {},
    "preferences": {}
  }
  ```
- Check system permissions (Accessibility, Automation)
- Log analytics event: `onboarding_started`

### Navigation
- **Continue:** Proceeds to Screen 2
- **Maybe Later:** Shows confirmation dialog:
  - "You can set up OpenPaw anytime from the menu bar. Continue to basic mode?"
  - **Yes:** Skip to minimal mode (menu bar only, limited features)
  - **No:** Stay on welcome screen
- **Back:** N/A (first screen)
- **Skip:** N/A

### Error States
- **System Permissions Missing:** Warning banner at top:
  - "⚠️ OpenPaw needs Accessibility permissions to help you effectively. [Grant Permission] [Continue Anyway]"
- **Disk Space Low:** Warning:
  - "⚠️ OpenPaw requires at least 100MB of free space. Please free up space and try again."

### Accessibility
- Logo alt text: "OpenPaw logo - a friendly paw print"
- Heading level: H1 for "Hi! I'm OpenPaw"
- Button focus indicators with 2px blue outline
- VoiceOver description: "Welcome to OpenPaw setup screen. Continue button to proceed with setup."
- Keyboard navigation: Tab cycles through buttons, Enter activates

---

## Screen 2: Name Input

### Visual Design
- Same window size and background as Screen 1
- Smaller OpenPaw logo (64x64px) in top-left corner
- Progress indicator: "Step 2 of 7"

### Content & Copy

```
[Small OpenPaw logo]                                      Step 2 of 7

What should I call you?

I'll use this name when we chat and in notifications.

[Name Input Field - Large, centered]
[Placeholder: "Enter your name"]

[Back Button - Secondary]    [Continue Button - Primary, disabled until input]
```

### Input Fields
- **Name Field:**
  - Type: Text input
  - Max length: 50 characters
  - Validation: Must contain at least 1 letter, no special characters except hyphens, apostrophes, spaces
  - Auto-focus on screen load
  - Auto-capitalization enabled

### Validation Rules
- **Real-time validation:**
  - Show green checkmark when valid
  - Show red warning for invalid characters
  - Enable Continue button only when valid
- **Error messages:**
  - Empty: "Please enter your name"
  - Invalid chars: "Names can only contain letters, spaces, hyphens, and apostrophes"
  - Too long: "Please keep your name under 50 characters"

### Under the Hood
- Save to config:
  ```json
  {
    "user": {
      "name": "User Input",
      "created_at": "2026-02-17T17:47:00Z"
    },
    "onboarding": {
      "current_step": 2
    }
  }
  ```
- Log analytics: `onboarding_name_entered`

### Navigation
- **Continue:** Validates input, saves name, proceeds to Screen 3
- **Back:** Returns to Screen 1 (preserves name input)
- **Skip:** Hover shows "Skip and use 'User'" tooltip, sets name to "User"

### Error States
- **Invalid Input:** Red border on field, error message below
- **Network Error (if cloud sync):** "Couldn't save your preferences. Continue anyway?"

### Accessibility
- Label: "What should I call you?"
- Required field indicator
- Error messages announced by screen reader
- Placeholder text: "Enter your name"
- Help text: "I'll use this name when we chat and in notifications"

---

## Screen 3: AI Provider Selection

### Visual Design
- Same window layout
- Three provider cards in vertical layout
- Each card shows logo, name, description, and "Select" button

### Content & Copy

```
[Small OpenPaw logo]                                      Step 3 of 7

Choose your AI provider

OpenPaw works with multiple AI providers. You can change this anytime.

┌─ OpenAI (ChatGPT) ─────────────────────────────────┐
│ [OpenAI Logo] GPT-4, DALL-E, advanced reasoning    │
│ Best for: Complex tasks, creative work             │
│ Cost: ~$20/month for typical usage                 │
│ [Select OpenAI] [Get API Key →]                    │
└────────────────────────────────────────────────────┘

┌─ Anthropic (Claude) ───────────────────────────────┐
│ [Anthropic Logo] Claude 3.5, thoughtful responses  │
│ Best for: Analysis, writing, conversation          │
│ Cost: ~$15/month for typical usage                 │
│ [Select Anthropic] [Get API Key →]                 │
└────────────────────────────────────────────────────┘

┌─ Local Model (Ollama) ─────────────────────────────┐
│ [Ollama Logo] Run AI on your Mac, private & free   │
│ Best for: Privacy, offline use, experimentation    │
│ Requires: 8GB+ RAM, will download ~4GB             │
│ [Select Local] [Install Ollama →]                  │
└────────────────────────────────────────────────────┘

After selecting, you'll enter your API key or install software.

[Back Button]                            [Continue Button - disabled]
```

### Input Fields
- **Provider Selection:** Radio button group (single selection required)
- **API Key Field:** Appears after provider selection (except for Local)
  - Type: Password field (masked input)
  - Placeholder: "Paste your API key here"
  - Validation: Provider-specific format checking

### Provider-Specific Flows

#### OpenAI Selected:
```
✓ OpenAI Selected

API Key: [Password field - sk-...]
[Test Connection Button]

New to OpenAI? [Get API Key →] opens https://platform.openai.com/api-keys
```

#### Anthropic Selected:
```
✓ Anthropic Selected  

API Key: [Password field - sk-ant...]
[Test Connection Button]

New to Anthropic? [Get API Key →] opens https://console.anthropic.com/
```

#### Local Model Selected:
```
✓ Local Model Selected

Checking for Ollama installation...
[Progress Spinner] Installing Ollama and Llama2 model (4.2GB)...

Or install manually: [Download Ollama →] opens https://ollama.ai/download
```

### Under the Hood
- **OpenAI/Anthropic:**
  - Test API key with simple request: `{"model": "...", "messages": [{"role": "user", "content": "Hi"}]}`
  - Save encrypted API key to keychain
  - Save provider config:
    ```json
    {
      "ai": {
        "provider": "openai",
        "model": "gpt-4",
        "api_key_saved": true,
        "verified_at": "2026-02-17T17:47:00Z"
      }
    }
    ```

- **Local Model:**
  - Check if Ollama installed: `which ollama`
  - If not installed, download and install Ollama
  - Pull default model: `ollama pull llama2`
  - Test local connection: `ollama list`

### Navigation
- **Continue:** Enabled only after successful provider setup and API key verification
- **Back:** Returns to Screen 2
- **Skip:** Shows warning: "OpenPaw needs an AI provider to work. Choose one to continue."

### Error States
- **Invalid API Key:**
  - Red border on API key field
  - Error message: "Invalid API key. Please check and try again."
  - "Get API Key" link highlighted
  
- **Network Error:**
  - "Couldn't connect to [Provider]. Check your internet and try again."
  - [Retry Button] [Continue Offline] options

- **Ollama Install Failed:**
  - "Couldn't install Ollama automatically. Please download manually:"
  - [Download Ollama] button prominent
  - [Try Again] [Choose Different Provider] options

- **Insufficient Disk Space (Local):**
  - "Need 5GB free space for local AI model. Free up space or choose a cloud provider."

### Accessibility
- Fieldset with legend: "Choose your AI provider"
- Each provider card is a radio button with detailed description
- API key fields labeled clearly: "API key for [Provider]"
- Loading states announced: "Installing Ollama, please wait"
- Error messages associated with fields via aria-describedby

---

## Screen 4: Service Connections

### Visual Design
- Grid layout of service cards (2x3 or 2x4)
- Each service shows icon, name, status, and connect button
- "Skip All" option at bottom

### Content & Copy

```
[Small OpenPaw logo]                                      Step 4 of 7

Connect your services

I can help you more effectively with access to your accounts.
You can skip any service and connect them later in settings.

┌─ Gmail ─────┐ ┌─ Calendar ──┐ ┌─ Contacts ──┐
│ [📧 Gmail]  │ │ [📅 Cal]    │ │ [👥 People] │
│ Read & send │ │ Events &    │ │ Find people │
│ emails      │ │ scheduling  │ │ & details   │
│ [Connect]   │ │ [Connect]   │ │ [Connect]   │
└─────────────┘ └─────────────┘ └─────────────┘

┌─ Spotify ───┐ ┌─ Slack ─────┐ ┌─ Files ─────┐
│ [🎵 Music]  │ │ [💬 Slack]  │ │ [📁 Files]  │
│ Control     │ │ Send msgs & │ │ Search &    │
│ playback    │ │ read chats  │ │ organize    │
│ [Connect]   │ │ [Connect]   │ │ [Connect]   │
└─────────────┘ └─────────────┘ └─────────────┘

✓ = Connected    ⚠️ = Needs attention    ⊘ = Skipped

[Skip All Services]          [Back]    [Continue]
```

### Service Connection Flows

Each service opens OAuth flow or shows specific instructions:

#### Gmail Connection:
```
Connecting to Gmail...

This will open your browser to sign in with Google.
OpenPaw will request permission to:
• Read and send emails
• Access your contacts  
• Manage your calendar

[Open Browser] [Cancel]

After authorization:
✓ Connected to [user@gmail.com]
```

#### Slack Connection:
```
Connecting to Slack...

Choose your workspace:
• [Workspace 1] My Company Slack
• [Workspace 2] Side Project Team
• [+ Add Different Workspace]

[Continue with My Company Slack]

✓ Connected to My Company Slack
```

### Under the Hood
- **OAuth Flow Management:**
  - Start local HTTP server on random port (e.g., localhost:8080)
  - Open browser to provider's OAuth URL with redirect_uri
  - Capture authorization code
  - Exchange for access/refresh tokens
  - Store encrypted tokens in keychain
  - Update config:
    ```json
    {
      "services": {
        "gmail": {
          "connected": true,
          "email": "user@gmail.com",
          "connected_at": "2026-02-17T17:47:00Z",
          "scopes": ["gmail.readonly", "gmail.send", "calendar.readonly"]
        }
      }
    }
    ```

- **Permission Handling:**
  - Request macOS permissions for each service as needed
  - Store permission status in config

### Navigation
- **Continue:** Enabled always (can proceed with 0 connected services)
- **Back:** Returns to Screen 3
- **Skip All:** Marks all services as skipped, proceeds to Screen 5
- **Individual Skip:** Per-service skip option

### Error States
- **OAuth Failed:**
  - "Couldn't connect to [Service]. This might be due to:"
  - "• Network connection issues"
  - "• [Service] being temporarily unavailable"
  - "• Browser blocking the connection"
  - [Try Again] [Skip This Service] [Get Help]

- **Permission Denied:**
  - "OpenPaw needs permission to access [Service]. Enable in System Preferences > Privacy & Security > [Service]"
  - [Open System Preferences] [Continue Without]

- **Service Unavailable:**
  - "Can't reach [Service] right now. You can connect later in Settings."
  - Status shown as ⚠️ with retry option

### Accessibility
- Grid navigation with arrow keys
- Each service card is a focusable button
- Connection status announced: "Gmail: Connected" or "Slack: Not connected"
- OAuth flow includes screen reader announcements
- Loading states clearly communicated

---

## Screen 5: What Should I Help With?

### Visual Design
- Toggle grid of capability categories
- Each category has icon, title, description, and toggle switch
- Categories grouped logically

### Content & Copy

```
[Small OpenPaw logo]                                      Step 5 of 7

What should I help with?

Choose areas where you'd like OpenPaw to assist. You can change these anytime.

PRODUCTIVITY                               [Toggle On/Off]
┌─ Email Management ────────────────────────────────── [●]─┐
│ 📧 Smart sorting, quick replies, follow-up reminders    │
└──────────────────────────────────────────────────────────┘

┌─ Calendar & Scheduling ───────────────────────────── [●]─┐
│ 📅 Meeting coordination, time blocking, reminders      │
└──────────────────────────────────────────────────────────┘

┌─ Task & Project Management ──────────────────────── [ ]─┐
│ ✅ Break down projects, track progress, set deadlines   │
└──────────────────────────────────────────────────────────┘

COMMUNICATION                              [Toggle On/Off]
┌─ Message Writing ─────────────────────────────────── [●]─┐
│ ✍️ Draft emails, texts, and messages with your tone     │
└──────────────────────────────────────────────────────────┘

┌─ Meeting Support ─────────────────────────────────── [ ]─┐
│ 🎙️ Take notes, summarize calls, track action items      │
└──────────────────────────────────────────────────────────┘

RESEARCH & ANALYSIS                        [Toggle On/Off]
┌─ Web Research ────────────────────────────────────── [●]─┐
│ 🔍 Find information, compare options, fact-check        │
└──────────────────────────────────────────────────────────┘

┌─ Document Analysis ───────────────────────────────── [ ]─┐
│ 📄 Summarize PDFs, extract key info, compare docs       │
└──────────────────────────────────────────────────────────┘

CREATIVE & PERSONAL                        [Toggle On/Off]
┌─ Writing & Creativity ────────────────────────────── [ ]─┐
│ ✨ Help with writing, brainstorming, creative projects   │
└──────────────────────────────────────────────────────────┘

┌─ Personal Organization ───────────────────────────── [●]─┐
│ 🏠 Manage personal tasks, shopping lists, reminders     │
└──────────────────────────────────────────────────────────┘

[Select All] [Select None]      [Back]    [Continue]
```

### Input Fields
- **Category Toggles:** 9 toggle switches (default: 4 enabled as shown)
- **Custom Categories:** Optional text field: "Anything else? Describe what you'd like help with..."

### Toggle Behavior
- Each toggle is independent
- Visual feedback: Smooth animation, color change
- At least one category must be enabled to continue
- Defaults chosen based on most common use cases

### Under the Hood
- Save preferences:
  ```json
  {
    "preferences": {
      "capabilities": {
        "email_management": true,
        "calendar_scheduling": true,
        "task_management": false,
        "message_writing": true,
        "meeting_support": false,
        "web_research": true,
        "document_analysis": false,
        "writing_creativity": false,
        "personal_organization": true
      },
      "custom_help": "Help me with home automation and tech troubleshooting"
    }
  }
  ```

- Initialize relevant services and permissions based on selections
- Set up default workflows for enabled categories

### Navigation
- **Continue:** Enabled only if at least one category selected
- **Back:** Returns to Screen 4 (preserves toggle states)
- **Skip:** Not available (must select at least one)

### Error States
- **No Categories Selected:**
  - Message appears: "Please select at least one area where you'd like help"
  - Continue button remains disabled
  - Suggested action: Highlight most popular categories

### Accessibility
- Fieldset with legend: "What should I help with?"
- Each toggle labeled with category name and description
- Toggle state announced: "Email Management: On" or "Task Management: Off"
- Keyboard navigation: Tab between toggles, Space to activate
- "Select All" and "Select None" buttons for quick configuration

---

## Screen 6: Approval Preferences

### Visual Design
- Three approval modes presented as cards
- Each card shows icon, name, description, and example
- Clear recommendation for new users

### Content & Copy

```
[Small OpenPaw logo]                                      Step 6 of 7

How should I ask permission?

Choose how OpenPaw should handle actions that affect your data or accounts.

┌─ AUTOMATIC ─────────────────────────────────────────────┐
│ 🚀 Go ahead and do it                                    │
│                                                         │
│ OpenPaw performs actions immediately based on context.  │
│ Example: "Send that email" → Email sent instantly      │
│                                                         │
│ ⚡ Fastest experience                                    │
│ ⚠️  Higher risk if I misunderstand                     │
│                                                         │
│ [ Select Automatic ]                                    │
└─────────────────────────────────────────────────────────┘

┌─ CONFIRM FIRST ─────────────────────────────────────────┐
│ ✋ Show me first, then I'll approve        [RECOMMENDED] │
│                                                         │
│ OpenPaw drafts actions and asks for your OK.           │
│ Example: "Send this email? [Draft] [Send] [Edit]"      │
│                                                         │
│ ⚖️  Balanced speed and safety                          │
│ 👍 Most popular choice                                  │
│                                                         │
│ [ Select Confirm First ]                                │
└─────────────────────────────────────────────────────────┘

┌─ STRICT APPROVAL ───────────────────────────────────────┐
│ 🛡️ Ask me before every action                           │
│                                                         │
│ OpenPaw asks permission for all actions, even small.   │
│ Example: "May I search your email for that receipt?"   │
│                                                         │
│ 🔒 Maximum control and privacy                          │
│ 🐢 Slower, but you approve everything                   │
│                                                         │
│ [ Select Strict Approval ]                              │
└─────────────────────────────────────────────────────────┘

You can change this anytime in Settings, or set per-action overrides.

[Back]                                          [Continue]
```

### Input Fields
- **Approval Mode:** Radio button group (single selection required)
- Default selection: "Confirm First" (recommended)

### Under the Hood
- Save preference:
  ```json
  {
    "preferences": {
      "approval_mode": "confirm_first",
      "action_overrides": {},
      "trusted_actions": [],
      "set_at": "2026-02-17T17:47:00Z"
    }
  }
  ```

- Configure action handling system based on selection:
  - **Automatic:** Set default timeout of 2 seconds for confirmations
  - **Confirm First:** Show preview dialogs for major actions
  - **Strict:** Require explicit permission for all actions

### Approval Mode Details

#### Automatic Mode Setup:
- Create whitelist of safe actions (read-only operations, basic searches)
- Set up smart context detection for ambiguous requests
- Configure rollback mechanisms for reversible actions

#### Confirm First Mode Setup:
- Define "major actions" requiring approval (send email, delete files, make purchases)
- Set up draft preview system
- Configure quick approval shortcuts (Cmd+Enter = approve)

#### Strict Mode Setup:
- All actions require explicit permission
- Set up detailed permission explanations
- Configure granular action categories for bulk approval

### Navigation
- **Continue:** Enabled always (default selection: Confirm First)
- **Back:** Returns to Screen 5

### Error States
- **Configuration Error:**
  - "Couldn't save your approval preferences. Your selection: [Mode]. Continue anyway?"
  - [Retry] [Continue] buttons

### Accessibility
- Fieldset with legend: "How should I ask permission?"
- Each mode card is a radio button with detailed description
- Recommended option clearly marked and announced
- Examples help users understand each mode
- Keyboard navigation with Enter to select

---

## Screen 7: Ready! First Task Suggestion

### Visual Design
- Celebration/completion feel
- Large checkmark or completion animation
- Suggested first tasks as clickable cards
- Clear path to main application

### Content & Copy

```
[Large checkmark animation]

🎉 You're all set, [User Name]!

OpenPaw is ready to help. Here are some things you can try right away:

┌─ Quick Wins ──────────────────────────────────────────────┐
│ 📧 "Check my email and summarize important messages"       │
│ 📅 "What's on my calendar today?"                         │
│ 🔍 "Find me the best coffee shops near downtown"          │
│ ✍️  "Help me write a professional follow-up email"        │
└───────────────────────────────────────────────────────────┘

┌─ Try This Now ────────────────────────────────────────────┐
│ Based on your setup, I suggest:                           │
│                                                           │
│ "Hi OpenPaw, check my Gmail and tell me if there's       │
│  anything urgent I should handle today."                  │
│                                                           │
│ [Try This Suggestion]                                     │
└───────────────────────────────────────────────────────────┘

OpenPaw is now running in your menu bar. Just click the paw icon 🐾 
or use the keyboard shortcut ⌘⇧O to start a conversation anytime.

[Open OpenPaw Now]                    [Finish Setup]
```

### Dynamic Content
The "Try This Now" suggestion is personalized based on onboarding choices:

- **If Gmail connected:** Email-related suggestion
- **If Calendar connected:** Schedule/meeting suggestion  
- **If Research enabled:** Information-finding suggestion
- **If Creative Writing enabled:** Writing assistance suggestion
- **Multiple connections:** Combined suggestion

### Input Fields
- **Pre-filled suggestion:** Text area with suggested first command (editable)
- User can modify the suggestion before trying it

### Under the Hood
- Mark onboarding as complete:
  ```json
  {
    "onboarding": {
      "completed": true,
      "completed_at": "2026-02-17T17:47:00Z",
      "total_time_minutes": 8
    }
  }
  ```

- Initialize core systems:
  - Start background services for connected accounts
  - Set up menu bar icon and global shortcuts
  - Initialize chat interface
  - Register global hotkey (⌘⇧O)
  - Start periodic sync for connected services

- Log analytics:
  - `onboarding_completed`
  - Selected services, AI provider, approval mode
  - Time to complete each step

### Navigation
- **Try This Suggestion:** Opens main chat window with suggestion pre-filled
- **Open OpenPaw Now:** Opens main chat window (blank)
- **Finish Setup:** Closes onboarding, shows menu bar icon
- **Back:** Returns to Screen 6 (for any last-minute changes)

### Error States
- **Service Initialization Failed:**
  - "Setup complete! Note: [Service] couldn't start automatically. You can reconnect in Settings."
  - Continue to main app with warning indicator

- **Menu Bar Registration Failed:**
  - "Setup complete! Couldn't add menu bar icon due to system permissions. OpenPaw is still running."
  - [Try Again] [Open System Preferences] options

### Accessibility
- Completion announcement: "Setup complete! OpenPaw is ready to use."
- Suggested tasks announced as actionable buttons
- Keyboard shortcuts clearly stated
- First task suggestion is editable text area with label

---

## Global Navigation & Error Recovery

### Progress Tracking
- Visual progress indicator on all screens (Step X of 7)
- Progress bar at bottom showing completion percentage
- Ability to jump to any completed step

### Universal Back/Forward
- **Back button:** Always available (except Screen 1)
- **Keyboard shortcuts:** ⌘← (back), ⌘→ (forward), ESC (cancel)
- **Progress preservation:** All inputs saved when navigating
- **Session recovery:** Resume onboarding if app crashes/quits

### Skip Patterns
- **Individual skips:** Most screens allow skipping specific items
- **Section skips:** Entire sections can be skipped with warnings
- **Quick setup:** "Skip to basic setup" option on every screen after Screen 2

### Error Recovery
- **Network errors:** Offline mode with sync when connection returns
- **Permission errors:** Clear guidance to system preferences with deep links
- **Service errors:** Graceful degradation, continue without failed service
- **Data corruption:** Reset to last known good state

### Accessibility Standards
- **WCAG 2.1 AA compliance:** All screens meet accessibility guidelines
- **Keyboard navigation:** Full keyboard operation without mouse
- **Screen reader support:** Proper labels, descriptions, and announcements
- **High contrast mode:** Support for system high contrast settings
- **Voice control:** Support for macOS Voice Control commands
- **Reduced motion:** Respect system animation preferences

### Responsive Design
- **Window resizing:** Minimum 400x500px, maximum 600x800px
- **Text scaling:** Support for system text size preferences
- **Layout adaptation:** Reflow for different window sizes
- **Component scaling:** UI elements scale with system settings

---

## Technical Implementation Notes

### Configuration File Structure
```json
{
  "version": "1.0.0",
  "user": {
    "name": "John Doe",
    "created_at": "2026-02-17T17:47:00Z",
    "timezone": "America/New_York"
  },
  "ai": {
    "provider": "openai",
    "model": "gpt-4",
    "api_key_saved": true,
    "verified_at": "2026-02-17T17:47:00Z"
  },
  "services": {
    "gmail": {
      "connected": true,
      "email": "user@gmail.com",
      "connected_at": "2026-02-17T17:47:00Z",
      "scopes": ["gmail.readonly", "gmail.send"]
    }
  },
  "preferences": {
    "approval_mode": "confirm_first",
    "capabilities": {
      "email_management": true,
      "calendar_scheduling": true
    }
  },
  "onboarding": {
    "completed": true,
    "completed_at": "2026-02-17T17:47:00Z"
  }
}
```

### Security Considerations
- **API Keys:** Stored in macOS Keychain, never in plain text
- **OAuth Tokens:** Encrypted and stored securely  
- **Local Storage:** Configuration files have restricted permissions (600)
- **Network:** All API calls use HTTPS with certificate pinning
- **Validation:** Input sanitization and validation on all user inputs