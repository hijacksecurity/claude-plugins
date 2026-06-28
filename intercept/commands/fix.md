---
description: Fix a specific Intercept finding end-to-end — verify, apply the fix, prove it, mark it resolved.
---

Fix the specified finding using the **fixing-intercept-findings** skill (load **using-intercept-mcp** first).

Steps:
1. Locate the finding (by the id/description below), capturing its `file_path`, line, and `resolution_id`.
2. Verify it's real; if it's a false positive, mark it `false_positive` and stop.
3. Propose the minimal root-cause fix, apply the edit, and prove it (run the project's tests / a targeted test / build). For a secret, confirm removal and remind the user to rotate it.
4. Mark `update_finding_status(resolution_id, "fixed", notes="<what changed + how verified>")`, then optionally `trigger_scan` to let the scanners confirm closure.
5. Report the fix, the verification result, and the new status.

Finding to fix (id or description): $ARGUMENTS

Never weaken a check or suppress a finding to make it pass — fix the underlying issue.
