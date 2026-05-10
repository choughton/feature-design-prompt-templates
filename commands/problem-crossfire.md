---
description: Run the problem-statement crossfire stage. Default mode dispatches to three model CLIs from inside the plugin; --external mode hands the prompt to an external crossfire tool and waits for response files.
argument-hint: [--external | --internal] [--allow-partial]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# /fd:problem-crossfire

Run the problem-statement crossfire stage. Wraps Template 05.

Two dispatch modes:

- **Internal** (default): plugin invokes the `claude-cli`, `codex`, and `gemini` skills sequentially, each with the substituted crossfire prompt. Each skill shells out to its CLI from inside the Bash tool. Useful when you have all three CLIs installed and want everything in-flow.
- **External**: plugin doesn't dispatch. It prints the substituted crossfire prompt so you can run it through your own crossfire tool (e.g., the Crossfire product), then waits for you to drop the three response files into the session folder before continuing.

The default for this command is whatever `state.crossfire.dispatch_mode` says (set during `/fd:start`). Pass `--external` or `--internal` to override the session default for this single invocation.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and `.feature-design/<slug>/state.json`. Prerequisite: `problem` in `completed_stages`. Refuse unless completed or `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/03-problem-statement.md` — the source document under review
- The crossfire template `05 - PROBLEM_STATEMENT_CROSSFIRE_PROMPT.md`. Use Glob with pattern `**/05 - PROBLEM_STATEMENT_CROSSFIRE_PROMPT.md` to locate it inside the plugin's install directory, then Read the match.

## Step 3 — Substitute placeholders (file paths, NOT inlined content)

The dispatched models run from the project's working directory and read referenced files via their CLI's filesystem tools. **Do not inline file contents** — pass relative paths and let the reviewer read them. Inlining defeats the file-access affordance and reduces crossfire to "respond to whatever the moderator remembered to paste."

In the template's `## The Prompt` body:

- `{{PROBLEM_STATEMENT_DOCUMENT_OR_PATH}}` → the relative path `.feature-design/<slug>/03-problem-statement.md`
- `{{#each COMPANION_DOCS}}…{{/each}}` → expand using paths from `state.project_docs`:
  - `<docs_dir>/...` for files in the docs directory (enumerate with Glob to get the actual filenames)
  - Individual paths from `prd_path`, `codex_path`, `design_philosophy_path` if populated
  - Each entry should include the path + a brief description of what's in it (e.g., `docs/PRD.md` — esp. §3 Non-Goals, §4.1 Pipeline)
  - Match the user's example pattern: relative path + section callouts where useful

Save the substituted prompt (the prompt + file references, NOT the file contents) to `.feature-design/<slug>/.crossfire-prompts/05-problem.md`. This file is what the user copies if they need to retry a model externally, and it's the audit record of what got dispatched.

## Step 4 — Determine dispatch mode

Resolve mode in this order:
1. Explicit `--external` or `--internal` flag in `$ARGUMENTS` overrides everything.
2. Otherwise, use `state.crossfire.dispatch_mode` (set at `/fd:start`).
3. Otherwise, default to `internal`.

## Step 5a — Internal dispatch (plugin invokes the three skills)

Skip this step if mode is `external`.

Invoke each model-runner skill in turn. Sequential is intentional — easier to debug, no parallel-Bash complexity.

### Model 1: fresh Claude

Use the `claude-cli` skill. Pass the substituted crossfire prompt as the input. Capture stdout. Save to `.feature-design/<slug>/05a-claude.md` with this header:

```markdown
# Problem Statement Crossfire — Claude (fresh session)

**Date:** <ISO date>
**Source:** Template 05
**Session:** <slug>
**Model:** Claude (via claude-cli skill, fresh-context)
**Dispatch:** Internal (plugin)

---

<full response>
```

### Model 2: Codex

Use the `codex` skill with the same crossfire prompt. Save stdout to `.feature-design/<slug>/05b-codex.md` with an analogous header.

