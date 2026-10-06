# Parallel agent sessions with git worktrees

One agent session per worktree; one worktree per independent workstream. Each worktree gets its own checkout and branch, so N agents work simultaneously with no lock contention and no merge interference — the merge is the coordination point, by design.

## Lifecycle

```bash
bin/wt new feature/auth    # creates ../<repo>-feature-auth on branch feature/auth
                           # → open a fresh agent session in that path
# ... work, verify-before-push, push, open PR ...
bin/wt rm feature/auth     # after merge: removes worktree + local branch
```

## When to parallelize (the important part)

- **Yes:** independent modules — a Dockerfile while a domain refactor lands; two unrelated bug fixes.
- **No:** two halves of the same migration, anything touching the same files, workstreams with an ordering dependency. Sequential with one agent beats parallel with a merge war.

## Merge order

1. Rebase/merge in the order workstreams actually landed.
2. Run the full suite after **each** merge — worktree isolation prevents textual conflicts, not semantic clashes.
3. If two sessions both need to touch `AGENTS.md`, the second one rebases and re-reads it.
