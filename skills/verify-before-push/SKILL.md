---
name: verify-before-push
description: Never push red — run the full verification loop (build, tests, watch CI to completion) and fix environment gotchas before remote code moves. Use before every push, merge, or release.
---
# Verify before push

## Local loop

1. Build + full test suite on the dev machine. Fix everything red before touching remote — CI is for verification, not discovery.
2. Lint/format in the same pass if the project has it configured.

## Remote loop

1. Push, then **watch CI to completion**. Do not assume green; do not context-switch away first.
2. For releases: the tag's release workflow must finish, and the artifact must be verified (download it, run it) before announcing anything.

## Environment gotchas to check first (they look like code bugs but aren't)

- Exec bits stripped by Windows checkouts → `git update-index --chmod=+x <script>` (CI symptom: exit code 126 on a script step).
- GitHub Actions major-version deprecation warnings → bump the action version in the same commit, not "later".
- Non-interactive shells missing toolchain init (SDKman, nvm, etc.) → export `JAVA_HOME`/`PATH` explicitly in remote build commands.

## When red anyway

Report the exact failing command and its raw output, fix forward, and only then move on. A workaround that turns CI off is a bug you shipped on purpose.
