# How do I… code-review

Structured, multi-dimensional review across quality, bugs, security, best
practices, complexity, and breaking changes. One skill, one command, one
internal agent.

Back to the [index](./README.md).

---

## How do I get code reviewed?

- **Command** — none; the `code-review-agent` skill auto-activates
- **Just ask** — "Review this function for bugs" · "Check `src/api/users.ts` for
  security issues" · "Look over this PR diff" · "Take a look at this"
- **What happens** — resolves which files to review from git status, the branch
  diff, or an explicit path, then applies a fixed six-dimension checklist:
  quality, bugs, security, best practices, complexity (rated Low → Very High),
  and breaking changes / regressions. Output is a severity-ranked report with
  line numbers and suggested fixes.
- **Gotcha** — it reports; it does not edit. To have the findings applied for
  you, use `/code-review:fix-branch` instead.

---

## Commands

### How do I fix every issue on my branch in one pass?

- **Command** — `/code-review:fix-branch [base-branch]` — defaults to the remote
  default branch, falling back to `main`, then `master`
- **What happens** — reviews every file changed on the branch vs. the base,
  classifies findings as blocking / major / minor / unfixable, and applies the
  fixes itself via `Edit`/`Write`. It checks repo rules (`.claude/rules/*.md`),
  project conventions (`CLAUDE.md`), frontmatter validity (SKILL.md,
  `plugin.json`, `marketplace.json`), and the verification sections of any
  modified `docs/plans/` file. SKILL.md and agent definitions are delegated to
  the `skill-reviewer` plugin and its findings merged in.
- **Gotcha** — it **refuses to run on a dirty working tree**: commit or stash
  first, so the fixes land isolated from your own work. It also leaves the fixes
  **uncommitted** on purpose — inspect with `git diff` before committing. And
  its scope is conventions and correctness-of-form, not logic: it does no
  security or performance review, so run the `code-review-agent` skill for that.

---

## Agent

`agent-code-reviewer` is an internal delegation target, not something you invoke
directly — other agents reach it with
`Agent(subagent_type: "code-review:agent-code-reviewer", ...)`. It runs
read-only (`permissionMode: plan`, `Write`/`Edit` disallowed), in the
background, with project-scoped memory under
`.claude/agent-memory/agent-code-reviewer/`, and filters harder than the skill —
only high-confidence findings surface. For your own reviews, use the skill.
