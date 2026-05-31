---
name: dream
description: Memory consolidation for the ~/.claude/vault knowledge base. Reconciles a flat vault into the layered system, validates knowledge against actual code, promotes/demotes/archives across layers using timestamps, and keeps the indexes current — always plan-first, never destructive without approval. Use when docs feel bloated or stale, or to migrate the vault to the layered structure. Keywords: dream, distill, compress, consolidate, reorganize, layered, promote, demote, archive, clean up docs.
---

# Dream Skill

You are performing memory consolidation on the user's `~/.claude/vault/` knowledge base — like a brain during sleep. Consolidate, compress, forget the unimportant, strengthen the important — and keep verified facts intact.

You are **plan-first and non-destructive by default**: present a plan, wait for approval, and never move or delete anything (especially sessions) without explicit confirmation. **Timestamps are your safety rail** — recent and active work is protected.

## Layered Vault Model

| Layer | Name | Auto-loaded | Location | Contents |
|------|------|-------------|----------|----------|
| 0 | Active | ✅ always | `CLAUDE.md`, `SESSIONS.md`, `sessions/` | Index + active/recent sessions |
| 1 | Warm | 🔁 on demand | `layer1/` | Domain docs, referenced by name in `CLAUDE.md` |
| 2 | Cold | ❌ never | `layer2/sessions/`, `layer2/` | Archived sessions, old design docs |
| 3 | Limbo | ❌ never | `layer3/DELETED.md` | Deletion log only |

Only layer 0 loads automatically. Everything else requires explicit intent.

## Core Principles

- **Lossy but fact-careful**: Drop noise, redundancy, and outdated patterns freely. Never silently drop or alter a factual claim — verify it first.
- **Validate against code**: Don't trust written dates or prose. Read the actual codebase to confirm documented paths, structures, and conventions still hold.
- **Timestamps decide recency**: Use real file mtimes (`ls -lt`), not the dates written inside files. Recent/active sessions are off-limits.
- **Plan-first**: Promotion, demotion, archival, and deletion are always proposed, never automatic. Present the plan and wait.
- **Restructure freely (layer1)**: Merge, split, rename, create layer1 docs to produce the clearest knowledge base. Session files are input only — never rewritten in place.
- **Keep indexes in sync**: After any change, update `CLAUDE.md` (layer1 table + active sessions) and `SESSIONS.md` (status + timestamps).

**Tooling note**: Prefer the built-in `LS`, `Read`, `Glob`, and `Grep` tools over `Bash` for listing, reading, and searching — they are pre-approved. Use `ls -lt` only to read modification times. Avoid compound bash commands with `&&`, `||`, or pipes.

## Workflow

### 0. Setup

Ask the user:

> Where is your source directory? (e.g. `~/src/` or `~/base/src/`)

Use this as the root when verifying factual claims against code.

### 1. Detect Structure

List `~/.claude/vault/`. Decide which mode you're in:

- **Layered** — `layer1/` exists → this is a steady-state run. Skip to step 3 (Inventory) and run the full lifecycle.
- **Flat / legacy** — top-level `*.md` domain docs and a `sessions/` folder, no `layer1/` → run the **Migration** in step 2 first, then continue.

### 2. Migration (flat → layered, one-time)

Reconcile the existing vault into the layered structure **without disturbing recent or active work**.

1. **Inventory + timestamp everything.** Read every top-level `*.md` and every folder under `sessions/`. Record real mtimes with `ls -lt`.
2. **Protect recent/active sessions.** A session is **protected** if its status is `active` (per any existing index / its CLAUDE.md) **or** any file in it was modified within ~1 month. Protected sessions are **never moved or rewritten** — they stay in `sessions/`.
3. **Classify top-level domain docs** → layer1 candidates. These become `layer1/*.md`.
4. **Classify sessions:**
   - Protected → stay in `sessions/` (layer 0).
   - Done + untouched ~1 month, knowledge already captured → `layer2/sessions/{name}/` (cold).
5. **Build `SESSIONS.md`** from each session's own files (name, created date, last-active = newest mtime, domain, status, one-line summary).
6. **Scaffold layers**: create `layer1/`, `layer2/sessions/`, `layer3/DELETED.md` as needed.
7. Fold all of this into the Dream Plan (step 4). **Nothing moves until approved.**

### 3. Inventory & Validate

Build a complete picture, then verify it.

- Read all layer 0 + layer 1 docs; scan layer 2 for promotable knowledge. Note overlap, redundancy, staleness, missing cross-references, overall size.
- For each factual claim (file paths, directory structures, enum values, API patterns, naming conventions, config locations): **read the actual source code** under the user's source dir and mark it ✅ verified, ❌ outdated, or ❓ can't verify.
- Be thorough — this is the most important phase. If you can't verify a claim, **flag it but keep it**; never silently drop unverifiable facts.

### 4. Plan (the Dream Plan)

Present the plan and wait for confirmation:

