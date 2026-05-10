---
description: Synthesize the three problem-statement crossfire reviews + your numbered rulings into a validated problem statement and feature-proposal source document.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:problem-decision

Run the problem-statement decision synthesis stage. Wraps Template 06.

This stage **requires human decisions**. The three crossfire reviews surface contested items; the human (you) makes numbered rulings; the moderator (this command) encodes those rulings into a validated problem statement plus a feature-proposal source document.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: `problem-crossfire` in `completed_stages`. Refuse unless completed or `--force`.

Sanity-check the response files before proceeding: count how many of `05[abc]-*.md` are real responses (not retry stubs). If fewer than 2 are real, refuse to synthesize — single-perspective synthesis is just summarization. If exactly 2 are real (i.e., `state.crossfire.problem_round.partial === true` or one stub remains), accept and synthesize with a "produced from 2 of 3" annotation in the output.

## Step 2 — Load inputs

- `.feature-design/<slug>/03-problem-statement.md` — the original problem statement
- The crossfire review files. Look at `state.crossfire.problem_round.responses`:
  - **Three-file mode:** the entries point to `05a-*.md`, `05b-*.md`, `05c-*.md`. Read each (skipping any that are retry stubs).
  - **Combined-file mode:** a single entry with `model: "combined"` and `perspective_count: <N>` points to one file (e.g., `05-crossfire.md`). Read the whole file; the synthesis prompt below tells you to identify each independent response within it.
- The synthesis template `06 - PROBLEM_STATEMENT_DECISION_SYNTHESIS_PROMPT.md`. Use Glob with pattern `**/06 - PROBLEM_STATEMENT_DECISION_SYNTHESIS_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Present reviews and elicit rulings

Read the reviews in full. If three-file mode, that's three documents. If combined-file mode, that's one document with multiple sections — identify each model's response by section headers / delimiters (most crossfire tools produce structure like `## Claude`, `## Codex`, `## Gemini` or similar; if structure is ambiguous, ask the user to clarify which sections correspond to which perspectives).

Identify:

1. **Convergent concerns** — issues raised by 2 or more reviewers. These are likely real and should default to being addressed unless the user has a reason to overrule.
2. **Divergent suggestions** — items where reviewers disagree. These need explicit user rulings.
3. **Solo-but-substantive** — issues raised by only one reviewer but worth surfacing because they're high-impact or well-argued.

Present a structured digest to the user:

```
Problem statement crossfire findings:

CONVERGENT (likely real — propose to address unless overruled):
1. <finding> — flagged by Claude + Codex + Gemini
2. <finding> — flagged by Claude + Gemini
   ...

DIVERGENT (need your ruling):
3. <issue> — Claude says X, Codex says Y, Gemini says Z
4. <issue> — Claude flagged this, the others didn't see it
   ...

SOLO BUT SUBSTANTIVE:
5. <finding> — only Codex, but high-impact

Please make numbered rulings using this format:
  1. [Description]
     Ruling: [Address / Acknowledge / Reject]
     Rationale: [One sentence]
```

Wait for the user to provide rulings. If the user says "use your judgment on the convergent ones," accept that and only require explicit rulings on divergent items.

## Step 4 — Substitute placeholders in template

In the synthesis template's body:

- `{{PRD_PATH}}`, `{{DESIGN_PHILOSOPHY_PATH}}`, `{{CODEX_PATH}}` from `state.project_docs`
- The "crossfire responses" and "product owner decision rulings" sections are filled inline from what's already loaded into context (the three review files + the user's rulings from Step 3)

## Step 5 — Run the synthesis

Following the template's `## 6. Dimensions` and `## 7. Outcome Criteria`, produce **two outputs**:

**Output A: Validated Problem Statement** — an updated version of `03-problem-statement.md` incorporating accepted corrections.

**Output B: Feature Proposal Prompt** — a source document for `/fd:proposal-crossfire`. Must include:
- Problem summary (3-5 sentences)
- Settled decisions (cumulative — these are final)
- Constraints (technical, architectural, design)
- Proposal dimensions
- Evaluation criteria
- Explicit exclusions
- Companion doc references with section callouts
- UI drafting applicability flag — `Required` if user-facing with significant UI surface, `Not applicable` if backend-only. If Required, include affected surfaces and primary user decisions (5-10 lines).

Run the template's `## 5. Validity Preconditions` first — if any contested item lacks a ruling, stop and ask the user. Do not silently work around missing rulings.

## Step 6 — Write outputs

Write Output A to `.feature-design/<slug>/06-validated-problem.md`:

```markdown
# Validated Problem Statement: <feature_name>

**Date:** <ISO date>
**Source:** Template 06 — Output A
**Session:** <slug>
**Status:** Validated post-crossfire. Incorporates accepted corrections from 05a/b/c reviews and product owner rulings.
<if partial>
**Note:** This synthesis was produced from 2 of 3 crossfire perspectives. <Missing model> did not respond during dispatch. Convergence interpretation should account for the missing diversity-of-thought.
</if>

---

<validated problem statement body>
```

Write Output B to `.feature-design/<slug>/06-feature-proposal-source.md`:

```markdown
# Feature Proposal Source Document: <feature_name>

**Date:** <ISO date>
**Source:** Template 06 — Output B
**Session:** <slug>
**Status:** Source document for /fd:proposal-crossfire. Sent verbatim to three independent models.
**UI drafting applicability:** <Required | Not applicable>

---

<feature proposal source body>
```

## Step 7 — Update state

- Append `"problem-decision"` to `completed_stages`
- Set `current_stage` to `"ui-draft"` if Output B's UI applicability is `Required`, else `"proposal-crossfire"`
- Update `state.user_facing` based on the UI applicability flag if it was `null`
- Save the rulings in state.json under `crossfire.problem_round.rulings` for downstream reference and audit trail

## Step 8 — Report

```
Validated problem statement → 06-validated-problem.md
Feature proposal source     → 06-feature-proposal-source.md
Settled decisions: <count>
Deferred items: <count>
UI drafting: <Required | Not applicable>

Recommended next: /fd:next  (will run /fd:<next-stage>)
```
