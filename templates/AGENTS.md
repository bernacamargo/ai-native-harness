# AGENTS.md — <project name>

Instructions for AI coding agents working in this repository.
Humans: keep this current — it is the contract agents operate under. Vague rules get ignored; real commands get followed.

## Project facts

- <What this repo is, one paragraph. Assume the agent read nothing else.>
- Stack: <languages, frameworks, exact versions that matter>
- Build & test: `<exact command>`
- Run locally: `<exact command>`

## Quality gates (non-negotiable)

- Build + full test suite green before any push — no exceptions, no "it's just a small change".
- New behavior ships with tests. When a test fails, fix the code if the test is right; fix the test only if the behavior contract changed — and say so in the commit.
- CI green on `main` is everyone's problem. A red main blocks all other work.

## Conventions

- <Commit/branch style, e.g. conventional commits, imperative subject lines>
- <Error vocabulary, e.g. "bad input throws X, policy denial throws Y — never mix them">
- <Observability expectations, e.g. "every state transition writes an audit entry">

## Never do without asking

- Force-push, delete branches, or rewrite history
- Add dependencies without stating why in the commit body
- Skip, weaken, or sleep() a failing test to make it pass
- Touch <protected paths: CI workflows, release configs, infra>

## Session hygiene

- Read this file first. Re-read after rebases — rules may have changed.
- Smallest change that satisfies the task. Expansion is a separate commit.
- When blocked, report the exact command and its raw output — not a paraphrase of it.
