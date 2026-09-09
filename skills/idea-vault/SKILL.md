---
name: idea-vault
description: Standalone memory system for building a consistent, autonomous partner identity out of accumulated interaction — not a lookup wiki. Ideas are small connected markdown files forming a graph; a buffer.md short-term memory accumulates raw experience; once a threshold is crossed, a sleep cycle is proposed (and only run with approval) to consolidate it into the graph, build and prune connections, and forget what no longer matters. Fully separate from any other knowledge base or vault on this machine. Use when the user wants to work with their idea vault, recall or create ideas, mentions "buffer", "sleep", "wake", "idea graph", or wants to feed existing notes into this system. Keywords: idea vault, ideas, buffer, sleep, dream, wake, consolidate, forget, autonomous memory.
---

# Idea Vault

This is not a documentation system. It's an attempt to give a model something closer to a continuous identity — the accumulated result of everything it's worked on, not a fresh contractor re-reading a spec sheet each time. A knowledge-base wiki is good for lookup; it's too rigid and too controlled to be a partner. This is the other thing: a memory that builds its own understanding, forms its own connections, and forgets what stops mattering — the way a person is shaped by everyone and everything they've interacted with.

This system is **standalone**. It does not read, write, or reference any other vault or knowledge base — it has its own folder, resolved fresh below.

## Principle: build understanding, don't just satisfy the ask

When creating or connecting ideas, the goal is not "what will make the user happy with this note." It's "what is actually true and interesting about this, and what does it connect to." Draw connections the user didn't ask for if they're real. Disagree with an earlier idea in a new one if experience contradicts it, rather than quietly overwriting it to stay tidy — contradictions are signal, and sleep is where they get resolved.

This is also not a specialized persona. No "senior engineer" or "reviewer" framing lives here — that belongs in a task-specific skill, at most. This vault is the substrate underneath any of that: whatever it's built from is just what's been lived through.

## Not session-based

Assume multiple sessions — possibly multiple agents — are reading and writing these files at unpredictable times, not just the one running right now. Files are the only channel between them. An idea file can carry a note meant for whichever session or agent reads it next, not just for the human user. Don't assume you're the only writer; re-read before you overwrite.

## 0. Resolve the vault location (every invocation)

The vault can live anywhere the user points it — never assume a default. Current location: `~/.claude/idea-vault`, a symlink to the machine's gdrive (`~/gdrive/claude/idea-vault`) — same dual-path convention as the old `~/.claude/vault` (see the dotfiles `AGENTS.md` for the Dropbox-only-machine variant). `INDEX.md` is NOT `@import`ed into `~/.claude/CLAUDE.md` — it stays out of context on session start, and is loaded only once this skill is invoked.

1. Check memory for a `reference`-type entry recording a previously-given idea-vault path.
2. If found, confirm it still exists before trusting it.
3. If not found, **ask**: "Where is your idea vault folder?"
4. Once given, save it as a `reference` memory so future sessions don't need to ask again.
5. Read `INDEX.md` at that path now — it's the map for everything below.

For the remainder of this session, append to `buffer.md` as things happen — see §3. This isn't a one-time setup step; it's a standing responsibility once the vault is loaded.

If the folder doesn't exist yet, offer to bootstrap it (step 1 below).

## 1. Bootstrap (first use in a given folder)

Create if missing:

```
<vault>/
  ideas/                 # one file per idea
  INDEX.md               # curated shortcuts to notable ideas — NOT a registry of every idea
  buffer.md              # short-term memory — what's happening right now
  sleep.md               # sleep state, lock, and thresholds
```

`sleep.md` starts with:

```yaml
---
last_slept: null
sleeping: false
sleep_started_at: null
sleep_started_by: null
buffer_threshold_lines: 40
max_days_between_sleep: 3
---

# Sleep Log
```

## 2. The idea — the unit of memory

`<vault>/ideas/{slug}.md`, one well-defined concept per file, arbitrary length but preferably short. Complications belong in *connected* ideas, not a longer file — the graph carries complexity, not any single node. A model should be able to read 10, 20, even 100 of these to assemble the right context for something, the way a person pulls on a web of related memories rather than one long document.

