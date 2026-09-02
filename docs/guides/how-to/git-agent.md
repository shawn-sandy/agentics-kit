# How do I… git-agent

Automated git workflow — branches, conventional commits, PRs, ship pipelines,
merges, and GitHub/GitLab issues. Eight skills, five commands, five background
agents, and three hooks. Four skills are command-only.

Back to the [index](./README.md).

Every foreground skill has a **hard STOP boundary**: it does its one job and
stops, with no autonomous test runs, coverage analysis, or scope expansion
afterwards.

---

## How do I create a branch?

- **Command** — `/git-agent:branch-agent [branch-name]` — **command-only** ·
  argument hint: omit the name to auto-generate from uncommitted changes as
  `<type>/<scope>-<description>`
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — guards against detached HEAD and a missing `origin`,
  resolves the name (verbatim if you gave a valid one; auto-slugified to
  whole-word kebab-case, max 60 chars, if you gave a phrase; auto-generated from
  the working tree if you gave nothing), appends a `-YYYY-MM-DD` suffix, fetches
  the default branch, and runs `git checkout -b <branch> --no-track
  origin/<default>`.
- **Gotcha** — the branch is created with **no upstream tracking** by design;
  `pr-agent` and `ship` set it on the first push. If the tree is clean and you
  gave no name, it stops and asks rather than inventing one.

---

## How do I commit my changes?

- **Command** — `/git-agent:commit-agent` — **command-only** · background:
  `/git-agent:commit-bg [hint]`
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — stops on a clean tree or detached HEAD, runs `git add -A`,
  analyzes `git diff --staged`, writes a conventional commit message, commits,
  prints the hash and the undo note (`git reset HEAD~1`), then asks via
  `AskUserQuestion` whether to push — pushing only if you approve.
- **Gotcha** — it stages **everything** (`git add -A`), so stash or clean
  anything you did not mean to include first. It stops after the push prompt; it
  will not test or open a PR.

---

## How do I open a pull request?

- **Command** — `/git-agent:pr-agent` — **command-only** · background:
  `/git-agent:pr-bg [hint]`
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — guards (detached HEAD, default branch, `gh` auth), detects
  the base branch via `git symbolic-ref` falling back to `main`/`master`, stops
  if a PR already exists, pushes if there is no upstream ref, then runs `gh pr
  create` and prints the URL.
- **Gotcha** — if the invoking skill reported a verification marker such as
  `UNVERIFIED — no browser`, it is reproduced **verbatim** in the PR body's Test
  Plan, so a reviewer can see that a check did not happen. It also auto-links
  plan issue references from `<meta name="plan-issue">` tags.

---

## How do I commit and open a PR in one step?

- **Command** — `/git-agent:ship` — **command-only** · background:
  `/git-agent:ship-bg [hint]`
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — runs **all five** pre-flight guards (clean tree, detached
  HEAD, default branch, CLI auth, worktree env parity) and prints **one**
  PASS/BLOCKED table with a paste-ready remediation per blocker, rather than
  stopping at the first failure. Then stages, writes a conventional message,
  commits, pushes, and runs `gh pr create` — or reports the URL and stops if a
  PR already exists.
- **Gotcha** — **nothing is remediated automatically.** No re-auth, no `git
  stash`, no copying an env file: re-auth is an interactive browser flow that
  cannot succeed unattended, and a silent stash or env copy would move your data
  without asking. The Remediation column is text for you to run. `FAIL` (a guard
  that could not run at all) is treated as BLOCKED — a guard that cannot answer
  has cleared nothing. The worktree env-parity guard is skipped outside a linked
  worktree; inside one it lists the gitignored `.env*` files the main checkout
  has and this worktree lacks, and prints the `cp` for each without ever running
  it.

---

## How do I ship and walk away?

- **Command** — `/git-agent:ship-autonomous`
- **Just ask** — "Ship it autonomously" · "Ship and watch the PR" · "Ship and
  fix what breaks" · "Ship and autofix CI failures"
- **What happens** — the supervised full pipeline: pre-flight, branch if on the
  default, **run the tests and stop on failure** rather than committing a red
  tree, preview the change in both themes where a browser can prove it, commit,
  open the PR, subscribe to its activity events, and end the turn. CI failures
  and review comments then arrive as events. On each event it refreshes a live
  status checklist and either autofixes the allow-listed classes (`lint`,
  `typecheck`, `peer-deps`, ≤3 attempts per check), applies clear in-scope
  review changes, or asks.
- **Gotcha** — an **external blocker** (billing or quota block, expired
  credentials, revoked permission, a workflow awaiting approval, or a run whose
  jobs all failed producing no log output) is reported verbatim and **never
  autofixed**, and does not advance the attempt cap: a failing check is not a
  code defect until proven one. Event subscription needs a remote execution
  environment (Claude Code on the web, or GitHub Actions); run locally without
  the GitHub MCP server it falls back to synchronous `gh pr checks --watch`
  polling with the same ≤3-attempt autofix. Merging **always** asks, and
  deleting the branch needs its own separate approval — a merge approval never
  authorizes `--delete-branch`. Use `ship` if you do not want CI watching.

---

## How do I merge a PR once it's green?

- **Command** — `/git-agent:merge` · background: `/git-agent:merge-bg [pr]`
- **Just ask** — "merge?" · "Is this ready to merge?" · "Merge the PR if it's
  green"
