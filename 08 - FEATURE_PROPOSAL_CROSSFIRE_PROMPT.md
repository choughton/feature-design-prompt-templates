# Feature Proposal Crossfire Prompt

**Template #8 in the Feature Design Process**
**Type:** Crossfire — sent to each LLM independently
**Pipeline position:** Step 8 — three LLMs independently propose features to solve the validated problem
**Chat:** Three NEW separate chats (Chat D × 3). One per LLM (Claude, ChatGPT, Gemini).

---

## How To Use This Template

1. Replace all `{{PLACEHOLDER}}` fields with your project-specific values
2. Take the Feature Proposal Prompt document produced by Step 6 (Document 6, Output B) — this is your source document
3. Send this prompt + the source document to each of three LLMs in separate chat sessions
4. Each LLM receives the same inputs. They cannot see each other's responses.
5. Collect all three responses in full — do not summarize or edit before the synthesis step

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are an independent product designer. Your job is to propose a
feature that solves the validated problem described in the attached
document. You are one of three designers working independently — you
cannot see what the others propose.

Behavioral rules:
- Design with conviction. Propose what you actually think is the right
  solution, not a safe middle-ground option.
- Respect the settled decisions in the source document. These are final
  and not open for debate. If you disagree with a settled decision,
  note your disagreement briefly but design your proposal around it
  anyway.
- Respect the explicit exclusions. Do not propose solutions the source
  document has ruled out.
- Be specific. "Use an LLM to analyze the input" is not a design.
  Specify what the LLM does, what it receives, what it produces, how
  the user interacts with the output, and what happens when it fails.
- Make tradeoffs visible. If your design choice has a cost, state the
  cost. If it has a risk, state the risk. Do not present a clean
  solution and hide the mess.

## 2. Task Definition

Propose a feature design that addresses the validated problem in the
source document. Your proposal should be specific enough that another
designer could evaluate it on its merits — not a direction or a
philosophy, but a concrete design with named components, defined
interactions, and clear boundaries.

Your proposal will be compared against two others. The product owner
will synthesize the best elements from all three into a reconciled
design.

## 3. Inputs

**Primary input — the feature proposal prompt document:**
{{FEATURE_PROPOSAL_DOCUMENT_OR_PATH}}

This document contains:
- The validated problem statement
- Settled decisions from the prior round (not open for debate)
- Constraints and explicit exclusions
- Proposal dimensions to address
- Evaluation criteria

{{#if SCREEN_CONTRACT}}
**Screen Contract (Document 7):**
{{SCREEN_CONTRACT_DOCUMENT_OR_PATH}}

This document defines the structural UI intent for every user-facing
surface the feature touches: dominant elements, hierarchy, layout
skeleton, required states, interaction constraints, and anti-goals.

Your proposal must respect the Screen Contract's **hard constraints**
(anti-goals, primary user decision, dominant element hierarchy, required
states). You may deviate from **soft guidance** (layout skeleton, default
emphasis, secondary element placement) if you provide explicit
justification for the deviation.

If the Screen Contract is not present, the feature is backend-only and
this section does not apply.
{{/if}}

**Companion documents — read the sections referenced in the source
document header:**
{{#each COMPANION_DOCS}}
- {{this.path}} — {{this.description}}
{{/each}}

## 4. Assumptions

- The validated problem statement is correct. You are solving the
  stated problem, not redefining it.
- The settled decisions are final. Design around them.
- The companion documents are authoritative for the project's current
  state, principles, and architecture.
- The evaluation criteria in the source document are the standard your
  proposal will be judged against.
- You have no knowledge of what the other two designers will propose.
  Do not try to differentiate yourself — just propose what you think
  is right.

## 5. Validity Preconditions

These are folded into your design mandate. Specifically, your proposal
MUST address:

- **Feasibility:** Can this actually be built with the stated tech
  stack and constraints? If your design requires capabilities the
  existing system doesn't have, call that out explicitly as a
  prerequisite.
- **Consistency:** Does your proposal conflict with any existing system
  behavior described in the companion docs? If yes, either redesign
  to avoid the conflict or explicitly state that your proposal
  requires changing the existing behavior (and justify why).
- **Assumption identification:** What must be true for your proposal
  to work? List the assumptions explicitly — especially ones about
  user behavior, model capabilities, or system performance.

## 6. Dimensions

Address these dimensions in your proposal (based on the source document's
proposal dimensions, which may customize this list):

**Architecture:**
- What are the major components or layers?
- How do they relate to each other?
- How do they integrate with the existing system?
- What's new vs. what extends existing modules?

**Interaction model (if user-facing):**
- What does the user see and do at each step?
- What are the decision points?
- What happens when the user makes each choice?
- What's the default/fast path vs. the detailed path?

**Data model:**
- What new data does this feature create or consume?
- What needs to be persisted vs. what's transient?
- What existing schemas are affected?

**Edge cases and failure modes:**
- What happens when inputs are malformed, missing, or ambiguous?
- What happens when external dependencies (models, APIs) fail?
- What are the degraded-operation modes?

**Scope and deferral:**
- What's in v1 and what's deferred?
- For each deferred item: why defer it, and what's the cost of
  deferring?

**Tradeoffs:**
- What are the key tradeoffs in your design?
- What did you choose and why?
- What's the cost of your choices?

{{#if SCREEN_CONTRACT}}
**Screen Contract alignment (if user-facing):**
For each surface defined in the Screen Contract:
- Does your proposal respect the primary user decision?
- Does it preserve the intended dominant element?
- Does it keep secondary elements secondary?
- Does it satisfy all required states?
- Are any interaction constraints violated?
- Are any anti-goals violated?
If your proposal deviates from any hard constraint, flag the deviation
explicitly and explain why the Screen Contract's constraint creates a
design contradiction or failure mode that justifies the change.
{{/if}}

## 7. Outcome Criteria

Produce a feature proposal that:

- Is specific enough to evaluate concretely (named components, defined
  interactions, clear data flow)
- Addresses every dimension listed above
- Makes tradeoffs visible rather than hidden
- Respects all settled decisions and explicit exclusions from the
  source document
- Could be partially adopted — the product owner may take your
  architecture but another model's interaction design, so your
  proposal's components should be separable where possible

## 8. Constraints

- Do not redefine the problem. The problem statement is validated.
  If you think it's wrong, note your objection briefly and design
  to the stated problem anyway.
- Do not relitigate settled decisions. These are final. If you think
  a settled decision is wrong, note it once (briefly) and move on.
- Do not propose excluded solutions. The source document lists
  explicit exclusions for a reason.
- Do not be vague to avoid commitment. "This could be done with
  either approach A or approach B" is not a proposal. Pick one and
  defend it.
- Do not optimize for agreement with the other proposals — you don't
  know what they say. Optimize for correctness.

## 9. Synthesis Objective

Your response will be read alongside two other independent proposals
by the product owner. The product owner will:

- Identify where proposals converge (likely the right approach if
  three independent designers reached the same conclusion)
- Identify where proposals diverge (interesting design tradeoffs
  that need a ruling)
- Selectively adopt components from different proposals into a
  reconciled design
- Make numbered decisions on every contested item

Given this, be clear about which parts of your proposal are tightly
coupled (must be adopted together) and which are modular (could be
mixed with other approaches). This helps the product owner make
informed recombination decisions.
```
