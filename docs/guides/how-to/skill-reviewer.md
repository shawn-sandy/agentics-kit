# How do I… skill-reviewer

Auditing SKILL.md files and planning new skills, aligned with Anthropic's
skill-authoring guidance. Four skills, one command, one always-on hook. One
skill is command-only.

Back to the [index](./README.md).

---

## How do I get a skill audited and scored?

- **Command** — `/skill-reviewer:reviewing-skills`
- **Just ask** — "Review the SKILL.md at
  `plugins/my-plugin/skills/my-skill/SKILL.md`" · "Audit this skill and tell me
  if it follows best practices" · "Score my skill and generate a corrected
  version"
- **What happens** — scores the file 0–10 across five dimensions — frontmatter,
  body quality, structure, anti-patterns, discoverability — and grades it
  Excellent / Good / Needs Work / Rewrite. It validates against Anthropic's
  5,000-word limit and folder conventions, checks script quality (assumed
  installs, unqualified MCP tool references, voodoo constants, missing error
  handling), and can auto-correct frontmatter errors while flagging body issues
  as inline `<!-- SUGGESTION -->` comments.
- **Gotcha** — there is an optional **regression check** (Step 2c) that diffs
  against the last committed version and reports breaking changes (a renamed
  `name:`, a removed trigger phrase) and regressions (removed reference files, a
  >30% line reduction, new anti-patterns) as BREAKING / WARNING / INFO —
  separately from the 1–10 score, so a high score never hides a regression. Say
  "review my skill using the latest official guidelines" to fetch criteria live
  from `platform.claude.com` instead of the bundled reference. Overwrites always
  require a second explicit confirmation.

---

## How do I plan and scaffold a new skill?

- **Command** — `/skill-reviewer:planning-skills`
- **Just ask** — "Help me plan a new skill for code formatting" · "I want to
  create a skill that reviews PR descriptions" · "What design pattern should I
  use for a deploy workflow skill?"
- **What happens** — walks a structured workflow covering frontmatter, body,
  references, and scripts, identifies which design pattern fits — Sequential,
  Orchestrator, Iterative, or Adaptive — and generates the complete skill folder
  with SKILL.md, reference files, and scripts.
- **Gotcha** — pattern selection is the part worth slowing down for; the
  scaffold is easy to regenerate, the wrong pattern is not. Run
  `reviewing-skills` over the result before you ship it.

---

## How do I stop a skill prompting for permission mid-run?

- **Command** — `/skill-reviewer:auditing-allowed-tools [SKILL.md path] [session UUID]`
- **Just ask** — "What allowed-tools should
  `kit/plugins/foo/skills/bar/SKILL.md` have?" · "Fix the permissions on my
  skill so users don't get prompted mid-run" · "What tools did Claude actually
  use in this session?" · "Did `foo/bar/SKILL.md` actually need everything it
  declared?"
- **What happens** — works out the minimal `allowed-tools` frontmatter the skill
  needs and either recommends it or patches it in. It also parses Claude Code
  session JSONL transcripts to report what tools Claude *actually* invoked, and
  can cross-reference a skill against a real session.
- **Gotcha** — the transcript cross-reference is the high-value mode and the one
  people miss: a declared tool list derived from reading the skill is a guess,
  while one derived from a real run is evidence — in both directions, catching
  both the missing grant that interrupts a run and the over-broad grant nobody
  needed.

---

## How do I fix a skill's description and invocation setting?

- **Command** — `/skill-reviewer:optimizing-skill-frontmatter` — **command-only**
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — optimizes two frontmatter fields in one pass. It rewrites
  `description:` to the three-part format — short description ≤80 chars, then a
  capability sentence, then a trigger phrase, ≤200 chars total — and sets
  `disable-model-invocation` to the right value: `true` for write-heavy workflow
  skills, omitted for read-only advisory ones.
- **Gotcha** — command-only because it writes to SKILL.md files, so it must not
  fire on an offhand mention. The 200-char target is not arbitrary: Claude
  Code's default `skillListingBudgetFraction` (1% of the context window)
  allocates roughly 8,000 chars across *all* installed skills, so the safe
  average shrinks as you install more — ~200 chars at ≤40 skills, ~160 at ~50,
  ~80 at ~100. The three-part format is ordered so the short description
  survives truncation even at ~100 skills.

---

## Commands

### How do I check description lengths across many skills at once?

- **Command** — `/skill-reviewer:check-description [path-or-glob]`
- **What happens** — measures the `description:` frontmatter length for one or
  more SKILL.md files and warns on anything over budget. It shares
  `scripts/measure-description.sh` with the hook below, so the threshold is
  defined once.

---

## Hook

A `PostToolUse` hook fires on every Write/Edit/MultiEdit to a SKILL.md and warns
when the description exceeds budget, pointing you at
`/skill-reviewer:optimizing-skill-frontmatter`.

**Gotcha** — it only fires on SKILL.md files **inside the current git
repository**; external plugins installed under `~/.claude/plugins/` are skipped.
It also dedupes, firing only when the `description:` line actually changes
rather than on every write. To turn it off, override the matcher with an empty
hooks array in your project `.claude/settings.json`:

```json
{ "hooks": { "PostToolUse": [ { "matcher": "Write|Edit|MultiEdit", "hooks": [] } ] } }
```
