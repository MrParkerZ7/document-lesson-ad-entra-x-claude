# 7.5 My Apps Portal

## Overview

The My Apps portal (myapps.microsoft.com) is a web-based application launcher that provides users with centralized access to all their SSO-enabled applications. This lesson covers portal configuration, collections, customization, and user management features.

## Learning Objectives

- Understand My Apps portal capabilities
- Configure and customize the portal
- Create and manage app collections
- Enable self-service features
- Customize portal branding and layout

---

## My Apps Portal Overview

### Portal URL

**Primary URL**: https://myapps.microsoft.com

Alternative URLs:
- https://myapplications.microsoft.com
- https://account.activedirectory.windowsazure.com/r#/applications

### What Users Can Do

| Feature | Description |
|---------|-------------|
| Launch applications | Single-click access to SSO apps |
| View collections | Organized app groups |
| Request access | Request new app access |
| Manage groups | Self-service group management |
| Update profile | Manage security info |
| View activity | Recent sign-in history |

### Portal Components

```yaml
My Apps Portal:
  - Application tiles
  - Collections (categories)
  - Search functionality
  - Favorites
  - Recently used apps
  - Profile menu

Account Portal:
  - Security info management
  - Password change
  - Devices
  - Organizations
  - Subscriptions
```

---

## Application Assignment

### How Apps Appear in My Apps

Applications appear in My Apps when:

```yaml
Assignment Required = Yes:
  - User must be assigned directly
  - Or member of assigned group
  - No app tile without assignment

Assignment Required = No:
  - All users see the app
  - No explicit assignment needed
  - Good for company-wide apps
```

### Configuring Assignment

```
Enterprise applications → [App] → Properties
→ Assignment required?: Yes/No
```

### Assigning Users and Groups

```
Enterprise applications → [App] → Users and groups
→ Add user/group
```

```yaml
Assignment options:
  - Individual users
  - Security groups
  - Microsoft 365 groups
  - Dynamic groups

Role assignment:
  - Default access
  - App-defined roles
```

### Hiding Apps from My Apps

To hide an app from the portal:

```
Enterprise applications → [App] → Properties
→ Visible to users?: No
```

```yaml
Use cases for hiding:
  - Backend service apps
  - API-only applications
  - Apps accessed via other portals
  - Apps in development
```

---

## App Collections

### What are Collections?

Collections organize applications into logical groups in My Apps, making it easier for users to find relevant applications.

### Creating Collections

```
Microsoft Entra ID → Enterprise applications
→ App launchers → Collections → New collection
```

```yaml
Collection Configuration:
  Name: Finance Applications
  Description: Tools for the finance team

  Add applications:
    - SAP
    - Concur
    - Workday Finance
    - Power BI Finance

  Assign users:
    - GRP-SEC-Finance-Users
    - GRP-SEC-Finance-Admins
```

### Collection Best Practices

| Practice | Description |
|----------|-------------|
| Logical grouping | Group by department or function |
| Clear naming | Use descriptive collection names |
| Limit size | 10-15 apps per collection |
| Target audience | Assign to relevant groups |
| Regular review | Update collections as apps change |

### Example Collection Structure

```yaml
Collections:
  HR Applications:
    - Workday
    - SuccessFactors
    - LinkedIn Learning
    Assigned to: GRP-SEC-HR-All

  Sales Tools:
    - Salesforce
    - HubSpot
    - Zoom
    Assigned to: GRP-SEC-Sales-All

  IT Administration:
    - Azure Portal
    - Microsoft 365 Admin
    - ServiceNow
    Assigned to: GRP-SEC-IT-Admins

  Company-Wide:
    - Microsoft 365
    - Teams
    - SharePoint
    Assigned to: All Users
```

### PowerShell: Manage Collections

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# Get all app collections
Get-MgEntitlementManagementConnectedOrganization

# Note: App collections management is primarily done via portal
# Graph API support for collections is limited
```

---

## Portal Customization

### Branding Configuration

```
Microsoft Entra ID → Enterprise applications
→ User settings → My Apps preview settings
```

### Available Customizations

| Element | Options |
|---------|---------|
| Company logo | Upload PNG/JPG (max 215x215) |
| Background | Color or image |
| Banner color | Header color theme |
| Footer text | Custom footer message |
| Links | Custom navigation links |

### Company Branding

```
Microsoft Entra ID → Company branding → Configure
```

```yaml
Branding Elements:
  Sign-in page:
    - Background image
    - Banner logo
    - Square logo
    - Username hint
    - Sign-in page text

  My Apps portal:
    - Inherits company branding
    - Custom collections layout
