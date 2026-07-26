# Lab 06 — DoD IL2/IL4/IL5 Compliance Controls

## Objective
Map all implemented lab controls to DoD Impact Level requirements. 
Configure authentication strength policies, geographic access 
restrictions, session controls, and audit log verification. 
Document IL-specific requirements and identify gaps between 
lab environment capabilities and production DoD deployment.

## Environment
- Portal: entra.microsoft.com
- Framework: DoD CC SRG, NIST 800-53, CMMC
- Duration: 75 minutes

## Impact Level Reference

| Level | Data Type | Cloud Target | Key IAM Requirement |
|---|---|---|---|
| IL2 | Unclassified, non-controlled | Azure Commercial | Baseline MFA recommended |
| IL4 | Controlled Unclassified (CUI) | Azure Government | Mandatory MFA, PIV/CAC, PIM required |
| IL5 | CUI — National Security Systems | Azure Government | Phishing-resistant MFA only, PAWs required |
| IL6 | Classified SECRET | Azure Government Secret | Separate classified infrastructure |

## What I Configured

### Part 1 — Authentication Strength Policy
Reviewed CA grant controls for authentication strength options:
- Multifactor authentication — IL4 baseline
- Phishing-resistant MFA — IL5 requirement
  (FIDO2 security keys, certificate-based authentication)

Documented distinction between software-based MFA acceptable 
at IL4 vs phishing-resistant-only requirement at IL5.

### Part 2 — Geographic Restriction Policy
Configured CA policy scoped to block non-US access:
- Condition: Any location
- Exclude: Named locations (Trusted-Corporate-Network)
- Grant: Block access

This implements AC-17 remote access control aligned with 
DoD IL4 requirement to restrict access from non-approved 
geographic locations.

### Part 3 — Privileged Role Audit
Verified all privileged roles configured as PIM-eligible:
- No standing Global Admin assignments
- No standing Privileged Role Administrator assignments
- All activations require justification and MFA
- Activation window: 2 hours maximum

Standing privileged access confirmed absent — IL4 compliant.

### Part 4 — Session Control Configuration
Reviewed sign-in frequency controls:
- IL4 baseline: 8-hour re-authentication for standard users
- IL5 recommendation: shorter windows for sensitive applications
- Configured via CA policy Session controls

### Part 5 — Audit Log Verification
Confirmed Log Analytics workspace capturing:
- Sign-in logs
- Audit logs
- Provisioning logs
- Risky user events

Diagnostic settings verified active — AU-2 control evidence 
confirmed.

## IL4 vs IL5 — Key Differences

### Authentication
| Requirement | IL4 | IL5 |
|---|---|---|
| MFA | Mandatory all privileged users | Mandatory all users |
| Method | Any MFA including Authenticator app | Phishing-resistant only (FIDO2/PIV) |
| CAC/PIV | Required for CUI access | Required |

### Privileged Access
| Requirement | IL4 | IL5 |
|---|---|---|
| PIM | Mandatory, no standing access | Mandatory, approval workflow required |
| Activation | Self-activation with justification | Second-person approval required |
| Admin device | Standard managed device | Privileged Access Workstation (PAW) |

### Network Controls
| Requirement | IL4 | IL5 |
|---|---|---|
| Geo-restriction | Non-US blocking recommended | Strict approved IP ranges |
| CAE | Recommended | Required with strict enforcement |
| VPN | Required for remote CUI access | Required |

## NIST 800-53 Control Mapping

| Control | Requirement | Implementation | Status |
|---|---|---|---|
| AC-2 | Account Management | Entitlement management, access reviews | Implemented |
| AC-3 | Access Enforcement | Scoped RBAC assignments | Implemented |
| AC-6 | Least Privilege | PIM JIT, custom roles, RG-level scope | Implemented |
| AC-17 | Remote Access | Named location CA policies, geo-restriction | Implemented |
| IA-2 | MFA | Baseline and risk-based CA policy stack | Implemented |
| IA-2(6) | Phishing-resistant MFA | FIDO2 for SG-IAM-Admins | Partial |
| IA-5 | Authenticator Management | MFA registration, break-glass controls | Implemented |
| IA-8 | Non-Org Users | Federation controls, guest restrictions | Implemented |
| AU-2 | Audit Events | Log Analytics workspace | Implemented |
| AU-3 | Audit Content | Diagnostic settings all identity events | Implemented |
| SI-4 | System Monitoring | Entra ID Protection risk detection | Implemented |

## Gap Analysis

### IL4 Gaps
- CAC/PIV integration not configured — requires DoD PKI 
  federation setup beyond lab scope
- Device compliance CA condition not enforced — requires 
  Intune enrollment

### IL5 Gaps
- Phishing-resistant MFA not enforced for all users — 
  currently scoped to SG-IAM-Admins only
- PIM approval workflows not configured — self-activation 
  only in current setup
- Privileged Access Workstations not deployed — 
  infrastructure requirement beyond lab scope
- CAE strict enforcement — P2 licensing required

All gaps documented with production remediation path. 
Documented gaps with plans are preferable to undiscovered 
gaps during audit assessment.

## CAC/PIV Federation — Production Context
DoD users authenticate with physical smart cards — CAC for 
military personnel, PIV for civilians. Certificates issued 
by DoD PKI. Entra ID configured to federate with DoD PKI 
as external certificate authority enables CAC/PIV 
authentication into cloud resources. This is a core IAM 
engineer competency in the federal space not replicable 
in a personal lab tenant.

## Troubleshooting Notes
- Authentication strength Phishing-resistant MFA option 
  requires Entra ID P2 for full enforcement
- Geographic restriction CA policy must explicitly exclude 
  named trusted locations to avoid blocking your own access
- PIM approval workflows require additional approver 
  configuration — not available with single-user tenant

## Compliance Mapping Summary

| Lab Control | NIST 800-53 | IL4 | IL5 |
|---|---|---|---|
| MFA CA policy stack | IA-2 | Required | Required |
| PIM JIT 2-hour windows | AC-6 | Required | Required + approval |
| Scoped RBAC | AC-3 | Required | Required |
| Named location geo-restriction | AC-17 | Recommended | Required |
| Log Analytics audit logging | AU-2, AU-3 | Required | Required |
| Entitlement Management reviews | AC-2 | Required | Required |

## Validation Checklist
- [x] Impact level reference documented IL2 through IL6
- [x] Authentication strength options reviewed and documented
- [x] Geographic restriction CA policy configured
- [x] Privileged role audit — no standing assignments confirmed
- [x] Session control configuration reviewed
- [x] Log Analytics diagnostic settings verified active
- [x] IL4 vs IL5 differences documented
- [x] Gap analysis completed with remediation paths

## Key Learning
DoD compliance is not binary — it's a spectrum of controls 
mapped to data sensitivity and mission impact. An IAM engineer 
in a cleared environment must understand not just what controls 
exist but why each control maps to a specific IL requirement 
and what the operational consequence of a gap is. Documented 
gaps with remediation paths demonstrate mature security 
engineering judgment — more valuable in an audit than 
undocumented assumptions of compliance.
