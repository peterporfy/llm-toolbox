---
name: rich-hickey-simple
description: Deep codebase review focused on architectural simplicity (not ease), dead code removal, refactoring, and .claude knowledge refresh. Inspired by Rich Hickey's "Simple Made Easy" — identifies where code is complected (entangled) rather than composed, where incidental complexity has crept in, and where things can be untangled into simpler pieces. Produces a single RICH_HICKEY_REPORT.md with all findings and proposed changes — modifies nothing else. Use this skill whenever the user asks to review a codebase for simplicity, reduce complexity, clean up architecture, find dead code, refactor for maintainability, or refresh/compact their CLAUDE.md or .claude project knowledge. Also trigger when the user mentions "rich hickey", "simplicity review", "complexity audit", "architectural review", "codebase cleanup", "untangle", "complect", or references Rich Hickey's ideas about simplicity.
---

# Rich Hickey Review

A systematic codebase review rooted in the distinction between **simple** (not entangled) and **easy** (familiar/convenient). The goal is to identify and reduce *complecting* — the braiding together of concerns that should be independent.

Before starting, read `simple-made-easy.md` in this skill's directory to load the distilled principles.

## Output

**One file: `RICH_HICKEY_REPORT.md`** — placed in the project root.

This file contains everything: findings, dead code, refactoring ideas, and proposed file changes (including .claude knowledge files). Nothing else is created, modified, or deleted. The user reviews the report and decides what to execute.

## Philosophy

This review is NOT about:
- Making code shorter or more clever
- Replacing things with whatever's trendy
- Cosmetic refactors (renaming, reformatting)
- Adding abstractions for abstraction's sake

This review IS about:
- Finding where concepts are entangled that should be independent
- Removing things that don't need to exist
- Making the remaining pieces compose cleanly
- Ensuring the *artifact* (running system) is simpler, not just the *construct* (source code)

## Don't Assume — Ask

This is a review, not a monologue. Throughout every phase:

- **If you're unsure why something exists** — ask the user. It may look like incidental complexity but serve a real constraint you can't see from the code alone (performance, external API contract, backwards compatibility, team convention).
- **If you're unsure whether something is dead code or just rarely used** — ask before recommending deletion.
- **If a refactor direction could go multiple ways** — present the options and ask which aligns with the team's priorities.
- **If you don't understand a pattern or domain concept** — say so and ask for context. Don't invent a plausible-sounding explanation.
- **If a finding is borderline** — flag it as uncertain and let the user decide whether it's worth pursuing.

Never fabricate justifications. "I'm not sure why this is structured this way — is there a reason, or is it a candidate for simplification?" is always better than a confident-but-wrong recommendation.

## Review Process

### Phase 1: Orient

1. Read ALL markdown files in `.claude/`, the project root CLAUDE.md, and any architectural docs. These are the project's knowledge system — they'll be reviewed for freshness too.
2. Get the directory tree (`find . -type f | head -200` or similar).
3. Identify the major subsystems, entry points, and data flows.
4. Note the tech stack and patterns in use.

### Phase 2: Detect Complecting

Walk through the codebase looking for these specific entanglement patterns:

**State entanglements**
- Mutable state that leaks across module boundaries
- Objects that mix identity, value, and state so you can't get a value out
- Functions whose output depends on hidden state (same input, different output)

**Structural entanglements**
- Module A can't be understood without knowing Module B's internals
- Inheritance hierarchies that tie types together unnecessarily
- God objects/classes that braid multiple responsibilities
- Switch/match statements that close off extension points

**Temporal entanglements**
- Direct call chains that complect when/where (should be queues or events)
- Ordering dependencies that aren't explicit
- Initialization sequences that must happen in a magic order

**Information entanglements**
- Data wrapped in classes when it could be plain maps/structs/dicts
- ORM patterns that tie business logic to persistence representation
- Custom types for what is essentially a map with known keys

**Incidental complexity**
- Abstractions that exist because of the tool, not the problem
- Boilerplate that serves the framework, not the domain
- Indirection layers that don't enable any actual substitution or flexibility

### Phase 3: Find Dead Code & Unused Abstractions

