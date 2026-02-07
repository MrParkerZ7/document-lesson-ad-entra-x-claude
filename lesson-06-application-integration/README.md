# Lesson 06: Application Integration

## Overview

This lesson covers how to integrate applications with Microsoft Entra ID for authentication and authorization. You'll learn about app registrations, enterprise applications, and OAuth/OIDC protocols.

## Learning Objectives

By the end of this lesson, you will:
- Understand the difference between app registrations and enterprise apps
- Register and configure applications
- Implement OAuth 2.0 and OpenID Connect flows
- Configure permissions and consent
- Manage application credentials

---

## 1. Application Concepts

### App Registration vs Enterprise Application

| Aspect | App Registration | Enterprise Application |
|--------|-----------------|----------------------|
| Purpose | Development/Configuration | Access/Assignment |
| Scope | Global definition | Tenant-specific instance |
| Created by | Developer | Automatically or admin |
| Manages | Credentials, permissions, branding | Users, groups, SSO settings |

### Relationship Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    App Registration                          │
│                    (Application Object)                      │
│                                                              │
│   • Client ID                                                │
│   • Client secrets/certificates                              │
│   • Redirect URIs                                            │
│   • API permissions                                          │
│   • Expose API settings                                      │
└─────────────────────────────────────────────────────────────┘
                              │
                    Creates (automatically)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Enterprise Application                      │
│                  (Service Principal)                         │
│                                                              │
│   • User/Group assignments                                   │
│   • SSO configuration                                        │
│   • Provisioning settings                                    │
│   • Conditional Access                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Registering Applications

### Access App Registrations

```
Microsoft Entra ID → App registrations → New registration
```

### Registration Form

| Field | Description | Example |
|-------|-------------|---------|
| Name | Display name | Contoso CRM |
| Supported account types | Who can use this app | Single tenant |
| Redirect URI | Where to send auth responses | https://app.contoso.com/callback |

### Supported Account Types

```yaml
Single tenant:
  - "Accounts in this organizational directory only"
  - Only users in your tenant
  - Use for internal apps

Multi-tenant:
  - "Accounts in any organizational directory"
  - Users from any Entra ID tenant
  - Use for SaaS applications

Multi-tenant + Personal:
  - "Accounts in any organizational directory and personal Microsoft accounts"
  - Includes consumer accounts
  - Use for public applications

Personal only:
  - "Personal Microsoft accounts only"
  - Xbox, Outlook.com, etc.
```

### After Registration

You receive:
- **Application (client) ID**: Unique identifier
- **Directory (tenant) ID**: Your tenant ID
- **Object ID**: Internal identifier

---

## 3. Application Authentication

### Authentication Settings

```
App registrations → [Your App] → Authentication
```

### Platform Configurations

#### Web Application

```yaml
Platform: Web
Redirect URIs:
  - https://app.contoso.com/auth/callback
  - https://localhost:3000/auth/callback (development)

Front-channel logout URL: https://app.contoso.com/logout

Implicit grant and hybrid flows:
  □ Access tokens (for implicit flow)
  □ ID tokens (for implicit and hybrid flow)
```

#### Single-Page Application (SPA)

```yaml
Platform: Single-page application
Redirect URIs:
  - https://app.contoso.com
  - http://localhost:3000 (development)

Note: Uses PKCE, no implicit grant needed
```

#### Mobile/Desktop Application

```yaml
Platform: Mobile and desktop applications
Redirect URIs:
  - msauth://com.contoso.app
  - https://login.microsoftonline.com/common/oauth2/nativeclient

Allow public client flows: Yes (for ROPC, device code)
```

---

## 4. Credentials

### Client Secrets

```
App registrations → [Your App] → Certificates & secrets → New client secret
```

```yaml
Description: Production API Key
Expires: 24 months (recommended maximum)

# Result
Secret ID: [GUID]
Value: [Copy immediately - shown only once]
Expires: 2026-02-07
```

