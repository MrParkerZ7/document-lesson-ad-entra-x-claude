# 8.2 Privileged Identity Management (PIM)

## Overview

Privileged Identity Management (PIM) provides just-in-time (JIT) privileged access to minimize standing administrative rights. Instead of permanent admin access, users receive eligible role assignments that they activate when needed, with automatic deactivation after a time limit.

## Learning Objectives

- Understand PIM concepts and benefits
- Configure role settings for Entra ID roles
- Assign eligible roles to users
- Activate roles as an end user
- Set up PIM for Azure resources and groups
- Review PIM audit history

---

## Why PIM?

### The Problem with Permanent Admin Access

```yaml
Traditional Admin Model:
  Issue: Permanent privileged access
  Risks:
    - Larger attack surface
    - No accountability for access
    - Forgotten privileges
    - Compromised admin = full access
    - No time boundaries
```

### PIM Solution

```
Traditional Model         Just-in-Time Model (PIM)
      │                           │
      ▼                           ▼
┌─────────────┐           ┌─────────────┐
│  Permanent  │           │   Eligible  │
│    Admin    │           │     Role    │
│  (Always)   │           │             │
└─────────────┘           └──────┬──────┘
      │                          │
   Full Access              Request Activation
   24/7/365                      │
      │                          ▼
      │                   ┌─────────────┐
      │                   │   Active    │
      │                   │    Role     │
      │                   │  (4 hours)  │
      │                   └──────┬──────┘
      │                          │
      │                   Auto-Deactivate
      │                          │
      │                          ▼
      │                   ┌─────────────┐
      │                   │  Eligible   │
      │                   │   Again     │
      │                   └─────────────┘
```

---

## Key Concepts

### Assignment Types

| Type | Description | Use Case |
|------|-------------|----------|
| Eligible | Can activate role when needed | Primary PIM usage |
| Active | Currently has the role | Emergency/service accounts |
| Time-bound | Assignment with expiration | Temporary access |
| Permanent | No expiration (not recommended) | Break-glass only |

### Role States

```yaml
Eligible:
  - User can activate
  - No current access
  - Must request when needed

Active:
  - Currently has permissions
  - Time-limited
  - Auto-deactivates

Expired:
  - Assignment ended
  - No longer eligible
  - Must be reassigned
```

---

## Accessing PIM

### Portal Navigation

```
Microsoft Entra ID → Identity Governance → Privileged Identity Management
```

### PIM Sections

| Section | Purpose |
|---------|---------|
| My roles | User's own eligible/active roles |
| Microsoft Entra roles | Directory role management |
| Azure resources | Azure RBAC role management |
| Groups | Role-assignable group management |
| Approve requests | Pending activation approvals |
| Access reviews | PIM-related access reviews |

---

## Configuring Entra ID Roles

### Access Role Settings

```
PIM → Microsoft Entra roles → Roles → [Select Role] → Settings
```

### Activation Settings

```yaml
Role: Global Administrator

Activation:
  Maximum activation duration: 4 hours
  Require Azure MFA: Yes
  Require justification: Yes
  Require ticket information: Optional
  Require approval: Yes

Approvers:
  Primary: Security-Admins@contoso.com
  Fallback: IT-Directors@contoso.com
```

### Assignment Settings

```yaml
Assignment:
  Allow permanent eligible assignment: No
  Expire eligible assignments after: 12 months
  Allow permanent active assignment: No (strongly recommended)
  Require Azure MFA on active assignment: Yes
  Require justification on active assignment: Yes
```

### Notification Settings

```yaml
Notifications:
  Send notifications when members are assigned as eligible:
    Role assignee: Yes
    Assigned admin: Yes

  Send notifications when members are assigned as active:
    Role assignee: Yes
    Assigned admin: Yes
    All admins: Yes

  Send notifications when eligible members activate:
    Role assignee: Yes
    Approvers: Yes
    Assigned admin: Yes
```

---

## Role Settings by Sensitivity

### Highly Privileged Roles

```yaml
Roles:
  - Global Administrator
  - Privileged Role Administrator
  - Security Administrator
  - Exchange Administrator

Settings:
  Activation duration: 1-4 hours
  Require approval: Yes
  Require MFA: Yes
  Require justification: Yes
  Eligible assignment: 6 months max
  Active assignment: Not allowed
```

