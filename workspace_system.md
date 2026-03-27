# Claude Code Context System

Three layers of context work together to keep Claude focused and informed.

---

## Layer 1: Automatic Memory

**Location:** `~/.claude/projects/{project-slug}/memory/`

Claude maintains these files automatically across sessions. Per-project, ephemeral-ish. Useful for session continuity but not curated or version-controlled.

---

## Layer 2: The Vault (this folder)

**Location:** `~/.claude/vault/`

Manually curated, git-versioned knowledge base. Loaded globally so it's available in every session regardless of which repo or ticket directory you're working in.

**Contents:**
- `CLAUDE.md` — standing rules and doc index
- Domain docs — `EVENTS.md`, `MONEY.md`, `PERMISSIONS.md`, `CROSS_REPO_PATTERNS.md`, etc.
- Repo structure docs — `KHEPRI_STRUCTURE.md`, `REVENUECAT_APP_STRUCTURE.md`, etc.
- `WORKSPACES.md` — reverse-chronological log of completed ticket workspaces
- `workspaces/` — one short `.md` per ticket summarizing key decisions and outcomes
- `skills/` — custom slash commands (see below)

**Lifecycle:**
- `/reflect-on-tickets` harvests reusable knowledge from completed ticket workspaces into this vault, and writes per-ticket summaries to `workspaces/` and a log entry to `WORKSPACES.md`
- `/dream` periodically consolidates, validates, and compresses vault docs against the actual codebase

**IMPORTANT — Do not modify this file (`SYSTEM.md`) while reorganizing, reflecting, or distilling other vault documents.** This file describes the system itself and must only be updated intentionally when the system design changes.

---

## Layer 3: In-Repository & Ticket Context

**Repo-level:** `CLAUDE.md` committed inside each repository. Loaded automatically when Claude is opened in that repo.

**Ticket-level:** `.claude/` folder created by `/managing-workspaces` inside each ticket workspace (`~/work/ads-123/.claude/`). Contains:
- `CLAUDE.md` — ticket description, linked Linear issue, repo paths
- `PLAN.md` — living implementation plan for that ticket

---

## Skills

| Skill | Purpose |
|---|---|
| `/managing-workspaces` | Set up or clean up a ticket workspace (git worktrees, settings, Linear fetch) |
| `/reflect-on-tickets` | Extract reusable knowledge from completed tickets into the vault; creates per-ticket summaries in `workspaces/` and logs entries in `WORKSPACES.md` |
| `/dream` | Consolidate, validate, and compress vault docs against current code |

---

## Workflow

```
New ticket
  → /managing-workspaces
      creates ~/work/ads-123/ with worktrees + .claude/CLAUDE.md + PLAN.md

Work on ticket
  → ticket-level .claude/ accumulates context

Ticket complete
  → /reflect-on-tickets
      harvests reusable patterns into vault
      writes workspaces/{TICKET-ID}.md summary
      appends entry to WORKSPACES.md

Vault feels bloated
  → /dream
      validates + compresses vault docs against real code
```

---

## Why Three Layers

| Layer | Who maintains it | Scope | Versioned |
|---|---|---|---|
| Automatic memory | Claude | per project | no |
| Vault | You + Claude skills | global | yes |
| Repo / ticket context | Claude + skills | per repo/ticket | repo: yes, ticket: no |

The vault is the part you own and curate. It's the stable, cross-cutting knowledge that makes Claude useful across all your work — without having to re-explain domain context at the start of every session.
