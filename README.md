# Long-Running Tasks Skill for Claude Code

A Claude Code skill that provides patterns for sustained multi-session task execution with checkpointing and state recovery.

## Overview

When working on complex features that span multiple context windows, Claude can lose track of progress, miss features, or leave code in broken states. This skill solves these problems through:

- **Two-agent architecture** - Separate initialization and coding phases
- **Progress persistence** - Track work across sessions with structured files
- **Incremental execution** - Complete one feature at a time with verification
- **Git-based recovery** - Frequent commits enable rollback from failed changes

## Installation

Copy the `SKILL.md` file to your project's `.claude/skills/` directory, or add this repository as a git submodule:

```bash
mkdir -p .claude/skills
git submodule add git@github.com:andrueandersoncs/claude-skill-long-tasks.git .claude/skills/long-tasks
```

## How It Works

### Phase 1: Initialization

The first session creates these artifacts before any coding begins:

| File | Purpose |
|------|---------|
| `init.sh` | Script to start the development environment |
| `claude-progress.txt` | Human-readable progress log |
| `feature_list.json` | Comprehensive feature checklist (JSON format) |

### Phase 2: Coding Sessions

Each subsequent session follows this protocol:

1. Read `claude-progress.txt` to understand current state
2. Read `feature_list.json` to identify the next priority
3. Run `init.sh` to start the development environment
4. Work on exactly **one** feature
5. Verify with end-user testing (browser automation preferred)
6. Update progress files and commit changes
7. Exit cleanly with code in a mergeable state

## Key Principles

**One feature at a time** - Prevents context overload and ensures thorough testing.

**Never edit test definitions** - Only change `passes` status after verification. Editing definitions can lead to missing functionality.

**Always verify** - Use browser automation for e2e tests. Never trust "it should work."

**Commit frequently** - Good recovery points enable `git reset` when changes fail.

## When to Use This Skill

- Complex features spanning multiple context windows
- Long implementations requiring progress persistence
- Tasks where verification requires browser automation
- Projects where partial implementations would break the build

## License

MIT
