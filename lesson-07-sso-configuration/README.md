# Lesson 07: Single Sign-On (SSO) Configuration

## Overview

This lesson covers how to configure Single Sign-On (SSO) in Microsoft Entra ID. SSO enables users to sign in once and access multiple applications without re-authenticating.

## Learning Objectives

By the end of this lesson, you will:
- Understand SSO methods and protocols
- Configure SAML-based SSO
- Configure OIDC-based SSO
- Implement password-based SSO
- Troubleshoot SSO issues

---

## 1. SSO Concepts

### What is SSO?

Single Sign-On allows users to:
- Authenticate once
- Access multiple applications
- Reduce password fatigue
- Improve security posture

### SSO Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                   │
│  ┌─────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐        │
│  │User │────▶│ Sign in │────▶│ Session │────▶│  App 1  │        │
│  └─────┘     │ Entra ID│     │ Created │     └─────────┘        │
│              └─────────┘     └────┬────┘                         │
│                                   │                              │
│                    ┌──────────────┼──────────────┐               │
│                    │              │              │               │
│                    ▼              ▼              ▼               │
│              ┌─────────┐   ┌─────────┐   ┌─────────┐            │
│              │  App 2  │   │  App 3  │   │  App 4  │            │
│              │(No auth)│   │(No auth)│   │(No auth)│            │
│              └─────────┘   └─────────┘   └─────────┘            │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### SSO Methods in Entra ID

| Method | Protocol | Use Case |
|--------|----------|----------|
| SAML | SAML 2.0 | Enterprise apps, legacy |
| OpenID Connect | OAuth 2.0/OIDC | Modern apps, APIs |
| Password-based | N/A | Apps without federation |
| Linked | Redirect | External identity providers |
| Header-based | HTTP Headers | Legacy on-premises apps |

---

## 2. SAML-Based SSO

### SAML Concepts

| Term | Description |
|------|-------------|
| Identity Provider (IdP) | Entra ID - authenticates users |
| Service Provider (SP) | The application |
| Assertion | Security token with user claims |
| Entity ID | Unique identifier for IdP/SP |
| ACS URL | Where SAML response is sent |

### SAML Flow

```
┌──────┐         ┌──────────┐         ┌─────────┐
│ User │         │ Entra ID │         │   App   │
└──┬───┘         └────┬─────┘         └────┬────┘
   │                  │                    │
   │ 1. Access app    │                    │
   │─────────────────────────────────────▶ │
   │                  │                    │
   │ 2. Redirect to IdP (SAML Request)     │
   │◀─────────────────────────────────────  │
   │                  │                    │
   │ 3. Authenticate  │                    │
   │─────────────────▶│                    │
   │                  │                    │
   │ 4. SAML Response │                    │
   │◀─────────────────│                    │
   │                  │                    │
   │ 5. Post SAML Response to ACS URL      │
   │─────────────────────────────────────▶ │
   │                  │                    │
   │ 6. Access granted│                    │
   │◀─────────────────────────────────────  │
```

### Configuring SAML SSO

```
Enterprise applications → [App] → Single sign-on → SAML
```

#### Basic SAML Configuration

```yaml
Identifier (Entity ID):
  - Unique identifier for the app
  - Example: https://salesforce.com
  - Example: urn:amazon:webservices

Reply URL (Assertion Consumer Service URL):
  - Where SAML assertion is sent
  - Example: https://login.salesforce.com

Sign on URL (optional):
  - SP-initiated SSO start point
  - Example: https://app.example.com/login

Relay State (optional):
  - Where to redirect after sign-in
  - Example: /dashboard

Logout URL (optional):
  - For single logout (SLO)
  - Example: https://app.example.com/logout
```

#### User Attributes & Claims

```yaml
Required Claim:
  Name: http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
  Source: Attribute
  Value: user.userprincipalname

Additional Claims:
  - email: user.mail
  - firstName: user.givenname
  - lastName: user.surname
  - displayName: user.displayname
  - department: user.department
  - groups: user.groups [All groups]
```

#### SAML Signing Certificate

```yaml
Options:
  - Sign SAML response
  - Sign SAML assertion
  - Sign SAML response and assertion (most secure)

Certificate Download:
  - Certificate (Base64): For most apps
  - Certificate (Raw): Binary format
  - Federation Metadata XML: Auto-configuration

Certificate Management:
  - Create new certificate before expiry
  - Set notification email
  - Plan for rotation
```

### SAML Configuration Example: Salesforce