```

---

## Self-Service Features

### Self-Service Application Access

Enable users to request access to applications:

```
Enterprise applications → [App] → Self-service
→ Allow users to request access: Yes
```

```yaml
Self-Service Configuration:
  Allow users to request access: Yes

  Approvers:
    - Business owner
    - Specific approvers
    - Group owners

  Approval required: Yes

  Email notifications:
    - Notify approvers
    - Notify requester

  Access request form:
    - Business justification required
    - Custom questions
```

### Self-Service Group Management

```
Microsoft Entra ID → Groups → General
→ Self-service group management
```

```yaml
Settings:
  Owners can manage membership: Yes
  Users can request membership: Yes

  Restrictions:
    - Security groups only
    - Microsoft 365 groups
    - Limit number of groups owned
```

### Access Request Workflow

```
1. User finds app in gallery or requests access
          |
2. Request submitted with justification
          |
3. Approver receives email notification
          |
4. Approver reviews and approves/denies
          |
5. User notified of decision
          |
6. If approved, user assigned to app
          |
7. App appears in user's My Apps
```

---

## User Settings

### Portal Configuration for Users

```
Microsoft Entra ID → Enterprise applications → User settings
```

```yaml
Settings:
  Users can register applications: Yes/No
  Users can consent to apps accessing data: Yes/No
  Users can add gallery apps: Yes/No

  My Apps:
    - Collection: Enable/Disable
    - Preview features: Enable/Disable
```

### Launching Behaviors

| Setting | Description |
|---------|-------------|
| Open in new tab | Apps open in new browser tab |
| Open in same tab | Replaces My Apps page |
| Launch directly | For password SSO apps |

---

## My Apps Mobile

### Mobile App Availability

| Platform | App Name |
|----------|----------|
| iOS | Microsoft Authenticator (includes My Apps) |
| Android | Microsoft Authenticator (includes My Apps) |

### Mobile Features

```yaml
Features:
  - Application tiles
  - Touch ID/Face ID authentication
  - Push notifications
  - Offline access to favorites

Configuration:
  - App policies via Intune
  - Conditional Access for mobile
```

---

## Browser Extension

### My Apps Secure Sign-in Extension

Required for password-based SSO:

| Browser | Extension |
|---------|-----------|
| Chrome | My Apps Secure Sign-in Extension |
| Edge | My Apps Secure Sign-in Extension |
| Firefox | My Apps Secure Sign-in Extension |

### Extension Deployment

```yaml
Deployment Options:
  - User self-install from store
  - Group Policy (Chrome/Edge)
  - Intune app deployment
  - Configuration Manager

Group Policy Path (Chrome):
  Computer Configuration
  → Administrative Templates
  → Google Chrome
  → Extensions
  → Configure force-installed extensions
```

---

## Security and Access Control

### Conditional Access for My Apps

```yaml
Recommended Policy:
  Name: Require MFA for My Apps

  Assignments:
    Users: All users
    Cloud apps: My Apps

  Conditions:
    Location: Any
    Device platform: Any

  Access controls:
    Grant: Require MFA
```

### Session Controls

```yaml
Session Controls:
  Sign-in frequency: 8 hours (or as needed)
  Persistent browser session: No

Purpose:
  - Regular re-authentication
  - Limit session duration
  - Require fresh MFA
```

### Audit and Monitoring

```yaml
Audit Logs:
  Location: Microsoft Entra ID → Audit logs

  Key Events:
    - Application access
    - Self-service requests
    - Collection modifications
    - User assignments

Sign-in Logs:
  - My Apps access
  - Application launches
  - Authentication details
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| App not visible | Not assigned | Assign user/group |
| App hidden | Visible to users = No | Enable visibility |
| Wrong collection | Incorrect assignment | Update collection |
| Extension not working | Not installed | Install/enable extension |
| Login loop | SSO misconfiguration | Check SSO settings |

### User Not Seeing Apps

```yaml
Checklist:
  1. User assigned to app?
  2. Group membership correct?
  3. Assignment required setting?
  4. App visible to users?
  5. Collection assigned to user?
  6. License requirements met?
  7. Conditional Access blocking?
```

### KQL Query: My Apps Access

```kusto
SignInLogs
| where TimeGenerated > ago(7d)
| where AppDisplayName == "My Apps"
| project TimeGenerated, UserPrincipalName, ResultType,
          IPAddress, Location
| summarize Count = count() by UserPrincipalName
| order by Count desc
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- My Apps portal layout
- Collections organization
- Self-service workflow
- Access request flow

---

## Key Takeaways

- My Apps portal provides centralized SSO application access
- Collections organize apps into logical groups for users
- Self-service features enable users to request access
- Portal branding inherits from company branding settings
- Browser extension required for password-based SSO
- Conditional Access can secure My Apps access
- Mobile access available through Authenticator app

---

## Navigation

[7.4 Password & Linked SSO](../7.4-password-linked-sso/README.md) | [7.6 SSO for On-Premises](../7.6-sso-on-premises/README.md)
