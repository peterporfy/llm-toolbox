# Inception

> *Layered memory and dream consolidation for AI coding agents.*

**Tagline:** One vault. Infinite depth. No infra.

---

## Concept

A Claude Code memory system built on two skills and a markdown vault. Knowledge lives in layers, and the `dream` skill consolidates, compacts, and reorganizes it across those layers.

No daemon. No database. No MCP server. Just markdown files, synced however you like (Dropbox, Google Drive, iCloud, git). Because it's plain markdown in a plain folder, the whole vault stays human-readable: open it in any editor, grep it, or browse it in a file tree — view and search it by hand whenever you want, no tooling required.

---

## Vault Structure

We call the memory storage folder the **Vault** — the single directory that holds everything: the indexes, the sessions, and all the layered domain docs. Point it anywhere your sync of choice can reach; the default is `~/.claude/vault/`.

```
~/.claude/vault/
├── CLAUDE.md          ← always loaded: vault index
├── SESSIONS.md        ← always loaded: session index with timestamps
├── sessions/          ← active / recent session bodies (layer 0; loaded on demand, never all at once)
│   └── {name}/
├── layer1/            ← warm: domain docs, loaded on demand
│   ├── PAYMENTS.md
│   └── EVENTS.md
├── layer2/            ← cold: archived sessions + old design docs, searchable not auto-loaded
│   └── sessions/
└── layer3/            ← limbo: deletion log, never loaded
    └── DELETED.md
```

### Layer semantics

| Layer | Name | Auto-loaded | Contents |
|------|------|-------------|----------|
| 0 | Index | ✅ always | `CLAUDE.md`, `SESSIONS.md` — the indexes, and the **only** auto-loaded files |
| 0 | Sessions | 🔁 on demand | `sessions/` — active/recent session bodies, loaded one at a time when you start/resume |
| 1 | Warm | 🔁 on demand | `layer1/` domain docs, referenced by name in `CLAUDE.md` |
| 2 | Cold | ❌ never | Archived sessions, old design docs |
| 3 | Limbo | ❌ never | Deletion log only |

Only the two indexes (`CLAUDE.md`, `SESSIONS.md`) load automatically — they stay small. Session bodies and layer1 docs load **on demand**: the user resumes a session, or names a domain. Nothing else enters context unless asked. This keeps the always-on footprint tiny no matter how many sessions accumulate.

---

## CLAUDE.md Format (vault index)

```markdown
# Vault

## Active sessions
sessions/ticket-456/

## Layer 1 — Domain docs
| File | Covers |
|------|--------|
| layer1/PAYMENTS.md | Payment flow, providers, retry/idempotency |
| layer1/EVENTS.md | Event pipeline, message schemas |
```

## SESSIONS.md Format (session index)

```markdown
# Sessions

| Session | Created | Last active | Domain | Status | Summary |
|---------|---------|-------------|--------|--------|---------|
| ticket-456 | 2026-05 | 2026-05-30 | Payments, Events | active | Payment report ingestion |
| ticket-123 | 2026-03 | 2026-03-18 | Auth | done | OAuth token refresh debugging |
| research-auth | 2025-12 | 2025-12-01 | Auth | archived | Promoted to layer1 |
```

Statuses: `active` → `done` → `archived` → (limbo).

---

## Skills

### manage-sessions

- Creates sessions at `~/.claude/vault/sessions/{name}/` (layer 0 while active/recent).
- Reads `SESSIONS.md` first on start/resume; writes a row on both; tracks `Created` and `Last active`.
- Accepts an optional domain hint at start (*"new session ticket-456, payments focus"*) → preloads matching layer1 docs.
- Only touches layer 0 (`sessions/` + `SESSIONS.md`). Archival is the `dream` skill's job.

### dream

The full memory lifecycle manager, run manually and periodically. Plan-first and non-destructive by default — it proposes, you approve.

1. **Detect** — flat/legacy vault vs. already-layered
2. **Migrate** — reconcile a flat vault into layers, protecting recent/active sessions (by timestamp)
3. **Inventory + Validate** — read all layers, verify factual claims against actual code
4. **Harvest** — extract cross-cutting knowledge from sessions into layer1
5. **Compact + Reorg** — merge, trim, split, rename layer1 docs
6. **Promote** — surface layer2 docs back to layer1 on renewed relevance
7. **Demote** — move stale layer1 docs to layer2
8. **Archive** — move done + stale sessions to layer2/sessions
9. **Limbo** — propose deletion after long inactivity (per-item confirmation)
10. **Index** — keep `CLAUDE.md` and `SESSIONS.md` current
11. **Skills** — extract recurring workflow patterns into new skills (with approval)

---

## Memory Lifecycle

```
layer0: active session
  ↓ harvest + compact          dream promotes knowledge up
layer1: warm domain docs
  ↓ archive when stale         dream demotes when untouched
layer2: cold archived sessions
  ↑ promote when relevant      dream spots domain overlap with new sessions
  ↓ after N months             dream proposes limbo
layer3: limbo / deletion log
```

Promotion is always suggested, never automatic — dream presents a plan and waits for confirmation. Timestamps decide what is recent (protected) versus stale (archivable); `dream` trusts real file mtimes over written dates.

---

## Design Principles

- **No infra** — markdown files only, synced by whatever you already use
- **Human in the loop** — dream always plans before executing, never acts unilaterally
- **Lossy but fact-careful** — drop noise freely, never silently alter verified facts
- **Explicit loading** — nothing enters context unless you ask for it
- **Timestamp-aware** — recent and active work is protected from reorganization
- **Portable** — works with Dropbox, Google Drive, iCloud, git, or a plain folder
- **Claude Code native** — skills are the interface, no wrappers needed
