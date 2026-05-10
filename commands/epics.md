---
description: Decompose the spec into implementation-ready epics and stories. Requires codebase access.
argument-hint: (uses active session)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# /fd:epics

Run the spec-to-epics decomposition stage. Wraps Template 14.

**Important:** This stage REQUIRES codebase exploration. The template explicitly forbids writing the breakdown from the spec alone. Before producing any output, you must read enough of the working folder's actual code to ground every story in real file paths and existing patterns.

## Step 1 — Locate active session

Read `.feature-design/CURRENT` and state.json. Prerequisite: `spec` in `completed_stages`. Refuse unless completed or `--force`.

## Step 2 — Load inputs

- `.feature-design/<slug>/08-spec.md`
- The source template `14 - SPEC_TO_EPICS_PROMPT_TEMPLATE.md`. Use Glob with pattern `**/14 - SPEC_TO_EPICS_PROMPT_TEMPLATE.md` to locate it inside the plugin's install directory, then Read the match.
- The cwd codebase itself

## Step 3 — Substitute placeholders

- `{{SPEC_FILE_PATH_OR_INLINE}}` → `.feature-design/<slug>/08-spec.md`
- `{{REPO_NAME}}` → derive from cwd folder name or `git remote get-url origin` if available
- `{{BRANCH}}` → `git branch --show-current` if in a git repo, else `main`
- `{{TECH_STACK_SUMMARY}}` → `state.tech_stack`
- `{{CLAUDE_MD_OR_EQUIVALENT_PATH}}` → look for `CLAUDE.md`, `AGENTS.md`, or similar in cwd; if none, note the absence
- `{{#if DESIGN_DOCS}}…{{/if}}` → expand from `state.project_docs.additional_docs` if present

## Step 4 — Mandatory codebase exploration

Per the template's "Codebase Exploration (MANDATORY FIRST STEP)" section, before writing the breakdown:

1. **Integration points** — read files that will be modified or extended; identify exact functions, classes, endpoints, components
2. **Data flow** — understand how data moves through the system; where the new feature intercepts or extends it
3. **Existing patterns** — API endpoint conventions, DB access, frontend composition, test organization, error handling
4. **Dependency boundaries** — what modules will be imported from / into

Use Glob and Grep aggressively. Read the spec's architecture section, then trace integration points into the actual code.

## Step 5 — Execute

Produce the breakdown per the template's `## Output Structure`:

1. State Machine (if applicable — ASCII diagram + states/transitions table)
2. Decision/Truth Table (if applicable)
3. Event Sequence Table (if applicable)
4. **Backend / Frontend Contract** (MANDATORY — locks the API before any implementation story)
5. Screen Contract Validation (user-facing epics only — first frontend story carries the validation acceptance criterion)
6. **Epics and Stories**:
   - Epic boundaries (which layers; ordered by dependency)
   - For each story: implementation instruction with file paths, cross-story dependencies, ≥2 testable acceptance criteria, verification contract, test spec
   - No story exceeds ~200 lines of implementation
7. Dependency Graph (ASCII) + Implementation Phasing Table
8. Invariants table (must-never-be-true conditions, where to enforce)
9. Telemetry Requirements (if applicable)
10. Test Matrix (unit / integration / component / E2E, by epic)

Run all 9 of the template's Quality Gates before finalizing — especially: no orphan stories, contract-first, invariant coverage, file-path grounding, agent independence, verification-contract completeness, cross-story dependency coverage.

## Step 6 — Write output

Write to `.feature-design/<slug>/09-epics.md` with this header:

```markdown
# Implementation Breakdown: <feature_name>

**Date:** <ISO date>
**Source:** Template 14 — SPEC_TO_EPICS_PROMPT_TEMPLATE.md
**Session:** <slug>
**Spec source:** 08-spec.md
**Repo:** <repo>@<branch>

---

<breakdown body — sections 1 through 10 as applicable>
```

## Step 7 — Update state

- Append `"epics"` to `completed_stages`
- Set `current_stage` to `"pe-setup"`

## Step 8 — Optional: push to project tracker

If `state.connectors.project_tracker.enabled === true`, offer to push the breakdown to the user's project tracker. Use the `~~project tracker` placeholder — at runtime this resolves to whichever tracker MCP the user has authenticated (Linear is the primary case, but Asana / Atlassian / Monday / ClickUp also work).

Confirm with the user before pushing — this creates external state that's not easy to roll back. Ask:

> Push these <N> epics and <M> stories to <tracker>? This will create issues in your tracker.

If yes:

1. For each **epic**: create a parent issue in the project tracker. Title = epic name. Description = epic boundary statement + brief summary of stories under this epic.
2. For each **story**: create a child issue under its epic. Title = story title (e.g. "E1-S1: Add user-creation endpoint"). Description = the story's implementation instruction + acceptance criteria as a markdown list. Add `cross-story-dependencies` content to description if present.
3. **Labels**: tag each story issue with each invariant (`INV-1`, `INV-2`, etc.) it must preserve, so invariant coverage is queryable in the tracker.
4. **Capture the issue IDs**: as each issue is created, record its tracker ID (e.g. Linear's `ABC-123`) in `state.connectors.project_tracker.linked_issues` keyed by the epic/story identifier:

```json
"linked_issues": {
  "E1": "ABC-100",
  "E1-S1": "ABC-101",
  "E1-S2": "ABC-102",
  "E2": "ABC-103",
  "E2-S1": "ABC-104"
}
```

5. Save state.json after the push completes.

If the user declines the push, skip this step entirely — `09-epics.md` remains the canonical breakdown and nothing else changes.

If `state.connectors.project_tracker.enabled === false`, skip this step silently — don't ask. The user opted out at `/fd:start` time, and prompting them now would be noise.

## Step 9 — Report

```
Implementation breakdown complete → 09-epics.md
Epics: <count>
Stories: <count>
Invariants: <count>
Critical path: <chain>
Phases: <count>
<if pushed:> Tracker issues created: <count> (visible via /fd:status)

Recommended next: /fd:next  (will run /fd:pe-setup)
```
