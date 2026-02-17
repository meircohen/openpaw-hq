# Contributing to OpenPaw 🐾

Thanks for wanting to make OpenPaw better! Here's how to help.

## Quick Start

1. Fork the repo
2. Clone your fork: `git clone https://github.com/YOUR-USERNAME/openpaw.git`
3. Open `OpenPaw.xcodeproj` in Xcode 15+
4. Build and run (⌘R)
5. Make your changes
6. Submit a PR

## What We Need Help With

### Good First Issues 🟢
Look for the `good-first-issue` label. These are small, well-defined tasks perfect for your first contribution.

### Feature Work 🟡
Check the `feature` label. These are larger pieces — comment on the issue first to discuss approach before coding.

### Bug Fixes 🔴
`bug` label. Reproduce the bug, write a test that fails, fix it, verify the test passes.

## Code Standards

Read `/pilot/standards/coding-standards.md` before writing code. Key points:
- Swift, MVVM+Coordinator architecture
- SwiftLint for formatting (config in repo)
- Every public function needs a doc comment
- Every feature needs tests

## PR Process

1. Create a feature branch: `feat/your-feature-name`
2. Write your code + tests
3. Run the test suite locally: `⌘U` in Xcode
4. Push and open a PR against `main`
5. Fill out the PR template (what, why, how, testing)
6. Wait for CI to pass
7. A maintainer will review within 48 hours

## PR Requirements
- [ ] Tests pass
- [ ] SwiftLint passes
- [ ] No new warnings
- [ ] PR description explains the change
- [ ] Screenshots for UI changes
- [ ] Updated docs if behavior changed

## Architecture Decisions

If your change involves a significant architectural decision, write an ADR first. See `/pilot/architecture/decisions/` for examples. Discuss in an issue before coding.

## Communication

- **Issues:** Bug reports, feature requests, questions
- **Discussions:** Design proposals, RFC-style conversations
- **Discord:** Real-time chat (link in README)

## Code of Conduct

Be kind. Be constructive. Assume good intent. We're all here to make something great.

Specifically:
- No harassment, discrimination, or personal attacks
- Constructive feedback on code, not people
- Help newcomers — everyone was new once
- If you disagree, explain your reasoning respectfully

## License

By contributing, you agree that your contributions will be licensed under MIT, same as the project.

## Recognition

All contributors are listed in CONTRIBUTORS.md. We appreciate every contribution, no matter how small. 🐾
