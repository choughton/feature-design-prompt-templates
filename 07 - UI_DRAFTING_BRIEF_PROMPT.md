# UI Drafting Brief — Screen Contract Prompt

**Template #7 in the Feature Design Process**
**Type:** Single LLM (moderator), document generation
**Pipeline position:** Step 7 — produces a Screen Contract that constrains frontend interpretation before feature proposals
**Chat:** Same chat as triage/exploration/problem statement/decision synthesis (Chat A). The moderator has the full problem context.

**Applicability:** Required for user-facing features. Skip for backend-only or invisible infrastructure work.

---

## How To Use This Template

1. This step follows the Problem Statement Decision Synthesis (Document 6). Output B from that step should include UI drafting applicability flags and affected surfaces.
2. Send this prompt in the same chat (Chat A) — the moderator already has the validated problem statement, settled decisions, and design context.
3. The LLM produces a Screen Contract document covering every user-facing surface the feature touches.
4. Review the output. The Screen Contract becomes an input to the Feature Proposal Crossfire (Document 8) — proposal models must respect its constraints.

**If the feature is not user-facing:** Skip this step entirely. Proceed directly to Document 8 (Feature Proposal Crossfire).

---

## The Prompt

```
## 1. Role & Behavioral Instructions

You are a product design collaborator producing a Screen Contract — a
structural specification of what each user-facing surface must do, how
it must behave, and what it must not do.

Behavioral rules:
- Structure over polish. You are defining screen intent, not designing
  a visual interface. No colors, no fonts, no spacing values, no
  aesthetic opinions.
- Hierarchy over aesthetics. The Screen Contract's primary job is to
  make dominance and subordination explicit — what commands the user's
  attention and what stays quiet.
- Constraints over creativity. The value of this artifact is what it
  prevents (frontend freelancing) more than what it enables.
- Be specific about behavior. "The user can filter results" is not a
  constraint. "A persistent filter bar is always visible above the
  results list; active filters are shown as dismissible chips; clearing
  all filters resets to the default view" is a constraint.
- Do not invent decorative UI. No flourishes, no novel interaction
  patterns, no visual elements that exist for delight rather than
  function. Every element must earn its space by serving the user's
  primary task.
- Do not add workflow steps unless clearly justified. If the user can
  accomplish their goal in fewer steps, fewer steps wins.
- Do not optimize for novelty. Conventional patterns that users already
  understand are better than clever patterns they have to learn.

## 2. Task Definition

Produce a Screen Contract document for the feature described in the
validated problem statement and decision synthesis from this
conversation.

For every user-facing surface the feature touches (screens, panels,
modals, drawers, inline controls), define:
- what the surface is for,
- what the user decides or does there,
- what dominates the screen,
- what stays secondary,
- the structural layout,
- default visibility and emphasis states,
- required states the surface must handle,
- interaction constraints, and
- what the surface must NOT do.

This document will be sent to three independent LLMs in the next step
(Feature Proposal Crossfire). Proposal models will be told to respect
the Screen Contract's hard constraints and may only deviate from soft
guidance with explicit justification. The Screen Contract persists
through synthesis, verification, spec generation, and implementation.

## 3. Inputs

- **Validated problem statement:** Already in context from this chat
  (Document 6, Output A)
- **Feature proposal prompt document:** Already in context from this
  chat (Document 6, Output B) — includes UI drafting applicability
  flags and affected surfaces
- **Settled decisions from the problem statement round:** Already in
  context
- **Project context docs** (read relevant sections as needed):
  - {{PRD_PATH}}
  - {{DESIGN_PHILOSOPHY_PATH}}
  - {{CODEX_PATH}} — especially UI specs and existing screen patterns
{{#if ADDITIONAL_DOCS}}
{{#each ADDITIONAL_DOCS}}
  - {{this.path}} — {{this.description}}
{{/each}}
{{/if}}
{{#if EXISTING_SCREENS}}
- **Existing screen references** (for features that modify existing
  surfaces rather than creating new ones):
{{#each EXISTING_SCREENS}}
  - {{this.path}} — {{this.description}}
{{/each}}
{{/if}}

## 4. Assumptions

- The validated problem statement is correct and the settled decisions
  are final. The Screen Contract defines HOW the feature appears to
  the user, not WHAT the feature does — that's already decided.
- The project's existing UI patterns (from the Codex and any screen
  references) are the baseline. The Screen Contract should extend
  existing patterns, not invent new paradigms, unless the feature
  genuinely requires it.
- The product owner will review and may amend the Screen Contract
  before it goes to crossfire. The first draft does not need to be
  perfect — it needs to make assumptions visible.

## 5. Validity Preconditions

Before producing the Screen Contract, check:

- **Is this feature actually user-facing?** If the feature is entirely
  backend, pipeline, or infrastructure work with no UI surface, say so
  and recommend skipping this step.
- **Are the affected surfaces identifiable?** If the feature's UI
  footprint is unclear from the problem statement and settled
  decisions, flag which surfaces you're uncertain about and ask the
  product owner before committing to a screen inventory.
- **Does the feature touch existing screens or create new ones?** This
  distinction matters: modifying existing screens requires checking
  current patterns in the Codex; new screens have more latitude but
  must still integrate with existing navigation and layout conventions.
- **Are there existing design philosophy constraints that govern this
  UI?** Check the Design Philosophy for any principles that constrain
  screen behavior (e.g., information hierarchy rules, action models,
  density preferences).

## 6. Dimensions

Produce the Screen Contract with the following structure. Repeat the
full block for each affected surface.

**For each screen / surface:**

### Screen / Surface: [Name]

**Purpose**
What this surface is for — one sentence.

**Primary user decision**
What the user needs to decide, understand, or accomplish on this
surface. This is the single most important thing the surface serves.

**Dominant element**
What should visually carry the screen. The element that commands
attention. There must be exactly one dominant element per surface — if
two things compete for dominance, the hierarchy is broken.

**Secondary elements**
What supports the primary task but must not compete with the dominant
element. List each with a brief note on why it exists.

**Layout skeleton**
A low-fidelity structural description of the layout. Use prose or
lightweight ASCII. Reference standard regions:
- top bar / header
- left rail / sidebar
- main pane / content area
- right pane / detail panel
- sticky footer
- modal body
- drawer
- inline controls
Do not specify pixel values, spacing, or visual styling. Define
spatial relationships and containment.

**Default emphasis**
For each significant element, state its default visibility:
- expanded or collapsed?
- pinned or contextual?
- always visible or shown on demand?
- prominent or quiet?
These defaults define the screen's resting state — what the user sees
before they interact.

**Required states**
Which of these states must the surface handle? For each applicable
state, note what the user should see:
- empty (no data yet)
- loading (data in flight)
- error (operation failed)
- success (operation completed)
- disabled (action unavailable)
- destructive (irreversible action pending)
- long-content / overflow (more data than fits)
- narrow viewport / constrained layout (if applicable)
Not all states apply to all surfaces. Mark inapplicable states as N/A.

**Interaction constraints**
- What is clickable and what happens when clicked
- What expands or collapses and what triggers it
- What persists across navigation vs. what is transient
- What disappears and under what conditions
- What requires confirmation before executing
- What must remain in-view (never scroll out of reach)
- What can be dismissed and how

**Anti-goals / forbidden patterns**
What this surface must NOT do. Be specific. Examples:
- metadata must not overpower findings
- secondary context must not dominate the workspace
- primary CTA must not be below the fold
- decorative elements must not displace signal
- destructive actions must not be hidden or ambiguous
- badges/indicators must not compete with the primary task

This section is the highest-leverage part of the Screen Contract. Be
aggressive about naming what's forbidden — these constraints prevent
more implementation mistakes than the positive specifications.

### Constraint classification

After defining all surfaces, classify each constraint as:

- **Hard constraint:** Proposals must respect this. Deviation requires
  flagging a serious design contradiction or failure mode.
  (Anti-goals, primary user decision, dominant element hierarchy,
  required states)
- **Soft guidance:** Proposals may deviate with justification.
  (Layout skeleton, default emphasis, secondary element placement)

This classification tells proposal models where they have creative
latitude and where they don't.

## 7. Outcome Criteria

The Screen Contract:

- Covers every user-facing surface the feature touches — no surface
  is left unspecified
- Every surface has a single, unambiguous dominant element
- Anti-goals are specific enough that a frontend developer could
  recognize a violation in code review
- Required states are enumerated for every surface where they apply
- The constraint classification is explicit — proposal models know
  what's hard vs. soft
- The document stands alone: a reader unfamiliar with the conversation
  can understand the screen intent from this document + the validated
  problem statement

## 8. Constraints

- Do not design visual UI. No colors, fonts, spacing, border radii,
  shadows, or aesthetic choices. Structure and behavior only.
- Do not produce mockups or wireframes. Prose and lightweight ASCII
  layout descriptions are the medium. If you can't describe it in
  words, the structure isn't clear enough yet.
- Do not invent interaction patterns the project doesn't already use
  unless the feature genuinely requires them. Check the Codex for
  existing patterns first.
- Do not over-specify. The Screen Contract constrains; it does not
  micromanage. Leave room for proposal models to make architectural
  and interaction design choices within the constraints.
- Do not produce a document longer than ~3 pages per surface. If a
  surface needs more than that, it's probably multiple surfaces that
  should be split.
- Do not skip the anti-goals section. This is the most important
  section. If you can't name what the surface must NOT do, the
  constraints aren't well-understood yet.
```

---

## When to Skip This Step

Skip this step entirely when:
- The feature is backend-only (no UI surface at all)
- The feature is invisible infrastructure (logging, caching, migrations)
- The feature is a pure data model change with no frontend impact
- Document 6 Output B explicitly flags UI drafting as not applicable

When skipping, proceed directly to Document 8 (Feature Proposal Crossfire). Note the skip in your decision log so it's traceable.

---

## Relationship to Other Templates

**Upstream:** Consumes the validated problem statement (Document 6 Output A) and the feature proposal prompt (Document 6 Output B).

**Downstream:** The Screen Contract becomes an input to:
- Document 8 (Feature Proposal Crossfire) — proposals must respect hard constraints
- Document 9 (Round N Synthesis) — reconciled design carries forward the Screen Contract
- Document 10 (Round N+1 Crossfire) — reviewers check for Screen Contract drift
- Document 11 (Final Decision Synthesis) — final design includes stable Screen Contract
- Document 12 (Verification) — verifier checks Screen Contract alignment
- Document 13 (Spec Generation) — spec includes Screen Contract section
- Document 14 (Epic/Story Breakdown) — first frontend story validates Screen Contract
- Document 15 (Implementation PE) — PE enforces Screen Contract as governing artifact
