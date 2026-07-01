---
name: fixing-intercept-findings
description: Fix a specific Intercept finding end-to-end — verify it's real, propose and apply the code fix, run available tests, mark it resolved, and optionally trigger a rescan. Use for /intercept:fix or when the user asks to fix or remediate a specific finding.
---

# Fixing an Intercept finding

Take one finding from open to fixed: verify, fix the code, prove it, record it.

Load `using-intercept-mcp` first (the `resolution_id` ≠ `id` rule applies when you mark it resolved).

## Workflow
1. **Locate the finding** — **`list_findings` is the reliable locator for every category** (`sast`, `secrets`, `container`, `iac`, `pipeline`, `sbom_vuln`). By-id getters exist only for some types (`get_sast_finding` / `get_secrets_finding` / `get_sbom_vuln_finding`); **container and pipeline have no by-id getter — locate them via `list_findings`.** Capture its **`resolution_id`** (needed to mark it resolved), its **fix guidance** (the field name varies by type — `remediation`, `resolution`, or `fix_suggestion` — and it's always restated in the `description`), and — for code-located findings — its `file_path` + `line_start`. `file_path` is absent for image CVEs and dependency CVEs (their fix isn't a source-line edit, see step 4).
2. **Verify it's real** (reuse `triaging-intercept-findings` logic). If it's a false positive, mark `update_finding_status(resolution_id, "false_positive", notes=...)` and stop — don't "fix" a non-bug.
3. **Propose the fix** — design the minimal correct fix that addresses the root cause (not just the symptom), guided by the finding's `remediation`/description. Explain it to the user before applying anything risky or wide.
4. **Apply the fix — the action depends on the finding type:**
   - **Code-located** (`sast` / `iac` / `dockerfile` with `location_kind=source` / `pipeline` — has `file_path`+line): read the surrounding code and make the minimal, root-cause code edit in the working tree, consistent with the surrounding code.
   - **Image CVE** (`dockerfile`, `location_kind=image_cve` — no source line): the fix is an **upgrade, not a code edit** — repin/bump the base image in the relevant Dockerfile (`FROM` → a patched tag or digest) and/or add the package upgrade the `remediation` names, then rebuild. There is no single vulnerable line to edit; you're changing what the image is built from.
   - **Dependency CVE** (`sbom_vuln` / Packages): bump the package to the fixed version in the manifest (`requirements.txt` / `package.json` / `go.mod` / `Cargo.toml` …) per `remediation`.
   - **Platform** (org/repo posture): the fix is usually a platform/settings change, not a repo edit — guide the user to apply it. Note `/intercept:fix` can't currently locate or mark-resolve a platform finding through the MCP tools, so after the user applies the change, have them resolve it in the Intercept web UI.
5. **Prove it** — run whatever's available: the project's tests, a targeted test, a build/lint. For a secret, confirm it's removed from the code and (remind the user) rotated. Don't claim "fixed" without evidence.
   - **CI-workflow gate (hard requirement):** if the fix touched any file under `.github/workflows/**`, run `actionlint <changed-workflow-file>` and it MUST pass before this finding can be marked fixed. A malformed `${{ }}` expression parses as valid YAML but causes a GitHub Actions `startup_failure` at runtime — plain YAML validation won't catch it, `actionlint` will. If `actionlint` isn't installed, install it (`brew install actionlint` / `go install github.com/rhysd/actionlint/cmd/actionlint@latest`) or, if you can't, fall back to `yamllint` and state in the notes that actionlint was unavailable. Never declare a workflow edit done while `actionlint` reports errors.
6. **Record it** — `update_finding_status(resolution_id, "fixed", notes="<what changed + how verified>")`. Factual notes only.
7. **Optionally rescan** — `trigger_scan(repository_id)` so the scanners confirm closure on their own next pass (the AI fix-mark and the scanner reconcile are independent paths; a rescan reconciles them).
8. **Report** — the fix, the verification result, and the new status. If you couldn't fully verify (e.g. no test harness), say so plainly and mark accordingly rather than overstating.

## Discipline
- One finding at a time, fully. Don't batch-fix blindly across many findings.
- If the fix is risky or spans a lot of code, propose it and get the user's go-ahead before applying.
- Never weaken a check or suppress a finding to make it "pass" — fix the underlying issue.
- Any edit to a `.github/workflows/**` file must pass `actionlint` before you mark the finding fixed (see step 5). No exceptions — a `startup_failure` from a bad workflow edit can break a live pipeline.
