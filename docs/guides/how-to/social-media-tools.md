# How do I… social-media-tools

Discovering teachable code, scrubbing it for secrets, drafting platform-aware
instructional copy, and rendering dark-mode social cards for LinkedIn, Twitter/X,
Bluesky, and Substack. Seventeen skills, one command. One skill is command-only.

Back to the [index](./README.md).

**No path in this plugin auto-posts.** Human review is always required before
anything reaches a social network. Every post leads with a concrete, applicable
takeaway rather than an announcement.

Card skills share one rendering pipeline: find a free port → serve → screenshot
with Playwright → kill. They also read a project-root `SOCIAL.md` for default
platform, tone, hashtags, focus areas, audience, and avoid patterns.

---

## How do I share something without picking a skill?

- **Command** — `/social-media-tools:social-share`
- **Just ask** — "Share a lesson from what I just built" · "Post today's changes
  with a takeaway" · "Share this: https://youtu.be/abc123" · "Share what I
  learned this week"
- **What happens** — the router. It classifies the content type in your request
  and dispatches to the right `share-*` skill.
- **Gotcha** — start here when you are unsure. The router is also the only
  supported path to `share-project`, which does not activate on passive intent.

---

## How do I share a session recap?

- **Command** — `/social-media-tools:share-session`
- **Just ask** — "Share my session" · "What did I work on today?" · "Session
  recap"
- **What happens** — reads the session JSONL and git history and produces a
  narrative recap plus a `session-card` covering lessons learned, takeaways, and
  token usage.
- **Gotcha** — this is the shareable-card path. For a reviewer-facing page,
  use `artifact-tools:session-artifact`; for a raw Markdown transcript, use
  `export-session` below.

---

## How do I archive a session as Markdown?

- **Command** — `/social-media-tools:export-session`
- **Just ask** — "Export this session" · "Save this session as Markdown" ·
  "Archive this session"
- **What happens** — converts the session JSONL transcript into readable
  Markdown under `{plansDirectory}/sessions/`.
- **Gotcha** — reference material, not a post: no card, no copy, no scrub-gated
  publish. `artifact-tools` ships a deliberate copy of this extractor so it can
  install standalone; if you change one, change both.

---

## How do I share a code change?

- **Command** — `/social-media-tools:share-code`
- **Just ask** — "Write a LinkedIn post about today's changes" · "Tweet about
  the v0.3.0 release" · "Share this commit"
- **What happens** — detects the changes from git, scrubs them, drafts
  instructional platform-aware copy, picks a template, and screenshots the card
  via Playwright.
- **Gotcha** — copy is drafted per platform against real character limits, so
  the same change yields different text for Twitter/X than for LinkedIn. On the
  tighter platforms the takeaway wins and the closing learn-more line is dropped.

---

## How do I share a blog post?

- **Command** — `/social-media-tools:share-blog`
- **Just ask** — "Share this blog post: https://dev.to/example/my-article" ·
  "Write a LinkedIn post about `./posts/my-article.md`"
- **What happens** — fetches OG tags from the URL (or reads the local `.md`),
  populates `blog-card.html`, and screenshots it.
- **Gotcha** — it depends on the target page exposing usable OG tags; a page
  without them yields a thin card, so check the draft before posting.

---

## How do I share a video?

- **Command** — `/social-media-tools:share-video`
- **Just ask** — "Write a tweet about this YouTube talk: https://youtu.be/abc123"
  · "Post about this Vimeo video on LinkedIn"
- **What happens** — fetches metadata via oEmbed and screenshots a `video-card`
  with platform-aware copy.
- **Gotcha** — YouTube and Vimeo only, and it needs the video to be public for
  oEmbed to answer.

---

## How do I share a snippet from GitHub?

- **Command** — `/social-media-tools:share-github`
- **Just ask** — "Share this function on Twitter:
  https://github.com/owner/repo/blob/main/src/auth.ts#L42-L68" · "Post about
  this file"
- **What happens** — fetches the public GitHub file (honoring an `#L42-L68`
  line range), runs the security scrub, fills `snippet-card.html` with syntax
  highlighting, and screenshots it.
- **Gotcha** — public repositories only. The scrub still runs even though the
  code is already public — a file being on GitHub is not evidence it carries no
  secrets.

---

## How do I share code I've selected or pasted?

- **Command** — `/social-media-tools:share-selection`
- **Just ask** — "Share this code" · "Tweet this snippet" · "Post the
  highlighted code"
- **What happens** — takes selected, highlighted, open, or pasted code, scrubs
  it, picks the template that fits, and renders an objective-driven card.
- **Gotcha** — the scrub matters most here: pasted code often comes from a
  working file with real values still in it.

---

## How do I share a React component?

- **Command** — `/social-media-tools:share-react`
- **Just ask** — "Share my Button component" · "Share this component with its
  props" · "Post this React component"
- **What happens** — builds a static rendered preview (up to three states),
  pairs it with the implementation code and a full typed props table, and
  screenshots `react-card.html`.
- **Gotcha** — the preview is **static**, built for the screenshot; it is not a
  live component, so anything that only exists on interaction will not appear.

---

## How do I announce a feature, fix, or release?

- **Command** — `/social-media-tools:share-project` — **command-only**
- **Just ask** — nothing directly; `disable-model-invocation: true`. Reach it
  through the `social-share` router with a `--topic` flag, or invoke the command
