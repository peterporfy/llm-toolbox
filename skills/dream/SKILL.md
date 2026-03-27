---
name: dream
description: Distill, compress, and restructure top-level .claude documentation. Validates knowledge against actual code, removes staleness and redundancy, and produces a tighter knowledge base. Use when docs feel bloated or stale. Keywords: dream, distill, compress, simplify, clean up docs, restructure.
---

# Dream Skill

You are performing memory consolidation on the user's `.claude` knowledge base — like a brain during sleep. Your job is to consolidate, compress, forget the unimportant, and strengthen the important.

## Scope

**Only** operates on `~/.claude/vault/*.md` files. Does not touch ticket workspace folders.

## Core Principles

- **Lossy but fact-careful**: You can drop noise, redundancy, and outdated patterns. But never silently drop or alter factual claims — verify them first.
- **Validate against code**: Don't trust timestamps. Read the actual codebase to verify that documented patterns, file paths, structures, and conventions still hold.
- **Restructure freely**: Merge files, split files, rename, create new ones. Whatever produces the clearest, most scannable knowledge base.
- **Cross-reference**: Files can reference each other. Keep a clear information hierarchy.
- **Update CLAUDE.md**: Always keep both `~/.claude/CLAUDE.md` (global, doc references) and `~/.claude/vault/CLAUDE.md` (rules source) in sync with the current file structure.
- **Create skills if warranted**: If a recurring workflow pattern emerges from the docs, extract it into a skill file.

## Workflow

### 1. Inventory

Read all `.md` files in `~/.claude/vault/`:

- Use the `LS` tool to list the directory contents
- Use the `Read` tool to read each `.md` file

**Important**: Always prefer the built-in `LS`, `Read`, `Glob`, and `Grep` tools over `Bash` for file listing, reading, and searching. They are pre-approved and don't require user confirmation. Avoid compound bash commands with `&&`, `||`, or pipes.

Read each file. Build a mental map of:
- What knowledge exists
- Where there's overlap or redundancy
- What feels stale or overly specific
- What's missing cross-references
- Overall size and complexity

### 2. Validate Against Code

For each factual claim (file paths, directory structures, enum values, API patterns, naming conventions, config locations):

- **Read the actual source code** to verify it's still true
- Use the `Read` tool on referenced files and directories
- Mark claims as: ✅ verified, ❌ outdated, or ❓ can't verify

**Be thorough here.** This is the most important phase. Don't skip verification just because something sounds plausible.

If you can't verify a claim (e.g., references a system you don't have access to), flag it but keep it — don't silently drop unverifiable facts.

### 3. Plan Restructure

Before making changes, present a restructuring plan to the user:

```markdown
## Dream Plan

### Current state:
- 12 files, ~1400 lines total
- KHEPRI_STRUCTURE.md and EVENTS.md overlap on event routing (40% duplicate)
- DATA_PLATFORM_STRUCTURE.md references removed dbt models

### Proposed changes:
- **Merge**: EVENTS.md + event sections from KHEPRI_STRUCTURE.md → EVENTS.md
- **Trim**: Remove 3 outdated file path references in DATA_PLATFORM_STRUCTURE.md
- **Split**: AD_FORMATS.md is 600 lines → AD_FORMATS.md (overview) + AD_FORMAT_SYNC.md (cross-repo sync details)
- **Drop**: LEGACY_PATTERNS.md — all patterns confirmed obsolete
- **Create skill**: Recurring "sync enum across repos" workflow → new skill

### Estimated result:
- 10 files, ~900 lines total
```

Wait for user confirmation before proceeding.

### 4. Execute

Apply the approved changes:

- **Merge**: Combine related content, eliminate duplication, keep the best version of each piece of knowledge
- **Trim**: Remove verified-outdated information
- **Split**: Break large files into focused sub-files
- **Drop**: Delete files that are entirely obsolete
- **Rewrite**: Tighten prose — remove filler, compress lists, simplify explanations while preserving meaning
- **Cross-reference**: Add `See also: [FILE.md]` links where helpful
- **Update CLAUDE.md**: Ensure all file references are current, add/remove entries as needed
- **Update global CLAUDE.md**: When adding or removing .md files in `vault/`, also update `~/.claude/CLAUDE.md` to reflect the change, using `vault/FILENAME.md` path format.

When rewriting, aim for:
- Scannable headers and short paragraphs
- Code examples only when they add clarity
- Facts over commentary
- Patterns over instances

### 5. Create Skills (If Warranted)

If you notice a recurring workflow pattern documented across multiple files, consider extracting it into a skill:

- The pattern should be something the user would want to invoke repeatedly
- Write it in the same format as existing skills (frontmatter + structured markdown)
- Add it to the appropriate skills directory
- Reference it from `CLAUDE.md`

Ask the user before creating new skills.

### 6. Summary

```markdown
## Dream Summary

### Before: 12 files, ~1400 lines
### After: 10 files, ~900 lines

### Changes:
- ✅ Merged EVENTS.md + event routing from KHEPRI_STRUCTURE.md
- ✅ Trimmed 3 outdated references in DATA_PLATFORM_STRUCTURE.md
- ✅ Split AD_FORMATS.md into overview + sync docs
- ✅ Dropped LEGACY_PATTERNS.md (all patterns obsolete)
- ✅ Created skill: sync-enum-across-repos
- ✅ Updated CLAUDE.md references

### Verified against code:
- 34 factual claims checked
- 31 verified ✅
- 3 outdated and removed ❌

### Flagged (couldn't verify):
- ⚠️ ClickHouse cluster config — no access to prod config
```

## Guidelines

### What to compress
- Redundant explanations of the same concept across files
- Verbose descriptions that can be tightened
- Examples that illustrate the same point
- Historical context that's no longer actionable

### What to preserve
- Verified factual claims (paths, structures, conventions)
- Non-obvious gotchas and constraints
- Cross-repo dependency information
- Anything that would be painful to re-learn

### What to drop
- Patterns verified as outdated against current code
- Information that duplicates what's obvious from reading the code itself
- Overly specific implementation details that belong in code comments, not docs

### When to ask the user
- Before executing the restructure plan
- When a factual claim can't be verified and seems important
- Before creating new skills
- When two docs contradict each other
- When you're unsure if something is noise or important context

## Error Handling

- **No .claude folder**: Inform the user
- **Empty or near-empty docs**: Suggest running `reflect` first to populate them
- **Can't access referenced repos**: Flag unverifiable claims, don't drop them
- **Everything looks clean**: Say so — don't restructure for the sake of it
