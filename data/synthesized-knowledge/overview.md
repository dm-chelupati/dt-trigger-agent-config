---
name: overview
description: Top-level service index for Contoso Trading — order processing platform on Azure Container Apps
---

## Contoso Trading
Multi-service order processing platform. 5 Container Apps + PostgreSQL + Service Bus, deployed to `rg-contoso-swe` in subscription `cbf44432-7f45-4906-a85d-d2b14a1e8328`.

### Request Flow
User → Frontend (Node.js) → Gateway (.NET) → Order Service / Payment Service (.NET) → PostgreSQL
Worker (.NET) consumes from Service Bus `orders` queue asynchronously, completes orders in PostgreSQL.

### Glue
- **Code → Infra**: Single repo `dm-chelupati/contoso-trading` → all 5 Container Apps in `rg-contoso-swe`
- **Observability**: Dynatrace first (OTLP export from all services), App Insights/Log Analytics secondary
- **Deploy**: `azd up` with Bicep (infra/main.bicep), optional VNet mode
- **Version**: Check `az containerapp revision list` or container image tags in ACR `acr7defkiyvn3r44`

### Key Resources (suffix: 7defkiyvn3r44)
| Resource | Name |
|----------|------|
| Frontend | frontend-7defkiyvn3r44 |
| Gateway | gateway-7defkiyvn3r44 |
| Order Service | order-svc-7defkiyvn3r44 |
| Payment Service | payment-svc-7defkiyvn3r44 |
| Worker | worker-7defkiyvn3r44 |
| PostgreSQL | pg-7defkiyvn3r44 |
| Service Bus | sb-7defkiyvn3r44 |
| ACR | acr7defkiyvn3r44 |
| Log Analytics | law-7defkiyvn3r44 |
| App Insights | ai-7defkiyvn3r44 |
| Container Env | env-7defkiyvn3r44 |

### Alerts
- `contoso-orders-5xx` — 5xx errors on order service
- `contoso-gateway-4xx` — 4xx errors on gateway

### Quick Links
- [Architecture](architecture.md) — components, data flow, VNet layout
- [Team](team.md) — who knows what
- [Logs](logs.md) — Dynatrace first, App Insights secondary
- [Deployment](deployment.md) — azd up, rollback
- [Debugging](debugging.md) — common failure modes, first checks
