# Feature Design Process Guide

**Purpose:** End-to-end workflow for taking a feature idea from initial concept through implementation-ready epics and stories. This process codifies an adversarial multi-model design methodology developed and validated on real product work.

**Who this is for:** The product owner (you) coordinating with LLMs to design, validate, and scope new features.

**What this produces:** A feature specification, implementation breakdown, and Principal Engineer prompt ready for multi-agent coding sessions.

---

## 1. Purpose & When To Use This

This process exists because:

- Multi-model dispatch is expensive. Designing the wrong feature wastes implementation cycles the same way dispatching a bad prompt wastes model calls.
- Your own feature ideas have the same severity spectrum as user prompts: sometimes the intent is clear (Severity 1), sometimes the framing is wrong (Severity 2), sometimes the idea isn't fully formed (Severity 3).
- Adversarial pressure from multiple models surfaces failure modes, design tensions, and edge cases that a single designer misses — including you.

The process produces high-confidence specs by making the design survive structured criticism before any code is written.

---

## 2. Sizing & Shortcuts

Not every feature needs the full pipeline. The triage step (Document 2) determines the path, but here's the decision framework:

### When to run the full pipeline (Documents 2-15)

- The feature introduces a new interaction pattern not covered by the Design Philosophy
- The feature has significant open design questions — you're not sure what the right solution looks like
- The feature crosses multiple system boundaries (frontend + backend + pipeline + data model)
- The feature has downstream cost implications if designed wrong (like Planning — a bad design contaminates all future sessions)
- You have a hunch or half-formed idea, not a clear requirement

### When to skip to spec generation (Documents 13-14 only)

- The feature is already well-specified in the PRD and Codex
- The design questions are implementation-level, not product-level
- The triage prompt (Document 2) confirms full coverage in existing docs

### When to start mid-pipeline

- Problem is clear but solution is open → Start at Document 5 (problem statement crossfire), writing the problem statement yourself
- Problem and solution direction are clear but need validation → Start at Document 8 (feature proposal crossfire)

**The triage step (Document 2) is LLM-assisted.** The PRD and Codex are too large for a human to assess coverage manually. Send the feature idea to an LLM with the triage prompt — it reads the docs and tells you where the gaps are.

---

## 3. Pipeline Overview

```
  ┌─────────────────────────────────────────────────────────────┐
  │                    TRIAGE (Document 2)                      │
  │            Is this feature already defined?                 │
  └──────────┬───────────────────────────────────┬──────────────┘
             │                                   │
        Gaps found                         Fully defined
             │                                   │
             ▼                                   ▼
  ┌─────────────────────┐              ┌──────────────────────┐
  │  IDEA EXPLORATION   │              │  Skip to Document 13 │
  │  (Document 3)       │              │  (Spec Generation)   │
  └──────────┬──────────┘              └──────────────────────┘
             │
             ▼
  ┌─────────────────────┐
  │  PROBLEM STATEMENT  │
  │  SYNTHESIS          │
  │  (Document 4)       │
  └──────────┬──────────┘
             │
             ▼
  ┌─────────────────────┐     ┌─────────────────────┐
  │  CROSSFIRE:         │────▶│  DECISION           │
  │  Problem Review     │     │  CHECKPOINT #1      │
  │  (Document 5 × 3)  │     │  (You make rulings) │
  └─────────────────────┘     └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  DECISION SYNTHESIS  │
                              │  (Document 6)        │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────────────┐
                              │  UI DRAFTING BRIEF           │
                              │  Screen Contract             │
                              │  (Document 7)                │
                              │  [skip if not user-facing]   │
                              └──────────┬──────────────────┘
                                         │
                                         ▼
  ┌─────────────────────┐     ┌─────────────────────┐
  │  CROSSFIRE:         │────▶│  DECISION           │
  │  Feature Proposals  │     │  CHECKPOINT #2      │
  │  (Document 8 × 3)  │     │  (You make rulings) │
  └─────────────────────┘     └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────────────┐
                              │  ROUND N SYNTHESIS           │
                              │  (Document 9)                │
                              │  Reconcile + settle decisions│
                              └──────────┬──────────────────┘
                                         │
                                    ┌────┴────┐
                                    │ Another │
                                    │ round?  │
                                    └──┬───┬──┘
                                  Yes  │   │  No
                       ┌───────────────┘   └──────────┐
                       ▼                              │
           ┌───────────────────────┐                  │
           │  CROSSFIRE:           │                  │
           │  Round N+1 Review     │                  │
           │  (Document 10 × 3)   │                  │
           └───────────┬───────────┘                  │
                       │                              │
                       └──► Back to Document 9        │
                            (next round synthesis)    │
                                                      │
                                         ┌────────────┘
                                         ▼
                              ┌─────────────────────────────┐
                              │  FINAL DECISION SYNTHESIS    │
                              │  (Document 11)               │
                              └──────────┬──────────────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  VERIFICATION        │
                              │  (Document 12)       │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  SPEC GENERATION     │
                              │  (Document 13)       │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  EPIC/STORY          │
                              │  BREAKDOWN           │
                              │  (Document 14)       │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  IMPLEMENTATION      │
                              │  PE SETUP            │
                              │  (Document 15)       │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  MULTI-AGENT         │
                              │  CODING SESSIONS     │
                              │  (PE guides agents)  │
                              └─────────────────────┘
```

