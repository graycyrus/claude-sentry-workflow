---
name: brb-sentry-raise-issue
description: Create a well-structured GitHub issue from Sentry analysis — with root cause, cascade impact, proposed fix, and proper labels. Scrubs sensitive data automatically.
allowed-tools: Bash(gh *) Bash(git *) Read Grep Glob
---

# Raise a GitHub Issue from Sentry Analysis

## Pre-flight: check for duplicates

Before creating a new issue, search for existing ones:

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
gh issue list --repo $REPO --state open --search "<error keyword>"
gh issue list --repo $REPO --state open --search "<SENTRY-ISSUE-ID>"
```

If a matching issue already exists:
- **Add a comment** with the new Sentry data (event counts, additional affected users)
- **Do NOT create a duplicate**

---

## Sensitive data scrubbing

Before creating the issue, **scrub ALL sensitive information** from the issue body:

- API keys, tokens, secrets, passwords, session JWTs
- Real user emails, usernames, IPs, device IDs
- Full file paths containing usernames (e.g. `/Users/john/...` -> `~/...`)
- Internal backend URLs, database connection strings
- Any PII from Sentry event tags, breadcrumbs, or stacktrace locals

If unsure whether something is sensitive, redact it with `<redacted>`.

---

## Issue format

```bash
gh issue create --repo $REPO \
  --title "fix(<scope>): <short description>" \
  --label "sentry-traced-bug" \
  --label "bug" \
  --label "os:<platform>" \
  --label "category:<code-bug|infra>" \
  --label "priority: <level>" \
  --body "$(cat <<'EOF'
## Bug Report

### Description

<1-2 sentence summary of what's broken and the user impact.>

**Sentry issues:**
- [ISSUE-ID](sentry-url) — <title> (N events)
- [ISSUE-ID](sentry-url) — <related title> (N events)

**Combined impact:** <total events> events across <N> users. <What happens to the user.>

### Error

\`\`\`
<Full error message from Sentry — scrubbed of sensitive data>
\`\`\`

- Platform: <os + arch from tags>
- Version: <release tag from event>
- Environment: production

### Root Cause Analysis

**`<file_path>:<line_range>`** — <what the code does wrong>:

\`\`\`
<relevant code snippet>
\`\`\`

<Explanation of why this fails.>

### Cascade

<What else breaks when this error occurs.>

| Call site | Behavior | File:Line |
|-----------|----------|-----------|
| <subsystem> | <what happens> | `<file>:<line>` |

### Proposed Fix

1. <Concrete fix step with file reference>
2. <Another step>
3. <Test requirement>
EOF
)"
```

---

## Title conventions

Follow the repo's commit message style:

| Type | When |
|------|------|
| `fix(<scope>)` | Bug fix in a specific area |

Keep titles under 70 chars. Use the description for details.

---

## Required labels

### `sentry-traced-bug`

Always applied. Marks that this issue originated from Sentry triage.

### `os:<platform>`

The OS where the bug was observed, from Sentry event `os` / `os.name` tag.

| Sentry tag | GitHub label |
|------------|-------------|
| `windows` | `os:windows` |
| `macOS` / `Mac OS X` | `os:macos` |
| `linux` / `Linux` | `os:linux` |
| Multiple platforms | Add multiple `os:` labels |
| Platform-agnostic | `os:all` |

### `category:<code-bug|infra>`

From triage categorization. `category:code-bug` for code defects; `category:infra` for infrastructure/network/environment issues. Applied to every issue so infra/network bugs stay distinguishable — they are raised, never dropped.

### `priority: <level>`

From triage scoring: `priority: critical`, `priority: high`, `priority: medium`, `priority: low`.

### Labels that don't exist yet

If a label doesn't exist on the repo, create it:

```bash
gh label create "sentry-traced-bug" --repo $REPO --description "Bug traced from Sentry" --color "d73a4a" 2>/dev/null
gh label create "os:windows" --repo $REPO --color "0075ca" 2>/dev/null
gh label create "os:macos" --repo $REPO --color "0075ca" 2>/dev/null
gh label create "os:linux" --repo $REPO --color "0075ca" 2>/dev/null
gh label create "os:all" --repo $REPO --color "0075ca" 2>/dev/null
gh label create "category:code-bug" --repo $REPO --description "Code defect from Sentry triage" --color "5319e7" 2>/dev/null
gh label create "category:infra" --repo $REPO --description "Infra/network/environment issue from Sentry triage" --color "5319e7" 2>/dev/null
```

---

## What NOT to do

- **Do NOT include sensitive information** — see scrubbing rules above
- **Do NOT update Sentry issue status** — no resolving, archiving, or assigning in Sentry
- **Do NOT assign the GitHub issue** unless the user explicitly says to
- **Do NOT drop infra/network issues** — raise them like any other bug, tagged with their `category` label from triage
- **Do NOT create one issue per Sentry issue** — group by root cause
- **Do NOT skip the root cause analysis** — "there's a Sentry error" is not enough context

---

## After creating the issue

1. **Tell the user** — show the issue URL
2. **Move to the next bug** in the priority list
3. **Do NOT start fixing** unless the user asks — this workflow stops at issue creation
