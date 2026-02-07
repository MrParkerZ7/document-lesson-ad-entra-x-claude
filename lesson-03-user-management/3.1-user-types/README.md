# 3.1 User Types in Entra ID

## Overview

Microsoft Entra ID supports different user types to accommodate internal employees and external collaborators. Understanding user types is essential for proper access management and governance.

## Learning Objectives

- Understand the difference between Member and Guest users
- Know when to use each user type
- Configure appropriate permissions for each type

---

## User Types

### Member Users

Internal users belonging to your organization:

| Characteristic | Description |
|---------------|-------------|
| **UserType** | Member |
| **Source** | Cloud-only or synced from on-premises |
| **Directory Access** | Full read access by default |
| **Permissions** | Can be assigned any role |
| **Licensing** | Full license support |

**Common Scenarios:**
- Employees
- Contractors with internal accounts
- Service accounts

### Guest Users

External users invited for collaboration:

| Characteristic | Description |
|---------------|-------------|
| **UserType** | Guest |
| **Source** | External identity provider |
| **Directory Access** | Limited by default |
| **Permissions** | Restricted role assignments |
| **Licensing** | Limited licensing options |

**Common Scenarios:**
- B2B partners
- Vendors and consultants
- External collaborators

---

## Comparison Table

| Aspect | Member | Guest |
|--------|--------|-------|
| UserType property | Member | Guest |
| Directory access | Full | Limited |
| Default permissions | Read all users | Read own profile |
| Create groups | Yes (if allowed) | No (by default) |
| Role assignments | All roles | Limited roles |
| License assignment | Full support | Limited |
| Source | Internal | External invitation |
| MFA requirements | As configured | Can be enforced |

---

## Guest User Permissions

### Default Permissions

```
Guest users can:
├── Read their own profile
├── Read assigned groups
├── Access shared resources
└── Use assigned applications

Guest users cannot (by default):
├── Enumerate directory users
├── Read other user profiles
├── Create groups
├── Invite other guests
└── Access admin portals
```

### Configuring Guest Access

```
Microsoft Entra ID → User settings → External collaboration settings
```

| Setting | Options |
|---------|---------|
| Guest user access | Limited, Same as members, Restricted |
| Guest invite settings | Anyone, Admins only, Specific users |
| External user leave | Enable self-service |

---

## Converting User Types

### Member to Guest (Rare)

```powershell
# Not recommended - consider creating new guest account
Update-MgUser -UserId $userId -UserType "Guest"
```

### Guest to Member

```powershell
# Upgrade guest to member
Update-MgUser -UserId $userId -UserType "Member"
```

> **Note**: Converting user types affects permissions immediately.

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of user types.

---

## Key Takeaways

1. Member users are internal with full directory access
2. Guest users are external with limited permissions
3. Guest permissions are configurable at tenant level
4. UserType determines default capabilities
5. Both types can be targeted by Conditional Access

---

## Next Sub-Lesson

[3.2 Creating Users →](../3.2-creating-users/README.md)