### Chat Session Boundaries

| Chat | Documents | Why |
|---|---|---|
| **Chat A** | 2 → 3 → 4 → 6 → 7 | Triage through Screen Contract. Context is cumulative — the moderator needs the full exploration history to synthesize well and to draft screen intent from the problem context. Document 7 (Screen Contract) is skipped for non-user-facing features. |
| **Chat B** (× 3 LLMs) | 5 | Problem statement crossfire. Three separate LLM chats, each receiving the same source document. |
| **Chat C** | 9, 11 | Round N synthesis and final decision synthesis. Fresh context — avoids anchoring to the problem statement framing. Receives only the artifacts: problem statement + proposals + Screen Contract + your decisions. If iterating (Doc 9 → 10 → 9 loop), this chat accumulates the settled decisions across rounds. |
| **Chat D** (× 3 LLMs) | 8 | Initial feature proposal crossfire. Three separate LLM chats. |
| **Chat D′** (× 3 LLMs) | 10 | Iterative crossfire round N+1. Three new chats per iteration round. Each round gets fresh context: the reconciled design from Document 9, not the prior round's raw responses. |
| **Chat E** | 12 | Verification. Independent from the work it's checking — must not be the same chat that produced the reconciled design. |
| **Chat F** | 13 → 14 | Spec generation + epics breakdown. Sequential — the spec feeds directly into the breakdown. Fresh context so the spec stands on its own. |
| **Chat G** | 15 (persistent) | Implementation PE. Stays open for the duration of implementation. Receives handover/handback artifacts from coding agents. |

---

## 4. The Human's Role: Decision Checkpoints

**You are not the reviewer in this process. You are the decision-maker.**

The crossfire rounds (Documents 5, 8, and 10) generate decision surfaces — areas where the models disagree, propose alternatives, or surface tradeoffs. The synthesis prompts (Documents 6, 9, and 11) encode your resolutions into coherent artifacts.

If you skip a decision checkpoint or rubber-stamp it, the downstream artifacts inherit unresolved ambiguity. The spec will hedge. The implementation breakdown will have stories with conflicting acceptance criteria. The coding agents will make inconsistent choices.

### What to decide at each checkpoint

**Decision Checkpoint #1 (after problem statement crossfire):**

- Scope boundaries: what's in and what's out for this feature?
- Problem taxonomy: did the models surface severity levels or failure types you hadn't considered? Which ones are real?
- Cost claims: did any model challenge your cost-of-status-quo argument? Were they right?
- Design principle tensions: which principles take precedence when they conflict?
- Anything the models unanimously flagged as missing or wrong in the problem statement

**Decision Checkpoint #2 (after feature proposal crossfire):**

- Architecture: when models proposed different structural approaches, which one?
- Interaction model: how should the user experience this?
- Mutual exclusion rules: what must never coexist in the system?
- Deferred items: what was proposed but should be cut from this version, and why?
- Contested design choices: when models disagreed on a design decision, what's your ruling?

### How to structure your decisions

Use numbered rulings. For each contested item:

```
1. [Brief description of the contested item]
   Ruling: [Your decision]
   Rationale: [Why — one sentence is fine]
```

This format is machine-readable by the synthesis prompt and produces clean specs.

---

## 5. Step-by-Step Walkthrough

### Step 1: Triage (Document 2)

**Template:** `TRIAGE_PROMPT.md`
**Chat:** Start a new chat (Chat A)
**Input:** Your feature idea (can be as short as one sentence)
**Output:** Coverage assessment + recommended pipeline entry point
**Done when:** You know whether to run the full pipeline, start mid-pipeline, or skip to spec generation

