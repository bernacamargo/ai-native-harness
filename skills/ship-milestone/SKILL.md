---
name: ship-milestone
description: Land any multi-day feature or new repo as a sequence of verified, deployable milestones (M1 skeleton+CI, M2 core work, M3 production dressing) so progress is always shippable. Use when starting a new project or a large feature.
---
# Ship in milestones

Never build "the whole thing" — build M1, ship it, then M2, ship it.

## M1 — skeleton + CI first

1. Repo, build tooling, and a CI workflow (build + test + lint) before any real feature code.
2. One thin end-to-end slice to prove the stack compiles and tests run.
3. Nothing else happens until CI is green on `main`.

## M2 — core work

1. The smallest set of features that makes the thing useful to its first real user.
2. Each push keeps the full suite green — milestones are deployable checkpoints, not branches that live long.

## M3 — production dressing

1. Docs with actual design notes (the decisions, not the obvious).
2. Container image / release automation.
3. Tag `vX.Y.Z`, then **verify the artifact exists and runs** — a release you haven't downloaded is a release you haven't made.

## Rules

- No branch lives long enough to need a rebase strategy.
- "Done" always means CI green + artifact verified, never "code written".
