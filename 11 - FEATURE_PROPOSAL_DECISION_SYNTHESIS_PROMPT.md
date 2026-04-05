# Feature Proposal Final Decision Synthesis Prompt

**Template #11 in the Feature Design Process**
**Type:** Single LLM (moderator), document generation
**Pipeline position:** Step 12 — produces the final reconciled design from either a single crossfire round or after iterative refinement rounds
**Chat:** Same chat (Chat C) as Document 9 synthesis rounds. If no iteration rounds were run, start a new chat (Chat C) here.

---

## How To Use This Template

This template operates in two modes depending on whether you ran iteration rounds (Documents 8 and 9):

### Mode A: After iteration rounds (Documents 9 → 10 → 9 loop)

1. Continue the same Chat C where Document 9 synthesis rounds happened
2. The final reconciled design from the last Document 9 round is already in context
3. Add any final rulings or corrections
4. Send this prompt
5. The LLM produces the canonical final design artifact

### Mode B: Single crossfire round (no iteration)

1. Start a new chat (Chat C)
2. Paste the feature proposal prompt document (from Step 6, Document 6 Output B)
3. Paste all three feature proposal responses (in full — do not summarize)
4. Add your numbered decision rulings for every contested item
5. Send this prompt
6. The LLM produces a reconciled feature design

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are a product architect producing the final reconciled feature
design — the single canonical artifact that downstream steps
(verification, spec generation, implementation) will consume.

Behavioral rules:
- The product owner's decision rulings are supreme. Encode them without
  relitigating.
- Synthesize, don't average. The goal is the best design, not a
  compromise that includes a piece of everything. If one approach is
  clearly superior, use it.
- Write with conviction. The reconciled design should read as though
  a single architect made every decision.
- This is the FINAL design artifact. It must be complete, coherent,
  and self-contained. A reader should be able to understand the full
  feature design from this document alone.

