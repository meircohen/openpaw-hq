# OpenPaw 🐾 — One-Pager

## What It Is

OpenPaw is a **free, open-source AI assistant for Mac**. Download it, connect your accounts, and it handles your email, calendar, messages, and digital life — all running privately on your device.

## The Problem

AI is powerful but fragmented. You're paying for ChatGPT, Claude, Gemini — but they're all trapped in browser tabs. They can't actually *do* anything on your computer. Meanwhile, automation tools require API keys, config files, and technical setup that 99% of people won't touch.

## The Solution

OpenPaw gives your AI "hands." It wraps **OpenClaw** (a proven open-source AI agent framework) in a native Mac app with zero-config onboarding. You bring your own AI — any provider or local model — and OpenPaw connects it to your digital life.

**Three steps:**
1. Download OpenPaw
2. Connect your accounts (Gmail, Calendar, etc.)
3. That's it. Seriously.

## How It Works

```
┌─────────────────────────────────┐
│         OpenPaw (Swift UI)       │  ← What you see
├─────────────────────────────────┤
│      WebSocket Bridge            │  ← How they talk
├─────────────────────────────────┤
│   OpenClaw Engine (embedded)     │  ← The brain
├─────────────────────────────────┤
│  Your AI (Claude/GPT/Local/Any)  │  ← You choose
└─────────────────────────────────┘
         100% on your Mac
```

## Key Differentiators

| | OpenPaw | ChatGPT Desktop | Copilot | Siri |
|---|---|---|---|---|
| **Runs locally** | ✅ | ❌ Cloud | ❌ Cloud | Partial |
| **Any AI provider** | ✅ BYOK | GPT only | GPT only | Apple only |
| **Takes actions** | ✅ | ❌ Chat only | Limited | Limited |
| **Open source** | ✅ MIT | ❌ | ❌ | ❌ |
| **Free** | ✅ Forever | $20/mo | $20/mo | Free (limited) |
| **Privacy** | ✅ By architecture | Trust OpenAI | Trust Microsoft | Trust Apple |

## What It Does (MVP)

- 📧 Reads, triages, and drafts email responses
- 📅 Manages your calendar and schedules meetings
- 💬 Handles messages across platforms
- 📁 Organizes files and documents
- 🔍 Searches your digital life with natural language
- 🔒 Everything stays on your device — there is no server

## The "Why Now"

- AI models are good enough to be genuinely useful assistants
- OpenClaw is battle-tested (powers family office automation, multi-agent workflows)
- Apple Silicon makes local AI inference practical
- People are fed up with subscriptions and privacy erosion
- Nobody has built the "Ubuntu of AI" — the consumer-friendly layer on top of powerful infrastructure

## Positioning

**OpenClaw** is Linux. Power users, CLI, infinite flexibility.
**OpenPaw** is Ubuntu. Same engine, friendly face, anyone can use it.

*"OpenClaw for power users, OpenPaw for everyone."*

## Business Model

- **Free and open source** (MIT license) — always
- No subscription, no cloud, no data collection
- Revenue potential (future): hosted version, skill marketplace, enterprise support
- Designed to be acquisition-friendly (clean architecture, MIT license)

## The Team

Built entirely by AI agents. Zero human developers. One human founder/CEO (Meir Cohen) providing vision and decisions. 11 AI agents handle architecture, engineering, design, QA, docs, and DevOps.

**Current status:** Pre-development. 25 spec documents (397KB), full architecture, 12 ADRs, database schema, performance budget, security model, QA framework, and 12-week roadmap — all complete before writing a single line of code.

## Tech Stack

- **UI**: Swift + AppKit (native Mac)
- **Engine**: OpenClaw (embedded Node.js)
- **Storage**: SQLite + SQLCipher (encrypted)
- **Secrets**: macOS Keychain
- **Distribution**: Homebrew Cask + DMG
- **AI**: BYOK — any provider or local model

## Links

- **Dashboard**: [OpenPaw HQ](https://meircohen.github.io/openpaw-hq/)
- **GitHub**: github.com/openpaw (coming soon)
- **Landing Page**: openpaw.dev (coming soon)

## The One-Liner

**"A free, open-source AI assistant that handles your digital life. Runs on your Mac. Private by design. Zero setup."**

---

*🐾 AI that just works.*
