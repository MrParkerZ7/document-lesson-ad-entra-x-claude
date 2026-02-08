# 6.8 Enterprise Applications

## Overview

Enterprise Applications (Service Principals) are the tenant-specific instances of applications. While App Registrations define what an app can do, Enterprise Applications control who can use it and how they access it. This lesson covers user assignment, SSO configuration, and provisioning.

## Learning Objectives

- Understand the Enterprise Applications blade
- Configure user and group assignments
- Set up Single Sign-On (SSO)
- Configure user provisioning
- Manage application properties and access

---

## Accessing Enterprise Applications

### Portal Navigation

```
Microsoft Entra ID → Enterprise applications
```

### Application Categories

| Category | Description |
|----------|-------------|
| Enterprise Applications | Business applications |
| Microsoft Applications | Office 365, Azure services |
| Managed Identities | System/User-assigned identities |
| All applications | Everything combined |

---

## User and Group Assignment

### Why Assignment Matters

```yaml
Assignment Required = Yes:
  - Only assigned users can access
  - Better security control
  - Audit who has access
  - Recommended for internal apps

Assignment Required = No:
  - All users in tenant can access
  - Common for collaboration tools
  - Less administrative overhead
```

### Configuring Assignment Requirement

```
Enterprise applications → [App] → Properties
→ Assignment required? → Yes/No
```

### Adding Assignments

```
Enterprise applications → [App] → Users and groups → Add user/group
```

### Assignment Options

| Type | Description |
|------|-------------|
| User | Individual user assignment |
| Group | All members get access |
| Role | Assign specific app role |

### PowerShell: Manage Assignments

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All", "AppRoleAssignment.ReadWrite.All"

# Get service principal
$sp = Get-MgServicePrincipal -Filter "displayName eq 'Contoso CRM'"

# Assign user
$user = Get-MgUser -UserId "user@contoso.com"
$assignment = @{
    PrincipalId = $user.Id
    ResourceId = $sp.Id
    AppRoleId = "00000000-0000-0000-0000-000000000000" # Default role
}
New-MgUserAppRoleAssignment -UserId $user.Id -BodyParameter $assignment

# Assign group
$group = Get-MgGroup -Filter "displayName eq 'CRM Users'"
$groupAssignment = @{
    PrincipalId = $group.Id
    ResourceId = $sp.Id
    AppRoleId = "00000000-0000-0000-0000-000000000000"
}
New-MgGroupAppRoleAssignment -GroupId $group.Id -BodyParameter $groupAssignment
```

---

## Single Sign-On Configuration

### SSO Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| OIDC/OAuth | Modern authentication | New applications |
| SAML | Enterprise standard | Legacy enterprise apps |
| Password-based | Credential vaulting | Apps without SSO support |
| Linked | Redirect to URL | Custom identity providers |
| Disabled | No SSO | App doesn't need SSO |

### Configuring SSO

```
Enterprise applications → [App] → Single sign-on
→ Select method
```

### SAML SSO Configuration

For SAML-based applications:

```yaml
Basic SAML Configuration:
  Identifier (Entity ID): https://app.contoso.com
  Reply URL (ACS URL): https://app.contoso.com/sso/saml
  Sign on URL: https://app.contoso.com/login
  Relay State: (optional)
  Logout URL: https://app.contoso.com/logout

User Attributes & Claims:
  Required:
    - Unique User Identifier (Name ID): user.userprincipalname
  Additional:
    - emailaddress: user.mail
    - givenname: user.givenname
    - surname: user.surname
    - name: user.displayname
```

### SAML Certificate

```yaml
Signing Certificate:
  - Automatically generated
  - Download for app configuration
  - Renew before expiration

Options:
  - Sign SAML response
  - Sign SAML assertion
  - Sign both

Download formats:
  - Federation Metadata XML
  - Certificate (Base64)
  - Certificate (Raw)
```

---

## User Provisioning

### What is Provisioning?

Automated user lifecycle management between Entra ID and the application.

### Provisioning Modes

| Mode | Description |
|------|-------------|
| Automatic | Entra ID syncs users/groups to app |
| Manual | Admin creates users manually in app |
| None | No provisioning needed |

### Configuring Automatic Provisioning

```
Enterprise applications → [App] → Provisioning
→ Provisioning Mode: Automatic
```

### Provisioning Settings

```yaml
Admin Credentials:
  Tenant URL: https://app.contoso.com/scim
  Secret Token: [SCIM API token from app]

