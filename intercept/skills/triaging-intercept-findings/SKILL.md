---
name: triaging-intercept-findings
description: Verify open Intercept findings by reading the code and following the data/taint flow, classify each as true/false/unknown, and write the verdict back via update_finding_status. Works on both scanner and AI findings. Use for /intercept:triage or when the user asks to verify, triage, or clear false positives.
---

# Triaging Intercept findings

Pull open findings, verify each against the actual code, and record a verdict. This is where you clear the scanners' false positives and confirm the real ones. Scanner findings and AI findings both surface here as resolutions — triage them uniformly.

Load `using-intercept-mcp` first (especially the **`resolution_id` ≠ `id`** rule — triage writes target `resolution_id`).

## Workflow
1. **Resolve the repo** → `list_repositories` → `repository_id`.
2. **Pull open findings** → `list_findings(finding_type, repository_id, status="open")` for each `finding_type` you're triaging. **Capture each item's `resolution_id`** (the write target). Page until `next_page_cursor` is null if the user wants the full set; otherwise triage the most severe first.
3. **Verify each finding** — read the implicated file/region, follow the data/taint flow from source to sink, check whether the guard/validation the finding claims is missing is actually missing. Decide: **true** (real), **false** (false positive), or **unknown** (can't determine without runtime/more context).
4. **Write the verdict:**
   - `update_finding_status(resolution_id, status, notes)` where `status` ∈ `false_positive | accepted_risk | mitigated | wont_fix | fixed | open`.
   - `notes` = a **factual** justification (≤4000 chars): what you checked and why it's a FP / why it's real. No `reasoning`/`confidence` fields — the justification goes in `notes` as prose, keep your chain-of-thought in the conversation.
   - When several findings share a verdict (e.g. a whole rule firing as FP across files), use `bulk_update_finding_status(resolution_ids, status, notes)` (≤500 ids).
   - For a real finding you're leaving open (e.g. needs a fix, not just a status), use `comment_on_finding(resolution_id, note)` to record the triage without closing it.
5. **Summarize** — counts by verdict, the FPs you cleared, and the confirmed-real findings worth fixing (hand those to `/intercept:fix`).

## Discipline
- Be conservative marking `false_positive` — only when you've actually traced the code and the finding is wrong. "Looks noisy" is not a verdict. When you can't tell, leave it `open` with a `comment_on_finding` note rather than guessing.
- Don't change a status the user set deliberately (e.g. `accepted_risk`) without their say-so.
- An AI finding marked `false_positive` is excluded from the score/compliance immediately — so accurate triage directly improves the user's posture signal.
