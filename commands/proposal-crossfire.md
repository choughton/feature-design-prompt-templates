---
description: Run the feature-proposal crossfire stage. Default mode dispatches to three model CLIs internally; --external hands the prompt to an external crossfire tool.
argument-hint: [--external | --internal] [--allow-partial]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# /fd:proposal-crossfire

Run the feature-proposal crossfire stage. Wraps Template 08.

Sends the validated problem + feature-proposal source (and Screen Contract, if user-facing) to three independent models. Each produces a full feature proposal without seeing the others.

Two dispatch modes (same model as `/fd:problem-crossfire`):

- **Internal** (default): plugin invokes `claude-cli`, `codex`, `gemini` skills sequentially.
- **External**: plugin saves the prompt for you to run through your own crossfire tool. You drop the three response files into the session folder and reply "ready" to continue.

Default mode comes from `state.crossfire.dispatch_mode`. Override with `--external` or `--internal`.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisites:

- `problem-decision` in `completed_stages`
- If `state.user_facing === true`, also `ui-draft` in `completed_stages`

Refuse unless prerequisites are met or `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/06-feature-proposal-source.md` — the source document (Output B from /fd:problem-decision)
- `.feature-design/<slug>/04-ui-contract.md` — the Screen Contract, if user-facing
- The crossfire template `08 - FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md`. Use Glob with pattern `**/08 - FEATURE_PROPOSAL_CROSSFIRE_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Substitute placeholders (file paths, NOT inlined content)

Same principle as `/fd:problem-crossfire` Step 3 — pass relative paths, not file contents. The dispatched models read files from the project working directory via their CLI's filesystem tools. Inlining defeats this.

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

Same pattern as `/fd:problem-crossfire` Step 5a — invoke `claude-cli`, `codex`, `gemini` skills in sequence with the substituted prompt. Save outputs:

- `.feature-design/<slug>/08a-claude.md`
- `.feature-design/<slug>/08b-codex.md`
- `.feature-design/<slug>/08c-gemini.md`

Each with the standard header (date, source template, session, model, fresh-context note for Claude, dispatch mode).

**Threshold policy** — same as `/fd:problem-crossfire`:

- 3/3 → proceed.
- 2/3 → stop and prompt user (proceed-partial / retry / external). `--allow-partial` skips the prompt.
- 1/3 or 0/3 → hard fail.

When any model fails, write a retry stub to that file with the prompt path and file list. The synthesis stage tolerates 2/3 (annotates its output) but refuses 1/3 or 0/3. For 0/3, additionally suggest `--external`.

The empirical basis: two independent reviewers capture roughly 90% of the convergence signal; the third's value is diversity-of-thought, not statistical reliability. So 2/3 with explicit acknowledgment is workable for most rounds; 1/3 isn't crossfire at all.

## Step 5b — External dispatch

Skip if mode is `internal`.

Print:

```
External crossfire mode.

The substituted crossfire prompt has been saved to:
  .feature-design/<slug>/.crossfire-prompts/08-proposal.md

Run this prompt through your crossfire tool. Save responses two ways:

  (a) Three separate files matching 08[abc]-*.md:
        .feature-design/<slug>/08a-claude.md   ← Claude run
        .feature-design/<slug>/08b-codex.md    ← Codex/ChatGPT run
        .feature-design/<slug>/08c-gemini.md   ← Gemini run
      (Rename a/b/c suffixes to your actual models — keep the 08[abc]-*.md pattern.)

  (b) A single combined file with all responses:
        .feature-design/<slug>/08-crossfire.md
      (Any name matching 08-*.md that doesn't use the a/b/c letter suffix.)

When ready, reply "ready" to continue.
```

Wait for confirmation. Discover what's there using the same logic as `/fd:problem-crossfire` Step 5b — three-file mode (`08[abc]-*.md`) or combined-file mode (`08-*.md` without a/b/c suffix). Apply the same threshold policy: 3 perspectives → proceed; 2 → prompt the user (proceed / retry / abort, or `--allow-partial` to skip prompt); 1 or 0 → refuse. For combined-file mode, ask the user how many perspectives the file contains.

## Step 6 — Update state (only if 3/3 or user-accepted 2/3)

If only 0 or 1 succeeded, or if 2/3 succeeded and the user chose retry/abort, **do not update state**. Stay at `current_stage: "proposal-crossfire"` so the user can retry. Print the failure summary and stop.

If 3/3 succeeded, or 2/3 succeeded and the user chose proceed (or `--allow-partial`):
- Append `"proposal-crossfire"` to `completed_stages`
- Set `current_stage` to `"proposal-synthesis"`
- Initialize `state.crossfire.current_round` to `1` (this is Round 1 of feature proposal review)
- Initialize `state.crossfire.proposal_rounds` as an empty array; will get appended to by `/fd:proposal-synthesis`
- Record `dispatch_mode_used` on this round (internal | external) and `partial: true | false` for audit

## Step 7 — Report

```
Feature proposal crossfire (Round 1) complete (mode: <internal | external>).
  <a-file>:  <status>
  <b-file>:  <status>
  <c-file>:  <status>

<N>/3 proposals collected.

Recommended next: /fd:next  (will run /fd:proposal-synthesis)

You will be asked to rule on contested items between proposals. Read all three before invoking the synthesis.
```
