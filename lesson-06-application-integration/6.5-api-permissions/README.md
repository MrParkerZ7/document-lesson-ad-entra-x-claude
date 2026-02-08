# 6.5 API Permissions

## Overview

API permissions determine what resources your application can access. This lesson covers delegated and application permissions, the consent framework, and how to properly request and grant permissions.

## Learning Objectives

- Understand delegated vs application permissions
- Request permissions for Microsoft Graph and other APIs
- Configure admin consent requirements
- Implement the principle of least privilege
- Troubleshoot permission-related issues

---

## Types of Permissions

### Delegated Permissions

Application acts **on behalf of a signed-in user**:

```yaml
Characteristics:
  - Requires user sign-in
  - Combined with user's access rights
  - User cannot grant more than they have
  - Can be consented by user or admin

Example: User.Read
  - App can read the signed-in user's profile
  - Cannot read other users' profiles
```

### Application Permissions

Application acts **as itself** (no user):

```yaml
Characteristics:
  - No user sign-in required
  - Direct API access
  - Typically powerful/broad access
  - Always requires admin consent

Example: User.Read.All
  - App can read ALL users in directory
  - Used by background services, daemons
```

### Comparison

| Aspect | Delegated | Application |
|--------|-----------|-------------|
| User required | Yes | No |
| Consent | User or Admin | Admin only |
| Scope | User's permissions | All allowed resources |
| Use case | Interactive apps | Daemons, services |
| Token type | Access token (user context) | Access token (app context) |

---

## Common Microsoft Graph Permissions

### Delegated Permissions

| Permission | Description | Consent |
|------------|-------------|---------|
| User.Read | Read signed-in user profile | User |
| User.ReadBasic.All | Read all users' basic profiles | User |
| Mail.Read | Read user's mail | User |
| Mail.Send | Send mail as user | User |
| Calendars.ReadWrite | Full access to calendars | User |
| Files.Read | Read user's files | User |
| Directory.Read.All | Read directory data | Admin |
| Directory.AccessAsUser.All | Access directory as user | Admin |

### Application Permissions

| Permission | Description | Consent |
|------------|-------------|---------|
| User.Read.All | Read all users | Admin |
| User.ReadWrite.All | Read/write all users | Admin |
| Mail.Read | Read all mailboxes | Admin |
| Mail.Send | Send mail as any user | Admin |
| Directory.Read.All | Read directory data | Admin |
| Application.ReadWrite.All | Manage all apps | Admin |

---

## Requesting Permissions

### Portal Navigation

```
App registrations → [Your App] → API permissions → Add a permission
```

### Adding a Permission

```yaml
Step 1: Select API
  - Microsoft Graph (most common)
  - APIs my organization uses
  - APIs in other organizations
  - My APIs (custom APIs)

Step 2: Select Permission Type
  - Delegated permissions (user context)
  - Application permissions (app-only)

Step 3: Select Specific Permissions
  - Search or browse
  - Check required permissions
  - Click Add permissions
```

### Example: Adding Graph Permissions

```yaml
For a user-facing app:
  1. Add a permission → Microsoft Graph
  2. Delegated permissions
  3. Select:
     - User.Read
     - Mail.Read
     - Calendars.Read

For a background service:
  1. Add a permission → Microsoft Graph
  2. Application permissions
  3. Select:
     - User.Read.All
     - Mail.ReadBasic.All
```

---

## Consent Framework

### Types of Consent

| Type | Who | When | Scope |
|------|-----|------|-------|
| User consent | End user | First use | User-consentable permissions |
| Admin consent | Admin | Explicit grant | All permissions |
| Pre-authorization | App owner | Setup time | Known client apps |

### User Consent Flow

```
User attempts to sign in
        │
        ▼
Entra ID checks required permissions
        │
        ▼
User sees consent prompt
        │
        ▼
User approves → Access granted
User denies → Access blocked
```

### Admin Consent Flow

```
Admin navigates to API permissions
        │
        ▼
Reviews requested permissions
        │
        ▼
Clicks "Grant admin consent for [tenant]"
        │
        ▼
All users can use app without individual consent
```

### Granting Admin Consent

```
API permissions → Grant admin consent for [Tenant Name]
```

**Required for:**
- Application permissions (always)
- Sensitive delegated permissions
- Tenant-wide consent

---

## Consent Settings

### User Consent Settings

Configure who can consent:

```
Microsoft Entra ID → Enterprise applications
→ Consent and permissions → User consent settings
```

