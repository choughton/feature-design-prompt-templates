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
triage
  → explore
  → problem
  → problem-crossfire
  → problem-decision
  → ui-draft (user-facing only)
  → proposal-crossfire
  → proposal-synthesis  (Round 1)
  → [proposal-iterate → proposal-synthesis]*  (optional iteration loop)
  → proposal-final
  → verify
  → spec
  → epics
  → pe-setup
  → complete
```

**Skip rules:**

- Skip `ui-draft` if `state.user_facing === false`.
- If `state.entry_point === "spec-only"`, skip from `triage` directly to `spec` (skip everything in between).
- If `state.entry_point === "mid-pipeline"`, treat the same as `full` — the user can manually skip stages with `--force`.

**Iteration handling — the special branch:**

After any `/fd:proposal-synthesis` completes, `current_stage` is set to `"proposal-iterate-or-final"`. This is a **virtual stage** that has two valid next moves:

1. `/fd:proposal-iterate` — run another adversarial round on the reconciled design
2. `/fd:proposal-final` — accept the design and lock it in

`/fd:next` cannot decide this for the user. When it sees `current_stage === "proposal-iterate-or-final"`, it should refuse to auto-advance and instead print:

```
You're at a decision point.
  /fd:proposal-iterate   — another adversarial round (recommended if convergence was weak)
  /fd:proposal-final     — finalize this design and proceed to verification

The Round <N> synthesis report flagged convergence as: <signal from state>.
```

Stop after this print. Do not pick.

**Round counting:**

When `proposal-iterate` runs, it appends `proposal-iterate-round-<N+1>` to `completed_stages`. The follow-up `proposal-synthesis` for round N+1 appends `proposal-synthesis-round-<N+1>`. This naming preserves round history in the completed-stages list — the determinator just looks for the highest-numbered `proposal-synthesis-round-*` entry to know what's done.

The next stage is the first stage in the order that is NOT in `state.completed_stages` (treating round-suffixed stages as a single virtual stage for ordering) AND is not skipped.

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
| problem-crossfire | `/fd:problem-crossfire` |
| problem-decision | `/fd:problem-decision` |
| ui-draft | `/fd:ui-draft` |
| proposal-crossfire | `/fd:proposal-crossfire` |
| proposal-synthesis | `/fd:proposal-synthesis` |
| proposal-iterate-or-final | (virtual stage — refuse to auto-advance, see Step 2) |
| proposal-final | `/fd:proposal-final` |
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
