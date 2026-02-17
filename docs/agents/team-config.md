# Pilot — AI Development Team Configuration

## How Development Works

Pilot is built entirely by AI agents orchestrated by the CTO agent (OpenClaw main session). No human developers. The CEO (Meir) provides vision and approvals.

## Agent Spawning Rules

1. **One agent per task.** Never two agents writing the same file.
2. **Sonnet 4 for everything.** 100% success rate, no exceptions.
3. **600 second timeout** on all spawns.
4. **Verify every output.** Read the file, check it makes sense, before reporting to CEO.
5. **Parallel when independent.** Sequential when dependent.

## Development Agents

### Code Writer
- **Trigger:** New feature or component to build
- **Input:** Spec from architecture doc + acceptance criteria
- **Output:** Swift source files written to `/pilot/src/`
- **Verification:** File exists, compiles (if build system set up), meets acceptance criteria

### Code Reviewer
- **Trigger:** After code writer completes
- **Input:** Source files + coding standards
- **Output:** Review notes, approved/rejected with specific feedback
- **Verification:** Review is substantive, not rubber-stamp

### Test Writer
- **Trigger:** After code approved
- **Input:** Source files + QA framework + acceptance criteria
- **Output:** XCTest files written to `/pilot/tests/`
- **Verification:** Tests exist, cover acceptance criteria

### Documentation Writer
- **Trigger:** After feature complete
- **Input:** Source files + architecture docs
- **Output:** Updated docs in `/pilot/docs/`
- **Verification:** Docs match implementation

## Build Pipeline (Future — Week 3+)

```
Code Writer → Code Reviewer → Test Writer → Build → Test → Merge
     ↑              ↓
     └── Revision ──┘
```

## File Ownership Rules

No two agents write the same file. Ownership:
- `/pilot/specs/` — Product Lead agent
- `/pilot/architecture/` — Architecture Lead agent
- `/pilot/src/` — Code Writer agent (per-component)
- `/pilot/tests/` — Test Writer agent
- `/pilot/qa/` — QA Lead agent
- `/pilot/roadmap/` — CTO (main session)
- `/pilot/decisions/` — CTO (main session)
- `/pilot/security/` — CTO (main session)

## Handoff Protocol

When handing work between agents:
1. Writer completes → writes file → reports to CTO
2. CTO verifies file exists and is non-empty
3. CTO spawns next agent with explicit file path as input
4. Next agent reads the file, does its work, writes output
5. CTO verifies and reports to CEO if milestone reached
