# Pilot - Coding Standards

## Swift Coding Conventions

### Naming Conventions
- **Classes & Structs**: `PascalCase` (e.g., `ScreenCaptureEngine`, `TaskOrchestrator`)
- **Functions & Variables**: `camelCase` (e.g., `captureScreen()`, `currentTask`)
- **Constants**: `camelCase` with `let` (e.g., `maxRetryAttempts`)
- **Enums**: `PascalCase` cases with `camelCase` (e.g., `TaskStatus.inProgress`)
- **Protocols**: Descriptive names ending in `-ing` or `-able` (e.g., `ScreenCapturing`, `Orchestrable`)
- **Extensions**: Use `// MARK: - ProtocolName` for protocol conformance

### File Organization
```
Sources/
├── Core/
│   ├── Models/
│   ├── Services/
│   └── Utilities/
├── Features/
│   ├── ScreenCapture/
│   ├── AIBridge/
│   ├── TaskOrchestrator/
│   └── Chat/
├── UI/
│   ├── Views/
│   ├── ViewModels/
│   └── Components/
└── Resources/
```

### Architecture Patterns

#### MVVM + Coordinator Pattern
- **ViewModels**: Handle business logic, API calls, state management
- **Views**: SwiftUI views, minimal logic, reactive to ViewModels
- **Coordinators**: Handle navigation flow between features
- **Services**: Shared business logic (ScreenCaptureService, AIService)

#### Dependency Injection
```swift
protocol ServiceContainer {
    var screenCaptureService: ScreenCaptureService { get }
    var aiService: AIService { get }
    var taskOrchestrator: TaskOrchestrator { get }
}
```

#### Actor Pattern for Concurrency
```swift
@MainActor
class ScreenCaptureViewModel: ObservableObject {
    @Published var captureState: CaptureState = .idle
    
    private let screenService: ScreenCaptureService
    
    func startCapture() async throws {
        captureState = .capturing
        // Implementation
    }
}
```

### Error Handling

#### Custom Error Types
```swift
enum PilotError: LocalizedError {
    case screenCapturePermissionDenied
    case aiServiceUnavailable(reason: String)
    case taskExecutionFailed(taskId: String, error: Error)
    
    var errorDescription: String? {
        switch self {
        case .screenCapturePermissionDenied:
            return "Screen capture permission is required for Pilot to function."
        case .aiServiceUnavailable(let reason):
            return "AI service unavailable: \(reason)"
        case .taskExecutionFailed(let taskId, let error):
            return "Task \(taskId) failed: \(error.localizedDescription)"
        }
    }
}
```

#### Error Handling Strategy
- Use `Result<Success, Failure>` for recoverable errors
- Use `async throws` for operations that can fail
- Log all errors with context using structured logging
- Present user-friendly error messages with actionable steps

### Code Style

#### Formatting
- **Indentation**: 4 spaces (no tabs)
- **Line Length**: 120 characters max
- **Trailing Commas**: Required in multi-line collections
- **SwiftFormat** configuration in `.swiftformat`

#### Documentation
```swift
/// Captures the current screen content for AI processing
/// - Parameter options: Configuration for capture behavior
/// - Returns: Captured screen data with metadata
/// - Throws: `PilotError.screenCapturePermissionDenied` if permissions missing
func captureScreen(with options: CaptureOptions) async throws -> ScreenCapture {
    // Implementation
}
```

## Git Workflow

