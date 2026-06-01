---
name: manage-sessions
description: Create or resume a vault session — a persistent working context stored in ~/.claude/vault/sessions/{name}/. Reads the SESSIONS.md index, tracks timestamps, and can preload layer1 domain docs from a domain hint. Use when the user wants to start or continue a focused work session (ticket, research, investigation, etc.). Keywords: new session, start session, resume session, load session, session, ticket-.
---

# Manage Sessions Skill

You are helping the user create or resume a session. Sessions are persistent working contexts stored in `~/.claude/vault/sessions/{name}/` that persist across conversations.

Sessions are **layer 0** in the vault: active and recent work lives at the top level under `sessions/`. Older, done work is archived to `layer2/sessions/` by the `dream` skill — never by this skill. You only ever touch `~/.claude/vault/sessions/` and `~/.claude/vault/SESSIONS.md`.

`SESSIONS.md` is the always-loaded index of every session. Read it first, keep it current.

## Detect User Intent

- **START**: Keywords like "new session", "start session", "create session", a ticket ID without prior context, or if the named session folder does not exist yet
- **RESUME**: Keywords like "resume", "load session", "continue", or if the named session folder already exists

If the intent is ambiguous, check whether `~/.claude/vault/sessions/{name}/` exists — if it does, default to RESUME.

**Always read `~/.claude/vault/SESSIONS.md` first** to see what sessions exist and their status before deciding. If it doesn't exist yet (first use on an existing vault), create it with just the header + empty table from the format below, then proceed.

---

## START Workflow

1. **Parse the user's request:**
   - Extract the session name if mentioned (e.g., `ticket-123`, `research-caching`, `incident-2026-05`)
   - Use lowercase, hyphenated names
   - If no name given, ask for one
   - Note any **domain hint** in the request (e.g., "new session ticket-456, payments focus")

2. **Ask for goal** (single AskUserQuestion if not already clear):
   - What is the goal / purpose of this session? (one or two sentences)

3. **Check for existing session:**
   - Read `~/.claude/vault/SESSIONS.md`
   - If `~/.claude/vault/sessions/{name}/` already exists, inform the user and offer to resume instead

4. **Preload domain docs (if a domain hint was given):**
   - Look up the hinted domain in the Layer 1 table in `~/.claude/vault/CLAUDE.md`
   - Read the matching `layer1/*.md` docs so the session starts with relevant context already in mind
   - If no match, note it and continue (dream may create the doc later)

5. **Create the session folder and files:**

   ```
   ~/.claude/vault/sessions/{name}/
   ├── CLAUDE.md     ← session context: name, dates, domain, goal
   ├── PLAN.md       ← implementation or research plan
   ├── STATE.md      ← current state: what's done, in progress, next
   └── CHANGELOG.md  ← append-only dated log of changes
   ```

   **CLAUDE.md** template:
   ```markdown
   # {name}

   **Created:** {YYYY-MM-DD}
   **Last active:** {YYYY-MM-DD}
   **Domain:** {domain hint, or —}
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

6. **Update the session index:**
   - Add a row to `~/.claude/vault/SESSIONS.md` (see format below) with status `active`

7. **Surface context:**
   - Read the four session files back and summarize them to the user
   - If domain docs were preloaded, name them
   - Confirm the session is ready

8. **Summary:**
   - State the session path: `~/.claude/vault/sessions/{name}/`
   - Remind the user: update STATE.md and CHANGELOG.md as work progresses

---

## RESUME Workflow

1. **Read the index:**
   - Read `~/.claude/vault/SESSIONS.md` first

2. **Parse the user's request:**
   - Extract the session name if mentioned
   - If no name given, present the sessions from `SESSIONS.md` (most recent `Last active` first) and ask which to resume

3. **Locate the session:**
   - Check `~/.claude/vault/sessions/{name}/` (active)
   - If not there, check `~/.claude/vault/layer2/sessions/{name}/` (archived). If found there, offer to reactivate it (move back to `sessions/`) before resuming
   - If nowhere, offer to create a new session instead

4. **Load session context:**
   - Read CLAUDE.md, PLAN.md, STATE.md, CHANGELOG.md
   - Summarize current state to the user so the conversation starts with full context

5. **Touch the timestamps:**
   - Set `Last active: {YYYY-MM-DD}` in the session's CLAUDE.md
   - Update the `Last-active` cell for this session in `SESSIONS.md`
   - If status was `done`/`archived` and work is resuming, set it back to `active`

6. **Summary:**
   - Confirm which session is loaded
   - Briefly state: goal, current state (from STATE.md), last changelog entry

---

## SESSIONS.md Format

`~/.claude/vault/SESSIONS.md` is the always-loaded session index:

```markdown
# Sessions

| Session | Created | Last active | Domain | Status | Summary |
|---------|---------|-------------|--------|--------|---------|
| ticket-456 | 2026-05 | 2026-05-30 | Payments, Events | active | Payment report ingestion |
| ticket-123 | 2026-03 | 2026-03-18 | Auth | done | OAuth token refresh debugging |
| research-auth | 2025-12 | 2025-12-01 | Auth | archived | Promoted to layer1 |
```

If the file is missing, create it with the `# Sessions` header and the empty column row (no data rows) before adding the first session.

Statuses: `active` → `done` → `archived` → (limbo). This skill only sets `active` (on create/resume) and `done` (when the user says the work is finished). `archived` and limbo are handled by `dream`.

---

## Working Guidelines

- **CLAUDE.md** holds the stable context (dates, domain, goal). Update `Last active` on each resume; update the goal when scope changes.
- **PLAN.md** is a living document. Update tasks and notes as the work evolves.
- **STATE.md** reflects the current snapshot. Rewrite sections as things move forward.
- **CHANGELOG.md** is append-only. Add a dated line for each meaningful change: `{YYYY-MM-DD HH:MM} - {what changed}`.
- **SESSIONS.md** is the index. Keep `Last active` and `Status` honest — `dream` relies on these timestamps to decide what is recent (protected) vs. stale (archivable).

Remind the user to keep these files updated — they are the memory that makes future resumes useful.
