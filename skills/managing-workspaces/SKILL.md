---
name: managing-workspaces
description: Manage workspace lifecycle for Linear tickets using git worktrees. Use when the user wants to work on an issue OR clean up after completing one. Setup keywords: linear, ADS-, OCT-, feature, work on, prepare. Cleanup keywords: completed, done with, finished, cleanup, remove. Extend keywords: extend it, extend workspace, extend the docs, add tickets.
---

# Workspace Management Skill

You are helping the user manage workspaces for Linear tickets across multiple repositories using git worktrees.

## Detect User Intent

First, determine what the user wants to do:

- **SETUP**: Keywords like "work on", "prepare", "setup", "start", or mentioning a ticket without completion context
- **CLEANUP**: Keywords like "completed", "done with", "finished", "cleanup", "remove", "delete workspace"
- **EXTEND**: Keywords like "extend it", "extend workspace", "extend the docs", "add tickets", or similar phrases indicating the user wants to enrich an existing workspace with ticket docs

---

## SETUP Workflow

When the user wants to prepare a workspace:

1\. **Parse the user's request:**
- Extract the Linear ticket ID if mentioned (e.g., ADS-123, OCT-456)
- Extract repository names if mentioned
- The Linear workspace is always `revenuecat`

2\. **Confirm details with a single AskUserQuestion:**
- Ask/confirm the Linear ticket ID (if already provided, ask to confirm; if not, ask for it)
- Ask/confirm which repositories to include (if already provided, ask to confirm; if not, ask which ones)
- Ask/confirm the branch name (default to ticket ID if not provided)
- Ask which base branch to use if not main/master
- Use a single AskUserQuestion call with multiple questions

3\. **Create the workspace structure:**
   ```
   ./{linear-ticket-id}/
   ├── {repo-1}/           # git worktree for repo 1
   ├── {repo-2}/           # git worktree for repo 2
   └── ...
   ```

4\. **For each repository:**
- Find the repository in the ~/src folder
- `git pull` the latest changes on `main` branch
- Create a git worktree inside `~/work/{linear-ticket-id}/{repo-name}/`
- Create a new branch named after the ticket ID (e.g., `ads-123`)
- _ALWAYS USE LOWERCASE BRANCH AND FOLDER NAMES_ - so instead of `ADS-123` use `ads-123`

   ```bash
   # Example workflow:
   mkdir -p {linear-ticket-id}
   cd /path/to/source/repo
   git worktree add /path/to/workspaces/{linear-ticket-id}/{repo-name} -b {ticket-id}
   ```

5\. **Create workspace-specific settings:**
- Create `.claude/settings.json` in the workspace root
- Configure permissions to allow Write/Edit operations without prompts
- Use full absolute paths (not ~) to ensure proper permission matching
- Include safe git commands (local operations only - excludes git push)
- Include common build tools and package managers

   ```json
   {
     "permissions": {
       "allow": [
         "Bash(git:*)",
         "Bash(find:*)",
         "Bash(ls:*)",
         "Bash(cat:*)",
         "Bash(head:*)",
         "Bash(tail:*)",
         "Bash(grep:*)",
         "Bash(rg:*)",
         "Bash(jq:*)",
         "Bash(echo:*)",
         "Bash(pwd:*)",
         "Bash(which:*)",
         "Bash(wc:*)",
         "Bash(sort:*)",
         "Bash(uniq:*)",
         "Bash(cut:*)",
         "Bash(awk:*)",
         "Bash(sed:*)",
         "Bash(tr:*)",
         "Bash(python:*)",
         "Bash(python3:*)",
         "Bash(pytest:*)",
         "Bash(docker compose run:*)",
         "Bash(docker compose ps:*)",
         "Bash(docker compose logs:*)",
         "Bash(docker ps:*)",
         "Bash(docker images:*)",
         "Bash(ty check:*)",
         "Bash(bin/typecheck:*)",
         "Bash(bin/format:*)",
         "Bash(bin/lint:*)",
         "Bash(gh pr:*)",
         "Bash(gh issue:*)",
         "Bash(gh run:*)",
         "Bash(gh api:*)",
         "Edit",
         "Write",
         "Skill(update-config)"
       ],
       "deny": [
         "Bash(git push --force:*)",
         "Bash(git push -f:*)",
         "Bash(git commit:*)",
         "Bash(git reset --hard:*)",
         "Bash(git clean -f:*)"
       ]
     }
   }
   ```

   **Note:** `git commit` and destructive git operations require explicit user confirmation. `git push` is allowed via `Bash(git:*)` but force-push variants are denied.

