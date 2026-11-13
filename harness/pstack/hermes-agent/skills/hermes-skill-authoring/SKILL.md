---
name: hermes-skill-authoring
description: Create and format Hermes SKILL.md to the house standard.
version: 0.1.0
author: Hermes
metadata:
  hermes:
    tags: [hermes, skills, authoring, skill-md, frontmatter, learn]
---

# Hermes Skill Authoring

How to create a Hermes skill correctly and format it strictly: the SKILL.md frontmatter contract, the standard section order, where skills live, and the agent-side `skill_manage` actions that enforce the format at write time. This skill covers creation and formatting. It does not cover hub install flows or trust levels. Curator behavior is covered only where it changes what lands on disk: curator writes apply directly and never sit in the pending queue.

Source of record: https://hermes-agent.nousresearch.com/docs/user-guide/features/skills (agentskills.io-compatible). A fuller authoring guide is at /docs/developer-guide/creating-skills; both are fetchable with `web_extract`.

## When to Use

- "create a skill", "write a SKILL.md", "save this as a skill", "/learn ..."
- "format my skill", "skill won't load / won't route / wrong description"
- Adding frontmatter fields: platforms, tags, category, fallback/requires toolsets, config, required env vars
- Grouping skills behind one slash command (skill bundles)
- Asking what the agent should do when it records a workflow itself

## How to Run

Three creation paths, pick by who is authoring:

1. Agent writes it (default in-session path): the agent authors SKILL.md and saves it with the `skill_manage` tool. No shell needed. New skills land in the profile's `~/.hermes/skills/` (or the configured `skills.create_dir`).
2. User hand-writes it: use the fill-in template at `templates/skill-md.md` with `write_file`, then verify with `skill_view(name=...)`.
3. `/learn <source>`: the agent gathers the source (URL, path, pasted notes, "what we just did") and authors a standards-compliant skill. Large sources (books, paper stacks, big docs) become knowledge-base skills: lean SKILL.md plus one `references/` file per chapter, loaded on demand via `skill_view`.

## SKILL.md Format (strict)

Frontmatter, exact keys:

```yaml
---
name: my-skill                    # required; kebab-case directory slug
description: Brief description of what this skill does   # required; <=60 chars
version: 1.0.0
author: Hermes                    # literal value, never from host env
platforms: [macos, linux]         # optional; omit to load everywhere
metadata:
  hermes:
    tags: [python, automation]
category: devops                  # optional top-level key OR under metadata.hermes
fallback_for_toolsets: [web]      # optional conditional-activation fields
requires_toolsets: [terminal]
config:
  - key: my.setting
description: "What this controls"
default: "value"
prompt: "Prompt for setup"
---
```

Body, standard section order (drop a section only when it has no content):

```markdown
# Skill Title

## When to Use        # concrete trigger conditions
## Procedure          # numbered steps, copy-paste-exact commands
## Pitfalls           # known failure modes and fixes
## Verification       # how to confirm it worked
```

Description is the routing surface: the system-prompt index shows only name + description + category, and it is truncated at 60 chars. Write the capability, not the implementation; anything past char 60 never routes. Count characters before saving.

Platform values are exactly `macos` (Darwin), `linux`, `windows`. When set, the skill is hidden from the system prompt, `skills_list()`, and slash commands on incompatible platforms; omitted means all platforms.

Conditional activation (fallback skills):

- `fallback_for_toolsets` / `fallback_for_tools` — hidden when those are available, shown when missing.
- `requires_toolsets` / `requires_tools` — hidden when unavailable, shown when present.
- No conditional fields: always shown. Example: a curl-based search skill with `fallback_for_toolsets: [web]` appears only when the web toolset is off.

Secrets and settings:

- `required_environment_variables: - name: TENOR_API_KEY / prompt: ... / help: ... / required_for: ...` — asked securely only when the skill is loaded in the local CLI; never in messaging chat. Once set, declared env vars pass automatically into `execute_code` and `terminal` sandboxes.
- `metadata.hermes.config` entries are non-secret, stored under `skills.config` in config.yaml; `hermes config migrate` prompts for missing values and resolved values are injected into context on load.

## Directory Structure

```text
~/.hermes/skills/            # single source of truth for this profile
├── <category>/              # e.g. mlops/, devops/
│   └── my-skill/
│       ├── SKILL.md         # required
│       ├── references/      # additional docs
│       ├── templates/       # output formats
│       ├── scripts/         # helper scripts callable from the skill
│       ├── examples/        # referenced example outputs
│       └── assets/          # supplementary files
└── .bundled_manifest        # origin hashes of seeded bundled skills
```

Category directories may be symlinks. On this machine `autonomous-ai-agents`, `devops`, `github`, `mlops`, and `software-development` under `~/.hermes/skills/` link into `/Users/I564554/j/dot/skills/` (a git repo, jlmq-repo). A skill write lands in the link target and adds untracked files to that repo. Check `readlink` before assuming a write stays inside `~/.hermes/skills/`.

