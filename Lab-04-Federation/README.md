# Lab 04 — Federation Protocols (SAML, OAuth 2.0 & OIDC)

## Objective
Configure SAML 2.0 enterprise application SSO and an OIDC app 
registration. Test federation flows as target test users in 
private browser sessions. Decode a live ID token via jwt.ms 
to verify claims. Understand the distinction between 
authentication and authorization protocols in a federated 
identity architecture.

## Environment
- Portal: entra.microsoft.com + portal.azure.com
- Tools: jwt.ms for token decoding
- Duration: 75 minutes

## What I Configured

### Part 1 — SAML 2.0 Enterprise Application
Created SAML-LAB-APP enterprise application with:
- Entity ID configured
- Reply URL (ACS URL): samllab.securelab.local/acs
- Signing certificate generated
- Attribute mappings configured for UPN and email claims
- SSO tested as iam.analyst01 via My Apps portal

### Part 2 — OIDC App Registration
Created OIDC-LAB-APP app registration with:
- Supported account types: Single tenant
- Redirect URI: https://jwt.ms
- Client secret generated with expiration date noted
- ID token implicit grant enabled for lab testing
- API permissions: delegated — openid, profile, email

### Part 3 — OIDC Authorization Flow Test
Manually constructed authorization URL and executed implicit flow:
- response_type: id_token
- scope: openid profile email
- nonce included for replay attack prevention
- Token received and decoded at jwt.ms

### Part 4 — ID Token Claims Verified

| Claim | Value | Purpose |
|---|---|---|
| iss | Entra ID tenant URL | Confirms authoritative IdP |
| aud | OIDC-LAB-APP client ID | Confirms correct audience |
| oid | iam.analyst01 object ID | Immutable user identifier |
| preferred_username | iam.analyst01 UPN | User identity |
| nonce | anyRandomString123 | Replay attack protection |
| exp | Unix timestamp | Token expiration |

## Protocol Comparison

| Protocol | Type | Artifact | Use Case |
|---|---|---|---|
| SAML 2.0 | XML-based | Signed XML Assertion | Legacy enterprise, federal systems |
| OAuth 2.0 | Token-based | Access Token | Delegated API authorization |
| OIDC | OAuth 2.0 + identity | ID Token + Access Token | Modern app authentication |

## Key Findings

**SAML flow completed successfully despite DNS failure.** 
Browser redirected to samllab.securelab.local/acs with 
DNS_PROBE_FINISHED_NXDOMAIN. Root cause: ACS URL points to 
a non-existent lab domain — no application server configured 
to receive the assertion. The Entra ID SAML flow completed 
correctly. Redirect to ACS URL confirms the IdP processed 
the auth request and issued a signed assertion. Failure was 
at the network layer, not the identity layer.

**AADSTS700054 — implicit flow not enabled.** Implicit grant 
for ID tokens is disabled by default in modern Entra ID app 
registrations — a deliberate security control. Enabled for 
lab testing only. Production recommendation: authorization 
code flow with PKCE, which never exposes tokens in the URL.

**SSO must be tested as the target user.** Testing as Global 
Admin masks missing user assignments, CA policy differences, 
and attribute mapping failures. Always test in a private 
browser session as the actual assigned user.

**App Registration vs Enterprise Application distinction.**
App Registration defines the application identity and OAuth/OIDC 
settings. Enterprise Application governs its use in the tenant — 
user assignments, SAML SSO configuration, and provisioning. 
Two sides of the same coin.

## Troubleshooting Notes
- DNS failure at ACS URL is a lab infrastructure limitation — 
  not a SAML configuration error. IdP side functioned correctly
- AADSTS700054 resolved by enabling ID token implicit grant 
  under App Registration → Authentication → Implicit grant
- SSO test must be performed as target user in private browser — 
  not as Global Admin from the Entra admin center test blade
- My Apps portal at myapps.microsoft.com is the correct 
  end-user SSO test surface

## Compliance Mapping

| Control | Requirement | Implementation |
|---|---|---|
| IA-8 | Non-organizational user identification | Federation trust via SAML/OIDC |
| SC-8 | Transmission confidentiality | HTTPS enforced on all federation endpoints |
| AC-17 | Remote access | SSO federation as controlled access mechanism |
| IA-2 | Identification and authentication | OIDC token-based authentication verified |

## Validation Checklist
- [x] SAML-LAB-APP enterprise application created and configured
- [x] SSO tested as iam.analyst01 via My Apps portal
- [x] SAML redirect to ACS URL confirmed — IdP flow working
- [x] OIDC-LAB-APP app registration created
- [x] Client secret generated with expiration noted
- [x] Authorization URL manually constructed and executed
- [x] ID token received and decoded at jwt.ms
- [x] All key claims verified: iss, aud, oid, nonce, exp

## Key Learning
Federation is about trust delegation — applications accept 
identity assertions from a trusted IdP rather than managing 
credentials themselves. SAML uses self-contained signed XML 
assertions validated cryptographically. OIDC uses JWTs where 
the ID token carries identity claims and the access token 
carries authorization scope. In DoD environments legacy systems 
speak SAML while modern APIs use OIDC — an IAM engineer must 
be fluent in both and understand exactly what each token or 
assertion contains and why.
