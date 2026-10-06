---
name: verify-before-push
description: Never push red — run the full verification loop (build, test suites, behavior exercise, watch CI to completion) and report evidence, not plausibility. Use before every push, merge, or release, when a change needs testing, or when a CI/test failure needs triage.
---

# Verify before push

Prove that things work. The bar is **evidence, not plausibility**: you ran
something, and you can say what it printed.

## 1. Suites by area

- Build + full test suite on the dev machine. Fix everything red before
  touching remote — CI is for verification, not discovery.
- Define per-area commands once (AGENTS.md), then run the ones the diff
  touches: JVM → `./gradlew test`, Python → `pytest`, Go → `go test ./...`,
  compose → `docker compose config --quiet`, and so on.
- A change that crosses a shared contract (schema, message format, golden
  fixture) must pass **every** suite that asserts it — drift must fail on both
  sides.
- Lint/format in the same pass if the project has it configured.
- **Full sweep before declaring substantial work done:** every suite the repo
  has, green.

## 2. Behavior verification (beyond suites)

Suites prove the logic holds; the task promised an observable behavior — an
endpoint response, a message flow, a state transition. Verify that too, at the
smallest scope that proves it:

1. Name the observable behavior the task promised.
2. **Bug-fix tasks — capture the failure first.** Reproduce the broken
   behavior *before* writing the fix and keep the artifact (failing test
   output, curl request/response, query result). That capture is the "before"
   half of the evidence pair. Never invent a before state; if the fix is
   already in the working tree, use the test that fails without it, or say so.
3. Exercise it for real: call the endpoint (and prove an invalid credential is
   *rejected*), watch the message flow through the queue, check the state
   transition in the datastore. Fixtures and test doubles only — never a live
   external service in a verification context.
4. When full-stack verification is impractical in the moment (long boots,
   missing services), say exactly what was verified (unit/integration suites)
   and what was NOT (the end-to-end path) — never imply more confidence than
   you earned.

## 3. Remote loop

1. Push, then **watch CI to completion**. Do not assume green; do not
   context-switch away first.
2. For releases: the tag's release workflow must finish, and the artifact must
   be verified (download it, run it) before announcing anything.

## 4. Environment gotchas (they look like code bugs but aren't)

- Exec bits stripped by Windows checkouts → `git update-index --chmod=+x <script>`
  (CI symptom: exit code 126 on a script step).
- GitHub Actions major-version deprecation warnings → bump the action version
  in the same commit, not "later".
- Non-interactive shells missing toolchain init (SDKman, nvm, etc.) → export
  `JAVA_HOME`/`PATH` explicitly in remote build commands.

## 5. Rules

- **Deterministic.** Rerunning a suite gives the same result. Flakiness is a
  finding, not noise to retry past.
- **Isolated.** No verification step may hit a real external service or paid
  API. If a change seems to require live calls to verify, that's a design
  smell — flag it.
- **Failures verbatim.** Report the command, the failing assertion, the error
  — then fix within the task's scope or escalate. Never summarize a failure
  away.
- **Honest gaps.** If tests were not run (toolchain missing, services down),
  the honest output is "not verified, here's why and what's needed".
- **When red anyway:** report the exact failing command and its raw output,
  fix forward, and only then move on. A workaround that turns CI off is a bug
  you shipped on purpose.
