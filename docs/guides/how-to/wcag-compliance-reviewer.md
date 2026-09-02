# How do I… wcag-compliance-reviewer

WCAG 2.2 Level AA accessibility review for HTML/CSS and React/TypeScript. One
skill; no commands, no agents.

Back to the [index](./README.md).

---

## How do I check my code for accessibility violations?

- **Command** — none; the `wcag-compliance-reviewer` skill auto-activates
- **Just ask** — "Review this component for accessibility issues" · "Check my
  form for WCAG compliance" · "Audit this page for a11y standards" · "Are there
  any accessibility violations in this code?"
- **What happens** — identifies the code's characteristics (file types,
  interactive elements, dynamic content, media), loads the relevant criteria,
  and audits against WCAG 2.2 AA organized by the four principles — Perceivable,
  Operable, Understandable, Robust. Findings come back categorized as Errors,
  Warnings, or Recommendations, each with a before/after code fix, plus a
  recommendation of which automated testing tools suit the code's complexity.
  WCAG 2.1 and 2.0 criteria are covered too, since 2.2 includes them.
- **Gotcha** — by default it reads the **bundled** criteria at
  `references/wcag-aa-guidelines.md` rather than fetching from W3C, which is
  faster but pinned to WCAG 2.2. Say "review this using the latest official
  guidelines" to make it fetch from the W3C site instead — worth doing if you
  need to know about anything published after 2.2. Automated review also cannot
  substitute for the manual checks it recommends: keyboard walkthroughs and
  screen-reader passes are still yours to run.
