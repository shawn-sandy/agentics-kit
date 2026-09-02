# How do I… content-tools

Moving a work product — an HTML artifact or a Markdown file — into a real static
site as a draft post. One skill; no commands, no agents.

Back to the [index](./README.md).

---

## How do I turn an artifact into a blog post?

- **Command** — none; the `artifact-to-post` skill auto-activates
- **Just ask** — "Turn this artifact into a blog post" · "Turn `./docs/notes.md`
  into a draft post" · "Convert
  `.claude/artifacts/dashboard-2026-07-20.html` into a post for the site"
- **What happens** — eleven phases: locate assets, resolve the source, a
  blocking security scrub, config and prerequisite checks, extract, prose
  rewrite, MDX-safety pass, screenshots, write, verify, publish gate. Each
  content block is placed on the highest rung of a fidelity ladder that holds —
  native Markdown, then scoped inline HTML, then scoped HTML plus the artifact's
  own script, and a screenshot only as a last resort. The artifact's CSS is
  scoped to a wrapper container, which is what stops it colliding with the
  site's design tokens *and* what keeps `<details>`, `<dialog>`, and range
  inputs working. A Markdown source skips extraction and screenshots; the scrub,
  config checks, and safety pass still run.
- **Gotcha** — it cannot consume a **published claude.ai artifact URL**:
  `WebFetch` cannot read authenticated or private URLs, so the skill refuses one
  and points you at `social-media-tools:save-artifact`, which produces exactly
  the local `.html` this skill wants. Nothing site-specific is hardcoded either
  — output directory, extension, frontmatter keys, draft flag, images
  directory, preview URL, build command, and the interactivity ceiling all come
  from a project-root `CONTENT.md`. The first run in a repo without one asks
  once and offers to write it. Output is always a **draft**; the publish gate at
  the end defaults to no.
