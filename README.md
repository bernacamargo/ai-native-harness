# ai-native-harness

[![CI](https://github.com/bernacamargo/ai-native-harness/actions/workflows/ci.yml/badge.svg)](https://github.com/bernacamargo/ai-native-harness/actions/workflows/ci.yml)

The **AI-native development playbook** I use to ship solo at team speed. Distilled from real production use: [vagaremota.dev](https://vagaremota.dev) — a live remote-jobs platform built end-to-end with AI agents under a repo harness of `AGENTS.md`, 16 project-specific skills, and worktree-based parallel sessions — plus two production-grade MCP servers: [iam-mcp-server](https://github.com/bernacamargo/iam-mcp-server) and [incident-mcp-server](https://github.com/bernacamargo/incident-mcp-server).

Three pieces, each boring on purpose:

| Piece | What it does | Where |
|---|---|---|
| **AGENTS.md conventions** | Every agent session inherits project facts, quality gates, and boundaries — no rediscovering, no drift | [`templates/AGENTS.md`](templates/AGENTS.md) |
| **Agent skills** | Reusable playbooks that encode *how* work ships here, not just what to build — milestones, verification, multi-agent waves | [`skills/`](skills/) |
| **Subagents** | Read-only reviewer specialists (architecture, security, QA, PR) plus wave workers — specialists that analyze and report, never edit | [`agents/`](agents/) |
| **Git worktree strategy** | N agent sessions in parallel, each in an isolated worktree, merging clean | [`worktrees/`](worktrees/), [`bin/wt`](bin/wt) |

## Why it works

- **Context is pre-loaded.** A skill is a distilled senior-engineer decision; AGENTS.md is the repo's contract. The agent starts where you'd want a new hire to start on day three, not day zero.
- **Guardrails over vibes.** Agents act through typed interfaces (MCP tools, test suites, CI) with verification after every step. "Never push red" is a skill, not a hope.
- **Reviewer specialists are read-only by constitution.** The architecture, security, QA, and PR reviewers in [`agents/`](agents/) may run tests and read everything, but editing stays with the primary agent — separation of concerns between judging and fixing.
- **Isolation enables parallelism.** Worktrees make concurrent sessions safe; the strategy doc says when *not* to parallelize, which matters more.

## The multi-agent pattern: agent-waves

The pieces compose: [`skills/agent-waves`](skills/agent-waves/SKILL.md) runs a whole feature as **parallel waves** — the coordinator decomposes an approved plan, creates one worktree per task with `bin/wt`, dispatches [`agents/wave-worker.md`](agents/wave-worker.md) subagents concurrently, verifies evidence between waves, and integrates in landing order. The only human gates: approve the plan, approve the merge.

## Adopting it

1. Copy [`templates/AGENTS.md`](templates/AGENTS.md) to your repo root and fill the placeholders with real commands — vague gates are ignored gates.
2. Copy the skills you want into wherever your agent reads `SKILL.md` files from (Claude Code-compatible agents: `~/.claude/skills/` or your plugin dir; Cursor: adapt as rules).
3. Try the worktree loop on your next task with two independent workstreams: `bin/wt new <branch>`, open a session in the printed path, merge in landing order.

The skills follow the open `SKILL.md` convention (YAML frontmatter: `name`, `description`, then instructions). Ports and pull requests welcome — if you run a variant that works, I want to see it.
