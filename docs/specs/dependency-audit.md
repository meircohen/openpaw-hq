# OpenPaw — Dependency Audit

## Principle
Minimize dependencies. Every dependency is a risk (security, maintenance, license, breakage).

## Core Dependencies

### Runtime Dependencies (shipped with app)

| Dependency | Version | License | Purpose | Risk | Mitigation |
|-----------|---------|---------|---------|------|------------|
| Node.js | 22.x LTS | MIT | OpenClaw runtime | Low — LTS, well-maintained | Pin version, bundle in .app |
| OpenClaw | Fork of latest | MIT | Agent engine | Medium — active development | Fork, own our version |
| SQLCipher | 4.x | BSD | Encrypted SQLite | Low — mature, stable | Static link |
| Sparkle | 2.x | MIT | Auto-updates | Low — standard macOS updater | Well-established |

### Build Dependencies (not shipped)

| Dependency | Purpose | License |
|-----------|---------|---------|
| Xcode 15+ | Build toolchain | Apple EULA |
| Swift Package Manager | Dependency management | Apache 2.0 |
| create-dmg | DMG packaging | MIT |
| xcnotary / notarytool | Notarization | Apple tooling |
| GitHub Actions | CI/CD | GitHub TOS |

### OpenClaw's Dependencies (inherited)
These come with OpenClaw. We don't control them but need to audit:

| Category | Count | License Risk |
|----------|-------|-------------|
| npm packages | ~200-400 | Mostly MIT/ISC — need full audit |
| Native bindings | Few (better-sqlite3, etc.) | MIT |

**Action:** Run `npm audit` and `license-checker` on OpenClaw fork before shipping. Flag any GPL or AGPL packages — they may conflict with MIT.

### Optional Dependencies (user-installed)

| Dependency | Purpose | Required? |
|-----------|---------|-----------|
| gog CLI | Gmail/Calendar integration | Only if user connects Google |
| gh CLI | GitHub integration | Only if user connects GitHub |
| Ollama | Local AI models | Only if user chooses local AI |

## License Compatibility Matrix

| Our License | Dependency License | Compatible? |
|------------|-------------------|-------------|
| MIT | MIT | ✅ |
| MIT | BSD | ✅ |
| MIT | ISC | ✅ |
| MIT | Apache 2.0 | ✅ |
| MIT | GPL v2 | ❌ Must remove or isolate |
| MIT | GPL v3 | ❌ Must remove or isolate |
| MIT | AGPL | ❌ Must remove |
| MIT | LGPL | ⚠️ Dynamic linking OK, static linking problematic |

## Security Audit Checklist

- [ ] Run `npm audit` on OpenClaw fork — zero critical/high vulnerabilities
- [ ] Run `license-checker` — zero GPL/AGPL in dependency tree
- [ ] Verify SQLCipher version has no known CVEs
- [ ] Verify Sparkle version supports Ed25519 signatures
- [ ] Check all native bindings compile on arm64 (Apple Silicon)
- [ ] Verify Node.js version has no known CVEs
- [ ] Static analysis of all Swift code (SwiftLint)
- [ ] No telemetry/tracking in any dependency without explicit opt-in

## Supply Chain Security

- **Lock files:** Package-lock.json committed, `npm ci` for reproducible builds
- **Signing:** All releases code-signed with Apple Developer cert + notarized
- **Checksums:** SHA-256 checksums published with every release
- **Reproducible builds:** Document exact build environment for verification
- **SBOMs:** Generate Software Bill of Materials with each release (CycloneDX format)
