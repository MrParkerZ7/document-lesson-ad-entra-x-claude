# 6.2 Registering Applications

## Overview

Registering an application in Microsoft Entra ID creates the identity for your application. This lesson walks through the registration process, explaining account types, redirect URIs, and the information you receive after registration.

## Learning Objectives

- Register a new application in the Azure portal
- Understand supported account types and when to use each
- Configure redirect URIs correctly
- Understand the identifiers received after registration

---

## Accessing App Registrations

### Portal Navigation

```
Microsoft Entra ID → App registrations → New registration
```

### Registration Form

When creating a new app registration, you provide:

| Field | Required | Description |
|-------|----------|-------------|
| Name | Yes | Display name for the application |
| Supported account types | Yes | Who can use this application |
| Redirect URI | No | Where auth responses are sent |

---

## Supported Account Types

### Option 1: Single Tenant

```yaml
Setting: "Accounts in this organizational directory only"
Audience: AzureADMyOrg

Use when:
  - Internal line-of-business applications
  - Apps for employees only
  - Sensitive internal tools

Example: HR management system, internal wiki
```

### Option 2: Multi-Tenant

```yaml
Setting: "Accounts in any organizational directory"
Audience: AzureADMultipleOrgs

Use when:
  - SaaS applications for other organizations
  - Partner integrations
  - B2B collaboration tools

Example: Project management SaaS, collaboration platform
```

### Option 3: Multi-Tenant + Personal

```yaml
Setting: "Accounts in any organizational directory and personal Microsoft accounts"
Audience: AzureADandPersonalMicrosoftAccount

Use when:
  - Consumer-facing applications
  - Apps for both work and personal users
  - Public applications

Example: Social app, public file sharing service
```

### Option 4: Personal Only

```yaml
Setting: "Personal Microsoft accounts only"
Audience: PersonalMicrosoftAccount

Use when:
  - Consumer-only applications
  - Gaming applications
  - Xbox, Outlook.com integrations

Example: Gaming leaderboard, consumer photo app
```

### Comparison Table

| Account Type | Work/School | Personal | External Tenants |
|--------------|-------------|----------|------------------|
| Single Tenant | ✅ (your tenant) | ❌ | ❌ |
| Multi-Tenant | ✅ (all tenants) | ❌ | ✅ |
| Multi-Tenant + Personal | ✅ | ✅ | ✅ |
| Personal Only | ❌ | ✅ | ❌ |

---

## Redirect URIs

### What is a Redirect URI?

The URL where Entra ID sends authentication responses (tokens or codes).

### Types by Platform

| Platform | Example Redirect URI |
|----------|---------------------|
| Web | `https://app.contoso.com/auth/callback` |
| SPA | `https://spa.contoso.com` |
| Mobile/Desktop | `msauth://com.contoso.app` |
| Native | `https://login.microsoftonline.com/common/oauth2/nativeclient` |

### Redirect URI Rules

```yaml
Requirements:
  - Must be HTTPS (except localhost)
  - Maximum 256 characters
  - Case-sensitive
  - No wildcards in path
  - No fragments (#)

Allowed for development:
  - http://localhost:3000
  - http://127.0.0.1:8080

Production:
  - https://app.contoso.com/callback
  - https://api.contoso.com/auth/complete
```

### Adding Multiple URIs

```yaml
Development:
  - http://localhost:3000/auth/callback
  - http://localhost:5000/auth/callback

Staging:
  - https://staging.contoso.com/auth/callback

Production:
  - https://app.contoso.com/auth/callback
  - https://www.contoso.com/auth/callback
```

---

## After Registration

### Identifiers Received

After successful registration, you receive:

| Identifier | Description | Usage |
|------------|-------------|-------|
| Application (client) ID | Unique identifier for the app | Used in authentication requests |
| Directory (tenant) ID | Your Entra ID tenant | Used in endpoint URLs |
| Object ID | Internal resource identifier | Used in Graph API for this app |

### Example Values

```yaml
Application (client) ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
Directory (tenant) ID: t1e2n3a4-n5t6-7890-wxyz-ab1234567890
Object ID: o1b2j3e4-c5t6-7890-qrst-uv1234567890
```

### What to Do Next

After registration:

