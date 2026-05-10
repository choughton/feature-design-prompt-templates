---
description: Run an iterative crossfire round (N+1) on the reconciled design. Default dispatches internally; --external hands off to your crossfire tool.
argument-hint: [--external | --internal] [--allow-partial]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# /feature-design:proposal-iterate

Run an iterative crossfire round. Wraps Template 10.

The posture is different from the initial `/feature-design:proposal-crossfire`: reviewers are looking at an already-reconciled design and are asked to identify what's still weak, incomplete, or inconsistent. They cannot relitigate settled decisions from prior rounds.

This command increments the round counter, dispatches the prior round's reconciled design to three models for review, and saves the responses. After this completes, run `/feature-design:proposal-synthesis` again to produce the next round's reconciled design.

Two dispatch modes (same model as the other crossfire commands):

- **Internal** (default): plugin invokes the three model-runner skills.
- **External**: plugin saves the substituted prompt; you run your own crossfire tool and drop the three response files into the session folder.

Default mode comes from `state.crossfire.dispatch_mode`. Override with `--external` or `--internal`.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: most recent stage is `proposal-synthesis-round-<N>` for some N. Refuse unless that's true or `--force`.

## Step 2 — Determine next round number

Read `state.crossfire.current_round`. The new round is `current + 1`. Increment in state.json (will be saved at Step 5).

## Step 3 — Load inputs

- The most recent `09-round<N>-synthesis.md` — the reconciled design under review
- `06-feature-proposal-source.md` for problem context and constraints
- `04-ui-contract.md` if user-facing
- The iterative crossfire template `10 - FEATURE_PROPOSAL_ROUND_N_PLUS_1_CROSSFIRE_PROMPT.md`. Use Glob with pattern `**/10 - FEATURE_PROPOSAL_ROUND_N_PLUS_1_CROSSFIRE_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 4 — Substitute placeholders (file paths, NOT inlined content)

Same principle as the other crossfire commands — pass relative paths so the dispatched models can read files via their CLI's filesystem tools. Don't inline.

In the template body:

- `{{ROUND_NUMBER}}` → the round being reviewed (the prior round, not the new round). For example, on the second invocation of `/feature-design:proposal-iterate` (which produces Round 3 reviews of the Round 2 reconciled design), `{{ROUND_NUMBER}}` is `2`.
- `{{RECONCILED_DESIGN_DOCUMENT_OR_PATH}}` → relative path `.feature-design/<slug>/09-round<N>-synthesis.md`
- `{{#each COMPANION_DOCS}}…{{/each}}` → expand from `state.project_docs` paths with section callouts

Save the substituted prompt (paths only) to `.feature-design/<slug>/.crossfire-prompts/10-round<N+1>.md`.

## Step 5 — Determine dispatch mode

Resolve mode in this order:
1. Explicit `--external` or `--internal` flag in `$ARGUMENTS`.
2. `state.crossfire.dispatch_mode`.
3. Default `internal`.

## Step 5a — Internal dispatch

Skip if mode is `external`.

Same pattern as the other crossfire commands — invoke `claude-cli`, `codex`, `gemini` skills with the substituted prompt. Save outputs:

- `.feature-design/<slug>/10a-round<N+