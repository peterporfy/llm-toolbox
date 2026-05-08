# Vault Memory System

The vault is a manually curated knowledge base loaded globally into every Claude session.

**Location:** `~/.claude/vault/` (symlink to your cloud storage)

---

## What it is

A structured set of markdown docs covering cross-cutting knowledge — architecture, domain patterns, repo structure, conventions — that would otherwise need to be re-explained at the start of every session.

**Contents:**
- `CLAUDE.md` — doc index
- Domain docs — cross-cutting knowledge about your systems and architecture
- `sessions/` — active working contexts (one folder per session, created by `/manage-sessions`)
- `workspaces/` — _(legacy)_ one short `.md` per completed ticket summarizing key decisions and outcomes
- `WORKSPACES.md` — _(legacy)_ reverse-chronological log of completed ticket workspaces

---

## How it stays up to date

| Skill | Role |
|---|---|
| `/manage-sessions` | Creates and resumes persistent working contexts inside `vault/sessions/` |
| `/dream` | Periodically consolidates vault docs: promotes cross-cutting knowledge from sessions and subdirs into top-level docs, validates against code, compresses |
| `/reflect-on-tickets` | _(legacy)_ Harvests reusable knowledge from completed git worktree workspaces into the vault |

**Current lifecycle (sessions-based):**

```
Start work
  → /manage-sessions start {name}
      creates vault/sessions/{name}/ with CLAUDE.md, PLAN.md, STATE.md, CHANGELOG.md
      loads context into conversation

Resume work
  → /manage-sessions resume {name}
      reads session files and surfaces context

Vault feels bloated or stale
  → /dream
      reads all vault subdirs (including sessions/)
      promotes cross-cutting knowledge into top-level docs
      validates factual claims against current code
      consolidates and compresses docs
      flags stale sessions (untouched 1+ month) for deletion after approval
```

**Legacy lifecycle (git worktree-based):**

```
Ticket complete
  → /reflect-on-tickets
      extracts reusable patterns into vault docs
      writes workspaces/{TICKET-ID}.md summary
      appends entry to WORKSPACES.md
```

---

## Why it works

The vault is the part you own and curate. It's the stable, cross-cutting knowledge that makes Claude useful across all your work — without having to re-explain domain context at the start of every session.

*Update this file intentionally when the vault system design changes.*
