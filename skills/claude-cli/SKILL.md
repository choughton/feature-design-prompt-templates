---
name: claude-cli
description: Invoke Claude Code CLI from inside a Cowork session to get an independent Claude response, separate from the moderator's session but with full access to the project's files. Trigger when the user asks for a "fresh Claude opinion", "second Claude session", "independent Claude review", or when running a multi-model crossfire that needs a Claude-side perspective. Spawns a separate `claude` process via the Bash tool from the project's working directory so it can read referenced files, captures the response, and returns it.
---

# Claude Code CLI Runner — Independent Claude Session

Invokes Claude Code CLI in non-interactive mode to spawn an **independent** Claude session, separate from the parent (moderator) Cowork session but with **full access to the project's filesystem** so it can read any files referenced in the prompt.

This matters because crossfire reviewers don't just respond to inlined prompt text — they read project files (PRD, Codex, Design Philosophy, the problem statement file itself, the existing codebase) the way a human reviewer would. Without filesystem access, crossfire reduces to pattern-matching on whatever was inlined.

## When to use

- A crossfire-style multi-model review needs a Claude perspective that doesn't share context with the moderator
- The user explicitly asks for "a second Claude opinion", "fresh Claude take", "independent Claude review"
- Sanity-checking the moderator's own reasoning by running the same question through an isolated Claude process

Do NOT trigger when:
- The user just wants the *current* Claude's opinion — answer directly, don't spawn a subprocess
- Claude Code CLI isn't installed (see Setup)

## Setup the user needs

