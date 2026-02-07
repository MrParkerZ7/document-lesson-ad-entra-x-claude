# Lesson 03: User and Group Management

## Overview

This lesson covers how to manage users and groups in Microsoft Entra ID. Effective user and group management is fundamental to identity governance and access control.

## Learning Objectives

By the end of this lesson, you will:
- Create and manage user accounts
- Understand different user types
- Create and manage groups
- Implement dynamic group membership
- Assign licenses through groups

---

## 1. User Types in Entra ID

### Member Users

Internal users belonging to your organization:
- Full access to directory resources
- Can be assigned any role
- Synced from on-premises or cloud-only

### Guest Users

External users invited for collaboration:
- Limited directory access by default
- B2B collaboration scenarios
- Managed through External Identities

### Comparison

| Aspect | Member | Guest |
|--------|--------|-------|
| UserType property | Member | Guest |
| Directory access | Full | Limited |
| Default permissions | Read all users | Read own profile |
| License assignment | Full support | Limited |
| Source | Internal | External invitation |

---

## 2. Creating Users

### Method 1: Azure Portal

```
Microsoft Entra ID → Users → New user → Create new user
```

#### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| User principal name | Login identifier | john.smith@contoso.com |
| Display name | Full name | John Smith |
| Password | Initial password | Auto-generate recommended |

#### Optional Fields

| Field | Description |
|-------|-------------|
| First name, Last name | Name components |
| Job title | Position |
| Department | Organizational unit |
| Manager | Reporting structure |
| Usage location | Required for licensing |

### Method 2: PowerShell

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Create a new user
$passwordProfile = @{
    Password = "SecureP@ssw0rd!"
    ForceChangePasswordNextSignIn = $true
}

$userParams = @{
    DisplayName = "John Smith"
    UserPrincipalName = "john.smith@contoso.com"
    MailNickname = "john.smith"
    PasswordProfile = $passwordProfile
    AccountEnabled = $true
    UsageLocation = "US"
    Department = "IT"
    JobTitle = "Systems Administrator"
}

New-MgUser @userParams
```

### Method 3: Bulk Operations

#### CSV Import Template

```csv
userPrincipalName,displayName,givenName,surname,department,jobTitle,usageLocation
alice.johnson@contoso.com,Alice Johnson,Alice,Johnson,Sales,Account Executive,US
bob.wilson@contoso.com,Bob Wilson,Bob,Wilson,Engineering,Developer,US
carol.davis@contoso.com,Carol Davis,Carol,Davis,HR,HR Specialist,US
```

#### Portal Bulk Operations

```
Microsoft Entra ID → Users → Bulk operations → Bulk create
```

---

## 3. User Properties and Attributes

### Standard Attributes

```yaml
Identity:
  - userPrincipalName: Primary login
  - mail: Email address
  - displayName: Full name
  - givenName: First name
  - surname: Last name

Job Information:
  - jobTitle: Position
  - department: Department
  - companyName: Company
  - employeeId: Employee number
  - employeeType: Employee category

Contact:
  - mobilePhone: Mobile number
  - businessPhones: Office numbers
  - streetAddress: Address
  - city: City
  - state: State
  - postalCode: ZIP code
  - country: Country

Settings:
  - usageLocation: Country code (required for licensing)
  - preferredLanguage: UI language
  - accountEnabled: Active/Disabled
```

### Extension Attributes

Custom attributes for organization-specific data:

```powershell
# Update extension attribute
Update-MgUser -UserId "john.smith@contoso.com" -AdditionalProperties @{
    "extension_<appId>_CostCenter" = "CC001"
    "extension_<appId>_EmployeeBadge" = "12345"
}
```

---

## 4. User Lifecycle Management

### Onboarding Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Create    │───▶│   Assign    │───▶│   Assign    │
│   User      │    │   Groups    │    │   Licenses  │
└─────────────┘    └─────────────┘    └─────────────┘
                           │
                           ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Configure  │◀───│   Assign    │◀───│   Send      │
│   Apps      │    │   Roles     │    │   Welcome   │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Offboarding Workflow

```powershell
# Step 1: Block sign-in
Update-MgUser -UserId $userId -AccountEnabled:$false

# Step 2: Revoke sessions
Revoke-MgUserSignInSession -UserId $userId

# Step 3: Remove from groups (optional - preserve for audit)
$groups = Get-MgUserMemberOf -UserId $userId
foreach ($group in $groups) {
    Remove-MgGroupMember -GroupId $group.Id -DirectoryObjectId $userId
}

# Step 4: Remove licenses
$licenses = Get-MgUserLicenseDetail -UserId $userId
Set-MgUserLicense -UserId $userId -RemoveLicenses $licenses.SkuId -AddLicenses @()

