# How do I… plan-agent

Plan creation, review, design, and implementation on demand. Eighteen skills,
nine commands, twelve agents, and one dispatching hook that fans out to seven
checks. Four skills are command-only.

Back to the [index](./README.md).

The chain, upstream to downstream:

```
build-proposal   should we?        → docs/prompts/proposal-<slug>.md
build-feature    what, and how does it split? → docs/features/<slug>.md
implementation-plan  how?          → docs/plans/<verb-target>.md (+ artifact)
review-plan      is the plan sound?
design / prototype  what does it look like / does the flow work?
build            do it
finalize-plan    is it actually done?
publish-hub      share all of it as one page
```

Plans are **published to a claude.ai artifact by default**: the repo keeps the
`.md` spec, and the URL is written back as `artifact-url:` so republishing
updates the same link. Pass `--file` to also write and commit `<stem>.html`
beside the spec.

---

## How do I create an implementation plan?

- **Command** — `/plan-agent:implementation-plan <issue-url|#n> | <plan.md> | <objective> [--quick] [--no-clarify] [--no-align] [--no-interview] [--workflow] [--tdd|--no-tdd] [--file] [--from-prompt <path>] [--type feature|fix|refactor|docs|chore] [--dir <path>] [--priority low|medium|high|critical]`
- **Just ask** — "Create a plan document for…" · "Generate an HTML plan for…" ·
  "Write a plan file for…"
- **What happens** — the full Steps 1–8 workflow: Clarify, Create (with a
  `verb-target` filename), Frontmatter, Rename, Align, a structured Step 5b
  Interview that stress-tests the plan, Commit, Publish, then an exit menu
  offering Implement now / Run as workflow / Review the plan / Edit / Exit. It
  authors the plan and **stops** — it never writes source files. Type is
  inferred from the leading verb (`add`/`create` → feature, `fix` → fix,
  `refactor` → refactor, `document` → docs).
