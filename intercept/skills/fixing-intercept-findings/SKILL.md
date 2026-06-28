---
name: fixing-intercept-findings
description: Fix a specific Intercept finding end-to-end — verify it's real, propose and apply the code fix, run available tests, mark it resolved, and optionally trigger a rescan. Use for /intercept-fix or when the user asks to fix or remediate a specific finding.
---

# Fixing an Intercept finding

Take one finding from open to fixed: verify, fix the code, prove it, record it.

Load `using-intercept-mcp` first (the `resolution_id` ≠ `id` rule applies when you mark it resolved).

## Workflow
1. **Locate the finding** — if the user gave a finding id, fetch it (`get_sast_finding` / `get_secrets_finding` / `get_sbom_vuln_finding`, or `list_findings` to find it). Capture its `file_path`, `line_start`, and its **`resolution_id`**.
2. **Verify it's real** (reuse `triaging-intercept-findings` logic). If it's a false positive, mark `update_finding_status(resolution_id, "false_positive", notes=...)` and stop — don't "fix" a non-bug.
3. **Propose the fix** — read the surrounding code, design the minimal correct fix that addresses the root cause (not just the symptom). Explain it to the user.
4. **Apply the edit** — make the change in the working tree. Keep it focused and consistent with the surrounding code.
5. **Prove it** — run whatever's available: the project's tests, a targeted test, a build/lint. For a secret, confirm it's removed from the code and (remind the user) rotated. Don't claim "fixed" without evidence.
6. **Record it** — `update_finding_status(resolution_id, "fixed", notes="<what changed + how verified>")`. Factual notes only.
7. **Optionally rescan** — `trigger_scan(repository_id)` so the scanners confirm closure on their own next pass (the AI fix-mark and the scanner reconcile are independent paths; a rescan reconciles them).
8. **Report** — the fix, the verification result, and the new status. If you couldn't fully verify (e.g. no test harness), say so plainly and mark accordingly rather than overstating.

## Discipline
- One finding at a time, fully. Don't batch-fix blindly across many findings.
- If the fix is risky or spans a lot of code, propose it and get the user's go-ahead before applying.
- Never weaken a check or suppress a finding to make it "pass" — fix the underlying issue.
