# OpenClaw Integration Specification for OpenPaw

**Version:** 1.0  
**Date:** February 17, 2026  
**Target:** OpenPaw Mac Application v1.0  

## Overview

This document specifies how OpenPaw (Swift-based Mac application) embeds and integrates with OpenClaw (Node.js-based AI agent platform). OpenPaw provides a native macOS interface while OpenClaw handles the core AI agent functionality.

## Node.js Runtime Bundling

### Bundling Strategy

OpenPaw uses **pkg** (vercel/pkg) to create a standalone Node.js executable that includes:
- Node.js runtime (specific version: 22.x LTS)
- OpenClaw core modules
- All npm dependencies
- Bundled skills and tools

### Build Process

```bash
# Production build command
pkg openclaw-bundle.js --target node22-macos-arm64 --output OpenClaw-Darwin-arm64
```

### Bundle Configuration (pkg.json)

```json
{
  "name": "openclaw-embedded",
  "version": "1.0.0",
  "bin": "src/openclaw-main.js",
  "pkg": {
    "targets": ["node22-macos-arm64"],
    "outputPath": "dist/",
    "assets": [
      "skills/**/*",
      "templates/**/*",
      "config/defaults.json"
    ],
    "scripts": [
      "src/**/*.js",
      "node_modules/@anthropic-ai/**/*.js"
    ]
  }
}
```

### Alternative: Bundled Runtime Approach

If pkg proves inadequate, use bundled Node.js runtime:
- Ship Node.js 22.x binary inside app bundle
- OpenClaw source code as uncompiled JavaScript
- Use `child_process.spawn()` to launch Node.js with OpenClaw entry point

## App Bundle Directory Structure

```
OpenPaw.app/
├── Contents/
│   ├── Info.plist
│   ├── MacOS/
│   │   ├── OpenPaw                    # Swift executable
│   │   └── OpenClaw-Darwin-arm64      # Bundled OpenClaw binary
│   ├── Resources/
│   │   ├── AppIcon.icns
│   │   ├── openclaw-config/           # OpenClaw configuration templates
│   │   │   ├── defaults.json          # Default OpenClaw settings
│   │   │   ├── agent-templates/       # Template files for agents
│   │   │   │   ├── SOUL.template.md
│   │   │   │   ├── USER.template.md
│   │   │   │   └── AGENTS.template.md
│   │   │   └── skill-registry.json    # Available skills catalog
│   │   ├── skills/                    # Bundled skills
│   │   │   ├── core/                  # Core OpenClaw skills
│   │   │   └── optional/              # Optional skills
│   │   └── node_runtime/              # (Alternative) Node.js runtime
│   │       ├── bin/node
│   │       └── openclaw-src/          # OpenClaw source files
│   └── Frameworks/                    # Swift frameworks
```

## OpenClaw Workspace Directory

User-specific OpenClaw workspace (separate from app bundle):

```
~/Library/Application Support/OpenPaw/
├── workspace/                         # OpenClaw workspace root
│   ├── openclaw.json                 # Generated configuration
│   ├── SOUL.md                       # Generated agent personality
│   ├── USER.md                       # Generated user profile
│   ├── AGENTS.md                     # Generated agent instructions
│   ├── MEMORY.md                     # Agent memory file
│   ├── skills/                       # User-installed skills
│   ├── memory/                       # Daily memory files
│   ├── config/                       # Runtime configuration
│   └── logs/                         # OpenClaw logs
├── cache/                            # Temporary files
└── user-settings.json                # OpenPaw-specific settings
```

## Configuration Auto-Generation

### openclaw.json Generation

Generated during onboarding wizard completion:

