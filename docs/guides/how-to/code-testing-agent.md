# How do I… code-testing-agent

Purpose-driven tests tied to actual behavior and intent — not arbitrary coverage.
Six skills, no commands, no agents. Two are command-only.

Back to the [index](./README.md).

---

## How do I find out what tests I should write?

- **Command** — `/code-testing-agent:code-testing-agent`
- **Just ask** — "Suggest tests for `src/services/auth.ts`" · "What tests should
  I write for this function?" · "Test the `parseJWT` method in
  `src/utils/token.ts`" · "What should I test in my current branch?"
- **What happens** — parses a file path and optional function name straight out
  of your message (falling back to conversation context or recent git changes),
  looks for an implementation plan to understand your intent, analyzes the code
  for behavioral summary, critical paths, integration points, implicit
  contracts, and fragility, detects your test framework and existing patterns,
  then proposes prioritized tests — each naming the specific code it validates
  and why it matters. It offers to write the files afterwards.
- **Gotcha** — the output includes a **"Tests NOT Suggested"** section
  explaining why certain obvious-looking tests would be low-value; that section
  is the point of the skill, not filler. Coverage-only tests are tagged
  `[coverage-only]` and appear only when needed to hit a declared coverage
  target, and the coverage assessment is qualitative — it names covered
  functions and uncovered gaps rather than guessing a percentage.

---

## How do I find out whether my existing tests are any good?

- **Command** — `/code-testing-agent:reviewing-tests`
- **Just ask** — "Review my tests for `src/services/auth.test.ts`" · "Are my
  tests good?" · "Audit my test suite" · "How can I improve these tests?"
- **What happens** — locates the test files and the source they cover (via
  imports, naming conventions, directory structure), analyzes the source across
  five dimensions, then reviews each test against that analysis across nine:
  behavior vs. implementation, naming, assertion focus, coverage gaps, mock
  hygiene, fragility, isolation, plan alignment, and coverage-target progress.
  You get Critical Issues (with line numbers and concrete fixes), Improvements,
  Coverage Gaps ranked by priority, and What's Working Well.
- **Gotcha** — this is the counterpart to the suggest skill, not a replacement:
  it judges tests that exist, and its Coverage Gaps section is what tells you to
  go run `code-testing-agent` for the behaviors nothing covers yet. Fixes are
  offered, not applied silently.

---

## How do I run my tests?

- **Command** — `/code-testing-agent:running-tests`
- **Just ask** — "Run tests for `src/services/auth.ts`" · "Check if tests pass"
  · "Run my tests and show me what's failing" · "Verify my changes didn't break
  anything"
- **What happens** — detects the framework from project config (`package.json`,
  `pytest.ini`, `Cargo.toml`, …), scopes the run to the file or directory you
  named, runs it, and reports pass/fail with the failing assertions and error
  messages surfaced.
- **Gotcha** — it scopes to your target rather than running everything, which is
  fast but means a green result proves only the scope you asked for. Name a
  directory, or nothing, when you want the whole suite.

---

## How do I fix a bug test-first?

- **Command** — `/code-testing-agent:tdd-fix <bug description>` — **command-only**
- **Just ask** — nothing; `disable-model-invocation: true` means natural
  language will not start it
- **What happens** — reads the bug description, locates the source, writes a
  minimal failing test that reproduces the bug (red), applies the smallest fix
  that makes it pass (green), and repeats for up to **10** iterations if the fix
  introduces new failures. Stops and reports when everything passes or the cap
  is reached.
- **Gotcha** — command-only is deliberate: it writes both test and source
  autonomously, so it should never fire on an offhand mention of a bug. The
  10-iteration cap is a stop, not a guarantee — if it reports hitting the limit,
  the bug is not fixed. Use `tdd-loop` for a new feature instead.

---

## How do I build a feature test-first?

- **Command** — `/code-testing-agent:tdd-loop <feature description>` —
  **command-only**
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — scopes the implementation target, writes a set of failing
  tests defining the expected behavior (red), implements incrementally to pass
  each one (green), refactors after each passing cycle without breaking tests,
  and repeats for up to **20** red-green-refactor rounds.
- **Gotcha** — red is free here, which is why this is the right skill for new
  code and `verified-change` is the right one for code that already works. The
  20-round cap works the same way as `tdd-fix`'s: hitting it means unfinished,
  not done.

---

## How do I prove a change is merge-ready before I open a PR?

- **Command** — `/code-testing-agent:verified-change`
- **Just ask** — "Prove this is merge-ready" · "Verify my change locally" · "Run
  the merge gate"
- **What happens** — runs `scripts/verify.sh` **once before touching anything**,
  so a pre-existing failure is never mistaken for yours. Then it writes the
  regression assertion first, mutation-checks it by breaking the implementation
  on purpose to prove the assertion can go red, restores from a scratchpad copy
  under a `trap` with a `cmp -s` proof, implements the change, and loops the gate
  up to **8** times. Rendered changes are browser-verified at 390px and 1280px
  in both themes with an axe run. It emits a VERIFICATION section recording what
  actually ran.
- **Gotcha** — this is for code that **already works**, where a test written
  after the fact can pass for the wrong reason; the mutation check is what
  catches that, and a test that survives its own mutation is rejected and
  rewritten. No gate in the repo? Run `install-verify-gate` from the repo root —
  it writes `scripts/verify.sh` and refuses to overwrite an existing one without
  `--force`. The gate runs typecheck, lint, unit, then e2e, stopping at the first
  failure and printing `SKIP (not configured)` for stages it cannot find. It
  executes whatever tooling the target repo declares, so run it only in repos you
  already trust.