### Moderately Privileged Roles

```yaml
Roles:
  - User Administrator
  - Application Administrator
  - Groups Administrator
  - Helpdesk Administrator

Settings:
  Activation duration: 4-8 hours
  Require approval: Optional
  Require MFA: Yes
  Require justification: Yes
  Eligible assignment: 12 months max
```

### Operational Roles

```yaml
Roles:
  - Directory Readers
  - Reports Reader
  - Message Center Reader

Settings:
  Activation duration: 8 hours
  Require approval: No
  Require MFA: Optional
  Require justification: Yes
  Eligible assignment: Permanent allowed
```

---

## Assigning Eligible Roles

### Portal Assignment

```
PIM → Microsoft Entra roles → Roles → [Role] → Add assignments
```

### Assignment Configuration

```yaml
Step 1 - Select members:
  Members: john.smith@contoso.com

Step 2 - Settings:
  Assignment type: Eligible
  Assignment start: Immediately
  Assignment end: 12 months from now

Step 3 - Justification:
  Reason: IT Administrator requiring Global Admin for emergency tasks
```

### PowerShell Assignment

```powershell
Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory"

# Get the role definition
$role = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayName eq 'Global Administrator'"

# Get the user
$user = Get-MgUser -Filter "userPrincipalName eq 'john.smith@contoso.com'"

# Create eligible assignment
$params = @{
    Action = "adminAssign"
    PrincipalId = $user.Id
    RoleDefinitionId = $role.Id
    DirectoryScopeId = "/"
    Justification = "Admin access for IT operations"
    ScheduleInfo = @{
        StartDateTime = (Get-Date)
        Expiration = @{
            Type = "AfterDuration"
            Duration = "P365D"  # 365 days
        }
    }
}

New-MgRoleManagementDirectoryRoleEligibilityScheduleRequest -BodyParameter $params
```

---

## Activating Roles (User Experience)

### Portal Activation

```
PIM → My roles → Eligible assignments → [Role] → Activate
```

### Activation Steps

```yaml
Step 1 - Scope:
  Scope: Directory (or specific administrative unit)

Step 2 - Duration:
  Duration: 2 hours (up to max allowed)

Step 3 - Justification:
  Reason: Need to create new admin account for helpdesk
  Ticket: INC-12345 (if required)

Step 4 - MFA:
  Complete MFA challenge

Step 5 - Wait for approval (if required)
```

### Activation Flow

```
User requests activation
         ↓
Provide justification + ticket
         ↓
Complete MFA verification
         ↓
    ┌────┴────┐
    ▼         ▼
Approval    No Approval
Required    Required
    │            │
    ▼            │
Wait for         │
Approval         │
    │            │
    └─────┬──────┘
          ▼
    Role Activated
          │
    Timer starts
          │
          ▼
   Auto-deactivates
```

### PowerShell Activation

```powershell
# Self-activate a role
$params = @{
    Action = "selfActivate"
    PrincipalId = (Get-MgContext).Account
    RoleDefinitionId = $role.Id
    DirectoryScopeId = "/"
    Justification = "Creating new admin account per ticket INC-12345"
    ScheduleInfo = @{
        StartDateTime = (Get-Date)
        Expiration = @{
            Type = "AfterDuration"
            Duration = "PT4H"  # 4 hours
        }
    }
}

New-MgRoleManagementDirectoryRoleAssignmentScheduleRequest -BodyParameter $params
```

---

## Approving Activations

### Approver Experience

```
PIM → Approve requests → [Pending request]
```

### Review and Decision

```yaml
Request Details:
  Requestor: john.smith@contoso.com
  Role: Global Administrator
  Duration: 4 hours
  Justification: Emergency user account issue

Actions:
  - Approve (with reason)
  - Deny (with reason)
```

### Email Notification

Approvers receive email with:
- Requestor name
- Role requested
- Duration
- Justification
- Approve/Deny links

---

## PIM for Azure Resources

### Enable PIM for Subscription

```
PIM → Azure resources → Discover resources → [Subscription] → Manage resource
```

### Configure Azure Role Settings