### Model 3: Gemini

Use the `gemini` skill. Save stdout to `.feature-design/<slug>/05c-gemini.md` with an analogous header.

### Failure handling

Crossfire derives signal primarily from convergence between independent perspectives. Empirically, two reviewers capture roughly 90% of the convergence signal; the third reviewer's value is diversity of thought (different training corpora, different reasoning style) rather than statistical reliability. So the threshold isn't strict 3/3 — but 1/3 isn't crossfire at all, since a single response has nothing to converge against.

Policy:

- **3/3 succeeded** → proceed.
- **2/3 succeeded** → stop and prompt the user (or accept silently if `--allow-partial` was passed). Three options:
  1. **Retry the failed model** — fix the CLI / auth / network and re-run the dispatch.
  2. **Run it externally** — copy the prompt from `.crossfire-prompts/05-problem.md`, run against any model in that provider's family, save response to the failed file slot, reply ready.
  3. **Proceed with 2/3** — accept the diversity tradeoff; synthesis will run with a "produced from 2 of 3 perspectives" annotation.
- **1/3 succeeded** → hard fail. Single-perspective review isn't crossfire. The user can read the one response on its own merits, but synthesis is just summarization at that point.
- **0/3 succeeded** → hard fail. If running internal, suggest switching to `--external` for this stage.

When any model fails, write a **retry stub** to that response file:

```markdown
# Problem Statement Crossfire — <Model> (FAILED — RETRY REQUIRED)

**Reason:** <specific error: CLI not installed / auth expired / network error / etc.>
**Date attempted:** <ISO date>

## To resolve

Run the prompt manually against <Model> (or any equivalent <provider>-side model) and overwrite this file with the response.

- **Prompt to run:** `.feature-design/<slug>/.crossfire-prompts/05-problem.md`
- **Files the reviewer needs to read** (referenced in the prompt):
  - `.feature-design/<slug>/03-problem-statement.md`
  - `<list of companion files from state.project_docs>`
- **Where to save the response:** overwrite this file (`.feature-design/<slug>/05<a|b|c>-<model>.md`) with the full response. Replace this stub entirely, including the heading. Keep the standard header (date, source, session, model, dispatch).

When done, re-run `/fd:problem-crossfire` to re-validate, or run `/fd:next` directly if you've populated the file with a real response.
```

For the 2/3 case: print the summary and prompt:

```
Crossfire round produced 2 of 3 responses.

Successful:
  <list of files>

Failed (retry stub written):
  <model>: <reason>

Two reviewers capture most of the convergence signal; the missing third is diversity-of-thought. You can:
  1. Retry the failed model (fix the CLI/auth/network and re-run /fd:problem-crossfire)
  2. Run the failed model externally — prompt is at .crossfire-prompts/05-problem.md, save response over the retry stub
  3. Proceed with 2/3 — synthesis will note the missing perspective. Reply "proceed".

What would you like to do?
```

For 1/3 or 0/3, do not prompt — print the summary and stop. State does not advance.

For 3/3 or accepted-2/3, the stage advances normally; in the 2/3 case, set `state.crossfire.problem_round.partial = true` so the synthesis stage knows to annotate its output.

## Step 5b — External dispatch (user runs crossfire via their own tool)

Skip this step if mode is `internal`.

Print the substituted crossfire prompt path and instructions:

```
External crossfire mode.

The substituted crossfire prompt has been saved to:
  .feature-design/<slug>/.crossfire-prompts/05-problem.md

Run this prompt through your crossfire tool. You can save responses two ways:

  (a) Three separate files — one per model — matching the 05[abc]-*.md pattern:
        .feature-design/<slug>/05a-claude.md   ← Claude / Anthropic-side run
        .feature-design/<slug>/05b-codex.md    ← Codex / OpenAI-side run
        .feature-design/<slug>/05c-gemini.md   ← Gemini / Google-side run
      (Suffix names can be anything — e.g., 05a-opus.md, 05b-grok.md — as long as
      the 05[abc]-*.md pattern holds.)

  (b) A single combined file containing all responses — useful if your crossfire
      tool produces one output document. Save as:
        .feature-design/<slug>/05-crossfire.md
      (Or any name matching 05-*.md that doesn't use the a/b/c letter suffix.)

When ready, reply "ready" to continue.
```

