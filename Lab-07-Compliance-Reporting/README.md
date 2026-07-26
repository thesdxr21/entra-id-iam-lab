# Lab 07 — IAM Compliance Reporting & Auditing

## Objective
Build the evidence layer that makes all prior lab controls 
credible to auditors and authorizing officials. Configure 
access reviews for privileged roles, query sign-in and audit 
logs using KQL, and compile a compliance summary mapping every 
implemented control to its NIST 800-53 control number. 
Demonstrate both implementation evidence and effectiveness 
evidence across the full lab series.

## Environment
- Portal: entra.microsoft.com + Log Analytics workspace
- Workspace: law-oyainvestments-iam
- Duration: 60 minutes

## What I Configured

### Part 1 — Access Reviews
Configured recurring access review for privileged roles:
- Scope: Global Administrator, Privileged Role Administrator
- Review frequency: Quarterly
- Reviewers: Designated reviewer account
- Auto-remove: Unreviewed assignments automatically removed
- First review cycle documented as evidence artifact

This directly implements NIST 800-53 AC-2(j) — reviewing 
accounts for compliance with account management requirements.

### Part 2 — PIM Audit Report
Exported PIM audit history covering full lab period:
- All iam.admin01 activations logged
- Activation duration confirmed within 2-hour window
- Justification text captured per activation
- No standing privileged access confirmed

This serves as AC-6 control evidence demonstrating JIT 
enforcement was active throughout the lab series.

### Part 3 — Sign-In Log Analysis
Queried sign-in logs for test users across lab period:
- CA policies applied per sign-in confirmed
- MFA outcomes verified
- Risk detections reviewed
- Authentication events summarized as AU-2 evidence

### Part 4 — KQL Queries

**Privileged role activations — last 30 days:**
```kusto
AuditLogs
| where TimeGenerated > ago(30d)
| where OperationName == "Add member to role completed (PIM activation)"
| project TimeGenerated, InitiatedBy, TargetResources, ResultDescription
| order by TimeGenerated desc
```

**Failed sign-ins by user:**
```kusto
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize FailureCount = count() by UserPrincipalName, ResultDescription
| order by FailureCount desc
```

**CA policy applied per sign-in:**
```kusto
SigninLogs
| where TimeGenerated > ago(24h)
| mv-expand ConditionalAccessPolicies
| project TimeGenerated, UserPrincipalName, 
  ConditionalAccessPolicies.displayName, 
  ConditionalAccessPolicies.result
```

**MFA registration gaps:**
```kusto
UserRegistrationDetails
| where IsMfaRegistered == false
| project UserPrincipalName, IsPasswordlessCapable, 
  MethodsRegistered
```

### Part 5 — Identity Secure Score Trend
Documented score progression across lab series:
- Baseline: Pre-lab tenant score
- Final: 57.27% following Labs 1-6 implementation
- Upward trend confirmed in score history chart
- Remaining gaps attributed to P2 licensing requirements

## Control Implementation Matrix

| Control | Requirement | Implementation | Evidence | Status |
|---|---|---|---|---|
| IA-2 | MFA enforcement | CA-001 MFA policy | Sign-in logs | Implemented |
| AC-6 | Least privilege | PIM JIT, scoped RBAC | PIM audit export | Implemented |
| AC-2 | Account management | Entitlement management | Access review report | Implemented |
| AC-2(j) | Account review | Quarterly access reviews | Review cycle record | Implemented |
| AU-2 | Audit events | Log Analytics workspace | KQL query results | Implemented |
| AU-3 | Audit content | Diagnostic settings | Workspace config | Implemented |
| AC-3 | Access enforcement | Scoped RBAC assignments | Role assignment screenshots | Implemented |
| AC-17 | Remote access | Named location CA policies | CA policy config | Implemented |
| IA-5 | Authenticator management | MFA registration enforcement | Auth methods report | Implemented |
| IA-8 | Non-org users | Guest restrictions, federation | Tenant settings | Implemented |
| SI-4 | System monitoring | Entra ID Protection risk policies | Risk event logs | Implemented |

## Two Types of Compliance Evidence

### Implementation Evidence
Shows the control exists:
- Screenshots of PIM policy configuration
- CA policy settings and grant controls
- Access review schedule and configuration
- Log Analytics diagnostic settings

### Effectiveness Evidence
Shows the control is working:
- PIM audit logs confirming every activation was time-limited
- Access review records showing certifications completed
- Sign-in logs confirming MFA enforced on every auth
- Risk event logs showing detections and responses

Both types mapped to NIST 800-53 control numbers in the 
control implementation matrix above.

## Gap Analysis with Remediation Paths

| Gap | Reason | Production Remediation |
|---|---|---|
| Phishing-resistant MFA all users | P2 licensing | Enforce auth strength CA grant at IL5 |
| PIM approval workflows | Single-user tenant | Add approval stage in PIM role settings |
| CAC/PIV integration | DoD PKI required | Configure certificate-based auth with DoD PKI |
| CAE strict enforcement | P2 licensing | Enable per CA policy — Session controls |
| Device compliance enforcement | Intune required | Add compliant device CA grant condition |
| Privileged Access Workstations | Infrastructure | Deploy hardened admin workstations |

## Lab Series Summary

| Lab | Topic | Primary Controls | Evidence Type |
|---|---|---|---|
| Lab 01 | Tenant Setup | AC-2, IA-5 | Configuration screenshots |
| Lab 02 | RBAC + PIM | AC-3, AC-6 | Role assignments, PIM audit |
| Lab 03 | MFA + CA | IA-2, AC-17 | Sign-in logs, policy config |
| Lab 04 | Federation | IA-8, SC-8 | Token decode, SSO test |
| Lab 05 | Zero Trust | AC-17, SI-4 | Secure Score, named locations |
| Lab 06 | DoD IL Compliance | Full NIST mapping | Gap analysis, IL matrix |
| Lab 07 | Reporting + Auditing | AU-2, AU-3, AC-2 | KQL results, access reviews |

## Troubleshooting Notes
- Log Analytics data may take 15-30 minutes to appear 
  after diagnostic settings are configured
- KQL queries require correct table names — SigninLogs 
  and AuditLogs are case sensitive
- Access reviews require Entra ID P2 licensing to configure
- UserRegistrationDetails table requires Microsoft Entra 
  ID reporting permissions

## Compliance Mapping
This lab serves as the capstone evidence package for the 
full control framework. All controls above reference specific 
NIST 800-53 identifiers traceable to the DoD IL4 Moderate 
baseline requirement set.

## Validation Checklist
- [x] Access review configured for Global Admin and 
      Privileged Role Administrator
- [x] Quarterly recurrence and auto-remove confirmed
- [x] PIM audit history exported for lab period
- [x] Sign-in logs queried and MFA outcomes verified
- [x] KQL queries written and tested in Log Analytics
- [x] Identity Secure Score trend documented
- [x] Control implementation matrix completed
- [x] Gap analysis documented with remediation paths
- [x] Both implementation and effectiveness evidence captured

## Key Learning
Compliance reporting is the difference between having controls 
and being able to prove controls. An Authorizing Official 
granting an ATO does not take your word for it — they need 
traceable evidence mapped to specific control requirements. 
Implementation evidence shows the control exists. Effectiveness 
evidence shows it worked. KQL gives you the ability to query 
your own environment and produce that evidence on demand rather 
than scrambling during an audit. In a DoD cleared environment 
this capability — knowing how to interrogate your own logs and 
produce control evidence — is what separates an IAM analyst 
from an IAM engineer.
