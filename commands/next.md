---
description: Run the next stage in the active feature-design session. Hard-gates on prerequisites unless --force.
argument-hint: [--force]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:next

Advance the active feature-design session by one stage.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` from the current working directory to get the active slug. If the file doesn't exist, tell the user there's no active session and suggest `/fd:start <feature-name>`. Stop.

Read `.feature-design/<slug>/state.json`.

## Step 2 — Determine next stage

Use this canonical stage order:

```
triage → explore → problem → ui-draft → verify → spec → epics → pe-setup → complete
```

**Skip rules:**

- Skip `ui-draft` if `state.user_facing === false`.
- If `state.entry_point === "spec-only"`, skip from `triage` directly to `spec` (skip `explore`, `problem`, `ui-draft`, `verify`).
- If `state.entry_point === "mid-pipeline"`, the user is starting the workflow at problem-statement crossfire (deferred) or feature-proposal crossfire (deferred). For this v1 (no crossfire), treat `mid-pipeline` the same as `full`.

The next stage is the first stage in the order that is NOT in `state.completed_stages` AND is not skipped.

## Step 3 — Hard gate (unless --force)

If `state.strict_mode === true` AND the user did NOT pass `--force`:

- Verify every prerequisite stage (everything earlier in the order, minus skipped stages) is in `state.completed_stages`.
- If any prerequisite is missing, refuse to proceed. Print:

  ```
  Cannot run <next-stage> — prerequisite <missing-stage> has not completed.
  Run /fd:<missing-stage> first, or pass --force to skip the gate.
  ```

  Stop.

If `--force` is passed, log the override in the response (so it's auditable) but proceed.

## Step 4 — Dispatch

Invoke the corresponding step command:

| Next stage | Command |
|---|---|
| triage | `/fd:triage` |
| explore | `/fd:explore` |
| problem | `/fd:problem` |
| ui-draft | `/fd:ui-draft` |
| verify | `/fd:verify` |
| spec | `/fd:spec` |
| epics | `/fd:epics` |
| pe-setup | `/fd:pe-setup` |
| complete | Print "Session complete. All stages done." and stop. |

## Step 5 — Update state

After the dispatched command finishes successfully:

- Append the just-completed stage to `state.completed_stages`.
- Set `state.current_stage` to the *new* next stage (re-run the determination from Step 2).
- Write state.json back.

If the dispatched command itself updates state (recommended), skip this step — `/fd:next` does not double-write.
