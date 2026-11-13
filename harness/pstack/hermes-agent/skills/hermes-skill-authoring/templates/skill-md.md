---
name:
description:
version: 1.0.0
author: Hermes
platforms: [macos, linux]         # delete this line if the skill is portable
category: <category>
metadata:
  hermes:
    tags: [tag-one, tag-two]
    # --- optional: conditional activation (fallback skills) ---
    # fallback_for_toolsets: [web]      # hidden while those toolsets exist
    # requires_toolsets: [terminal]     # hidden while those are missing
    # fallback_for_tools: [web_search]
    # requires_tools: [terminal]
    # --- optional: non-secret settings (stored under skills.config) ---
    # config:
    #   - key: my.setting
    #     description: "What this controls"
    #     default: "value"
    #     prompt: "Prompt for setup"
    # --- optional: secrets asked on load (local CLI only) ---
    # required_environment_variables:
    #   - name: SOME_API_KEY
    #     prompt: Some API key
    #     help: Where to get it
    #     required_for: full functionality
---

# Skill Title

Two to three sentences: what this does, what it does NOT do, and the key
dependency stance (e.g. "stdlib only").

## When to Use

- Concrete trigger phrases and situations, not marketing.

## Prerequisites

Exact env vars, installs, credentials. Omit if none.

## How to Run

Canonical invocation, framed through Hermes tools (`terminal`, `read_file`,
`write_file`, `search_files`, `patch`, `web_extract`, ...). Do not name shell
utilities the agent already wraps.

## Quick Reference

Flat command/endpoint list, no narration. Omit if nothing to list.

## Procedure

1. Step one, with copy-paste-exact commands.
2. Step two.

## Pitfalls

- Known failure modes and fixes. A pitfall is a generalizable rule plus one
  clause of why. No incident narration, no PR numbers, no dates.

## Verification

A single command or check that proves the skill worked.
