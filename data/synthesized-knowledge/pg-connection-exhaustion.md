---
name: pg-connection-exhaustion
description: Incident pattern — PostgreSQL 53300 connection exhaustion caused by order-service autoscaling without connection pooling
---

## Incident: INC0010009 (2026-06-17)

### Symptoms
- All `GET /orders` returning HTTP 500
- Gateway `/api/orders` propagating 500 upstream
- Error message: `53300: remaining connection slots are reserved for azure replication users`
- PostgreSQL in `Restarting` state

### Root Cause
Mismatch between Container App autoscaling and PostgreSQL connection capacity:
- **order-service** creates `new NpgsqlConnection(dbConn)` per request (no pooling)
- Revision scaled to **20 replicas** (RunningAtMaxScale)
- PostgreSQL `max_connections = 50` on `Standard_B1ms` Burstable SKU
- 20 replicas × concurrent requests > 50 connection limit → exhaustion

### Key Evidence Sources
- **Dynatrace logs**: `fetch logs | filter dt.entity.service == "SERVICE-98A1541771EE582C" | filter loglevel == "ERROR"` → look for error 53300
- **Azure CLI**: `az postgres flexible-server show` → check `state` field
- **Azure CLI**: `az postgres flexible-server parameter show --name max_connections` → confirm limit
- **Azure CLI**: `az containerapp revision show` → check `replicas` count
- **Dynatrace spans**: `fetch spans | filter http.response.status_code >= 500` → all on `/orders` route

### Error Onset Correlation
- Zero errors for 6 hours prior
- Errors started exactly when new revision deployed and scaled to 20 replicas
- Request volume surged from ~120/10min to 2081/10min at error onset

### Mitigation Steps
1. Scale down replicas: `az containerapp update --min-replicas 1 --max-replicas 3`
2. Wait for PostgreSQL to finish restarting
3. Implement `NpgsqlDataSource` (singleton) for connection pooling
4. Set `Maximum Pool Size` per replica in connection string

### Prevention
- Upgrade PostgreSQL SKU (Standard_B2s or General Purpose)
- Cap Container App max replicas relative to DB capacity
- Add PgBouncer for connection multiplexing
- Add health check that validates DB connectivity

### GitHub Issue
- [#77](https://github.com/dm-chelupati/contoso-trading/issues/77)

### Affected Code
- `order-service/Program.cs` line 58-66 (GET /orders handler, NpgsqlConnection per request)
- `payment-service/Program.cs` has same pattern (tech debt)
- `worker/Program.cs` has same pattern (tech debt)
- `infra/modules/database.bicep` — SKU hardcoded to Standard_B1ms