```javascript
// Swift → OpenClaw config generation
class ConfigGenerator {
  static generateOpenClawConfig(onboardingData) {
    return {
      "agent": {
        "name": onboardingData.agentName,
        "model": onboardingData.selectedModel,
        "personality": onboardingData.personalityProfile,
        "capabilities": onboardingData.enabledCapabilities
      },
      "gateway": {
        "port": 3001,
        "host": "127.0.0.1",
        "websocket": {
          "enabled": true,
          "port": 3002
        }
      },
      "integrations": {
        "oauth": onboardingData.oauthTokens,
        "apis": onboardingData.apiConfigs,
        "tools": onboardingData.enabledTools
      },
      "workspace": {
        "path": "~/Library/Application Support/OpenPaw/workspace",
        "autoBackup": true,
        "maxMemoryDays": 90
      },
      "security": {
        "allowedHosts": ["127.0.0.1"],
        "toolPolicy": "restricted",
        "requireApproval": onboardingData.requireApproval
      }
    }
  }
}
```

### Template-Based File Generation

#### SOUL.md Generation

```swift
func generateSoulMD(from profile: PersonalityProfile) -> String {
  let template = loadTemplate("SOUL.template.md")
  return template
    .replacingOccurrences(of: "{{AGENT_NAME}}", with: profile.name)
    .replacingOccurrences(of: "{{PERSONALITY}}", with: profile.traits.joined(separator: ", "))
    .replacingOccurrences(of: "{{COMMUNICATION_STYLE}}", with: profile.communicationStyle)
    .replacingOccurrences(of: "{{EXPERTISE}}", with: profile.expertise.joined(separator: ", "))
}
```

#### USER.md Generation

```swift
func generateUserMD(from userProfile: UserProfile) -> String {
  return """
# USER.md - About Your Human

## Basic Info
- **Name:** \(userProfile.name)
- **Role:** \(userProfile.role)
- **Location:** \(userProfile.location)
- **Timezone:** \(userProfile.timezone)

## Preferences
- **Work hours:** \(userProfile.workHours)
- **Communication style:** \(userProfile.communicationPreference)
- **Priorities:** \(userProfile.priorities.joined(separator: ", "))

## Context
\(userProfile.backgroundContext)

## Goals
\(userProfile.goals.map { "- \($0)" }.joined(separator: "\n"))
"""
}
```

## Skills Management

### Skill Installation Flow

1. **Discovery:** OpenPaw queries skill registry API
2. **Installation:** Swift downloads and validates skill packages
3. **Integration:** Skills installed to workspace/skills/ directory
4. **Registration:** OpenClaw config updated with new skills
5. **Activation:** OpenClaw gateway restarted to load new skills

### Skill Package Format

```json
{
  "name": "github-integration",
  "version": "1.2.0",
  "description": "GitHub API integration",
  "author": "OpenClaw Community",
  "license": "MIT",
  "openclaw": {
    "minVersion": "1.0.0",
    "maxVersion": "2.0.0"
  },
  "dependencies": {
    "octokit": "^3.0.0"
  },
  "files": {
    "main": "skill.js",
    "config": "config.json",
    "docs": "README.md"
  },
  "permissions": [
    "network.external",
    "file.read",
    "config.oauth"
  ]
}
```

### Skill Registry Integration

```swift
class SkillManager {
  private let registryURL = "https://registry.openclaw.com/skills"
  
  func installSkill(skillID: String) async throws {
    // 1. Download skill package
    let package = try await downloadSkill(skillID)
    
    // 2. Validate signature and permissions
    try validateSkill(package)
    
    // 3. Install to workspace
    try installSkillFiles(package)
    
    // 4. Update OpenClaw config
    try updateOpenClawConfig(with: package)
    
    // 5. Restart gateway to load skill
    try await restartOpenClawGateway()
  }
}
```

## OpenClaw Gateway Lifecycle Management

### Startup Sequence