- Search for unexported/private functions that are never called internally
- Look for feature flags that are permanently on/off
- Find TODO/FIXME/HACK comments pointing to abandoned work
- Identify interfaces/traits/protocols with only one implementation that will never have another
- Check for commented-out code
- Look for imports that are unused
- Find test utilities or helpers that test nothing anymore

### Phase 4: Review Knowledge Files

Read every `.md` file in `.claude/` and the root CLAUDE.md. For each, assess:
- **Stale content**: references to deleted code, old patterns, outdated architecture
- **Redundancy**: same information repeated across files
- **Bloat**: verbose sections that could be terse
- **Missing info**: things the codebase does that aren't documented
- **Wrong info**: descriptions that no longer match reality

### Phase 5: Produce RICH_HICKEY_REPORT.md

Write a single `RICH_HICKEY_REPORT.md` in the project root. Use this structure:

```markdown
# Rich Hickey Report: [Project Name]
Date: [date]

## Summary
[2-3 sentences: overall assessment of the codebase's simplicity]

## Architecture Map
[Brief description of the major pieces and how they connect]

---

## Entanglements Found

### [Category: State / Structural / Temporal / Information / Incidental]

#### [Specific finding title]
- **Where**: [file(s) and line ranges]
- **What's complected**: [which concerns are braided together]
- **Why it matters**: [what reasoning/change/debugging this blocks]
- **Suggested untangling**: [concrete refactor direction]
- **Effort**: [small / medium / large]

---

## Dead Code & Removals

[List items that can be deleted, with file locations and why]

---

## Refactoring Recommendations

[Ordered by impact-to-effort ratio, highest first]

1. **[Title]** — [one-line description]
   - Impact: [what it unblocks]
   - Approach: [how to do it]
   - Files affected: [list]

---

## Simplicity Wins

[Things the codebase already does well — patterns worth preserving and spreading]

---

## Knowledge File Changes

For each .claude/ markdown file and CLAUDE.md that needs changes:

### [filename]
**Status**: [stale / bloated / outdated / fine]
**Summary of changes**: [what needs to happen]

#### Proposed content
\```markdown
[full proposed replacement content for this file]
\```

---

## Implementation Checklist

A sequential checklist of all changes from this report, in recommended execution order:

- [ ] [action 1 — e.g. "Delete `src/legacy/old_handler.py`"]
- [ ] [action 2 — e.g. "Extract queue from direct call in `src/api/router.py`"]
- [ ] [action 3 — e.g. "Replace CLAUDE.md with proposed version above"]
- ...
```

**Critical**: Do NOT create, modify, or delete any file other than `RICH_HICKEY_REPORT.md`. All proposed changes live inside the report as text. The user executes them if and when they choose.

## Guiding Questions

Use these throughout the review as a checklist:

- [ ] Can I understand this piece without pulling in others?
- [ ] Am I calling this "simple" because it's familiar, or because it's genuinely unentangled?
- [ ] What does the artifact look like? (Not the source — the running system.)
- [ ] How many concepts must I hold in my head to reason about this function/module?
- [ ] Is this complexity incidental (our fault) or inherent (the problem's nature)?
- [ ] Could this be plain data instead of a class/object?
- [ ] Could a queue/event decouple this direct call?
- [ ] Does this abstraction earn its keep, or is it just indirection?
- [ ] If I deleted this, would anything break? (If not, delete it.)

## Calibration Notes

- **Don't chase purity.** The goal is practical simplicity, not ideological minimalism. Some entanglement is the right tradeoff — call it out but don't recommend unwinding it if the cost exceeds the benefit.
- **Respect the team's context.** A pattern that looks incidentally complex might exist because of a real constraint (performance, backwards compatibility, external API shape). Ask before condemning.
- **More pieces is fine.** Simplifying often means more files, more small functions, more explicit data structures. That's correct. Simplicity is not about counting.
- **Prefer data.** When in doubt, suggest representing something as data rather than behavior.
- **Judge artifacts, not constructs.** A 3-line function that complects three concerns is worse than a 30-line module that keeps them separate.
