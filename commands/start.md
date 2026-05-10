---
description: Start a new feature-design session. Initializes session folder, captures the raw idea, runs triage.
argument-hint: <feature-name> [-- raw idea text]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:start

Initialize a new feature-design session and run triage.

## Arguments

`$ARGUMENTS` should be a feature name, optionally followed by `--` and a free-form description of the raw idea.

Example: `/fd:start auth-rewrite -- session tokens are stored unencrypted in localStorage`

## Step 1 — Parse arguments

Parse `$ARGUMENTS`:

1. Everything before `--` is the **feature name**. Slugify it: lowercase, replace spaces with hyphens, strip non-alphanumeric/hyphen chars. Call this `<slug>`.
2. Everything after `--` is the **raw feature idea** (free text). If absent, ask the user for the raw idea before proceeding.

## Step 2 — Check for existing session

Check whether `.feature-design/<slug>/state.json` already exists in the current working directory.

- If it exists, **do not overwrite**. Report that a session already exists, show its `current_stage` from state.json, and suggest `/fd:status` or `/fd:next`. Stop.
- If not, proceed.

## Step 3 — Gather missing context

Before creating the session, the user needs to provide their canonical project doc paths. These are referenced by every downstream stage and locking them in once avoids re-prompting.

Use the AskUserQuestion tool to ask:

1. **PRD path** — relative path from cwd. Default: ask the user to type it.
2. **Codex / architecture doc path** — relative path. Default: ask.
3. **Design philosophy / principles doc path** — relative path. Default: ask.
4. **Is this feature user-facing?** — yes / no / unsure (triage will determine). This decides whether the UI Contract stage applies.
5. **Tech stack one-liner** — e.g. `Python 3.11 + FastAPI, React 18 TS, SQLite`. Optional but useful for spec/epics stages.

If the user provides a path that doesn't exist, warn them but accept it — the docs may be added later.

## Step 4 — Create session folder

Create:

- `.feature-design/<slug>/` directory
- `.feature-design/CURRENT` file containing just the slug (no newline)
- `.feature-design/<slug>/00-feature-idea.md` — write a markdown file with `# <feature-name>` heading and the raw idea body
- `.feature-design/<slug>/state.json` — see schema below

### state.json schema

```json
{
  "feature_slug": "<slug>",
  "feature_name": "<original name>",
  "created_at": "<ISO 8601 UTC>",
  "user_facing": true | false | null,
  "current_stage": "triage",
  "completed_stages": [],
  "entry_point": "full",
  "strict_mode": true,
  "project_docs": {
    "prd_path": "<path or empty>",
    "codex_path": "<path or empty>",
    "design_philosophy_path": "<path or empty>",
    "additional_docs": []
  },
  "tech_stack": "<string>",
  "stage_outputs": {
    "triage": "01-triage.md",
    "explore": "02-exploration.md",
    "problem": "03-problem-statement.md",
    "ui-draft": "04-ui-contract.md",
    "verify": "07-verification.md",
    "spec": "08-spec.md",
    "epics": "09-epics.md",
    "pe-setup": "10-pe-prompt.md"
  }
}
```

`current_stage` values: `triage` | `explore` | `problem` | `ui-draft` | `verify` | `spec` | `epics` | `pe-setup` | `complete`

`entry_point` values: `full` | `mid-pipeline` | `spec-only` (set by triage; defaults to `full`)

The numbered output files leave gaps at `05`, `06`, `08`/`09`/`10`/`11` (in the original 15-doc numbering) for the crossfire/synthesis docs to be added later — that's why `verify` is `07-verification.md` not `05`.

## Step 5 — Run triage

Once the session is created, immediately invoke `/fd:triage` so the user gets coverage analysis before doing anything else. Do not pause for confirmation — they just opted into the pipeline by running `/fd:start`.

## Step 6 — Report

Print a concise summary:

```
Started session: <slug>
Folder: .feature-design/<slug>/
Idea captured: 00-feature-idea.md
Running /fd:triage now...
```

Then proceed to triage.
