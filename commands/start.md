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

7. **Linear team or project URL** — only if you want `/fd:epics` to optionally push the implementation breakdown to Linear later. Leave blank to skip; you can add it later by editing `state.json`.
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
    "docs_dir": "<path to local /docs folder>",
    "prd_path": "<path or empty — PRD is optional>",
    "codex_path": "<path or empty>",
    "design_philosophy_path": "<path or empty>",
    "additional_docs": []
  },
  "tech_stack": "<string>",
  "connectors": {
    "project_tracker": {
      "enabled": false,
      "type": null,
      "team_id": null,
      "project_id": null,
      "url": null,
      "linked_issues": {}
    },
    "drive_folder_id": null
  },
  "crossfire": {
    "dispatch_mode": "internal",
    "current_round": 0,
    "iteration_complete": false,
    "final_design_file": null,
    "problem_round": {
      "responses": [],
      "rulings": []
    },
    "proposal_rounds": []
  },
  "stage_outputs": {
    "triage": "01-triage.md",
    "explore": "02-exploration.md",
    "problem": "03-problem-statement.md",
    "problem-crossfire": ["05a-claude.md", "05b-codex.md", "05c-gemini.md"],
    "problem-decision": ["06-validated-problem.md", "06-feature-proposal-source.md"],
    "ui-draft": "04-ui-contract.md",
    "proposal-crossfire": ["08a-claude.md", "08b-codex.md", "08c-gemini.md"],
    "proposal-synthesis": "09-round{N}-synthesis.md",
    "proposal-iterate": ["10a-round{N}-claude.md", "10b-round{N}-codex.md", "10c-round{N}-gemini.md"],
    "proposal-final": "11-final-design.md",
    "verify": "07-verification.md",
    "spec": "08-spec.md",
    "epics": "09-epics.md",
    "pe-setup": "10-pe-prompt.md"
  }
}
```

**Crossfire fields:**
- `crossfire.current_round` — 0 before any proposal crossfire runs; incremented to 1 after `/fd:proposal-crossfire`, then again by each `/fd:proposal-iterate`.
- `crossfire.iteration_complete` — `true` once `/fd:proposal-final` has run. After this, the crossfire pipeline is closed.
- `crossfire.final_design_file` — the canonical design file (`11-final-design.md`) once finalized. Downstream stages (`/fd:verify`, `/fd:spec`, etc.) consume this rather than the round-by-round synthesis files.
- `crossfire.problem_round.responses` — array of `{model, file, status}` capturing which models responded successfully to the problem statement crossfire.
- `crossfire.problem_round.rulings` — the user's numbered rulings from `/fd:problem-decision`, retained for audit.
- `crossfire.proposal_rounds` — array of per-round metadata (`round`, `synthesis_output`, `rulings`, `diminishing_returns_signal`).

**Note on stage_outputs paths:** several names appear in two places by design (`08-spec.md` is both `proposal-crossfire` outputs `08a/b/c-*.md` and the spec command's output `08-spec.md`). The numbering doesn't actually collide because crossfire outputs use letter suffixes (a/b/c) and round numbers, while standalone outputs are plain. If you ever notice a real collision in your session folder, file a bug.

**Connector fields:**
- `connectors.project_tracker.enabled` — set to `true` if the user supplied a Linear (or other tracker) URL during setup. Toggles `/fd:epics` push behavior and `/fd:status` issue-state querying.
- `connectors.project_tracker.type` — `"linear"`, `"asana"`, etc. Filled if the user explicitly named one; left null if Claude should auto-resolve via the `~~project tracker` placeholder.
- `connectors.project_tracker.linked_issues` — populated by `/fd:epics` after pushing: `{ "E1-S1": "ABC-123", ... }`.
- `connectors.drive_folder_id` — Google Drive folder ID for supplementary materials. Optional; commands check for it before attempting Drive lookups.

`current_stage` values: `triage` | `explore` | `problem` | `problem-crossfire` | `problem-decision` | `ui-draft` | `proposal-crossfire` | `proposal-synthesis` | `proposal-iterate-or-final` (virtual) | `proposal-final` | `verify` | `spec` | `epics` | `pe-setup` | `complete`

For per-round stages, `completed_stages` uses suffixed names — e.g. `proposal-synthesis-round-1`, `proposal-iterate-round-2`, `proposal-synthesis-round-2` — so the round history is preserved.

`entry_point` values: `full` | `mid-pipeline` | `spec-only` (set by triage; defaults to `full`).

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
