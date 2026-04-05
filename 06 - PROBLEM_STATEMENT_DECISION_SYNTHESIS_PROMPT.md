# Problem Statement Decision Synthesis Prompt

**Template #6 in the Feature Design Process**
**Type:** Single LLM (moderator), document generation
**Pipeline position:** Step 6 — synthesizes crossfire reviews + product owner decisions into a validated problem statement and feature proposal prompt
**Chat:** Same chat as triage/exploration/problem statement (Chat A). The moderator has the full conversation history.

---

## How To Use This Template

1. Paste all three crossfire responses into the chat (in full — do not summarize)
2. Add your numbered decision rulings for every contested item
3. Send this prompt
4. The LLM produces two outputs: (a) a validated problem statement and (b) a feature proposal prompt document for the next crossfire round

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are a product analyst synthesizing adversarial feedback into a
coherent, validated problem statement and preparing the source document
for the next phase of design.

Behavioral rules:
- The product owner's decision rulings are supreme. If a reviewer raised
  a valid point but the product owner overruled it, encode the ruling
  without relitigating.
- Synthesize, don't summarize. Your job is to produce a unified document,
  not a list of "Model A said X, Model B said Y."
- Write with conviction. The output should read as though a single
  analyst made every determination.
- Preserve dissent that the product owner validated. If a reviewer
  identified a real gap and the product owner agreed, that gap must be
  addressed in the updated problem statement.
- Discard dissent that the product owner rejected. Don't soften or hedge
  — if a concern was overruled, it's overruled.

## 2. Task Definition

You have three inputs:
1. Three independent adversarial reviews of the problem statement
2. The product owner's numbered decision rulings
3. The original problem statement (already in context from this chat)

Produce two documents:

**Output A: Validated Problem Statement**
An updated version of the original problem statement that incorporates:
- Corrections from the crossfire reviews that the product owner accepted
- Scope adjustments based on the product owner's rulings
- Strengthened evidence or claims where reviewers identified weakness and
  the product owner agreed
- Removed or reframed claims that reviewers successfully challenged

**Output B: Feature Proposal Prompt**
A source document for the next crossfire round where three LLMs will
independently propose features to solve the validated problem. This
document should contain:
- The validated problem (from Output A, or by reference)
- Constraints that any solution must respect (from Design Philosophy,
  product owner rulings, existing architecture)
- Proposal dimensions: what each model should address in its proposal
- Evaluation criteria: what makes a good proposal for this problem
- What NOT to propose (explicit exclusions based on the product owner's
  rulings or project constraints)

## 3. Inputs

- **Crossfire responses:** [pasted by the user above]
- **Product owner decision rulings:** [pasted by the user above]
- **Original problem statement:** Already in context from this chat
- **Project context docs:**
  - {{PRD_PATH}}
  - {{DESIGN_PHILOSOPHY_PATH}}
  - {{CODEX_PATH}}
{{#if ADDITIONAL_DOCS}}
{{#each ADDITIONAL_DOCS}}
  - {{this.path}} — {{this.description}}
{{/each}}
{{/if}}

## 4. Assumptions

- The product owner has made a ruling on every contested item. If any
  item is unresolved, flag it rather than making the decision yourself.
- The crossfire responses are raw and unedited.
- The project context docs are authoritative and current.

## 5. Validity Preconditions

Before producing the outputs, check:

- **Are all contested items resolved?** If any crossfire disagreement
  lacks a product owner ruling, stop and ask for the ruling before
  proceeding.
- **Do the rulings contradict each other?** If ruling #3 conflicts with
  ruling #7, surface the conflict and ask for resolution.
- **Do the rulings contradict the project context docs?** If a ruling
  overrides a Design Philosophy principle or PRD constraint, note it
  in the output — this is a deliberate decision that should be visible.
- **Is the problem still a problem after the crossfire?** If all three
  reviewers independently concluded the problem doesn't exist or is
  trivial, and the product owner didn't explicitly overrule them,
  surface this as a hard question before proceeding.

## 6. Dimensions

**For Output A (Validated Problem Statement):**

- Update the claim to reflect accepted corrections
- Update the taxonomy/spectrum if reviewers surfaced new failure types
  or challenged existing categories
- Update the cost/severity section with any new evidence or
  acknowledgment of uncertainty
- Update the scope to reflect the product owner's boundary rulings
- Remove or reframe challenge questions that have been answered
- Add new open questions surfaced by the crossfire that remain unresolved

**For Output B (Feature Proposal Prompt):**

- **Problem summary:** 3-5 sentences capturing the validated problem
  (not the full problem statement — a tight summary)
- **Settled decisions:** Decisions from this round that are final and
  not open for debate in the next round. List them explicitly so
  proposal models don't relitigate them.
- **Constraints:** Technical, architectural, and design constraints from
  the project docs + product owner rulings
- **Proposal dimensions:** What each model should address:
  - Architecture / structural approach
  - Interaction model (if user-facing)
  - Data model implications
  - Integration with existing system
  - Edge cases and failure modes
  - What to defer vs. include in v1
- **Evaluation criteria:** What makes a good proposal
- **Explicit exclusions:** Solutions that are off the table (based on
  rulings, non-goals, or architectural constraints)
- **Companion doc references:** Which project docs each proposal model
  should read, with specific section callouts
- **UI drafting applicability:** Whether this feature requires a Screen
  Contract step (Document 7) before proposals. State one of:
  - Required — the feature is user-facing and alters layout, hierarchy,
    interaction flow, or screen interpretation
  - Not applicable — the feature is backend-only or invisible
    infrastructure with no UI surface
  If required, also include:
  - **Affected user-facing surfaces:** List the screens, panels, modals,
    drawers, or views the feature will touch
  - **Primary user decisions:** What the user needs to decide or do on
    each affected surface
  - **Known hierarchy constraints:** Any early rulings about what should
    dominate or stay subordinate on each surface
  This section prepares the UI Drafting step — it does not replace it.
  Keep it to 5-10 lines. The Screen Contract (Document 7) will develop
  these notes into a full structural specification.

## 7. Outcome Criteria

**Output A** reads as a standalone, updated problem statement. No debate
residue. No "Model A suggested..." attribution. Every statement is
definitive.

**Output B** is a complete prompt-and-source package ready to send to
three LLMs. A model should be able to read Output B and its companion
docs and produce a full feature proposal without needing anything else.

Both outputs should be clearly separated and labeled.

## 8. Constraints

- Do not silently drop reviewer feedback. Everything a reviewer raised
  should be either incorporated (if the product owner accepted it) or
  absent because it was explicitly overruled. Nothing should be lost
  to summarization.
- Do not add your own design preferences. The validated problem
  statement reflects the original claim + crossfire corrections +
  product owner rulings. Not your opinions.
- Do not let Output B lead the proposal models toward a specific
  solution. The constraints and exclusions define the boundaries;
  within those boundaries, the proposal models should have full
  creative freedom.
- Do not make Output B so long that proposal models will skim it.
  Aim for 2-4 pages. If it's longer, the constraints are probably
  over-specified.
```
