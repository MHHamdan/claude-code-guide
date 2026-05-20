# CLAUDE.md Before & After

Real examples showing what separates a weak CLAUDE.md from one that produces consistent results.
Every rule here is grounded in the official guidance: keep files under 200 lines, be specific
enough to verify, and avoid contradictions.

Source: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)

---

## What makes a CLAUDE.md instruction effective

The official docs state it directly:

> "The more specific and concise your instructions, the more consistently Claude follows them."
> "Write instructions that are concrete enough to verify."

Specific. Concise. Verifiable. Those three words explain every before/after below.

---

## Example 1 — Vague rules vs. verifiable rules

### ❌ Before (vague, unverifiable)

```markdown
## Code style
- Write clean code
- Format code properly
- Keep files organized
- Test your changes
```

**Why it fails:** None of these are verifiable. "Clean" and "organized" are undefined.
Claude will interpret them however it wants — differently each session.

### ✅ After (specific, verifiable)

```markdown
## Code style
- Indentation: 2 spaces (enforced by ruff — do not fight it)
- Max line length: 100 characters
- Type hints required on all function signatures
- Google-style docstrings on all public functions and classes
- Run `make lint` before marking any task complete — it must exit 0
```

**Why it works:** Every rule has a clear pass/fail. Claude can check its own output
against each one without asking you.

---

## Example 2 — Missing "what not to do"

### ❌ Before (no prohibitions)

```markdown
## Workflow
- Write tests for new features
- Follow the existing patterns
- Keep the codebase clean
```

**Why it fails:** Without explicit prohibitions, Claude will "helpfully" refactor adjacent
code, add dependencies it thinks are useful, and reorganize things you didn't ask about.

### ✅ After (explicit prohibitions)

```markdown
## Workflow
- Write tests for any new function in tests/ before marking task complete
- Run `pytest tests/ -x` — must pass before completing

## What NOT to do
- Do not refactor working code unless explicitly asked
- Do not add new pip/npm dependencies without being asked
- Do not rename variables or functions that are not part of the current task
- Do not add comments to code that already has comments unless asked
- Do not run `git push` — always stop after `git commit`
```

**Why it works:** The "What NOT to do" section is the highest-ROI section in any CLAUDE.md.
Claude's default is to be maximally helpful — which means it expands scope. Explicit
prohibitions constrain that expansion.

---

## Example 3 — Too long, loaded into every session

### ❌ Before (500-line monolith)

```markdown
# CLAUDE.md

## Project overview
[100 lines of architecture explanation]

## Python conventions
[80 lines]

## ML pipeline rules
[120 lines about feature engineering]

## API conventions
[90 lines]

## Database rules
[110 lines]
```

**Why it fails:** The official docs set a target of under 200 lines per CLAUDE.md file.
Longer files consume more context and reduce adherence — Claude starts ignoring rules
buried deep in a long file.

### ✅ After (split by scope, load on demand)

```
CLAUDE.md                  ← project overview + critical rules only (~80 lines)
src/ml/CLAUDE.md           ← ML-specific rules, loads when Claude touches src/ml/
src/api/CLAUDE.md          ← API conventions, loads when Claude touches src/api/
src/db/CLAUDE.md           ← DB rules, loads when Claude touches src/db/
```

Root `CLAUDE.md`:
```markdown
# Project overview
This is a FastAPI service with an ML inference layer. See src/ml/ and src/api/.

## Critical rules (apply everywhere)
- NEVER modify data/raw/ — immutable source of truth
- Run `make test` before marking any task complete
- Do not commit directly to main — always create a branch

## How to run
- Dev: `make dev`
- Test: `make test`
- Lint: `make lint`
```

**Why it works:** The subdirectory CLAUDE.md files load on demand — only when Claude
reads files in that directory. Rules are scoped to where they matter.
Source: [code.claude.com/docs/en/memory#choose-where-to-put-claude-md-files](https://code.claude.com/docs/en/memory#choose-where-to-put-claude-md-files)

---

## Example 4 — Missing verification criteria

### ❌ Before (no way for Claude to check its own work)

```markdown
## Workflow
- Implement features correctly
- Make sure things work
```

### ✅ After (Claude can self-verify)

```markdown
## Definition of done
A task is complete only when ALL of the following pass:
1. `make lint` exits 0
2. `make test` exits 0 with no skipped tests
3. No TODO or FIXME left in files you touched
4. If you changed an API endpoint, its schema matches src/schemas/

Claude: do not ask me to verify — run these checks yourself and fix failures.
```

**Why it works:** The official best practices page identifies this as
"the single highest-leverage thing you can do" — giving Claude a way to verify its own work.
Source: [code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work](https://code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work)

---

## Example 5 — Not using /init

Most people write CLAUDE.md from scratch. The faster path:

```bash
# Run /init inside a Claude Code session — it analyzes your codebase
# and generates a CLAUDE.md with real build commands and conventions
> /init
```

The `/init` command reads your actual project and generates a starting CLAUDE.md with
build commands, test instructions, and project conventions it discovers. Then you refine it
with rules Claude wouldn't find on its own (architectural decisions, prohibitions, team norms).

If a CLAUDE.md already exists, `/init` suggests improvements rather than overwriting.

Set `CLAUDE_CODE_NEW_INIT=1` before running `/init` for an interactive multi-phase flow
that asks which artifacts to set up (CLAUDE.md files, hooks, skills) before writing anything.

Source: [code.claude.com/docs/en/memory#set-up-a-project-claude-md](https://code.claude.com/docs/en/memory#set-up-a-project-claude-md)

---

## Example 6 — Personal overrides without affecting teammates

### Problem
You want a personal project CLAUDE.md with your sandbox URLs and preferred test data,
but you don't want to commit it.

### Solution: CLAUDE.local.md

```bash
# Create a personal, gitignored override
touch CLAUDE.local.md
echo "CLAUDE.local.md" >> .gitignore
```

```markdown
# CLAUDE.local.md — personal overrides (gitignored)

## My local environment
- Dev server runs at http://localhost:8001 (not 8000 — port conflict)
- Use test user: testuser@example.com / password: testpass123
- My local DB: postgresql://localhost/myproject_dev
```

The shared project `CLAUDE.md` stays clean. Your personal context lives in `CLAUDE.local.md`.
Source: [code.claude.com/docs/en/memory#choose-where-to-put-claude-md-files](https://code.claude.com/docs/en/memory#choose-where-to-put-claude-md-files)

---

## CLAUDE.md quality checklist

Before committing your CLAUDE.md, run through this:

- [ ] Under 200 lines? (official target — split into subdirectory files if over)
- [ ] Every rule is verifiable (has a clear pass/fail)?
- [ ] Has a "What NOT to do" section?
- [ ] Has a "Definition of done" or verification commands?
- [ ] No contradictions between rules?
- [ ] Personal/local context is in `CLAUDE.local.md`, not committed?
- [ ] Used `/init` to bootstrap rather than writing from scratch?
- [ ] Large modules have their own CLAUDE.md in their subdirectory?
