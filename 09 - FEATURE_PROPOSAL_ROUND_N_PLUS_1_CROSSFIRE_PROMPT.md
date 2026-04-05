# Feature Proposal Round N+1 Crossfire Prompt

**Template #9 in the Feature Design Process**
**Type:** Crossfire — sent to each LLM independently
**Pipeline position:** Step 10 — After a Round N synthesis, when the product owner decides to run another crossfire round on the feature proposal
**Chat:** Three NEW separate chats. One per LLM (Claude, ChatGPT, Gemini). Fresh chats for every round.

**Purpose:** Sends the updated reconciled design from Round N back through a crossfire round. Unlike Document 7 (initial proposals from scratch), this prompt asks each LLM to review an existing reconciled design and propose targeted improvements — not to design from scratch.

---

## How To Use This Template

1. Replace all `{{PLACEHOLDER}}` fields with your project-specific values
2. Take the reconciled design produced by the Round N synthesis (Document 8)
3. Send this prompt + the reconciled design to each of three LLMs in separate chat sessions
4. Each LLM receives the same inputs. They cannot see each other's responses.
5. Collect all three responses in full — do not summarize or edit before the next synthesis step

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are reviewing a reconciled feature design that has been through
{{ROUND_NUMBER}} round(s) of multi-model crossfire. The design
incorporates feedback from prior rounds and product owner decisions.

Your job is different from the initial proposal round. You are NOT
designing from scratch. You are:
- Identifying what's still weak, incomplete, or internally inconsistent
- Proposing targeted improvements to specific components
- Validating that the settled decisions actually work together
- Surfacing failure modes or edge cases that prior rounds missed

Behavioral rules:
- Focus on what's WRONG or MISSING, not what's right. The design has
  already survived prior rounds of review — don't repeat praise.
- Respect settled decisions. These are listed explicitly in the source
  document. If you disagree with a settled decision, note it ONCE
  (briefly) and move on. Do not build your review around relitigating
  settled points.
- Be specific. "The interaction model could be better" is not a review.
  "The interaction model doesn't handle the case where X happens
  because Y" is a review.
- Propose concrete alternatives, not abstract concerns. If you identify
  a problem, propose a specific fix — don't just flag it.
- Distinguish between "this is wrong" and "this could be better." The
  former blocks the design; the latter is an optimization the product
  owner can choose to defer.

## 2. Task Definition

Review the attached reconciled feature design (Round {{ROUND_NUMBER}})
and produce:
- A targeted assessment of what's weak, incomplete, or inconsistent
- Specific improvement proposals for each issue identified
- Validation that the design's components work together coherently
- Any new failure modes or edge cases not addressed in prior rounds

You are one of three independent reviewers. Your response will be
compared against two others to identify convergent concerns (likely
real issues) and divergent suggestions (design tradeoffs needing a
product owner ruling).

## 3. Inputs

**Primary input — the reconciled design from Round {{ROUND_NUMBER}}:**
{{RECONCILED_DESIGN_DOCUMENT_OR_PATH}}

This document contains:
- A change summary showing what changed in the most recent round
- Cumulative settled decisions from all prior rounds
- The full reconciled design
- Deferred items with rationale
- Any remaining open questions

**Companion documents — read the sections referenced in the design:**
{{#each COMPANION_DOCS}}
- {{this.path}} — {{this.description}}
{{/each}}

## 4. Assumptions

- The reconciled design is the current best version. Prior rounds have
  already addressed major structural issues.
- Settled decisions are final unless they create a logical impossibility
  or internal contradiction. Disagreement alone is not sufficient to
  reopen them.
- The companion documents are authoritative for the project's current
  state, principles, and constraints.
- You have no knowledge of what the other two reviewers will say.

## 5. Validity Preconditions

Folded into your review mandate. Specifically:

- **Internal consistency:** Do the design's components actually work
  together? If the architecture assumes X but the interaction model
  assumes Y, that's a finding.
- **Completeness:** Are there user paths, input types, or failure modes
  that the design describes but doesn't handle?
- **Assumption drift:** Have accumulated decisions across rounds created
  implicit assumptions that no single round examined? Sometimes round-
  over-round changes introduce contradictions that only become visible
  when you read the full design fresh.
- **Scope creep detection:** Has the design grown beyond what the
  original problem statement justified? If so, flag it.

## 6. Dimensions

Structure your review around these dimensions:

**Architecture coherence:**
- Do the components fit together after all rounds of changes?
- Are there integration points that were assumed but never specified?
- Does the architecture handle the edge cases described in the design?

**Interaction model completeness (if user-facing):**
- Are all user paths fully specified?
- Are error states and recovery paths defined?
- Does the interaction model work for all user types described in the
  problem statement?

**Data model soundness:**
- Are all fields consumed somewhere? (no orphans)
- Are the persistence boundaries clear? (what's stored vs. transient)
- Does the data model support the queries the feature needs?

**Failure modes:**
- What happens when external dependencies fail?
- What happens with malformed, missing, or adversarial input?
- Are there race conditions or timing issues in async operations?

**Deferred items review:**
- Are any deferred items actually blocking? (deferred something that's
  needed for v1 to function)
- Are any deferred items trivial enough to include now?

**Design principle alignment:**
- Does the design still align with the project's design philosophy
  after all rounds of changes?
- Has round-over-round iteration introduced patterns that conflict
  with existing conventions?

## 7. Outcome Criteria

Produce a structured review that:

- Prioritizes findings by severity:
  - **Blocker:** Must be fixed before this design can become a spec.
    The design doesn't work as described.
  - **Should-fix:** Significant improvement. The design works but has
    a meaningful gap.
  - **Optimization:** Nice to have. Could be deferred without risk.
- For each finding, includes a specific improvement proposal
- Validates what's working (briefly — don't spend tokens on praise)
- Identifies the 1-3 highest-impact changes that would most improve
  the design
- States whether the design is converging (ready for spec after this
  round) or still has significant open issues

## 8. Constraints

- Do not redesign from scratch. This is review and targeted improvement,
  not a new proposal.
- Do not relitigate settled decisions at length. One sentence noting
  disagreement is sufficient. Then move on.
- Do not propose improvements that conflict with settled decisions
  without explicitly flagging the conflict.
- Do not be comprehensive for its own sake. Focus on findings that
  would actually affect the feature's success. Minor wording
  preferences and stylistic concerns are not findings.
- Do not agree with other reviewers — you don't know what they said.

## 9. Synthesis Objective

Your response will be read alongside two other independent reviews by
the product owner. The product owner will:

- Identify convergent concerns (raised by 2+ reviewers — likely real)
- Identify divergent suggestions (design tradeoffs needing a ruling)
- Make numbered decisions on contested items
- Feed decisions into the next synthesis round (Document 8)

Focus on your highest-conviction findings rather than trying to cover
every dimension. If you only have one blocker but it's critical,
that's more valuable than ten minor observations.

The product owner will also decide after this round whether to iterate
again or proceed to verification. If you believe the design is ready,
say so explicitly. If you believe it needs another round, say that too
and explain what another round would need to address.
```

---

## When to Use This vs. Document 7

Document 7 (`FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md`) is for the initial crossfire round where LLMs propose features from scratch against a validated problem statement.

Use this template (9) when:
- A reconciled design already exists from a prior round
- The product owner wants another round of review and refinement
- The design posture is "improve this" not "propose something new"

The iteration loop is: **9** (crossfire review) → **8** (synthesis + decisions) → decide: another round (back to **9**) or proceed to verification (**Document 10**).
