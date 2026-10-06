# ai-native-harness

[![CI](https://github.com/bernacamargo/ai-native-harness/actions/workflows/ci.yml/badge.svg)](https://github.com/bernacamargo/ai-native-harness/actions/workflows/ci.yml)

The **AI-native development playbook** I use to ship solo at team speed. It's the setup behind [vagaremota.dev](https://vagaremota.dev) — a live remote-jobs platform built end-to-end with AI agents — and two production-grade MCP servers: [iam-mcp-server](https://github.com/bernacamargo/iam-mcp-server) and [incident-mcp-server](https://github.com/bernacamargo/incident-mcp-server).

Three pieces, each boring on purpose:

| Piece | What it does | Where |
|---|---|---|
| **AGENTS.md conventions** | Every agent session inherits project facts, quality gates, and boundaries — no rediscovering, no drift | [`templates/AGENTS.md`](templates/AGENTS.md) |
| **Agent skills** | Reusable playbooks that encode *how* work ships here, not just what to build | [`skills/`](skills/) |
| **Git worktree strategy** | N agent sessions in parallel, each in an isolated worktree, merging clean | [`worktrees/`](worktrees/), [`bin/wt`](bin/wt) |

## Why it works

- **Context is pre-loaded.** A skill is a distilled senior-engineer decision; AGENTS.md is the repo's contract. The agent starts where you'd want a new hire to start on day three, not day zero.
- **Guardrails over vibes.** Agents act through typed interfaces (MCP tools, test suites, CI) with verification after every step. "Never push red" is a skill, not a hope.
- **Isolation enables parallelism.** Worktrees make concurrent sessions safe; the strategy doc says when *not* to parallelize, which matters more.

## Adopting it

1. Copy [`templates/AGENTS.md`](templates/AGENTS.md) to your repo root and fill the placeholders with real commands — vague gates are ignored gates.
2. Copy the skills you want into wherever your agent reads `SKILL.md` files from (Claude Code-compatible agents: `~/.claude/skills/` or your plugin dir; Cursor: adapt as rules).
3. Try the worktree loop on your next task with two independent workstreams: `bin/wt new <branch>`, open a session in the printed path, merge in landing order.

The skills follow the open `SKILL.md` convention (YAML frontmatter: `name`, `description`, then instructions). Ports and pull requests welcome — if you run a variant that works, I want to see it.
