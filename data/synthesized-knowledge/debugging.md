---
name: debugging
description: Investigation index for Contoso Trading — runbooks, incident patterns, first checks, troubleshooting heuristics
---

## First Checks (any incident)
1. **Pull latest code** — `git pull origin main` on all repo clones before investigating (stale clones miss source-level root causes)
2. **Dynatrace** — query active problems, then error spans and logs for affected service
3. **Azure CLI** — check Container App status, revision list, PostgreSQL state, Service Bus status
4. **Correlate timing** — compare error onset with deployment timestamps (`az containerapp revision list`)
5. **Check dependencies** — PostgreSQL state, Service Bus status, downstream service health
6. **Source-code RCA** — always trace to the code, identify the exact line/commit, and propose a fix via PR

## Incident Patterns

- [PostgreSQL Connection Exhaustion](pg-connection-exhaustion.md) — 53300 errors, connection slots reserved for replication, caused by missing connection pooling + autoscaling

## Investigation Heuristics

### Order Service 500 errors
- Check PostgreSQL state first (`az postgres flexible-server show`)
- Look for error 53300 (connection exhaustion) or connection timeout in Dynatrace logs
- Check order-service replica count vs `max_connections` on PostgreSQL
- Code creates new `NpgsqlConnection` per request — no pooling (known tech debt)

### Gateway 502/500 errors
- Usually propagated from downstream (order-service or payment-service)
- Check downstream service health first before investigating gateway itself

### Payment Service 504 errors
- 5% simulated failure rate is **expected** — not a bug
- Only investigate if rate exceeds ~5% sustained

## Quick Links
- [Overview](overview.md) — service summary, resource names
- [Architecture](architecture.md) — error-to-code mapping
- [Logs](logs.md) — Dynatrace DQL patterns
