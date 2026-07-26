# Lab 05 — Zero Trust Architecture Implementation

## Objective
Implement Zero Trust controls in Entra ID by configuring named 
locations, reviewing Identity Secure Score, and documenting 
Continuous Access Evaluation capabilities. Connect prior lab 
controls into a cohesive Zero Trust architecture mapped to 
NIST 800-207 and DoD Zero Trust Reference Architecture.

## Environment
- Portal: entra.microsoft.com
- Duration: 60 minutes

## What I Configured

### Part 1 — Named Locations
Created Trusted-Corporate-Network named location:
- Type: IP ranges location
- IP range: [REDACTED]/32 (single host CIDR notation)
- Marked as trusted location

Common error encountered and resolved: bare IP address without 
CIDR notation is invalid. Entra ID requires /32 for a single 
IP address. Added subnet mask to resolve validation error.

### Part 2 — Identity Secure Score Review
Reviewed tenant security posture after Labs 1-4 completion:
- Overall score: 57.27%
- Total recommendations: 14
- Security recommendations: 12
- Best practice recommendations: 2
- Score history shows upward trend from April 2026

Upward trend directly attributed to controls implemented 
across Labs 1-4 — MFA enforcement, CA policy stack, RBAC 
scoping, and PIM eligible assignments.

### Part 3 — Continuous Access Evaluation (CAE)
CAE strict enforcement mode not available in lab tenant.

**Lab Finding:** CAE requires Entra ID P2 licensing and is 
surfaced under Security → Continuous Access Evaluation in 
production tenants. The standalone tenant-wide CAE toggle 
was deprecated by Microsoft — CAE is now configured per-policy 
in Conditional Access → Session controls as Strict Location 
Enforcement (Public Preview).

In a production DoD environment CAE strict location enforcement 
would be enabled to ensure token revocation occurs in real time 
when a user's network location changes outside trusted ranges — 
directly supporting Zero Trust continuous verification principles 
rather than waiting for token expiration (up to 75 minutes).

## Zero Trust Principles Applied

### Verify Explicitly
Every access request evaluated against all available signals — 
identity, location, device, risk level. Implemented via layered 
CA policy stack from Lab 03.

### Use Least Privilege Access
JIT access via PIM, scoped RBAC assignments, time-limited 
entitlement packages. Implemented across Labs 01-02.

### Assume Breach
Log Analytics workspace capturing all identity events. Risk 
policies detecting anomalous sign-ins. Break-glass accounts 
isolated from standard CA policies.

## How Prior Labs Map to Zero Trust

| What I Built | Zero Trust Principle | Pillar |
|---|---|---|
| Entitlement Management 90-day expiration | Least privilege | Identity |
| PIM eligible assignments | Just-in-time access | Identity |
| Risk-based CA policies | Verify explicitly | Identity |
| Break-glass exclusions | Assume breach | Identity |
| SAML/OIDC federation | Verify explicitly | Applications |
| Scoped RBAC assignments | Least privilege | Infrastructure |
| Log Analytics workspace | Assume breach | Identity |

## Identity Secure Score — Improvement Actions Noted
Remaining gaps primarily require Entra ID P2 licensing:
- Insider Risk condition in Conditional Access (P2 required)
- Additional Identity Protection policies (P2 required)
- Microsoft Defender for Identity recommendations (separate license)

Documented as expected lab constraints — not misconfigurations.

## Troubleshooting Notes
- Named location requires CIDR notation — bare IP without /32 
  is invalid and throws validation error
- CAE standalone toggle deprecated by Microsoft — platform 
  drift documented as lab finding
- CAE now configured per CA policy under Session controls, 
  not as a tenant-wide setting
- Identity Secure Score refreshes every 24 hours — recent 
  configurations may not reflect immediately

## Compliance Mapping

| Control | Requirement | Implementation |
|---|---|---|
| AC-17 | Remote access | Named location trusted network definition |
| SI-4 | System monitoring | Secure Score continuous assessment |
| AC-2 | Account management | Secure Score improvement actions reviewed |
| SC-5 | Denial of service protection | CAE real-time token revocation (documented) |

## Validation Checklist
- [x] Named location Trusted-Corporate-Network created with /32 CIDR
- [x] Location marked as trusted
- [x] Identity Secure Score reviewed — 57.27%
- [x] Score history upward trend documented
- [x] Top improvement actions reviewed and documented
- [x] CAE licensing limitation identified and documented as finding
- [x] Zero Trust control mapping completed across all labs

## Key Learning
Zero Trust is not a product or a feature — it's an architectural 
philosophy operationalized through layered technical controls. 
Named locations make network context an explicit trust signal 
rather than an implicit assumption. CAE extends that verification 
into active sessions — not just at login. Identity Secure Score 
provides continuous measurement of posture against Zero Trust 
best practices. The 57.27% score with an upward trend is direct 
evidence that the lab control implementations had measurable 
security impact on the tenant — a concrete outcome to reference 
in compliance reporting and interviews.
