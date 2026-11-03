# pstack in Claude Code and Hermes

pstack skills use Cursor names. In Claude Code or Hermes, read this file once per session, then map each Cursor name in a skill with the tables below, using the column for your harness. `<pstack>` means the folder that holds `skills/` and `docs/`. If a Cursor field has no row, drop it and say so. Never pass a field or value the tool schema rejects.

## Tools

| Cursor | Claude Code | Hermes |
| --- | --- | --- |
| `Task` | `Agent` | `delegate_task`. One entry in the `tasks` array spawns one subagent; extra entries run in parallel. |
| `AskQuestion` | `AskUserQuestion`. In `-p` mode it is absent, so ask in plain text. | `clarify`: an array of up to 5 `questions`, each with optional `choices` and `multi_select`. It is disabled in one-shot runs (`hermes -z`, `hermes chat -q`), so ask in plain text. |
| Open a todolist | Use `TodoWrite` if the session offers it. Otherwise keep the list in your reply and update it at each phase. | Use `todo_list` if the session offers it. Otherwise keep the list in your reply and update it at each phase. |
| Run another pstack skill (`/name` or "the **name** skill") | `Read` `<pstack>/skills/<name>/SKILL.md` and follow it. The `Skill` tool refuses most pstack skills. When you tell the user to type it, write `/pstack:<name>`. | `skill_view(name="pstack:<name>")` when the pstack plugin is enabled; it loads the `SKILL.md`. Otherwise `read_file` `<pstack>/skills/<name>/SKILL.md` and follow it. When you tell the user to type it, write `/pstack:<name>`. |
| `Grep`, `Glob` | `Bash` with `rg` when the session has no `Grep` or `Glob`. | One tool: `search_files` with `target: "content"` (ripgrep regex) or `target: "files"` (filename glob). |

## Subagent fields

| Cursor | Claude Code | Hermes |
| --- | --- | --- |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` | Omit. `delegate_task` has no subagent type; the child inherits the session's toolsets. |
| `poteto-agent`, `comment-sicko` | `pstack:poteto-agent`, `pstack:comment-sicko` | No named subagent types. Put the agent's instructions in the task's `goal` and `context`, and spawn with `delegate_task`. |
| `readonly: true` | `subagent_type: "Explore"`. It keeps `Bash`, so the brief also says "do not edit files", and you check `git status` afterward. | There is no read-only flag, and the child keeps the session's toolsets, including `terminal`. Say "do not edit files" in the brief, and check `git status` afterward. |
| `readonly: false` | `general-purpose`. It keeps the session's MCP servers. | Omit. The child inherits the session's toolsets, including their MCP servers. |
| `environment: "cloud"` | `isolation: "worktree"`. Name the branch or commit the worker checks out. | No per-task field. Children run locally in the shared workspace unless `delegation.worktree_isolation: true` in `~/.hermes/config.yaml`, which gives each child a worktree on `hermes-subagent/<id>`. Name the branch or commit in the brief. |
| `environment: "local"` | Omit. | Omit. |
| `cloud_base_branch: <b>` | Tell the worker to check out `<b>` first. | Same as Claude Code: tell the worker to check out `<b>` first. |
| `run_in_background: true` | Omit. Subagents run in the background by default and notify you when they finish. | Omit. Top-level subagents always run in the background, and their results arrive as messages. |
| `resume` | Spawn a new agent and paste the earlier result into its brief. | Same: `action: "steer"` reaches only live children, so spawn a new task with the earlier result in `context`. |
| `file_attachments` | List the paths in the brief. | List the paths in `context`. Screenshots and other images go in the task's `images` (max 8). |

## Models

| Cursor | Claude Code | Hermes |
| --- | --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`, such as `claude-opus-5-5-xhigh` | `model: "opus"` and `effort: "xhigh"`. `model` takes only `opus`, `sonnet`, `haiku`, `fable`. Drop `-fast`. | `delegate_task` has no model or effort field; the child inherits the session's model. To pin subagents, set `delegation.model`, `delegation.provider`, and `delegation.reasoning_effort` (values: `minimal`, `low`, `medium`, `high`, `xhigh`, `max`, `ultra`) in `~/.hermes/config.yaml`. Drop `-fast`, and say what you did. |
| A `grok-*`, `gpt-*`, or other non-Claude slug | Omit `model` so the agent runs on the session model, and say so. | Same as Claude Code: omit; the child runs on the session model. |
| `auto` or `inherit-parent` | Omit `model`. | Same as Claude Code: omit; the child inherits the session's model. |
| The `pstack-models.mdc` rule, `~/.cursor/rules/pstack-models.mdc` | `~/.claude/rules/pstack-models.md`. Claude Code loads it into every new session, so its role lines are already in context. If they are not, read the file. If the file or the role's line is missing, use the skill's default slug. | There is no auto-loaded rules directory. The always-on homes are `~/.hermes/SOUL.md` (global, every session) and `HERMES.md` or `.hermes.md` at the repo root (project). The role lines there are guidance only, because model and effort come from `~/.hermes/config.yaml`, not the invocation. If the file or the role's line is missing, use the skill's default slug. |
| A role value `<model> <effort>`, such as `opus high` | `model: "opus"` and `effort: "high"`. | No per-invocation fields: model is `model.default` in `~/.hermes/config.yaml` (`delegation.model` for subagents), and effort is `agent.reasoning_effort` or `delegation.reasoning_effort`. Say so. |

