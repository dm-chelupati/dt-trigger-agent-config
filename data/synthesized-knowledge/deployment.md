---
name: deployment
description: Deployment flow for Contoso Trading — azd up with Bicep, no CI/CD pipeline currently
---

## Deployment

### Flow
- **Tool**: Azure Developer CLI (`azd up`)
- **IaC**: Bicep templates in `infra/main.bicep` + modules
- **CI/CD**: None currently — manual `azd up` from local
- **Registry**: ACR `acr7defkiyvn3r44` — images built and pushed by azd

### Modes
- **Standard** (`enableVnet=false`): Public endpoints, simple
- **VNet-integrated** (`enableVnet=true`, default): Private, Azure Firewall egress control

### Key Parameters
- `environmentName` → resource group name (`rg-${environmentName}`)
- `enableDynatrace` → toggles OTLP export to Dynatrace
- `enableVnet` → toggles VNet integration

### Find Current Version
- `az containerapp revision list -g rg-contoso-swe -n <app-name> --subscription cbf44432-7f45-4906-a85d-d2b14a1e8328 -o table`
- Check container image tags in ACR

### Rollback
- Activate previous revision: `az containerapp revision activate`
- Or redeploy with `azd up` after reverting code
