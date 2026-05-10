---
description: Independent verification pass over the reconciled design. Checks consistency, alignment, completeness, and decision encoding.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:verify

Run the verification stage. Wraps Template 12.

**Important:** This stage normally runs in a fresh chat (Chat E in the original process) so the verifier isn't biased by having generated the design. In a single-agent plugin context, simulate that independence: read the inputs cleanly, do not rely on prior conversational reasoning, and treat the design document as something you're seeing for the first time.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. The required prerequisite is `problem` (or `ui-draft` if user-facing). Refuse unless those are completed or `--force`.

## Step 2 — Load inputs

The reconciled design in v1 (no crossfire yet) is the problem statement plus the UI Contract:

- `.feature-design/<slug>/03-problem-statement.md` — the design's claims
- `.feature-design/<slug>/04-ui-contract.md` (if `state.user_facing === true`)
- The source template `12 - VERIFICATION_PROMPT.md`. Use Glob with pattern `**/12 - VERIFICATION_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

When crossfire support is added later, this stage should consume the final reconciled design (output of Template 11) instead.

## Step 3 — Substitute placeholders

- `{{RECONCILED_DESIGN_DOCUMENT_OR_PATH}}` → the contents of `03-problem-statement.md` plus `04-ui-contract.md` (if present), concatenated with section dividers
- `{{DECISIONS_DOCUMENT_OR_PATH}}` → state that no formal decision document exists yet (crossfire deferred); the verifier should flag any internal inconsistencies in the problem/UI artifacts on their own merits
- `{{PRD_PATH}}`, `{{CODEX_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}` from state

## Step 4 — Execute

Produce the verification report per the template's `## 6. Dimensions`:

- **Internal consistency** — components fit together, mutual exclusion preserved, state machines complete, no orphan fields, all flows handled
- **External alignment** — design doesn't contradict the PRD, Codex, or Design Philosophy (cite specific sections)
- **Completeness** — handled paths, failure modes, integration points, deferred-table rationale, success metrics
- **Decision encoding** — N/A for v1 since crossfire decisions don't exist yet; note this limitation in the report
- **Screen Contract alignment** (if user-facing) — hierarchy preserved, required states handled, anti-goals respected, layout skeleton intact

**Behavioral rules:**
- Be specific — cite document and section
- Don't redesign or propose alternatives
- Report findings of "no issue" too
- Don't relitigate decisions, only flag logical impossibilities

## Step 5 — Write output

Write to `.feature-design/<slug>/07-verification.md` with this header:

```markdown
# Verification Report: <feature_name>

**Date:** <ISO date>
**Source:** Template 12 — VERIFICATION_PROMPT.md
**Session:** <slug>
**Verifier mode:** Single-agent independent pass (crossfire deferred)

---

<verification report body>
```

Output structure (per template `## 7`):
- Summary (one paragraph, ready-or-not)
- Findings (each with: dimension, severity, finding, recommendation)
- Clean dimensions
- Missing from deferred table

## Step 6 — Update state

- Append `"verify"` to `completed_stages`
- Set `current_stage` to `"spec"`

## Step 7 — Report

```
Verification complete → 07-verification.md
Blockers: <count>
Warnings: <count>
Notes: <count>
Clean dimensions: <list>

Recommended next: /fd:next  (will run /fd:spec)
```

If any blockers were found, recommend the user resolve them in the upstream artifacts before proceeding to spec.