## setup-pstack

Claude Code loads every `.md` file in `~/.claude/rules/` into each new session, so `setup-pstack` writes its rule there. Follow its steps with these changes:

- Detect. The detected models are the `model` values that the `Agent` tool schema lists, each with an effort from its `effort` values. Write a real model as `<model> <effort>`, such as `opus xhigh`. The effort is the last token, so the budget step works as written.
- Defaults. Map each default slug with the rows above: `claude-opus-5-5-xhigh` is `opus xhigh`, and `grok-4.7-xhigh-fast` is `inherit-parent`.
- Write. Write `~/.claude/rules/pstack-models.md` with no frontmatter, then the comment lines, the `# budget` line, and one line per role from step 5. Overwrite the whole file. Never put the role lines in `CLAUDE.md` or `AGENTS.md`.

## Paths

| Cursor | Claude Code | Hermes |
| --- | --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.claude/skills/`, `~/.claude/skills/` | `.hermes/skills/` (or `.agents/skills/`) at the git root for project skills, trusted with `hermes skills trust`, and `~/.hermes/skills/` for user skills. |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `~/.claude/plugins/`, `.claude-plugin/plugin.json` | `~/.hermes/plugins/<name>/`, with a `plugin.yaml` manifest; the portable `plugin.json` that pstack ships is accepted as-is. Discovery ignores `.cursor-plugin/`. |
| `.cursor/rules/*.mdc`, always-applied rules | `CLAUDE.md`, or `AGENTS.md` when no `CLAUDE.md` exists. Claude Code never reads `.mdc` files. The model rule is the exception. See Models. | `HERMES.md` or `.hermes.md` at the repo root, then the `AGENTS.md` chain, then `CLAUDE.md` in cwd; the first non-empty wins. Hermes does read `.cursor/rules/*.mdc`, but only from cwd, never global. The model rule is the exception. See Models. |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | `<pstack>/<path>` when that file exists. The plugin ships the skills, `docs/guide/`, and `README.md`. Read the local copy and never fetch it. Give the user the URL when they need a link. | Same as Claude Code: read the local `<pstack>/<path>`, never fetch it. |
| Transcripts in `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.claude/projects/<slug>/<session>.jsonl`. The slug is the workspace path with each `/` and `.` changed to `-`, so `/Users/you/proj` is `-Users-you-proj`. The newest `.jsonl` directly in that folder is the current session. Its `<session>/subagents/` folder holds subagent transcripts. Read only this workspace's folder. | There are no transcript files: sessions live in SQLite `~/.hermes/state.db` (`sessions` and `messages`; subagent rows have `source='subagent'` and a `parent_session_id`). Use the `session_search` tool, or `hermes sessions export --format jsonl` in `terminal`. `~/.hermes/sessions/` holds request-dump breadcrumbs only. |

## Cursor-only features

Claude Code and Hermes have no Cursor Automations, Bugbot, Cursor cloud agents, or built-in PR tool. When a step needs one, put one line in your reply, `Skipped in Claude Code: <step>. <reason>.` (in Hermes, `Skipped in Hermes: <step>. <reason>.`), then go on to the next step. Do not stop, ask, or retry. When a later step needs what the skipped step made, such as a URL or a key, leave a placeholder and name it in the line.

| Cursor | Claude Code | Hermes |
| --- | --- | --- |
| Cursor Automations: routines, `update_state` routine calls, `api2.cursor.sh/automations` webhooks, routine wakes, and `SendToUser` secret requests | Skip with the line. | Skip with the line. Local `cronjob_manage` jobs and `hermes webhook subscribe` are not equivalents: no `update_state`, no Cursor webhook, no `SendToUser`. Offer a cron job only if the user asks. |
| Bugbot | Bugbot comments on a GitHub PR are ordinary review threads, so triage them with the forge command that the step gives. Skip a step that starts, waits for, or configures Bugbot when the PR has no Bugbot check or comment. | Same as Claude Code. |
| Cursor cloud agents | `environment: "cloud"` maps to a worktree, as in Subagent fields. Skip a step that needs the cloud agent itself, such as its dashboard status or its own machine. | There are no cloud agents; subagents run locally, as in Subagent fields. Skip a step that needs the cloud agent itself. |
| A built-in PR tool | Use the forge command that the step gives, such as `gh pr create`. That is not a skip, so write no line. | Same as Claude Code. |
