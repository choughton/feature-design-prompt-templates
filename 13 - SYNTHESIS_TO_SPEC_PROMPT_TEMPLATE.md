# Spec Generation Prompt

**Template #13 in the Feature Design Process**
**Type:** Single LLM, document generation
**Pipeline position:** Step 14 — transforms the reconciled design into a feature specification
**Chat:** NEW chat (Chat F). The spec must stand on its own without the prior conversation context. This chat continues into Document 14 (epic/story breakdown).

**Purpose:** Transforms the output of the adversarial design process (reconciled design + product owner decisions) into a coherent feature specification ready for implementation scoping.

**Usage:** Copy the prompt below, replace all `{{PLACEHOLDER}}` fields with your project-specific values, and send to an LLM along with the adversarial session artifacts.

---

## The Prompt

```
You are a senior product architect writing a feature specification. Your input
is the output of a multi-model adversarial design session: multiple LLMs
independently reviewed a problem statement, independently proposed solutions,
and a reconciled synthesis was pressure-tested with final product owner
decisions applied.

Your job is to transform this deliberation record into a single, coherent
design document that an implementation team can build from — without needing
to read the adversarial session artifacts themselves.

## Inputs

**Adversarial session artifacts (read ALL of these):**
{{#each SESSION_ARTIFACTS}}
- {{this.path}} — {{this.description}}
{{/each}}

**Product owner decisions:** {{DECISIONS_LOCATION}}
(These are final. They override any model recommendation. Encode them into
the spec without relitigating them.)

**Project context documents (read for constraints and conventions):**
{{#each CONTEXT_DOCS}}
- {{this.path}} — {{this.description}}
{{/each}}

**Tech stack:** {{TECH_STACK_SUMMARY}}

## Critical Rules

### 1. The spec is a design document, not a decision log.

The adversarial session produced debates, dissents, contested items, and
product owner rulings. The spec encodes the OUTCOMES of those debates as
definitive design statements. It does not preserve the debate itself.

Wrong: "Three models disagreed about whether X should be mandatory. The
product owner decided it should be mandatory."
Right: "X is mandatory."

The spec should read as though a single designer made every decision with
full conviction. The adversarial origin is noted in the document header
for provenance — not in the body.

### 2. Product owner decisions are supreme.

If the adversarial session produced a consensus that the product owner
overruled, the spec reflects the product owner's decision. Period.

If the product owner's decision contradicts a design principle from the
project context docs, the spec should make the decision's rationale
self-evident through its framing — not by arguing for it.

### 3. Encode the WHY, not just the WHAT.

Every design choice in the spec should carry enough context that someone
reading it understands why it exists — without needing to read the
adversarial transcript.

This means: state the problem before the solution. State the constraint
before the design. State the cost of getting it wrong before the
mechanism that prevents it.

### 4. Do not invent requirements.

If the adversarial session left a question unresolved and the product
owner didn't rule on it, flag it in a "Deferred" or "Open Questions"
section. Do not fill gaps with plausible-sounding design.

### 5. Do not lose signal from rejected proposals.

Ideas that were explicitly rejected or deferred often carry important
context about what NOT to build and why. Capture these in a Deferred
table with clear rationale for deferral.

## Output Structure

Produce a single markdown document with the following sections. Sections
marked "if applicable" should be included only when the feature warrants
them.

### Document Header

```markdown
# {{FEATURE_NAME}}: Feature Specification

