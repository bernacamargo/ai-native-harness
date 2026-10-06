---
name: pr-stack
description: GitHub-native stacked-PR delivery — cut an approved plan into an ordered chain of PRs (each base is the layer below, merged bottom-up). Requires gh >= 2.90 with the github/gh-stack extension, trunk main. Use when the user asks for stacked PRs or after a substantial plan is approved; not for trivial one-file fixes, and never before a human has approved the layer list.
---

# Stacked PR delivery

You are delivering a feature as **one GitHub stack** — an ordered chain of PRs
where each PR's base is the layer below it, merged bottom-up. Small PRs get
real review; one giant PR gets a rubber stamp.

Tooling: `gh` ≥ 2.90 and `gh extension install github/gh-stack`. Trunk is
`main`. Implementation quality still follows your repo's implementation skill;
this skill is how work is cut, branched, submitted, and merged.

## Hard rules

- No `gh stack init`, no feature branches, no push until the **human has
  approved** the layer list and interfaces.
- Never `git add .` / `git add -A` across layers. Stage only files that belong
  to the current layer.
- Do not split "write the test" and "implement" into separate PRs. Tests ship
  with the layer that adds the behavior.
- Do not give several agents the **same checkout** and tell each to take a
  slice — that is a merge-fight factory, not parallelism.
- Never rebase, amend, or force-push **`main`**. Never rewrite commits already
  on trunk.
- After plan approval, `gh stack submit` and `gh stack sync` may push **stack
  branches only**. Cascading rebase of unmerged stack branches via
  `gh stack rebase` / `sync` is allowed and expected.

## 1. Plan the layers

- Restate the feature against the approved plan. Invariant or contract change
  → design doc / ADR first.
- Dispatch read-only reviewers (`architecture-reviewer`, `security-reviewer`,
  Explore) **in parallel** when their concerns are independent.
- Output an ordered **layer list**. Each layer names:
  - **Files** (create/modify/test)
  - **Interfaces:** Consumes / Produces — the signatures later layers rely on
  - Validation command(s) from `AGENTS.md`

**Checkpoint:** you can name each PR in the stack and what would make that
layer mergeable *without* the layers above it.

## 2. Plan approved

Stop. Wait for explicit approval of layer order and interfaces. This is the
human gate; `gh stack init` does not happen before it.

## 3. Cut the layers

Typical ordering when a feature spans the stack — omit layers that do not
apply, and keep tightly coupled changes in one layer:

1. **Contracts** — schemas, migrations, message formats, fixtures in **every**
   suite that asserts them (schema drift must fail on both sides)
2. **Core services** — API / consumers / auth: the only paths that touch the
   system of record
3. **Workers / integrations** — background processing, external calls
4. **UI / BFF** — last, consuming the locked interfaces

Fold doc updates into the layer that introduces the behavior, unless the docs
change is large enough to review alone.

Each layer must stay green against **its** base. A lower layer must not depend
on unmerged code from an upper layer.

## 4. Implement (serial by default)

Per layer: implement, add the tests for that behavior, run the validation
commands, review the diff. Then:

```text
gh stack init          # first layer only; name the branch; trunk = main
# conventional commit(s) for this layer only
gh stack add layer-N   # next layer; or: gh stack add -Am "feat: ..."
```

- **Default:** one agent (or one subagent) per layer; the next branch is
  created with `gh stack add` from the layer below. The primary agent reviews
  the diff before adding the next layer.
- **Worktrees:** a second agent may start layer N+1 only if layer N's
  **Produces** signatures are locked **and** the parent commit exists.
  Otherwise it is fake parallelism and you will pay for it in merge fights.
- Run [`verify-before-push`](../verify-before-push/SKILL.md) **per layer**,
  not once at the top — every layer must be shippable on its own.

## 5. Submit and review

```text
gh stack submit        # push stack branches, open/link PRs
gh stack view
```

After submit, reviews **are** parallel: the human, bot reviewers, and QA
reviewers can each sit on a different layer. [`pr-reviewer`](../../agents/pr-reviewer.md)
is **one dispatch for the whole stack** — give it every PR (or just the head
branch) and it reviews the chain as one unit before the URLs go to the human;
never one instance per PR.

- PR titles: Conventional Commit scoped to the layer, e.g. `feat(api): …`.
- PR bodies: `gh stack submit` opens layer PRs with default bodies. Fill one
  body per layer PR following the repo's PR template, scoping WHAT / HOW /
  Evidence to that layer's diff.

## 6. Sync and merge

```text
gh stack sync          # fetch, cascading rebase, push, retarget PR bases
gh stack merge         # contiguous bottom-up; branch protections still apply
```

Merging a middle layer lands everything beneath it. A partial merge of the
bottom is fine; upper PRs stay open and retarget.

If the repo enables a merge queue, re-check GitHub's stacked-PR merge docs
(queue support has been rolling out in public preview).

## Done when

- The approved plan's layers match `gh stack view`.
- Each PR's CI is meaningful for that layer.
- Every layer PR body follows the repo's PR template.
- Merge proceeds from the bottom when that layer is approved.
