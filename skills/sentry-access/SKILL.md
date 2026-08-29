---
name: sentry-access
description: Read and diagnose Sentry issues and events in dev or production. Read-only; never mutates Sentry.
---

# Sentry Access

Use the portable helper; it pins the org/project/environment, reads only
`SENTRY_AUTH_TOKEN` from `packages/backend/.env`, and omits user, request-header,
breadcrumb, and third-party stack data. It never accepts `SENTRY_WEBHOOK_SECRET`
as API authentication and never writes a report file.

| Alias | Sentry project | Required event environment |
|---|---|---|
| `dev` | `willow-dev` | `staging` |
| `prod` | `willow-production` | `production` |

```bash
# Prove both projects are readable; any partial failure exits nonzero.
python3 scripts/agent-access/sentry_access.py preflight

# Sanitized unresolved-issue summaries.
python3 scripts/agent-access/sentry_access.py list prod --period 24h --level error
python3 scripts/agent-access/sentry_access.py list dev --period 7d --limit 50

# Latest sanitized event for a numeric Sentry issue id.
python3 scripts/agent-access/sentry_access.py show prod <issue-id>

# Newest sanitized events for one issue, including shape-checked explore-search
# latency fields when present.
python3 scripts/agent-access/sentry_access.py events prod <issue-id> --limit 20
```

For an unspecified current-issues request, use `prod`, `--period 24h`, and
`--level all`. Supported periods are `1h`, `24h`, `7d`, `14d`, and `30d`; run
the helper with `--help` for the complete bounded interface. Re-read Sentry in
the turn where reporting mutable status, counts, latest events, or releases.

Report the queried project, pinned environment, period, and query time. Keep
issue count (`count`) distinct from affected-user count (`userCount`) and
include the sanitized issue ID or permalink when present. A `401` or `403`
means the token is missing, expired, or lacks scope; never expose its value.

An empty issue list is not proof that the runtime is healthy: confirm the
corresponding Vercel `SENTRY_DSN` is present with `vercel-environment`. The
helper fails closed on unknown targets, periods, levels, issue IDs, API errors,
or an event whose environment does not match the requested target. Do not
bypass it with the legacy report-writing TypeScript client or raw API GETs:
those paths can emit request, user, breadcrumb, or third-party stack data that
the portable helper deliberately excludes.
