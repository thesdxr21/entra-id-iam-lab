# Lab 02 — RBAC & Privileged Access Management

## Objective
Design and implement a least-privilege RBAC model mirroring 
how a DoD-facing IAM engineer controls access to Azure resources. 
Assign built-in roles at minimum necessary scope, create a custom 
role with surgical permissions, and configure PIM for just-in-time 
privileged access.

## Environment
- Portal: portal.azure.com
- Resource Groups: RG-Production, RG-Development
- Duration: 60 minutes

## What I Configured

### Part 1 — Resource Groups
Created two resource groups simulating environment isolation:

| Resource Group | Region | Tag | Purpose |
|---|---|---|---|
| RG-Production | East US | Environment=Production | Production scope |
| RG-Development | East US | Environment=Development | Development scope |

### Part 2 — Built-In Role Assignments

| User | Role | Scope | Verified |
|---|---|---|---|
| iam.readonly01 | Reader | RG-Production only | Cannot create or delete |
| iam.analyst01 | Contributor | RG-Development only | Cannot access RG-Production |

Both assignments tested in private browser sessions as the 
target user — not as Global Admin.

### Part 3 — Custom RBAC Role
Created **Custom-IAM-SecReader** with surgical permissions:
- `Microsoft.Storage/storageAccounts/read`
- `Microsoft.Storage/storageAccounts/listKeys/action`

Assigned to iam.admin01 scoped to RG-Production. Verified 
delete operations are denied.

### Part 4 — Privileged Identity Management (PIM)
Configured Security Administrator as PIM-eligible for iam.admin01:
- Assignment type: Eligible (not permanent)
- Maximum activation duration: 2 hours
- Require justification: Yes
- Require MFA on activation: Yes

Tested full JIT activation flow as iam.admin01 in private browser.

## Key Findings

**Scope inheritance flows down, never up.** An assignment at 
subscription level grants access to everything below it. Scoping 
to the resource group level is the least-privilege baseline — 
not the subscription.

**Custom roles vs built-in roles.** Built-in roles are blunt 
instruments. A custom role lets you define exactly which actions 
are permitted — nothing else. Essential when a built-in role 
would over-provision.

**Standing privileged access is a finding.** In DoD IL4 
environments, permanent privileged role assignments fail 
compliance review. PIM eligible assignments with JIT activation 
is the architectural remediation.

## Troubleshooting Notes
- Custom roles appear under the **Roles tab**, not Role assignments
- Contributor appears under Privileged administrator roles, 
  not Job function roles
- PIM duration was initially set to one year — corrected to 
  2 hours per least-privilege JIT principles
- Custom role JSON error: `Microsoft.KeyVault/vaults/secrets/list/action` 
  is not a valid control plane action — removed from role definition
- Custom role replication can take up to 10 minutes to propagate

## Compliance Mapping

| Control | Requirement | Implementation |
|---|---|---|
| AC-3 | Access Enforcement | Scoped RBAC assignments per user tier |
| AC-6 | Least Privilege | Custom role, RG-level scope only |
| AC-6(5) | Privileged Accounts | PIM eligible assignments — no standing admin |
| AC-17 | Remote Access | Scope boundaries enforce environment isolation |

## Validation Checklist
- [x] RG-Production and RG-Development created with environment tags
- [x] iam.readonly01 Reader on RG-Production confirmed via private browser
- [x] iam.analyst01 Contributor on RG-Development — no RG-Production access confirmed
- [x] Custom-IAM-SecReader created with specific storage permissions
- [x] Custom role assigned to iam.admin01 scoped to RG-Production
- [x] PIM eligible assignment configured with 2-hour max and MFA requirement
- [x] PIM activation flow tested end-to-end as iam.admin01

## Key Learning
Every role assignment must be defensible in an audit. If you 
cannot explain why a principal needs a specific role at a specific 
scope, the assignment should not exist. PIM converts that 
principle from a policy statement into a technical enforcement 
mechanism — privileged access only exists when it has been 
explicitly justified and is actively needed. In DoD IL4 
environments this is not optional — standing privileged access 
is a compliance finding.
