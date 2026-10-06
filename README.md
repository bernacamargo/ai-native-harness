# ai-native-harness

[![CI](https://github.com/bernacamargo/ai-native-harness/actions/workflows/ci.yml/badge.svg)](https://github.com/bernacamargo/ai-native-harness/actions/workflows/ci.yml)

The **AI-native development playbook** I use to ship solo at team speed. Distilled from real production use: [vagaremota.dev](https://vagaremota.dev) — a live remote-jobs platform built end-to-end with AI agents under a repo harness of `AGENTS.md`, 16 project-specific skills, and worktree-based parallel sessions — plus two production-grade MCP servers: [iam-mcp-server](https://github.com/bernacamargo/iam-mcp-server) and [incident-mcp-server](https://github.com/bernacamargo/incident-mcp-server).

Four pieces, each boring on purpose:

| Piece | What it does | Where |
|---|---|---|
| **AGENTS.md conventions** | Every agent session inherits project facts, quality gates, and boundaries — no rediscovering, no drift | [`templates/AGENTS.md`](templates/AGENTS.md) |
| **Agent skills** | Reusable playbooks that encode *how* work ships here, not just what to build — milestones, verification, multi-agent waves, stacked-PR delivery | [`skills/`](skills/) |
| **Subagents** | Read-only reviewer specialists (architecture, security, QA, PR) plus wave workers — specialists that analyze and report, never edit | [`agents/`](agents/) |
| **Git worktree strategy** | N agent sessions in parallel, each in an isolated worktree, merging clean — [`bin/wt` usage with screenshots](bin/README.md) | [`worktrees/`](worktrees/), [`bin/wt`](bin/wt) |

## Why it works

- **Context is pre-loaded.** A skill is a distilled senior-engineer decision; AGENTS.md is the repo's contract. The agent starts where you'd want a new hire to start on day three, not day zero.
- **Guardrails over vibes.** Agents act through typed interfaces (MCP tools, test suites, CI) with verification after every step. "Never push red" is a skill, not a hope.
- **Reviewer specialists are read-only by constitution.** The architecture, security, QA, and PR reviewers in [`agents/`](agents/) may run tests and read everything, but editing stays with the primary agent — separation of concerns between judging and fixing.
- **Isolation enables parallelism.** Worktrees make concurrent sessions safe; the strategy doc says when *not* to parallelize, which matters more.

## The multi-agent pattern: agent-waves

The pieces compose: [`skills/agent-waves`](skills/agent-waves/SKILL.md) runs a whole feature as **parallel waves** — the coordinator decomposes an approved plan, creates one worktree per task with `bin/wt`, dispatches [`agents/wave-worker.md`](agents/wave-worker.md) subagents concurrently, verifies evidence between waves, and integrates in landing order. The only human gates: approve the plan, approve the merge.

When a feature needs review granularity instead of one big landing, [`skills/pr-stack`](skills/pr-stack/SKILL.md) delivers the same approved plan as a **stacked-PR chain** (`gh` + `gh-stack`): contracts first, core services above them, UI last; every layer green against its own base; merged bottom-up. [`agents/pr-reviewer.md`](agents/pr-reviewer.md) reviews the whole chain in one dispatch, and [`skills/verify-before-push`](skills/verify-before-push/SKILL.md) runs per layer — a stack is only as mergeable as its bottom layer. That closes the loop: **plan → waves → verify → stacked PRs → release.**

## Third-party skills in rotation

The harness story above is mine, but not every skill in my working set is — the SKILL.md ecosystem is a real open-source exchange, and good work deserves attribution:

- **[grill-me / grilling](https://github.com/mattpocock/skills)** (Matt Pocock, MIT) — interrogates a plan with hard questions *before* any code exists. I run it ahead of the human approval gates in [agent-waves](skills/agent-waves/SKILL.md) and [pr-stack](skills/pr-stack/SKILL.md): the machine attacks the plan so the human approves a survivor.
- **[tdd](https://github.com/mattpocock/skills) · [to-spec](https://github.com/mattpocock/skills) · [code-review](https://github.com/mattpocock/skills)** (same MIT set) — red-green discipline, spec-first framing, and structured review on top of the [read-only reviewers](agents/).
- **[unslop](https://github.com/michaelshimeles/skills/tree/main/unslop)** (Lauren Tan, MIT) — keeps docs and PR bodies from reading like AI output.
- **Vendor packs, installed on demand** — [angular/skills](https://github.com/angular/skills), spring-boot and kotlin skills, plus Stripe/Twilio/Resend doc-skills: domain depth when a task enters their territory, not part of the core story.

The rule I follow: **link and credit, don't fork silently** — and vendor-specific skills never make it into the reusable playbook.

## Adopting it

1. Copy [`templates/AGENTS.md`](templates/AGENTS.md) to your repo root and fill the placeholders with real commands — vague gates are ignored gates.
2. Copy the skills you want into wherever your agent reads `SKILL.md` files from (Claude Code-compatible agents: `~/.claude/skills/` or your plugin dir; Cursor: adapt as rules).
3. Try the worktree loop on your next task with two independent workstreams: `bin/wt new <branch>`, open a session in the printed path, merge in landing order.

The skills follow the open `SKILL.md` convention (YAML frontmatter: `name`, `description`, then instructions). Ports and pull requests welcome — if you run a variant that works, I want to see it.
