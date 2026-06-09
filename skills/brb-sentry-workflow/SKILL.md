---
name: brb-sentry-workflow
description: Full Sentry bug triage workflow — fetch errors, categorize by type (including infra/network), analyze root causes, deduplicate against GitHub, and raise well-structured issues for every issue. Orchestrates triage, analysis, and issue creation.
allowed-tools: Bash(git *) Bash(gh *) Read Grep Glob Agent(*) mcp__claude_ai_Sentry__*
argument-hint: "[sentry-project-slug]"
---

# Sentry Bug Crusher: Full Workflow

You are running an AI-assisted Sentry bug triage workflow. Follow every step in order. Do NOT skip steps. Ask the user when instructed to ask.

This workflow ends at **issue creation**. It does NOT fix bugs — it triages, analyzes, and creates well-structured GitHub issues.

## Prerequisites

This workflow requires:
- **Sentry MCP tools** — `find_organizations`, `find_projects`, `search_issues`, `search_issue_events`
- **GitHub CLI** (`gh`) — authenticated and configured
- A Sentry organization with projects configured

## Environment detection

```bash
# Repo (owner/repo)
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null || echo "")

# GitHub username
GH_USER=$(gh api user -q .login 2>/dev/null || echo "")
```

If `REPO` is empty, **STOP and ask the user** — they may need to run `gh auth login`.

---

## Step 1: Connect to Sentry and ask preferences

### 1a. Get the Sentry org

```
Use Sentry MCP: find_organizations
```

If multiple orgs, ask the user which one to use. Save as `SENTRY_ORG`.

### 1b. List available projects

```
Use Sentry MCP: find_projects (organizationSlug: $SENTRY_ORG)
```

### 1c. Ask the user three questions

If `$ARGUMENTS` contains a project slug, use that. Otherwise ask:

1. **"Which project(s) should I pull Sentry issues from?"** — show the project list
2. **"Should I only show issues assigned to you, or all unresolved issues?"** — determines whether to add `assigned:me` to the query

**Wait for answers before continuing.**

---

## Step 2: Fetch unresolved issues

```
Use Sentry MCP: search_issues
  organizationSlug: $SENTRY_ORG
  projectSlug: <user-selected project>
  query: "is:unresolved"  (add "assigned:me" if user requested)
  sortBy: "freq"
  regionUrl: <from find_organizations result>
```

If the user selected multiple projects, run `search_issues` for each in parallel and combine results.

For each issue, capture:
- Issue ID
- Title / error message
- Event count
- Assigned to
- First seen / last seen
- Sentry URL
- **OS / platform** — from event tags (`os`, `os.name`)

---

## Step 3: Triage — categorize and rank

Run `/brb-sentry-triage` to categorize each issue (code-bug vs infra/network), group related issues, filter out existing GitHub issues, and score priority. **No issue is dropped** — infra/network issues are categorized and carried forward like any other bug.

Present the ranked list to the user. **Wait for confirmation before proceeding to analysis.**

---

## Step 4: Analyze each bug

For each issue (starting from highest priority), run `/brb-sentry-analyze`.

This deep-dives into the Sentry event, maps it to source code, identifies root cause, and prepares the issue body.

---

## Step 5: Raise GitHub issue

For each analyzed bug, run `/brb-sentry-raise-issue` to create a well-structured GitHub issue with full Sentry context and root cause analysis.

---

## Step 6: Repeat

Move down the priority list. Process each bug through Steps 4-5 until:
- All CRITICAL and HIGH bugs have GitHub issues
- Or the user says to stop

---

## Quick reference

| Step | Action | Tool |
|------|--------|------|
| 1 | Connect to Sentry, ask project + assignee filter | `find_organizations`, `find_projects` (Sentry MCP) |
| 2 | Fetch unresolved issues | `search_issues` (Sentry MCP) |
| 3 | Triage: categorize, dedup, score (nothing dropped) | `/brb-sentry-triage` |
| 4 | Analyze event + source code | `/brb-sentry-analyze` |
| 5 | Raise GitHub issue | `/brb-sentry-raise-issue` |
| 6 | Repeat | Next bug in priority order |

## Rules

- **Read-only on Sentry** — never resolve, archive, or update Sentry issue status
- **One GitHub issue per root cause** — group related Sentry issues
- **Always search before creating** — `gh issue list --search` to avoid duplicates
- **Link Sentry issues in the body** — so the fixer has full context
- **Include file:line references** — so the fixer knows exactly where to look
- **Scrub sensitive data** — no API keys, tokens, PII, real emails, internal URLs in issue bodies
- **Always add `sentry-traced-bug` label** — marks Sentry-sourced issues
- **Always add `os:<platform>` label** — from event tags
- **Always add `category:<code-bug|infra>` label** — from triage, so infra/network issues stay distinguishable
- **Never drop issues as noise** — infra/network issues are categorized and raised like any other bug
