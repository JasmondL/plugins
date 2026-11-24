---
name: swarm
description: "Fan out N parallel workers, drain them, and return one report. Use for /swarm, 'swarm this', or parallel coverage, races, gauntlets, and exploration."
disable-model-invocation: true
---

# Swarm

In Hermes, read the Hermes adapter once per session before you follow this skill. It maps the Cursor names in pstack skills to Hermes. Read it with `read_file` at `../../docs/harness/claude-code.md` (relative to this skill's directory); `skill_view` cannot serve files outside the skill directory.

## Start

Open a todolist with one entry per phase before launching anything.

1. Frame
2. Fan out
3. Aggregate
4. Report

## Phase A: Frame

1. State the done predicate and the artifact or report the swarm must return.
2. Choose the shape. Partition into slices, race N workers on identical briefs, or mix both. For a race or mixed shape, declare `first pass`, `rank all`, or `best-of` before spawning.
3. Pick the worker model from the `swarm workers` line in `~/.hermes/SOUL.md` (advisory: which family to pin). Every `delegate_task` child inherits the session model, so pin `delegation.model`/`delegation.provider`/`delegation.reasoning_effort` in `~/.hermes/config.yaml` to change it (drop `-fast`). For a model race, one fan-out can't mix models per arm: name each arm's intended model up front and re-pin between arms, or note the arms share one inherited model. Pick each subagent's reasoning level for the job, per the Subagents section of the **poteto-mode** skill.
4. Give each worker its own writable output when it writes. When workers verify or measure commits, each brief names the exact SHAs. A measurement brief also names the method (sample count, what one sample is, order). The worker records both in its result.

## Phase B: Fan out

Spawn all N workers as one `tasks` array in a single `delegate_task` call (each entry one worker; extras run in parallel, background by default). Children inherit the session's tools and run locally; for worktree isolation set `delegation.worktree_isolation: true`.

When a worker must start from a non-default branch, name the branch in its brief (Hermes has no `cloud_base_branch` field).

Every brief stands alone. Include the goal, scope, exact slice or race arm, how to verify, and what to report. Reports use `PASS`, `ISSUES`, or `BLOCKED` with evidence. A worker that can prove a defect reports `ISSUES` and lists every issue it can prove, not only the first.

If a worker drops out, proceed with N-1 and note it.

## Phase C: Aggregate

Read the terminal results. Drop a result that does not record the SHAs and method its brief names, and respawn that worker once. After a second miss, record a gap. A gap does not count as a pass. For coverage, every required slice needs a result. For a race, apply the selection rule declared up front. Use first pass, rank all, or best-of. Do not paste raw worker dumps.

## Phase D: Report

Return one consolidated in-chat report with a result table, issue one-liners, gaps or dropouts, and the race rule when used.