Mappings:
  Users:
    - userName → userPrincipalName
    - emails[type eq "work"].value → mail
    - displayName → displayName
    - active → accountEnabled

  Groups:
    - displayName → displayName
    - members → members

Scope:
  - Sync all users and groups
  - Sync only assigned users and groups (recommended)
```

### Provisioning Status

| Status | Description |
|--------|-------------|
| On | Actively syncing |
| Off | Provisioning disabled |
| Quarantine | Errors detected, syncing paused |

### Provisioning Logs

```
Enterprise applications → [App] → Provisioning → Provisioning logs
```

View:
- User creation/updates
- Attribute changes
- Errors and failures

---

## Application Properties

### Key Properties

```
Enterprise applications → [App] → Properties
```

| Property | Description |
|----------|-------------|
| Name | Display name |
| Logo | Application icon |
| Homepage URL | Main app URL |
| Enabled for users to sign-in | Allow/block sign-in |
| Visible to users | Show in My Apps portal |
| Assignment required | Restrict to assigned users |
| Notes | Admin documentation |

### Controlling Access

```yaml
To block access temporarily:
  Enabled for users to sign-in: No

To hide from users but allow access:
  Visible to users: No

To restrict to specific users:
  Assignment required: Yes
```

---

## Self-Service Application Access

### Enabling Self-Service

Allow users to request access without admin intervention:

```
Enterprise applications → [App] → Self-service
```

Configuration:
```yaml
Allow users to request access: Yes
Group to which users should be added: [Select group]
Require approval: Yes/No
Approvers: [Select approvers]
```

### Access Request Flow

```
User requests access in My Apps
        │
        ▼
Request sent to approvers (if required)
        │
        ▼
Approver approves/denies
        │
        ▼
User added to group → Access granted
```

---

## My Apps Portal

### What is My Apps?

The My Apps portal (https://myapps.microsoft.com) shows users their assigned applications.

### Application Visibility

For an app to appear in My Apps:
1. User must be assigned (if required)
2. "Visible to users" must be Yes
3. App must have a homepage URL

### Collections

Organize apps into collections for users:

```
Enterprise applications → [App] → Collections
→ Add to collection
```

---

## PowerShell Management

### List Enterprise Applications

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

# All enterprise apps
Get-MgServicePrincipal -All | Select-Object DisplayName, AppId

# Filter by name
Get-MgServicePrincipal -Filter "displayName eq 'Contoso CRM'"

# Get assignments
$sp = Get-MgServicePrincipal -Filter "displayName eq 'Contoso CRM'"
Get-MgServicePrincipalAppRoleAssignedTo -ServicePrincipalId $sp.Id
```

### Disable Application

```powershell
$sp = Get-MgServicePrincipal -Filter "displayName eq 'Old App'"

Update-MgServicePrincipal -ServicePrincipalId $sp.Id -AccountEnabled $false
```

### Remove User Assignment

```powershell
$sp = Get-MgServicePrincipal -Filter "displayName eq 'Contoso CRM'"
$user = Get-MgUser -UserId "user@contoso.com"

$assignment = Get-MgUserAppRoleAssignment -UserId $user.Id |
    Where-Object { $_.ResourceId -eq $sp.Id }

Remove-MgUserAppRoleAssignment -UserId $user.Id -AppRoleAssignmentId $assignment.Id
```

---

## Best Practices

### Access Control

```yaml
Do:
  - Enable assignment requirement for internal apps
  - Use groups for assignment (easier to manage)
  - Regular access reviews
  - Document access policies

Don't:
  - Assign users individually (use groups)
  - Leave assignment open for sensitive apps
  - Forget to remove access when users leave
```

### SSO

```yaml
Recommendations:
  - Use OIDC/OAuth for new apps
  - SAML for enterprise apps without OIDC
  - Password-based only when no other option
  - Test SSO thoroughly before rollout
```

### Provisioning

```yaml
Best Practices:
  - Start with assigned users only
  - Test with small group first
  - Monitor provisioning logs
  - Handle provisioning errors promptly
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Enterprise Application configuration areas
- User assignment flow
- SSO and provisioning setup

---

## Key Takeaways

- Enterprise Applications control access and SSO at tenant level
- Enable "Assignment required" for sensitive applications
- Use groups for assignment instead of individual users
- Configure SSO based on application capabilities
- Set up provisioning for automatic user lifecycle
- Regular access reviews maintain security

---

## Navigation

← [6.7 Exposing APIs](../6.7-exposing-apis/README.md) | [6.9 Gallery Apps & App Proxy →](../6.9-gallery-apps-app-proxy/README.md)
