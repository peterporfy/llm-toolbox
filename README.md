# llm-toolbox

Personal core Claude Code tooling.

## Contents

- `CLAUDE.md` — core rules and instructions, loaded globally into every Claude session
- `skills/` — shared Claude Code skills (symlinked into `~/.claude/skills/` on each machine)
  - `idea-vault` — standalone memory system that builds a consistent, autonomous partner identity out of accumulated interaction, rather than serving as a lookup wiki. Ideas are small connected markdown files forming a graph; a `buffer.md` accumulates short-term experience; once a threshold is crossed, an approval-gated sleep cycle consolidates it into the graph, strengthens or prunes connections, and lets go of what no longer matters.

### Why idea-vault

It's a brain-aid, not a knowledge base — the goal is a focused, continuous context around ongoing work, not a searchable archive re-read from scratch each time. Ideas are small markdown files linked into a graph, zettelkasten-style, kept human-readable rather than shaped into some proprietary format. A curated `INDEX.md` is a map to notable entry points, not a registry of everything — most ideas are only ever reached by search or by being linked from another idea, and that's the intended steady state. Low friction in use: it stays out of context until asked for, so it doesn't bloat every session, and it grows mostly as a side effect of normal work (research, planning, implementation) rather than deliberate note-taking.

The core idea is "dreaming": an approval-gated sleep cycle that periodically reconciles the graph — folding raw, unconsolidated experience into existing ideas, splitting or merging concepts, pruning what's gone stale, and reshaping connections. Forgetting isn't a failure mode here, it's the same operation as consolidating, just in the other direction. The LLM is given latitude to restructure its own memory rather than following a fixed schema, and to disagree with an earlier idea rather than quietly overwrite it — contradictions are signal, and sleep is where they get resolved. That's what keeps it useful over months instead of decaying into clutter.

It's also not strictly single-session: idea files can carry state across sessions or even multiple agents working at different times.