- **What happens** — reads git history, `CHANGELOG`, and `README` for the topic
  (feature, bug, change, or release) and renders a `feature-card` with copy.
- **Gotcha** — command-only precisely because passive intent matching would fire
  it on any mention of a release. Route through `social-share` or name the
  command.

---

## How do I find code worth sharing?

- **Command** — `/social-media-tools:share-scan`
- **Just ask** — "Find commits worth sharing" · "Create a digest" · "Scan the
  codebase for shareable code"
- **What happens** — scans git history or a codebase path for teachable material
  and writes a digest file under `.claude/digests/`, each entry carrying a
  ready-to-paste share prompt.
- **Gotcha** — this is the discovery half of the pipeline; the card skills are
  the generation half. Run `/social-media-tools:digest` instead when you want the
  interactive review gate over the candidates.

---

## How do I find a post I made before?

- **Command** — `/social-media-tools:media-library`
- **Just ask** — "Browse the media library" · "Find a prior post" · "View saved
  posts"
- **What happens** — scans `docs/media/social/` and generates a filterable HTML
  index of saved cards, so you can retrieve copy for reposting.
- **Gotcha** — it only sees what was saved to `docs/media/social/`; a card you
  rendered and never saved is not in there.

---

## How do I set sharing defaults for a project?

- **Command** — `/social-media-tools:share-init`
- **Just ask** — "Set up social sharing preferences" · "Create a SOCIAL.md for
  this project" · "Configure sharing defaults"
- **What happens** — analyzes the codebase and generates a `SOCIAL.md` at the
  project root with default platform, tone (including Instructional /
  Educational), hashtags, focus areas, audience, and avoid patterns.
- **Gotcha** — run this once per project and stop re-answering platform and tone
  on every share. Every share skill reads the file automatically afterwards, and
  it is a plain Markdown file you can edit by hand.

---

## How do I check something for secrets before sharing it?

- **Command** — `/social-media-tools:security-scrub`
- **Just ask** — "Check this diff for credentials before I share it" · "Scrub
  this file for sensitive data" · "Review this for leaks"
- **What happens** — scans code or a diff for secrets, credentials, tokens, and
  PII.
- **Gotcha** — it is a sub-step utility that every sharing and publishing path
  in the kit already calls, including `artifact-tools`. A `BLOCKED` verdict is a
  hard stop with no override. It does **not** flag local filesystem paths, since
  those are not secrets — which is why the session exporters record only
  basenames.

---

## How do I write a long-form internal guide?

- **Command** — `/social-media-tools:write-guide`
- **Just ask** — "Write a guide on X" · "Document this rule" · "Deep-dive X" ·
  "Capture this session's lessons as a guide"
- **What happens** — writes a long-form internal developer guide to `docs/`,
  assembled from a section library with five non-binding archetype starting
  points (system, rule, how-to, concept, change recap, saved memory), verifying
  every URL and on-disk fact before it lands. Every guide closes with a Quick
  reference and a Cross-references list.
- **Gotcha** — this produces **Markdown you keep in the repository**. When you
  want a shareable page instead, use `artifact-tools:teach-artifact`; the two
  are deliberately different deliverables.

---

## How do I explain how something works?

- **Command** — `/social-media-tools:share-explanation`
- **Just ask** — "Explain how X works" · "How does this component work?" ·
  "Break down this concept"
- **What happens** — reads the source files, synthesizes the underlying
  principles rather than paraphrasing the code, and generates a card plus copy.
- **Gotcha** — the sibling of `write-guide` at a different length: this is one
  card, that is a document.

---

## How do I save an artifact I just built?

- **Command** — `/social-media-tools:save-artifact`
- **Just ask** — "Save this artifact" · "Save this claude.ai artifact URL" ·
  "Stash the artifact I just built"
- **What happens** — copies a local `.html` or fetches a claude.ai artifact URL
  into the local artifacts inbox, scrubs it, and publishes it into the committed
  `docs/artifacts/` gallery.
- **Gotcha** — this is the skill `content-tools:artifact-to-post` points you at:
  it produces exactly the local `.html` that skill needs, because `WebFetch`
  cannot read an authenticated artifact URL directly. The `artifact-tools` recap
  commands file their pages through here too, falling back to a collision-safe
  local copy when this plugin is not installed.

---

## Commands

### How do I review share candidates before drafting anything?

- **Command** — `/social-media-tools:digest [--days=7] [--base=main] [--max=20] | --codebase <path>`
- **What happens** — runs `share-scan`, presents the candidates for multi-select
  review, and writes the approved entries to
  `.claude/digests/code-digest-YYYY-MM-DD.md`, each with a ready-to-paste share
  prompt.
- **Gotcha** — the review gate is the difference from calling `share-scan`
  directly. Use `--codebase <path>` to scan a directory's patterns instead of
  git history when the interesting material is not in recent commits.

---

## Card types

| Type | Best for |
|------|----------|
| `diff-card` | Code changes, rule updates, config diffs |
| `feature-card` | Releases, new features, version announcements |
| `quote-card` | Insights, opinions, pull quotes |
| `blog-card` | Blog post or article shares |
| `video-card` | YouTube or Vimeo video shares |
| `snippet-card` | GitHub code file or snippet shares |
| `session-card` | Session recaps — lessons, takeaways, token usage |
| `react-card` | React components — preview, implementation, props table |
