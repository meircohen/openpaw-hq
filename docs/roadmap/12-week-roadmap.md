# Pilot - 12-Week MVP Roadmap

**Roadmap Period:** February 17 - May 12, 2025  
**Team Size:** 4 engineers + 1 designer + 1 PM  
**Budget:** $500K development budget  

## Executive Summary

Pilot is an AI-powered Mac app that acts as a digital assistant by understanding screen content and automating tasks. The MVP focuses on email triage as the primary use case, with a chat interface for user interaction and a robust screen capture + AI processing engine.

**Success Criteria:**
- ✅ Successfully capture and process screen content in real-time
- ✅ Integrate ChatGPT/Claude APIs for intelligent task understanding
- ✅ Execute one complete workflow (email triage) end-to-end
- ✅ Ship a polished beta to 100 internal users
- ✅ Achieve <2 second response times for typical tasks

---

## Week 1-2: Project Foundation & Architecture
**Sprint Goal:** Establish technical foundation and development infrastructure

### Week 1 (Feb 17-23)
**Team Focus:** Setup & Architecture (All hands)

#### Deliverables
- **Project Setup**
  - Xcode project structure with modular architecture
  - SwiftUI + Combine reactive framework
  - Dependency injection container
  - **Done:** `xcodebuild` succeeds, all team members can build locally

- **Development Infrastructure**
  - GitHub repo with branch protection rules
  - CI/CD pipeline (GitHub Actions)
  - Code quality tools (SwiftLint, SwiftFormat)
  - **Done:** PR workflow functional, automated tests run on every commit

- **Core Architecture Design**
  - Service layer interfaces defined
  - MVVM + Coordinator pattern implemented
  - Actor-based concurrency model
  - **Done:** Architecture decision record published, core protocols defined

#### Resource Allocation
- **2 Senior Engineers:** Project setup, architecture design
- **1 Junior Engineer:** Development environment, tooling
- **1 Designer:** Design system foundations, component library
- **1 PM:** Requirements refinement, sprint planning

### Week 2 (Feb 24-Mar 2)
**Team Focus:** Core Framework & Permissions

#### Deliverables
- **Permission Management System**
  - Screen recording permission handling
  - Accessibility API permissions
  - File system access permissions
  - **Done:** Permission flow UI complete, all required permissions obtainable

- **Core Services Framework**
  - ScreenCaptureService interface + mock implementation
  - AIService interface + mock implementation
  - TaskOrchestrator basic structure
  - **Done:** Service dependency injection working, unit tests >70% coverage

- **Logging & Diagnostics**
  - Structured logging system
  - Error reporting (Crashlytics/Sentry)
  - Performance monitoring hooks
  - **Done:** Logs are readable, errors captured in dashboard

#### Dependencies & Blockers
- **Apple Developer Account:** Need paid account for screen recording entitlements
- **AI API Access:** ChatGPT/Claude API keys and rate limits confirmed
- **macOS Version Support:** Decide minimum macOS version (12.0+ recommended)

#### Risk Checkpoint 1 🚨
**Go/No-Go Decision Point:** If permission system isn't working or AI API access blocked, pivot to simpler proof-of-concept

---

## Week 3-4: Screen Capture Engine
**Sprint Goal:** Robust screen capture with virtual display support

### Week 3 (Mar 3-9)
**Team Focus:** Screen Capture Foundation

#### Deliverables
- **Screen Capture Core**
  - Real-time screen capture using ScreenCaptureKit
  - Multi-display support
  - Capture region selection (full screen, window, custom area)
  - **Done:** Can capture any screen region at 30fps with <100ms latency

- **Virtual Display Integration**
  - Create/manage virtual displays for AI processing
  - Headless capture capabilities
  - Display configuration management
  - **Done:** Virtual display appears in system, apps can render to it

- **Image Processing Pipeline**
  - Image optimization for AI processing
  - Format conversion (PNG/JPEG)
  - Compression without quality loss
  - **Done:** Consistent image format, file sizes <2MB for typical captures

#### Resource Allocation
- **2 Senior Engineers:** ScreenCaptureKit implementation, virtual display
- **1 Junior Engineer:** Image processing utilities, testing
- **1 Designer:** Capture UI components, region selection interface

