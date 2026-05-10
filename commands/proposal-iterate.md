---
description: Run an iterative crossfire round (N+1) on the reconciled design. Default dispatches internally; --external hands off to your crossfire tool.
argument-hint: [--external | --internal] [--allow-partial]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# /fd:proposal-iterate

Run an iterative crossfire round. Wraps Template 10.

The posture is different from the initial `/fd:proposal-crossfire`: reviewers are looking at an already-reconciled design and are asked to identify what's still weak, incomplete, or inconsistent. They cannot relitigate settled decisions from prior rounds.

This command increments the round counter, dispatches the prior round's reconciled design to three models for review, and saves the responses. After this completes, run `/fd:proposal-synthesis` again to produce the next round's reconciled design.

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

- `{{ROUND_NUMBER}}` → the round being reviewed (the prior round, not the new round). For example, on the second invocation of `/fd:proposal-iterate` (which produces Round 3 reviews of the Round 2 reconciled design), `{{ROUND_NUMBER}}` is `2`.
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

- `.feature-design/<slug>/10a-round<N+1>-claude.md`
- `.feature-design/<slug>/10b-round<N+1>-codex.md`
- `.feature-design/<slug>/10c-round<N+1>-gemini.md`

Same headers. **Same threshold policy as the other crossfire commands**: 3/3 proceeds, 2/3 prompts the user (or `--allow-partial` skips), 1/3 and 0/3 hard fail. Failed models get retry stubs containing the prompt path and file list.

## Step 5b — External dispatch

Skip if mode is `internal`.

Print:

```
External crossfire mode (Round <N+1>).

The substituted prompt has been saved to:
  .feature-design/<slug>/.crossfire-prompts/10-round<N+1>.md

Run this prompt through your crossfire tool. Save responses two ways:

  (a) Three separate files matching 10[abc]-round<N+1>-*.md:
        .feature-design/<slug>/10a-round<N+1>-claude.md
        .feature-design/<slug>/10b-round<N+1>-codex.md
        .feature-design/<slug>/10c-round<N+1>-gemini.md
      (Suffix is whatever model you actually used.)

  (b) A single combined file:
        .feature-design/<slug>/10-round<N+1>-crossfire.md
      (Or any 10-round<N+1>-*.md that doesn't use the a/b/c letter suffix.)

When ready, reply "ready" to continue.
```

Wait for user confirmation. Discover what's there (three-file mode or combined-file mode), apply the same threshold policy: 3 → proceed; 2 → prompt user (or `--allow-partial`); 1 or 0 → refuse. For combined-file mode, ask the user how many perspectives the file contains.

## Step 6 — Update state (only if 3/3 or user-accepted 2/3)

If only 0 or 1 succeeded, or 2/3 with user choosing retry/abort, **do not update state**. Stay at the current state and print the failure summary.

If 3/3, or 2/3 with user choosing proceed (or `--allow-partial`):
- Increment `state.crossfire.current_round` to `<N+1>`
- Append `"proposal-iterate-round-<N+1>"` to `completed_stages`
- Set `current_stage` to `"proposal-synthesis"`
- Record `partial: true | false` on the round metadata for audit

## Step 7 — Report

```
Iterative crossfire (Round <N+1>) complete.
  Claude:  10a-round<N+1>-claude.md  (<status>)
  Codex:   10b-round<N+1>-codex.md   (<status>)
  Gemini:  10c-round<N+1>-gemini.md  (<status>)

<N>/3 reviews collected.

Recommended next: /fd:next  (will run /fd:proposal-synthesis for Round <N+1>)

After the synthesis, you'll again decide whether to iterate further or finalize.
```