> **Warning**: Store secrets securely. Use Azure Key Vault in production.

### Certificates (Recommended)

```yaml
Benefits:
  - More secure than secrets
  - Can't be accidentally exposed in logs
  - Hardware-backed options available

Upload:
  - .cer, .pem, or .crt file
  - Contains public key only
  - Private key stays with application
```

#### Generate Self-Signed Certificate (PowerShell)

```powershell
$cert = New-SelfSignedCertificate `
    -Subject "CN=ContosoApp" `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyExportPolicy Exportable `
    -KeySpec Signature `
    -KeyLength 2048 `
    -KeyAlgorithm RSA `
    -HashAlgorithm SHA256 `
    -NotAfter (Get-Date).AddYears(2)

# Export public key for upload
Export-Certificate -Cert $cert -FilePath "ContosoApp.cer"

# Export with private key for application
$password = ConvertTo-SecureString -String "YourPassword" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath "ContosoApp.pfx" -Password $password
```

---

## 5. API Permissions

### Types of Permissions

| Type | Description | Consent |
|------|-------------|---------|
| Delegated | App acts on behalf of user | User or Admin |
| Application | App acts as itself | Admin only |

### Common Microsoft Graph Permissions

```yaml
Delegated:
  - User.Read: Read signed-in user profile
  - User.ReadBasic.All: Read basic profiles
  - Mail.Read: Read user's mail
  - Calendars.Read: Read user's calendars
  - Files.Read: Read user's files

Application:
  - User.Read.All: Read all users
  - Directory.Read.All: Read directory data
  - Mail.ReadWrite: Read/write all mail
  - Application.ReadWrite.All: Manage apps
```

### Adding Permissions

```
App registrations → [Your App] → API permissions → Add a permission
```

```yaml
1. Select API:
   - Microsoft Graph
   - APIs my organization uses
   - My APIs

2. Select permission type:
   - Delegated permissions
   - Application permissions

3. Select permissions:
   - Search or browse
   - Check required permissions

4. Grant admin consent (if required)
```

### Admin Consent

Required for:
- Application permissions
- Sensitive delegated permissions
- Tenant-wide consent

```
App registrations → [Your App] → API permissions
→ Grant admin consent for [Tenant Name]
```

---

## 6. OAuth 2.0 and OpenID Connect

### Protocol Overview

```
OAuth 2.0: Authorization (what can you access?)
OpenID Connect: Authentication (who are you?)
```

### Common Flows

#### Authorization Code Flow (Web Apps)

```
┌──────┐                              ┌──────────┐                    ┌─────────┐
│ User │                              │   App    │                    │ Entra   │
└──┬───┘                              └────┬─────┘                    └────┬────┘
   │                                       │                               │
   │ 1. Click "Sign In"                    │                               │
   │──────────────────────────────────────▶│                               │
   │                                       │                               │
   │ 2. Redirect to authorization endpoint │                               │
   │◀──────────────────────────────────────│                               │
   │                                       │                               │
   │ 3. User authenticates                 │                               │
   │──────────────────────────────────────────────────────────────────────▶│
   │                                       │                               │
   │ 4. Redirect with authorization code   │                               │
   │◀──────────────────────────────────────────────────────────────────────│
   │                                       │                               │
   │ 5. Code sent to app                   │                               │
   │──────────────────────────────────────▶│                               │
   │                                       │                               │
   │                                       │ 6. Exchange code for tokens   │
   │                                       │──────────────────────────────▶│
   │                                       │                               │
   │                                       │ 7. Access + ID tokens         │
   │                                       │◀──────────────────────────────│
   │                                       │                               │
   │ 8. User signed in                     │                               │
   │◀──────────────────────────────────────│                               │