### Week 4 (Mar 10-16)
**Team Focus:** Virtual Input & Interaction

#### Deliverables
- **Virtual Input System**
  - Programmatic mouse clicks and keyboard input
  - Accessibility API integration for UI element identification
  - Input coordinate mapping and scaling
  - **Done:** Can click buttons and type text in any macOS app programmatically

- **Screen Analysis**
  - UI element detection and classification
  - Text recognition (built-in macOS OCR)
  - Interactive element identification
  - **Done:** Can identify buttons, text fields, menus in captured screens

- **Capture Session Management**
  - Start/stop capture sessions
  - Multiple concurrent capture contexts
  - Session state persistence
  - **Done:** Capture sessions survive app restart, multiple apps can be monitored

#### Dependencies & Blockers
- **Accessibility Permissions:** May require user approval in Security & Privacy
- **Virtual Display APIs:** Undocumented APIs may be unstable
- **Input Security:** System integrity protection may block some input methods

#### Risk Checkpoint 2 🚨
**Go/No-Go Decision Point:** If virtual input is blocked by macOS security, pivot to click-coordinates or user-guided interaction

---

## Week 5-6: AI App Bridge
**Sprint Goal:** Intelligent screen understanding and task interpretation

### Week 5 (Mar 17-23)
**Team Focus:** AI Integration Foundation

#### Deliverables
- **AI Service Implementation**
  - ChatGPT API integration with GPT-4V for vision
  - Claude API integration (Anthropic)
  - Fallback service switching (primary/secondary AI)
  - **Done:** Can send screen images to AI and get structured responses

- **Prompt Engineering**
  - Screen analysis prompts for UI understanding
  - Task interpretation prompts
  - Context-aware prompt templates
  - **Done:** AI accurately identifies UI elements 80%+ of the time

- **Response Parsing**
  - Structured JSON response handling
  - Action command extraction
  - Confidence scoring for AI suggestions
  - **Done:** AI responses parsed into actionable commands reliably

#### Resource Allocation
- **2 Senior Engineers:** AI API integration, prompt engineering
- **1 Junior Engineer:** Response parsing, error handling
- **1 Designer:** AI processing UI, loading states

### Week 6 (Mar 24-30)
**Team Focus:** Intelligent Task Processing

#### Deliverables
- **Context Management**
  - Multi-turn conversation support
  - Screen history and context retention
  - User preference learning
  - **Done:** AI remembers context across multiple interactions

- **Vision Processing Pipeline**
  - Image preprocessing for optimal AI analysis
  - Region-of-interest detection
  - Text extraction and enhancement
  - **Done:** AI vision accuracy >85% on typical screen content

- **Error Handling & Fallbacks**
  - AI service timeout handling
  - Graceful degradation when AI unavailable
  - User override for AI suggestions
  - **Done:** App remains functional when AI services are down

#### Dependencies & Blockers
- **API Rate Limits:** Ensure sufficient quota for development and testing
- **Response Times:** AI processing must be <5 seconds for good UX
- **Cost Management:** Track AI API costs, implement usage controls

#### Risk Checkpoint 3 🚨
**Go/No-Go Decision Point:** If AI accuracy is <70% or response times >10 seconds, reassess model choice or approach

---

## Week 7-8: Task Orchestrator & Memory System
**Sprint Goal:** Intelligent task execution with learning capabilities

### Week 7 (Mar 31-Apr 6)
**Team Focus:** Task Orchestration Engine

#### Deliverables
- **Task Orchestrator Core**
  - Action sequence planning and execution
  - Step-by-step task breakdown
  - Error recovery and retry logic
  - **Done:** Can execute multi-step tasks reliably with rollback on failure

- **Action Execution Engine**
  - Screen interaction commands (click, type, scroll)
  - Application launching and window management
  - File system operations (when needed)
  - **Done:** Can perform all actions needed for email triage workflow

- **Progress Tracking**
  - Task execution monitoring
  - Real-time progress feedback
  - Execution logging and replay
  - **Done:** User can see task progress and understand what's happening

#### Resource Allocation
- **2 Senior Engineers:** Task orchestration, action execution
- **1 Junior Engineer:** Progress tracking, logging
- **1 PM:** Task definition, user story validation

### Week 8 (Apr 7-13)
**Team Focus:** Memory System & First Workflow