1. **Configure Authentication** - Add platform configurations
2. **Add Credentials** - Create secrets or upload certificates
3. **Set Permissions** - Request API permissions
4. **Configure Branding** - Add logo and publisher info

---

## PowerShell Registration

### Register Application

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# Create new application
$appParams = @{
    DisplayName = "Contoso CRM"
    SignInAudience = "AzureADMyOrg"
    Web = @{
        RedirectUris = @(
            "https://crm.contoso.com/auth/callback"
            "http://localhost:3000/auth/callback"
        )
    }
}

$app = New-MgApplication @appParams
$app | Select-Object DisplayName, AppId, Id
```

### Output

```
DisplayName  AppId                                Id
-----------  -----                                --
Contoso CRM  a1b2c3d4-e5f6-7890-abcd-ef12345...  o1b2j3e4-c5t6-7890-qrst-uv12...
```

### Register Multi-Tenant App

```powershell
$appParams = @{
    DisplayName = "Contoso SaaS"
    SignInAudience = "AzureADMultipleOrgs"
    Web = @{
        RedirectUris = @("https://saas.contoso.com/callback")
    }
}

$app = New-MgApplication @appParams
```

### Register SPA

```powershell
$appParams = @{
    DisplayName = "Contoso SPA"
    SignInAudience = "AzureADMyOrg"
    Spa = @{
        RedirectUris = @(
            "http://localhost:3000"
            "https://spa.contoso.com"
        )
    }
}

$app = New-MgApplication @appParams
```

---

## Best Practices

### Naming Conventions

```yaml
Format: [Company]-[Environment]-[AppName]-[Type]

Examples:
  - Contoso-Prod-CRM-Web
  - Contoso-Dev-Portal-API
  - Contoso-Test-Mobile-Client

Benefits:
  - Easy to identify in large tenant
  - Clear environment separation
  - Simplifies auditing
```

### Redirect URI Planning

```yaml
Do:
  - Plan all environments upfront
  - Use consistent callback paths
  - Document all URIs

Don't:
  - Add unnecessary URIs
  - Use wildcard patterns
  - Share URIs across apps
```

### Account Type Selection

```yaml
Default to Single Tenant:
  - More secure
  - Simpler consent model
  - Easier to manage

Only use Multi-Tenant when:
  - Building SaaS application
  - Enabling B2B access
  - Intentionally public
```

---

## Common Scenarios

### Scenario 1: Internal Web Application

```yaml
Name: Contoso HR Portal
Account Type: Single tenant
Redirect URIs:
  - https://hr.contoso.com/auth/callback
  - http://localhost:3000/auth/callback
```

### Scenario 2: SaaS Application

```yaml
Name: Contoso Project Management
Account Type: Multi-tenant
Redirect URIs:
  - https://pm.contoso.com/auth/callback
  - https://*.customers.contoso.com/auth/callback  # Using custom domains
```

### Scenario 3: Mobile Application

```yaml
Name: Contoso Mobile
Account Type: Single tenant
Redirect URIs:
  - msauth://com.contoso.mobile
  - msauth://com.contoso.mobile/callback
```

---

## Troubleshooting

### Error: AADSTS50011 - Invalid Reply URL

**Cause**: Redirect URI doesn't match registered URIs

**Solution**:
1. Check exact URI including trailing slashes
2. Verify HTTPS vs HTTP
3. Check for typos
4. Ensure URI is added to app registration

### Error: AADSTS700016 - Application Not Found

**Cause**: Wrong tenant or client ID

**Solution**:
1. Verify Application (client) ID
2. Confirm correct tenant
3. Check if app was deleted

### Error: AADSTS50020 - User Account Not in Tenant

**Cause**: Single-tenant app accessed from different tenant

**Solution**:
1. Change to multi-tenant if needed
2. Invite user as guest
3. Verify correct account type

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Registration form and options
- Account type decision flow
- Redirect URI configuration

---

## Key Takeaways

- Choose account type based on your application's audience
- Plan redirect URIs for all environments before registration
- Client ID is the primary identifier used in code
- Use consistent naming conventions
- Default to single-tenant for security
- Redirect URIs are case-sensitive and must match exactly

---

## Navigation

← [6.1 Application Concepts](../6.1-application-concepts/README.md) | [6.3 Authentication Settings →](../6.3-authentication-settings/README.md)
