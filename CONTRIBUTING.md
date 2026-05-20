# Contributing

Contributions are welcome. This repo is a practical reference — if something
is wrong, outdated, or missing, a PR is the right fix.

## What belongs here

- CLAUDE.md templates that work in real projects
- Hook scripts that solve real problems
- GitHub Actions workflows tested against actual Claude Code behaviour
- `.claueignore` patterns for real project types
- Guides that are accurate to the official docs at [code.claude.com/docs](https://code.claude.com/docs/en/overview)

## What does NOT belong here

- Content that isn't verified against the official docs
- Workarounds for bugs that have since been fixed
- Generic prompt engineering advice unrelated to Claude Code

## Accuracy standard

All content must be verifiable against the official documentation:

- [code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview)
- [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)
- [code.claude.com/docs/en/permission-modes](https://code.claude.com/docs/en/permission-modes)
- [code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

If you're adding a claim about how Claude Code behaves, link the source doc in your PR.
If the official docs don't cover it, mark it clearly as "observed behaviour, not documented."

## PR checklist

- [ ] Content is accurate to current official docs (link the source)
- [ ] Shell scripts are tested and include `set -euo pipefail`
- [ ] JSON configs are valid (run `python3 -m json.tool file.json` to check)
- [ ] CLAUDE.md templates follow the under-200-lines guideline
- [ ] No secrets, tokens, or personal data
- [ ] PR description explains what was added and why it's useful

## Reporting issues

Use the issue templates in `.github/ISSUE_TEMPLATE/`:
- **outdated-content.md** — something here contradicts the current docs
- **missing-content.md** — a use case or template that should exist but doesn't
- **bug-report.md** — a script or config that doesn't work as described
