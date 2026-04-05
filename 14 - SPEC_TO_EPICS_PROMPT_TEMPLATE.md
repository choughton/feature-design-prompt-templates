# Spec-to-Epics Decomposition Prompt

**Template #14 in the Feature Design Process**
**Type:** Single LLM, document generation
**Pipeline position:** Step 15 — decomposes the feature spec into implementation-ready epics and stories
**Chat:** Same chat as spec generation (Chat F). The breakdown follows directly from the spec.

**Purpose:** Decomposes a feature specification into implementation-ready epics and stories suitable for multi-agent coding sessions.

**Usage:** Copy the prompt below, replace all `{{PLACEHOLDER}}` fields with your project-specific values, and send to a coding agent (Claude Code, Codex, etc.) with the spec document attached or referenced.

---

## The Prompt

```
You are a senior implementation architect. Your job is to decompose the attached
feature specification into a structured implementation breakdown that multiple
coding agents can execute in parallel with minimal coordination overhead.

## Inputs

**Spec document:** {{SPEC_FILE_PATH_OR_INLINE}} (Output from Document 13 — Spec Generation)
**Codebase reference:** {{REPO_NAME}} @ {{BRANCH}}
**Tech stack:** {{TECH_STACK_SUMMARY}}
**Project context file(s):** {{CLAUDE_MD_OR_EQUIVALENT_PATH}}
{{#if DESIGN_DOCS}}
**Design reference docs (read specific sections as needed, not upfront):**
{{#each DESIGN_DOCS}}
- {{this.path}} — {{this.description}}
{{/each}}
{{/if}}

## Codebase Exploration (MANDATORY FIRST STEP)

Before writing any breakdown, you MUST explore the existing codebase to understand:

1. **Integration points** — Where does this feature hook into existing code?
   Read the files that will be modified or extended. Identify the exact
   functions, classes, endpoints, and components involved.

2. **Data flow** — How does data currently move through the system? Where
   does the new feature intercept, extend, or reroute that flow?

3. **Existing patterns** — What conventions does the codebase use for:
   - API endpoint structure and request/response schemas
   - Database access and schema changes
   - Frontend component composition and state management
   - Test organization and assertion patterns
   - Error handling and edge case conventions

4. **Dependency boundaries** — What modules will this feature need to import
   from? What modules will need to import from this feature?

Do NOT write the breakdown from the spec alone. The spec defines WHAT to build.
The codebase defines HOW it integrates. Both are required.

## Output Structure

Produce a single markdown document with the following sections, in order:

### 1. State Machine (if applicable)

If the feature introduces a lifecycle with discrete states and transitions,
draw the state machine as an ASCII diagram. Include:

- Every state with a short description
- Every transition with the trigger event
- State ownership (frontend, backend, or shared)
- A table with columns: State | Description | Transitions

Skip this section if the feature has no state lifecycle.

### 2. Decision/Truth Table (if applicable)

If the feature has conditional logic where different user paths produce
different system behavior, create a truth table showing:

- Every user path as a row
- Every system behavior dimension as a column
- Clear INCLUDED / NOT INCLUDED / SUPERSEDED markers

This table becomes the single source of truth that agents reference when
implementing branching logic. If two agents disagree about behavior, the
truth table resolves it.

Skip this section if the feature has no conditional branching.

### 3. Event Sequence Table (if applicable)

If the feature involves a multi-step user interaction, create a numbered
sequence table with columns:

- # (sequence number)
- User Action
- Frontend Event
- Backend Call
- UI State Change
- Data Written

Add notes below the table for non-obvious behaviors (e.g., async calls,
data preloading, optimistic updates).

Skip this section if the feature is a single-step operation.

### 4. Backend / Frontend Contract

This section is MANDATORY. Define the API contract before any implementation
stories. It locks the interface between backend and frontend agents.

For each endpoint (new or modified):

```
#### `METHOD /path/to/endpoint`

**Request:**
(JSON schema with types and nullability)

**Response:**
(JSON schema with types)