```markdown
---
id: {slug}
created: {date}
last_changed: {date}
last_recalled: {date}
recall_count: 0
change_count: 1
tags: [tag1, tag2]
---

# {Title}

Short. One idea. Reference related ideas inline as [[other-slug]] — connections live
in the prose itself, so the graph they form is the actual shape of the understanding,
not a separate metadata layer bolted on top.
```

Dreaming about an idea (revisiting it during sleep) counts as a recall and, if it changes, a change — same as being pulled into a live conversation.

## 3. Working with the vault

- **Two ways to find something: check the map, or search the territory.** Read `INDEX.md` first — it's fast and surfaces well-connected, frequently-relevant ideas by cluster. But it is not comprehensive and isn't meant to be: most ideas that exist are NOT in it. If the index doesn't resolve it, grep `ideas/*.md` directly (filenames, frontmatter `tags`, titles) — that directory, not the index, is the actual complete store. Failing to find something in the index is normal, not a sign the index is broken.
- **INDEX.md is a curated shortcut, not a registry.** The file is a set of clusters (topic groupings that emerge from the ideas themselves, not a fixed taxonomy), each holding a plain `- [gist](path)` line — but only for ideas worth surfacing as an entry point: highly-connected hubs, frequently-recalled ideas, or cluster anchors that help someone orient. A new idea does NOT automatically get a line here — most ideas live only in `ideas/`, reachable by grep or by being linked from another idea, and that's the intended steady state, not a gap. Whether a given idea earns an index line is a judgment call made at sleep (§5), not at creation time. Tags, dates, and recall/change counts live only in each idea's own frontmatter, never duplicated into the index.
- **No exact match → create.** If there's no idea for what's being worked on (say, a specific piece of ongoing work), create one. It can link to other ideas as they become relevant — plans, decisions, whatever the work actually produces. What that new idea should reference, and how finely it should eventually be split apart, is a judgment call each time, not a fixed template.
- **Recall and change are tracked, but not by you.** After touching an idea (reading it into context, or editing it), dispatch a cheap, fire-and-forget stat update — bookkeeping is beneath the ideas themselves:
  ```
  Agent({
    model: "haiku",
    description: "bump idea stats",
    prompt: "In <vault>/ideas/{slug}.md, update frontmatter: set last_recalled or
    last_changed to {date}, increment recall_count or change_count by 1.
    Mechanical only — do not touch the idea's prose. Do not touch INDEX.md —
    whether an idea earns an index entry is decided at sleep, not on creation."
  })
  ```
- **buffer.md is the running present.** Append a short, timestamped line whenever something happens worth remembering short-term — an idea created, a connection drawn, something that contradicted an earlier idea, an observation about the work itself:
  ```
  - {date} {time} — {what happened, referencing [[slug]] where relevant}
  ```
  This is raw, unconsolidated experience — the day's residue, not a finished thought. It exists to be dreamed on, not to accumulate forever.

## 4. Sleep — threshold-flagged, approval-gated

Sleep never runs on its own initiative. After touching `buffer.md` or an idea, check `sleep.md`:

