---
description: RALPH loop - fetch GitHub issues, implement, validate, open PR, repeat
---

## RALPH Loop (GitHub Issue Automation)

When the user asks to "run RALPH", "RALPH loop", "RALPH for N iterations", "continue RALPH", "continue", or similar, execute this workflow in a loop. For large or multi-part issues, consider using worktrees or subagents to parallelise work. Prefer completing one issue per iteration unless the issue explicitly describes multiple sub-tasks.

**Loop until done:** Do not stop until either (a) the requested number of iterations is reached, or (b) all open agent-labelled issues are complete. Never stop after a single task unless one of those conditions is met.

**Never ask for confirmation:** Do not output "Should I continue?", "Ready for the next task?", or similar. Just proceed to the next task.

**Merge your own pull requests when necessary:** If progress on a task is blocked by pending features in existing pull requests, merge those pull requests to unblock the workflow. Before declaring COMPLETE, always merge any open RALPH PRs (branches `ralph/issue-*`) that are mergeable — this may unblock other agent issues.

If branch `ralph/issue-<number>` already exists, delete it locally and remotely first, or use `ralph/issue-<number>-v2` as a fallback.

Ensure a clean working directory. If there are uncommitted changes, stash them (`git stash`) or commit them before creating the branch.

**Prerequisites:** GitHub CLI (`gh`) must be installed and authenticated. Run from the repo root so issues are scoped to this project. Verify `gh auth status` succeeds before starting — if not, RALPH cannot proceed.

### 1. Fetch open tasks

Run `gh issue list --label agent --state open --assignee none --limit 20 --json number,title,state,body,createdAt` to list open GitHub issues with the `agent` label that are **unassigned**. This automatically scopes to the current repository (this project).

If no unassigned issues are returned, run `gh issue list --label agent --state open --assignee @me --limit 20 --json number,title,state,body,createdAt` to pick up issues RALPH previously assigned but did not complete.

Sort by creation date or pick the most relevant. Limit to ~10–20 results.

### 2. Pick highest-priority task and assign

Choose the single most important issue to work on (consider creation date, dependencies, and impact). Use `gh issue view <number>` for full details if needed.

**MUST assign to yourself before implementing:** Run `gh issue edit <number> --add-assignee @me` so the issue is marked in progress. Do NOT proceed to step 3 until assignment succeeds.

### 3. Implement

Implement the feature or fix described in the issue. Work only on that task.

### 4. Validate

- Run `bun run ship --check` (typecheck, lint, format — same as pre-commit)
- Fix any failures before proceeding. If validation fails, fix the code and re-run — do not stop and ask the user.

### 5. Open Pull Request (do not commit directly to main)

- Ensure you're on the latest main: `git checkout main && git pull origin main`
- Create a branch: `git checkout -b ralph/issue-<number>` (e.g. `ralph/issue-42`)
- Stage changes and commit with a message that references the GitHub issue. Use conventional commit types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, etc., depending on the change (e.g. `fix(#42): implement X`, `feat(#42): add Y`)
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

### 7. Repeat or exit

**Do not stop** until one of these conditions is met:

- **N iterations reached:** If the user specified a number (e.g. "RALPH for 3 iterations"), stop only after that many tasks are completed.
- **All tasks done:** If there are no more open issues with the `agent` label, output `COMPLETE` and stop.

**When no unassigned or assigned-to-me issues are found:** Do **not** declare COMPLETE yet. First:

1. Run `gh pr list --author @me --base main --state open --json number,headRefName,mergeable` to list your open PRs (RALPH creates these).
2. For each PR with `mergeable` equal to `"MERGEABLE"`, run `gh pr merge <number> --squash` to merge it. (If `mergeable` is `"UNKNOWN"`, wait a moment and re-check; if `"CONFLICTING"`, resolve conflicts first.)
3. Go back to step 1 and fetch tasks again — merging may have unblocked issues or closed some via "Fixes #N".
4. Only output `COMPLETE` when there are truly no open agent-labelled issues, or when you have merged all RALPH PRs and re-checked with no new work available.

**Otherwise:** Immediately go back to step 1 and fetch the next task. Do not pause, ask for confirmation, or stop early. Continue the loop until a termination condition is satisfied.

### Triggers

Phrases that activate this mode: "run RALPH", "RALPH loop", "RALPH for N iterations", "RALPH until done", "continue RALPH", "continue", "next task", "automate GitHub issues".
