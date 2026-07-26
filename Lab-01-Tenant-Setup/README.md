# Lab 01 — Entra ID Tenant Setup & Identity Governance

## Objective
Stand up a fully configured Entra ID environment mirroring an 
enterprise IAM framework. Configure tenant security settings, 
establish a user hierarchy, build security groups including 
dynamic membership, and implement Entitlement Management for 
governed access lifecycle management.

## Environment
- Tenant: [REDACTED].onmicrosoft.com
- Portal: entra.microsoft.com
- Duration: 45 minutes

## What I Configured

### Part 1 — Tenant Hardening
- Disabled user application registration to enforce least privilege
- Restricted guest invite permissions to admin roles only
- Configured company branding to simulate enterprise tenant

### Part 2 — User Hierarchy
Created three users representing distinct access tiers:

| User | Display Name | Access Tier | Purpose |
|---|---|---|---|
| iam.admin01 | IAM-Admin-01 | Privileged | PIM-eligible administrative account |
| iam.analyst01 | IAM-Analyst-01 | Standard | Analyst access, OIDC test user |
| iam.readonly01 | IAM-ReadOnly-01 | Read-Only | Monitoring and audit access |

### Part 3 — Security Groups
- SG-IAM-Admins — Assigned membership, contains iam.admin01
- SG-IAM-Analysts — Assigned membership, contains iam.analyst01
- SG-Dynamic-CyberTeam — Dynamic membership rule: 
  `(user.department -eq 'Cybersecurity')`

Dynamic group auto-populated all three users within 5 minutes 
of rule creation.

### Part 4 — Entitlement Management
Created access package: **Cybersecurity-Analyst-Access**
- Resource: SG-IAM-Analysts group membership
- Approval: Required, single-stage
- Expiration: 90 days
- Access reviews: Quarterly

## Key Findings

**Dynamic groups eliminate manual provisioning.** In enterprise 
IAM, department-based dynamic membership ensures access tracks 
HR system changes automatically — critical for DoD onboarding 
and offboarding compliance.

**Restricting guest invitations to admins** is a baseline DoD 
control. Uncontrolled external collaboration is a common audit 
finding in IL4 environments.

**Entitlement Management implements a governed access lifecycle** — 
Request → Approve → Use → Expire → Review — rather than ad-hoc 
group assignments with no accountability chain.

## Troubleshooting Notes
- Dynamic group membership takes up to 5 minutes to populate 
  after rule creation — not an error, expected propagation delay
- Entitlement Management requires Entra ID P2 licensing — 
  activate trial from Entra admin center under Licenses if 
  menu is not visible

## Compliance Mapping

| Control | Requirement | Implementation |
|---|---|---|
| AC-2 | Account Management | User hierarchy, named accounts per access tier |
| AC-2(j) | Account Review | Quarterly access reviews via Entitlement Management |
| AC-3 | Access Enforcement | Security group membership controls resource access |
| IA-5 | Authenticator Management | User creation with temporary password and forced reset |

## Validation Checklist
- [x] Tenant settings hardened: app registration disabled, guest invite restricted
- [x] Three users created with Department = Cybersecurity
- [x] Three security groups created
- [x] Dynamic group auto-populated all three users
- [x] Access package created with 90-day expiration and quarterly review
- [x] My Access portal link saved

## Key Learning
Identity governance starts at the tenant level. Hardening settings 
before provisioning users ensures every identity created inherits 
a secure baseline — not retrofitted after the fact. In a DoD IL4 
environment, these tenant-level controls are reviewed during the 
authorization to operate process and must be documented with evidence.
