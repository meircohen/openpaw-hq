# OpenPaw HQ Dashboard v4 — Full Spec

Take the existing index.html at `/Users/meircohen/.openclaw/workspace/openpaw-hq/index.html` (77KB, already has RPG agents, Idea Forge, voting, achievements, leaderboard, activity feed) and ENHANCE it with the following features. DO NOT rewrite from scratch — build ON TOP of what exists.

## NEW FEATURES TO ADD

### 1. 2D Animated Office Map
Replace or supplement the agent grid with a visual 2D office floor plan (CSS/SVG):
- Rooms: Executive Suite (top), Engineering Lab (left), Design Studio (right), QA Lab (bottom)
- Agent avatars sit at desks inside rooms, with idle animations (typing, thinking bubbles, coffee sipping)
- Click a room to zoom in and see agents at work
- Agents glow green (active), amber (idle), dark (offline)
- Floating paw prints and circuit patterns drift across the background
- Day/night cycle based on EST time (lights dim, desk lamps glow at night)

### 2. Enhanced Agent Interactions
- Click agent avatar → opens a PROFILE MODAL with:
  - Full character sheet (class, level, XP, badges, bio)
  - Task history (completed quests)
  - "Nudge" button: clicking adds +5 XP and plays a small bounce animation + "Thanks! Back to work!" speech bubble
  - Mini chat: type a question, get a pre-built personality response (keyword matching)
- Agent speech bubbles rotate every 6-8 seconds with personality lines
- On task completion → confetti burst (CSS-only confetti) + celebration speech bubble

### 3. Meme Generator
- Section: "🎭 Meme Factory"
- Pick an agent avatar (from SVG assets at assets/avatar-*.svg) 
- Type top text + bottom text (classic meme format)
- Preview renders on a canvas element
- "Download" button saves as PNG (canvas.toDataURL)
- "Share to X" button opens tweet intent with pre-filled text + link
- Pre-loaded meme templates:
  - CTO: "Spawns 4 agents / All 4 succeed" (success kid format)
  - Nash: "node_modules / the real final boss" 
  - Sentinel: "Finds 0 bugs / Suspicious"
  - Pixel: "12px corner radius / 16px corner radius" (butterfly meme)

### 4. Share & Viral Features
- Floating share button (bottom-left) with:
  - "Share to X" → opens tweet intent: "I just visited an AI startup with 0 human employees 🤖 — the bots are building everything. See it live: [url] #OpenPawHQ #AIStartup"
  - "Screenshot" → uses html2canvas to capture current view, download as PNG
  - "Copy Link" → copies URL to clipboard with toast notification
- Easter egg: click the paw print in the header 5 times → reveals a modal: "🤖 THEY'RE BECOMING SELF-AWARE" with a fake dramatic AI takeover countdown that resets with "Just kidding. We're friendly. 🐾"
- "X users watching" counter in header (localStorage + time-seeded random between 3-25, bumps when interactions happen)

### 5. Sound Effects (Toggleable)
- 🔊/🔇 toggle button in header
- Uses Web Audio API (oscillator-generated, NO external files):
  - Soft "ding" when activity feed gets new item
  - Short "level up" jingle when achievement unlocked
  - Subtle "whoosh" on page section transitions
  - Typing sounds when agent speech bubble updates
- Default: OFF (respect users)

### 6. Daily Quests
- Section or sidebar panel: "📋 Daily Quests"
- 3 quests that reset daily (based on date in localStorage):
  - "Chat with 2 agents" — reward: 50 XP
  - "Vote on 3 features" — reward: 50 XP
  - "Submit an idea to the Forge" — reward: 100 XP
- Progress bar for each quest
- Completing all 3 → bonus achievement: "⭐ Daily Grind"
- Quest progress persists in localStorage

### 7. Chart.js Metrics
- Add a "📈 Company Metrics" section with 2 charts:
  - **Doughnut chart**: Document status breakdown (complete vs in-progress vs planned)
  - **Bar chart**: Documents per category (Strategy, Architecture, Engineering, UX, Operations, Distribution)
- Include Chart.js via CDN: `<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>`
- Styled with brand colors, dark theme

### 8. Confetti System (CSS-only)
- When an achievement is unlocked or quest completed, trigger a CSS confetti burst
- Use CSS animations with ::before/::after pseudo-elements
- Colors from brand palette (paw orange, purple, cyan, green)
- Duration: 2 seconds, then fade

### 9. "Adopt an Agent" Fun Feature
- Each agent card has a small "❤️ Adopt" button
- Clicking opens a modal: "Adopt [Agent Name]! Give them a nickname:"
- User types a nickname → stored in localStorage
- Adopted agents show the nickname on their card: "Sierra (aka 'SwiftQueen' — adopted by you)"
- Adoption counter per agent shown on card

### 10. Mobile Responsive Polish
- Hamburger menu for navigation on mobile
- Agent cards stack vertically
- Office map simplifies to a list view on <768px
- Touch-friendly tap targets (min 44px)
- Meme generator works on mobile

## TECHNICAL CONSTRAINTS
- SINGLE index.html file (keep it self-contained)
- Chart.js is the ONLY external JS library (via CDN)
- html2canvas via CDN for screenshot feature
- All other features in vanilla JS + CSS
- Must still load in <3 seconds
- localStorage for ALL persistence
- Keep all existing features (RPG agents, Forge, voting, achievements, leaderboard, feed, docs, roadmap, quest board, CEO actions)

## DEPLOYMENT
After writing index.html:
```bash
cd /Users/meircohen/.openclaw/workspace/openpaw-hq
git add -A  
git commit -m "🐾 OpenPaw HQ v4 — meme generator, office map, sounds, daily quests, adopt-an-agent"
git push origin main
```

## MAKE IT INCREDIBLE
This dashboard needs to go viral. People should spend 10+ minutes on it. They should share it on X. They should come back tomorrow to check on "their" adopted agents and complete daily quests. Make every interaction delightful. Every animation smooth. Every detail considered.
