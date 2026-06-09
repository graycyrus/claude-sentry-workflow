# Full Workflow: Sentry Bug Triage

This is the complete ordered flow for triaging Sentry errors, analyzing root causes, and raising GitHub issues. The workflow ends at **issue creation**.

---

## Step 1: Connect to Sentry

Get the Sentry organization and list available projects.

```
Use Sentry MCP: find_organizations
Use Sentry MCP: find_projects (organizationSlug: <org>)
```

**Ask the user:**
1. Which project(s) to pull issues from
2. Whether to filter by assignee (all issues vs only assigned to them)

---

## Step 2: Fetch unresolved issues

Pull unresolved issues from the selected project(s), sorted by frequency.

```
Use Sentry MCP: search_issues
  organizationSlug: <org>
  projectSlug: <project>
  query: "is:unresolved"  (+ "assigned:me" if requested)
  sortBy: "freq"
```

For multiple projects, run in parallel and combine results.

Capture: issue ID, title, event count, assignee, first/last seen, URL, OS/platform.

---

## Step 3: Triage

### 3a. Categorize each issue

Tag each issue by category — `infra` (5xx, rate limits, network errors, user env issues) or `code-bug` (unknown methods, parse failures, race conditions, etc.). **Nothing is dropped**: infra/network issues are categorized and carried forward, then raised as issues like any other bug. Category is metadata, not a filter.

### 3b. Group by root cause

Multiple Sentry issues can share one root cause. Group them and sum event counts.

### 3c. Filter out existing GitHub issues

Search GitHub for each bug group:

```bash
gh issue list --repo $REPO --state all --label "sentry-traced-bug" --search "<error keyword>"
```

- **Open issue exists** -> skip (already tracked)
- **Closed issue + still happening** -> keep (regression)
- **No issue** -> keep

### 3d. Score priority

```
Priority = Events x Blast Radius
```

| Blast radius | Priority |
|-------------|----------|
| Blocks boot / startup | CRITICAL |
| Breaks core functionality | CRITICAL |
| Breaks a specific feature | HIGH |
| Data corruption / loss | HIGH |
| Edge case / race condition | MEDIUM |
| Single occurrence / niche | LOW |

### 3e. Present ranked list

Show a table per priority level. Wait for user confirmation before analysis.

---

## Step 4: Analyze each bug

For each issue (highest priority first):

1. **Read the Sentry event** — full stacktrace, tags, platform, version
2. **Map to source code** — find the file/function that produced the error
3. **Read the code** — error site, callers, recovery path
4. **Identify root cause** — why it fails, not just where
5. **Assess fix complexity** — auto-fixable vs needs discussion
6. **Document cascade** — what else breaks

---

## Step 5: Raise GitHub issue

Create a well-structured issue with:
- Sentry issue links + event counts
- Error message (scrubbed of sensitive data)
- Platform/version/environment
- Root cause analysis with file:line references
- Cascade impact
- Proposed fix approach

Labels: `sentry-traced-bug`, `os:<platform>`, `category:<code-bug|infra>`, `priority: <level>`, `bug`

---

## Step 6: Repeat

Process each bug through Steps 4-5 until all CRITICAL and HIGH bugs have issues, or the user says to stop.

---

## Quick reference

| Step | Action | Tool |
|------|--------|------|
| 1 | Connect to Sentry, ask preferences | `find_organizations`, `find_projects` |
| 2 | Fetch unresolved issues | `search_issues` |
| 3a | Categorize (code-bug vs infra), nothing dropped | Manual |
| 3b | Group by root cause | Manual |
| 3c | Dedup against GitHub | `gh issue list --search` |
| 3d | Score priority | Events x Blast Radius |
| 4 | Analyze root cause | `search_issue_events` + `Read`/`Grep` |
| 5 | Raise GitHub issue | `gh issue create` |
| 6 | Repeat | Next in priority |

## Rules

- **Read-only on Sentry** — never resolve, archive, or update Sentry issue status
- **One GitHub issue per root cause** — group related Sentry issues
- **Always search before creating** — avoid duplicates
- **Scrub sensitive data** — no tokens, PII, internal URLs in issue bodies
- **Always add `sentry-traced-bug` label** — marks Sentry-sourced issues
- **Always add `os:<platform>` label** — from event tags
- **Always add `category:<code-bug|infra>` label** — so infra/network issues stay distinguishable
- **Never drop issues as noise** — categorize infra/network and raise them like any other bug
