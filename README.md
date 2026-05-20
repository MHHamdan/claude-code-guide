# Claude Code — Engineer's Field Guide

Practical patterns and production-ready templates for Claude Code. All content verified against the official docs at [code.claude.com/docs](https://code.claude.com/docs/en/overview).

---

## Start here

If you're new to Claude Code, run this first:

```bash
# Install (macOS/Linux/WSL — auto-updates)
curl -fsSL https://claude.ai/install.sh | bash

# Start in your project
cd your-project
claude
```

Claude will prompt you to authenticate on first run. Then read [How Claude Code actually works](#how-it-works) before you do anything else — the mental model matters.

---

## Contents

1. [How it works](#how-it-works) — the agentic loop, context, and permission model
2. [CLAUDE.md engineering](#claudemd-engineering) — the single biggest lever for quality
3. [The `.claude/` directory](#the-claude-directory) — settings, memory, and project config
4. [Permission modes](#permission-modes) — when to trust Claude with what
5. [Referencing context with `@`](#referencing-context-with-)  — scoping what Claude reads
6. [MCP integration](#mcp-integration) — connecting external tools and data
7. [Hooks](#hooks) — guardrails and automation around tool calls
8. [CI/CD automation](#cicd-automation) — GitHub Actions, GitLab, headless mode
9. [Subagents and parallel work](#subagents-and-parallel-work) — running multiple agents
10. [Model selection and cost](#model-selection-and-cost) — which model, when, at what cost
11. [Common mistakes](#common-mistakes) — what goes wrong and why

---

## How it works

Claude Code is a **stateless agentic loop**. Understanding this prevents most frustration.

```
Session start:
  Load CLAUDE.md (global → project → subdirectory)
  Load .claude/memory/ files
  Load .claude/settings.json

Each turn:
  User prompt → Claude plans tool sequence → executes tools in loop:
    ├── Read files / glob / search
    ├── Write files  ← requires permission (or acceptEdits mode)
    ├── Run bash     ← requires permission (or bypassPermissions mode)
    ├── Call MCP servers
    └── Web fetch / search
  Observe tool outputs → next action → repeat until done
```

**Critical mental model points:**

**No memory between sessions.** Every new `claude` invocation starts fresh. Persistence comes from files — `CLAUDE.md`, `.claude/memory/`, and your codebase itself. If you want Claude to remember something, write it to a file.

**Context window is the budget.** Everything Claude reads (CLAUDE.md + files + conversation) consumes context. Large codebases require explicit scoping or you hit limits. Use `/compact` mid-session to summarize history. Use `.claueignore` to exclude noise.

**Read-only by default.** File writes and shell commands require your approval unless you change the permission mode. This is a feature — use `--permission-mode plan` to preview an agentic task before letting it run.

**MCP is how you connect the outside world.** Jira, Slack, Google Drive, custom APIs — all MCP. Not a bolt-on; it's the designed extension point.

---

## CLAUDE.md engineering

`CLAUDE.md` is the highest-leverage thing you control. Claude reads it at the start of every session. Treat it as a persistent system prompt scoped to your project.

### Load order

```
~/.claude/CLAUDE.md              ← global (all projects)
{project-root}/CLAUDE.md         ← project-level
{subdirectory}/CLAUDE.md         ← loaded when Claude reads files in that subtree
```

The hierarchy is additive — all three load and stack. Put global preferences (coding style, commit format) in the global file. Put architecture decisions and project-specific rules in the project file. Put module-specific context in subdirectory files so it only loads when relevant.

### Import syntax

You can include other files in CLAUDE.md to keep it modular:

```markdown
@docs/architecture.md
@docs/api-conventions.md
```

Claude will read those files as part of CLAUDE.md. Useful for keeping the root CLAUDE.md short while pulling in detailed reference docs when needed.

### What actually belongs in CLAUDE.md

The sections that produce the most consistent results:

```markdown
## Critical rules
<!-- Things that, if violated, would break prod or cause real pain -->
- NEVER modify files in data/raw/ — immutable source of truth
- All PRs must pass `make test` before you mark the task complete
- Do not change DB migration files that have already been applied

## What NOT to do
<!-- Explicit prohibitions prevent "helpful" overreach -->
- Do not refactor working code unless explicitly asked
- Do not add logging statements unless debugging a specific issue
- Do not add dependencies without updating pyproject.toml via `uv add`
- Do not use print() — use logging.getLogger(__name__)

## Architecture decisions
<!-- Choices already made that Claude shouldn't re-litigate -->
- Single-process service — no threading, use async/await throughout
- Read from replica, write to primary (connection strings in .env)
- HTTP client: httpx only (requests is banned per DEPENDENCIES.md)

## Code style
<!-- Only things your linter doesn't enforce -->
- Type hints required on all function signatures
- Google-style docstrings
- Commit format: Conventional Commits (<type>(<scope>): <description>)

## How to run and test
<!-- Claude needs to know this to verify its own work -->
- Run: make dev
- Test: make test (runs pytest + coverage; must pass before completing)
- Lint: make lint (ruff check + ruff format)
```

**Templates in this repo:**

| Template | Use case |
|---|---|
| [`examples/claude-md-templates/global-defaults.md`](examples/claude-md-templates/global-defaults.md) | `~/.claude/CLAUDE.md` baseline |
| [`examples/claude-md-templates/python-project.md`](examples/claude-md-templates/python-project.md) | FastAPI / Django / data services |
| [`examples/claude-md-templates/ml-project.md`](examples/claude-md-templates/ml-project.md) | ML training pipelines — data leakage guardrails included |
| [`examples/claude-md-templates/node-project.md`](examples/claude-md-templates/node-project.md) | Node.js / TypeScript services |
| [`examples/claude-md-templates/monorepo.md`](examples/claude-md-templates/monorepo.md) | Multi-service monorepos |

---

## The `.claude/` directory

`.claude/` lives in your project root and contains Claude Code's per-project config. Commit it to version control — it's part of your project's tooling.

```
.claude/
├── settings.json       ← permissions, hooks, model config
├── memory/             ← persistent memory files (updated by Claude via /memory)
│   └── *.md
└── commands/           ← custom slash commands (project-specific)
    └── *.md
```

### settings.json reference

```jsonc
{
  // Model override for this project (optional — uses account default otherwise)
  "model": "claude-sonnet-4-6",

  // Permission mode: "default" | "acceptEdits" | "bypassPermissions" | "plan"
  "defaultPermissionMode": "default",

  // Paths Claude is allowed to write to (allowlist approach)
  "allowedPaths": [
    "src/",
    "tests/",
    "docs/"
  ],

  // Paths Claude can never write to regardless of permission mode
  "deniedPaths": [
    "data/raw/",
    ".env.production",
    "migrations/"
  ],

  // Hook configuration
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",          // Tool name: Bash, Write, Read, MCP, etc.
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/no-prod-writes.sh"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "*",             // Wildcard matches all tools
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/audit-log.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/require-tests.sh"
          }
        ]
      }
    ]
  },

  // MCP servers scoped to this project
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### Memory system

Claude can persist information across sessions via memory files in `.claude/memory/`. You manage these with the `/memory` command inside a session.

```bash
# In a Claude session:
> /memory
# Opens an editor to view/edit persistent memory

# Ask Claude to remember something:
> "Remember that the staging DB password is in 1Password under 'staging-db-write'"
# Claude writes this to .claude/memory/
```

**What to store in memory:**
- Team conventions that aren't in CLAUDE.md yet ("we always use UTC timestamps")
- Recurring context Claude needs ("the auth service owns the /api/auth/* namespace")
- Decisions in progress ("we decided to migrate from Redis to Valkey — track this")

**What NOT to store:** Secrets (memory files are plaintext and often committed), temporary task state, or anything that belongs in CLAUDE.md itself.

### Custom slash commands

Create project-specific commands in `.claude/commands/`:

```markdown
<!-- .claude/commands/review.md -->
# /review

Review the current diff for:
1. Logic errors and missed edge cases
2. Security issues (injection, auth bypass, secrets in code)
3. Missing error handling for external calls
4. Breaking API changes

Post your findings as a numbered list. Flag blockers explicitly.
```

Use it in a session: `> /review`

This is useful for codifying team review checklists or frequent task prompts.

---

## Permission modes

| Mode | File writes | Shell commands | When to use |
|---|---|---|---|
| `default` | Asks | Asks | Interactive development — the safe default |
| `acceptEdits` | Auto-approved | Asks | You've reviewed the plan and trust the writes |
| `bypassPermissions` | Auto-approved | Auto-approved | CI/CD pipelines in sandboxed environments only |
| `plan` | Blocked | Blocked | Preview what Claude intends before committing |

```bash
# Preview before running a risky refactor
claude --permission-mode plan "migrate all v1 API endpoints to v2 format"

# Accept file edits but still approve shell commands
claude --permission-mode acceptEdits "implement the search feature from the spec"

# Fully automated CI (only in sandboxed runners!)
claude --permission-mode bypassPermissions "run tests and fix failures"
```

**Set a project default in `.claude/settings.json`:**
```json
{ "defaultPermissionMode": "acceptEdits" }
```

> ⚠️ `bypassPermissions` in CI should always run in a network/filesystem-restricted environment. Scope it: no write access outside the repo, no outbound network except the Anthropic API.

---

## Referencing context with `@`

The `@` syntax lets you explicitly inject files or symbols into Claude's context without Claude having to search for them. This is faster, more precise, and doesn't burn context on Claude's search overhead.

```bash
# Reference a specific file
claude "refactor the auth logic in @src/auth/jwt.py"

# Reference a directory (Claude reads all files in it)
claude "write tests for everything in @src/services/"

# Reference multiple files
claude "make @src/api/users.py consistent with the pattern in @src/api/orders.py"
```

**In-session:**
```
> @src/models/user.py update the User model to add a `last_login` timestamp field
> @docs/api-spec.md implement the /search endpoint described in section 3
```

**Practical rule:** Use `@` when you know exactly what Claude needs to read. Let Claude search when the relevant files aren't obvious — it's better at codebase traversal than you might expect.

---

## MCP integration

MCP (Model Context Protocol) is how Claude Code talks to external systems. Each MCP server exposes tools (functions Claude can call) and/or resources (data Claude can read).

### Configuring MCP servers

```bash
# Add a server scoped to the current project (.claude/settings.json)
claude mcp add --scope project github npx @modelcontextprotocol/server-github

# Add globally (available in all projects)
claude mcp add --scope user my-server python -m my_mcp_server

# List all configured servers
claude mcp list

# Remove a server
claude mcp remove github

# Check server status
claude mcp get github
```

### Ready-to-use configs in this repo

| Config | Connects to |
|---|---|
| [`examples/mcp-configs/github.json`](examples/mcp-configs/github.json) | GitHub (issues, PRs, repos) |
| [`examples/mcp-configs/google-drive.json`](examples/mcp-configs/google-drive.json) | Google Drive docs and specs |
| [`examples/mcp-configs/slack.json`](examples/mcp-configs/slack.json) | Slack threads for context |
| [`examples/mcp-configs/custom-api.json`](examples/mcp-configs/custom-api.json) | Template for internal APIs |

### Practical MCP workflows

```bash
# With GitHub MCP: Claude can read issues and open PRs natively
claude "Read GitHub issue #234 and implement the described feature"

# With Google Drive MCP: Claude reads your spec docs
claude "Read the PRD titled 'Auth Redesign Q3' in Google Drive and implement the login flow"

# Combine MCP with local work
claude "Check Slack #backend for any discussion about the rate limiter, then review our
        implementation in @src/middleware/rate_limit.py for issues they mentioned"
```

### Building a custom MCP server

If your team has internal tools (ticketing, data catalog, deployment system), an MCP server lets Claude use them natively. See [`examples/mcp-configs/custom-api.json`](examples/mcp-configs/custom-api.json) for the config template and the [MCP quickstart](https://modelcontextprotocol.io/quickstart/server) for implementation.

---

## Hooks

Hooks intercept Claude's tool calls — before (`PreToolUse`) or after (`PostToolUse`). Use them for safety guardrails, audit logging, and enforcing team policies that don't belong in the prompt.

**A hook that exits non-zero blocks the tool call.** This is the mechanism for hard guardrails.

### Hook types

| Hook | When it fires |
|---|---|
| `PreToolUse` | Before any tool call — can block execution |
| `PostToolUse` | After any tool call — observability only |
| `Stop` | When Claude ends a session — good for "did tests pass?" checks |
| `Notification` | On long-running task updates |

### Hook scripts in this repo

| Script | What it does |
|---|---|
| [`examples/hooks/no-prod-writes.sh`](examples/hooks/no-prod-writes.sh) | Blocks writes to `/etc`, `/var`, prod paths |
| [`examples/hooks/audit-log.sh`](examples/hooks/audit-log.sh) | Logs every tool call with session ID and timestamp |
| [`examples/hooks/secrets-scan.sh`](examples/hooks/secrets-scan.sh) | Runs gitleaks before any git commit |
| [`examples/hooks/require-tests.sh`](examples/hooks/require-tests.sh) | Blocks session end if tests are failing |

### Wiring hooks in settings.json

See [settings.json reference](#settingsjson-reference) above for the full hook config schema.

---

## CI/CD automation

Claude Code has a native GitHub Actions integration via `anthropics/claude-code-action@beta`.

### Automated PR review

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review
on:
  pull_request:
    types: [opened, synchronize]
    
permissions:
  pull-requests: write
  contents: read

jobs:
  review:
    runs-on: ubuntu-latest
    if: github.event.pull_request.draft == false
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: anthropics/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          model: claude-sonnet-4-6
          direct_prompt: |
            Review this PR. Focus on: logic errors, security issues (injection, auth bypass,
            secrets in code), missing error handling, and missing tests for changed logic.
            Post inline comments on specific lines. Flag blockers explicitly. Skip style nits.
```

### Headless mode for scripts

Outside of GitHub Actions, you can drive Claude Code headlessly with `--print` (non-interactive, outputs to stdout):

```bash
# Run a task and capture output
RESULT=$(claude --print --permission-mode bypassPermissions "summarize the changes in the last 10 commits")

# Pipe into another tool
claude --print "generate a changelog entry for this release" >> CHANGELOG.md

# Use in a Makefile target
release-notes:
    claude --print "write release notes for $(VERSION) based on commits since $(LAST_TAG)"
```

### Headless with JSON output

```bash
# Output structured JSON for downstream processing
claude --print --output-format json "analyze src/ for unused exports and return as JSON array"
```

More workflows in [`examples/github-actions/`](examples/github-actions/).

---

## Subagents and parallel work

Claude Code can spawn subagents — additional Claude instances that run tasks in parallel. This is useful for large tasks where independent subtasks can run concurrently (e.g., writing tests for multiple modules simultaneously).

### Using the Agent SDK

```bash
npm install @anthropic-ai/claude-code
```

```typescript
import { query } from "@anthropic-ai/claude-code";

// Run a single agentic task programmatically
const result = await query({
  prompt: "Add type hints to all functions in src/utils.py",
  options: {
    permissionMode: "acceptEdits",
    model: "claude-sonnet-4-6",
  }
});

console.log(result.result);  // Task output
```

### Parallel subagents (worktree pattern)

The safe way to run parallel agents on the same codebase is git worktrees — each agent gets an isolated working copy:

```bash
#!/usr/bin/env bash
# Spawn parallel agents on separate modules, then merge
MODULES=("auth" "payments" "notifications")

for module in "${MODULES[@]}"; do
  git worktree add "../worktree-$module" -b "agent/$module"
  (
    cd "../worktree-$module"
    claude --permission-mode bypassPermissions \
      "Write comprehensive tests for src/$module/. Run them and fix any failures."
  ) &
done

wait  # Wait for all agents to complete

# Merge results
for module in "${MODULES[@]}"; do
  git checkout main
  git merge "agent/$module" --no-ff -m "feat(tests): add $module test suite (agent)"
  git worktree remove "../worktree-$module"
done
```

**Warning:** Don't run parallel agents on the same working directory — they'll conflict on file writes. Worktrees or separate repo clones are required.

---

## Model selection and cost

Claude Code uses your account's default model unless overridden. Current models:

| Model | Best for | Cost relative |
|---|---|---|
| `claude-opus-4-6` | Complex architecture decisions, long autonomous sessions, deep reasoning | Highest |
| `claude-sonnet-4-6` | Most coding tasks — the inflection point for production reliability | Medium |
| `claude-haiku-4-5` | High-volume automated tasks, fast iteration on simple changes | Lowest |

**In practice:** Sonnet handles 80%+ of coding tasks well. Use Opus when Claude is making repeated mistakes on a complex task, or for architecture-level decisions where quality matters more than cost. Use Haiku in CI for tasks like linting, changelog generation, or issue triage.

### Override model

```bash
# Per-session
claude --model claude-opus-4-6 "redesign the caching layer architecture"

# Per-project default in .claude/settings.json
{ "model": "claude-sonnet-4-6" }
```

### Prompt caching

Claude Code automatically caches prompts — specifically your CLAUDE.md and frequently referenced files. A cache hit costs ~10% of a full read. **To maximize cache hits:**

- Keep CLAUDE.md stable between sessions (don't edit it mid-conversation)
- Use `@file` references for files you'll access repeatedly in a session
- Cache TTL is 5 minutes, so back-to-back sessions on the same project benefit significantly

### Context window costs

Each file Claude reads burns tokens. Strategies to reduce unnecessary reads:

- `.claueignore` to exclude `node_modules/`, `data/raw/`, build artifacts, binary files
- Subdirectory CLAUDE.md files (only loaded when relevant)
- Explicit `@` references instead of letting Claude search
- `/compact` to summarize long sessions before starting a new subtask

---

## Common mistakes

### 1. Vague prompts on agentic tasks

```bash
# Bad — Claude will interpret "fix bugs" maximally
claude "fix the bugs in the codebase"

# Good — explicit scope and stopping condition
claude "Fix the three failing tests in tests/test_auth.py.
        Run pytest after each fix. Stop after all three pass — do not touch other files."
```

Agentic Claude expands to fill scope. Under-specified tasks lead to unwanted refactors, spurious dependency additions, or changes far outside the intended area.

### 2. Not using plan mode before risky tasks

Before any task that touches many files or runs destructive commands:

```bash
claude --permission-mode plan "migrate all API endpoints from v1 to v2 format"
```

Review what Claude intends to do. Redirect if needed. Then re-run without `plan` mode.

### 3. Expecting cross-session memory without files

If you tell Claude something in one session and start a new session expecting it to remember — it won't. Write important context to CLAUDE.md or use `/memory` to persist it to `.claude/memory/`.

### 4. No `.claueignore` on large repos

Without `.claueignore`, Claude indexes your entire directory. On a monorepo with `node_modules/` or large `data/` directories, you'll burn context and slow down every session. Minimum `.claueignore`:

```
node_modules/
.venv/
dist/
build/
*.lock
data/raw/
**/__pycache__/
*.pyc
```

### 5. Using `bypassPermissions` outside CI

Fully automated mode is for sandboxed CI runners, not local development. Locally, `acceptEdits` is the right balance — Claude writes files freely but still asks before running shell commands.

### 6. Monolithic CLAUDE.md files

A 500-line CLAUDE.md burns context on every session, even when 80% of it isn't relevant. Break it up:

```
CLAUDE.md                    ← project overview, critical rules, global conventions
src/ml/CLAUDE.md             ← ML-specific: feature engineering rules, MLflow config
src/api/CLAUDE.md            ← API-specific: router patterns, auth middleware
src/data/CLAUDE.md           ← Data pipeline: immutable sources, output paths
```

### 7. Fighting Claude on style instead of enforcing it via tools

Don't write CLAUDE.md rules for things your linter already enforces. If ruff or ESLint catches it, let the tools handle it (`make lint` in CLAUDE.md is enough). CLAUDE.md rules should cover things that can't be automated.

---

## Quick reference

### CLI flags

```bash
claude                            # Interactive session
claude "prompt"                   # One-shot (interactive output)
claude --print "prompt"           # Headless / non-interactive (stdout only)
claude --print --output-format json "prompt"  # JSON output for scripting
claude --continue                 # Resume most recent session
claude --resume <id>              # Resume specific session
claude --model claude-opus-4-6    # Override model
claude --permission-mode plan     # Preview only — no writes or execution
claude --no-cache                 # Disable prompt caching
```

### In-session slash commands

```
/plan           Preview planned steps before executing
/compact        Summarize conversation history to free context
/clear          Reset conversation (fresh context)
/status         Show model, session ID, token usage
/memory         View and edit persistent memory
/mcp            List and manage MCP connections
/permissions    Show current permission settings
/review         Run custom review command (if defined in .claude/commands/)
```

### MCP management

```bash
claude mcp list                             # All configured servers
claude mcp add --scope project <name> ...   # Add project-scoped server
claude mcp add --scope user <name> ...      # Add globally
claude mcp remove <name>                    # Remove server
claude mcp get <name>                       # Show server details
```

---

## Resources

- [Official Claude Code docs](https://code.claude.com/docs/en/overview)
- [Claude Code changelog](https://code.claude.com/docs/en/changelog)
- [GitHub Actions integration](https://code.claude.com/docs/en/github-actions)
- [Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview)
- [MCP specification](https://modelcontextprotocol.io)
- [Anthropic API docs](https://platform.claude.com/docs/en/home)
