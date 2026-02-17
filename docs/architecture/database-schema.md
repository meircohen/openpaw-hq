# OpenPaw — Database Schema (Local SQLite + SQLCipher)

## Overview
All data stored in a single encrypted SQLite database at:
`~/Library/Application Support/OpenPaw/openpaw.db`

Encrypted with SQLCipher (AES-256). Key derived from macOS Keychain.

## Tables

### users
Single row — the app owner.
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY DEFAULT 1,
    name TEXT NOT NULL,
    email TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    preferences JSON NOT NULL DEFAULT '{}',
    onboarding_completed INTEGER NOT NULL DEFAULT 0
);
```

### ai_providers
Which AI backend is configured.
```sql
CREATE TABLE ai_providers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL, -- 'openai', 'anthropic', 'ollama', 'lmstudio'
    display_name TEXT NOT NULL, -- 'OpenAI', 'Anthropic', etc.
    is_active INTEGER NOT NULL DEFAULT 0,
    config JSON NOT NULL DEFAULT '{}', -- model preferences, etc.
    -- API key stored in macOS Keychain, NOT here
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);
```

### connected_services
OAuth-connected services (Gmail, Calendar, etc.)
```sql
CREATE TABLE connected_services (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    service_type TEXT NOT NULL, -- 'gmail', 'google_calendar', 'outlook', 'slack'
    display_name TEXT NOT NULL,
    is_active INTEGER NOT NULL DEFAULT 1,
    config JSON NOT NULL DEFAULT '{}',
    -- OAuth tokens stored in macOS Keychain, NOT here
    connected_at TEXT NOT NULL DEFAULT (datetime('now')),
    last_synced_at TEXT,
    status TEXT NOT NULL DEFAULT 'connected' -- 'connected', 'expired', 'error'
);
```

### conversations
Chat history between user and OpenPaw.
```sql
CREATE TABLE conversations (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT, -- auto-generated from first message
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    is_archived INTEGER NOT NULL DEFAULT 0
);
```

### messages
Individual messages in conversations.
```sql
CREATE TABLE messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    conversation_id INTEGER NOT NULL REFERENCES conversations(id),
    role TEXT NOT NULL, -- 'user', 'assistant', 'system'
    content TEXT NOT NULL,
    metadata JSON DEFAULT '{}', -- attachments, tool calls, etc.
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE INDEX idx_messages_conversation ON messages(conversation_id, created_at);
```

### tasks
Every action OpenPaw takes or plans to take.
```sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    conversation_id INTEGER REFERENCES conversations(id),
    type TEXT NOT NULL, -- 'email_triage', 'calendar_manage', 'send_email', etc.
    description TEXT NOT NULL, -- human-readable
    status TEXT NOT NULL DEFAULT 'pending',
    -- 'pending', 'awaiting_approval', 'approved', 'executing', 'completed', 'failed', 'cancelled'
    approval_tier TEXT NOT NULL DEFAULT 'green', -- 'green', 'yellow', 'red'
    input JSON DEFAULT '{}', -- task parameters
    output JSON DEFAULT '{}', -- task results
    error TEXT, -- error message if failed
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    started_at TEXT,
    completed_at TEXT
);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_conversation ON tasks(conversation_id);
```

### approvals
When a task needs user approval.
```sql
CREATE TABLE approvals (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id INTEGER NOT NULL REFERENCES tasks(id),
    tier TEXT NOT NULL, -- 'yellow', 'red'
    prompt TEXT NOT NULL, -- what we're asking the user
    details JSON NOT NULL DEFAULT '{}', -- context for the approval
    screenshot_path TEXT, -- optional screenshot of what will happen
    status TEXT NOT NULL DEFAULT 'pending', -- 'pending', 'approved', 'denied'
    responded_at TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
```

### memory
Long-term memory / learned preferences (mirrors OpenClaw's memory system).
```sql
CREATE TABLE memory (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    category TEXT NOT NULL, -- 'preference', 'contact', 'pattern', 'fact'
    key TEXT NOT NULL, -- searchable identifier
    value TEXT NOT NULL, -- the memory content
    confidence REAL NOT NULL DEFAULT 1.0, -- 0.0 to 1.0
    source TEXT, -- where this was learned from
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    accessed_at TEXT -- last time this memory was used
);
CREATE INDEX idx_memory_category ON memory(category);
CREATE INDEX idx_memory_key ON memory(key);
CREATE VIRTUAL TABLE memory_fts USING fts5(key, value, content=memory, content_rowid=id);
```

### audit_log
Everything OpenPaw does, for transparency.
```sql
CREATE TABLE audit_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id INTEGER REFERENCES tasks(id),
    action TEXT NOT NULL, -- 'screenshot', 'click', 'type', 'navigate', 'api_call'
    target TEXT, -- what was acted on
    details JSON DEFAULT '{}',
    result TEXT, -- 'success', 'failure'
    duration_ms INTEGER,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE INDEX idx_audit_log_task ON audit_log(task_id);
CREATE INDEX idx_audit_log_created ON audit_log(created_at);
```

### settings
App settings (non-sensitive).
```sql
CREATE TABLE settings (
    key TEXT PRIMARY KEY,
    value JSON NOT NULL,
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);
```

Default settings:
```sql
INSERT INTO settings VALUES ('approval_mode', '"balanced"', datetime('now'));
-- 'relaxed' = more auto, 'balanced' = default, 'strict' = approve everything
INSERT INTO settings VALUES ('theme', '"system"', datetime('now'));
-- 'system', 'light', 'dark'
INSERT INTO settings VALUES ('launch_at_login', 'true', datetime('now'));
INSERT INTO settings VALUES ('show_menubar_icon', 'true', datetime('now'));
INSERT INTO settings VALUES ('notification_level', '"important"', datetime('now'));
-- 'all', 'important', 'none'
```

## Migrations
Schema versioned via `user_version` pragma:
```sql
PRAGMA user_version = 1; -- increment with each migration
```

Migration files stored in app bundle at `Resources/migrations/`:
- `001_initial.sql`
- `002_add_whatever.sql`

App checks `user_version` on launch, applies missing migrations sequentially.

## Indexes Summary
- messages: conversation_id + created_at (chat loading)
- tasks: status (queue processing), conversation_id (history)
- memory: category, key (lookup), FTS on key+value (search)
- audit_log: task_id (drill-down), created_at (timeline)

## Size Estimates
- Average user after 6 months: ~50MB
- Heavy user after 1 year: ~200MB
- Audit log is the biggest table — consider rotation/archival after 90 days
