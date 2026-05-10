---
description: Synthesize the current crossfire round (Round N) of feature proposals or review feedback into a reconciled design. Re-runnable across iteration rounds.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /feature-design:proposal-synthesis

Run the round-N synthesis stage. Wraps Template 09.

This command is **re-runnable across iteration rounds**. On the first invocation, it synthesizes the initial proposals from `/feature-design:proposal-crossfire`. On subsequent invocations (after `/feature-design:proposal-iterate`), it synthesizes the iterative review responses into an updated reconciled design. State tracks which round we're on.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: either `proposal-crossfire` in `completed_stages` (Round 1 case) or the most recent stage is `proposal-iterate-round-*` (Round N+1 case). Refuse unless one of those is true or `--force`.

Sanity-check the response files for the round being synthesized: count how many are real responses (not retry stubs). If fewer than 2 real responses exist, refuse — single-perspective synthesis is just summarization. If exactly 2 are real, proceed with a "produced from 2 of 3" annotation on the output.

## Step 2 — Determine round number

Read `state.crossfire.current_round`. This is the round we're synthesizing.

For Round 1: source documents are `08a-claude.md`, `08b-codex.md`, `08c-gemini.md` (initial proposals).

For Round N (N≥2): source documents are `10a-round{N}-claude.md`, `10b-round{N}-codex.md`, `10c-round{N}-gemini.md` (iterative reviews on the prior round's reconciled design).

## Step 3 — Load inputs

- The round's response files. Look at the round metadata in `state.crossfire.proposal_rounds[<N>]` (or `responses` for Round 1):
  - **Three-file mode:** `08a/b/c-*.md` for Round 1, `10a/b/c-round<N>-*.md` for iterations.
  - **Combined-file mode:** a single file like `08-crossfire.md` or `10-round<N>-crossfire.md` containing multiple perspectives. Identify each response by section headers when reading.
- For Round N≥2, also: the prior synthesis output `09-round{N-1}-synthesis.md`
- For all rounds: `06-feature-proposal-source.md` for the validated problem and constraints
- For user-facing: `04-ui-contract.md`
- The synthesis template `09 - FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md`. Use Glob with pattern `**/09 - FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 4 — Present and elicit rulings

Read the round's responses in full. Identify convergent / divergent / solo-substantive items as in `/feature-design:problem-decision` Step 3. Present the digest to the user, ask for numbered rulings.

For Round 1, frame items as "proposal divergences" (the three proposals took different approaches; rule on which to adopt).

For Round N≥2, frame items as "review findings" (the reviewers identified weaknesses in the reconciled design; rule on which to address).

If the round is heavily convergent (reviewers largely agreed on minor improvements), tell the user explicitly — this is a signal that the design has converged and they may want to proceed to `/feature-design:proposal-final` rather than another iteration.

## Step 5 — Substitute placeholders

In the template body:

- `{{ROUND_NUMBER}}` → current round number
- `{{NEXT_ROUND_NUMBER}}` → current + 1
- `{{ANOTHER_ROUND}}` → presence/absence drives template branching. Default to "another round possible" — the user decides whether to iterate or finalize after this synthesis.
- `{{PRIOR_SETTLED_DECISIONS}}` → for Round 1, the settled decisions from the problem statement round (in `06-feature-proposal-source.md`). For Round N≥2, the cumulative settled decisions from `09-round{N-1}-synthesis.md` plus all earlier rounds.
- `{{PRD_PATH}}`, `{{CODEX_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}` from `state.project_docs`

## Step 6 — Run the synthesis

Following the template's `## 6. Dimensions`, produce a reconciled design with these parts:

1. **Round N Change Summary** — what changed and why (keyed to rulings)
2. **Cumulative Settled Decisions** — all decisions from all rounds, by topic
3. **Updated Reconciled Design** — by design dimension (architecture, interaction model, data model, edge cases, scope), with round markers
4. **Screen Contract** (if user-facing) — carry forward and update only what changed this round; do not dissolve into general prose
5. **Deferred Items** (cumulative)
6. **Open Questions** (if any)

Run the template's `## 5. Validity Preconditions` — if rulings are missing, contradictory, or violate prior decisions, stop and ask the user.

## Step 7 — Write output

Write to `.feature-design/<slug>/09-round{N}-synthesis.md`:

```markdown
# Reconciled Design — Round <N>

**Date:** <ISO date>
**Source:** Template 09
**Session:** <slug>
**Round:** <N>
**Status:** Reconciled. Pending decision: another round (/feature-design:proposal-iterate) or finalize (/feature-design:proposal-final).
<if this round was 2 of 3>
**Note:** Round <N> crossfire captured 2 of 3 perspectives. <Missing model> did not respond. Reconciliation reflects the available reviews; downstream stages should account for the missing diversity-of-thought.
</if>

---

<reconciled design body — Parts 1 through 6>
```

## Step 8 — Update state

- Append the round metadata to `state.crossfire.proposal_rounds`:

```json
{
  "round": <N>,
  "synthesis_output": "09-round<N>-synthesis.md",
  "rulings": [...],
  "diminishing_returns_signal": <true | false>
}
```

- Append `"proposal-synthesis-round-<N>"` to `completed_stages` (this lets `/feature-design:next` dis