# OpenCode as the coding delegate

When the session mandates coding via the local opencode agent (`opencode_run`), typically through the micro-agent pipeline where each micro agent (a `delegate_task` child) drives one opencode agent for one vertical slice:

- Call `opencode_models` once per session before any spawn; translate each role's slug to a real `provider/model` from that list. opencode has no effort parameter and no grok family: drop `-fast`, and when the role's family is absent map to the closest available family (judgment defaults -> the strongest `anthropic/claude-opus-*`, mechanical slices -> `anthropic/claude-sonnet-*`). Record the mapping in the orchestrator baseline and name it in the final report.
- `opencode_run` can die at the 420s gateway ceiling even with a larger `timeout` passed. Keep each brief to one bounded artifact; after a ceiling hit, check `ps` + partial files before any re-dispatch (per delegate-task-orchestration's opencode section).
- The opencode task text shares NO context with the parent: absolute paths, the exact file contract, constraints repeated inside, and a verify-before-finish checklist (parse + dry-run) go in the task string itself.
- The micro agent owns verification before returning: it runs the slice's self-tests, then the parent re-verifies on disk (self-report is a claim, not evidence).