```

#### Client Credentials Flow (Daemon/Service Apps)

```
┌─────────┐                                          ┌─────────┐
│   App   │                                          │ Entra   │
└────┬────┘                                          └────┬────┘
     │                                                    │
     │ 1. Request token (client_id + client_secret)       │
     │───────────────────────────────────────────────────▶│
     │                                                    │
     │ 2. Access token                                    │
     │◀───────────────────────────────────────────────────│
     │                                                    │
     │ 3. Call API with access token                      │
     │───────────────────────────────────────────────────▶│
```

### Endpoints

```yaml
Authorization: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
Token: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Logout: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/logout

Where {tenant} is:
  - Your tenant ID (GUID)
  - Your domain (contoso.onmicrosoft.com)
  - "common" for multi-tenant
  - "organizations" for any work/school
  - "consumers" for personal accounts
```

---

## 7. Code Examples

### Node.js with MSAL

```javascript
// Install: npm install @azure/msal-node

const msal = require('@azure/msal-node');

const config = {
    auth: {
        clientId: "YOUR_CLIENT_ID",
        authority: "https://login.microsoftonline.com/YOUR_TENANT_ID",
        clientSecret: "YOUR_CLIENT_SECRET"
    }
};

const cca = new msal.ConfidentialClientApplication(config);

// Authorization Code Flow
const authCodeUrlParameters = {
    scopes: ["user.read"],
    redirectUri: "http://localhost:3000/redirect"
};

const authUrl = await cca.getAuthCodeUrl(authCodeUrlParameters);
// Redirect user to authUrl

// After redirect, exchange code for tokens
const tokenRequest = {
    code: "AUTHORIZATION_CODE_FROM_REDIRECT",
    scopes: ["user.read"],
    redirectUri: "http://localhost:3000/redirect"
};

const response = await cca.acquireTokenByCode(tokenRequest);
console.log(response.accessToken);
```

### Python with MSAL

```python
# Install: pip install msal

from msal import ConfidentialClientApplication

config = {
    "client_id": "YOUR_CLIENT_ID",
    "authority": "https://login.microsoftonline.com/YOUR_TENANT_ID",
    "client_secret": "YOUR_CLIENT_SECRET"
}

app = ConfidentialClientApplication(
    config["client_id"],
    authority=config["authority"],
    client_credential=config["client_secret"]
)

# Client Credentials Flow
result = app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])

if "access_token" in result:
    print(result["access_token"])
else:
    print(result.get("error"))
```

### C# with MSAL

```csharp
// Install: dotnet add package Microsoft.Identity.Client

using Microsoft.Identity.Client;

var app = ConfidentialClientApplicationBuilder
    .Create("YOUR_CLIENT_ID")
    .WithClientSecret("YOUR_CLIENT_SECRET")
    .WithAuthority("https://login.microsoftonline.com/YOUR_TENANT_ID")
    .Build();

// Client Credentials Flow
var result = await app.AcquireTokenForClient(
    new[] { "https://graph.microsoft.com/.default" })
    .ExecuteAsync();

Console.WriteLine(result.AccessToken);
```

---

## 8. Exposing APIs

### Configure Your API

```
App registrations → [Your App] → Expose an API
```

### Application ID URI

```yaml
Default: api://{client-id}
Custom: https://api.contoso.com
```

### Define Scopes

```yaml
Scope name: access_as_user
Admin consent display name: Access API as user
Admin consent description: Allows the app to access the API on behalf of the user
User consent display name: Access API as you
User consent description: Allow this app to access the API on your behalf
State: Enabled
```

### Authorize Client Applications

Pre-authorize known client apps:

```yaml
Client application: {client-id-of-frontend-app}
Authorized scopes:
  - api://{your-api}/access_as_user
```

---

## 9. Enterprise Applications

### Access Enterprise Apps

```
Microsoft Entra ID → Enterprise applications
```

### Key Functions

| Function | Description |
|----------|-------------|
| User assignment | Control who can access |
| SSO configuration | Configure sign-in method |
| Provisioning | Sync users to the app |
| Properties | Enable/disable access |

### User Assignment

```
Enterprise applications → [App] → Users and groups → Add user/group
```

```yaml
Assignment required: Yes
  - Only assigned users can access
  - Recommended for internal apps