```swift
class OpenClawManager: ObservableObject {
  private var gatewayProcess: Process?
  private var websocketClient: WebSocketClient?
  
  func startOpenClaw() async throws {
    // 1. Validate workspace
    try validateWorkspace()
    
    // 2. Generate/update configuration
    try generateConfiguration()
    
    // 3. Launch OpenClaw process
    gatewayProcess = try launchGatewayProcess()
    
    // 4. Wait for gateway ready
    try await waitForGatewayReady()
    
    // 5. Establish WebSocket connection
    websocketClient = try await connectWebSocket()
  }
  
  private func launchGatewayProcess() throws -> Process {
    let process = Process()
    let bundle = Bundle.main
    let openclawBinary = bundle.path(forResource: "OpenClaw-Darwin-arm64", ofType: nil)!
    
    process.executableURL = URL(fileURLWithPath: openclawBinary)
    process.arguments = [
      "gateway", "start",
      "--config", workspaceConfigPath,
      "--port", "3001",
      "--websocket-port", "3002",
      "--log-level", "info"
    ]
    
    // Set environment
    process.environment = [
      "OPENCLAW_WORKSPACE": workspacePath,
      "OPENCLAW_MODE": "embedded",
      "NODE_ENV": "production"
    ]
    
    // Capture output
    let outputPipe = Pipe()
    process.standardOutput = outputPipe
    process.standardError = outputPipe
    
    try process.run()
    
    // Monitor output
    monitorProcessOutput(outputPipe)
    
    return process
  }
}
```

### Shutdown Sequence

```swift
func stopOpenClaw() async {
  // 1. Close WebSocket connection gracefully
  await websocketClient?.disconnect()
  
  // 2. Send SIGTERM to gateway process
  gatewayProcess?.terminate()
  
  // 3. Wait for graceful shutdown (5 seconds)
  let shutdownSuccess = await withTimeout(5.0) {
    return gatewayProcess?.isRunning == false
  }
  
  // 4. Force kill if necessary
  if !shutdownSuccess {
    gatewayProcess?.kill()
  }
  
  gatewayProcess = nil
}
```

## WebSocket Communication Protocol

### Connection Management

```swift
class WebSocketClient: ObservableObject {
  private let url = URL(string: "ws://127.0.0.1:3002")!
  private var webSocket: URLSessionWebSocketTask?
  
  func connect() async throws {
    webSocket = URLSession.shared.webSocketTask(with: url)
    webSocket?.resume()
    
    // Send authentication
    try await authenticate()
    
    // Start message listening loop
    Task { await listenForMessages() }
  }
  
  private func authenticate() async throws {
    let authMessage = WebSocketMessage.authenticate(
      clientType: "openpaw",
      version: "1.0.0",
      sessionId: UUID().uuidString
    )
    try await send(authMessage)
  }
}
```

### Message Protocol

All messages follow this envelope format:

```json
{
  "id": "unique-message-id",
  "type": "message-type",
  "timestamp": "2026-02-17T17:47:00Z",
  "payload": { ... }
}
```

## OAuth Token Flow

### Token Storage and Management

```swift
class OAuthManager {
  private let keychain = Keychain(service: "com.openpaw.oauth")
  
  func storeOAuthToken(_ token: OAuthToken, for service: String) throws {
    let tokenData = try JSONEncoder().encode(token)
    try keychain.set(tokenData, key: "oauth_\(service)")
    
    // Update OpenClaw config
    try updateOpenClawOAuthConfig(service: service, token: token)
  }
  
  private func updateOpenClawOAuthConfig(service: String, token: OAuthToken) throws {
    var config = try loadOpenClawConfig()
    config.integrations.oauth[service] = [
      "access_token": token.accessToken,
      "refresh_token": token.refreshToken,
      "expires_at": token.expiresAt,
      "scopes": token.scopes
    ]
    try saveOpenClawConfig(config)
    
    // Notify OpenClaw of config change
    notifyConfigUpdate()
  }
}
```

### Token Refresh Flow

```swift
func refreshTokenIfNeeded(for service: String) async throws {
  guard let token = try getOAuthToken(for: service),
        token.isExpiring(within: TimeInterval(300)) else { return }
  
  let refreshedToken = try await refreshOAuthToken(token, for: service)
  try storeOAuthToken(refreshedToken, for: service)
}
```

## Error Propagation

### Error Categories

```swift
enum OpenClawError: Error, LocalizedError {
  case gatewayStartupFailed(String)
  case gatewayUnresponsive
  case configurationError(String)
  case skillInstallationFailed(String)
  case websocketConnectionLost
  case authenticationFailed(String)
  case taskExecutionError(String)
  
  var errorDescription: String? {
    switch self {
    case .gatewayStartupFailed(let reason):
      return "OpenClaw gateway failed to start: \(reason)"
    case .gatewayUnresponsive:
      return "OpenClaw gateway is not responding"
    // ... other cases
    }
  }
}
```

