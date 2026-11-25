---
name: arena
description: "Spawn N parallel candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape."
disable-model-invocation: true
---

# Arena

In Hermes, read the Hermes adapter once per session before you follow this skill. It maps the Cursor names in pstack skills to Hermes. Read it with `read_file` at `../../docs/harness/claude-code.md` (relative to this skill's directory); `skill_view` cannot serve files outside the skill directory.

Open a todolist with one entry per phase (Frame, Fan out, Cross-judge, Pick, Graft, Verify) before launching anything.

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric, 3-6 concrete gradeable criteria for what success looks like on *this* task. It is the picker's tool in Phase D. Candidates only see the task.
3. Pick the runners from the `arena runners` line in `~/.hermes/SOUL.md`; that line is advisory guidance for the model family you want per seat. Every `delegate_task` child in this fan-out inherits the session model, so pin `delegation.model`/`delegation.provider`/`delegation.reasoning_effort` in `~/.hermes/config.yaml` to change it for the run (drop the `-fast` suffix). One fan-out cannot mix models per seat, so a genuinely multi-model panel needs re-pinning between seats; otherwise all runners run on the same model and the diversity comes from the distinct briefs. Keep at least two runners, and spawn more for multiple design directions.
4. Give each candidate its own output location (a git worktree where possible, otherwise `/tmp/arena-<slug>/candidate-<n>/`), per the **separate-before-serializing-shared-state** principle skill.

## Phase B: Fan out

Spawn all N subagents as one `tasks` array in a single `delegate_task` call (each entry is one runner; extra entries run in parallel, in the background by default), each with the task, the path to the shared grounding, its own output path, and instructions to produce the artifact and a short rationale naming the alternatives it considered and what it rejected.

If a candidate produces no output, proceed with N-1 and note the dropout.

## Phase C: Cross-judge

After all Phase B candidates complete, spawn one `delegate_task` judge, briefed not to edit files (confirm with `git status` afterward), from the `arena cross-judge pool` line in `~/.hermes/SOUL.md` (advisory: which model family to pin). The judge inherits the session model, or the model pinned in `delegation.model`. For cross-vendor independence, pin a different vendor's model in `delegation.model` before spawning the judge, and say so if you cannot. While you read in Phase D, the judge scores each candidate, by path label, on each rubric criterion and recommends a base.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric criterion by criterion, not on holistic feel, and compare with the cross-judge when Phase C ran one. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or smaller API, per the Laziness Protocol.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base, usually one or two things per candidate, not most of it. Fold each graft in by hand, per the **redesign-from-first-principles** principle skill, so the result stays coherent under one mental model. Don't paste mechanically.

If the candidates converge on one shape, that is strong agreement. Note it and ship the consensus shape. No graft is needed. If they wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

Verify the synthesized artifact like any other output, per the **prove-it-works** principle skill. If verification finds a problem the arena missed, either Phase A was wrong (re-frame and re-run) or a candidate caught it and you missed the graft (go back to Phase E). Don't paper over it.

## Outputs

One synthesized artifact, with a short synthesis note alongside naming the base and why, the cross-judge's verdict or that Phase C skipped the judge, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.
