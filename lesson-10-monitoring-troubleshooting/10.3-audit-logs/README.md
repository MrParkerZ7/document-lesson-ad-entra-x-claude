# 10.3 Audit Logs

## Overview

Audit logs in Microsoft Entra ID capture all changes made to your directory, including user management, group operations, application configurations, and administrative actions. These logs are essential for security investigations, compliance reporting, and tracking who made what changes and when.

## Learning Objectives

- Understand audit log categories and what they capture
- Know the key fields in audit log entries
- Identify common audit events and their significance
- Apply filters to find specific audit activities
- Use audit logs for security and compliance scenarios

---

## Audit Log Categories

Audit logs are organized into categories based on the type of resource or activity:

| Category | Description | Example Activities |
|----------|-------------|-------------------|
| **UserManagement** | User lifecycle operations | Create, update, delete users; password changes |
| **GroupManagement** | Group operations | Create groups, add/remove members, change owners |
| **ApplicationManagement** | App registration changes | Register apps, update permissions, delete apps |
| **RoleManagement** | Admin role operations | Assign roles, activate PIM, remove role members |
| **DirectoryManagement** | Tenant-level settings | Update tenant settings, domain changes |
| **Policy** | Policy changes | CA policies, authentication methods, named locations |
| **Authentication** | Auth method changes | Register MFA, update security info |
| **DeviceManagement** | Device operations | Register devices, delete devices, update device state |
| **B2BManagement** | External identity operations | Invite guests, redeem invitations |
| **ProvisioningManagement** | Provisioning configuration | Configure app provisioning, sync settings |

### Category Details

```yaml
UserManagement:
  Activities:
    - Add user
    - Delete user
    - Update user
    - Change user password
    - Reset user password
    - Set password policy
    - Restore user
    - Hard delete user
    - Set license
    - Remove license

GroupManagement:
  Activities:
    - Add group
    - Delete group
    - Add member to group
    - Remove member from group
    - Add owner to group
    - Remove owner from group
    - Update group
    - Add group to group (nesting)

ApplicationManagement:
  Activities:
    - Add application
    - Update application
    - Delete application
    - Add service principal
    - Update service principal
    - Add app role assignment
    - Remove app role assignment
    - Consent to application
    - Add OAuth2PermissionGrant
    - Remove OAuth2PermissionGrant

RoleManagement:
  Activities:
    - Add member to role
    - Remove member from role
    - Add eligible member to role (PIM)
    - Activate PIM role
    - Add role assignment
    - Remove role assignment
    - Update role
```

---

## Key Fields in Audit Logs

| Field | Description | Example Values |
|-------|-------------|----------------|
| **Date** | Timestamp of the action | 2024-01-15 14:32:45 UTC |
| **Service** | Service that logged the event | Core Directory, PIM, Application |
| **Category** | Category of the activity | UserManagement, GroupManagement |
| **Activity** | Specific activity performed | Add user, Delete group |
| **Activity Status** | Result of the action | Success, Failure |
| **Status Reason** | Details if failure occurred | None (success), Permission denied |
| **Initiated By (Actor)** | Who performed the action | User UPN, Service Principal, System |
| **Target(s)** | What was affected | User object, Group object, App |
| **Modified Properties** | Properties that changed | displayName, userType, memberOf |
| **Correlation ID** | Links related events | Same across related operations |

### Actor Types

```yaml
Actor Types:
  User:
    - UPN of the admin user
    - User ID
    - IP address (if available)

  Service Principal:
    - App display name
    - App ID
    - Service principal ID

  System:
    - "Microsoft Identity Protection"
    - "Azure AD Sync"
    - "Self-service"

  Delegated:
    - Application acting on behalf of user
    - Shows both app and user
```

### Target Information

```yaml
Target Object Details:
  For Users:
    - displayName
    - userPrincipalName
    - id (Object ID)
    - userType (Member/Guest)

  For Groups:
    - displayName
    - id
    - groupTypes
    - membershipRule (if dynamic)

  For Applications:
    - displayName
    - appId
    - servicePrincipalId
```

---

## Common Audit Events

### High-Priority Security Events