#### Deliverables
- **Memory & Learning System**
  - User preference storage
  - Task history and patterns
  - Success/failure learning
  - **Done:** App improves suggestions based on user behavior

- **Email Triage Workflow Implementation**
  - Gmail/Mail.app screen capture and analysis
  - Email priority classification
  - Action suggestions (reply, delete, archive, flag)
  - **Done:** Complete email triage workflow works end-to-end

- **Workflow Testing & Refinement**
  - End-to-end email triage testing
  - Error case handling
  - Performance optimization
  - **Done:** Email triage completes in <30 seconds, 90%+ accuracy

#### Dependencies & Blockers
- **Email App Compatibility:** Must work with both Mail.app and Gmail web
- **Email Classification Accuracy:** AI must correctly identify email types
- **User Data Security:** Email content must be handled securely

#### Risk Checkpoint 4 🚨
**Go/No-Go Decision Point:** If email triage workflow doesn't work reliably, focus on simpler tasks or manual guidance mode

---

## Week 9-10: Chat UI & User Experience
**Sprint Goal:** Polished user interface and seamless interaction

### Week 9 (Apr 14-20)
**Team Focus:** Chat Interface & User Interaction

#### Deliverables
- **Chat UI Implementation**
  - Message thread interface
  - Typing indicators and AI thinking states
  - Rich message formatting (code, links, images)
  - **Done:** Chat interface feels responsive and polished like modern messaging apps

- **Task Suggestion Interface**
  - AI-generated task recommendations
  - One-click task approval/execution
  - Task customization and parameters
  - **Done:** Users can easily approve, modify, or reject AI suggestions

- **Real-time Feedback**
  - Live task execution visualization
  - Screen highlighting for actions being taken
  - Success/error notifications
  - **Done:** User always knows what the app is doing and why

#### Resource Allocation
- **1 Senior Engineer:** Chat UI backend, message handling
- **1 Junior Engineer:** UI components, animations
- **1 Designer:** Full-time UI/UX design and refinement
- **1 PM:** User testing coordination, feedback integration

### Week 10 (Apr 21-27)
**Team Focus:** Onboarding & Settings

#### Deliverables
- **Onboarding Flow**
  - Welcome screens and app explanation
  - Permission setup wizard
  - First task walkthrough
  - **Done:** New users can set up and use the app without external help

- **Settings & Configuration**
  - AI service preferences (GPT vs Claude)
  - Privacy and data retention controls
  - Workflow customization options
  - **Done:** Users can configure app behavior to their preferences

- **Help & Documentation**
  - In-app help system
  - Troubleshooting guides
  - Video tutorials for key workflows
  - **Done:** Users can self-serve for common issues and questions

#### Dependencies & Blockers
- **User Testing Feedback:** Need real user testing to refine onboarding
- **Privacy Compliance:** Ensure GDPR/CCPA compliance for data handling
- **Accessibility:** Must pass basic accessibility audits

---

## Week 11-12: QA, Polish & Beta Release
**Sprint Goal:** Production-ready beta with comprehensive testing

### Week 11 (Apr 28-May 4)
**Team Focus:** Quality Assurance & Polish

#### Deliverables
- **Comprehensive Testing**
  - Full automated test suite (unit, integration, UI)
  - Performance testing and optimization
  - Edge case and error condition testing
  - **Done:** Test coverage >80%, all critical paths tested

- **UI/UX Polish**
  - Animation refinement and performance
  - Visual design consistency
  - Dark mode support
  - **Done:** App feels polished and professional

- **Security Hardening**
  - Code security review
  - Dependency vulnerability scanning
  - Data encryption and privacy controls
  - **Done:** No high/critical security vulnerabilities

#### Resource Allocation
- **1 Senior Engineer:** Performance optimization, security review
- **2 Engineers:** Bug fixes, polish, testing
- **1 Designer:** Final UI polish, icon design
- **1 PM:** Beta planning, user communication

### Week 12 (May 5-11)
**Team Focus:** Beta Release & Launch Preparation

#### Deliverables
- **Production Deployment**
  - App signing and notarization
  - Crash reporting and analytics
  - Auto-update system
  - **Done:** App installs and runs on fresh Mac without issues

