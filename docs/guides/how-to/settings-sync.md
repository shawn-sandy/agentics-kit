# How do I… settings-sync

Backing up and restoring Claude Code user settings to a dedicated git repo.
Skills-only: two auto-activating skills, no commands, no agents.

Back to the [index](./README.md).

What travels: `settings.json`, `CLAUDE.md`, `keybindings.json`, and the
`rules/`, `commands/`, `skills/`, and `hooks/` directories.
`settings.local.json` is opt-in. Auto-generated files — sessions, caches,
plugins, telemetry — are excluded.

---

## How do I back up my Claude Code settings?

- **Command** — none; the `settings-backup` skill auto-activates ·
  argument hint: `[repo-path]`
- **Just ask** — "Back up my Claude settings to `~/dotfiles/claude-settings`" ·
  "Back up my Claude settings" (once a repo is configured) · "Sync my Claude
  Code config"
- **What happens** — resolves the repo path, validates or initializes the git
  repo, scans for secrets on the first run, copies the files (rsync, with a `cp`
  fallback), writes metadata, commits, and pushes. Missing files are handled
  without failing, and a run that changed nothing is logged to `.sync-log` for
  the audit trail.
- **Gotcha** — **never run this first on a new machine.** A fresh install has an
  empty local config, and the backup would copy that emptiness over your good
  backup and push it. Restore first. Once `~/.claude/settings-sync.json` holds
  the repo path the skill runs without prompts, which is what makes it safe to
  put on a schedule — set one up with `/schedule` and the prompt "Back up my
  Claude settings to `<repo-path>`".

---

## How do I restore my settings on a new machine?

- **Command** — none; the `settings-restore` skill auto-activates ·
  argument hint: `[repo-path-or-url]`
- **Just ask** — "Restore my Claude settings from
  `https://github.com/you/claude-settings-backup.git`" · "Restore my Claude
  settings from `~/dotfiles/claude-settings`" · "Set up my Claude config on this
  machine"
- **What happens** — resolves the source (local path or clone URL), clones if
  needed, pulls latest, builds the file list from the backup repo root minus its
  control files, shows you a file-level diff summary, and copies only after you
  confirm. On a new machine the repo is cloned to `~/.claude-settings-backup`
  and that path is saved to `~/.claude/settings-sync.json` for later runs.
- **Gotcha** — on a fresh machine there is no local backup repo, so you must
  pass the **clone URL**, not a path. Restore is always interactive — it will
  not overwrite `~/.claude/` without your confirmation. Restart Claude Code
  afterwards: plugins reinstall themselves from the `enabledPlugins` and
  `extraKnownMarketplaces` entries in the restored `settings.json`, and none of
  it takes effect until the restart.
