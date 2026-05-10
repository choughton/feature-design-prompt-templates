---
description: Show current stage, completed stages, and recommended next action for the active feature-design session.
argument-hint: [<slug>]
allowed-tools: Read, Glob, Bash
---

# /feature-design:status

Print a snapshot of the active feature-design session.

## Step 1 — Locate session

If `$ARGUMENTS` provides a slug, use that. Otherwise read `.feature-design/CURRENT` from cwd.

If neither resolves, list all sessions found under `.feature-design/*/state.json` (if any) and tell the user to either pass a slug, run `/feature-design:start`, or set `.feature-design/CURRENT` manually.

## Step 2 — Read state

Read `.feature-design/<slug>/state.json`.

## Step 3 — Print status

Format:

```
Feature: <feature_name>
Slug: <feature_slug>
Created: <created_at>
User-facing: <user_facing>
Entry point: <entry_point>
Strict mode: <strict_mode>

Pipeline:
  [✓] triage          (01-triage.md)
  [✓] explore         (02-exploration.md)
  [→] problem         (current — 03-problem-statement.md)
  [ ] ui-draft        (04-ui-contract.md)
  [ ] verify          (07-verification.md)
  [ ] spec            (08-spec.md)
  [ ] epics           (09-epics.md)
  [ ] pe-setup        (10-pe-prompt.md)

Project docs:
  PRD: <path or "(not set)">
  Codex: <path or "(not set)">
  Design Philosophy: <path or "(not set)">

Tech stack: <tech_stack or "(not set)">

Recommended next: /feature-design:next   (will run /feature-design:problem)
```

Marker semantics:
- `[✓]` — stage in `completed_stages`
- `[→]` — stage equals `current_stage` and is not yet completed
- `[ ]` — pending
- `[—]` — skipped (e.g. `ui-draft` when `user_facing === false`)

## Step 4 — If complete

If `current_stage === "complete"`, replace the recommended-next line with:

```
Session complete. Implementation breakdown is in 09-epics.md and the PE prompt is in 10-pe-prompt.md.
```

## Step 5 — Optional: project tracker integra