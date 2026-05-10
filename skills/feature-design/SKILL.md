---
name: feature-design
description: Adversarial-pressure feature design pipeline. Trigger when the user has a new feature idea, wants to triage whether something needs design work, asks for help turning an idea into a spec, wants to break a spec into epics and stories, or needs an implementation Principal Engineer prompt. Use whenever the user mentions "design a feature", "feature idea", "triage", "problem statement", "screen contract", "ui contract", "feature spec", "epics", "stories breakdown", or "implementation handoff".
---

# Feature Design Pipeline

This skill provides a structured pipeline for taking a feature from a raw idea to implementation-ready epics, stories, and a Principal Engineer prompt. It wraps eight prompt templates into slash commands and tracks state across stages in a per-feature session folder.

## When to suggest this skill

Trigger this skill when the user:

- Has a new feature idea and isn't sure how to scope it
- Asks "is this feature already covered by our docs?" or wants triage
- Wants to draft a problem statement
- Wants a Screen Contract / UI structural contract for a user-facing feature
- Wants a feature specification
- Wants to break a spec into epics and stories
- Wants to set up a Principal Engineer prompt for multi-agent implementation
- Mentions any of: feature design, feature idea, triage, problem statement, screen contract, feature spec, epic breakdown, story breakdown, PE setup, principal engineer prompt

Do NOT trigger this skill for:

- Simple bug fixes
- Pure refactors with no design questions
- Tasks that already have settled requirements and just need stories (use `/fd:start` with `--entry-point spec-only` if needed, but prefer skipping the plugin for these)

## How to route the user

There are three entry points depending on session state:

### No active session yet

Recommend `/fd:start <feature-name> -- <raw idea>`. Example:

> Sounds like a feature design task. Run:
>
> ```
> /fd:start <feature-name> -- <one-line description of the idea>
> ```
>
> That will create a session folder, capture the idea, and run triage. From there `/fd:next` walks the rest of the pipeline.

### Active session already exists

Look for `.feature-design/CURRENT` in the working folder. If it exists, read the slug and `.feature-design/<slug>/state.json` to find the current stage. Recommend `/fd:next` to advance, or `/fd:status` for a snapshot.

### Mid-pipeline ask

If the user asks for a specific stage out of context (e.g., "draft a problem statement", "give me a screen contract"), point them at the corresponding command:

| User intent | Command |
|---|---|
| Triage / coverage check | `/fd:triage` |
| Explore the idea | `/fd:explore` |
| Draft problem statement | `/fd:problem` |
| UI / Screen Contract | `/fd:ui-draft` |
| Verify the design | `/fd:verify` |
| Generate spec | `/fd:spec` |
| Break spec into epics | `/fd:epics` |
| Generate PE prompt | `/fd:pe-setup` |

Each step command requires an active session (created via `/fd:start`). If the user wants a one-shot of a stage without ceremony, recommend `/fd:start` first — the session folder is lightweight and the staging makes downstream stages work.

## Session folder layout

Sessions live in `.feature-design/<slug>/` in the user's working folder. The plugin writes:

- `00-feature-idea.md` — raw idea capture
- `01-triage.md` — triage assessment
- `02-exploration.md` — exploration transcript
- `03-problem-statement.md`
- `04-ui-contract.md` (user-facing only)
- `07-verification.md` (slots 05/06 reserved for future crossfire docs)
- `08-spec.md`
- `09-epics.md`
- `10-pe-prompt.md`
- `state.json` — pipeline state

A `.feature-design/CURRENT` file holds the active session's slug.

## What's deferred

Crossfire and decision-synthesis stages (Templates 05, 06, 08, 09, 10, 11 in the original repo) are not in this v1. The pipeline runs single-agent through verification. When crossfire is added later, the numbering gaps in the output filenames give the new stages a place to land without renumbering existing artifacts.