```yaml
Basic SAML Configuration:
  Identifier: https://saml.salesforce.com
  Reply URL: https://yourcompany.my.salesforce.com?so=00D...
  Sign on URL: https://yourcompany.my.salesforce.com

Attributes & Claims:
  Unique User Identifier: user.userprincipalname
  Federation ID: user.mail
  ProfileId: user.assignedroles

Certificate:
  Signing Option: Sign SAML Response and Assertion
  Algorithm: SHA-256
```

---

## 3. OpenID Connect SSO

### OIDC vs SAML

| Aspect | SAML | OIDC |
|--------|------|------|
| Protocol | XML-based | JSON-based |
| Token | Assertion | JWT |
| Best for | Enterprise apps | Modern apps, APIs |
| Mobile support | Limited | Excellent |
| Implementation | Complex | Simpler |

### OIDC Configuration

For apps using OIDC, configuration is typically done through App Registration:

```
App registrations → [App] → Authentication
```

```yaml
Platform: Web / SPA / Mobile

Redirect URIs:
  - https://app.example.com/auth/callback

ID tokens: Enabled (for authentication)
Access tokens: Enabled (if API access needed)

Token configuration:
  - Add optional claims as needed
```

### Claims Configuration

```
App registrations → [App] → Token configuration
```

```yaml
Optional claims:
  ID Token:
    - email
    - family_name
    - given_name
    - upn

  Access Token:
    - idtyp (token type)

Groups claim:
  - Security groups
  - Emit as: Group ID / sAMAccountName / NetBIOS
```

---

## 4. Password-Based SSO

### When to Use

- Application doesn't support federation
- Legacy applications
- Quick SSO implementation

### How It Works

```
┌──────┐         ┌──────────┐         ┌─────────┐
│ User │         │ Entra ID │         │   App   │
└──┬───┘         └────┬─────┘         └────┬────┘
   │                  │                    │
   │ 1. Access app    │                    │
   │─────────────────▶│                    │
   │                  │                    │
   │ 2. Retrieve      │                    │
   │    credentials   │                    │
   │◀─────────────────│                    │
   │                  │                    │
   │ 3. Auto-fill and submit credentials   │
   │─────────────────────────────────────▶ │
   │                  │                    │
   │ 4. Access granted│                    │
   │◀─────────────────────────────────────  │
```

### Configuration

```
Enterprise applications → [App] → Single sign-on → Password-based
```

```yaml
Sign-on URL: https://app.example.com/login

Credential settings:
  - Users enter their own credentials
  - Admin assigns credentials
  - Users share credentials

Field detection:
  - Automatic (extension detects fields)
  - Manual (specify field selectors)
```

### Requirements

- My Apps browser extension installed
- Or My Apps mobile app

---

## 5. Linked SSO

### Purpose

Redirect users to another identity provider or portal.

### Configuration

```
Enterprise applications → [App] → Single sign-on → Linked
```

```yaml
Sign-on URL: https://external-idp.com/app/login
```

---

## 6. My Apps Portal

### Overview

User-facing portal for accessing SSO-enabled applications.

**URL**: https://myapps.microsoft.com

### Features

- Application tiles for all assigned apps
- Self-service group management
- Access requests
- App launch

### Customization

```
Microsoft Entra ID → Enterprise applications → User settings
```

```yaml
Collections:
  - Group apps into collections
  - Example: "Sales Tools", "HR Applications"

Custom branding:
  - Company logo
  - Background color
  - Custom links
```

### App Collections

```
Microsoft Entra ID → Enterprise applications → App launchers → Collections
→ New collection
```

```yaml
Collection name: Finance Applications
Apps:
  - SAP
  - Concur
  - Workday
Users: GRP-SEC-Finance-Users
```

---

## 7. SSO for On-Premises Apps

### Application Proxy + SSO

Combine Application Proxy with SSO for on-premises apps:

```yaml
SSO Methods for App Proxy:
  - Integrated Windows Authentication (IWA)
  - Header-based authentication
  - Password-based
  - SAML
```

### Integrated Windows Authentication

```
Enterprise applications → [App Proxy App] → Single sign-on
→ Integrated Windows Authentication
```

```yaml
Internal Application SPN:
  HTTP/intranet.contoso.com

Delegated Login Identity:
  - User Principal Name
  - On-Premises SAM Account Name
  - On-Premises User Principal Name
```

### Kerberos Constrained Delegation

Requirements:
1. Connector server joined to domain
2. SPN configured for internal app
3. KCD delegation configured
4. Connector service account permissions

```powershell
# Set SPN for the application
setspn -S HTTP/intranet.contoso.com DOMAIN\AppServiceAccount

# Configure KCD on connector server
# (Done via Active Directory Users and Computers)
```

---

## 8. Testing SSO

### SAML Testing

```
Enterprise applications → [App] → Single sign-on → Test
```