Assignment required: No
  - All users in tenant can access
  - Common for collaboration tools
```

### SSO Configuration

```
Enterprise applications → [App] → Single sign-on
```

SSO Methods:
- **SAML**: Traditional enterprise SSO
- **OpenID Connect/OAuth**: Modern authentication
- **Password-based**: Credential vaulting
- **Linked**: Redirect to another identity provider
- **Disabled**: No SSO

---

## 10. Gallery Applications

### What is the App Gallery?

Pre-integrated applications with:
- Pre-configured SSO settings
- Documentation and tutorials
- Automatic updates
- Support from Microsoft and vendor

### Adding Gallery Apps

```
Enterprise applications → New application → Browse Azure AD Gallery
```

### Popular Gallery Apps

| App | SSO Type | Features |
|-----|----------|----------|
| Salesforce | SAML | SSO, Provisioning |
| ServiceNow | SAML | SSO, Provisioning |
| Zoom | SAML/OIDC | SSO |
| Slack | SAML | SSO, Provisioning |
| AWS | SAML | SSO |
| Google Workspace | SAML | SSO |

### SAML Configuration Example

```yaml
Basic SAML Configuration:
  Identifier (Entity ID): https://app.example.com
  Reply URL (ACS URL): https://app.example.com/sso/saml
  Sign on URL: https://app.example.com/login
  Relay State: (optional)
  Logout URL: https://app.example.com/logout

User Attributes & Claims:
  - user.mail → email
  - user.displayname → name
  - user.userprincipalname → nameidentifier

SAML Signing Certificate:
  - Download Federation Metadata XML
  - Or download Certificate (Base64)
```

---

## 11. Application Proxy

### Purpose

Publish on-premises web applications for external access without VPN.

### Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   External   │────▶│   Entra ID   │────▶│   Connector  │
│    User      │     │   Cloud      │     │  (On-prem)   │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                                          ┌───────▼───────┐
                                          │  Internal App │
                                          │  (On-prem)    │
                                          └───────────────┘
```

### Setup Steps

1. Install Application Proxy Connector on-premises
2. Register application in Entra ID
3. Configure SSO settings
4. Assign users and groups
5. Configure Conditional Access

---

## 12. Best Practices

### Security

- [ ] Use certificates instead of secrets when possible
- [ ] Set short expiration for secrets (6-12 months)
- [ ] Use least privilege permissions
- [ ] Request only needed scopes
- [ ] Implement proper token storage

### Development

- [ ] Use MSAL libraries (not ADAL)
- [ ] Implement token caching
- [ ] Handle token refresh properly
- [ ] Use PKCE for public clients
- [ ] Test with multiple account types

### Operations

- [ ] Monitor app sign-in logs
- [ ] Set up alerts for failures
- [ ] Review permissions periodically
- [ ] Document app purposes
- [ ] Plan credential rotation

---

## Summary

Application integration in Entra ID enables:
- Centralized authentication for all apps
- SSO across cloud and on-premises
- Granular permission control
- Modern OAuth/OIDC protocols
- Enterprise app management

---

## Hands-On Exercise

1. Register a new application
2. Configure authentication settings
3. Add Microsoft Graph permissions
4. Create a client secret
5. Test authentication flow
6. Configure user assignment

---

## Next Lesson

[Lesson 07: SSO Configuration →](../lesson-07-sso-configuration/README.md)

---

## Additional Resources

- [App Registration Overview](https://learn.microsoft.com/en-us/entra/identity-platform/application-model)
- [Microsoft Graph Permissions](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [MSAL Documentation](https://learn.microsoft.com/en-us/entra/identity-platform/msal-overview)
- [OAuth 2.0 Flows](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
