# Connectors

This plugin uses MCP connectors to integrate with project trackers, code repos, and supplementary doc stores. Following Anthropic's productivity-plugin pattern, plugin commands reference connectors by **category placeholder** rather than specific product. Claude resolves the placeholder at runtime to whichever MCP server you've authenticated.

Authenticate connectors via `/mcp` after the plugin is enabled. You only need to authenticate the ones you actually use — the others stay dormant.

## How tool references work

Plugin command files use `~~category` as a placeholder for whatever tool the user connects in that category. For example, `~~project tracker` resolves to Linear, Asana, Atlassian (Jira), Monday, or ClickUp — whichever one you have authenticated.

If multiple options in a category are authenticated, Claude picks based on context (e.g., active session metadata, the user's preference stated in conversation).

## Categories used by this plugin

| Category | Placeholder | Bundled options | Used by |
|---|---|---|---|
| Project tracker | `~~project tracker` | Linear (primary), Asana, Atlassian, Monday, ClickUp | `/fd:epics`, `/fd:status` |
| Code repo | `~~code repo` | GitHub (via Cowork's built-in GitHub connector — not bundled in this plugin's `.mcp.json`) | `/fd:epics` (file-path grounding), `/fd:start` (repo metadata) |
| Knowledge base | `~~knowledge base` | Notion | Optional — `/fd:start`, `/fd:problem`, `/fd:explore` for supplementary docs (Confluence/Coda also work if installed separately) |
| Drive / file store | `~~drive` | Google Drive (user-installed; not bundled) | Optional — `/fd:explore`, `/fd:problem` for customer feedback, samples, supplementary materials |

`~~drive` is a custom placeholder defined for this plugin specifically. Anthropic's official taxonomy doesn't include a Drive category — `~~knowledge base` covers wiki-style repositories (Notion, Confluence) but not file storage. We define `~~drive` here so commands can pull from your Drive without conflating it with wiki-style sources.

## What this plugin does NOT bundle

- **Google Drive MCP** — there's no canonical Anthropic-hosted Drive MCP server. The community one (`@modelcontextprotocol/server-gdrive`) is in the archived servers repo. If you have a Drive connector configured in Cowork (or installed manually), the `~~drive` placeholder will resolve to it. If not, commands fall back to local-files-only behavior.
- **GitHub MCP** — Cowork ships a built-in GitHub connector that handles GitHub access more cleanly than a plugin-declared MCP would. We don't redeclare it in this plugin's `.mcp.json` to avoid duplicate registration. The `~~code repo` placeholder resolves to whatever GitHub setup you have configured in Cowork.

## Per-command behavior

### `/fd:start`

Asks for:
- The local `/docs` folder path (e.g., `./docs/`) — **canonical project docs live here as markdown**
- Optional individual file paths for PRD, Codex, Design Philosophy if you have them — leave blank if not
- Optional Linear team/project URL — only needed if you want `/fd:epics` to push automatically
- Optional `~~drive` folder ID — only needed if you want commands to pull supplementary materials (customer feedback, samples)

PRD specifically is **never required** — many users don't write PRDs.

### `/fd:explore`, `/fd:triage`, `/fd:problem`

Reads canonical project docs from `state.project_docs.docs_dir`. If a `~~drive` folder is configured, optionally pulls supplementary context (customer feedback, samples) when the conversation requires it.

### `/fd:ui-draft`

Reads from `~~knowledge base` if existing UI patterns / design system docs live there. Otherwise reads from the local `/docs` folder.

### `/fd:epics`

Generates the implementation breakdown locally as `09-epics.md`. After completion, offers to push the epics and stories to `~~project tracker`:
- Epic → parent issue
- Stories → child issues, with acceptance criteria in description and invariants as labels
- Linear issue IDs are saved back into `state.json` for `/fd:status` to track

The push is opt-in. Skipping it doesn't break anything — `09-epics.md` stays as the canonical breakdown.

### `/fd:status`

Pipeline status from `state.json` is always shown. If `~~project tracker` is connected and stories have linked issue IDs, additionally shows live open/closed/in-progress state for each linked story.

## What about `~~chat`, `~~email`, `~~calendar`, `~~office suite`?

This plugin doesn't currently use those — they're productivity-plugin categories, not feature-design ones. If a future fd command needs them (e.g., scanning Slack for feature requests during triage), this doc and the command's `allowed-tools` will be updated.