### Step 2: Idea Exploration (Document 3)

**Template:** `IDEA_EXPLORATION_PROMPT.md`
**Chat:** Same chat (Chat A) — the moderator carries forward triage context
**Input:** Your feature idea + triage output
**Output:** A structured understanding of the problem through conversation
**Done when:** The moderator suggests readiness for problem statement synthesis AND you agree. This transition is always your call.

### Step 3: Problem Statement Synthesis (Document 4)

**Template:** `PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md`
**Chat:** Same chat (Chat A) — the moderator synthesizes the exploration it just had with you
**Input:** The exploration conversation (already in context)
**Output:** A standalone problem statement document with challenge questions for crossfire review
**Done when:** You've reviewed the problem statement and confirmed it captures the problem accurately

### Step 4: Problem Statement Crossfire (Document 5)

**Template:** `PROBLEM_STATEMENT_CROSSFIRE_PROMPT.md`
**Chat:** Three NEW separate chats (Chat B × 3) — one per LLM (Claude, ChatGPT, Gemini)
**Input:** The problem statement document from Step 3 + companion project docs
**Output:** Three independent adversarial reviews
**Done when:** All three responses collected

### Step 5: Decision Checkpoint #1

**No template — this is your work.**
**Chat:** Return to Chat A (the moderator chat)
**Input:** The three crossfire responses (paste them in) + your numbered rulings
**What to decide:** Scope, taxonomy, cost claims, design tensions (see §4)
**Done when:** Every contested item has a ruling

### Step 6: Problem Statement Decision Synthesis (Document 6)

**Template:** `PROBLEM_STATEMENT_DECISION_SYNTHESIS_PROMPT.md`
**Chat:** Same chat (Chat A) — the moderator encodes your decisions
**Input:** The three crossfire responses (already pasted) + your rulings
**Output:** A validated, updated problem statement incorporating the crossfire feedback and your decisions. This also produces a feature proposal prompt document — the source doc sent to the next crossfire round.
**Done when:** The feature proposal prompt document captures the validated problem + constraints + what you're asking each model to propose

### Step 7: UI Drafting Brief — Screen Contract (Document 7) *(user-facing features only)*

**Template:** `UI_DRAFTING_BRIEF_PROMPT.md`
**Chat:** Same chat (Chat A) — the moderator has the full problem context from triage through decision synthesis
**Input:** Validated problem statement + feature proposal prompt (Output B from Step 6) + project context docs (especially existing UI patterns in the Codex)
**Output:** A Screen Contract document defining every user-facing surface: purpose, primary user decision, dominant element, layout skeleton, required states, interaction constraints, and anti-goals
**Done when:** Every user-facing surface touched by the feature has a defined hierarchy, required states, and anti-goals. The constraint classification (hard vs. soft) is explicit.
**Skip when:** The feature is backend-only or invisible infrastructure with no UI surface. Note the skip in your decision log and proceed directly to Step 8.

### Step 8: Feature Proposal Crossfire (Document 8)

**Template:** `FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md`
**Chat:** Three NEW separate chats (Chat D × 3) — one per LLM
**Input:** The feature proposal prompt document from Step 6 + Screen Contract from Step 7 (if applicable) + companion project docs
**Output:** Three independent feature proposals. For user-facing features, proposals must respect the Screen Contract's hard constraints.
**Done when:** All three responses collected

### Step 9: Decision Checkpoint #2

**No template — this is your work.**
**Chat:** Start a new chat (Chat C)
**Input:** The feature proposal prompt document + Screen Contract (if applicable) + three crossfire responses + your numbered rulings
**What to decide:** Architecture, interaction model, mutual exclusion, deferred items (see §4)
**Done when:** Every contested item has a ruling

### Step 10: Feature Proposal Round N Synthesis (Document 9)

**Template:** `FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md`
**Chat:** Same chat (Chat C)
**Input:** The three proposals + your rulings (already in context). For subsequent iterations, the prior reconciled design + three Round N+1 responses + your new rulings.
**Output:** A reconciled feature design with settled decisions marked, open items flagged, Screen Contract carried forward (if applicable), and a clear record of what changed vs. what was carried forward from prior rounds.
**Done when:** The reconciled design is coherent and encodes all your rulings. You now decide: is this design ready to finalize, or does it need another adversarial pass?

### Step 11: Iterative Crossfire — Round N+1 (Document 10) *(optional, repeatable)*

