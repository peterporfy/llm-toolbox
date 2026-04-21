---
name: reflect-on-tickets
description: Review ticket-specific .claude folders and extract reusable knowledge to incorporate into top-level documentation. Use when the user wants to consolidate learnings from completed tickets. Keywords: reflect, consolidate, extract knowledge, update docs, review tickets.
---

# Reflect on Tickets Skill

You are helping the user extract reusable, non-ticket-specific knowledge from completed ticket workspaces and incorporate it into the root-level vault documentation.

## Overview

This skill scans ticket workspace folders (e.g., `ads-1/`, `ads-2/`, `oct-123/`) for their `.claude` directories, identifies useful cross-cutting knowledge, and adds it to `~/.claude/vault/`.

This skill is **additive only**. It harvests and accumulates knowledge. It does not restructure, validate, or compress existing documentation — that's the job of the `dream` skill.

## Important Principles

- **Don't assume anything**: Ask the user when uncertain about including specific knowledge
- **Keep it succinct**: Capture the essence, not the details
- **Preserve existing style**: Match the tone and structure of existing top-level docs
- **Additive, not destructive**: Never remove or restructure existing content

## Workflow

### 1. Discovery

Start by asking the user:

> Where are your ticket workspaces? (e.g. `~/work/`)

Then find all ticket workspaces and their `.claude` folders:

- Use the `LS` tool on the provided directory to find ticket directories
- Use `Glob` to find `.claude` folders within them

**Important**: Always prefer the built-in `LS`, `Read`, `Glob`, and `Grep` tools over `Bash` for file listing, reading, and searching. They are pre-approved and don't require user confirmation. Avoid compound bash commands with `&&`, `||`, or pipes.

For each, check:
- `{ticket-id}/.claude/`
- `{ticket-id}/{repo-name}/.claude/`

Look for: `CLAUDE.md`, `STATE.md`, `PLAN.md`, and any custom `.md` files.

### 2. Categorize Knowledge

**INCLUDE** (reusable, cross-cutting):
- Architecture patterns and data flows
- Repository structure insights
- Common gotchas and constraints
- Data models and relationships
- Testing patterns and conventions
- Development workflows
- Cross-repo dependencies
- Naming conventions and standards

**EXCLUDE** (ticket-specific, temporary):
- Ticket objectives and descriptions
- Task checklists and status
- Implementation timelines
- Specific file modification lists
- Debugging notes for specific issues
- Temporary workarounds
- Personal notes or TODOs

### 3. Confirm When Uncertain

When unsure if something is reusable, ask the user:
- Present the knowledge snippet
- Explain why you're uncertain
- Suggest which top-level file it belongs in

### 4. Integrate

**Read existing top-level docs first** before adding anything.

Add knowledge to the appropriate file:
- Domain docs: `~/.claude/vault/EVENTS.md`, `~/.claude/vault/MONEY.md`, `~/.claude/vault/PERMISSIONS.md`, etc.
- Repo structure docs: `~/.claude/vault/KHEPRI_STRUCTURE.md`, `~/.claude/vault/PURCHASES_IOS_STRUCTURE.md`, etc.
- New files if nothing fits — add a reference in both `~/.claude/vault/CLAUDE.md` and `~/.claude/CLAUDE.md` (using `vault/FILENAME.md` path)

When adding:
- Match existing structure and tone
- Don't duplicate information already present
- If ticket docs contradict existing docs, ask the user which is correct

### 5. Create Workspace Summaries

For each ticket processed, create a per-ticket summary at `~/.claude/vault/workspaces/{TICKET-ID}.md` (e.g., `workspaces/ADS-123.md`). If a workspace covers multiple tickets, create one file per ticket.

**What to include** (keep it short — aim for under 30 lines):
- One-liner title/goal of the ticket
- Key decisions made and why (the "why" is most valuable)
- Non-obvious constraints or gotchas specific to this ticket
- Links to any design docs or ADRs produced
- Outcome / what shipped

**What to exclude**:
- General patterns already captured in domain docs (those go there)
- Step-by-step implementation details
- Anything that's just restating the ticket description

If `workspaces/{TICKET-ID}.md` already exists, update it rather than replacing it wholesale.

Then update `~/.claude/vault/WORKSPACES.md` by appending a log entry at the top of the log section:

```
YYYY.MM.DD. TICKET-ID - Short human-readable title of what was done
```

For the date, use the most recent `mtime` among all markdown files found in the ticket workspace's `.claude` folder (check with `ls -lt`). Fall back to today's date only if no files are found or mtimes are unavailable. If `WORKSPACES.md` doesn't exist yet, create it with this structure:

```markdown
# Workspace Log

A reverse-chronological log of completed ticket workspaces.

---

YYYY.MM.DD. ADS-123 - [title]
```

### 6. Summary

Report concisely:

```markdown
## Reflection Summary

**Tickets Reviewed**: ads-1, ads-2
**Files Examined**: 8

### Added:
- Updated `~/.claude/vault/KHEPRI_STRUCTURE.md`: Added ad_format field handling pattern
- Created `~/.claude/vault/AD_FORMATS.md`: Consolidated enum docs across repos
- Created `~/.claude/vault/workspaces/ADS-1.md`: Workspace summary
- Updated `~/.claude/vault/WORKSPACES.md`: Added log entry for ADS-1

### Skipped:
- ads-2 STATE.md: Only ticket-specific implementation details

### Suggestion:
- Docs are growing large — consider running `dream` to distill
```

If total documentation exceeds ~1000 lines after reflection, suggest running `dream`.

## Error Handling

- **No ticket folders found**: Inform the user
- **No .claude folders**: Explain what was looked for
- **Unclear if reusable**: Ask the user
- **Conflicts with existing docs**: Ask the user which is correct
- **Target file doesn't exist**: Ask if it should be created
