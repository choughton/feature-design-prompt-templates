---
description: Synthesize the exploration conversation into a standalone problem statement document.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:problem

Run the problem-statement synthesis stage. Wraps Template 04.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. If `explore` is not in `completed_stages` and `entry_point !== "mid-pipeline"`, refuse unless `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/02-exploration.md` — the exploration transcript
- `.feature-design/<slug>/01-triage.md` (for grounding)
- The source template `04 - PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md`. Use Glob with pattern `**/04 - PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Substitute placeholders

In the template's `## The Prompt` body:

- `{{PRD_PATH}}`, `{{CODEX_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}` from state
- `{{#if ADDITIONAL_DOCS}}…{{/if}}` — expand if non-empty

## Step 4 — Execute

Synthesize the exploration into a standalone problem statement following the template's `## 6. Dimensions`:

- Document header (title, date, status, companion docs, reviewer instructions)
- The Claim (2-3 sentences, falsifiable)
- Problem taxonomy (if applicable)
- Why this problem matters here specifically
- Cost of the status quo
- Design principle tensions
- Challenge questions (5-10, hard, attack the weakest points)

**Behavioral rules from the template:**
- Write with conviction. This is a claims document, not a summary.
- Be solution-neutral.
- The document must stand alone — a reviewer should evaluate it without reading the exploration.

If any of the template's `## 5. Validity Preconditions` fail (problem assumed into existence, overlap with existing features, not actually a user problem, framing isn't solution-neutral), surface the issue to the user before producing the document.

## Step 5 — Write output

Write to `.feature-design/<slug>/03-problem-statement.md` with this header:

```markdown
# Problem Statement: <feature_name>

**Date:** <ISO date>
**Source:** Template 04 — PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md
**Session:** <slug>
**Status:** Draft (crossfire review deferred — proceeding directly to next stage)

---

<problem statement body>
```

## Step 6 — Update state

- Append `"problem"` to `completed_stages`
- Set `current_stage` to `"ui-draft"` if `state.user_facing === true`, else `"verify"`

## Step 7 — Report

```
Problem statement complete → 03-problem-statement.md
Length: <line count> lines
Challenge questions: <count>

Recommended next: /fd:next  (will run /fd:<next-stage>)

Note: Crossfire review (Templates 05, 06) is deferred. The downstream pipeline will treat this draft as if it had passed crossfire. When crossfire support is added later, you can re-enter the pipeline here.
```