**Template:** `FEATURE_PROPOSAL_ROUND_N_PLUS_1_CROSSFIRE_PROMPT.md`
**Chat:** Three NEW chats (Chat D′ × 3) — one per LLM
**Input:** The reconciled design from Step 10 + companion project docs. The posture is "improve this," not "propose from scratch." Models must respect settled decisions and focus on what's weak or incomplete. For user-facing features, reviewers also check for Screen Contract drift.
**Output:** Three independent reviews of the reconciled design
**Done when:** All three responses collected. Return to Step 9 (make new rulings on what the models surfaced), then Step 10 (next round synthesis). Repeat until you're satisfied the design is solid.

**When to loop vs. proceed:** Loop if the reconciled design has unresolved tensions, areas where you're uncertain about the right call, or if the models surfaced substantive concerns you want pressure-tested further. Proceed when the design is stable — additional rounds would produce diminishing returns.

### Step 12: Final Decision Synthesis (Document 11)

**Template:** `FEATURE_PROPOSAL_DECISION_SYNTHESIS_PROMPT.md`
**Chat:** Same chat (Chat C) — carries the full history of settled decisions
**Input:** The final reconciled design from Step 10 (after all iteration rounds are complete) + your cumulative rulings
**Output:** A final reconciled feature design ready for verification. Includes the final Screen Contract (if applicable). This is the single canonical design artifact that downstream steps consume.
**Done when:** The design is stable, all contested items are resolved, and no open questions remain

### Step 13: Verification (Document 12)

**Template:** `VERIFICATION_PROMPT.md`
**Chat:** NEW chat (Chat E) — independent from the design work
**Input:** The final reconciled design from Step 12 + all project reference docs (PRD, Codex, Design Philosophy)
**Output:** A verification report: what's missing, what contradicts existing docs, what's internally inconsistent. For user-facing features, includes Screen Contract alignment checks.
**Done when:** You've reviewed the verification findings and made any final corrections to the reconciled design

### Step 14: Spec Generation (Document 13)

**Template:** `SYNTHESIS_TO_SPEC_PROMPT_TEMPLATE.md`
**Chat:** NEW chat (Chat F) — spec must stand on its own
**Input:** All upstream artifacts (problem statement, proposals, reconciled design, Screen Contract, your decisions)
**Output:** A feature specification ready for implementation scoping. For user-facing features, includes a Screen Contract section (§3.5).
**Done when:** The spec reads as a coherent design document, not a decision log

### Step 15: Epic/Story Breakdown (Document 14)

**Template:** `SPEC_TO_EPICS_PROMPT_TEMPLATE.md`
**Chat:** Same chat (Chat F) — breakdown follows directly from spec
**Input:** The feature spec + access to the codebase
**Output:** Implementation breakdown with epics, stories, acceptance criteria, dependency graph, invariants, test matrix. For user-facing epics, the first frontend story includes a Screen Contract validation acceptance criterion.
**Done when:** Every story has acceptance criteria and test specs, the dependency graph has no cycles, and the invariant table covers all mutual exclusion rules from the spec

### Step 16: Implementation PE Setup (Document 15)

**Template:** `IMPLEMENTATION_PE_PROMPT_TEMPLATE.md`
**Chat:** NEW persistent chat (Chat G) — stays open for the duration of implementation
**Input:** The implementation breakdown from Step 15 + the feature spec + project context docs (CLAUDE.md, AGENTS.md, Codex)
**Output:** A configured Principal Engineer LLM that guides coding agents through implementation. For user-facing features, the PE treats the Screen Contract as a governing artifact.
**Done when:** The PE has read and acknowledged the breakdown, invariants, and phase plan. Implementation can begin.
**Stays open:** This chat remains active throughout implementation. Agents' handover/handback artifacts are routed through the PE for review and sequencing decisions.

---

## 6. Crossfire Session Mechanics

### Running a crossfire round

A crossfire round means sending the same source document and prompt to three independent LLMs in separate chat sessions. The models cannot see each other's responses.

**Model selection:** Use three different frontier-tier models. The standard set is Claude, ChatGPT, and Gemini. Use the best available tier for each — the quality of crossfire feedback is bounded by the weakest model in the round.

**What to send each model:**

1. The source document (problem statement or feature proposal prompt)
2. The crossfire prompt from the relevant template
3. Any companion documents referenced in the source document header

The source document header should list which companion documents to read. Each model reads them independently.

**Collecting responses:**