Precedence for duplicate names: `project → local (~/.hermes/skills/) → skills.create_dir → external_dirs`; the shadowed copy is logged. Project-local skills live in `<project-root>/.hermes/skills/` or `<project-root>/.agents/skills/` and only load from trusted repos (`hermes skills trust`); every project skill is security-scanned at scan time, and a `dangerous` verdict quarantines it. `skills.create_dir` (config.yaml) redirects where `skill_manage` create writes new skills; existing skills are still modified in place wherever they live.

## skill_manage Actions (agent-side writes)

| Action | Use for | Key params |
|---|---|---|
| `create` | New skill from scratch | `name`, `content` (full SKILL.md), optional `category` |
| `patch` | Targeted fixes (preferred — only changed text in the call) | `name`, `old_string`, `new_string` |
| `patch` with `content` | Major structural rewrite (replaces whole SKILL.md; `edit` is the legacy alias) | `name`, `content` |
| `delete` | Remove a skill | `name` |
| `write_file` | Add/update supporting files | `name`, `file_path`, `file_content` |
| `remove_file` | Remove a supporting file | `name`, `file_path` |

Each action has its own text slot; an op carrying another action's slot is rejected before any op in the batch applies. An advisory linter runs on create, SKILL.md patches, and references writes; the shape rules `incident-log-shape`, `references-sprawl` (>60 reference files), and `oversized-body` (SKILL.md past ~24k chars) warn but never block.

Content standard (what the linter and review enforce): lessons, not logs — a pitfall is a generalizable rule plus one clause of why, stated once. No PR/issue numbers, dates, or quoted chat. Always-on rules live in SKILL.md; `references/` holds a small set of topic-named files extended in place, not one file per session. Skills never restate what is already loaded every turn (AGENTS.md, tool schemas).

## Write Approval and Locking

- `skills.write_approval: true` (config.yaml) stages every skill_manage write under `~/.hermes/pending/skills/` instead of committing. Review with `/skills pending`, `/skills diff <id>`, `/skills approve <id>` (or `all`), `/skills reject <id>`; toggle with `/skills approval on|off`. Every `skill_manage` write stages this way, in foreground or background-review turns alike. The curator subsystem is separate: it applies its own writes directly, logs them in `~/.hermes/skills/.curator_ledger.jsonl`, and never produces a pending entry.
- `hermes curator lock <skill>` locks one skill: still readable and auto-loadable, but `skill_manage`, `write_file`, `patch`, and `terminal` refuse to change it. Your own edits still work.
- `skills.guard_agent_created` is a content scanner, not an approval gate; the two are independent.

## Bundles (optional)

Bundles group skills under one slash command. YAML at `~/.hermes/skill-bundles/<slug>.yaml` with fields `name`, `description`, `skills` (required, non-empty), `instruction` (optional, prepended). Manage: `hermes bundles list|show|create|delete|reload`; in chat, `/bundles`. A bundle whose slug collides with a skill wins. Missing listed skills are skipped, not fatal. Bundles do not install skills — the skills must already exist.

## Pitfalls

- Description over 60 chars: the index truncates, so a long sentence silently stops routing past char 60. Count it.
- `platforms` with any value other than `macos`/`linux`/`windows`: the skill is silently hidden on your own machine.
- Same skill name in two scanned directories: the higher-precedence directory wins and the shadowed copy is only logged — no error surfaces at use time.
- Ambiguous bare name (two different skills, same name, one directory): loading fails with `Ambiguous skill name <n>: use one of a/one, b/two`; use the relative path.
- Copy-pasting a bundled skill back to "undo my edits": the manifest still holds the old origin hash, so sync keeps flagging it user-modified. Escape hatch: `hermes skills reset <name> --restore`.
- Editing skills by hand in a project repo: untrusted repos never load project skills; trust is repo-level (`hermes skills trust`), and rescans happen on content change (hash cache in `~/.hermes/cache/project_skill_scans/`).
- Expecting messaging platforms to prompt for `required_environment_variables`: they never ask for secrets in chat; use `hermes setup` or `~/.hermes/.env` locally.
- A new skill is invisible in an already-open conversation until the next user message (one note per change); the system prompt itself updates at next compaction.
- Expecting a curator-created skill in `/skills pending`: it is not there. Curator writes apply directly and leave only a `.curator_ledger.jsonl` entry. Verify new skills on disk, not in the pending queue.

## Verification

1. `skill_view(name="<name>")` returns the full content with parsed metadata, `readiness_status: available`, and no `setup_needed`.
2. `hermes skills check` (terminal) passes the format scan.
3. The slash command works: send `/<skill-name>` with a trivial instruction and confirm the skill loaded.
4. With `write_approval: true`, the write appears in `/skills pending` with a correct diff before approval.

## Publishing (optional)

A tap is any GitHub repo with a `skills/` directory of skill folders (directory name = install slug; each needs a standard SKILL.md; `references/`, `templates/`, `scripts/`, `assets/` are downloaded at install; names starting with `.` or `_` are ignored). Recipients: `hermes skills tap add owner/repo`, then `hermes skills install owner/repo/<skill>`. Non-default paths edit the tap entry in `~/.hermes/skills/.hub/taps.json`. Single-skill sharing without a tap: `hermes skills install owner/repo/skills/my-workflow`.
