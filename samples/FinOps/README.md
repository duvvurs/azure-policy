# FinOps Policy Definitions

> Azure Policy definitions for cloud cost governance. Designed for enterprise FinOps practices managing Azure spend across multi-subscription environments.

## Policies

| Policy | Effect | Purpose |
|--------|--------|---------|
| [`require-cost-centre-tag`](./require-cost-centre-tag.json) | Deny | Blocks resource creation without cost-centre tag |
| [`require-environment-tag`](./require-environment-tag.json) | Audit | Audits resources missing environment classification |
| [`allowed-vm-skus-finops`](./allowed-vm-skus-finops.json) | Deny | Restricts VMs to cost-effective SKUs, blocks premium sizes |
| [`deny-premium-storage-skus`](./deny-premium-storage-skus.json) | Deny | Prevents Premium_ZRS and Ultra SSD unless exempted |
| [`inherit-cost-centre-from-rg`](./inherit-cost-centre-from-rg.json) | Modify | Auto-inherits cost-centre tag from Resource Group |
| [`require-budget-on-subscription`](./require-budget-on-subscription.json) | Audit | Ensures every subscription has at least one cost budget |

## Usage

### Assign at Management Group level

```bash
az policy assignment create \
  --name 'finops-require-cost-centre' \
  --policy '/samples/FinOps/require-cost-centre-tag.json' \
  --scope '/providers/Microsoft.Management/managementGroups/{mgId}' \
  --display-name 'FinOps — Require cost-centre tag'
```

### Assign via Bicep

```bicep
resource policyAssignment 'Microsoft.Authorization/policyAssignments@2023-04-01' = {
  name: 'finops-require-cost-centre'
  properties: {
    policyDefinitionId: requireCostCentrePolicy.id
    enforcementMode: 'Default'
    parameters: {}
  }
}
```

## Rollout Strategy

| Phase | Duration | Effect | Goal |
|-------|----------|--------|------|
| **1. Audit** | Weeks 1-4 | `audit` | Baseline measurement |
| **2. Inherit** | Weeks 5-8 | `modify` | Auto-tag from RG (>85% compliance) |
| **3. Enforce** | Week 13+ | `deny` for new | 100% on new resources |

## Design Principles

1. **Start with audit, escalate to deny** — avoids blocking deployments while building compliance
2. **Inherit from Resource Group** — tag once at RG level, children inherit automatically
3. **Allow exemptions** — use `exemptions` API for legitimate exceptions (e.g., shared platform resources)
4. **Tag regex validation** — cost-centre must match `^[A-Z]{2,4}-[0-9]{3,6}$` format