# Step 5: Convert to shared mailbox (if Exchange Online)
# Done through Exchange Admin Center

# Step 6: Delete or retain user
# Remove-MgUser -UserId $userId  # After retention period
```

### Soft Delete and Restore

```powershell
# Deleted users are retained for 30 days
Get-MgDirectoryDeletedItemAsUser

# Restore deleted user
Restore-MgDirectoryDeletedItem -DirectoryObjectId $deletedUserId
```

---

## 5. Group Types

### Security Groups

For access management:
- Application access control
- Resource permissions
- Conditional Access targeting
- Can be synced from on-premises

### Microsoft 365 Groups

For collaboration:
- Shared mailbox
- SharePoint site
- Teams team
- Planner board
- Cloud-only

### Group Comparison

| Feature | Security Group | Microsoft 365 Group |
|---------|---------------|-------------------|
| Purpose | Access control | Collaboration |
| Membership types | Assigned, Dynamic | Assigned, Dynamic |
| Mail-enabled | Optional | Yes |
| Owners | Optional | Required |
| Nested groups | Yes | Limited |
| Sync from AD | Yes | No |

---

## 6. Creating Groups

### Method 1: Azure Portal

```
Microsoft Entra ID → Groups → New group
```

### Configuration Options

| Setting | Options | Description |
|---------|---------|-------------|
| Group type | Security / Microsoft 365 | Purpose of group |
| Group name | String | Display name |
| Group email | String | For mail-enabled |
| Group description | String | Purpose documentation |
| Membership type | Assigned / Dynamic | How members are added |
| Owners | Users | Who manages the group |
| Members | Users/Groups | Initial membership |

### Method 2: PowerShell

```powershell
# Create Security Group
New-MgGroup -DisplayName "GRP-SEC-IT-Admins" `
    -Description "IT Administrators security group" `
    -MailEnabled:$false `
    -MailNickname "it-admins" `
    -SecurityEnabled:$true `
    -GroupTypes @()

# Create Microsoft 365 Group
New-MgGroup -DisplayName "GRP-M365-Project-Alpha" `
    -Description "Project Alpha team collaboration" `
    -MailEnabled:$true `
    -MailNickname "project-alpha" `
    -SecurityEnabled:$true `
    -GroupTypes @("Unified")
```

---

## 7. Dynamic Groups

### What are Dynamic Groups?

Groups where membership is automatically managed based on user/device attributes.

### Membership Rules Syntax

```
(user.property -operator "value")
```

### Common Operators

| Operator | Description | Example |
|----------|-------------|---------|
| -eq | Equals | `user.department -eq "Sales"` |
| -ne | Not equals | `user.department -ne "IT"` |
| -contains | Contains substring | `user.jobTitle -contains "Manager"` |
| -match | Regex match | `user.mail -match "@contoso.com$"` |
| -in | In list | `user.country -in ["US","CA","MX"]` |

### Example Dynamic Rules

#### All Users in IT Department

```
(user.department -eq "IT")
```

#### All Managers

```
(user.jobTitle -contains "Manager") or (user.jobTitle -contains "Director")
```

#### Users from Specific Countries

```
(user.country -eq "US") or (user.country -eq "Canada")
```

#### Users with Specific License

```
(user.assignedPlans -any (assignedPlan.servicePlanId -eq "guid" -and assignedPlan.capabilityStatus -eq "Enabled"))
```

#### Complex Rule - Full-time US Employees in Engineering

```
(user.department -eq "Engineering") and (user.country -eq "US") and (user.employeeType -eq "Full-Time")
```

### Creating Dynamic Group via Portal

```
Microsoft Entra ID → Groups → New group
1. Membership type: Dynamic user (or Dynamic device)
2. Click "Add dynamic query"
3. Build rule using UI or write advanced syntax
4. Click "Validate Rules" to test with sample users
5. Save and create group
```

### PowerShell Example

```powershell
$dynamicRule = '(user.department -eq "Sales") and (user.country -eq "US")'

New-MgGroup -DisplayName "GRP-DYN-US-Sales" `
    -Description "Dynamic group for US Sales team" `
    -MailEnabled:$false `
    -MailNickname "us-sales-dyn" `
    -SecurityEnabled:$true `
    -GroupTypes @("DynamicMembership") `
    -MembershipRule $dynamicRule `
    -MembershipRuleProcessingState "On"
```

---

## 8. Group-Based Licensing

### Benefits

- Automatic license assignment based on group membership
- Simplified license management
- Works with dynamic groups for automation
- Audit trail for compliance

### Setup Process

```
Microsoft Entra ID → Groups → [Select Group] → Licenses → Assignments
```

### Example: License Assignment by Department

