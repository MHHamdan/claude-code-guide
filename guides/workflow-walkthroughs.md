# End-to-End Workflow Walkthroughs

Complete, real workflows using Claude Code — not just command lists.
Every step maps to verified official documentation.

Source: [code.claude.com/docs/en/common-workflows](https://code.claude.com/docs/en/common-workflows)
Source: [code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

---

## Workflow 1 — Onboard yourself to an unfamiliar codebase

**Goal:** Understand a codebase you've never seen before, in one session.

The official best practices recommend: explore first, then plan, then code.
Use plan mode for the exploration phase — reads without changing anything.
Source: [code.claude.com/docs/en/best-practices#explore-first-then-plan-then-code](https://code.claude.com/docs/en/best-practices#explore-first-then-plan-then-code)

```bash
# Start in plan mode — read-only, no changes
claude --permission-mode plan
```

**Step 1: High-level orientation**
```
> Read the top-level directory, README, and any CLAUDE.md files.
  Give me a 1-paragraph summary of what this project does, its
  main components, and the tech stack.
```

**Step 2: Entry points and data flow**
```
> Trace the entry point of the application. Follow the main request
  path from the HTTP handler to the database and back. Describe
  each layer and what it does.
```

**Step 3: Architecture decisions**
```
> What architectural decisions stand out? What patterns does the
  codebase use (dependency injection, repository pattern, etc.)?
  What would a new contributor need to know that isn't obvious?
```

**Step 4: Find the rough edges**
```
> Where is the code weakest? Find areas with no tests, TODOs,
  duplicated logic, or inconsistent patterns. List them ranked
  by risk.
```

**Step 5: Ask Claude to write the onboarding doc**
```
> Write an ONBOARDING.md that captures everything we've discussed.
  Include: project overview, architecture, how to run locally,
  where the main logic lives, known rough edges, and gotchas.
  Keep it under 300 lines.
```

Exit plan mode, review the ONBOARDING.md, commit it.

---

## Workflow 2 — Implement a feature from a spec

**Goal:** Build a feature end-to-end — from spec doc to passing tests to PR.

The recommended workflow from the official docs is: Explore → Plan → Implement → Commit.
Source: [code.claude.com/docs/en/best-practices#explore-first-then-plan-then-code](https://code.claude.com/docs/en/best-practices#explore-first-then-plan-then-code)

```bash
git checkout -b feature/my-feature
claude --permission-mode plan
```

**Step 1: Load the spec**
```
> @docs/feature-spec.md
  Read this spec. Summarize what needs to be built in plain terms.
  Ask me about anything that's ambiguous before we plan.
```

**Step 2: Explore relevant code**
```
> Read the files that will need to change for this feature.
  Map out what exists vs what needs to be created.
```

**Step 3: Create a plan**
```
> Write a detailed implementation plan. List:
  1. Files to create
  2. Files to modify and exactly what changes
  3. Tests to write
  4. Order of implementation
```

Press `Ctrl+G` to open the plan in your editor. Review and edit it.
Source: [code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

**Step 4: Switch to acceptEdits and implement**
```bash
# Switch mode with Shift+Tab (or restart with flag)
claude --permission-mode acceptEdits
```
```
> Implement the plan. Write tests first, then the implementation.
  Run the tests after each file. Fix failures before moving on.
  Do not modify files outside the plan.
```

**Step 5: Verify and commit**
```
> Run the full test suite. Fix any failures.
  Run the linter. Fix any errors.
  Summarize the changes you made.
  Create a commit with a conventional commit message.
```

---

## Workflow 3 — Debug a production issue

**Goal:** Given an error or symptom, find root cause and fix it without breaking anything else.

The official docs recommend always addressing root causes, not symptoms.
Source: [code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work](https://code.claude.com/docs/en/best-practices#give-claude-a-way-to-verify-its-work)

```bash
git checkout -b fix/issue-description
claude --permission-mode plan
```

**Step 1: Give Claude the full error context**
```
> This error is happening in production:

  [paste full stack trace or error message]

  Context:
  - It started after deploy X
  - It only happens when Y
  - Frequency: Z

  Do not fix anything yet. Read the relevant code and explain
  what you think is causing this.
```

**Step 2: Validate the diagnosis**
```
> Based on your analysis, what test would reproduce this bug?
  Write it. Run it. Confirm it fails.
```

**Step 3: Fix with verification**
```
> Now fix the root cause. After fixing:
  1. Run the test you wrote — it must pass
  2. Run the full test suite — it must pass
  3. Explain why your fix addresses the root cause,
     not just the symptom
```

**Step 4: Regression check**
```
> Are there other places in the codebase with the same pattern
  that could have the same bug? List them. Fix any you find.
```

---

## Workflow 4 — Weekly maintenance run

**Goal:** Keep a codebase healthy with an automated maintenance session.

This maps to what the official docs call "automate the work you keep putting off."
Source: [code.claude.com/docs/en/overview#what-you-can-do](https://code.claude.com/docs/en/overview#what-you-can-do)

```bash
git checkout -b maintenance/$(date +%Y-%m-%d)
claude --permission-mode acceptEdits
```

Paste this as your prompt:

```
Run a maintenance pass on this codebase. Work through these in order:

1. DEPENDENCIES
   Update all outdated dependencies to their latest minor/patch versions.
   Run tests after updating. Revert any update that breaks tests.

2. DEAD CODE
   Find and remove unused imports, unreachable functions, and commented-out code.
   Run the linter after each removal to confirm nothing broke.

3. TODOs
   List all TODO and FIXME comments in the codebase. For each one:
   - If it's trivial (< 15 min), fix it now
   - If it's complex, create a GitHub issue and replace the comment
     with a reference to the issue number

4. TESTS
   Find any public functions with no test coverage. Write tests for
   the top 5 highest-risk ones (most called, most complex, or near
   critical paths).

5. DOCS
   Check if README.md is up to date with the current setup instructions.
   Update any steps that are wrong or missing.

After each section, run `make test` and confirm it passes before moving on.
At the end, create a single commit titled:
"chore: weekly maintenance (deps, dead code, TODOs, tests, docs)"
```

---

## Workflow 5 — Migrate or refactor a module

**Goal:** Safely refactor or migrate a module without breaking anything.

Always start with a checkpoint and plan mode for risky refactors.

```bash
git add -A && git stash   # safety checkpoint
git checkout -b refactor/module-name
claude --permission-mode plan
```

**Step 1: Map the blast radius**
```
> I want to refactor [module/pattern]. Before touching anything:
  1. Find every file that imports or uses this module
  2. List all public interfaces (functions, classes, types) that
     external code depends on
  3. Identify what can change safely vs what is a breaking change
```

**Step 2: Plan the migration**
```
> Write a step-by-step migration plan that:
  - Keeps tests passing at every step (no red states)
  - Starts with the leaf nodes and works inward
  - Lists each file in order of modification
```

**Step 3: Implement incrementally**

Switch to `acceptEdits` mode. Have Claude work through the plan one file at a time,
running tests after each change:

```
> Implement step 1 of the plan only. Run `make test` after. Stop and 
  report before continuing to step 2.
```

This gives you a checkpoint after each step — you can review, course-correct, or stop.

**Step 4: Final validation**
```
> Run the full test suite. Run the linter. Confirm there are no
  references left to the old pattern. Summarize all changes.
  Create a commit.
```
