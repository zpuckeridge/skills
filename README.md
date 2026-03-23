# Skills

Skill definitions live in **`skills/<name>/SKILL.md`**. Copy or symlink a folder into **Cursor**’s skills location (for example `~/.cursor/skills/<name>/` for personal use, or `.cursor/skills/<name>/` inside a project) so the agent can discover them.

---

## Ship (`skills/ship/SKILL.md`)

**Ship** tells the agent how to **review changes**, run **checks** (format, lint, types, optional Convex, build, tests, audit—Turbo monorepo vs single package), then **stage, semantic commit, and push**. **Verify** is the same pipeline **without** commit/push.

| | |
| --- | --- |
| **What it does** | Step 0: code review changed files, fix what’s safe, escalate critical decisions. Then run the check pipeline; handle Convex env warnings, audit confirmation, and “no tests” acknowledgments. Git: scope from `git status`, include root/lockfile/workspace changes when relevant; hooks on by default unless you ask to skip. |
| **How to run** | Say *ship* or *verify* (not vague “just push”). Merge into `main` only if you asked to merge while on a branch. |
| **You need** | A repo that matches the patterns in the skill (or adapt the skill). Convex dry-run needs `CONVEX_DEPLOYMENT` when applicable. |

The skill is **instructions for the agent**, not a shell script—commands run in chat so you can see and fix failures.

### Rule or skill?

**Use the skill** for this workflow. Skills are **on-demand**: Cursor matches them when you say *ship* / *verify*, so the long procedure is not injected into every conversation.

**Use a rule** if you want something **always on**—for example always-apply coding standards, or a one-line pointer: “When the user says ship or verify, follow the Ship skill.” A full ship checklist as an always-apply rule is usually **too heavy** and noisy; the skill keeps scope tight.

---

## RALPH (`skills/ralph/SKILL.md`)

**RALPH** is a skill that automates a **GitHub issue → implement → validate → PR → repeat** loop (issues labelled `agent`, `gh` for listing/editing/PRs).

| | |
| --- | --- |
| **What it does** | Fetches open `agent`-labelled issues, assigns one, implements it, validates, opens a PR on `ralph/issue-<n>`, comments on the issue, repeats until a limit or no work remains. See the skill file for branch naming, merge rules, and “don’t ask to continue” behavior. |
| **How to run** | Say things like *run RALPH*, *RALPH loop*, *RALPH for N iterations*, *continue RALPH*, *next task*. |
| **You need** | [GitHub CLI](https://cli.github.com/) (`gh`) authenticated (`gh auth status`). Run from the repo root. |

Your target repo **does not have to match** every detail in the skill (scripts, validate commands). Treat it as a template and adapt.

**Recommended:** Bake in a **validate-before-ship** habit—run tests, typecheck, lint, or whatever your stack uses, and fix failures **before** opening PRs or pushing. Resolving failures up front beats follow-up “fix CI” commits.

For long loops, **Cursor CLI** pairs well because it tends not to stop and ask whether to continue.
