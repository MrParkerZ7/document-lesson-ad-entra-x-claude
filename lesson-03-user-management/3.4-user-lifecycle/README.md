# 3.4 User Lifecycle Management

## Overview

User lifecycle management covers the complete journey from onboarding to offboarding. Proper lifecycle management ensures security, compliance, and efficient administration.

## Learning Objectives

- Implement user onboarding workflows
- Execute secure offboarding procedures
- Manage deleted users and restoration
- Automate lifecycle with provisioning

---

## User Lifecycle Stages

```
┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
│  Pre-hire │───▶│ Onboard   │───▶│  Active   │───▶│ Offboard  │
│           │    │           │    │           │    │           │
└───────────┘    └───────────┘    └───────────┘    └───────────┘
      │                │                │                │
      ▼                ▼                ▼                ▼
  Identity         Configure        Maintain         Terminate
  Planning         Access           Access           Access
```

---

## Onboarding Workflow

### Step 1: Create User Account

```powershell
$newUser = @{
    DisplayName = "John Smith"
    UserPrincipalName = "john.smith@contoso.com"
    MailNickname = "john.smith"
    Department = "IT"
    JobTitle = "Systems Administrator"
    UsageLocation = "US"
    AccountEnabled = $true
    PasswordProfile = @{
        Password = "TempP@ss123!"
        ForceChangePasswordNextSignIn = $true
    }
}

$user = New-MgUser @newUser
```

### Step 2: Assign to Groups

```powershell
# Add to department group
New-MgGroupMember -GroupId $deptGroupId -DirectoryObjectId $user.Id

# Add to role group
New-MgGroupMember -GroupId $roleGroupId -DirectoryObjectId $user.Id
```

### Step 3: Assign Licenses

```powershell
# Assign license via group membership (preferred)
# Or directly:
Set-MgUserLicense -UserId $user.Id -AddLicenses @(
    @{ SkuId = $m365SkuId }
) -RemoveLicenses @()
```

### Step 4: Assign Roles (if needed)

```powershell
# Assign directory role
$roleParams = @{
    "@odata.type" = "#microsoft.graph.unifiedRoleAssignment"
    PrincipalId = $user.Id
    RoleDefinitionId = $roleId
    DirectoryScopeId = "/"
}
New-MgRoleManagementDirectoryRoleAssignment @roleParams
```

### Step 5: Send Welcome Email

```
Subject: Welcome to Contoso - Your Account Details

Your account has been created:
- Username: john.smith@contoso.com
- Temporary Password: [provided separately]
- MFA Setup: Required within 14 days

Next Steps:
1. Sign in at https://myapps.microsoft.com
2. Change your password
3. Set up MFA
4. Review assigned applications
```

---

## Offboarding Workflow

### Immediate Actions (Day 0)

```powershell
$userId = "john.smith@contoso.com"

# Step 1: Block sign-in immediately
Update-MgUser -UserId $userId -AccountEnabled:$false
Write-Host "✅ Account disabled"

# Step 2: Revoke all active sessions
Revoke-MgUserSignInSession -UserId $userId
Write-Host "✅ Sessions revoked"

# Step 3: Reset password (prevent known password use)
$newPassword = @{
    Password = [System.Web.Security.Membership]::GeneratePassword(16, 4)
    ForceChangePasswordNextSignIn = $true
}
Update-MgUser -UserId $userId -PasswordProfile $newPassword
Write-Host "✅ Password reset"
```

### Data Retention Actions

```powershell
# Step 4: Convert mailbox to shared (Exchange Online)
# Run in Exchange Online PowerShell:
# Set-Mailbox -Identity $userId -Type Shared

# Step 5: Grant manager access to mailbox
# Add-MailboxPermission -Identity $userId -User $managerId -AccessRights FullAccess

# Step 6: Set out-of-office reply
# Set-MailboxAutoReplyConfiguration -Identity $userId -AutoReplyState Enabled -InternalMessage "..."
```

### Cleanup Actions (After Retention Period)

```powershell
# Step 7: Remove from groups
$groups = Get-MgUserMemberOf -UserId $userId
foreach ($group in $groups) {
    if ($group.AdditionalProperties["@odata.type"] -eq "#microsoft.graph.group") {
        Remove-MgGroupMember -GroupId $group.Id -DirectoryObjectId $userId
    }
}
Write-Host "✅ Removed from all groups"

# Step 8: Remove licenses
$licenses = Get-MgUserLicenseDetail -UserId $userId
Set-MgUserLicense -UserId $userId -RemoveLicenses $licenses.SkuId -AddLicenses @()
Write-Host "✅ Licenses removed"

# Step 9: Delete user (after retention period)
Remove-MgUser -UserId $userId
Write-Host "✅ User deleted (soft delete)"
```

---

## Soft Delete and Restoration

### Deleted User Retention

| Phase | Duration | State |
|-------|----------|-------|
| Soft deleted | 30 days | Recoverable |
| Permanently deleted | After 30 days | Not recoverable |

### View Deleted Users

```powershell
# List all deleted users
Get-MgDirectoryDeletedItemAsUser | Select-Object DisplayName, UserPrincipalName, DeletedDateTime
```

### Restore Deleted User

```powershell
# Restore by ID
Restore-MgDirectoryDeletedItem -DirectoryObjectId $deletedUserId

# Verify restoration
Get-MgUser -UserId $restoredUserUpn
```

### Permanently Delete (Before 30 Days)

```powershell
# Permanently remove (cannot undo!)
Remove-MgDirectoryDeletedItem -DirectoryObjectId $deletedUserId
```

---

## Lifecycle Automation

### Lifecycle Workflows (Entra ID Governance)

```
Trigger: Employee hire date
├── Create account
├── Add to groups
├── Assign licenses
├── Send welcome email
└── Notify manager

Trigger: Employee termination date
├── Disable account
├── Revoke sessions
├── Remove from groups
├── Transfer data
└── Delete after 30 days
```

### HR-Driven Provisioning

```
HR System (Workday, SAP) → API → Entra ID Provisioning Service → User Created
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of user lifecycle.

---

## Key Takeaways

1. Onboarding should follow a consistent checklist
2. Offboarding requires immediate security actions
3. Deleted users are retained for 30 days
4. Automate lifecycle with workflows or HR integration
5. Document and audit all lifecycle changes

---

## Next Sub-Lesson

[3.5 Group Types →](../3.5-group-types/README.md)
