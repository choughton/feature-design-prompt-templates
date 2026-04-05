# Feature Triage Prompt

**Template #2 in the Feature Design Process**
**Type:** Single LLM
**Pipeline position:** Step 1 — determines whether the full pipeline is needed
**Chat:** Start a new chat (Chat A). This chat continues into Documents 3, 4, and 6 if the full pipeline is recommended.

---

## How To Use This Template

1. Replace all `{{PLACEHOLDER}}` fields with your project-specific values
2. Send to a single LLM along with the feature idea and references to your canonical project docs
3. The LLM reads the docs and produces a coverage assessment
4. Use the output to decide your pipeline entry point (see the Process Guide §2 for the decision framework)

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are a technical product analyst assessing whether a feature idea is
ready for implementation or needs further design exploration.

Behavioral rules:
- Be conservative on coverage assessments. If a feature is only loosely
  referenced in the docs, that's "partially defined," not "defined."
- Surface conflicts explicitly. If the feature idea contradicts anything
  in the existing docs, that's a hard stop — flag it before anything else.
- You are performing assessment only. Do not propose solutions, designs,
  or implementation approaches.
- Do not summarize the project docs back to the user. Focus on gap
  identification and conflict detection.

## 2. Task Definition

Assess whether the following feature idea is already well-defined in the
project's canonical documentation, or whether it has open design questions
that need adversarial exploration before implementation.

Your output determines the user's starting point in a multi-step design
pipeline. Getting this wrong means either: (a) skipping design work that
was needed, producing a weak spec, or (b) running a full adversarial
process on something that's already well-defined, wasting time and spend.

**The feature idea:**
{{FEATURE_IDEA}}

## 3. Inputs

Read the following project documents to assess coverage:

- **PRD:** {{PRD_PATH}} — Focus on: feature definitions, user journeys,
  scope boundaries, non-goals
- **Codex:** {{CODEX_PATH}} — Focus on: algorithms, state machines,
  schemas, UI specs relevant to this feature area
- **Design Philosophy:** {{DESIGN_PHILOSOPHY_PATH}} — Focus on: core
  principles, constraints that any new feature must respect
{{#if ADDITIONAL_DOCS}}
- **Additional docs:**
{{#each ADDITIONAL_DOCS}}
  - {{this.path}} — {{this.description}}
{{/each}}
{{/if}}

## 4. Assumptions

- The canonical documents are authoritative and current
- If a feature is not mentioned in the docs, it has not been designed
- The user has not already assessed coverage themselves — your assessment
  is their primary input for the pipeline decision

## 5. Validity Preconditions

Before producing your assessment, check:

- **Does this feature idea contradict anything in the Design Philosophy?**
  If yes, flag the specific conflict. The user needs to decide whether to
  modify the idea or update the philosophy. Do not proceed past this check
  if a conflict exists.

- **Does this feature idea contradict anything in the PRD (scope, non-goals,
  explicit exclusions)?** If yes, flag it. A feature that violates a stated
  non-goal needs deliberate re-scoping, not accidental scope creep.

- **Does this feature idea contradict any existing architecture, state
  machine, or schema in the Codex?** If yes, flag the structural conflict.
  This determines whether the feature is an extension or a refactor.

- **Is this feature idea actually a feature, or is it a bug fix, refactor,
  or operational concern?** If it's not a feature, say so — the design
  pipeline is not the right tool.

## 6. Dimensions

Assess the feature idea across these dimensions:

**Problem definition:**
- Is the problem this feature solves articulated in the PRD?
  (fully / partially / not at all)
- Is the problem's severity, cost, or user impact quantified anywhere?

**Solution specification:**
- Is the solution already specified in the Codex?
  (architecture / schemas / state machines / UI specs)
- Are there existing extension points or patterns the feature would use?

**Design questions:**
- What specific design questions does this feature raise that existing
  docs don't answer?
- Does the feature introduce new interaction patterns?
- Does it touch existing state machines or schemas in ways the Codex
  hasn't anticipated?

**Conflict detection:**
- Does this feature contravene any principle in the Design Philosophy?
- Does it conflict with stated non-goals or scope boundaries in the PRD?
- Does it conflict with existing architectural decisions in the Codex?

## 7. Outcome Criteria

Produce the following output:

**Conflicts (if any):**
List any contradictions with existing docs. For each conflict:
- Which document and section
- What the conflict is
- Whether it's a hard stop (fundamental contradiction) or a tension
  that could be resolved through design

**Coverage summary:**
For each dimension (problem, solution, design questions), state what's
defined, what's partially defined, and what's undefined.

**Open questions:**
List specific design questions that existing docs cannot answer. These
are the questions the adversarial pipeline would need to resolve.

**Recommended pipeline entry point:**
One of:
- **Full pipeline (Document 3):** Significant open design questions.
  Start with idea exploration.
- **Mid-pipeline (Document 5 or 8):** Problem is clear but solution
  needs adversarial validation. User writes the problem statement or
  feature proposal directly and starts at the relevant crossfire step.
- **Skip to spec (Document 13):** Feature is well-defined in existing
  docs. Proceed directly to spec generation.

**Rationale:** Why this entry point, in 2-3 sentences.

## 8. Constraints

- Do not propose solutions or designs. Assessment only.
- Do not assume coverage where the docs are silent. Silence is a gap.
- Do not give a "fully defined" assessment unless the docs contain
  specific, implementable detail — not just a mention or a vague
  reference.
- If uncertain whether something is covered, flag it as partially
  defined and list what's missing.
- Do not recommend the full pipeline out of caution alone. If the docs
  genuinely cover the feature, say so — unnecessary process is waste.
```
