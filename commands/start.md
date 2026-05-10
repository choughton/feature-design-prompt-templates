---
description: Start a new feature-design session. Initializes session folder, captures the raw idea, runs triage.
argument-hint: <feature-name> [-- raw idea text]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /feature-design:start

Initialize a new feature-design session and run triage.

## Arguments

`$ARGUMENTS` should be a feature name, optionally followed by `--` and a free-form description of the raw idea.

Example: `/feature-design:start auth-rewrite -- session tokens are stored unencrypted in localStorage`

## Step 1 — Parse arguments

Parse `$ARGUMENTS`:

1. Everything before `--` is the **feature name**. Slugify it: lowercase, replace spaces with hyphens, strip non-alphanumeric/hyphen chars. Call this `<slug>`.
2. Everything after `--` is the **raw feature idea** (free text). If absent, ask the user for the raw idea before proceeding.

## Step 2 — Check for existing session

Check whether `.feature-design/<slug>/state.json` already exists in the current working directory.

- If it exists, **do not overwrite**. Report that a session already exists, show its `current_stage` from state.json, and suggest `/feature-design:status` or `/feature-design:next`. Stop.
- If not, proceed.

## Step 3 — Gather missing context

Before creating the session, the user needs to provide context that downstream stages will reference. Asking once up front avoids re-prompting later.

**Required questions** (use AskUserQuestion):

1. **Docs directory** — relative path to the folder where canonical project docs live as markdown (e.g., `./docs/`). This is the **primary** anchor for triage / problem / spec stages. Most users have a `/docs` folder in their repo.
2. **Is this feature user-facing?** — yes / no / unsure (triage will determine). This decides whether the UI Contract stage applies.
3. **Tech stack one-liner** — e.g. `Python 3.11 + FastAPI, React 18 TS, SQLite`. Optional but useful for spec/epics stages.

**Optional doc paths** (ask once; skip-able):

4. **PRD path** — only if the user has a separate PRD document. Many users don't write PRDs; this is fine. Leave blank to skip.
5. **Codex / architecture doc path** — only if there's a single canonical architecture doc. If absent, leave blank — the plugin will scan `docs_dir` for relevant content.
6. **Design philosophy / principles doc path** — only if separately maintained.

**Optional connector setup** (ask once; skip-able):

7. **Linear team or project URL** — only if you want `/feature-design:epics` to optionally push the implementation breakdown to Linear later. Leave blank to skip; you can add it later by editing `state.json`.
8. **Google Drive folder ID** — only if you want commands to be able to pull supplementary materials (customer feedback, samples) from a specific Drive folder. Leave blank to skip.

**Crossfire dispatch preference** (ask once; sets default for all three crossfire commands):

9. **How will crossfire dispatch run?** — `internal` (plugin invokes Codex / Gemini / fresh Claude via the model-runner skills) or `external` (you handle dispatch yourself via your own crossfire tool, then save the response files into the session folder).

   - Pick `internal` if you want the plugin to handle everything end-to-end and you have at least one of the three CLIs (`codex`, `gemini`, `claude`) installed.
   - Pick `external` if you have a dedicated crossfire tool you'd rather use. The plugin still produces the substituted prompts and runs the synthesis stages — it just doesn't invoke the dispatch itself.

   Default for new sessions: `internal`. The choice is stored in `state.crossfire.dispatch_mode` and individual crossfire commands accept `--external` or `--internal` flags to override per-command.

If the user provides a path that doesn't exist, warn but accept — docs may be added later. If they skip everything optional, that's fine; the plugin works with just the docs directory.

## Step 4 — Create session folder

Create:

- `.feature-design/<slug>/` directory
- `.feature-design/CURRENT` file containing just the slug (no newline)
- `.feature-design/<slug>/00-feature-idea.md` 