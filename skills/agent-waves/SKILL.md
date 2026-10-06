---
name: agent-waves
description: >-
  Orchestrate a goal as parallel "agent waves": the coordinator chat (strong
  model) decomposes an approved plan into tasks, then dispatches one background
  worker subagent per task, each isolated in its own git worktree (via the
  worktree helper), running concurrently and reporting back before the next
  wave. Use when the user asks to run work "as waves", to parallelize a
  multi-part feature across agents, or wants worktree branches driven without
  opening a chat per worktree. Not for single-task work or read-only research
  fan-out (plain subagents already do that).
---

# Agent waves — coordinator + parallel workers in worktrees

One chat does everything. The **coordinator** is this chat: it owns the plan,
the architecture, and integration, and it dispatches **workers** — background
subagents (see [`agents/wave-worker.md`](../../agents/wave-worker.md)), each
pointed at its own worktree — in concurrent batches called **waves**. The user
never opens a chat per worktree; the only human gate is approving the wave
plan.

## Division of labor

- **Coordinator (this chat)**: decomposes the goal into tasks; creates one
  worktree + branch per task (`bin/wt new <branch>`); dispatches waves;
  verifies each wave's branches; decides the next wave; integrates (stacked
  PRs where the repo uses them). Owns all architecture decisions — workers
  implement, they never redesign.
- **Worker (one subagent per task)**: implements exactly its task spec inside
  its worktree, commits Conventional Commits on its branch, reports evidence.
  Its constraints are in [`agents/wave-worker.md`](../../agents/wave-worker.md).

## Wave lifecycle

1. **Plan gate.** Decompose the approved goal into tasks that are parallelizable
   per [`worktrees/README.md`](../../worktrees/README.md) — independent files
   or modules, no ordering dependencies. Present the wave plan (tasks, files,
   Consumes/Produces) and get the human's approval **before** creating
   worktrees.
2. **Setup.** `bin/wt new <branch>` per task. Write each task spec: goal, exact
   files, what it consumes from earlier waves, what it must produce, the tests
   that prove it, and the commit convention.
3. **Dispatch a wave.** One `wave-worker` subagent per task, all in the same
   message (they run concurrently). Each dispatch carries: absolute worktree
   path, branch, full task spec.
4. **Verify the wave.** For each worker report: read the evidence, run the
   repo's test suite in the worktree yourself, review the diff against the
   spec. Failures either bounce back to the same worker with a corrected spec
   or become a task in the next wave.
5. **Next wave.** Tasks whose inputs exist only after this wave's outputs.
   Repeat from 3 until the plan is exhausted.
6. **Integrate.** Merge branches in landing order, full suite after each
   merge, worktrees removed with `bin/wt rm <branch>` only after the human
   confirms.

## Hard rules

- The human approves the wave plan and the integration — agents never merge
  to `main` unreviewed.
- A worker never widens scope. Discovered work becomes a wave task, not a
  side quest.
- If a wave's tasks turn out to touch the same files, stop and re-plan:
  the failure is in the decomposition, not in the workers.