### Error Monitoring

```swift
class ErrorMonitor {
  func handleOpenClawError(_ error: OpenClawError) {
    // Log error
    logger.error("OpenClaw error: \(error)")
    
    // Update UI state
    DispatchQueue.main.async {
      self.errorState = .error(error)
      self.showErrorAlert = true
    }
    
    // Attempt recovery
    Task {
      try await attemptRecovery(for: error)
    }
  }
  
  private func attemptRecovery(for error: OpenClawError) async throws {
    switch error {
    case .gatewayUnresponsive:
      try await restartGateway()
    case .websocketConnectionLost:
      try await reconnectWebSocket()
    case .configurationError:
      try regenerateConfiguration()
    default:
      break
    }
  }
}
```

## OpenClaw Updates

### Update Strategy: Bundled with App Updates

- OpenClaw updates are bundled with OpenPaw app updates
- No separate OpenClaw update mechanism
- Ensures compatibility between Swift UI and OpenClaw gateway
- Simplified deployment and testing

### Version Compatibility Matrix

```swift
struct VersionCompatibility {
  static let supportedOpenClawVersions = [
    "1.0.0": "OpenPaw 1.0.0-1.0.x",
    "1.1.0": "OpenPaw 1.1.0-1.1.x",
    "1.2.0": "OpenPaw 1.2.0+"
  ]
  
  func isCompatible(openclawVersion: String, openpawVersion: String) -> Bool {
    // Implementation to check compatibility
    return true
  }
}
```

### Migration Handling

```swift
class MigrationManager {
  func migrateWorkspace(from oldVersion: String, to newVersion: String) throws {
    let migrations = getMigrationsFor(from: oldVersion, to: newVersion)
    
    for migration in migrations {
      try migration.execute(workspacePath: workspacePath)
    }
  }
}
```

## Process Management and Health Monitoring

### Pilot Process Monitor

```swift
class ProcessMonitor: ObservableObject {
  @Published var openclawStatus: ProcessStatus = .stopped
  @Published var memoryUsage: MemoryUsage = .init()
  @Published var cpuUsage: Double = 0.0
  
  private var healthCheckTimer: Timer?
  
  func startMonitoring() {
    healthCheckTimer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: true) { _ in
      Task {
        await self.performHealthCheck()
      }
    }
  }
  
  private func performHealthCheck() async {
    guard let process = gatewayProcess else {
      openclawStatus = .stopped
      return
    }
    
    // Check process is running
    if !process.isRunning {
      openclawStatus = .crashed
      try? await restartGateway()
      return
    }
    
    // Check memory/CPU usage
    let usage = getProcessUsage(process.processIdentifier)
    await MainActor.run {
      self.memoryUsage = usage.memory
      self.cpuUsage = usage.cpu
    }
    
    // Check if gateway is responsive
    let isResponsive = await checkGatewayResponsiveness()
    openclawStatus = isResponsive ? .running : .unresponsive
    
    // Restart if unresponsive for too long
    if !isResponsive {
      unresponsiveCount += 1
      if unresponsiveCount >= 3 {
        try? await restartGateway()
      }
    } else {
      unresponsiveCount = 0
    }
  }
}
```

### Crash Recovery

```swift
func handleProcessCrash() async {
  logger.error("OpenClaw process crashed, attempting restart")
  
  // Clean up resources
  cleanup()
  
  // Wait before restart
  try? await Task.sleep(nanoseconds: 2_000_000_000) // 2 seconds
  
  // Restart with crash recovery flag
  do {
    try await startOpenClaw(crashRecovery: true)
    logger.info("OpenClaw successfully restarted after crash")
  } catch {
    logger.error("Failed to restart OpenClaw: \(error)")
    // Show user error dialog
  }
}
```

## Logging and Diagnostics

### Log Management

