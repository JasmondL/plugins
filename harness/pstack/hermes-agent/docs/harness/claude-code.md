# pstack in Hermes

pstack skills use Cursor names; map each to Hermes. `<pstack>` = the folder with `skills/` and `docs/`. No row for a field: drop it and say so.

## Tools

| Cursor | Hermes |
| --- | --- |
| `Task` | `delegate_task`; one `tasks` entry per subagent, extras parallel |
| `AskQuestion` | `clarify`; up to 5 questions, optional `choices`/`multi_select`; absent in one-shots, ask plainly |
| Open a todolist | `todo_list` if offered, else in reply |
| Run another pstack skill | `skill_view(name="pstack:<name>")` if plugin enabled, else `read_file` `<pstack>/skills/<name>/SKILL.md`; user types `/pstack:<name>` |
| `Grep`, `Glob` | `search_files`, `target: "content"` (regex) or `target: "files"` (glob) |

## Subagent fields

`delegate_task` children inherit the session's toolsets; unlisted fields are omitted.

| Cursor | Hermes |
| --- | --- |
| `subagent_type: generalPurpose` | omit |
| `poteto-agent`, `comment-sicko` | no named types; instructions in `goal`/`context` |
| `readonly: true` | no flag; brief says "do not edit files", check `git status` after |
| `environment: "cloud"` | no per-task field; runs locally unless `delegation.worktree_isolation: true`; name branch/commit in brief |
| `resume` | `steer` reaches only live children; spawn a new task with result in `context` |
| `file_attachments` | paths in `context`; images in `images` (max 8) |

## Models

`delegate_task` has no model/effort field; the child inherits the session model. To pin subagents set `delegation.model`, `delegation.provider`, `delegation.reasoning_effort` in `~/.hermes/config.yaml`.

| Cursor | Hermes |
| --- | --- |
| a `claude-…`, `grok-…`, `gpt-…`, other slug, or `auto`/`inherit-parent` | pin via `delegation.model`/`reasoning_effort` (or omit); drop `-fast`, say so |
| the `pstack-models.mdc` rule | no auto-loaded rules dir; homes are `~/.hermes/SOUL.md` (global), `HERMES.md`/`.hermes.md` (repo root); role lines guidance; missing file/line: skill default slug |
| a role value `<model> <effort>` e.g. `opus high` | model `model.default`, effort `agent.reasoning_effort`/`delegation.reasoning_effort`; say so |

## Paths

| Cursor | Hermes |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.hermes/skills/` (or `.agents/skills/`) at git root, trusted with `hermes skills trust`; `~/.hermes/skills/` for user skills |
| `.cursor/rules/*.mdc` | `HERMES.md`/`.hermes.md` at repo root, then the `AGENTS.md` chain, then `CLAUDE.md`; first non-empty wins; reads `.mdc` only from cwd; model rule excepted |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | local `<pstack>/<path>`; never fetch |

## Cursor-only features

Hermes has no Cursor Automations, Bugbot, cloud agents, or built-in PR tool. When a step needs one, write one line, `Skipped in Hermes: <step>. <reason>.`, then go on; no stop/ask/retry. Bugbot PR comments are ordinary review threads; a PR tool means the step's forge command, e.g. `gh pr create` (not a skip).
