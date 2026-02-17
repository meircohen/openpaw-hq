# Pilot — CTO Operations Manual

## Org Structure

**CEO:** Meir Cohen — Product vision, business decisions, final approvals
**CTO:** AI Agent (OpenClaw) — Everything else

### Virtual Team (Sub-Agent Roles)

| Role | Responsibility | Model |
|------|---------------|-------|
| Architecture Lead | System design, tech stack decisions, API contracts | Sonnet 4 |
| Product Lead | PRD, user stories, competitive analysis | Sonnet 4 |
| Frontend Engineer | Swift/AppKit UI, chat interface, onboarding | Sonnet 4 |
| Systems Engineer | Virtual display, screen capture, input injection | Sonnet 4 |
| AI Bridge Engineer | ChatGPT/Claude app integration, prompt orchestration | Sonnet 4 |
| QA Lead | Test framework, automated tests, security audit | Sonnet 4 |
| DevOps | Build pipeline, code signing, distribution | Sonnet 4 |

Each role is a sub-agent spawned for specific tasks. No permanent agents running — spawn on demand, verify output, merge work.

## Decision Framework

**CEO decides:**
- Product direction and features
- Business model and pricing
- Brand, naming, marketing
- Legal structure
- Budget allocation

**CTO decides (autonomously):**
- Technology stack
- Architecture patterns
- Code structure and conventions
- Testing strategy
- Build and deploy pipeline
- Sub-agent task allocation
- Technical trade-offs
- Security implementation

**Escalate to CEO when:**
- Feature scope changes
- Timeline shifts >1 week
- Cost implications >$100/month
- User-facing design decisions
- Privacy/legal gray areas

## Development Workflow

### Code Management
- **Repo:** GitHub (to be created — need org name from CEO)
- **Branching:** trunk-based with feature branches
- **Commits:** Conventional commits (feat:, fix:, docs:, refactor:)
- **Reviews:** All code reviewed by QA sub-agent before merge
- **CI:** GitHub Actions for build + test on every PR

### Task Management
- Tasks tracked in `/pilot/tasks/` as markdown files
- Each task has: ID, status, assignee (role), acceptance criteria
- Statuses: backlog → in-progress → review → done
- Sprint planning: weekly, documented in `/pilot/tasks/sprints/`

### Quality Gates
1. **Code complete** — implementation done, compiles, basic tests pass
2. **Review complete** — QA agent reviewed, no critical issues
3. **Integration tested** — works with other components
4. **Acceptance tested** — meets user story criteria
5. **Security reviewed** — no data leaks, no privilege escalation

## Communication Protocol

- **Daily summary:** Written to `/pilot/tasks/daily/YYYY-MM-DD.md`
- **Weekly report to CEO:** Key accomplishments, blockers, decisions needed
- **Decision records:** `/pilot/decisions/YYYY-MM-DD-slug.md`
- **Architecture decisions:** ADR format in `/pilot/architecture/decisions/`

## Risk Management

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Apple rejects app (automation permissions) | Medium | Critical | Research App Store guidelines early, prepare for notarized DMG distribution outside App Store |
| CGVirtualDisplay API changes/breaks | Low | High | Abstract display layer, support fallback capture methods |
| ChatGPT app changes UI frequently | High | Medium | Vision-based approach is resilient to UI changes; maintain prompt templates |
| Local model quality insufficient | Medium | Medium | Hybrid approach — local for simple, cloud-optional for complex |
| Performance too slow on older Macs | Medium | Medium | Define minimum hardware requirements, optimize capture pipeline |
| User's AI app rate-limited | High | Medium | Smart batching, prompt efficiency, queue management |

## Infrastructure Needs (from CEO)

### Needed Before Development Starts
- [ ] GitHub repo/org created (or permission to create under meircohen)
- [ ] Decision: distribute via App Store or direct download (DMG)?

### Needed Before Beta
- [ ] Apple Developer Account ($99/yr)
- [ ] Domain name (pilotapp.com, getpilot.ai, usepilot.app, etc.)
- [ ] Code signing certificate

### Needed Before Launch
- [ ] Landing page / website
- [ ] Payment processing (Stripe, Gumroad, or similar)
- [ ] Support email
- [ ] Privacy policy / Terms of service

## Current Status

**Phase:** Pre-development setup
**Next milestone:** Complete specs + architecture → CEO review → begin Week 1 coding