```swift
class LogManager {
  private let logDirectory = FileManager.default
    .urls(for: .applicationSupportDirectory, in: .userDomainMask)[0]
    .appendingPathComponent("OpenPaw/logs")
  
  func setupLogging() throws {
    // Create log directory
    try FileManager.default.createDirectory(at: logDirectory, 
                                          withIntermediateDirectories: true)
    
    // Configure log rotation
    let logFile = logDirectory.appendingPathComponent("openpaw.log")
    let openclawLogFile = logDirectory.appendingPathComponent("openclaw.log")
    
    // Setup log rotation (keep last 7 days, max 100MB)
    setupLogRotation(logFile: logFile, maxSize: 100_000_000, maxFiles: 7)
  }
  
  func collectDiagnostics() -> DiagnosticReport {
    return DiagnosticReport(
      openclawVersion: getOpenClawVersion(),
      processStatus: processMonitor.openclawStatus,
      memoryUsage: processMonitor.memoryUsage,
      recentLogs: getRecentLogs(lines: 100),
      configurationState: getConfigurationState(),
      skillStatus: getSkillStatus()
    )
  }
}
```

### Diagnostic Export

```swift
func exportDiagnostics() async throws -> URL {
  let report = logManager.collectDiagnostics()
  let exportURL = temporaryDirectory.appendingPathComponent("openpaw-diagnostics.json")
  
  let encoder = JSONEncoder()
  encoder.outputFormatting = .prettyPrinted
  let data = try encoder.encode(report)
  
  try data.write(to: exportURL)
  return exportURL
}
```

## Resource Management

### Memory Budget

- **Maximum heap size:** 512MB for OpenClaw process
- **Warning threshold:** 384MB (75%)
- **Critical threshold:** 448MB (87.5%)

### CPU Budget

- **Maximum CPU usage:** 25% average over 1 minute
- **Burst allowance:** 50% for up to 30 seconds
- **Background mode:** 5% maximum when app not active

### Resource Monitoring

```swift
struct ResourceLimits {
  static let maxMemoryBytes: UInt64 = 512 * 1024 * 1024 // 512MB
  static let maxCPUPercent: Double = 25.0
  static let warningMemoryBytes: UInt64 = maxMemoryBytes * 3 / 4
}

class ResourceMonitor {
  func enforceResourceLimits() async {
    let usage = getCurrentResourceUsage()
    
    if usage.memoryBytes > ResourceLimits.maxMemoryBytes {
      logger.warning("OpenClaw memory usage exceeded limit, restarting")
      try? await restartGateway()
    }
    
    if usage.cpuPercent > ResourceLimits.maxCPUPercent {
      // Throttle OpenClaw process
      throttleProcess()
    }
  }
}
```

## Security Considerations

### Sandboxing

- OpenClaw process runs with restricted permissions
- File access limited to workspace directory
- Network access limited to localhost and approved APIs
- No admin/root privileges required

### Configuration Validation

```swift
func validateConfiguration(_ config: OpenClawConfig) throws {
  // Validate network settings
  guard config.gateway.host == "127.0.0.1" else {
    throw ConfigError.invalidHost
  }
  
  // Validate port ranges
  guard (3000...3999).contains(config.gateway.port) else {
    throw ConfigError.invalidPort
  }
  
  // Validate OAuth tokens
  for (service, token) in config.integrations.oauth {
    try validateOAuthToken(token, for: service)
  }
}
```

## Performance Targets

- **Gateway startup time:** < 3 seconds
- **WebSocket connection time:** < 500ms
- **Message round-trip latency:** < 100ms
- **Memory usage:** < 512MB steady state
- **CPU usage:** < 25% average

## Testing Strategy

### Integration Tests

```swift
class OpenClawIntegrationTests: XCTestCase {
  func testGatewayStartup() async throws {
    let manager = OpenClawManager()
    try await manager.startOpenClaw()
    
    // Verify gateway is running
    XCTAssertEqual(manager.status, .running)
    
    // Verify WebSocket connection
    XCTAssertTrue(manager.isConnected)
    
    await manager.stopOpenClaw()
  }
  
  func testMessageRoundTrip() async throws {
    // Test full message flow
    let response = try await sendTestMessage()
    XCTAssertNotNil(response)
  }
}
```

---

**Document Status:** Draft v1.0  
**Next Review:** March 1, 2026  
**Implementation Target:** Q2 2026