# OpenPaw — Performance Budget

## Principle
OpenPaw should feel native. If it feels slower than a regular Mac app, we've failed.

## Hard Limits

| Metric | Target | Maximum | Measurement |
|--------|--------|---------|-------------|
| App launch to usable | < 2s | 3s | Time from dock click to chat input ready |
| Cold start (first ever) | < 5s | 8s | Includes OpenClaw engine boot |
| Message response start | < 1s | 2s | Time from send to first token appearing |
| Full response complete | < 10s | 30s | Depends on AI provider, but UI should stream |
| Approval dialog appear | < 500ms | 1s | Agent requests approval → dialog visible |
| Settings save | < 200ms | 500ms | User changes setting → confirmed saved |
| Service connection (OAuth) | < 3s | 5s | Click connect → browser opens |
| Memory usage (idle) | < 150MB | 250MB | App + OpenClaw engine, no active tasks |
| Memory usage (active) | < 300MB | 500MB | During task execution |
| CPU usage (idle) | < 2% | 5% | No active tasks, heartbeat only |
| CPU usage (active) | < 30% | 50% | During task execution |
| Battery impact | < 5% | 10% | Per hour of background operation |
| Disk usage (app) | < 200MB | 300MB | App bundle including Node.js + OpenClaw |
| Disk usage (data) | < 100MB | 500MB | After 6 months of normal use |

## Streaming Requirement
AI responses MUST stream to the UI token-by-token. Users should never stare at a blank screen waiting. Show partial response immediately.

## Background Operation
When user isn't actively chatting:
- OpenClaw heartbeat every 30min (same as current setup)
- CPU should be near-zero between heartbeats
- No continuous polling, no busy loops
- Wake on schedule, do work, sleep

## Startup Optimization
1. Show app window immediately (< 200ms) with loading indicator
2. Boot OpenClaw engine in background
3. Chat becomes interactive when engine ready
4. Preload last conversation while engine boots
5. Never block the UI thread

## Measurement
- Built-in performance profiling (opt-in developer mode)
- Track: launch time, response latency, memory high-water mark
- Log to local file for debugging, never sent externally
- CI performance regression tests: fail build if launch > 3s or idle memory > 250MB
