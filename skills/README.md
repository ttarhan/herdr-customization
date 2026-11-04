# Agent skills

Skills for coding agents working alongside Herdr. Install with
`bin/install-skills`, which symlinks each folder into `~/.claude/skills/`.

## herdr-core

Based on the official stub in
[herdrdev/herdr-core `skills/herdr-core/SKILL.md`](https://github.com/herdrdev/herdr-core/blob/master/skills/herdr-core/SKILL.md)
(upstream commit `ae1ba8a50ff9`, 2026-10-27). Like upstream, it only says
when to use hc and points to `hc --skill` / `herdr-app --skill`. It never
copies the manuals, which the installed tools print for their own version.

One deliberate change: **sub-agents stay on Claude Code's own Agent tool.**
Upstream says to prefer hc "whenever you want sub-agents… over your own
sub-agent tool". Foreman needs native agents, because rule-skill preloading,
`SendMessage` resume and its seat rails all depend on them. Probes on
2026-11-04 showed that `claude --agent foreman:<role>` under hc keeps the
role, model, effort and rails, but not the preloaded `skills:`. Ask
explicitly to use hc agents.

Second change: the skill reads `$HC_TERMINAL_ID` itself when it loads
(`` !`printenv HC_TERMINAL_ID || true` ``, pre-approved through `allowed-tools`).
Without that, the agent has to check with a shell command. When that command
needed a permission prompt, the agent skipped hc. The `|| true` matters: with a
bare `printenv`, the skill failed to load outside Herdr.
`hc` itself is deliberately not pre-approved, because `hc run` can run anything.

### Tested 2026-11-04 (Claude Code 2.1.318, hc 0.1.18)

Headless `claude -p` runs in a scratch git repo with Foreman installed:

| Request | Runs | Result |
|---|---|---|
| Start a server, check it, stop it | 3 | all used `hc run --bg` / `hc kill` / `hc rm` |
| Run a ~30 s test suite | 7 | 6 used `hc run --bg` + `hc wait` (2 with the final skill). The 1 miss came from the first version, where the env check needed a permission prompt; that's what led to reading the env at load time |
| "Use a sub-agent to…" | 3 | all used the `Agent` tool; skill not loaded |
| "Start a foreman:scout agent…" | 3 | all used `Agent(foreman:scout)`; skill not loaded |
| "Use hc to start a helper Claude agent" | 1 | `hc run --bg --name … claude "…"`, `hc wait`, report file, `kill`/`rm` |
| "What files are in this folder?" | 2 | plain `ls`; no hc |
| `HC_TERMINAL_ID` unset | 2 | skill loads, value empty, agent asks before using hc (1 slow-suite run, 1 load check with the final skill) |

When upstream changes, diff it against this file and carry the change over,
keeping both changes.