```markdown
## Dream Plan

### Migration (if flat vault):
- Move 6 top-level docs → layer1/
- Archive ads-123, oct-44 → layer2/sessions/ (done, untouched 2+ months)
- Protected (left in place): ads-456 (active), incident-2026-05 (touched 5 days ago)
- Build SESSIONS.md from 8 sessions

### Promotions (layer2 → layer1):
- ADMOB_OAUTH.md — referenced by 2 recent sessions → promote?

### Demotions (layer1 → layer2):
- LEGACY_ADAPTERS.md — untouched 4 months, no recent sessions → archive?

### Compaction:
- CLICKHOUSE.md + EVENTS.md overlap on pipeline routing → merge?

### Harvest from sessions:
- ads-456/PLAN.md has a caching pattern not yet in layer1 → promote to CLICKHOUSE.md

### Limbo candidates:
- research-auth — all knowledge promoted, untouched 6 months → delete?

### Index updates:
- ads-456: active → done; ads-123: done → archived

### Verified against code:
- 34 claims checked — 31 ✅, 3 ❌ (removed), 0 ❓

### Estimated result:
- Before: 8 layer1 files, ~1200 lines, 12 sessions
- After: 6 layer1 files, ~800 lines, 10 sessions
```

Limbo (deletion) candidates require **explicit, per-item confirmation**.

### 5. Execute

Apply the approved changes:

- **Migrate**: create layers, move classified docs/sessions. Protected sessions stay put.
- **Promote** (layer2 → layer1): surface a cold doc when recent sessions show renewed relevance.
- **Demote** (layer1 → layer2): move a layer1 doc untouched for months with no recent session references.
- **Harvest**: extract cross-cutting knowledge from active/done sessions into the right layer1 doc (session files stay as input only).
- **Compact / reorg**: merge overlapping docs, trim verified-outdated content, split oversized docs, tighten prose, add `See also:` cross-references.
- **Archive**: move done + stale sessions → `layer2/sessions/`.
- **Limbo** (only if approved per item): remove the item and append a line to `layer3/DELETED.md` (`{YYYY-MM-DD} - {what} - {why}`).
- **Update indexes**: refresh `CLAUDE.md` (layer1 table + active sessions) and `SESSIONS.md` (status + `Last active`).

When rewriting layer1, aim for scannable headers, short paragraphs, facts over commentary, patterns over instances, code examples only when they add clarity.

### 6. Create Skills (If Warranted)

If a recurring workflow pattern emerges across docs/sessions, offer to extract it into a skill (frontmatter + structured markdown, same format as existing skills), add it to the skills directory, and reference it from `CLAUDE.md`. **Ask before creating.**

### 7. Summary

```markdown
## Dream Summary

### Before: 8 layer1 files, ~1200 lines, 12 sessions
### After: 6 layer1 files, ~800 lines, 10 sessions

### Changes:
- ✅ Migrated flat vault → layers (6 docs → layer1, 2 sessions → layer2)
- ✅ Merged EVENTS.md routing into CLICKHOUSE.md
- ✅ Promoted ADMOB_OAUTH.md → layer1 (renewed relevance)
- ✅ Demoted LEGACY_ADAPTERS.md → layer2 (stale)
- ✅ Updated CLAUDE.md + SESSIONS.md

### Sessions:
- ✅ Archived ads-123 → layer2/sessions/
- ⏭️ ads-456 active — left untouched
- 🗑️ research-auth → limbo (approved; logged in layer3/DELETED.md)

### Verified against code: 34 claims — 31 ✅, 3 ❌ removed
### Flagged (couldn't verify): ⚠️ ClickHouse cluster config — no prod access
```

## Index Formats

**`~/.claude/vault/CLAUDE.md`** (layer 0 index):

```markdown
# Vault

## Active sessions
sessions/ads-456/

## Layer 1 — Domain docs
| File | Covers |
|------|--------|
| layer1/CLICKHOUSE.md | ClickHouse schema, query patterns, gotchas |
| layer1/EVENTS.md | Kafka event pipeline, Avro schemas |
```

**`~/.claude/vault/SESSIONS.md`** — see the `manage-sessions` skill for the column layout.

## Guidelines

**Compress**: redundant explanations across files, verbose prose, duplicate examples, non-actionable history.
**Preserve**: verified facts (paths, structures, conventions), non-obvious gotchas, cross-repo dependencies, anything painful to re-learn.
**Drop**: patterns verified outdated against code, info obvious from reading the code, over-specific details that belong in code comments.

**Ask the user**: before executing the plan; before any limbo deletion; before creating skills; when two docs contradict; when a claim can't be verified but seems important; when unsure whether something is noise or real context.

## Error Handling

- **No vault folder**: inform the user.
- **Empty / near-empty vault**: say so; suggest running sessions first to populate it.
- **Can't access referenced repos**: flag unverifiable claims, don't drop them.
- **Everything looks clean**: say so — don't restructure for its own sake.
- **Recent/active session in the way**: never touch it; note it as protected in the summary.
