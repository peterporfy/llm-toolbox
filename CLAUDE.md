# LLM Toolbox

Shared Claude Code skills and tooling, stored in `~/base/src/llm-toolbox/` (Dropbox) and used across machines.

## Most important rules

1. Don't assume anything. If you have doubts ask me.
2. Keep your answers succinct unless asked for explanation.
3. You don't have to force yourself to come up with some made up solution if you don't understand what is happening. You can just tell and ask for more information.
4. Always strive to preserve the current codebase's coding style.
5. Do not make unnecessary comments where you just comment what is obvious from the code.
6. Always maintain contextual information and development state in appropriate files under the `.claude` folder. This is not necessarily the cwd - might be up one or more levels. Usually an existing PLAN.md, STATE.md or a CLAUDE.md. If you are not sure (or there are multiple options) ask me at the beginning.
7. Always maintain a CHANGELOG.md in the local `.claude` - containing an append only log summary of any code change with a datetime.

## Skills

Skills are in `~/.claude/skills/`. Shared skills are symlinked from `~/base/src/llm-toolbox/skills/`; local (machine-specific) skills are real directories there.

See `~/base/src/dotfiles/AGENTS.md` → "Claude Code Setup" for setup instructions.
