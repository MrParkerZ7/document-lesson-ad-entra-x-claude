# 2.7 Administrative Units

## Overview

Administrative Units (AUs) enable delegated administration by scoping admin permissions to specific subsets of users, groups, or devices. This supports organizational structures and least-privilege access.

## Learning Objectives

- Understand the purpose of Administrative Units
- Create and configure AUs
- Assign scoped roles to administrators
- Apply AU best practices

---

## What are Administrative Units?

AUs provide a container for identity objects that can be used to:

| Purpose | Description |
|---------|-------------|
| **Delegation** | Scope admin rights to specific objects |
| **Organizational structure** | Mirror departments, regions, subsidiaries |
| **Least privilege** | Limit admin access to what's needed |
| **Compliance** | Segregate administration duties |

---

## Administrative Unit Structure

```
Contoso Corporation
├── AU: IT Department
│   ├── Members: All IT users
│   └── Admins: IT Helpdesk (password reset)
├── AU: Sales Department
│   ├── Members: All Sales users
│   └── Admins: Sales Managers (group management)
├── AU: Europe Region
│   ├── Members: All EU users
│   └── Admins: EU IT Team (full user management)
└── AU: Contractors
    ├── Members: External contractors
    └── Admins: HR Team (limited management)
```

---

## Creating Administrative Units

### Via Portal

```
Microsoft Entra ID → Administrative units → New administrative unit
```

### Configuration

| Field | Description |
|-------|-------------|
| **Name** | Display name (e.g., "IT Department") |
| **Description** | Purpose and scope |
| **Restricted management** | Limit who can modify AU |

### Via PowerShell

```powershell
# Connect
Connect-MgGraph -Scopes "AdministrativeUnit.ReadWrite.All"

# Create AU
New-MgDirectoryAdministrativeUnit -DisplayName "IT Department" `
    -Description "IT department users and groups"

# Add members
$au = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq 'IT Department'"
$user = Get-MgUser -Filter "userPrincipalName eq 'john@contoso.com'"

New-MgDirectoryAdministrativeUnitMember -AdministrativeUnitId $au.Id `
    -DirectoryObjectId $user.Id
```

---

## Scoped Role Assignment

Assign roles that only apply within the AU:

### Available Scoped Roles

| Role | Permissions within AU |
|------|----------------------|
| **User Administrator** | Full user management |
| **Helpdesk Administrator** | Password reset only |
| **Groups Administrator** | Manage groups |
| **Authentication Administrator** | Manage auth methods |
| **License Administrator** | Manage licenses |

### Assignment Process

```
1. Navigate to AU → Roles and administrators
2. Select role (e.g., Helpdesk Administrator)
3. Add assignment → Select user
4. Admin now has role only within this AU
```

### Via PowerShell

```powershell
# Get AU and role definition
$au = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq 'IT Department'"
$role = Get-MgRoleManagementDirectoryRoleDefinition -Filter "displayName eq 'Helpdesk Administrator'"
$admin = Get-MgUser -Filter "userPrincipalName eq 'helpdesk@contoso.com'"

# Create scoped role assignment
New-MgRoleManagementDirectoryRoleAssignment `
    -DirectoryScopeId "/administrativeUnits/$($au.Id)" `
    -RoleDefinitionId $role.Id `
    -PrincipalId $admin.Id
```

---

## Dynamic Membership

AUs support dynamic membership rules (like dynamic groups):

```
Rule: (user.department -eq "IT")

Result: All users with department=IT automatically added
```

### Enable Dynamic Membership

```
AU Properties → Membership type → Dynamic User
Add dynamic query → Configure rule
```

---

## Restricted Management AUs

Prevent unauthorized modification:

| Setting | Effect |
|---------|--------|
| **Restricted** | Only AU admins can modify members |
| **Not restricted** | Any admin with rights can modify |

> **Note**: Use for sensitive groups like executives or security team.

---

## Use Cases

### Regional Delegation

```
AU: North America
├── Scope: Users with country = US, CA, MX
├── Admin: NA-IT-Team (User Administrator)
└── Permissions: Full user management for region
```

### Department Helpdesk

```
AU: Finance Department
├── Scope: Finance users
├── Admin: Finance-Helpdesk (Helpdesk Admin)
└── Permissions: Password reset only
```

### External User Management

```
AU: External Contractors
├── Scope: Guest users + contractors
├── Admin: Vendor-Management (User Admin)
└── Permissions: Manage external accounts
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Administrative Units.

---

## Key Takeaways

1. AUs enable scoped, delegated administration
2. Roles assigned at AU scope only apply to AU members
3. Dynamic membership automates AU population
4. Restricted AUs protect sensitive objects
5. Mirror organizational structure for effective delegation

---

## Lesson 02 Complete

You've completed Lesson 02! You now understand:
- Creating and configuring tenants
- Managing tenant properties
- Setting up custom domains
- Configuring company branding
- Tenant settings and security
- Administrative Units for delegation

---

## Next Lesson

[Lesson 03: User and Group Management →](../../lesson-03-user-management/README.md)
