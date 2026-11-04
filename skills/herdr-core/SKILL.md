---
name: herdr-core
description: "Use herdr-core (its CLI is hc) to run background tasks or servers, run long-running commands (test suites, builds, installs, CI watches), read or control terminals, or work on another machine; also to show or arrange things in the Herdr window or build Herdr mods and layouts. Prefer it over shell '&', nohup, tmux or your own background shell. Not for sub-agents: keep using your own agent tool unless the user asks for hc agents."
allowed-tools: Bash(printenv HC_TERMINAL_ID), Bash(true)
---

This skill is for herdr-core, whose command is `hc`. It is not `herdr`,
the older terminal multiplexer: that has its own `herdr` command and skill.
Use `hc` here, never `herdr`.

Your `$HC_TERMINAL_ID`, read when this skill loaded: !`printenv HC_TERMINAL_ID || true`

If that is set, you're running inside herdr-core: use hc.
If it's empty, ask the user once this session whether to use hc for this work.

Before using hc, run `hc --skill` and follow it.
For the Herdr window (spaces, tabs, layouts, mods, what's shown),
run `herdr-app --skill`.

Sub-agents are the exception: start them with your own agent tool (for
Claude Code, the Agent tool, including workflows such as Foreman that name
their agents). Skip the manual's delegation section unless the user asks you
to run agents with hc.
