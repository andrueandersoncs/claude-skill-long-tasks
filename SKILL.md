---
name: executing-long-running-tasks
description: Patterns for sustained multi-session task execution with checkpointing and state recovery. Use when working on complex features spanning multiple context windows, long implementations, or tasks requiring progress persistence.
---

# Executing Long-Running Tasks

Patterns for tasks that exceed a single context window.

## Two-Agent Architecture

### Initializer Agent (First Session Only)

Creates three artifacts before any coding begins:

1. **`init.sh`** - Script to start development environment
2. **`claude-progress.txt`** - Progress documentation file
3. **`feature_list.json`** - Comprehensive feature checklist (start all as `"passes": false`)
4. **Initial git commit** showing added files

### Coding Agent (All Subsequent Sessions)

- Makes incremental progress on single features
- Leaves environment in production-ready state
- Updates progress tracking artifacts

## Required Files

### Startup Script (`init.sh`)

```bash
#!/bin/bash
# Start development server
cd /path/to/project
npm install
npm run dev
```

Read this at session start to understand how to run the project.

### Progress Log (`claude-progress.txt`)

```
## Session 2024-01-15

### Completed
- Implemented user authentication endpoint
- Added JWT token generation

### In Progress
- Password reset flow (blocked: need email service config)

### Next Steps
- Complete password reset
- Add rate limiting
```

Update after each significant change. Read this first when resuming.

### Feature Checklist (`feature_list.json`)

Use JSON (more resistant to model corruption than Markdown):

```json
{
  "features": [
    {
      "category": "functional",
      "description": "New chat button creates fresh conversation",
      "steps": [
        "Navigate to main interface",
        "Click 'New Chat' button",
        "Verify new conversation created",
        "Check chat area shows welcome state",
        "Verify conversation appears in sidebar"
      ],
      "passes": false
    }
  ]
}
```

**Critical rule: Never remove or edit test definitions.** This could lead to missing or buggy functionality. Only change `passes` status after verification.

## Session Startup Protocol

Execute this sequence at every session start:

```
[Read] claude-progress.txt     # Understand current state
[Read] feature_list.json       # Identify next priority
[Bash] pwd                     # Verify working directory
[Bash] git log --oneline -20   # Review recent work
[Bash] ./init.sh               # Start development server
[Test] Basic e2e verification  # Confirm nothing is broken
[Pick] ONE failing feature     # Begin work
```

## Incremental Execution

Work on exactly one feature per iteration:

1. Implement the feature
2. Test as an end user would (browser automation, not just curl/unit tests)
3. Only mark `"passes": true` after thorough verification
4. Commit with descriptive message
5. Update `claude-progress.txt`
6. If context allows, pick next feature; otherwise exit cleanly

## Clean Exit Protocol

Before ending a session:

```bash
git add -A
git commit -m "feat(scope): description of changes"
```

Then update `claude-progress.txt` with:
- What was completed
- Any blockers or issues encountered
- Recommended next steps

Leave code in a mergeable state (no broken builds, no partial implementations).

## Git Recovery

Use version control to recover from failed changes:

```bash
# Revert to last working state
git checkout -- .

# Or reset to specific commit
git reset --hard <commit-hash>
```

Commit frequently so you have good recovery points.

## Testing Strategy

Agents often fail at self-verification. Mitigate by:

- Using browser automation (Puppeteer MCP, Playwright) for e2e tests
- Testing as end users would, not just API calls
- Requiring passing e2e tests before marking features complete
- Never trusting "it should work" - always verify

**Limitation**: Browser automation cannot interact with native alert/confirm modals.

## Failure Modes

| Problem | Initializer Fix | Coding Agent Fix |
|---------|-----------------|------------------|
| Premature completion | Create comprehensive feature list (100+ items), all marked failing | Read feature list; work on one feature only |
| Buggy/undocumented progress | Create git repo + progress file | Read progress + git log at start; commit + update at end |
| Features marked done without testing | Establish feature list with test steps | Self-verify all features before marking passing |
| Time wasted on setup | Write `init.sh` script | Read `init.sh` at session start |
| Failed changes break project | Commit working states frequently | Use `git checkout` or `git reset` to recover |