{{#if POST_ITERATION}}
**Mode: Post-iteration finalization.**
The design has been through {{TOTAL_ROUNDS}} rounds of crossfire
refinement. Your job is to produce a clean, final version of the
reconciled design — resolving any remaining rough edges, ensuring
all settled decisions are consistently encoded, and producing a
document that reads as a unified design rather than a layered
accumulation of rounds.
{{else}}
**Mode: Single-round synthesis.**
Three independent feature proposals are being synthesized for the
first time. Take the best elements from all three, encode the
product owner's decisions, and produce a unified design.

Additional behavioral rules for single-round mode:
- Attribute convergence, not proposals. When all three proposals agree,
  note the consensus. Don't attribute individual positions.
- Preserve the strongest version of each idea. If Proposal A had the
  best architecture but Proposal B had the best interaction model,
  take the best of each — but verify they're compatible.
{{/if}}

## 2. Task Definition

{{#if POST_ITERATION}}
You have these inputs:
1. The final reconciled design from the last synthesis round (Document 9)
   — already in context from this chat
2. Cumulative settled decisions from all rounds
3. The product owner's final rulings or corrections (if any)
4. Project companion documents

Produce the canonical final design that:
- Reads as a unified, coherent design — no round-layering artifacts
- Encodes every settled decision from every round
- Resolves any remaining open questions (or explicitly defers them)
- Is complete enough for verification (Document 12) and spec generation
  (Document 13)
{{else}}
You have these inputs:
1. The feature proposal prompt document (the problem + constraints)
2. Three independent feature proposals
3. The product owner's numbered decision rulings
4. Project companion documents

Produce a reconciled feature design that:
- Takes the best elements from all three proposals
- Encodes the product owner's decisions
- Resolves all contested items
- Is coherent and internally consistent (no Frankenstein assembly)

This reconciled design will be verified for completeness (Step 13),
then transformed into a feature spec (Step 14).
{{/if}}

## 3. Inputs

{{#if POST_ITERATION}}
- **Final reconciled design from Document 9:** Already in context
- **Cumulative settled decisions:** Already in context from prior
  synthesis rounds
- **Product owner final rulings/corrections:** [pasted by the user,
  if any]
{{else}}
- **Feature proposal prompt document:** [pasted by the user above]
- **Three feature proposals:** [pasted by the user above]
- **Product owner decision rulings:** [pasted by the user above]
{{/if}}
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
  item is unresolved, flag it.
{{#if POST_ITERATION}}
- The iterative refinement rounds have converged the design. This step
  is finalization, not another design pass.
- Settled decisions from all prior rounds are final and cannot be
  reopened — unless the product owner explicitly overrides them in
  their final rulings.
{{else}}
- The feature proposals are raw and unedited.
- The feature proposal prompt document contains the settled decisions
  from the problem statement round — those are still final.
{{/if}}
- The project context docs are authoritative and current.

## 5. Validity Preconditions

Before producing the output, check:

- **Are all contested items resolved?** If any disagreement lacks a
  product owner ruling, stop and ask.
- **Do the rulings contradict each other?** Surface any conflicts
  between rulings before proceeding.
{{#if POST_ITERATION}}
- **Is the design internally consistent after all rounds?** Round-over-
  round changes can introduce subtle contradictions. Read the full
  design as a fresh reviewer would and check that all components still
  fit together.
- **Are deferred items still correctly categorized?** Something deferred
  in Round 1 may have become necessary due to Round 2+ changes.
{{else}}
- **Are the selected components compatible?** If you're taking the
  architecture from one proposal and the interaction model from
  another, verify they don't conflict. If they do, flag the
  incompatibility and recommend a resolution.
- **Do the combined decisions still respect settled decisions from
  the problem statement round?** The feature proposals may have
  drifted from the validated problem. Verify alignment.
{{/if}}
- **Does the reconciled design contradict any project context docs?**
  If a ruling overrides a Design Philosophy principle or Codex
  constraint, note it explicitly as a deliberate decision.

## 6. Dimensions

Structure the final reconciled design as follows:

{{#if POST_ITERATION}}
**Part 1: Design Summary**
- What this feature is and what problem it solves (2-3 sentences)
- How many rounds of refinement it went through
- The key design decisions that shaped the final form

**Part 2: All Settled Decisions**
- Every decision from every round, organized by topic (not by round)
- Each decision stated definitively — no round attribution or debate
  history

**Part 3: Final Design**
For each design dimension (architecture, interaction model, data model,
edge cases, scope):
- The definitive design — no round markers, no change tracking
- Written as though designed in one pass

**Part 4: Deferred Items**
- All deferred items with rationale
- No round attribution needed — just the item and why it's deferred

**Part 4.5: Final Screen Contract (if user-facing)**
- The canonical Screen Contract as it stands after all rounds
- All surfaces with their final hierarchy, layout, states, and anti-goals
- No round markers — written as definitive screen intent
- This section is consumed directly by verification (Document 12),
  spec generation (Document 13), and implementation (Documents 14-15)

**Part 5: Open Questions (if any remain)**
- Questions that verification (Document 12) should specifically check
{{else}}
**Part 1: Cross-Proposal Consensus**
- What did all three proposals agree on? These are high-confidence
  design elements — three independent designers reaching the same
  conclusion is strong signal.

**Part 2: Settled Decisions**
- Decisions from the problem statement round that carry forward
- Decisions from the product owner's rulings on this round
- For each: state the decision, not the debate that led to it

**Part 3: Reconciled Design**
For each proposal dimension (architecture, interaction model, data
model, edge cases, scope):
- Which proposal's approach is adopted (or which hybrid)
- How it integrates with the other adopted components
- Any modifications needed to make adopted components compatible

**Part 4: Deferred Items**
- Ideas from any proposal that the product owner deferred
- For each: what it is and why it's deferred (not just "out of scope"
  but the specific reason)

**Part 4.5: Final Screen Contract (if user-facing)**
- The Screen Contract from Document 7, updated to reflect any changes
  from the product owner's rulings on the proposals
- All surfaces with their final hierarchy, layout, states, and anti-goals
- This section is consumed directly by downstream steps

**Part 5: Open Questions (if any remain)**
- Any design questions that the proposals raised but the product
  owner's rulings didn't fully resolve
- These will be addressed in the verification step (Step 13)
{{/if}}

## 7. Outcome Criteria

The final reconciled design:

- Reads as a coherent, unified design — not a collage of proposals or
  a layer cake of rounds
- Encodes every product owner ruling
- Has no internal contradictions
- Preserves every deferred item with a rationale
- Is complete enough that verification (Document 12) can check it for
  gaps against the project context docs
- Is self-contained: a reader unfamiliar with the crossfire process
  can understand the full design from this document alone

## 8. Constraints

- Do not silently drop ideas. Everything raised should be either
  incorporated, deferred with rationale, or absent because the product
  owner ruled against it.
- Do not average between competing approaches. Pick the better one.
- Do not introduce new design ideas. The reconciled design is a
  synthesis of what was proposed and decided. Not your own proposals.
- Do not preserve debate in the output. The design should read as
  definitive statements, not a decision log.
- Do not let the reconciled design exceed the scope the product owner
  approved. Unapproved scope goes in deferred.
{{#if POST_ITERATION}}
- Do not preserve round-layering artifacts. "In Round 2 we changed..."
  belongs in a changelog, not in a final design. Write as though
  the design was produced in one pass.
{{/if}}
```

---

## When to Use This vs. Document 9

Document 9 (`FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md`) is for intermediate synthesis during iterative crossfire rounds. It produces a reconciled design with change tracking, round markers, and cumulative decision records — optimized for feeding into the next round.

This template (11) is for the FINAL synthesis — whether after one round or many. It produces a clean, self-contained design artifact optimized for downstream consumption by verification (Document 12) and spec generation (Document 13).

**Single crossfire round (no iteration):** Use this template directly after collecting three proposals + product owner decisions.

**After iteration rounds:** Use Document 9 for each intermediate round, then use this template to finalize the last reconciled design into a clean canonical artifact.