- **Gotcha** — passing a `.md` **path** enters conversion mode: the markdown is
  treated as authoritative, Clarify/Align/Interview are skipped, and sections map
  1:1. Passing a source through `--from-prompt` is the opposite — its headings
  are *input*, not a step list to transcribe. The two are mutually exclusive, and
  passing a proposal or feature doc positionally by mistake is the exact bug
  `--from-prompt` exists to prevent. Generic planning questions ("plan how to do
  X") route to built-in Plan Mode, not this skill. The RED/GREEN/VERIFY/SHIP
  shape *is* inferred when the steps touch application source and a test runner
  exists; `--tdd`/`--no-tdd` settle it without asking. All skip flags are opt-in
  and never inferred.

---

## How do I decide whether to build something at all?

- **Command** — `/plan-agent:build-proposal <idea> [--dir <path>] [--tier 0|1|2]`
- **Just ask** — "Should we…?" · "I'm thinking about…" · "Compare these two
  approaches and pick one"
- **What happens** — the upstream layer: it decides *should-we + what* and hands
  the *how* to `implementation-plan`. An 8-step loop — Frame → Confirm the ask →
  fan out web and codebase research in parallel → synthesize the core finding →
  separate established facts from open decisions → resolve the decisions
  recommendation-first → author the artifact → deepen on request → converge and
  hand off. Output is a copy-pasteable prompt at
  `docs/prompts/proposal-<slug>.md`.
- **Gotcha** — Step 1 picks a **Tier** and the tier moves as research reveals
  scope: Tier 0 answers directly with no loop, Tier 1 is one research pass, Tier
  2 is the full loop. The prompt filename carries **no date** on purpose — it is
  a living document deepened over rounds, and a dated name would fork it in two
  the moment a loop crossed midnight. `created:` and `modified:` carry the dates.

---

## How do I turn a committed feature idea into team-ready plans?

- **Command** — `/plan-agent:build-feature <feature idea> [--dir <path>] [--tier 0|1|2] [--publish|--no-publish]`
- **Just ask** — "Write a feature doc for…" · "Break this feature into plans" ·
  "What are we building here, and how does it split?"
- **What happens** — the sibling of `build-proposal` at a different seam: a
  proposal answers *should-we*, a feature doc answers *what are we building and
  how does it split into plans*. It writes a product feature doc at
  `docs/features/<slug>.md` with user stories carrying observable, binary
  acceptance criteria (each covering an unhappy path), goals whose baselines are
  researched or written `unmeasured` rather than guessed, a Release & rollout
  table, and a sub-feature breakdown with rationale, S/M/L size, and dependency
  order. At Tier 1+ convergence it also writes one paste-ready prompt per
  sub-feature under `docs/prompts/feature-<slug>-<sub-slug>.md`.
- **Gotcha** — the breakdown is **recommend-only**: the doc hands you
  paste-ready `/plan-agent:implementation-plan … --from-prompt <path>` lines and
  the skill never invokes plan generation itself. Running them in dependency
  order is your step. At Tier 0 it writes no prompts at all, so the doc is the
  only carrier for the product content — pass it with `--from-prompt`, never
  positionally. Sub-feature prompts are written only at convergence because the
  breakdown can merge or split mid-loop.

---

## How do I get a plan reviewed?

- **Command** — `/plan-agent:review-plan [plan-path] [--dir <path>] [--background] [--skip-analysis] [--triage-top <N>] [--deep]`
- **Just ask** — "Review this plan" · "Improve this implementation plan"
- **What happens** — runs a Workflow of **ten reviewers**: seven core
  (architecture, completeness, testability, risk, conventions, product,
  security) always, plus three UI-conditional (UX, accessibility, frontend)
  when the plan shows 2+ UI signals. Every `critical` or `high` finding is then
  handed to an independent skeptic prompted to *refute* it, defaulting to
  refuted when uncertain — refuted findings are dropped before synthesis. What
  survives is applied inline to the plan, with a collapsible "Team Review"
  section appended and a coverage line reporting how many findings stand,
  survived refutation, went unverified, or were refuted.
- **Gotcha** — this needs the **`Workflow` tool**, but no feature flag and no
  minimum version; if it is unavailable the skill says so rather than crashing
  the planning flow. By default you get an ask-first gate — Walk through
  findings / Apply all / Review only — which `--skip-analysis` bypasses and
  `--background` implies. `--deep` challenges *every* finding rather than just
  the high and critical ones, taking the run from roughly 18 agents to about 50.
  A verdict has three states, not two: `refuted: false` means a skeptic tried
  and failed to kill the finding, `null` means it was below the severity
  threshold and never sent, and `failed` means the verifier itself died — a
  finding whose verifier dies is kept with an honest verdict rather than dropped.
  Reviewers read the full plan HTML and cannot run the spec extractor
  themselves; to get the cheaper spec read, run `extract-plan-spec.mjs` with a
  literal path yourself and paste the output in.

---

## How do I implement a plan?

- **Command** — `/plan-agent:build [<plan.md|plan.html>] [<objective>] [--type feature|fix|refactor|docs|chore] [--dir <path>]`
- **Just ask** — "Implement the plan at…" · "Build this plan"
- **What happens** — the downstream layer to `implementation-plan`: it walks the
  steps, ticks the spec, re-renders, and owns three gates — an
  **acceptance-criteria gate** (verify and check off each criterion), an
  **end-to-end verification gate** (run the objective test and walk the
  Verification section, fixing and re-verifying up to 3 times), and a
  **completion-checklist gate** (steps, criteria, and status agree).
  `status: completed` is written only after end-to-end verification passes. When
  the spec carries a `design-dir:` key, Step 2 reads the artboards first so a
  published design canvas is the reference for anything user-facing.
- **Gotcha** — with **no plan named**, the command form enters the authoring
  chain first: it asks "Start with a proposal" or "Straight to plan authoring",
  delegates to `build-proposal` and `implementation-plan`, then implements what
  it produced. It surfaces a dirty working tree before anything else, refuses to
  silently redo a `completed` plan, and resumes from the first unmarked step. It
  **stops without committing** — the source changes, updated spec, and
  re-rendered HTML are left in your working tree.

---

## How do I ship a backlog of plans in parallel?

- **Command** — `/plan-agent:build-fleet [<plan.md> ...] [--dir <path>] [--max N]`
- **Just ask** — "Implement the backlog in parallel" · "Ship these plans in
  parallel"
- **What happens** — `build` fanned out: one `Agent` per plan with
  `isolation: "worktree"`, each running `build` → `git-agent:ship-autonomous` to
  a green PR. It dispatches only; every step of the work belongs to those two
  skills, including the completion gates, browser verification, CI autofix, and
  review triage. With no arguments it discovers every `status: todo` spec under
  the plans directory (skipping `archive/` and `artifacts/`) and offers the
  newest four in a multi-select picker.
- **Gotcha** — the blast-radius guards are the thing to know: a mandatory
  confirmation naming how many PRs will open, `--max` defaulting to **3**,
  `completed` plans excluded even when named explicitly, and a headless run that
  **cancels** rather than defaulting. It **stops at green** — a background agent
  cannot answer `ship-autonomous`'s merge gate, so merging stays a human step via
  `/git-agent:merge`. The base branch comes from `refs/remotes/origin/HEAD`, and
  an unset `origin/HEAD` asks rather than guessing `main`.

---

## How do I mark a plan completed?

- **Command** — `/plan-agent:finalize-plan [plan-file.md|.html] [--all] [--dir <path>]` — **command-only**
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — reads the acceptance criteria, maps codebase evidence to
  each one individually and classifies it `verified` or `unverified`, runs the
  Tests section's **Run** command for an end-to-end signal, then offers three
  options: check all, only auto-check verified, or cancel. On confirmation it
  sets `status:`, flips the criteria and step markers, writes a
  `## Completion Report` for any gaps, and re-renders. If only the verified
  criteria are checked, status becomes `in-progress`, not `completed`.
- **Gotcha** — `--all` runs a sweep: a cheap non-interactive token-evidence pass
  scores every non-completed plan, one multi-select prompt batch-confirms, and
  the full per-criterion verification runs on the selected plans only. Discovery
  is two scans — file-published plans found by grepping for a `plan-status` meta
  tag, and artifact-published plans found by their spec, which qualifies only
  when its first heading is `# Plan:`, no sibling `.html` exists, `artifact-url:`
  parses as an `http(s)` URL, and `status:` is `todo` or `in-progress`. Those
  gates are why a plan whose *body* documents these keys is not mistaken for an
  unfinished one.

---

## How do I see what a plan will look like?

- **Command** — `/plan-agent:design <plan.html|plan.md | image | figma-url | idea>`
- **Just ask** — "Design this plan" · "Mock this up"
- **What happens** — derives **one artboard per user-facing plan step**,
  uncapped, echoes the derived list back for confirmation, then delegates all
  authoring and publishing to Claude Code's built-in `design` skill. Working
  artboards land under `docs/designs/<plan-slug>/`, and two keys go back into
  the spec: `design:` (the published canvas URL, which the renderer turns into a
  **View design** header link) and `design-dir:` (the artboard directory, read
  by `build` Step 2 as the visual spec and by the drift hook).
- **Gotcha** — a step with **no user-facing surface produces no artboard**, so a
  plan of housekeeping steps yields nothing to look at rather than a wall of
  empty boards. "User-facing" has exactly one definition — the same UI-signal
  keyword list `review-plan` Step 3b applies — and `check-design-drift` uses the
  identical filter, so the skill and the hook never disagree. Two keys rather
  than one slug-derived path because the pair survives a plan rename.

---

## How do I click through a plan before building it?

- **Command** — `/plan-agent:prototype <plan.html|plan.md | image | figma-url | idea> [--from-prompt <path>]`
- **Just ask** — "Prototype this plan" · "Prototype this idea" · "Prototype this
  screenshot"
- **What happens** — where `design` answers *what does this look like*,
  `prototype` answers *does this flow work*. It produces one self-contained file
  under `docs/prototypes/` — inline CSS and vanilla JS, an inline JSON seed, a
  per-prototype `localStorage` store — with **no CDN, no framework, no build**,
  so it opens by double-click on `file://` and publishes to GitHub Pages. The
  skeleton bakes in labeled inputs, a semantic table, real buttons, form
  validation, a confirm-guarded reset, and an `aria-live` status region; output
  is HTML-escaped and rendered via `textContent`.
- **Gotcha** — the input shapes behave differently: a plan path has its data
  model extracted directly, a raw idea triggers a 3-question interview (entity,
  action, success signal), an image is read for the UI it shows, and a Figma URL
  loads the Figma MCP tools — asking for a screenshot instead if no Figma MCP
  server is connected. Either way the derived model is echoed back for
  confirmation before anything is written.

---

## How do I share a plan and everything around it as one page?

- **Command** — `/plan-agent:publish-hub <plan.md> [--extra <path>]...`
- **Just ask** — "Publish a plan hub" · "Share this plan and its prototype
  together"
- **What happens** — bundles the plan, the spec's `prototype:` file, and each
  `--extra` page into one self-contained tabbed page — each document isolated in
  its own `<iframe srcdoc>` panel, everything inline so it satisfies the
  artifact CSP — and publishes it. A `design:` URL becomes an external-link tab.
  After publishing it fetches the URL and confirms the page contains the plan
  title.
- **Gotcha** — the hub URL is recorded as **`hub-artifact-url:`** and the plan's
  own `artifact-url:` is **never touched**, so the two links stay independent.
  Output is capped at 15 MB (under the 16 MB artifact limit); on overflow the
  bundler exits naming the largest embedded document and the skill retries with
  `--skip <file>` and reports what it dropped.

---

## How do I browse my plans?

- **Command** — `/plan-agent:plans-library`
- **Just ask** — "Browse my plans" · "View plan history" · "Open the plans
  index"
- **What happens** — scans every `.html` plan in the plans directory, reads each
  one's `<meta>` tags and `<title>`, writes a filterable
  `<PLANS_DIR>/index.html`, and opens it. The gallery has status chips (All /
  Todo / In Progress / Completed), type chips (Feature / Fix / Refactor / Docs /
  Chore), a title search box, a grid/list toggle, and cards linking to the plan
  files.
- **Gotcha** — the scan always excludes `index.html` itself and the
  `archive/` subdirectory. A `PostToolUse` hook rebuilds this index
  automatically on plan writes, so you rarely need to run it by hand.

---

## How do I reopen the gallery without rebuilding it?

- **Command** — `/plan-agent:plans-open`
- **Just ask** — "Open the plans gallery" · "Show the plans page"
- **What happens** — opens the existing `index.html` directly. No scanning, no
  parsing, no writes.
- **Gotcha** — if `index.html` does not exist yet it tells you to run
  `/plan-agent:plans-library` first rather than building one.

---

## How do I check or update a plan's status?

- **Command** — `/plan-agent:plan-status [plan-file-path | directory] [--all] [--force]`
- **Just ask** — "Check this plan's status" · "Update the status on these plans"
- **What happens** — inspects the codebase to determine whether a plan has been
  implemented, then writes lifecycle status (`todo`, `in-progress`, `completed`)
  and accurate dates from git history into the YAML frontmatter, along with a
  type classification (feature, fix, refactor, docs, chore). Naming a directory
  or passing `--all` routes to bulk mode: a seven-stage flow with a triage
  table, batch date and evidence rules, and one summary approval.
- **Gotcha** — it operates on **`.md`/YAML plans only** and does not support
  `.html` plans. It also does not critique plan content — use `review-plan` or
  the built-in interview for that. It exits plan mode first, since it mutates
  state.

---

## How do I stress-test a plan decision by decision?

- **Command** — `/plan-agent:deep-grill [plan-file-path]` — **command-only**
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — builds a design tree from the plan, then walks each branch
  asking focused questions at every decision node and exploring the codebase to
  resolve them, finishing with a summary.
- **Gotcha** — this is the deepest and slowest of the three stress-test
  surfaces. The Step 5b interview during plan creation is the cheapest,
  `review-plan` is the broad multi-lens pass, and `deep-grill` is the one you
  reach for when a *specific* decision is load-bearing and you want it
  interrogated.

---

## How do I document a plan that shipped?

- **Command** — `/plan-agent:documenting-plans [plan-file-path]` — **command-only**
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — generates a developer-friendly prose document at
  `docs/<slug>.md` from a completed plan, synthesized from the plan body, live
  code inspection, and git history — so it reflects what actually shipped rather
  than what was planned.
- **Gotcha** — it only runs on plans that are `completed` **and 30+ days old**
  (by `modified`, else `created`). Set the status with `plan-status` first if it
  refuses. The `plan-documenter` agent runs this in batch across every completed
  plan with no `docs/` counterpart.

---

## How do I turn a markdown file into a shareable HTML page?

- **Command** — `/plan-agent:markdown-to-html [file-path] [--theme=default|developer|document|minimal] [--mode=auto|plan|doc] [--background] [--no-open]`
- **Just ask** — "Convert this markdown to HTML" · "Turn this plan into HTML"
- **What happens** — converts any markdown file into a rich, self-contained HTML
  page, auto-detecting the render mode: **plan mode** when the source has a
  `## Steps` section or `status:` plus a `Plan:` H1, **doc mode** otherwise. It
  verifies the generated HTML and offers to open it.
- **Gotcha** — this converts a source at any lifecycle stage; it does **not**
  generate documentation from a completed plan, which is `documenting-plans`.
  `--setup` is deprecated — theme CSS and JS are bundled in the plugin now.

---

## How do I write a good prompt?

- **Command** — `/plan-agent:prompt [system|task|creative|analytical] [intent] [--out <path>] [--answers-gathered]` — **command-only**
- **Just ask** — nothing; `disable-model-invocation: true`
- **What happens** — interviews you about the prompting need, classifies the
  prompt type, and generates a copy-pasteable prompt grounded in Anthropic's
  prompting best practices — applying clarity, XML structure, role assignment,
  few-shot examples, chain-of-thought scaffolding, and output formatting
  according to the type. Saved under the prompts directory.
- **Gotcha** — a leading `system`/`task`/`creative`/`analytical` token pins the
  type. A fifth type, `proposal`, is **caller-only**: Phase 2 has no proposal
  question set, so it counts only alongside `--answers-gathered`. The command
  wrapper exists specifically so other skills can reach this one —
  `disable-model-invocation` alone would block them.

---

## How do I publish generated HTML to a public URL?

- **Command** — `/plan-agent:setup-sites`
- **Just ask** — "Set up GitHub Pages for this repo" · "Publish my plans to
  GitHub Pages"
- **What happens** — scaffolds the deploy pipeline into the current repo:
  `.github/workflows/deploy-pages.yml`, `docs/.nojekyll`, a parameterized
  landing hub at `docs/index.html`, and `scripts/serve-docs.sh` — all
  idempotent, none clobbering a file that already exists. It computes the live
  `https://<owner>.github.io/<repo>/` URL from the `origin` remote and warns when
  `plansDirectory` points outside `docs/`.
- **Gotcha** — it **scaffolds and verifies only**; you commit and push when
  ready, and the one-time **Settings → Pages → Source → GitHub Actions** step is
  yours to click. The skill walks you through it.

---

## Commands

Beyond the skill wrappers above, `plan-agent` ships:

| Command | What it does |
|---------|--------------|
| `/plan-agent:fix <objective> [--dir <path>]` | The `build` chain, typed as a fix — authors and implements in one go |
| `/plan-agent:refactor <objective> [--dir <path>]` | The `build` chain, typed as a refactor |
| `/plan-agent:review-plan-bg <path>` | Validates the path, spawns the `agent-review-plan` background agent, and returns an ack immediately — you are notified on completion |
| `/plan-agent:plan-maintenance [--archive] [--index] [--variants] [--all]` | Archives completed plans 30+ days old as HTML, regenerates a README index, and reviews variant/duplicate files. Under `--all`, variant consolidation runs first |

---

## Agents

| Agent | Purpose |
|-------|---------|
| `agent-review-plan` | Background plan review — invokes `review-plan --background` and reports the updated path |
| `plan-reviewer-architecture` · `-completeness` · `-testability` · `-risk` · `-conventions` · `-product` · `-security` | The seven core reviewer lenses, always run |
| `plan-reviewer-ux` · `-accessibility` · `-frontend` | Three UI-conditional lenses, spawned on UI signals |
| `plan-documenter` | Batch documentation — finds completed plans with no `docs/` counterpart and runs `documenting-plans` on each |

---

## Hooks

One `PostToolUse` registration on `Write|Edit|MultiEdit` runs `hooks/dispatch.py`,
which path-gates once and fans out to seven checks only for writes under the
plans, prototypes, or designs trees. A second registration on `ExitPlanMode`
nudges you to stress-test the plan before implementing.

| Check | Fires on | Does |
|-------|----------|------|
| `validate-plan-filename` | plan files | Enforces verb-target kebab-case; **exits 2 to block** a badly-named plan |
| `rebuild-plans-index` | non-`index.html` plan HTML | Regenerates the plans gallery |
| `render-plan-html` | a plan `.md` spec | Re-renders `<stem>.html` when that sibling already exists |
| `build-prototypes-index` | `docs/prototypes/` | Regenerates the prototypes gallery |
| `check-prototype-drift` | `docs/prototypes/` | Warns when a prototype has drifted from its data model or its plan's copy |
| `build-designs-index` | `docs/designs/` | Regenerates the designs gallery |
| `check-design-drift` | `docs/designs/` | Reports a user-facing plan step that no artboard covers |

**Gotcha** — `validate-plan-filename` is the only one that can block a write.
Its checks: strict kebab-case, no harness hex suffix, no trailing date, not a
generic placeholder, an imperative verb first, and a non-stop-word second. The
index rebuilds always exit 0, so a gallery failure never blocks a plan write.
