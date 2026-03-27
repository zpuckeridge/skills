---
name: review
description: >-
  Review a git diff—summarizes changes, audits for bugs, security, and lists findings by severity. Use for code review or risk checks.
---

1. Summarise what changed in 2–4 bullets (intent + scope only).
2. Review the diff for: correctness bugs, behavioural regressions, security (auth, data exposure, injection, secrets), concurrency/races, error handling, and missing or inadequate tests for new behaviour.
3. List findings ordered by severity (Critical → High → Medium → Low). Each item: what, where (file/function if obvious from diff), why it matters, suggested fix (one sentence).

   **Severity rubric** (one line each—use for consistent ordering):
   - **Critical:** Exploitable security issue, secret exposure, guaranteed data loss or corruption, or a change that will clearly break production for typical use.
   - **High:** Likely bug or regression, broken authz/authn boundary, injection or unsafe deserialization, race that can corrupt state, or missing coverage for high-risk new behaviour.
   - **Medium:** Possible bug in an edge case, fragile error handling, performance or reliability concern, or tests that should exist but gaps are not catastrophic.
   - **Low:** Style, naming, minor clarity, optional hardening, or nitpicks with little practical risk.

4. If the diff is too large, focus on the riskiest hunks first and say what you did not fully cover.
5. Do not rewrite the code or apply fixes unless the user explicitly asks for patches or edits.