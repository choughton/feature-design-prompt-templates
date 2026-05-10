---
description: Generate a feature specification from the reconciled design and verification artifacts.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /feature-design:spec

Run the spec generation stage. Wraps Template 13.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: `verify` in `completed_stages`. Refuse unless completed or `--force`.

For `entry_point === "spec-only"` sessions, only `triage` is required.

## Step 2 — Load inputs

Treat all upstream artifacts as session inputs:

- `.feature-design/<slug>/03-problem-statement.md`
- `.feature-design/<slug>/04-ui-contract.md` (if user-facing)
- `.feature-design/<slug>/07-verification.md`
- The source template `13 - SYNTHESIS_TO_SPEC_PROMPT_TEMPLATE.md`. Use Glob with pattern `**/13 - SYNTHESIS_TO_SPEC_PROMPT_TEMPLATE.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Substitute placeholders

- `{{SESSION_ARTIFACTS}}` → expand into the list of session files described above with brief descriptions
- `{{DECISIONS_LOCATION}}` → "No formal product-owner ruling document exists in this session (crossfire deferred). Treat verification findings and the problem statement as the canonical decisions for v1."
- `{{CONTEXT_DOCS}}` → expand from `state.project_docs`
- `{{TECH_STACK_SUMMARY}}` → `state.tech_stack` (or "(not specified)")
- `{{FEATURE_NAME}}` → `state.feature_name`
- `{{DATE}}` → today's date
- `{{ADVERSARIAL_PROCESS_SUMMARY}}` → "Single-agent feature-design plugin pipeline (no crossfire). Problem statement, UI contract, and verification report produced sequentially in one session."

## Step 4 — Execute

Produce the spec per the template's `## Output Structure`:

1. Document Header
2. §1 Overview (what / why / cost / taxonomy)
3. §2 Architecture (components, relationships, integration, constraints)
4. §3 Interaction Model (if user-facing)
5. §3.5 Screen Contract (if user-facing — copy forward from `04-ui-contract.md`)
6. §4 Input Handling Matrix (if applicable)
7. §5 Data Model
8. §6 Transparency / Auditability (if applicable)
9. §7 Deferred Items
10. §8 Metrics / Success Criteria
11. §9 Experiential Quality (if user-facing)

**Critical rules from the template:**
- The spec is a design document, not a decision log
- No "Model A said / Model B said" attribution
- Encode the WHY, not just the WHAT
- Don't invent requirements; flag gaps in §7 or as Open Questions
- Don't lose signal from rejected/deferred ideas

Run the template's quality gates before finalizing — especially decision completeness, no-debate-residue, and edge-case coverage.

## Step 5 — Write output

Write to `.feature-design/<slug>/08-spec.md` with the document header from the template (already includes title/date/version/status/companion docs/origin).

## Step 6 — Update state

- Append `"spec"` to `completed_stages`
- Set `current_stage` to `"epics"`

## Step 7 — Report

```
Spec complete → 08-spec.md