---
description: Produce a Screen Contract for the feature's user-facing surfaces. Skipped for backend-only features.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:ui-draft

Run the UI Drafting Brief / Screen Contract stage. Wraps Template 07.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json.

If `state.user_facing === false`, refuse and tell the user this stage doesn't apply. Suggest `/fd:next` to skip directly to verify. Stop.

If `state.user_facing === null`, ask the user to confirm whether this feature has a UI surface before proceeding.

If `problem` is not in `completed_stages`, refuse unless `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/03-problem-statement.md`
- `.feature-design/<slug>/01-triage.md`
- The source template `07 - UI_DRAFTING_BRIEF_PROMPT.md`. Use Glob with pattern `**/07 - UI_DRAFTING_BRIEF_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Substitute placeholders

- `{{PRD_PATH}}`, `{{CODEX_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}` from state
- `{{#if EXISTING_SCREENS}}…{{/if}}` — if the feature modifies existing screens, ask the user for the relevant references before proceeding; otherwise remove the block

## Step 4 — Execute

Produce the Screen Contract per the template's `## 6. Dimensions`. For each user-facing surface:

- Executive contract (60-word hard-constraints summary, written last but placed first)
- Purpose
- Primary user decision
- Dominant element (exactly one — or a tiebreaker cluster if genuinely justified)
- Secondary elements
- Layout skeleton (prose / lightweight ASCII)
- Default emphasis
- First-paint priority
- Required states (empty / loading / partial-stale / error / success / disabled / destructive / overflow / narrow / degraded)
- Interaction constraints
- Keyboard / shortcut contract
- Anti-goals — split into Emphasis and Behavior groups

After all surfaces, include:
- Constraint classification (hard vs. soft, per surface)
- Surface transitions / flow invariants (focus management, scroll preservation, modal stacking, toast placement, deep-link behavior, refresh ordering, undo scope, hydration failure)

**Behavioral rules:**
- Structure over polish — no colors, fonts, spacing values
- Hierarchy over aesthetics — make dominance/subordination explicit
- Constraints over creativity — prevention is the point
- No mockups, no wireframes, no decorative UI

If `## 5. Validity Preconditions` flag the feature as not user-facing or affected surfaces as unclear, surface those before producing the contract.

## Step 5 — Write output

Write to `.feature-design/<slug>/04-ui-contract.md` with this header:

```markdown
# UI Screen Contract: <feature_name>

**Date:** <ISO date>
**Source:** Template 07 — UI_DRAFTING_BRIEF_PROMPT.md
**Session:** <slug>
**Status:** Governing artifact — flows into spec §3.5 and is enforced through verification, epics, and PE setup.

---

<screen contract body>
```

## Step 6 — Update state

- Append `"ui-draft"` to `completed_stages`
- Set `current_stage` to `"verify"`

## Step 7 — Report

```
Screen Contract complete → 04-ui-contract.md
Surfaces covered: <count>
Hard constraints: <count>
Soft guidance items: <count>

Recommended next: /fd:next  (will run /fd:verify)
```
