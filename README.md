# Skills

## RALPH (`cursor/rules/ralph.mdc`)

**RALPH** is an agent rule that automates a **GitHub issue → implement → validate → PR → repeat** loop.

| | |
| --- | --- |
| **What it does** | Picks open issues labelled `agent`, assigns one to you, implements it, validates (per your rule/project), opens a PR on `ralph/issue-<n>`, comments on the issue, then moves to the next issue. Keeps going until you hit an iteration limit or no agent work remains. |
| **How to run** | Say things like *run RALPH*, *RALPH loop*, *RALPH for N iterations*, *continue RALPH*, *next task*. |
| **You need** | [GitHub CLI](https://cli.github.com/) (`gh`) authenticated. |

Your repo **does not have to match** the example workflow in the rule (specific scripts, commit style, etc.). Treat those as a template and swap in whatever fits the project.

**Recommended:** Bake in a **validate-before-ship** habit—run tests, typecheck, lint, or whatever your stack uses, and fix failures **before** opening PRs or pushing commits. The rule may spell out one command; adapt it so the agent repeatedly checks until green, not only right at PR time. Resolving these issues up front is far preferable to making follow-up commits to fix broken PRs after the fact.

Drop `ralph.mdc` into **Cursor rules** (or any agent that reads markdown rules, just change the extension). Tested with **Cursor CLI**; the same instructions work anywhere the agent can run shell + `gh`. Loops work well with the Cursor CLI because it tends not to stop and ask whether to continue.
