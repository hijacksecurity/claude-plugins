---
description: Pull this repo's AI Advisors, verify each recommendation against your actual local code/tool state, and report a per-rec verdict (already-mitigated / valid / overstated-FP / partially-done) with local evidence. Read-only.
---

Verify the current repository's Intercept AI Advisors against the real local code using the **advising-on-intercept-recommendations** skill (load **using-intercept-mcp** first).

Steps:
1. Resolve the repo's `repository_id` (`list_repositories`, match the git remote) and pull the advisors with `get_ai_advisors(repository_id)` (add `get_ai_insights` for narrative context). If they aren't computed yet (`{available:false}`), say so and stop.
2. Check for **drift**: compare the run's `commit_sha` to your local `git rev-parse HEAD` and flag it if they differ — the advisors may be judging code you've already changed.
3. Flatten the recommendations from `lenses[].recommendations[]` and the Lead's `action_plan.actions[]`, and classify each as **localizable** or **architectural** (see the skill's guardrail).
4. **Verify locally** — resolve each rec's `cited_finding_ids` (they're `[finding_type, key]` pairs, not UUIDs) to real files where you can, read the *current* code, run the relevant check, and decide the verdict from local truth (not the rec's prose). Catch overstated recs (e.g. a high-priority "JWT alg-confusion" rec is an overstated-FP if HS256 is actually pinned).
5. Emit a **local report only**, grouped by verdict, each rec with its lens, priority, verdict, concrete evidence (file:line / command output), and a recommended next action.

Optional narrowing (a repo name, a lens, or a priority to focus on): $ARGUMENTS

`/advise` is read + verify + report only — it never edits code and never calls `/intercept:fix` itself, and it never writes verdicts back to Intercept (there is no advisor-verdict write path).
