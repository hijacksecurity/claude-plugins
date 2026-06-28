---
description: Pull open Intercept findings, verify each against the code, and clear false positives / confirm real ones.
---

Triage the open findings for the current repository using the **triaging-intercept-findings** skill (load **using-intercept-mcp** first).

Steps:
1. Resolve the repo's `repository_id` and pull open findings per category, capturing each item's `resolution_id` (the triage write target — never the finding `id`).
2. For each finding, read the implicated code and follow the data/taint flow; decide true / false / unknown.
3. Write the verdict with `update_finding_status(resolution_id, status, notes)` (factual notes), or `bulk_update_finding_status` when many share a verdict. Leave a `comment_on_finding` note on real findings you're keeping open.
4. Summarize: false positives cleared, real findings confirmed (hand those to `/intercept-fix`).

Optional scope (e.g. a category or severity to focus on): $ARGUMENTS

Be conservative marking false positives — only after actually tracing the code.
