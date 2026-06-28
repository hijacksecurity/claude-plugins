---
name: intercept-stack-context
description: Establish what's security-relevant about this repo before an Intercept scan or hunt — detect the stack from the code, ask the platform (get_repo_context) what it already knows, and confirm only the gaps it can't infer. Never writes anything into the repo. Use at the start of /intercept-scan and /intercept-hunt, or when the user mentions where they deploy, what they build, or their cloud/runtime.
---

# Intercept stack context: detect → ask the platform → confirm

Before scanning, figure out what actually matters for THIS repo so the scan targets the right risks (image contents, cloud posture, the right deploy surface) instead of guessing. Do it without making the user fill out a questionnaire and **without writing anything into their repo**: detect what you can, ask the platform what it knows, and confirm only the genuine gaps.

## 1. Detect (local — every run, zero questions, read-only)
Scan the working tree for stack signals and tell the user what you found:

| Signal | Inferred aspect | Emphasize in the scan |
|---|---|---|
| `Dockerfile`, `*.dockerfile`, `Containerfile` | builds a container image | image-content review → `dockerfile` findings |
| `*.tf`, `cdk.json`, `Pulumi.yaml`, `serverless.yml`, `samconfig.toml` | IaC / cloud provisioning | cloud/infra posture → `iac` findings |
| `vercel.json`, `netlify.toml`, `app.yaml`, `fly.toml` | PaaS deploy target | deploy-surface review |
| cloud SDK imports (`boto3`, `@aws-sdk/*`, `google-cloud-*`, `@azure/*`) | cloud runtime | SSRF / credential / IAM emphasis |
| `.github/workflows/*`, `.gitlab-ci.yml`, `azure-pipelines.yml`, `Jenkinsfile` | CI/CD present | pipeline permissions/posture → `pipeline` findings |
| `k8s/`, manifests with `kind:` | Kubernetes | container/platform review |
| `package.json` / `requirements.txt` / `go.mod` / `pom.xml` / `Cargo.toml` | language + deps | `sast` / `sbom_vuln` emphasis |

## 1b. Detect available access & tooling (what you could dig into)
The deep value comes from *actively* scanning — pulling the real image, querying live cloud — not just reading files. So check what's available on this machine (don't run anything live yet; just check what exists):

| Check | Enables |
|---|---|
| `docker` / `podman` present and the daemon reachable | building/pulling images, inspecting layers, extracting the filesystem |
| `aws` / `gcloud` / `az` CLI present **and** credentials configured | live, **read-only** cloud-posture queries |
| `kubectl` with a reachable context | live cluster/workload inspection |
| local `trivy` / `grype` / `syft` / `checkov` | running a real scan of the built image / tree and reasoning over results |
| registry access to the image refs the project uses | pulling published images to inspect |

Note which are present — they define what an active scan *could* do. Don't use them until the user consents (step 3).

## 2. Ask the platform what it already knows
Call `get_repo_context(repository_id)` (resolve the repo via `list_repositories` first). It returns the primary language, the pipeline **activity tags** Intercept already detected (e.g. `build:docker`, `deploy:aws`, `iac:terraform`), whether the repo has Dockerfiles/IaC, the open finding counts per category, and the current score+grade. Where this agrees with your local detection, you're confident; where it shows something you haven't scanned, that's a prompt to scan it.

**The platform is the memory.** Intercept already remembers what it knows about the repo — that's what `get_repo_context` is for. You do **not** keep a local memory of your own.

## 3. Confirm scope + consent (adaptive — ask only the gaps; don't persist to the repo)
After detect + the platform view, ask the user **only what you genuinely could not infer** and that changes the scan. Two kinds of question:

**What's relevant (scope)** — ask when the repo doesn't make it clear:
- **Where is the app deployed?** (AWS / GCP / Azure / Vercel / Fly / Kubernetes / on-prem) — if not obvious from the repo.
- **How and where is the image built and published?** (CI or local; which registry; is the running image different from this repo's Dockerfile?)
- **Which environment does the default branch represent** — prod or staging?
- "I see Terraform + `boto3` — is **live AWS cloud-posture** in scope, or repo-only?"

Prefer to **infer** these from the repo; when you can't, **just ask**. A couple of well-targeted questions makes the scan far more accurate.

**What you may do (consent for active scanning)** — only ask about capabilities you actually detected in step 1b, and only if the element is in scope:
- "You have Docker — may I **build/pull and inspect the image** (`docker history`, extract the filesystem) for vulnerable packages, a bad/EOL base image, and running-as-root?" (Secret detection is out of scope — the static scanner owns that.)
- "Your AWS CLI is authenticated — may I run **read-only** posture queries against the live account (S3 policies, IAM, security groups)? I will never change anything."
- "You have `trivy` installed — may I run it against the built image and reason over the results?"

**Default to the safe interpretation** (read repo files only) until the user grants more. **Use the answers for this scan only — do NOT write them to any file in the repo.** Never create a `.intercept/` directory, a context file, or any config in the user's working tree; the scan is read-only and must not modify the repo. Re-asking one or two quick questions occasionally is fine; polluting the repo is not. (If cross-scan memory is wanted later, it belongs on the platform, not the repo.)

## Output of this step
A short "scan plan" you show the user before scanning: detected stack, what the platform already knows (`get_repo_context`), any answers you just collected this session, and which categories you'll emphasize — especially the scanner-can't-reach ones: image contents, cloud/infra posture, cross-file logic.
