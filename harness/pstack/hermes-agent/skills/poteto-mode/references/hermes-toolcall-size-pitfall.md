# Hermes: large tool_call arguments drop mid-stream

In this Hermes deployment (MoM ingress), a `tool_call` whose argument strings are
large (a long embedded task/spec, several KB) is severed by the gateway 300s
request timeout while the arguments are still streaming. It surfaces as a
deterministic JSON parse error at a fixed character offset (e.g. "Expecting ','
delimiter: line 1 column 7151") or as an approval-gate timeout ("No answer within
1 minute"). The root cause is transport, not the payload's content, so retrying
the same large call fails the same way. Small `tool_call`s (short todo updates,
one short argument) succeed.

## What to do

- Do not embed a multi-KB task, spec, or transcript inline in a `tool_call`
  argument (this hit `delegate_task` and the deferred `opencode_run`).
- Write the full brief/spec to a scratch file with `write_file`, then pass only a
  short pointer path in the `tool_call` argument. The child reads the file.
- For `opencode_run`, several managers bypassed it entirely and drove the
  `opencode` CLI directly from `terminal` (`opencode run --auto <task-file>
  --format json`), which does not go through the dropped argument path. That is a
  valid fallback when the tool keeps dropping.
- A foreground `terminal` call also has a ~420s wall; a long `opencode run` can be
  killed at that wall after its work already completed. Check the on-disk artifact
  and the opencode session NDJSON before assuming it failed.
- `terminal` blocks `&` inline backgrounding; use `background=true` for
  servers/mocks and drive them with `process_manage`.

Observed 2026-11-04 during the cli-tool-setup skill refactor fan-out (7
vertical features via `delegate_task` -> `opencode_run`).
