# 3.5 Group Types

## Overview

Microsoft Entra ID supports different group types for various purposes. Understanding when to use each type is essential for effective access management.

## Learning Objectives

- Differentiate between Security and Microsoft 365 groups
- Understand membership types (Assigned vs Dynamic)
- Create and configure groups
- Apply group best practices

---

## Group Types Overview

| Type | Purpose | Features |
|------|---------|----------|
| **Security Group** | Access control | Permissions, CA targeting, role assignment |
| **Microsoft 365 Group** | Collaboration | Mailbox, SharePoint, Teams, Planner |

---

## Security Groups

### Purpose

Primarily for managing access to resources:
- Application access control
- File share permissions
- Conditional Access targeting
- Role assignments
- License assignment

### Characteristics

| Feature | Value |
|---------|-------|
| Mail-enabled | Optional |
| Membership types | Assigned, Dynamic |
| Owners required | No |
| Nested groups | Yes |
| Sync from AD | Yes |
| Role-assignable | Optional (P1+) |

### Creating Security Group

```powershell
# Basic security group
New-MgGroup -DisplayName "GRP-SEC-IT-Admins" `
    -Description "IT Administrators security group" `
    -MailEnabled:$false `
    -MailNickname "it-admins" `
    -SecurityEnabled:$true `
    -GroupTypes @()

# Role-assignable security group (P1+)
New-MgGroup -DisplayName "GRP-SEC-ROLE-GlobalAdmins" `
    -Description "Global Admin role assignment group" `
    -MailEnabled:$false `
    -MailNickname "global-admins" `
    -SecurityEnabled:$true `
    -IsAssignableToRole:$true `
    -GroupTypes @()
```

---

## Microsoft 365 Groups

### Purpose

Collaboration and teamwork:
- Shared Exchange mailbox
- SharePoint document library
- Microsoft Teams team
- Planner board
- Yammer community

### Characteristics

| Feature | Value |
|---------|-------|
| Mail-enabled | Always |
| Membership types | Assigned, Dynamic |
| Owners required | Yes (1+) |
| Nested groups | Limited |
| Sync from AD | No |
| Visibility | Public or Private |

### Creating Microsoft 365 Group

```powershell
New-MgGroup -DisplayName "GRP-M365-Project-Alpha" `
    -Description "Project Alpha collaboration team" `
    -MailEnabled:$true `
    -MailNickname "project-alpha" `
    -SecurityEnabled:$true `
    -GroupTypes @("Unified") `
    -Visibility "Private"
```

---

## Membership Types

### Assigned Membership

Manual member management:

```powershell
# Add member
New-MgGroupMember -GroupId $groupId -DirectoryObjectId $userId

# Remove member
Remove-MgGroupMember -GroupId $groupId -DirectoryObjectId $userId

# List members
Get-MgGroupMember -GroupId $groupId
```

### Dynamic Membership

Automatic based on rules (covered in 3.6):

```powershell
New-MgGroup -DisplayName "GRP-DYN-All-Sales" `
    -MailEnabled:$false `
    -MailNickname "all-sales" `
    -SecurityEnabled:$true `
    -GroupTypes @("DynamicMembership") `
    -MembershipRule '(user.department -eq "Sales")' `
    -MembershipRuleProcessingState "On"
```

---

## Comparison Table

| Feature | Security Group | Microsoft 365 Group |
|---------|---------------|-------------------|
| **Purpose** | Access control | Collaboration |
| **Membership** | Assigned, Dynamic | Assigned, Dynamic |
| **Mail-enabled** | Optional | Always |
| **Exchange mailbox** | No | Yes |
| **SharePoint site** | No | Yes |
| **Teams integration** | No | Yes |
| **Owners** | Optional | Required |
| **Nested groups** | Yes | Limited |
| **Sync from AD** | Yes | No |
| **Role assignment** | Yes | No |
| **License assignment** | Yes | Yes |
| **CA targeting** | Yes | Yes |

---

## Group Naming Conventions

### Recommended Prefixes

```
Security Groups:
  GRP-SEC-[Purpose]-[Detail]
  GRP-SEC-APP-Salesforce-Users
  GRP-SEC-ROLE-Helpdesk-Admins
  GRP-SEC-DEPT-Engineering

Microsoft 365 Groups:
  GRP-M365-[Team/Project]
  GRP-M365-Project-Phoenix
  GRP-M365-Team-Marketing

Dynamic Groups:
  GRP-DYN-[Criteria]
  GRP-DYN-All-Employees
  GRP-DYN-US-Contractors
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual comparison of group types.

---

## Key Takeaways

1. Security groups are for access control
2. M365 groups are for collaboration
3. Dynamic groups automate membership
4. Use consistent naming conventions
5. Choose type based on use case

---

## Next Sub-Lesson

[3.6 Dynamic Groups →](../3.6-dynamic-groups/README.md)
