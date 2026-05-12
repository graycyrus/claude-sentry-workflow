# claude-sentry-workflow

AI-assisted Sentry bug triage workflow for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Fetch errors from Sentry, classify signal vs noise, analyze root causes, and raise well-structured GitHub issues — all with structured agent orchestration.

Companion to [claude-workflow](https://github.com/graycyrus/claude-workflow). That workflow picks up GitHub issues and ships PRs. This one **creates** those issues from Sentry errors.

## What is this?

A set of Claude Code **skills** (slash commands) that give Claude a structured process for triaging Sentry bugs:

### Workflow skills

| Skill | What it does |
|---|---|
| `/brb-sentry-workflow` | Full orchestrator — Sentry fetch to GitHub issue |
| `/brb-sentry-triage` | Classify noise vs real bugs, group by root cause, deduplicate, score priority |
| `/brb-sentry-analyze` | Deep-dive into a bug — map to source, identify root cause, assess complexity |
| `/brb-sentry-raise-issue` | Create a well-structured GitHub issue with full Sentry context |
| `/brb-sentry-update` | Pull latest skills from GitHub and update your install |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated
- **Sentry MCP tools** configured in Claude Code (provides `find_organizations`, `find_projects`, `search_issues`, `search_issue_events`)

## Install

### Option A: Interactive installer

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/graycyrus/claude-sentry-workflow/main/install.sh)
```

Choose between:
1. **Copy — Global** (`~/.claude/skills/`) — works everywhere, update via `/brb-sentry-update`
2. **Copy — Project** (`.claude/skills/`) — per-project, update via `/brb-sentry-update`
3. **Symlink — Global** — clones to `~/.claude/claude-sentry-workflow/`, symlinks skills. `git pull` = instant updates.

### Option B: One-liner (global copy, non-interactive)

```bash
curl -fsSL https://raw.githubusercontent.com/graycyrus/claude-sentry-workflow/main/install.sh | bash
```

### Option C: Manual

```bash
git clone https://github.com/graycyrus/claude-sentry-workflow.git /tmp/claude-sentry-workflow
mkdir -p ~/.claude/skills
cp -r /tmp/claude-sentry-workflow/skills/* ~/.claude/skills/
rm -rf /tmp/claude-sentry-workflow
```

## Usage

```
# Full workflow — fetches from Sentry, triages, analyzes, raises issues
/brb-sentry-workflow

# Full workflow for a specific Sentry project
/brb-sentry-workflow openhuman-tauri

# Individual steps
/brb-sentry-triage
/brb-sentry-analyze ISSUE-ID-123
/brb-sentry-raise-issue
```

### Typical session

```
You: /brb-sentry-workflow
Claude: [connects to Sentry, finds org]
        "I found 4 projects. Which one(s) should I pull issues from?"
You: openhuman-tauri and openhuman-react
Claude: "Should I show only issues assigned to you, or all unresolved?"
You: all
Claude: [fetches 47 unresolved issues]
        [classifies: 31 noise, 16 real bugs]
        [groups into 9 root causes]
        [checks GitHub — 3 already have issues]
        [scores remaining 6]

        "Here are the bugs ranked by priority:

        CRITICAL
        | # | Issues | Error | Events |
        |---|--------|-------|--------|
        | 1 | BM, BK, BJ | Config TOML parse — blocks boot | 320+ |

        HIGH
        | # | Issues | Error | Events |
        |---|--------|-------|--------|
        | 2 | 20 | unknown param 'api_key' | 57 |

        Proceed with analysis?"
You: yes
Claude: [reads Sentry events, maps to source code]
        [identifies root cause, traces cascade]
        [creates GitHub issue with full context]
        "Created: fix(config): handle corrupted TOML — #234"
```

## How it works

The workflow follows a structured pipeline:

1. **Fetch** — Pull unresolved Sentry issues via MCP tools
2. **Triage** — Classify noise vs real bugs using pattern matching
3. **Group** — Combine related issues sharing the same root cause
4. **Deduplicate** — Filter out bugs that already have GitHub issues
5. **Score** — Rank by Events x Blast Radius
6. **Analyze** — Deep-dive into source code, identify root cause
7. **Raise** — Create GitHub issues with full context, scrubbed of sensitive data

## Philosophy

- **Ask before fetching.** Always ask which project and assignee filter.
- **Signal over noise.** Most Sentry issues are infrastructure/network noise. The workflow filters aggressively.
- **One issue per root cause.** Group related Sentry issues — don't create duplicates.
- **Deduplicate against GitHub.** Check existing issues before creating new ones.
- **Scrub sensitive data.** No tokens, PII, or internal URLs in public issue bodies.
- **Read-only on Sentry.** Never resolve, archive, or update Sentry issue status.
- **Stop at issue creation.** This workflow creates issues. Use [claude-workflow](https://github.com/graycyrus/claude-workflow) to fix them.

## Updating

### If you installed via copy

Run `/brb-sentry-update` inside Claude Code.

### If you installed via symlink

```bash
cd ~/.claude/claude-sentry-workflow && git pull
```

## Docs

Detailed reference docs in [`docs/`](docs/):

- [Full workflow](docs/00-full-workflow.md) — complete step-by-step
- [Triage](docs/01-triage.md) — noise patterns, bug patterns, priority scoring
- [Analysis](docs/02-analysis.md) — root cause investigation
- [Raising issues](docs/03-raising-issues.md) — issue format, labels, sensitive data

## Uninstall

```bash
# Global
rm -rf ~/.claude/skills/brb-sentry-{workflow,triage,analyze,raise-issue,update}

# Per-project
rm -rf .claude/skills/brb-sentry-{workflow,triage,analyze,raise-issue,update}

# If symlink install, also remove the cloned repo
rm -rf ~/.claude/claude-sentry-workflow
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs welcome.

## License

MIT