- **Beta Distribution**
  - TestFlight setup or direct distribution
  - Beta user onboarding materials
  - Feedback collection system
  - **Done:** 100 beta users can install and use the app

- **Launch Readiness**
  - App Store submission preparation
  - Marketing website and materials
  - Support documentation and processes
  - **Done:** Ready for App Store review and public launch

#### Dependencies & Blockers
- **Apple Review Process:** App Store review can take 1-7 days
- **Beta User Recruitment:** Need committed beta testers
- **Legal Review:** Privacy policy, terms of service finalized

#### Risk Checkpoint 5 🚨
**Final Go/No-Go Decision:** If beta feedback reveals major usability issues or bugs, delay public release

---

## Resource Allocation Summary

### Team Structure
- **2 Senior Engineers** (iOS/macOS specialists) - $120K/quarter each
- **1 Junior Engineer** (recent CS grad) - $60K/quarter
- **1 UI/UX Designer** (Mac app experience) - $80K/quarter
- **1 Product Manager** (technical PM) - $90K/quarter
- **Total Salary Cost:** $470K for 12 weeks

### Infrastructure & Tools
- **Apple Developer Program:** $99/year
- **AI API Costs:** $5K/month (GPT-4V + Claude)
- **Development Tools:** $2K (Figma, analytics, CI/CD)
- **Testing Devices:** $5K (various Mac models)
- **Total Infrastructure:** $30K

### Risk Mitigation Budget
- **Contingency Fund:** 20% of total budget ($100K)
- **External Security Audit:** $15K
- **User Research & Testing:** $10K

## Success Metrics & KPIs

### Technical Metrics
- **Screen Capture Performance:** <500ms latency, 30fps
- **AI Response Time:** <3 seconds average
- **Task Success Rate:** >90% for email triage
- **App Stability:** <0.1% crash rate
- **Memory Usage:** <500MB typical, <1GB peak

### User Experience Metrics
- **Onboarding Completion:** >80% complete first task
- **Daily Active Users:** >50% of beta users
- **Task Completion Rate:** >75% of attempted tasks
- **User Satisfaction:** >4.0/5 rating
- **Support Ticket Rate:** <10% of users need help

### Business Metrics
- **Beta User Retention:** >60% 30-day retention
- **Feature Usage:** Email triage used by >80% of users
- **Performance Against Goals:** All P0 features delivered
- **Budget Performance:** <10% over budget
- **Timeline Performance:** <2 weeks delay acceptable

## Dependencies & External Risks

### Technical Dependencies
- **macOS API Stability:** ScreenCaptureKit and Accessibility APIs
- **AI Service Availability:** OpenAI and Anthropic service uptime
- **Apple Developer Program:** Signing certificates and entitlements
- **Third-Party Libraries:** SwiftUI, Combine framework compatibility

### Business Dependencies
- **AI API Pricing:** Cost changes could impact unit economics
- **App Store Approval:** Review process and policy compliance
- **User Demand Validation:** Product-market fit assumptions
- **Competitive Landscape:** Similar products launching

### Mitigation Strategies
- **Technical:** Multiple AI providers, fallback modes, extensive testing
- **Business:** Conservative cost estimates, early App Store engagement
- **Schedule:** 20% buffer time, parallel work streams where possible
- **Quality:** Automated testing, continuous integration, user feedback loops

---

## Weekly Checkpoint Process

### Monday Sprint Planning
- Review previous week deliverables and metrics
- Identify blockers and dependency issues
- Adjust current week priorities if needed
- Update risk assessment and mitigation plans

### Wednesday Mid-Week Review
- Progress check against weekly goals
- Early warning system for schedule risks
- Resource reallocation if needed
- Stakeholder communication on any issues

### Friday Sprint Review & Retrospective
- Demo completed features to stakeholders
- Evaluate "done" criteria for each deliverable
- Document lessons learned and process improvements
- Plan weekend/off-hours work if critical path items are at risk

### Risk Escalation Process
- **Green (On Track):** Normal weekly reporting
- **Yellow (At Risk):** Daily check-ins, mitigation planning
- **Red (Critical):** Immediate escalation, all-hands meeting, scope/timeline adjustment

This roadmap provides a clear path to MVP with specific, measurable deliverables and built-in checkpoints to ensure we deliver a high-quality product on time and within budget.