1. **Claude Code CLI installed** and on PATH. On Windows: `winget install Anthropic.ClaudeCode` (recommended) or `npm install -g @anthropic-ai/claude-code` (Node 20+).
2. **Auth configured** — pick one:
   - **OAuth (Pro/Max subscription) — recommended**: Run `claude auth login` once interactively, OAuth completes, credentials cached in `~/.claude/`. Subsequent CLI calls use the subscription, no API key needed.
   - **API key**: `ANTHROPIC_API_KEY` env var. Required if using `--bare` (which we don't, by design — see below).
3. **Git for Windows installed** (Windows only) so the Bash tool inside the spawned Claude works. Without it, the spawned Claude's shell tool falls back to PowerShell, which sometimes breaks downstream tooling.

## Why we don't use `--bare` and don't isolate the cwd

`--bare` is Anthropic's documented "fully reproducible / scripted" mode that skips auto-discovery of CLAUDE.md, hooks, skills, plugins, MCP servers, and OAuth keychain. **It also requires `ANTHROPIC_API_KEY`** — the env-var-only auth path. Most users prefer to stay on their Pro/Max subscription rather than pay per-token, so this skill drops `--bare`.

The skill also runs from the **project working directory** (no `cd /tmp` isolation). This is intentional: a crossfire reviewer needs to read the project's PRD, Codex, Design Philosophy, the problem statement file, and any code referenced in the prompt. Isolating from the cwd removes that capability and reduces crossfire to "respond to whatever the moderator inlined" — which defeats the point.

What we DO want to avoid:
- The moderator's *conversation* leaking into the spawned Claude. This doesn't happen automatically — `claude -p` does not auto-resume the parent's session. So a bare `claude -p` from the project dir is already independent in the way that matters.

What we accept:
- The project's CLAUDE.md, project skills, and user-global `~/.claude/` config are loaded by the spawned Claude. For crossfire this is appropriate context, not contamination — design principles in CLAUDE.md are the kind of thing a reviewer should know.

If you specifically need a *guaranteed-bare* run (no global config, no project context), set `ANTHROPIC_API_KEY` and add `--bare` to the invocation. That's a deliberate opt-in for cases where you want pure prompt-in / text-out with zero environment.

## How to invoke

### Step 1 — Check that Claude Code CLI is available

```bash
command -v claude && claude --version
```

If unavailable, tell the user:

> Claude Code CLI doesn't appear to be installed or on PATH. Install with `winget install Anthropic.ClaudeCode` (Windows recommended) or `npm install -g @anthropic-ai/claude-code`, then run `claude auth login` to authenticate via OAuth, then retry.

Stop.

### Step 2 — Construct the prompt

Assemble the prompt for the fresh Claude. Critical: **do not include any reference to the parent session's context** unless explicitly intended. The whole point of this skill is independence. If the parent flow has source documents that should be available to the fresh Claude, include them inline in the prompt or write them to a temp file the fresh Claude reads.

For long prompts (anything over a few KB), use a file. Stdin to `claude -p` is capped at 10MB.

### Step 3 — Run Claude from the project directory

```bash
claude -p "$PROMPT" \
  --permission-mode default \
  --allowed-tools "Read,Glob,Grep,WebFetch" \
  --max-turns 8 \
  --no-session-persistence \
  --output-format text \
  2>/dev/null
```

Run from the project working directory (the cwd where the user is running the plugin — typically the repo root). The spawned Claude needs to read files referenced in the prompt; it cannot do that from a temp directory.

Flag rationale:
- `-p "$PROMPT"` — non-interactive print mode. Does not auto-resume the parent session.
- `--permission-mode default` — prompts before risky actions but, in practice, the allowed-tools list is read-only so nothing risky comes up. Don't use `dontAsk` here — it would deny the read calls we explicitly want.
- `--allowed-tools "Read,Glob,Grep,WebFetch"` — read-only toolset. The reviewer can read project files, search them, and fetch external references, but cannot edit, write, or shell out. This is the right shape for adversarial review.
- `--max-turns 8` — generous turn cap so the reviewer can do multi-step file exploration before producing the final response. Adjust upward for complex reviews; the cap exists to prevent runaway loops, not to constrain legitimate work.
- `--no-session-persistence` — don't write this session to disk. Keeps your `/resume` history clean.
- `--output-format text` — plain text on stdout.
- `2>/dev/null` — suppresses spinners and progress indicators.

For long prompts, write to a file in the session folder and pass via stdin or `$(cat ...)`:

```bash
claude -p "$(cat /path/to/prompt.md)" \
  --permission-mode default \
  --allowed-tools "Read,Glob,Grep,WebFetch" \
  --max-turns 8 \
  --no-session-persistence \
  --output-format text \
  2>/dev/null
```

### Step 4 — Capture and return

Capture stdout. Present back with framing that makes the source clear:

> **Independent Claude session says:**
>
> <response>

The framing matters because the user needs to understand this is NOT the moderator (Cowork Claude) talking — it's a separately-spawned Claude with no context inheritance.

## Optional flags worth knowing

- `--model sonnet` or `--model opus` — pin a specific Claude model. Useful when the user wants the fresh Claude on a different model than Cowork is using (e.g., have Opus review what Sonnet produced).
- `--append-system-prompt "<role>"` — set a reviewer persona. For crossfire prompts that already establish role internally, leave this off; for ad-hoc reviews, can be useful.
- `--max-budget-usd 1.00` — cost guard. Adds a hard cap on the spawned session's cost.
- `--continue` / `-c` or `--resume` / `-r` — DO NOT USE for fresh-context calls. These resume an existing session and defeat the purpose of this skill.

## Error handling

| Symptom | Likely cause | Action |
|---|---|---|
| `command not found` | Claude Code CLI not installed | Install per Setup |
| Auth error / "no credentials" | `claude auth login` not run, or OAuth expired | Tell user to re-auth |
| Stdin concat issues | Heredoc or other accidental stdin attachment to `-p` | Pass prompt as positional arg, not via redirected stdin |
| Reviewer can't see referenced files | Spawned Claude was run from a different cwd than where files live | Always run from the project root, not a temp dir |
| First call slow | Cold start (model + tools warmup) | Expected; subsequent calls are faster |
| Tries to use tools beyond Read/Glob/Grep | `--allowed-tools` not enforced or wrong values | Verify the flag is passed correctly; the spawned Claude only gets tools you allowlist |

## Recursion safety

Spawning `claude` from inside a Claude (Cowork) session does NOT create shared session state. They're separate processes that share only the host filesystem and env vars. There's no loop or contamination at the process level — the contamination concern is purely about shared config files (mitigated by the temp-dir trick).

## Windows / Cowork sandbox note

Same caveat as the other model-runner skills: Claude Code CLI installed on Windows may not be reachable from Cowork's Linux Bash sandbox. If `command -v claude` fails inside the sandbox:

1. Install Claude Code CLI inside the sandbox (`npm install -g @anthropic-ai/claude-code`).
2. The sandbox is ephemeral, so `claude auth login` won't persist. You have two options:
   - Set `ANTHROPIC_API_KEY` env var (forces you back into the API-key path — defeats the OAuth goal).
   - Find a way to mount or copy `~/.claude/` from the host into the sandbox, or have Cowork itself expose host credentials. This is environment-specific and may not be supported.

This caveat is the most likely failure point on Windows + Cowork. Worth verifying first.
