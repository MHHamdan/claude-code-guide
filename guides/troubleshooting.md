# Troubleshooting Claude Code

Common failure modes, root causes, and verified fixes.
All solutions reference the official docs at [code.claude.com/docs](https://code.claude.com/docs/en/overview).

---

## 1. Claude is ignoring my CLAUDE.md instructions

**Symptoms:** Claude repeats mistakes you've already corrected. Rules you wrote aren't being followed.

**Root causes and fixes:**

**A. File is too long.**
The official docs set a target of under 200 lines per CLAUDE.md. Longer files reduce adherence — rules buried deep in a long file get less attention.
→ Split into subdirectory CLAUDE.md files scoped to the relevant module.
Source: [code.claude.com/docs/en/memory#write-effective-instructions](https://code.claude.com/docs/en/memory#write-effective-instructions)

**B. Rules are too vague to act on.**
"Write clean code" is not a rule Claude can verify. "Run `make lint` and fix all errors before completing" is.
→ Rewrite rules to be specific and verifiable.

**C. Contradicting rules.**
If two rules conflict, Claude picks one arbitrarily. Review your CLAUDE.md files and subdirectory files for contradictions.
Source: [code.claude.com/docs/en/memory#write-effective-instructions](https://code.claude.com/docs/en/memory#write-effective-instructions)

**D. Instruction belongs in the prompt, not CLAUDE.md.**
CLAUDE.md is for persistent project context. One-off task instructions belong in the prompt.
→ Keep CLAUDE.md for standing rules. Say task-specific things in your prompt.

**E. Verify the file is actually loading.**
Run `> /status` in a session and check which CLAUDE.md files are listed as loaded.

---

## 2. Context fills up mid-task — Claude starts making mistakes

**Symptoms:** Claude "forgets" earlier instructions, starts hallucinating file paths, or produces output that contradicts what it did earlier in the session.

**Root cause:** The context window is full. Performance degrades as the context fills.
Source: [code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

**Fixes:**

**A. Run `/compact` before starting a new subtask.**
`/compact` summarizes the conversation history, freeing context budget.
```
> /compact
```

**B. Scope the task more tightly.**
Instead of:
```
claude "refactor the entire auth system"
```
Do:
```
claude "refactor src/auth/jwt.py to use the new TokenStore interface in src/auth/store.py"
```

**C. Add a `.claueignore` file** to exclude directories Claude shouldn't index.
See the [.claueignore patterns library](../examples/claueignore-patterns/) in this repo.

**D. Use subdirectory CLAUDE.md files** so module-specific context only loads when needed.

**E. Start a fresh session for a new task.**
Long sessions accumulate context fast. Use `--continue` to resume only when genuinely continuing the same task.

---

## 3. An agentic task went off-rails — Claude changed things I didn't ask for

**Symptoms:** Claude refactored code you didn't ask it to touch, added dependencies, or renamed things outside the task scope.

**Root cause:** Vague task scope. Claude interprets "fix the bug" as license to improve everything it notices.

**Fixes:**

**A. Use plan mode first.**
```bash
claude --permission-mode plan "your task here"
```
Plan mode reads and proposes without executing. Review the plan, redirect if needed, then re-run without `plan` mode.
Source: [code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode](https://code.claude.com/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)

**B. Add explicit scope limits to your prompt.**
```
"Fix the failing test in tests/test_auth.py. 
Do not modify any other file. 
Stop after the test passes."
```

**C. Add "What NOT to do" to your CLAUDE.md.**
```markdown
## What NOT to do
- Do not refactor working code unless explicitly asked
- Do not rename anything outside the files mentioned in the task
```

**D. Use git as a safety net.**
Before any agentic task:
```bash
git add -A && git stash
# or
git checkout -b claude/attempt-$(date +%s)
```
If Claude goes sideways: `git checkout main` undoes everything.

---

## 4. A hook isn't firing

**Symptoms:** Your pre/post-tool hook runs manually but doesn't trigger during Claude Code sessions.

**Fixes:**

**A. Check the hook is executable.**
```bash
chmod +x .claude/hooks/your-hook.sh
```

**B. Check the settings.json syntax.**
JSON syntax errors silently disable hooks. Validate with:
```bash
python3 -m json.tool .claude/settings.json
```

**C. Check the matcher.**
The `matcher` field must exactly match the tool name. Valid tool names include: `Bash`, `Write`, `Read`, `MCP`, `*` (wildcard).
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{"type": "command", "command": ".claude/hooks/no-prod-writes.sh"}]
      }
    ]
  }
}
```

**D. Check the hook exit code.**
A hook that exits non-zero blocks the tool call. A hook that crashes (exits non-zero unexpectedly) also blocks. Add error handling to your hooks.

---

## 5. MCP server not connecting

**Symptoms:** Claude says it can't access a tool you've configured. `claude mcp list` shows the server but it's not working.

**Fixes:**

**A. Check the server is running.**
```bash
claude mcp get <server-name>
```

**B. Verify environment variables are set.**
If your MCP config uses `${ENV_VAR}`, that variable must be in your shell environment when you start Claude Code — not just in your `.env` file.
```bash
export GITHUB_TOKEN=your_token
claude
```

**C. Check the command path.**
`npx` and `python` paths can differ between your shell and Claude Code's environment.
Use absolute paths if `npx` commands aren't found:
```json
{
  "command": "/usr/local/bin/npx",
  "args": ["-y", "@modelcontextprotocol/server-github"]
}
```

**D. Re-add the server.**
```bash
claude mcp remove <name>
claude mcp add --scope project <name> <command>
```

---

## 6. Permission mode isn't behaving as expected

**Current permission modes** (source: [code.claude.com/docs/en/permission-modes](https://code.claude.com/docs/en/permission-modes)):

| Mode | What runs without asking |
|---|---|
| `default` | Reads only |
| `acceptEdits` | Reads, file edits, and specific bash commands: `mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed` |
| `plan` | Reads only (no writes or execution) |
| `auto` | Everything, with background safety checks |
| `dontAsk` | Only pre-approved tools |
| `bypassPermissions` | Everything — isolated environments only |

**Switch modes mid-session:** Press `Shift+Tab` to cycle `default` → `acceptEdits` → `plan`.

**Set a default in `.claude/settings.json`:**
```json
{
  "permissions": {
    "defaultMode": "acceptEdits"
  }
}
```

**Common confusion:** `acceptEdits` is NOT the same as `bypassPermissions`. In `acceptEdits` mode, shell commands beyond the specific approved list (`mkdir`, `touch`, `mv`, `cp`, `rm`, `rmdir`, `sed`) still require approval. Use `bypassPermissions` only in sandboxed CI environments.

---

## 7. Installation or update issues

**Native install (recommended — auto-updates):**
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Homebrew does NOT auto-update.** Run manually:
```bash
brew upgrade claude-code        # stable channel
brew upgrade claude-code@latest # latest channel
```

**WinGet does NOT auto-update:**
```bash
winget upgrade Anthropic.ClaudeCode
```

**Check current version:**
```bash
claude --version
```

Source: [code.claude.com/docs/en/overview#get-started](https://code.claude.com/docs/en/overview#get-started)

---

## 8. Claude keeps asking for permission on the same action

**Cause:** You're in `default` mode and the action isn't in the auto-approved list.

**Options:**

1. Switch to `acceptEdits` mode (press `Shift+Tab` once) — auto-approves file writes and common filesystem commands.
2. Set a persistent default in `.claude/settings.json`:
```json
{"permissions": {"defaultMode": "acceptEdits"}}
```
3. Pre-approve a specific tool in settings using `permissions.allow`.

Source: [code.claude.com/docs/en/permission-modes](https://code.claude.com/docs/en/permission-modes)

---

## Sources

All troubleshooting in this guide is based on:
- [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)
- [code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)
- [code.claude.com/docs/en/permission-modes](https://code.claude.com/docs/en/permission-modes)
- [code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview)