```
PIM → Azure resources → [Subscription] → Roles → [Role] → Settings
```

### Common Azure Roles

```yaml
High Privilege:
  - Owner
  - User Access Administrator
  - Contributor (with sensitive resources)

Settings:
  Similar to Entra ID roles
  Scope can be subscription, resource group, or resource
```

### Assigning Azure Roles

```yaml
Role: Contributor
Scope: /subscriptions/{subscription-id}/resourceGroups/Production
Member: alice.developer@contoso.com
Assignment: Eligible
Duration: 6 months
```

---

## PIM for Groups

### Enable Group for PIM

Requirements:
- Group must be role-assignable
- Or security group assigned to Azure roles

```
PIM → Groups → [Group] → Settings
```

### Group PIM Settings

```yaml
Member settings:
  Allow permanent eligible: No
  Allow permanent active: No
  Require MFA on activation: Yes
  Require justification: Yes
  Require approval: Yes
  Approvers: Group owners

Owner settings:
  Similar configuration
```

### Use Cases

```yaml
Group-based access:
  Group: GRP-SEC-SQL-Admins
  Assigned to: SQL Server Contributor role

  User becomes eligible for group membership
  Activates group membership → Gets SQL access
  Deactivates → Loses SQL access
```

---

## Audit and Reporting

### Access Audit History

```
PIM → Microsoft Entra roles → Audit history
```

### Audit Events

| Event | Description |
|-------|-------------|
| Add member to role | Eligible assignment created |
| Remove member from role | Assignment removed |
| Activate role | User activated eligible role |
| Deactivate role | User deactivated or expired |
| Update role setting | Settings changed |

### Export Audit Logs

```powershell
# Get PIM audit logs
Get-MgAuditLogDirectoryAudit -Filter "category eq 'RoleManagement'" -Top 100 |
    Select-Object ActivityDateTime, ActivityDisplayName,
        @{N='User';E={$_.InitiatedBy.User.UserPrincipalName}},
        @{N='Target';E={$_.TargetResources[0].DisplayName}}
```

### KQL Queries for PIM

```kusto
// Role activations in last 7 days
AuditLogs
| where TimeGenerated > ago(7d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "activation"
| project TimeGenerated, InitiatedBy.user.userPrincipalName,
    TargetResources[0].displayName, Result

// Failed activation requests
AuditLogs
| where Category == "RoleManagement"
| where Result == "failure"
| where ActivityDisplayName has "Activate"
| summarize Count = count() by InitiatedBy.user.userPrincipalName
```

---

## Access Reviews for PIM

### Create PIM Access Review

```
PIM → Microsoft Entra roles → Access reviews → New
```

### Configuration

```yaml
Review type: Microsoft Entra roles

Roles to review:
  - Global Administrator
  - Privileged Role Administrator

Review scope:
  - Eligible assignments only
  - Active assignments only
  - Both

Reviewers: Role assignees (self-review) or specific users

Settings:
  Recurrence: Quarterly
  Duration: 14 days
  Auto-apply: Yes
  If no response: Remove access
```

---

## Best Practices

### Configuration

```yaml
Do:
  - Require approval for Global Admin
  - Enable MFA on all activations
  - Set short activation durations
  - Require justification always
  - Set assignment expiration
  - Configure notifications

Don't:
  - Allow permanent active assignments
  - Set long activation durations
  - Skip approval for sensitive roles
  - Ignore audit logs
```

### Operational

```yaml
Recommendations:
  - Review eligible assignments quarterly
  - Monitor activation patterns
  - Investigate unusual activations
  - Maintain emergency access accounts
  - Train users on activation process
  - Document approval procedures
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- PIM activation flow
- Role assignment lifecycle
- Approval workflow
- Audit trail

---

## Key Takeaways

- PIM eliminates standing privileged access
- Users have eligible assignments, not permanent access
- Activations are time-bound and audited
- Approval can be required for sensitive roles
- MFA should always be required for activation
- Access reviews ensure ongoing governance
- PIM works for Entra roles, Azure roles, and groups

---

## Navigation

[← 8.1 Identity Protection](../8.1-identity-protection/README.md) | [8.3 Access Reviews →](../8.3-access-reviews/README.md)