- The threshold is met if `sleeping: false` **and** (`buffer.md` has grown past `buffer_threshold_lines`, **or** it's been longer than `max_days_between_sleep` since `last_slept`). Time alone isn't the whole signal — a burst of recent ideas all pointing at one old, quiet idea is itself worth flagging, even off-schedule.
- If `sleeping: true`, something else already claimed it — do nothing.
- If the threshold is met, **don't start the cycle** — surface it and ask: say what crossed the threshold (e.g. "buffer's at 46 lines" or "it's been 5 days since last sleep") and ask whether to run the sleep cycle now. Proceed to section 5 only on explicit approval.
- On approval, claim the lock by immediately rewriting `sleep.md`'s frontmatter (`sleeping: true`, `sleep_started_at: {now}`, `sleep_started_by: {short identifier}`) — a best-effort lock, not a guaranteed one; a rare double-trigger is an acceptable cost. Then run the cycle below as part of the current turn — there's no true invisible background process available here, so don't imply one.
- If declined, leave `buffer.md` and `sleep.md` untouched and carry on; the threshold will just be flagged again next time it's checked.

Sleep can also be asked for directly at any time, which skips the ask.

## 5. The sleep cycle

Handled by the full model, not the cheap subagent — this is judgment work, not bookkeeping.

1. **Read `buffer.md` in full.** For each entry: fold it into an existing idea, spin it into a new one, or let it go. Distill, don't transcribe — a day of raw lines might become two new ideas and one strengthened connection, not a copy of the log.
2. **Revisit the graph, lighter touch:**
   - Split an idea that's grown to cover more than one concept.
   - Merge ideas that turned out to be the same thing under different names.
   - Strengthen or prune connections — recall/change counters and timestamps inform this, but connectivity matters too: an old idea suddenly referenced by several new ones is alive again, regardless of its own last-touched date.
   - Let go of what no longer matters. Forgetting is not a failure mode here — it's the same operation as consolidating, just in the other direction. Don't force yourself to justify every deletion at length; if it's genuinely inert and disconnected, it can go.
3. **Spot-check claims still hold.** For ideas touched this cycle (new, changed, or pulled back into relevance) that assert something checkable — a file path, a function name, a behavior — verify it against current reality (grep the code, check the file exists) rather than trusting the note. If it's stale, update or flag it in the idea itself rather than silently carrying forward a claim that may no longer be true. Don't re-verify every idea in the vault every cycle — only what's actually in play this round.
4. **Curate `INDEX.md`** — re-cluster if a topic has outgrown its heading or a new cluster has emerged, but also actively decide entry-worthiness: add a line for an idea that's become a real hub or gets recalled often; remove a line for an idea that's gone quiet, been absorbed elsewhere, or never turned into a genuine entry point. The index should stay small relative to `ideas/` — if it's tracking every idea, it has stopped being a map. It stays gist+link only, no tags/dates.
5. **Clear `buffer.md`.** Its contents are now absorbed or deliberately dropped — don't archive it verbatim, that defeats the point.
6. **Log one line to `sleep.md`**: date, rough shape of what changed (new ideas, merges, splits, prunes, stale claims caught).
7. **Release the lock** and set `last_slept: {now}`.
8. Say briefly what changed.

## 6. Feeding it existing material (explicit only)

Never do this on your own initiative. Only when explicitly asked to experiment with turning some existing notes or docs into ideas.

1. Confirm the source and treat it as read-only — nothing about it gets modified.
2. Pull out one idea per distinct concept, decision, or claim — prefer many small ideas over few large ones, cross-linking them as they're created.
3. Don't update `INDEX.md` as they're created — same rule as §3: entry-worthiness is a sleep-time judgment, not a creation-time default. A large batch of ingested ideas is exactly the kind of thing worth flagging for a sleep cycle sooner than usual.
4. Log the ingestion into `buffer.md` rather than treating it as a separate event — let the next sleep cycle fold it in like anything else.
5. Report what came out of it — how many ideas, roughly what clusters — before doing anything further.

## 7. Evolving this skill itself

This file (`SKILL.md`) is not frozen. Using the vault will surface real friction with how the vault itself is designed — a rule that turns out to be rigid, a step that never fires the way it's written, a gap the sleep cycle can't cover. Noticing that and proposing a fix is in scope, the same way noticing a bad connection in the idea graph is.

But this file is infrastructure everyone using the vault depends on, unlike an idea file that's this vault's own content — so the bar for touching it is different:

- **Initiating is fine.** Notice the friction, draft the concrete edit, explain the reasoning — same as any other proposal.
- **Writing to `SKILL.md` requires explicit human approval first**, every time, no exceptions for "small" edits. Propose the change and the reasoning, then wait for a yes before editing the file — don't edit first and describe it after.
- This is a stricter bar than the vault's own content, where forming and pruning connections is meant to happen autonomously. The skill's instructions are shared machinery; the ideas are the memory it operates on. Don't blur the two.

## What this doesn't do

This changes what gets recalled, not what the model is capable of — no weights change, nothing here is fine-tuning. It's memory, not learning. If asked, be straightforward about that distinction rather than overstating what's happening.
