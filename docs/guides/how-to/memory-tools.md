# How do I… memory-tools

Auditing and optimizing Claude Code configuration — `CLAUDE.md` memory files,
scoped rules, and usage-insights follow-through. Skills-only: three
auto-activating skills, no commands, no agents.

Back to the [index](./README.md).

---

## How do I audit or optimize my CLAUDE.md?

- **Command** — none; the `agentic-memory-management` skill auto-activates
- **Just ask** — "Audit my CLAUDE.md file" · "Optimize my project's Claude
  instructions" · "My Claude is ignoring my CLAUDE.md instructions — what's
  wrong?" · "Review `~/.claude/CLAUDE.md` for issues"
- **What happens** — resolves the target (explicit path → `CLAUDE.md` in the
  current directory → `.claude/CLAUDE.md` → `~/.claude/CLAUDE.md`), measures
  line count, instruction count, sections, secrets, and `@import` references
  with their effective line cost, then scores six dimensions: Instruction
  Budget, Section Quality, the 80% Rule, Progressive Disclosure, Safety &
  Hygiene, and Structure. You get a graded audit — Optimized / Functional /
  Needs work / Rewrite — naming the default-restating rules driving any
  deduction. The core filter is: keep only rules that would change Claude's
  behavior versus its built-in defaults, and cut the rest.
- **Gotcha** — the rewrite is **opt-in**, gated behind an explicit confirmation,
  as is extracting sections into `.claude/rules/` files; an audit alone changes
  nothing. Scope is `CLAUDE.md` and project memory only — it does not cover
  SKILL.md files, slash commands, or general markdown. For skills, use the
  `skill-reviewer` plugin.

---

## How do I add rules that only apply to certain files?

- **Command** — none; the `path-rules-advisor` skill auto-activates
- **Just ask** — "Create path-specific rules for my TypeScript files" · "Add a
  rule for `src/api/**/*.ts` — all endpoints must validate input" · "Analyze my
  project and suggest what should go in `.claude/rules/`"
- **What happens** — two modes. Give it a glob plus a description and it writes
  the scoped rule file directly (Mode A). Give it nothing and it reads your
  project structure and `CLAUDE.md`, recommends which sections deserve to become
  scoped rules, and offers to generate them (Mode B).
- **Gotcha** — it writes to `.claude/rules/` only. It will not create or
  overwrite `CLAUDE.md` or global memory entries — that is
  `agentic-memory-management`'s job. Moving a section into a scoped rule is the
  point: rules that only load for the paths they govern stop costing context in
  every unrelated session.

---

## How do I act on a usage-insights report?

- **Command** — none; the `implementing-insights` skill auto-activates
- **Just ask** — "Implement the findings from this insights report:
  `docs/insights-2026-08.md`" · "Act on the usage-insights report at \<artifact
  URL\>" · "Which of these insights recommendations are already covered by my
  config?"
- **What happens** — parses the report (file path, artifact URL, or pasted
  content) into numbered items, then triages every item against your existing
  config — `~/.claude/`, installed plugins, each target repo — into three
  buckets: already implemented (with the citation), conflicts with an existing
  rule (rejected, with the citation), or genuinely open. Open items are placed
  at the right layer — a plugin for workflow-shaped ones, `~/.claude/` for
  machine-wide, the repo for repo-specific — then implemented one change at a
  time, one PR per repo, with worktree isolation where parallel agents share a
  repo. It finishes with a verified outcome ledger and cleans up its worktrees
  and merged branches.
- **Gotcha** — it implements *only* the genuinely open items, which is the
  whole point: an insights report typically recommends several things your
  config already does. It does not generate the report itself, and it **never
  merges a PR without explicit approval**. Repos are discovered from
  `~/.claude/projects/` first; it only asks for a projects directory when
  discovery fails.
