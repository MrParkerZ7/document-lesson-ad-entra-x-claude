# Lesson 06: Application Integration

## Overview

This lesson covers how to integrate applications with Microsoft Entra ID for authentication and authorization. You'll learn about app registrations, enterprise applications, OAuth/OIDC protocols, and how to securely connect both cloud and on-premises applications.

## Learning Objectives

By the end of this lesson, you will:
- Understand the difference between app registrations and enterprise apps
- Register and configure applications
- Implement OAuth 2.0 and OpenID Connect flows
- Configure permissions and consent
- Manage application credentials securely
- Expose your own APIs
- Configure SSO and provisioning
- Use gallery apps and Application Proxy

---

## Sub-Lessons

### [6.1 Application Concepts](./6.1-application-concepts/README.md)
Understanding the difference between App Registrations and Enterprise Applications, their relationship, and when to use each.

### [6.2 Registering Applications](./6.2-registering-applications/README.md)
How to register applications, choose account types, configure redirect URIs, and understand the identifiers received.

### [6.3 Authentication Settings](./6.3-authentication-settings/README.md)
Configure platform-specific authentication settings, implicit grant options, and logout configuration.

### [6.4 Credentials Management](./6.4-credentials-management/README.md)
Create and manage client secrets and certificates, implement secure storage, and plan credential rotation.

### [6.5 API Permissions](./6.5-api-permissions/README.md)
Understand delegated vs application permissions, request permissions, and manage admin consent.

### [6.6 OAuth/OIDC Flows](./6.6-oauth-oidc-flows/README.md)
Implement authorization code flow, client credentials, device code, and use MSAL libraries.

### [6.7 Exposing APIs](./6.7-exposing-apis/README.md)
Configure your application as an API, define scopes, pre-authorize clients, and validate tokens.

### [6.8 Enterprise Applications](./6.8-enterprise-applications/README.md)
Configure user assignment, SSO settings, provisioning, and manage application properties.

### [6.9 Gallery Apps & App Proxy](./6.9-gallery-apps-app-proxy/README.md)
Add applications from the gallery, configure SAML SSO, and set up Application Proxy for on-premises apps.

---

## Key Concepts

### Application Objects

| Object | Also Known As | Purpose |
|--------|---------------|---------|
| App Registration | Application Object | Global definition (developer) |
| Enterprise Application | Service Principal | Tenant-specific access control |

### Authentication Protocols

| Protocol | Purpose | Use Case |
|----------|---------|----------|
| OAuth 2.0 | Authorization | API access |
| OpenID Connect | Authentication | User sign-in |
| SAML | SSO | Enterprise applications |

### Permission Types

| Type | Description | Consent |
|------|-------------|---------|
| Delegated | On behalf of user | User or Admin |
| Application | App acts alone | Admin only |

---

## Quick Reference

### Portal Navigation

| Feature | Path |
|---------|------|
| App Registrations | Entra ID → App registrations |
| Enterprise Apps | Entra ID → Enterprise applications |
| Application Proxy | Entra ID → Application proxy |
| App Gallery | Enterprise apps → New → Browse gallery |

### Key PowerShell Commands

```powershell
# Connect to Graph
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# List app registrations
Get-MgApplication -All | Select-Object DisplayName, AppId

# List enterprise applications
Get-MgServicePrincipal -All | Select-Object DisplayName, AppId

# Create new application
New-MgApplication -DisplayName "My App" -SignInAudience "AzureADMyOrg"
```

---

## Best Practices Summary

### Security
- [ ] Use certificates over secrets in production
- [ ] Store credentials in Azure Key Vault
- [ ] Request minimum necessary permissions
- [ ] Enable "Assignment required" for sensitive apps

### Development
- [ ] Use MSAL libraries for authentication
- [ ] Implement token caching
- [ ] Use PKCE for public clients
- [ ] Validate tokens in APIs

### Operations
- [ ] Plan credential rotation before expiration
- [ ] Monitor application sign-in logs
- [ ] Regular permission reviews
- [ ] Document all applications

---

## Hands-On Exercises

1. Register a new single-tenant application
2. Configure authentication for a web platform
3. Create a client secret and store in Key Vault
4. Add Microsoft Graph permissions and grant consent
5. Implement authorization code flow with MSAL
6. Create and expose an API with custom scopes
7. Configure user assignment on enterprise application
8. Add a gallery application with SAML SSO
9. (Advanced) Set up Application Proxy for on-premises app

---

## Additional Resources

- [App Registration Overview](https://learn.microsoft.com/en-us/entra/identity-platform/application-model)
- [Microsoft Graph Permissions](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [MSAL Documentation](https://learn.microsoft.com/en-us/entra/identity-platform/msal-overview)
- [OAuth 2.0 Flows](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
- [Application Proxy](https://learn.microsoft.com/en-us/entra/identity/app-proxy/application-proxy)

---

## Navigation

← [Lesson 05: Conditional Access](../lesson-05-conditional-access/README.md) | [Lesson 07: SSO Configuration →](../lesson-07-sso-configuration/README.md)
