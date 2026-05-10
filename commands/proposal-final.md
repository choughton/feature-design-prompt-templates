---
description: Lock in the final reconciled design from the crossfire pipeline. Produces the canonical design artifact consumed by /feature-design:verify, /feature-design:spec, /feature-design:epics.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /feature-design:proposal-final

Run the final decision synthesis stage. Wraps Template 11.

This produces the **canonical final design** — the single artifact that downstream stages (verify / spec / epics / pe-setup) consume. After this, the crossfire pipeline is closed and the design cannot be reopened without restarting from `/feature-design:proposal-iterate`.

The template operates in two modes:

- **Mode A — Post-iteration:** the design has been through one or more `proposal-synthesis` → `proposal-iterate` cycles. The latest `09-round<N>-synthesis.md` is the input.
- **Mode B — Single-round:** only one round of proposal crossfire ran (no iteration). The three initial proposals (`08a/b/c`) plus user rulings are the input. This is faster but skips iterative pressure.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: most recent stage is `proposal-synthesis-round-<N>` for some N (Mode A) OR `proposal-crossfire` directly with the user invoking `/feature-design:proposal-final` instead of `/feature-design:proposal-synthesis` (Mode B — must pass `--mode single` explicitly to indicate intent).

For Mode A (default): refuse unless a `proposal-synthesis-round-<N>` is the most recent stage.

For Mode B (`--mode single`): refuse unless `proposal-crossfire` is in `completed_stages` and no `proposal-synthesis-round-*` has run yet.

## Step 2 — Determine mode and load inputs

### Mode A (Post-iteration)

- The latest `09-round<N>-synthesis.md` — already reconciled
- `06-feature-proposal-source.md` for problem context
- `04-ui-contract.md` if user-facing
- All prior round synthesis files (for cumulative settled-decision tracing)

### Mode B (Single-round)

- `08a-claude.md`, `08b-codex.md`, `08c-gemini.md` — the three initial proposals (skip stubs)
- `06-feature-proposal-source.md`
- `04-ui-contract.md` if user-facing

For both modes:
- The synthesis template `11 - FEATURE_PROPOSAL_DECISION_SYNTHESIS_PROMPT.md`. Use Glob with pattern `**/11 - FEATURE_PROPOSAL_DECISION_SYNTHESIS_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Elicit final rulings

Mode A: ask if the user has any final corrections or overrides to apply. If they say "no," proceed with the existing reconciled design as-is.

Mode B: present the three proposals and elicit numbered rulings as in `/feature-design:problem-decision` Step 3.

## Step 4 — Substitute placeholders

In the template body:

- `{{POST_ITERATION}}` → branching variable. Set true for Mode A, false for Mode B. The template has both branches; remove the inactive one.
- `{{TOTAL_ROUNDS}}` → in Mode A, `state.crossfire.current_round` (the highest round number reached)
- `{{PRD_PATH}}`, `{{CODEX_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}` from `state.project_docs`

## Step 5 — Run the synthesis

Following the template's `## 6. Dimensions` for the active mode:

**Mode A** produces a clean canonical design with:
1. Design Summary
2. All Settled Decisions (organized by topic, no round attribution)
3. Final Design (no round markers — written as one-pass design)
4. Deferred Items (cumulative, no round attribution)
5. Final Screen Contract (if user-facing)
6. Open Questions (if any)

**Mode B** produces a reconciled design from a single round with:
1. Cross-Proposal Consensus
2. Settled Decisions (problem-round + this round)
3. Reconciled Design (which proposal's approach for each dimension, plus integration notes)
4. Deferred Items (with rationale)
5. Final Screen Contract (if user-facing)
6. Open Questions (if any)

Run the template's `## 5. Validity Preconditions` first — if items are unresolved or rulings contradict, stop and ask.

## Step 6 — Write output

Write to `.feature-design/<slug>/11-final-design.md`:

```markdown
# Final Reconciled Design: <feature_name>

**Date:** <ISO date>
**Source:** Template 11 — Mode <A | B>
**Session:** <slug>
**Total crossfire rounds:** <N>
**Status:** Final. Locked. Consumed by /feature-design:verify, /feature-design:spec, /feature-design:epics, /feature-design:pe-setup.
<if any round was partial>
**Note:** This design incorporates rounds where crossfire captured 2 of 3 perspectives: <list of partial rounds with which model was missing each time>. The final design reflects available reviews; verification (Document 12) should pay particular attention to the dimensions where the missing model would have had the strongest signal.
</if>

---

<final design body — Parts 1 through 6>
```

## Step 7 — Update state

- Append `"proposal-final"` to `completed_stages`
- Set `current_stage` to `"verify"`
- Set `state.crossfire.iteration_complete` to `true`
- Set `state.crossfire.final_design_file` to `"11-final-design.md"`

## Step 8 — Report

```
Final design locked → 11-final-design.md
Mode: <Post-iteration | Single-round>
Total rounds: <N>
Settled decisions: <count>
Deferred it