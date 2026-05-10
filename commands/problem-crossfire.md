---
description: Run the problem-statement crossfire stage. Default mode dispatches to three model CLIs from inside the plugin; --external mode hands the prompt to an external crossfire tool and waits for response files.
argument-hint: [--external | --internal] [--allow-partial]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Skill
---

# /feature-design:problem-crossfire

Run the problem-statement crossfire stage. Wraps Template 05.

Two dispatch modes:

- **Internal** (default): plugin invokes the `claude-cli`, `codex`, and `gemini` skills sequentially, each with the substituted crossfire prompt. Each skill shells out to its CLI from inside the Bash tool. Useful when you have all three CLIs installed and want everything in-flow.
- **External**: plugin doesn't dispatch. It prints the substituted crossfire prompt so you can run it through your own crossfire tool (e.g., the Crossfire product), then waits for you to drop the three response files into the session folder before continuing.

The default for this command is whatever `state.crossfire.dispatch_mode` says (set during `/feature-design:start`). Pass `--external` or `--internal` to override the session default for this single invocation.

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
2. Otherwise, use `state.crossfire.dispatch_mode` (set at `/feature-design:start`).
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

Crossfire derives signal primarily from convergence between independent perspectives. Empirically, two reviewers capture roughly 90% of the convergence signal; the third reviewer's value is diversity of thought (different training corpora, different reasoning style) rather than statistical reliability. So the threshol