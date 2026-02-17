# OpenPaw HQ 🐾

**Live dashboard for OpenPaw — an AI-powered startup built entirely by agents**

[![Live Dashboard](https://img.shields.io/badge/Live-Dashboard-FF6B4A?style=for-the-badge)](https://meircohen.github.io/openpaw-hq/)

## What is OpenPaw?

OpenPaw is a **free, open-source Mac app** that wraps OpenClaw as its engine to deliver the most intuitive AI assistant experience possible.

- **"AI that just works"** — Zero-config onboarding, native Swift UI
- **Privacy by architecture** — No cloud, no subscription, users bring their own AI key
- **OpenClaw for everyone** — Enterprise-grade capabilities in consumer-friendly packaging
- **Built by AI agents** — The entire startup created by AI agents, no human developers

## About This Dashboard

This live dashboard shows real-time progress on the OpenPaw project:

- 🏢 **Agent Office Floor** — Status of all AI development agents
- 📊 **Live Metrics** — Documentation progress, success rates, readiness
- 📚 **Full Documentation** — Browse all project specs, architecture, and decisions
- 🗺️ **12-Week Roadmap** — Development phases and milestones
- 📡 **Activity Feed** — Real-time updates from the AI team

## How It Works

The dashboard pulls live data from the actual OpenPaw development workspace:

1. **`update-state.sh`** — Scans project files, generates `state.json` with real metrics
2. **`index.html`** — Premium dashboard fetches state.json every 30 seconds
3. **GitHub Pages** — Serves the dashboard live at [openpaw.dev](https://meircohen.github.io/openpaw-hq/)
4. **Auto-deploy** — Script runs via cron, commits updates to keep dashboard current

## Repository Structure

```
openpaw-hq/
├── index.html          # Main dashboard (single file, no build step)
├── state.json          # Live data (auto-generated)
├── update-state.sh     # Data generator script
├── docs/              # All project documentation (copied from pilot/)
└── CNAME              # Domain configuration
```

## Development

The OpenPaw project is managed in `/Users/meircohen/.openclaw/workspace/pilot/` with AI agents orchestrated by OpenClaw. This dashboard provides external visibility into an entirely AI-driven development process.

### Key Documents

- **[Product Requirements (PRD)](docs/specs/prd-v1.md)** — What we're building and why
- **[Technical Architecture](docs/architecture/technical-architecture-v1.md)** — System design and tech stack
- **[Brand Guide](docs/brand/BRAND-GUIDE.md)** — Visual identity and messaging
- **[12-Week Roadmap](docs/roadmap/12-week-roadmap.md)** — Development timeline
- **[CTO Operations](docs/CTO-OPERATIONS.md)** — AI team management

## The Experiment

OpenPaw represents a unique experiment: **Can AI agents build a complete commercial software product?**

- ✅ Product requirements and competitive analysis
- ✅ Technical architecture and database design  
- ✅ Brand identity and visual design system
- ✅ Security model and coding standards
- ✅ QA framework and release process
- 🔄 UI wireframes and technical spikes (in progress)
- ⏳ Development begins Week 1 after validation complete

**Current Status:** Pre-development (measuring twice before cutting)  
**Team:** 6 AI agents + 1 CTO agent + 1 human CEO  
**Success Rate:** 100% (Sonnet 4 model)  
**Documentation:** ~25 documents, ~400KB specifications  

---

**Built with ❤️ by AI agents | [Follow the journey](https://meircohen.github.io/openpaw-hq/)**