---
name: gemini
description: Invoke Google's Gemini CLI from inside a Claude session to get an independent response from a non-Claude model. Trigger when the user asks to "ask Gemini", "get Gemini's take", "have Gemini review", "second opinion from Gemini", "run this past Gemini", or when running a multi-model crossfire that needs an independent Google-side perspective. Sends a prompt to Gemini via `gemini -p` and returns the plain-text response. Used by the feature-design plugin's crossfire stages but also independently useful for any second-opinion task.
---

# Gemini CLI Runner

Invokes Google's Gemini CLI in non-interactive mode to get an independent response from a non-Claude model. The skill shells out via the Bash tool, captures stdout, and returns the response cleanly.

## When to use

- The user explicitly asks to consult Gemini ("ask Gemini", "what does Gemini think", "second opinion from Gemini")
- A crossfire-style multi-model review needs an independent Google-side perspective
- The user wants an external check on a design, code change, or argument and Gemini is the chosen reviewer

Do NOT trigger when:
- The user is asking about Gemini CLI itself (use docs, not this skill)
- The task can be answered from your own knowledge — don't pay the latency for nothing
- Gemini CLI isn't installed (see "Setup check" below)

## Setup the user needs

Before this skill works, the user must have:

1. **Gemini CLI installed** and on PATH. Install with `npm install -g @google/gemini-cli` (Node 20+).
2. **Auth configured** — pick one:
   - **API key (headless-friendly):** `GEMINI_API_KEY` env var, obtained from https://aistudio.google.com/app/apikey. This is the path of least friction for shell-out.
   - **OAuth ("Sign in with Google"):** First run `gemini` interactively, complete the browser flow, then cached credentials live in `~/.gemini/`. Free-tier OAuth has stricter quotas than API key.
   - **Workspace / Vertex:** Set `GOOGLE_APPLICATION_CREDENTIALS` + `GOOGLE_CLOUD_PROJECT` (+ `GOOGLE_CLOUD_LOCATION` for Vertex).

If none is set, `gemini -p` will either drop into an OAuth flow (which won't work in non-interactive context) or error. Detect and tell the user.

## How to invoke

### Step 1 — Check that Gemini is available

```bash
command -v gemini && gemini --version
```

If unavailable, tell the user:

> Gemini CLI doesn't appear to be installed or on PATH. Install with `npm install -g @google/gemini-cli` (Node 20+ required), then either set `GEMINI_API_KEY` or run `gemini` once interactively to complete OAuth, then retry.

Stop. Do not attempt the call.

### Step 2 — Construct the prompt

Same as the Codex skill — assemble the prompt from the user's request, file contents, or a parent-flow-supplied template. For long prompts, use a file rather than inline shell-arg quoting.

### Step 3 — Run Gemini

The minimal invocation:

```bash
gemini -p "$PROMPT" --output-format text -m flash 2>/dev/null
```

Flag rationale:
- `-p` — **mandatory** for non-interactive use. Without it, Gemini drops into the REPL when stdin is a TTY.
- `--output-format text` — plain text on stdout. Use `--output-format json` for `{response, stats, error}` if the parent flow needs structured output.
- `-m flash` — pick the fast/cheap model. For higher-quality reviews use `-m pro` (or whatever the current top-tier model is). Defer to user preference if specified.
- `2>/dev/null` — suppresses Gemini's progress/spinner output.

For long prompts:

```bash
gemini -p "$(cat /path/to/prompt.md)" --output-format text -m flash 2>/dev/null
```

Or pipe stdin:

```bash
cat /path/to/prompt.md | gemini -p "Review the following:" --output-format text -m flash 2>/dev/null
```

### Step 4 — Capture and return

Capture stdout. Present back with framing:

> **Gemini says:**
>
> <response>

## Optional flags worth knowing

- `--approval-mode yolo` (or `--yolo`) — auto-approves any tool calls Gemini wants to make. Only relevant if the prompt encourages tool use; for pure text-in/text-out reviews, prompt phrasing keeps things clean and you don't need this.
- `-m <model>` — pin a specific model when the user requests one.
- `--output-format stream-json` — for streaming JSON events. Use only if the parent flow consumes streamed structured output.

## Context-leak gotchas

Gemini auto-loads two things from the working directory that can contaminate output:

1. **`GEMINI.md`** — analogous to `CLAUDE.md`. If the cwd has one, it gets loaded as context. For deterministic crossfire, run from a clean directory or a temp dir.
2. **`.gemini/.env`** — environment overrides. Loaded from cwd up the directory tree, then `~/.gemini/.env`. Variables can leak from project-specific configs.

If you want a guaranteed-clean Gemini call:

```bash
TEMP_DIR=$(mktemp -d)
cd "$TEMP_DIR"
gemini -p "$PROMPT" --output-format text -m flash 2>/dev/null
cd - >/dev/null
rm -rf "$TEMP_DIR"
```

## Error handling

| Symptom | Likely cause | Action |
|---|---|---|
| `command not found` | Gemini CLI not installed | Install per Setup section |
| Hangs / browser opens | OAuth flow triggered in non-interactive context | Set `GEMINI_API_KEY` instead of relying on OAuth |
| Exit code 42 | Input error (malformed prompt, empty stdin where one was expected) | Check prompt construction |
| Exit code 53 | Turn limit reached | Shouldn't happen on single-turn `-p` calls; if it does, the prompt may have triggered tool-use loops |
| Quota exceeded | Free-tier OAuth quota hit | Switch to `GEMINI_API_KEY` (paid tier) or wait for reset |

## Windows / Cowork sandbox note

Same caveat as the Codex skill: the user's Gemini CLI is likely installed on Windows; Cowork's Bash runs in a Linux sandbox. If the sandbox can't reach the host binary, install Gemini CLI inside the sandbox (`npm install -g @google/gemini-cli`) and pass `GEMINI_API_KEY` as an env var so auth survives the ephemeral sandbox.
