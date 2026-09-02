# How do I…?

One brief entry per skill across the eleven `agentics-kit` plugins. Each entry
has the same four parts:

- **Command** — the slash command and its argument hint, or `none` when the
  skill has no command wrapper.
- **Just ask** — the plain-English phrasing that triggers it. Skills marked
  **command-only** carry `disable-model-invocation: true` in their frontmatter:
  natural language will *not* start them, no matter how well it matches.
- **What happens** — what the skill actually does, in a sentence or two.
- **Gotcha** — the thing worth knowing before you run it.

Skills are grouped by plugin. Install a plugin before its entries work:

```
/plugin marketplace add shawn-sandy/agentics-kit
/plugin install <plugin>@agentics-kit
```

| Plugin | Guide | Skills |
|--------|-------|--------|
| artifact-tools | [How do I… artifact-tools](./artifact-tools.md) | 5 |
| code-review | [How do I… code-review](./code-review.md) | 1 |
| code-testing-agent | [How do I… code-testing-agent](./code-testing-agent.md) | 6 |
| content-tools | [How do I… content-tools](./content-tools.md) | 1 |
| git-agent | [How do I… git-agent](./git-agent.md) | 8 |
| memory-tools | [How do I… memory-tools](./memory-tools.md) | 3 |
| plan-agent | [How do I… plan-agent](./plan-agent.md) | 18 |
| settings-sync | [How do I… settings-sync](./settings-sync.md) | 2 |
| skill-reviewer | [How do I… skill-reviewer](./skill-reviewer.md) | 4 |
| social-media-tools | [How do I… social-media-tools](./social-media-tools.md) | 17 |
| wcag-compliance-reviewer | [How do I… wcag-compliance-reviewer](./wcag-compliance-reviewer.md) | 1 |

Total: 66 skills across 11 plugins, 12 of them command-only.

For versions and component counts, see the
[Plugin Reference Table](../../../README.md#plugin-reference-table). For the
full behavior of any skill, see its plugin's README under
`kit/plugins/<plugin>/README.md`.
