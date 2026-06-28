---
description: Show this repo's Intercept security posture — score, grade, finding breakdown, and what's driving it.
---

Show the current repository's Intercept posture using the **using-intercept-mcp** skill.

Steps:
1. Resolve the current repo to its `repository_id` (`list_repositories`, match the git remote). If it isn't connected to Intercept, say so and point the user to connect it.
2. Call `get_repository_posture(repository_id)` for the score, grade, and per-control pass/fail, and `get_repo_context(repository_id)` for the stack + open finding counts. Optionally `get_ai_insights` / `get_ai_advisors` for the computed narrative (a `{available:false}` is normal).
3. Present: score + grade, per-category open counts, the top failing controls and the most severe open findings (with their `resolution_id` so the user can pipe into `/intercept:fix`), and any "Confirmed by AI" corroboration. Keep it scannable.

Optional focus (e.g. a category): $ARGUMENTS

This is the quickest way to confirm the Intercept connection is working and see where the repo stands.
