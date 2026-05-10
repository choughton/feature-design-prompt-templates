---
description: Run the feature-proposal crossfire stage. Default mode dispatches to three model CLIs internally; --external hands the prompt to an external crossfire tool.
argument-hint: [--external | --internal] [--allow-partial]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# /feature-design:proposal-crossfire

Run the feature-proposal crossfire stage. Wraps Template 08.

Sends the validated problem + feature-proposal source (and Screen Contract, if user-facing) to three independent models. Each produces a full feature proposal without seeing the others.

Two dispatch modes (same model as `/feature-design:problem-crossfire`):

- **Internal** (default): plugin invokes `claude-cli`, `codex`, `gemini` skills sequentially.
- **External**: plugin saves the prompt for you to run through your own crossfire tool. You drop the three response files into the session folder and reply "ready" to continue.

Default mode comes from `state.crossfire.dispatch_mode`. Override with `--external` or `--internal`.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisites:

- `problem-decision` in `completed_stages`
- If `state.user_facing === true`, also `ui-draft` in `completed_stages`

Refuse unless prerequisites are met or `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/06-feature-proposal-source.md` — the source document (Output B from /feature-design:problem-decision)
- `.feature-design/<slug>/04-ui-contract.md` — the Screen Contract, if user-facing
- The crossfire template `08 - FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md`. Use Glob with pattern `**/08 - FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Substitute placeholders (file paths, NOT inlined content)

Same principle as `/feature-design:problem-crossfire` Step 3 — pass relative paths, not file contents. The dispatched models read files from the project working directory via their CLI's filesystem tools. Inlining defeats this.

In the template body:

- `{{FEATURE_PROPOSAL_DOCUMENT_OR_PATH}}` → relative path `.feature-design/<slug>/06-feature-proposal-source.md`
- `{{#if SCREEN_CONTRACT}}…{{/if}}` block:
  - If `state.user_facing === true`: `{{SCREEN_CONTRACT_DOCUMENT_OR_PATH}}` → relative path `.feature-design/<slug>/04-ui-contract.md`
  - If false: remove the block entirely
- `{{#each COMPANION_DOCS}}…{{/each}}` → expand from `state.project_docs` paths, with section callouts where useful (match the user's example prompt pattern: relative path + § references)

Save the substituted prompt (paths only, no file contents) to `.feature-design/<slug>/.crossfire-prompts/08-proposal.md` for retry/audit.

## Step 4 — Determine dispatch mode

Resolve mode in this order:
1. Explicit `--external` or `--internal` flag in `$ARGUMENTS` overrides everything.
2. Otherwise, use `state.crossfire.dispatch_mode`.
3. Otherwise, default to `internal`.

## Step 5a — Internal dispatch

Skip if mode is `external`.

Same pattern as `/feature-design:problem-crossfire` Step 5a — invoke `claude-cli`, `codex`, `gemini` skills in sequence with the substituted prompt. Save outputs:

- `.feature-design/<slug>/08a-claude.md`
- `.feature-design/<slug>/08b-codex.md`
- `.feature-design/<slug>/08c-gemini.md`

Each with the standard header (date, sou