```
Group: GRP-DYN-All-Employees
├── License: Microsoft 365 E3
│   └── Services: All enabled
│
Group: GRP-DYN-Developers
├── License: Visual Studio Enterprise
│   └── Services: Azure DevOps, Azure credits
│
Group: GRP-DYN-Executives
├── License: Microsoft 365 E5
    └── Services: All enabled (including Phone System)
```

### Handling License Conflicts

```powershell
# Check for license assignment errors
Get-MgGroup -GroupId $groupId -ExpandProperty memberswithLicenseErrors

# Common issues:
# - Not enough licenses available
# - Conflicting service plans
# - Missing usage location
```

### PowerShell License Assignment

```powershell
# Get SKU ID
$sku = Get-MgSubscribedSku | Where-Object { $_.SkuPartNumber -eq "ENTERPRISEPACK" }

# Assign license to group
Set-MgGroupLicense -GroupId $groupId -AddLicenses @(
    @{
        SkuId = $sku.SkuId
        DisabledPlans = @()  # Or specify plans to disable
    }
) -RemoveLicenses @()
```

---

## 9. Nested Groups

### Supported Nesting

```
GRP-SEC-All-IT
├── GRP-SEC-IT-Admins
│   ├── User: admin1@contoso.com
│   └── User: admin2@contoso.com
├── GRP-SEC-IT-Helpdesk
│   ├── User: help1@contoso.com
│   └── User: help2@contoso.com
└── GRP-SEC-IT-Developers
    ├── User: dev1@contoso.com
    └── User: dev2@contoso.com
```

### Limitations

| Scenario | Supported |
|----------|-----------|
| Security group in Security group | ✅ Yes |
| Microsoft 365 in Security group | ⚠️ Limited |
| Security in Microsoft 365 group | ❌ No |
| Dynamic groups nesting | ❌ No |
| Role-assignable group nesting | ❌ No |

---

## 10. Self-Service Group Management

### Enable Self-Service

```
Microsoft Entra ID → Groups → General settings
```

### Settings

| Setting | Description |
|---------|-------------|
| Self-service group management enabled | Allow users to create groups |
| Users can create security groups | Applies to security groups |
| Users can create Microsoft 365 groups | Applies to M365 groups |
| Group naming policy | Enforce naming conventions |
| Expiration policy | Auto-delete inactive groups |

### Naming Policy

```
Prefix: GRP-[GroupType]-
Suffix: -[Department]
Blocked words: [CEO, Admin, Payroll, etc.]

Example result: GRP-Project-Marketing-Sales
```

### Group Expiration

```
Microsoft Entra ID → Groups → Expiration
```

| Setting | Options |
|---------|---------|
| Group lifetime | 180, 365, or custom days |
| Email contact | Owners or specific email |
| Groups to expire | All or selected |

---

## 11. Best Practices

### User Management

- [ ] Use consistent naming conventions
- [ ] Set usage location at user creation
- [ ] Implement proper onboarding/offboarding workflows
- [ ] Regular access reviews
- [ ] Avoid direct license assignment (use groups)

### Group Management

- [ ] Use descriptive group names with prefixes
- [ ] Document group purposes in descriptions
- [ ] Prefer dynamic groups where possible
- [ ] Limit who can create groups
- [ ] Implement group expiration policies
- [ ] Regular membership audits

### Naming Convention Examples

```
Users:
  firstname.lastname@domain.com

Security Groups:
  GRP-SEC-[Purpose]-[Detail]
  Examples:
    GRP-SEC-APP-Salesforce-Users
    GRP-SEC-ROLE-Global-Readers
    GRP-SEC-DEPT-Engineering

Microsoft 365 Groups:
  GRP-M365-[Project/Team]
  Examples:
    GRP-M365-Project-Phoenix
    GRP-M365-Team-Marketing

Dynamic Groups:
  GRP-DYN-[Criteria]
  Examples:
    GRP-DYN-All-Employees
    GRP-DYN-US-Managers
```

---

## Summary

Effective user and group management enables:
- Streamlined identity lifecycle
- Automated access management
- Simplified license administration
- Scalable organizational structure

---

## Hands-On Exercise

1. Create 3 test users with different departments
2. Create a security group with assigned membership
3. Create a dynamic group based on department
4. Configure group-based licensing
5. Test the dynamic membership rule
6. Practice user disable/enable workflow

---

## Next Lesson

[Lesson 04: Authentication Methods →](../lesson-04-authentication/README.md)

---

## Additional Resources

- [Manage Users](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-manage-user-profile-info)
- [Manage Groups](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-manage-groups)
- [Dynamic Groups](https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership)
- [Group-Based Licensing](https://learn.microsoft.com/en-us/entra/fundamentals/concept-group-based-licensing)
