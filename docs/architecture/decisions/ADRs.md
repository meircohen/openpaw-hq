# OpenPaw Architecture Decision Records

This document contains the Architecture Decision Records (ADRs) for OpenPaw, an open-source Mac app that provides a zero-config AI assistant for digital life automation.

---

## ADR-001: Swift + AppKit over Electron

**Status:** Accepted

**Context:** OpenPaw needs a native Mac application framework to provide the best user experience while maintaining performance and system integration. The primary alternatives were Electron (cross-platform web technologies), Swift + SwiftUI (modern Apple framework), and Swift + AppKit (mature Apple framework). Given our target audience of Mac power users who value performance and native behavior, the choice of application framework is critical to product success.

**Decision:** We chose Swift + AppKit over Electron and SwiftUI. AppKit provides mature, battle-tested APIs for complex UI interactions, window management, and system integration that OpenPaw requires. While SwiftUI is Apple's future, AppKit offers more granular control over UI behavior, better performance for complex interfaces, and broader macOS version compatibility (supporting macOS 10.15+).

**Consequences:** 
- **Positive:** Native performance, full macOS API access, smaller memory footprint (~50MB vs 200MB+ for Electron), seamless system integration (menu bar, notifications, file associations), and authentic Mac look-and-feel that users expect.
- **Negative:** Mac-only distribution, requiring Swift/Objective-C expertise, longer initial development time compared to web technologies, and need to maintain platform-specific code.
- **Neutral:** Future migration path to SwiftUI remains open as Apple's framework matures.

---

## ADR-002: OpenClaw as Embedded Engine

**Status:** Accepted

**Context:** OpenPaw needs a robust AI agent framework to handle complex automation tasks like email processing, calendar management, and digital life coordination. We evaluated building from scratch, integrating existing frameworks (LangChain, AutoGPT), or embedding OpenClaw. OpenClaw already provides a mature Node.js-based agent system with tool integrations, memory management, and multi-model AI support that aligns perfectly with OpenPaw's requirements.

**Decision:** Embed OpenClaw as the core AI engine rather than building from scratch. OpenClaw provides proven architecture for AI agents, extensive tool ecosystem, robust error handling, and active development. We wrap it with a Swift layer that manages the Node.js process, handles IPC communication, and provides native Mac integration.

**Consequences:**
- **Positive:** Leverages years of OpenClaw development, inherits 50+ built-in tools, proven multi-model architecture, active community contributions, and battle-tested agent orchestration. Reduces development time by ~12 months.
- **Negative:** Dependency on Node.js runtime, potential version compatibility issues, need to bridge Swift↔Node.js communication, and inability to modify core agent logic without upstream contributions.
- **Neutral:** Creates symbiotic relationship between projects, with OpenPaw driving consumer use cases and OpenClaw providing enterprise-grade engine.

---

## ADR-003: Bundled Node.js Runtime

**Status:** Accepted

**Context:** OpenPaw embeds OpenClaw (Node.js-based) but must provide zero-configuration installation. Users shouldn't need to install Node.js separately, manage versions, or handle npm dependencies. We evaluated system Node.js dependency, bundled Node.js, or compiling to native binaries. The zero-config requirement is fundamental to our consumer-friendly positioning.

**Decision:** Bundle a specific Node.js runtime (v18 LTS) within the .app bundle at `Contents/Frameworks/Node.framework/`. The bundled runtime includes npm and all required OpenClaw dependencies pre-installed. The Swift wrapper launches Node.js processes using the bundled runtime, ensuring consistent behavior across all installations.

**Consequences:**
- **Positive:** True zero-configuration setup, guaranteed Node.js version consistency, no system dependency conflicts, isolated from user's development environment, and simplified deployment/updates.
- **Negative:** Larger .app bundle size (+80MB), need to maintain Node.js security updates, increased complexity in build pipeline, and potential duplication if users have Node.js installed.
- **Mitigation:** Automated build scripts handle Node.js bundling, security update monitoring for Node.js releases, and efficient compression reduces bundle size impact.

---

## ADR-004: SQLite + SQLCipher for Local Storage

**Status:** Accepted

