---
name: brb-sentry-analyze
description: Deep-dive into a Sentry bug — read the event, map to source code, identify root cause, assess fix complexity, and document cascade impact.
allowed-tools: Read Grep Glob Agent(*)
argument-hint: "<sentry-issue-id>"
---

# Analyze a Sentry Bug

For each real code bug identified in triage, perform a deep analysis before raising an issue.

## Step 1: Read the Sentry event

```
Use Sentry MCP: search_issue_events
  organizationSlug: $SENTRY_ORG
  issueId: "<issue-id>"
  regionUrl: "https://us.sentry.io"
```

Capture from the event:
- **Full error message** — the complete string, not truncated
- **Stacktrace** — if available
- **Tags**: `os`, `release`, `environment`, `cpu_arch`
- **Platform**: native, javascript, python, etc.
- **User/geo**: rough idea of who's affected
- **Timestamp**: when it happened (correlate with releases)

---

## Step 2: Map to source code

Based on the error message and stacktrace, find the code that produces it.

### Strategy

1. **Search for the error text** — grep the codebase for the unique part of the error message
2. **Follow the stacktrace** — if available, go directly to the file:line
3. **Check error handlers** — search for `error!`, `log::error!`, `console.error`, `throw new Error`, etc.
4. **Check callers** — who calls this function, what happens when it returns an error

Use agents for deep exploration if the codebase is complex.

---

## Step 3: Read the code

Once you find the file, read:
1. **The error site** — the exact line that produces the error
2. **The caller** — who calls this function, what happens when it returns an error
3. **The recovery path** — is there a fallback? Does it crash? Does it retry?
4. **Related code** — similar patterns elsewhere that might have the same bug

---

## Step 4: Identify root cause

Determine **why** the error happens, not just **where**.

Common root cause patterns:

| Symptom | Root cause | Fix pattern |
|---------|-----------|-------------|
| `unknown method` | Caller updated, handler didn't add the method | Add method to handler/schema |
| `unknown param` | Caller sends new field, handler doesn't accept it | Add field to schema |
| `no such column` | Code references DB column that migration didn't create | Add migration or fix query |
| `Failed to parse` | Schema changed incompatibly, or file corrupted | Add fallback/recovery |
| `not found` (internal resource) | Resource destroyed before handler fires | Check existence before operating |
| `port in use` | No retry with next port | Add port retry logic |
| `already active` | No idempotency guard | Check existing before starting new |
| `not initialized` | Called before setup completes | Add initialization guard |

---

## Step 5: Assess fix complexity

| Complexity | Criteria | Action |
|-----------|----------|--------|
| **Auto-fixable** | Clear root cause, localized change, < 50 lines, testable | Raise issue, can be picked up immediately |
| **Needs discussion** | Ambiguous cause, cross-cutting change, design decision | Raise issue with "needs discussion" note |
| **External dependency** | Fix requires changes in another repo/service | Raise issue, tag appropriately |

---

## Step 6: Document the cascade

For critical bugs, trace what else breaks when this error occurs:

```
When <function> fails:
  -> subsystem A falls back to defaults (degraded)
  -> subsystem B never initializes (broken)
  -> subsystem C returns error to UI (broken)
  -> app boots but is non-functional
```

Include this cascade in the GitHub issue — it justifies the priority.

---

## Output

After analysis, you should have:
1. **Sentry issue ID(s)** — grouped by root cause
2. **Total events** — sum across related issues
3. **Error message** — full text
4. **Platform/version/env** — from event tags
5. **Root cause** — with file:line references
6. **Cascade impact** — what else breaks
7. **Fix complexity** — auto-fixable, needs discussion, or external dependency
8. **Proposed fix** — concrete approach with code references

This becomes the body of the GitHub issue in the next step.