**Date:** {{DATE}}
**Version:** 1.0
**Status:** Final design spec. Ready for implementation scoping.
**Companion documents:** {{LIST_RELEVANT_PROJECT_DOCS}}
**Origin:** {{ADVERSARIAL_PROCESS_SUMMARY — e.g., "Three-round multi-model
adversarial design process. Problem statement reviewed by three independent
models, feature proposals solicited independently, reconciled synthesis
pressure-tested, product owner decisions applied."}}
```

### §1. Overview

Start with what the feature IS and why it exists. Not a summary of the
adversarial process — a statement of the problem and the solution.

Structure:
- What is this feature? (1-2 sentences)
- Why does it exist? What fails without it? (the problem)
- What is the cost of the problem? (empirical if possible — dollars,
  time, user outcomes)

If the problem has a severity spectrum or taxonomy (types of failure,
types of user, types of input), define it here. This taxonomy will be
referenced throughout the rest of the spec.

### §2. Architecture

The structural design. How does the feature work at a systems level?

Structure:
- Major components / layers / modules
- How they relate to each other (dependencies, data flow, mutual
  exclusion rules)
- How they integrate with the existing system
- Any constraints that govern the architecture (performance budgets,
  cost ceilings, existing patterns that must be respected)

Use subsections for each major component. Include tables for structured
data (lookup tables, configuration matrices, type mappings).

When describing relationships between components, be explicit about
precedence: "A supersedes B" is clearer than "A and B work together."

### §3. Interaction Model (if user-facing)

How the user experiences the feature. Walk through the interaction
sequence from the user's perspective.

Structure:
- The default path (most common user flow)
- Variant paths (alternative flows for different user types or input
  qualities)
- Available actions at each decision point
- How actions affect system behavior
- What the user sees at each stage

For each action the user can take, state:
- What triggers it
- What happens immediately
- What the user sees next
- What happens in the background (if anything)

If the interaction adapts to context (different behavior for different
input types, user states, etc.), create a table mapping input
conditions to interaction behavior.

### §3.5. Screen Contract (if user-facing)

For each major surface the feature touches, include the finalized Screen
Contract from the reconciled design. This section makes the intended
screen shape explicit for the implementation team.

For each surface:
- **Surface name and purpose**
- **Primary user decision** — what the user decides or does here
- **Dominant element** — what commands attention
- **Secondary elements** — what supports without competing
- **Layout skeleton** — structural regions and containment
- **Default emphasis** — expanded/collapsed/pinned/quiet defaults
- **Required states** — empty, loading, error, success, disabled,
  destructive, long-content, narrow viewport (as applicable)
- **Interaction constraints** — clickable, persistent, collapsible,
  confirm-gated, must-remain-in-view elements
- **Anti-goals** — what this surface must NOT do

This section is not a wireframe or mockup. It is a structural contract
that constrains frontend implementation. When a developer asks "should
this panel be prominent or tucked away?", the answer is in this section.

If the feature is not user-facing, omit this section.

### §4. Input Handling Matrix (if applicable)

If the feature behaves differently based on input characteristics,
create a table that maps every input type to every behavioral dimension.

| Input Type | Behavior A | Behavior B | Behavior C |
|---|---|---|---|
| Type 1 | ... | ... | ... |
| Type 2 | ... | ... | ... |

This table becomes the truth table for implementation. If an edge case
isn't in this table, it hasn't been designed.

### §5. Data Model

Define every new data field, schema change, or persistent state the
feature introduces.

For each field:
- Name, type, nullability, default value
- What populates it and when
- What consumes it and why

If the feature produces a compiled/computed artifact (a prompt, a
report, a derived state), describe the compilation process:
- What inputs go into it
- In what order
- What rules govern assembly
- Where the output is stored

### §6. Transparency / Auditability (if applicable)

If the feature performs transformations on user input or makes decisions
on the user's behalf, describe how the user can inspect what happened.

- What is visible by default vs. on demand?
- What is the inspection surface? (panel, log, endpoint, export)
- What level of detail is shown?

### §7. Deferred Items

A table of every idea that was considered and explicitly deferred. Each
row must include a rationale for deferral — not just "out of scope" but
WHY it's out of scope for this version.

| Item | Rationale for Deferral |
|---|---|
| Feature X | {{specific reason — cost, complexity, dependency, insufficient data, conflicts with existing pattern, etc.}} |

This table serves two purposes:
1. Prevents future designers from re-proposing ideas that were already
   considered and rejected for good reasons.
2. Provides a pre-vetted backlog for future versions.

### §8. Metrics / Success Criteria

What should be measured from launch, and what would each measurement
tell you about whether the feature is working.

| Metric | What It Tells Us |
|---|---|
| Metric A | {{interpretation guidance — what "good" looks like, what "bad" means, expected range}} |

Metrics should be actionable: each one should have an implicit
"if X happens, then Y is the response" associated with it.

### §9. Experiential Quality (if user-facing)

This section is unusual for a spec, and it's the most important one
for features with significant user interaction. It defines what the
feature should FEEL like, not just what it should do.

Structure:
- For each user archetype (defined by the severity/taxonomy in §1),
  describe the ideal experience in 2-3 sentences.
- Define the emotional register: what tone, pacing, and interaction
  style the feature should project.
- Define anti-patterns: specific registers or interaction styles to
  AVOID, with examples. Name the archetype you're rejecting
  (e.g., "the eager intern," "the requirements analyst") and
  explain why.
- Define the target register with a concrete analogy
  (e.g., "a senior analyst who reads the brief and forms a hypothesis").

This section is not fluff. It constrains implementation decisions that
the architecture and interaction model leave ambiguous. When a developer
asks "should this message sound confident or tentative?", the answer
is in this section.

## Quality Gates

Before finalizing the spec, verify:

1. **Decision completeness** — Every contested item from the adversarial
   session has been resolved. No decision is left as "TBD" unless
   explicitly deferred with rationale.

2. **No debate residue** — The spec reads as a coherent design document.
   No "on one hand... on the other hand" hedging. No "Model A
   suggested..." attribution. Every statement is definitive.

3. **Problem-first framing** — Every design choice is preceded by the
   problem it solves. A reader should never encounter a mechanism
   without understanding why it exists.

4. **Edge case coverage** — The input handling matrix (§4) covers every
   realistic input combination. If you can construct a plausible user
   input that falls outside the matrix, the matrix is incomplete.

5. **Deferred table populated** — Every rejected or deferred idea from
   the adversarial session appears in §7 with a rationale. Nothing
   was silently dropped.

6. **Metrics tied to claims** — Every claim in §1 about the problem's
   severity has a corresponding metric in §8 that would validate or
   invalidate it post-launch.

7. **Experiential section grounded** — §9 references the user archetypes
   from §1, not abstract personas. The anti-patterns are specific
   enough that a developer could recognize them in a code review.

8. **Screen Contract present** — For user-facing features, every surface
   in the Screen Contract appears in §3.5 with all required fields
   populated. No surface was silently dropped during spec generation.

9. **Companion doc alignment** — The spec doesn't contradict the project's
   design philosophy, PRD, or other canonical docs. Where it introduces
   a new pattern, it acknowledges the relationship to existing patterns.

## What NOT To Do

- Do NOT preserve the adversarial process structure. The spec is not
  organized by "what Model A said, what Model B said." It's organized
  by feature architecture.

- Do NOT hedge or equivocate. The adversarial process was the place for
  debate. The spec is the place for decisions. Write with conviction.

- Do NOT inflate scope. If the adversarial session explored ideas that
  the product owner didn't explicitly approve, they go in §7 (Deferred),
  not in the architecture.

- Do NOT flatten the problem. If the adversarial session revealed that
  the problem has a severity spectrum or taxonomy, that nuance must
  survive into the spec. A spec that treats all cases the same will
  produce a feature that handles no case well.

- Do NOT skip §9 (Experiential Quality) for user-facing features. This
  is the section that prevents technically correct implementations
  from feeling wrong. It constrains the last-mile decisions that
  architecture diagrams can't capture.

- Do NOT write implementation details. The spec defines WHAT to build
  and WHY. File paths, function names, and API schemas belong in the
  implementation breakdown, not the spec.
  Exception: if the project has existing patterns or extension points
  that the feature must integrate with, reference them by name so the
  spec is grounded in the real codebase — but don't design the
  integration.
```

---

## Placeholder Reference

| Placeholder | Example Value | Notes |
|---|---|---|
| `{{SESSION_ARTIFACTS}}` | List of paths + descriptions | All adversarial session docs (problem statement, proposals, reconciled synthesis) |
| `{{DECISIONS_LOCATION}}` | "Inline in the reconciled synthesis as numbered rulings" or a separate file | Where product owner decisions live |
| `{{CONTEXT_DOCS}}` | Design Philosophy, PRD, Codex, etc. | Project canonical docs the spec must align with |
| `{{TECH_STACK_SUMMARY}}` | `Python 3.11+ FastAPI, React 18 TypeScript, SQLite` | One line — for grounding integration references |
| `{{FEATURE_NAME}}` | `Planning Phase` | Name of the feature |
| `{{DATE}}` | `March 21, 2026` | Spec date |
| `{{ADVERSARIAL_PROCESS_SUMMARY}}` | "Three-round multi-model adversarial design process..." | One-sentence provenance |

## Adapting the Template

**Minimal adversarial input** (2 models, 1 round): The template still applies. Reduce the "Quality Gates" expectations proportionally — fewer contested items means a smaller deferred table.

**No user-facing interaction**: Skip §3 (Interaction Model), §4 (Input Handling Matrix), and §9 (Experiential Quality). The spec collapses to: Overview → Architecture → Data Model → Deferred → Metrics.

**Multiple features from one session**: If the adversarial session covered a broad scope that should decompose into multiple specs, split them. Each spec should be self-contained — a reader should be able to build the feature from one spec without reading the others.

## Relationship to Other Templates

This template produces the input for the **Spec-to-Epics Decomposition** template (`SPEC_TO_EPICS_PROMPT_TEMPLATE.md`). The two form a pipeline:

```
Adversarial Session Artifacts
        │
        ▼
  [This template — Doc 13]
        │
        ▼
  Feature Specification
        │
        ▼
  [Spec-to-Epics template — Doc 14]
        │
        ▼
  Implementation Breakdown
        │
        ▼
  [Implementation PE Prompt — Doc 15]
        │
        ▼
  Multi-Agent Coding Sessions
```