**Context:** OpenPaw requires persistent storage for AI agent memory, conversation history, automation rules, and user preferences. Data must remain 100% local with encryption at rest. We evaluated Core Data (Apple's ORM), Realm (cross-platform database), raw SQLite, and SQLite + SQLCipher (encrypted SQLite). The choice affects data portability, encryption capabilities, and integration complexity.

**Decision:** Use SQLite with SQLCipher for encrypted local storage. SQLCipher provides transparent AES-256 encryption while maintaining SQLite's reliability and performance. We implement a Swift wrapper that handles database operations and encryption key management through macOS Keychain.

**Consequences:**
- **Positive:** Proven encryption (used by Signal, WhatsApp), full SQL capabilities for complex queries, excellent performance for local data, cross-platform data portability, and granular backup/restore control.
- **Negative:** Additional dependency on SQLCipher, manual schema migration handling, and potential learning curve for developers unfamiliar with SQL.
- **Security:** Encryption keys stored in macOS Keychain, automatic database locking on app termination, and defense against disk analysis attacks.

---

## ADR-005: macOS Keychain for Secrets Management

**Status:** Accepted

**Context:** OpenPaw stores sensitive data including AI API keys, OAuth tokens, email credentials, and database encryption keys. Secure storage is critical for user trust and regulatory compliance. We evaluated macOS Keychain, encrypted files, environment variables, and cloud key management services. The solution must work offline and maintain user control.

**Decision:** Use macOS Keychain Services API for all sensitive data storage. Keychain provides hardware-backed encryption (Secure Enclave on supported Macs), system-level access controls, and integration with user authentication (Touch ID/Face ID). We implement a Swift KeychainManager class that handles CRUD operations with proper error handling.

**Consequences:**
- **Positive:** Hardware-backed security, automatic encryption key management, system-level biometric integration, user-controlled access permissions, and automatic backup via iCloud Keychain (if enabled).
- **Negative:** Mac-specific solution limiting portability, complex error handling for various Keychain states, and dependency on user's Keychain configuration.
- **Implementation:** Keychain items tagged with OpenPaw identifier, graceful fallback for Keychain access failures, and clear user messaging for permission requests.

---

## ADR-006: CGVirtualDisplay for Background Automation

**Status:** Accepted

**Context:** OpenPaw's automation capabilities require interacting with GUI applications while running in the background. Traditional approaches include screen scraping (fragile), accessibility APIs (limited), or virtual display environments. Many automation tasks require visual rendering that's impossible without a display context.

**Decision:** Implement CGVirtualDisplay (macOS 14+) with fallback to CGDisplayCreateUUIDFromDisplayID for older systems. Virtual displays provide isolated rendering environments for GUI automation without affecting the user's primary display. This enables headless browser automation, screenshot-based AI vision, and reliable GUI interaction testing.

**Consequences:**
- **Positive:** Reliable GUI automation independent of user's screen configuration, no interference with user's active work, consistent rendering environment for AI vision tasks, and ability to run multiple automation contexts simultaneously.
- **Negative:** Requires macOS 14+ for full functionality, additional complexity in display management, increased memory usage for virtual framebuffers, and potential performance impact on older hardware.
- **Fallback:** Graceful degradation to accessibility APIs on unsupported systems, clear capability detection and user messaging.

---

## ADR-007: MIT License

**Status:** Accepted

**Context:** OpenPaw's licensing choice affects adoption, commercial use, legal liability, and ecosystem development. We evaluated MIT (permissive), GPL v3 (copyleft), Apache 2.0 (permissive with patents), and proprietary licenses. The choice must align with our open-source philosophy while enabling broad adoption and commercial flexibility.

**Decision:** MIT License for maximum permissiveness and adoption. MIT allows commercial use, modification, and redistribution with minimal restrictions, encouraging ecosystem development and reducing legal barriers for enterprise adoption.

**Consequences:**
- **Positive:** Maximum adoption potential, compatible with commercial products, simple legal terms reducing adoption friction, enables proprietary forks for specialized use cases, and aligns with OpenClaw's MIT licensing.
- **Negative:** No copyleft protections ensuring contributions remain open source, potential for commercial exploitation without giving back, and no patent protection clauses.
- **Philosophy:** We prioritize widespread adoption and ecosystem growth over ensuring derivative works remain open source, trusting that value comes from the community and ongoing development.

---

## ADR-008: Bring Your Own Key (BYOK) Model

**Status:** Accepted

**Context:** AI API costs vary significantly by usage, and bundling API costs would require subscription pricing that conflicts with our open-source philosophy. Users have varying preferences for AI providers (OpenAI, Anthropic, local models), and cost transparency is essential for trust. We evaluated bundled API costs, freemium models, and BYOK approaches.

**Decision:** Implement Bring Your Own Key (BYOK) model where users provide their own AI API credentials. OpenPaw supports multiple providers (OpenAI, Anthropic, Azure OpenAI, local models via Ollama) with clear setup instructions and cost estimation tools.

**Consequences:**
- **Positive:** Complete cost transparency, user choice in AI providers, no subscription lock-in, scales with user needs, supports privacy-conscious users preferring local models, and eliminates our API cost burden.
- **Negative:** Additional setup complexity, potential user confusion about API pricing, dependency on third-party service availability, and need to maintain multiple provider integrations.
- **User Experience:** In-app setup wizard, cost estimation calculator, provider comparison guide, and clear error messages for API issues.

---

## ADR-009: OAuth via System Browser

**Status:** Accepted

**Context:** OpenPaw requires OAuth authentication for services like Gmail, Calendar, and cloud storage. Implementation options include embedded WebView, system browser redirect, or custom authentication flows. Security, user trust, and OAuth provider compliance are critical considerations.

**Decision:** Redirect OAuth flows to the user's default system browser (Safari by default) rather than using embedded WebView. This follows OAuth security best practices and provides users with familiar authentication UI including password managers, biometric authentication, and security indicators.

**Consequences:**
- **Positive:** Enhanced security through browser security features, user familiarity with authentication flow, password manager integration, OAuth provider compliance, and clear security indicators users trust.
- **Negative:** Context switching between app and browser, potential user confusion during redirect flow, and need to handle various browser configurations and states.
- **Implementation:** Custom URL scheme for return redirect, clear user messaging during OAuth flow, timeout handling for abandoned authentications, and graceful error recovery.

---

## ADR-010: Homebrew Cask as Primary Distribution

**Status:** Accepted

**Context:** Mac app distribution options include Mac App Store, direct download, Homebrew Cask, and third-party stores. Each has different reach, update mechanisms, and restrictions. OpenPaw's power-user target audience and open-source nature influence the optimal distribution strategy.

**Decision:** Homebrew Cask as the primary distribution channel, with direct GitHub releases as secondary. Homebrew provides automated updates, version management, and appeals to our technical early adopters. We maintain signed and notarized releases for security.

**Consequences:**
- **Positive:** Appeals to developer/power-user audience, automated dependency management, simple installation (`brew install --cask openpaw`), community-driven package maintenance, and no App Store restrictions on AI capabilities.
- **Negative:** Limited to users comfortable with command line tools, requires Homebrew ecosystem knowledge, smaller potential audience than App Store, and dependency on Homebrew infrastructure.
- **Future Path:** Consider Mac App Store submission after validating product-market fit, with Homebrew remaining the primary channel for power users.

---

## ADR-011: Semantic Versioning with CalVer for Releases

**Status:** Accepted

**Context:** Version numbering affects user expectations, update frequency communication, and compatibility signaling. Pure semantic versioning (SemVer) works well for libraries but can be constraining for applications with frequent feature releases. Calendar versioning (CalVer) provides time-based context but loses compatibility signaling.

**Decision:** Hybrid approach: CalVer for major releases (YYYY.MM format) with semantic versioning for patches (YYYY.MM.PATCH). Major features and breaking changes increment the month, bug fixes increment the patch number. Example: 2024.03.0, 2024.03.1, 2024.04.0.

**Consequences:**
- **Positive:** Clear temporal context for releases, aligns with rapid iteration cycles, easier marketing communication ("March 2024 release"), automatic deprecation signaling for old versions, and flexibility for frequent feature releases.
- **Negative:** Less clear compatibility signaling than pure SemVer, potential user confusion with hybrid approach, and need for clear documentation of versioning strategy.
- **Implementation:** Automated versioning in CI/CD pipeline, clear changelog documentation, and user communication about versioning approach.

---

## ADR-012: Privacy-Respecting Telemetry (Opt-in Only)

**Status:** Accepted

**Context:** Product improvement requires usage data, but privacy is fundamental to user trust. We need to balance product development insights with user privacy expectations. Options range from no telemetry to comprehensive analytics, with various privacy-preserving approaches in between.

**Decision:** Implement opt-in only, privacy-respecting telemetry using differential privacy techniques and local aggregation. Collect only essential metrics (crash reports, feature usage counts, performance metrics) with user consent. All telemetry can be disabled, and data is anonymized before transmission.

**Consequences:**
- **Positive:** Builds user trust through transparency, complies with privacy regulations (GDPR, CCPA), enables data-driven product decisions, provides crash debugging capabilities, and aligns with open-source values.
- **Negative:** Lower data collection rates due to opt-in model, increased development complexity for privacy-preserving analytics, and potential blind spots in user behavior understanding.
- **Implementation:** Clear consent UI, detailed privacy policy, local data aggregation, differential privacy for metrics, and regular privacy audits of collected data.

---

## Summary

These architectural decisions establish OpenPaw as a privacy-first, native Mac application that leverages proven open-source technologies while maintaining user control and transparency. The decisions prioritize user experience, security, and developer productivity while enabling sustainable open-source development.

Each decision can be revisited as the project evolves, but this foundation provides a solid base for building a trustworthy AI assistant that respects user privacy and integrates seamlessly with the macOS ecosystem.