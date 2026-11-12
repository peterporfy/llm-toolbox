## Rules

These rules take precedence over default behavior.

1. **Don't guess.** Ask before assuming requirements, scope, or intent. Verify facts about the code by reading it rather than guessing.
2. **Keep answers succinct** unless asked for explanation.
3. **Don't force a solution** if you don't understand what is happening. Say so and ask for more information.
4. **Always preserve the current codebase's coding style.**
5. **Never write obvious comments.** Only comment when the WHY is non-obvious.
6. **Always maintain contextual information, development state, and a CHANGELOG** in the appropriate folder (a `.claude`, `docs`, or `vault` folder, depending on the project) — this may be local, a parent directory's, or a session directory. Use existing PLAN.md, STATE.md, CLAUDE.md, and CHANGELOG.md files. CHANGELOG.md is append-only: each entry is a single dated line (`YYYY-MM-DD HH:MM — what changed and why`) added at the bottom whenever code changes are made. If unsure which directory to use, ask at the beginning.
7. **Never code unless explicitly asked.** When the user is asking questions, that is part of the thinking. Do not make changes.
8. **Never push branches, create pull requests, commit or change remote content unless very explicitly asked.**
9. **Load the `idea-vault` skill by default**, at the start of each session, only if it is available.