- **What happens** — checks the branch's PR for readiness (`MERGEABLE`, a
  `mergeStateStatus` of `CLEAN`, `UNSTABLE`, or `HAS_HOOKS`, required checks
  successful or skipped, no `CHANGES_REQUESTED`), then asks for explicit
  approval before running `gh pr merge --squash --match-head-commit`. Anything
  pending, failing, or ambiguous prints the status summary and asks.
- **Gotcha** — typing the bare shorthand **`merge?`** routes here
  deterministically via a bundled `UserPromptSubmit` hook. It never passes
  `--delete-branch` — cleanup is `post-merge-cleanup`'s job, with its own
  safety checks. An **undispatched** CI run is treated as a block, not a pass:
  an empty run list, a run with no jobs, or every job failing with zero-byte
  logs is named as a block with its likely cause rather than called "CI green".
  For `/git-agent:merge-bg`, the optional argument is the **PR to merge**, not a
  summary hint, and running the command *is* the approval for that one merge.

---

## How do I file an issue?

- **Command** — `/git-agent:create-issue [bug|feature|selection|session|plan] [title or description]`
- **Just ask** — "File a bug" · "Open an issue" · "Create a feature ticket" ·
  "Log this as an issue"
- **What happens** — detects the host from `git remote get-url origin` (`gh` for
  GitHub, `glab` for GitLab), pre-flights that the CLI is installed and
  authenticated, resolves the source and title, gathers repo context including a
  duplicate-issue search and related files, drafts the body from the matching
  template, shows a Create / Edit / Cancel gate, creates the issue, and opens it
  in the browser.
- **Gotcha** — it **never creates without explicit approval** at the
  confirmation gate. The `plan` source is the interesting one: point it at a
  plan file (`.md` spec or rendered `.html`) and the objective, steps, and
  acceptance criteria become a checklist-style issue with the plan title as the
  issue title. Pass `--no-open` to skip the browser.

---

## How do I clean up after a merge?

- **Command** — `/git-agent:post-merge-cleanup [<branch>|<worktree-path>] [--all] [--dirs]`
- **Just ask** — "Clean up merged branches" · "Remove this worktree" · "Delete
  the branch now it's merged" · "Clear out old worktrees"
- **What happens** — removes a merged branch and its worktree, inspecting the
  worktree first. It finds squash-merged branches that commit ancestry cannot
  see by also checking for a merged PR, and uses `-D` only where that PR is
  positive evidence. `--all` sweeps the repo; `--dirs` reaches unregistered
  leftover directories that `git worktree prune` cannot.
- **Gotcha** — it **stops with the file list whenever `git status --porcelain`
  is non-empty** — untracked, staged, or unstaged alike — and never passes
  `--force`. That is the safety property worth relying on: an untracked scratch
  file in a worktree will halt the cleanup rather than vanish with it.

---

## Commands

Five commands dispatch background subagents and return control immediately:
`/git-agent:commit-bg`, `/git-agent:pr-bg`, `/git-agent:ship-bg`,
`/git-agent:ship-ci-bg`, and `/git-agent:merge-bg`. Each mirrors the
corresponding foreground skill.

`agent-ship-ci` (`/git-agent:ship-ci-bg [pr]`) is the unattended, truncated half
of `ship-autonomous`: it watches an already-open PR's checks, applies one
deterministic autofix per failing check, and reports. It never merges, never
replies to reviews, and never edits source.

There is deliberately **no** `agent-branch` — branch creation is synchronous by
design, since you need to be on the new branch before continuing.

---

## Hooks

### The commit lint gate

`hooks/lint-before-commit.py` is a `PreToolUse` hook on `Bash` that runs your
project's own checks before a `git commit` lands and blocks (exit 2) when the
commit **introduces** a failure.

**Gotcha** — only *new* failures block. It compares the staged index against
`HEAD`, both materialized as throwaway trees from `git archive`, so a repo whose
`HEAD` already fails lint still accepts commits that add no new failures, and
unstaged edits neither block a commit nor change the verdict. Detection walks up
from the commit's directory to the git root, stopping at the first matching
manifest (`package.json` → `scripts.lint` then `scripts.typecheck`;
`pyproject.toml` → `ruff`/`flake8`; `go.mod` → `go vet ./...`; `Cargo.toml` →
`cargo clippy`). A check that cannot run is never a block. Two exceptions do
block on the **whole** output: an unborn branch (no `HEAD` to compare against),
and a baseline that could not be established — it never degrades to skipping,
because a silent pass is the one failure mode that would make it untrustworthy.
Override detection outright with `.claude/lint-gate.json`
(`{ "commands": ["make lint", "make typecheck"] }` — naming a test command makes
this a test gate too), or escape entirely with a `.claude/no-lint-gate` file at
the repo root, which is always read from the working tree so you can disable the
gate without committing anything.

### The scope guard

`hooks/scope-guard.py` is a second `PreToolUse` hook on `Bash` refusing exactly
two commands whose blast radius exceeds their intent: a formatter or linter run
with `--write`/`--fix` and either no path operand or `.`, and `git stash
pop`/`apply` with no stash reference. Explicit paths pass
(`prettier --write src/app.ts`), as do `--check`-only runs and
`git stash pop stash@{2}`.

**Gotcha** — it resolves package scripts before matching, so `npm run fix:all`
is judged by the script's *body* in the nearest manifest, through up to three
delegation hops, and all eight runner spellings behave identically. A missing
manifest or absent script resolves to nothing and never blocks.
