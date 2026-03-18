---
description: RALPH loop - fetch GitHub issues, implement, validate, open PR, repeat
---
## RALPH Loop (GitHub Issue Automation)

When the user asks to "run RALPH", "RALPH loop", "RALPH for N iterations", "continue RALPH", "continue", or similar, execute this workflow in a loop.

**Loop until done:** Do not stop until either (a) the requested number of iterations is reached, or (b) all open agent-labelled issues are complete. Never stop after a single task unless one of those conditions is met.

**Never ask for confirmation:** Do not output "Should I continue?", "Ready for the next task?", or similar. Just proceed to the next task.

If `plans/ralph-progress.txt` exists, read it for context on recent work before starting.

**Prerequisites:** GitHub CLI (`gh`) must be installed and authenticated. Run from the repo root so issues are scoped to this project.

### 1. Fetch open tasks

Run `gh issue list --label agent --state open --assignee none --limit 20 --json number,title,state,body,createdAt` to list open GitHub issues with the `agent` label that are **unassigned**. This automatically scopes to the current repository (this project).

Sort by creation date or pick the most relevant. Limit to ~10–20 results.

### 2. Pick highest-priority task and assign

Choose the single most important issue to work on (consider creation date, dependencies, and impact). Use `gh issue view <number>` for full details if needed.

**Assign to yourself:** Run `gh issue edit <number> --add-assignee @me` so the issue is marked in progress and others know it is being worked on.

### 3. Implement

Implement the feature or fix described in the issue. Work only on that task.

### 4. Validate

- Run `bun run typecheck`
- Run `bun run check` (Ultracite)
- Run `bun fix` before committing
- Fix any failures before proceeding. If validation fails, fix the code and re-run — do not stop and ask the user.

### 5. Open Pull Request (do not commit directly to main)

- Ensure you're on the latest main: `git checkout main && git pull origin main`
- Create a branch: `git checkout -b ralph/issue-<number>` (e.g. `ralph/issue-42`)
- Stage changes and commit with a message that references the GitHub issue, e.g. `fix(#42): implement X` or `feat(#42): add Y`
- Use Australian English in commit messages
- Push the branch: `git push -u origin ralph/issue-<number>`
- Create a PR: `gh pr create --title "fix(#<number>): <brief summary>" --body "Fixes #<number>

**RALPH completed this task**

- [Brief summary of changes]"`

The `Fixes #<number>` in the PR body will auto-close the issue when the PR is merged.

### 6. Update GitHub issue

- **Add a comment** via `gh issue comment <number> --body "**RALPH opened a PR for this task**

- [Brief summary of changes]
- PR: [link from gh pr create output]"`

- **Append to progress file:** Append a line to `plans/ralph-progress.txt` with the date, issue number, PR URL (or branch name), and a one-line summary. This helps the next session (or human) pick up where you left off.

### 7. Repeat or exit

**Do not stop** until one of these conditions is met:

- **N iterations reached:** If the user specified a number (e.g. "RALPH for 3 iterations"), stop only after that many tasks are completed.
- **All tasks done:** If there are no more open issues with the `agent` label, output `<promise>COMPLETE</promise>` and stop.

**Otherwise:** Immediately go back to step 1 and fetch the next task. Do not pause, ask for confirmation, or stop early. Continue the loop until a termination condition is satisfied.

### Triggers

Phrases that activate this mode: "run RALPH", "RALPH loop", "RALPH for N iterations", "RALPH until done", "continue RALPH", "continue", "next task", "automate GitHub issues".

### AFK mode (headless)

To run RALPH from the terminal while away (e.g. via SSH from mobile):

1. Install Cursor CLI: `curl https://cursor.com/install -fsS | bash`
2. Set `CURSOR_API_KEY` or run `agent login`
3. Ensure `gh` is installed and authenticated
4. Run: `bun run ralph:afk` or `./scripts/ralph-afk.sh [max-calls]`

The script streams output in real time and retries if the agent stops early. Output `<promise>COMPLETE</promise>` when all tasks are done.