Copy each model's full response. Do not summarize or edit before pasting into the synthesis step. The moderator LLM and your own judgment both work better from raw outputs than summaries.

### The shared structure of crossfire prompts

Every crossfire prompt (Documents 5, 8, and 10) follows this structure:

1. **Role & Behavioral Instructions** — who you are, how you behave
2. **Task Definition** — what you're doing and what you're producing
3. **Inputs** — artifacts and docs available to you
4. **Assumptions** — what you can take as given
5. **Validity Preconditions** — what would invalidate the premise (folded into adversarial mandate)
6. **Dimensions** — the analytical frame
7. **Outcome Criteria** — what "done" looks like
8. **Constraints** — what not to do
9. **Synthesis Objective** — what the human will do with your response alongside the others

Element 9 is unique to crossfire prompts. It tells each model that its response will be compared against two other independent responses, so it should focus on its strongest arguments rather than trying to be comprehensive on every dimension.

---

## 7. Template Index

| # | Template | File | Type | Used In |
|---|---|---|---|---|
| 2 | Triage Prompt | `TRIAGE_PROMPT.md` | Single LLM | Step 1 |
| 3 | Idea Exploration Prompt | `IDEA_EXPLORATION_PROMPT.md` | Single LLM (moderator) | Step 2 |
| 4 | Problem Statement Synthesis Prompt | `PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md` | Single LLM (moderator) | Step 3 |
| 5 | Problem Statement Crossfire Prompt | `PROBLEM_STATEMENT_CROSSFIRE_PROMPT.md` | Crossfire (each LLM) | Step 4 |
| 6 | Problem Statement Decision Synthesis Prompt | `PROBLEM_STATEMENT_DECISION_SYNTHESIS_PROMPT.md` | Single LLM (moderator) | Step 6 |
| 7 | UI Drafting Brief — Screen Contract | `UI_DRAFTING_BRIEF_PROMPT.md` | Single LLM (moderator) | Step 7 |
| 8 | Feature Proposal Crossfire Prompt | `FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md` | Crossfire (each LLM) | Step 8 |
| 9 | Feature Proposal Round N Synthesis Prompt | `FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md` | Single LLM (moderator) | Step 10 |
| 10 | Feature Proposal Round N+1 Crossfire Prompt | `FEATURE_PROPOSAL_ROUND_N_PLUS_1_CROSSFIRE_PROMPT.md` | Crossfire (each LLM) | Step 11 |
| 11 | Feature Proposal Final Decision Synthesis Prompt | `FEATURE_PROPOSAL_DECISION_SYNTHESIS_PROMPT.md` | Single LLM (moderator) | Step 12 |
| 12 | Verification Prompt | `VERIFICATION_PROMPT.md` | Single LLM | Step 13 |
| 13 | Spec Generation Prompt | `SYNTHESIS_TO_SPEC_PROMPT_TEMPLATE.md` | Single LLM | Step 14 |
| 14 | Spec-to-Epics Decomposition Prompt | `SPEC_TO_EPICS_PROMPT_TEMPLATE.md` | Single LLM | Step 15 |
| 15 | Implementation PE Prompt | `IMPLEMENTATION_PE_PROMPT_TEMPLATE.md` | Single LLM (persistent) | Step 16 |

All templates are located in the root of this repository alongside this process guide.

---

## Appendix: Universal Prompt Skeleton

Every template in this set follows this structural skeleton:

1. **Role & Behavioral Instructions** — who you are, how you behave
2. **Task Definition** — what you're doing and what you're producing
3. **Inputs** — artifacts, docs, context available to you
4. **Assumptions** — what you can take as given without verifying
5. **Validity Preconditions** — what would invalidate the premise, calibrated by prompt type:
   - *Gate prompts* (triage, synthesis): full battery — challenge inputs, check for solved problems, verify problem actually exists
   - *Exploration prompts* (idea exploration): lines of inquiry — what would need to be true for this to be worth building
   - *Crossfire prompts* (adversarial review): folded into the adversarial mandate — "identify assumptions that, if false, collapse the proposal"
   - *Execution prompts* (spec generation, epics): sanity check — do the inputs look well-formed and consistent
6. **Dimensions** — the analytical frame or exploration space
7. **Outcome Criteria** — what "done" looks like, output structure
8. **Constraints** — what not to do, boundaries, anti-patterns

Crossfire prompts add a 9th element: **Synthesis Objective** — what the human will do with your response alongside the other models' responses.