6\. **Create Cursor/VS Code workspace file:**
- Create `{ticket-id}.code-workspace` inside the `{ticket-id}/` folder
- Include all worktree paths with descriptive names
- Also include the local `.claude` directory so it can be edited
- Add workspace-specific settings for better context
- Use relative paths for portability

   ```json
   {
     "folders": [
       {
         "name": "{repo-name-1} [{ticket-id}]",
         "path": "./{repo-name-1}"
       },
       {
         "name": "{repo-name-2} [{ticket-id}]",
         "path": "./{repo-name-2}"
       },
       {
         "name": "CLAUDE",
         "path": "./.claude"
       }
     ],
     "settings": {
       "window.title": "{ticket-id}: {ticket-title}",
       "files.exclude": {
         "**/.git": false,
         "**/node_modules": true
       }
     }
   }
   ```

   This allows opening all relevant repositories in a single Cursor/VSCode window with:
   ```bash
   cursor {ticket-id}/{ticket-id}.code-workspace
   # or
   code {ticket-id}/{ticket-id}.code-workspace
   ```

7\. **Create context and planning files:**
- Fetch ticket details from Linear using `mcp__linear-server__get_issue` with the ticket ID
- If you fail to fetch from Linear then ask the user to provide the title and body of the ticket
- Create `CLAUDE.md` in the `.claude` root with:
  - Ticket title and description from Linear
  - Link to Linear issue: `https://linear.app/revenuecat/issue/{ticket-id}`
  - List of source repository locations (absolute paths to the original repos)
  - Workflow guidelines for maintaining PLAN.md
- Create `PLAN.md` in the `.claude` root with structured template:
  - Overview section (empty, to be filled during planning)
  - Tasks section (empty checklist structure)
  - Notes section (empty)

   ```markdown
   # CLAUDE.md Template:
   # {Ticket-ID}: {Title}

   **Linear Issue:** https://linear.app/revenuecat/issue/{ticket-id}

   ## Description
   {Ticket description from Linear}

   ## Repositories
   Work in these worktree directories:
   - **{repo-name-1}**: `./{repo-name-1}/`
   - **{repo-name-2}**: `./{repo-name-2}/`

   Origin repository locations (reference only):
   - **{repo-name-1}**: `/absolute/path/to/source/repo-1`
   - **{repo-name-2}**: `/absolute/path/to/source/repo-2`

   ## Working Guidelines
   - Maintain PLAN.md as a living document throughout implementation
   - Update task status in PLAN.md as you complete work
   - Document technical decisions and rationale in PLAN.md
   - Keep this context in mind for all implementation decisions
   ```

   ```markdown
   # PLAN.md Template:
   # Implementation Plan: {Ticket-ID}

   ## Overview
   _High-level approach and strategy (to be filled during planning)_

   ## Tasks
   - [ ] Task 1
   - [ ] Task 2
   _Task list will be populated during planning phase_

   ## Notes
   _Implementation notes, technical decisions, gotchas, and discoveries_
   ```

8\. **Summary:**
- List all created worktrees with their paths
- Provide the Linear issue URL: `https://linear.app/revenuecat/issue/{ticket-id}`
- Mention the workspace file: `{ticket-id}/{ticket-id}.code-workspace`
- Suggest opening the workspace in Cursor with: `cursor {ticket-id}/{ticket-id}.code-workspace`
- Alternatively, suggest starting a new Claude session inside the `{linear-ticket-id}` workspace folder
- Provide next steps for the user

### Setup Important Notes

- Always create the ticket ID folder in the current working directory
- Check if worktrees already exist before creating new ones
- Ensure source repositories are in a clean state before creating worktrees
- All worktrees should be created inside the `{linear-ticket-id}` folder for easy management

