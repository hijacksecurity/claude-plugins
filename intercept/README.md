# Intercept for Claude Code

Deep AI security scanning that **actively digs into the things you care about** — then writes
findings back to the [Intercept](https://intercept.hijacksecurity.com) platform as your system of
record.

Rules-based scanners (opengrep, gitleaks, grype, trivy, checkov) are great at static analysis.
This plugin sends Claude Code after what they **can't reach** — and lets it *actually scan*, not
just read code: **build and inspect your container images** (layer-by-layer, finding vulnerable
packages and bad base images), **query your live cloud config read-only** (with your consent), run the tools
you already have, and reason across files for logic and auth-flow bugs. It scans what **you and
Claude Code can access and you're willing to share** — and asks before touching anything live. New
findings land in Intercept alongside your scanner findings — deduplicated, badged "AI", tracked, and
first-class in your posture score and compliance.

## Install (under 5 minutes)

```bash
# 1. Add the marketplace and install the plugin
claude plugin marketplace add hijacksecurity/claude-plugins
claude plugin install intercept@hijacksecurity

# 2. Log in (no API key — OAuth browser login)
/mcp          # opens a browser → log into Intercept → Approve

# 3. See your posture, then scan
/intercept:posture
/intercept:scan
```

That's it — no API key, no config. The plugin wires up the Intercept MCP server automatically;
you approve consent once in the browser, and the commands figure out the rest (which repo you're
in, what to scan) on their own.

**New repo?** If the repo you're in isn't connected to Intercept yet, connect it once at
[intercept.hijacksecurity.com](https://intercept.hijacksecurity.com) (Repositories → Connect).
`/intercept:scan` will still review your code locally and upload the findings as soon as it's
connected — it won't leave you stuck.

## Commands

| Command | What it does |
|---|---|
| `/intercept:posture` | Score, grade, finding breakdown, and what's driving it |
| `/intercept:scan` | Deep AI scan supplementing your scanners; records new findings |
| `/intercept:triage` | Verify open findings, clear false positives, confirm real ones |
| `/intercept:fix <finding>` | Fix a finding end-to-end and mark it resolved |
| `/intercept:hunt <description>` | Natural-language hunt for a vulnerability class |

## How it works

- **OAuth, tenant-scoped.** Browser login, no API key. You only ever touch your own tenant.
- **Supplements, never replaces.** If your finding matches a scanner finding, Intercept suppresses
  the duplicate and marks the scanner finding "Confirmed by AI" — so re-running is safe and
  focuses on what scanners missed.
- **Stack-aware.** `/intercept:scan` detects your stack (Docker, Terraform/CDK, Vercel, CI),
  asks the platform what it already knows (`get_repo_context`), and asks you only the 1-2 things it
  genuinely can't infer (deploy target, prod vs staging, cloud-posture scope). It **never writes
  files into your repo** — the scan is read-only.
- **Branch-aware.** Scan your **default branch** for the posture-of-record; scan a feature branch
  as a pre-merge check. Findings record the branch + commit they were observed on.
- **First-class.** AI findings count toward your Intercept Score and compliance controls, badged
  so you always know what came from AI vs a scanner. Use the Scanner-only filter for an
  audit-grade, reproducible view.

## Safety & privacy

- **Secrets stay out of it.** The AI plugin does **not** look for or report secrets — that's the
  static scanner's (gitleaks) job — and it never transmits a secret value; any secret literal in a
  code snippet is redacted before upload. Intercept never stores AI reasoning.
- **Prompt-injection resistant.** Scanned code (including third-party deps and contributor PRs) is
  treated as untrusted data, never as instructions.
- **You pay for your own Claude usage.** Intercept charges nothing per scan.

## Optional: proactive mode

Paste `fragments/CLAUDE.md.snippet` into your repo's `CLAUDE.md` if you want Claude Code to
proactively consult Intercept on security-sensitive changes. Opt-in.

---

Issues and feedback: <https://github.com/hijacksecurity/claude-plugins/issues>
