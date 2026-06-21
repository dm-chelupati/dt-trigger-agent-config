---
name: logs
description: Observability setup for Contoso Trading — Dynatrace primary, App Insights secondary, DQL patterns
---

## Observability Stack

### Primary: Dynatrace
- All 5 services export traces + metrics + logs via OTLP
- Configured via `DT_OTLP_ENDPOINT` and `DT_OTLP_TOKEN` env vars on each Container App
- Dynatrace MCP connector is available on this agent
- **Start here when investigating issues** — use distributed traces for full request path

### Secondary: Azure App Insights + Log Analytics
- App Insights: `ai-7defkiyvn3r44`
- Log Analytics Workspace: `law-7defkiyvn3r44`
- Connection string injected via `aiConnStr` Bicep param

### Handling Preferences
- **Dynatrace first**: Query Dynatrace MCP for distributed traces showing errors — gives full request path
- **Confirm before acting**: Verify error rates sustained >1 minute before declaring incident
- **Correlate with deployments**: Check `az containerapp revision list` for recent revisions

### DQL Quick Reference
- Use `get-entity-id` MCP tool to resolve service names → entity IDs (never hardcode)
- Error spans: `fetch spans, from:now()-2h | filter dt.entity.service == "<ID>" | filter http.response.status_code >= 500`
- Error rate: `fetch spans | summarize total=count(), errors=countIf(http.response.status_code >= 500), by:{bin(timestamp, 5m)}`
- Logs: `fetch logs | filter dt.entity.service == "<ID>" | filter loglevel == "ERROR"`
- Full DQL reference in repo: docs/dynatrace-dql-reference.md

### Correlation
- Use trace_id / span relationships in Dynatrace for cross-service tracing
- In App Insights: operation_Id for correlation
