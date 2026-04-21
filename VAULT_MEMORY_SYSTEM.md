# Vault Memory System

The vault is a manually curated knowledge base loaded globally into every Claude session.

**Location:** `~/.claude/vault/` (symlink to your cloud storage)

---

## What it is

A structured set of markdown docs covering cross-cutting knowledge — architecture, domain patterns, repo structure, conventions — that would otherwise need to be re-explained at the start of every session.

**Contents:**
- `CLAUDE.md` — doc index
- Domain docs — cross-cutting knowledge about your systems and architecture
- `WORKSPACES.md` — reverse-chronological log of completed ticket workspaces
- `workspaces/` — one short `.md` per ticket summarizing key decisions and outcomes

---

## How it stays up to date

| Skill | Role |
|---|---|
| `/reflect-on-tickets` | Harvests reusable knowledge from completed ticket workspaces into the vault |
| `/dream` | Periodically consolidates, validates, and compresses vault docs against actual code |

**Lifecycle:**

```
Ticket complete
  → /reflect-on-tickets
      extracts reusable patterns into vault docs
      writes workspaces/{TICKET-ID}.md summary
      appends entry to WORKSPACES.md

Vault feels bloated or stale
  → /dream
      validates factual claims against current code
      consolidates and compresses docs
```

---

## Why it works

The vault is the part you own and curate. It's the stable, cross-cutting knowledge that makes Claude useful across all your work — without having to re-explain domain context at the start of every session.

*Update this file intentionally when the vault system design changes.*
