---
name: ship
description: >-
  Ship or verify: code-review changed files and apply safe fixes, then run
  checks (Turbo monorepo vs single package), stage, semantic commit, and push.
  Use when the user says ship or verify (not generic deploy/push). Commit is
  the last deploy step; build runs remotely.
---

# Ship & verify

**Do not run a `ship` shell script.** Follow this skill so steps stay visible and fixable in chat.

**Ship** = Step 0 review → checks → commit → push (user asked to ship).

**Verify** = Step 0 review → checks only (no commit/push).

Triggers: **ship** and **verify** are different intents; do not treat casual “push” or “deploy” as this workflow unless the user clearly means the same thing.

---

## 0. Code review (first)

Before automated checks, **review the working changes** (diff and/or files from `git status` relevant to this ship).

1. **Review:** correctness, edge cases, security (auth, injection, secrets), consistency with surrounding code, obvious bugs, and small improvements that fit the diff.
2. **Implement** straightforward fixes and reasonable improvements inline—match existing style; keep scope tight.
3. **Escalate:** if something **critical** needs a product/architecture/security decision (breaking behavior, ambiguous requirements, risky tradeoffs), **stop and ask the user** before proceeding—do not guess past blockers.

Include Step 0 in **ship** and **verify** flows.

---

## 1. Repo shape

- **Turbo monorepo:** `turbo.json` (or `turbo.jsonc`) at root and workspace apps under `apps/<name>/`. Use Turbo filters / workspace commands for scoped work.
- **Otherwise:** **Single package** — root `package.json` only; no Ultracite/Turbo assumptions.

Infer **which apps** matter from **`git status`** (paths under `apps/`). No interactive app picker; no `deploy` script in `package.json` as a gate—**git changes define scope**.

If Turbo package names differ from folder names, read `apps/<dir>/package.json` `name` for `--filter` / workspace selectors.

---

## 2. Checks (order)

Run only what exists in the relevant `package.json` files. Prefer **bun** when the repo uses Bun; else use the repo’s package manager.

**Single package (root):**

1. Format: `fix` → else `format`
2. Lint: `check` → else `lint`
3. `typecheck` if present
4. Convex dry-run — see below
5. Build: `build:opennext` → else `build` (remote build still validates locally when this script exists)
6. Tests: `test` script → else `bun test` if `test/` or `tests/` exists
7. Security: lockfile present → `bun audit --audit-level=moderate`

**Turbo monorepo:** For each app that has changes (from git status), run the same *categories* using Turbo/workspace (e.g. ultracite on changed app paths, `turbo run typecheck --filter …`, per-app Convex from `apps/<app>`, `turbo run build:opennext --filter …`, then root `bun test` and root `bun audit` as in your pipeline). If nothing matches a step, skip it.

### Convex

Include dry-run only when **`CONVEX_DEPLOYMENT` is set** and the app has Convex scripts or artifacts (`convex.config.*`, `convex.json`, `convex/`).

If Convex applies but **`CONVEX_DEPLOYMENT` is missing**, **warn loudly** and do not pretend the step ran.

### Secrets

Never print env values, tokens, or deployment URLs in logs or chat.

### Soft outcomes (must acknowledge)

- **Audit failed:** Stop; show summary; **require explicit user confirmation** before commit/push.
- **No tests found:** One-line acknowledgment in the summary, then continue.

---

## 3. Git

### Scope

- **Changed apps** from `git status` under `apps/…`.
- **Always include** root `package.json`, lockfiles, and workspace/turbo config if they appear in status—same logical ship as app changes.

### Commit

- **Semantic messages** from the diff (`feat` / `fix` / `chore` / … per conventional commits), not `chore` for everything.
- **`git commit`:** use hooks by default (`--no-verify` **only if the user explicitly asks** to skip hooks).

### Push

- After commit, `git push` (no `--no-verify` unless the user explicitly asked).

### Nothing to do

- **No changes** under scope → no empty commit; say clearly **nothing staged**.
- **Nothing to push** (clean and up to date) vs **unpushed commits** — state which case applies.

### Merge

- If on a **branch** (not `main`) and the user **instructed merge**, complete merge (e.g. PR via `gh` or local merge into **`main`**). If merge intent is unclear, ask.

Default branch name: **`main`**.

---

## 4. Agent summary

After running: brief **review** notes (issues found, fixes applied, anything left for the user), then each **check** pass/fail, Convex warning if env missing, audit/no-test acknowledgments, confirmation obtained or not, commit message, commit hash, push result, and merge result if applicable.
