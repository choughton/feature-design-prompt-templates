---
description: Generate the persistent Principal Engineer prompt for guiding multi-agent implementation.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:pe-setup

Run the implementation PE setup stage. Wraps Template 15.

This stage produces a **prompt artifact**, not a live PE conversation. The output is a configured Principal Engineer prompt ready to paste into a fresh chat (Chat G in the original process) that will then stay open for the duration of implementation.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: `epics` in `completed_stages`. Refuse unless completed or `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/09-epics.md` — the implementation breakdown
- `.feature-design/<slug>/08-spec.md` — design intent
- The source template `15 - IMPLEMENTATION_PE_PROMPT_TEMPLATE.md`. Use Glob with pattern `**/15 - IMPLEMENTATION_PE_PROMPT_TEMPLATE.md` to locate it inside the plugin's install directory, then Read the match.
- The cwd: look for `CLAUDE.md`, `AGENTS.md`, and the architecture doc referenced in `state.project_docs.codex_path`

## Step 3 — Substitute placeholders

The template has many placeholders. Fill them by reading the breakdown and project context:

- `{{FEATURE_NAME}}` → `state.feature_name`
- `{{PROJECT_NAME}}` → derive from cwd folder name, repo name, or ask the user
- `{{IMPLEMENTATION_BREAKDOWN_PATH}}` → `.feature-design/<slug>/09-epics.md`
- `{{FEATURE_SPEC_PATH}}` → `.feature-design/<slug>/08-spec.md`
- `{{PROJECT_LEDGER_PATH}}` → path to `CLAUDE.md` if found, else note absence
- `{{AGENTS_MD_PATH}}` → path to `AGENTS.md` if found; otherwise drop the `{{#if AGENTS_MD_PATH}}…{{/if}}` block
- `{{CODEX_PATH}}` → `state.project_docs.codex_path`
- `{{KEY_FILES_TABLE}}` → assemble from §6 (Epics and Stories) of the breakdown — every file path referenced in any story, with a brief purpose
- `{{AGENT_ROLES_TABLE}}` → ask the user which coding agents they plan to use (e.g., Codex for backend, Antigravity for frontend, Claude Code as auditor); if they don't know, default to a single-agent layout (one coding agent + the PE)
- `{{STATE_SYNC_DESCRIPTION}}` → if multi-agent, describe a `docs/agents/HANDOVER.md` + `HANDBACK.md` + Git SHA flow; if single-agent, simplify to "agent commits to Git; PE reviews handback against story acceptance criteria"
- `{{PHASES_TABLE}}` → copy from §7 (Dependency Graph and Implementation Phasing) of the breakdown
- `{{CRITICAL_PATH}}` → from §7 of the breakdown
- `{{INVARIANTS_TABLE}}` → copy from §8 (Invariants) of the breakdown verbatim
- `{{SETTLED_DECISIONS}}` → extract from the spec (§2 Architecture and §7 Deferred Items)
- `{{HARD_CONSTRAINTS}}` → extract from `CLAUDE.md` / `AGENTS.md` if available; otherwise list from `state.tech_stack` and reasonable project defaults

## Step 4 — Execute

Produce the fully-populated PE prompt by substituting the values into Template 15's body. Preserve the template's structure verbatim — sections 1 through 9 plus the Agent Communication Protocol (§8.1 through §8.5).

The output is a prompt the user will paste into a NEW chat. It is not the PE itself running — it is the configuration artifact.

## Step 5 — Write output

Write to `.feature-design/<slug>/10-pe-prompt.md` with this header:

```markdown
# Principal Engineer Prompt: <feature_name>

**Date:** <ISO date>
**Source:** Template 15 — IMPLEMENTATION_PE_PROMPT_TEMPLATE.md
**Session:** <slug>
**Status:** Ready to paste into a fresh persistent chat (Chat G).

---

## Usage

Open a new chat with your chosen PE model (typically Gemini, but any frontier model). Paste everything below the divider as the system / first prompt. Keep that chat open for the duration of implementation. Route handover/handback artifacts from coding agents through it.

---

<populated PE prompt>
```

## Step 6 — Update state

- Append `"pe-setup"` to `completed_stages`
- Set `current_stage` to `"complete"`

## Step 7 — Report

```
PE prompt complete → 10-pe-prompt.md

Session complete. Pipeline summary:
  ✓ Triage              01-triage.md
  ✓ Exploration         02-exploration.md
  ✓ Problem statement   03-problem-statement.md
  ✓ UI Contract         04-ui-contract.md  (if user-facing)
  ✓ Verification        07-verification.md
  ✓ Spec                08-spec.md
  ✓ Epics breakdown     09-epics.md
  ✓ PE prompt           10-pe-prompt.md

Next step is outside the plugin: paste 10-pe-prompt.md into a fresh persistent chat (Chat G) and begin implementation.
```