| Activity | Category | Why It Matters |
|----------|----------|----------------|
| Add member to role | RoleManagement | New admin access granted |
| Remove member from role | RoleManagement | Admin access removed |
| Consent to application | ApplicationManagement | New app permissions granted |
| Add OAuth2PermissionGrant | ApplicationManagement | API permissions delegated |
| Update conditional access policy | Policy | Access controls modified |
| Delete conditional access policy | Policy | Security policy removed |
| Add service principal credentials | ApplicationManagement | New app secret/certificate |
| Reset user password | UserManagement | Password change (potential compromise) |
| Disable account | UserManagement | Account lockout |
| Delete user | UserManagement | User removed from directory |

### Administrative Events

| Activity | Category | Description |
|----------|----------|-------------|
| Add user | UserManagement | New user account created |
| Invite external user | B2BManagement | Guest user invited |
| Add group | GroupManagement | New group created |
| Add member to group | GroupManagement | Group membership changed |
| Add owner to group | GroupManagement | Group ownership changed |
| Add application | ApplicationManagement | New app registered |
| Add service principal | ApplicationManagement | Enterprise app added |
| Update tenant settings | DirectoryManagement | Tenant configuration changed |

### Self-Service Events

| Activity | Category | Description |
|----------|----------|-------------|
| User registered security info | Authentication | MFA method registered |
| Self-service password reset | UserManagement | User reset own password |
| User updated security info | Authentication | MFA method updated |
| User deleted security info | Authentication | MFA method removed |

---

## Filtering Examples

### Find Role Assignment Changes

```yaml
Filter Configuration:
  Date range: Last 30 days
  Category: RoleManagement
  Activity: Add member to role

Expected Results:
  - Who was assigned to which role
  - Who made the assignment
  - When it occurred
```

### Find User Deletions

```yaml
Filter Configuration:
  Date range: Last 7 days
  Category: UserManagement
  Activity: Delete user

Expected Results:
  - Which users were deleted
  - Who performed the deletion
  - Timestamp of deletion
```

### Find Application Consent Grants

```yaml
Filter Configuration:
  Date range: Last 30 days
  Category: ApplicationManagement
  Activity: Consent to application

Expected Results:
  - Which applications received consent
  - What permissions were granted
  - Who granted consent (admin or user)
```

### Find Conditional Access Policy Changes

```yaml
Filter Configuration:
  Date range: Last 30 days
  Category: Policy
  Activity: (Any of the following)
    - Add conditional access policy
    - Update conditional access policy
    - Delete conditional access policy

Expected Results:
  - Policy name affected
  - Type of change
  - Who made the change
```

### Find Password Resets

```yaml
Filter Configuration:
  Date range: Last 24 hours
  Category: UserManagement
  Activity: (Any of the following)
    - Reset user password
    - Change user password
    - Reset password (self-service)

Expected Results:
  - Target user
  - Method used (admin reset vs self-service)
  - Initiator
```

---

## Use Cases for Audit Logs

### Security Investigation

```yaml
Scenario: Suspected Account Compromise
  Steps:
    1. Filter by target user
    2. Check for password changes
    3. Look for MFA method changes
    4. Check for unusual role assignments
    5. Review consent grants
    6. Check device registrations

  Key Events to Find:
    - Reset user password
    - User registered security info
    - Add member to role
    - Consent to application
```

### Compliance Reporting

```yaml
Scenario: Privileged Access Review
  Steps:
    1. Filter Category: RoleManagement
    2. Filter Date: Last 90 days
    3. Export to CSV/Excel
    4. Review role assignments
    5. Validate with business justification

  Report Contents:
    - Role assignment date
    - Role name
    - User assigned
    - Assigned by (admin)
```

### Change Tracking

```yaml
Scenario: Track Group Membership Changes
  Steps:
    1. Filter Category: GroupManagement
    2. Filter Target: [Specific Group]
    3. Review add/remove member events
    4. Identify patterns

  Report Contents:
    - Member added/removed
    - Date of change
    - Who made the change
```

### Application Governance