Wait for the user to confirm. After they reply "ready" (or equivalent), discover what's there:

1. **Three-file mode:** look for files matching `.feature-design/<slug>/05[abc]-*.md`. If present, count how many have nontrivial content (not empty, not retry stubs from a previous internal-dispatch attempt).
2. **Combined-file mode:** look for files matching `.feature-design/<slug>/05-*.md` that do NOT have an `a`/`b`/`c` letter immediately after `05`. If exactly one such file is found, treat it as a combined-perspectives document.
3. **Hybrid:** if both patterns match (e.g., user has both `05a-claude.md` and `05-crossfire.md`), prefer whichever has more apparent perspectives. Tell the user what you found and ask which to use.

For three-file mode, apply the standard threshold policy:
- 3/3 → proceed.
- 2/3 → prompt the user (proceed-partial / retry / abort). `--allow-partial` skips the prompt.
- 1/3 → refuse.
- 0/3 → refuse; suggest checking the file paths.

For combined-file mode, ask the user how many independent perspectives the file contains:
- 3 → proceed; the synthesis stage will be told to parse 3 independent responses out of the combined input.
- 2 → prompt to confirm proceeding partial (or accept silently with `--allow-partial`); flag synthesis to expect 2.
- 1 → refuse — single-perspective isn't crossfire.

Once threshold is met, update `state.crossfire.problem_round.responses` to reflect what was found:

- For three-file mode: array of `{model, file, status: "ok", dispatch: "external"}` entries.
- For combined-file mode: a single entry like `{model: "combined", file: "05-crossfire.md", perspective_count: 3, dispatch: "external"}`. The synthesis stage reads this and parses the file into independent responses.

Set `partial: true` on the round metadata if proceeding from 2 perspectives, regardless of file mode.

## Step 6 — Update state (only if 3/3 or user-accepted 2/3)

If only 0 or 1 succeeded, or if 2/3 succeeded and the user chose retry/external, **do not update state**. Stay at `current_stage: "problem-crossfire"` so the user can retry. Print the failure summary and stop.

If 3/3 succeeded, or 2/3 succeeded and the user chose proceed (or `--allow-partial` was passed):

- Append `"problem-crossfire"` to `completed_stages`
- Set `current_stage` to `"problem-decision"`
- Set `crossfire.problem_round.responses` in state.json to an array tracking which models responded and how:

```json
"crossfire": {
  "problem_round": {
    "responses": [
      {"model": "claude", "file": "05a-claude.md", "status": "ok", "dispatch": "internal"},
      {"model": "codex",  "file": "05b-codex.md",  "status": "ok", "dispatch": "internal"},
      {"model": "gemini", "file": "05c-gemini.md", "status": "ok", "dispatch": "internal"}
    ]
  }
}
```

For external mode, `model` reflects whatever the user actually ran (could be "opus", "sonnet45", "grok" — pulled from the filename suffix or asked of the user), and `dispatch` is `"external"`.

Also set `state.crossfire.problem_round.partial = true` if proceeding from 2/3, otherwise `false`.

## Step 7 — Report (only on full or accepted-partial success)

```
Problem statement crossfire complete (mode: <internal | external>).
  <a-file>:  ok
  <b-file>:  ok
  <c-file>:  <ok | retry-stub: <reason>>

<3/3 | 2/3 — proceeding partial> perspectives collected.

Recommended next: /fd:next  (will run /fd:problem-decision)
```

If proceeding partial, additionally print:

```
Note: This crossfire round captured 2 of 3 perspectives. The synthesis output will carry a "produced from 2 of 3" annotation so downstream stages know.
```
