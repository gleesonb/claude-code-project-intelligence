# Onboarding: New Machine Setup

Get your full Project Intelligence environment running on a new machine in 3 steps.

## Prerequisites

- Claude Code installed and working
- `sqlite3`, `curl`, `jq`, `git` (all ship with macOS/Linux)
- `ANTHROPIC_API_KEY` set in your shell environment

## Step 1: Install the Tool

```bash
git clone https://github.com/theaichimera/claude-code-project-intelligence.git ~/.claude/project-intelligence
cd ~/.claude/project-intelligence
./install.sh
```

This creates the database, registers hooks in `~/.claude/settings.json`, and installs the `/recall`, `/save-skill`, `/progress`, and `/reflect` skills.

## Step 2: Connect Your Knowledge Repo

```bash
pi-knowledge-init git@github.com:you/claude-knowledge.git
```

This clones your Git-backed knowledge repo to `~/.claude/knowledge/`. You immediately get:
- All project skills (synthesized patterns and manually saved insights)
- All progressions (reasoning chains, corrections, open questions)
- Project context files

## Step 3: Import Session History

On your **old machine**, export everything:

```bash
pi-export ~/Desktop/pi-backup.tar.gz
```

Transfer the file to the new machine (USB, AirDrop, scp, cloud drive), then:

```bash
pi-import ~/Desktop/pi-backup.tar.gz
```

This imports:
- All JSONL session archives (organized by project)
- The SQLite database (sessions, summaries, FTS5 search index)
- Duplicates are automatically skipped

After import, `/recall` searches work across your full history from both machines.

## Verify

```bash
# Check session count
pi-query --recent 5

# Search across all history
pi-query "the thing you remember working on"

# Check skills and progressions
pi-progression-status --project <your-project>
```

## What Lives Where

| Data | Location | Syncs via |
|------|----------|-----------|
| Skills + progressions | `~/.claude/knowledge/` | Git (pi-knowledge-init) |
| Session archives (JSONL) | `~/.claude/project-intelligence/archives/` | pi-export / pi-import |
| SQLite database | `~/.claude/memory/episodic.db` | pi-export / pi-import (or regenerate via pi-backfill) |
| The tool itself | `~/.claude/project-intelligence/` | git clone |

## Optional: Generate Missing Summaries

If you imported with `--no-db` or want summaries for sessions that don't have them:

```bash
pi-backfill --retry-summaries
```

This calls the Opus API to generate structured summaries for any sessions missing them (~$0.05-0.15 per session).

## Optional: Export a Single Project

```bash
pi-export --project myapp ~/Desktop/myapp-sessions.tar.gz
```

## Ongoing Sync

Session history does **not** auto-sync between machines. When you want to bring a second machine up to date:

1. On the source machine: `pi-export ~/Desktop/pi-latest.tar.gz`
2. Transfer the file
3. On the target machine: `pi-import ~/Desktop/pi-latest.tar.gz`

Skills and progressions sync automatically via the knowledge repo Git on every session start.
