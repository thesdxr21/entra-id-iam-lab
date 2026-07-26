# Lab 03 — MFA & Conditional Access Policies

## Objective
Implement a layered MFA and Conditional Access policy framework 
enforcing Zero Trust principles. Build policies that enforce MFA 
based on user risk, location, and application sensitivity — 
mirroring real DoD access control requirements.

## Environment
- Portal: entra.microsoft.com
- Prerequisite: Entra ID P2 trial license active
- Duration: 60 minutes

## What I Configured

### Part 1 — MFA Registration Policy
- Enabled Microsoft Authenticator for all users
- Enabled FIDO2 Security Keys for SG-IAM-Admins group only
- Set MFA registration policy to enforce registration for all users

### Part 2 — Conditional Access Policy Stack

| Policy | Condition | Grant Control | Purpose |
|---|---|---|---|
| CA-001-Require-MFA-AllUsers | All users, all apps | Require MFA | Baseline enforcement |
| CA-002-Block-Legacy-Auth | Legacy auth clients | Block access | Eliminate non-MFA protocols |
| CA-003-Require-Compliant-Device-Admins | SG-IAM-Admins | MFA + Compliant device | Privileged access hardening |
| CA-004-SignIn-Risk-Block-High | Sign-in risk = High | Block access | Active threat response |

Break-glass account EmergencyAdmin excluded from all policies.
CA-003 deployed in Report-only mode before enforcement.

### Part 3 — Policy Stack Testing
Tested end-to-end as iam.analyst01 in private browser session.
Confirmed CA-001 fired and MFA prompt appeared. Reviewed CA 
Insights workbook to confirm policy application order.

## Key Findings

**One Grant control per CA policy.** You cannot combine Block 
and Require MFA in a single policy — these are separate 
enforcement outcomes requiring separate policies. This keeps 
logic clean and auditable.

**Report-only mode is mandatory before enforcement.** In a DoD 
environment a misconfigured CA policy blocking admin access is 
a critical incident. Report-only lets you see impact before 
anyone is affected.

**Break-glass accounts must be excluded from every CA policy.** 
If MFA is enforced and the break-glass account has no registered 
MFA method, you've locked yourself out of the tenant.

**Policy conflicts resolve to most restrictive.** If two policies 
apply to the same user and one says Require MFA while another 
says Block — Block wins.

## Troubleshooting Notes
- High-risk and Medium-risk require separate CA policies — 
  cannot combine different Grant controls in one policy
- Sign-in log visibility issues caused by propagation delay, 
  token caching, and wrong log tab — wait and verify correct tab
- CA Insights 401 error traced to missing Log Analytics 
  workspace dependency — workspace must be linked to Entra 
  diagnostic settings before workbook functions

## Compliance Mapping

| Control | Requirement | Implementation |
|---|---|---|
| IA-2 | MFA for privileged users | CA-001 baseline + CA-003 admin hardening |
| IA-2(1) | MFA for network access | CA-001 scoped to all cloud apps |
| AC-17 | Remote access controls | Location and risk-based CA conditions |
| SI-4 | System monitoring | Risk policies fed by Entra ID Protection |
| CM-7 | Least functionality | CA-002 blocking legacy authentication protocols |

## Validation Checklist
- [x] Microsoft Authenticator enabled for all users
- [x] FIDO2 Security Keys enabled for SG-IAM-Admins only
- [x] CA-001 enabled with break-glass exclusion
- [x] CA-002 blocking legacy auth clients
- [x] CA-003 in Report-only mode with impact reviewed
- [x] CA-004 blocking High risk sign-ins
- [x] MFA test completed as iam.analyst01 in private browser

## Key Learning
Conditional Access is not about requiring MFA — it's a policy 
engine that evaluates trust signals at authentication time and 
enforces access conditions. Identity, device, location, and risk 
signals all feed the decision. In a DoD IL4 environment this 
policy stack is the primary technical enforcement mechanism for 
Zero Trust — not a checkbox, but an architecture.
