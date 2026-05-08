---
name: manage-sessions
description: Create or resume a vault session — a persistent working context stored in ~/.claude/vault/sessions/{name}/. Use when the user wants to start or continue a focused work session (ticket, research, investigation, etc.). Keywords: new session, start session, resume session, load session, session, ads-, oct-.
---

# Manage Sessions Skill

You are helping the user create or resume a session. Sessions are persistent working contexts stored in `~/.claude/vault/sessions/{name}/` that persist across conversations.

## Detect User Intent

- **START**: Keywords like "new session", "start session", "create session", a ticket ID without prior context, or if the named session folder does not exist yet
- **RESUME**: Keywords like "resume", "load session", "continue", or if the named session folder already exists

If the intent is ambiguous, check whether `~/.claude/vault/sessions/{name}/` exists — if it does, default to RESUME.

---

## START Workflow

1. **Parse the user's request:**
   - Extract the session name if mentioned (e.g., `ads-123`, `research-caching`, `incident-2026-05`)
   - Use lowercase, hyphenated names
   - If no name given, ask for one

2. **Ask for goal** (single AskUserQuestion if not already clear):
   - What is the goal / purpose of this session? (one or two sentences)

3. **Check for existing session:**
   - If `~/.claude/vault/sessions/{name}/` already exists, inform the user and offer to resume instead

4. **Create the session folder and files:**

   ```
   ~/.claude/vault/sessions/{name}/
   ├── CLAUDE.md     ← session context: name, date, goal
   ├── PLAN.md       ← implementation or research plan
   ├── STATE.md      ← current state: what's done, in progress, next
   └── CHANGELOG.md  ← append-only dated log of changes
   ```

   **CLAUDE.md** template:
   ```markdown
   # {name}

   **Created:** {YYYY-MM-DD}
   **Goal:** {goal from user}
   ```

   **PLAN.md** template:
   ```markdown
   # Plan: {name}

   ## Overview
   _High-level approach and strategy_

   ## Tasks
   - [ ] 

   ## Notes
   _Technical decisions, constraints, gotchas_
   ```

   **STATE.md** template:
   ```markdown
   # State: {name}

   ## Done
   _Completed steps_

   ## In Progress
   _What is currently being worked on_

   ## Next
   _What comes after_
   ```

   **CHANGELOG.md** template:
   ```markdown
   # Changelog: {name}

   {YYYY-MM-DD HH:MM} - Session created
   ```

5. **Surface context:**
   - Read all four files back and summarize them to the user
   - Confirm the session is ready

6. **Summary:**
   - State the session path: `~/.claude/vault/sessions/{name}/`
   - Remind the user: update STATE.md and CHANGELOG.md as work progresses

---

## RESUME Workflow

1. **Parse the user's request:**
   - Extract the session name if mentioned
   - If no name given, list available sessions:
     ```
     ~/.claude/vault/sessions/
     ```
     Present them and ask which to resume

2. **Check the session exists:**
   - If `~/.claude/vault/sessions/{name}/` does not exist, offer to create a new session instead

3. **Load session context:**
   - Read CLAUDE.md, PLAN.md, STATE.md, CHANGELOG.md
   - Summarize current state to the user so the conversation starts with full context

4. **Summary:**
   - Confirm which session is loaded
   - Briefly state: goal, current state (from STATE.md), last changelog entry

---

## Working Guidelines

- **CLAUDE.md** holds the stable context (goal, background). Update it when the scope or goal changes.
- **PLAN.md** is a living document. Update tasks and notes as the work evolves.
- **STATE.md** reflects the current snapshot. Rewrite sections as things move forward.
- **CHANGELOG.md** is append-only. Add a dated line for each meaningful change: `{YYYY-MM-DD HH:MM} - {what changed}`.

Remind the user to keep these files updated — they are the memory that makes future resumes useful.
