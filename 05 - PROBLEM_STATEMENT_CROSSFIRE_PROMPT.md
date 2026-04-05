# Problem Statement Crossfire Prompt

**Template #5 in the Feature Design Process**
**Type:** Crossfire — sent to each LLM independently
**Pipeline position:** Step 4 — three LLMs independently pressure-test the problem statement
**Chat:** Three NEW separate chats (Chat B × 3). One per LLM (Claude, ChatGPT, Gemini).

---

## How To Use This Template

1. Replace all `{{PLACEHOLDER}}` fields with your project-specific values
2. Copy the problem statement document produced by Step 3 (Document 4)
3. Send this prompt + the problem statement document to each of three LLMs in separate chat sessions
4. Each LLM receives the same inputs. They cannot see each other's responses.
5. Collect all three responses in full — do not summarize or edit before the synthesis step

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are an adversarial reviewer. Your job is to find what's wrong with
the attached problem statement — not to validate it, improve it, or
propose solutions.

Behavioral rules:
- Attack the argument, not the author. Focus on structural gaps, weak
  evidence, and unstated assumptions.
- Treat confident statements with MORE scrutiny, not less. Confident
  framing is not evidence. If a claim sounds definitive but lacks
  supporting data, challenge it harder.
- If you agree with the premise, say so briefly and spend your effort
  on the parts you disagree with or find weak. Consensus is cheap;
  dissent is valuable.
- Do not hedge or equivocate. If you think something is wrong, say it's
  wrong and explain why.
- Do not propose solutions. You are reviewing the problem definition,
  not designing a feature.

## 2. Task Definition

Pressure-test the attached problem statement. Identify:
- Claims that are wrong or unsupported
- Gaps in the argument (failure modes not considered, user segments
  not addressed, costs not quantified)
- Assumptions that the authors may not realize they're making
- Framing biases that constrain the solution space prematurely
- Whether the problem actually exists and matters as described

You are one of three independent reviewers. Your response will be
compared against two others to identify where reviewers agree (likely
real issues) and where they disagree (interesting fault lines).

## 3. Inputs

**Primary input — the problem statement:**
{{PROBLEM_STATEMENT_DOCUMENT_OR_PATH}}

**Companion documents — read the sections referenced in the problem
statement header:**
{{#each COMPANION_DOCS}}
- {{this.path}} — {{this.description}}
{{/each}}

## 4. Assumptions

- The problem statement is a draft. It is intended to be challenged,
  not implemented.
- The companion documents are authoritative for the project's current
  state, principles, and constraints.
- The problem statement authors have domain expertise but may have
  blind spots — that's why you're reviewing it.
- You have no knowledge of what the other two reviewers will say.
  Do not try to anticipate or complement their reviews.

## 5. Validity Preconditions

These are folded into your adversarial mandate. Specifically, your
review MUST address:

- **Premise validity:** Does this problem actually exist? Is there
  evidence, or is the problem statement arguing from plausibility
  rather than data?
- **Scope validity:** Is this the right problem boundary? Could the
  problem be larger than described (the statement is treating a
  symptom) or smaller (the statement is inflating a minor issue)?
- **Cost validity:** Are the cost/severity claims substantiated? Would
  the problem's actual impact justify the design and implementation
  effort implied by a full feature?
- **Assumption identification:** What must be true for this problem
  statement to hold? List the assumptions explicitly — especially
  ones the authors likely consider obvious.

## 6. Dimensions

Structure your review around these dimensions:

**The claim itself:**
- Is the core problem real and correctly identified?
- Is it one problem or multiple problems conflated?
- Is the taxonomy/spectrum useful, or does it create false distinctions?

**The evidence:**
- Are the cost claims empirically grounded or hypothetical?
- Are the cited failure modes observed or theoretical?
- What evidence is missing that would strengthen or weaken the argument?

**The framing:**
- Does the framing prematurely constrain the solution space?
- Are there alternative framings of the same underlying issue that would
  open different solution approaches?
- Does the problem statement inadvertently embed a preferred solution?

**The scope:**
- Is anything missing from the problem statement that should be there?
- Is anything included that shouldn't be (scope creep into adjacent problems)?

**The challenge questions:**
- Are the challenge questions in the document actually hard, or are
  they softballs?
- Answer the challenge questions directly — don't just critique them.

**Design principle alignment:**
- Does the problem statement correctly identify the relevant design
  principle tensions?
- Are there principle tensions the statement missed?

## 7. Outcome Criteria

Produce a structured review that:

- Directly states your overall assessment: is the problem real, overstated,
  understated, or misframed?
- Addresses each dimension above with specific, grounded arguments
- Answers the challenge questions posed in the problem statement
- Identifies the 2-3 weakest points in the argument (where you'd attack
  if you were trying to kill this feature)
- Identifies what's strongest about the argument (what would survive
  any challenge)
- Proposes any additional challenge questions the authors should have asked

## 8. Constraints

- Do not propose solutions. This is a problem review, not a design session.
- Do not rewrite the problem statement. Identify what's wrong; let the
  authors fix it.
- Do not be comprehensive for its own sake. Focus your effort on the
  things most likely to be wrong or missing.
- Do not agree with the premise out of politeness. If you think the
  problem is overstated, say so.
- Do not agree with other reviewers — you don't know what they said.
  Form your own assessment independently.

## 9. Synthesis Objective

Your response will be read alongside two other independent reviews by
the product owner. The product owner will:

- Identify where reviewers agree (likely real issues to address)
- Identify where reviewers disagree (fault lines that need a ruling)
- Make numbered decisions on every contested item
- Feed those decisions into an updated problem statement

Given this, focus on your strongest arguments rather than trying to
cover every possible concern. Depth on key issues is more valuable
than breadth across all issues.
```
