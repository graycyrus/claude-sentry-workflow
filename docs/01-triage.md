# Triage: Categorize Every Issue

Detailed reference for categorizing Sentry issues. **Nothing is skipped** — every issue is categorized and carried forward to analysis and issue creation. Category is metadata that informs analysis and priority, not a filter.

## Infra / network / environment patterns

These often originate outside the code. Tag them `category: infra` and carry them forward — they can still hide missing retries, weak error handling, or a real upstream regression worth tracking.

| Pattern | Category | Why |
|---------|----------|-----|
| `429 Too Many Requests` | Rate limit | Backend rate limiting |
| `504 Gateway Timeout` | Backend infra | Server-side timeout |
| `502 Bad Gateway` | Backend infra | Server unreachable |
| `401 Unauthorized` / `Session expired` | Auth flow | Expected behavior |
| `Budget exceeded` / `Insufficient budget` | User state | User ran out of credits |
| `Connection reset` / `broken pipe` | Network | Transient connectivity |
| `DNS lookup failed` / `No such host` | Network | User offline |
| `Network is unreachable` | Network | User offline |
| `tls handshake eof` | Network | TLS issue |
| `error sending request for url` | Network | External API down |

## Code-bug patterns

Clear code bugs. Tag them `category: code-bug`.

| Pattern | What it means |
|---------|---------------|
| `unknown method` | Caller references a method that doesn't exist |
| `unknown param` | Caller sends a field the handler doesn't accept |
| `no such column` | DB query references a nonexistent column |
| `Failed to parse` | Data corruption with no recovery path |
| `not found` (internal) | Lifecycle bug — resource gone before access |
| `port in use` / `bind failed` | Port collision, no retry |
| `already active` | Missing idempotency guard |
| `not initialized` | Race condition at startup |

## Customizing category patterns

Every project has its own patterns. If you see errors that are consistently a given category for your project, add them to the tables above. The skill will respect project-specific patterns if documented in the project's CLAUDE.md or similar. Use them to tag the category — never to drop an issue.

## Grouping

Multiple Sentry issues often share one root cause. Group them before scoring:

1. Look for the same error message across different code paths
2. Same underlying failure, different callers
3. Sum event counts across the group

## Priority scoring

```
Priority = Events x Blast Radius
```

CRITICAL > HIGH > MEDIUM > LOW

Score every group the same way regardless of category. Infra/network issues are scored on real impact too — a high-volume timeout that blocks core functionality is CRITICAL, while a one-off transient blip is LOW. Category is context, not a discount.

Focus on CRITICAL and HIGH first. MEDIUM and LOW can wait.