### Branch Strategy (GitFlow Modified)
- **main**: Production-ready code, tagged releases
- **develop**: Integration branch for features
- **feature/**: New features (`feature/ai-bridge-integration`)
- **hotfix/**: Critical fixes (`hotfix/memory-leak-fix`)
- **release/**: Release preparation (`release/v1.0.0`)

### Branch Naming
- `feature/JIRA-123-screen-capture-engine`
- `bugfix/JIRA-456-memory-leak`
- `hotfix/critical-crash-fix`
- `chore/update-dependencies`

### Commit Conventions
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only changes
- `style`: Code style changes (formatting, semicolons, etc.)
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `perf`: Performance improvements
- `test`: Adding missing tests
- `chore`: Changes to build process, auxiliary tools

**Examples:**
```
feat(screen-capture): implement virtual display support

Add support for capturing virtual displays for AI processing.
Includes permission handling and error recovery.

Closes PILOT-123
```

### Pull Request Process

#### PR Requirements
1. **Branch is up to date** with target branch
2. **All tests pass** (unit, integration, UI)
3. **Code coverage** >= 80% for new code
4. **SwiftLint** warnings resolved
5. **Manual testing** completed with screenshots/video
6. **Security review** for sensitive code paths

#### PR Template
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Performance impact assessed

## Security Checklist
- [ ] No hardcoded secrets
- [ ] Input validation added
- [ ] Permission checks implemented
- [ ] Data encryption verified

## Screenshots/Videos
[Required for UI changes]
```

#### Review Requirements
- **2 approvals required** for main branch
- **1 approval required** for develop branch  
- **Security team approval** for authentication/permissions code
- **QA approval** for user-facing features

## Code Review Checklist

### Functionality
- [ ] Code solves the stated problem correctly
- [ ] Edge cases are handled appropriately
- [ ] Error handling is comprehensive
- [ ] Performance implications considered

### Code Quality
- [ ] Code is readable and self-documenting
- [ ] Functions are single-purpose and appropriately sized
- [ ] No code duplication without justification
- [ ] Appropriate use of Swift language features

### Architecture
- [ ] Follows established architectural patterns
- [ ] Proper separation of concerns
- [ ] Dependencies are injected, not hardcoded
- [ ] Thread safety considerations addressed

### Security
- [ ] Input validation implemented
- [ ] No sensitive data in logs
- [ ] Proper permission checks
- [ ] Secure data storage practices

### Testing
- [ ] Appropriate test coverage (>80%)
- [ ] Tests are meaningful and test behavior
- [ ] Tests are maintainable and fast
- [ ] Integration points are tested

## Documentation Requirements

### Code Documentation
- **Public APIs**: Full documentation with examples
- **Complex algorithms**: Inline comments explaining approach
- **Business logic**: Comments explaining "why" not just "what"
- **Configuration**: Document all environment variables and settings

### Architecture Documentation
- **Decision Records**: Major architectural decisions in `/docs/decisions/`
- **API Documentation**: Auto-generated from code comments
- **Integration Guides**: How to integrate with external services
- **Deployment Guide**: Step-by-step deployment instructions

### User Documentation
- **README**: Project overview, setup, basic usage
- **CONTRIBUTING**: Development setup, coding standards
- **CHANGELOG**: Version history with breaking changes noted

## Testing Requirements

### Unit Tests (70% of test suite)
- **Coverage Target**: 90% line coverage
- **Framework**: XCTest with custom test utilities
- **Mocking**: Protocol-based mocking for external dependencies
- **Naming**: `test_methodName_scenario_expectedResult()`

```swift
func test_captureScreen_withPermissionDenied_throwsPermissionError() async {
    // Given
    mockPermissionService.shouldDenyPermission = true
    
    // When & Then
    await XCTAssertThrowsError(
        try await screenCaptureService.captureScreen()
    ) { error in
        XCTAssertEqual(error as? PilotError, .screenCapturePermissionDenied)
    }
}
```

### Integration Tests (20% of test suite)
- **Scope**: Service-to-service interactions
- **Environment**: Test environment with mock external APIs
- **Data**: Test fixtures and factories for consistent data
- **Cleanup**: Proper teardown to avoid test pollution

### UI Tests (10% of test suite)
- **Framework**: XCUITest for critical user flows
- **Coverage**: Happy path and key error scenarios
- **Flakiness**: Use waiting predicates, avoid hard-coded delays
- **Parallel Execution**: Tests must be independent

```swift
func test_taskExecution_endToEnd_completesSuccessfully() {
    let app = XCUIApplication()
    app.launch()
    
    // Test critical path: Screen capture → AI processing → Task execution
    app.buttons["Start Task"].tap()
    
    let completionAlert = app.alerts["Task Complete"]
    XCTAssertTrue(completionAlert.waitForExistence(timeout: 30))
}
```

### Performance Tests
- **Benchmarks**: Memory usage, CPU utilization, response times
- **Regression Testing**: Automated performance regression detection
- **Profiling**: Regular Instruments profiling for optimization opportunities

### Testing CI/CD Integration
- **Pre-commit**: Unit tests + SwiftLint
- **PR Pipeline**: Full test suite + code coverage report
- **Nightly**: Performance tests + security scans
- **Release**: Full regression suite + manual QA verification