---
description: Triage a feature idea against canonical project docs to determine pipeline entry point.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:triage

Run the triage stage on the active feature-design session. Wraps Template 02.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` from cwd to get `<slug>`. Read `.feature-design/<slug>/state.json`.

If no session is active, tell the user to run `/fd:start <feature-name>` first. Stop.

## Step 2 — Load inputs

Read:

- `.feature-design/<slug>/00-feature-idea.md` — the raw idea
- The source template `02 - TRIAGE_PROMPT.md`. Use Glob with pattern `**/02 - TRIAGE_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

The template's body is in the fenced `## The Prompt` block. Use that body as the prompt skeleton.

## Step 3 — Substitute placeholders

In the prompt body, replace:

- `{{FEATURE_IDEA}}` → contents of `00-feature-idea.md` (just the body, not the heading)
- `{{PRD_PATH}}` → `state.project_docs.prd_path`
- `{{CODEX_PATH}}` → `state.project_docs.codex_path`
- `{{DESIGN_PHILOSOPHY_PATH}}` → `state.project_docs.design_philosophy_path`
- `{{#if ADDITIONAL_DOCS}}…{{/if}}` block → expand if `state.project_docs.additional_docs` is non-empty; otherwise remove the block entirely

If any required path is empty, ask the user to provide it before executing. Update state.json with whatever they give.

## Step 4 — Execute

Run the substituted prompt against yourself (no separate model). Read the project docs as instructed in the prompt. Produce the structured assessment described in the template's `## 7. Outcome Criteria`.

Required output sections:
- Conflicts (if any)
- Coverage summary
- Open questions
- Recommended pipeline entry point (one of: full / mid-pipeline / spec-only)
- Rationale

## Step 5 — Write output

Write the assessment to `.feature-design/<slug>/01-triage.md` with this header:

```markdown
# Triage Assessment: <feature_name>

**Date:** <ISO date>
**Source:** Template 02 — TRIAGE_PROMPT.md
**Session:** <slug>

---

<assessment body>
```

## Step 6 — Update state

Update `state.json`:

- Append `"triage"` to `completed_stages` (if not already present)
- Set `current_stage` to the next stage based on the recommended entry point:
  - **full** → `"explore"`, `entry_point: "full"`
  - **mid-pipeline** → `"explore"` for now (crossfire is deferred), `entry_point: "mid-pipeline"`
  - **spec-only** → `"spec"`, `entry_point: "spec-only"`
- If the assessment surfaced UI considerations and `state.user_facing` was `null`, infer it from the assessment (or ask the user explicitly).

## Step 7 — Report

Print:

```
Triage complete → 01-triage.md
Entry point: <entry_point>
Recommended next: /fd:next  (will run /fd:<next-stage>)

Conflicts: <count>
Open questions: <count>
```

If conflicts were flagged as hard stops, recommend the user resolve them before proceeding regardless of `/fd:next`.
