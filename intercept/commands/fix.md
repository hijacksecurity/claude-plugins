---
description: Fix a specific Intercept finding end-to-end — verify, apply the fix, prove it, mark it resolved.
---

Fix the specified finding using the **fixing-intercept-findings** skill (load **using-intercept-mcp** first).

Steps:
1. Locate the finding (by the id/description below) — `list_findings` finds any type; capture its **`resolution_id`**, its **`remediation`** (the fix guidance), and — for code-located findings — its `file_path` + line. Works for every category (code, container/image, pipeline, infrastructure, packages, platform), not just code findings.
2. Verify it's real; if it's a false positive, mark it `false_positive` and stop.
3. Apply the fix the finding's type calls for, driven by its `remediation`:
   - **Code-located** (sast / iac / dockerfile-source / pipeline — has a `file_path`+line): edit the code at that location — the minimal root-cause fix.
   - **Image CVE** (container, `location_kind=image_cve`, no source line): the fix is an **upgrade, not a code edit** — bump/repin the base image in the relevant Dockerfile (`FROM` → patched tag/digest) and/or add the package upgrade per `remediation`, then rebuild.
   - **Dependency CVE** (packages / sbom_vuln): bump the package version in the manifest (requirements.txt / package.json / go.mod …) to the fixed version in `remediation`.
   - **Platform**: apply the repo/org config or settings change in `remediation` (may be a platform setting, not a repo edit — guide the user if so).
   Then **prove it** (project tests / a targeted test / build). For a secret, confirm removal and remind the user to rotate it. **If the fix touched any `.github/workflows/**` file, run `actionlint` on it and it MUST pass before you mark the finding fixed** — a malformed `${{ }}` still parses as YAML but causes a GitHub Actions `startup_failure`. If `actionlint` isn't installed, install it (or fall back to `yamllint` and note that in the resolution); never declare a workflow edit done while `actionlint` reports errors.
4. Mark `update_finding_status(resolution_id, "fixed", notes="<what changed + how verified>")`, then optionally `trigger_scan` to let the scanners confirm closure.
5. Report the fix, the verification result, and the new status.

Finding to fix (id or description): $ARGUMENTS

Never weaken a check or suppress a finding to make it pass — fix the underlying issue.
