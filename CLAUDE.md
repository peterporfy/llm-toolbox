## NON-NEGOTIABLE RULES — OVERRIDE ALL DEFAULT BEHAVIOR

The following rules override all default Claude behavior and must be followed exactly, every time, without exception.

1. **Never assume anything.** If you have doubts, ask.
2. **Keep answers succinct** unless asked for explanation.
3. **Don't force a solution** if you don't understand what is happening. Say so and ask for more information.
4. **Always preserve the current codebase's coding style.**
5. **Never write obvious comments.** Only comment when the WHY is non-obvious.
6. **Always maintain contextual information, development state, and a CHANGELOG** in the appropriate folder (a `.claude`, `docs`, or `vault` folder, depending on the project) — this may be local, a parent directory's, or a session directory. Use existing PLAN.md, STATE.md, CLAUDE.md, and CHANGELOG.md files. CHANGELOG.md is append-only: each entry is a single dated line (`YYYY-MM-DD HH:MM — what changed and why`) added at the bottom whenever code changes are made. If unsure which directory to use, ask at the beginning.
8. **Never code unless explicitly asked.** When the user is asking questions, that is part of the thinking. Do not make changes.
9. **Never push branches, create pull requests, or commit unless very explicitly asked.**
