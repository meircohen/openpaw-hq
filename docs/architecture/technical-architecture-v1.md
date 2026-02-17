# Pilot Technical Architecture v1.0
**Robotic Process Automation Agent for macOS**

---

## Table of Contents
1. [System Overview](#system-overview)
2. [Component Architecture](#component-architecture)
3. [Data Flow Analysis](#data-flow-analysis)
4. [Technology Stack](#technology-stack)
5. [Performance Requirements](#performance-requirements)
6. [Security & Privacy Model](#security--privacy-model)
7. [AI App Integrations](#ai-app-integrations)
8. [Local Storage Schema](#local-storage-schema)
9. [Plugin & Workflow Extension Model](#plugin--workflow-extension-model)
10. [Technical Risks & Mitigations](#technical-risks--mitigations)

---

## System Overview

Pilot is a macOS-native RPA agent that automates desktop tasks by leveraging existing AI applications as reasoning engines. The system operates entirely locally without cloud dependencies or API keys.

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     Pilot Application                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Chat UI       │  │ Task Orchestrator│  │ Memory Store │ │
│  │   (AppKit)      │  │   (Swift)        │  │  (SQLite)    │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
│           │                     │                   │       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │ AI App Bridge   │  │ Screen Capture   │  │ Virtual Input│ │
│  │ (Multi-modal)   │  │  (CGDisplay)     │  │ (CGEvent)    │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
           │                     │                   │
┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐
│ Virtual Display │  │   User Display   │  │ Target Apps  │
│ (CGVirtualDisp) │  │  (Screenshots)   │  │ (Automated)  │
└─────────────────┘  └─────────────────┘  └──────────────┘
           │                     
┌─────────────────┐              
│   AI Desktop    │              
│   Applications  │              
│ (ChatGPT/Claude)│              
└─────────────────┘              

         Fallback Path
┌─────────────────┐
│ MLX Local Model │
│  (Lightweight)  │
└─────────────────┘
```

### Core Principles
- **Zero Cloud Dependencies**: All processing occurs on local hardware
- **AI App Agnostic**: Works with any desktop AI application
- **Non-Intrusive Operation**: Uses virtual display to avoid user interference
- **Privacy First**: No data leaves the user's machine
- **Extensible**: Plugin architecture for custom workflows

---

## Component Architecture

### 1. Chat UI Component
**Technology**: AppKit (NSViewController, NSTableView, NSTextField)
**Responsibility**: Primary user interface for task input and status monitoring

```swift
class ChatViewController: NSViewController {
    @IBOutlet weak var conversationTableView: NSTableView
    @IBOutlet weak var inputTextField: NSTextField
    
    private let taskOrchestrator = TaskOrchestrator()
    private let memoryStore = MemoryStore()
    
    // Real-time task status updates
    // Conversation history display
    // Task queue management UI
}
```

**Key Features**:
- Real-time conversation view with AI responses
- Task queue visualization with progress indicators
- Manual intervention controls (pause/resume/cancel)
- Screenshot preview window for debugging
- Memory context browser

### 2. Task Orchestrator Component
**Technology**: Swift with Combine framework for reactive programming
**Responsibility**: Core business logic and task execution coordination

```swift
class TaskOrchestrator: ObservableObject {
    private let screenCapture = ScreenCaptureManager()
    private let aiAppBridge = AIAppBridgeManager()
    private let virtualInput = VirtualInputManager()
    private let memoryStore = MemoryStore()
    
    @Published var currentTask: Task?
    @Published var taskQueue: [Task] = []
    @Published var executionState: ExecutionState = .idle
    
    func executeTask(_ task: Task) -> AnyPublisher<TaskResult, TaskError>
    func pauseExecution()
    func resumeExecution()
    func cancelTask(_ taskId: UUID)
}
```

**Execution Pipeline**:
1. Natural language parsing and intent recognition
2. Screenshot capture and context analysis
3. AI app communication and response parsing
4. Action sequence generation and validation
5. Automated execution with error handling
6. Results verification and memory storage

### 3. Screen Capture Component
**Technology**: Core Graphics (CGDisplay, CGImage, CGWindowList)
**Responsibility**: High-performance screenshot capture and image processing

```swift
class ScreenCaptureManager {
    private let captureQueue = DispatchQueue(label: "com.pilot.screencapture", qos: .userInitiated)
    private var virtualDisplayID: CGDirectDisplayID?
    
    func captureMainDisplay() -> CGImage?
    func captureWindow(withID windowID: CGWindowID) -> CGImage?
    func captureVirtualDisplay() -> CGImage?
    func createVirtualDisplay(width: Int, height: Int) -> CGDirectDisplayID?
    
    // Optimizations
    func captureRegion(_ rect: CGRect) -> CGImage?
    func captureDifference(from previous: CGImage) -> CGImage?
}
```

**Performance Optimizations**:
- Differential image capture (only changed regions)
- Configurable capture regions to reduce processing overhead
- Hardware acceleration via Metal Performance Shaders for image processing
- Adaptive resolution scaling based on task complexity

### 4. Virtual Input Component
**Technology**: Core Graphics Events (CGEvent, CGEventSource)
**Responsibility**: Synthetic user input generation and gesture automation

```swift
class VirtualInputManager {
    private let eventSource: CGEventSource
    private var clickQueue: [InputEvent] = []
    
    func click(at point: CGPoint, button: CGMouseButton = .left)
    func doubleClick(at point: CGPoint)
    func drag(from: CGPoint, to: CGPoint)
    func scroll(at point: CGPoint, deltaY: Int32)
    func type(text: String)
    func keyPress(_ key: CGKeyCode, modifiers: CGEventFlags = [])
    
    // Advanced gestures
    func rightClickMenu(at point: CGPoint) -> [MenuItem]
    func selectText(from: CGPoint, to: CGPoint)
    func executeKeyboardShortcut(_ shortcut: KeyboardShortcut)
}
```

**Safety Features**:
- Input rate limiting to prevent system overload
- Emergency stop mechanism (specific key combination)
- Input validation and bounds checking
- Undo stack for reversible actions

### 5. AI App Bridge Component
**Technology**: Accessibility APIs (AXUIElement) + Computer Vision
**Responsibility**: Multi-modal communication with AI desktop applications

```swift
class AIAppBridgeManager {
    private let supportedApps: [AIAppAdapter] = [
        ChatGPTAdapter(),
        ClaudeAdapter(),
        GeminiAdapter(),
        OllamaAdapter(),
        LocalMLXAdapter()
    ]
    
    func detectAvailableAIApps() -> [AIApp]
    func sendMessage(_ message: String, to app: AIApp) -> AnyPublisher<AIResponse, AIError>
    func sendScreenshot(_ image: CGImage, to app: AIApp) -> AnyPublisher<AIResponse, AIError>
    func readResponse(from app: AIApp) -> String?
    
    // Fallback mechanism
    func useLocalModel(_ prompt: String) -> String
}
```

**App-Specific Adapters**:
Each AI app requires a custom adapter implementing the `AIAppProtocol`:

```swift
protocol AIAppProtocol {
    func isRunning() -> Bool
    func bringToForeground()
    func sendText(_ text: String)
    func uploadImage(_ image: CGImage)
    func readLastResponse() -> String?
    func clearConversation()
    func getWindowBounds() -> CGRect
}
```

### 6. Memory Store Component
**Technology**: SQLite with Core Data wrapper
**Responsibility**: Persistent storage and context management

```swift
class MemoryStore {
    private let persistentContainer: NSPersistentContainer
    private let context: NSManagedObjectContext
    
    func saveTask(_ task: Task)
    func loadTaskHistory() -> [Task]
    func saveConversation(_ conversation: Conversation)
    func loadRecentConversations(limit: Int) -> [Conversation]
    func saveScreenshot(_ image: CGImage, taskId: UUID)
    func loadScreenshots(for taskId: UUID) -> [Screenshot]
    
    // Context retrieval
    func getRelevantContext(for task: Task) -> TaskContext
    func updateUserPreferences(_ preferences: UserPreferences)
}
```

---

## Data Flow Analysis

### Typical Task: "Check my email and draft replies"

```
1. User Input Processing
   User: "Check my email and draft replies"
   ├─→ TaskOrchestrator.parseIntent(input)
   ├─→ MemoryStore.getRelevantContext(intent: .emailManagement)
   └─→ Task.create(type: .emailCheck, context: userContext)

2. Environment Assessment
   ├─→ ScreenCapture.captureMainDisplay()
   ├─→ AIAppBridge.detectAvailableAIApps()
   └─→ VirtualInputManager.validateSystemAccess()

3. Task Execution Loop
   While (task.isComplete == false) {
     a. Context Capture
        ├─→ ScreenCapture.captureCurrentState()
        ├─→ MemoryStore.getTaskHistory(similar: task)
        └─→ TaskOrchestrator.buildPrompt(context, history, goal)
     
     b. AI Reasoning
        ├─→ AIAppBridge.sendScreenshot(currentScreen, to: primaryAI)
        ├─→ AIAppBridge.sendPrompt(contextualPrompt, to: primaryAI)
        ├─→ AIAppBridge.readResponse(from: primaryAI)
        └─→ TaskOrchestrator.parseActionSequence(aiResponse)
     
     c. Action Execution
        ├─→ VirtualInputManager.executeActions(actionSequence)
        ├─→ ScreenCapture.captureResult()
        ├─→ TaskOrchestrator.validateSuccess(expected, actual)
        └─→ MemoryStore.saveStep(task, action, result)
   }

4. Results Processing
   ├─→ TaskOrchestrator.generateSummary(task.steps)
   ├─→ MemoryStore.saveTask(completedTask)
   ├─→ ChatUI.displayResults(summary, artifacts)
   └─→ User.notify(taskComplete)
```

### Data Flow Diagram

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ User Input  │─1─→│Task Parser  │─2─→│ Context     │
│  "Check     │    │ Intent:     │    │ Builder     │
│   Email"    │    │ EMAIL_CHECK │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
                                              │3
                                              ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Screenshot   │◄4──│Screen       │◄5──│Task         │
│ current.png │    │ Capture     │    │Orchestrator │
│ 1920x1080   │    │ Manager     │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
        │                                     │6
        └──────────────┐               ┌─────┘
                       ▼               ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   ChatGPT   │◄7──│AI App       │◄8──│ Prompt      │
│ Desktop App │    │ Bridge      │    │ Generator   │
│  response   │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
        │9                 │10              
        ▼                  ▼               
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Action       │─11→│Virtual      │─12→│Target       │
│Sequence     │    │Input        │    │Application  │
│[click, type]│    │Manager      │    │ (Mail.app)  │
└─────────────┘    └─────────────┘    └─────────────┘
                            │13
                            ▼
                   ┌─────────────┐
                   │ Memory      │
                   │ Store       │
                   │ (SQLite)    │
                   └─────────────┘
```

---

## Technology Stack

### Core Framework: Swift + AppKit
**Justification**: 
- Native macOS performance and system integration
- Direct access to Core Graphics and Accessibility APIs
- Automatic memory management prevents leaks during long-running tasks
- Strong type safety reduces runtime errors in automation scenarios

### Screen Capture: Core Graphics + Metal
**Components**:
- `CGDisplayCreateImage()` for full screen capture
- `CGWindowListCreateImage()` for window-specific capture
- Metal Performance Shaders for image processing acceleration
- `IOSurface` for efficient image buffer management

**Performance Rationale**:
- Hardware-accelerated image processing
- Minimal CPU overhead for continuous capture
- Direct GPU memory access eliminates CPU-GPU transfer bottlenecks

### Virtual Display: CGVirtualDisplay (macOS 14+)
**Implementation**:
```swift
let virtualDisplayDescriptor = CGVirtualDisplayDescriptor()
virtualDisplayDescriptor.width = 1920
virtualDisplayDescriptor.height = 1080
virtualDisplayDescriptor.colorSpace = CGColorSpace(name: CGColorSpace.sRGB)
virtualDisplayDescriptor.name = "Pilot Virtual Display"

let virtualDisplay = CGVirtualDisplay(descriptor: virtualDisplayDescriptor)
```

**Benefits**:
- Isolated AI app environment prevents user interference
- Predictable screen dimensions for consistent automation
- Hidden from user's normal workflow

### Input Automation: Core Graphics Events
**Event System**:
- `CGEventSource` for synthetic input generation
- `CGEventCreateMouseEvent()` for precise clicking
- `CGEventCreateKeyboardEvent()` for text input
- `CGEventPost()` with configurable targets

**Rate Limiting**:
```swift
private let inputThrottle = AsyncThrottler(interval: 0.1) // 100ms between actions
```

### Database: SQLite + Core Data
**Schema Design**:
- Entity relationship mapping with managed object contexts
- Automatic migration support for version updates
- Full-text search capabilities for conversation history
- Efficient indexing for task retrieval

### Local AI: MLX Framework
**Fallback Model Integration**:
```swift
class MLXAdapter: AIAppProtocol {
    private let model: MLXLanguageModel
    
    init() {
        // Load quantized 3B parameter model (~2GB RAM)
        self.model = MLXLanguageModel.load("phi-3-mini-4k-instruct-q4")
    }
    
    func generateResponse(_ prompt: String) -> String {
        return model.generate(prompt, maxTokens: 512)
    }
}
```

**Model Selection Criteria**:
- Sub-3B parameters for reasonable inference speed
- Quantized (4-bit) to minimize memory footprint
- Strong instruction-following capability
- Commercial-use license compatibility

### Networking: None (Local-Only Architecture)
**Zero Network Dependencies**:
- No API calls to external services
- No telemetry or analytics collection
- All processing occurs on local hardware
- Update mechanism via local package installer

---

## Performance Requirements

### Response Time Targets
- **User Input to AI Response**: < 5 seconds
- **AI Response to Action Execution**: < 2 seconds
- **Screenshot Capture**: < 100ms
- **Action Execution**: < 500ms per action
- **Task Completion Notification**: < 1 second

### Resource Utilization Constraints
- **CPU Usage**: Average < 20%, peak < 50% (excluding AI inference)
- **Memory Usage**: Base application < 100MB, with task context < 500MB
- **Disk I/O**: Screenshot storage optimized, max 10MB/hour typical usage
- **Battery Impact**: Minimal when idle, moderate during active automation

### Scalability Metrics
- **Concurrent Tasks**: Support up to 3 parallel task executions
- **Screenshot History**: Retain last 100 screenshots per task
- **Conversation Memory**: 10,000 messages with 30-day auto-cleanup
- **Task Queue**: Handle up to 20 queued tasks

### Platform Specifications
**Minimum Requirements**:
- macOS 14.0 (Sonoma) for CGVirtualDisplay support
- Apple Silicon (M1/M2/M3) or Intel x64 with 8GB RAM
- 2GB available storage for models and data
- Accessibility permissions enabled

**Recommended Configuration**:
- macOS 15.0+ (Sequoia) for latest security features
- Apple Silicon M2 Pro or better for AI model inference
- 16GB RAM for optimal performance with multiple AI apps
- SSD storage for fast database operations

### Performance Monitoring
```swift
class PerformanceMonitor {
    func trackTaskLatency(_ task: Task, duration: TimeInterval)
    func trackMemoryUsage() -> MemoryReport
    func trackCPUUsage() -> CPUReport
    func generateDailyReport() -> PerformanceReport
}
```

---

## Security & Privacy Model

### Data Protection Principles
1. **Local Processing Only**: No data transmission to external servers
2. **Minimal Data Retention**: Configurable history cleanup policies
3. **User Consent**: Explicit permission for each automation capability
4. **Transparent Operation**: Full audit trail of all actions performed

### macOS Security Integration

#### Permissions Management
```swift
class SecurityManager {
    func requestAccessibilityPermissions() -> Bool
    func requestScreenRecordingPermissions() -> Bool
    func requestInputMonitoringPermissions() -> Bool
    func validateSecurityRequirements() -> [SecurityRequirement]
    
    private func checkSystemIntegrityProtection() -> Bool
    private func validateCodeSigning() -> Bool
}
```

**Required Permissions**:
- **Accessibility**: Required for reading UI elements and sending synthetic events
- **Screen Recording**: Required for capturing screenshots
- **Input Monitoring**: Required for detecting user intervention commands

#### Sandboxing Strategy
- **App Sandbox**: Disabled due to system automation requirements
- **Network Entitlements**: None (no network access requested)
- **Hardware Entitlements**: Camera/microphone access denied
- **File System**: Limited to user-authorized directories

### Data Encryption
```swift
class SecureStorage {
    private let encryptionKey: SymmetricKey
    
    func encrypt(_ data: Data) throws -> Data
    func decrypt(_ encryptedData: Data) throws -> Data
    
    // Sensitive data encryption at rest
    func storeSecurePreference<T: Codable>(_ value: T, key: String)
    func loadSecurePreference<T: Codable>(key: String, type: T.Type) -> T?
}
```

**Encryption Scope**:
- User preferences and configuration
- Task history containing sensitive information
- Screenshot data with personally identifiable content
- Conversation logs with AI applications

### Privacy Controls

#### Data Lifecycle Management
```swift
enum DataRetentionPolicy {
    case session     // Delete on app quit
    case daily       // Delete after 24 hours
    case weekly      // Delete after 7 days
    case monthly     // Delete after 30 days
    case manual      // User-controlled deletion
}

class DataLifecycleManager {
    func setRetentionPolicy(for dataType: DataType, policy: DataRetentionPolicy)
    func executeCleanupPolicy()
    func exportUserData() -> DataExport
    func deleteAllUserData()
}
```

#### Sensitive Content Detection
```swift
class PrivacyFilter {
    func detectSensitiveContent(in image: CGImage) -> [SensitiveRegion]
    func redactSensitiveContent(in image: CGImage) -> CGImage
    func shouldStoreScreenshot(_ image: CGImage) -> Bool
    
    // Pattern recognition for common sensitive data
    private func detectCreditCardNumbers(_ text: String) -> Bool
    private func detectSSNs(_ text: String) -> Bool
    private func detectPasswords(_ text: String) -> Bool
}
```

### Security Incident Response
```swift
class SecurityIncidentManager {
    func detectAnomalousActivity() -> [SecurityAlert]
    func handleSecurityIncident(_ incident: SecurityIncident)
    func generateSecurityReport() -> SecurityReport
    
    // Automatic safeguards
    func emergencyStop() // Halt all automation
    func enterSafeMode()  // Disable high-risk features
    func alertUser(_ alert: SecurityAlert)
}
```

---

## AI App Integrations

### Supported Applications

#### 1. ChatGPT Desktop (Primary)
**Detection Method**: Bundle identifier `com.openai.chat`
**Communication Protocol**: 
```swift
class ChatGPTAdapter: AIAppProtocol {
    private let appBundle = NSWorkspace.shared.urlForApplication(withBundleIdentifier: "com.openai.chat")
    
    func sendText(_ text: String) {
        // Focus text input area
        let textField = findElement(role: .textField, identifier: "prompt-textarea")
        textField.setValue(text)
        
        // Trigger send button
        let sendButton = findElement(role: .button, identifier: "send-button")
        sendButton.performAction(.press)
    }
    
    func uploadImage(_ image: CGImage) {
        // Save to temporary location
        let tempURL = saveImageToTemp(image)
        
        // Drag and drop file to chat interface
        performDragDrop(from: tempURL, to: chatArea)
        
        cleanup(tempURL)
    }
    
    func readLastResponse() -> String? {
        let responseElements = findElements(role: .textArea, containsText: "assistant")
        return responseElements.last?.stringValue
    }
}
```

**Reliability Features**:
- Window state detection and recovery
- Response completion polling with timeout
- Error message detection and retry logic
- Conversation context preservation

#### 2. Claude Desktop (Anthropic)
**Detection Method**: Bundle identifier `com.anthropic.claude`
**Special Considerations**:
- Image upload via file picker dialog
- Markdown response parsing
- Conversation threading model
- Rate limiting awareness

```swift
class ClaudeAdapter: AIAppProtocol {
    func uploadImage(_ image: CGImage) {
        // Click attachment button
        let attachButton = findElement(role: .button, identifier: "attach-file")
        attachButton.performAction(.press)
        
        // Handle file picker dialog
        let filePicker = NSOpenPanel()
        filePicker.allowedContentTypes = [.image]
        
        let tempURL = saveImageToTemp(image)
        filePicker.url = tempURL
        filePicker.runModal()
    }
    
    func readLastResponse() -> String? {
        // Parse markdown-formatted responses
        let responseText = getLatestResponseText()
        return parseMarkdownContent(responseText)
    }
}
```

#### 3. Gemini (Google)
**Detection Method**: Web app detection in supported browsers
**Browser Integration**:
```swift
class GeminiAdapter: AIAppProtocol {
    private let supportedBrowsers = ["com.google.Chrome", "com.apple.Safari"]
    private var targetTab: BrowserTab?
    
    func detectGeminiTab() -> BrowserTab? {
        for browser in supportedBrowsers {
            let tabs = getBrowserTabs(bundleId: browser)
            if let geminiTab = tabs.first(where: { $0.url.contains("gemini.google.com") }) {
                return geminiTab
            }
        }
        return nil
    }
    
    func sendText(_ text: String) {
        guard let tab = targetTab else { return }
        
        // JavaScript injection for web automation
        let script = """
        document.querySelector('[contenteditable="true"]').innerText = '\(text)';
        document.querySelector('button[aria-label="Send message"]').click();
        """
        
        executeJavaScript(script, in: tab)
    }
}
```

#### 4. Ollama (Local Models)
**Detection Method**: Process monitoring for `ollama serve`
**API Integration**:
```swift
class OllamaAdapter: AIAppProtocol {
    private let baseURL = "http://localhost:11434"
    
    func sendMessage(_ message: String) -> String {
        let request = OllamaRequest(
            model: detectInstalledModel(),
            prompt: message,
            stream: false
        )
        
        let response = sendHTTPRequest(to: "\(baseURL)/api/generate", body: request)
        return parseOllamaResponse(response)
    }
    
    private func detectInstalledModel() -> String {
        // Query available models
        let models = sendHTTPRequest(to: "\(baseURL)/api/tags", method: .GET)
        return models.first?.name ?? "llama3.2"
    }
}
```

#### 5. MLX Local Fallback
**Implementation**: Embedded lightweight model for offline operation
```swift
class MLXAdapter: AIAppProtocol {
    private let model: MLXLanguageModel
    private let tokenizer: MLXTokenizer
    
    init() throws {
        // Load quantized 3B parameter model
        self.model = try MLXLanguageModel.load(path: modelPath)
        self.tokenizer = try MLXTokenizer.load(path: tokenizerPath)
    }
    
    func generateResponse(_ prompt: String) -> String {
        let tokens = tokenizer.encode(prompt)
        let generated = model.generate(
            tokens: tokens,
            maxTokens: 512,
            temperature: 0.7
        )
        return tokenizer.decode(generated)
    }
}
```

### Communication Protocol Architecture

```swift
protocol AIAppProtocol {
    // App lifecycle
    func isRunning() -> Bool
    func launch() throws
    func bringToForeground()
    func getWindowBounds() -> CGRect
    
    // Communication
    func sendText(_ text: String) throws
    func sendImage(_ image: CGImage) throws
    func sendFiles(_ urls: [URL]) throws
    
    // Response handling
    func readLastResponse() -> String?
    func waitForResponse(timeout: TimeInterval) -> String?
    func isResponseComplete() -> Bool
    
    // Conversation management
    func startNewConversation()
    func clearConversation()
    func getConversationHistory() -> [Message]
    
    // Error handling
    func getLastError() -> AIAppError?
    func recoverFromError() throws
}
```

### Reliability and Error Handling

```swift
class AIAppManager {
    private var adapters: [AIApp: AIAppProtocol] = [:]
    private var primaryApp: AIApp?
    private var fallbackApp: AIApp?
    
    func sendMessage(_ message: String) -> AnyPublisher<String, AIError> {
        return Future { promise in
            // Try primary app first
            if let primary = self.primaryApp,
               let adapter = self.adapters[primary],
               adapter.isRunning() {
                
                do {
                    try adapter.sendText(message)
                    let response = adapter.waitForResponse(timeout: 30.0)
                    promise(.success(response ?? "No response"))
                } catch {
                    // Fallback to next available app
                    self.tryFallbackApp(message: message, promise: promise)
                }
            } else {
                self.tryFallbackApp(message: message, promise: promise)
            }
        }
        .eraseToAnyPublisher()
    }
    
    private func tryFallbackApp(message: String, promise: @escaping (Result<String, AIError>) -> Void) {
        // Attempt fallback apps in priority order
        let fallbackOrder: [AIApp] = [.claude, .gemini, .ollama, .mlx]
        
        for app in fallbackOrder {
            if let adapter = adapters[app], adapter.isRunning() {
                do {
                    try adapter.sendText(message)
                    let response = adapter.waitForResponse(timeout: 30.0)
                    promise(.success(response ?? "Fallback response"))
                    return
                } catch {
                    continue
                }
            }
        }
        
        promise(.failure(.noAvailableApps))
    }
}
```

---

## Local Storage Schema

### SQLite Database Design

```sql
-- Core Data Schema
CREATE TABLE tasks (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT,
    status TEXT NOT NULL CHECK (status IN ('pending', 'running', 'completed', 'failed', 'cancelled')),
    priority INTEGER DEFAULT 5,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    user_input TEXT,
    ai_app_used TEXT,
    execution_time_ms INTEGER,
    error_message TEXT,
    metadata JSON
);

CREATE TABLE task_steps (
    id TEXT PRIMARY KEY,
    task_id TEXT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    step_number INTEGER NOT NULL,
    action_type TEXT NOT NULL,
    action_data JSON,
    screenshot_path TEXT,
    ai_reasoning TEXT,
    execution_result JSON,
    execution_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(task_id, step_number)
);

CREATE TABLE conversations (
    id TEXT PRIMARY KEY,
    task_id TEXT REFERENCES tasks(id) ON DELETE CASCADE,
    ai_app TEXT NOT NULL,
    message_type TEXT NOT NULL CHECK (message_type IN ('user', 'assistant', 'system')),
    content TEXT NOT NULL,
    image_path TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    metadata JSON
);

CREATE TABLE screenshots (
    id TEXT PRIMARY KEY,
    task_id TEXT REFERENCES tasks(id) ON DELETE CASCADE,
    step_id TEXT REFERENCES task_steps(id) ON DELETE CASCADE,
    file_path TEXT NOT NULL,
    file_size INTEGER,
    resolution TEXT,
    capture_type TEXT CHECK (capture_type IN ('full_screen', 'window', 'region')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_preferences (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,
    data_type TEXT NOT NULL CHECK (data_type IN ('string', 'integer', 'boolean', 'json')),
    category TEXT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE workflow_templates (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    category TEXT,
    steps JSON NOT NULL,
    variables JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    usage_count INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE plugins (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    bundle_path TEXT NOT NULL,
    enabled BOOLEAN DEFAULT FALSE,
    configuration JSON,
    installed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_used TIMESTAMP
);

-- Performance Indexes
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_created_at ON tasks(created_at);
CREATE INDEX idx_task_steps_task_id ON task_steps(task_id);
CREATE INDEX idx_conversations_task_id ON conversations(task_id);
CREATE INDEX idx_screenshots_task_id ON screenshots(task_id);
CREATE INDEX idx_user_preferences_category ON user_preferences(category);
CREATE INDEX idx_workflow_templates_category ON workflow_templates(category);

-- Full-text search
CREATE VIRTUAL TABLE conversations_fts USING fts5(
    content,
    content='conversations',
    content_rowid='id'
);

-- Triggers for FTS synchronization
CREATE TRIGGER conversations_ai AFTER INSERT ON conversations BEGIN
    INSERT INTO conversations_fts(rowid, content) VALUES (new.id, new.content);
END;

CREATE TRIGGER conversations_ad AFTER DELETE ON conversations BEGIN
    INSERT INTO conversations_fts(conversations_fts, rowid, content) VALUES('delete', old.id, old.content);
END;

CREATE TRIGGER conversations_au AFTER UPDATE ON conversations BEGIN
    INSERT INTO conversations_fts(conversations_fts, rowid, content) VALUES('delete', old.id, old.content);
    INSERT INTO conversations_fts(rowid, content) VALUES (new.id, new.content);
END;
```

### Core Data Model (Swift)

```swift
// Task Entity
@objc(Task)
public class Task: NSManagedObject {
    @NSManaged public var id: UUID
    @NSManaged public var title: String
    @NSManaged public var taskDescription: String?
    @NSManaged public var status: String
    @NSManaged public var priority: Int16
    @NSManaged public var createdAt: Date
    @NSManaged public var updatedAt: Date
    @NSManaged public var completedAt: Date?
    @NSManaged public var userInput: String?
    @NSManaged public var aiAppUsed: String?
    @NSManaged public var executionTimeMs: Int32
    @NSManaged public var errorMessage: String?
    @NSManaged public var metadata: Data?
    
    // Relationships
    @NSManaged public var steps: NSSet?
    @NSManaged public var conversations: NSSet?
    @NSManaged public var screenshots: NSSet?
}

// TaskStep Entity
@objc(TaskStep)
public class TaskStep: NSManagedObject {
    @NSManaged public var id: UUID
    @NSManaged public var stepNumber: Int16
    @NSManaged public var actionType: String
    @NSManaged public var actionData: Data?
    @NSManaged public var screenshotPath: String?
    @NSManaged public var aiReasoning: String?
    @NSManaged public var executionResult: Data?
    @NSManaged public var executionTimeMs: Int32
    @NSManaged public var createdAt: Date
    
    // Relationships
    @NSManaged public var task: Task
}

// Conversation Entity
@objc(Conversation)
public class Conversation: NSManagedObject {
    @NSManaged public var id: UUID
    @NSManaged public var aiApp: String
    @NSManaged public var messageType: String
    @NSManaged public var content: String
    @NSManaged public var imagePath: String?
    @NSManaged public var createdAt: Date
    @NSManaged public var metadata: Data?
    
    // Relationships
    @NSManaged public var task: Task?
}
```

### Data Access Layer

```swift
class DataManager {
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "PilotDataModel")
        container.loadPersistentStores { _, error in
            if let error = error as NSError? {
                fatalError("Core Data error: \(error), \(error.userInfo)")
            }
        }
        return container
    }()
    
    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
    
    func saveContext() {
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                let nsError = error as NSError
                fatalError("Unresolved error \(nsError), \(nsError.userInfo)")
            }
        }
    }
    
    // Task Management
    func createTask(title: String, description: String?, userInput: String?) -> Task {
        let task = Task(context: context)
        task.id = UUID()
        task.title = title
        task.taskDescription = description
        task.status = "pending"
        task.priority = 5
        task.createdAt = Date()
        task.updatedAt = Date()
        task.userInput = userInput
        
        saveContext()
        return task
    }
    
    func fetchActiveTasks() -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.predicate = NSPredicate(format: "status IN %@", ["pending", "running"])
        request.sortDescriptors = [NSSortDescriptor(keyPath: \Task.createdAt, ascending: false)]
        
        do {
            return try context.fetch(request)
        } catch {
            print("Error fetching active tasks: \(error)")
            return []
        }
    }
    
    func fetchTaskHistory(limit: Int = 100) -> [Task] {
        let request: NSFetchRequest<Task> = Task.fetchRequest()
        request.fetchLimit = limit
        request.sortDescriptors = [NSSortDescriptor(keyPath: \Task.updatedAt, ascending: false)]
        
        do {
            return try context.fetch(request)
        } catch {
            print("Error fetching task history: \(error)")
            return []
        }
    }
    
    // Screenshot Management
    func saveScreenshot(for task: Task, imagePath: String, captureType: String) -> Screenshot {
        let screenshot = Screenshot(context: context)
        screenshot.id = UUID()
        screenshot.task = task
        screenshot.filePath = imagePath
        screenshot.captureType = captureType
        screenshot.createdAt = Date()
        
        saveContext()
        return screenshot
    }
    
    // Full-text Search
    func searchConversations(query: String) -> [Conversation] {
        let request: NSFetchRequest<Conversation> = Conversation.fetchRequest()
        request.predicate = NSPredicate(format: "content CONTAINS[cd] %@", query)
        request.sortDescriptors = [NSSortDescriptor(keyPath: \Conversation.createdAt, ascending: false)]
        
        do {
            return try context.fetch(request)
        } catch {
            print("Error searching conversations: \(error)")
            return []
        }
    }
    
    // Data Cleanup
    func deleteOldData(olderThan days: Int) {
        let cutoffDate = Calendar.current.date(byAdding: .day, value: -days, to: Date())!
        
        let taskRequest: NSFetchRequest<Task> = Task.fetchRequest()
        taskRequest.predicate = NSPredicate(format: "updatedAt < %@ AND status IN %@", 
                                          cutoffDate as NSDate, 
                                          ["completed", "failed", "cancelled"])
        
        do {
            let oldTasks = try context.fetch(taskRequest)
            oldTasks.forEach { context.delete($0) }
            saveContext()
        } catch {
            print("Error deleting old data: \(error)")
        }
    }
}
```

---

## Plugin & Workflow Extension Model

### Plugin Architecture

```swift
protocol PilotPlugin {
    var identifier: String { get }
    var name: String { get }
    var version: String { get }
    var description: String { get }
    var supportedActionTypes: [String] { get }
    
    func initialize(context: PluginContext) throws
    func executeAction(type: String, parameters: [String: Any]) throws -> ActionResult
    func cleanup()
}

struct PluginContext {
    let dataManager: DataManager
    let screenCapture: ScreenCaptureManager
    let virtualInput: VirtualInputManager
    let preferences: UserPreferences
}

struct ActionResult {
    let success: Bool
    let output: String?
    let artifacts: [URL]?
    let nextActions: [String]?
}
```

### Built-in Workflow Templates

```swift
enum WorkflowTemplate {
    case emailManagement
    case documentProcessing
    case dataEntry
    case webResearch
    case systemMaintenance
    
    var steps: [WorkflowStep] {
        switch self {
        case .emailManagement:
            return [
                WorkflowStep(action: "screenshot", target: "email_app"),
                WorkflowStep(action: "analyze_inbox", aiPrompt: "Count unread emails and identify urgent items"),
                WorkflowStep(action: "draft_replies", aiPrompt: "Draft appropriate replies for urgent emails"),
                WorkflowStep(action: "confirm_actions", requiresApproval: true)
            ]
        case .documentProcessing:
            return [
                WorkflowStep(action: "scan_folder", target: "~/Downloads"),
                WorkflowStep(action: "categorize_files", aiPrompt: "Categorize documents by type and importance"),
                WorkflowStep(action: "move_files", target: "organized_folders"),
                WorkflowStep(action: "generate_summary", output: "processing_report.txt")
            ]
        // Additional templates...
        }
    }
}

struct WorkflowStep {
    let action: String
    let target: String?
    let aiPrompt: String?
    let parameters: [String: Any]?
    let requiresApproval: Bool
    let timeout: TimeInterval
    let onFailure: FailureStrategy
    let output: String?
    
    init(action: String, 
         target: String? = nil,
         aiPrompt: String? = nil,
         parameters: [String: Any]? = nil,
         requiresApproval: Bool = false,
         timeout: TimeInterval = 30.0,
         onFailure: FailureStrategy = .retry,
         output: String? = nil) {
        self.action = action
        self.target = target
        self.aiPrompt = aiPrompt
        self.parameters = parameters
        self.requiresApproval = requiresApproval
        self.timeout = timeout
        self.onFailure = onFailure
        self.output = output
    }
}

enum FailureStrategy {
    case retry
    case skip
    case abort
    case fallback(String)
}
```

### Custom Workflow Builder

```swift
class WorkflowBuilder {
    private var steps: [WorkflowStep] = []
    
    func addStep(_ step: WorkflowStep) -> WorkflowBuilder {
        steps.append(step)
        return self
    }
    
    func screenshot(of target: String) -> WorkflowBuilder {
        return addStep(WorkflowStep(action: "screenshot", target: target))
    }
    
    func analyze(with prompt: String) -> WorkflowBuilder {
        return addStep(WorkflowStep(action: "analyze", aiPrompt: prompt))
    }
    
    func click(at target: String) -> WorkflowBuilder {
        return addStep(WorkflowStep(action: "click", target: target))
    }
    
    func type(_ text: String) -> WorkflowBuilder {
        return addStep(WorkflowStep(action: "type", parameters: ["text": text]))
    }
    
    func waitForApproval() -> WorkflowBuilder {
        return addStep(WorkflowStep(action: "approval_gate", requiresApproval: true))
    }
    
    func build() -> Workflow {
        return Workflow(steps: steps)
    }
}

// Usage example:
let customWorkflow = WorkflowBuilder()
    .screenshot(of: "mail_app")
    .analyze(with: "Find the most recent email from my boss")
    .click(at: "reply_button")
    .type("I'll get back to you by end of day.")
    .waitForApproval()
    .click(at: "send_button")
    .build()
```

### Plugin Development Kit

```swift
// SDK for third-party plugin development
public class PilotSDK {
    public static let shared = PilotSDK()
    
    private init() {}
    
    // Screen interaction
    public func captureScreen(region: CGRect? = nil) -> CGImage? {
        return ScreenCaptureManager.shared.captureRegion(region ?? CGRect.infinite)
    }
    
    public func click(at point: CGPoint) {
        VirtualInputManager.shared.click(at: point)
    }
    
    public func type(text: String) {
        VirtualInputManager.shared.type(text: text)
    }
    
    // AI interaction
    public func askAI(question: String, context: String? = nil) -> String? {
        let prompt = context.map { "\($0)\n\n\(question)" } ?? question
        return AIAppBridgeManager.shared.sendMessage(prompt)
    }
    
    // Data persistence
    public func storeData<T: Codable>(_ data: T, key: String) {
        UserDefaults.standard.setCodable(data, forKey: key)
    }
    
    public func loadData<T: Codable>(key: String, type: T.Type) -> T? {
        return UserDefaults.standard.getCodable(T.self, forKey: key)
    }
    
    // Utilities
    public func showNotification(title: String, message: String) {
        NotificationCenter.default.post(name: .pilotNotification, object: PilotNotification(title: title, message: message))
    }
    
    public func logMessage(_ message: String, level: LogLevel = .info) {
        Logger.shared.log(message, level: level)
    }
}

// Plugin manifest structure
public struct PluginManifest: Codable {
    let identifier: String
    let name: String
    let version: String
    let description: String
    let author: String
    let minimumPilotVersion: String
    let supportedActionTypes: [String]
    let permissions: [Permission]
    let dependencies: [String]
    
    enum Permission: String, Codable {
        case screenCapture
        case virtualInput
        case fileSystem
        case networkAccess
        case notifications
    }
}
```

### Extension Distribution

```swift
class PluginManager {
    private var loadedPlugins: [String: PilotPlugin] = [:]
    private let pluginDirectory: URL
    
    init() {
        pluginDirectory = FileManager.default.urls(for: .applicationSupportDirectory, 
                                                 in: .userDomainMask)[0]
            .appendingPathComponent("Pilot/Plugins")
    }
    
    func loadPlugins() {
        guard FileManager.default.fileExists(atPath: pluginDirectory.path) else { return }
        
        do {
            let pluginPaths = try FileManager.default.contentsOfDirectory(at: pluginDirectory, 
                                                                        includingPropertiesForKeys: nil)
            
            for path in pluginPaths.filter({ $0.pathExtension == "pilotplugin" }) {
                loadPlugin(at: path)
            }
        } catch {
            print("Error loading plugins: \(error)")
        }
    }
    
    private func loadPlugin(at url: URL) {
        // Load plugin bundle and validate manifest
        guard let bundle = Bundle(url: url),
              let manifestData = bundle.url(forResource: "manifest", withExtension: "json").flatMap({ try? Data(contentsOf: $0) }),
              let manifest = try? JSONDecoder().decode(PluginManifest.self, from: manifestData) else {
            print("Invalid plugin at \(url)")
            return
        }
        
        // Validate permissions and dependencies
        guard validatePluginPermissions(manifest.permissions),
              validatePluginDependencies(manifest.dependencies) else {
            print("Plugin \(manifest.name) failed validation")
            return
        }
        
        // Load and initialize plugin
        if let pluginClass = bundle.principalClass as? PilotPlugin.Type {
            let plugin = pluginClass.init()
            try? plugin.initialize(context: PluginContext(
                dataManager: DataManager.shared,
                screenCapture: ScreenCaptureManager.shared,
                virtualInput: VirtualInputManager.shared,
                preferences: UserPreferences.shared
            ))
            
            loadedPlugins[manifest.identifier] = plugin
            print("Loaded plugin: \(manifest.name) v\(manifest.version)")
        }
    }
    
    func executePluginAction(identifier: String, action: String, parameters: [String: Any]) -> ActionResult? {
        return loadedPlugins[identifier]?.executeAction(type: action, parameters: parameters)
    }
    
    private func validatePluginPermissions(_ permissions: [PluginManifest.Permission]) -> Bool {
        // Check if user has granted required system permissions
        for permission in permissions {
            switch permission {
            case .screenCapture:
                guard CGPreflightScreenCaptureAccess() else { return false }
            case .virtualInput:
                guard AXIsProcessTrusted() else { return false }
            // Additional permission checks...
            }
        }
        return true
    }
    
    private func validatePluginDependencies(_ dependencies: [String]) -> Bool {
        // Verify all required dependencies are available
        return dependencies.allSatisfy { loadedPlugins.keys.contains($0) }
    }
}
```

---

## Technical Risks & Mitigations

### Risk Category 1: System Stability

#### Risk: Application Crashes During Automation
**Probability**: Medium | **Impact**: High
**Description**: Synthetic input events or screen capture operations could crash target applications or the system.

**Mitigations**:
```swift
class StabilityMonitor {
    private let crashDetector = CrashDetector()
    private let recoveryManager = RecoveryManager()
    
    func monitorTargetApplication(_ bundleId: String) {
        crashDetector.watchApplication(bundleId) { crash in
            self.recoveryManager.handleApplicationCrash(bundleId, crash)
        }
    }
    
    func executeWithSafeguards<T>(_ operation: () throws -> T) -> Result<T, Error> {
        let checkpoint = createSystemCheckpoint()
        
        do {
            let result = try operation()
            return .success(result)
        } catch {
            restoreFromCheckpoint(checkpoint)
            return .failure(error)
        }
    }
}

class RateLimiter {
    private var lastAction: Date = Date.distantPast
    private let minimumInterval: TimeInterval = 0.1 // 100ms between actions
    
    func throttleAction(_ action: () -> Void) {
        let elapsed = Date().timeIntervalSince(lastAction)
        if elapsed < minimumInterval {
            Thread.sleep(forTimeInterval: minimumInterval - elapsed)
        }
        action()
        lastAction = Date()
    }
}
```

#### Risk: Memory Leaks from Continuous Screenshot Capture
**Probability**: High | **Impact**: Medium
**Description**: Continuous screen capture could accumulate memory usage over time.

**Mitigations**:
- Automatic image compression and cleanup
- Memory usage monitoring with automatic throttling
- Configurable capture frequency based on system resources

```swift
class MemoryManager {
    private let maxMemoryUsage: Int = 512 * 1024 * 1024 // 512MB
    private let cleanupThreshold: Int = 400 * 1024 * 1024 // 400MB
    
    func monitorMemoryUsage() {
        Timer.scheduledTimer(withTimeInterval: 30.0, repeats: true) { _ in
            let usage = self.getCurrentMemoryUsage()
            if usage > self.cleanupThreshold {
                self.performMemoryCleanup()
            }
            if usage > self.maxMemoryUsage {
                self.triggerEmergencyCleanup()
            }
        }
    }
    
    private func performMemoryCleanup() {
        // Remove old screenshots
        // Clear inactive conversation history
        // Compress stored images
        ScreenshotManager.shared.cleanupOldScreenshots()
        DataManager.shared.compressOldData()
    }
}
```

### Risk Category 2: Security Vulnerabilities

#### Risk: Privilege Escalation via Accessibility APIs
**Probability**: Low | **Impact**: Critical
**Description**: Malicious actors could exploit accessibility permissions to gain system control.

**Mitigations**:
```swift
class SecurityValidator {
    func validateActionSafety(_ action: VirtualInputAction) -> Bool {
        // Prevent system-critical actions
        let dangerousActions = [
            "sudo", "rm -rf", "chmod 777", "killall",
            "System Preferences", "Security & Privacy"
        ]
        
        return !dangerousActions.contains { action.description.contains($0) }
    }
    
    func enforceUserApproval(for action: VirtualInputAction) -> Bool {
        if action.riskLevel >= .high {
            return requestUserConfirmation(action)
        }
        return true
    }
    
    private func requestUserConfirmation(_ action: VirtualInputAction) -> Bool {
        let alert = NSAlert()
        alert.messageText = "Confirm High-Risk Action"
        alert.informativeText = "Pilot wants to perform: \(action.description)"
        alert.addButton(withTitle: "Allow")
        alert.addButton(withTitle: "Deny")
        alert.alertStyle = .warning
        
        return alert.runModal() == .alertFirstButtonReturn
    }
}

class ActionWhitelist {
    private let allowedApplications = Set([
        "com.apple.mail",
        "com.microsoft.Outlook",
        "com.google.Chrome",
        "com.apple.Safari"
    ])
    
    func isActionAllowed(_ action: VirtualInputAction, in app: String) -> Bool {
        return allowedApplications.contains(app) && 
               !action.involvesSystemUI() &&
               !action.modifiesSecuritySettings()
    }
}
```

#### Risk: Data Exfiltration via AI App Communication
**Probability**: Medium | **Impact**: High
**Description**: Sensitive information could be inadvertently shared with AI applications.

**Mitigations**:
- Content filtering before AI communication
- User consent for each data sharing operation
- Local processing preference over AI app usage

```swift
class DataClassifier {
    private let sensitivePatterns = [
        NSRegularExpression.creditCard,
        NSRegularExpression.socialSecurity,
        NSRegularExpression.phoneNumber,
        NSRegularExpression.emailAddress
    ]
    
    func classifyContent(_ content: String) -> DataClassification {
        var classification = DataClassification.public
        
        for pattern in sensitivePatterns {
            if pattern.firstMatch(in: content, range: NSRange(location: 0, length: content.count)) != nil {
                classification = .sensitive
                break
            }
        }
        
        return classification
    }
    
    func redactSensitiveContent(_ content: String) -> String {
        var redacted = content
        
        for pattern in sensitivePatterns {
            redacted = pattern.stringByReplacingMatches(
                in: redacted,
                range: NSRange(location: 0, length: redacted.count),
                withTemplate: "[REDACTED]"
            )
        }
        
        return redacted
    }
}

enum DataClassification {
    case public
    case internal
    case confidential
    case sensitive
    
    var requiresUserConsent: Bool {
        return [.confidential, .sensitive].contains(self)
    }
}
```

### Risk Category 3: Integration Failures

#### Risk: AI Application Updates Breaking Integration
**Probability**: High | **Impact**: Medium
**Description**: UI changes in AI applications could break automation adapters.

**Mitigations**:
- Multiple detection strategies per app (accessibility + visual + position-based)
- Automated integration testing
- Graceful degradation to alternative AI apps

```swift
class AdaptiveUIDetection {
    func findElement(criteria: ElementCriteria) -> AXUIElement? {
        // Try multiple detection strategies in order of reliability
        
        // Strategy 1: Accessibility identifier
        if let element = findByAccessibilityId(criteria.identifier) {
            return element
        }
        
        // Strategy 2: Role and description
        if let element = findByRoleAndDescription(criteria.role, criteria.description) {
            return element
        }
        
        // Strategy 3: Visual pattern matching
        if let element = findByVisualPattern(criteria.visualPattern) {
            return element
        }
        
        // Strategy 4: Relative positioning
        if let element = findByRelativePosition(criteria.relativeAnchor, criteria.offset) {
            return element
        }
        
        return nil
    }
    
    func validateIntegration(for app: AIApp) -> IntegrationHealth {
        let testCases = getTestCases(for: app)
        var successCount = 0
        
        for testCase in testCases {
            if executeTestCase(testCase) {
                successCount += 1
            }
        }
        
        let successRate = Double(successCount) / Double(testCases.count)
        
        switch successRate {
        case 0.9...1.0:
            return .healthy
        case 0.7..<0.9:
            return .degraded
        default:
            return .failing
        }
    }
}

enum IntegrationHealth {
    case healthy
    case degraded
    case failing
    
    var shouldUseFallback: Bool {
        return self == .failing
    }
}
```

#### Risk: macOS System Updates Breaking Core APIs
**Probability**: Medium | **Impact**: High
**Description**: macOS updates could deprecate or modify Core Graphics or Accessibility APIs.

**Mitigations**:
- Version-specific API implementations
- Runtime capability detection
- Automatic fallback to supported API versions

```swift
class APICompatibilityManager {
    private let minimumSupportedVersion = OperatingSystemVersion(majorVersion: 14, minorVersion: 0, patchVersion: 0)
    
    func checkSystemCompatibility() -> CompatibilityReport {
        let currentVersion = ProcessInfo.processInfo.operatingSystemVersion
        
        var report = CompatibilityReport()
        
        // Core Graphics compatibility
        if #available(macOS 14.0, *) {
            report.coreGraphicsSupport = .full
            report.virtualDisplaySupport = .full
        } else {
            report.coreGraphicsSupport = .limited
            report.virtualDisplaySupport = .unavailable
        }
        
        // Accessibility API compatibility
        report.accessibilitySupport = AXIsProcessTrusted() ? .full : .requiresPermission
        
        return report
    }
    
    func getAPIImplementation<T>(for capability: SystemCapability) -> T? {
        switch capability {
        case .virtualDisplay:
            if #available(macOS 14.0, *) {
                return CGVirtualDisplayImplementation() as? T
            } else {
                return LegacyDisplayImplementation() as? T
            }
        case .screenCapture:
            return CoreGraphicsImplementation() as? T
        }
    }
}

struct CompatibilityReport {
    var coreGraphicsSupport: SupportLevel = .unknown
    var virtualDisplaySupport: SupportLevel = .unknown
    var accessibilitySupport: SupportLevel = .unknown
    var mlxSupport: SupportLevel = .unknown
    
    var overallCompatibility: SupportLevel {
        let levels = [coreGraphicsSupport, virtualDisplaySupport, accessibilitySupport, mlxSupport]
        return levels.min() ?? .unknown
    }
}

enum SupportLevel: Comparable {
    case full
    case limited
    case requiresPermission
    case unavailable
    case unknown
}
```

### Risk Category 4: Performance Issues

#### Risk: AI Model Inference Bottlenecks
**Probability**: Medium | **Impact**: Medium
**Description**: Local MLX model inference could be too slow for real-time automation.

**Mitigations**:
```swift
class InferenceOptimizer {
    private let modelCache = ModelCache()
    private let requestQueue = DispatchQueue(label: "inference", qos: .userInitiated)
    
    func optimizeInference(for model: MLXModel) {
        // Model quantization
        let quantizedModel = model.quantized(bits: 4)
        modelCache.store(quantizedModel)
        
        // Prompt caching
        let commonPrompts = getCommonPrompts()
        precomputePromptEmbeddings(commonPrompts)
        
        // Batch processing
        enableBatchInference(maxBatchSize: 3)
    }
    
    func scheduleInference(prompt: String, priority: TaskPriority) -> Future<String, InferenceError> {
        return Future { promise in
            self.requestQueue.async {
                let start = Date()
                let result = self.modelCache.getCachedModel().generate(prompt)
                let duration = Date().timeIntervalSince(start)
                
                // Track performance metrics
                PerformanceMetrics.shared.recordInference(duration: duration, tokens: result.tokenCount)
                
                promise(.success(result.text))
            }
        }
    }
}

class PerformanceMetrics {
    static let shared = PerformanceMetrics()
    
    private var inferenceMetrics: [InferenceMetric] = []
    
    func recordInference(duration: TimeInterval, tokens: Int) {
        let metric = InferenceMetric(
            duration: duration,
            tokens: tokens,
            tokensPerSecond: Double(tokens) / duration,
            timestamp: Date()
        )
        
        inferenceMetrics.append(metric)
        
        // Alert if performance degrades
        if metric.tokensPerSecond < 10 { // Below 10 tokens/second
            NotificationCenter.default.post(name: .performanceAlert, 
                                          object: "Inference performance degraded")
        }
    }
    
    func getAveragePerformance(lastN: Int = 10) -> Double {
        let recent = inferenceMetrics.suffix(lastN)
        return recent.map { $0.tokensPerSecond }.reduce(0, +) / Double(recent.count)
    }
}

struct InferenceMetric {
    let duration: TimeInterval
    let tokens: Int
    let tokensPerSecond: Double
    let timestamp: Date
}
```

### Risk Category 5: User Experience Issues

#### Risk: Automation Interfering with Normal Computer Use
**Probability**: High | **Impact**: Medium
**Description**: Background automation could conflict with user's manual computer usage.

**Mitigations**:
- Virtual display isolation for AI app interactions
- User presence detection with automatic pause
- Manual override controls always accessible

```swift
class UserPresenceDetector {
    private var isUserActive = false
    private let activityMonitor = CGEventSource(stateID: .combinedSessionState)
    
    func startMonitoring() {
        // Monitor for user input events
        let eventMask = CGEventMask(1 << CGEventType.mouseMoved.rawValue | 1 << CGEventType.keyDown.rawValue)
        
        guard let eventTap = CGEvent.tapCreate(
            tap: .cgSessionEventTap,
            place: .headInsertEventTap,
            options: .defaultTap,
            eventsOfInterest: eventMask,
            callback: { _, _, event, _ in
                UserPresenceDetector.shared.detectUserActivity()
                return Unmanaged.passRetained(event)
            },
            userInfo: nil
        ) else {
            print("Failed to create event tap for user presence detection")
            return
        }
        
        CFRunLoopAddSource(CFRunLoopGetCurrent(), eventTap, .commonModes)
        CGEvent.tapEnable(tap: eventTap, enable: true)
    }
    
    private func detectUserActivity() {
        isUserActive = true
        
        // Pause automation if user becomes active
        TaskOrchestrator.shared.pauseIfUserActive()
        
        // Reset after inactivity period
        DispatchQueue.main.asyncAfter(deadline: .now() + 30) {
            self.isUserActive = false
        }
    }
}

class ConflictResolver {
    func resolveInputConflict() -> ConflictResolution {
        if UserPresenceDetector.shared.isUserActive {
            return .pauseAutomation
        } else {
            return .continueAutomation
        }
    }
    
    func handleWindowFocusConflict(automationWindow: NSWindow, userWindow: NSWindow) {
        // Prefer user's active window
        userWindow.makeKeyAndOrderFront(nil)
        
        // Delay automation until user window becomes inactive
        WindowActivityMonitor.shared.waitForWindowInactive(userWindow) {
            automationWindow.makeKeyAndOrderFront(nil)
            TaskOrchestrator.shared.resumeAutomation()
        }
    }
}

enum ConflictResolution {
    case pauseAutomation
    case continueAutomation
    case requestUserDecision
}
```

---

## Conclusion

This technical architecture provides a comprehensive foundation for Pilot, a Mac-native RPA agent that leverages existing AI applications for intelligent automation. The design prioritizes local operation, user privacy, system stability, and extensibility while managing the inherent complexity of desktop automation.

### Key Architectural Strengths

1. **Zero Cloud Dependencies**: Complete local operation ensures data privacy and eliminates external service dependencies
2. **AI App Agnostic Design**: Multi-modal communication strategy provides flexibility and resilience
3. **Virtual Display Isolation**: Non-intrusive operation prevents user workflow interference
4. **Extensible Plugin Architecture**: Supports custom workflows and third-party extensions
5. **Comprehensive Error Handling**: Multiple fallback strategies and recovery mechanisms
6. **Performance Optimization**: Hardware acceleration and intelligent resource management

### Implementation Priorities

**Phase 1: Core Infrastructure** (Weeks 1-4)
- Screen capture and virtual input systems
- Basic AI app integration (ChatGPT + Claude)
- SQLite database and Core Data models
- Security framework and permissions management

**Phase 2: Task Orchestration** (Weeks 5-8)
- Task execution engine and workflow processing
- Multi-AI app fallback mechanisms
- Memory store and conversation management
- Basic chat UI implementation

**Phase 3: Advanced Features** (Weeks 9-12)
- Plugin architecture and SDK
- Workflow templates and custom builders
- Performance monitoring and optimization
- Comprehensive testing and error handling

**Phase 4: Production Readiness** (Weeks 13-16)
- Security hardening and penetration testing
- User experience polish and accessibility
- Documentation and deployment automation
- Beta testing and feedback integration

This architecture document serves as the definitive technical specification for Pilot development, providing detailed implementation guidance while maintaining flexibility for iterative improvements based on real-world usage and feedback.

---

*Document Version: 1.0*  
*Last Updated: February 17, 2026*  
*Review Date: March 17, 2026*