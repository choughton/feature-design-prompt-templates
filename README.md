# Feature Design Templates

A structured, adversarial multi-model workflow for taking a feature from a raw idea to implementation-ready epics, stories, and an execution handoff.

![LLM Feature Design Process](./assets/process_map.jpg)

## What This Repo Contains

This repo is **both** a set of standalone prompt templates **and** a Claude plugin that wraps them into slash commands.

- **15 numbered prompt templates** — usable independently in any LLM. One process guide plus fourteen prompts covering discovery, adversarial review, synthesis, verification, spec generation, and implementation setup.
- **A Claude plugin (`fd`)** — slash commands and a discovery skill that automate triage, exploration, problem statement, UI contract, verification, spec, epics, and PE setup. State is tracked per-feature in a session folder.

The method is best suited to products that already have canonical source docs, a human decision-maker who will make explicit rulings, and a willingness to run multiple independent LLM sessions in parallel.

## Use As a Claude Plugin

The plugin runs the in-scope stages of the pipeline (everything except crossfire, deferred for now) as slash commands.

### Install

This repo is a self-contained plugin — install it as a local plugin in Claude Code or Cowork mode. Once installed, all commands are namespaced under `/fd:`.

### Quick start

```
/fd:start <feature-name> -- <one-line idea>
/fd:next      # walks the rest of the pipeline; hard-gates prerequisites
/fd:status    # snapshot of the active session
```

### Pipeline (v1, no crossfire)

| Command | Wraps template | Output |
|---|---|---|
| `/fd:triage` | 02 | `01-triage.md` |
| `/fd:explore` | 03 | `02-exploration.md` (interactive) |
| `/fd:problem` | 04 | `03-problem-statement.md` |
| `/fd:ui-draft` | 07 | `04-ui-contract.md` (user-facing only) |
| `/fd:verify` | 12 | `07-verification.md` |
| `/fd:spec` | 13 | `08-spec.md` |
| `/fd:epics` | 14 | `09-epics.md` |
| `/fd:pe-setup` | 15 | `10-pe-prompt.md` |

Sessions live in `.feature-design/<slug>/` in your working folder. The numbering gaps at `05`, `06` are reserved for crossfire artifacts when that support lands.

The orchestrator (`/fd:next`) hard-gates progression — it refuses to advance if a prerequisite stage hasn't completed. Pass `--force` to override deliberately.

### Multi-model skills

The plugin also ships three skills that let Cowork Claude shell out to other model CLIs for independent perspectives. They're foundation pieces for the still-deferred crossfire stages, but they're independently usable today for any "second-opinion" task.

| Skill | What it does | CLI it wraps |
|---|---|---|
| `codex` | Invoke OpenAI Codex CLI for an independent OpenAI-side response | `codex exec` |
| `gemini` | Invoke Google Gemini CLI for an independent Google-side response | `gemini -p` |
| `claude-cli` | Spawn a fresh-context Claude session, separate from the moderator | `claude -p` from a scratch directory |

Each skill checks whether its CLI is installed before calling, fails fast with a clear setup message if not, and documents the auth model the user needs (OAuth-friendly where possible — `claude-cli` deliberately avoids `--bare` so subscription auth keeps working).

The moderator (Cowork Claude) doesn't participate in crossfire reviews itself — that would contaminate the adversarial signal. It coordinates and synthesizes. The three skills give it the three independent perspectives crossfire requires.

### What's deferred

Crossfire and decision-synthesis stages (Templates 05, 06, 08, 09, 10, 11) are not yet wired into slash commands. The skills are in place to support them when they land. Use the standalone templates directly if you want to run crossfire manually in the meantime.

## What This Is Not

- **Not an agile story-decomposition tool.** If you already have accepted requirements and just need stories, skip to Document 14.
- **Not a replacement for user research.** There are no real users in this loop — the adversarial pressure comes from models, not humans.
- **Not a single-LLM prompt-tuning framework.** The core value comes from comparing *independent* outputs across three different frontier models. One model with a clever prompt does not produce the same signal.
- **Not a PRD substitute.** You need canonical source docs (PRD, architecture notes, design principles) *before* Document 2 can triage anything.

## Companion Repo

Same underlying idea — *the way you structure AI changes the quality of thinking you get back* — applied at a different layer:

- **[LLM Directive Framework](https://github.com/choughton/llm-directive-framework)** — tune *one* model for sharper, less sycophantic output
- **This repo** — run structured adversarial design pressure across *multiple* models

## Read This First

Start with [`01 - FEATURE_DESIGN_PROCESS.md`](./01%20-%20FEATURE_DESIGN_PROCESS.md).

That document is the operating guide for:

- when to run the full pipeline vs. use shortcuts
- which templates share a chat and which require fresh chats
- what the human must decide at each checkpoint
- how to move from design artifacts into spec, epics, and implementation

## Pipeline Overview

```text
Triage
  -> Idea Exploration
  -> Problem Statement Synthesis
  -> Problem Statement Crossfire
  -> Decision Checkpoint #1
  -> Problem Statement Decision Synthesis
  -> UI Drafting Brief / Screen Contract (user-facing features only)
  -> Feature Proposal Crossfire
  -> Decision Checkpoint #2
  -> Round N Synthesis
  -> Optional Round N+1 Crossfire loops
  -> Final Decision Synthesis
  -> Verification
  -> Spec Generation
  -> Epic / Story Breakdown
  -> Implementation PE Setup
```

## Entry Modes

Not every feature should start at Document 2 and run the full pipeline.

### Full pipeline (Documents 2-15)

Use the full workflow when:

- the feature introduces a new interaction pattern
- the design has significant open questions
- the feature crosses multiple system boundaries
- a bad design would have downstream cost
- you have a hunch or a half-formed idea rather than a settled requirement

### Spec-only shortcut (Documents 13-14)

Skip directly to spec generation and breakdown when:

- the feature is already well-specified in your canonical docs
- the remaining questions are implementation-level, not product-level
- triage confirms the design is already fully covered

### Mid-pipeline starts

Start at a later document when:

- the **problem is clear but the solution is open** -> start at problem-statement crossfire
- the **problem and solution direction are clear but need validation** -> start at feature-proposal crossfire

## Cost & Time Expectations

A full-pipeline run is not cheap and it is not fast. Calibrate before you start.

- **LLM sessions:** ~12-18 distinct chat sessions for a single feature when the pipeline runs end-to-end (three crossfire rounds × three models, plus moderator, synthesis, verification, spec, epics, and PE chats). Each iterative crossfire loop adds three more sessions.
- **Human decision time:** Budget **several hours** across two non-trivial decision checkpoints, plus review time for each crossfire response set. This is not background work — the rulings are the product.
- **Calendar time:** A single feature typically spans multiple sittings. Treat it as a multi-day process, not an afternoon.
- **When to scale down:** Use the **Spec-only shortcut** or a **mid-pipeline start** whenever triage (Document 2) confirms the design is already covered or the open questions are narrow. Running the full pipeline on a small feature is waste.

## Quick Start

1. Open the process guide (`01`) and choose your entry mode.
2. Fill the `{{PLACEHOLDER}}` fields in the relevant template(s).
3. Run prompts in the correct chat/session boundaries.
4. At each decision checkpoint, make explicit numbered rulings yourself.
5. Carry the resulting artifacts forward into verification, spec generation, and implementation setup.

## Chat / Session Map

The workflow is intentionally split across separate chats so synthesis steps do not inherit the wrong context.

| Chat | Documents | Purpose |
|---|---|---|
| **Chat A** | 2 -> 3 -> 4 -> 6 -> 7 | Moderator chat from triage through decision synthesis and optional Screen Contract drafting |
| **Chat B** | 5 | Problem-statement crossfire: 3 separate LLM chats using the same source document (ideally different frontier models like Claude, ChatGPT, Gemini for best adversarial divergence) |
| **Chat C** | 9, 11 | Reconciliation and final synthesis after proposal crossfire |
| **Chat D** | 8 | Initial feature-proposal crossfire: 3 separate LLM chats (ideally different frontier models) |
| **Chat D'** | 10 | Optional iterative crossfire rounds: 3 fresh chats per round (ideally different frontier models) |
| **Chat E** | 12 | Independent verification chat |
| **Chat F** | 13 -> 14 | Spec generation and epic/story decomposition |
| **Chat G** | 15 | Persistent implementation PE chat that stays open during execution |

## Human Role: Decision-Maker, Not Reviewer

This process only works if the human makes explicit rulings.

There are two core decision checkpoints:

- **Checkpoint #1:** after problem-statement crossfire
- **Checkpoint #2:** after feature-proposal crossfire

At each checkpoint, the LLMs generate a decision surface. You decide scope boundaries, contested design choices, architecture tradeoffs, deferred items, and mutual exclusions. The synthesis prompts then encode your rulings into downstream artifacts.

Recommended ruling format:

```text
1. [Contested item]
   Ruling: [Your decision]
   Rationale: [Why]
```

If you skip the rulings, the downstream spec and implementation breakdown will inherit ambiguity.

## Output Artifacts

If you run the full workflow, the repo should produce:

- a triage outcome and recommended entry point
- an explored and synthesized problem statement
- three independent crossfire reviews of the problem
- a validated problem statement plus feature-proposal source document
- a **Screen Contract** for user-facing work
- three independent feature proposals
- a reconciled feature design, optionally across multiple rounds
- an independent verification report
- a feature specification
- an implementation breakdown with epics, stories, acceptance criteria, and dependencies
- a persistent Principal Engineer prompt for multi-agent coding sessions

## Template Families

| Range | Role |
|---|---|
| `01` | Process guide / operating instructions |
| `02-06` | Discovery, problem framing, and problem-level crossfire |
| `07` | UI Drafting Brief / Screen Contract for user-facing features |
| `08-12` | Proposal crossfire, iterative refinement, final synthesis, and verification |
| `13-15` | Implementation handoff: spec, epics, and PE setup |

## Template Index

| # | File | Type |
|---|---|---|
| 01 | [`01 - FEATURE_DESIGN_PROCESS.md`](./01%20-%20FEATURE_DESIGN_PROCESS.md) | Reference doc |
| 02 | [`02 - TRIAGE_PROMPT.md`](./02%20-%20TRIAGE_PROMPT.md) | Single LLM |
| 03 | [`03 - IDEA_EXPLORATION_PROMPT.md`](./03%20-%20IDEA_EXPLORATION_PROMPT.md) | Single LLM (interactive) |
| 04 | [`04 - PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md`](./04%20-%20PROBLEM_STATEMENT_SYNTHESIS_PROMPT.md) | Single LLM |
| 05 | [`05 - PROBLEM_STATEMENT_CROSSFIRE_PROMPT.md`](./05%20-%20PROBLEM_STATEMENT_CROSSFIRE_PROMPT.md) | Crossfire (×3 LLMs) |
| 06 | [`06 - PROBLEM_STATEMENT_DECISION_SYNTHESIS_PROMPT.md`](./06%20-%20PROBLEM_STATEMENT_DECISION_SYNTHESIS_PROMPT.md) | Single LLM |
| 07 | [`07 - UI_DRAFTING_BRIEF_PROMPT.md`](./07%20-%20UI_DRAFTING_BRIEF_PROMPT.md) | Single LLM |
| 08 | [`08 - FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md`](./08%20-%20FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md) | Crossfire (×3 LLMs) |
| 09 | [`09 - FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md`](./09%20-%20FEATURE_PROPOSAL_ROUND_N_SYNTHESIS_PROMPT.md) | Single LLM |
| 10 | [`10 - FEATURE_PROPOSAL_ROUND_N_PLUS_1_CROSSFIRE_PROMPT.md`](./10%20-%20FEATURE_PROPOSAL_ROUND_N_PLUS_1_CROSSFIRE_PROMPT.md) | Crossfire (×3 LLMs) |
| 11 | [`11 - FEATURE_PROPOSAL_DECISION_SYNTHESIS_PROMPT.md`](./11%20-%20FEATURE_PROPOSAL_DECISION_SYNTHESIS_PROMPT.md) | Single LLM |
| 12 | [`12 - VERIFICATION_PROMPT.md`](./12%20-%20VERIFICATION_PROMPT.md) | Single LLM |
| 13 | [`13 - SYNTHESIS_TO_SPEC_PROMPT_TEMPLATE.md`](./13%20-%20SYNTHESIS_TO_SPEC_PROMPT_TEMPLATE.md) | Single LLM |
| 14 | [`14 - SPEC_TO_EPICS_PROMPT_TEMPLATE.md`](./14%20-%20SPEC_TO_EPICS_PROMPT_TEMPLATE.md) | Single LLM |
| 15 | [`15 - IMPLEMENTATION_PE_PROMPT_TEMPLATE.md`](./15%20-%20IMPLEMENTATION_PE_PROMPT_TEMPLATE.md) | Single LLM (persistent) |

## Key Concepts

**Crossfire rounds** send the same source document to three independent LLMs that cannot see one another's responses. Convergence increases confidence. Divergence surfaces the tradeoffs that need a human ruling.

**Screen Contract** (Document 7) is a governing artifact for user-facing features. It defines what each surface must do, what dominates, what must stay quiet, required states, and anti-goals. Backend-only or invisible infrastructure work should skip this step.

**Iterative crossfire** (Documents 9 and 10) allows repeated pressure-testing of the reconciled design. The posture changes from "propose from scratch" to "improve this." Settled decisions accumulate and are not reopened unless the human explicitly changes them.

**Universal prompt skeleton** structures every template to ensure rigorous, predictable outputs. The core elements are:
1. **Role:** The persona the LLM must adopt.
2. **Task:** The specific action required in this round.
3. **Inputs:** The artifacts provided for context.
4. **Assumptions:** Ground truths the LLM must not challenge.
5. **Validity Preconditions:** Checks the LLM must perform before answering.
6. **Dimensions:** The specific angles to analyze (e.g., UX, Edge Cases).
7. **Outcome Criteria:** What a successful output looks like.
8. **Constraints:** Hard boundaries the LLM must not cross.
9. **Synthesis Objective:** (For Crossfire prompts only) Instructions on how to disagree constructively.

## Origins

This repository grew out of a simple idea: **the way you frame AI changes the quality of thinking you get back.**

It started as a frustration. Default AI assistants are often optimized to feel helpful rather than to think sharply. To fix that, I created the **[LLM Directive Framework](https://github.com/choughton/llm-directive-framework)** — a set of personal instructions designed to pull more rigorous, challenging behavior out of models like Claude, ChatGPT, and Gemini. Those directives didn't just change the tone of the interaction; they actively accelerated the design, specification, and build process by keeping the focus on clarity and reducing "flattery noise."

This repository is the next step in the same philosophy. Once you have multiple strong model opinions, the challenge is no longer generation — it's reconciliation. Three different analyses don't automatically create clarity; without structure, they create a new judgment problem. The feature design process outlined in these templates is built to impose that structure, turning multi-model disagreement into a cleaner surface for human decision-making. The goal isn't a nicer AI — it's a sharper one.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.