**Notes:**
- Behavioral notes, edge cases, error conditions
```

Include a data flow diagram showing how frontend components call backend
endpoints and how data flows through the system.

### 5. Screen Contract Validation (user-facing epics only)

For each epic that includes frontend work, the first frontend story
must include the following acceptance criterion:

**Screen Contract check:** Before proceeding with remaining frontend
stories in this epic, validate that the implementation approach matches
the Screen Contract from the spec (§3.5):
- Layout skeleton matches the specified regions
- Dominant element is visually dominant (not competing with secondary)
- Required states are accounted for in component structure
- Anti-goals are not violated by the proposed implementation approach
- Interaction constraints are implementable with the chosen component
  architecture

This is not a separate blocking story — it is a required acceptance
criterion on the first frontend story. The PE or product owner reviews
this before the remaining frontend stories in the epic proceed.

### 6. Epics and Stories

Decompose into epics and stories following these rules:

**Epic boundaries:**
- Each epic has a clear BOUNDARY statement: which layer(s) it touches
  (backend only, frontend only, database only, cross-cutting)
- Epics are ordered by dependency — no epic references work from a later epic
- Each epic can be assigned to a single agent without coordination with
  agents working other epics (after the contract in §4 is locked)

**Story structure:**
For each story within an epic:

```
**E{N}-S{M}: {Story title}**
- {Implementation instruction — what to create or modify, with file paths}
- {Specific logic or behavior to implement}
- **Cross-story dependencies:**
  - CONSTRAINT from E{X}-S{Y}: {What constraint this story inherits from
    another story, and why it matters. Example: "E9-S2 processing modal:
    CONSTRAINT — Round 1 only, per E9-S1 fire-in-place behavior for
    Round 2+. Do NOT render this modal when round > 1."}
  - (Omit this section if the story has no cross-story dependencies.)
- **Acceptance criteria:**
  - {Testable criterion 1}
  - {Testable criterion 2}
  - {Testable criterion 3}
- **Verification contract:**
  - {A concrete self-check the implementing agent MUST perform before
    marking this story done. Not a test — a manual or visual check that
    proves the feature works as intended in context.}
  - {Example: "Render the workspace with a fresh Round 1 session.
    Confirm the Inspector panel is collapsed on initial load."}
  - {Example: "Navigate from Session A to Session B. Confirm Session A's
    data is not visible in Session B's workspace."}
  - {Every story MUST have at least one verification contract step.
    These are NOT duplicates of acceptance criteria — they test the
    story's behavior in its real environment, not in isolation.}
- **Test:** {What to test and how — unit, integration, component, or E2E}
```

**Story rules:**
- Every story references concrete file paths in the codebase
- Every story has at least 2 acceptance criteria
- Every acceptance criteria is machine-testable (not "works correctly")
- Every story has a test specification
- Every story has at least one verification contract step
- Every story with behavior constrained by another story MUST have a
  cross-story dependency callout naming the constraining story and the
  specific constraint. Do not leave cross-story constraints implicit.
- Stories within an epic are ordered by implementation sequence
- No story exceeds ~200 lines of implementation code. If it does, split it.

### 7. Dependency Graph

Draw an ASCII dependency graph showing:
- Which epics depend on which
- Which epics can run in parallel
- The critical path (longest sequential chain)

Follow with an implementation phasing table:

| Phase | Epics | Rationale |
|---|---|---|
| Phase 1 | E{N} → E{M} | Why these go first |
| Phase 2 | E{X} + E{Y} (parallel) | Why these can overlap |
| ... | ... | ... |

Identify: critical path, parallel work opportunities, and the phase
where the backend contract locks (after which frontend work can begin).

### 8. Invariants (Must Never Happen)

Create a table of invariants — conditions that must NEVER be true in any
valid state of the system. These are the rules that, if violated, indicate
a structural bug.

| # | Invariant | Why It Matters | Where To Enforce |
|---|---|---|---|
| INV-1 | {condition that must never be true} | {consequence of violation} | {file/function where guard goes} |

Invariants are derived from:
- Mutual exclusion rules in the spec
- Data integrity constraints
- Timing/ordering requirements
- Security boundaries
- Business logic that must hold across all code paths

These serve as the cross-agent coordination contract. Every agent must
verify their work against the invariant table before marking a story done.

### 9. Telemetry Requirements (if applicable)

If the spec defines metrics or analytics:

**Metrics table:**
| Metric | Computation | Source |

**Events table:**
| Event | Payload | Emitter (backend/frontend) |

Skip this section if the spec has no telemetry requirements.

### 10. Test Matrix

Organize all tests from the stories into a consolidated matrix:

**Unit Tests (Backend):**
| Test | Epic | What It Verifies |

**Integration Tests (Backend):**
| Test | Epic | What It Verifies |

**Component Tests (Frontend):**
| Test | Epic | What It Verifies |

**End-to-End Tests:**
| Test | What It Verifies |

The test matrix serves two purposes:
1. Agents use it as their verification checklist
2. The orchestrator uses it to verify the feature is complete

## Quality Gates

Before finalizing the breakdown, verify:

1. **No orphan stories** — Every story is reachable from the dependency graph.
   No story exists outside an epic.

2. **No circular dependencies** — The dependency graph is a DAG. No epic
   depends on a later epic.

3. **Contract-first** — The API contract (§4) is complete before any
   implementation story references it. Frontend stories reference the
   contract, not backend implementation details.

4. **Invariant coverage** — Every mutual exclusion rule, timing constraint,
   and data integrity requirement from the spec appears in the invariant
   table.

5. **Test coverage** — Every acceptance criterion in every story has a
   corresponding entry in the test matrix. No gaps.

6. **File path grounding** — Every story references real file paths from
   the current codebase (for modifications) or specific new paths (for
   new files) that follow existing project conventions.

7. **Agent independence** — After Phase 2 (contract lock), any frontend
   epic can be worked without knowing backend implementation details,
   and vice versa. If this isn't true, the epic boundaries are wrong.

8. **Verification contract completeness** — Every story has at least one
   verification contract step that tests the story's behavior in its
   real environment (not just unit test isolation). Verification steps
   should catch the class of bugs where implementation passes tests
   but breaks in context (wrong default state, stale data on
   navigation, modal appearing when it shouldn't, etc.).

9. **Cross-story dependency coverage** — Every constraint that flows
   from one story to another is explicitly called out in the
   constrained story's cross-story dependency section. To verify:
   scan each story's behavior and check whether any other story
   modifies the same component, endpoint, or state. If so, there
   must be a dependency callout. Missing dependency callouts are
   the #1 source of bugs in multi-agent implementation.

## What NOT To Do

- Do NOT invent requirements not in the spec. If something seems missing,
  flag it as a question — do not fill the gap with assumptions.
- Do NOT reorganize or re-scope the feature. The spec is the spec.
  Implementation stories decompose it; they don't redesign it.
- Do NOT skip codebase exploration. A breakdown written from the spec
  alone will have wrong file paths, missed integration points, and
  stories that conflict with existing patterns.
- Do NOT create stories without acceptance criteria. "Implement X" is
  not a story. "Implement X such that Y and Z are true" is a story.
- Do NOT create monolithic epics. If an epic has more than 4 stories,
  consider splitting it. If an epic touches both backend and frontend,
  it's probably two epics.
```

---

## Placeholder Reference

| Placeholder | Example Value | Notes |
|---|---|---|
| `{{SPEC_FILE_PATH_OR_INLINE}}` | `docs/PLANNING_PHASE_SPEC.md` | Path or paste inline (from Document 13) |
| `{{REPO_NAME}}` | `crossfire` | Repository name |
| `{{BRANCH}}` | `main` | Branch to explore |
| `{{TECH_STACK_SUMMARY}}` | `Python 3.11+ FastAPI backend, React 18 TypeScript + Vite frontend, SQLite` | One line |
| `{{CLAUDE_MD_OR_EQUIVALENT_PATH}}` | `CLAUDE.md` | Project ledger / context file |
| `{{DESIGN_DOCS}}` | List of paths + descriptions | Optional reference docs |

## Adapting the Template

The template produces all 9 sections by default. Some sections are marked "skip if not applicable" — the agent should make that call based on the spec's complexity:

- **Simple backend-only feature** (e.g., new API endpoint): Sections 4, 5, 6, 7, 9 only
- **Full-stack feature with UI flow**: All 9 sections
- **Database migration or schema change**: Sections 5, 6, 7, 9 — plus a migration safety section
- **Cross-cutting refactor**: Sections 5, 6, 7, 9 — with emphasis on the invariant table

## Tips for Multi-Agent Sessions

1. **Lock the contract first.** The API contract (§4) is the coordination boundary. One agent writes it, everyone else builds to it. Changes after lock require all-agent notification.

2. **Assign epics, not stories.** Each agent gets one or more full epics. Stories within an epic are sequential and should be done by the same agent.

3. **Phase the work.** Backend agents start in Phase 1. Frontend agents start when the contract locks (usually Phase 2 or 3). Telemetry agent goes last.

4. **Use invariants as merge criteria.** Before merging any epic's work, run the invariant table as a checklist. If any invariant is violated, the merge is blocked.

5. **Verification contracts = agent self-check.** Before an agent marks a story done, it must execute every verification contract step and report the result. These catch the "it passes tests but doesn't work in context" class of bugs. A story is not done until verification contracts pass.

6. **Cross-story dependencies = the leash.** Agents building in isolation will miss constraints from other stories. The cross-story dependency callouts make these constraints explicit and unavoidable. When an agent reads a story, the dependencies tell it what NOT to build (or what conditions apply) before it writes a line of code.

7. **Test matrix = done criteria.** The feature is complete when every row in the test matrix passes. Not before.
