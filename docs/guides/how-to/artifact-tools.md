# How do I… artifact-tools

Publishing the things teams review most — diffs, sessions, plans, prompts, and
explanations — as live claude.ai artifact pages. Five skills, three commands.

Back to the [index](./README.md).

All five skills record the returned URL so later sessions republish to the
**same** link, and all five fall back to local HTML when publishing is
unavailable. Publishing needs a claude.ai login on Pro or higher; sharing an
artifact beyond its author is a Team/Enterprise feature, so on Pro and Max the
local-HTML fallback is how pages actually reach teammates — a first-class path,
not an error.

---

## How do I publish a diff for review?

- **Command** — `/artifact-tools:diff-artifact`
- **Just ask** — "Publish this diff for review" · "Publish the diff for PR #42"
  · "Publish a walkthrough of `abc123..def456`"
- **What happens** — resolves the diff from the branch (default), a commit
  range, or a PR number via `gh`, runs the scrub gate, annotates each meaningful
  hunk with the *reasoning* behind the change, and builds one self-contained
  page with a sticky file sidebar, per-hunk reviewer notes, and severity labels.
- **Gotcha** — annotation is capped at **20 files and 8 hunks per file** to stay
  under the 16 MiB artifact limit; files past the budget render as one-line
  summary rows, and the report tells you how many were summarized rather than
  annotated. This skill scrubs **twice** — once on the raw diff and again on the
  finished page, because annotating a hunk can quote file context the diff never
  contained, and the second scan is the one that actually covers what ships.
  Without `gh` or a GitHub remote, PR mode degrades to branch mode with a
  message.

---

## How do I share a recap of this session?

- **Command** — `/artifact-tools:session-artifact`
- **Just ask** — "Share a recap of this session" · "Publish what we did today"
- **What happens** — finds the transcript (explicit path, session ID, or newest
  for the project) and extracts turns with a bundled `export_session.py`, which
  keeps the raw JSONL out of context. The recap is a reviewer-first document —
  Summary, Decisions with rationale, Learnings, Files touched — saved under
  `{plansDirectory}/sessions/` so its `artifact-url:` frontmatter is committed
  and survives for republish, then published as Markdown.
- **Gotcha** — the record under `{plansDirectory}/sessions/` is what makes the
  stable URL work; if it is not committed, a later session mints a new page
  instead of updating the one you shared. The extractor records only the
  transcript **basename** on purpose — an absolute path would leak your username
  and repo layout into a shared page, and the scrub gate does not catch local
  paths because they are not secrets.

---

## How do I publish a plan?

- **Command** — `/artifact-tools:plan-artifact`
- **Just ask** — "Publish `docs/plans/add-dark-mode.html`" · "Share this plan"
- **What happens** — a thin publish wrapper; plan HTML needs no generation. It
  reads `artifact-url:` from the plan's sibling `.md` spec and passes it to the
  `Artifact` tool's `url` parameter so republishes hit the same page, writing
  the URL back into the spec on a first publish. Viewers then watch steps check
  off live as you work.
- **Gotcha** — **never hand-edit the plan HTML.** It is generated, and the next
  rebuild overwrites your edit. Edit the `.md` spec instead. This is also the
  only one of the five with no scrub gate, because it publishes prose you
  already wrote rather than raw code or prompt text.

---

## How do I share a saved prompt?

- **Command** — `/artifact-tools:prompt-artifact [--library]`
- **Just ask** — "Share `docs/prompts/task-refactor.md`" · "Publish my prompt
  library"
- **What happens** — publishes prompts saved by `plan-agent:prompt`, resolving
  the prompts directory exactly as that skill does (`promptsDirectory` from
  settings, then `{git-root}/docs/prompts`, then cwd-relative). Default mode
  publishes one prompt and records `artifact-url:` in its frontmatter;
  `--library` publishes one filterable gallery of every saved prompt, with chips
  for all five types (`task`, `system`, `creative`, `analytical`, `proposal`) —
  including types with nothing in them, since an absent chip reads as a broken
  filter. The copy button is the point of the page and does a verified
  byte-for-byte round-trip back to the source text.
