# ak-base

A base project template by Aivokone.

## What's Included

- **README.md** - This file. Instructions and suggestions to set up a project.
- **AGENTS.md** - Workflow instructions for all AI agents
- **CLAUDE.md** - Claude-specific configuration (references AGENTS.md)

## Usage

Use this repo as a GitHub template.

```bash
gh repo create my-project --template tkfin/ak-base --private --clone
```

## Beads Issue Tracker

It is recommended to use Beads as a memory and version tracker for agents: https://github.com/steveyegge/beads

Pre-requisites:

```bash
brew install beads
```

Upgrade if needed:

```bash
brew upgrade beads
```

After creating a project from this template:

```bash
bd init --branch beads-sync
```

If init asks if you're contributing to someone elses project, answer NO.

```bash
bd setup codex
bd setup claude
bd hooks install
bd migrate sync beads-sync
bd ready
```

For enhanced UX with slash commands in Claude Code:

```bash
# In Claude Code
/plugin marketplace add steveyegge/beads
/plugin install beads
# Restart Claude Code
```

The plugin adds:
- Slash commands: `/beads:ready`, `/beads:create`, `/beads:show`, `/beads:update`, `/beads:close`, etc.
- Task agent for autonomous execution

After installing, verify `bd` is working:

```bash
bd version
bd doctor
bd help
```


Notes:
- Confirm `.beads/.gitignore` exists and **do not** add `.beads/` to the root `.gitignore`.
- Then use Beads normally (e.g., `bd ready`, `bd create`, `bd list`).

Installing for VS Code et al.: https://github.com/steveyegge/beads/blob/main/docs/INSTALLING.md

## Getting Started (Beads)

```bash
bd ready   # Find available work
bd create  # Create new issue
bd list    # View issues
```
