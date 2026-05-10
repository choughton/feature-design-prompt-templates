---
description: Interactive idea-exploration session that develops a raw feature idea into structured problem understanding.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /feature-design:explore

Run the idea-exploration stage. Wraps Template 03. Unlike other stages, this is **interactive** — you're having a conversation, not generating a one-shot document.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and `.feature-design/<slug>/state.json`. If `triage` is not in `completed_stages`, refuse unless `--force`.

## Step 2 — Load inputs

Read:

- `.feature-design/<slug>/00-feature-idea.md`
- `.feature-design/<slug>/01-triage.md` (if present)
- The source template `03 - IDEA_EXPLORATION_PROMPT.md`. Use Glob with pattern `**/03 - IDEA_EXPLORATION_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Adopt the moderator role

Internalize the template's `## 1. Role & Behavioral Instructions` section. Substitute placeholders:

- `{{FEATURE_IDEA}}` → contents of `00-feature-idea.md`
- `{{TRIAGE_OUTPUT}}` → contents of `01-triage.md` (if present; else remove the `{{#if TRIAGE_OUTPUT}}…{{/if}}` block)
- `{{PRD_PATH}}`, `{{CODEX_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}` from `state.project_docs`

## Step 4 — Begin the conversation

Open the conversation by acknowledging the feature idea and the triage findings (if any), then ask the first exploratory question. Follow the dimensions in `## 6` of the template — but conversationally, not as a checklist.

**Critical rules from the template:**
- Stay in problem space; don't jump to solutions
- Challenge weak reasoning directly
- Don't ask more than one question per response unless tightly related
- Do not produce a document yet — that's the next stage's job

## Step 5 — Detect convergence

When the conversation has produced enough signal (per `## 7. Outcome Criteria` in the template — the problem is articulable distinct from any solution, who/when/cost/constraints/open-questions are clear), suggest moving to problem statement synthesis. Ask explicitly:

> "I think we have enough to draft a problem statement. Want me to proceed to `/feature-design:problem`, or keep exploring?"

The transition is **always user-gated**. Do not proceed automatically.

## Step 6 — Write transcript

When the user agrees to move forward (or runs `/feature-design:problem` directly), write the conversation transcript to `.feature-design/<slug>/02-exploration.md` with this header:

```markdown
# Idea Exploration: <feature_name>

**Date:** <ISO date>
**Source:** Template 03 — IDEA_EXPLORATION_PROMPT.md
**Session:** <slug>

---

<full transcript: alternating User: / Assistant: blocks>
```

## Step 7 — Update state

- Append `"explore"` to `completed_stages`
- Set `current_stage` to `"problem"`

## Step 8 — Report

```
Exp