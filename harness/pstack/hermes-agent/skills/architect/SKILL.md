---
name: architect
description: "Sketch types, signatures, and module structure before code, then stay in the loop while implementation fills in. Use for /architect, 'architect this', 'design this', or non-trivial work where jumping to code would lock in the wrong shape."
disable-model-invocation: true
---

# Architect

In Hermes, read the Hermes adapter once per session before you follow this skill. It maps the Cursor names in pstack skills to Hermes. Read it with `read_file` at `/Users/I564554/.hermes/skills/pstack/docs/harness/claude-code.md`; `skill_view` cannot serve files outside the skill directory.

Before writing code, trace every system the change touches, then sketch two different shapes for it (types, signatures, module boundaries) and pick the one where each piece of state has one owner, each task has one way to do it, and callers need to know least. Redesign instead of patching if workarounds repeat.

For a large or contested design, run the **arena** skill instead of sketching alone.