Steps:
1. Click "Test this application"
2. Select test mode: Current user or other user
3. Review SAML response
4. Check for errors

### Common SAML Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid signature | Certificate mismatch | Re-download certificate |
| Invalid audience | Wrong Entity ID | Verify Identifier setting |
| User not assigned | Missing assignment | Assign user/group |
| Invalid NameID | Wrong attribute mapped | Check NameID claim |
| Time skew | Clock differences | Sync NTP |

### SAML Tracer Extension

Use browser extensions to capture SAML traffic:
- SAML-tracer (Firefox/Chrome)
- SAML Chrome Panel

---

## 9. Provisioning with SSO

### Automatic User Provisioning

Sync users from Entra ID to applications:

```
Enterprise applications → [App] → Provisioning
```

### Provisioning Modes

| Mode | Description |
|------|-------------|
| Automatic | SCIM-based sync |
| Manual | Admin creates users |
| None | No provisioning |

### SCIM Provisioning Setup

```yaml
Provisioning Mode: Automatic

Admin Credentials:
  Tenant URL: https://app.example.com/scim/v2
  Secret Token: [API token from app]

Mappings:
  - Provision Azure Active Directory Users: Yes
  - Provision Azure Active Directory Groups: Yes

Attribute Mappings:
  Entra ID → App
  userPrincipalName → userName
  displayName → displayName
  mail → emails[type eq "work"].value
  department → department
```

### Provisioning + SSO Flow

```
User Created      User Provisioned     User Signs In
in Entra ID  ───▶   to App via    ───▶  via SAML/OIDC
                      SCIM
```

---

## 10. SSO Troubleshooting

### Sign-in Logs

```
Microsoft Entra ID → Sign-in logs → Filter by application
```

Key information:
- Status (Success/Failure)
- Error code and message
- Conditional Access results
- Authentication details

### Common Issues and Solutions

#### Issue: User cannot access application

```yaml
Check:
  1. Is user assigned to the app?
  2. Is assignment required? (Properties)
  3. Any Conditional Access blocking?
  4. Is the app enabled?

Solution:
  - Assign user/group to app
  - Review CA policies
  - Enable the app
```

#### Issue: SAML signature validation failed

```yaml
Check:
  1. Is certificate expired?
  2. Was certificate recently rotated?
  3. Is correct certificate uploaded to SP?

Solution:
  - Download new certificate
  - Upload to application
  - Verify signing options match
```

#### Issue: Claims not appearing in token

```yaml
Check:
  1. Is claim configured in Token Configuration?
  2. Is claim included in SAML Attributes & Claims?
  3. Does source attribute have value?

Solution:
  - Add required claims
  - Verify attribute mappings
  - Check user attribute values
```

#### Issue: Password SSO not working

```yaml
Check:
  1. Is browser extension installed?
  2. Is extension signed in?
  3. Can extension detect login fields?

Solution:
  - Install My Apps extension
  - Sign in to extension
  - Configure manual field detection
```

### Diagnostic Tools

```
Enterprise applications → [App] → Single sign-on
→ Troubleshoot → Run diagnostics
```

---

## 11. SSO Best Practices

### Security

- [ ] Use SAML or OIDC over password-based
- [ ] Enable MFA via Conditional Access
- [ ] Rotate certificates before expiry
- [ ] Monitor sign-in logs for anomalies
- [ ] Implement least privilege access

### Operations

- [ ] Document SSO configurations
- [ ] Test SSO after certificate rotation
- [ ] Configure certificate expiry notifications
- [ ] Maintain break-glass procedures
- [ ] Regular access reviews

### Planning

- [ ] Inventory all applications
- [ ] Prioritize apps for SSO integration
- [ ] Choose appropriate SSO method per app
- [ ] Plan provisioning alongside SSO
- [ ] Test in non-production first

---

## Summary

SSO in Microsoft Entra ID provides:
- Unified authentication experience
- Multiple SSO methods for different needs
- Secure access to cloud and on-premises apps
- Centralized access management
- Improved user productivity

---

## Hands-On Exercise

1. Add a gallery application with SAML SSO
2. Configure user attributes and claims
3. Assign users/groups to the application
4. Test SSO using the built-in test feature
5. Review sign-in logs for the application
6. Configure password-based SSO for comparison

---

## Next Lesson

[Lesson 08: Security and Governance →](../lesson-08-security-governance/README.md)

---

## Additional Resources

- [SAML-based SSO](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-saml-single-sign-on)
- [OIDC-based SSO](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc)
- [Password-based SSO](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-password-single-sign-on-non-gallery-applications)
- [Troubleshoot SAML](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/debug-saml-sso-issues)
