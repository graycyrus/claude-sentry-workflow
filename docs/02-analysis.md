# Analysis: Root Cause Investigation

Detailed reference for analyzing Sentry bugs.

## Reading the event

Use `search_issue_events` to get the full event data. Capture:

- **Full error message** — not truncated
- **Stacktrace** — if available
- **Tags**: os, release, environment, architecture
- **Platform**: what runtime produced the error
- **Timestamp**: correlate with releases

## Mapping to source

### Strategy

1. **Grep for the error text** — search for the unique part of the error message
2. **Follow the stacktrace** — go directly to file:line if available
3. **Check error emission points** — `error!`, `log::error!`, `console.error`, `throw new Error`
4. **Trace callers** — what calls this function, what happens on error

### Reading the code

At the error site, understand:
1. The exact line that produces the error
2. Who calls this, what happens when it fails
3. Is there a fallback/recovery?
4. Similar patterns elsewhere with the same bug

## Root cause patterns

| Symptom | Likely cause | Fix pattern |
|---------|-------------|-------------|
| `unknown method` | Caller updated, handler didn't | Add method to handler |
| `unknown param` | Caller sends new field | Add field to schema |
| `no such column` | Missing migration | Add migration |
| `Failed to parse` | Incompatible schema change | Add fallback/recovery |
| `not found` (internal) | Resource destroyed early | Check existence first |
| `port in use` | No retry | Add port retry |
| `already active` | No guard | Check existing first |
| `not initialized` | Race condition | Add init guard |

## Fix complexity

| Level | Criteria | Action |
|-------|----------|--------|
| Auto-fixable | Clear cause, < 50 lines, testable | Issue can be picked up immediately |
| Needs discussion | Ambiguous, cross-cutting, design decision | Note in issue |
| External dependency | Fix is in another repo/service | Tag appropriately |

## Cascade analysis

For critical bugs, trace downstream impact:

```
When X fails:
  -> Y falls back to defaults (degraded)
  -> Z never initializes (broken)
  -> App boots but is non-functional
```

Include this in the issue body — it justifies the priority.
