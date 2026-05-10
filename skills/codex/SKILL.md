---
name: codex
description: Invoke OpenAI Codex CLI from inside a Claude session to get an independent response from a non-Claude model. Trigger when the user asks to "ask Codex", "get Codex's take", "have Codex review", "second opinion from Codex", "run this past Codex", or when running a multi-model crossfire that needs an independent OpenAI-side perspective. Sends a prompt to Codex via `codex exec` and returns the plain-text response. Used by the feature-design plugin's crossfire stages but also independently useful for any second-opinion task.
---

# Codex CLI Runner

Invokes OpenAI's Codex CLI in non-interactive mode to get an independent response from a non-Claude model. The skill shells out via the Bash tool, captures stdout, and returns the response cleanly.

## When to use

- The user explicitly asks to consult Codex ("ask Codex", "what does Codex think", "second opinion from Codex")
- A crossfire-style multi-model review needs an independent OpenAI-side perspective
- The user wants an external check on a design, code change, or argument and Codex is the chosen reviewer

Do NOT trigger when:
- The user is asking about Codex CLI itself (use docs, not this skill)
- The task can be answered from your own knowledge — don't pay the latency/cost for nothing
- Codex CLI isn't installed (see "Setup check" below)

## Setup the user needs

Before this skill works, the user must have:

1. **Codex CLI installed** and on PATH. Install with `npm install -g @openai/codex` (Node 20+) or via the official installer.
2. **Auth configured** — either:
   - `codex login` has been run (creates `~/.codex/auth.json`), OR
   - `CODEX_API_KEY` env var is set (note: not `OPENAI_API_KEY` — the `exec` subcommand specifically reads `CODEX_API_KEY`)

If the user hasn't done this, the skill should tell them and stop, not try to proceed.

## How to invoke

### Step 1 — Check that Codex is available

Run a quick sanity check via Bash:

```bash
command -v codex && codex --version
```

If the command fails or returns nothing, tell the user:

> Codex CLI doesn't appear to be installed or on PATH. Install with `npm install -g @openai/codex` (Node 20+ required), then run `codex login` to authenticate, then retry.

Stop. Do not attempt the call.

### Step 2 — Construct the prompt

Decide what prompt to send Codex. For most uses this comes from:
- The user's request (if they asked for Codex's take on a specific question)
- A file path the user provided
- A prompt the parent flow assembled (e.g. a crossfire prompt + source document, in the future feature-design crossfire commands)

If the prompt is long, write it to a temp file rather than passing it inline — long shell-arg strings have quoting hazards.

### Step 3 — Run Codex from the project directory

```bash
codex exec --skip-git-repo-check --sandbox read-only "$PROMPT" 2>/dev/null
```

Run from the project's working directory (typically the repo root) so Codex can read files referenced in the prompt. Crossfire reviewers don't just respond to inlined text — they read the project's PRD, Codex docs, code, etc. via the CLI's filesystem tools.

Flag rationale:
- `exec` — non-interactive subcommand; required to avoid landing in the Codex REPL
- `--skip-git-repo-check` — Codex refuses to run outside a git repo by default; this disables the check (handy if the project happens to not be a git repo, though most are)
- `--sandbox read-only` — safest default for review work. Codex can read files; cannot write, edit, or hit network. Right shape for adversarial review.
- `2>/dev/null` — suppresses Codex's progress/thinking-token stream on stderr; the final response goes to stdout

For long prompts, prefer reading from a file:

```bash
codex exec --skip-git-repo-check --sandbox read-only "$(cat /path/to/prompt.md)" 2>/dev/null
```

### Step 4 — Capture and return

Capture stdout. The full response is the model's final message. Present it back to the user with light framing — typically:

> **Codex says:**
>
> <response>

If the response is long, summarize first and offer the full text on request.

## Optional flags worth knowing

- `--ephemeral` — don't persist the session rollout to disk. Use for crossfire so review runs don't pollute the user's session history.
- `-m <model>` — pick a specific model. Default is fine for most uses; specify when the user asks for a particular Codex model.
- `-o /path/last.md` — also write the final response to a file. Useful when the parent flow wants the response saved alongside other crossfire artifacts.
- `--json` — return newline-delimited JSON events instead of plain text. Use only if the parent flow needs structured event output.

## Error handling

| Symptom | Likely cause | Action |
|---|---|---|
| `command not found` | Codex CLI not installed or not on PATH | Tell user to install per Setup section |
| `Authentication failed` / 401 | `codex login` not run, or auth expired, or `CODEX_API_KEY` missing | Tell user to re-auth |
| `outside a git repo` | Missing `--skip-git-repo-check` | Add the flag (this skill always passes it; if you see this, the invocation got mangled) |
| Hangs forever | Codex waiting on TTY input — happens when stdin is an empty pipe | Always pass the prompt as a positional arg, not via redirected stdin. See [openai/codex#20919](https://github.com/openai/codex/issues/20919) |
| Empty stdout, content on stderr | `2>/dev/null` was dropped or the run errored before producing output | Re-run without stderr suppression to see the actual error |

## Windows / Cowork sandbox note

Codex CLI installs to the host OS (typically Windows in Cowork's case). Cowork's Bash tool runs in an isolated Linux sandbox. If the sandbox can't reach the host's `codex` binary, this skill will fail at the Step-1 sanity check. Workarounds in priority order:
1. Install Codex CLI inside the sandbox (`npm install -g @openai/codex` from within Bash) — auth files won't persist between sessions, so re-auth each session via `CODEX_API_KEY` env var.
2. If Cowork exposes a way to invoke Windows binaries from Bash, use that path. Verify by trying `codex --version` from within Bash; if it works, you're set.

This is a known caveat to validate during first-time setup, not a blocker.
