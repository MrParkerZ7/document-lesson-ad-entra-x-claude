# 2.5 Tenant Settings

## Overview

Tenant settings control organization-wide behaviors for users, applications, and external collaboration. This sub-lesson covers essential settings for securing and configuring your tenant.

## Learning Objectives

- Configure user settings
- Set up external collaboration policies
- Manage guest access restrictions
- Apply recommended configurations

---

## User Settings

```
Microsoft Entra ID → User settings
```

### Key Settings

| Setting | Options | Recommendation |
|---------|---------|----------------|
| **App registrations** | Users can register apps | No (restrict) |
| **Admin portal access** | Restrict non-admins | Yes (restrict) |
| **LinkedIn connections** | Allow account linking | Based on policy |
| **Show keep user signed in** | Show KMSI prompt | Yes |

---

## Guest User Settings

```
Microsoft Entra ID → User settings → Manage external collaboration settings
```

### Guest Access Levels

| Level | Permissions |
|-------|-------------|
| **Same as members** | Full directory access |
| **Limited access** | Can see own profile, limited directory |
| **Most restrictive** | Own profile only, no enumeration |

### Recommended Configuration

```yaml
Guest user access restrictions:
  Level: Most restrictive

Guest users can:
  - See own profile: Yes
  - See other users: No
  - Enumerate groups: No
  - Read directory: No
```

---

## External Collaboration Settings

### Invitation Settings

| Setting | Description |
|---------|-------------|
| **Admin and guest inviter** | Admins and designated users |
| **Member users** | Allow/deny member invitations |
| **Guests** | Allow/deny guest invitations |

### Recommended Policy

```yaml
Who can invite:
  - Admins: Yes
  - Guest inviter role: Yes
  - Members: No (or limited)
  - Guests: No
```

### Domain Restrictions

Control which domains can be invited:

**Allow list (recommended for security):**
```yaml
Collaboration restrictions: Allow
Allowed domains:
  - partner1.com
  - supplier.com
  - contractor.org
```

**Deny list:**
```yaml
Collaboration restrictions: Deny
Blocked domains:
  - competitor.com
  - untrusted.org
```

---

## Application Settings

### User Consent for Apps

```
Microsoft Entra ID → Enterprise applications → Consent and permissions
```

| Setting | Options |
|---------|---------|
| **User consent** | Allow / Allow for verified / Disable |
| **Group owner consent** | Enable / Disable |
| **Admin consent workflow** | Enable / Disable |

### Recommended Settings

```yaml
User consent for applications:
  - Allow user consent: No (disable)
  - Admin consent workflow: Yes (enable)
  - Consent request reviewers: IT-Security-Group

Result: Users request access, admins approve
```

---

## Device Settings

```
Microsoft Entra ID → Devices → Device settings
```

| Setting | Description |
|---------|-------------|
| **Users may join devices** | Who can Azure AD join |
| **Additional local admins** | Device admin permissions |
| **Require MFA for device join** | Security requirement |
| **Maximum devices per user** | Limit registrations |

---

## PowerShell Configuration

```powershell
# Get current authorization policy
Get-MgPolicyAuthorizationPolicy | Select-Object -ExpandProperty DefaultUserRolePermissions

# Update guest restrictions
$params = @{
    GuestUserRoleId = "2af84b1e-32c8-42b7-82bc-daa82404023b"  # Restricted guest
}
Update-MgPolicyAuthorizationPolicy -AuthorizationPolicyId "authorizationPolicy" -BodyParameter $params

# Configure external collaboration
Update-MgPolicyAuthorizationPolicy `
    -AllowInvitesFrom "adminsAndGuestInviters" `
    -AllowedToSignUpEmailBasedSubscriptions $false
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual overview of tenant settings.

---

## Key Takeaways

1. Restrict app registration to prevent shadow IT
2. Use most restrictive guest access by default
3. Control who can invite external users
4. Enable admin consent workflow for apps
5. Limit user self-service capabilities

---

## Next Sub-Lesson

[2.6 Security Defaults →](../2.6-security-defaults/README.md)