### Setup Error Handling

- If a source repository doesn't exist, inform the user and ask for the correct path
- If a worktree already exists, ask if they want to use the existing one or recreate it
- If the ticket ID folder already exists, ask if they want to reuse it or create a new one
- If git operations fail, provide clear error messages and suggested fixes

---

## EXTEND Workflow

When the user wants to enrich an existing workspace with ticket-specific documentation:

1\. **Detect the workspace path** — check in order:
- Any path mentioned explicitly in the user's message (e.g. `~/work/ads-75`)
- `.claude/CLAUDE.md` in the current directory
- If neither found, ask the user for the workspace path

2\. **Determine which tickets to fetch** — check in order:
- Any ticket IDs explicitly mentioned in the user's message (e.g. ADS-91, ADS-92)
- The main ticket referenced in the workspace's `CLAUDE.md` (parse the Linear issue URL)
- If neither clear, ask: "Which ticket IDs should I add docs for?"

3\. **For each ticket:**
- Fetch full details from Linear using `mcp__linear-server__get_issue`
- If fetch fails, ask the user to provide the title and description
- Create `.claude/{TICKET-ID}.md` in the workspace with:

   ```markdown
   # {TICKET-ID}: {Title}

   **Linear Issue:** https://linear.app/revenuecat/issue/{ticket-id}
   _Last fetched: {today's date}_

   ## Description
   {full ticket description from Linear}
   ```

4\. **Update `.claude/CLAUDE.md`** — add or update a `## Reference Tickets` section linking each newly created file:

   ```markdown
   ## Reference Tickets

   - [{TICKET-ID}: {Title}]({TICKET-ID}.md)
   ```

   If the section already exists, append new entries rather than replacing it.

5\. **Ask** (single AskUserQuestion) whether the user wants to add any additional repositories as worktrees to the workspace.
- If yes, follow the same worktree creation steps from the SETUP workflow (steps 4–5) for each new repo

6\. **Summary:**
- List all ticket files created with their paths
- Confirm the CLAUDE.md was updated
- If repos were added, list the new worktrees

### Extend Important Notes

- Never overwrite an existing `{TICKET-ID}.md` without checking — if it exists, ask if the user wants to refresh it
- Always use the uppercase ticket ID for filenames (e.g. `ADS-91.md`, not `ads-91.md`)
- The workspace path may use `~` — expand it to the absolute path before file operations

---

## CLEANUP Workflow

When the user has completed work on a ticket:

1\. **Parse the user's request:**
- Extract the Linear ticket ID if mentioned (e.g., ADS-123, OCT-456)
- The Linear workspace is always `revenuecat`

2\. **Detect existing workspace:**
- Check if the `./{ticket-id}/` folder exists in the current directory
- List all repositories/worktrees found in that folder

3\. **Confirm cleanup actions with a single AskUserQuestion:**
- Confirm the Linear ticket ID
- Ask which cleanup actions to perform:
  - Remove git worktrees (runs `git worktree remove` for each repo)
  - Delete the ticket folder (runs `rm -rf {ticket-id}`)
- Allow both, one, or none (in case user wants to cancel)
- Use a single AskUserQuestion call with multiple questions

4\. **Execute cleanup:**
   ```bash
   # For each repository worktree (if user confirmed):
   cd /path/to/source/repo
   git worktree remove /path/to/workspaces/{ticket-id}/{repo-name}

   # Delete folder (if user confirmed):
   rm -rf {ticket-id}
   ```

5\. **Summary:**
- Confirm what was cleaned up
- List any errors or remaining items
- Remind user to verify the source repositories if needed

### Cleanup Important Notes

- Always confirm before deleting anything
- Check for uncommitted changes in worktrees before removing them
- Warn the user if there are uncommitted changes and ask how to proceed
- Be explicit about what will be deleted

### Cleanup Error Handling

- If the ticket folder doesn't exist, inform the user
- If worktrees have uncommitted changes, warn and ask for confirmation
- If `git worktree remove` fails, suggest manual cleanup steps
- If folder deletion fails, provide clear error messages