```yaml
Scenario: Monitor App Consent Activity
  Steps:
    1. Filter Category: ApplicationManagement
    2. Filter Activity: Consent to application
    3. Review permissions granted
    4. Identify risky consents

  Risky Indicators:
    - Consent to unknown apps
    - High-privilege API access
    - User consent (vs admin consent)
```

---

## PowerShell Queries

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All"

# Get recent audit logs
Get-MgAuditLogDirectoryAudit -Top 100 |
    Select-Object ActivityDateTime, ActivityDisplayName,
                  @{N='InitiatedBy';E={$_.InitiatedBy.User.UserPrincipalName}},
                  @{N='Target';E={$_.TargetResources[0].DisplayName}},
                  Result

# Get role assignment changes
Get-MgAuditLogDirectoryAudit -Filter "category eq 'RoleManagement'" -Top 50 |
    Where-Object { $_.ActivityDisplayName -like "*member to role*" }

# Get user deletions
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Delete user'" -Top 50

# Get consent grants
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Consent to application'" -Top 50

# Export audit logs to CSV
$audits = Get-MgAuditLogDirectoryAudit -Filter "activityDateTime ge 2024-01-01" -All
$audits | Export-Csv "AuditLogs.csv" -NoTypeInformation
```

---

## KQL Queries for Log Analytics

```kusto
// All role assignments in the last 30 days
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "Add member to role"
| extend RoleName = tostring(TargetResources[0].displayName)
| extend UserAdded = tostring(TargetResources[1].userPrincipalName)
| extend AddedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, AddedBy, UserAdded, RoleName
| order by TimeGenerated desc

// Application consent activity
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName == "Consent to application"
| extend AppName = tostring(TargetResources[0].displayName)
| extend ConsentedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, ConsentedBy, AppName, Result

// User lifecycle events
AuditLogs
| where TimeGenerated > ago(7d)
| where Category == "UserManagement"
| where ActivityDisplayName in ("Add user", "Delete user", "Update user")
| extend TargetUser = tostring(TargetResources[0].userPrincipalName)
| extend Actor = coalesce(
    tostring(InitiatedBy.user.userPrincipalName),
    tostring(InitiatedBy.app.displayName),
    "System")
| project TimeGenerated, ActivityDisplayName, TargetUser, Actor, Result

// Conditional Access policy changes
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "Policy"
| where ActivityDisplayName has "conditional access"
| extend PolicyName = tostring(TargetResources[0].displayName)
| extend ModifiedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, ActivityDisplayName, PolicyName, ModifiedBy

// Password reset activity
AuditLogs
| where TimeGenerated > ago(7d)
| where ActivityDisplayName has "password"
| extend TargetUser = tostring(TargetResources[0].userPrincipalName)
| extend Actor = coalesce(
    tostring(InitiatedBy.user.userPrincipalName),
    "Self-service")
| project TimeGenerated, ActivityDisplayName, TargetUser, Actor
| order by TimeGenerated desc
```

---

## Audit Log Retention

| Scenario | Retention Period |
|----------|------------------|
| Free tier in portal | 7 days |
| Premium (P1/P2) in portal | 30 days |
| Exported to Log Analytics | Configurable (up to 2 years) |
| Exported to Storage Account | Unlimited |

### Best Practices for Retention

```yaml
Recommendations:
  1. Export to Log Analytics for querying:
     - Minimum 90 days for security investigations
     - 1-2 years for compliance requirements

  2. Archive to Storage for compliance:
     - Use lifecycle policies for cost management
     - Enable immutable storage for legal hold

  3. Set up alerts for critical events:
     - Role assignments
     - CA policy changes
     - Mass deletions
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of audit log categories and common events.

---

## Key Takeaways

1. Audit logs capture all directory changes including user, group, app, and role operations
2. Key fields include date, service, category, activity, status, initiated by, and target
3. Categories help organize events by resource type (UserManagement, RoleManagement, etc.)
4. Common high-priority events include role assignments, consent grants, and policy changes
5. Audit logs are essential for security investigations, compliance reporting, and change tracking

---

[<- 10.2 Sign-in Logs](../10.2-sign-in-logs/README.md) | [10.4 Diagnostic Settings ->](../10.4-diagnostic-settings/README.md)
