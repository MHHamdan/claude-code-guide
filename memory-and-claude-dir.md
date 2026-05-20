# The `.claude/` Directory and Memory System

The `.claude/` directory is Claude Code's project-level brain. Understanding it is key to getting consistent results across sessions.

## Full directory layout

```
.claude/
├── settings.json          # Permissions, hooks, model, MCP servers
├── settings.local.json    # Local overrides (gitignored — personal preferences)
├── memory/                # Persistent memory files
│   ├── project-context.md
│   ├── decisions.md
│   └── *.md               # Any markdown file Claude writes here persists
└── commands/              # Custom slash commands
    ├── review.md
    ├── deploy.md
    └── *.md
```

**What to commit vs gitignore:**
- Commit: `settings.json`, `commands/`, `memory/` (if team-wide)
- Gitignore: `settings.local.json`, personal memory files you don't want shared

## settings.local.json

For personal overrides that shouldn't affect teammates:

```json
{
  "model": "claude-opus-4-6",
  "defaultPermissionMode": "acceptEdits"
}
```

Add to `.gitignore`:
```
.claude/settings.local.json
```

## Memory in depth

Claude Code has two memory mechanisms:

**1. CLAUDE.md** — Static, version-controlled, loaded at session start. Best for stable project knowledge: architecture decisions, conventions, critical rules.

**2. `.claude/memory/`** — Dynamic, updated during sessions via `/memory`. Best for evolving context: work-in-progress decisions, team context, session carry-overs.

### Managing memory in a session

```
> /memory
```

Opens an editor showing all memory files. You can read, edit, and delete entries.

```
> "Remember that we decided to drop the Redis dependency in favor of in-process caching"
```

Claude writes this to `.claude/memory/` and will recall it in future sessions.

### Structuring memory files

Claude writes freeform markdown to memory. You can structure it yourself for clarity:

```markdown
<!-- .claude/memory/decisions.md -->
# Architecture Decisions

## 2025-01-15 — Dropped Redis
We removed the Redis dependency. All caching is now in-process (LRU via `lru-cache`).
Reason: operational overhead not justified for our traffic levels.

## 2025-01-20 — Auth migration in progress
We're migrating from JWT to session cookies. Auth service is at 60%.
Remaining: refresh token flow, logout propagation.
DO NOT add new JWT-based endpoints until migration completes.
```

### When memory beats CLAUDE.md

Use memory for:
- Decisions that are temporary ("we're in the middle of migrating X")
- Context that changes weekly ("the staging environment is down for maintenance")
- Things you tell Claude during a session that you want to persist

Use CLAUDE.md for:
- Permanent architecture decisions
- Team conventions that won't change
- Project structure that's unlikely to shift

## Custom slash commands in depth

Commands in `.claude/commands/` become `/command-name` in any session.

**Anatomy of a command:**

```markdown
<!-- .claude/commands/deploy-check.md -->
# /deploy-check

You are doing a pre-deployment review. Check the following:

1. Run `make test` and confirm all tests pass
2. Run `make lint` and confirm no errors
3. Check `git diff main` for any accidentally committed debug code, TODOs marked DONOTSHIP, or hardcoded values
4. Verify `CHANGELOG.md` has an entry for this release
5. Check that no `.env` files are staged

Report findings. If anything fails, stop and explain what needs fixing before deploying.
```

**Command naming:** The filename (without `.md`) becomes the command name. `deploy-check.md` → `/deploy-check`.

**Parameterized commands:** Commands can reference context from your prompt:

```markdown
<!-- .claude/commands/fix-issue.md -->
# /fix-issue

Read the GitHub issue number provided. Fetch its details via the GitHub MCP server.
Understand the bug report or feature request. Implement a fix or feature in the relevant code.
Write a test that covers the case. Run the full test suite.
```

Usage: `> /fix-issue #234`

## Practical workflow

Here's how to use the full system effectively on a real project:

```bash
# Day 1: Set up the project
echo "project-root"
git init
mkdir -p .claude/{memory,commands}

# Create settings.json with your team's preferences
cat > .claude/settings.json << 'JSON'
{
  "model": "claude-sonnet-4-6",
  "defaultPermissionMode": "acceptEdits",
  "deniedPaths": ["data/raw/", "migrations/", ".env*"],
  "hooks": {
    "Stop": [{"hooks": [{"type": "command", "command": ".claude/hooks/require-tests.sh"}]}]
  }
}
JSON

# Create CLAUDE.md with project context
claude "Read the codebase and generate a CLAUDE.md that captures the architecture,
        key conventions, and critical rules for this project"

# Review and edit CLAUDE.md, then commit
git add .claude/ CLAUDE.md
git commit -m "chore: add Claude Code project configuration"
```

```bash
# Daily use: start a session, let Claude load context
claude

# Mid-session: compact if context is getting long
> /compact

# At end: ask Claude to capture decisions made
> "Update .claude/memory/decisions.md with any architectural decisions we made today"
```
