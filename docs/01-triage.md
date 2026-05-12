# Triage: Signal vs Noise

Detailed reference for classifying Sentry issues.

## Noise patterns

These are NOT code bugs. Skip them.

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

## Real bug patterns

These ARE code bugs. Keep them.

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

## Customizing noise patterns

Every project has its own noise. If you see patterns that are consistently noise for your project, add them to the noise list. The skill will respect project-specific patterns if documented in the project's CLAUDE.md or similar.

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

Focus on CRITICAL and HIGH first. MEDIUM and LOW can wait.
