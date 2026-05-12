# Raising GitHub Issues

Detailed reference for creating issues from Sentry analysis.

## Pre-flight

Always search for duplicates first:

```bash
gh issue list --repo $REPO --state all --label "sentry-traced-bug" --search "<keyword>"
gh issue list --repo $REPO --state all --search "<SENTRY-ID>"
```

If a duplicate exists, add a comment with new data instead of creating a new issue.

## Sensitive data

**Never include in issue bodies:**
- API keys, tokens, secrets, passwords
- Real emails, usernames, IPs, device IDs
- Full paths with usernames (`/Users/john/...` -> `~/...`)
- Internal URLs, database connection strings
- PII from Sentry tags, breadcrumbs, or locals

When in doubt, use `<redacted>`.

## Issue structure

1. **Title**: `fix(<scope>): short description` (under 70 chars)
2. **Description**: 1-2 sentences on impact
3. **Sentry links**: issue IDs with URLs and event counts
4. **Error**: full message (scrubbed)
5. **Platform/version/env**: from event tags
6. **Root cause**: file:line references + explanation
7. **Cascade**: what else breaks
8. **Proposed fix**: concrete steps

## Labels

Every Sentry issue gets:
- `sentry-traced-bug` — always
- `bug` — always
- `os:<platform>` — from event OS tags (`os:windows`, `os:macos`, `os:linux`, `os:all`)
- `priority: <level>` — from triage scoring

Create labels that don't exist yet:
```bash
gh label create "sentry-traced-bug" --repo $REPO --description "Bug traced from Sentry" --color "d73a4a" 2>/dev/null
```

## Rules

- One issue per root cause (group related Sentry issues)
- Never update Sentry issue status
- Never assign unless user says to
- Never skip root cause analysis
- Never create issues for noise
