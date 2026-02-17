# OpenPaw — Release Process

## Versioning

Semantic versioning: `MAJOR.MINOR.PATCH`
- **MAJOR** (1.0, 2.0): Breaking changes, major features
- **MINOR** (1.1, 1.2): New features, backwards compatible
- **PATCH** (1.0.1, 1.0.2): Bug fixes only

Pre-release: `1.0.0-beta.1`, `1.0.0-rc.1`

## Release Cadence

- **Patch releases:** As needed for bugs (can be same day)
- **Minor releases:** Every 2-4 weeks
- **Major releases:** When significant milestones warrant
- **Beta releases:** Weekly during pre-launch

## Release Checklist

### Pre-Release
- [ ] All planned issues closed or moved to next milestone
- [ ] All tests passing on CI (unit + integration + UI)
- [ ] Performance benchmarks within budget
- [ ] No critical or high security vulnerabilities (`npm audit`)
- [ ] CHANGELOG.md updated with all changes
- [ ] Version bumped in Xcode project + Info.plist
- [ ] README updated if needed
- [ ] Migration tested (fresh install + upgrade from previous version)

### Build & Sign
- [ ] Build release configuration in Xcode
- [ ] Code sign with Developer ID certificate
- [ ] Notarize with Apple (`notarytool submit`)
- [ ] Wait for notarization approval
- [ ] Staple notarization ticket to app
- [ ] Create DMG with `create-dmg`
- [ ] Generate SHA-256 checksum of DMG

### Publish
- [ ] Create GitHub Release with tag `vX.Y.Z`
- [ ] Upload DMG + checksum to release
- [ ] Write release notes (human-readable, not just commit list)
- [ ] Update Homebrew Cask formula (PR to homebrew-cask)
- [ ] Update landing page download link
- [ ] Sparkle appcast.xml updated for auto-update

### Post-Release
- [ ] Verify auto-update works (test on previous version)
- [ ] Verify Homebrew install works
- [ ] Verify DMG download + install works
- [ ] Monitor GitHub Issues for regression reports (48 hours)
- [ ] Tweet release announcement from @openpaw
- [ ] Update Discord with release notes

## Rollback Plan

If a release has critical bugs:
1. Yank the GitHub Release (mark as pre-release)
2. Update Homebrew Cask to previous version
3. Push Sparkle update pointing to previous version
4. Post incident report on GitHub Discussions
5. Fix forward with a patch release ASAP

## CI/CD Pipeline (GitHub Actions)

```yaml
# Triggered on every push/PR
build-and-test:
  - Checkout code
  - Install dependencies (npm ci for OpenClaw, SPM resolve for Swift)
  - Build (xcodebuild)
  - Run unit tests
  - Run integration tests
  - Run SwiftLint
  - Run npm audit on OpenClaw
  - Performance benchmark (fail if over budget)

# Triggered on tag push (vX.Y.Z)
release:
  - All of above
  - Build release configuration
  - Code sign
  - Notarize
  - Create DMG
  - Generate checksums
  - Create GitHub Release
  - Update appcast.xml
  - Notify Discord webhook
```

## Hotfix Process

For critical bugs in production:
1. Branch from the release tag: `hotfix/description`
2. Fix the bug + add regression test
3. PR against main (expedited review)
4. Cherry-pick to release branch if needed
5. Cut patch release immediately
6. Follow full release checklist (no shortcuts on signing/notarization)
