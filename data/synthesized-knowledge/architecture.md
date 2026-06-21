---
name: architecture
description: Contoso Trading architecture — components, data flow, code-to-infra mapping, error-to-code mapping, OTEL service names
---

## Components

| Component | Language | Runs on | Port | Dependencies |
|-----------|----------|---------|------|--------------|
| Frontend | Node.js (Express) | Container App `frontend-7defkiyvn3r44` | 3000 | Gateway |
| Gateway | .NET 8 (ASP.NET) | Container App `gateway-7defkiyvn3r44` | 8080 | Order Service, Payment Service |
| Order Service | .NET 8 | Container App `order-svc-7defkiyvn3r44` | 8080 | PostgreSQL, Service Bus |
| Payment Service | .NET 8 | Container App `payment-svc-7defkiyvn3r44` | 8080 | PostgreSQL |
| Worker | .NET 8 | Container App `worker-7defkiyvn3r44` | 8080 | Service Bus, PostgreSQL |
| PostgreSQL | Flexible Server | `pg-7defkiyvn3r44` | 5432 | — |
| Service Bus | Standard tier | `sb-7defkiyvn3r44` | — | — |

## Data Flow
```
User → Frontend (Node.js, port 3000)
  → Gateway (.NET, port 8080)
    → GET /api/orders  → Order Service GET /orders → PostgreSQL
    → POST /api/orders → Order Service POST /orders → Service Bus "orders" queue
    → GET /api/payments → Payment Service GET /payments → PostgreSQL
    → POST /api/payments → Payment Service POST /payments/process → (simulated, 5% fail rate)
  Worker (background) → listens on Service Bus "orders" queue → processes orders (max 5 concurrent)
```

## Code → Infra Mapping
- Single repo: `dm-chelupati/contoso-trading` (GitHub, branch `main`)
- IaC: `infra/main.bicep` + 13 modules in `infra/modules/`
- Deploy: `azd up` (no CI/CD pipeline, manual)
- All 5 services in shared Container Apps Environment `env-7defkiyvn3r44`

## OTEL Service Names (for Dynatrace entity resolution)
| Service | OTEL name | Source |
|---------|-----------|--------|
| Frontend | `frontend` | `frontend/tracing.js` |
| Gateway | `gateway` | `gateway/Program.cs` |
| Order Service | `order-service` | `order-service/Program.cs` |
| Payment Service | `payment-service` | `payment-service/Program.cs` |
| Worker | `worker` | `worker/Program.cs` |

Use `get-entity-id` Dynatrace tool with these names to resolve entity IDs for DQL queries.

## Error → Code Mapping

| Error message | HTTP status | Source file | Line |
|--------------|-------------|-------------|------|
| `Database error: {ex.Message}` | 500 | `order-service/Program.cs` | ~64 |
| `Queue error: {ex.Message}` | 500 | `order-service/Program.cs` | ~84 |
| `Payment gateway timeout` | 504 | `payment-service/Program.cs` | ~69 |
| `Database error: {ex.Message}` | 500 | `payment-service/Program.cs` | ~59 |
| `Gateway unreachable` | 502 | `frontend/server.js` | ~20, ~32 |
| `Order service error: {ex.Message}` | 502 | `gateway/Program.cs` | ~59 |
| `Payment service error: {ex.Message}` | 502 | `gateway/Program.cs` | ~75 |
| `Queue error: {ex.Exception.Message}` | stderr | `worker/Program.cs` | ~68 |

## Known Behaviors (not bugs)
- Payment Service has a **5% simulated failure rate** on `POST /payments/process` — returns 504
- Gateway root path `/` returns 404 — normal, no handler registered; Container App health probes hit this
- Order Service returns mock data when `DATABASE_URL` is empty
- Worker only processes queue when `WORKER_MODE=true` env var is set

## VNet Layout (when enableVnet=true)
- `snet-cae` (10.0.0.0/21) — Container Apps Environment
- `snet-pe` (10.0.10.0/24) — PostgreSQL Private Endpoint
- `AzureFirewallSubnet` (10.0.11.0/26) — Azure Firewall egress control
- `snet-sre-agent` (10.0.12.0/28) — SRE Agent VNet connection

## GitHub Actions
- `sre-agent-pr-guard.yml` — fires webhook to SRE Agent on PR open/sync/reopen

## Quick Links
- [Overview](overview.md) — service summary, resource names
- [Logs](logs.md) — Dynatrace first, App Insights secondary
- [Deployment](deployment.md) — azd up, rollback
