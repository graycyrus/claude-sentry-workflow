---
name: brb-sentry-triage
description: Classify Sentry issues as noise or real bugs, group by root cause, filter out existing GitHub issues, and score priority.
allowed-tools: Bash(gh *) Read Grep Glob
---

# Triage Sentry Issues

You have a list of Sentry issues. Your job is to classify them, group them, deduplicate against GitHub, and rank by priority.

## Step 1: Classify noise vs. signal

Go through each issue and tag it as **noise** or **real bug**.

### Noise patterns — skip these entirely

| Pattern | Category |
|---------|----------|
| `429 Too Many Requests` | Rate limit (backend) |
| `504 Gateway Timeout` | Backend infra |
| `502 Bad Gateway` | Backend infra |
| `401 Unauthorized` / `Session expired` | Expected auth flow |
| `Budget exceeded` / `Insufficient budget` | User out of credits |
| `Connection reset` / `broken pipe` / `os error 54` / `os error 10054` | Transient network |
| `DNS lookup failed` / `No such host` / `os error 11001` | User offline |
| `Network is unreachable` / `os error 51` | User offline |
| `tls handshake eof` | Network/TLS issue |
| `error sending request for url` (to external APIs) | Network/API down |

These are infrastructure, network, or expected-behavior issues — not code bugs.

### Real bug patterns — keep these

| Pattern | What it means |
|---------|---------------|
| `unknown method` | Code calls a method that doesn't exist |
| `unknown param` | Code sends a parameter the handler doesn't accept |
| `no such column` | DB query references a column that doesn't exist (missing migration) |
| `Failed to parse` | Config/data corruption with no recovery |
| `not found` (for internal resources) | Lifecycle bug — resource destroyed before access |
| `port in use` / `bind failed` | Port collision with no retry |
| `already active` / `already exists` | Missing idempotency guard |
| `not initialized` | Race condition — called before setup completes |

**Project-specific noise patterns**: If the project has known noise patterns beyond the defaults above, the user or project docs should provide them. Ask if unsure.

---

## Step 2: Group related issues

Multiple Sentry issues can share the same root cause. Group them:

```
Example:
  Issue A — config parse failed (boot)
  Issue B — config parse failed (runtime)
  Issue C — RPC invoke failed (same config error, 314 events)

→ One root cause: config parse failure
→ One GitHub issue covers all 3 Sentry issues
```

Sum event counts across all issues in the group.

---

## Step 3: Filter out bugs that already have GitHub issues

Before scoring, check if a GitHub issue already exists for each group. Search by error keywords and Sentry issue IDs:

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
gh issue list --repo $REPO --state all --label "sentry-traced-bug" --search "<error keyword>"
gh issue list --repo $REPO --state all --search "<SENTRY-ISSUE-ID>"
```

For each group:
- **Open issue exists** → remove from the list (already tracked)
- **Closed/resolved issue exists** → check if the Sentry bug is still happening (new events after the fix). If yes, keep it — it's a **regression**. If no, remove it.
- **No issue exists** → keep it, proceed to scoring

This prevents duplicate issue creation and saves analysis time.

---

## Step 4: Score priority

For each remaining group (or standalone bug):

1. **Sum events** across all related Sentry issues
2. **Assess blast radius**:

| Blast radius | Priority |
|-------------|----------|
| Blocks boot / app startup | CRITICAL |
| Breaks core functionality | CRITICAL |
| Breaks a specific feature | HIGH |
| Data corruption / loss | HIGH |
| Edge case / race condition | MEDIUM |
| Single occurrence / niche scenario | LOW |

3. **Final priority** = Events x Blast Radius

---

## Step 5: Present the ranked list

Show a clean table per priority level:

```markdown
## CRITICAL

| # | Issue(s) | Error | Total Events |
|---|----------|-------|-------------|
| 1 | A, B, C | Config parse — blocks boot | 320+ |

## HIGH

| # | Issue(s) | Error | Total Events |
|---|----------|-------|-------------|
| 2 | D | unknown param 'api_key' | 57 |
```

**Wait for user to confirm before proceeding to analysis.**