Options:
```yaml
Do not allow user consent:
  - All permissions require admin

Allow user consent for apps from verified publishers:
  - Safer, recommended

Allow user consent for all apps:
  - Less restrictive
```

### Risk-Based Consent

```yaml
Low-risk permissions (typically user-consentable):
  - User.Read
  - openid, profile, email
  - offline_access

Higher-risk permissions (typically admin consent):
  - Mail.Read
  - Files.ReadWrite
  - Directory.Read.All
```

---

## PowerShell: Manage Permissions

### View Requested Permissions

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

$app = Get-MgApplication -Filter "displayName eq 'Contoso CRM'"

# Delegated permissions
$app.RequiredResourceAccess | ForEach-Object {
    $resourceAppId = $_.ResourceAppId
    $_.ResourceAccess | Where-Object { $_.Type -eq "Scope" } | ForEach-Object {
        [PSCustomObject]@{
            ResourceAppId = $resourceAppId
            PermissionId = $_.Id
            Type = "Delegated"
        }
    }
}

# Application permissions
$app.RequiredResourceAccess | ForEach-Object {
    $_.ResourceAccess | Where-Object { $_.Type -eq "Role" } | ForEach-Object {
        [PSCustomObject]@{
            PermissionId = $_.Id
            Type = "Application"
        }
    }
}
```

### Add Permissions

```powershell
$graphId = "00000003-0000-0000-c000-000000000000" # Microsoft Graph

# Get permission IDs
$graphSp = Get-MgServicePrincipal -Filter "appId eq '$graphId'"
$userReadPermission = $graphSp.Oauth2PermissionScopes |
    Where-Object { $_.Value -eq "User.Read" }

# Add permission to app
$permission = @{
    ResourceAppId = $graphId
    ResourceAccess = @(
        @{
            Id = $userReadPermission.Id
            Type = "Scope"  # Delegated
        }
    )
}

Update-MgApplication -ApplicationId $appId -RequiredResourceAccess $permission
```

### Grant Admin Consent

```powershell
# Get service principal
$sp = Get-MgServicePrincipal -Filter "appId eq '$clientId'"

# Create OAuth2 permission grant
$grant = @{
    ClientId = $sp.Id
    ConsentType = "AllPrincipals"
    ResourceId = $graphSp.Id
    Scope = "User.Read Mail.Read"
}

New-MgOauth2PermissionGrant -BodyParameter $grant
```

---

## Best Practices

### Principle of Least Privilege

```yaml
Do:
  - Request only needed permissions
  - Use delegated when user context available
  - Start minimal, add as needed
  - Document why each permission is needed

Don't:
  - Request "just in case" permissions
  - Use application permissions for user apps
  - Request .All permissions when not needed
```

### Permission Categories

```yaml
Minimal (User.Read, openid, profile):
  - Basic user info
  - Most apps need these
  - Low risk

Moderate (Mail.Read, Calendar.Read):
  - Access user data
  - User can consent
  - Document need

Sensitive (Directory.Read.All, Mail.ReadWrite):
  - Broad access
  - Requires admin consent
  - Regular review needed

Administrative (.All permissions):
  - Full access to resource
  - Strict justification required
  - Audit usage
```

### Review Checklist

- [ ] Do I need this permission?
- [ ] Is delegated sufficient, or do I need application?
- [ ] Is there a more limited permission available?
- [ ] Have I documented why this permission is needed?
- [ ] Will admin consent be required?

---

## Troubleshooting

### AADSTS65001: No Permission to Access

**Cause**: App doesn't have required permission

**Solution**:
1. Add required permission to app registration
2. Grant admin consent if needed
3. User must re-consent if permissions changed

### AADSTS50105: User Not Assigned

**Cause**: User assignment required but user not assigned

**Solution**:
1. Assign user to enterprise application
2. Or disable "Assignment required" setting

### Insufficient Privileges in API Call

**Cause**: Token doesn't have required scope

**Solution**:
1. Request additional scopes in token request
2. Check consented permissions vs requested
3. Verify application vs delegated context

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Delegated vs Application permissions flow
- Consent framework
- Permission request process

---

## Key Takeaways

- Use delegated permissions for user-interactive apps
- Use application permissions for background services only
- Request minimum necessary permissions
- Application permissions always require admin consent
- Document and justify all permissions requested
- Regular review and remove unused permissions

---

## Navigation

← [6.4 Credentials Management](../6.4-credentials-management/README.md) | [6.6 OAuth/OIDC Flows →](../6.6-oauth-oidc-flows/README.md)