- **Gotcha** — library mode tracks its URL in a **`.artifact-url` sidecar** in
  the prompts directory, since a gallery has no source `.md` to hold
  frontmatter. **Commit that sidecar** — gitignoring it gives every clone its own
  gallery URL, which is exactly the link rot the stable-URL design exists to
  prevent. In library mode a scrub finding in *any* prompt stops the whole
  publish rather than dropping one card, because a gallery that silently omits
  work reads as complete.

---

## How do I publish a page explaining how something works?

- **Command** — `/artifact-tools:teach-artifact`
- **Just ask** — "Publish a page teaching how this works" · "Publish an
  explainer for PR #455"
- **What happens** — the one page here that is not a report. The others answer
  *what changed*; this answers *how this works*, taking the change as raw
  material and the system as the subject. It follows a five-part spine fixed for
  both sources — mental model, how it works today, one path walked end to end,
  why it is built this way rather than the obvious alternative, where to look
  next — and reads the real diff hunks (capped at 20 files) because signatures
  and error paths are the part commit bodies never carry.
- **Gotcha** — the mental-model section earns a diagram **by default**,
  inverting the recap rule that a diagram is earned only where something
  changed: a system that did not move this week is the one most in need of a
  picture. Every diagram carries a prose sentence beside its caption, because
  the documented fallback ships diagram blocks as plain text when the browser
  pane is unavailable. Its neighbour is `social-media-tools:write-guide`, which
  produces long-form Markdown you keep in the repo; this produces a shareable
  page.

---

## Commands

All three recap commands are framings over `session-artifact`, and all three
take the same sources: no argument (or a session ID or `.jsonl` path) reads the
session transcript; `#453`, a PR URL, or `--pr 453` reads the pull request
instead — `gh pr view`, the changed-file list, commit bodies, and review
discussion. Without `gh` or a GitHub remote, PR mode says so and falls back to
session mode rather than failing.

### How do I write up a session for the product team?

- **Command** — `/artifact-tools:product-doc [session-id|path|#PR]`
- **What happens** — asks for prose any non-engineer can follow and replaces the
  recap's Learnings section with a release-note shape: Features, Bug fixes,
  Decisions, Logic and behavior changes, Implementation plan details, Known gaps
  and follow-ups. Empty sections are dropped rather than printed as bare
  headings.
- **Gotcha** — its republish key is `product-artifact-url:`, deliberately *not*
  `artifact-url:` — reusing that key would republish the product recap over the
  reviewer recap's page. Re-running against the same PR updates the same page,
  so a link sent on day one shows the merged state on day five.

### How do I write up a session for the whole team?

- **Command** — `/artifact-tools:team-recap [session-id|path|#PR]`
- **What happens** — a page you can skim: an at-a-glance stat strip, one card
  per change, mermaid diagrams for anything whose shape changed, a before/after
  table of changed rules and defaults, decisions with the options that lost,
  learnings, open items, files touched, and a glossary of every internal term
  used. Readable by engineers and non-engineers in one pass.
- **Gotcha** — diagrams are `<pre class="mermaid">` blocks, which artifacts
  render natively; the artifact CSP blocks external scripts and assets outright,
  so there is no chart library to reach for. Learnings is usually empty in PR
  mode — a diff records what shipped, never what was tried and abandoned.
  Republish key: `team-artifact-url:`.

### How do I write up a session for the next engineer?

- **Command** — `/artifact-tools:eng-recap [session-id|path|#PR]`
- **What happens** — the inverse of `team-recap`: lead with the technical fact,
  assume the vocabulary, carry no glossary, use code where code is the shortest
  correct statement. Eight sections — stat strip, architecture and code paths,
  decisions with rationale, tradeoffs and rejected options, learnings, tests and
  verification including what is *knowingly* untested, review follow-ups and
  tech debt, files touched.
- **Gotcha** — this is the only recap command that reads the **diff hunks**,
  capped at 20 files, falling back to `--name-only` past the cap and reporting
  how many were summarized rather than read. Tradeoffs and Learnings stay
  distinct on purpose: a tradeoff is a decision that was weighed, a learning is
  a dead end that was walked, and collapsing them loses the dead ends.
  Republish key: `eng-artifact-url:`.
