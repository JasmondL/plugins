---
name: setup-pstack
description: Configure which model pstack's subagents use and at what reasoning budget. Detects your available models, writes delegation settings into your Hermes config, and records advisory per-role guidance. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

In Hermes, read the Hermes adapter once per session before you follow this skill. It maps the Cursor names in pstack skills to Hermes. Read it with `read_file` at `../../docs/harness/claude-code.md` (relative to this skill's directory); `skill_view` cannot serve files outside the skill directory.

Hermes has one subagent model pin, not a per-role model table. `delegate_task` children inherit the session model unless you pin `delegation.model`, `delegation.provider`, and `delegation.reasoning_effort` in `~/.hermes/config.yaml`. This skill picks one model and budget, writes that pin, and records the per-role guidance the routed skills read as advisory role lines in `~/.hermes/SOUL.md`.

**Be honest about the limit.** One `delegation.model` covers every subagent at once. The per-role and per-seat model diversity the pstack skills describe (arena panels, interrogate reviewers, swarm races) is advisory in Hermes. A fan-out runs all arms on the one pinned model unless you re-pin `delegation.model` between arms. State this when you confirm the config.

## Steps

### 1. Detect available models

Enumerate the models you can pin. `opencode_models` lists the locally installed opencode providers and models when that tool is present. Otherwise read the provider's own listing, or the models already named under `delegation`/`model` in `~/.hermes/config.yaml`. If you cannot detect any, ask the user to paste the model names and providers they have access to. Never write a model you have not confirmed. `auto` and `inherit-parent` are always valid; both mean leave the pin unset so children inherit the session model.

### 2. Load current state

Read `delegation.model`, `delegation.provider`, and `delegation.reasoning_effort` from `~/.hermes/config.yaml` for the current pin. Read the `## pstack roles` block in `~/.hermes/SOUL.md` for the current advisory role lines, if any. Treat those as the current choices. With neither present, start from this skill's defaults: one judgment-grade model at `xhigh` reasoning effort, which the role lines in step 5 record per role.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use `clarify` over free text. Offer these three options with these exact labels, and name the current budget when the config records one. With no config, say that `large` matches the skill defaults.

- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Pick one model.** Choose the single model that becomes `delegation.model` and `delegation.provider`. Pick the strongest judgment-grade model the user has, because it carries the hardest code and all prose and judgment work. If the user wants children to stay on the session model, pick `auto`/`inherit-parent` and leave the pin unset. The budget sets `delegation.reasoning_effort` on the ladder `xhigh` > `high` > `medium` > `low`. `large` writes `xhigh`, `medium` writes `high`, `small` writes `medium`.

**(c) Show and confirm.** Show the model, provider, and effort you will write, plus the advisory role lines from step 5. Say plainly that the one pin covers all subagents and that per-role diversity is advisory. Ask whether to accept or change the model, provider, or budget. Use `clarify` over free text.

### 4. Validate

The chosen model and provider must be ones you confirmed in step 1. `auto`/`inherit-parent` always pass and mean no pin. If the chosen model is not available, stop and ask again.

### 5. Write the config

Write two idempotent blocks. Overwrite each whole block so re-runs stay clean.

**(a) The pin.** Set `delegation.model`, `delegation.provider`, and `delegation.reasoning_effort` in `~/.hermes/config.yaml` to the chosen values. For `auto`/`inherit-parent`, remove those three keys so children inherit the session model.

**(b) The advisory role lines.** Under a `## pstack roles` heading in `~/.hermes/SOUL.md`, write one line per role with the role's target effort for this budget. These are guidance the routed skills (`how`, `why`, `arena`, `swarm`, `architect`, `interrogate`, `reflect`) read when deciding how hard to push a spawn; they do not change the one `delegation.model` pin. Shape:

```
## pstack roles
# One line per role. Advisory in Hermes: one delegation.model covers all subagents.
# Re-pin delegation.model between arms to vary a panel. `inherit-parent`/`auto` = session model.
# budget: large (xhigh)
feature, refactoring: xhigh
bug-fix: xhigh
perf-issue: xhigh
hillclimb: xhigh
judgment and prose: xhigh
arena runners: xhigh (advisory; re-pin per arm to vary model)
arena cross-judge pool: xhigh (advisory; one judge, re-pin to a different model)
swarm workers: high
interrogate reviewers: xhigh (advisory; re-pin per reviewer to vary model)
```

Move each role's effort by the budget: `large` keeps the values above, `medium` lowers each one step, `small` lowers each two steps, floored at `low`.

### 6. Confirm

Tell the user the config was written, that the pin applies to new `delegate_task` children, and that the role lines are advisory. Re-running this skill updates both blocks.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke `/create-formal-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
