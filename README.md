# Feature Design Templates

A structured, adversarial multi-model design process for taking a feature idea from raw concept through implementation-ready epics and stories.

## What This Is

14 prompt templates that codify a repeatable pipeline for feature design using multiple LLMs. The process sends design artifacts through independent adversarial review rounds (crossfire), where three frontier-tier models pressure-test claims, propose solutions, and surface failure modes — then a human decision-maker synthesizes the results.

The pipeline was developed and validated during the design of [LLM Crossfire](https://github.com/choughton/llm-crossfire)'s Planning feature. These templates generalize that process for any product.

## The Pipeline

```
Triage → Exploration → Problem Statement → Crossfire Review
    → Decision Synthesis → Feature Proposals → Iterative Crossfire
    → Final Synthesis → Verification → Spec → Epics → PE Setup
```

The process doc (`01 - FEATURE_DESIGN_PROCESS.md`) covers the full workflow, chat session boundaries, decision checkpoint guidance, and when to use shortcuts.

## Templates

| # | Template | Type |
|---|---|---|
| 01 | Process Guide | Reference doc |
| 02 | Triage | Single LLM |
| 03 | Idea Exploration | Single LLM (interactive) |
| 04 | Problem Statement Synthesis | Single LLM |
| 05 | Problem Statement Crossfire | Crossfire (×3 LLMs) |
| 06 | Problem Statement Decision Synthesis | Single LLM |
| 07 | Feature Proposal Crossfire | Crossfire (×3 LLMs) |
| 08 | Feature Proposal Round N Synthesis | Single LLM |
| 09 | Feature Proposal Round N+1 Crossfire | Crossfire (×3 LLMs) |
| 10 | Final Decision Synthesis | Single LLM |
| 11 | Verification | Single LLM |
| 12 | Spec Generation | Single LLM |
| 13 | Epic/Story Breakdown | Single LLM |
| 14 | Implementation PE Setup | Single LLM (persistent) |

## How To Use

1. Start with the process guide (`01`) to understand the pipeline and sizing shortcuts
2. Each template has `{{PLACEHOLDER}}` fields — replace them with your project-specific values
3. Follow the chat session boundaries in the process guide (which templates share a chat vs. need fresh context)
4. You are the decision-maker at each checkpoint — the LLMs generate decision surfaces, you make rulings

## Key Concepts

**Crossfire rounds** send the same source document to three independent LLMs. They can't see each other's responses. Convergence = high confidence. Divergence = interesting design tradeoff that needs a human ruling.

**Iterative crossfire** (Documents 8 and 9) lets you loop the design through additional review rounds. Each round's posture shifts from "propose from scratch" to "improve this." Settled decisions accumulate and can't be relitigated.

**The universal prompt skeleton** (8 elements + a 9th for crossfire prompts) structures every template: Role, Task, Inputs, Assumptions, Validity Preconditions, Dimensions, Outcome Criteria, Constraints, and Synthesis Objective.

## License

Private repository. All rights reserved.
