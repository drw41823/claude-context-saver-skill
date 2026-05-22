# Context Saver Skill

A Claude Code skill for saving and loading experiment context snapshots, designed to handle long-running research sessions that exceed context window limits.

## Features

- **Save Archive** — Snapshot current experiment state (configs, results, scripts, known issues, next steps)
- **Load Archive** — Select and restore a previous snapshot to continue work in a new session
- **List Archives** — View all saved snapshots with summaries
- **Delete Archive** — Remove outdated snapshots

## Installation

```bash
# Via npx
npx skills add --path <path-to-this-repo>
```

Or copy manually to `~/.claude/skills/context-saver/`.

## Usage

Trigger the skill in Claude Code with these commands:

| Command | Action |
|---------|--------|
| `save-archive` / `存档` | Save a snapshot |
| `load-archive` / `读档` | Load an archive |
| `list-archives` / `存档列表` | List all archives |
| `delete-archive` / `删档` | Delete an archive |

Archives are stored in `archives/` under the project root.

## File Structure

```
context-saver/
├── SKILL.md    — Skill definition with full workflow and templates
└── README.md   — This file
```

## Snapshot Contents

Each archive includes up to 14 sections:
1. Experiment config (model, LoRA, training hyperparameters)
2. Environment setup (conda, GPU, env vars)
3. External models & API configuration
4. Config parameter guide
5. Experiment results summary
6. Current operation state
7. Errors & known issues
8. Pending tasks / next steps
9. Key script index
10. Data pipeline documentation
11. Experiment directory structure
12. Persistent memory (CLAUDE.md / MEMORY.md)
13. Recent operation log
14. Full experiment context
